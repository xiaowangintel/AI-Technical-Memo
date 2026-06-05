# dedup-by-constancy.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/dedup-by-constancy.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises AMD-specific lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的AMD 专用降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that AMD-specific lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 AMD 专用降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-25
```mlir
// CHECK-LABEL: dedup_by_constancy_mfma
// CHECK-COUNT-2: llvm.icmp "slt"
// CHECK-NOT: llvm.icmp "slt"
// For a 32x32 tensor A with mfma layout, each thread holds 16 elements, which are divided
// into 4 groups. E.g. thread 0 holds elements A[0:3,0], A[8:11,0], A[16:19,0], and A[24:27,0].
// In this example, constancy of the tensor is 16 for dim 0, meaning A[0:15,0] have same values
// and A[16:31,0] have same values. Therefore, for thread 0, the first 8 elements are duplicated
// and the last 8 elements are duplicated.
#mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 1], instrShape = [32, 32, 8], isTransposed = false}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @dedup_by_constancy_mfma(%arg0: i32 {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}) {
    %0 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #mma}>>
    %1 = tt.splat %arg0 : i32 -> tensor<32xi32, #ttg.slice<{dim = 1, parent = #mma}>>
    %2 = arith.cmpi slt, %0, %1 : tensor<32xi32, #ttg.slice<{dim = 1, parent = #mma}>>
    %3 = tt.expand_dims %2 {axis = 1 : i32} : tensor<32xi1, #ttg.slice<{dim = 1, parent = #mma}>> -> tensor<32x1xi1, #mma>
    %4 = tt.broadcast %3 : tensor<32x1xi1, #mma> -> tensor<32x32xi1, #mma>
    %cst = arith.constant dense<0.100000e+00> : tensor<32x32xf16, #mma>
    %5 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<32x1x!tt.ptr<f16>, #mma>
    %6 = tt.broadcast %5 : tensor<32x1x!tt.ptr<f16>, #mma> -> tensor<32x32x!tt.ptr<f16>, #mma>
    tt.store %6, %cst, %4 : tensor<32x32x!tt.ptr<f16>, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `dedup_by_constancy_mfma`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `dedup_by_constancy_mfma`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on AMD-specific lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 AMD 专用降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--convert-builtin-func-to-llvm`.  
  **CN:** `RUN` 流水线会驱动 `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--convert-builtin-func-to-llvm` 等工具/pass。
- **EN:** The IR mixes `llvm` (LLVM dialect IR), `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `llvm`（LLVM 方言 IR）、`ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `llvm`, `ttg`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`llvm`、`ttg`、`tt`、`arith`。
