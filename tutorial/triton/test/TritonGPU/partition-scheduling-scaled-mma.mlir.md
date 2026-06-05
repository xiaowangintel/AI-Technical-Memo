# partition-scheduling-scaled-mma.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/partition-scheduling-scaled-mma.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--tritongpu-hoist-tmem-alloc, --tritongpu-partition-scheduling` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--tritongpu-hoist-tmem-alloc, --tritongpu-partition-scheduling` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s --split-input-file --tritongpu-hoist-tmem-alloc --tritongpu-partition-scheduling -allow-unregistered-dialect | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s --split-input-file --tritongpu-hoist-tmem-alloc --tritongpu-partition-scheduling -allow-unregistered-dialect | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```mlir
1| // RUN: triton-opt %s --split-input-file --tritongpu-hoist-tmem-alloc --tritongpu-partition-scheduling -allow-unregistered-dialect | FileCheck %s
2| 
3| // Verify that TCGen5MMAScaledOp is classified as a data value in partition
4| // scheduling, just like TCGen5MMAOp. Both ops have an optional async token
5| // as output 0, and initialDataValues should mark it as a data value so that
6| // partition scheduling properly propagates the data dependency.
7| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s --split-input-file --tritongpu-hoist-tmem-alloc --tritongpu-partition-scheduling -allow-unregistered-dialect | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s --split-input-file --tritongpu-hoist-tmem-alloc --tritongpu-partition-scheduling -allow-unregistered-dialect | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 8-17
```mlir
 8| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
 9| #load_blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
10| 
11| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
12| #shared_T = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
13| #shared_scales = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [4, 3, 2, 1, 0]}>
14| 
15| #smem = #ttg.shared_memory
16| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
17| 
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 18-19
```mlir
18| module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
19| 
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 20-20
```mlir
20| // CHECK-LABEL: @scaled_mma_with_loads
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject. Labels such as CHECK-LABEL: @scaled_mma_with_loads anchor the match scope.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。 像 CHECK-LABEL: @scaled_mma_with_loads 这样的标签用于锚定匹配范围。

### Lines 21-33
```mlir
21| tt.func public @scaled_mma_with_loads(
22|   %A_shared: !ttg.memdesc<128x128xf16, #shared, #smem>,
23|   %B_desc: !tt.tensordesc<128x128xf16, #shared>,
24|   %A_scale_shared: !ttg.memdesc<1x2x32x4x4xi8, #shared_scales, #smem>,
25|   %B_scale_shared: !ttg.memdesc<1x2x32x4x4xi8, #shared_scales, #smem>,
26|   %n_tiles: i32
27| ) {
28|   %true = arith.constant true
29|   %c0_i32 = arith.constant 0 : i32
30|   %c1_i32 = arith.constant 1 : i32
31| 
32|   %acc_tmem, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
33| 
```
**EN:** This function-oriented block defines or enters `scaled_mma_with_loads`. Within it, the test exercises constants, tt.func, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `scaled_mma_with_loads` 为核心。测试在其中演示 常量、tt.func、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 34-45
```mlir
34|   // CHECK: scf.for
35|   %loop_out:2 = scf.for %i = %c0_i32 to %n_tiles step %c1_i32 iter_args(
36|     %iter_acc_tok = %acc_tok,
37|     %iter_acc_tmem = %acc_tmem
38|   ) -> (
39|     !ttg.async.token,
40|     !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
41|   ) : i32 {
42| 
43|     // Load partition. Feeding this load into the MMA keeps the test live after
44|     // canonicalization while still requiring the scaled MMA token result to
45|     // propagate the dependency to tmem_load.
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining structured loops.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 结构化循环。

### Lines 46-51
```mlir
46|     // CHECK-COUNT-2: ttg.partition = array<i32: 2>
47|     %B = tt.descriptor_load %B_desc[%i, %c0_i32] : !tt.tensordesc<128x128xf16, #shared> -> tensor<128x128xf16, #load_blocked>
48|     %B_shared = ttg.local_alloc %B : (tensor<128x128xf16, #load_blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem>
49| 
50|     // Compute partition: tc_gen5_mma_scaled should get partition 1
51|     // just like tc_gen5_mma does in the existing tests.
```
**EN:** This operational block carries the concrete IR steps for the test, centered on tt.descriptor_load, shared/local memory allocation. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 tt.descriptor_load、共享/本地内存分配。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 52-53
```mlir
52|     // CHECK: ttg.memdesc_trans {{.*}} {order = array<i32: 1, 0>, ttg.partition = array<i32: 1>}
53|     %B_trans = ttg.memdesc_trans %B_shared {order = array<i32: 1, 0>} : !ttg.memdesc<128x128xf16, #shared, #smem> -> !ttg.memdesc<128x128xf16, #shared_T, #smem>
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 54-57
```mlir
54|     // CHECK: ttng.tc_gen5_mma_scaled {{.*}} {ttg.partition = array<i32: 1>}
55|     %mma_tok = ttng.tc_gen5_mma_scaled %A_shared, %B_trans, %iter_acc_tmem[%iter_acc_tok], %A_scale_shared, %B_scale_shared, %true, %true lhs = e5m2 rhs = e5m2 : !ttg.memdesc<128x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf16, #shared_T, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<1x2x32x4x4xi8, #shared_scales, #smem>, !ttg.memdesc<1x2x32x4x4xi8, #shared_scales, #smem>
56| 
57|     // Data partition: tmem_load should get partition 0
```
**EN:** These lines close the current region/function/module and finalize the value flow established above.
**CN:** 这些行关闭当前 region/函数/模块，并收束前面建立的数值流。

### Lines 58-63
```mlir
58|     // CHECK-COUNT-2: ttg.partition = array<i32: 0>
59|     %QK, %QK_load_tok = ttng.tmem_load %iter_acc_tmem[%mma_tok] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
60| 
61|     "use"(%QK) {data} : (tensor<128x128xf32, #blocked>) -> ()
62| 
63|     scf.yield %QK_load_tok, %iter_acc_tmem : !ttg.async.token, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.tmem_load, loop/if yielded values.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.tmem_load、循环/分支产出值。

### Lines 64-72
```mlir
64|     // CHECK: scf.yield {ttg.partition = array<i32: 0, 1, 2>}
65|     // CHECK: ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>]
66|   } {tt.warp_specialize}
67| 
68|   "use"(%loop_out#0) : (!ttg.async.token) -> ()
69|   tt.return
70| }
71| 
72| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 tt.return。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `--split-input-file`, `--tritongpu-hoist-tmem-alloc`, `--tritongpu-partition-scheduling`, `-allow-unregistered-dialect`
- **CN:** 主要 pass 选项：`--split-input-file`，`--tritongpu-hoist-tmem-alloc`，`--tritongpu-partition-scheduling`，`-allow-unregistered-dialect`
- **EN:** Dominant operations include `arith.constant`, `tt.func`, `module`, `ttng.tmem_alloc`, `scf.for`, `tt.descriptor_load`, `ttg.local_alloc`, `ttg.memdesc_trans`, `ttng.tc_gen5_mma_scaled`, `ttng.tmem_load`.
- **CN:** 主要操作包括 `arith.constant`、`tt.func`、`module`、`ttng.tmem_alloc`、`scf.for`、`tt.descriptor_load`、`ttg.local_alloc`、`ttg.memdesc_trans`、`ttng.tc_gen5_mma_scaled`、`ttng.tmem_load`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x5, CHECK-COUNT x2, CHECK-LABEL x1. Important labels include @scaled_mma_with_loads. It validates that the selected pass pipeline rewrites each labeled test case into the checked form.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×5，CHECK-COUNT ×2，CHECK-LABEL ×1。 关键标签包括 @scaled_mma_with_loads。 它验证所选 pass 流水线会把每个带标签的测试用例重写成检查所要求的形式。

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
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。