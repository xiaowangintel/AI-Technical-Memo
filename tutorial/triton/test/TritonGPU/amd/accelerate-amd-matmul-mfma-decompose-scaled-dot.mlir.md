# accelerate-amd-matmul-mfma-decompose-scaled-dot.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/TritonGPU/amd/accelerate-amd-matmul-mfma-decompose-scaled-dot.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises MFMA lowering in Triton's TritonGPU transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton TritonGPU 变换中的MFMA 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=0" -tritongpu-remove-layout-conversions | FileCheck %s --check-prefixes CHECK` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×32, CHECK-LABEL×3; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×32, CHECK-LABEL×3；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that MFMA lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 MFMA 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=0" -tritongpu-remove-layout-conversions | FileCheck %s --check-prefixes CHECK
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=0" -tritongpu-remove-layout-conversions | FileCheck %s --check-prefixes CHECK` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --tritonamdgpu-accelerate-matmul="gfx-arch=gfx950 matrix-instruction-size=0" -tritongpu-remove-layout-conversions | FileCheck %s --check-prefixes CHECK`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-37
```mlir
// CHECK-LABEL: mfma_dot_scaled_bf16_fp8e4
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_dot_scaled_bf16_fp8e4(
      %arg0: tensor<32x64x!tt.ptr<bf16>, #blocked2>,
      %arg1: tensor<64x32x!tt.ptr<f8E4M3FN>, #blocked>,
      %arg2: tensor<32x2x!tt.ptr<i8>, #blocked1>,
      %arg3: tensor<32x32x!tt.ptr<f32>, #blocked>
    ) {
    // CHECK: %[[CST:.*]] = arith.constant dense<7> : tensor<2x32xi16, #ttg.slice<{dim = 2, parent = #linear{{.*}}}>>
    // CHECK: %[[B:.*]] = ttg.convert_layout %{{.*}} : tensor<64x32xf8E4M3FN, #blocked{{.*}}> -> tensor<64x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    // CHECK: %[[S:.*]] = ttg.convert_layout %{{.*}} : tensor<32x2xi8, #blocked{{.*}}> -> tensor<32x2xi8, #linear{{.*}}>
    // CHECK: %[[TS:.*]] = tt.trans %[[S]] {order = array<i32: 1, 0>}
    // CHECK: %[[ES:.*]] = arith.extui %[[TS]]
    // CHECK: %[[SHS:.*]] = arith.shli %[[ES]], %[[CST]]
    // CHECK: %[[BS:.*]] = tt.bitcast %[[SHS]] : tensor<2x32xi16, #ttg.slice<{dim = 2, parent = #linear{{.*}}}>> -> tensor<2x32xbf16, #ttg.slice<{dim = 2, parent = #linear{{.*}}}>>
    // CHECK: %[[EPS:.*]] = tt.expand_dims %[[BS]] {axis = 2 : i32} : tensor<2x32xbf16, #ttg.slice<{dim = 2, parent = #linear{{.*}}}>> -> tensor<2x32x1xbf16, #linear{{.*}}>
    // CHECK: %[[BCS:.*]] = tt.broadcast %[[EPS]] : tensor<2x32x1xbf16, #linear{{.*}}> -> tensor<2x32x32xbf16, #linear{{.*}}>
    // CHECK: %[[TBCS:.*]] = tt.trans %[[BCS]] {order = array<i32: 0, 2, 1>} : tensor<2x32x32xbf16, #linear{{.*}}> -> tensor<2x32x32xbf16, #linear{{.*}}>
    // CHECK: %[[RTBCS:.*]] = tt.reshape %[[TBCS]] : tensor<2x32x32xbf16, #linear{{.*}}> -> tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    // CHECK: %[[UB:.*]] = amdg.scaled_upcast_fp8 %[[B]] scale %[[RTBCS]] : tensor<64x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>, tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    // CHECK: %[[SELECTEDB:.*]] = arith.select %{{.*}}, %{{.*}}, %[[UB]] : tensor<64x32xi1, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>, tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    // CHECK: %[[A:.*]] = ttg.convert_layout %{{.*}} : tensor<32x64xbf16, #blocked{{.*}}> -> tensor<32x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    // CHECK: %{{.*}} = tt.dot %[[A]], %[[SELECTEDB]], %{{.*}} : tensor<32x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<32x32xf32, #mma>
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    %1 = tt.load %arg0 : tensor<32x64x!tt.ptr<bf16>, #blocked2>
    %2 = tt.load %arg1 : tensor<64x32x!tt.ptr<f8E4M3FN>, #blocked>
    %3 = tt.load %arg2 : tensor<32x2x!tt.ptr<i8>, #blocked1>
    %4 = tt.dot_scaled %1, %2 scale %3, %cst lhs = bf16 rhs = e4m3 {fastMath = false} : tensor<32x64xbf16, #blocked2> * tensor<64x32xf8E4M3FN, #blocked>, tensor<32x2xi8, #blocked1> -> tensor<32x32xf32, #blocked>
    tt.store %arg3, %4 : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_dot_scaled_bf16_fp8e4`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_dot_scaled_bf16_fp8e4`。

### Line 39
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 41-62
```mlir
// CHECK-LABEL: mfma_dot_scaled_bf16_fp8e4_fast_math
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_dot_scaled_bf16_fp8e4_fast_math(
      %arg0: tensor<32x64x!tt.ptr<bf16>, #blocked2>,
      %arg1: tensor<64x32x!tt.ptr<f8E4M3FN>, #blocked>,
      %arg2: tensor<32x2x!tt.ptr<i8>, #blocked1>,
      %arg3: tensor<32x32x!tt.ptr<f32>, #blocked>
    ) {
    // CHECK: %[[UB:.*]] = amdg.scaled_upcast_fp8 %{{.*}} scale %{{.*}} : tensor<64x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>, tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    // CHECK: %{{.*}} = tt.dot %{{.*}}, %[[UB]], %{{.*}} : tensor<32x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<32x32xf32, #mma>
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    %1 = tt.load %arg0 : tensor<32x64x!tt.ptr<bf16>, #blocked2>
    %2 = tt.load %arg1 : tensor<64x32x!tt.ptr<f8E4M3FN>, #blocked>
    %3 = tt.load %arg2 : tensor<32x2x!tt.ptr<i8>, #blocked1>
    %4 = tt.dot_scaled %1, %2 scale %3, %cst lhs = bf16 rhs = e4m3 {fastMath = true} : tensor<32x64xbf16, #blocked2> * tensor<64x32xf8E4M3FN, #blocked>, tensor<32x2xi8, #blocked1> -> tensor<32x32xf32, #blocked>
    tt.store %arg3, %4 : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_dot_scaled_bf16_fp8e4_fast_math`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_dot_scaled_bf16_fp8e4_fast_math`。

### Line 64
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 66-99
```mlir
// CHECK-LABEL: mfma_dot_scaled_bf16_fp4
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @mfma_dot_scaled_bf16_fp4(
      %arg0: tensor<32x64x!tt.ptr<bf16>, #blocked2>,
      %arg1: tensor<32x32x!tt.ptr<i8>, #blocked>,
      %arg2: tensor<32x2x!tt.ptr<i8>, #blocked1>,
      %arg3: tensor<32x32x!tt.ptr<f32>, #blocked>
    ) {
    // CHECK: %[[CST:.*]] = arith.constant dense<7> : tensor<2x32xi16, #ttg.slice<{dim = 2, parent = #linear{{.*}}}>>
    // CHECK: %[[B:.*]] = ttg.convert_layout %{{.*}} : tensor<32x32xi8, #blocked{{.*}}> -> tensor<32x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
    // CHECK: %[[S:.*]] = ttg.convert_layout %{{.*}} : tensor<32x2xi8, #blocked{{.*}}> -> tensor<32x2xi8, #linear{{.*}}>
    // CHECK: %[[TS:.*]] = tt.trans %[[S]] {order = array<i32: 1, 0>}
    // CHECK: %[[ES:.*]] = arith.extui %[[TS]]
    // CHECK: %[[SHS:.*]] = arith.shli %[[ES]], %[[CST]]
    // CHECK: %[[BS:.*]] = tt.bitcast %[[SHS]] : tensor<2x32xi16, #ttg.slice<{dim = 2, parent = #linear{{.*}}}>> -> tensor<2x32xbf16, #ttg.slice<{dim = 2, parent = #linear{{.*}}}>>
    // CHECK: %[[EPS:.*]] = tt.expand_dims %[[BS]] {axis = 2 : i32} : tensor<2x32xbf16, #ttg.slice<{dim = 2, parent = #linear{{.*}}}>> -> tensor<2x32x1xbf16, #linear{{.*}}>
    // CHECK: %[[BCS:.*]] = tt.broadcast %[[EPS]] : tensor<2x32x1xbf16, #linear{{.*}}> -> tensor<2x32x32xbf16, #linear{{.*}}>
    // CHECK: %[[TBCS:.*]] = tt.trans %[[BCS]] {order = array<i32: 0, 2, 1>} : tensor<2x32x32xbf16, #linear{{.*}}> -> tensor<2x32x32xbf16, #linear{{.*}}>
    // CHECK: %[[RTBCS:.*]] = tt.reshape %[[TBCS]] : tensor<2x32x32xbf16, #linear{{.*}}> -> tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    // CHECK: %[[UB:.*]] = amdg.scaled_upcast_fp4 %[[B]] scale %[[RTBCS]] {axis = 0 : i32} : tensor<32x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>, tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
    // CHECK: %[[A:.*]] = ttg.convert_layout %{{.*}} : tensor<32x64xbf16, #blocked{{.*}}> -> tensor<32x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    // CHECK: %{{.*}} = tt.dot %[[A]], %[[UB]], %{{.*}} : tensor<32x64xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<64x32xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<32x32xf32, #mma>
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    %1 = tt.load %arg0 : tensor<32x64x!tt.ptr<bf16>, #blocked2>
    %2 = tt.load %arg1 : tensor<32x32x!tt.ptr<i8>, #blocked>
    %3 = tt.load %arg2 : tensor<32x2x!tt.ptr<i8>, #blocked1>
    %4 = tt.dot_scaled %1, %2 scale %3, %cst lhs = bf16 rhs = e2m1 {fastMath = true} : tensor<32x64xbf16, #blocked2> * tensor<32x32xi8, #blocked>, tensor<32x2xi8, #blocked1> -> tensor<32x32xf32, #blocked>
    tt.store %arg3, %4 : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mfma_dot_scaled_bf16_fp4`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mfma_dot_scaled_bf16_fp4`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's TritonGPU transforms coverage and focuses on MFMA lowering.  
  **CN:** 该文件属于 Triton 的 TritonGPU 变换 测试覆盖，关注点是 MFMA 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx950`, `-tritongpu-remove-layout-conversions`, `--check-prefixes`.  
  **CN:** `RUN` 流水线会驱动 `--tritonamdgpu-accelerate-matmul="gfx-arch=gfx950`, `-tritongpu-remove-layout-conversions`, `--check-prefixes` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`。
