# wmma-v1-shortcut.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/wmma-v1-shortcut.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises WMMA lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的WMMA 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx1100" -split-input-file | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×1, CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×2; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×1, CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×2；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that WMMA lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 WMMA 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx1100" -split-input-file | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx1100" -split-input-file | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx1100" -split-input-file | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-4
```mlir
#wmmaT = #ttg.amd_wmma<{version = 1, ctaLayout = {warp = []}, isTranspose = true}>
#dotop0 = #ttg.dot_op<{opIdx = 0, parent = #wmmaT, kWidth=16}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#wmmaT`, `#dotop0`. They parameterize later tests with compact names for `#ttg.amd_wmma`, `#ttg.dot_op`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#wmmaT`, `#dotop0`。它们为后续测试提供紧凑名称，用来表示 `#ttg.amd_wmma`, `#ttg.dot_op`。

### Lines 6-16
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: wmma_dot_cvt_bf16_wmma
  tt.func public @wmma_dot_cvt_bf16_wmma(%arg0: tensor<16x16xbf16, #wmmaT>) {
    // CHECK-NOT: store
    // CHECK-NOT: load
    // CHECK-COUNT-4: rocdl.permlanex16
    // CHECK: llvm.return
    %0 = ttg.convert_layout %arg0 : tensor<16x16xbf16, #wmmaT> -> tensor<16x16xbf16, #dotop0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_cvt_bf16_wmma`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_cvt_bf16_wmma`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on WMMA lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 WMMA 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--tritongpu-reduce-data-duplication`, `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch="gfx1100"`.  
  **CN:** `RUN` 流水线会驱动 `--tritongpu-reduce-data-duplication`, `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch="gfx1100"` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `llvm` (LLVM dialect IR).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `llvm`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`llvm`。
