# proxy_fence_insertion.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/proxy_fence_insertion.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-triton-nvidia-gpu-proxy-fence-insertion` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-triton-nvidia-gpu-proxy-fence-insertion` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -triton-nvidia-gpu-proxy-fence-insertion --split-input-file -allow-unregistered-dialect | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -triton-nvidia-gpu-proxy-fence-insertion --split-input-file -allow-unregistered-dialect | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -triton-nvidia-gpu-proxy-fence-insertion --split-input-file -allow-unregistered-dialect | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -triton-nvidia-gpu-proxy-fence-insertion --split-input-file -allow-unregistered-dialect | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -triton-nvidia-gpu-proxy-fence-insertion --split-input-file -allow-unregistered-dialect | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-6
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
4| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
5| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
6| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 7-7
```mlir
7| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 8-8
```mlir
8|   // CHECK-LABEL: fence_write_after_read
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: fence_write_after_read anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: fence_write_after_read 这样的标签用于锚定匹配范围。

### Lines 9-9
```mlir
9|   tt.func @fence_write_after_read(%arg0: !tt.tensordesc<64x64xf32, #shared>, %arg1: !ttg.memdesc<1xi64, #shared1, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `fence_write_after_read`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fence_write_after_read` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 10-23
```mlir
10|     // CHECK: ttg.local_load
11|     // CHECK: ttng.fence_async_shared
12|     // CHECK: ttng.async_tma_copy_global_to_local
13|     %c0_i32 = arith.constant 0 : i32
14|     %true = arith.constant true
15|     %0 = ttg.local_alloc {allocation.offset = 16 : i32} : () -> !ttg.memdesc<32x64xf32, #shared, #smem, mutable>
16|     %1 = ttg.local_load %0 : !ttg.memdesc<32x64xf32, #shared, #smem, mutable> -> tensor<32x64xf32, #blocked>
17|     "test.keep"(%1) : (tensor<32x64xf32, #blocked>) -> ()
18|     %2 = ttg.local_alloc {allocation.offset = 32 : i32} : () -> !ttg.memdesc<64x64xf32, #shared, #smem, mutable>
19|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32] %2, %arg1, %true : !tt.tensordesc<64x64xf32, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<64x64xf32, #shared, #smem, mutable>
20|     tt.return
21|   }
22| }
23| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, shared/local memory allocation, local/shared memory loads, ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、共享/本地内存分配、本地/共享内存加载、ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 24-24
```mlir
24| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 25-29
```mlir
25| 
26| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
27| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
28| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
29| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 30-30
```mlir
30| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 31-31
```mlir
31|   // CHECK-LABEL: missing_proxy_fence_memdesc_index_alias_single
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: missing_proxy_fence_memdesc_index_alias_single anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: missing_proxy_fence_memdesc_index_alias_single 这样的标签用于锚定匹配范围。

### Lines 32-33
```mlir
32|   tt.func @missing_proxy_fence_memdesc_index_alias_single(%arg0: !tt.tensordesc<64x64xf32, #shared>, %arg1: !ttg.memdesc<1xi64, #shared1, #smem, mutable>) {
33|     // Keep the first fence to clear dependencies from local_alloc.
```
**EN:** This function-oriented block defines or enters `missing_proxy_fence_memdesc_index_alias_single`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `missing_proxy_fence_memdesc_index_alias_single` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 34-50
```mlir
34|     // CHECK: ttng.fence_async_shared
35|     // CHECK: ttg.local_load
36|     // CHECK-NEXT: "test.keep"
37|     // CHECK-NEXT: ttng.fence_async_shared
38|     // CHECK-NEXT: ttng.async_tma_copy_global_to_local
39|     %c0_i32 = arith.constant 0 : i32
40|     %true = arith.constant true
41|     %0 = ttg.local_alloc {allocation.offset = 16 : i32} : () -> !ttg.memdesc<1x64x64xf32, #shared, #smem, mutable>
42|     %1 = ttg.memdesc_index %0[%c0_i32] : !ttg.memdesc<1x64x64xf32, #shared, #smem, mutable> -> !ttg.memdesc<64x64xf32, #shared, #smem, mutable>
43|     ttng.fence_async_shared {bCluster = false}
44|     %2 = ttg.local_load %1 : !ttg.memdesc<64x64xf32, #shared, #smem, mutable> -> tensor<64x64xf32, #blocked>
45|     "test.keep"(%2) : (tensor<64x64xf32, #blocked>) -> ()
46|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32] %1, %arg1, %true : !tt.tensordesc<64x64xf32, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<64x64xf32, #shared, #smem, mutable>
47|     tt.return
48|   }
49| }
50| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, shared/local memory allocation, ttg.memdesc_index, ttng.fence_async_shared, local/shared memory loads. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、共享/本地内存分配、ttg.memdesc_index、ttng.fence_async_shared、本地/共享内存加载。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 51-51
```mlir
51| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 52-56
```mlir
52| 
53| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
54| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
55| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
56| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 57-57
```mlir
57| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 58-58
```mlir
58|   // CHECK-LABEL: async_proxy_after_async_proxy
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: async_proxy_after_async_proxy anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: async_proxy_after_async_proxy 这样的标签用于锚定匹配范围。

### Lines 59-59
```mlir
59|   tt.func @async_proxy_after_async_proxy(%arg0: !tt.tensordesc<64x64xf32, #shared>, %arg1: !ttg.memdesc<1xi64, #shared1, #smem, mutable>) {
```
**EN:** This function-oriented block defines or enters `async_proxy_after_async_proxy`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_proxy_after_async_proxy` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 60-73
```mlir
60|     // CHECK: ttng.async_tma_copy_global_to_local
61|     // CHECK-NOT: ttng.fence_async_shared
62|     // CHECK: ttng.async_tma_copy_global_to_local
63|     %c0_i32 = arith.constant 0 : i32
64|     %true = arith.constant true
65|     %0 = ttg.local_alloc {allocation.offset = 16 : i32} : () -> !ttg.memdesc<64x64xf32, #shared, #smem, mutable>
66|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32] %0, %arg1, %true : !tt.tensordesc<64x64xf32, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<64x64xf32, #shared, #smem, mutable>
67|     ttng.async_tma_store_wait {pendings = 0 : i32}
68|     %2 = ttg.local_alloc {allocation.offset = 32 : i32} : () -> !ttg.memdesc<64x64xf32, #shared, #smem, mutable>
69|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32] %2, %arg1, %true : !tt.tensordesc<64x64xf32, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<64x64xf32, #shared, #smem, mutable>
70|     tt.return
71|   }
72| }
73| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, shared/local memory allocation, ttng.async_tma_copy_global_to_local, ttng.async_tma_store_wait, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、共享/本地内存分配、ttng.async_tma_copy_global_to_local、ttng.async_tma_store_wait、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 74-74
```mlir
74| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 75-78
```mlir
75| 
76| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
77| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
78| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 79-79
```mlir
79| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 80-80
```mlir
80|   // CHECK-LABEL: missing_proxy_fence_local_store_before_async_tma_copy_local_to_global
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: missing_proxy_fence_local_store_before_async_tma_copy_local_to_global anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: missing_proxy_fence_local_store_before_async_tma_copy_local_to_global 这样的标签用于锚定匹配范围。

### Lines 81-81
```mlir
81|   tt.func @missing_proxy_fence_local_store_before_async_tma_copy_local_to_global(%arg0: !tt.tensordesc<128x256xf32, #shared>, %arg1: tensor<128x256xf32, #blocked>) {
```
**EN:** This function-oriented block defines or enters `missing_proxy_fence_local_store_before_async_tma_copy_local_to_global`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `missing_proxy_fence_local_store_before_async_tma_copy_local_to_global` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 82-94
```mlir
82|     // CHECK: ttng.async_tma_store_wait {pendings = 1 : i32}
83|     // CHECK-NEXT: ttg.local_store
84|     // CHECK-NEXT: ttng.fence_async_shared
85|     // CHECK-NEXT: ttng.async_tma_copy_local_to_global
86|     %c0_i32 = arith.constant 0 : i32
87|     %0 = ttg.local_alloc {allocation.offset = 16 : i32} : () -> !ttg.memdesc<128x256xf32, #shared, #smem, mutable>
88|     ttng.async_tma_store_wait {pendings = 1 : i32}
89|     ttg.local_store %arg1, %0 : tensor<128x256xf32, #blocked> -> !ttg.memdesc<128x256xf32, #shared, #smem, mutable>
90|     ttng.async_tma_copy_local_to_global %arg0[%c0_i32, %c0_i32] %0 : !tt.tensordesc<128x256xf32, #shared>, !ttg.memdesc<128x256xf32, #shared, #smem, mutable>
91|     tt.return
92|   }
93| }
94| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on constants, shared/local memory allocation, ttng.async_tma_store_wait, local/shared memory stores, ttng.async_tma_copy_local_to_global. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 常量、共享/本地内存分配、ttng.async_tma_store_wait、本地/共享内存存储、ttng.async_tma_copy_local_to_global。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 95-95
```mlir
95| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 96-100
```mlir
 96| 
 97| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
 98| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
 99| #smem = #ttg.shared_memory
100| #tmem = #ttng.tensor_memory_scales_encoding<>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 101-101
```mlir
101| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 102-102
```mlir
102|   // CHECK-LABEL: missing_proxy_fence_local_store_before_tmem_copy
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: missing_proxy_fence_local_store_before_tmem_copy anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: missing_proxy_fence_local_store_before_tmem_copy 这样的标签用于锚定匹配范围。

### Lines 103-104
```mlir
103|   tt.func @missing_proxy_fence_local_store_before_tmem_copy(%arg0: tensor<128x4xi8, #blocked>,
104|       %arg1: !ttg.memdesc<128x4xi8, #tmem, #ttng.tensor_memory, mutable>) {
```
**EN:** This function-oriented block defines or enters `missing_proxy_fence_local_store_before_tmem_copy`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `missing_proxy_fence_local_store_before_tmem_copy` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 105-113
```mlir
105|     // CHECK: ttg.local_store
106|     // CHECK-NEXT: ttng.fence_async_shared
107|     // CHECK-NEXT: ttng.tmem_copy
108|     %0 = ttg.local_alloc {allocation.offset = 16 : i32} : () -> !ttg.memdesc<128x4xi8, #shared1, #smem, mutable>
109|     ttg.local_store %arg0, %0 : tensor<128x4xi8, #blocked> -> !ttg.memdesc<128x4xi8, #shared1, #smem, mutable>
110|     ttng.tmem_copy %0, %arg1 : !ttg.memdesc<128x4xi8, #shared1, #smem, mutable>, !ttg.memdesc<128x4xi8, #tmem, #ttng.tensor_memory, mutable>
111|     tt.return
112|   }
113| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, local/shared memory stores, tensor-memory copies, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、本地/共享内存存储、张量内存拷贝、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-triton-nvidia-gpu-proxy-fence-insertion`, `--split-input-file`, `-allow-unregistered-dialect`
- **CN:** 主要 pass 选项：`-triton-nvidia-gpu-proxy-fence-insertion`，`--split-input-file`，`-allow-unregistered-dialect`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `ttg.local_alloc`, `module`, `tt.return`, `ttng.async_tma_copy_global_to_local`, `ttg.local_load`, `ttng.async_tma_store_wait`, `ttg.local_store`, `ttg.memdesc_index`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`ttg.local_alloc`、`module`、`tt.return`、`ttng.async_tma_copy_global_to_local`、`ttg.local_load`、`ttng.async_tma_store_wait`、`ttg.local_store`、`ttg.memdesc_index`。
- **EN:** The file contains 4 independently testable section(s). Check styles used: CHECK x9, CHECK-NEXT x8, CHECK-LABEL x5, CHECK-NOT x1. Important labels include fence_write_after_read, missing_proxy_fence_memdesc_index_alias_single, async_proxy_after_async_proxy, missing_proxy_fence_local_store_before_async_tma_copy_local_to_global. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 4 个可独立测试的分段。使用的检查类型：CHECK ×9，CHECK-NEXT ×8，CHECK-LABEL ×5，CHECK-NOT ×1。 关键标签包括 fence_write_after_read，missing_proxy_fence_memdesc_index_alias_single，async_proxy_after_async_proxy，missing_proxy_fence_local_store_before_async_tma_copy_local_to_global。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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