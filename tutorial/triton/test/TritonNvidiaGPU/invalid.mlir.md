# invalid.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonNvidiaGPU/invalid.mlir`
- **EN:** Negative/diagnostic test that checks verifier or pass failures for invalid Triton IR.
- **CN:** 这是一个负向/诊断测试，用来检查无效 Triton IR 是否会触发 verifier 或 pass 失败。
- **EN:** RUN pipeline(s): `// RUN: triton-opt --split-input-file %s --verify-diagnostics`
- **CN:** RUN 流水线：`// RUN: triton-opt --split-input-file %s --verify-diagnostics`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt --split-input-file %s --verify-diagnostics
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt --split-input-file %s --verify-diagnostics` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt --split-input-file %s --verify-diagnostics`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-3
```mlir
3| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 4-4
```mlir
4| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65536 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 5-5
```mlir
5|   tt.func public @alloc_tensor_memory() {
```
**EN:** This function-oriented block defines or enters `alloc_tensor_memory`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_tensor_memory` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 6-11
```mlir
 6|     // expected-error @+1 {{uninitialized alloc must have a mutable memdesc type}}
 7|     %0 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>
 8|     tt.return
 9|   }
10| }
11| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 12-12
```mlir
12| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 13-14
```mlir
13| 
14| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 15-15
```mlir
15| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65536 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 16-16
```mlir
16|   tt.func public @tmem_layout_cta_mismatch() {
```
**EN:** This function-oriented block defines or enters `tmem_layout_cta_mismatch`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_layout_cta_mismatch` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 17-22
```mlir
17|     // expected-error @+1 {{Layout has 1 CTAs per CGA, but the context requires 2 CTAs per CGA.}}
18|     %0 = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
19|     tt.return
20|   }
21| }
22| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 23-23
```mlir
23| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 24-27
```mlir
24| 
25| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
26| 
27| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 28-28
```mlir
28| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65536 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 29-32
```mlir
29|   tt.func public @alloc_tensor_memory() {
30|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
31|     %true = arith.constant true
32|     %0 = ttng.tmem_alloc %cst : (tensor<128x128xf32, #blocked>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>
```
**EN:** This function-oriented block defines or enters `alloc_tensor_memory`. Within it, the test exercises tt.func, constants, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_tensor_memory` 为核心。测试在其中演示 tt.func、常量、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 33-38
```mlir
33|     // expected-error @+1 {{Cannot store into an immutable alloc}}
34|     ttng.tmem_store %cst, %0, %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>
35|     tt.return
36|   }
37| }
38| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_store, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_store、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 39-39
```mlir
39| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 40-43
```mlir
40| 
41| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
42| #scales = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
43| #tmem = #ttng.tensor_memory_scales_encoding<>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 44-44
```mlir
44| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65536 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 45-47
```mlir
45|   tt.func public @alloc_tensor_memory(%arg: !ttg.memdesc<128x4xi8, #shared1, #ttg.shared_memory, mutable>) {
46|     %cst = arith.constant dense<0> : tensor<128x4xi8, #scales>
47|     %0 = ttng.tmem_alloc %cst : (tensor<128x4xi8, #scales>) -> !ttg.memdesc<128x4xi8, #tmem, #ttng.tensor_memory>
```
**EN:** This function-oriented block defines or enters `alloc_tensor_memory`. Within it, the test exercises tt.func, constants, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `alloc_tensor_memory` 为核心。测试在其中演示 tt.func、常量、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 48-53
```mlir
48|     // expected-error @+1 {{Cannot copy into an immutable alloc}}
49|     ttng.tmem_copy %arg, %0 : !ttg.memdesc<128x4xi8, #shared1, #ttg.shared_memory, mutable>, !ttg.memdesc<128x4xi8, #tmem, #ttng.tensor_memory>
50|     tt.return
51|   }
52| }
53| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory copies, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存拷贝、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 54-54
```mlir
54| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 55-57
```mlir
55| 
56| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
57| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 58-58
```mlir
58| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65536 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 59-60
```mlir
59|   tt.func public @init_barrier_zero_count() {
60|     %bar = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `init_barrier_zero_count`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `init_barrier_zero_count` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 61-66
```mlir
61|     // expected-error @+1 {{count must be greater than or equal to 1}}
62|     ttng.init_barrier %bar, 0 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
63|     tt.return
64|   }
65| }
66| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.init_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.init_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 67-67
```mlir
67| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 68-73
```mlir
68| 
69| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16}>
70| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
71| 
72| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
73| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 74-74
```mlir
74| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 75-78
```mlir
75| tt.func @async_tma_gather(%desc: !tt.tensordesc<1x128xbf16, #shared>, %x_offsets: tensor<32xi32, #blocked>, %y_offset: i32,
76|                           %bar: !ttg.memdesc<2xi32, #shared1, #ttg.shared_memory, mutable>,
77|                           %result: !ttg.memdesc<32x128xbf16, #shared, #ttg.shared_memory, mutable>,
78|                           %pred: i1) {
```
**EN:** This function-oriented block defines or enters `async_tma_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 79-84
```mlir
79|   // expected-error @below {{barrier allocation must be a descriptor of Nxi64 type with N <= number of CTAs}}
80|   ttng.async_tma_gather %desc[%x_offsets, %y_offset] %result, %bar, %pred : !tt.tensordesc<1x128xbf16, #shared>, tensor<32xi32, #blocked>, i32, !ttg.memdesc<2xi32, #shared1, #ttg.shared_memory, mutable>, !ttg.memdesc<32x128xbf16, #shared, #ttg.shared_memory, mutable>, i1
81|   tt.return
82| }
83| }
84| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 85-85
```mlir
85| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 86-91
```mlir
86| 
87| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16}>
88| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
89| 
90| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
91| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 92-92
```mlir
92| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 93-96
```mlir
93| tt.func @async_tma_gather(%desc: !tt.tensordesc<1x128xbf16, #shared>, %x_offsets: tensor<32xi32, #blocked>, %y_offset: i32,
94|                           %bar: !ttg.memdesc<1xi64, #shared1, #ttg.shared_memory, mutable>,
95|                           %result: !ttg.memdesc<32x128xbf16, #shared, #ttg.shared_memory>,
96|                           %pred: i1) {
```
**EN:** This function-oriented block defines or enters `async_tma_gather`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_gather` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 97-102
```mlir
 97|   // expected-error @below {{cannot store into immutable memory}}
 98|   ttng.async_tma_gather %desc[%x_offsets, %y_offset] %result, %bar, %pred : !tt.tensordesc<1x128xbf16, #shared>, tensor<32xi32, #blocked>, i32, !ttg.memdesc<1xi64, #shared1, #ttg.shared_memory, mutable>, !ttg.memdesc<32x128xbf16, #shared, #ttg.shared_memory>, i1
 99|   tt.return
100| }
101| }
102| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 103-103
```mlir
103| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 104-108
```mlir
104| 
105| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 256, 32]}>
106| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
107| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
108| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 109-109
```mlir
109| module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 110-110
```mlir
110| tt.func @wgmma(%a: tensor<128x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>, %b: !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>, %c: tensor<128x128xf16, #mma>) {
```
**EN:** This function-oriented block defines or enters `wgmma`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wgmma` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 111-116
```mlir
111|   // expected-error @below {{in-register LHS operand must have a kWidth of 2 but got 1}}
112|   %0 = ttng.warp_group_dot %a, %b, %c : tensor<128x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory> -> tensor<128x128xf16, #mma>
113|   tt.return
114| }
115| }
116| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.warp_group_dot, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.warp_group_dot、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 117-117
```mlir
117| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 118-122
```mlir
118| 
119| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
120| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
121| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
122| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 123-123
```mlir
123| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 124-128
```mlir
124|   tt.func public @async_tma_copy_global_to_local(%arg0: !tt.tensordesc<1x256x32xf32, #shared>) -> tensor<256x32xf32, #blocked> {
125|     %true = arith.constant true
126|     %c32_i32 = arith.constant 32 : i32
127|     %0 = ttg.local_alloc : () -> !ttg.memdesc<256x32xf32, #shared, #smem, mutable>
128|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared1, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_copy_global_to_local`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_copy_global_to_local` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 129-133
```mlir
129|     // expected-error @below {{TMA descriptor must have NVMMA shared layout}}
130|     ttng.async_tma_copy_global_to_local %arg0[%c32_i32, %c32_i32, %c32_i32] %0, %1, %true : !tt.tensordesc<1x256x32xf32, #shared>, !ttg.memdesc<1xi64, #shared1, #smem, mutable> -> !ttg.memdesc<256x32xf32, #shared, #smem, mutable>
131|   }
132| }
133| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 134-134
```mlir
134| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 135-139
```mlir
135| 
136| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
137| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
138| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
139| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 140-140
```mlir
140| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 141-145
```mlir
141|   tt.func public @async_tma_copy_global_to_local(%arg0: !tt.tensordesc<1x256x32xf32, #shared>) -> tensor<256x32xf32, #blocked> {
142|     %true = arith.constant true
143|     %c32_i32 = arith.constant 32 : i32
144|     %0 = ttg.local_alloc : () -> !ttg.memdesc<256x32xf32, #shared, #smem, mutable>
145|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared2, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_copy_global_to_local`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_copy_global_to_local` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 146-150
```mlir
146|     // expected-error @below {{TMA descriptor layout must not be transposed}}
147|     ttng.async_tma_copy_global_to_local %arg0[%c32_i32, %c32_i32, %c32_i32] %0, %1, %true : !tt.tensordesc<1x256x32xf32, #shared>, !ttg.memdesc<1xi64, #shared2, #smem, mutable> -> !ttg.memdesc<256x32xf32, #shared, #smem, mutable>
148|   }
149| }
150| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 151-151
```mlir
151| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 152-157
```mlir
152| 
153| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
154| #nvmma32 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
155| #nvmma64 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
156| #shared_mbar = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
157| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 158-158
```mlir
158| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 159-163
```mlir
159|   tt.func public @async_tma_copy_global_to_local(%arg0: !tt.tensordesc<1x256x64xf32, #nvmma32>) {
160|     %true = arith.constant true
161|     %c32_i32 = arith.constant 32 : i32
162|     %0 = ttg.local_alloc : () -> !ttg.memdesc<256x64xf32, #nvmma64, #smem, mutable>
163|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared_mbar, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_copy_global_to_local`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_copy_global_to_local` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 164-168
```mlir
164|     // expected-error @below {{TMA descriptor layout must match shared layout}}
165|     ttng.async_tma_copy_global_to_local %arg0[%c32_i32, %c32_i32, %c32_i32] %0, %1, %true : !tt.tensordesc<1x256x64xf32, #nvmma32>, !ttg.memdesc<1xi64, #shared_mbar, #smem, mutable> -> !ttg.memdesc<256x64xf32, #nvmma64, #smem, mutable>
166|     tt.return
167|   }
168| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 169-169
```mlir
169| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 170-173
```mlir
170| 
171| #nvmma = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0]]}>
172| #barrier = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
173| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 174-174
```mlir
174| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttng.two-ctas" = false, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 175-180
```mlir
175|   tt.func public @async_tma_copy_global_to_local_requires_1d_barrier_layout(
176|       %arg0: !tt.tensordesc<64x128xf16, #nvmma>) {
177|     %true = arith.constant true
178|     %c0_i32 = arith.constant 0 : i32
179|     %0 = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
180|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #barrier, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_copy_global_to_local_requires_1d_barrier_layout`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_copy_global_to_local_requires_1d_barrier_layout` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 181-186
```mlir
181|     // expected-error @below {{TMA barrier cga_layout must be [[1]], got [[0]]}}
182|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32] %0, %1, %true : !tt.tensordesc<64x128xf16, #nvmma>, !ttg.memdesc<1xi64, #barrier, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
183|     tt.return
184|   }
185| }
186| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 187-187
```mlir
187| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 188-191
```mlir
188| 
189| #nvmma = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0], [0, 1]]}>
190| #barrier = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0], [0]]}>
191| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 192-192
```mlir
192| module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32, "ttng.two-ctas" = true, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 193-198
```mlir
193|   tt.func public @async_tma_copy_global_to_local_requires_two_cta_barrier_layout(
194|       %arg0: !tt.tensordesc<64x128xf16, #nvmma>) {
195|     %true = arith.constant true
196|     %c0_i32 = arith.constant 0 : i32
197|     %0 = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
198|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #barrier, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_copy_global_to_local_requires_two_cta_barrier_layout`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_copy_global_to_local_requires_two_cta_barrier_layout` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 199-204
```mlir
199|     // expected-error @below {{TMA barrier cga_layout must be [[1], [2]] or [[0], [1]], got [[0], [0]]}}
200|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32] %0, %1, %true : !tt.tensordesc<64x128xf16, #nvmma>, !ttg.memdesc<1xi64, #barrier, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma, #smem, mutable>
201|     tt.return
202|   }
203| }
204| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 205-205
```mlir
205| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 206-210
```mlir
206| 
207| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
208| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
209| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
210| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 211-211
```mlir
211| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 212-216
```mlir
212|   tt.func public @tma_im2col_missing_offsets(%arg0: !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>) {
213|     %true = arith.constant true
214|     %c0_i32 = arith.constant 0 : i32
215|     %0 = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
216|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared2, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `tma_im2col_missing_offsets`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_im2col_missing_offsets` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 217-222
```mlir
217|     // expected-error @below {{IM2COL mode requires offsets to be provided}}
218|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32, %c0_i32, %c0_i32] %0, %1, %true : !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>, !ttg.memdesc<1xi64, #shared2, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
219|     tt.return
220|   }
221| }
222| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 223-223
```mlir
223| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 224-228
```mlir
224| 
225| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
226| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
227| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
228| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 229-229
```mlir
229| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 230-235
```mlir
230|   tt.func public @tma_im2col_wrong_offset_count(%arg0: !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>) {
231|     %true = arith.constant true
232|     %c0_i32 = arith.constant 0 : i32
233|     %c1_i16 = arith.constant 1 : i16
234|     %0 = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
235|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared2, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `tma_im2col_wrong_offset_count`. Within it, the test exercises constants, tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_im2col_wrong_offset_count` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 236-241
```mlir
236|     // expected-error @below {{IM2COL mode with 4D coordinates requires 2 offsets, but got 1}}
237|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32, %c0_i32, %c0_i32] offsets = [%c1_i16] %0, %1, %true : !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>, !ttg.memdesc<1xi64, #shared2, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
238|     tt.return
239|   }
240| }
241| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 242-242
```mlir
242| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 243-247
```mlir
243| 
244| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
245| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
246| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
247| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 248-248
```mlir
248| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 249-254
```mlir
249|   tt.func public @tma_tiled_with_offsets(%arg0: !tt.tensordesc<64x128xf16, #nvmma_128>) {
250|     %true = arith.constant true
251|     %c0_i32 = arith.constant 0 : i32
252|     %c1_i16 = arith.constant 1 : i16
253|     %0 = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
254|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared2, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `tma_tiled_with_offsets`. Within it, the test exercises constants, tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_tiled_with_offsets` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 255-260
```mlir
255|     // expected-error @below {{TILED mode does not support offsets}}
256|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32] offsets = [%c1_i16] %0, %1, %true : !tt.tensordesc<64x128xf16, #nvmma_128>, !ttg.memdesc<1xi64, #shared2, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
257|     tt.return
258|   }
259| }
260| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 261-261
```mlir
261| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 262-266
```mlir
262| 
263| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
264| #nvmma_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
265| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
266| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 267-267
```mlir
267| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 268-272
```mlir
268|   tt.func public @tma_im2col_2d_invalid(%arg0: !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>) {
269|     %true = arith.constant true
270|     %c0_i32 = arith.constant 0 : i32
271|     %0 = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
272|     %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared2, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `tma_im2col_2d_invalid`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tma_im2col_2d_invalid` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 273-278
```mlir
273|     // expected-error @below {{IM2COL mode requires at least 3D coordinates, but got 2D}}
274|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32] %0, %1, %true : !ttng.tensordesc_im2col<64x128xf16, #nvmma_128>, !ttg.memdesc<1xi64, #shared2, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma_128, #smem, mutable>
275|     tt.return
276|   }
277| }
278| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 279-279
```mlir
279| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 280-284
```mlir
280| 
281| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
282| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
283| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
284| #tmem_f16 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 2>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 285-285
```mlir
285| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 286-292
```mlir
286|   tt.func @tcgen5(%a: !ttg.memdesc<128x128xbf16, #shared, #ttg.shared_memory>,
287|                   %b: !ttg.memdesc<128x256xbf16, #shared1, #ttg.shared_memory>,
288|                   %c: !ttg.memdesc<128x256xf16, #tmem_f16, #ttng.tensor_memory, mutable>,
289|                   %accUse: i1,
290|                   %pred: i1,
291|                   %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>,
292|                   %barrierPred: i1) {
```
**EN:** This function-oriented block defines or enters `tcgen5`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen5` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 293-302
```mlir
293|     // expected-error @below {{unsupported accumulator dtype for operand types 'bf16' and 'bf16', accumulator dtype is 'f16' but must be one of ['f32']}}
294|     ttng.tc_gen5_mma %a, %b, %c, %accUse, %pred, %barrier[%barrierPred] {is_async} :
295|        !ttg.memdesc<128x128xbf16, #shared, #ttg.shared_memory>,
296|        !ttg.memdesc<128x256xbf16, #shared1, #ttg.shared_memory>,
297|        !ttg.memdesc<128x256xf16, #tmem_f16, #ttng.tensor_memory, mutable>,
298|        !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
299|     tt.return
300|   }
301| }
302| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 303-303
```mlir
303| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 304-308
```mlir
304| 
305| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 16, CGALayout = [[0, 0]]}>
306| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 16, CGALayout = [[0, 1]]}>
307| #barrier = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
308| #tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 32, colStride = 1, CGALayout = [[0, 1]]>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 309-309
```mlir
309| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 310-317
```mlir
310|   tt.func @tcgen5_completion_barrier_cga_layout(
311|       %a: !ttg.memdesc<128x16xf16, #shared, #ttg.shared_memory>,
312|       %b: !ttg.memdesc<16x128xf16, #shared1, #ttg.shared_memory>,
313|       %c: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
314|       %accUse: i1,
315|       %pred: i1,
316|       %bar: !ttg.memdesc<1xi64, #barrier, #ttg.shared_memory>,
317|       %barPred: i1) {
```
**EN:** This function-oriented block defines or enters `tcgen5_completion_barrier_cga_layout`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen5_completion_barrier_cga_layout` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 318-327
```mlir
318|     // expected-error @below {{completion barrier cga_layout must be}}
319|     ttng.tc_gen5_mma %a, %b, %c, %accUse, %pred, %bar[%barPred] {is_async} :
320|        !ttg.memdesc<128x16xf16, #shared, #ttg.shared_memory>,
321|        !ttg.memdesc<16x128xf16, #shared1, #ttg.shared_memory>,
322|        !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
323|        !ttg.memdesc<1xi64, #barrier, #ttg.shared_memory>
324|     tt.return
325|   }
326| }
327| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on Gen5 tensor-core MMA ops, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 Gen5 张量核 MMA 操作、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 328-328
```mlir
328| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 329-331
```mlir
329| 
330| #barrier = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
331| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 332-332
```mlir
332| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 333-334
```mlir
333|   tt.func @tcgen5_commit_completion_barrier_cga_layout(
334|       %bar: !ttg.memdesc<1xi64, #barrier, #smem, mutable>, %pred: i1) {
```
**EN:** This function-oriented block defines or enters `tcgen5_commit_completion_barrier_cga_layout`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen5_commit_completion_barrier_cga_layout` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 335-340
```mlir
335|     // expected-error @below {{completion barrier cga_layout must be}}
336|     ttng.tc_gen5_commit %bar, %pred : !ttg.memdesc<1xi64, #barrier, #smem, mutable>
337|     tt.return
338|   }
339| }
340| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tc_gen5_commit, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tc_gen5_commit、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 341-341
```mlir
341| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 342-347
```mlir
342| 
343| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
344| #sharedT = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
345| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8}>
346| #barrier = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
347| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 348-348
```mlir
348| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 349-358
```mlir
349|   tt.func @tcgen5_mma_scaled_sync_with_barrier(
350|       %a: !ttg.memdesc<128x256xi8, #shared, #ttg.shared_memory>,
351|       %b: !ttg.memdesc<256x64xi8, #sharedT, #ttg.shared_memory>,
352|       %c: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>,
353|       %scale_a: !ttg.memdesc<128x8xf8E4M3FN, #shared1, #ttg.shared_memory>,
354|       %scale_b: !ttg.memdesc<64x8xf8E4M3FN, #shared1, #ttg.shared_memory>,
355|       %useAcc: i1,
356|       %pred: i1,
357|       %bar: !ttg.memdesc<1xi64, #barrier, #ttg.shared_memory, mutable>,
358|       %barPred: i1) {
```
**EN:** This function-oriented block defines or enters `tcgen5_mma_scaled_sync_with_barrier`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tcgen5_mma_scaled_sync_with_barrier` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 359-370
```mlir
359|     // expected-error @below {{The op is synchronous but a barrier is present.}}
360|     ttng.tc_gen5_mma_scaled %a, %b, %c, %scale_a, %scale_b, %useAcc, %pred lhs = e2m1 rhs = e2m1, %bar[%barPred] :
361|       !ttg.memdesc<128x256xi8, #shared, #ttg.shared_memory>,
362|       !ttg.memdesc<256x64xi8, #sharedT, #ttg.shared_memory>,
363|       !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>,
364|       !ttg.memdesc<128x8xf8E4M3FN, #shared1, #ttg.shared_memory>,
365|       !ttg.memdesc<64x8xf8E4M3FN, #shared1, #ttg.shared_memory>,
366|       !ttg.memdesc<1xi64, #barrier, #ttg.shared_memory, mutable>
367|     tt.return
368|   }
369| }
370| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on scaled tensor-core MMA ops, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 带缩放的张量核 MMA 操作、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 371-371
```mlir
371| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 372-375
```mlir
372| 
373| #shared_clc = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0], [0]]}>
374| #barrier = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0], [0]]}>
375| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 376-376
```mlir
376| module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 377-379
```mlir
377|   tt.func @clc_try_cancel_completion_barrier_cga_layout(
378|       %result: !ttg.memdesc<2xi64, #shared_clc, #smem>,
379|       %mbar: !ttg.memdesc<1xi64, #barrier, #smem>) {
```
**EN:** This function-oriented block defines or enters `clc_try_cancel_completion_barrier_cga_layout`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `clc_try_cancel_completion_barrier_cga_layout` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 380-386
```mlir
380|     // expected-error @below {{completion barrier cga_layout must be}}
381|     ttng.clc_try_cancel %result, %mbar :
382|       !ttg.memdesc<2xi64, #shared_clc, #smem>, !ttg.memdesc<1xi64, #barrier, #smem>
383|     tt.return
384|   }
385| }
386| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.clc_try_cancel, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.clc_try_cancel、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 387-387
```mlir
387| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 388-391
```mlir
388| 
389| #shared_clc_bad = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
390| #barrier = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
391| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 392-392
```mlir
392| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 393-395
```mlir
393|   tt.func @clc_try_cancel_result_cga_layout_bases_nonzero(
394|       %result: !ttg.memdesc<2xi64, #shared_clc_bad, #smem>,
395|       %mbar: !ttg.memdesc<1xi64, #barrier, #smem>) {
```
**EN:** This function-oriented block defines or enters `clc_try_cancel_result_cga_layout_bases_nonzero`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `clc_try_cancel_result_cga_layout_bases_nonzero` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 396-402
```mlir
396|     // expected-error @below {{Expected CLC result buffer cga_layout bases to be all zeros. Got [[1]]}}
397|     ttng.clc_try_cancel %result, %mbar :
398|       !ttg.memdesc<2xi64, #shared_clc_bad, #smem>, !ttg.memdesc<1xi64, #barrier, #smem>
399|     tt.return
400|   }
401| }
402| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.clc_try_cancel, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.clc_try_cancel、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 403-403
```mlir
403| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 404-405
```mlir
404| 
405| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 406-406
```mlir
406|   tt.func @fence_mbarrier_init_release_cluster_invalid() {
```
**EN:** This function-oriented block defines or enters `fence_mbarrier_init_release_cluster_invalid`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `fence_mbarrier_init_release_cluster_invalid` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 407-412
```mlir
407|     // expected-error @below {{requires ttg.num-ctas > 1}}
408|     ttng.fence_mbarrier_init_release_cluster
409|     tt.return
410|   }
411| }
412| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.fence_mbarrier_init_release_cluster, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.fence_mbarrier_init_release_cluster、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 413-413
```mlir
413| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 414-415
```mlir
414| 
415| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 416-416
```mlir
416|   tt.func @cluster_arrive_invalid() {
```
**EN:** This function-oriented block defines or enters `cluster_arrive_invalid`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cluster_arrive_invalid` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 417-422
```mlir
417|     // expected-error @below {{requires ttg.num-ctas > 1}}
418|     ttng.cluster_arrive
419|     tt.return
420|   }
421| }
422| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.cluster_arrive, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.cluster_arrive、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 423-423
```mlir
423| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 424-425
```mlir
424| 
425| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 426-426
```mlir
426|   tt.func @cluster_wait_invalid() {
```
**EN:** This function-oriented block defines or enters `cluster_wait_invalid`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cluster_wait_invalid` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 427-432
```mlir
427|     // expected-error @below {{requires ttg.num-ctas > 1}}
428|     ttng.cluster_wait
429|     tt.return
430|   }
431| }
432| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.cluster_wait, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.cluster_wait、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 433-433
```mlir
433| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 434-435
```mlir
434| 
435| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 436-438
```mlir
436|   tt.func @cluster_arrive_in_default_region_invalid() {
437|     ttg.warp_specialize()
438|     default {
```
**EN:** This function-oriented block defines or enters `cluster_arrive_in_default_region_invalid`. Within it, the test exercises tt.func, ttg.warp_specialize, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cluster_arrive_in_default_region_invalid` 为核心。测试在其中演示 tt.func、ttg.warp_specialize，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 439-449
```mlir
439|       // expected-error @below {{cannot be used inside `ttg.warp_specialize`}}
440|       ttng.cluster_arrive
441|       ttg.warp_yield
442|     }
443|     partition0() num_warps(4) {
444|       ttg.warp_return
445|     } : () -> ()
446|     tt.return
447|   }
448| }
449| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.cluster_arrive, ttg.warp_yield, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.cluster_arrive、ttg.warp_yield、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 450-450
```mlir
450| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 451-452
```mlir
451| 
452| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 453-458
```mlir
453|   tt.func @cluster_wait_in_partition_invalid() {
454|     ttg.warp_specialize()
455|     default {
456|       ttg.warp_yield
457|     }
458|     partition0() num_warps(4) {
```
**EN:** This function-oriented block defines or enters `cluster_wait_in_partition_invalid`. Within it, the test exercises tt.func, ttg.warp_specialize, ttg.warp_yield, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cluster_wait_in_partition_invalid` 为核心。测试在其中演示 tt.func、ttg.warp_specialize、ttg.warp_yield，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 459-466
```mlir
459|       // expected-error @below {{cannot be used inside `ttg.warp_specialize`}}
460|       ttng.cluster_wait
461|       ttg.warp_return
462|     } : () -> ()
463|     tt.return
464|   }
465| }
466| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.cluster_wait, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.cluster_wait、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 467-467
```mlir
467| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 468-469
```mlir
468| 
469| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 470-470
```mlir
470|   tt.func @cluster_barrier_invalid() {
```
**EN:** This function-oriented block defines or enters `cluster_barrier_invalid`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cluster_barrier_invalid` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 471-476
```mlir
471|     // expected-error @below {{requires ttg.num-ctas > 1}}
472|     ttng.cluster_barrier
473|     tt.return
474|   }
475| }
476| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.cluster_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.cluster_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 477-477
```mlir
477| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 478-479
```mlir
478| 
479| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 480-482
```mlir
480|   tt.func @cluster_barrier_in_default_region_invalid() {
481|     ttg.warp_specialize()
482|     default {
```
**EN:** This function-oriented block defines or enters `cluster_barrier_in_default_region_invalid`. Within it, the test exercises tt.func, ttg.warp_specialize, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cluster_barrier_in_default_region_invalid` 为核心。测试在其中演示 tt.func、ttg.warp_specialize，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 483-493
```mlir
483|       // expected-error @below {{cannot be used inside `ttg.warp_specialize`}}
484|       ttng.cluster_barrier
485|       ttg.warp_yield
486|     }
487|     partition0() num_warps(4) {
488|       ttg.warp_return
489|     } : () -> ()
490|     tt.return
491|   }
492| }
493| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.cluster_barrier, ttg.warp_yield, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.cluster_barrier、ttg.warp_yield、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 494-494
```mlir
494| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 495-496
```mlir
495| 
496| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 497-502
```mlir
497|   tt.func @cluster_barrier_in_partition_invalid() {
498|     ttg.warp_specialize()
499|     default {
500|       ttg.warp_yield
501|     }
502|     partition0() num_warps(4) {
```
**EN:** This function-oriented block defines or enters `cluster_barrier_in_partition_invalid`. Within it, the test exercises tt.func, ttg.warp_specialize, ttg.warp_yield, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `cluster_barrier_in_partition_invalid` 为核心。测试在其中演示 tt.func、ttg.warp_specialize、ttg.warp_yield，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 503-510
```mlir
503|       // expected-error @below {{cannot be used inside `ttg.warp_specialize`}}
504|       ttng.cluster_barrier
505|       ttg.warp_return
506|     } : () -> ()
507|     tt.return
508|   }
509| }
510| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.cluster_barrier, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.cluster_barrier、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 511-511
```mlir
511| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 512-515
```mlir
512| 
513| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
514| #smem = #ttg.shared_memory
515| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 516-516
```mlir
516| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90"} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 517-520
```mlir
517|   tt.func @init_barrier_in_default_region_invalid() {
518|     %bar = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
519|     ttg.warp_specialize()
520|     default {
```
**EN:** This function-oriented block defines or enters `init_barrier_in_default_region_invalid`. Within it, the test exercises tt.func, shared/local memory allocation, ttg.warp_specialize, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `init_barrier_in_default_region_invalid` 为核心。测试在其中演示 tt.func、共享/本地内存分配、ttg.warp_specialize，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 521-531
```mlir
521|       // expected-error @below {{cannot be used inside `ttg.warp_specialize`}}
522|       ttng.init_barrier %bar, 1 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
523|       ttg.warp_yield
524|     }
525|     partition0() num_warps(4) {
526|       ttg.warp_return
527|     } : () -> ()
528|     tt.return
529|   }
530| }
531| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.init_barrier, ttg.warp_yield, ttg.warp_return, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.init_barrier、ttg.warp_yield、ttg.warp_return、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 532-532
```mlir
532| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 533-534
```mlir
533| 
534| // expected-error @+1 {{LinearEncodingAttr requires a permutation matrix layout after removing broadcast bases}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 535-535
```mlir
535| #linear = #ttg.linear<{register = [[0, 2], [0, 4], [0, 8], [0, 16], [0, 32]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 1]], warp = [[16, 0], [8, 0]], block = []}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 536-536
```mlir
536| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 537-541
```mlir
537|   tt.func @invalid_linear_layout(%arg0: tensor<32x64xi32, #linear>) {
538|     tt.return
539|   }
540| }
541| 
```
**EN:** This function-oriented block defines or enters `invalid_linear_layout`. Within it, the test exercises tt.func, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `invalid_linear_layout` 为核心。测试在其中演示 tt.func、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 542-542
```mlir
542| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 543-545
```mlir
543| 
544| // Test that reduction with warps split across N dimension is rejected
545| // 128x256 with 8 warps -> warpsPerCTA = [4, 2] (2 warps in N)
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 546-548
```mlir
546| #blocked_split = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 2], order = [0, 1]}>
547| #blocked_red = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
548| #tmem_warp_split = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 549-549
```mlir
549| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:107", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:107`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:107`。

### Lines 550-552
```mlir
550|   tt.func public @tensor_memory_ld_red_warp_split_rejected() {
551|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked_split>
552|     %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x256xf32, #blocked_split>) -> !ttg.memdesc<128x256xf32, #tmem_warp_split, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tensor_memory_ld_red_warp_split_rejected`. Within it, the test exercises tt.func, constants, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tensor_memory_ld_red_warp_split_rejected` 为核心。测试在其中演示 tt.func、常量、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 553-558
```mlir
553|     // expected-error @below {{tmem_load reduction with N dimension sharded across threads is not supported.}}
554|     %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<min>} : !ttg.memdesc<128x256xf32, #tmem_warp_split, #ttng.tensor_memory, mutable> -> tensor<128x256xf32, #blocked_split>, tensor<128xf32, #blocked_red>
555|     tt.return
556|   }
557| }
558| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 559-559
```mlir
559| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 560-561
```mlir
560| 
561| // Test that reduction with N shared across threads is rejected
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 562-564
```mlir
562| #blocked_split = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [0, 1]}>
563| #blocked_red = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
564| #bm64_bn128 = #ttng.tensor_memory_encoding<blockM = 64, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 565-565
```mlir
565| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:107", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:107`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:107`。

### Lines 566-568
```mlir
566|   tt.func public @tensor_memory_ld_red_16x32bx2_atom_rejected() {
567|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<64x128xf32, #blocked_split>
568|     %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<64x128xf32, #blocked_split>) -> !ttg.memdesc<64x128xf32, #bm64_bn128, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tensor_memory_ld_red_16x32bx2_atom_rejected`. Within it, the test exercises tt.func, constants, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tensor_memory_ld_red_16x32bx2_atom_rejected` 为核心。测试在其中演示 tt.func、常量、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 569-574
```mlir
569|     // expected-error @below {{tmem_load reduction with N dimension sharded across threads is not supported.}}
570|     %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<min>} : !ttg.memdesc<64x128xf32, #bm64_bn128, #ttng.tensor_memory, mutable> -> tensor<64x128xf32, #blocked_split>, tensor<64xf32, #blocked_red>
571|     tt.return
572|   }
573| }
574| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 575-575
```mlir
575| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 576-577
```mlir
576| 
577| // Test: abs requires redOp to be set
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 578-579
```mlir
578| #blocked_abs = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
579| #tmem_abs = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 580-580
```mlir
580| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:107", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:107`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:107`。

### Lines 581-583
```mlir
581|   tt.func public @tensor_memory_ld_abs_requires_redop() {
582|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked_abs>
583|     %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked_abs>) -> !ttg.memdesc<128x128xf32, #tmem_abs, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tensor_memory_ld_abs_requires_redop`. Within it, the test exercises tt.func, constants, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tensor_memory_ld_abs_requires_redop` 为核心。测试在其中演示 tt.func、常量、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 584-589
```mlir
584|     // expected-error @below {{'abs' requires 'redOp' to be set}}
585|     %result = ttng.tmem_load %0 {abs = true} : !ttg.memdesc<128x128xf32, #tmem_abs, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked_abs>
586|     tt.return
587|   }
588| }
589| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 590-590
```mlir
590| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 591-592
```mlir
591| 
592| // Test: NaN requires redOp to be set
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 593-594
```mlir
593| #blocked_nan = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
594| #tmem_nan = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 595-595
```mlir
595| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:107", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:107`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:107`。

### Lines 596-598
```mlir
596|   tt.func public @tensor_memory_ld_nan_requires_redop() {
597|     %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked_nan>
598|     %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked_nan>) -> !ttg.memdesc<128x128xf32, #tmem_nan, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tensor_memory_ld_nan_requires_redop`. Within it, the test exercises tt.func, constants, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tensor_memory_ld_nan_requires_redop` 为核心。测试在其中演示 tt.func、常量、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 599-604
```mlir
599|     // expected-error @below {{'NaN' requires 'redOp' to be set}}
600|     %result = ttng.tmem_load %0 {NaN = true} : !ttg.memdesc<128x128xf32, #tmem_nan, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked_nan>
601|     tt.return
602|   }
603| }
604| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 605-605
```mlir
605| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 606-607
```mlir
606| 
607| // Test: abs requires f32 element type
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 608-610
```mlir
608| #blocked_abs_i32 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
609| #blocked_red_abs_i32 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
610| #tmem_abs_i32 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 611-611
```mlir
611| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:107", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:107`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:107`。

### Lines 612-614
```mlir
612|   tt.func public @tensor_memory_ld_abs_requires_f32() {
613|     %cst_0 = arith.constant dense<0> : tensor<128x128xi32, #blocked_abs_i32>
614|     %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xi32, #blocked_abs_i32>) -> !ttg.memdesc<128x128xi32, #tmem_abs_i32, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tensor_memory_ld_abs_requires_f32`. Within it, the test exercises tt.func, constants, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tensor_memory_ld_abs_requires_f32` 为核心。测试在其中演示 tt.func、常量、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 615-620
```mlir
615|     // expected-error @below {{'abs' requires floating-point element type (f32)}}
616|     %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<min>, abs = true} : !ttg.memdesc<128x128xi32, #tmem_abs_i32, #ttng.tensor_memory, mutable> -> tensor<128x128xi32, #blocked_abs_i32>, tensor<128xi32, #blocked_red_abs_i32>
617|     tt.return
618|   }
619| }
620| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 621-621
```mlir
621| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 622-623
```mlir
622| 
623| // Test: NaN requires f32 element type
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 624-626
```mlir
624| #blocked_nan_i32 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
625| #blocked_red_nan_i32 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
626| #tmem_nan_i32 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 627-627
```mlir
627| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:107", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:107`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:107`。

### Lines 628-630
```mlir
628|   tt.func public @tensor_memory_ld_nan_requires_f32() {
629|     %cst_0 = arith.constant dense<0> : tensor<128x128xi32, #blocked_nan_i32>
630|     %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xi32, #blocked_nan_i32>) -> !ttg.memdesc<128x128xi32, #tmem_nan_i32, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tensor_memory_ld_nan_requires_f32`. Within it, the test exercises tt.func, constants, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tensor_memory_ld_nan_requires_f32` 为核心。测试在其中演示 tt.func、常量、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 631-636
```mlir
631|     // expected-error @below {{'NaN' requires floating-point element type (f32)}}
632|     %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<min>, NaN = true} : !ttg.memdesc<128x128xi32, #tmem_nan_i32, #ttng.tensor_memory, mutable> -> tensor<128x128xi32, #blocked_nan_i32>, tensor<128xi32, #blocked_red_nan_i32>
633|     tt.return
634|   }
635| }
636| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_load, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_load、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 637-637
```mlir
637| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 638-642
```mlir
638| 
639| #blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
640| #nvmma_no_broadcast = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0]]}>
641| #shared_bar = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
642| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 643-643
```mlir
643| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 644-648
```mlir
644|   tt.func public @async_tma_copy_multicast_requires_broadcast(%arg0: !tt.tensordesc<64x128xf16, #nvmma_no_broadcast>) {
645|     %true = arith.constant true
646|     %c0_i32 = arith.constant 0 : i32
647|     %0 = ttg.local_alloc : () -> !ttg.memdesc<64x128xf16, #nvmma_no_broadcast, #smem, mutable>
648|     %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #shared_bar, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_copy_multicast_requires_broadcast`. Within it, the test exercises tt.func, constants, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_copy_multicast_requires_broadcast` 为核心。测试在其中演示 tt.func、常量、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 649-654
```mlir
649|     // expected-error @below {{multicast requires the shared layout to broadcast across CTAs}}
650|     ttng.async_tma_copy_global_to_local %arg0[%c0_i32, %c0_i32] %0, %1, %true {multicast} : !tt.tensordesc<64x128xf16, #nvmma_no_broadcast>, !ttg.memdesc<2xi64, #shared_bar, #smem, mutable> -> !ttg.memdesc<64x128xf16, #nvmma_no_broadcast, #smem, mutable>
651|     tt.return
652|   }
653| }
654| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_copy_global_to_local, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_copy_global_to_local、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 655-655
```mlir
655| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 656-660
```mlir
656| 
657| #blocked_broadcast = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0], CGALayout = [[0]]}>
658| #nvmma_no_broadcast = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0]]}>
659| #shared_bar = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
660| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 661-661
```mlir
661| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 662-667
```mlir
662|   tt.func public @async_tma_gather_multicast_requires_broadcast(%arg0: !tt.tensordesc<1x128xf16, #nvmma_no_broadcast>) {
663|     %true = arith.constant true
664|     %c0_i32 = arith.constant 0 : i32
665|     %x_offsets = arith.constant dense<0> : tensor<32xi32, #blocked_broadcast>
666|     %bar = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared_bar, #smem, mutable>
667|     %result = ttg.local_alloc : () -> !ttg.memdesc<32x128xf16, #nvmma_no_broadcast, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_gather_multicast_requires_broadcast`. Within it, the test exercises constants, tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_gather_multicast_requires_broadcast` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 668-673
```mlir
668|     // expected-error @below {{multicast requires the shared layout to broadcast across CTAs}}
669|     ttng.async_tma_gather %arg0[%x_offsets, %c0_i32] %result, %bar, %true {multicast} : !tt.tensordesc<1x128xf16, #nvmma_no_broadcast>, tensor<32xi32, #blocked_broadcast>, i32, !ttg.memdesc<1xi64, #shared_bar, #smem, mutable>, !ttg.memdesc<32x128xf16, #nvmma_no_broadcast, #smem, mutable>, i1
670|     tt.return
671|   }
672| }
673| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 674-674
```mlir
674| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 675-680
```mlir
675| 
676| #blocked_broadcast_parent = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0], CGALayout = [[0, 0], [0, 0]]}>
677| #blocked_broadcast = #ttg.slice<{dim = 0, parent = #blocked_broadcast_parent}>
678| #nvmma_partial_broadcast = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0], [0, 0]]}>
679| #shared_bar = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0], [0]]}>
680| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 681-681
```mlir
681| module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 682-687
```mlir
682|   tt.func public @async_tma_gather_multicast_requires_matching_x_offset_cga(%arg0: !tt.tensordesc<1x128xf16, #nvmma_partial_broadcast>) {
683|     %true = arith.constant true
684|     %c0_i32 = arith.constant 0 : i32
685|     %x_offsets = arith.constant dense<0> : tensor<32xi32, #blocked_broadcast>
686|     %bar = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared_bar, #smem, mutable>
687|     %result = ttg.local_alloc : () -> !ttg.memdesc<32x128xf16, #nvmma_partial_broadcast, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_gather_multicast_requires_matching_x_offset_cga`. Within it, the test exercises constants, tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_gather_multicast_requires_matching_x_offset_cga` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 688-693
```mlir
688|     // expected-error @below {{x offsets must have the same row CGA layout as the memdesc}}
689|     ttng.async_tma_gather %arg0[%x_offsets, %c0_i32] %result, %bar, %true {multicast} : !tt.tensordesc<1x128xf16, #nvmma_partial_broadcast>, tensor<32xi32, #blocked_broadcast>, i32, !ttg.memdesc<1xi64, #shared_bar, #smem, mutable>, !ttg.memdesc<32x128xf16, #nvmma_partial_broadcast, #smem, mutable>, i1
690|     tt.return
691|   }
692| }
693| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 694-694
```mlir
694| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 695-700
```mlir
695| 
696| #blocked_split_parent = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0], CGALayout = [[0, 1]]}>
697| #blocked_split = #ttg.slice<{dim = 0, parent = #blocked_split_parent}>
698| #nvmma_broadcast = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[0, 0]]}>
699| #shared_bar = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
700| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 701-701
```mlir
701| module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 702-707
```mlir
702|   tt.func public @async_tma_gather_multicast_requires_uniform_x_offsets(%arg0: !tt.tensordesc<1x128xf16, #nvmma_broadcast>) {
703|     %true = arith.constant true
704|     %c0_i32 = arith.constant 0 : i32
705|     %x_offsets = arith.constant dense<0> : tensor<32xi32, #blocked_split>
706|     %bar = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared_bar, #smem, mutable>
707|     %result = ttg.local_alloc : () -> !ttg.memdesc<32x128xf16, #nvmma_broadcast, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_gather_multicast_requires_uniform_x_offsets`. Within it, the test exercises constants, tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_gather_multicast_requires_uniform_x_offsets` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 708-714
```mlir
708|     // expected-error @below {{x offsets must have the same row CGA layout as the memdesc}}
709|     ttng.async_tma_gather %arg0[%x_offsets, %c0_i32] %result, %bar, %true {multicast} : !tt.tensordesc<1x128xf16, #nvmma_broadcast>, tensor<32xi32, #blocked_split>, i32, !ttg.memdesc<1xi64, #shared_bar, #smem, mutable>, !ttg.memdesc<32x128xf16, #nvmma_broadcast, #smem, mutable>, i1
710|     tt.return
711|   }
712| 
713| }
714| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 715-715
```mlir
715| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 716-717
```mlir
716| 
717| // Test invalid TensorDescIm2ColType: rank-3 blockType (must be rank-2)
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 718-718
```mlir
718| module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 719-719
```mlir
719|   // expected-error @below {{TensorDescIm2ColType requires rank-2 shape, got rank 3}}
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 720-724
```mlir
720|   tt.func @tensordesc_im2col_wrong_rank(%desc: !ttng.tensordesc_im2col<32x64x128xf16>) {
721|     tt.return
722|   }
723| }
724| 
```
**EN:** This function-oriented block defines or enters `tensordesc_im2col_wrong_rank`. Within it, the test exercises tt.func, tt.return, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tensordesc_im2col_wrong_rank` 为核心。测试在其中演示 tt.func、tt.return，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 725-725
```mlir
725| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 726-728
```mlir
726| 
727| #shared_bad = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0], [2], [1]]}>
728| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 729-729
```mlir
729| module attributes {"ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 730-730
```mlir
730|   tt.func @wait_barrier_invalid_cga_layout(%bar: !ttg.memdesc<4xi64, #shared_bad, #smem, mutable>, %phase: i32) {
```
**EN:** This function-oriented block defines or enters `wait_barrier_invalid_cga_layout`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `wait_barrier_invalid_cga_layout` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 731-736
```mlir
731|     // expected-error @below {{broadcasted cluster barriers require bases to be the sequence}}
732|     ttng.wait_barrier %bar, %phase : !ttg.memdesc<4xi64, #shared_bad, #smem, mutable>
733|     tt.return
734|   }
735| }
736| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.wait_barrier, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.wait_barrier、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 737-737
```mlir
737| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 738-739
```mlir
738| 
739| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 740-740
```mlir
740| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 741-741
```mlir
741|   tt.func public @memdesc_reinterpret_changed_storage_size_tmem(%arg0: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory>) {
```
**EN:** This function-oriented block defines or enters `memdesc_reinterpret_changed_storage_size_tmem`. Within it, the test exercises tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `memdesc_reinterpret_changed_storage_size_tmem` 为核心。测试在其中演示 tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 742-747
```mlir
742|     // expected-error @+1 {{source and result must have the same logical storage size}}
743|     %0 = ttg.memdesc_reinterpret %arg0 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory> -> !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory>
744|     tt.return
745|   }
746| }
747| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttg.memdesc_reinterpret, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttg.memdesc_reinterpret、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 748-748
```mlir
748| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 749-751
```mlir
749| 
750| #shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
751| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 752-752
```mlir
752| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 753-754
```mlir
753|   tt.func public @tmem_subslice_non_tmem_source() {
754|     %md = ttg.local_alloc : () -> !ttg.memdesc<128x128xi32, #shared1, #ttg.shared_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tmem_subslice_non_tmem_source`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_subslice_non_tmem_source` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 755-760
```mlir
755|     // expected-error @+1 {{The source must be a tensor memory buffer.}}
756|     %sub = ttng.tmem_subslice %md {N = 0 : i32} : !ttg.memdesc<128x128xi32, #shared1, #ttg.shared_memory, mutable> -> !ttg.memdesc<128x128xi32, #shared1, #ttg.shared_memory, mutable>
757|     tt.return
758|   }
759| }
760| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 761-761
```mlir
761| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 762-763
```mlir
762| 
763| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 764-764
```mlir
764| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 765-766
```mlir
765|   tt.func public @tmem_subslice_rank_not_2() {
766|     %md = ttng.tmem_alloc : () -> !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tmem_subslice_rank_not_2`. Within it, the test exercises tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_subslice_rank_not_2` 为核心。测试在其中演示 tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 767-772
```mlir
767|     // expected-error @+1 {{The result must be a 2D tensor memory buffer.}}
768|     %sub = ttng.tmem_subslice %md {N = 0 : i32} : !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<2x128x128xf32, #tmem, #ttng.tensor_memory, mutable>
769|     tt.return
770|   }
771| }
772| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 773-773
```mlir
773| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 774-775
```mlir
774| 
775| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 776-776
```mlir
776| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 777-778
```mlir
777|   tt.func public @tmem_subslice_rows_mismatch() {
778|     %md = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tmem_subslice_rows_mismatch`. Within it, the test exercises tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_subslice_rows_mismatch` 为核心。测试在其中演示 tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 779-784
```mlir
779|     // expected-error @+1 {{The result must have the same number of rows as the source.}}
780|     %sub = ttng.tmem_subslice %md {N = 0 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<64x128xf32, #tmem, #ttng.tensor_memory, mutable, 128x128>
781|     tt.return
782|   }
783| }
784| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 785-785
```mlir
785| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 786-787
```mlir
786| 
787| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 788-788
```mlir
788| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 789-790
```mlir
789|   tt.func public @tmem_subslice_element_type_mismatch() {
790|     %md = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tmem_subslice_element_type_mismatch`. Within it, the test exercises tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_subslice_element_type_mismatch` 为核心。测试在其中演示 tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 791-796
```mlir
791|     // expected-error @+1 {{The source and result must have the same element type.}}
792|     %sub = ttng.tmem_subslice %md {N = 0 : i32} : !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable, 128x128>
793|     tt.return
794|   }
795| }
796| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 797-797
```mlir
797| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 798-799
```mlir
798| 
799| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 800-800
```mlir
800| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 801-802
```mlir
801|   tt.func public @tmem_subslice_alloc_shape_mismatch() {
802|     %md = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tmem_subslice_alloc_shape_mismatch`. Within it, the test exercises tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_subslice_alloc_shape_mismatch` 为核心。测试在其中演示 tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 803-808
```mlir
803|     // expected-error @+1 {{The source and result must have the same alloc shape.}}
804|     %sub = ttng.tmem_subslice %md {N = 0 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable, 2x128x128>
805|     tt.return
806|   }
807| }
808| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 809-809
```mlir
809| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 810-811
```mlir
810| 
811| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 812-812
```mlir
812| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 813-814
```mlir
813|   tt.func public @tmem_subslice_offset_alignment_invalid() {
814|     %md = ttng.tmem_alloc : () -> !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tmem_subslice_offset_alignment_invalid`. Within it, the test exercises tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_subslice_offset_alignment_invalid` 为核心。测试在其中演示 tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 815-820
```mlir
815|     // expected-error @+1 {{The split offset may not touch the tile}}
816|     %sub = ttng.tmem_subslice %md {N = 32 : i32} : !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable, 128x256>
817|     tt.return
818|   }
819| }
820| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 821-821
```mlir
821| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 822-823
```mlir
822| 
823| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 824-824
```mlir
824| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 825-826
```mlir
825|   tt.func public @tmem_subslice_offset_exceed() {
826|     %md = ttng.tmem_alloc : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This function-oriented block defines or enters `tmem_subslice_offset_exceed`. Within it, the test exercises tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `tmem_subslice_offset_exceed` 为核心。测试在其中演示 tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 827-832
```mlir
827|     // expected-error @+1 {{The split offset may not exceed the source shape}}
828|     %sub = ttng.tmem_subslice %md {N = 128 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable, 128x128>
829|     tt.return
830|   }
831| }
832| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.tmem_subslice, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.tmem_subslice、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 833-833
```mlir
833| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 834-836
```mlir
834| 
835| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
836| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 837-837
```mlir
837| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 838-839
```mlir
838|   tt.func public @async_tma_reduce_rejects_unsupported_kind(%arg0: !tt.tensordesc<32x32xf32, #shared>, %x: i32) {
839|     %src = ttg.local_alloc : () -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_reduce_rejects_unsupported_kind`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_reduce_rejects_unsupported_kind` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 840-845
```mlir
840|     // expected-error @below {{unsupported reduce kind inc for element type 'f32'}}
841|     ttng.async_tma_reduce inc, %arg0[%x, %x] %src : !tt.tensordesc<32x32xf32, #shared>, !ttg.memdesc<32x32xf32, #shared, #smem, mutable>
842|     tt.return
843|   }
844| }
845| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_reduce, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_reduce、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 846-846
```mlir
846| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 847-851
```mlir
847| 
848| #blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
849| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
850| #shared_bar = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
851| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 852-852
```mlir
852| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 853-858
```mlir
853|   tt.func public @async_tma_gather_requires_legal_x_offsets(%arg0: !tt.tensordesc<1x128xf16, #shared>) {
854|     %true = arith.constant true
855|     %c0_i32 = arith.constant 0 : i32
856|     %x_offsets = arith.constant dense<0> : tensor<32xi32, #blocked>
857|     %bar = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared_bar, #smem, mutable>
858|     %result = ttg.local_alloc : () -> !ttg.memdesc<32x128xf16, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_gather_requires_legal_x_offsets`. Within it, the test exercises constants, tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_gather_requires_legal_x_offsets` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 859-864
```mlir
859|     // expected-error @below {{x offsets must have at least 4 contiguous elements per thread}}
860|     ttng.async_tma_gather %arg0[%x_offsets, %c0_i32] %result, %bar, %true : !tt.tensordesc<1x128xf16, #shared>, tensor<32xi32, #blocked>, i32, !ttg.memdesc<1xi64, #shared_bar, #smem, mutable>, !ttg.memdesc<32x128xf16, #shared, #smem, mutable>, i1
861|     tt.return
862|   }
863| }
864| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_gather, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_gather、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 865-865
```mlir
865| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 866-868
```mlir
866| 
867| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 64}>
868| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 869-869
```mlir
869| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 870-871
```mlir
870|   tt.func public @async_tma_reduce_rejects_signed_i64_add(%arg0: !tt.tensordesc<32x32xsi64, #shared>, %x: i32) {
871|     %src = ttg.local_alloc : () -> !ttg.memdesc<32x32xi64, #shared, #smem, mutable>
```
**EN:** This function-oriented block defines or enters `async_tma_reduce_rejects_signed_i64_add`. Within it, the test exercises tt.func, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `async_tma_reduce_rejects_signed_i64_add` 为核心。测试在其中演示 tt.func、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 872-876
```mlir
872|     // expected-error @below {{unsupported reduce kind add for element type 'si64'}}
873|     ttng.async_tma_reduce add, %arg0[%x, %x] %src : !tt.tensordesc<32x32xsi64, #shared>, !ttg.memdesc<32x32xi64, #shared, #smem, mutable>
874|     tt.return
875|   }
876| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on ttng.async_tma_reduce, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 ttng.async_tma_reduce、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `--split-input-file`, `--verify-diagnostics`
- **CN:** 主要 pass 选项：`--split-input-file`，`--verify-diagnostics`
- **EN:** Dominant operations include `tt.func`, `module`, `tt.return`, `arith.constant`, `ttg.local_alloc`, `ttng.tmem_alloc`, `ttng.async_tma_copy_global_to_local`, `ttng.tmem_subslice`, `ttng.async_tma_gather`, `ttng.tmem_load`.
- **CN:** 主要操作包括 `tt.func`、`module`、`tt.return`、`arith.constant`、`ttg.local_alloc`、`ttng.tmem_alloc`、`ttng.async_tma_copy_global_to_local`、`ttng.tmem_subslice`、`ttng.async_tma_gather`、`ttng.tmem_load`。
- **EN:** The file contains 55 independently testable section(s). Expected diagnostic comments specify the exact verifier/pass failures. It validates that invalid IR is rejected with the intended diagnostics.
- **CN:** 该文件包含 55 个可独立测试的分段。expected 诊断注释给出了 verifier/pass 需要触发的精确报错。 它验证无效 IR 会被拒绝，并产生预期诊断。

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
- **EN:** Diagnostic verification is part of the contract, so parser/verifier error strings are also dependencies of the test.
- **CN:** 诊断验证也是该测试契约的一部分，因此 parser/verifier 的报错文本同样是依赖项。