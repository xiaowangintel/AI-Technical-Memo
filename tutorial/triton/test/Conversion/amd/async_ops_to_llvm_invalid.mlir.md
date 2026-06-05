# async_ops_to_llvm_invalid.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/async_ops_to_llvm_invalid.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises asynchronous copy/lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的异步拷贝/降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --verify-diagnostics`<br>`triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --verify-diagnostics` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK-LABEL×1; diagnostics annotations: error×4. **CN:** FileCheck 标记为 CHECK-LABEL×1；诊断标注为 error×4。
- **Validation / 验证内容:** **EN:** The file mainly validates that asynchronous copy/lowering rejects invalid inputs or reports the expected diagnostics. **CN:** 该文件主要验证 异步拷贝/降级 在遇到非法输入时会拒绝它们，或给出预期的诊断信息。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --verify-diagnostics
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --verify-diagnostics
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --verify-diagnostics; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --verify-diagnostics` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --verify-diagnostics; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --verify-diagnostics`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-17
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @async_copy_1_byte(%arg0: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<32x64xi8, #shared, #smem, mutable>) {
    %1 = tt.splat %arg0 : !tt.ptr<i8> -> tensor<32x64x!tt.ptr<i8>, #blocked>
    // AsyncCopyGlobalToLocal is only supported for >= 4 bytes
    // expected-error@+1 {{failed to legalize operation 'ttg.async_copy_global_to_local' that was explicitly marked illegal}}
    %2 = ttg.async_copy_global_to_local %1, %arg2 : tensor<32x64x!tt.ptr<i8>, #blocked> -> <32x64xi8, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_1_byte`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_1_byte`。

### Line 19
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 21-34
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @async_copy_2_bytes(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<32x64xf16, #shared, #smem, mutable>) {
    %1 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<32x64x!tt.ptr<f16>, #blocked>
    // AsyncCopyGlobalToLocal is only supported for >= 4 bytes
    // expected-error@+1 {{failed to legalize operation 'ttg.async_copy_global_to_local' that was explicitly marked illegal}}
    %2 = ttg.async_copy_global_to_local %1, %arg2 : tensor<32x64x!tt.ptr<f16>, #blocked> -> <32x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_2_bytes`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_2_bytes`。

### Line 36
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 38-51
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
// Padding interval of 1 forces vec==1 which we cannot lower because it's less than 32bits per lane
#shared = #ttg.padded_shared<[1:+2] {order = [1, 0], shape = [32, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @async_copy_padded_invalid_vec(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                    %arg1: i32 {tt.divisibility = 16 : i32},
                                    %arg2: !ttg.memdesc<32x64xf16, #shared, #smem, mutable>) {
    // We need the index calculation so AxisAnalysis sees that we can vectorize the load
    %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %2 = tt.expand_dims %1 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
    %3 = tt.broadcast %2 : tensor<1x64xi32, #blocked> -> tensor<32x64xi32, #blocked>
    %4 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<32x64x!tt.ptr<f16>, #blocked>
    %5 = tt.addptr %4, %3 : tensor<32x64x!tt.ptr<f16>, #blocked>, tensor<32x64xi32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_padded_invalid_vec`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_padded_invalid_vec`。

### Lines 53-57
```mlir
    // expected-error@+1 {{failed to legalize operation 'ttg.async_copy_global_to_local' that was explicitly marked illegal}}
    %6 = ttg.async_copy_global_to_local %5, %arg2 : tensor<32x64x!tt.ptr<f16>, #blocked> -> <32x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 59
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 61-76
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
// Padding interval of 16 cannot write warp coalesced since each warp writes at least 256 bytes (4bytes * 64 lanes)
#shared = #ttg.padded_shared<[16:+4] {order = [1, 0], shape = [32, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_copy_padded_too_small_interval
  tt.func public @async_copy_padded_too_small_interval(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<32x64xf32, #shared, #smem, mutable>) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x64x!tt.ptr<f32>, #blocked>
    // expected-error@+1 {{failed to legalize operation 'ttg.async_copy_global_to_local' that was explicitly marked illegal}}
    %2 = ttg.async_copy_global_to_local %1, %arg2 : tensor<32x64x!tt.ptr<f32>, #blocked> -> <32x64xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_padded_too_small_interval`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_padded_too_small_interval`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on asynchronous copy/lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 异步拷贝/降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--verify-diagnostics`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--verify-diagnostics`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。
- **EN:** Expected-diagnostic annotations make negative tests precise and reproducible.  
  **CN:** 期望诊断标注让负向测试更精确、可复现。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`.  
  **CN:** 外部测试工具：`triton-opt`。
- **EN:** IR dialect dependencies: `ttg`, `tt`.  
  **CN:** IR 方言依赖：`ttg`、`tt`。
