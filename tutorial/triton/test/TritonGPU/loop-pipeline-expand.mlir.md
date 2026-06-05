# loop-pipeline-expand.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/loop-pipeline-expand.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritongpu-pipeline` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritongpu-pipeline` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritongpu-pipeline | FileCheck %s --check-prefixes=CHECK`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritongpu-pipeline | FileCheck %s --check-prefixes=CHECK`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritongpu-pipeline | FileCheck %s --check-prefixes=CHECK
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritongpu-pipeline | FileCheck %s --check-prefixes=CHECK` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritongpu-pipeline | FileCheck %s --check-prefixes=CHECK`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-8
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
4| #blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
5| #mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 8]}>
6| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 32}>
7| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 32}>
8| #smem = #ttg.shared_memory
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 9-9
```mlir
9| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:90`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:90`。

### Lines 10-10
```mlir
10|   // CHECK-LABEL: @pipeline_load_mmav3
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @pipeline_load_mmav3 anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @pipeline_load_mmav3 这样的标签用于锚定匹配范围。

### Lines 11-14
```mlir
11|   tt.func public @pipeline_load_mmav3(%arg0: tensor<256x128xf32, #mma>, %arg1: tensor<256x32x!tt.ptr<f32>, #blocked>, %arg2: tensor<32x128x!tt.ptr<f32>, #blocked1>, %arg3: tensor<256x32xi32, #blocked>, %arg4: tensor<32x128xi32, #blocked1>) -> (tensor<256x128xf32, #mma>, tensor<256x32x!tt.ptr<f32>, #blocked>, tensor<32x128x!tt.ptr<f32>, #blocked1>) {
12|     %c0_i32 = arith.constant 0 : i32
13|     %c1_i32 = arith.constant 1 : i32
14|     %c128_i32 = arith.constant 128 : i32
```
**EN:** This function-oriented block defines or enters `pipeline_load_mmav3`. Within it, the test exercises constants, tt.func, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `pipeline_load_mmav3` 为核心。测试在其中演示 常量、tt.func，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 15-17
```mlir
15|     // CHECK: ttg.local_alloc : () -> !ttg.memdesc<4x256x32xf32
16|     // CHECK: ttg.local_alloc : () -> !ttg.memdesc<4x32x128xf32
17|     %0:3 = scf.for %arg5 = %c0_i32 to %c128_i32 step %c1_i32 iter_args(%arg6 = %arg0, %arg7 = %arg1, %arg8 = %arg2) -> (tensor<256x128xf32, #mma>, tensor<256x32x!tt.ptr<f32>, #blocked>, tensor<32x128x!tt.ptr<f32>, #blocked1>)  : i32 {
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 18-35
```mlir
18|       // CHECK: ttg.memdesc_index {{.*}} : !ttg.memdesc<4x256x32xf32
19|       // CHECK: ttg.async_wait {{.*}} {num = 4 : i32}
20|       // CHECK: ttg.memdesc_index {{.*}} : !ttg.memdesc<4x32x128xf32
21|       // CHECK: ttng.warp_group_dot {{.*}} {inputPrecision = 0 : i32, isAsync = true}
22|       // CHECK: ttng.warp_group_dot_wait {{.*}} {pendings = 1 : i32}
23|       %1 = tt.load %arg7 {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<256x32x!tt.ptr<f32>, #blocked>
24|       %2 = ttg.local_alloc %1 {loop.cluster = 0 : i32, loop.stage = 3 : i32} : (tensor<256x32xf32, #blocked>) -> !ttg.memdesc<256x32xf32, #shared, #smem>
25|       %3 = tt.load %arg8 {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<32x128x!tt.ptr<f32>, #blocked1>
26|       %4 = ttg.local_alloc %3 {loop.cluster = 0 : i32, loop.stage = 3 : i32} : (tensor<32x128xf32, #blocked1>) -> !ttg.memdesc<32x128xf32, #shared1, #smem>
27|       %5 = ttng.warp_group_dot %2, %4, %arg6 {inputPrecision = 0 : i32, loop.cluster = 0 : i32, loop.stage = 3 : i32} : !ttg.memdesc<256x32xf32, #shared, #smem> * !ttg.memdesc<32x128xf32, #shared1, #smem> -> tensor<256x128xf32, #mma>
28|       %6 = tt.addptr %arg7, %arg3 {loop.cluster = 3 : i32, loop.stage = 1 : i32} : tensor<256x32x!tt.ptr<f32>, #blocked>, tensor<256x32xi32, #blocked>
29|       %7 = tt.addptr %arg8, %arg4 {loop.cluster = 3 : i32, loop.stage = 1 : i32} : tensor<32x128x!tt.ptr<f32>, #blocked1>, tensor<32x128xi32, #blocked1>
30|       scf.yield %5, %6, %7 : tensor<256x128xf32, #mma>, tensor<256x32x!tt.ptr<f32>, #blocked>, tensor<32x128x!tt.ptr<f32>, #blocked1>
31|     } {tt.num_stages = 4 : i32, tt.scheduled_max_stage = 1 : i32}
32|     tt.return %0#0, %0#1, %0#2 : tensor<256x128xf32, #mma>, tensor<256x32x!tt.ptr<f32>, #blocked>, tensor<32x128x!tt.ptr<f32>, #blocked1>
33|   }
34| }
35| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining masked or vectorized loads, shared/local memory allocation, pointer arithmetic, ttng.warp_group_dot, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 带掩码或向量化的加载、共享/本地内存分配、指针算术、ttng.warp_group_dot、循环/分支产出值。

### Lines 36-36
```mlir
36| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 37-38
```mlir
37| 
38| #s = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 39-39
```mlir
39| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 40-40
```mlir
40|   // CHECK-LABEL: @expand_loop_without_results
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @expand_loop_without_results anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @expand_loop_without_results 这样的标签用于锚定匹配范围。

### Lines 41-48
```mlir
41|   tt.func public @expand_loop_without_results() {
42|     %c0 = arith.constant 0 : i32
43|     %c16 = arith.constant 16 : i32
44|     %true = arith.constant true
45|     %a = ttng.tmem_alloc : () -> !ttg.memdesc<64x64xbf16, #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>, #ttng.tensor_memory, mutable>
46|     %b = ttg.local_alloc : () -> !ttg.memdesc<64x64xbf16, #s, #ttg.shared_memory, mutable>
47|     %c = ttng.tmem_alloc : () -> !ttg.memdesc<64x64xf32, #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>, #ttng.tensor_memory, mutable>
48|     %bar = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>, #ttg.shared_memory, mutable>
```
**EN:** This function-oriented block defines or enters `expand_loop_without_results`. Within it, the test exercises constants, tt.func, tensor-memory allocation, shared/local memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `expand_loop_without_results` 为核心。测试在其中演示 常量、tt.func、张量内存分配、共享/本地内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 49-60
```mlir
49|     // CHECK: scf.for
50|     // CHECK:   ttng.tc_gen5_mma
51|     // CHECK:   ttng.wait_barrier
52|     scf.for %j = %c0 to %c16 step %c16 : i32 {
53|       ttng.tc_gen5_mma %a, %b, %c, %true, %true, %bar[%true] {is_async, loop.cluster = 2 : i32, loop.stage = 0 : i32} : !ttg.memdesc<64x64xbf16, #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>, #ttng.tensor_memory, mutable>, !ttg.memdesc<64x64xbf16, #s, #ttg.shared_memory, mutable>, !ttg.memdesc<64x64xf32, #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>, #ttng.tensor_memory, mutable>, !ttg.memdesc<1xi64, #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>, #ttg.shared_memory, mutable>
54|       ttng.wait_barrier %bar, %c0 deps %a, %b {loop.cluster = 1 : i32, loop.stage = 1 : i32} : !ttg.memdesc<1xi64, #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>, #ttg.shared_memory, mutable>, !ttg.memdesc<64x64xbf16, #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>, #ttng.tensor_memory, mutable>, !ttg.memdesc<64x64xbf16, #s, #ttg.shared_memory, mutable>
55|       scf.yield
56|     } {tt.num_stages = 4 : i32, tt.scheduled_max_stage = 1 : i32}
57|     tt.return
58|   }
59| }
60| 
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops, Gen5 tensor-core MMA ops, ttng.wait_barrier, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环、Gen5 张量核 MMA 操作、ttng.wait_barrier、循环/分支产出值、tt.return。

### Lines 61-61
```mlir
61| // -----
```
**EN:** This separator starts a new independent test case inside the same file, typically used with `-split-input-file`.
**CN:** 这个分隔符在同一文件中开启新的独立测试用例，通常与 `-split-input-file` 配合使用。

### Lines 62-67
```mlir
62| 
63| #blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [8, 1], order = [0, 1]}>
64| #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
65| #shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
66| #smem = #ttg.shared_memory
67| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 68-68
```mlir
68| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 69-69
```mlir
69|   // CHECK-LABEL: @nested_loop_gen5_mma
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @nested_loop_gen5_mma anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @nested_loop_gen5_mma 这样的标签用于锚定匹配范围。

### Lines 70-87
```mlir
70|   tt.func public @nested_loop_gen5_mma(%arg0: !tt.ptr<bf16>, %arg1: i1) {
71|     %cst = arith.constant dense<0.000000e+00> : tensor<1024x64xf32, #blocked>
72|     %true = arith.constant true
73|     %false = arith.constant false
74|     %c0_i32 = arith.constant 0 : i32
75|     %c16_i32 = arith.constant 16 : i32
76|     %c32_i32 = arith.constant 32 : i32
77|     %0 = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<64x64x!tt.ptr<bf16>, #blocked>
78|     %1 = tt.load %0 : tensor<64x64x!tt.ptr<bf16>, #blocked>
79|     %2 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared, #smem, mutable>
80|     %3 = ttg.local_alloc %1 {loop.cluster = 2 : i32, loop.stage = 0 : i32} : (tensor<64x64xbf16, #blocked>) -> !ttg.memdesc<64x64xbf16, #shared1, #smem>
81|     %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<1024x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
82|     %4 = ttng.tmem_store %cst, %result[%token], %true : tensor<1024x64xf32, #blocked> -> !ttg.memdesc<1024x64xf32, #tmem, #ttng.tensor_memory, mutable>
83|     %result_0 = ttng.tmem_alloc {loop.cluster = 0 : i32, loop.stage = 0 : i32} : () -> !ttg.memdesc<1024x64xbf16, #tmem, #ttng.tensor_memory, mutable>
84|     scf.for %arg2 = %c0_i32 to %c32_i32 step %c16_i32  : i32 {
85|       // In order for both the outer and inner loop to be pipelined, the inner
86|       // loop cannot be directly nested in the outer loop, so add an if in the
87|       // middle.
```
**EN:** This function-oriented block defines or enters `nested_loop_gen5_mma`. Within it, the test exercises constants, tt.func, shared/local memory allocation, tensor-memory allocation, broadcasted scalars or pointers, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `nested_loop_gen5_mma` 为核心。测试在其中演示 常量、tt.func、共享/本地内存分配、张量内存分配、广播后的标量或指针，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 88-98
```mlir
88|       scf.if %arg1 {
89|         %5 = scf.for %arg3 = %c0_i32 to %arg2 step %c16_i32 iter_args(%arg4 = %4) -> (!ttg.async.token)  : i32 {
90|           %6 = ttng.tc_gen5_mma %result_0, %3, %result[%arg4], %false, %true, %2[%true] {is_async, loop.cluster = 2 : i32, loop.stage = 0 : i32} : !ttg.memdesc<1024x64xbf16, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<64x64xbf16, #shared1, #smem>, !ttg.memdesc<1024x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1xi64, #shared, #smem, mutable>
91|           ttng.wait_barrier %2, %c0_i32 deps %result_0, %3 {loop.cluster = 1 : i32, loop.stage = 1 : i32} : !ttg.memdesc<1xi64, #shared, #smem, mutable>, !ttg.memdesc<1024x64xbf16, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<64x64xbf16, #shared1, #smem>
92|           scf.yield %6 : !ttg.async.token
93|         } {tt.num_stages = 4 : i32, tt.scheduled_max_stage = 1 : i32}
94|       } {loop.cluster = 2 : i32, loop.stage = 1 : i32}
95|     } {tt.num_stages = 2 : i32, tt.scheduled_max_stage = 1 : i32}
96|     tt.return
97|   }
98| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured conditionals, structured loops, Gen5 tensor-core MMA ops, ttng.wait_barrier, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化条件分支、结构化循环、Gen5 张量核 MMA 操作、ttng.wait_barrier、循环/分支产出值。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritongpu-pipeline`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritongpu-pipeline`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `ttg.local_alloc`, `scf.for`, `ttng.tmem_alloc`, `module`, `tt.load`, `scf.yield`, `tt.return`, `tt.addptr`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`ttg.local_alloc`、`scf.for`、`ttng.tmem_alloc`、`module`、`tt.load`、`scf.yield`、`tt.return`、`tt.addptr`。
- **EN:** The file contains 2 independently testable section(s). Check styles used: CHECK x11, CHECK-LABEL x3. Important labels include @pipeline_load_mmav3, @expand_loop_without_results, @nested_loop_gen5_mma. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 2 个可独立测试的分段。使用的检查类型：CHECK ×11，CHECK-LABEL ×3。 关键标签包括 @pipeline_load_mmav3，@expand_loop_without_results，@nested_loop_gen5_mma。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

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