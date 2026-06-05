# accelerate-amd-matmul-mfma.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/TritonGPU/amd/accelerate-amd-matmul-mfma.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises MFMA lowering in Triton's TritonGPU transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton TritonGPU 变换中的MFMA 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `split-file %s %t`<br>`cat %t/common.mlir %t/mfma0.mlir > %t/run-mfma0.mlir`<br>`triton-opt %t/run-mfma0.mlir -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=0" --verify-diagnostics | FileCheck %t/run-mfma0.mlir --check-prefixes=MFMA0,CHECK`<br>`cat %t/common.mlir %t/mfma16.mlir > %t/run-mfma16.mlir`<br>`triton-opt %t/run-mfma16.mlir -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=16" --verify-diagnostics | FileCheck %t/run-mfma16.mlir --check-prefixes=MFMA16,CHECK` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×12, CHECK-LABEL×6; diagnostics annotations: remark×8. **CN:** FileCheck 标记为 CHECK×12, CHECK-LABEL×6；诊断标注为 remark×8。
- **Validation / 验证内容:** **EN:** The file mainly validates that MFMA lowering emits the expected analysis remarks/notes. **CN:** 该文件主要验证 MFMA 降级 会发出预期的分析备注/说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-5
```mlir
// RUN: split-file %s %t
// RUN: cat %t/common.mlir %t/mfma0.mlir > %t/run-mfma0.mlir
// RUN: triton-opt %t/run-mfma0.mlir -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=0" --verify-diagnostics | FileCheck %t/run-mfma0.mlir --check-prefixes=MFMA0,CHECK
// RUN: cat %t/common.mlir %t/mfma16.mlir > %t/run-mfma16.mlir
// RUN: triton-opt %t/run-mfma16.mlir -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=16" --verify-diagnostics | FileCheck %t/run-mfma16.mlir --check-prefixes=MFMA16,CHECK
```
**EN:** This block defines lit execution commands. The pipeline runs `split-file %s %t; cat %t/common.mlir %t/mfma0.mlir > %t/run-mfma0.mlir; triton-opt %t/run-mfma0.mlir -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=0" --verify-diagnostics | FileCheck %t/run-mfma0.mlir --check-prefixes=MFMA0,CHECK; cat %t/common.mlir %t/mfma16.mlir > %t/run-mfma16.mlir; triton-opt %t/run-mfma16.mlir -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=16" --verify-diagnostics | FileCheck %t/run-mfma16.mlir --check-prefixes=MFMA16,CHECK` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `split-file %s %t; cat %t/common.mlir %t/mfma0.mlir > %t/run-mfma0.mlir; triton-opt %t/run-mfma0.mlir -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=0" --verify-diagnostics | FileCheck %t/run-mfma0.mlir --check-prefixes=MFMA0,CHECK; cat %t/common.mlir %t/mfma16.mlir > %t/run-mfma16.mlir; triton-opt %t/run-mfma16.mlir -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=16" --verify-diagnostics | FileCheck %t/run-mfma16.mlir --check-prefixes=MFMA16,CHECK`，从而以预期的 pass 顺序和校验模式执行测试。

### Line 7
```mlir
//--- common.mlir
```
**EN:** This comment block provides context for the surrounding test logic: `//--- common.mlir`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`//--- common.mlir`。

### Lines 9-28
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 4], order = [1, 0]}>
// CHECK-LABEL: mfma_dot_fp8e5m2_fp8e4m3fn
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_dot_fp8e5m2_fp8e4m3fn(
      %arg0: tensor<128x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
      %arg1: tensor<64x256xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
      %arg2: tensor<128x256x!tt.ptr<f32>, #blocked>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked>
    // CHECK: %[[A0:.+]] = ttg.convert_layout %arg0 : {{.*}} -> tensor<128x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
    // CHECK: %[[A1:.+]] = tt.fp_to_fp %[[A0]] : {{.*}} -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
    // CHECK: %[[B0:.+]] = ttg.convert_layout %arg1 : {{.*}} -> tensor<64x256xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
    // CHECK: %[[B1:.+]] = tt.fp_to_fp %[[B0]] : tensor<64x256xf8E4M3FN, {{.*}} -> tensor<64x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
    // CHECK: tt.dot %[[A1]], %[[B1]]
    // expected-remark @+2 {{missing native support for fp8 variant on current architecture; emulated with fp16 so low performance}}
    // expected-remark @+1 {{for gfx942 please use native supported fp8 variants}}
    %1 = tt.dot %arg0, %arg1, %cst : tensor<128x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x256xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<128x256xf32, #blocked>
    tt.store %arg2, %1 : tensor<128x256x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_dot_fp8e5m2_fp8e4m3fn`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_dot_fp8e5m2_fp8e4m3fn`。

### Line 30
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 32-51
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 4], order = [1, 0]}>
// CHECK-LABEL: mfma_dot_fp8e4m3fn_fp8e5m2
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_dot_fp8e4m3fn_fp8e5m2(
      %arg0: tensor<128x64xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
      %arg1: tensor<64x256xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
      %arg2: tensor<128x256x!tt.ptr<f32>, #blocked>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked>
    // CHECK: %[[A0:.+]] = ttg.convert_layout %arg0 : {{.*}} -> tensor<128x64xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
    // CHECK: %[[A1:.+]] = tt.fp_to_fp %[[A0]] : {{.*}} -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
    // CHECK: %[[B0:.+]] = ttg.convert_layout %arg1 : {{.*}} -> tensor<64x256xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
    // CHECK: %[[B1:.+]] = tt.fp_to_fp %[[B0]] : tensor<64x256xf8E5M2, {{.*}} -> tensor<64x256xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
    // CHECK: tt.dot %[[A1]], %[[B1]]
    // expected-remark @+2 {{missing native support for fp8 variant on current architecture; emulated with fp16 so low performance}}
    // expected-remark @+1 {{for gfx942 please use native supported fp8 variants}}
    %1 = tt.dot %arg0, %arg1, %cst : tensor<128x64xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x256xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<128x256xf32, #blocked>
    tt.store %arg2, %1 : tensor<128x256x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_dot_fp8e4m3fn_fp8e5m2`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_dot_fp8e4m3fn_fp8e5m2`。

### Line 53
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 55-68
```mlir
// MFMA0: #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 2], instrShape = [4, 64, 64], isTransposed = false}>
// MFMA16: #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 2], instrShape = [16, 16, 16], isTransposed = true}>
// CHECK-LABEL: small_m_size_mfma
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [1, 64], warpsPerCTA = [1, 2], order = [1, 0]}>
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @small_m_size_mfma(
    %a: tensor<4x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
    %b: tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>)
    -> tensor<4x128xf32, #blocked> {
    %zero_f32 = arith.constant dense<0.000000e+00> : tensor<4x128xf32, #blocked>
    %result = tt.dot %a, %b, %zero_f32 : tensor<4x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<4x128xf32, #blocked>
    tt.return %result : tensor<4x128xf32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `small_m_size_mfma`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `small_m_size_mfma`。

### Line 70
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 72-88
```mlir
// MFMA0-NOT: amd_mfma
// MFMA16-NOT: amd_mfma
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 4], order = [1, 0]}>
// CHECK-LABEL: mfma_dot_small_k
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_dot_small_k(
      %arg0: tensor<128x4xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
      %arg1: tensor<4x256xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
      %arg2: tensor<128x256x!tt.ptr<f32>, #blocked> ) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked>
    // expected-remark @+2 {{Unable to select MFMA intrinsic}}
    // expected-remark @+1 {{Attempting to map dot operation to FMA intrinsic.}}
    %1 = tt.dot %arg0, %arg1, %cst : tensor<128x4xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<4x256xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<128x256xf32, #blocked>
    tt.store %arg2, %1 : tensor<128x256x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_dot_small_k`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_dot_small_k`。

### Line 90
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 92
```mlir
//--- mfma0.mlir
```
**EN:** This comment block provides context for the surrounding test logic: `//--- mfma0.mlir`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`//--- mfma0.mlir`。

### Lines 94-109
```mlir
// MFMA0-NOT: amd_mfma
// MFMA16: #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 2], instrShape = [16, 16, 16], isTransposed = true}>
// CHECK-LABEL: small_m_size_fma
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [1, 64], warpsPerCTA = [1, 2], order = [1, 0]}>
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @small_m_size_fma(
    %a: tensor<1x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
    %b: tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>)
    -> tensor<1x128xf32, #blocked> {
    %zero_f32 = arith.constant dense<0.000000e+00> : tensor<1x128xf32, #blocked>
    // expected-remark @+2 {{Unable to select MFMA intrinsic}}
    // expected-remark @+1 {{Attempting to map dot operation to FMA intrinsic.}}
    %result = tt.dot %a, %b, %zero_f32 : tensor<1x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<1x128xf32, #blocked>
    tt.return %result : tensor<1x128xf32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `small_m_size_fma`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `small_m_size_fma`。

### Line 111
```mlir
//--- mfma16.mlir
```
**EN:** This comment block provides context for the surrounding test logic: `//--- mfma16.mlir`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`//--- mfma16.mlir`。

### Lines 113-126
```mlir
// MFMA0-NOT: amd_mfma
// MFMA16: #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 2], instrShape = [16, 16, 16], isTransposed = true}>
// CHECK-LABEL: small_m_size_fma
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [1, 64], warpsPerCTA = [1, 2], order = [1, 0]}>
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @small_m_size_fma(
    %a: tensor<1x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
    %b: tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>)
    -> tensor<1x128xf32, #blocked> {
    %zero_f32 = arith.constant dense<0.000000e+00> : tensor<1x128xf32, #blocked>
    %result = tt.dot %a, %b, %zero_f32 : tensor<1x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<1x128xf32, #blocked>
    tt.return %result : tensor<1x128xf32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `small_m_size_fma`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `small_m_size_fma`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's TritonGPU transforms coverage and focuses on MFMA lowering.  
  **CN:** 该文件属于 Triton 的 TritonGPU 变换 测试覆盖，关注点是 MFMA 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx942`, `--verify-diagnostics`, `--check-prefixes=MFMA0,CHECK`, `--check-prefixes=MFMA16,CHECK`.  
  **CN:** `RUN` 流水线会驱动 `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx942`, `--verify-diagnostics`, `--check-prefixes=MFMA0,CHECK`, `--check-prefixes=MFMA16,CHECK` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。
- **EN:** Expected remarks/notes are used to pin down the exact analysis output produced by the pass.  
  **CN:** 这里的期望备注/说明用于固定 pass 产生的精确分析输出。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`。
