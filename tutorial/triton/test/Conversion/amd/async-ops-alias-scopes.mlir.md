# async-ops-alias-scopes.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/async-ops-alias-scopes.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises alias analysis in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的别名分析相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --convert-scf-to-cf | FileCheck %s --check-prefixes=COMMON,GFX950`<br>`triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-scf-to-cf | FileCheck %s --check-prefixes=COMMON,GFX942` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Validation / 验证内容:** **EN:** The file validates that alias analysis runs successfully under the specified pass pipeline. **CN:** 该文件验证 别名分析 能在指定的 pass 流水线下成功运行。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --convert-scf-to-cf | FileCheck %s --check-prefixes=COMMON,GFX950
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-scf-to-cf | FileCheck %s --check-prefixes=COMMON,GFX942
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --convert-scf-to-cf | FileCheck %s --check-prefixes=COMMON,GFX950; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-scf-to-cf | FileCheck %s --check-prefixes=COMMON,GFX942` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --convert-scf-to-cf | FileCheck %s --check-prefixes=COMMON,GFX950; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-scf-to-cf | FileCheck %s --check-prefixes=COMMON,GFX942`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-17
```mlir
// COMMON: [[$ASYNC_COPY_SCOPE:#.*]] = #llvm.alias_scope<id = "amdg.AsyncCopies"
// COMMON: [[$LOCAL_LOAD_SCOPE:#.*]] = #llvm.alias_scope<id = "amdg.LocalLoads"
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [64, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [0, 1]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: @async_copy_alias
  tt.func public @async_copy_alias(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                   %arg1: !ttg.memdesc<64x1xf32, #shared, #smem, mutable>,
                                   %maskVal: i1) {
    %other = arith.constant dense<1.000000e+00> : tensor<64x1xf32, #blocked>
    // We need the splat to allow the AxisAnalysis to work during lowering
    %ptr = tt.splat %arg0 : !tt.ptr<f32> -> tensor<64x1x!tt.ptr<f32>, #blocked>
    %mask = tt.splat %maskVal : i1 -> tensor<64x1xi1, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_alias`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_alias`。

### Lines 19-22
```mlir
    // COMMON: rocdl.global.load.async.lds {{.*}} {alias_scopes = [[[$ASYNC_COPY_SCOPE]]]
    // Check that store for 'other' has alias information set
    // COMMON: llvm.store {{.*}} {alias_scopes = [[[$LOCAL_LOAD_SCOPE]]], {{.*}}, noalias_scopes = [[[$ASYNC_COPY_SCOPE]]]
    %0 = ttg.async_copy_global_to_local %ptr, %arg1 mask %mask other %other : tensor<64x1x!tt.ptr<f32>, #blocked> -> <64x1xf32, #shared, #smem, mutable>
```
**EN:** This block contributes intermediate IR built from `rocdl.global.load.async.lds`, `llvm.store`, `ttg.async_copy_global_to_local`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.global.load.async.lds`, `llvm.store`, `ttg.async_copy_global_to_local`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 24-27
```mlir
    // COMMON: llvm.return
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `llvm.return`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.return`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 29
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 31-42
```mlir
// COMMON: [[$ASYNC_COPY_SCOPE:#.*]] = #llvm.alias_scope<id = "amdg.AsyncCopies"
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: @buffer_load_to_local_alias
  tt.func public @buffer_load_to_local_alias(%maskVal: i1,
                                             %arg1: !tt.ptr<f32>,
                                             %arg2: tensor<8x64xi32, #blocked>,
                                             %arg3: !ttg.memdesc<8x64xf32, #shared, #smem, mutable>) {
    %mask = tt.splat %maskVal : i1 -> tensor<8x64xi1, #blocked>
    %other = arith.constant dense<1.000000e+00> : tensor<8x64xf32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_to_local_alias`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_to_local_alias`。

### Lines 44-47
```mlir
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds {{.*}} {alias_scopes = [[[$ASYNC_COPY_SCOPE]]]
    // Check that store for 'other' has alias information set
    // COMMON: llvm.store {{.*}} {alias_scopes = [[[$LOCAL_LOAD_SCOPE]]], {{.*}}, noalias_scopes = [[[$ASYNC_COPY_SCOPE]]]
    %65 = amdg.buffer_load_to_local %arg1[%arg2] mask=%mask other=%other into %arg3 : <f32>[tensor<8x64xi32, #blocked>] tensor<8x64xf32, #blocked> -> <8x64xf32, #shared, #smem, mutable>
```
**EN:** This block contributes intermediate IR built from `rocdl.raw.ptr.buffer.load.async.lds`, `llvm.store`, `amdg.buffer_load_to_local`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.raw.ptr.buffer.load.async.lds`, `llvm.store`, `amdg.buffer_load_to_local` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 49-52
```mlir
    // COMMON: llvm.return
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `llvm.return`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.return`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 54
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 56-68
```mlir
// COMMON: [[$LOCAL_LOAD_SCOPE:#.*]] = #llvm.alias_scope<id = "amdg.LocalLoads"
// COMMON: [[$ASYNC_COPY_SCOPE:#.*]] = #llvm.alias_scope<id = "amdg.AsyncCopies"
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [64, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [0, 1]}>
#shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
#smem = #ttg.shared_memory
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 1], instrShape = [32, 32, 16], isTransposed = true}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: @local_loads_with_token_from_async_wait
  tt.func public @local_loads_with_token_from_async_wait(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                                         %arg1: !ttg.memdesc<64x1xf16, #shared, #smem, mutable>,
                                                         %arg2: !ttg.memdesc<16x16xf16, #shared, #smem, mutable>) {
    %3 = amdg.async_wait {num_inst = 1 : i32}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `local_loads_with_token_from_async_wait`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `local_loads_with_token_from_async_wait`。

### Line 70
```mlir
    // Check alias information is added for different lowering paths
```
**EN:** This comment block provides context for the surrounding test logic: `// Check alias information is added for different lowering paths`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Check alias information is added for different lowering paths`。

### Lines 72-74
```mlir
    // Test lowering path in common MemoryOpToLLVM pattern
    // COMMON: llvm.load {{.*}} {alias_scopes = [[[$LOCAL_LOAD_SCOPE]]], noalias_scopes = [[[$ASYNC_COPY_SCOPE]]]
    %4 = ttg.local_load %arg1 token %3 : !ttg.memdesc<64x1xf16, #shared, #smem, mutable> -> tensor<64x1xf16, #blocked>
```
**EN:** This block contributes intermediate IR built from `llvm.load`, `ttg.local_load`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.load`, `ttg.local_load`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 76-79
```mlir
    // Test lowering path in AMD's MemoryOpToLLVM pattern
    // GFX942: llvm.load {{.*}} {alias_scopes = [[[$LOCAL_LOAD_SCOPE]]], noalias_scopes = [[[$ASYNC_COPY_SCOPE]]]
    // GFX950: rocdl.ds.read.tr16.b64 {{.*}} {alias_scopes = [[[$LOCAL_LOAD_SCOPE]]], noalias_scopes = [[[$ASYNC_COPY_SCOPE]]]
    %5 = ttg.local_load %arg2 token %3 : !ttg.memdesc<16x16xf16, #shared, #smem, mutable> -> tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
```
**EN:** This block contributes intermediate IR built from `llvm.load`, `rocdl.ds.read.tr16.b64`, `ttg.local_load`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.load`, `rocdl.ds.read.tr16.b64`, `ttg.local_load`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 81-85
```mlir
    // Stores to keep the local_loads
    %ptr = tt.splat %arg0 : !tt.ptr<f16> -> tensor<64x1x!tt.ptr<f16>, #blocked>
    tt.store %ptr, %4 : tensor<64x1x!tt.ptr<f16>, #blocked>
    %ptr2 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<16x16x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    tt.store %ptr2, %5 : tensor<16x16x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 87-90
```mlir
    // COMMON: llvm.return
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `llvm.return`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.return`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 92
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 94
```mlir
// Same as above but LocalLoad does not use the token from AsyncWait
```
**EN:** This comment block provides context for the surrounding test logic: `// Same as above but LocalLoad does not use the token from AsyncWait`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Same as above but LocalLoad does not use the token from AsyncWait`。

### Lines 96-108
```mlir
// COMMON: [[$ASYNC_COPY_SCOPE:#.*]] = #llvm.alias_scope<id = "amdg.AsyncCopies"
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [64, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [0, 1]}>
#shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
#smem = #ttg.shared_memory
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 1], instrShape = [32, 32, 16], isTransposed = true}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: @local_loads_without_token_from_async_wait
  tt.func public @local_loads_without_token_from_async_wait(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                                            %arg1: !ttg.memdesc<64x1xf32, #shared, #smem, mutable>,
                                                            %arg4: !ttg.memdesc<16x16xf32, #shared, #smem, mutable>) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %ptr = tt.splat %arg0 : !tt.ptr<f32> -> tensor<64x1x!tt.ptr<f32>, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `local_loads_without_token_from_async_wait`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `local_loads_without_token_from_async_wait`。

### Lines 110-112
```mlir
    // COMMON: rocdl.global.load.async.lds {{.*}} {alias_scopes = [[[$ASYNC_COPY_SCOPE]]]
    %0 = ttg.async_copy_global_to_local %ptr, %arg1 : tensor<64x1x!tt.ptr<f32>, #blocked> -> <64x1xf32, #shared, #smem, mutable>
    %1 = ttg.async_commit_group tokens %0
```
**EN:** This block contributes intermediate IR built from `rocdl.global.load.async.lds`, `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.async_commit_group`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.global.load.async.lds`, `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.async_commit_group` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 114
```mlir
    %3 = amdg.async_wait %1 {num_inst = 1 : i32}
```
**EN:** This block contributes intermediate IR built from `amdg.async_wait`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `amdg.async_wait` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 116-117
```mlir
    // Check alias information is not used at all for different lowering paths
    // COMMON-NOT: [[$ASYNC_COPY_SCOPE]]
```
**EN:** This comment block provides context for the surrounding test logic: `// Check alias information is not used at all for different lowering paths`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Check alias information is not used at all for different lowering paths`。

### Lines 119-121
```mlir
    // Test lowering path in common MemoryOpToLLVM pattern
    %4 = ttg.local_load %arg1 token %0 : !ttg.memdesc<64x1xf32, #shared, #smem, mutable> -> tensor<64x1xf32, #blocked>
    %5 = ttg.local_load %arg1 : !ttg.memdesc<64x1xf32, #shared, #smem, mutable> -> tensor<64x1xf32, #blocked>
```
**EN:** This block contributes intermediate IR built from `ttg.local_load`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_load`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 123-125
```mlir
    // Test lowering path in AMD's MemoryOpToLLVM pattern
    %7 = ttg.local_load %arg4 token %0 : !ttg.memdesc<16x16xf32, #shared, #smem, mutable> -> tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    %8 = ttg.local_load %arg4 : !ttg.memdesc<16x16xf32, #shared, #smem, mutable> -> tensor<16x16xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
```
**EN:** This block contributes intermediate IR built from `ttg.local_load`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_load`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 127-130
```mlir
    // COMMON: llvm.return
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `llvm.return`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.return`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 132
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 134-147
```mlir
// COMMON: [[$LOCAL_LOAD_SCOPE:#.*]] = #llvm.alias_scope<id = "amdg.LocalLoads"
// COMMON: [[$ASYNC_COPY_SCOPE:#.*]] = #llvm.alias_scope<id = "amdg.AsyncCopies"
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [64, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 8, maxPhase = 2, order = [0, 1]}>
#shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
#smem = #ttg.shared_memory
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [1, 1], instrShape = [32, 32, 16], isTransposed = true}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: @local_loads_with_loop_carried_token
  tt.func public @local_loads_with_loop_carried_token(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                                         %arg1: !ttg.memdesc<64x1xf16, #shared, #smem, mutable>,
                                                         %loopIterCount: i32) {
    %c0_i32 = arith.constant 0 : i32
    %c1_i32 = arith.constant 1 : i32
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `local_loads_with_loop_carried_token`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `local_loads_with_loop_carried_token`。

### Lines 149-151
```mlir
    %1 = amdg.async_wait {num_inst = 1 : i32}
    // COMMON: llvm.load
    %2 = ttg.local_load %arg1 token %1 : !ttg.memdesc<64x1xf16, #shared, #smem, mutable> -> tensor<64x1xf16, #blocked>
```
**EN:** This block contributes intermediate IR built from `amdg.async_wait`, `llvm.load`, `ttg.local_load`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `amdg.async_wait`, `llvm.load`, `ttg.local_load`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 153-158
```mlir
    %loop_result:2 = scf.for %arg14 = %c0_i32 to %loopIterCount step %c1_i32 iter_args(%arg10 = %1, %arg11 = %2) -> (!ttg.async.token, tensor<64x1xf16, #blocked>)  : i32 {
      // COMMON: llvm.load {{.*}} {alias_scopes = [[[$LOCAL_LOAD_SCOPE]]], noalias_scopes = [[[$ASYNC_COPY_SCOPE]]]
      %3 = ttg.local_load %arg1 token %arg10 : !ttg.memdesc<64x1xf16, #shared, #smem, mutable> -> tensor<64x1xf16, #blocked>
      %4 = amdg.async_wait {num_inst = 1 : i32}
      scf.yield %4, %3: !ttg.async.token, tensor<64x1xf16, #blocked>
    }
```
**EN:** This block contributes intermediate IR built from `scf.for`, `ttg.async.token`, `llvm.load`, `ttg.local_load`, `ttg.memdesc`, `amdg.async_wait`, `scf.yield`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `scf.for`, `ttg.async.token`, `llvm.load`, `ttg.local_load`, `ttg.memdesc`, `amdg.async_wait`, `scf.yield` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 160-162
```mlir
    // Stores to keep the local_loads
    %ptr = tt.splat %arg0 : !tt.ptr<f16> -> tensor<64x1x!tt.ptr<f16>, #blocked>
    tt.store %ptr, %loop_result#1 : tensor<64x1x!tt.ptr<f16>, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 164-167
```mlir
    // COMMON: llvm.return
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `llvm.return`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.return`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on alias analysis.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 别名分析。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--convert-scf-to-cf`, `--check-prefixes=COMMON,GFX950`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--check-prefixes=COMMON,GFX942`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--convert-scf-to-cf`, `--check-prefixes=COMMON,GFX950`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--check-prefixes=COMMON,GFX942` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `llvm` (LLVM dialect IR), `scf` (structured control flow).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`llvm`（LLVM 方言 IR）、`scf`（结构化控制流）。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `llvm`, `scf`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`llvm`、`scf`。
