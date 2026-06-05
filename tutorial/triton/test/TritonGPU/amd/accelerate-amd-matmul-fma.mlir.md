# accelerate-amd-matmul-fma.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/TritonGPU/amd/accelerate-amd-matmul-fma.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises matmul acceleration in Triton's TritonGPU transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton TritonGPU 变换中的矩阵乘加速相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942" | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×15, CHECK-DAG×2; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×15, CHECK-DAG×2；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that matmul acceleration produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 矩阵乘加速 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942" | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942" | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942" | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-16
```mlir
// CHECK: fma_dot_fp16_fp16
// CHECK: %[[D:.*]] = tt.dot {{.*}} : tensor<2x64xf16, {{.*}}> * tensor<64x64xf16, {{.*}}> -> tensor<2x64xf16, {{.*}}>
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @fma_dot_fp16_fp16(
      %arg0: tensor<2x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
      %arg1: tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
      %arg2: tensor<2x64x!tt.ptr<f16>, #blocked> ) {
    %cst = arith.constant dense<0.0> : tensor<2x64xf16, #blocked>
    %1 = tt.dot %arg0, %arg1, %cst : tensor<2x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<2x64xf16, #blocked>
    tt.store %arg2, %1 : tensor<2x64x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fma_dot_fp16_fp16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fma_dot_fp16_fp16`。

### Line 18
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 20-33
```mlir
// CHECK: fma_dot_fp32_fp32
// CHECK: tt.dot {{.*}} : tensor<2x64xf32, {{.*}}> * tensor<64x64xf32, {{.*}}> -> tensor<2x64xf32, {{.*}}>
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @fma_dot_fp32_fp32(
      %arg0: tensor<2x64xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
      %arg1: tensor<64x64xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
      %arg2: tensor<2x64x!tt.ptr<f32>, #blocked> ) {
    %cst = arith.constant dense<0.0> : tensor<2x64xf32, #blocked>
    %1 = tt.dot %arg0, %arg1, %cst : tensor<2x64xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x64xf32, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<2x64xf32, #blocked>
    tt.store %arg2, %1 : tensor<2x64x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fma_dot_fp32_fp32`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fma_dot_fp32_fp32`。

### Line 35
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 37-51
```mlir
// CHECK: #[[BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
// CHECK: fma_dot_i8
// CHECK: tt.dot {{.*}} : tensor<2x64xi8, #ttg.dot_op<{opIdx = 0, parent = #[[BLOCKED]]}>> * tensor<64x64xi8, #ttg.dot_op<{opIdx = 1, parent = #[[BLOCKED]]}>> -> tensor<2x64xi32, #[[BLOCKED]]>
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @fma_dot_i8(
      %arg0: tensor<2x64xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
      %arg1: tensor<64x64xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
      %arg2: tensor<2x64x!tt.ptr<i32>, #blocked> ) {
    %cst = arith.constant dense<0> : tensor<2x64xi32, #blocked>
    %1 = tt.dot %arg0, %arg1, %cst : tensor<2x64xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x64xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<2x64xi32, #blocked>
    tt.store %arg2, %1 : tensor<2x64x!tt.ptr<i32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fma_dot_i8`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fma_dot_i8`。

### Line 53
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 55-69
```mlir
// CHECK: #[[BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
// CHECK: fma_dot_f16
// CHECK: tt.dot {{.*}} : tensor<2x64xf16, #ttg.dot_op<{opIdx = 0, parent = #[[BLOCKED]]}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #[[BLOCKED]]}>> -> tensor<2x64xf32, #[[BLOCKED]]>
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @fma_dot_f16(
      %arg0: tensor<2x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
      %arg1: tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
      %arg2: tensor<2x64x!tt.ptr<f32>, #blocked> ) {
    %cst = arith.constant dense<0.0> : tensor<2x64xf32, #blocked>
    %1 = tt.dot %arg0, %arg1, %cst : tensor<2x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<2x64xf32, #blocked>
    tt.store %arg2, %1 : tensor<2x64x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fma_dot_f16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fma_dot_f16`。

### Line 71
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 73-87
```mlir
// CHECK: #[[BLOCKED:.*]] = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
// CHECK: fma_dot_f8
// CHECK: tt.dot {{.*}} : tensor<2x64xf32, #ttg.dot_op<{opIdx = 0, parent = #[[BLOCKED]]}>> * tensor<64x64xf32, #ttg.dot_op<{opIdx = 1, parent = #[[BLOCKED]]}>> -> tensor<2x64xf32, #[[BLOCKED]]>
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @fma_dot_f8(
      %arg0: tensor<2x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
      %arg1: tensor<64x64xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
      %arg2: tensor<2x64x!tt.ptr<f32>, #blocked> ) {
    %cst = arith.constant dense<0.0> : tensor<2x64xf32, #blocked>
    %1 = tt.dot %arg0, %arg1, %cst : tensor<2x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x64xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<2x64xf32, #blocked>
    tt.store %arg2, %1 : tensor<2x64x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fma_dot_f8`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fma_dot_f8`。

### Line 89
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 91-106
```mlir
// CHECK: fma_dot_i8_i8
// CHECK-DAG: %[[A:.*]] = arith.sitofp
// CHECK-DAG: %[[B:.*]] = arith.sitofp
// CHECK: %[[D:.*]] = tt.dot %[[A]], %[[B]], {{.*}} : tensor<2x64xf16, {{.*}}> * tensor<64x64xf16, {{.*}}> -> tensor<2x64xf16, {{.*}}>
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @fma_dot_i8_i8(
      %arg0: tensor<2x64xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
      %arg1: tensor<64x64xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
      %arg2: tensor<2x64x!tt.ptr<i8>, #blocked> ) {
    %cst = arith.constant dense<0> : tensor<2x64xi8, #blocked>
    %1 = tt.dot %arg0, %arg1, %cst : tensor<2x64xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x64xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<2x64xi8, #blocked>
    tt.store %arg2, %1 : tensor<2x64x!tt.ptr<i8>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fma_dot_i8_i8`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fma_dot_i8_i8`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's TritonGPU transforms coverage and focuses on matmul acceleration.  
  **CN:** 该文件属于 Triton 的 TritonGPU 变换 测试覆盖，关注点是 矩阵乘加速。
- **EN:** The `RUN` pipeline drives tools/passes such as `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx942"`.  
  **CN:** `RUN` 流水线会驱动 `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx942"` 等工具/pass。
- **EN:** The IR mixes `tt` (Triton core ops), `ttg` (TritonGPU ops/layouts), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `tt`（Triton 核心操作）、`ttg`（TritonGPU 操作/布局）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `tt`, `ttg`, `arith`.  
  **CN:** IR 方言依赖：`tt`、`ttg`、`arith`。
