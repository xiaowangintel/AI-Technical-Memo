# amd-consan.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-consan.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritoninstrument-prepare-consan-captures="target=amd", -tritoninstrument-concurrency-sanitizer` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritoninstrument-prepare-consan-captures="target=amd", -tritoninstrument-concurrency-sanitizer` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=amd" -tritoninstrument-concurrency-sanitizer | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=amd" -tritoninstrument-concurrency-sanitizer | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=amd" -tritoninstrument-concurrency-sanitizer | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=amd" -tritoninstrument-concurrency-sanitizer | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -tritoninstrument-prepare-consan-captures="target=amd" -tritoninstrument-concurrency-sanitizer | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-6
```mlir
3| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
4| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
5| #smem = #ttg.shared_memory
6| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 7-7
```mlir
7| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 8-8
```mlir
8|   // CHECK-LABEL: @single_local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @single_local_alloc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @single_local_alloc 这样的标签用于锚定匹配范围。

### Lines 9-9
```mlir
9|   tt.func public @single_local_alloc() {
```
**EN:** This function-oriented block defines or enters `single_local_alloc`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `single_local_alloc` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 10-27
```mlir
10|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
11| 
12|     // CHECK: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
13|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
14| 
15|     // CHECK: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
16|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
17| 
18|     // CHECK: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
19|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
20| 
21|     // CHECK: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
22|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
23|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
24|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
25|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
26|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
27|     tt.return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, amdg.init_barrier, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、amdg.init_barrier、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 28-30
```mlir
28|   }
29| }
30| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 31-31
```mlir
31| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 32-36
```mlir
32| 
33| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
34| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
35| #smem = #ttg.shared_memory
36| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 37-37
```mlir
37| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 38-38
```mlir
38|   // CHECK-LABEL: @two_local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @two_local_alloc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @two_local_alloc 这样的标签用于锚定匹配范围。

### Lines 39-39
```mlir
39|   tt.func public @two_local_alloc() {
```
**EN:** This function-oriented block defines or enters `two_local_alloc`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `two_local_alloc` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 40-57
```mlir
40|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0, 4096], [{{.*}}], shared_mem : tensor<2xi64
41| 
42|     // CHECK: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
43|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
44| 
45|     // CHECK: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
46|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
47| 
48|     // CHECK: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 2 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
49|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
50| 
51|     // CHECK: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
52|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
53|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
54|     %1 = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
55|     %bar = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
56|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
57|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, amdg.init_barrier, local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、amdg.init_barrier、本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 58-62
```mlir
58|     ttg.local_load %1 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
59|     tt.return
60|   }
61| }
62| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 63-63
```mlir
63| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 64-68
```mlir
64| 
65| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
66| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
67| #smem = #ttg.shared_memory
68| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 69-69
```mlir
69| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 70-70
```mlir
70|   // CHECK-LABEL: @three_local_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @three_local_alloc anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @three_local_alloc 这样的标签用于锚定匹配范围。

### Lines 71-71
```mlir
71|   tt.func public @three_local_alloc() {
```
**EN:** This function-oriented block defines or enters `three_local_alloc`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `three_local_alloc` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 72-89
```mlir
72|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0, 4096, 8192, 0], [{{.*}}], shared_mem : tensor<4xi64,
73| 
74|     // CHECK: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
75|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
76| 
77|     // CHECK: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
78|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
79| 
80|     // CHECK: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 4 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
81|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
82| 
83|     // CHECK: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
84|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
85|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
86|     %1 = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
87|     %2 = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
88|     %bar = ttg.local_alloc {allocation.offset = 12288 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
89|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 90-96
```mlir
90|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
91|     ttg.local_load %1 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
92|     ttg.local_load %2 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
93|     tt.return
94|   }
95| }
96| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 97-97
```mlir
97| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 98-102
```mlir
 98| 
 99| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
100| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
101| #smem = #ttg.shared_memory
102| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 103-103
```mlir
103| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 104-104
```mlir
104|   // CHECK-LABEL: @three_sub_bufs
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @three_sub_bufs anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @three_sub_bufs 这样的标签用于锚定匹配范围。

### Lines 105-105
```mlir
105|   tt.func public @three_sub_bufs() {
```
**EN:** This function-oriented block defines or enters `three_sub_bufs`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `three_sub_bufs` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 106-123
```mlir
106|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0, 4096, 8192, 0], [{{.*}}], shared_mem : tensor<4xi64,
107| 
108|     // CHECK: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
109|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
110| 
111|     // CHECK: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
112|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
113| 
114|     // CHECK: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 4 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
115|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
116| 
117|     // CHECK: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 32 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
118|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
119|     %c0_i32 = arith.constant 0 : i32
120|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<3x32x32xf32, #shared, #smem, mutable>
121|     %1 = ttg.memdesc_index %0[%c0_i32] : !ttg.memdesc<3x32x32xf32, #shared, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
122|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
123|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, constants, ttg.memdesc_index, amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、常量、ttg.memdesc_index、amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 124-128
```mlir
124|     ttg.local_load %1 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
125|     tt.return
126|   }
127| }
128| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 129-129
```mlir
129| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 130-134
```mlir
130| 
131| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
132| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
133| #smem = #ttg.shared_memory
134| #blocked = #ttg.blocked<{sizePerThread = [2, 4], threadsPerWarp = [1, 32], warpsPerCTA = [1, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 135-135
```mlir
135| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 136-137
```mlir
136|   // CHECK: #[[READ_BARS_L:.*]] = #ttg.blocked<{sizePerThread = [2, 4], threadsPerWarp = [1, 32], warpsPerCTA = [1, 1], order = [0, 1]}>
137|   // CHECK: @read_bars_alloc
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 138-138
```mlir
138|   tt.func public @read_bars_alloc() {
```
**EN:** This function-oriented block defines or enters `read_bars_alloc`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `read_bars_alloc` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 139-151
```mlir
139|     // CHECK: %[[READ_BARS_G:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
140|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_BARS_G]], %c0_i8
141|     %c0 = arith.constant 0 : i32
142|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<2x32x32xf32, #shared, #smem, mutable>
143|     %bar = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<4x1xi64, #shared1, #smem, mutable>
144|     %bar_sub = ttg.memdesc_index %bar[%c0] : !ttg.memdesc<4x1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
145|     amdg.init_barrier %bar_sub, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
146|     %buf_sub = ttg.memdesc_index %0[%c0] : !ttg.memdesc<2x32x32xf32, #shared, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
147|     ttg.local_load %buf_sub : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
148|     tt.return
149|   }
150| }
151| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttg.memdesc_index, constants, amdg.init_barrier, local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttg.memdesc_index、常量、amdg.init_barrier、本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 152-152
```mlir
152| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 153-156
```mlir
153| 
154| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
155| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
156| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 157-157
```mlir
157| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 158-158
```mlir
158|   // CHECK-LABEL: @async_copy_global_to_local
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_copy_global_to_local anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_copy_global_to_local 这样的标签用于锚定匹配范围。

### Lines 159-159
```mlir
159|   tt.func public @async_copy_global_to_local(%ptr: tensor<32x32x!tt.ptr<f16>, #blocked>) {
```
**EN:** This function-oriented block defines or enters `async_copy_global_to_local`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_global_to_local` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 160-177
```mlir
160|     // CHECK: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
161|     // CHECK: %[[WRT_COMMITS_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
162|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRT_COMMITS_GLOB]], %c0_i8
163| 
164|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A:.*]] :
165|     // CHECK: tt.call @__triton_consan_verify_write_visibility_noalias_nw1{{.*}}(%[[A_I64]]
166|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
167|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
168|     // CHECK: tt.call @__triton_consan_check_outstanding_commits{{.*}}(%[[A_I64]], {{.*}}, %[[THREAD_BIT]], %[[BUFFERS]], %[[WRT_COMMITS_GLOB]]
169|     // CHECK: tt.call @__triton_consan_verify_read_visibility_noalias_nw1
170|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
171|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
172|     // CHECK: tt.call @__triton_consan_stage_access_for_commit_nw1{{.*}}(%[[A_I64]], {{.*}}, %[[THREAD_BIT]], %[[BUFFERS]], %[[WRT_COMMITS_GLOB]]
173|     // CHECK: ttg.async_copy_global_to_local %{{.*}}, %[[A]]
174| 
175|     %shmem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable>
176|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<32x32x!tt.ptr<f16>, #blocked> -> <32x32xf16, #shared, #smem, mutable>
177|     tt.return
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 178-180
```mlir
178|   }
179| }
180| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 181-181
```mlir
181| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 182-186
```mlir
182| 
183| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
184| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
185| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
186| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 187-187
```mlir
187| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 188-188
```mlir
188|   // CHECK-LABEL: @async_copy_global_to_local_with_barriers
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_copy_global_to_local_with_barriers anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_copy_global_to_local_with_barriers 这样的标签用于锚定匹配范围。

### Lines 189-189
```mlir
189|   tt.func public @async_copy_global_to_local_with_barriers(%ptr: tensor<32x32x!tt.ptr<f16>, #blocked>) {
```
**EN:** This function-oriented block defines or enters `async_copy_global_to_local_with_barriers`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_copy_global_to_local_with_barriers` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 190-207
```mlir
190|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
191|     // CHECK-DAG: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
192|     // CHECK-DAG: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
193|     // CHECK-DAG: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
194|     // CHECK-DAG: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
195| 
196|     // CHECK-DAG: %[[WRT_COMMITS_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
197| 
198|     // CHECK: tt.call @__triton_consan_init_barrier_state
199| 
200|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A:.*]] :
201|     // CHECK: tt.call @__triton_consan_verify_write_visibility_noalias{{.*}}(%[[A_I64]]
202|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
203|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
204|     // CHECK: tt.call @__triton_consan_check_outstanding_commits{{.*}}(%[[A_I64]], {{.*}}, %[[THREAD_BIT]], %[[BUFFERS]], %[[WRT_COMMITS_GLOB]]
205|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
206|     // CHECK: tt.call @__triton_consan_verify_read_visibility{{.*}}(%[[A_I64]]
207|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 208-218
```mlir
208|     // CHECK: %[[A_I64:.*]] = tti.experimental_memdesc_to_i32 %[[A]] :
209|     // CHECK: tt.call @__triton_consan_stage_access_for_commit{{.*}}(%[[A_I64]], {{.*}}, %[[THREAD_BIT]], %[[BUFFERS]], %[[WRT_COMMITS_GLOB]]
210|     // CHECK: ttg.async_copy_global_to_local %{{.*}}, %[[A]]
211|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
212|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
213|     %shmem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable>
214|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<32x32x!tt.ptr<f16>, #blocked> -> <32x32xf16, #shared, #smem, mutable>
215|     tt.return
216|   }
217| }
218| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, amdg.init_barrier, ttg.async_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、amdg.init_barrier、ttg.async_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 219-219
```mlir
219| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 220-223
```mlir
220| 
221| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
222| #smem = #ttg.shared_memory
223| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 224-224
```mlir
224| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 225-225
```mlir
225|   // CHECK-LABEL: @async_commit_group
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_commit_group anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_commit_group 这样的标签用于锚定匹配范围。

### Lines 226-226
```mlir
226|   tt.func public @async_commit_group() {
```
**EN:** This function-oriented block defines or enters `async_commit_group`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_commit_group` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 227-234
```mlir
227|     // CHECK: tt.call @__triton_consan_commit_accesses
228|     %shmem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable>
229|     ttg.async_commit_group
230|     ttg.local_load %shmem : !ttg.memdesc<32x32xf16, #shared, #smem, mutable> -> tensor<32x32xf16, #blocked>
231|     tt.return
232|   }
233| }
234| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, async copy commit groups, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、异步拷贝提交组、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 235-235
```mlir
235| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 236-239
```mlir
236| 
237| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
238| #smem = #ttg.shared_memory
239| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 240-240
```mlir
240| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 241-241
```mlir
241|   // CHECK-LABEL: @async_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_wait anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_wait 这样的标签用于锚定匹配范围。

### Lines 242-242
```mlir
242|   tt.func public @async_wait() {
```
**EN:** This function-oriented block defines or enters `async_wait`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_wait` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 243-254
```mlir
243|     // CHECK: tti.experimental_lock_acquire
244|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
245|     // CHECK: %[[THREAD_MASK:.*]] = arith.constant 1 : i64
246|     // CHECK: %[[OUTSTANDING_NUM:.*]] = arith.constant 42 : i32
247|     // CHECK: tt.call @__triton_consan_clear_outstanding_commits_transfer_writes{{.*}}(%[[THREAD_BIT]], %[[THREAD_MASK]], %[[OUTSTANDING_NUM]]
248|     %shmem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable>
249|     ttg.async_wait {num = 42 : i32}
250|     ttg.local_load %shmem : !ttg.memdesc<32x32xf16, #shared, #smem, mutable> -> tensor<32x32xf16, #blocked>
251|     tt.return
252|   }
253| }
254| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, async wait synchronization, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、异步等待同步、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 255-255
```mlir
255| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 256-260
```mlir
256| 
257| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
258| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
259| #smem = #ttg.shared_memory
260| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 261-261
```mlir
261| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 262-262
```mlir
262|   // CHECK-LABEL: @local_alloc_with_src
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @local_alloc_with_src anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @local_alloc_with_src 这样的标签用于锚定匹配范围。

### Lines 263-263
```mlir
263|   tt.func public @local_alloc_with_src(%data: tensor<32x32xf16, #blocked>) {
```
**EN:** This function-oriented block defines or enters `local_alloc_with_src`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_alloc_with_src` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 264-275
```mlir
264|     // CHECK: %[[BUF:.*]] = ttg.local_alloc
265|     // CHECK: %[[BUF_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BUF:.*]] :
266|     // CHECK: tt.call @__triton_consan_verify_write_visibility{{.*}}(%[[BUF_I64]]
267|     // CHECK: %[[BUF_I64:.*]] = tti.experimental_memdesc_to_i32 %[[BUF:.*]] :
268|     // CHECK: tt.call @__triton_consan_verify_read_visibility{{.*}}(%[[BUF_I64]]
269|     %buf = ttg.local_alloc %data {allocation.offset = 0 : i32} : (tensor<32x32xf16, #blocked>) -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable>
270|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
271|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
272|     tt.return
273|   }
274| }
275| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, amdg.init_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、amdg.init_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 276-276
```mlir
276| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 277-279
```mlir
277| 
278| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
279| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 280-280
```mlir
280| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 281-281
```mlir
281|   // CHECK-LABEL: @alias_matrix_shared
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alias_matrix_shared anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alias_matrix_shared 这样的标签用于锚定匹配范围。

### Lines 282-282
```mlir
282|   tt.func public @alias_matrix_shared() {
```
**EN:** This function-oriented block defines or enters `alias_matrix_shared`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alias_matrix_shared` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 283-294
```mlir
283|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, 16], [128, 128], shared_mem : tensor<2xi64
284|     // CHECK-DAG: arith.constant dense<true> : tensor<2x2xi1
285|     %buf0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
286|     %buf1 = ttg.local_alloc {allocation.offset = 16 : i32} : () -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
287|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
288|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
289|     ttg.local_load %buf0 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
290|     ttg.local_load %buf1 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
291|     tt.return
292|   }
293| }
294| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory loads, amdg.init_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存加载、amdg.init_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 295-295
```mlir
295| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 296-298
```mlir
296| 
297| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
298| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 299-299
```mlir
299| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 300-300
```mlir
300|   // CHECK-LABEL: @alias_matrix_shared_indexed
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alias_matrix_shared_indexed anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alias_matrix_shared_indexed 这样的标签用于锚定匹配范围。

### Lines 301-303
```mlir
301|   tt.func public @alias_matrix_shared_indexed() {
302|     %c0_i32 = arith.constant 0 : i32
303|     %c1_i32 = arith.constant 1 : i32
```
**EN:** This function-oriented block defines or enters `alias_matrix_shared_indexed`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alias_matrix_shared_indexed` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 304-316
```mlir
304|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, 128], [128, 128], shared_mem : tensor<2xi64
305|     // CHECK-NOT: arith.constant dense<{{\[\[true, false\], \[false, true\]\]}}> : tensor<2x2xi1
306|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<2x32xf32, #shared, #smem, mutable>
307|     %buf0 = ttg.memdesc_index %smem[%c0_i32] : !ttg.memdesc<2x32xf32, #shared, #smem, mutable> -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
308|     %buf1 = ttg.memdesc_index %smem[%c1_i32] : !ttg.memdesc<2x32xf32, #shared, #smem, mutable> -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
309|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
310|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
311|     ttg.local_load %buf0 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
312|     ttg.local_load %buf1 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
313|     tt.return
314|   }
315| }
316| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, ttg.memdesc_index, local/shared memory loads, amdg.init_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、ttg.memdesc_index、本地/共享内存加载、amdg.init_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 317-317
```mlir
317| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 318-320
```mlir
318| 
319| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
320| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 321-321
```mlir
321| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 322-322
```mlir
322|   // CHECK-LABEL: @alias_matrix_shared_subslice
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @alias_matrix_shared_subslice anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @alias_matrix_shared_subslice 这样的标签用于锚定匹配范围。

### Lines 323-323
```mlir
323|   tt.func public @alias_matrix_shared_subslice() {
```
**EN:** This function-oriented block defines or enters `alias_matrix_shared_subslice`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alias_matrix_shared_subslice` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 324-335
```mlir
324|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, 128], [256, 128], shared_mem : tensor<2xi64
325|     // CHECK-DAG: arith.constant dense<true> : tensor<2x2xi1
326|     %buf0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<64xf32, #shared, #smem, mutable>
327|     %buf1 = ttg.memdesc_subslice %buf0 [32] : !ttg.memdesc<64xf32, #shared, #smem, mutable> -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
328|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
329|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
330|     ttg.local_load %buf0 : !ttg.memdesc<64xf32, #shared, #smem, mutable> -> tensor<64xf32>
331|     ttg.local_load %buf1 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
332|     tt.return
333|   }
334| }
335| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory loads, ttg.memdesc_subslice, amdg.init_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存加载、ttg.memdesc_subslice、amdg.init_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 336-336
```mlir
336| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 337-341
```mlir
337| 
338| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
339| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
340| #smem = #ttg.shared_memory
341| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 342-342
```mlir
342| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 343-343
```mlir
343|   // CHECK-LABEL: @amdg_init_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @amdg_init_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @amdg_init_barrier 这样的标签用于锚定匹配范围。

### Lines 344-346
```mlir
344|   tt.func public @amdg_init_barrier() {
345|     %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
346|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `amdg_init_barrier`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `amdg_init_barrier` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 347-348
```mlir
347|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
348|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 349-354
```mlir
349|     // CHECK: tt.call @__triton_consan_init_barrier_state
350|     ttg.local_load %buf : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
351|     tt.return
352|   }
353| }
354| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 355-355
```mlir
355| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 356-360
```mlir
356| 
357| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
358| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
359| #smem = #ttg.shared_memory
360| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 361-361
```mlir
361| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 362-362
```mlir
362|   // CHECK-LABEL: @amdg_wait_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @amdg_wait_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @amdg_wait_barrier 这样的标签用于锚定匹配范围。

### Lines 363-363
```mlir
363|   tt.func public @amdg_wait_barrier() {
```
**EN:** This function-oriented block defines or enters `amdg_wait_barrier`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `amdg_wait_barrier` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 364-381
```mlir
364|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
365| 
366|     // CHECK-DAG: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
367|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
368| 
369|     // CHECK-DAG: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
370|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
371| 
372|     // CHECK-DAG: %[[BARRIERS:.*]] = tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem : tensor<1xi64
373| 
374|     // CHECK-DAG: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
375|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
376| 
377|     // CHECK-DAG: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
378|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
379|     %c0_i32 = arith.constant 0 : i32
380|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
381|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 382-383
```mlir
382|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
383|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 384-388
```mlir
384|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
385|     // CHECK-DAG: tt.call @__triton_consan_set_waiting
386|     // CHECK-DAG: tt.call @__triton_consan_check_all_active_waiting
387|     // CHECK: amdg.wait_barrier
388|     amdg.wait_barrier %bar, %c0_i32 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.wait_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.wait_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 389-398
```mlir
389|     // CHECK: tti.experimental_lock_acquire
390|     // CHECK: tt.call @__triton_consan_transfer_visible_writes{{.*}}%[[BARRIERS]], %[[WRITE_VISIBILITY_GLOB]], %[[WRITE_TRACKING_GLOB]]
391|     // CHECK: tt.call @__triton_consan_transfer_visible_reads{{.*}}%[[BARRIERS]], %[[READ_VISIBILITY_GLOB]], %[[READ_TRACKING_GLOB]]
392|     // CHECK: tt.call @__triton_consan_clear_waiting
393|     // CHECK: tti.experimental_lock_release
394|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
395|     tt.return
396|   }
397| }
398| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 399-399
```mlir
399| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 400-404
```mlir
400| 
401| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
402| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
403| #smem = #ttg.shared_memory
404| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 405-405
```mlir
405| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 406-406
```mlir
406|   // CHECK-LABEL: @amdg_arrive_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @amdg_arrive_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @amdg_arrive_barrier 这样的标签用于锚定匹配范围。

### Lines 407-407
```mlir
407|   tt.func public @amdg_arrive_barrier() {
```
**EN:** This function-oriented block defines or enters `amdg_arrive_barrier`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `amdg_arrive_barrier` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 408-411
```mlir
408|     // CHECK-DAG: %[[BSTATE_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 4 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i32>
409|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[BSTATE_GLOB]], %c0_i32
410|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
411|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 412-413
```mlir
412|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
413|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 414-427
```mlir
414|     // CHECK: tt.call @__triton_consan_init_barrier_state
415|     // CHECK: tti.experimental_lock_acquire
416|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
417|     // CHECK: tt.call @__triton_consan_track_visible_writes
418|     // CHECK: tt.call @__triton_consan_track_visible_reads
419|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
420|     // CHECK: tt.call @__triton_consan_update_barrier_state
421|     // CHECK: tti.experimental_lock_release
422|     %phase = amdg.arrive_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> i32
423|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
424|     tt.return
425|   }
426| }
427| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.arrive_barrier, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.arrive_barrier、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 428-428
```mlir
428| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 429-431
```mlir
429| 
430| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
431| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 432-432
```mlir
432| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 433-433
```mlir
433|   // CHECK-LABEL: @amdg_wait_barrier_without_init
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @amdg_wait_barrier_without_init anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @amdg_wait_barrier_without_init 这样的标签用于锚定匹配范围。

### Lines 434-436
```mlir
434|   tt.func public @amdg_wait_barrier_without_init() {
435|     %c0_i32 = arith.constant 0 : i32
436|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `amdg_wait_barrier_without_init`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `amdg_wait_barrier_without_init` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 437-443
```mlir
437|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
438|     // CHECK: tt.call @__triton_consan_set_waiting
439|     amdg.wait_barrier %bar, %c0_i32 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
440|     tt.return
441|   }
442| }
443| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.wait_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.wait_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 444-444
```mlir
444| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 445-447
```mlir
445| 
446| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
447| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 448-448
```mlir
448| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 449-449
```mlir
449|   // CHECK-LABEL: @amdg_arrive_barrier_without_init
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @amdg_arrive_barrier_without_init anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @amdg_arrive_barrier_without_init 这样的标签用于锚定匹配范围。

### Lines 450-451
```mlir
450|   tt.func public @amdg_arrive_barrier_without_init() {
451|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `amdg_arrive_barrier_without_init`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `amdg_arrive_barrier_without_init` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 452-459
```mlir
452|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
453|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
454|     // CHECK: tt.call @__triton_consan_update_barrier_state
455|     %phase = amdg.arrive_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> i32
456|     tt.return
457|   }
458| }
459| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.arrive_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.arrive_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 460-460
```mlir
460| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 461-464
```mlir
461| 
462| #shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [32, 32]}>
463| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
464| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 465-465
```mlir
465| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 466-466
```mlir
466|   // CHECK-LABEL: @async_tdm_copy_global_to_local
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tdm_copy_global_to_local anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tdm_copy_global_to_local 这样的标签用于锚定匹配范围。

### Lines 467-467
```mlir
467|   tt.func public @async_tdm_copy_global_to_local(%desc: !tt.tensordesc<32x32xf32>) {
```
**EN:** This function-oriented block defines or enters `async_tdm_copy_global_to_local`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tdm_copy_global_to_local` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 468-485
```mlir
468|     // CHECK-DAG: %[[BUFFERS:.*]] = tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
469| 
470|     // CHECK-DAG: %[[WRITE_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
471|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_VISIBILITY_GLOB]], %c0_i64
472| 
473|     // CHECK-DAG: %[[READ_VISIBILITY_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 16 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
474|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_VISIBILITY_GLOB]], %c0_i64
475| 
476|     // CHECK-DAG: %[[BARRIERS:.*]] = tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem : tensor<1xi64
477|     // CHECK-DAG: %[[WRITE_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 1 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i8>
478|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[WRITE_TRACKING_GLOB]], %c0_i8
479| 
480|     // CHECK-DAG: %[[READ_TRACKING_GLOB:.*]] = ttg.global_scratch_alloc {alignment = 16 : i32, nbytes = 8 : i32, shared_cluster_state, third_party_allocation, tt.divisibility = 16 : i64} : !tt.ptr<i64>
481|     // CHECK: call {{.*}}fill_global_tensor{{.*}}(%[[READ_TRACKING_GLOB]], %c0_i64
482|     %c0_i32 = arith.constant 0 : i32
483|     %pred = arith.constant 1 : i32
484|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
485|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 486-487
```mlir
486|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
487|     amdg.init_barrier %bar, 8 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 488-503
```mlir
488|     // CHECK: tt.call @__triton_consan_init_barrier_state
489|     // CHECK: tt.call @__triton_consan_verify_write_visibility
490|     // CHECK: tt.call @__triton_consan_verify_read_visibility
491|     // CHECK: tt.call @__triton_consan_set_write_visibility
492|     // CHECK: tt.call @__triton_consan_clear_write_tracking
493|     // CHECK: tt.call @__triton_consan_clear_read_visibility
494|     // CHECK: tt.call @__triton_consan_clear_read_tracking
495|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
496|     // CHECK: tt.call @__triton_consan_track_visible_writes
497|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
498|     // CHECK: tt.call @__triton_consan_update_barrier_state
499|     %1 = amdg.async_tdm_copy_global_to_local %desc[%c0_i32, %c0_i32] into %0, pred = %pred, barrier = %bar : !tt.tensordesc<32x32xf32>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
500|     tt.return
501|   }
502| }
503| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 504-504
```mlir
504| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 505-509
```mlir
505| 
506| #shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [32, 32]}>
507| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
508| #smem = #ttg.shared_memory
509| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 510-510
```mlir
510| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 511-511
```mlir
511|   // CHECK-LABEL: @async_tdm_copy_global_to_local_two_bufs_one_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tdm_copy_global_to_local_two_bufs_one_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tdm_copy_global_to_local_two_bufs_one_barrier 这样的标签用于锚定匹配范围。

### Lines 512-520
```mlir
512|   tt.func public @async_tdm_copy_global_to_local_two_bufs_one_barrier(
513|       %a: !tt.tensordesc<32x32xf32>,
514|       %b: !tt.tensordesc<32x32xf32>) {
515|     %c0_i32 = arith.constant 0 : i32
516|     %pred = arith.constant 1 : i32
517| 
518|     %a_smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
519|     %b_smem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
520|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tdm_copy_global_to_local_two_bufs_one_barrier`. Within it, the test exercises shared/local memory allocation, tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tdm_copy_global_to_local_two_bufs_one_barrier` 为核心。测试在其中演示 共享/本地内存分配、tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 521-523
```mlir
521|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
522|     amdg.init_barrier %bar, 16 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
523| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 524-525
```mlir
524|     // CHECK: tt.call @__triton_consan_init_barrier_state
525|     // First TDM copy: full effects + barrier instrumentation
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 526-538
```mlir
526|     // CHECK: tt.call @__triton_consan_verify_write_visibility
527|     // CHECK: tt.call @__triton_consan_verify_read_visibility
528|     // CHECK: tt.call @__triton_consan_set_write_visibility
529|     // CHECK: tt.call @__triton_consan_clear_write_tracking
530|     // CHECK: tt.call @__triton_consan_clear_read_visibility
531|     // CHECK: tt.call @__triton_consan_clear_read_tracking
532|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
533|     // CHECK: tt.call @__triton_consan_track_visible_writes
534|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
535|     // CHECK: tt.call @__triton_consan_update_barrier_state
536|     %0 = amdg.async_tdm_copy_global_to_local %a[%c0_i32, %c0_i32] into %a_smem, pred = %pred, barrier = %bar : !tt.tensordesc<32x32xf32>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
537| 
538|     // Second TDM copy: same full instrumentation
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_global_to_local. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_global_to_local。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 539-556
```mlir
539|     // CHECK: tt.call @__triton_consan_verify_write_visibility
540|     // CHECK: tt.call @__triton_consan_verify_read_visibility
541|     // CHECK: tt.call @__triton_consan_set_write_visibility
542|     // CHECK: tt.call @__triton_consan_clear_write_tracking
543|     // CHECK: tt.call @__triton_consan_clear_read_visibility
544|     // CHECK: tt.call @__triton_consan_clear_read_tracking
545|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
546|     // CHECK: tt.call @__triton_consan_track_visible_writes
547|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
548|     // CHECK: tt.call @__triton_consan_update_barrier_state
549|     %1 = amdg.async_tdm_copy_global_to_local %b[%c0_i32, %c0_i32] into %b_smem, pred = %pred, barrier = %bar : !tt.tensordesc<32x32xf32>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
550| 
551|     %c0_phase = arith.constant 0 : i32
552|     amdg.wait_barrier %bar, %c0_phase : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
553| 
554|     %va = ttg.local_load %a_smem : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
555|     %vb = ttg.local_load %b_smem : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
556|     %_ = arith.addf %va, %vb : tensor<32x32xf32, #blocked>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, amdg.async_tdm_copy_global_to_local, constants, amdg.wait_barrier, floating-point additions. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、amdg.async_tdm_copy_global_to_local、常量、amdg.wait_barrier、浮点加法。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 557-560
```mlir
557|     tt.return
558|   }
559| }
560| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 561-561
```mlir
561| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 562-564
```mlir
562| 
563| #shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [32, 32]}>
564| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 565-565
```mlir
565| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 566-566
```mlir
566|   // CHECK-LABEL: @async_tdm_copy_global_to_local_no_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tdm_copy_global_to_local_no_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tdm_copy_global_to_local_no_barrier 这样的标签用于锚定匹配范围。

### Lines 567-570
```mlir
567|   tt.func public @async_tdm_copy_global_to_local_no_barrier(%desc: !tt.tensordesc<32x32xf32>) {
568|     %c0_i32 = arith.constant 0 : i32
569|     %pred = arith.constant 1 : i32
570|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tdm_copy_global_to_local_no_barrier`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tdm_copy_global_to_local_no_barrier` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 571-582
```mlir
571|     // CHECK: tt.call @__triton_consan_verify_write_visibility
572|     // CHECK: tt.call @__triton_consan_check_outstanding_commits_excl_self_noalias
573|     // CHECK: tt.call @__triton_consan_verify_read_visibility
574|     // CHECK: tt.call @__triton_consan_check_outstanding_commits_excl_self_noalias
575|     // CHECK: tt.call @__triton_consan_stage_access_for_commit
576|     // CHECK: tt.call @__triton_consan_commit_accesses
577|     // CHECK-NOT: tt.call @__triton_consan_verify_barrier_arrive
578|     %1 = amdg.async_tdm_copy_global_to_local %desc[%c0_i32, %c0_i32] into %0, pred = %pred : !tt.tensordesc<32x32xf32> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
579|     tt.return
580|   }
581| }
582| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 583-583
```mlir
583| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 584-587
```mlir
584| 
585| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
586| #smem = #ttg.shared_memory
587| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 588-588
```mlir
588| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 589-589
```mlir
589|   // CHECK-LABEL: @async_tdm_copy_local_to_global
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tdm_copy_local_to_global anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tdm_copy_local_to_global 这样的标签用于锚定匹配范围。

### Lines 590-595
```mlir
590|   tt.func public @async_tdm_copy_local_to_global(%desc: !tt.tensordesc<32x32xf32>, %ptr: tensor<128x128x!tt.ptr<f16>, #blocked>) {
591|     %c0_i32 = arith.constant 0 : i32
592|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
593|     %shmem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
594|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<128x128x!tt.ptr<f16>, #blocked> -> <128x128xf16, #shared, #smem, mutable>
595| 
```
**EN:** This function-oriented block defines or enters `async_tdm_copy_local_to_global`. Within it, the test exercises tt.func, shared/local memory allocation, constants, ttg.async_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tdm_copy_local_to_global` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量、ttg.async_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 596-605
```mlir
596|     // CHECK: tt.call @__triton_consan_verify_write_visibility
597|     // CHECK: tt.call @__triton_consan_check_outstanding_commits_noalias
598|     // CHECK: tt.call @__triton_consan_check_outstanding_commits_excl_self_noalias
599|     // CHECK: tt.call @__triton_consan_stage_access_for_commit
600|     // CHECK: tt.call @__triton_consan_commit_accesses
601|     amdg.async_tdm_copy_local_to_global %desc[%c0_i32, %c0_i32] from %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> !tt.tensordesc<32x32xf32>
602|     tt.return
603|   }
604| }
605| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_local_to_global, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_local_to_global、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 606-606
```mlir
606| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 607-609
```mlir
607| 
608| #shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [32, 32]}>
609| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 610-610
```mlir
610| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 611-611
```mlir
611|   // CHECK-LABEL: @async_tdm_load_store_no_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tdm_load_store_no_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tdm_load_store_no_barrier 这样的标签用于锚定匹配范围。

### Lines 612-615
```mlir
612|   tt.func public @async_tdm_load_store_no_barrier(%in_desc: !tt.tensordesc<32x32xf32>, %out_desc: !tt.tensordesc<32x32xf32>) {
613|     %c0_i32 = arith.constant 0 : i32
614|     %pred = arith.constant 1 : i32
615|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tdm_load_store_no_barrier`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tdm_load_store_no_barrier` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 616-620
```mlir
616|     // CHECK: tt.call @__triton_consan_verify_write_visibility
617|     // CHECK: tt.call @__triton_consan_check_outstanding_commits_excl_self_noalias
618|     // CHECK: tt.call @__triton_consan_stage_access_for_commit
619|     // CHECK: tt.call @__triton_consan_commit_accesses
620|     %1 = amdg.async_tdm_copy_global_to_local %in_desc[%c0_i32, %c0_i32] into %0, pred = %pred : !tt.tensordesc<32x32xf32> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_global_to_local. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_global_to_local。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 621-628
```mlir
621|     // CHECK: tt.call @__triton_consan_check_outstanding_commits_excl_self_noalias
622|     // CHECK: tt.call @__triton_consan_stage_access_for_commit
623|     // CHECK: tt.call @__triton_consan_commit_accesses
624|     amdg.async_tdm_copy_local_to_global %out_desc[%c0_i32, %c0_i32] from %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> !tt.tensordesc<32x32xf32>
625|     tt.return
626|   }
627| }
628| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_local_to_global, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_local_to_global、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 629-629
```mlir
629| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 630-633
```mlir
630| 
631| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
632| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
633| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 634-634
```mlir
634| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 635-635
```mlir
635|   // CHECK-LABEL: @async_tdm_copy_local_to_global_with_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tdm_copy_local_to_global_with_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tdm_copy_local_to_global_with_barrier 这样的标签用于锚定匹配范围。

### Lines 636-639
```mlir
636|   tt.func public @async_tdm_copy_local_to_global_with_barrier(%desc: !tt.tensordesc<32x32xf32>) {
637|     %c0_i32 = arith.constant 0 : i32
638|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
639|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tdm_copy_local_to_global_with_barrier`. Within it, the test exercises tt.func, shared/local memory allocation, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tdm_copy_local_to_global_with_barrier` 为核心。测试在其中演示 tt.func、共享/本地内存分配、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 640-641
```mlir
640|     // CHECK: tt.call @__triton_consan_verify_barrier_can_init
641|     amdg.init_barrier %bar, 8 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 642-655
```mlir
642|     // CHECK: tt.call @__triton_consan_init_barrier_state
643|     // CHECK: tt.call @__triton_consan_verify_write_visibility
644|     // CHECK: tt.call @__triton_consan_set_read_visibility
645|     // CHECK: tt.call @__triton_consan_verify_barrier_initialized
646|     // CHECK: tt.call @__triton_consan_track_visible_writes
647|     // CHECK: tt.call @__triton_consan_track_visible_reads
648|     // CHECK: tt.call @__triton_consan_verify_barrier_arrive
649|     // CHECK: tt.call @__triton_consan_update_barrier_state
650|     // CHECK-NOT: tt.call @__triton_consan_stage_access_for_commit
651|     amdg.async_tdm_copy_local_to_global %desc[%c0_i32, %c0_i32] from %0, barrier = %bar : !ttg.memdesc<32x32xf32, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !tt.tensordesc<32x32xf32>
652|     tt.return
653|   }
654| }
655| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_copy_local_to_global, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_copy_local_to_global、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 656-656
```mlir
656| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 657-660
```mlir
657| 
658| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
659| #smem = #ttg.shared_memory
660| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 661-661
```mlir
661| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 662-662
```mlir
662|   // CHECK-LABEL: @async_tdm_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tdm_wait anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tdm_wait 这样的标签用于锚定匹配范围。

### Lines 663-665
```mlir
663|   tt.func public @async_tdm_wait() {
664|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
665| 
```
**EN:** This function-oriented block defines or enters `async_tdm_wait`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tdm_wait` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 666-673
```mlir
666|     // CHECK: tt.call @__triton_consan_clear_outstanding_commits_transfer_both
667|     amdg.async_tdm_wait {num = 0 : i32}
668| 
669|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
670|     tt.return
671|   }
672| }
673| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_wait, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_wait、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 674-674
```mlir
674| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 675-678
```mlir
675| 
676| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
677| #smem = #ttg.shared_memory
678| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 679-679
```mlir
679| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 680-680
```mlir
680|   // CHECK-LABEL: @async_tdm_intrinsic_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @async_tdm_intrinsic_wait anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @async_tdm_intrinsic_wait 这样的标签用于锚定匹配范围。

### Lines 681-683
```mlir
681|   tt.func public @async_tdm_intrinsic_wait() {
682|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
683| 
```
**EN:** This function-oriented block defines or enters `async_tdm_intrinsic_wait`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tdm_intrinsic_wait` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 684-691
```mlir
684|     // CHECK: tt.call @__triton_consan_clear_outstanding_commits_transfer_both
685|     amdg.async_tdm_intrinsic_wait {"ttg.num_tdm_ops" = 2 : i64, count = 2 : i32}
686| 
687|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
688|     tt.return
689|   }
690| }
691| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_intrinsic_wait, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_intrinsic_wait、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 692-692
```mlir
692| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 693-696
```mlir
693| 
694| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
695| #smem = #ttg.shared_memory
696| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 697-697
```mlir
697| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 698-698
```mlir
698|   // CHECK-LABEL: @amdg_async_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @amdg_async_wait anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @amdg_async_wait 这样的标签用于锚定匹配范围。

### Lines 699-699
```mlir
699|   tt.func public @amdg_async_wait() {
```
**EN:** This function-oriented block defines or enters `amdg_async_wait`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `amdg_async_wait` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 700-711
```mlir
700|     // CHECK: tti.experimental_lock_acquire
701|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
702|     // CHECK: %[[THREAD_MASK:.*]] = arith.constant 1 : i64
703|     // CHECK: %[[OUTSTANDING_NUM:.*]] = arith.constant 42 : i32
704|     // CHECK: tt.call @__triton_consan_clear_outstanding_commits_transfer_writes{{.*}}(%[[THREAD_BIT]], %[[THREAD_MASK]], %[[OUTSTANDING_NUM]]
705|     %shmem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable>
706|     amdg.async_wait {"ttg.num_commit_groups" = 42 : i64, num_inst = 42 : i32}
707|     ttg.local_load %shmem : !ttg.memdesc<32x32xf16, #shared, #smem, mutable> -> tensor<32x32xf16, #blocked>
708|     tt.return
709|   }
710| }
711| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, amdg.async_wait, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、amdg.async_wait、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 712-712
```mlir
712| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 713-716
```mlir
713| 
714| #shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [32, 32]}>
715| #smem = #ttg.shared_memory
716| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 717-717
```mlir
717| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 718-718
```mlir
718|   // CHECK-LABEL: @tdm_load_no_barrier_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tdm_load_no_barrier_wait anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tdm_load_no_barrier_wait 这样的标签用于锚定匹配范围。

### Lines 719-723
```mlir
719|   tt.func public @tdm_load_no_barrier_wait(%desc: !tt.tensordesc<32x32xf32>) {
720|     %c0_i32 = arith.constant 0 : i32
721|     %pred = arith.constant 1 : i32
722|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
723|     %1 = amdg.async_tdm_copy_global_to_local %desc[%c0_i32, %c0_i32] into %0, pred = %pred : !tt.tensordesc<32x32xf32> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `tdm_load_no_barrier_wait`. Within it, the test exercises tt.func, constants, shared/local memory allocation, amdg.async_tdm_copy_global_to_local, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tdm_load_no_barrier_wait` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、amdg.async_tdm_copy_global_to_local，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 724-730
```mlir
724|     // CHECK: tt.call @__triton_consan_clear_outstanding_commits_transfer_both
725|     amdg.async_tdm_wait {num = 0 : i32}
726|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
727|     tt.return
728|   }
729| }
730| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_wait, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_wait、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 731-731
```mlir
731| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 732-735
```mlir
732| 
733| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
734| #smem = #ttg.shared_memory
735| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 736-736
```mlir
736| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 737-737
```mlir
737|   // CHECK-LABEL: @tdm_store_no_barrier_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tdm_store_no_barrier_wait anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tdm_store_no_barrier_wait 这样的标签用于锚定匹配范围。

### Lines 738-741
```mlir
738|   tt.func public @tdm_store_no_barrier_wait(%desc: !tt.tensordesc<32x32xf32>) {
739|     %c0_i32 = arith.constant 0 : i32
740|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
741|     amdg.async_tdm_copy_local_to_global %desc[%c0_i32, %c0_i32] from %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> !tt.tensordesc<32x32xf32>
```
**EN:** This function-oriented block defines or enters `tdm_store_no_barrier_wait`. Within it, the test exercises tt.func, constants, shared/local memory allocation, amdg.async_tdm_copy_local_to_global, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tdm_store_no_barrier_wait` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、amdg.async_tdm_copy_local_to_global，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 742-748
```mlir
742|     // CHECK: tt.call @__triton_consan_clear_outstanding_commits_transfer_both
743|     amdg.async_tdm_wait {num = 0 : i32}
744|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
745|     tt.return
746|   }
747| }
748| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_wait, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_wait、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 749-749
```mlir
749| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 750-753
```mlir
750| 
751| #shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [32, 32]}>
752| #smem = #ttg.shared_memory
753| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 754-754
```mlir
754| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 755-755
```mlir
755|   // CHECK-LABEL: @tdm_load_store_no_barrier_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @tdm_load_store_no_barrier_wait anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @tdm_load_store_no_barrier_wait 这样的标签用于锚定匹配范围。

### Lines 756-761
```mlir
756|   tt.func public @tdm_load_store_no_barrier_wait(%desc: !tt.tensordesc<32x32xf32>) {
757|     %c0_i32 = arith.constant 0 : i32
758|     %pred = arith.constant 1 : i32
759|     %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
760|     %1 = amdg.async_tdm_copy_global_to_local %desc[%c0_i32, %c0_i32] into %0, pred = %pred : !tt.tensordesc<32x32xf32> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
761|     amdg.async_tdm_copy_local_to_global %desc[%c0_i32, %c0_i32] from %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> !tt.tensordesc<32x32xf32>
```
**EN:** This function-oriented block defines or enters `tdm_load_store_no_barrier_wait`. Within it, the test exercises tt.func, constants, shared/local memory allocation, amdg.async_tdm_copy_global_to_local, amdg.async_tdm_copy_local_to_global, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tdm_load_store_no_barrier_wait` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配、amdg.async_tdm_copy_global_to_local、amdg.async_tdm_copy_local_to_global，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 762-768
```mlir
762|     // CHECK: tt.call @__triton_consan_clear_outstanding_commits_transfer_both
763|     amdg.async_tdm_wait {num = 0 : i32}
764|     ttg.local_load %0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
765|     tt.return
766|   }
767| }
768| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on amdg.async_tdm_wait, local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 amdg.async_tdm_wait、本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 769-769
```mlir
769| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 770-774
```mlir
770| 
771| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
772| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
773| #smem = #ttg.shared_memory
774| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 775-775
```mlir
775| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 776-776
```mlir
776|   // CHECK-LABEL: @local_load_barriers
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @local_load_barriers anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @local_load_barriers 这样的标签用于锚定匹配范围。

### Lines 777-777
```mlir
777|   tt.func public @local_load_barriers() {
```
**EN:** This function-oriented block defines or enters `local_load_barriers`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_load_barriers` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 778-781
```mlir
778|     // CHECK: tti.experimental_buffer_descriptors
779|     %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
780|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
781|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 782-790
```mlir
782|     // CHECK: tti.experimental_lock_acquire
783|     // CHECK: tt.call @__triton_consan_verify_write_visibility
784|     // CHECK: tt.call @__triton_consan_set_read_visibility
785|     // CHECK: tti.experimental_lock_release
786|     ttg.local_load %buf : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
787|     tt.return
788|   }
789| }
790| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 791-791
```mlir
791| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 792-796
```mlir
792| 
793| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
794| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
795| #smem = #ttg.shared_memory
796| #blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 797-797
```mlir
797| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 798-798
```mlir
798|   // CHECK-LABEL: @local_load_barriers_cp_async
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @local_load_barriers_cp_async anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @local_load_barriers_cp_async 这样的标签用于锚定匹配范围。

### Lines 799-799
```mlir
799|   tt.func public @local_load_barriers_cp_async(%ptr: tensor<32x32x!tt.ptr<f16>, #blocked>) {
```
**EN:** This function-oriented block defines or enters `local_load_barriers_cp_async`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `local_load_barriers_cp_async` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 800-805
```mlir
800|     // CHECK: tti.experimental_buffer_descriptors
801|     %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
802|     %shmem = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable>
803|     %bar = ttg.local_alloc {allocation.offset = 8192 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
804|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
805|     ttg.async_copy_global_to_local %ptr, %shmem : tensor<32x32x!tt.ptr<f16>, #blocked> -> <32x32xf16, #shared, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, amdg.init_barrier, ttg.async_copy_global_to_local. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、amdg.init_barrier、ttg.async_copy_global_to_local。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 806-817
```mlir
806|     // CHECK: ttg.async_copy_global_to_local
807|     // CHECK: tti.experimental_lock_acquire
808|     // CHECK: tt.call @__triton_consan_verify_write_visibility
809|     // CHECK: tt.call @__triton_consan_check_outstanding_commits
810|     // CHECK: tt.call @__triton_consan_set_read_visibility
811|     // CHECK: tti.experimental_lock_release
812|     // CHECK: ttg.local_load
813|     ttg.local_load %buf : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32xf32, #blocked>
814|     tt.return
815|   }
816| }
817| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 818-818
```mlir
818| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 819-822
```mlir
819| 
820| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
821| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
822| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 823-823
```mlir
823| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 824-824
```mlir
824|   // CHECK-LABEL: @ws_allocation
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ws_allocation anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ws_allocation 这样的标签用于锚定匹配范围。

### Lines 825-825
```mlir
825|   tt.func public @ws_allocation() {
```
**EN:** This function-oriented block defines or enters `ws_allocation`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ws_allocation` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 826-830
```mlir
826|     // CHECK-DAG: tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem : tensor<1xi64,
827|     // CHECK-DAG: tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
828|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
829|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
830|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 831-838
```mlir
831|     // CHECK: tti.experimental_lock_acquire
832|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
833|     // CHECK: %[[THREAD_MASK:.*]] = arith.constant 2 : i64
834|     // CHECK: tt.call @__triton_consan_copy_write_visibility{{.*}}(%[[THREAD_BIT]], %[[THREAD_MASK]]
835|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
836|     // CHECK: tt.call @__triton_consan_copy_read_visibility{{.*}}(%[[THREAD_BIT]]
837|     ttg.warp_specialize(%smem, %bar) attributes {actualRegisters = array<i32: 480, 32>, allocation.offset = 512 : i32, requestedRegisters = array<i32: 32>, warpGroupStartIds = array<i32: 4>}
838|     default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 839-846
```mlir
839|       // CHECK: tti.experimental_lock_acquire
840|       // CHECK: tt.call @__triton_consan_verify_write_visibility
841|       // CHECK: tt.call @__triton_consan_set_read_visibility
842|       // CHECK: tti.experimental_lock_release
843|       ttg.local_load %smem : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16>
844|       ttg.warp_yield
845|     }
846|     partition0(%arg1: !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<1xi64, #shared1, #smem, mutable>) num_warps(4) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 847-860
```mlir
847|       // CHECK: partition0
848|       // CHECK-DAG: tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem : tensor<1xi64,
849|       // CHECK-DAG: tti.experimental_buffer_descriptors [0], [{{.*}}], shared_mem : tensor<1xi64
850|       // CHECK: tti.experimental_lock_acquire
851|       // CHECK: tt.call @__triton_consan_verify_write_visibility
852|       // CHECK: tt.call @__triton_consan_set_read_visibility
853|       // CHECK: tti.experimental_lock_release
854|       ttg.local_load %arg1 : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16>
855|       ttg.warp_return
856|     } : (!ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>) -> ()
857|     tt.return
858|   }
859| }
860| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 861-861
```mlir
861| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 862-865
```mlir
862| 
863| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
864| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
865| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 866-866
```mlir
866| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 867-867
```mlir
867|   // CHECK-LABEL: @ws_buf_ptrs_default
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ws_buf_ptrs_default anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ws_buf_ptrs_default 这样的标签用于锚定匹配范围。

### Lines 868-868
```mlir
868|   tt.func public @ws_buf_ptrs_default() {
```
**EN:** This function-oriented block defines or enters `ws_buf_ptrs_default`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ws_buf_ptrs_default` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 869-873
```mlir
869|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, {{.*}}], [{{.*}}], shared_mem
870|     // CHECK-DAG: tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem
871|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>
872|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
873|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 874-891
```mlir
874|     // CHECK: tti.experimental_lock_acquire
875|     // CHECK: arith.constant 1 : i32
876|     // CHECK: tt.call @__triton_consan_set_active_mask
877|     // CHECK: tti.experimental_lock_release
878|     ttg.warp_specialize(%smem, %bar) attributes {actualRegisters = array<i32: 480, 32>, allocation.offset = 512 : i32, requestedRegisters = array<i32: 32>, warpGroupStartIds = array<i32: 4>}
879|     default {
880|       %c0_i32 = arith.constant 0 : i32
881|       %1 = ttg.memdesc_index %smem[%c0_i32] : !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
882|       ttg.local_load %1 : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16>
883|       ttg.warp_yield
884|     }
885|     partition0(%arg1: !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<1xi64, #shared1, #smem, mutable>) num_warps(4) {
886|       ttg.warp_return
887|     } : (!ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>) -> ()
888|     tt.return
889|   }
890| }
891| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize, constants, ttg.memdesc_index, local/shared memory loads, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize、常量、ttg.memdesc_index、本地/共享内存加载、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 892-892
```mlir
892| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 893-896
```mlir
893| 
894| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
895| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
896| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 897-897
```mlir
897| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 898-898
```mlir
898|   // CHECK-LABEL: @ws_buf_ptrs_partition0
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ws_buf_ptrs_partition0 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ws_buf_ptrs_partition0 这样的标签用于锚定匹配范围。

### Lines 899-899
```mlir
899|   tt.func public @ws_buf_ptrs_partition0() {
```
**EN:** This function-oriented block defines or enters `ws_buf_ptrs_partition0`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ws_buf_ptrs_partition0` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 900-904
```mlir
900|     // CHECK-DAG: tti.experimental_buffer_descriptors [0, {{.*}}], [{{.*}}], shared_mem
901|     // CHECK-DAG: tti.experimental_buffer_descriptors [65536], [{{.*}}], shared_mem
902|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>
903|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
904|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, amdg.init_barrier. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、amdg.init_barrier。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 905-922
```mlir
905|     // CHECK: tti.experimental_lock_acquire
906|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
907|     // CHECK: %[[THREAD_MASK:.*]] = arith.constant 2 : i64
908|     // CHECK: tt.call @__triton_consan_copy_write_visibility{{.*}}(%[[THREAD_BIT]], %[[THREAD_MASK]]
909|     // CHECK: %[[THREAD_BIT:.*]] = arith.constant 0 : i32
910|     // CHECK: tt.call @__triton_consan_copy_read_visibility{{.*}}(%[[THREAD_BIT]]
911|     ttg.warp_specialize(%smem, %bar) attributes {actualRegisters = array<i32: 480, 32>, allocation.offset = 512 : i32, requestedRegisters = array<i32: 32>, warpGroupStartIds = array<i32: 4>}
912|     default {
913|       ttg.warp_yield
914|     }
915|     partition0(%arg1: !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<1xi64, #shared1, #smem, mutable>) num_warps(4) {
916|       %c0_i32 = arith.constant 0 : i32
917|       %1 = ttg.memdesc_index %arg1[%c0_i32] : !ttg.memdesc<3x128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
918|       ttg.local_load %1 : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16>
919|       ttg.warp_return
920|     } : (!ttg.memdesc<3x128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>) -> ()
921|     tt.return
922|   }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize, ttg.warp_yield, constants, ttg.memdesc_index, local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize、ttg.warp_yield、常量、ttg.memdesc_index、本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 923-924
```mlir
923| }
924| 
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 925-925
```mlir
925| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 926-930
```mlir
926| 
927| #shared = #ttg.swizzled_shared<{vec = 4, perPhase = 4, maxPhase = 4, order = [1, 0]}>
928| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
929| #smem = #ttg.shared_memory
930| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 931-931
```mlir
931| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 932-932
```mlir
932|   // CHECK-LABEL: @ws_wait_barrier
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ws_wait_barrier anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ws_wait_barrier 这样的标签用于锚定匹配范围。

### Lines 933-936
```mlir
933|   tt.func public @ws_wait_barrier() {
934|     %smem = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
935|     %bar = ttg.local_alloc {allocation.offset = 65536 : i32} : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
936|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `ws_wait_barrier`. Within it, the test exercises tt.func, shared/local memory allocation, amdg.init_barrier, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ws_wait_barrier` 为核心。测试在其中演示 tt.func、共享/本地内存分配、amdg.init_barrier，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 937-940
```mlir
937|     // CHECK: %[[ACTIVE_MASK:.*]] = arith.constant 5 : i32
938|     // CHECK: tt.call @__triton_consan_set_active_mask{{.*}}(%[[ACTIVE_MASK]],
939|     ttg.warp_specialize(%smem, %bar) attributes {actualRegisters = array<i32: 480, 32>, allocation.offset = 512 : i32, requestedRegisters = array<i32: 32>, warpGroupStartIds = array<i32: 4>}
940|     default {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.warp_specialize. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.warp_specialize。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 941-949
```mlir
941|       // CHECK: tti.experimental_lock_acquire
942|       // CHECK: tt.call @__triton_consan_set_waiting
943|       // CHECK: tt.call @__triton_consan_check_all_active_waiting
944|       // CHECK: tti.experimental_lock_release
945|       %c0_i32 = arith.constant 0 : i32
946|       amdg.wait_barrier %bar, %c0_i32 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
947|       ttg.warp_yield
948|     }
949|     partition0(%arg1: !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<1xi64, #shared1, #smem, mutable>) num_warps(4) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, amdg.wait_barrier, ttg.warp_yield. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、amdg.wait_barrier、ttg.warp_yield。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 950-962
```mlir
950|       // CHECK: partition0
951|       // CHECK: tti.experimental_lock_acquire
952|       // CHECK: tt.call @__triton_consan_set_waiting
953|       // CHECK: tt.call @__triton_consan_check_all_active_waiting
954|       // CHECK: tti.experimental_lock_release
955|       %c0_i32 = arith.constant 0 : i32
956|       amdg.wait_barrier %arg2, %c0_i32 : !ttg.memdesc<1xi64, #shared1, #smem, mutable>
957|       ttg.warp_return
958|     } : (!ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared1, #smem, mutable>) -> ()
959|     tt.return
960|   }
961| }
962| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, amdg.wait_barrier, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、amdg.wait_barrier、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 963-963
```mlir
963| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 964-967
```mlir
964| 
965| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
966| #smem = #ttg.shared_memory
967| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 968-968
```mlir
968| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx1250`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx1250`。

### Lines 969-969
```mlir
969|   // CHECK-LABEL: @ws_alias_matrix
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @ws_alias_matrix anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @ws_alias_matrix 这样的标签用于锚定匹配范围。

### Lines 970-972
```mlir
970|   tt.func public @ws_alias_matrix() {
971|     // We expect the alias matrix constant to appear once for the default region
972|     // and once for partition0 when we lower warp_specialize.
```
**EN:** This function-oriented block defines or enters `ws_alias_matrix`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `ws_alias_matrix` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 973-986
```mlir
973|     // CHECK-DAG: arith.constant dense<true> : tensor<2x2xi1
974|     %smem0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
975|     %smem1 = ttg.local_alloc {allocation.offset = 16 : i32} : () -> !ttg.memdesc<32xf32, #shared, #smem, mutable>
976|     %bar = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
977|     amdg.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
978| 
979|     ttg.warp_specialize(%smem0, %smem1, %bar) attributes {actualRegisters = array<i32: 32, 32>, allocation.offset = 0 : i32, requestedRegisters = array<i32: 32>, warpGroupStartIds = array<i32: 0>}
980|     default {
981|       %c0 = arith.constant 0 : i32
982|       ttg.local_load %smem0 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
983|       ttg.local_load %smem1 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
984|       ttg.warp_yield
985|     }
986|     partition0(%arg0: !ttg.memdesc<32xf32, #shared, #smem, mutable>, %arg1: !ttg.memdesc<32xf32, #shared, #smem, mutable>, %arg2: !ttg.memdesc<1xi64, #shared, #smem, mutable>) num_warps(1) {
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory loads, amdg.init_barrier, ttg.warp_specialize, constants. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存加载、amdg.init_barrier、ttg.warp_specialize、常量。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 987-995
```mlir
987|       // CHECK: arith.constant dense<true> : tensor<2x2xi1
988|       %c0 = arith.constant 0 : i32
989|       ttg.local_load %arg0 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
990|       ttg.local_load %arg1 : !ttg.memdesc<32xf32, #shared, #smem, mutable> -> tensor<32xf32>
991|       ttg.warp_return
992|     } : (!ttg.memdesc<32xf32, #shared, #smem, mutable>, !ttg.memdesc<32xf32, #shared, #smem, mutable>, !ttg.memdesc<1xi64, #shared, #smem, mutable>) -> ()
993|     tt.return
994|   }
995| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on local/shared memory loads, constants, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 本地/共享内存加载、常量、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-allow-unregistered-dialect`, `-tritoninstrument-prepare-consan-captures="target=amd"`, `-tritoninstrument-concurrency-sanitizer`
- **CN:** 主要 pass 选项：`-split-input-file`，`-allow-unregistered-dialect`，`-tritoninstrument-prepare-consan-captures="target=amd"`，`-tritoninstrument-concurrency-sanitizer`
- **EN:** Dominant operations include `tt.func`, `ttg.local_alloc`, `module`, `ttg.local_load`, `tt.return`, `arith.constant`, `amdg.init_barrier`, `ttg.memdesc_index`, `amdg.async_tdm_copy_global_to_local`, `amdg.wait_barrier`.
- **CN:** 主要操作包括 `tt.func`、`ttg.local_alloc`、`module`、`ttg.local_load`、`tt.return`、`arith.constant`、`amdg.init_barrier`、`ttg.memdesc_index`、`amdg.async_tdm_copy_global_to_local`、`amdg.wait_barrier`。
- **EN:** The file contains 36 independently testable section(s). Check styles used: CHECK x226, CHECK-DAG x39, CHECK-LABEL x36, CHECK-NOT x3. Important labels include @single_local_alloc, @two_local_alloc, @three_local_alloc, @three_sub_bufs. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 36 个可独立测试的分段。使用的检查类型：CHECK ×226，CHECK-DAG ×39，CHECK-LABEL ×36，CHECK-NOT ×3。 关键标签包括 @single_local_alloc，@two_local_alloc，@three_local_alloc，@three_sub_bufs。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。