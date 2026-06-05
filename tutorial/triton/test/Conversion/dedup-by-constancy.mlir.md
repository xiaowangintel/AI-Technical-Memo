# dedup-by-constancy.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/dedup-by-constancy.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises LLVM lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的LLVM 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --convert-triton-gpu-to-llvm --llvm-optimize-for-nvvm-target | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×6, CHECK-COUNT×6, CHECK-LABEL×2, CHECK-NOT×9; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×6, CHECK-COUNT×6, CHECK-LABEL×2, CHECK-NOT×9；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that LLVM lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 LLVM 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --convert-triton-gpu-to-llvm --llvm-optimize-for-nvvm-target | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --convert-triton-gpu-to-llvm --llvm-optimize-for-nvvm-target | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --convert-triton-gpu-to-llvm --llvm-optimize-for-nvvm-target | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-34
```mlir
// CHECK-LABEL: dedup_by_constancy_full
// CHECK-COUNT-2: llvm.add
// CHECK-NOT: llvm.add
// CHECK: llvm.icmp "slt"
// CHECK-NOT: llvm.icmp "slt"
// CHECK: llvm.sdiv
// CHECK-NOT: llvm.sdiv
// CHECK: llvm.getelementptr %arg0[[[REGISTER:%[0-9]+]]]
// CHECK-COUNT-7: llvm.getelementptr %arg0[[[REGISTER]]]
// CHECK-NOT: llvm.getelementptr %arg0[[[REGISTER]]]
#blocked = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @dedup_by_constancy_full(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}) {
    %cst = arith.constant dense<256> : tensor<1024xi32, #blocked>
    %c1024_i32 = arith.constant 1024 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c1024_i32 : i32
    %2 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
    %3 = tt.splat %1 : i32 -> tensor<1024xi32, #blocked>
    %4 = arith.addi %3, %2 : tensor<1024xi32, #blocked>
    %5 = tt.splat %arg2 : i32 -> tensor<1024xi32, #blocked>
    %6 = arith.cmpi slt, %4, %5 : tensor<1024xi32, #blocked>
    %7 = arith.divsi %4, %cst : tensor<1024xi32, #blocked>
    %8 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<1024x!tt.ptr<f16>, #blocked>
    %9 = tt.addptr %8, %7 : tensor<1024x!tt.ptr<f16>, #blocked>, tensor<1024xi32, #blocked>
    %10 = tt.load %9, %6 : tensor<1024x!tt.ptr<f16>, #blocked>
    %11 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<1024x!tt.ptr<f16>, #blocked>
    %12 = tt.addptr %11, %4 : tensor<1024x!tt.ptr<f16>, #blocked>, tensor<1024xi32, #blocked>
    tt.store %12, %10, %6 : tensor<1024x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `dedup_by_constancy_full`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `dedup_by_constancy_full`。

### Line 36
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 38-72
```mlir
// CHECK-LABEL: dedup_by_constancy_partial
// CHECK-COUNT-4: llvm.add
// CHECK-NOT: llvm.add
// CHECK: llvm.icmp "slt"
// CHECK-NOT: llvm.icmp "slt"
// CHECK-COUNT-2: llvm.sdiv
// CHECK-NOT: llvm.sdiv
// CHECK: llvm.getelementptr %arg0[[[REGISTER1:%[0-9]+]]]
// CHECK-COUNT-3: llvm.getelementptr %arg0[[[REGISTER1]]]
// CHECK-NOT: llvm.getelementptr %arg0[[[REGISTER1]]]
// CHECK: llvm.getelementptr %arg0[[[REGISTER2:%[0-9]+]]]
// CHECK-COUNT-3: llvm.getelementptr %arg0[[[REGISTER2]]]
// CHECK-NOT: llvm.getelementptr %arg0[[[REGISTER2]]]
#blocked = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @dedup_by_constancy_partial(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg2: i32 {tt.divisibility = 16 : i32, tt.max_divisibility = 8 : i32}) {
    %cst = arith.constant dense<4> : tensor<1024xi32, #blocked>
    %c1024_i32 = arith.constant 1024 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c1024_i32 : i32
    %2 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
    %3 = tt.splat %1 : i32 -> tensor<1024xi32, #blocked>
    %4 = arith.addi %3, %2 : tensor<1024xi32, #blocked>
    %5 = tt.splat %arg2 : i32 -> tensor<1024xi32, #blocked>
    %6 = arith.cmpi slt, %4, %5 : tensor<1024xi32, #blocked>
    %7 = arith.divsi %4, %cst : tensor<1024xi32, #blocked>
    %8 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<1024x!tt.ptr<f16>, #blocked>
    %9 = tt.addptr %8, %7 : tensor<1024x!tt.ptr<f16>, #blocked>, tensor<1024xi32, #blocked>
    %10 = tt.load %9, %6 : tensor<1024x!tt.ptr<f16>, #blocked>
    %11 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<1024x!tt.ptr<f16>, #blocked>
    %12 = tt.addptr %11, %4 : tensor<1024x!tt.ptr<f16>, #blocked>, tensor<1024xi32, #blocked>
    tt.store %12, %10, %6 : tensor<1024x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `dedup_by_constancy_partial`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `dedup_by_constancy_partial`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on LLVM lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 LLVM 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--convert-triton-gpu-to-llvm`, `--llvm-optimize-for-nvvm-target`.  
  **CN:** `RUN` 流水线会驱动 `--convert-triton-gpu-to-llvm`, `--llvm-optimize-for-nvvm-target` 等工具/pass。
- **EN:** The IR mixes `llvm` (LLVM dialect IR), `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `llvm`（LLVM 方言 IR）、`ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `llvm`, `ttg`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`llvm`、`ttg`、`tt`、`arith`。
