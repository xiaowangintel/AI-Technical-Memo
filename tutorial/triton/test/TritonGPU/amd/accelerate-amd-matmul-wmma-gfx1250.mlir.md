# accelerate-amd-matmul-wmma-gfx1250.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/TritonGPU/amd/accelerate-amd-matmul-wmma-gfx1250.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises WMMA lowering in Triton's TritonGPU transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton TritonGPU 变换中的WMMA 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx1250" | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×106, CHECK-DAG×7, CHECK-LABEL×13, CHECK-NOT×13; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×106, CHECK-DAG×7, CHECK-LABEL×13, CHECK-NOT×13；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that WMMA lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 WMMA 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx1250" | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx1250" | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx1250" | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-34
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
// CHECK{LITERAL}: #linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
// CHECK{LITERAL}: #linear1 = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
// CHECK{LITERAL}: #mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 128]}>
// CHECK{LITERAL}: #mma1 = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 64]}>
// CHECK-LABEL: wmma_dot_scaled_mxfp4_mxfp4
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_scaled_mxfp4_mxfp4(
      %arg0: tensor<32x64xi8, #blocked>,
      %arg1: tensor<64x32xi8, #blocked1>,
      %arg2: tensor<32x4xi8, #blocked2>,
      %arg3: tensor<32x4xi8, #blocked2>,
      %arg4: tensor<32x32x!tt.ptr<f32>, #blocked3>
      ) {
    // CHECK-NOT: arith.constant dense<127> : tensor<32x4xi8, #linear>
    // CHECK-NOT: arith.constant dense<127> : tensor<32x4xi8, #linear1>
    // CHECK-NOT: tt.fp_to_fp
    // CHECK: %[[C:.+]] = ttg.convert_layout {{.*}} : tensor<32x32xf32, #blocked3> -> tensor<32x32xf32, #mma>
    // CHECK: %[[A:.+]] = ttg.convert_layout {{.*}} : tensor<32x64xi8, #blocked> -> tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 16}>>
    // CHECK: %[[B:.+]] = ttg.convert_layout {{.*}} : tensor<64x32xi8, #blocked1> -> tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 16}>>
    // CHECK: %[[SCALE0:.+]] = ttg.convert_layout {{.*}} : tensor<32x4xi8, #blocked2> -> tensor<32x4xi8, #linear>
    // CHECK: %[[SCALE1:.+]] = ttg.convert_layout {{.*}} : tensor<32x4xi8, #blocked2> -> tensor<32x4xi8, #linear1>
    // CHECK: tt.dot_scaled %[[A]] scale %[[SCALE0]], %[[B]] scale %[[SCALE1]], %[[C]] lhs = e2m1 rhs = e2m1
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked3>
    %1 = tt.dot_scaled %arg0 scale %arg2, %arg1 scale %arg3, %cst lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<32x64xi8, #blocked>, tensor<32x4xi8, #blocked2> * tensor<64x32xi8, #blocked1>, tensor<32x4xi8, #blocked2> -> tensor<32x32xf32, #blocked3>
    tt.store %arg4, %1 : tensor<32x32x!tt.ptr<f32>, #blocked3>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_scaled_mxfp4_mxfp4`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_scaled_mxfp4_mxfp4`。

### Line 36
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 38-67
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
// CHECK{LITERAL}: #linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
// CHECK{LITERAL}: #linear1 = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
// CHECK{LITERAL}: #mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 128]}>
// CHECK{LITERAL}: #mma1 = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 64]}>
// CHECK-LABEL: wmma_dot_scaled_mxfp4_mxfp8
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_scaled_mxfp4_mxfp8(
      %arg0: tensor<32x64xi8, #blocked>,
      %arg1: tensor<128x32xf8E4M3FN, #blocked1>,
      %arg2: tensor<32x4xi8, #blocked2>,
      %arg3: tensor<32x4xi8, #blocked2>,
      %arg4: tensor<32x32x!tt.ptr<f32>, #blocked3>
      ) {
    // CHECK-NOT: tt.fp_to_fp
    // CHECK: %[[C:.+]] = ttg.convert_layout {{.*}} : tensor<32x32xf32, #blocked3> -> tensor<32x32xf32, #mma>
    // CHECK: %[[A:.+]] = ttg.convert_layout {{.*}} : tensor<32x64xi8, #blocked> -> tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 16}>>
    // CHECK: %[[B:.+]] = ttg.convert_layout {{.*}} : tensor<128x32xf8E4M3FN, #blocked1> -> tensor<128x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    // CHECK: %[[SCALE0:.+]] = ttg.convert_layout {{.*}} : tensor<32x4xi8, #blocked2> -> tensor<32x4xi8, #linear>
    // CHECK: %[[SCALE1:.+]] = ttg.convert_layout {{.*}} : tensor<32x4xi8, #blocked2> -> tensor<32x4xi8, #linear1>
    // CHECK: tt.dot_scaled %[[A]] scale %[[SCALE0]], %[[B]] scale %[[SCALE1]], %[[C]] lhs = e2m1 rhs = e4m3
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked3>
    %1 = tt.dot_scaled %arg0 scale %arg2, %arg1 scale %arg3, %cst lhs = e2m1 rhs = e4m3 {fastMath = false} : tensor<32x64xi8, #blocked>, tensor<32x4xi8, #blocked2> * tensor<128x32xf8E4M3FN, #blocked1>, tensor<32x4xi8, #blocked2> -> tensor<32x32xf32, #blocked3>
    tt.store %arg4, %1 : tensor<32x32x!tt.ptr<f32>, #blocked3>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_scaled_mxfp4_mxfp8`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_scaled_mxfp4_mxfp8`。

### Line 69
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 71-99
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
// CHECK{LITERAL}: #linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
// CHECK{LITERAL}: #linear1 = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
// CHECK{LITERAL}: #mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 128]}>
// CHECK-LABEL: wmma_dot_scaled_mxfp8
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_scaled_mxfp8(
      %arg0: tensor<32x128xf8E4M3FN, #blocked>,
      %arg1: tensor<128x32xf8E4M3FN, #blocked1>,
      %arg2: tensor<32x4xi8, #blocked2>,
      %arg3: tensor<32x4xi8, #blocked2>,
      %arg4: tensor<32x32x!tt.ptr<f32>, #blocked3>
      ) {
    // CHECK-NOT: tt.fp_to_fp
    // CHECK: %[[C:.+]] = ttg.convert_layout {{.*}} : tensor<32x32xf32, #blocked3> -> tensor<32x32xf32, #mma>
    // CHECK: %[[A:.+]] = ttg.convert_layout {{.*}} : tensor<32x128xf8E4M3FN, #blocked> -> tensor<32x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
    // CHECK: %[[B:.+]] = ttg.convert_layout {{.*}} : tensor<128x32xf8E4M3FN, #blocked1> -> tensor<128x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    // CHECK: %[[SCALE0:.+]] = ttg.convert_layout {{.*}} : tensor<32x4xi8, #blocked2> -> tensor<32x4xi8, #linear>
    // CHECK: %[[SCALE1:.+]] = ttg.convert_layout {{.*}} : tensor<32x4xi8, #blocked2> -> tensor<32x4xi8, #linear1>
    // CHECK: tt.dot_scaled %[[A]] scale %[[SCALE0]], %[[B]] scale %[[SCALE1]], %[[C]] lhs = e4m3 rhs = e4m3
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked3>
    %1 = tt.dot_scaled %arg0 scale %arg2, %arg1 scale %arg3, %cst lhs = e4m3 rhs = e4m3 {fastMath = false} : tensor<32x128xf8E4M3FN, #blocked>, tensor<32x4xi8, #blocked2> * tensor<128x32xf8E4M3FN, #blocked1>, tensor<32x4xi8, #blocked2> -> tensor<32x32xf32, #blocked3>
    tt.store %arg4, %1 : tensor<32x32x!tt.ptr<f32>, #blocked3>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_scaled_mxfp8`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_scaled_mxfp8`。

### Line 101
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 103-131
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
// CHECK{LITERAL}: #linear = #ttg.linear<{register = [[0, 1], [0, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
// CHECK{LITERAL}: #linear1 = #ttg.linear<{register = [[0, 1], [0, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
// CHECK{LITERAL}: #mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 128]}>
// CHECK-LABEL: wmma_dot_scaled_mxfp8_k64
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_scaled_mxfp8_k64(
      %arg0: tensor<32x64xf8E4M3FN, #blocked>,
      %arg1: tensor<64x32xf8E4M3FN, #blocked1>,
      %arg2: tensor<32x2xi8, #blocked2>,
      %arg3: tensor<32x2xi8, #blocked2>,
      %arg4: tensor<32x32x!tt.ptr<f32>, #blocked3>
      ) {
    // CHECK-NOT: tt.fp_to_fp
    // CHECK: %[[C:.+]] = ttg.convert_layout {{.*}} : tensor<32x32xf32, #blocked3> -> tensor<32x32xf32, #mma>
    // CHECK: %[[A:.+]] = ttg.convert_layout {{.*}} : tensor<32x64xf8E4M3FN, #blocked> -> tensor<32x64xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
    // CHECK: %[[B:.+]] = ttg.convert_layout {{.*}} : tensor<64x32xf8E4M3FN, #blocked1> -> tensor<64x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    // CHECK: %[[SCALE0:.+]] = ttg.convert_layout {{.*}} : tensor<32x2xi8, #blocked2> -> tensor<32x2xi8, #linear>
    // CHECK: %[[SCALE1:.+]] = ttg.convert_layout {{.*}} : tensor<32x2xi8, #blocked2> -> tensor<32x2xi8, #linear1>
    // CHECK: tt.dot_scaled %[[A]] scale %[[SCALE0]], %[[B]] scale %[[SCALE1]], %[[C]] lhs = e4m3 rhs = e4m3
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked3>
    %1 = tt.dot_scaled %arg0 scale %arg2, %arg1 scale %arg3, %cst lhs = e4m3 rhs = e4m3 {fastMath = false} : tensor<32x64xf8E4M3FN, #blocked>, tensor<32x2xi8, #blocked2> * tensor<64x32xf8E4M3FN, #blocked1>, tensor<32x2xi8, #blocked2> -> tensor<32x32xf32, #blocked3>
    tt.store %arg4, %1 : tensor<32x32x!tt.ptr<f32>, #blocked3>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_scaled_mxfp8_k64`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_scaled_mxfp8_k64`。

### Line 133
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 135-163
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
// CHECK{LITERAL}: #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
// CHECK{LITERAL}: #linear1 = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
// CHECK{LITERAL}: #mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 128]}>
// CHECK-LABEL: wmma_dot_scaled_mxfp8_repeat_k
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_scaled_mxfp8_repeat_k(
      %arg0: tensor<32x256xf8E4M3FN, #blocked>,
      %arg1: tensor<256x32xf8E4M3FN, #blocked1>,
      %arg2: tensor<32x8xi8, #blocked2>,
      %arg3: tensor<32x8xi8, #blocked2>,
      %arg4: tensor<32x32x!tt.ptr<f32>, #blocked3>
      ) {
    // CHECK-NOT: tt.fp_to_fp
    // CHECK: %[[C:.+]] = ttg.convert_layout {{.*}} : tensor<32x32xf32, #blocked3> -> tensor<32x32xf32, #mma>
    // CHECK: %[[A:.+]] = ttg.convert_layout {{.*}} : tensor<32x256xf8E4M3FN, #blocked> -> tensor<32x256xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
    // CHECK: %[[B:.+]] = ttg.convert_layout {{.*}} : tensor<256x32xf8E4M3FN, #blocked1> -> tensor<256x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    // CHECK: %[[SCALE0:.+]] = ttg.convert_layout {{.*}} : tensor<32x8xi8, #blocked2> -> tensor<32x8xi8, #linear>
    // CHECK: %[[SCALE1:.+]] = ttg.convert_layout {{.*}} : tensor<32x8xi8, #blocked2> -> tensor<32x8xi8, #linear1>
    // CHECK: tt.dot_scaled %[[A]] scale %[[SCALE0]], %[[B]] scale %[[SCALE1]], %[[C]] lhs = e4m3 rhs = e4m3
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked3>
    %1 = tt.dot_scaled %arg0 scale %arg2, %arg1 scale %arg3, %cst lhs = e4m3 rhs = e4m3 {fastMath = false} : tensor<32x256xf8E4M3FN, #blocked>, tensor<32x8xi8, #blocked2> * tensor<256x32xf8E4M3FN, #blocked1>, tensor<32x8xi8, #blocked2> -> tensor<32x32xf32, #blocked3>
    tt.store %arg4, %1 : tensor<32x32x!tt.ptr<f32>, #blocked3>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_scaled_mxfp8_repeat_k`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_scaled_mxfp8_repeat_k`。

### Line 166
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 168-196
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
// CHECK{LITERAL}: #linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [32, 0]], warp = [[0, 0], [16, 0]], block = []}>
// CHECK{LITERAL}: #linear1 = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [32, 0]], warp = [[16, 0], [0, 0]], block = []}>
// CHECK{LITERAL}: #mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 128]}>
// CHECK-LABEL: wmma_dot_scaled_mxfp8_repeat_mn
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_scaled_mxfp8_repeat_mn(
      %arg0: tensor<64x128xf8E4M3FN, #blocked>,
      %arg1: tensor<128x64xf8E4M3FN, #blocked1>,
      %arg2: tensor<64x4xi8, #blocked2>,
      %arg3: tensor<64x4xi8, #blocked2>,
      %arg4: tensor<64x64x!tt.ptr<f32>, #blocked3>
      ) {
    // CHECK-NOT: tt.fp_to_fp
    // CHECK: %[[C:.+]] = ttg.convert_layout {{.*}} : tensor<64x64xf32, #blocked3> -> tensor<64x64xf32, #mma>
    // CHECK: %[[A:.+]] = ttg.convert_layout {{.*}} : tensor<64x128xf8E4M3FN, #blocked> -> tensor<64x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>
    // CHECK: %[[B:.+]] = ttg.convert_layout {{.*}} : tensor<128x64xf8E4M3FN, #blocked1> -> tensor<128x64xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>
    // CHECK: %[[SCALE0:.+]] = ttg.convert_layout {{.*}} : tensor<64x4xi8, #blocked2> -> tensor<64x4xi8, #linear>
    // CHECK: %[[SCALE1:.+]] = ttg.convert_layout {{.*}} : tensor<64x4xi8, #blocked2> -> tensor<64x4xi8, #linear1>
    // CHECK: tt.dot_scaled %[[A]] scale %[[SCALE0]], %[[B]] scale %[[SCALE1]], %[[C]] lhs = e4m3 rhs = e4m3
    %cst = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #blocked3>
    %1 = tt.dot_scaled %arg0 scale %arg2, %arg1 scale %arg3, %cst lhs = e4m3 rhs = e4m3 {fastMath = false} : tensor<64x128xf8E4M3FN, #blocked>, tensor<64x4xi8, #blocked2> * tensor<128x64xf8E4M3FN, #blocked1>, tensor<64x4xi8, #blocked2> -> tensor<64x64xf32, #blocked3>
    tt.store %arg4, %1 : tensor<64x64x!tt.ptr<f32>, #blocked3>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_scaled_mxfp8_repeat_mn`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_scaled_mxfp8_repeat_mn`。

### Line 198
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 200-224
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked4 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
// CHECK{LITERAL}: #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], lane = [[0, 32], [0, 64], [1, 0], [2, 0], [4, 0]], warp = [[8, 0], [16, 0]], block = []}>
// CHECK-LABEL: wmma_dot_scaled_mxfp8_bf16
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_scaled_mxfp8_bf16(
      %arg0: tensor<32x128x!tt.ptr<f8E4M3FN>, #blocked4>,
      %arg1: tensor<32x4x!tt.ptr<i8>, #blocked2>,
      %arg2: tensor<128x32x!tt.ptr<bf16>, #blocked>,
      %output: tensor<32x32x!tt.ptr<f32>, #blocked>
      ) {
    // CHECK: tt.load %arg1 {amdg.decomposed_dot_scaled_source = true} : tensor<32x4x!tt.ptr<i8>, #blocked1>
    // CHECK: %[[SCALE:.*]] = tt.reshape {{.*}} : tensor<32x4x32xi8, #blocked3> -> tensor<32x128xi8, #linear>
    // CHECK: %[[CVT0:.*]]  = ttg.convert_layout %[[SCALE]] : tensor<32x128xi8, #linear> -> tensor<32x128xi8, #blocked>
    // CHECK: %[[UPCASTED:.*]] = amdg.scaled_upcast_fp8 {{.*}} scale %[[CVT0]] : tensor<32x128xf8E4M3FN, #blocked>, tensor<32x128xi8, #blocked> -> tensor<32x128xbf16, #blocked>
    // CHECK: %[[SEL:.*]] = arith.select {{.*}}, {{.*}}, %[[UPCASTED]]
    // CHECK: %[[CVT1:.*]] = ttg.convert_layout %[[SEL]] : tensor<32x128xbf16, #blocked> -> tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
    // CHECK: %[[OPND0:.*]] = ttg.convert_layout %[[CVT1]] : tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<32x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    // CHECK: tt.dot %[[OPND0]]
    %a = tt.load %arg0 : tensor<32x128x!tt.ptr<f8E4M3FN>, #blocked4>
    %scale = tt.load %arg1 : tensor<32x4x!tt.ptr<i8>, #blocked2>
    %b = tt.load %arg2 : tensor<128x32x!tt.ptr<bf16>, #blocked>
    %c = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    %res = tt.dot_scaled %a scale %scale, %b, %c lhs = e4m3 rhs = bf16 {fastMath = false} : tensor<32x128xf8E4M3FN, #blocked4>, tensor<32x4xi8, #blocked2> * tensor<128x32xbf16, #blocked> -> tensor<32x32xf32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_scaled_mxfp8_bf16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_scaled_mxfp8_bf16`。

### Lines 226-229
```mlir
    tt.store %output, %res : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.store`, `tt.ptr`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.store`, `tt.ptr`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 231
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 233-257
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked4 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
// CHECK{LITERAL}: #linear = #ttg.linear<{register = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], lane = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], warp = [[32, 0], [64, 0]], block = []}>
// CHECK-LABEL: wmma_dot_scaled_f16_mxfp8
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_scaled_f16_mxfp8(
      %arg0: tensor<32x128x!tt.ptr<f16>, #blocked4>,
      %arg1: tensor<32x4x!tt.ptr<i8>, #blocked2>,
      %arg2: tensor<128x32x!tt.ptr<f8E5M2>, #blocked>,
      %output: tensor<32x32x!tt.ptr<f32>, #blocked>
      ) {
    // CHECK: %[[TRANS:.*]] = tt.trans {{.*}} {order = array<i32: 0, 2, 1>} : tensor<4x32x32xi8, #blocked4> -> tensor<4x32x32xi8, #blocked5>
    // CHECK: %[[SCALE:.*]] = tt.reshape %[[TRANS]] : tensor<4x32x32xi8, #blocked5> -> tensor<128x32xi8, #linear>
    // CHECK: %[[CVT0:.*]] = ttg.convert_layout %[[SCALE]] : tensor<128x32xi8, #linear> -> tensor<128x32xi8, #blocked2>
    // CHECK: %[[UPCASTED:.*]] = amdg.scaled_upcast_fp8 {{.*}} scale %[[CVT0]] : tensor<128x32xf8E5M2, #blocked2>, tensor<128x32xi8, #blocked2> -> tensor<128x32xf16, #blocked2>
    // CHECK: %[[SEL:.*]] = arith.select {{.*}}, %cst, %[[UPCASTED]] : tensor<128x32xi1, #blocked2>, tensor<128x32xf16, #blocked2>
    // CHECK: %[[CVT1:.*]] = ttg.convert_layout %[[SEL]] : tensor<128x32xf16, #blocked2> -> tensor<128x32xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked2}>>
    // CHECK: %[[OPND1:.*]] = ttg.convert_layout %[[CVT1]] : tensor<128x32xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked2}>> -> tensor<128x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    // CHECK: = tt.dot {{.*}}, %[[OPND1]]
    %a = tt.load %arg0 : tensor<32x128x!tt.ptr<f16>, #blocked4>
    %scale = tt.load %arg1 : tensor<32x4x!tt.ptr<i8>, #blocked2>
    %b = tt.load %arg2 : tensor<128x32x!tt.ptr<f8E5M2>, #blocked>
    %c = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    %res = tt.dot_scaled %a, %b scale %scale, %c lhs = fp16 rhs = e5m2 {fastMath = false} : tensor<32x128xf16, #blocked4> * tensor<128x32xf8E5M2, #blocked>,  tensor<32x4xi8, #blocked2> -> tensor<32x32xf32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_scaled_f16_mxfp8`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_scaled_f16_mxfp8`。

### Lines 259-262
```mlir
    tt.store %output, %res : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.store`, `tt.ptr`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.store`, `tt.ptr`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 264
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 266-291
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked5 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
// CHECK{LITERAL}: #linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], lane = [[0, 32], [1, 0], [2, 0], [4, 0], [8, 0]], warp = [[0, 0], [0, 0]], block = []}>
// CHECK-LABEL: wmma_dot_scaled_mxfp4_bf16
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_scaled_mxfp4_bf16(
      %arg0: tensor<16x32x!tt.ptr<i8>, #blocked5>,
      %arg1: tensor<16x2x!tt.ptr<i8>, #blocked2>,
      %arg2: tensor<64x16x!tt.ptr<bf16>, #blocked>,
      %output: tensor<16x16x!tt.ptr<f32>, #blocked>
      ) {
    // CHECK: tt.load %arg1 {amdg.decomposed_dot_scaled_source = true} : tensor<16x2x!tt.ptr<i8>, #blocked1>
    // CHECK: %[[SCALE:.*]] = tt.reshape {{.*}} : tensor<16x2x32xi8, #[[RESHAPE_LAYOUT:.+]]> -> tensor<16x64xi8, #linear>
    // CHECK: %[[CVT0:.*]] = ttg.convert_layout %[[SCALE]] : tensor<16x64xi8, #linear> -> tensor<16x64xi8, #[[UPCAST_LAYOUT:.+]]>
    // CHECK: %[[UPCASTED:.*]] = amdg.scaled_upcast_fp4 {{.+}} scale %[[CVT0]] {axis = 1 : i32} : tensor<16x32xi8, #blocked>, tensor<16x64xi8, #[[UPCAST_LAYOUT]]> -> tensor<16x64xbf16, #[[UPCAST_LAYOUT]]>
    // CHECK: %[[SEL:.*]] = arith.select {{.*}}, %{{.*}}, %[[UPCASTED]] : tensor<16x64xi1, #[[UPCAST_LAYOUT]]>, tensor<16x64xbf16, #[[UPCAST_LAYOUT]]>
    // CHECK: %[[CVT1:.*]] = ttg.convert_layout %[[SEL]] : tensor<16x64xbf16, #[[UPCAST_LAYOUT]]> -> tensor<16x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
    // CHECK: %[[OPND0:.*]] = ttg.convert_layout %[[CVT1]] : tensor<16x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<16x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    // CHECK: tt.dot %[[OPND0]]
    %a = tt.load %arg0 : tensor<16x32x!tt.ptr<i8>, #blocked5>
    %scale = tt.load %arg1 : tensor<16x2x!tt.ptr<i8>, #blocked2>
    %b = tt.load %arg2 : tensor<64x16x!tt.ptr<bf16>, #blocked>
    %c = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #blocked>
    %res = tt.dot_scaled %a scale %scale, %b, %c lhs = e2m1 rhs = bf16 {fastMath = false} : tensor<16x32xi8, #blocked5>, tensor<16x2xi8, #blocked2> * tensor<64x16xbf16, #blocked> -> tensor<16x16xf32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_scaled_mxfp4_bf16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_scaled_mxfp4_bf16`。

### Lines 293-296
```mlir
    tt.store %output, %res : tensor<16x16x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.store`, `tt.ptr`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.store`, `tt.ptr`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 298
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 300-324
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked5 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
// CHECK{LITERAL}: #linear = #ttg.linear<{register = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], lane = [[0, 1], [0, 2], [0, 4], [0, 8], [32, 0]], warp = [[0, 0], [0, 0]], block = []}>
// CHECK-LABEL: wmma_dot_scaled_fp16_mxfp4
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_scaled_fp16_mxfp4(
      %arg0: tensor<16x64x!tt.ptr<f16>, #blocked5>,
      %arg1: tensor<16x2x!tt.ptr<i8>, #blocked2>,
      %arg2: tensor<32x16x!tt.ptr<i8>, #blocked>,
      %output: tensor<16x16x!tt.ptr<f32>, #blocked>
      ) {
    // CHECK: tt.load %arg1 {amdg.decomposed_dot_scaled_source = true} : tensor<16x2x!tt.ptr<i8>, #[[LOAD_LAYOUT:.+]]>
    // CHECK: %[[SCALE:.*]] = tt.reshape {{.*}} : tensor<2x32x16xi8, #[[RESHAPE_LAYOUT:.+]]> -> tensor<64x16xi8, #linear>
    // CHECK: %[[CVT0:.*]] = ttg.convert_layout %[[SCALE]] : tensor<64x16xi8, #linear> -> tensor<64x16xi8, #[[UPCAST_LAYOUT:.+]]>
    // CHECK: %[[UPCASTED:.*]] = amdg.scaled_upcast_fp4 {{.+}} scale %[[CVT0]] {axis = 0 : i32} : tensor<32x16xi8, #blocked2>, tensor<64x16xi8, #[[UPCAST_LAYOUT]]> -> tensor<64x16xf16, #[[UPCAST_LAYOUT]]>
    // CHECK: %[[SEL:.*]] = arith.select {{.*}}, %cst, %[[UPCASTED]] : tensor<64x16xi1, #[[UPCAST_LAYOUT]]>, tensor<64x16xf16, #[[UPCAST_LAYOUT]]>
    // CHECK: %[[CVT1:.*]] = ttg.convert_layout %[[SEL]] : tensor<64x16xf16, #[[UPCAST_LAYOUT]]> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked2}>>
    // CHECK: %[[OPND1:.*]] = ttg.convert_layout %[[CVT1]] : tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked2}>> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    // CHECK: tt.dot {{.*}}, %[[OPND1]]
    %a = tt.load %arg0 : tensor<16x64x!tt.ptr<f16>, #blocked5>
    %scale = tt.load %arg1 : tensor<16x2x!tt.ptr<i8>, #blocked2>
    %b = tt.load %arg2 : tensor<32x16x!tt.ptr<i8>, #blocked>
    %c = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #blocked>
    %res = tt.dot_scaled %a, %b scale %scale, %c lhs = fp16 rhs = e2m1 {fastMath = false} : tensor<16x64xf16, #blocked5> * tensor<32x16xi8, #blocked>, tensor<16x2xi8, #blocked2> -> tensor<16x16xf32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_scaled_fp16_mxfp4`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_scaled_fp16_mxfp4`。

### Lines 326-329
```mlir
    tt.store %output, %res : tensor<16x16x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.store`, `tt.ptr`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.store`, `tt.ptr`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 331
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 333-335
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
#op0 = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
#op1 = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#op0`, `#op1`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.dot_op`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#op0`, `#op1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.dot_op`。

### Lines 337-347
```mlir
// CHECK{LITERAL}: #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [0, 2], [1, 0]]}, instrShape = [16, 16, 64]}>
// CHECK-LABEL: wmma_dot_i8_i32
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_i8_i32(
      %arg0: tensor<64x128x!tt.ptr<i8>, #op0>,
      %arg1: tensor<128x128x!tt.ptr<i8>, #op1>,
      %arg2: tensor<64x128x!tt.ptr<i32>, #blocked>
      ) {
    %a = tt.load %arg0 : tensor<64x128x!tt.ptr<i8>, #op0>
    %b = tt.load %arg1 : tensor<128x128x!tt.ptr<i8>, #op1>
    %c = arith.constant dense<0> : tensor<64x128xi32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_i8_i32`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_i8_i32`。

### Lines 349-353
```mlir
    %res = tt.dot %a, %b, %c : tensor<64x128xi8, #op0> * tensor<128x128xi8, #op1> -> tensor<64x128xi32, #blocked>
    tt.store %arg2, %res : tensor<64x128x!tt.ptr<i32>, #blocked>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.dot`, `tt.store`, `tt.ptr`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot`, `tt.store`, `tt.ptr`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 355
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 357-359
```mlir
#blocked = #ttg.blocked<{sizePerThread = [2, 2], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0] }>
#op0 = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
#op1 = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#op0`, `#op1`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.dot_op`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#op0`, `#op1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.dot_op`。

### Lines 361-378
```mlir
// CHECK{LITERAL}: #mma = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = [[0, 1], [1, 0]]}, instrShape = [16, 16, 32]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_f16_f32_smallk(
      %arg0: tensor<32x8x!tt.ptr<f16>, #op0>,
      %arg1: tensor<8x32x!tt.ptr<f16>, #op1>,
      %arg2: tensor<32x32x!tt.ptr<f32>, #blocked>
      ) {
    %a = tt.load %arg0 : tensor<32x8x!tt.ptr<f16>, #op0>
    %b = tt.load %arg1 : tensor<8x32x!tt.ptr<f16>, #op1>
    %c = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    // CHECK: %[[OPND0:.*]] = ttg.convert_layout {{.*}} : tensor<32x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> -> tensor<32x8xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    // CHECK: %[[OPND1:.*]] = ttg.convert_layout {{.*}} : tensor<8x32xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<8x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    // CHECK: tt.dot %[[OPND0]], %[[OPND1]], %{{.*}} : tensor<32x8xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<8x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<32x32xf32, #mma>
    %res = tt.dot %a, %b, %c : tensor<32x8xf16, #op0> * tensor<8x32xf16, #op1> -> tensor<32x32xf32, #blocked>
    tt.store %arg2, %res : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_f16_f32_smallk`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_f16_f32_smallk`。

### Line 380
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 382-393
```mlir
// NOTE: A/B/C/D's CGA-layout are not necessarily equal when num-ctas > 1
//  - D and C's should have the same CGA-layout, in this case [[0, 1], [1, 0]]
//  - A and B's CGA-layout is derived from D-CGA-layout by clearing the elements,
//    corresponding to the K dim, in the bases to zero. Hence, one have layout
//    [[0, 0] [1, ]], the other one has layout [[0, 1], [0, 0]]
//  - However, A and B's CGAlayout is inferred on the fly. We only see parent's
//    encoding in the IR.
//
// CHECK-NOT: #mma{{.*}} = #ttg.amd_wmma<{version = 3, isTranspose = true, {{.*}} CGALayout = {{\[\[0, 0\], \[1, 0\]\]}}
// CHECK-NOT: #mma{{.*}} = #ttg.amd_wmma<{version = 3, isTranspose = true, {{.*}} CGALayout = {{\[\[0, 1\], \[0, 0\]\]}}
// CHECK: #mma{{.*}} = #ttg.amd_wmma<{version = 3, isTranspose = true, {{.*}} CGALayout = {{\[\[0, 1\], \[1, 0\]\]}}
// CHECK-LABEL: test_multi_ctas
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-LABEL`, `CHECK-NOT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-LABEL`, `CHECK-NOT`），从而保证后端/代码生成结果稳定。

### Lines 395-406
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[0, 1], [1, 0]]}>
module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @test_multi_ctas(
    %0: tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
    %1: tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
    %2: tensor<32x32x!tt.ptr<i32>, #blocked>) {
    %3 = arith.constant dense<0> : tensor<32x32xi32, #blocked>
    %4 = tt.dot %0, %1, %3 : tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<32x32xi32, #blocked>
    tt.store %2, %4 : tensor<32x32x!tt.ptr<i32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_multi_ctas`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_multi_ctas`。

### Line 408
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 410-414
```mlir
// CHECK-DAG: [[LINEAR1:#linear.*]] = #ttg.linear<{{.*}} block = {{\[\[0, 0\], \[64, 0\]\]}}
// CHECK-DAG: [[LINEAR2:#linear.*]] = #ttg.linear<{{.*}} block = {{\[\[64, 0\], \[0, 0\]\]}}
// CHECK-DAG: [[MMA:#mma.*]] = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {{.*}}warp = {{\[\[0, 1\], \[1, 0\]\]}}{{.*}} CGALayout = {{\[\[0, 1\], \[1, 0\]\]}}, instrShape = {{\[16, 16, 128\]}}
// CHECK: tt.func public @mxfp_matmul_multi_cta
// CHECK: tt.dot_scaled {{.*}} tensor<128x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<128x2xi8, [[LINEAR1]]> * tensor<64x128xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<128x2xi8, [[LINEAR2]]> -> tensor<128x128xf32, [[MMA]]>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-DAG`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-DAG`），从而保证后端/代码生成结果稳定。

### Lines 416-435
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0], CGALayout = [[0, 1], [1, 0]]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[0, 0], [1, 0]]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[1, 0], [0, 0]]}>
#blocked3 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[0, 1], [1, 0]]}>
#blocked4 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0], CGALayout = [[0, 0], [1, 0]]}>
#blocked5 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0], CGALayout = [[0, 1], [0, 0]]}>
module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @mxfp_matmul_multi_cta(
    %a: tensor<128x64xf8E5M2, #blocked4>,
    %b: tensor<64x128xf8E5M2, #blocked5>,
    %a_scale: tensor<128x2xi8, #blocked1>,
    %b_scale: tensor<128x2xi8, #blocked2>,
    %d: tensor<128x128x!tt.ptr<f32>, #blocked3>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %res = tt.dot_scaled %a scale %a_scale, %b scale %b_scale, %cst lhs = e5m2 rhs = e5m2 {fastMath = false} : tensor<128x64xf8E5M2, #blocked4>, tensor<128x2xi8, #blocked1> * tensor<64x128xf8E5M2, #blocked5>, tensor<128x2xi8, #blocked2> -> tensor<128x128xf32, #blocked>
    %cvt = ttg.convert_layout %res : tensor<128x128xf32, #blocked> -> tensor<128x128xf32, #blocked3>
    tt.store %d, %cvt : tensor<128x128x!tt.ptr<f32>, #blocked3>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mxfp_matmul_multi_cta`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mxfp_matmul_multi_cta`。

### Line 437
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 439-469
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
// CHECK-DAG: #[[$SCALE0_LAYOUT:.+]] = #ttg.linear{{<{register = \[\[0, 1\], \[0, 2\]\], lane = \[\[1, 0\], \[2, 0\], \[4, 0\], \[8, 0\], \[0, 0\]\], warp = \[\[0, 0\], \[16, 0\]\], block = \[\]}>}}
// CHECK-DAG: #[[$SCALE1_LAYOUT:.+]] = #ttg.linear{{<{register = \[\[0, 1\], \[0, 2\]\], lane = \[\[1, 0\], \[2, 0\], \[4, 0\], \[8, 0\], \[0, 0\]\], warp = \[\[16, 0\], \[0, 0\]\], block = \[\]}>}}
// CHECK-DAG: #[[$MMA:.+]] = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = {{\[\[0, 1\], \[1, 0\]\]}}}, instrShape = [16, 16, 128]}>
// CHECK-DAG: #[[$MMA1:.+]] = #ttg.amd_wmma<{version = 3, isTranspose = true, ctaLayout = {warp = {{\[\[0, 1\], \[1, 0\]\]}}}, instrShape = [16, 16, 64]}>
// CHECK-LABEL: wmma_dot_scaled_mxfp4_mxfp4_missing_scale
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_scaled_mxfp4_mxfp4_missing_scale(
      %arg0: tensor<32x64xi8, #blocked>,
      %arg1: tensor<64x32xi8, #blocked1>,
      %arg2: tensor<32x4xi8, #blocked2>,
      %arg3: tensor<32x32x!tt.ptr<f32>, #blocked3>
      ) {
    // CHECK-NOT: arith.constant dense<127> : tensor<32x4xi8, #[[$SCALE0_LAYOUT]]>
    // CHECK: %[[SCALE1:.+]] = arith.constant dense<127> : tensor<32x4xi8, #[[$SCALE1_LAYOUT]]>
    // CHECK-NOT: arith.constant dense<127> : tensor<32x4xi8, #[[$SCALE0_LAYOUT]]>
    // CHECK-NOT: tt.fp_to_fp
    // CHECK: %[[C:.+]] = ttg.convert_layout {{.*}} : tensor<32x32xf32, {{.*}}> -> tensor<32x32xf32, #[[$MMA]]>
    // CHECK: %[[A:.+]] = ttg.convert_layout {{.*}} : tensor<32x64xi8, {{.*}}> -> tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #[[$MMA1]], kWidth = 16}>>
    // CHECK: %[[B:.+]] = ttg.convert_layout {{.*}} : tensor<64x32xi8, {{.*}}> -> tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #[[$MMA1]], kWidth = 16}>>
    // CHECK: %[[SCALE0:.+]] = ttg.convert_layout {{.*}} : tensor<32x4xi8, {{.*}}> -> tensor<32x4xi8, #[[$SCALE0_LAYOUT]]>
    // CHECK: tt.dot_scaled %[[A]] scale %[[SCALE0]], %[[B]] scale %[[SCALE1]], %[[C]] lhs = e2m1 rhs = e2m1
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked3>
    %1 = tt.dot_scaled %arg0 scale %arg2, %arg1, %cst lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<32x64xi8, #blocked>, tensor<32x4xi8, #blocked2> * tensor<64x32xi8, #blocked1>, tensor<32x4xi8, #blocked2> -> tensor<32x32xf32, #blocked3>
    tt.store %arg3, %1 : tensor<32x32x!tt.ptr<f32>, #blocked3>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_scaled_mxfp4_mxfp4_missing_scale`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_scaled_mxfp4_mxfp4_missing_scale`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's TritonGPU transforms coverage and focuses on WMMA lowering.  
  **CN:** 该文件属于 Triton 的 TritonGPU 变换 测试覆盖，关注点是 WMMA 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx1250"`.  
  **CN:** `RUN` 流水线会驱动 `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx1250"` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`。
