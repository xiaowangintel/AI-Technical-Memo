# mfma-shortcut.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/mfma-shortcut.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises MFMA lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的MFMA 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx942" -split-input-file | FileCheck %s --check-prefix=GFX942`<br>`triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx950" -split-input-file | FileCheck %s --check-prefix=GFX950` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Validation / 验证内容:** **EN:** The file validates that MFMA lowering runs successfully under the specified pass pipeline. **CN:** 该文件验证 MFMA 降级 能在指定的 pass 流水线下成功运行。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx942" -split-input-file | FileCheck %s --check-prefix=GFX942
// RUN: triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx950" -split-input-file | FileCheck %s --check-prefix=GFX950
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx942" -split-input-file | FileCheck %s --check-prefix=GFX942; triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx950" -split-input-file | FileCheck %s --check-prefix=GFX950` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx942" -split-input-file | FileCheck %s --check-prefix=GFX942; triton-opt %s --tritongpu-reduce-data-duplication --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx950" -split-input-file | FileCheck %s --check-prefix=GFX950`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-15
```mlir
#mfma = #ttg.amd_mfma<{version = 2, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true}>
#dotop = #ttg.dot_op<{opIdx = 0, parent = #mfma, kWidth=4}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // GFX942-LABEL: shortcut_mfma16
  tt.func public @shortcut_mfma16(%arg0: tensor<16x16xf16, #mfma>) {
    // GFX942-NOT: store
    // GFX942-NOT: load
    // GFX942: llvm.return
    %0 = ttg.convert_layout %arg0 : tensor<16x16xf16, #mfma> -> tensor<16x16xf16, #dotop>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `shortcut_mfma16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `shortcut_mfma16`。

### Line 17
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 19-20
```mlir
#mfma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>
#dotop0 = #ttg.dot_op<{opIdx = 0, parent = #mfma, kWidth=8}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mfma`, `#dotop0`. They parameterize later tests with compact names for `#ttg.amd_mfma`, `#ttg.dot_op`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mfma`, `#dotop0`。它们为后续测试提供紧凑名称，用来表示 `#ttg.amd_mfma`, `#ttg.dot_op`。

### Lines 22-32
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // GFX942-LABEL: mfma_dot_cvt_bf8_mfma32_v3
  tt.func public @mfma_dot_cvt_bf8_mfma32_v3(%arg0: tensor<128x32xf8E5M2, #mfma>) {
    // GFX942-NOT: store
    // GFX942-NOT: load
    // GFX942: rocdl.ds_bpermute
    // GFX942: llvm.return
    %0 = ttg.convert_layout %arg0 : tensor<128x32xf8E5M2, #mfma> -> tensor<128x32xf8E5M2, #dotop0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_dot_cvt_bf8_mfma32_v3`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_dot_cvt_bf8_mfma32_v3`。

### Line 34
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 36-37
```mlir
#mfma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
#dotop0 = #ttg.dot_op<{opIdx = 0, parent = #mfma, kWidth=8}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mfma`, `#dotop0`. They parameterize later tests with compact names for `#ttg.amd_mfma`, `#ttg.dot_op`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mfma`, `#dotop0`。它们为后续测试提供紧凑名称，用来表示 `#ttg.amd_mfma`, `#ttg.dot_op`。

### Lines 39-47
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // GFX950-LABEL: mfma_dot_cvt_bf8_mfma32_v4
  tt.func public @mfma_dot_cvt_bf8_mfma32_v4(%arg0: tensor<128x32xf8E5M2, #mfma>) {
    // GFX950-NOT: rocdl.ds_bpermute
    // GFX950-COUNT-2: llvm.call_intrinsic "llvm.amdgcn.permlane32.swap"
    %0 = ttg.convert_layout %arg0 : tensor<128x32xf8E5M2, #mfma> -> tensor<128x32xf8E5M2, #dotop0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_dot_cvt_bf8_mfma32_v4`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_dot_cvt_bf8_mfma32_v4`。

### Line 49
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 51-52
```mlir
#mfma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true}>
#dotop0 = #ttg.dot_op<{opIdx = 0, parent = #mfma, kWidth=8}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mfma`, `#dotop0`. They parameterize later tests with compact names for `#ttg.amd_mfma`, `#ttg.dot_op`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mfma`, `#dotop0`。它们为后续测试提供紧凑名称，用来表示 `#ttg.amd_mfma`, `#ttg.dot_op`。

### Lines 54-64
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // GFX942-LABEL: mfma_dot_cvt_bf8_mfma16_v3
  tt.func public @mfma_dot_cvt_bf8_mfma16_v3(%arg0: tensor<128x32xf8E5M2, #mfma>) {
    // GFX942-NOT: store
    // GFX942-NOT: load
    // GFX942: rocdl.ds_bpermute
    // GFX942: llvm.return
    %0 = ttg.convert_layout %arg0 : tensor<128x32xf8E5M2, #mfma> -> tensor<128x32xf8E5M2, #dotop0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_dot_cvt_bf8_mfma16_v3`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_dot_cvt_bf8_mfma16_v3`。

### Line 66
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 68-69
```mlir
#mfma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [16, 16, 32], isTransposed = true}>
#dotop0 = #ttg.dot_op<{opIdx = 0, parent = #mfma, kWidth=8}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mfma`, `#dotop0`. They parameterize later tests with compact names for `#ttg.amd_mfma`, `#ttg.dot_op`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mfma`, `#dotop0`。它们为后续测试提供紧凑名称，用来表示 `#ttg.amd_mfma`, `#ttg.dot_op`。

### Lines 71-82
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // GFX950-LABEL: mfma_dot_cvt_bf8_mfma16_v4
  tt.func public @mfma_dot_cvt_bf8_mfma16_v4(%arg0: tensor<128x32xf8E5M2, #mfma>) {
    // GFX950-NOT: rocdl.ds_bpermute
    // GFX950: llvm.call_intrinsic "llvm.amdgcn.permlane32.swap"
    // GFX950: llvm.call_intrinsic "llvm.amdgcn.permlane16.swap"
    // GFX950: llvm.call_intrinsic "llvm.amdgcn.permlane32.swap"
    // GFX950: llvm.call_intrinsic "llvm.amdgcn.permlane16.swap"
    %0 = ttg.convert_layout %arg0 : tensor<128x32xf8E5M2, #mfma> -> tensor<128x32xf8E5M2, #dotop0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_dot_cvt_bf8_mfma16_v4`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_dot_cvt_bf8_mfma16_v4`。

### Line 84
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 86-96
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 4], warpsPerCTA = [4, 1], order = [0, 1]}>
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 16], [0, 32], [0, 64]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 8]], warp = [[32, 0], [64, 0]], block = []}>
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
module attributes {"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // GFX950-LABEL: mfma_linear_permlane_swap
  tt.func public @mfma_linear_permlane_swap(%arg0: tensor<128x128xf16, #mma>) {
  // GFX950-COUNT-16: llvm.call_intrinsic "llvm.amdgcn.permlane32.swap"
    %1 = ttg.convert_layout %arg0: tensor<128x128xf16, #mma> -> tensor<128x128xf16, #linear>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_linear_permlane_swap`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_linear_permlane_swap`。

### Line 98
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 100-110
```mlir
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
#mma1 = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], tilesPerWarp = [2, 1], instrShape = [16, 16, 32], isTransposed = true}>
module attributes {"ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // GFX950-LABEL: mfma_dotop_permlane_swap
  tt.func public @mfma_dotop_permlane_swap(%arg0: tensor<128x16xf16, #mma1>) {
  // GFX950-NOT: load
  // GFX950-COUNT-2: llvm.call_intrinsic "llvm.amdgcn.permlane16.swap"
    %1 = ttg.convert_layout %arg0: tensor<128x16xf16, #mma1> -> tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_dotop_permlane_swap`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_dotop_permlane_swap`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on MFMA lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 MFMA 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--tritongpu-reduce-data-duplication`, `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch="gfx942"`, `--check-prefix=GFX942`, `--convert-triton-amdgpu-to-llvm=gfx-arch="gfx950"`, `--check-prefix=GFX950`.  
  **CN:** `RUN` 流水线会驱动 `--tritongpu-reduce-data-duplication`, `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch="gfx942"`, `--check-prefix=GFX942`, `--convert-triton-amdgpu-to-llvm=gfx-arch="gfx950"`, `--check-prefix=GFX950` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `llvm` (LLVM dialect IR).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `llvm`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`llvm`。
