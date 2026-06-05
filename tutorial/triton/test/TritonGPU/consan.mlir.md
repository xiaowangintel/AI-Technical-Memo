# consan.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/consan.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritoninstrument-prepare-consan-captures="target=nvidia", -tritoninstrument-concurrency-sanitizer` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritoninstrument-prepare-consan-captures="target=nvidia", -tritoninstrument-concurrency-sanitizer` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=nvidia" -tritoninstrument-concurrency-sanitizer | FileCheck %s`; `// RUN: env TRITON_CONSAN_INIT_ALLOCATIONS=0 triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=nvidia" -tritoninstrument-concurrency-sanitizer | FileCheck %s --check-prefix=NO-INIT`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=nvidia" -tritoninstrument-concurrency-sanitizer | FileCheck %s`；`// RUN: env TRITON_CONSAN_INIT_ALLOCATIONS=0 triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=nvidia" -tritoninstrument-concurrency-sanitizer | FileCheck %s --check-prefix=NO-INIT`

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=nvidia" -tritoninstrument-concurrency-sanitizer | FileCheck %s
2| // RUN: env TRITON_CONSAN_INIT_ALLOCATIONS=0 triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=nvidia" -tritoninstrument-concurrency-sanitizer | FileCheck %s --check-prefix=NO-INIT
3| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=nvidia" -tritoninstrument-concurrency-sanitizer | FileCheck %s` ; ` env TRITON_CONSAN_INIT_ALLOCATIONS=0 triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=nvidia" -tritoninstrument-concurrency-sanitizer | FileCheck %s --check-prefix=NO-INIT` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=nvidia" -tritoninstrument-concurrency-sanitizer | FileCheck %s`；` env TRITON_CONSAN_INIT_ALLOCATIONS=0 triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=nvidia" -tritoninstrument-concurrency-sanitizer | FileCheck %s --check-prefix=NO-INIT`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 4-7
```mlir
4| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
5| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
6| #smem = #ttg.shared_memory
7| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 8-8
```mlir
8| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 9-12
```mlir
 9|   // CHECK-DAG: #[[BUFS_L:.*]] = #ttg.linear<{register = [], lane = {{\[}}[0], [0], [0], [0], [0]], warp = [], block = []}>
10|   // CHECK-DAG: #[[BUFS_THREADS_L:.*]] = #ttg.linear<{register = [], lane = {{\[}}[0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0], [0, 0, 0]], warp = [], block = []}>
11|   // CHECK-DAG: #[[BUFS_BARS_L:.*]] = #ttg.linear<{register = [], lane = {{\[}}[0, 0], [0, 0], [0, 0], [0, 0], [0, 0]], warp = [], block = []}>
12|   // CHECK: @single_local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 13-13
```mlir
13|   tt.func public @single_local_alloc() {
```
**EN:** This function-oriented block defines or enters `single_local_alloc`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `single_local_alloc` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 14-31
```mlir
14|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64, #{{.*}}>
15| 
16|     // CHECK: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
17|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
18| 
19|     // CHECK: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
20|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
21| 
22|     // CHECK: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
23|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
24| 
25|     // CHECK: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
26|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
27|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
28|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
29|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
30|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
31|     tt.return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 32-34
```mlir
32|   }
33| }
34| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 35-35
```mlir
35| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 36-40
```mlir
36| 
37| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0], CGALayout = [[1, 0]]}>
38| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
39| #smem = #ttg.shared_memory
40| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1], CGALayout = [[1, 0]]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 41-41
```mlir
41| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 42-43
```mlir
42|   // CHECK: tti.experimental_cluster_cta_id : i32
43|   // CHECK-LABEL: @single_local_alloc_multi_cta
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @single_local_alloc_multi_cta anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @single_local_alloc_multi_cta 这样的标签用于锚定匹配范围。

### Lines 44-44
```mlir
44|   tt.func public @single_local_alloc_multi_cta() {
```
**EN:** This function-oriented block defines or enters `single_local_alloc_multi_cta`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `single_local_alloc_multi_cta` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 45-62
```mlir
45|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
46|     // CHECK: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
47|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
48|     // CHECK: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 64 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
49|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
50|     // CHECK: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 4 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
51|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
52|     // CHECK: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 64 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
53|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
54|     // CHECK-NOT: publish_cluster_visibility
55|     // CHECK: tt.return
56|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
57|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
58|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
59|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
60|     tt.return
61|   }
62| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 64-65
```mlir
64| 
65| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 65-69
```mlir
65| 
66| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
67| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
68| #smem = #ttg.shared_memory
69| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 70-70
```mlir
70| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 71-71
```mlir
71|   // CHECK-LABEL: @two_local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @two_local_alloc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @two_local_alloc 这样的标签用于锚定匹配范围。

### Lines 72-72
```mlir
72|   tt.func public @two_local_alloc() {
```
**EN:** This function-oriented block defines or enters `two_local_alloc`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_local_alloc` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 73-90
```mlir
73|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0, 4096], [{{.*}}], shared_mem : tensor<2xi64,
74| 
75|     // CHECK: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
76|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
77| 
78|     // CHECK: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
79|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
80| 
81|     // CHECK: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 2 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
82|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
83| 
84|     // CHECK: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
85|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
86|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
87|     %1 = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
88|     %bar = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
89|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
90|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier, local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier、本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 91-95
```mlir
91|     ttg.local_load %1 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
92|     tt.return
93|   }
94| }
95| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 96-96
```mlir
96| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 97-101
```mlir
 97| 
 98| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
 99| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
100| #smem = #ttg.shared_memory
101| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 102-102
```mlir
102| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 103-103
```mlir
103|   // CHECK-LABEL: @three_local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @three_local_alloc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @three_local_alloc 这样的标签用于锚定匹配范围。

### Lines 104-104
```mlir
104|   tt.func public @three_local_alloc() {
```
**EN:** This function-oriented block defines or enters `three_local_alloc`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `three_local_alloc` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 105-122
```mlir
105|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0, 4096, 8192, 0], [{{.*}}], shared_mem : tensor<4xi64,
106| 
107|     // CHECK: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
108|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
109| 
110|     // CHECK: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
111|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
112| 
113|     // CHECK: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 4 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
114|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
115| 
116|     // CHECK: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
117|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
118|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
119|     %1 = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
120|     %2 = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
121|     %bar = ttg.local_alloc {allocation.offset = 12288 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
122|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 123-129
```mlir
123|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
124|     ttg.local_load %1 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
125|     ttg.local_load %2 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
126|     tt.return
127|   }
128| }
129| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 130-130
```mlir
130| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 131-135
```mlir
131| 
132| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
133| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
134| #smem = #ttg.shared_memory
135| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 136-136
```mlir
136| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 137-137
```mlir
137|   // CHECK-LABEL: @three_sub_bufs
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @three_sub_bufs anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @three_sub_bufs 这样的标签用于锚定匹配范围。

### Lines 138-138
```mlir
138|   tt.func public @three_sub_bufs() {
```
**EN:** This function-oriented block defines or enters `three_sub_bufs`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `three_sub_bufs` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 139-156
```mlir
139|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0, 4096, 8192, 0], [{{.*}}], shared_mem : tensor<4xi64,
140| 
141|     // CHECK: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
142|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
143| 
144|     // CHECK: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
145|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
146| 
147|     // CHECK: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 4 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
148|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
149| 
150|     // CHECK: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
151|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
152|     %c0_i32 = arith.constant 0 : i32
153|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<3x32x32xf32, #shared, #smem, mutable>
154|     %1 = ttg.memdesc_index %0[%c0_i32] : !ttg.memdesc<3x32x32xf32, #shared, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
155|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
156|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, constants, ttg.memdesc_index, ttng.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、常量、ttg.memdesc_index、ttng.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 157-161
```mlir
157|     ttg.local_load %1 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
158|     tt.return
159|   }
160| }
161| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 162-162
```mlir
162| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 163-167
```mlir
163| 
164| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
165| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
166| #smem = #ttg.shared_memory
167| #blocked = #ttg.blocked<{sizePerThread = [2, 4], threadsPerWarp = [1, 32], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 168-168
```mlir
168| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 169-170
```mlir
169|   // CHECK: #[[READ_BARS_L:.*]] = #ttg.blocked<{sizePerThread = [2, 4], threadsPerWarp = [1, 32], warpsPerCTA = [1, 1], order = [0, 1]}>
170|   // CHECK: @read_bars_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 171-171
```mlir
171|   tt.func public @read_bars_alloc() {
```
**EN:** This function-oriented block defines or enters `read_bars_alloc`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `read_bars_alloc` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 172-184
```mlir
172|     // CHECK: %[[READ_BARS_G:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
173|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_BARS_G]], %c0_i8
174|     %c0 = arith.constant 0 : i32
175|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<2x32x32xf32, #shared, #smem, mutable>
176|     %bar = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<4x1xi64, #shared1, #smem, mutable>
177|     %bar_sub = ttg.memdesc_index %bar[%c0] : !ttg.memdesc<4x1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
178|     ttng.init_barrier %bar_sub, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
179|     %buf_sub = ttg.memdesc_index %0[%c0] : !ttg.memdesc<2x32x32xf32, #shared, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
180|     ttg.local_load %buf_sub : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
181|     tt.return
182|   }
183| }
184| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttg.memdesc_index, constants, ttng.init_barrier, local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttg.memdesc_index、常量、ttng.init_barrier、本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 185-185
```mlir
185| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 186-190
```mlir
186| 
187| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
188| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
189| #smem = #ttg.shared_memory
190| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 191-191
```mlir
191| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 192-193
```mlir
192|   // CHECK: #[[BUFS_L:.*]] = #ttg.linear<{register = [], lane = {{\[}}[0], [0], [0], [0], [0]], warp = [], block = []}>
193|   // CHECK: @tmem_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 194-194
```mlir
194|   tt.func public @tmem_alloc() {
```
**EN:** This function-oriented block defines or enters `tmem_alloc`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_alloc` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 195-204
```mlir
195|     // CHECK-DAG: %[[TMEM_BUFS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], tensor_mem : tensor<1xi64, #{{.*}}>
196|     // CHECK-DAG: %[[BARRIERS:.*]] = tti.experimental_buffer_descriptors [4096], [{{.*}}], shared_mem : tensor<1xi64, #{{.*}}>
197|     %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
198|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
199|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
200|     ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32>
201|     tt.return
202|   }
203| }
204| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, shared/local memory allocation, ttng.init_barrier, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、共享/本地内存分配、ttng.init_barrier、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 205-205
```mlir
205| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 206-209
```mlir
206| 
207| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
208| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
209| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 210-210
```mlir
210| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 211-211
```mlir
211|   // CHECK-LABEL: @async_tma_copy_global_to_local
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tma_copy_global_to_local anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tma_copy_global_to_local 这样的标签用于锚定匹配范围。

### Lines 212-212
```mlir
212|   tt.func public @async_tma_copy_global_to_local(%arg0: !tt.tensordesc<32x32xf32, #shared>) {
```
**EN:** This function-oriented block defines or enters `async_tma_copy_global_to_local`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_copy_global_to_local` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 213-230
```mlir
213|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
214| 
215|     // CHECK-DAG: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
216|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
217| 
218|     // CHECK-DAG: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
219|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
220| 
221|     // CHECK-DAG: %[[BARRIERS:.*]] = tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem : tensor<1xi64
222|     // CHECK-DAG: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
223|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
224| 
225|     // CHECK-DAG: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
226|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
227|     %true = arith.constant true
228|     %c0_i32 = arith.constant 0 : i32
229|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
230|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 231-235
```mlir
231|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
232|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
233|     // Model the async TMA completion mechanism: barrier_expect corresponds to
234|     // mbarrier.arrive.expect_tx and is what should update ConSan's barrier state.
235|     ttng.barrier_expect %bar, 4096, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.init_barrier, ttng.barrier_expect. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.init_barrier、ttng.barrier_expect。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 236-253
```mlir
236|     // CHECK: tt.call @__triton_consan_init_barrier_state
237|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
238|     // CHECK: tt.call @__triton_consan_track_visible_writes
239|     // CHECK: tt.call @__triton_consan_track_visible_reads
240|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
241|     // CHECK: tt.call @__triton_consan_update_barrier_state
242|     // CHECK: tt.call @__triton_consan_verify_write_visibility
243|     // CHECK: tt.call @__triton_consan_verify_read_visibility
244|     // CHECK: tt.call @__triton_consan_set_write_visibility
245|     // CHECK: tt.call @__triton_consan_clear_write_tracking
246|     // CHECK: tt.call @__triton_consan_clear_read_visibility
247|     // CHECK: tt.call @__triton_consan_clear_read_tracking
248|     // CHECK: tt.call @__triton_consan_track_barrier_write_for_buffer
249|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
250|     // CHECK: tt.call @__triton_consan_update_barrier_state
251|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32] %0, %bar, %true : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
252|     tt.return
253|   }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 254-255
```mlir
254| }
255| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 256-256
```mlir
256| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 257-260
```mlir
257| 
258| #shared_clc = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
259| #barrier = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
260| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 261-261
```mlir
261| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 262-262
```mlir
262|   // CHECK-LABEL: @clc_try_cancel_diagonal_effect_recipients
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @clc_try_cancel_diagonal_effect_recipients anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @clc_try_cancel_diagonal_effect_recipients 这样的标签用于锚定匹配范围。

### Lines 263-268
```mlir
263|   tt.func public @clc_try_cancel_diagonal_effect_recipients() {
264|     %true = arith.constant true
265|     %c0_i32 = arith.constant 0 : i32
266|     %result = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<2xi64, #shared_clc, #smem, mutable>
267|     %bar = ttg.local_alloc {allocation.offset = 16 : i32} : () -> !ttg.memdesc<1xi64, #barrier, #smem, mutable>
268|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #barrier, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `clc_try_cancel_diagonal_effect_recipients`. Within it, the test exercises tt.func, constants, shared/local memory allocation, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `clc_try_cancel_diagonal_effect_recipients` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 269-278
```mlir
269|     // CHECK: tti.experimental_cluster_cta_id
270|     // CHECK: arith.cmpi eq
271|     // CHECK: %[[CLC_THREAD:.*]] = arith.constant 1 : i32
272|     // CHECK: tt.call @__triton_consan_verify_write_visibility{{.*}}%[[CLC_THREAD]]
273|     // CHECK: %[[CLC_MASK:.*]] = arith.constant 2 : i64
274|     // CHECK: tt.call @__triton_consan_set_write_visibility{{.*}}%[[CLC_MASK]]
275|     // CHECK: tt.call @__triton_consan_track_barrier_write_for_buffer{{.*}}_I1
276|     ttng.clc_try_cancel %result, %bar : !ttg.memdesc<2xi64, #shared_clc, #smem, mutable>, !ttg.memdesc<1xi64, #barrier, #smem, mutable>
277|     ttng.barrier_expect %bar, 16, %true : !ttg.memdesc<1xi64, #barrier, #smem, mutable>
278|     ttng.wait_barrier %bar, %c0_i32, %true : !ttg.memdesc<1xi64, #barrier, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.clc_try_cancel, ttng.barrier_expect, ttng.wait_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.clc_try_cancel、ttng.barrier_expect、ttng.wait_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 279-285
```mlir
279|     // CHECK: tt.call @__triton_consan_verify_write_visibility
280|     // CHECK: ttng.clc_load_result
281|     %clc_res = ttng.clc_load_result %result : !ttg.memdesc<2xi64, #shared_clc, #smem, mutable> -> i128
282|     tt.return
283|   }
284| }
285| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.clc_load_result, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.clc_load_result、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 286-286
```mlir
286| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 287-291
```mlir
287| 
288| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32, CGALayout = [[0, 0]]}>
289| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
290| #smem = #ttg.shared_memory
291| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1], CGALayout = [[0, 0]]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 292-292
```mlir
292| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 293-298
```mlir
293|   // CHECK-LABEL: tt.func private @__triton_consan_check_outstanding_commits_noalias{{.*}}T2x1x1xI8
294|   // CHECK-SAME: %arg6: i32
295|   // CHECK-NOT: tti.experimental_cluster_cta_id
296|   // CHECK: tt.splat %arg6 : i32 -> tensor<2x1x1xi32
297|   // CHECK: arith.shrui
298|   // CHECK-LABEL: @outstanding_commits_multicast_tma_recipients
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: tt.func private @__triton_consan_check_outstanding_commits_noalias{{.*}}T2x1x1xI8; CHECK-LABEL: @outstanding_commits_multicast_tma_recipients anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: tt.func private @__triton_consan_check_outstanding_commits_noalias{{.*}}T2x1x1xI8；CHECK-LABEL: @outstanding_commits_multicast_tma_recipients 这样的标签用于锚定匹配范围。

### Lines 299-313
```mlir
299|   tt.func public @outstanding_commits_multicast_tma_recipients(
300|       %desc: !tt.tensordesc<32x32xf32, #shared>,
301|       %ptr: tensor<32x32x!tt.ptr<f32>, #blocked>) {
302|     %true = arith.constant true
303|     %c0_i32 = arith.constant 0 : i32
304|     %c1_i32 = arith.constant 1 : i32
305|     %shmem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
306|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<2xi64, #shared1, #smem, mutable>
307|     ttng.init_barrier %bar, 1 : !ttg.memdesc<2xi64, #shared1, #smem, mutable>
308|     ttng.barrier_expect %bar, 4096, %true : !ttg.memdesc<2xi64, #shared1, #smem, mutable>
309|     %cta = tti.experimental_cluster_cta_id : i32
310|     %non_issuer_cta = arith.cmpi eq, %cta, %c1_i32 : i32
311|     %mask = tt.splat %non_issuer_cta : i1 -> tensor<32x32xi1, #blocked>
312|     ttg.async_copy_global_to_local %ptr, %shmem mask %mask : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
313|     ttg.async_commit_group
```
**EN:** This function-oriented block defines or enters `outstanding_commits_multicast_tma_recipients`. Within it, the test exercises constants, tt.func, shared/local memory allocation, ttng.init_barrier, ttng.barrier_expect, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `outstanding_commits_multicast_tma_recipients` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、ttng.init_barrier、ttng.barrier_expect，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 314-326
```mlir
314|     // CHECK: tt.call @__triton_consan_stage_access_for_commit
315|     // CHECK: ttg.async_copy_global_to_local
316|     // CHECK: tt.call @__triton_consan_commit_accesses
317|     // CHECK: ttg.async_commit_group
318|     // CHECK: %[[PATTERN:.*]] = arith.constant 3 : i32
319|     // CHECK: %[[RECIPIENTS:.*]] = arith.shli %[[PATTERN]],
320|     // CHECK: tt.call @__triton_consan_check_outstanding_commits{{.*}}({{.*}}, %[[RECIPIENTS]])
321|     // CHECK: ttng.async_tma_copy_global_to_local
322|     ttng.async_tma_copy_global_to_local %desc[%c0_i32, %c0_i32] %shmem, %bar, %true {multicast} : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<2xi64, #shared1, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
323|     tt.return
324|   }
325| }
326| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 327-327
```mlir
327| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 328-333
```mlir
328| 
329| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32, CGALayout = [[0, 0]]}>
330| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
331| #smem = #ttg.shared_memory
332| #offset_parent = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [1, 0], CGALayout = [[0, 0]]}>
333| #offsets = #ttg.slice<{dim = 0, parent = #offset_parent}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 334-334
```mlir
334| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 1 : i32, "ttng.two-ctas" = true, ttg.shared = 65544 : i32, ttg.target = "cuda:100", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 335-335
```mlir
335|   // CHECK-LABEL: @multicast_gather_two_cta_tx_count
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @multicast_gather_two_cta_tx_count anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @multicast_gather_two_cta_tx_count 这样的标签用于锚定匹配范围。

### Lines 336-345
```mlir
336|   tt.func public @multicast_gather_two_cta_tx_count(%desc: !tt.tensordesc<1x32xf32, #shared>) {
337|     %true = arith.constant true
338|     %c0_i32 = arith.constant 0 : i32
339|     %c0 = arith.constant 0 : index
340|     %c1 = arith.constant 1 : index
341|     %c2 = arith.constant 2 : index
342|     %x_offsets = arith.constant dense<0> : tensor<32xi32, #offsets>
343|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
344|     %result = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
345|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `multicast_gather_two_cta_tx_count`. Within it, the test exercises constants, tt.func, shared/local memory allocation, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `multicast_gather_two_cta_tx_count` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 346-347
```mlir
346|     // CHECK: scf.for
347|     scf.for %i = %c0 to %c2 step %c1 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 348-351
```mlir
348|       // CHECK: arith.constant 4096 : i64
349|       // CHECK: tt.call @__triton_consan_verify_barrier_arrive
350|       // CHECK: ttng.barrier_expect
351|       ttng.barrier_expect %bar, 4096, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.barrier_expect. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.barrier_expect。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 352-360
```mlir
352|       // CHECK: arith.constant -8192 : i64
353|       // CHECK: tt.call @__triton_consan_verify_barrier_arrive
354|       // CHECK: ttng.async_tma_gather
355|       ttng.async_tma_gather %desc[%x_offsets, %c0_i32] %result, %bar, %true {multicast} : !tt.tensordesc<1x32xf32, #shared>, tensor<32xi32, #offsets>, i32, !ttg.memdesc<1xi64, #shared1, #smem, mutable>, !ttg.memdesc<32x32xf32, #shared, #smem, mutable>, i1
356|     }
357|     tt.return
358|   }
359| }
360| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 361-361
```mlir
361| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 362-366
```mlir
362| 
363| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
364| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
365| #smem = #ttg.shared_memory
366| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 367-367
```mlir
367| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 368-368
```mlir
368|   // CHECK-LABEL: @async_tma_copy_global_to_local_two_bufs_one_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tma_copy_global_to_local_two_bufs_one_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tma_copy_global_to_local_two_bufs_one_barrier 这样的标签用于锚定匹配范围。

### Lines 369-377
```mlir
369|   tt.func public @async_tma_copy_global_to_local_two_bufs_one_barrier(
370|       %a: !tt.tensordesc<32x32xf32, #shared>,
371|       %b: !tt.tensordesc<32x32xf32, #shared>) {
372|     %true = arith.constant true
373|     %c0_i32 = arith.constant 0 : i32
374| 
375|     %a_smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
376|     %b_smem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
377|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_copy_global_to_local_two_bufs_one_barrier`. Within it, the test exercises shared/local memory allocation, tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_copy_global_to_local_two_bufs_one_barrier` 为核心。测试在其中演示 共享/本地内存分配、tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 378-382
```mlir
378|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
379|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
380|     // Two TMA copies contribute to a single expected transaction.
381|     ttng.barrier_expect %bar, 8192, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
382| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.init_barrier, ttng.barrier_expect. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.init_barrier、ttng.barrier_expect。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 383-400
```mlir
383|     // CHECK: tt.call @__triton_consan_init_barrier_state
384|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
385|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
386|     // CHECK: tt.call @__triton_consan_update_barrier_state
387|     // CHECK: ttng.barrier_expect
388|     // CHECK-COUNT-2: tt.call @__triton_consan_track_barrier_write_for_buffer
389|     ttng.async_tma_copy_global_to_local %a[%c0_i32, %c0_i32] %a_smem, %bar, %true : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
390|     ttng.async_tma_copy_global_to_local %b[%c0_i32, %c0_i32] %b_smem, %bar, %true : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
391| 
392|     ttng.wait_barrier %bar, %c0_i32, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
393| 
394|     // Consume results to prevent DCE / to keep realistic ordering.
395|     %va = ttg.local_load %a_smem : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
396|     %vb = ttg.local_load %b_smem : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
397|     %_ = arith.addf %va, %vb : tensor<32x32xf32, #blocked>
398|     tt.return
399|   }
400| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, local/shared memory loads, ttng.wait_barrier, floating-point additions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、本地/共享内存加载、ttng.wait_barrier、浮点加法、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 402-403
```mlir
402| 
403| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 403-407
```mlir
403| 
404| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
405| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
406| #smem = #ttg.shared_memory
407| #blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 408-408
```mlir
408| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65552 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 409-420
```mlir
409|   // CHECK-LABEL: @async_tma_copy_global_to_local_two_bufs_two_barriers
410|   // CHECK: %[[A_SMEM:.*]] = ttg.local_alloc {allocation.offset = 0 : i32}
411|   // CHECK: %[[B_SMEM:.*]] = ttg.local_alloc {allocation.offset = 4096 : i32}
412|   // CHECK: %[[BAR0:.*]] = ttg.local_alloc {allocation.offset = 65536 : i32}
413|   // CHECK: %[[BAR1:.*]] = ttg.local_alloc {allocation.offset = 65544 : i32}
414|   // CHECK: ttng.barrier_expect %[[BAR0]], 4096, %true
415|   // CHECK: tt.call @__triton_consan_track_barrier_write_for_buffer{{.*}}({{[^,]+}}, {{[^,]+}}, %true, %[[A_TRACK:.*]], {{[^,]+}},
416|   // CHECK: ttng.async_tma_copy_global_to_local %arg0
417|   // CHECK: ttng.barrier_expect %[[BAR1]], 4096, %true
418|   // CHECK-NOT: tt.call @__triton_consan_track_barrier_write_for_buffer{{.*}}({{[^,]+}}, {{[^,]+}}, %true, %[[A_TRACK]], {{[^,]+}},
419|   // CHECK: tt.call @__triton_consan_track_barrier_write_for_buffer{{.*}}({{[^,]+}}, {{[^,]+}}, %true, %[[B_TRACK:.*]], {{[^,]+}},
420|   // CHECK: ttng.async_tma_copy_global_to_local %arg1
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tma_copy_global_to_local_two_bufs_two_barriers anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tma_copy_global_to_local_two_bufs_two_barriers 这样的标签用于锚定匹配范围。

### Lines 421-438
```mlir
421|   tt.func public @async_tma_copy_global_to_local_two_bufs_two_barriers(
422|       %a: !tt.tensordesc<32x32xf32, #shared>,
423|       %b: !tt.tensordesc<32x32xf32, #shared>) {
424|     %true = arith.constant true
425|     %c0_i32 = arith.constant 0 : i32
426|     %a_smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
427|     %b_smem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
428|     %bar0 = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
429|     %bar1 = ttg.local_alloc {allocation.offset = 65544 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
430|     ttng.init_barrier %bar0, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
431|     ttng.init_barrier %bar1, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
432|     ttng.barrier_expect %bar0, 4096, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
433|     ttng.async_tma_copy_global_to_local %a[%c0_i32, %c0_i32] %a_smem, %bar0, %true : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
434|     ttng.barrier_expect %bar1, 4096, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
435|     ttng.async_tma_copy_global_to_local %b[%c0_i32, %c0_i32] %b_smem, %bar1, %true : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
436|     ttng.wait_barrier %bar1, %c0_i32, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
437|     %va = ttg.local_load %a_smem : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
438|     %vb = ttg.local_load %b_smem : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `async_tma_copy_global_to_local_two_bufs_two_barriers`. Within it, the test exercises shared/local memory allocation, tt.func, constants, ttng.init_barrier, ttng.barrier_expect, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_copy_global_to_local_two_bufs_two_barriers` 为核心。测试在其中演示 共享/本地内存分配、tt.func、常量、ttng.init_barrier、ttng.barrier_expect，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 439-443
```mlir
439|     %_ = arith.addf %va, %vb : tensor<32x32xf32, #blocked>
440|     tt.return
441|   }
442| }
443| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on floating-point additions, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 浮点加法、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 444-444
```mlir
444| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 445-451
```mlir
445| 
446| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
447| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
448| #smem = #ttg.shared_memory
449| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
450| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
451| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 452-452
```mlir
452| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 453-453
```mlir
453|   // CHECK-LABEL: @async_tma_copy_local_to_global
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tma_copy_local_to_global anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tma_copy_local_to_global 这样的标签用于锚定匹配范围。

### Lines 454-460
```mlir
454|   tt.func public @async_tma_copy_local_to_global(%arg0: !tt.tensordesc<32x32xf32, #shared>, %ptr: tensor<128x128x!tt.ptr<f16>, #blocked>, %acc: tensor<128x128xf16, #mma>) {
455|     %true = arith.constant true
456|     %c0_i32 = arith.constant 0 : i32
457|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
458|     %shmem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
459|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<128x128x!tt.ptr<f16>, #blocked> -> <128x128xf16, #shared, #smem, mutable>
460| 
```
**EN:** This function-oriented block defines or enters `async_tma_copy_local_to_global`. Within it, the test exercises tt.func, constants, shared/local memory allocation, ttg.async_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_copy_local_to_global` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、ttg.async_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 461-469
```mlir
461|     // CHECK: tt.call @__triton_consan_verify_write_visibility
462|     // CHECK: tt.call @__triton_consan_check_outstanding_commits
463|     // CHECK: tt.call @__triton_consan_stage_access_for_commit
464|     // CHECK: tt.call @__triton_consan_commit_accesses
465|     ttng.async_tma_copy_local_to_global %arg0[%c0_i32, %c0_i32] %0 : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
466|     tt.return
467|   }
468| }
469| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_local_to_global, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_local_to_global、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 470-470
```mlir
470| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 471-477
```mlir
471| 
472| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
473| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
474| #smem = #ttg.shared_memory
475| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
476| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
477| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 478-478
```mlir
478| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 479-479
```mlir
479|   // CHECK-LABEL: @async_tma_store_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tma_store_wait anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tma_store_wait 这样的标签用于锚定匹配范围。

### Lines 480-484
```mlir
480|   tt.func public @async_tma_store_wait(%arg0: !tt.tensordesc<32x32xf32, #shared>, %ptr: tensor<128x128x!tt.ptr<f16>, #blocked>, %acc: tensor<128x128xf16, #mma>) {
481|     %true = arith.constant true
482|     %c0_i32 = arith.constant 0 : i32
483|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
484| 
```
**EN:** This function-oriented block defines or enters `async_tma_store_wait`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_store_wait` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 485-492
```mlir
485|     // CHECK: tt.call @__triton_consan_clear_outstanding_commits_transfer_reads
486|     ttng.async_tma_store_wait {pendings = 0 : i32}
487| 
488|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
489|     tt.return
490|   }
491| }
492| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_store_wait, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_store_wait、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 493-493
```mlir
493| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 494-500
```mlir
494| 
495| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
496| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
497| #smem = #ttg.shared_memory
498| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
499| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
500| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 501-501
```mlir
501| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 502-502
```mlir
502|   // CHECK-LABEL: @async_tma_gather
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tma_gather anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tma_gather 这样的标签用于锚定匹配范围。

### Lines 503-512
```mlir
503|   tt.func public @async_tma_gather(%arg0: !tt.tensordesc<1x32xf32, #shared>, %ptr: tensor<128x128x!tt.ptr<f16>, #blocked>, %acc: tensor<128x128xf16, #mma>) {
504|     %true = arith.constant true
505|     %c0_i32 = arith.constant 0 : i32
506|     %x_offsets = arith.constant dense<1> : tensor<32xi32>
507|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
508|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
509|     %shmem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
510|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
511|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<128x128x!tt.ptr<f16>, #blocked> -> <128x128xf16, #shared, #smem, mutable>
512|     ttng.warp_group_dot %shmem, %shmem, %acc : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> * !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #mma>
```
**EN:** This function-oriented block defines or enters `async_tma_gather`. Within it, the test exercises constants, shared/local memory allocation, tt.func, ttng.init_barrier, ttg.async_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_gather` 为核心。测试在其中演示 常量、共享/本地内存分配、tt.func、ttng.init_barrier、ttg.async_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 513-526
```mlir
513|     // CHECK: ttng.warp_group_dot
514| 
515|     // CHECK: tt.call @__triton_consan_verify_write_visibility
516|     // CHECK: tt.call @__triton_consan_verify_read_visibility
517|     // CHECK: tt.call @__triton_consan_set_write_visibility
518|     // CHECK: tt.call @__triton_consan_clear_write_tracking
519|     // CHECK: tt.call @__triton_consan_clear_read_visibility
520|     // CHECK: tt.call @__triton_consan_clear_read_tracking
521|     // CHECK: tt.call @__triton_consan_track_barrier_write_for_buffer
522|     ttng.async_tma_gather %arg0[%x_offsets, %c0_i32] %0, %bar, %true : !tt.tensordesc<1x32xf32, #shared>, tensor<32xi32>, i32, !ttg.memdesc<1xi64, #shared1, #smem, mutable>, !ttg.memdesc<32x32xf32, #shared, #smem, mutable>, i1
523|     tt.return
524|   }
525| }
526| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 527-527
```mlir
527| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 528-534
```mlir
528| 
529| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
530| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
531| #smem = #ttg.shared_memory
532| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
533| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
534| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 535-535
```mlir
535| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 536-536
```mlir
536|   // CHECK-LABEL: @async_tma_scatter
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tma_scatter anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tma_scatter 这样的标签用于锚定匹配范围。

### Lines 537-546
```mlir
537|   tt.func public @async_tma_scatter(%arg0: !tt.tensordesc<1x32xf32, #shared>, %ptr: tensor<128x128x!tt.ptr<f16>, #blocked>, %acc: tensor<128x128xf16, #mma>) {
538|     %true = arith.constant true
539|     %c0_i32 = arith.constant 0 : i32
540|     %x_offsets = arith.constant dense<1> : tensor<32xi32>
541|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
542|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
543|     %shmem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
544|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
545|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<128x128x!tt.ptr<f16>, #blocked> -> <128x128xf16, #shared, #smem, mutable>
546|     ttng.warp_group_dot %shmem, %shmem, %acc : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> * !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #mma>
```
**EN:** This function-oriented block defines or enters `async_tma_scatter`. Within it, the test exercises constants, shared/local memory allocation, tt.func, ttng.init_barrier, ttg.async_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_scatter` 为核心。测试在其中演示 常量、共享/本地内存分配、tt.func、ttng.init_barrier、ttg.async_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 547-557
```mlir
547|     // CHECK: ttng.warp_group_dot
548| 
549| 	    // CHECK: tt.call @__triton_consan_verify_write_visibility
550| 	    // CHECK: tt.call @__triton_consan_check_outstanding_commits
551| 	    // CHECK: tt.call @__triton_consan_stage_access_for_commit
552| 	    // CHECK: tt.call @__triton_consan_commit_accesses
553| 	    ttng.async_tma_scatter %arg0[%x_offsets, %c0_i32] %0 : !tt.tensordesc<1x32xf32, #shared>, tensor<32xi32>, i32, !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
554| 	    tt.return
555| 	  }
556| }
557| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_scatter, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_scatter、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 558-558
```mlir
558| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 559-564
```mlir
559| 
560| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
561| #smem = #ttg.shared_memory
562| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
563| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
564| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 565-565
```mlir
565| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65536 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 566-566
```mlir
566|   // CHECK-LABEL: @async_tma_reduce
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tma_reduce anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tma_reduce 这样的标签用于锚定匹配范围。

### Lines 567-572
```mlir
567|   tt.func public @async_tma_reduce(%arg0: !tt.tensordesc<32x32xf32, #shared>, %ptr: tensor<128x128x!tt.ptr<f16>, #blocked>, %acc: tensor<128x128xf16, #mma>) {
568|     %c0_i32 = arith.constant 0 : i32
569|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
570|     %shmem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
571|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<128x128x!tt.ptr<f16>, #blocked> -> <128x128xf16, #shared, #smem, mutable>
572| 
```
**EN:** This function-oriented block defines or enters `async_tma_reduce`. Within it, the test exercises tt.func, shared/local memory allocation, constants, ttg.async_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_reduce` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、ttg.async_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 573-581
```mlir
573|     // CHECK: tt.call @__triton_consan_verify_write_visibility
574|     // CHECK: tt.call @__triton_consan_check_outstanding_commits
575|     // CHECK: tt.call @__triton_consan_stage_access_for_commit
576|     // CHECK: tt.call @__triton_consan_commit_accesses
577|     ttng.async_tma_reduce add, %arg0[%c0_i32, %c0_i32] %0 : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
578|     tt.return
579|   }
580| }
581| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_reduce, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_reduce、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 582-582
```mlir
582| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 583-585
```mlir
583| 
584| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
585| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 586-586
```mlir
586| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 587-587
```mlir
587|   // CHECK-LABEL: @barrier_reinit_requires_invalidate
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @barrier_reinit_requires_invalidate anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @barrier_reinit_requires_invalidate 这样的标签用于锚定匹配范围。

### Lines 588-590
```mlir
588|   tt.func public @barrier_reinit_requires_invalidate() {
589|     %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32xi32, #shared1, #smem, mutable>
590|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `barrier_reinit_requires_invalidate`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `barrier_reinit_requires_invalidate` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 591-592
```mlir
591|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
592|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 593-594
```mlir
593|     // CHECK: tt.call @__triton_consan_init_barrier_state
594|     %tmp = ttg.local_load %buf : !ttg.memdesc<32xi32, #shared1, #smem, mutable> -> tensor<32xi32>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 595-596
```mlir
595|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
596|     ttng.inval_barrier %bar : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.inval_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.inval_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 597-601
```mlir
597|     // CHECK: tt.call @__triton_consan_invalidate_barrier_state
598|     // CHECK: tt.call @__triton_consan_clear_barrier_write_tracking
599|     // CHECK: tt.call @__triton_consan_clear_barrier_read_tracking
600|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
601|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 602-606
```mlir
602|     // CHECK: tt.call @__triton_consan_init_barrier_state
603|     tt.return
604|   }
605| }
606| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 607-607
```mlir
607| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 608-612
```mlir
608| 
609| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
610| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
611| #smem = #ttg.shared_memory
612| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 613-613
```mlir
613| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 614-614
```mlir
614|   // CHECK-LABEL: @wait_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @wait_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @wait_barrier 这样的标签用于锚定匹配范围。

### Lines 615-615
```mlir
615|   tt.func public @wait_barrier(%arg0: !tt.tensordesc<32x32xf32, #shared>) {
```
**EN:** This function-oriented block defines or enters `wait_barrier`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wait_barrier` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 616-633
```mlir
616|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64, #linear{{[0-9]*}}>
617| 
618|     // CHECK-DAG: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
619|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
620| 
621|     // CHECK-DAG: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
622|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
623| 
624|     // CHECK-DAG: %[[BARRIERS:.*]] = tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem : tensor<1xi64, #linear{{[0-9]*}}>
625| 
626|     // CHECK-DAG: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
627|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
628| 
629|     // CHECK-DAG: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
630|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
631|     %true = arith.constant true
632|     %c0_i32 = arith.constant 0 : i32
633|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 634-634
```mlir
634|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 635-636
```mlir
635|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
636|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 637-641
```mlir
637|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
638|     // CHECK-DAG: tt.call @__triton_consan_set_waiting
639|     // CHECK-DAG: tt.call @__triton_consan_check_all_active_waiting
640|     // CHECK: ttng.wait_barrier
641|     ttng.wait_barrier %bar, %c0_i32, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.wait_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.wait_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 642-651
```mlir
642|     // CHECK: tti.experimental_lock_acquire
643|     // CHECK: tt.call @__triton_consan_transfer_visible_writes{{.*}}%[[BARRIERS]], %[[WRITE_VISIBILITY_GLOB]], %[[WRITE_TRACKING_GLOB]]
644|     // CHECK: tt.call @__triton_consan_transfer_visible_reads{{.*}}%[[BARRIERS]], %[[READ_VISIBILITY_GLOB]], %[[READ_TRACKING_GLOB]]
645|     // CHECK: tt.call @__triton_consan_clear_waiting
646|     // CHECK: tti.experimental_lock_release
647|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
648|     tt.return
649|   }
650| }
651| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 652-652
```mlir
652| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 653-657
```mlir
653| 
654| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
655| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
656| #smem = #ttg.shared_memory
657| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 658-658
```mlir
658| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 659-659
```mlir
659|   // CHECK-LABEL: @arrive_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @arrive_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @arrive_barrier 这样的标签用于锚定匹配范围。

### Lines 660-660
```mlir
660|   tt.func public @arrive_barrier(%arg0: !tt.tensordesc<32x32xf32, #shared>) {
```
**EN:** This function-oriented block defines or enters `arrive_barrier`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `arrive_barrier` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 661-666
```mlir
661|     // CHECK-DAG: %[[BSTATE_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
662|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[BSTATE_GLOB]], %c0_i64
663|     %true = arith.constant true
664|     %c0_i32 = arith.constant 0 : i32
665|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
666|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 667-668
```mlir
667|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
668|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 669-682
```mlir
669|     // CHECK: tt.call @__triton_consan_init_barrier_state
670|     // CHECK: tti.experimental_lock_acquire
671|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
672|     // CHECK: tt.call @__triton_consan_track_visible_writes
673|     // CHECK: tt.call @__triton_consan_track_visible_reads
674|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
675|     // CHECK: tt.call @__triton_consan_update_barrier_state
676|     // CHECK: tti.experimental_lock_release
677|     ttng.arrive_barrier %bar, 2, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
678|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
679|     tt.return
680|   }
681| }
682| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.arrive_barrier, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.arrive_barrier、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 683-683
```mlir
683| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 684-686
```mlir
684| 
685| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
686| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 687-687
```mlir
687| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 688-688
```mlir
688|   // CHECK-LABEL: @wait_barrier_without_init
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @wait_barrier_without_init anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @wait_barrier_without_init 这样的标签用于锚定匹配范围。

### Lines 689-692
```mlir
689|   tt.func public @wait_barrier_without_init() {
690|     %true = arith.constant true
691|     %c0_i32 = arith.constant 0 : i32
692|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `wait_barrier_without_init`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wait_barrier_without_init` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 693-699
```mlir
693|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
694|     // CHECK: tt.call @__triton_consan_set_waiting
695|     ttng.wait_barrier %bar, %c0_i32, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
696|     tt.return
697|   }
698| }
699| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.wait_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.wait_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 700-700
```mlir
700| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 701-703
```mlir
701| 
702| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
703| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 704-704
```mlir
704| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 705-705
```mlir
705|   // CHECK-LABEL: @arrive_barrier_without_init
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @arrive_barrier_without_init anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @arrive_barrier_without_init 这样的标签用于锚定匹配范围。

### Lines 706-708
```mlir
706|   tt.func public @arrive_barrier_without_init() {
707|     %true = arith.constant true
708|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `arrive_barrier_without_init`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `arrive_barrier_without_init` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 709-716
```mlir
709|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
710|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
711|     // CHECK: tt.call @__triton_consan_update_barrier_state
712|     ttng.arrive_barrier %bar, 1, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
713|     tt.return
714|   }
715| }
716| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.arrive_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.arrive_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 717-717
```mlir
717| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 718-722
```mlir
718| 
719| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
720| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
721| #smem = #ttg.shared_memory
722| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 723-723
```mlir
723| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 724-724
```mlir
724|   // CHECK-LABEL: @tcgen5_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tcgen5_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tcgen5_mma 这样的标签用于锚定匹配范围。

### Lines 725-725
```mlir
725|   tt.func public @tcgen5_mma(%arg0: !tt.tensordesc<32x32xf32, #shared>) {
```
**EN:** This function-oriented block defines or enters `tcgen5_mma`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen5_mma` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 726-743
```mlir
726|     // CHECK-DAG: %[[SM_BUFS:.*]] = tti.experimental_buffer_descriptors [0, 32768], [{{.*}}], shared_mem : tensor<2xi64
727|     // CHECK-DAG: %[[SM_WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
728|     // CHECK-DAG: %[[SM_READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
729|     // CHECK-DAG: %[[TM_BUFS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], tensor_mem : tensor<1xi64
730|     // CHECK-DAG: %[[TM_WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
731|     // CHECK-DAG: %[[TM_READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
732|     // CHECK-DAG: %[[BARRIERS:.*]] = tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem : tensor<1xi64
733| 
734|     // CHECK-DAG: %[[SM_WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 2 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
735|     // CHECK-DAG: %[[SM_READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
736|     // CHECK-DAG: %[[TM_WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
737|     // CHECK-DAG: %[[TM_READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
738| 
739|     // CHECK: ttng.init_barrier
740|     // CHECK: arith.shli
741|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
742|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A:.*]] :
743|     // CHECK: tt.call @__triton_consan_verify_write_visibility{{.*}}%[[A_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_BIT]], %[[SM_BUFS]], %[[SM_WRITE_VISIBILITY_GLOB]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 744-761
```mlir
744|     // CHECK: %[[TC_MASK:.*]] = arith.constant 2 : i64
745|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
746|     // CHECK: tt.call @__triton_consan_set_read_visibility{{.*}}%[[A_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_MASK]], %[[SM_BUFS]], %[[SM_READ_VISIBILITY_GLOB]]
747|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
748|     // CHECK: %[[B_I64:.*]] = tti.experimental_memdesc_to_i32 %[[B:.*]] :
749|     // CHECK: tt.call @__triton_consan_verify_write_visibility{{.*}}%[[B_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_BIT]], %[[SM_BUFS]], %[[SM_WRITE_VISIBILITY_GLOB]]
750|     // CHECK: %[[TC_MASK:.*]] = arith.constant 2 : i64
751|     // CHECK: %[[B_I64:.*]] = tti.experimental_memdesc_to_i32 %[[B]] :
752|     // CHECK: tt.call @__triton_consan_set_read_visibility{{.*}}%[[B_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_MASK]], %[[SM_BUFS]], %[[SM_READ_VISIBILITY_GLOB]]
753|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
754|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC:.*]] :
755|     // CHECK: tt.call @__triton_consan_verify_write_visibility{{.*}}%[[ACC_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_BIT]], %[[TM_BUFS]], %[[TM_WRITE_VISIBILITY_GLOB]]
756|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
757|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC]] :
758|     // CHECK: tt.call @__triton_consan_verify_read_visibility{{.*}}%[[ACC_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_BIT]], %[[TM_BUFS]], %[[TM_READ_VISIBILITY_GLOB]]
759|     // CHECK: %[[TC_MASK:.*]] = arith.constant 2 : i64
760|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC]] :
761|     // CHECK: tt.call @__triton_consan_set_write_visibility{{.*}}%[[ACC_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_MASK]], %[[TM_BUFS]], %[[TM_WRITE_VISIBILITY_GLOB]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 762-779
```mlir
762|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC]] :
763|     // CHECK: tt.call @__triton_consan_clear_write_tracking{{.*}}%[[ACC_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TM_BUFS]], %[[TM_WRITE_TRACKING_GLOB]]
764|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC]] :
765|     // CHECK: tt.call @__triton_consan_clear_read_visibility{{.*}}%[[ACC_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TM_BUFS]], %[[TM_READ_VISIBILITY_GLOB]]
766|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC]] :
767|     // CHECK: tt.call @__triton_consan_clear_read_tracking{{.*}}%[[ACC_I64]], {{.*}}, %[[TM_BUFS]], %{{[^,]+}}
768|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
769|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
770|     // CHECK: %[[BAR_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BAR:.*]] :
771|     // CHECK: tt.call @__triton_consan_track_visible_writes{{.*}}%[[BAR_I64]], {{.*}}, %[[TC_BIT]], %[[BARRIERS]], %[[SM_WRITE_VISIBILITY_GLOB]], %[[SM_WRITE_TRACKING_GLOB]]
772|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
773|     // CHECK: %[[BAR_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BAR]] :
774|     // CHECK: tt.call @__triton_consan_track_visible_reads{{.*}}%[[BAR_I64]], {{.*}}, %[[TC_BIT]], %[[BARRIERS]], %[[SM_READ_VISIBILITY_GLOB]], %{{[^,]+}}
775|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
776|     // CHECK: %[[BAR_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BAR]] :
777|     // CHECK: tt.call @__triton_consan_track_visible_writes{{.*}}%[[BAR_I64]], {{.*}}, %[[TC_BIT]], %[[BARRIERS]], %[[TM_WRITE_VISIBILITY_GLOB]], %[[TM_WRITE_TRACKING_GLOB]]
778|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
779|     // CHECK: %[[BAR_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BAR]] :
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 780-793
```mlir
780|     // CHECK: tt.call @__triton_consan_track_visible_reads{{.*}}%[[BAR_I64]], {{.*}}, %[[TC_BIT]], %[[BARRIERS]], %[[TM_READ_VISIBILITY_GLOB]], %{{[^,]+}}
781|     // CHECK: ttng.tc_gen5_mma %[[A]], %[[B]], %[[ACC]][], {{.*}}, {{.*}}, %[[BAR]]
782|     %c0_i32 = arith.constant 0 : i32
783|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
784|     %1 = ttg.local_alloc {allocation.offset = 32768 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
785|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
786|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
787|     %result = ttng.tmem_alloc  {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>
788|     %true = arith.constant true
789|     ttng.tc_gen5_mma %0, %1, %result[], %true, %true, %bar[%true] {is_async} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>
790|     tt.return
791|   }
792| }
793| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, constants, ttng.init_barrier, tensor-memory allocation, Gen5 tensor-core MMA ops. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、常量、ttng.init_barrier、张量内存分配、Gen5 张量核 MMA 操作。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 794-794
```mlir
794| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 795-800
```mlir
795| 
796| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
797| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
798| #smem = #ttg.shared_memory
799| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
800| #tmem1 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 801-801
```mlir
801| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 802-802
```mlir
802|   // CHECK-LABEL: @tcgen5_mma_lhs_in_tmem
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tcgen5_mma_lhs_in_tmem anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tcgen5_mma_lhs_in_tmem 这样的标签用于锚定匹配范围。

### Lines 803-803
```mlir
803|   tt.func public @tcgen5_mma_lhs_in_tmem(%arg0: !tt.tensordesc<32x32xf32, #shared>) {
```
**EN:** This function-oriented block defines or enters `tcgen5_mma_lhs_in_tmem`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen5_mma_lhs_in_tmem` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 804-821
```mlir
804|     // CHECK-DAG: %[[SM_BUFS:.*]] = tti.experimental_buffer_descriptors [32768], [{{.*}}], shared_mem : tensor<1xi64
805|     // CHECK-DAG: %[[SM_WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
806|     // CHECK-DAG: %[[SM_READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
807|     // CHECK-DAG: %[[TM_BUFS:.*]] = tti.experimental_buffer_descriptors [0, 128], [{{.*}}], tensor_mem : tensor<2xi64
808|     // CHECK-DAG: %[[TM_WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
809|     // CHECK-DAG: %[[TM_READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
810|     // CHECK-DAG: %[[BARRIERS:.*]] = tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem : tensor<1xi64
811| 
812|     // CHECK-DAG: %[[SM_WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
813|     // CHECK-DAG: %[[SM_READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
814|     // CHECK-DAG: %[[TM_WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 2 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
815|     // CHECK-DAG: %[[TM_READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
816| 
817|     // CHECK: ttng.init_barrier
818|     // CHECK: arith.shli
819|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
820|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A:.*]] :
821|     // CHECK: tt.call @__triton_consan_verify_write_visibility{{.*}}%[[A_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_BIT]], %[[TM_BUFS]], %[[TM_WRITE_VISIBILITY_GLOB]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 822-839
```mlir
822|     // CHECK: %[[TC_MASK:.*]] = arith.constant 2 : i64
823|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
824|     // CHECK: tt.call @__triton_consan_set_read_visibility{{.*}}%[[A_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_MASK]], %[[TM_BUFS]], %[[TM_READ_VISIBILITY_GLOB]]
825|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
826|     // CHECK: %[[B_I64:.*]] = tti.experimental_memdesc_to_i32 %[[B:.*]] :
827|     // CHECK: tt.call @__triton_consan_verify_write_visibility{{.*}}%[[B_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_BIT]], %[[SM_BUFS]], %[[SM_WRITE_VISIBILITY_GLOB]]
828|     // CHECK: %[[TC_MASK:.*]] = arith.constant 2 : i64
829|     // CHECK: %[[B_I64:.*]] = tti.experimental_memdesc_to_i32 %[[B]] :
830|     // CHECK: tt.call @__triton_consan_set_read_visibility{{.*}}%[[B_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_MASK]], %[[SM_BUFS]], %[[SM_READ_VISIBILITY_GLOB]]
831|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
832|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC:.*]] :
833|     // CHECK: tt.call @__triton_consan_verify_write_visibility{{.*}}%[[ACC_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_BIT]], %[[TM_BUFS]], %[[TM_WRITE_VISIBILITY_GLOB]]
834|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
835|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC]] :
836|     // CHECK: tt.call @__triton_consan_verify_read_visibility{{.*}}%[[ACC_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_BIT]], %[[TM_BUFS]], %[[TM_READ_VISIBILITY_GLOB]]
837|     // CHECK: %[[TC_MASK:.*]] = arith.constant 2 : i64
838|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC]] :
839|     // CHECK: tt.call @__triton_consan_set_write_visibility{{.*}}%[[ACC_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TC_MASK]], %[[TM_BUFS]], %[[TM_WRITE_VISIBILITY_GLOB]]
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 840-857
```mlir
840|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC]] :
841|     // CHECK: tt.call @__triton_consan_clear_write_tracking{{.*}}%[[ACC_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TM_BUFS]], %[[TM_WRITE_TRACKING_GLOB]]
842|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC]] :
843|     // CHECK: tt.call @__triton_consan_clear_read_visibility{{.*}}%[[ACC_I64]], {{[^,]+}}, %{{[^,]+}}, %[[TM_BUFS]], %[[TM_READ_VISIBILITY_GLOB]]
844|     // CHECK: %[[ACC_I64:.*]] = tti.experimental_memdesc_to_i32 %[[ACC]] :
845|     // CHECK: tt.call @__triton_consan_clear_read_tracking{{.*}}%[[ACC_I64]], {{.*}}, %[[TM_BUFS]], %{{[^,]+}}
846|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
847|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
848|     // CHECK: %[[BAR_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BAR:.*]] :
849|     // CHECK: tt.call @__triton_consan_track_visible_writes{{.*}}%[[BAR_I64]], {{.*}}, %[[TC_BIT]], %[[BARRIERS]], %[[SM_WRITE_VISIBILITY_GLOB]], %[[SM_WRITE_TRACKING_GLOB]]
850|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
851|     // CHECK: %[[BAR_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BAR]] :
852|     // CHECK: tt.call @__triton_consan_track_visible_reads{{.*}}%[[BAR_I64]], {{.*}}, %[[TC_BIT]], %[[BARRIERS]], %[[SM_READ_VISIBILITY_GLOB]], %{{[^,]+}}
853|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
854|     // CHECK: %[[BAR_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BAR]] :
855|     // CHECK: tt.call @__triton_consan_track_visible_writes{{.*}}%[[BAR_I64]], {{.*}}, %[[TC_BIT]], %[[BARRIERS]], %[[TM_WRITE_VISIBILITY_GLOB]], %[[TM_WRITE_TRACKING_GLOB]]
856|     // CHECK: %[[TC_BIT:.*]] = arith.constant 1 : i32
857|     // CHECK: %[[BAR_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BAR]] :
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 858-874
```mlir
858|     // CHECK: tt.call @__triton_consan_track_visible_reads{{.*}}%[[BAR_I64]], {{.*}}, %[[TC_BIT]], %[[BARRIERS]], %[[TM_READ_VISIBILITY_GLOB]], %{{[^,]+}}
859|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
860|     // CHECK: tt.call @__triton_consan_update_barrier_state
861|     // CHECK: tti.experimental_lock_release
862|     // CHECK: ttng.tc_gen5_mma %[[A]], %[[B]], %[[ACC]][], {{.*}}, {{.*}}, %[[BAR]]
863|     %c0_i32 = arith.constant 0 : i32
864|     %0 = ttng.tmem_alloc  {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #tmem1, #ttng.tensor_memory, mutable>
865|     %1 = ttg.local_alloc {allocation.offset = 32768 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
866|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
867|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
868|     %result = ttng.tmem_alloc  {tensor_memory_col_offset = 128 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>
869|     %true = arith.constant true
870|     ttng.tc_gen5_mma %0, %1, %result[], %true, %true, %bar[%true] {is_async} : !ttg.memdesc<128x128xf16, #tmem1, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>
871|     tt.return
872|   }
873| }
874| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, tensor-memory allocation, shared/local memory allocation, ttng.init_barrier, Gen5 tensor-core MMA ops. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、张量内存分配、共享/本地内存分配、ttng.init_barrier、Gen5 张量核 MMA 操作。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 875-875
```mlir
875| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 876-881
```mlir
876| 
877| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
878| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
879| #smem = #ttg.shared_memory
880| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
881| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 882-882
```mlir
882| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 883-883
```mlir
883|   // CHECK-LABEL: @tcgen5_commit
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tcgen5_commit anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tcgen5_commit 这样的标签用于锚定匹配范围。

### Lines 884-890
```mlir
884|   tt.func public @tcgen5_commit(%arg0: !tt.tensordesc<32x32xf32, #shared>) {
885| 
886|     %c0_i32 = arith.constant 0 : i32
887|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
888|     %result = ttng.tmem_alloc  {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>
889|     %bar = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
890|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `tcgen5_commit`. Within it, the test exercises tt.func, shared/local memory allocation, constants, tensor-memory allocation, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen5_commit` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、张量内存分配、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 891-892
```mlir
891|     // CHECK: tt.call @__triton_consan_init_barrier_state
892|     %true = arith.constant true
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 893-905
```mlir
893|     // CHECK: tt.call @__triton_consan_track_visible_writes
894|     // CHECK: tt.call @__triton_consan_track_visible_reads
895|     // CHECK: tt.call @__triton_consan_track_visible_writes
896|     // CHECK: tt.call @__triton_consan_track_visible_reads
897|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
898|     // CHECK: tt.call @__triton_consan_update_barrier_state
899|     ttng.tc_gen5_commit %bar : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
900|     ttg.local_load %0 : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #blocked>
901|     ttng.tmem_load %result : !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf16>
902|     tt.return
903|   }
904| }
905| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tc_gen5_commit, local/shared memory loads, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tc_gen5_commit、本地/共享内存加载、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 906-906
```mlir
906| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 907-910
```mlir
907| 
908| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32, CGALayout = [[0, 0]]}>
909| #smem = #ttg.shared_memory
910| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1, CGALayout = [[0, 0]]>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 911-911
```mlir
911| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32, "ttng.two-ctas" = true} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 912-912
```mlir
912|   // CHECK-LABEL: @tmem_copy_2cta
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tmem_copy_2cta anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tmem_copy_2cta 这样的标签用于锚定匹配范围。

### Lines 913-915
```mlir
913|   tt.func public @tmem_copy_2cta() {
914|     %src = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #shared, #smem, mutable>
915|     %dst = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tmem_copy_2cta`. Within it, the test exercises tt.func, shared/local memory allocation, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_copy_2cta` 为核心。测试在其中演示 tt.func、共享/本地内存分配、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 916-922
```mlir
916|     // CHECK: arith.constant 3 : i32
917|     // CHECK: ttng.tmem_copy
918|     ttng.tmem_copy %src, %dst : !ttg.memdesc<128x128xf32, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
919|     tt.return
920|   }
921| }
922| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory copies, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存拷贝、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 923-923
```mlir
923| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 924-927
```mlir
924| 
925| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
926| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
927| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 928-928
```mlir
928| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 929-929
```mlir
929|   // CHECK-LABEL: @async_copy_global_to_local
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_copy_global_to_local anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_copy_global_to_local 这样的标签用于锚定匹配范围。

### Lines 930-930
```mlir
930|   tt.func public @async_copy_global_to_local(%ptr: tensor<128x128x!tt.ptr<f16>, #blocked>) {
```
**EN:** This function-oriented block defines or enters `async_copy_global_to_local`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_global_to_local` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 931-948
```mlir
931|     // CHECK: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
932|     // CHECK: %[[WRT_COMMITS_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
933|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRT_COMMITS_GLOB]], %c0_i8
934| 
935|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A:.*]] :
936|     // CHECK: tt.call @__triton_consan_verify_write_visibility_noalias_nw1{{.*}}(%[[A_I64]]
937|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
938|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
939|     // CHECK: tt.call @__triton_consan_check_outstanding_commits{{.*}}(%[[A_I64]], {{.*}}, %[[THREAD_BIT]], %[[BUFFERS]], %[[WRT_COMMITS_GLOB]]
940|     // CHECK: tt.call @__triton_consan_verify_read_visibility_noalias_nw1
941|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
942|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
943|     // CHECK: tt.call @__triton_consan_stage_access_for_commit_nw1{{.*}}(%[[A_I64]], {{.*}}, %[[THREAD_BIT]], %[[BUFFERS]], %[[WRT_COMMITS_GLOB]]
944|     // CHECK: ttg.async_copy_global_to_local %{{.*}}, %[[A]]
945| 
946|     %shmem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
947|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<128x128x!tt.ptr<f16>, #blocked> -> <128x128xf16, #shared, #smem, mutable>
948|     tt.return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 949-951
```mlir
949|   }
950| }
951| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 952-952
```mlir
952| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 953-958
```mlir
953| 
954| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
955| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
956| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
957| 
958| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 959-959
```mlir
959| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 960-960
```mlir
960|   // CHECK-LABEL: @async_copy_global_to_local_with_barriers
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_copy_global_to_local_with_barriers anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_copy_global_to_local_with_barriers 这样的标签用于锚定匹配范围。

### Lines 961-961
```mlir
961|   tt.func public @async_copy_global_to_local_with_barriers(%ptr: tensor<128x128x!tt.ptr<f16>, #blocked>) {
```
**EN:** This function-oriented block defines or enters `async_copy_global_to_local_with_barriers`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_global_to_local_with_barriers` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 962-979
```mlir
962|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
963|     // CHECK-DAG: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
964|     // CHECK-DAG: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
965|     // CHECK-DAG: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
966|     // CHECK-DAG: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
967| 
968|     // CHECK-DAG: %[[WRT_COMMITS_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
969| 
970|     // CHECK: tt.call @__triton_consan_init_barrier_state
971| 
972|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A:.*]] :
973|     // CHECK: tt.call @__triton_consan_verify_write_visibility_noalias{{.*}}(%[[A_I64]]
974|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
975|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
976|     // CHECK: tt.call @__triton_consan_check_outstanding_commits{{.*}}(%[[A_I64]], {{.*}}, %[[THREAD_BIT]], %[[BUFFERS]], %[[WRT_COMMITS_GLOB]]
977|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
978|     // CHECK: tt.call @__triton_consan_verify_read_visibility{{.*}}(%[[A_I64]]
979|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 980-990
```mlir
980|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
981|     // CHECK: tt.call @__triton_consan_stage_access_for_commit{{.*}}(%[[A_I64]], {{.*}}, %[[THREAD_BIT]], %[[BUFFERS]], %[[WRT_COMMITS_GLOB]]
982|     // CHECK: ttg.async_copy_global_to_local %{{.*}}, %[[A]]
983|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
984|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
985|     %shmem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
986|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<128x128x!tt.ptr<f16>, #blocked> -> <128x128xf16, #shared, #smem, mutable>
987|     tt.return
988|   }
989| }
990| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier, ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier、ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 991-991
```mlir
991| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 992-994
```mlir
992| 
993| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
994| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 995-995
```mlir
995| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 996-996
```mlir
996|   // CHECK-LABEL: @wait_barrier_multi_cta
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @wait_barrier_multi_cta anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @wait_barrier_multi_cta 这样的标签用于锚定匹配范围。

### Lines 997-1001
```mlir
 997|   tt.func public @wait_barrier_multi_cta() {
 998|     %true = arith.constant true
 999|     %c0_i32 = arith.constant 0 : i32
1000|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1001|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `wait_barrier_multi_cta`. Within it, the test exercises tt.func, constants, shared/local memory allocation, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wait_barrier_multi_cta` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1002-1015
```mlir
1002|     // CHECK: %[[WAIT_PRED:.*]] = arith.andi %true, %{{.*}} : i1
1003|     // CHECK-NEXT: tti.experimental_lock_acquire %{{.*}}, %[[WAIT_PRED]]
1004|     // CHECK: tt.call @__triton_consan_check_all_active_waiting
1005|     // CHECK-NEXT: tti.experimental_assert_uniform
1006|     // CHECK-NEXT: tti.experimental_lock_release %{{.*}}, %[[WAIT_PRED]]
1007|     // CHECK: ttng.wait_barrier
1008|     // CHECK-NEXT: tti.experimental_lock_acquire %{{.*}}, %[[WAIT_PRED]]
1009|     // CHECK: tt.call @__triton_consan_clear_waiting
1010|     // CHECK-NEXT: tti.experimental_lock_release %{{.*}}, %[[WAIT_PRED]]
1011|     ttng.wait_barrier %bar, %c0_i32, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1012|     tt.return
1013|   }
1014| }
1015| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.wait_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.wait_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1016-1016
```mlir
1016| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1017-1020
```mlir
1017| 
1018| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1019| #smem = #ttg.shared_memory
1020| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1021-1021
```mlir
1021| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1022-1022
```mlir
1022|   // CHECK-LABEL: @async_commit_group
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_commit_group anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_commit_group 这样的标签用于锚定匹配范围。

### Lines 1023-1023
```mlir
1023|   tt.func public @async_commit_group() {
```
**EN:** This function-oriented block defines or enters `async_commit_group`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_commit_group` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1024-1031
```mlir
1024|     // CHECK: tt.call @__triton_consan_commit_accesses
1025|     %shmem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1026|     ttg.async_commit_group
1027|     ttg.local_load %shmem : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #blocked>
1028|     tt.return
1029|   }
1030| }
1031| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, async copy commit groups, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、异步拷贝提交组、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1032-1032
```mlir
1032| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1033-1036
```mlir
1033| 
1034| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
1035| #smem = #ttg.shared_memory
1036| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1037-1037
```mlir
1037| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1038-1038
```mlir
1038|   // CHECK-LABEL: @async_commit_group
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_commit_group anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_commit_group 这样的标签用于锚定匹配范围。

### Lines 1039-1039
```mlir
1039|   tt.func public @async_commit_group() {
```
**EN:** This function-oriented block defines or enters `async_commit_group`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_commit_group` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1040-1051
```mlir
1040|     // CHECK: tti.experimental_lock_acquire
1041|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
1042|     // CHECK: %[[THREAD_MASK:.*]] = arith.constant 1 : i64
1043|     // CHECK: %[[OUTSTANDING_NUM:.*]] = arith.constant 42 : i32
1044|     // CHECK: tt.call @__triton_consan_clear_outstanding_commits_transfer_writes{{.*}}(%[[THREAD_BIT]], %[[THREAD_MASK]], %[[OUTSTANDING_NUM]]
1045|     %shmem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1046|     ttg.async_wait {num = 42 : i32}
1047|     ttg.local_load %shmem : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #blocked>
1048|     tt.return
1049|   }
1050| }
1051| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, async wait synchronization, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、异步等待同步、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1052-1052
```mlir
1052| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1053-1057
```mlir
1053| 
1054| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1055| #smem = #ttg.shared_memory
1056| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
1057| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1058-1058
```mlir
1058| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1059-1059
```mlir
1059|   // CHECK-LABEL: @tmem_load
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tmem_load anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tmem_load 这样的标签用于锚定匹配范围。

### Lines 1060-1063
```mlir
1060|   tt.func public @tmem_load() {
1061|     %result = ttng.tmem_alloc  {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>
1062|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1063|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `tmem_load`. Within it, the test exercises tt.func, tensor-memory allocation, shared/local memory allocation, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_load` 为核心。测试在其中演示 tt.func、张量内存分配、共享/本地内存分配、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1064-1069
```mlir
1064|     // CHECK: tt.call @__triton_consan_verify_write_visibility
1065|     ttng.tmem_load %result : !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf16>
1066|     tt.return
1067|   }
1068| }
1069| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1070-1070
```mlir
1070| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1071-1075
```mlir
1071| 
1072| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1073| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1074| #smem = #ttg.shared_memory
1075| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1076-1076
```mlir
1076| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1077-1077
```mlir
1077|   // CHECK-LABEL: @warp_group_dot
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @warp_group_dot anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @warp_group_dot 这样的标签用于锚定匹配范围。

### Lines 1078-1078
```mlir
1078|   tt.func public @warp_group_dot(%acc: tensor<128x128xf16, #mma>) {
```
**EN:** This function-oriented block defines or enters `warp_group_dot`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_group_dot` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1079-1096
```mlir
1079|     // CHECK-DAG: %[[SM_BUFS:.*]] = tti.experimental_buffer_descriptors [0, 32768], [{{.*}}], shared_mem : tensor<2xi64
1080| 
1081|     // CHECK-DAG: %[[SM_WGMMA_WRITES_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 2 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
1082|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[SM_WGMMA_WRITES_GLOB]], %c0_i8
1083| 
1084|     // CHECK: tt.call @__triton_consan_verify_write_visibility
1085|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
1086|     // CHECK: tt.call @__triton_consan_stage_access_for_commit{{.*}}(%[[A:.*]], {{.*}}, %[[THREAD_BIT]], %[[SM_BUFS]], %[[SM_WGMMA_WRITES_GLOB]]
1087|     // CHECK: tt.call @__triton_consan_verify_write_visibility
1088|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
1089|     // CHECK: tt.call @__triton_consan_stage_access_for_commit{{.*}}(%[[B:.*]], {{.*}}, %[[THREAD_BIT]], %[[SM_BUFS]], %[[SM_WGMMA_WRITES_GLOB]]
1090|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
1091|     // CHECK: tt.call @__triton_consan_commit_accesses{{.*}}(%[[THREAD_BIT]], {{.*}}, %[[SM_WGMMA_WRITES_GLOB]]
1092|     %c0_i32 = arith.constant 0 : i32
1093|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1094|     %1 = ttg.local_alloc {allocation.offset = 32768 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1095|     %true = arith.constant true
1096|     ttng.warp_group_dot %0, %1, %acc, %true {isAsync = true} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> * !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #mma>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, shared/local memory allocation, ttng.warp_group_dot. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、共享/本地内存分配、ttng.warp_group_dot。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1097-1100
```mlir
1097|     tt.return
1098|   }
1099| }
1100| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1101-1101
```mlir
1101| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1102-1106
```mlir
1102| 
1103| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1104| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1105| #smem = #ttg.shared_memory
1106| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1107-1107
```mlir
1107| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1108-1108
```mlir
1108|   // CHECK-LABEL: @warp_group_dot_sync
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @warp_group_dot_sync anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @warp_group_dot_sync 这样的标签用于锚定匹配范围。

### Lines 1109-1109
```mlir
1109|   tt.func public @warp_group_dot_sync(%acc: tensor<128x128xf16, #mma>) {
```
**EN:** This function-oriented block defines or enters `warp_group_dot_sync`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_group_dot_sync` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1110-1127
```mlir
1110|     // CHECK-DAG: %[[SM_BUFS:.*]] = tti.experimental_buffer_descriptors [0, 32768], [{{.*}}], shared_mem : tensor<2xi64
1111| 
1112|     // CHECK-DAG: %[[SM_WGMMA_WRITES_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 2 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
1113|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[SM_WGMMA_WRITES_GLOB]], %c0_i8
1114| 
1115|     // CHECK: "before_dot"
1116|     // CHECK-NOT: tt.call @__triton_consan_stage_access_for_commit
1117|     // CHECK-NOT: tt.call @__triton_consan_commit_accesses
1118|     %c0_i32 = arith.constant 0 : i32
1119|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1120|     %1 = ttg.local_alloc {allocation.offset = 32768 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1121|     %true = arith.constant true
1122|     "before_dot"() : () -> ()
1123|     ttng.warp_group_dot %0, %1, %acc, %true {isAsync = false} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> * !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #mma>
1124|     tt.return
1125|   }
1126| }
1127| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, shared/local memory allocation, ttng.warp_group_dot, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、共享/本地内存分配、ttng.warp_group_dot、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1128-1128
```mlir
1128| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1129-1134
```mlir
1129| 
1130| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1131| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1132| #smem = #ttg.shared_memory
1133| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
1134| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1135-1135
```mlir
1135| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1136-1136
```mlir
1136|   // CHECK-LABEL: @warp_group_dot_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @warp_group_dot_wait anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @warp_group_dot_wait 这样的标签用于锚定匹配范围。

### Lines 1137-1139
```mlir
1137|   tt.func public @warp_group_dot_wait(%acc: tensor<128x128xf16, #mma>) {
1138|     // Dummy buffer just to make the pass run
1139|     %dummy = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `warp_group_dot_wait`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `warp_group_dot_wait` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1140-1146
```mlir
1140|     // CHECK: tt.call @__triton_consan_clear_outstanding_commits_transfer_reads
1141|     ttng.warp_group_dot_wait %acc { pendings = 42 : i32 } : tensor<128x128xf16, #mma>
1142|     ttg.local_load %dummy : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #blocked>
1143|     tt.return
1144|   }
1145| }
1146| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.warp_group_dot_wait, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.warp_group_dot_wait、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1147-1147
```mlir
1147| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1148-1152
```mlir
1148| 
1149| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1150| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1151| #smem = #ttg.shared_memory
1152| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1153-1153
```mlir
1153| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1154-1154
```mlir
1154|   // CHECK-LABEL: @local_alloc_with_src
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @local_alloc_with_src anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @local_alloc_with_src 这样的标签用于锚定匹配范围。

### Lines 1155-1155
```mlir
1155|   tt.func public @local_alloc_with_src(%acc: tensor<128x128xf16, #mma>) {
```
**EN:** This function-oriented block defines or enters `local_alloc_with_src`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_alloc_with_src` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1156-1167
```mlir
1156|     // CHECK: %[[BUF:.*]] = ttg.local_alloc
1157|     // CHECK: %[[BUF_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BUF:.*]] :
1158|     // CHECK: tt.call @__triton_consan_verify_write_visibility{{.*}}(%[[BUF_I64]]
1159|     // CHECK: %[[BUF_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BUF:.*]] :
1160|     // CHECK: tt.call @__triton_consan_verify_read_visibility{{.*}}(%[[BUF_I64]]
1161|     %buf = ttg.local_alloc %acc {allocation.offset = 0 : i32} : (tensor<128x128xf16, #mma>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1162|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1163|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1164|     tt.return
1165|   }
1166| }
1167| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1168-1168
```mlir
1168| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1169-1175
```mlir
1169| 
1170| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1171| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1172| #smem = #ttg.shared_memory
1173| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1174| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
1175| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1176-1176
```mlir
1176| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1177-1177
```mlir
1177|   // CHECK-LABEL: @tmem_alloc_with_src
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tmem_alloc_with_src anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tmem_alloc_with_src 这样的标签用于锚定匹配范围。

### Lines 1178-1178
```mlir
1178|   tt.func public @tmem_alloc_with_src(%acc: tensor<128x128xf16, #blocked>) {
```
**EN:** This function-oriented block defines or enters `tmem_alloc_with_src`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_alloc_with_src` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1179-1190
```mlir
1179|     // CHECK: %[[BUF:.*]] = ttng.tmem_alloc
1180|     // CHECK: %[[BUF_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BUF:.*]] :
1181|     // CHECK: tt.call @__triton_consan_verify_write_visibility{{.*}}(%[[BUF_I64]]
1182|     // CHECK: %[[BUF_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BUF:.*]] :
1183|     // CHECK: tt.call @__triton_consan_verify_read_visibility{{.*}}(%[[BUF_I64]]
1184|     %buf = ttng.tmem_alloc %acc { tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32 } : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>
1185|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1186|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1187|     tt.return
1188|   }
1189| }
1190| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, shared/local memory allocation, ttng.init_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、共享/本地内存分配、ttng.init_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1191-1191
```mlir
1191| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1192-1196
```mlir
1192| 
1193| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1194| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1195| #smem = #ttg.shared_memory
1196| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1197-1197
```mlir
1197| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1198-1198
```mlir
1198|   // CHECK-LABEL: @local_load_barriers
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @local_load_barriers anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @local_load_barriers 这样的标签用于锚定匹配范围。

### Lines 1199-1199
```mlir
1199|   tt.func public @local_load_barriers() {
```
**EN:** This function-oriented block defines or enters `local_load_barriers`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_load_barriers` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1200-1203
```mlir
1200|     // CHECK: tti.experimental_buffer_descriptors
1201|     %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1202|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1203|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1204-1212
```mlir
1204|     // CHECK: tti.experimental_lock_acquire
1205|     // CHECK: tt.call @__triton_consan_verify_write_visibility
1206|     // CHECK: tt.call @__triton_consan_set_read_visibility
1207|     // CHECK: tti.experimental_lock_release
1208|     ttg.local_load %buf : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #blocked>
1209|     tt.return
1210|   }
1211| }
1212| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1213-1213
```mlir
1213| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1214-1218
```mlir
1214| 
1215| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1216| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1217| #smem = #ttg.shared_memory
1218| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1219-1219
```mlir
1219| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1220-1220
```mlir
1220|   // CHECK-LABEL: @local_load_barriers
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @local_load_barriers anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @local_load_barriers 这样的标签用于锚定匹配范围。

### Lines 1221-1221
```mlir
1221|   tt.func public @local_load_barriers_cp_async(%ptr: tensor<128x128x!tt.ptr<f16>, #blocked>) {
```
**EN:** This function-oriented block defines or enters `local_load_barriers_cp_async`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_load_barriers_cp_async` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1222-1228
```mlir
1222|     // CHECK: tti.experimental_buffer_descriptors
1223|     %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1224|     %shmem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1225|     %bar = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1226|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1227|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<128x128x!tt.ptr<f16>, #blocked> -> <128x128xf16, #shared, #smem, mutable>
1228| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier, ttg.async_copy_global_to_local. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier、ttg.async_copy_global_to_local。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1229-1241
```mlir
1229|     // CHECK: ttg.async_copy_global_to_local
1230| 
1231|     // CHECK: tti.experimental_lock_acquire
1232|     // CHECK: tt.call @__triton_consan_verify_write_visibility
1233|     // CHECK: tt.call @__triton_consan_check_outstanding_commits
1234|     // CHECK: tt.call @__triton_consan_set_read_visibility
1235|     // CHECK: tti.experimental_lock_release
1236|     // CHECK: ttg.local_load
1237|     ttg.local_load %buf : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #blocked>
1238|     tt.return
1239|   }
1240| }
1241| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1242-1242
```mlir
1242| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1243-1248
```mlir
1243| 
1244| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1245| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1246| #smem = #ttg.shared_memory
1247| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
1248| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1249-1249
```mlir
1249| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1250-1250
```mlir
1250|   // CHECK-LABEL: @local_store_barriers_cp_async_wgmma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @local_store_barriers_cp_async_wgmma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @local_store_barriers_cp_async_wgmma 这样的标签用于锚定匹配范围。

### Lines 1251-1251
```mlir
1251|   tt.func public @local_store_barriers_cp_async_wgmma(%ptr: tensor<128x128x!tt.ptr<f16>, #blocked>, %acc: tensor<128x128xf16, #mma>) {
```
**EN:** This function-oriented block defines or enters `local_store_barriers_cp_async_wgmma`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_store_barriers_cp_async_wgmma` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1252-1258
```mlir
1252|     // CHECK: tti.experimental_buffer_descriptors
1253|     %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1254|     %shmem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1255|     %bar = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1256|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1257|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<128x128x!tt.ptr<f16>, #blocked> -> <128x128xf16, #shared, #smem, mutable>
1258|     ttng.warp_group_dot %shmem, %shmem, %acc : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> * !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #mma>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier, ttg.async_copy_global_to_local, ttng.warp_group_dot. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier、ttg.async_copy_global_to_local、ttng.warp_group_dot。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1259-1276
```mlir
1259|     // CHECK: ttng.warp_group_dot
1260| 
1261|     // CHECK: tti.experimental_lock_acquire
1262|     // CHECK: tt.call @__triton_consan_verify_write_visibility
1263|     // CHECK: tt.call @__triton_consan_check_outstanding_commits
1264|     // CHECK: tt.call @__triton_consan_verify_read_visibility
1265|     // CHECK: tt.call @__triton_consan_check_outstanding_commits
1266|     // CHECK: tt.call @__triton_consan_set_write_visibility
1267|     // CHECK: tt.call @__triton_consan_clear_write_tracking
1268|     // CHECK: tt.call @__triton_consan_clear_read_visibility
1269|     // CHECK: tt.call @__triton_consan_clear_read_tracking
1270|     // CHECK: tti.experimental_lock_release
1271|     // CHECK: ttg.local_store
1272|     ttg.local_store %acc, %buf : tensor<128x128xf16, #mma> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1273|     tt.return
1274|   }
1275| }
1276| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory stores, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存存储、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1277-1277
```mlir
1277| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1278-1282
```mlir
1278| 
1279| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1280| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1281| #smem = #ttg.shared_memory
1282| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1283-1283
```mlir
1283| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1284-1284
```mlir
1284|   // CHECK-LABEL: @ws_allocation
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ws_allocation anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ws_allocation 这样的标签用于锚定匹配范围。

### Lines 1285-1285
```mlir
1285|   tt.func public @ws_allocation(%arg0: !tt.tensordesc<32x32xf32, #shared>) {
```
**EN:** This function-oriented block defines or enters `ws_allocation`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ws_allocation` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1286-1290
```mlir
1286|     // CHECK-DAG: tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem : tensor<1xi64,
1287|     // CHECK-DAG: tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
1288|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1289|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1290|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1291-1298
```mlir
1291|     // CHECK: tti.experimental_lock_acquire
1292|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
1293|     // CHECK: %[[THREAD_MASK:.*]] = arith.constant 2 : i64
1294|     // CHECK: tt.call @__triton_consan_copy_write_visibility{{.*}}(%[[THREAD_BIT]], %[[THREAD_MASK]]
1295|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
1296|     // CHECK: tt.call @__triton_consan_copy_read_visibility{{.*}}(%[[THREAD_BIT]]
1297|     ttg.warp_specialize(%smem, %bar) attributes {actualRegisters = array<i32: 480, 32>, allocation.offset = 512 : i32, requestedRegisters = array<i32: 32>, warpGroupStartIds = array<i32: 4>}
1298|     default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1299-1306
```mlir
1299|       // CHECK: tti.experimental_lock_acquire
1300|       // CHECK: tt.call @__triton_consan_verify_write_visibility
1301|       // CHECK: tt.call @__triton_consan_set_read_visibility
1302|       // CHECK: tti.experimental_lock_release
1303|       ttg.local_load %smem : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16>
1304|       ttg.warp_yield
1305|     }
1306|     partition0(%arg1: !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<1xi64, #shared1, #smem, mutable>) num_warps(4) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1307-1320
```mlir
1307|       // CHECK: partition0
1308|       // CHECK-DAG: tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem : tensor<1xi64,
1309|       // CHECK-DAG: tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
1310|       // CHECK: tti.experimental_lock_acquire
1311|       // CHECK: tt.call @__triton_consan_verify_write_visibility
1312|       // CHECK: tt.call @__triton_consan_set_read_visibility
1313|       // CHECK: tti.experimental_lock_release
1314|       ttg.local_load %arg1 : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16>
1315|       ttg.warp_return
1316|     } : (!ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>) -> ()
1317|     tt.return
1318|   }
1319| }
1320| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1321-1321
```mlir
1321| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1322-1326
```mlir
1322| 
1323| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1324| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1325| #smem = #ttg.shared_memory
1326| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1327-1327
```mlir
1327| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1328-1328
```mlir
1328|   // CHECK-LABEL: @ws_buf_ptrs_default
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ws_buf_ptrs_default anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ws_buf_ptrs_default 这样的标签用于锚定匹配范围。

### Lines 1329-1329
```mlir
1329|   tt.func public @ws_buf_ptrs_default(%arg0: !tt.tensordesc<32x32xf32, #shared>) {
```
**EN:** This function-oriented block defines or enters `ws_buf_ptrs_default`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ws_buf_ptrs_default` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1330-1334
```mlir
1330|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, 32768, 65536, 0], [{{.*}}], shared_mem
1331|     // CHECK-DAG: tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem
1332|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>
1333|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1334|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1335-1352
```mlir
1335|     // CHECK: tti.experimental_lock_acquire
1336|     // CHECK: arith.constant 1 : i32
1337|     // CHECK: tt.call @__triton_consan_set_active_mask
1338|     // CHECK: tti.experimental_lock_release
1339|     ttg.warp_specialize(%smem, %bar) attributes {actualRegisters = array<i32: 480, 32>, allocation.offset = 512 : i32, requestedRegisters = array<i32: 32>, warpGroupStartIds = array<i32: 4>}
1340|     default {
1341|       %c0_i32 = arith.constant 0 : i32
1342|       %1 = ttg.memdesc_index %smem[%c0_i32] : !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1343|       ttg.local_load %1 : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16>
1344|       ttg.warp_yield
1345|     }
1346|     partition0(%arg1: !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<1xi64, #shared1, #smem, mutable>) num_warps(4) {
1347|       ttg.warp_return
1348|     } : (!ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>) -> ()
1349|     tt.return
1350|   }
1351| }
1352| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize, constants, ttg.memdesc_index, local/shared memory loads, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize、常量、ttg.memdesc_index、本地/共享内存加载、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1353-1353
```mlir
1353| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1354-1359
```mlir
1354| 
1355| 
1356| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1357| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1358| #smem = #ttg.shared_memory
1359| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1360-1360
```mlir
1360| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1361-1361
```mlir
1361|   // CHECK-LABEL: @ws_buf_ptrs_partition0
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ws_buf_ptrs_partition0 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ws_buf_ptrs_partition0 这样的标签用于锚定匹配范围。

### Lines 1362-1362
```mlir
1362|   tt.func public @ws_buf_ptrs_partition0(%arg0: !tt.tensordesc<32x32xf32, #shared>) {
```
**EN:** This function-oriented block defines or enters `ws_buf_ptrs_partition0`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ws_buf_ptrs_partition0` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1363-1367
```mlir
1363|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, 32768, 65536, 0], [{{.*}}], shared_mem
1364|     // CHECK-DAG: tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem
1365|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>
1366|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1367|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttng.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttng.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1368-1385
```mlir
1368|     // CHECK: tti.experimental_lock_acquire
1369|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
1370|     // CHECK: %[[THREAD_MASK:.*]] = arith.constant 2 : i64
1371|     // CHECK: tt.call @__triton_consan_copy_write_visibility{{.*}}(%[[THREAD_BIT]], %[[THREAD_MASK]]
1372|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
1373|     // CHECK: tt.call @__triton_consan_copy_read_visibility{{.*}}(%[[THREAD_BIT]]
1374|     ttg.warp_specialize(%smem, %bar) attributes {actualRegisters = array<i32: 480, 32>, allocation.offset = 512 : i32, requestedRegisters = array<i32: 32>, warpGroupStartIds = array<i32: 4>}
1375|     default {
1376|       ttg.warp_yield
1377|     }
1378|     partition0(%arg1: !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<1xi64, #shared1, #smem, mutable>) num_warps(4) {
1379|       %c0_i32 = arith.constant 0 : i32
1380|       %1 = ttg.memdesc_index %arg1[%c0_i32] : !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1381|       ttg.local_load %1 : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16>
1382|       ttg.warp_return
1383|     } : (!ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>) -> ()
1384|     tt.return
1385|   }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize, ttg.warp_yield, constants, ttg.memdesc_index, local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize、ttg.warp_yield、常量、ttg.memdesc_index、本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1386-1387
```mlir
1386| }
1387| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 1388-1388
```mlir
1388| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1389-1394
```mlir
1389| 
1390| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1391| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1392| #smem = #ttg.shared_memory
1393| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
1394| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1395-1395
```mlir
1395| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1396-1396
```mlir
1396|   // CHECK-LABEL: @ws_wait_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ws_wait_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ws_wait_barrier 这样的标签用于锚定匹配范围。

### Lines 1397-1400
```mlir
1397|   tt.func public @ws_wait_barrier() {
1398|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
1399|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1400|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `ws_wait_barrier`. Within it, the test exercises tt.func, shared/local memory allocation, ttng.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ws_wait_barrier` 为核心。测试在其中演示 tt.func、共享/本地内存分配、ttng.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1401-1404
```mlir
1401|     // CHECK: %[[ACTIVE_MASK:.*]] = arith.constant 5 : i32
1402|     // CHECK: tt.call @__triton_consan_set_active_mask{{.*}}(%[[ACTIVE_MASK]],
1403|     ttg.warp_specialize(%smem, %bar) attributes {actualRegisters = array<i32: 480, 32>, allocation.offset = 512 : i32, requestedRegisters = array<i32: 32>, warpGroupStartIds = array<i32: 4>}
1404|     default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1405-1414
```mlir
1405|       // CHECK: tti.experimental_lock_acquire
1406|       // CHECK: tt.call @__triton_consan_set_waiting
1407|       // CHECK: tt.call @__triton_consan_check_all_active_waiting
1408|       // CHECK: tti.experimental_lock_release
1409|       %c0_i32 = arith.constant 0 : i32
1410|       %true = arith.constant true
1411|       ttng.wait_barrier %bar, %c0_i32, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1412|       ttg.warp_yield
1413|     }
1414|     partition0(%arg1: !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<1xi64, #shared1, #smem, mutable>) num_warps(4) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, ttng.wait_barrier, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、ttng.wait_barrier、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1415-1428
```mlir
1415|       // CHECK: partition0
1416|       // CHECK: tti.experimental_lock_acquire
1417|       // CHECK: tt.call @__triton_consan_set_waiting
1418|       // CHECK: tt.call @__triton_consan_check_all_active_waiting
1419|       // CHECK: tti.experimental_lock_release
1420|       %c0_i32 = arith.constant 0 : i32
1421|       %true = arith.constant true
1422|       ttng.wait_barrier %arg2, %c0_i32, %true : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
1423|       ttg.warp_return
1424|     } : (!ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>) -> ()
1425|     tt.return
1426|   }
1427| }
1428| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, ttng.wait_barrier, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、ttng.wait_barrier、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1429-1429
```mlir
1429| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1430-1435
```mlir
1430| 
1431| 
1432| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1433| #smem = #ttg.shared_memory
1434| #tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>
1435| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1436-1436
```mlir
1436| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1437-1437
```mlir
1437|   // CHECK-LABEL: @alias_matrix_shared
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alias_matrix_shared anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alias_matrix_shared 这样的标签用于锚定匹配范围。

### Lines 1438-1438
```mlir
1438|   tt.func public @alias_matrix_shared() {
```
**EN:** This function-oriented block defines or enters `alias_matrix_shared`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alias_matrix_shared` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1439-1450
```mlir
1439|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, 16], [128, 128], shared_mem : tensor<2xi64
1440|     // CHECK-DAG: arith.constant dense<true> : tensor<2x2xi1
1441|     %buf0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
1442|     %buf1 = ttg.local_alloc {allocation.offset = 16 : i32} : () -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
1443|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
1444|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
1445|     ttg.local_load %buf0 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
1446|     ttg.local_load %buf1 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
1447|     tt.return
1448|   }
1449| }
1450| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory loads, ttng.init_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存加载、ttng.init_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1451-1451
```mlir
1451| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1452-1456
```mlir
1452| 
1453| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1454| #smem = #ttg.shared_memory
1455| #tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>
1456| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1457-1457
```mlir
1457| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1458-1458
```mlir
1458|   // CHECK-LABEL: @alias_matrix_shared_indexed
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alias_matrix_shared_indexed anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alias_matrix_shared_indexed 这样的标签用于锚定匹配范围。

### Lines 1459-1461
```mlir
1459|   tt.func public @alias_matrix_shared_indexed() {
1460|     %c0_i32 = arith.constant 0 : i32
1461|     %c1_i32 = arith.constant 1 : i32
```
**EN:** This function-oriented block defines or enters `alias_matrix_shared_indexed`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alias_matrix_shared_indexed` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1462-1474
```mlir
1462|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, 128], [128, 128], shared_mem : tensor<2xi64
1463|     // CHECK-NOT: arith.constant dense<{{\[\[true, false\], \[false, true\]\]}}> : tensor<2x2xi1
1464|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<2x32xf32, #shared, #smem, mutable>
1465|     %buf0 = ttg.memdesc_index %smem[%c0_i32] : !ttg.memdesc<2x32xf32, #shared, #smem, mutable> -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
1466|     %buf1 = ttg.memdesc_index %smem[%c1_i32] : !ttg.memdesc<2x32xf32, #shared, #smem, mutable> -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
1467|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
1468|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
1469|     ttg.local_load %buf0 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
1470|     ttg.local_load %buf1 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
1471|     tt.return
1472|   }
1473| }
1474| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttg.memdesc_index, local/shared memory loads, ttng.init_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttg.memdesc_index、本地/共享内存加载、ttng.init_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1475-1475
```mlir
1475| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1476-1480
```mlir
1476| 
1477| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1478| #smem = #ttg.shared_memory
1479| #tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>
1480| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1481-1481
```mlir
1481| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1482-1482
```mlir
1482|   // CHECK-LABEL: @alias_matrix_shared_subslice
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alias_matrix_shared_subslice anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alias_matrix_shared_subslice 这样的标签用于锚定匹配范围。

### Lines 1483-1483
```mlir
1483|   tt.func public @alias_matrix_shared_subslice() {
```
**EN:** This function-oriented block defines or enters `alias_matrix_shared_subslice`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alias_matrix_shared_subslice` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1484-1495
```mlir
1484|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, 128], [256, 128], shared_mem : tensor<2xi64
1485|     // CHECK-DAG: arith.constant dense<true> : tensor<2x2xi1
1486|     %buf0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<64xf32, #shared, #smem, mutable>
1487|     %buf1 = ttg.memdesc_subslice %buf0 [32] : !ttg.memdesc<64xf32, #shared, #smem, mutable> -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
1488|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
1489|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
1490|     ttg.local_load %buf0 : !ttg.memdesc<64xf32, #shared, #smem, mutable> -> tensor<64xf32>
1491|     ttg.local_load %buf1 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
1492|     tt.return
1493|   }
1494| }
1495| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory loads, ttg.memdesc_subslice, ttng.init_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存加载、ttg.memdesc_subslice、ttng.init_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1496-1496
```mlir
1496| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1497-1500
```mlir
1497| 
1498| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1499| #smem = #ttg.shared_memory
1500| #tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1501-1501
```mlir
1501| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1502-1502
```mlir
1502|   // CHECK-LABEL: @alias_matrix_tensor
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alias_matrix_tensor anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alias_matrix_tensor 这样的标签用于锚定匹配范围。

### Lines 1503-1503
```mlir
1503|   tt.func public @alias_matrix_tensor() {
```
**EN:** This function-oriented block defines or enters `alias_matrix_tensor`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alias_matrix_tensor` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1504-1515
```mlir
1504|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, 32, 64, 0], [64, 32, 64, 0], tensor_mem : tensor<4xi64
1505|     // CHECK-DAG: arith.constant dense<{{\[\[true, true, false, false\], \[true, true, false, false\], \[false, false, true, false\], \[false, false, false, false\]\]}}> : tensor<4x4xi1
1506|     %buf0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable>
1507|     %buf1 = ttng.tmem_alloc {tensor_memory_col_offset = 64 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable>
1508|     %buf3 = ttng.tmem_subslice %buf0 {N = 32 : i32} : !ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<64x32xf32, #tmem, #ttng.tensor_memory, mutable, 64x64>
1509|     ttng.tmem_load %buf0 : !ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<64x64xf32>
1510|     ttng.tmem_load %buf1 : !ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<64x64xf32>
1511|     ttng.tmem_load %buf3 : !ttg.memdesc<64x32xf32, #tmem, #ttng.tensor_memory, mutable, 64x64> -> tensor<64x32xf32>
1512|     tt.return
1513|   }
1514| }
1515| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tensor-memory allocation, ttng.tmem_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、张量内存分配、ttng.tmem_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1516-1516
```mlir
1516| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1517-1521
```mlir
1517| 
1518| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1519| #smem = #ttg.shared_memory
1520| #tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>
1521| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1522-1522
```mlir
1522| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1523-1523
```mlir
1523|   // CHECK-LABEL: @alias_matrix_mixed
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alias_matrix_mixed anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alias_matrix_mixed 这样的标签用于锚定匹配范围。

### Lines 1524-1524
```mlir
1524|   tt.func public @alias_matrix_mixed() {
```
**EN:** This function-oriented block defines or enters `alias_matrix_mixed`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alias_matrix_mixed` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1525-1537
```mlir
1525|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, 16], [128, 128], shared_mem : tensor<2xi64
1526|     // CHECK-DAG: arith.constant dense<true> : tensor<2x2xi1
1527|     // CHECK-DAG: tti.experimental_buffer_descriptors [0], [64], tensor_mem : tensor<1xi64
1528|     %smem0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
1529|     %smem1 = ttg.local_alloc {allocation.offset = 16 : i32} : () -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
1530|     %tmem0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable>
1531|     ttng.tmem_load %tmem0 : !ttg.memdesc<64x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<64x64xf32>
1532|     ttg.local_load %smem0 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
1533|     ttg.local_load %smem1 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
1534|     tt.return
1535|   }
1536| }
1537| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory loads, tensor-memory allocation, ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存加载、张量内存分配、ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1538-1538
```mlir
1538| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1539-1542
```mlir
1539| 
1540| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
1541| #smem = #ttg.shared_memory
1542| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1543-1543
```mlir
1543| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1544-1544
```mlir
1544|   // CHECK-LABEL: @ws_alias_matrix
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ws_alias_matrix anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ws_alias_matrix 这样的标签用于锚定匹配范围。

### Lines 1545-1547
```mlir
1545|   tt.func public @ws_alias_matrix() {
1546|     // We expect the alias matrix constant to appear once for the default region
1547|     // and once for partition0 when we lower warp_specialize.
```
**EN:** This function-oriented block defines or enters `ws_alias_matrix`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ws_alias_matrix` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1548-1561
```mlir
1548|     // CHECK-DAG: arith.constant dense<true> : tensor<2x2xi1
1549|     %smem0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
1550|     %smem1 = ttg.local_alloc {allocation.offset = 16 : i32} : () -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
1551|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
1552|     ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
1553| 
1554|     ttg.warp_specialize(%smem0, %smem1, %bar) attributes {actualRegisters = array<i32: 32, 32>, allocation.offset = 0 : i32, requestedRegisters = array<i32: 32>, warpGroupStartIds = array<i32: 0>}
1555|     default {
1556|       %c0 = arith.constant 0 : i32
1557|       ttg.local_load %smem0 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
1558|       ttg.local_load %smem1 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
1559|       ttg.warp_yield
1560|     }
1561|     partition0(%arg0: !ttg.memdesc<32xf32, #shared, #smem, mutable>, %arg1: !ttg.memdesc<32xf32, #shared, #smem, mutable>, %arg2: !ttg.memdesc<1xi64, #shared, #smem, mutable>) num_warps(1) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory loads, ttng.init_barrier, ttg.warp_specialize, constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存加载、ttng.init_barrier、ttg.warp_specialize、常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1562-1571
```mlir
1562|       // CHECK: arith.constant dense<true> : tensor<2x2xi1
1563|       %c0 = arith.constant 0 : i32
1564|       ttg.local_load %arg0 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
1565|       ttg.local_load %arg1 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
1566|       ttg.warp_return
1567|     } : (!ttg.memdesc<32xf32, #shared, #smem, mutable>, !ttg.memdesc<32xf32, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared, #smem, mutable>) -> ()
1568|     tt.return
1569|   }
1570| }
1571| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, constants, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、常量、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1572-1572
```mlir
1572| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1573-1578
```mlir
1573| 
1574| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1575| #smem = #ttg.shared_memory
1576| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1577| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
1578| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1579-1579
```mlir
1579| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1580-1581
```mlir
1580|   // CHECK-LABEL: @initialize_uninitialized_allocs
1581|   // NO-INIT-LABEL: @initialize_uninitialized_allocs
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 1582-1582
```mlir
1582|   tt.func public @initialize_uninitialized_allocs() {
```
**EN:** This function-oriented block defines or enters `initialize_uninitialized_allocs`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `initialize_uninitialized_allocs` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1583-1598
```mlir
1583|     // CHECK: %[[SMEM:.*]] = ttg.local_alloc
1584|     // CHECK: %[[SMEM_POISON:.*]] = arith.constant dense<0x7FC00000> : tensor<128x128xf32
1585|     // CHECK: ttg.local_store %[[SMEM_POISON]], %[[SMEM]]
1586|     // CHECK: %[[TMEM:.*]] = ttng.tmem_alloc
1587|     // CHECK: %[[TMEM_POISON:.*]] = arith.constant dense<0x7FC00000> : tensor<128x128xf32
1588|     // CHECK: %[[TRUE:.*]] = arith.constant true
1589|     // CHECK: ttng.tmem_store %[[TMEM_POISON]], %[[TMEM]], %[[TRUE]]
1590|     // NO-INIT-NOT: ttg.local_store
1591|     // NO-INIT-NOT: ttng.tmem_store
1592|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #shared, #smem, mutable>
1593|     %tmem = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1594|     // NO-INIT: tt.return
1595|     tt.return
1596|   }
1597| }
1598| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, tensor-memory allocation, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、张量内存分配、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1599-1599
```mlir
1599| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1600-1604
```mlir
1600| 
1601| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1602| #smem = #ttg.shared_memory
1603| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
1604| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1605-1605
```mlir
1605| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 256 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1606-1606
```mlir
1606|   // CHECK-LABEL: @initialize_multibuffer_allocs
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @initialize_multibuffer_allocs anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @initialize_multibuffer_allocs 这样的标签用于锚定匹配范围。

### Lines 1607-1607
```mlir
1607|   tt.func public @initialize_multibuffer_allocs() {
```
**EN:** This function-oriented block defines or enters `initialize_multibuffer_allocs`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `initialize_multibuffer_allocs` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1608-1623
```mlir
1608|     // CHECK: %[[SMEM:.*]] = ttg.local_alloc
1609|     // CHECK: %[[SMEM_0:.*]] = ttg.memdesc_index %[[SMEM]]
1610|     // CHECK: %[[SMEM_1:.*]] = ttg.memdesc_index %[[SMEM]]
1611|     // CHECK: ttg.local_store {{.*}}, %[[SMEM_0]]
1612|     // CHECK: ttg.local_store {{.*}}, %[[SMEM_1]]
1613|     // CHECK: %[[TMEM:.*]] = ttng.tmem_alloc
1614|     // CHECK: %[[TMEM_0:.*]] = ttg.memdesc_index %[[TMEM]]
1615|     // CHECK: %[[TMEM_1:.*]] = ttg.memdesc_index %[[TMEM]]
1616|     // CHECK: ttng.tmem_store {{.*}}, %[[TMEM_0]],
1617|     // CHECK: ttng.tmem_store {{.*}}, %[[TMEM_1]],
1618|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<2x128x128xf32, #shared, #smem, mutable>
1619|     %tmem = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1620|     tt.return
1621|   }
1622| }
1623| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, tensor-memory allocation, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、张量内存分配、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1624-1624
```mlir
1624| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1625-1630
```mlir
1625| 
1626| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
1627| #smem = #ttg.shared_memory
1628| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
1629| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
1630| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1631-1631
```mlir
1631| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1632-1632
```mlir
1632|   // CHECK-LABEL: @skip_source_backed_allocs
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @skip_source_backed_allocs anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @skip_source_backed_allocs 这样的标签用于锚定匹配范围。

### Lines 1633-1633
```mlir
1633|   tt.func public @skip_source_backed_allocs(%src: tensor<128x128xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `skip_source_backed_allocs`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `skip_source_backed_allocs` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1634-1643
```mlir
1634|     // CHECK: ttg.local_alloc %{{.*}}
1635|     // CHECK-NOT: ttg.local_store
1636|     // CHECK: ttng.tmem_alloc %{{.*}}
1637|     // CHECK-NOT: ttng.tmem_store
1638|     %smem = ttg.local_alloc %src {allocation.offset = 0 : i32} : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #shared, #smem, mutable>
1639|     %tmem = ttng.tmem_alloc %src {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
1640|     tt.return
1641|   }
1642| }
1643| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, tensor-memory allocation, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、张量内存分配、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 1644-1644
```mlir
1644| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 1645-1648
```mlir
1645| 
1646| #shared_cluster_publish = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0], CGALayout = [[1, 0]]}>
1647| #smem_cluster_publish = #ttg.shared_memory
1648| #blocked_cluster_publish = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1], CGALayout = [[1, 0]]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 1649-1649
```mlir
1649| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 1650-1650
```mlir
1650|   // CHECK-LABEL: @cluster_barrier_publish_protocol
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @cluster_barrier_publish_protocol anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @cluster_barrier_publish_protocol 这样的标签用于锚定匹配范围。

### Lines 1651-1652
```mlir
1651|   tt.func public @cluster_barrier_publish_protocol() {
1652|     %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared_cluster_publish, #smem_cluster_publish, mutable>
```
**EN:** This function-oriented block defines or enters `cluster_barrier_publish_protocol`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cluster_barrier_publish_protocol` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 1653-1666
```mlir
1653|     // CHECK: ttg.local_load
1654|     // CHECK-NEXT: ttng.cluster_barrier
1655|     // CHECK: %[[PUB_CTA:.*]] = tti.experimental_cluster_cta_id : i32
1656|     // CHECK: %[[PUB_ZERO:.*]] = arith.constant 0 : i32
1657|     // CHECK: %[[PUB_PRED:.*]] = arith.cmpi eq, %[[PUB_CTA]], %[[PUB_ZERO]] : i32
1658|     // CHECK: tti.experimental_lock_acquire %{{.*}}, %[[PUB_PRED]]
1659|     // CHECK: tt.call @__triton_consan_publish_cluster_visibility{{.*}}(%[[PUB_PRED]],
1660|     // CHECK: tti.experimental_lock_release %{{.*}}, %[[PUB_PRED]]
1661|     // CHECK-NEXT: ttng.cluster_barrier
1662|     ttg.local_load %buf : !ttg.memdesc<32x32xf32, #shared_cluster_publish, #smem_cluster_publish, mutable> -> tensor<32x32xf32, #blocked_cluster_publish>
1663|     ttng.cluster_barrier
1664|     tt.return
1665|   }
1666| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, ttng.cluster_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、ttng.cluster_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-tritoninstrument-prepare-consan-captures="target=nvidia"`, `-tritoninstrument-concurrency-sanitizer`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-tritoninstrument-prepare-consan-captures="target=nvidia"`，`-tritoninstrument-concurrency-sanitizer`
- **EN:** Dominant operations include `tt.func`, `ttg.local_alloc`, `arith.constant`, `module`, `tt.return`, `ttg.local_load`, `ttng.init_barrier`, `ttng.tmem_alloc`, `ttg.async_copy_global_to_local`, `ttng.wait_barrier`.
- **CN:** 主要操作包括 `tt.func`、`ttg.local_alloc`、`arith.constant`、`module`、`tt.return`、`ttg.local_load`、`ttng.init_barrier`、`ttng.tmem_alloc`、`ttg.async_copy_global_to_local`、`ttng.wait_barrier`。
- **EN:** The file contains 54 independently testable section(s). Check styles used: CHECK x413, CHECK-DAG x76, CHECK-LABEL x53, CHECK-NOT x8. Important labels include @single_local_alloc_multi_cta, @two_local_alloc, @three_local_alloc, @three_sub_bufs. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 54 个可独立测试的分段。使用的检查类型：CHECK ×413，CHECK-DAG ×76，CHECK-LABEL ×53，CHECK-NOT ×8。 关键标签包括 @single_local_alloc_multi_cta，@two_local_alloc，@three_local_alloc，@three_sub_bufs。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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