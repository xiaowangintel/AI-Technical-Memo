# tritongpu_to_llvm_sm120.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/tritongpu_to_llvm_sm120.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises LLVM lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的LLVM 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --tritongpu-accelerate-matmul --allocate-shared-memory-nv='compute-capability=120' --convert-triton-gpu-to-llvm='compute-capability=120' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×1, CHECK-LABEL×1; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×1, CHECK-LABEL×1；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that LLVM lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 LLVM 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --tritongpu-accelerate-matmul --allocate-shared-memory-nv='compute-capability=120' --convert-triton-gpu-to-llvm='compute-capability=120' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --tritongpu-accelerate-matmul --allocate-shared-memory-nv='compute-capability=120' --convert-triton-gpu-to-llvm='compute-capability=120' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --tritongpu-accelerate-matmul --allocate-shared-memory-nv='compute-capability=120' --convert-triton-gpu-to-llvm='compute-capability=120' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-5
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
#blocked_k = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [0, 1]}>
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#blocked_k`, `#mma`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#blocked_k`, `#mma`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.nvidia_mma`。

### Lines 7-30
```mlir
module attributes {"ttg.target" = "cuda:120", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @sm120_mmav2_dot_scaled
  // CHECK: mma.sync.aligned.m16n8k32.row.col.kind::mxf8f6f4.block_scale.scale_vec::1X
  tt.func public @sm120_mmav2_dot_scaled(
    %a: tensor<128x32xf8E5M2, #blocked_k>,
    %sa: tensor<128x1xi8, #blocked>,
    %b: tensor<32x128xf8E5M2, #blocked>,
    %sb: tensor<128x1xi8, #blocked>,
    %out: !tt.ptr<f32>
  ){
    %c = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %a_d = ttg.convert_layout %a : tensor<128x32xf8E5M2, #blocked_k> -> tensor<128x32xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
    %b_d = ttg.convert_layout %b : tensor<32x128xf8E5M2, #blocked> -> tensor<32x128xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
    %d = tt.dot_scaled %a_d scale %sa, %b_d scale %sb, %c lhs = e5m2 rhs = e5m2 {fastMath = false}
      : tensor<128x32xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, tensor<128x1xi8, #blocked>
        * tensor<32x128xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>, tensor<128x1xi8, #blocked>
        -> tensor<128x128xf32, #blocked>
    %out_splat = tt.splat %out : !tt.ptr<f32> -> tensor<128x1x!tt.ptr<f32>, #blocked>
    %out_ptrs = tt.broadcast %out_splat : tensor<128x1x!tt.ptr<f32>, #blocked> -> tensor<128x128x!tt.ptr<f32>, #blocked>
    %zero = arith.constant dense<0> : tensor<128x128xi1, #blocked>
    tt.store %out_ptrs, %d, %zero : tensor<128x128x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `sm120_mmav2_dot_scaled`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `sm120_mmav2_dot_scaled`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on LLVM lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 LLVM 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--tritongpu-accelerate-matmul`, `--allocate-shared-memory-nv='compute-capability=120'`, `--convert-triton-gpu-to-llvm='compute-capability=120'`, `--convert-nv-gpu-to-llvm`, `--mlir-to-llvmir`.  
  **CN:** `RUN` 流水线会驱动 `--tritongpu-accelerate-matmul`, `--allocate-shared-memory-nv='compute-capability=120'`, `--convert-triton-gpu-to-llvm='compute-capability=120'`, `--convert-nv-gpu-to-llvm`, `--mlir-to-llvmir` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `mlir-translate`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`mlir-translate`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`。
