# test_promotion_to_tensor_memory.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonNvidiaGPU/test_promotion_to_tensor_memory.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-promote-lhs-to-tmem` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-promote-lhs-to-tmem` 的行为。
- **EN:** RUN pipeline(s): `// RUN:triton-opt %s -split-input-file -tritongpu-promote-lhs-to-tmem | FileCheck %s`
- **CN:** RUN 流水线：`// RUN:triton-opt %s -split-input-file -tritongpu-promote-lhs-to-tmem | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN:triton-opt %s -split-input-file -tritongpu-promote-lhs-to-tmem | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritongpu-promote-lhs-to-tmem | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritongpu-promote-lhs-to-tmem | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-7
```mlir
3| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 32}>
4| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 16}>
5| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
6| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 2], order = [0, 1]}>
7| // Incompatible access layout for tmem; tmem access requires one thread per datapath
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 8-10
```mlir
 8| #blocked1 = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 8], order = [0, 1]}>
 9| #blocked2 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 2], order = [1, 0]}>
10| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 11-11
```mlir
11| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 12-12
```mlir
12|   // CHECK-LABEL: @no_tmem_promotion
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @no_tmem_promotion anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @no_tmem_promotion 这样的标签用于锚定匹配范围。

### Lines 13-18
```mlir
13|   tt.func public @no_tmem_promotion(
14|     %lhs: tensor<128x32xf16, #blocked1>,
15|     %rhs: tensor<32x256xf16, #blocked2>
16|   ) {
17|     %true = arith.constant true
18|     %cst = arith.constant dense<0.0> : tensor<128x256xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `no_tmem_promotion`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `no_tmem_promotion` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 19-22
```mlir
19|     // CHECK: ttng.tmem_alloc %[[CST:.*]] : (tensor<128x256xf32, #[[BLOCKED:blocked[0-9]*]]>) -> !ttg.memdesc<128x256xf32, #tmem
20|     %tmem = ttng.tmem_alloc %cst :
21|       (tensor<128x256xf32, #blocked>) ->
22|       !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 23-35
```mlir
23|     // CHECK-NOT: ttng.tmem_alloc %[[ARG0:.*]] : (tensor<128x32xf32, #[[BLOCKED:blocked[0-9]*]]>) -> !ttg.memdesc<128x32xf32, #[[TMEM:tmem[0-9]*]]
24|     %lhs_shared = ttg.local_alloc %lhs : (tensor<128x32xf16, #blocked1>) -> !ttg.memdesc<128x32xf16, #shared, #ttg.shared_memory>
25|     %rhs_shared = ttg.local_alloc %rhs : (tensor<32x256xf16, #blocked2>) -> !ttg.memdesc<32x256xf16, #shared1, #ttg.shared_memory>
26| 
27|     ttng.tc_gen5_mma %lhs_shared, %rhs_shared, %tmem, %true, %true :
28|        !ttg.memdesc<128x32xf16, #shared, #ttg.shared_memory>,
29|        !ttg.memdesc<32x256xf16, #shared1, #ttg.shared_memory>,
30|        !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
31| 
32|     tt.return
33|   }
34| }
35| 
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, Gen5 tensor-core MMA ops, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、Gen5 张量核 MMA 操作、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 36-36
```mlir
36| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 37-43
```mlir
37| 
38| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 32}>
39| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 32}>
40| #shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
41| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 2], order = [0, 1]}>
42| #blocked2 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 2], order = [1, 0]}>
43| // Compatible layout for tmem access
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 44-45
```mlir
44| #blocked3 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [32, 1], warpsPerCTA = [4, 2], order = [0, 1]}>
45| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 46-46
```mlir
46| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。

### Lines 47-47
```mlir
47|   // CHECK-LABEL: @promote_lhs_to_tmem
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @promote_lhs_to_tmem anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @promote_lhs_to_tmem 这样的标签用于锚定匹配范围。

### Lines 48-53
```mlir
48|   tt.func public @promote_lhs_to_tmem(
49|     %lhs: tensor<128x32xf16, #blocked3>,
50|     %rhs: tensor<32x256xf16, #blocked2>
51|   ) {
52|     %true = arith.constant true
53|     %cst = arith.constant dense<0.0> : tensor<128x256xf32, #blocked>
```
**EN:** This function-oriented block defines or enters `promote_lhs_to_tmem`. Within it, the test exercises tt.func, constants, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `promote_lhs_to_tmem` 为核心。测试在其中演示 tt.func、常量，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 54-57
```mlir
54|     // CHECK: ttng.tmem_alloc %[[CST:.*]] : (tensor<128x256xf32, #[[BLOCKED:blocked[0-9]*]]>) -> !ttg.memdesc<128x256xf32, #tmem
55|     %tmem = ttng.tmem_alloc %cst :
56|       (tensor<128x256xf32, #blocked>) ->
57|       !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tensor-memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 张量内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 58-69
```mlir
58|     // CHECK: ttng.tmem_alloc %[[ARG0:.*]] : (tensor<128x32xf16, #[[BLOCKED:blocked[0-9]*]]>) -> !ttg.memdesc<128x32xf16, #[[TMEM:tmem[0-9]*]]
59|     %lhs_shared = ttg.local_alloc %lhs : (tensor<128x32xf16, #blocked3>) -> !ttg.memdesc<128x32xf16, #shared, #ttg.shared_memory>
60|     %rhs_shared = ttg.local_alloc %rhs : (tensor<32x256xf16, #blocked2>) -> !ttg.memdesc<32x256xf16, #shared1, #ttg.shared_memory>
61| 
62|     ttng.tc_gen5_mma %lhs_shared, %rhs_shared, %tmem, %true, %true :
63|        !ttg.memdesc<128x32xf16, #shared, #ttg.shared_memory>,
64|        !ttg.memdesc<32x256xf16, #shared1, #ttg.shared_memory>,
65|        !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
66| 
67|     tt.return
68|   }
69| }
```
**EN:** This operational block carries the concrete IR steps for the test, centered on shared/local memory allocation, Gen5 tensor-core MMA ops, tt.return. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 共享/本地内存分配、Gen5 张量核 MMA 操作、tt.return。这些行共同表达了 pass 流水线必须保持或重写的行为。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritongpu-promote-lhs-to-tmem`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritongpu-promote-lhs-to-tmem`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `ttg.local_alloc`, `module`, `ttng.tmem_alloc`, `ttng.tc_gen5_mma`, `tt.return`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`ttg.local_alloc`、`module`、`ttng.tmem_alloc`、`ttng.tc_gen5_mma`、`tt.return`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x3, CHECK-LABEL x2, CHECK-NOT x1. Important labels include @no_tmem_promotion, @promote_lhs_to_tmem. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×3，CHECK-LABEL ×2，CHECK-NOT ×1。 关键标签包括 @no_tmem_promotion，@promote_lhs_to_tmem。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。