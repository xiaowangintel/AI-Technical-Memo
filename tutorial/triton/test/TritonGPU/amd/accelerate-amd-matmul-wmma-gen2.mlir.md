# accelerate-amd-matmul-wmma-gen2.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/TritonGPU/amd/accelerate-amd-matmul-wmma-gen2.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises WMMA lowering in Triton's TritonGPU transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton TritonGPU 变换中的WMMA 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx1200 matrix-instruction-size=0" | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×57, CHECK-NOT×2, CHECK-SAME×39; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×57, CHECK-NOT×2, CHECK-SAME×39；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that WMMA lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 WMMA 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx1200 matrix-instruction-size=0" | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx1200 matrix-instruction-size=0" | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx1200 matrix-instruction-size=0" | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-29
```mlir
// CHECK: #[[DOT_OP_PARENT:.+]] = #ttg.blocked<{{.*}}>
// CHECK: #[[WMMA_0:.+]] = #ttg.amd_wmma<{version = 2, isTranspose = true, ctaLayout = {warp = {{\[\[0, 1\], \[0, 2\]\]}}}}>
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_cf32(
   // CHECK: %[[DOT0_ARG_A:.+]]: tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #[[DOT_OP_PARENT]]}>>
   %0: tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
   // CHECK-SAME: %[[DOT0_ARG_B:.+]]: tensor<64x256xf16, #ttg.dot_op<{opIdx = 1, parent = #[[DOT_OP_PARENT]]}>>
   %1: tensor<64x256xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
   %2: tensor<128x256x!tt.ptr<f32>, #blocked>) {
    // CHECK: %[[DOT0_ARG_C:.+]] = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #[[DOT_OP_PARENT]]>
    // CHECK: %[[DOT0_OP_C:.+]] = ttg.convert_layout %[[DOT0_ARG_C]]
    // CHECK-SAME: -> tensor<128x256xf32, #[[WMMA_0]]
    %3 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked>
    // CHECK: %[[DOT0_OP_A:.+]] = ttg.convert_layout %[[DOT0_ARG_A]]
    // CHECK-SAME: -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #[[WMMA_0]]
    // CHECK: %[[DOT0_OP_B:.+]] = ttg.convert_layout %[[DOT0_ARG_B]]
    // CHECK-SAME: -> tensor<64x256xf16, #ttg.dot_op<{opIdx = 1, parent = #[[WMMA_0]]
    // CHECK: %[[DOT0_WMMA_RES:.+]] = tt.dot %[[DOT0_OP_A]], %[[DOT0_OP_B]], %[[DOT0_OP_C]]
    // CHECK-SAME: -> tensor<128x256xf32, #[[WMMA_0]]
    %4 = tt.dot %0, %1, %3 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x256xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<128x256xf32, #blocked>
    // CHECK: ttg.convert_layout %[[DOT0_WMMA_RES]]
    // CHECK-SAME: -> tensor<128x256xf32, #[[DOT_OP_PARENT]]>
    tt.store %2, %4 : tensor<128x256x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_cf32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_cf32`。

### Line 31
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 33-63
```mlir
// CHECK: #[[DOT_OP_PARENT:.+]] = #ttg.blocked<{{.*}}>
// CHECK: #[[WMMA_1:.+]] = #ttg.amd_wmma<{version = 2, isTranspose = true, ctaLayout = {warp = {{\[\[0, 1\], \[1, 0\]\]}}}}>
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_cf16(
   // CHECK: %[[DOT1_ARG_A:.+]]: tensor<32x64xf16, #ttg.dot_op<{opIdx = 0, parent = #[[DOT_OP_PARENT]]}>>
   %0: tensor<32x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
   // CHECK-SAME: %[[DOT1_ARG_B:.+]]: tensor<64x32xf16, #ttg.dot_op<{opIdx = 1, parent = #[[DOT_OP_PARENT]]}>>
   %1: tensor<64x32xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
   %2: tensor<32x32x!tt.ptr<f16>, #blocked>) {
    // CHECK: %[[DOT1_ARG_C:.+]] = arith.constant dense<0.000000e+00> : tensor<32x32xf16, #[[DOT_OP_PARENT]]>
    // CHECK: %[[DOT1_OP_C:.+]] = ttg.convert_layout %[[DOT1_ARG_C]]
    // CHECK-SAME: -> tensor<32x32xf16, #[[WMMA_1]]>
    // CHECK: %[[DOT1_OP_C_EXT:.+]] = arith.extf %[[DOT1_OP_C]]
    // CHECK-SAME: to tensor<32x32xf32, #[[WMMA_1]]>
    %3 = arith.constant dense<0.000000e+00> : tensor<32x32xf16, #blocked>
    // CHECK: %[[DOT1_OP_A:.+]] = ttg.convert_layout %[[DOT1_ARG_A]]
    // CHECK-SAME: -> tensor<32x64xf16, #ttg.dot_op<{opIdx = 0, parent = #[[WMMA_1]]
    // CHECK: %[[DOT1_OP_B:.+]] = ttg.convert_layout %[[DOT1_ARG_B]]
    // CHECK-SAME: -> tensor<64x32xf16, #ttg.dot_op<{opIdx = 1, parent = #[[WMMA_1]]
    // CHECK: %[[DOT1_WMMA_RES:.+]] = tt.dot %[[DOT1_OP_A]], %[[DOT1_OP_B]], %[[DOT1_OP_C_EXT]]
    // CHECK-SAME: -> tensor<32x32xf32, #[[WMMA_1]]
    %4 = tt.dot %0, %1, %3 : tensor<32x64xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x32xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<32x32xf16, #blocked>
    // CHECK: %[[CONVERTED_RES:.+]] = ttg.convert_layout %[[DOT1_WMMA_RES]]
    // CHECK-SAME: -> tensor<32x32xf32, #[[DOT_OP_PARENT]]>
    // CHECK: arith.truncf %[[CONVERTED_RES]]
    // CHECK-SAME: to tensor<32x32xf16, #[[DOT_OP_PARENT]]>
    tt.store %2, %4 : tensor<32x32x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_cf16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_cf16`。

### Line 65
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 67-97
```mlir
// CHECK: #[[DOT_OP_PARENT:.+]] = #ttg.blocked<{{.*}}>
// CHECK: #[[WMMA_0:.+]] = #ttg.amd_wmma<{version = 2, isTranspose = true, ctaLayout = {warp = {{\[\[0, 1\], \[0, 2\]\]}}}}>
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_ab8_cf16(
   // CHECK: %[[DOT2_ARG_A:.+]]: tensor<32x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #[[DOT_OP_PARENT]]}>>
   %0: tensor<32x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
   // CHECK-SAME: %[[DOT2_ARG_B:.+]]: tensor<128x64xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #[[DOT_OP_PARENT]]}>>
   %1: tensor<128x64xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
   %2: tensor<32x64x!tt.ptr<f16>, #blocked>) {
    // CHECK: %[[DOT2_ARG_C:.+]] = arith.constant dense<0.000000e+00> : tensor<32x64xf16, #[[DOT_OP_PARENT]]>
    // CHECK: %[[DOT2_OP_C:.+]] = ttg.convert_layout %[[DOT2_ARG_C]]
    // CHECK-SAME: -> tensor<32x64xf16, #[[WMMA_0]]>
    // CHECK: %[[DOT2_OP_C_EXT:.+]] = arith.extf %[[DOT2_OP_C]]
    // CHECK-SAME: to tensor<32x64xf32, #[[WMMA_0]]>
    %3 = arith.constant dense<0.000000e+00> : tensor<32x64xf16, #blocked>
    // CHECK: %[[DOT2_OP_A:.+]] = ttg.convert_layout %[[DOT2_ARG_A]]
    // CHECK-SAME: -> tensor<32x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #[[WMMA_0]]
    // CHECK: %[[DOT2_OP_B:.+]] = ttg.convert_layout %[[DOT2_ARG_B]]
    // CHECK-SAME: -> tensor<128x64xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #[[WMMA_0]]
    // CHECK: %[[DOT2_WMMA_RES:.+]] = tt.dot %[[DOT2_OP_A]], %[[DOT2_OP_B]], %[[DOT2_OP_C_EXT]]
    // CHECK-SAME: -> tensor<32x64xf32, #[[WMMA_0]]
    %4 = tt.dot %0, %1, %3 : tensor<32x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<128x64xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<32x64xf16, #blocked>
    // CHECK: %[[CONVERTED_RES:.+]] = ttg.convert_layout %[[DOT2_WMMA_RES]]
    // CHECK-SAME: -> tensor<32x64xf32, #[[DOT_OP_PARENT]]>
    // CHECK: arith.truncf %[[CONVERTED_RES]]
    // CHECK-SAME: to tensor<32x64xf16, #[[DOT_OP_PARENT]]>
    tt.store %2, %4 : tensor<32x64x!tt.ptr<f16>, #blocked>
        tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_ab8_cf16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_ab8_cf16`。

### Line 99
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 101-127
```mlir
// CHECK: #[[DOT_OP_PARENT:.+]] = #ttg.blocked<{{.*}}>
// CHECK: #[[WMMA_1:.+]] = #ttg.amd_wmma<{version = 2, isTranspose = true, ctaLayout = {warp = {{\[\[0, 1\], \[1, 0\]\]}}}}>
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_i8_i32(
   // CHECK: %[[DOT1_ARG_A:.+]]: tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #[[DOT_OP_PARENT]]}>>
   %0: tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
   // CHECK-SAME: %[[DOT1_ARG_B:.+]]: tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #[[DOT_OP_PARENT]]}>>
   %1: tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
   %2: tensor<32x32x!tt.ptr<i32>, #blocked>) {
    // CHECK: %[[DOT1_ARG_C:.+]] = arith.constant dense<0> : tensor<32x32xi32, #[[DOT_OP_PARENT]]>
    // CHECK: %[[DOT1_OP_C:.+]] = ttg.convert_layout %[[DOT1_ARG_C]]
    // CHECK-SAME: -> tensor<32x32xi32, #[[WMMA_1]]
    %3 = arith.constant dense<0> : tensor<32x32xi32, #blocked>
    // CHECK: %[[DOT1_OP_A:.+]] = ttg.convert_layout %[[DOT1_ARG_A]]
    // CHECK-SAME: -> tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #[[WMMA_1]]
    // CHECK: %[[DOT1_OP_B:.+]] = ttg.convert_layout %[[DOT1_ARG_B]]
    // CHECK-SAME: -> tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #[[WMMA_1]]
    // CHECK: %[[DOT1_WMMA_RES:.+]] = tt.dot %[[DOT1_OP_A]], %[[DOT1_OP_B]], %[[DOT1_OP_C]]
    // CHECK-SAME: -> tensor<32x32xi32, #[[WMMA_1]]
    %4 = tt.dot %0, %1, %3 : tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<32x32xi32, #blocked>
    // CHECK: ttg.convert_layout %[[DOT1_WMMA_RES]]
    // CHECK-SAME: -> tensor<32x32xi32, #[[DOT_OP_PARENT]]>
    tt.store %2, %4 : tensor<32x32x!tt.ptr<i32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_i8_i32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_i8_i32`。

### Line 129
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 131-160
```mlir
// Test for mixed FP8 types (f8E4M3FN x f8E5M2) - verifies that no fp_to_fp
// conversion is generated since hardware supports mixed FP8 natively.
// CHECK: #[[DOT_OP_PARENT:.+]] = #ttg.blocked<{{.*}}>
// CHECK: #[[WMMA:.+]] = #ttg.amd_wmma<{version = 2, isTranspose = true, ctaLayout = {warp = {{\[\[0, 1\], \[1, 0\]\]}}}}>
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_mixed_fp8_bf8(
   // CHECK: %[[ARG_A:.+]]: tensor<32x64xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #[[DOT_OP_PARENT]]}>>
   %0: tensor<32x64xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
   // CHECK-SAME: %[[ARG_B:.+]]: tensor<64x32xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #[[DOT_OP_PARENT]]}>>
   %1: tensor<64x32xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
   %2: tensor<32x32x!tt.ptr<f32>, #blocked>) {
    // CHECK-NOT: tt.fp_to_fp
    // CHECK: %[[ARG_C:.+]] = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #[[DOT_OP_PARENT]]>
    // CHECK: %[[OP_C:.+]] = ttg.convert_layout %[[ARG_C]]
    // CHECK-SAME: -> tensor<32x32xf32, #[[WMMA]]>
    %3 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    // CHECK: %[[OP_A:.+]] = ttg.convert_layout %[[ARG_A]]
    // CHECK-SAME: -> tensor<32x64xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #[[WMMA]]
    // CHECK: %[[OP_B:.+]] = ttg.convert_layout %[[ARG_B]]
    // CHECK-SAME: -> tensor<64x32xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #[[WMMA]]
    // CHECK: %[[WMMA_RES:.+]] = tt.dot %[[OP_A]], %[[OP_B]], %[[OP_C]]
    // CHECK-SAME: tensor<32x64xf8E4M3FN, {{.*}}> * tensor<64x32xf8E5M2, {{.*}}> -> tensor<32x32xf32, #[[WMMA]]>
    %4 = tt.dot %0, %1, %3 : tensor<32x64xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x32xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<32x32xf32, #blocked>
    // CHECK: ttg.convert_layout %[[WMMA_RES]]
    // CHECK-SAME: -> tensor<32x32xf32, #[[DOT_OP_PARENT]]>
    tt.store %2, %4 : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_mixed_fp8_bf8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_mixed_fp8_bf8`。

### Line 162
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 164-191
```mlir
// Test for mixed FP8 types in reverse order (f8E5M2 x f8E4M3FN) - verifies
// hardware native support for bf8_fp8 WMMA instruction.
// CHECK: #[[DOT_OP_PARENT:.+]] = #ttg.blocked<{{.*}}>
// CHECK: #[[WMMA:.+]] = #ttg.amd_wmma<{version = 2, isTranspose = true, ctaLayout = {warp = {{\[\[0, 1\], \[1, 0\]\]}}}}>
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @wmma_dot_mixed_bf8_fp8(
   // CHECK: %[[ARG_A:.+]]: tensor<32x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #[[DOT_OP_PARENT]]}>>
   %0: tensor<32x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>,
   // CHECK-SAME: %[[ARG_B:.+]]: tensor<64x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #[[DOT_OP_PARENT]]}>>
   %1: tensor<64x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>,
   %2: tensor<32x32x!tt.ptr<f32>, #blocked>) {
    // CHECK-NOT: tt.fp_to_fp
    // CHECK: %[[ARG_C:.+]] = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #[[DOT_OP_PARENT]]>
    %3 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    // CHECK: %[[OP_A:.+]] = ttg.convert_layout %[[ARG_A]]
    // CHECK-SAME: -> tensor<32x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #[[WMMA]]
    // CHECK: %[[OP_B:.+]] = ttg.convert_layout %[[ARG_B]]
    // CHECK-SAME: -> tensor<64x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #[[WMMA]]
    // CHECK: %[[WMMA_RES:.+]] = tt.dot %[[OP_A]], %[[OP_B]]
    // CHECK-SAME: tensor<32x64xf8E5M2, {{.*}}> * tensor<64x32xf8E4M3FN, {{.*}}> -> tensor<32x32xf32, #[[WMMA]]>
    %4 = tt.dot %0, %1, %3 : tensor<32x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<64x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<32x32xf32, #blocked>
    // CHECK: ttg.convert_layout %[[WMMA_RES]]
    // CHECK-SAME: -> tensor<32x32xf32, #[[DOT_OP_PARENT]]>
    tt.store %2, %4 : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_dot_mixed_bf8_fp8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_dot_mixed_bf8_fp8`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's TritonGPU transforms coverage and focuses on WMMA lowering.  
  **CN:** 该文件属于 Triton 的 TritonGPU 变换 测试覆盖，关注点是 WMMA 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx1200`.  
  **CN:** `RUN` 流水线会驱动 `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx1200` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`。
