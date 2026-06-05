# loop-pipeline-combine-waits.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/loop-pipeline-combine-waits.mlir`
- **EN:** Test for the MLIR/Triton pipeline `-tritonamdgpu-schedule-loops="num_stages=3", -tritonamdgpu-pipeline="use_async_copy=1` using this file as input.
- **CN:** 该文件作为输入，测试 MLIR/Triton 流水线 `-tritonamdgpu-schedule-loops="num_stages=3", -tritonamdgpu-pipeline="use_async_copy=1` 的行为。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline="use_async_copy=1 use_pingpong=1" | FileCheck %s`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline="use_async_copy=1 use_pingpong=1" | FileCheck %s`

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
1| // RUN: triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline="use_async_copy=1 use_pingpong=1" | FileCheck %s
2| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline="use_async_copy=1 use_pingpong=1" | FileCheck %s` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -tritonamdgpu-schedule-loops="num_stages=3" -tritonamdgpu-pipeline="use_async_copy=1 use_pingpong=1" | FileCheck %s`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 3-4
```mlir
3| #blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
4| #mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable type/layout/encoding aliases that keep later IR readable. They predefine the structural context for the operations that follow, especially IR structure and attributes.
**CN:** 这些行声明了可复用的类型/布局/编码别名，使后续 IR 更易读。它们为后续操作预先设定结构上下文，尤其是 IR 结构与属性。

### Lines 5-5
```mlir
5| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 6-7
```mlir
6|   // CHECK-LABEL: tt.func @simple_pipelined_load
7|   // We expect one ttg.async_wait in the epilogue, one in the loop and one in the prologue
```
**EN:** This function-oriented block defines or enters `simple_pipelined_load`. Within it, the test exercises IR structure and attributes, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_pipelined_load` 为核心。测试在其中演示 IR 结构与属性，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 8-15
```mlir
 8|   // CHECK: ttg.async_wait
 9|   // CHECK-NOT: ttg.async_wait
10|   // CHECK: scf.for
11|   // CHECK: ttg.async_wait
12|   // CHECK-NOT: ttg.async_wait
13|   // CHECK: scf.yield
14|   // CHECK: ttg.async_wait
15|   // CHECK-NOT: ttg.async_wait
```
**EN:** This block contains FileCheck or expected-diagnostic assertions. It constrains the transformed IR text, forbids regressions, and documents what the pass should emit or reject.
**CN:** 这一段包含 FileCheck 或 expected-diagnostic 断言，用来约束变换后 IR 的文本形式、阻止回归，并说明 pass 应该产生或拒绝什么。

### Lines 16-33
```mlir
16|   tt.func @simple_pipelined_load(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg2: tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>, %arg3: i32, %arg4: i32) -> tensor<128x16xf32, #mma> {
17|     %c0_i32 = arith.constant 0 : i32
18|     %cst = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma>
19|     %0 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<1x16x!tt.ptr<f16>, #blocked>
20|     %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
21|     %2 = tt.expand_dims %1 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
22|     %3 = tt.broadcast %0 : tensor<1x16x!tt.ptr<f16>, #blocked> -> tensor<64x16x!tt.ptr<f16>, #blocked>
23|     %4 = tt.broadcast %2 : tensor<64x1xi32, #blocked> -> tensor<64x16xi32, #blocked>
24|     %5 = tt.addptr %3, %4 : tensor<64x16x!tt.ptr<f16>, #blocked>, tensor<64x16xi32, #blocked>
25|     %6 = scf.for %arg6 = %c0_i32 to %arg3 step %arg4 iter_args(%arg5 = %cst) -> (tensor<128x16xf32, #mma>)  : i32 {
26|       %7 = tt.load %5 : tensor<64x16x!tt.ptr<f16>, #blocked>
27|       %8 = ttg.convert_layout %7 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
28|       %9 = tt.dot %arg2, %8, %cst : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
29|       scf.yield %9 : tensor<128x16xf32, #mma>
30|     }
31|     tt.return %6 : tensor<128x16xf32, #mma>
32|   }
33| }
```
**EN:** This function-oriented block defines or enters `simple_pipelined_load`. Within it, the test exercises tt.func, constants, tensor broadcasting, broadcasted scalars or pointers, lane/block index ranges, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `simple_pipelined_load` 为核心。测试在其中演示 tt.func、常量、张量广播、广播后的标量或指针、lane/block 索引范围，这些操作构成了被优化、lower 或验证的主要数据流。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-tritonamdgpu-schedule-loops="num_stages=3"`, `-tritonamdgpu-pipeline="use_async_copy=1`
- **CN:** 主要 pass 选项：`-split-input-file`，`-tritonamdgpu-schedule-loops="num_stages=3"`，`-tritonamdgpu-pipeline="use_async_copy=1`
- **EN:** Dominant operations include `tt.func`, `arith.constant`, `tt.broadcast`, `module`, `tt.splat`, `tt.make_range`, `tt.expand_dims`, `tt.addptr`, `scf.for`, `tt.load`.
- **CN:** 主要操作包括 `tt.func`、`arith.constant`、`tt.broadcast`、`module`、`tt.splat`、`tt.make_range`、`tt.expand_dims`、`tt.addptr`、`scf.for`、`tt.load`。
- **EN:** The file contains 1 independently testable section(s). Check styles used: CHECK x5, CHECK-NOT x3, CHECK-LABEL x1. Important labels include tt.func @simple_pipelined_load. It validates both the presence of the desired rewrite and the absence of unwanted residual operations.
- **CN:** 该文件包含 1 个可独立测试的分段。使用的检查类型：CHECK ×5，CHECK-NOT ×3，CHECK-LABEL ×1。 关键标签包括 tt.func @simple_pipelined_load。 它同时验证目标重写已经出现，并且不再残留不需要的操作。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。