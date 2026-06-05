# amd-schedule-hint.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `./test/TritonGPU/amd/amd-schedule-hint.mlir`
- **EN:** Regression test for the pipeline `-triton-amdgpu-insert-instruction-sched-hints="variant=attention", -triton-amdgpu-lower-insert-instruction-sched-hints, -verify-diagnostics`; diagnostics stay enabled while FileCheck still verifies the rewritten IR.
- **CN:** 这是针对流水线 `-triton-amdgpu-insert-instruction-sched-hints="variant=attention", -triton-amdgpu-lower-insert-instruction-sched-hints, -verify-diagnostics` 的回归测试；在启用 diagnostics 的同时，仍通过 FileCheck 验证重写后的 IR。
- **EN:** RUN pipeline(s): `// RUN: triton-opt %s -split-input-file -triton-amdgpu-insert-instruction-sched-hints="variant=attention" | FileCheck %s -check-prefix=INSTR_HINT`; `// RUN: triton-opt %s -split-input-file -triton-amdgpu-insert-instruction-sched-hints="variant=attention" -triton-amdgpu-lower-insert-instruction-sched-hints -verify-diagnostics | FileCheck %s -check-prefix=LOWER_HINT`
- **CN:** RUN 流水线：`// RUN: triton-opt %s -split-input-file -triton-amdgpu-insert-instruction-sched-hints="variant=attention" | FileCheck %s -check-prefix=INSTR_HINT`；`// RUN: triton-opt %s -split-input-file -triton-amdgpu-insert-instruction-sched-hints="variant=attention" -triton-amdgpu-lower-insert-instruction-sched-hints -verify-diagnostics | FileCheck %s -check-prefix=LOWER_HINT`

## Line-by-Line Analysis / 逐行分析

### Lines 1-3
```mlir
1| // RUN: triton-opt %s -split-input-file -triton-amdgpu-insert-instruction-sched-hints="variant=attention" | FileCheck %s -check-prefix=INSTR_HINT
2| // RUN: triton-opt %s -split-input-file -triton-amdgpu-insert-instruction-sched-hints="variant=attention" -triton-amdgpu-lower-insert-instruction-sched-hints -verify-diagnostics | FileCheck %s -check-prefix=LOWER_HINT
3| 
```
**EN:** This header defines how the test is executed. It runs `triton-opt %s -split-input-file -triton-amdgpu-insert-instruction-sched-hints="variant=attention" | FileCheck %s -check-prefix=INSTR_HINT` ; ` triton-opt %s -split-input-file -triton-amdgpu-insert-instruction-sched-hints="variant=attention" -triton-amdgpu-lower-insert-instruction-sched-hints -verify-diagnostics | FileCheck %s -check-prefix=LOWER_HINT` so the selected pass pipeline, verifier, and checking tool can consume the same source file.
**CN:** 这一段定义测试执行方式。它运行 `triton-opt %s -split-input-file -triton-amdgpu-insert-instruction-sched-hints="variant=attention" | FileCheck %s -check-prefix=INSTR_HINT`；` triton-opt %s -split-input-file -triton-amdgpu-insert-instruction-sched-hints="variant=attention" -triton-amdgpu-lower-insert-instruction-sched-hints -verify-diagnostics | FileCheck %s -check-prefix=LOWER_HINT`，让选定的 pass 流水线、verifier 与检查工具基于同一份源文件工作。

### Lines 4-9
```mlir
4| #blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 4], order = [1, 0]}>
5| #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 4], instrShape = [32, 32, 8], isTransposed = true}>
6| #dot_op_a = #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>
7| #dot_op_b = #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>
8| // INSTR_HINT-LABEL: @insert_schedule_hint
9| // LOWER_HINT-LABEL: @insert_schedule_hint
```
**EN:** This operational block carries the concrete IR steps for the test, centered on IR structure and attributes. Together these lines express the behavior that the pass pipeline must preserve or rewrite.
**CN:** 这一段操作代码给出了测试的具体 IR 步骤，核心是 IR 结构与属性。这些行共同表达了 pass 流水线必须保持或重写的行为。

### Lines 10-10
```mlir
10| module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This block establishes a module boundary and its execution attributes, such as warps, CTAs, or backend-specific settings. The module also pins the target to `hip:gfx942`.
**CN:** 这一段建立模块边界以及其执行属性，例如 warps、CTA 数量或后端相关设置。 该模块还把目标平台固定为 `hip:gfx942`。

### Lines 11-28
```mlir
11|   tt.func public @insert_schedule_hint(
12|     %lb : index, %ub : index, %step : index,
13|     %arg0: tensor<128x128xf32, #dot_op_a>,
14|     %arg1: tensor<128x128xf32, #dot_op_b>,
15|     %arg2: tensor<128x128x!tt.ptr<f32>, #blocked>
16|   ) {
17|     %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
18|     // INSTR_HINT: scf.for
19|     // INSTR_HINT-NEXT: amdg.instruction_sched_hint
20|     // INSTR_HINT-SAME: variant = #amdg.SchedHintVariant<attention>
21| 
22|     // LOWER_HINT: scf.for
23|     // LOWER_HINT-NEXT: rocdl.sched.barrier 0
24|     // LOWER_HINT-COUNT-2: tt.dot
25|     // LOWER_HINT: rocdl.iglp.opt 2
26|     // LOWER_HINT-NEXT: rocdl.sched.barrier 0
27|     // LOWER_HINT-NEXT: scf.yield
28|     %loop = scf.for %iv = %lb to %ub step %step iter_args(%c = %cst) -> (tensor<128x128xf32, #mma>) {
```
**EN:** This function-oriented block defines or enters `insert_schedule_hint`. Within it, the test exercises tt.func, constants, structured loops, which forms the main dataflow being optimized, lowered, or verified.
**CN:** 这一段以函数 `insert_schedule_hint` 为核心。测试在其中演示 tt.func、常量、结构化循环，这些操作构成了被优化、lower 或验证的主要数据流。

### Lines 29-39
```mlir
29|       %4 = tt.dot %arg0, %arg1, %c : tensor<128x128xf32, #dot_op_a> * tensor<128x128xf32, #dot_op_b> -> tensor<128x128xf32, #mma>
30|       %5 = math.exp2 %4 : tensor<128x128xf32, #mma>
31|       %6 = ttg.convert_layout %5 : tensor<128x128xf32, #mma> -> tensor<128x128xf32, #dot_op_a>
32|       %7 = tt.dot %6, %arg1, %c : tensor<128x128xf32, #dot_op_a> * tensor<128x128xf32, #dot_op_b> -> tensor<128x128xf32, #mma>
33|       scf.yield %7 : tensor<128x128xf32, #mma>
34|     }
35|     %8 = ttg.convert_layout %loop : tensor<128x128xf32, #mma> -> tensor<128x128xf32, #blocked>
36|     tt.store %arg2, %8 : tensor<128x128x!tt.ptr<f32>, #blocked>
37|     tt.return
38|   }
39| }
```
**EN:** This control-flow block uses structured loops or conditionals to model pipeline stages, predication, or repeated memory/computation steps while combining dot-product or MMA-style math, layout conversions, math.exp2, loop/if yielded values, masked or vectorized stores.
**CN:** 这一段控制流使用结构化循环或条件分支来表达流水线阶段、谓词执行或重复的内存/计算步骤，并结合了 点积或 MMA 风格计算、布局转换、math.exp2、循环/分支产出值、带掩码或向量化的存储。

## Key Concepts / 关键概念

- **EN:** Primary pass options: `-split-input-file`, `-triton-amdgpu-insert-instruction-sched-hints="variant=attention"`, `-triton-amdgpu-lower-insert-instruction-sched-hints`, `-verify-diagnostics`
- **CN:** 主要 pass 选项：`-split-input-file`，`-triton-amdgpu-insert-instruction-sched-hints="variant=attention"`，`-triton-amdgpu-lower-insert-instruction-sched-hints`，`-verify-diagnostics`
- **EN:** Dominant operations include `tt.func`, `tt.dot`, `ttg.convert_layout`, `module`, `arith.constant`, `scf.for`, `math.exp2`, `scf.yield`, `tt.store`, `tt.return`.
- **CN:** 主要操作包括 `tt.func`、`tt.dot`、`ttg.convert_layout`、`module`、`arith.constant`、`scf.for`、`math.exp2`、`scf.yield`、`tt.store`、`tt.return`。
- **EN:** The file contains 1 independently testable section(s). The file relies mainly on the transformed IR itself rather than explicit FileCheck/diagnostic annotations. It validates diagnostic behavior and parser/verifier stability under this input.
- **CN:** 该文件包含 1 个可独立测试的分段。该文件主要依赖变换后的 IR 本身，而不是显式的 FileCheck/诊断注解。 它验证在该输入下的诊断行为以及 parser/verifier 的稳定性。

## Dependencies / 依赖关系

- **EN:** This test depends on Triton/MLIR textual IR parsing, the pass pipeline named in the RUN lines, and the dialect/type definitions referenced in the file.
- **CN:** 该测试依赖 Triton/MLIR 文本 IR 解析、RUN 行中指定的 pass 流水线，以及文件中引用的方言/类型定义。
- **EN:** `tt`: Triton high-level tensor/pointer operations.
- **CN:** `tt`：Triton 高层张量/指针操作。
- **EN:** `ttg`: TritonGPU layout, memory, and GPU-specific operations.
- **CN:** `ttg`：TritonGPU 布局、内存和 GPU 专用操作。
- **EN:** `scf`: Structured control flow for loops and conditionals.
- **CN:** `scf`：用于循环与条件分支的结构化控制流。
- **EN:** `arith`: MLIR arithmetic dialect for scalar/tensor math.
- **CN:** `arith`：用于标量/张量运算的 MLIR arithmetic 方言。
- **EN:** `FileCheck` is used to match required textual patterns after rewriting/lowering.
- **CN:** `FileCheck` 用于在重写/lower 之后匹配必须出现的文本模式。
- **EN:** Diagnostic verification is part of the contract, so parser/verifier error strings are also dependencies of the test.
- **CN:** 诊断验证也是该测试契约的一部分，因此 parser/verifier 的报错文本同样是依赖项。