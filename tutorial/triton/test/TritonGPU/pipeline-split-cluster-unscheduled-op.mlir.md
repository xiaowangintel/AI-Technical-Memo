# pipeline-split-cluster-unscheduled-op.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/pipeline-split-cluster-unscheduled-op.mlir`
- **EN:** Test for the MLIR/Triton pipeline `--crash, -tritongpu-test-pipeline-lower-loop` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `--crash, -tritongpu-test-pipeline-lower-loop` 的行为。
- **EN:** RUN pipeline(s): `// RUN: not --crash triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-test-pipeline-lower-loop 2>&1 | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: not --crash triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-test-pipeline-lower-loop 2>&1 | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```mlir
 1| // RUN: not --crash triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-test-pipeline-lower-loop 2>&1 | FileCheck %s
 2| 
 3| // Regression test for the `CoarseSchedule::splitClusterBefore`
 4| // implicit-insert bug.
 5| //
 6| // Before the fix, `splitClusterBefore` indexed `opToStageAndCluster`
 7| // (an `llvm::MapVector`) with `operator[]`, which silently inserts a
 8| // default entry for any key that is not present. Any op in the loop
 9| // body without `loop.cluster` / `loop.stage` attributes would therefore
10| // be added to the schedule with stage 0 and a default-constructed
11| // (invalid) cluster iterator. That "phantom" entry was later asserted
12| // on as "Op with invalid cluster!" inside
13| // `CoarseSchedule::getOpsInOrder`, crashing the pipeliner deep in the
14| // pipeline with no actionable diagnostic.
15| //
16| // With the fix, `splitClusterBefore` uses `find` and skips ops that
17| // are not in the schedule. The pass instead reports the unscheduled op
18| // cleanly via `lowerLoads`'s "op not found in the schedule" diagnostic
```
**EN:** This header defines how the test is executed. It runs `not --crash triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-test-pipeline-lower-loop 2>&1 | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `not --crash triton-opt %s -split-input-file -allow-unregistered-dialect -tritongpu-test-pipeline-lower-loop 2>&1 | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 19-20
```mlir
19| // at the offending operation.
20| 
```
**EN:** These comments provide local intent, caveats, or test expectations for the adjacent IR.
**CN:** 这些注释为相邻 IR 提供局部意图、限制条件或测试预期。

### Lines 21-24
```mlir
21| // CHECK: error: op not found in the schedule
22| // CHECK-NEXT: "unscheduled.op"() : () -> ()
23| // CHECK-NOT: Op with invalid cluster
24| 
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 25-28
```mlir
25| #blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
26| #shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
27| #smem = #ttg.shared_memory
28| #tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 29-29
```mlir
29| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `cuda:100`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `cuda:100`。

### Lines 30-47
```mlir
30|   tt.func public @split_cluster_with_unscheduled_op(
31|       %arg0: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
32|       %arg1: tensor<128x128x!tt.ptr<f16>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
33|       %arg2: tensor<128x128x!tt.ptr<f32>, #blocked> {tt.contiguity = dense<[1, 16]> : tensor<2xi32>, tt.divisibility = dense<[16, 16]> : tensor<2xi32>},
34|       %arg3: i32) {
35|     %true = arith.constant true
36|     %c0_i32 = arith.constant 0 : i32
37|     %c1_i32 = arith.constant 1 : i32
38|     %0, %acc_tok = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
39|     scf.for %arg5 = %c0_i32 to %arg3 step %c1_i32 iter_args(%tok = %acc_tok) -> !ttg.async.token : i32 {
40|       %2 = tt.load %arg0 {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
41|       %3 = ttg.local_alloc %2 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
42|       %4 = tt.load %arg1 {loop.cluster = 4 : i32, loop.stage = 0 : i32} : tensor<128x128x!tt.ptr<f16>, #blocked>
43|       %5 = ttg.local_alloc %4 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : (tensor<128x128xf16, #blocked>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
44|       %6 = tt.load %arg2 {loop.cluster = 2 : i32, loop.stage = 2 : i32} : tensor<128x128x!tt.ptr<f32>, #blocked>
45|       // An op missing loop.cluster / loop.stage attributes inside the
46|       // window scanned by splitClusterBefore. The buggy version would
47|       // default-insert this op into opToStageAndCluster and crash later
```
**EN:** This function-oriented block defines or enters `split_cluster_with_unscheduled_op`. Within it, the test exercises constants, masked or vectorized loads, tt.func, shared/local memory allocation, tensor-memory allocation, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `split_cluster_with_unscheduled_op` 为核心。测试在其中演示 常量、带掩码或向量化的加载、tt.func、共享/本地内存分配、张量内存分配，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 48-57
```mlir
48|       // with "Op with invalid cluster!". The fix must skip it instead.
49|       "unscheduled.op"() : () -> ()
50|       %store_tok = ttng.tmem_store %6, %0[%tok], %true {loop.cluster = 2 : i32, loop.stage = 2 : i32} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
51|       %mma_tok = ttng.tc_gen5_mma %3, %5, %0[%store_tok], %true, %true {loop.cluster = 2 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32} : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
52|       %res, %load_tok = ttng.tmem_load %0[%mma_tok] {loop.cluster = 2 : i32, loop.stage = 3 : i32} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
53|       scf.yield %load_tok : !ttg.async.token
54|     } {tt.scheduled_max_stage = 3 : i32}
55|     tt.return
56|   }
57| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining ttng.tmem_store, Gen5 tensor-core MMA ops, ttng.tmem_load, loop/if yielded values, tt.return.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 ttng.tmem_store、Gen5 张量核 MMA 操作、ttng.tmem_load、循环/分支产出值、tt.return。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `--crash`, `-split-input-file`, `-allow-unregistered-dialect`, `-tritongpu-test-pipeline-lower-loop`
- **CN:** 主要 pass 选项：`--crash`，`-split-input-file`，`-allow-unregistered-dialect`，`-tritongpu-test-pipeline-lower-loop`
- **EN:** Dominant operations include `arith.constant`, `tt.load`, `tt.func`, `ttg.local_alloc`, `module`, `ttng.tmem_alloc`, `scf.for`, `ttng.tmem_store`, `ttng.tc_gen5_mma`, `ttng.tmem_load`.
- **CN:** 主要操作包括 `arith.constant`、`tt.load`、`tt.func`、`ttg.local_alloc`、`module`、`ttng.tmem_alloc`、`scf.for`、`ttng.tmem_store`、`ttng.tc_gen5_mma`、`ttng.tmem_load`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x1, CHECK-NEXT x1, CHECK-NOT x1. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×1，CHECK-NEXT ×1，CHECK-NOT ×1。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

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