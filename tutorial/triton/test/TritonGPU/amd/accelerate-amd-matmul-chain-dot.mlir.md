# accelerate-amd-matmul-chain-dot.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/TritonGPU/amd/accelerate-amd-matmul-chain-dot.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises matmul acceleration in Triton's TritonGPU transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton TritonGPU 变换中的矩阵乘加速相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=16" | FileCheck %s --check-prefixes MFMA16,CHECK`<br>`triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=32" | FileCheck %s --check-prefixes MFMA32,CHECK`<br>`triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=32" | FileCheck %s --check-prefixes CHECK-GFX950`<br>`triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=16" | FileCheck %s --check-prefixes CHECK-GFX950` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×16, CHECK-LABEL×6; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×16, CHECK-LABEL×6；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that matmul acceleration produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 矩阵乘加速 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4
```mlir
// RUN: triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=16" | FileCheck %s --check-prefixes MFMA16,CHECK
// RUN: triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=32" | FileCheck %s --check-prefixes MFMA32,CHECK
// RUN: triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=32" | FileCheck %s --check-prefixes CHECK-GFX950
// RUN: triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=16" | FileCheck %s --check-prefixes CHECK-GFX950
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=16" | FileCheck %s --check-prefixes MFMA16,CHECK; triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=32" | FileCheck %s --check-prefixes MFMA32,CHECK; triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=32" | FileCheck %s --check-prefixes CHECK-GFX950; triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=16" | FileCheck %s --check-prefixes CHECK-GFX950` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=16" | FileCheck %s --check-prefixes MFMA16,CHECK; triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx942 matrix-instruction-size=32" | FileCheck %s --check-prefixes MFMA32,CHECK; triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=32" | FileCheck %s --check-prefixes CHECK-GFX950; triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=16" | FileCheck %s --check-prefixes CHECK-GFX950`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 6-8
```mlir
// Check the warpsPerCTA parameter of #mma layout of the two dot's.
// The 1st dot always has warpsPerCTA = [4, 1].
// The warpsPerCTA for the 2nd dot depends on mfma instruction size and BLOCK_M size.
```
**EN:** This comment block provides context for the surrounding test logic: `// Check the warpsPerCTA parameter of #mma layout of the two dot's.`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Check the warpsPerCTA parameter of #mma layout of the two dot's.`。

### Lines 11-36
```mlir
// BLOCK_M = 128
// warpsPerCTA = [4, 1] for mfma16 and mfma32
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [16, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#dotOp0 = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
#dotOp1 = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
// MFMA16{LITERAL}: #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true}>
// MFMA32{LITERAL}: #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>
// CHECK-LABEL: mfma_chain_dot_BM128
// CHECK: tt.dot {{.*}} : {{.*}} -> tensor<128x16xf32, #mma>
// CHECK: tt.dot {{.*}} : {{.*}} -> tensor<128x128xf32, #mma>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_chain_dot_BM128(
      %q: tensor<128x128xf16, #dotOp0>,
      %k: tensor<128x16xf16, #dotOp1>,
      %v: tensor<16x128xf16, #dotOp1>,
      %o_ptr: tensor<128x128x!tt.ptr<f32>, #blocked>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #blocked>
    %cst1 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %qk = tt.dot %q, %k, %cst : tensor<128x128xf16, #dotOp0> * tensor<128x16xf16, #dotOp1> -> tensor<128x16xf32, #blocked>
    %qk_f16 = arith.truncf %qk :  tensor<128x16xf32, #blocked> to tensor<128x16xf16, #blocked>
    %p = ttg.convert_layout %qk_f16 : tensor<128x16xf16, #blocked> -> tensor<128x16xf16, #dotOp0>
    %o = tt.dot %p, %v, %cst1 : tensor<128x16xf16, #dotOp0> * tensor<16x128xf16, #dotOp1> -> tensor<128x128xf32, #blocked>
    tt.store %o_ptr, %o : tensor<128x128x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_chain_dot_BM128`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_chain_dot_BM128`。

### Line 39
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 41-69
```mlir
// BLOCK_M = 64
// warpsPerCTA = [4, 1] for mfma16
// warpsPerCTA = [2, 2] for mfma32
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [16, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#dotOp0 = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
#dotOp1 = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
// MFMA16{LITERAL}: #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true}>
// MFMA32{LITERAL}: #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>
// MFMA32{LITERAL}: #mma1 = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 2], instrShape = [32, 32, 8], isTransposed = true}>
// CHECK-LABEL: mfma_chain_dot_BM64
// CHECK: tt.dot {{.*}} : {{.*}} -> tensor<64x16xf32, #mma>
// MFMA16: tt.dot {{.*}} : {{.*}} -> tensor<64x128xf32, #mma>
// MFMA32: tt.dot {{.*}} : {{.*}} -> tensor<64x128xf32, #mma1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_chain_dot_BM64(
      %q: tensor<64x128xf16, #dotOp0>,
      %k: tensor<128x16xf16, #dotOp1>,
      %v: tensor<16x128xf16, #dotOp1>,
      %o_ptr: tensor<64x128x!tt.ptr<f32>, #blocked>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<64x16xf32, #blocked>
    %cst1 = arith.constant dense<0.000000e+00> : tensor<64x128xf32, #blocked>
    %qk = tt.dot %q, %k, %cst : tensor<64x128xf16, #dotOp0> * tensor<128x16xf16, #dotOp1> -> tensor<64x16xf32, #blocked>
    %qk_f16 = arith.truncf %qk :  tensor<64x16xf32, #blocked> to tensor<64x16xf16, #blocked>
    %p = ttg.convert_layout %qk_f16 : tensor<64x16xf16, #blocked> -> tensor<64x16xf16, #dotOp0>
    %o = tt.dot %p, %v, %cst1 : tensor<64x16xf16, #dotOp0> * tensor<16x128xf16, #dotOp1> -> tensor<64x128xf32, #blocked>
    tt.store %o_ptr, %o : tensor<64x128x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_chain_dot_BM64`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_chain_dot_BM64`。

### Line 72
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 74-103
```mlir
// BLOCK_M = 32
// warpsPerCTA = [2, 2] for mfma16
// warpsPerCTA = [1, 4] for mfma32
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [16, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#dotOp0 = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
#dotOp1 = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
// MFMA16{LITERAL}: #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true}>
// MFMA32{LITERAL}: #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>
// MFMA16{LITERAL}: #mma1 = #ttg.amd_mfma<{version = 3, warpsPerCTA = [2, 2], instrShape = [16, 16, 16], isTransposed = true}>
// MFMA32{LITERAL}: #mma1 = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 4], instrShape = [32, 32, 8], isTransposed = true}>
// CHECK-LABEL: mfma_chain_dot_BM32
// CHECK: tt.dot {{.*}} : {{.*}} -> tensor<32x16xf32, #mma>
// MFMA16: tt.dot {{.*}} : {{.*}} -> tensor<32x128xf32, #mma1>
// MFMA32: tt.dot {{.*}} : {{.*}} -> tensor<32x128xf32, #mma1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_chain_dot_BM32(
      %q: tensor<32x128xf16, #dotOp0>,
      %k: tensor<128x16xf16, #dotOp1>,
      %v: tensor<16x128xf16, #dotOp1>,
      %o_ptr: tensor<32x128x!tt.ptr<f32>, #blocked>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x16xf32, #blocked>
    %cst1 = arith.constant dense<0.000000e+00> : tensor<32x128xf32, #blocked>
    %qk = tt.dot %q, %k, %cst : tensor<32x128xf16, #dotOp0> * tensor<128x16xf16, #dotOp1> -> tensor<32x16xf32, #blocked>
    %qk_f16 = arith.truncf %qk :  tensor<32x16xf32, #blocked> to tensor<32x16xf16, #blocked>
    %p = ttg.convert_layout %qk_f16 : tensor<32x16xf16, #blocked> -> tensor<32x16xf16, #dotOp0>
    %o = tt.dot %p, %v, %cst1 : tensor<32x16xf16, #dotOp0> * tensor<16x128xf16, #dotOp1> -> tensor<32x128xf32, #blocked>
    tt.store %o_ptr, %o : tensor<32x128x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_chain_dot_BM32`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_chain_dot_BM32`。

### Line 106
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 108-133
```mlir
// BLOCK_M = 16, only check mfma16 since it's too small for mfma32
// warpsPerCTA = [1, 4] for mfma16
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [16, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#dotOp0 = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
#dotOp1 = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
// MFMA16{LITERAL}: #mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true}>
// MFMA16{LITERAL}: #mma1 = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 4], instrShape = [16, 16, 16], isTransposed = true}>
// CHECK-LABEL: mfma_chain_dot_BM16
// CHECK: tt.dot {{.*}} : {{.*}} -> tensor<16x16xf32, #mma>
// MFMA16: tt.dot {{.*}} : {{.*}} -> tensor<16x128xf32, #mma1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_chain_dot_BM16(
      %q: tensor<16x128xf16, #dotOp0>,
      %k: tensor<128x16xf16, #dotOp1>,
      %v: tensor<16x128xf16, #dotOp1>,
      %o_ptr: tensor<16x128x!tt.ptr<f32>, #blocked>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #blocked>
    %cst1 = arith.constant dense<0.000000e+00> : tensor<16x128xf32, #blocked>
    %qk = tt.dot %q, %k, %cst : tensor<16x128xf16, #dotOp0> * tensor<128x16xf16, #dotOp1> -> tensor<16x16xf32, #blocked>
    %qk_f16 = arith.truncf %qk :  tensor<16x16xf32, #blocked> to tensor<16x16xf16, #blocked>
    %p = ttg.convert_layout %qk_f16 : tensor<16x16xf16, #blocked> -> tensor<16x16xf16, #dotOp0>
    %o = tt.dot %p, %v, %cst1 : tensor<16x16xf16, #dotOp0> * tensor<16x128xf16, #dotOp1> -> tensor<16x128xf32, #blocked>
    tt.store %o_ptr, %o : tensor<16x128x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_chain_dot_BM16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_chain_dot_BM16`。

### Line 136
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 138-139
```mlir
// Check kWidth of both operands of the 2nd dot. To avoid in-warp shuffle for
// the layout conversion from #mma to #dotOp, kWidth should be set to 4
```
**EN:** This comment block provides context for the surrounding test logic: `// Check kWidth of both operands of the 2nd dot. To avoid in-warp shuffle for`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Check kWidth of both operands of the 2nd dot. To avoid in-warp shuffle for`。

### Lines 141-161
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [16, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#dotOp0 = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
#dotOp1 = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
// CHECK-LABEL: mfma_chain_dot_kWidth_f16
// CHECK-GFX950: tt.dot {{.*}} : {{.*}} -> tensor<128x128xf32, #mma>
// CHECK-GFX950: tt.dot {{.*}} : tensor<128x128xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<128x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> {{.*}}
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_chain_dot_kWidth_f16(
      %q: tensor<128x128xf16, #dotOp0>,
      %k: tensor<128x128xf16, #dotOp1>,
      %v: tensor<128x128xf16, #dotOp1>,
      %o_ptr: tensor<128x128x!tt.ptr<f32>, #blocked>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %qk = tt.dot %q, %k, %cst : tensor<128x128xf16, #dotOp0> * tensor<128x128xf16, #dotOp1> -> tensor<128x128xf32, #blocked>
    %qk_f16 = arith.truncf %qk :  tensor<128x128xf32, #blocked> to tensor<128x128xf16, #blocked>
    %p = ttg.convert_layout %qk_f16 : tensor<128x128xf16, #blocked> -> tensor<128x128xf16, #dotOp0>
    %o = tt.dot %p, %v, %cst : tensor<128x128xf16, #dotOp0> * tensor<128x128xf16, #dotOp1> -> tensor<128x128xf32, #blocked>
    tt.store %o_ptr, %o : tensor<128x128x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_chain_dot_kWidth_f16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_chain_dot_kWidth_f16`。

### Line 163
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 165-185
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [16, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#dotOp0 = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
#dotOp1 = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
// CHECK-LABEL: mfma_chain_dot_kWidth_bf16
// CHECK-GFX950: tt.dot {{.*}} : {{.*}} -> tensor<128x128xf32, #mma>
// CHECK-GFX950: tt.dot {{.*}} : tensor<128x128xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<128x128xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> {{.*}}
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_chain_dot_kWidth_bf16(
      %q: tensor<128x128xbf16, #dotOp0>,
      %k: tensor<128x128xbf16, #dotOp1>,
      %v: tensor<128x128xbf16, #dotOp1>,
      %o_ptr: tensor<128x128x!tt.ptr<f32>, #blocked>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %qk = tt.dot %q, %k, %cst : tensor<128x128xbf16, #dotOp0> * tensor<128x128xbf16, #dotOp1> -> tensor<128x128xf32, #blocked>
    %qk_bf16 = arith.truncf %qk :  tensor<128x128xf32, #blocked> to tensor<128x128xbf16, #blocked>
    %p = ttg.convert_layout %qk_bf16 : tensor<128x128xbf16, #blocked> -> tensor<128x128xbf16, #dotOp0>
    %o = tt.dot %p, %v, %cst : tensor<128x128xbf16, #dotOp0> * tensor<128x128xbf16, #dotOp1> -> tensor<128x128xf32, #blocked>
    tt.store %o_ptr, %o : tensor<128x128x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_chain_dot_kWidth_bf16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_chain_dot_kWidth_bf16`。

### Line 188
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 190-191
```mlir
// Cross-iteration chain-dot: in pipelined FA the QK dot result is yielded and
// consumed as operand A of the PV dot on the next iteration.
```
**EN:** This comment block provides context for the surrounding test logic: `// Cross-iteration chain-dot: in pipelined FA the QK dot result is yielded and`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Cross-iteration chain-dot: in pipelined FA the QK dot result is yielded and`。

### Lines 193-218
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [16, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#dotOp0 = #ttg.dot_op<{opIdx = 0, parent = #blocked}>
#dotOp1 = #ttg.dot_op<{opIdx = 1, parent = #blocked}>
// CHECK-GFX950-LABEL: cross_iter_chain_dot_fa
// CHECK-GFX950: tt.dot {{.*}} : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<128x128xf32, #mma>
// CHECK-GFX950: tt.dot {{.*}} -> tensor<128x64xf32, #mma>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @cross_iter_chain_dot_fa(
      %q: tensor<128x128xf16, #dotOp0>,
      %k: tensor<128x64xf16, #dotOp1>,
      %v: tensor<64x128xf16, #dotOp1>,
      %acc_init: tensor<128x128xf32, #blocked>,
      %p_init: tensor<128x64xf16, #dotOp0>,
      %lb: index, %ub: index, %step: index) -> tensor<128x128xf32, #blocked> {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #blocked>
    %result:2 = scf.for %iv = %lb to %ub step %step
        iter_args(%acc = %acc_init, %p_prev = %p_init) -> (tensor<128x128xf32, #blocked>, tensor<128x64xf16, #dotOp0>) {
      %pv = tt.dot %p_prev, %v, %acc : tensor<128x64xf16, #dotOp0> * tensor<64x128xf16, #dotOp1> -> tensor<128x128xf32, #blocked>
      %qk = tt.dot %q, %k, %cst : tensor<128x128xf16, #dotOp0> * tensor<128x64xf16, #dotOp1> -> tensor<128x64xf32, #blocked>
      %qk_f16 = arith.truncf %qk : tensor<128x64xf32, #blocked> to tensor<128x64xf16, #blocked>
      %p_next = ttg.convert_layout %qk_f16 : tensor<128x64xf16, #blocked> -> tensor<128x64xf16, #dotOp0>
      scf.yield %pv, %p_next : tensor<128x128xf32, #blocked>, tensor<128x64xf16, #dotOp0>
    }
    tt.return %result#0 : tensor<128x128xf32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cross_iter_chain_dot_fa`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cross_iter_chain_dot_fa`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's TritonGPU transforms coverage and focuses on matmul acceleration.  
  **CN:** 该文件属于 Triton 的 TritonGPU 变换 测试覆盖，关注点是 矩阵乘加速。
- **EN:** The `RUN` pipeline drives tools/passes such as `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx942`, `--check-prefixes`, `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx950`.  
  **CN:** `RUN` 流水线会驱动 `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx942`, `--check-prefixes`, `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx950` 等工具/pass。
- **EN:** The IR mixes `tt` (Triton core ops), `ttg` (TritonGPU ops/layouts), `arith` (scalar/tensor arithmetic), `scf` (structured control flow).  
  **CN:** 该 IR 混合使用了 `tt`（Triton 核心操作）、`ttg`（TritonGPU 操作/布局）、`arith`（标量/张量算术）、`scf`（结构化控制流）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `tt`, `ttg`, `arith`, `scf`.  
  **CN:** IR 方言依赖：`tt`、`ttg`、`arith`、`scf`。
