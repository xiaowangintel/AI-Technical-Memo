# load_store.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/load_store.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises load/store lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的load/store 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK-COUNT×3, CHECK-LABEL×2; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK-COUNT×3, CHECK-LABEL×2；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that load/store lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 load/store 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-29
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: global_load_store_vec8
    tt.func @global_load_store_vec8(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32) {
    %c256_i32 = arith.constant 256 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c256_i32 : i32
    %2 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
    %3 = tt.splat %1 : i32 -> tensor<256xi32, #blocked0>
    %4 = arith.addi %3, %2 : tensor<256xi32, #blocked0>
    %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %6 = tt.addptr %5, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
    %7 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %8 = tt.addptr %7, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
    // Load 8 elements from A with two vectorized load instruction
    // CHECK-COUNT-2: llvm.load {{.*}} : !llvm.ptr<1> -> vector<4xf32>
    %9 = tt.load %6 {cache = 1 : i32, evict = 1 : i32, isVolatile = false} : tensor<256x!tt.ptr<f32>, #blocked0>
    // Load 8 elements from B with two vectorized load instruction
    // CHECK-COUNT-2: llvm.load {{.*}} : !llvm.ptr<1> -> vector<4xf32>
    %10 = tt.load %8 {cache = 1 : i32, evict = 1 : i32, isVolatile = false} : tensor<256x!tt.ptr<f32>, #blocked0>
    %11 = arith.addf %9, %10 : tensor<256xf32, #blocked0>
    %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %13 = tt.addptr %12, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
    tt.store %13, %11 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `global_load_store_vec8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `global_load_store_vec8`。

### Line 31
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 33-58
```mlir
#mma = #ttg.amd_mfma<{version = 3, warpsPerCTA = [1, 1], instrShape = [16, 16, 4], isTransposed = true}>
module attributes {"ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: global_store_mfma_vec16
  tt.func public @global_store_mfma_vec16(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
    %cst_0 = arith.constant dense<1.230000e+02> : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
    %cst_1 = arith.constant dense<1.230000e+02> : tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
    %0 = tt.dot %cst_0, %cst_1, %cst : tensor<32x32xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x32xf32, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<32x32xf32, #mma>
    %1 = math.exp2 %0 : tensor<32x32xf32, #mma>
    %2 = arith.truncf %1 : tensor<32x32xf32, #mma> to tensor<32x32xf16, #mma>
    %c32_i32 = arith.constant 32 : i32
    %100 = tt.get_program_id x : i32
    %101 = arith.muli %100, %c32_i32 : i32
    %102 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #mma}>>
    %300 = tt.expand_dims %102 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #mma}>> -> tensor<1x32xi32, #mma>
    %200 = tt.broadcast %300 : tensor<1x32xi32, #mma> -> tensor<32x32xi32, #mma>
    %103 = tt.splat %101 : i32 -> tensor<32x32xi32, #mma>
    %104 = arith.addi %103, %200 : tensor<32x32xi32, #mma>
    %105 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<32x32x!tt.ptr<f16>, #mma>
    %106 = tt.addptr %105, %104 : tensor<32x32x!tt.ptr<f16>, #mma>, tensor<32x32xi32, #mma>
    // Store 16 elements with four vectorized store instruction
    // CHECK-COUNT-4: llvm.store {{.*}} : vector<4xf16>, !llvm.ptr<1>
    tt.store %106, %2 : tensor<32x32x!tt.ptr<f16>, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `global_store_mfma_vec16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `global_store_mfma_vec16`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on load/store lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 load/store 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--convert-builtin-func-to-llvm`.  
  **CN:** `RUN` 流水线会驱动 `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--convert-builtin-func-to-llvm` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `llvm` (LLVM dialect IR), `math` (math library ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`llvm`（LLVM 方言 IR）、`math`（数学库操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `llvm`, `math`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`llvm`、`math`。
