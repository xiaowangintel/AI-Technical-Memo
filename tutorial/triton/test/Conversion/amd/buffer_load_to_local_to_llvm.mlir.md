# buffer_load_to_local_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/buffer_load_to_local_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises buffer operations in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的缓冲区操作相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefixes=COMMON,GFX950`<br>`triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --verify-diagnostics | FileCheck %s --check-prefixes=COMMON,GFX942` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×7, CHECK-LABEL×2, CHECK-NEXT×1; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×7, CHECK-LABEL×2, CHECK-NEXT×1；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that buffer operations produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 缓冲区操作 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefixes=COMMON,GFX950
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --verify-diagnostics | FileCheck %s --check-prefixes=COMMON,GFX942
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefixes=COMMON,GFX950; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --verify-diagnostics | FileCheck %s --check-prefixes=COMMON,GFX942` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefixes=COMMON,GFX950; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --verify-diagnostics | FileCheck %s --check-prefixes=COMMON,GFX942`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-21
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: buffer_load_to_local_simple
  tt.func public @buffer_load_to_local_simple(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: !tt.ptr<f32>,
                                %arg2: tensor<32x64xi32, #blocked>,
                                %arg3: !ttg.memdesc<32x64xf32, #shared, #smem, mutable>) {
    // Each thread needs to load 8 elements and we load 1 (sizePerThread) per buffer load instruction
    // COMMON: rocdl.make.buffer.rsrc
    // COMMON-NOT: rocdl.make.buffer.rsrc
    // COMMON-COUNT-8: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON-NOT: rocdl.raw.ptr.buffer.load.async.lds
    %65 = amdg.buffer_load_to_local %arg1[%arg2] into %arg3 : <f32>[tensor<32x64xi32, #blocked>] -> <32x64xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_to_local_simple`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_to_local_simple`。

### Line 23
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 25-39
```mlir
#blocked = #ttg.blocked<{sizePerThread = [2, 1], threadsPerWarp = [32, 2], warpsPerCTA = [1, 32], order = [0, 1]}>
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 1, maxPhase = 1, order = [0, 1]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 32 : i32, ttg.shared = 0 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: buffer_load_to_local_vectorized_2xf16
  tt.func public @buffer_load_to_local_vectorized_2xf16(%arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: !ttg.memdesc<64x64xf16, #shared, #smem, mutable>) {
    %cst = arith.constant dense<64> : tensor<1x64xi32, #blocked>
    %0 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %2 = tt.expand_dims %0 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
    %3 = tt.broadcast %2 : tensor<64x1xi32, #blocked> -> tensor<64x64xi32, #blocked>
    %4 = tt.expand_dims %1 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
    %5 = arith.muli %4, %cst : tensor<1x64xi32, #blocked>
    %6 = tt.broadcast %5 : tensor<1x64xi32, #blocked> -> tensor<64x64xi32, #blocked>
    %7 = arith.addi %3, %6 : tensor<64x64xi32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_to_local_vectorized_2xf16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_to_local_vectorized_2xf16`。

### Lines 41-49
```mlir
    // Each thread needs to load 2 elements and we load 2 (sizePerThread) per buffer load instruction
    // COMMON: rocdl.make.buffer.rsrc
    // COMMON-NOT: rocdl.make.buffer.rsrc
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON-NOT: rocdl.raw.ptr.buffer.load.async.lds
    %8 = amdg.buffer_load_to_local %arg1[%7] into %arg2 : <f16>[tensor<64x64xi32, #blocked>]  -> <64x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `rocdl.make.buffer.rsrc`, `rocdl.raw.ptr.buffer.load.async.lds`, `amdg.buffer_load_to_local`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.make.buffer.rsrc`, `rocdl.raw.ptr.buffer.load.async.lds`, `amdg.buffer_load_to_local`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 51
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 53-67
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 32], order = [0, 1]}>
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 1, maxPhase = 1, order = [0, 1]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 32 : i32, ttg.shared = 0 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: buffer_load_to_local_vectorized_8xf16
  tt.func public @buffer_load_to_local_vectorized_8xf16(%arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: !ttg.memdesc<64x64xf16, #shared, #smem, mutable>) {
    %cst = arith.constant dense<64> : tensor<1x64xi32, #blocked>
    %0 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %2 = tt.expand_dims %0 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
    %3 = tt.broadcast %2 : tensor<64x1xi32, #blocked> -> tensor<64x64xi32, #blocked>
    %4 = tt.expand_dims %1 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
    %5 = arith.muli %4, %cst : tensor<1x64xi32, #blocked>
    %6 = tt.broadcast %5 : tensor<1x64xi32, #blocked> -> tensor<64x64xi32, #blocked>
    %7 = arith.addi %3, %6 : tensor<64x64xi32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_to_local_vectorized_8xf16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_to_local_vectorized_8xf16`。

### Lines 69-73
```mlir
    // Each thread needs to load 8 elements and we load 8 (sizePerThread) per buffer load instruction
    // GFX950: rocdl.make.buffer.rsrc
    // GFX950-NOT: rocdl.make.buffer.rsrc
    // GFX950: rocdl.raw.ptr.buffer.load.async.lds
    // GFX950-NOT: rocdl.raw.ptr.buffer.load.async.lds
```
**EN:** This comment block provides context for the surrounding test logic: `// Each thread needs to load 8 elements and we load 8 (sizePerThread) per buffer load instruction`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Each thread needs to load 8 elements and we load 8 (sizePerThread) per buffer load instruction`。

### Lines 75-81
```mlir
    // GFX942 does not support vectorization > 4bytes so we cannot lower it
    // GFX942-NOT: rocdl.raw.ptr.buffer.load.async.lds
    // GFX942: amdg.buffer_load_to_local
    %8 = amdg.buffer_load_to_local %arg1[%7] into %arg2 : <f16>[tensor<64x64xi32, #blocked>]  -> <64x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `rocdl.raw.ptr.buffer.load.async.lds`, `amdg.buffer_load_to_local`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.raw.ptr.buffer.load.async.lds`, `amdg.buffer_load_to_local`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 83
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 85-99
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [64, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 0 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: buffer_load_to_local_vectorized_8xf16
  tt.func public @buffer_load_to_local_vectorized_8xf16(%arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: !ttg.memdesc<256x8xf16, #shared, #smem, mutable>) {
    %cst = arith.constant dense<8> : tensor<256x1xi32, #blocked>
    %0 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %1 = tt.make_range {end = 8 : i32, start = 0 : i32} : tensor<8xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %2 = tt.expand_dims %0 {axis = 1 : i32} : tensor<256xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<256x1xi32, #blocked>
    %3 = arith.muli %2, %cst : tensor<256x1xi32, #blocked>
    %4 = tt.broadcast %3 : tensor<256x1xi32, #blocked> -> tensor<256x8xi32, #blocked>
    %5 = tt.expand_dims %1 {axis = 0 : i32} : tensor<8xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x8xi32, #blocked>
    %6 = tt.broadcast %5 : tensor<1x8xi32, #blocked> -> tensor<256x8xi32, #blocked>
    %7 = arith.addi %4, %6 : tensor<256x8xi32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_to_local_vectorized_8xf16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_to_local_vectorized_8xf16`。

### Lines 101-105
```mlir
    // Each thread needs to load 8 elements and we load 8 (sizePerThread) per buffer load instruction
    // GFX950: rocdl.make.buffer.rsrc
    // GFX950-NOT: rocdl.make.buffer.rsrc
    // GFX950: rocdl.raw.ptr.buffer.load.async.lds
    // GFX950-NOT: rocdl.raw.ptr.buffer.load.async.lds
```
**EN:** This comment block provides context for the surrounding test logic: `// Each thread needs to load 8 elements and we load 8 (sizePerThread) per buffer load instruction`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Each thread needs to load 8 elements and we load 8 (sizePerThread) per buffer load instruction`。

### Lines 107-113
```mlir
    // GFX942 does not support vectorization > 4bytes so we cannot lower it
    // GFX942-NOT: rocdl.raw.ptr.buffer.load.async.lds
    // GFX942: amdg.buffer_load_to_local
    %8 = amdg.buffer_load_to_local %arg1[%7] into %arg2 : <f16>[tensor<256x8xi32, #blocked>]  -> <256x8xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `rocdl.raw.ptr.buffer.load.async.lds`, `amdg.buffer_load_to_local`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.raw.ptr.buffer.load.async.lds`, `amdg.buffer_load_to_local`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 115
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 117-135
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: buffer_load_to_local_mask_other
  tt.func public @buffer_load_to_local_mask_other(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: !tt.ptr<f32>,
                                %arg2: tensor<32x32xi32, #blocked>,
                                %arg3: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>,
                                %arg4: i32) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    %c0_i32 = arith.constant 0 : i32
    %c32_i32 = arith.constant 32 : i32
    %c31_i32 = arith.constant 31 : i32
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    %29 = arith.addi %arg4, %c31_i32 : i32
    %30 = arith.divsi %29, %c32_i32 : i32
    %31 = arith.cmpi sgt, %30, %c0_i32 : i32
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_to_local_mask_other`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_to_local_mask_other`。

### Lines 137-141
```mlir
    %51 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %52 = tt.expand_dims %51 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xi32, #blocked>
    %65 = tt.splat %arg4 : i32 -> tensor<32x1xi32, #blocked>
    %66 = arith.cmpi slt, %52, %65 : tensor<32x1xi32, #blocked>
    %67 = tt.broadcast %66 : tensor<32x1xi1, #blocked> -> tensor<32x32xi1, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.make_range`, `tt.expand_dims`, `tt.splat`, `arith.cmpi`, `tt.broadcast`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.make_range`, `tt.expand_dims`, `tt.splat`, `arith.cmpi`, `tt.broadcast` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 143-144
```mlir
    %70 = tt.splat %31 : i1 -> tensor<32x32xi1, #blocked>
    %71 = arith.andi %70, %67 : tensor<32x32xi1, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `arith.andi`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `arith.andi` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 146-147
```mlir
    // Each thread needs to load 4 elements and we load 1 (sizePerThread) per buffer load instruction
    // Note that mask/other alignment is 1 so we need 4 conditionals
```
**EN:** This comment block provides context for the surrounding test logic: `// Each thread needs to load 4 elements and we load 1 (sizePerThread) per buffer load instruction`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Each thread needs to load 4 elements and we load 1 (sizePerThread) per buffer load instruction`。

### Lines 149-151
```mlir
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON: llvm.cond_br
    // COMMON: llvm.store
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: rocdl.raw.ptr.buffer.load.async.lds`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: rocdl.raw.ptr.buffer.load.async.lds`。

### Lines 153-155
```mlir
    // Make sure branch condition is set properly when there is other value.
    // COMMON: [[AND:%.*]] = llvm.and
    // COMMON: llvm.cond_br [[AND]]
```
**EN:** This comment block provides context for the surrounding test logic: `// Make sure branch condition is set properly when there is other value.`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Make sure branch condition is set properly when there is other value.`。

### Lines 157-159
```mlir
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON: llvm.cond_br
    // COMMON: llvm.store
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: rocdl.raw.ptr.buffer.load.async.lds`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: rocdl.raw.ptr.buffer.load.async.lds`。

### Lines 161-163
```mlir
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON: llvm.cond_br
    // COMMON: llvm.store
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: rocdl.raw.ptr.buffer.load.async.lds`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: rocdl.raw.ptr.buffer.load.async.lds`。

### Lines 165-167
```mlir
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON: llvm.cond_br
    // COMMON: llvm.store
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: rocdl.raw.ptr.buffer.load.async.lds`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: rocdl.raw.ptr.buffer.load.async.lds`。

### Lines 169-172
```mlir
    // COMMON-NOT: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON-NOT: _predicated_store
    // COMMON-NOT: llvm.cond_br
    // COMMON-NOT: llvm.store
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON-NOT: rocdl.raw.ptr.buffer.load.async.lds`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON-NOT: rocdl.raw.ptr.buffer.load.async.lds`。

### Lines 174-177
```mlir
    amdg.buffer_load_to_local %arg1[%arg2] mask=%67 other=%cst_0 into %arg3 : <f32>[tensor<32x32xi32, #blocked>] tensor<32x32xf32, #blocked>  -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `amdg.buffer_load_to_local`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `amdg.buffer_load_to_local`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 179
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 181-204
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: buffer_load_to_local_cache_mods
  tt.func public @buffer_load_to_local_cache_mods(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg2: !ttg.memdesc<64xf32, #shared, #smem, mutable>) {
    %0 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked>
    // The first constant 0 skips the LDS offset which is also 0.
    // Match the i32 select used to compute voffset
    // COMMON: %[[VOFFSET:.*]] = llvm.select {{.*}} : i1, i32
    // COMMON: %[[IMM0:.*]] = llvm.mlir.constant(0 : i32) : i32
    // COMMON: %[[aux_ca:.*]] = llvm.mlir.constant(0 : i32) : i32
    // COMMON: %[[IMM1:.*]] = llvm.mlir.constant(0 : i32) : i32
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds {{.*}}, {{.*}}, {{.*}}, %[[VOFFSET]], %[[IMM1]], %[[IMM0]], %[[aux_ca]]
    %1 = amdg.buffer_load_to_local %arg0[%0] cacheModifier = ca into %arg2: <f32>[tensor<64xi32, #blocked>] -> <64xf32, #shared, #smem, mutable>
    // COMMON: llvm.getelementptr
    // COMMON: %[[aux_cg:.*]] = llvm.mlir.constant(3 : i32) : i32
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds {{.*}}, {{.*}}, {{.*}}, {{.*}}, {{.*}}, {{.*}}, %[[aux_cg]]
    %2 = amdg.buffer_load_to_local %arg0[%0] cacheModifier = cg into %arg2: <f32>[tensor<64xi32, #blocked>] -> <64xf32, #shared, #smem, mutable>
    // COMMON: llvm.getelementptr
    // COMMON: %[[aux_cv:.*]] = llvm.mlir.constant(17 : i32) : i32
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds {{.*}}, {{.*}}, {{.*}}, {{.*}}, {{.*}}, {{.*}}, %[[aux_cv]]
    %3 = amdg.buffer_load_to_local %arg0[%0] cacheModifier = cv into %arg2: <f32>[tensor<64xi32, #blocked>] -> <64xf32, #shared, #smem, mutable>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_to_local_cache_mods`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_to_local_cache_mods`。

### Lines 206-208
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 210
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 212-232
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [8, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: buffer_load_swizzled_simple
  tt.func public @buffer_load_swizzled_simple(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: !tt.ptr<f32>,
                                %arg2: tensor<16x64xi32, #blocked>,
                                %arg3: !ttg.memdesc<16x64xf32, #shared, #smem, mutable>) {
    // Each thread needs to load 2 elements and we load 1 (sizePerThread) per buffer load instruction
    // COMMON: rocdl.make.buffer.rsrc
    // COMMON-NOT: rocdl.make.buffer.rsrc
    // COMMON: rocdl.ds_bpermute
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON: rocdl.ds_bpermute
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON-NOT: rocdl.raw.ptr.buffer.load.async.lds
    %65 = amdg.buffer_load_to_local %arg1[%arg2] into %arg3 : <f32>[tensor<16x64xi32, #blocked>] -> <16x64xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_swizzled_simple`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_swizzled_simple`。

### Line 234
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 236-254
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 2, maxPhase = 8, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: buffer_load_to_local_swizzled_mask_other
  tt.func public @buffer_load_to_local_swizzled_mask_other(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: !tt.ptr<f32>,
                                %arg2: tensor<32x32xi32, #blocked>,
                                %arg3: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>,
                                %arg4: i32) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    %c0_i32 = arith.constant 0 : i32
    %c32_i32 = arith.constant 32 : i32
    %c31_i32 = arith.constant 31 : i32
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    %29 = arith.addi %arg4, %c31_i32 : i32
    %30 = arith.divsi %29, %c32_i32 : i32
    %31 = arith.cmpi sgt, %30, %c0_i32 : i32
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_to_local_swizzled_mask_other`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_to_local_swizzled_mask_other`。

### Lines 256-260
```mlir
    %51 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %52 = tt.expand_dims %51 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xi32, #blocked>
    %65 = tt.splat %arg4 : i32 -> tensor<32x1xi32, #blocked>
    %66 = arith.cmpi slt, %52, %65 : tensor<32x1xi32, #blocked>
    %67 = tt.broadcast %66 : tensor<32x1xi1, #blocked> -> tensor<32x32xi1, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.make_range`, `tt.expand_dims`, `tt.splat`, `arith.cmpi`, `tt.broadcast`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.make_range`, `tt.expand_dims`, `tt.splat`, `arith.cmpi`, `tt.broadcast` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 262-263
```mlir
    %70 = tt.splat %31 : i1 -> tensor<32x32xi1, #blocked>
    %71 = arith.andi %70, %67 : tensor<32x32xi1, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `arith.andi`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `arith.andi` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 265-266
```mlir
    // Each thread needs to load 4 elements and we load 1 (sizePerThread) per buffer load instruction
    // Note that mask/other alignment is 1 so we need 4 conditionals
```
**EN:** This comment block provides context for the surrounding test logic: `// Each thread needs to load 4 elements and we load 1 (sizePerThread) per buffer load instruction`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Each thread needs to load 4 elements and we load 1 (sizePerThread) per buffer load instruction`。

### Lines 268-272
```mlir
    // COMMON: rocdl.ds_bpermute
    // COMMON: rocdl.ballot
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON: llvm.cond_br
    // COMMON: llvm.store
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: rocdl.ds_bpermute`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: rocdl.ds_bpermute`。

### Lines 274-278
```mlir
    // COMMON: rocdl.ds_bpermute
    // COMMON: rocdl.ballot
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON: llvm.cond_br
    // COMMON: llvm.store
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: rocdl.ds_bpermute`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: rocdl.ds_bpermute`。

### Lines 280-284
```mlir
    // COMMON: rocdl.ds_bpermute
    // COMMON: rocdl.ballot
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON: llvm.cond_br
    // COMMON: llvm.store
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: rocdl.ds_bpermute`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: rocdl.ds_bpermute`。

### Lines 286-290
```mlir
    // COMMON: rocdl.ds_bpermute
    // COMMON: rocdl.ballot
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON: llvm.cond_br
    // COMMON: llvm.store
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: rocdl.ds_bpermute`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: rocdl.ds_bpermute`。

### Lines 292-295
```mlir
    // COMMON-NOT: rocdl.ds_bpermute
    // COMMON-NOT: rocdl.ballot
    // COMMON-NOT: rocdl.raw.ptr.buffer.load.async.lds
    // COMMON-NOT: _predicated_store
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON-NOT: rocdl.ds_bpermute`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON-NOT: rocdl.ds_bpermute`。

### Lines 297-300
```mlir
    amdg.buffer_load_to_local %arg1[%arg2] mask=%67 other=%cst_0 into %arg3 : <f32>[tensor<32x32xi32, #blocked>] tensor<32x32xf32, #blocked>  -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `amdg.buffer_load_to_local`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `amdg.buffer_load_to_local`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 302
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 304-318
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 32], order = [0, 1]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 4, maxPhase = 16, order = [0, 1]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 32 : i32, ttg.shared = 0 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: buffer_load_to_local_swizzled_vectorized_8xf16
  tt.func public @buffer_load_to_local_swizzled_vectorized_8xf16(%arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: !ttg.memdesc<64x64xf16, #shared, #smem, mutable>) {
    %cst = arith.constant dense<64> : tensor<1x64xi32, #blocked>
    %0 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %2 = tt.expand_dims %0 {axis = 1 : i32} : tensor<64xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<64x1xi32, #blocked>
    %3 = tt.broadcast %2 : tensor<64x1xi32, #blocked> -> tensor<64x64xi32, #blocked>
    %4 = tt.expand_dims %1 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
    %5 = arith.muli %4, %cst : tensor<1x64xi32, #blocked>
    %6 = tt.broadcast %5 : tensor<1x64xi32, #blocked> -> tensor<64x64xi32, #blocked>
    %7 = arith.addi %3, %6 : tensor<64x64xi32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_to_local_swizzled_vectorized_8xf16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_to_local_swizzled_vectorized_8xf16`。

### Lines 320-323
```mlir
    // Each thread needs to load 8 elements and we load 8 (sizePerThread) per buffer load instruction
    // GFX950: rocdl.make.buffer.rsrc
    // GFX950: rocdl.raw.ptr.buffer.load.async.lds
    // GFX950-NOT: rocdl.raw.ptr.buffer.load.async.lds
```
**EN:** This comment block provides context for the surrounding test logic: `// Each thread needs to load 8 elements and we load 8 (sizePerThread) per buffer load instruction`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Each thread needs to load 8 elements and we load 8 (sizePerThread) per buffer load instruction`。

### Lines 325-331
```mlir
    // GFX942 does not support vectorization > 4bytes so we cannot lower it
    // GFX942-NOT: rocdl.raw.ptr.buffer.load.async.lds
    // GFX942: amdg.buffer_load_to_local
    %8 = amdg.buffer_load_to_local %arg1[%7] into %arg2 : <f16>[tensor<64x64xi32, #blocked>]  -> <64x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `rocdl.raw.ptr.buffer.load.async.lds`, `amdg.buffer_load_to_local`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.raw.ptr.buffer.load.async.lds`, `amdg.buffer_load_to_local`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 333
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 335-347
```mlir
#blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
#shared1D = #ttg.swizzled_shared<{vec = 2, perPhase = 1, maxPhase = 8, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // COMMON-LABEL: buffer_load_to_local_contiguity_hint
  tt.func @buffer_load_to_local_contiguity_hint(%ptr: !tt.ptr<f16>, %off: tensor<256xi32, #blocked>, %lds: !ttg.memdesc<256xf16, #shared1D, #smem, mutable>) {
    // Check we load 4 bytes
    // COMMON: %[[LOAD_BYTES:.*]] = llvm.mlir.constant(4 : i32) : i32
    // COMMON: rocdl.raw.ptr.buffer.load.async.lds %{{.*}}, %{{.*}}, %[[LOAD_BYTES]]
    %0 = amdg.buffer_load_to_local %ptr[%off] into %lds {contiguity = 2 : i32} : <f16>[tensor<256xi32, #blocked>] -> <256xf16, #shared1D, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_to_local_contiguity_hint`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_to_local_contiguity_hint`。

### Line 349
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 351-366
```mlir
// Load 64 elements with 4 warps so we end up with redundant warps which should be masked out via LDS out-of-range address
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: @buffer_load_warp_uniform_thread_pred
  tt.func @buffer_load_warp_uniform_thread_pred(%ptr: !tt.ptr<f32>, %lds: !ttg.memdesc<64xf32, #shared, #smem, mutable>) {
    %off = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked>
    // CHECK: %[[OOB_I32:.*]] = llvm.mlir.constant(2147483647 : i32) : i32
    // CHECK: %[[OOB_PTR:.*]] = llvm.inttoptr %[[OOB_I32]] : i32 to !llvm.ptr<3>
    // CHECK: %[[PRED_ADDR:.*]] = llvm.select {{.*}}, {{.*}}, %[[OOB_PTR]] : i1, !llvm.ptr<3>
    // CHECK: rocdl.raw.ptr.buffer.load.async.lds {{.*}}, %[[PRED_ADDR]], {{.*}}
    %0 = amdg.buffer_load_to_local %ptr[%off] into %lds : <f32>[tensor<64xi32, #blocked>] -> <64xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_warp_uniform_thread_pred`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_warp_uniform_thread_pred`。

### Line 368
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 370-387
```mlir
// Load 64 elements with 4 warps to get warp-uniform threadPred but we should still see the branch because of the mask and other values.
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: @buffer_load_warp_uniform_thread_pred_with_per_lane_mask
  tt.func @buffer_load_warp_uniform_thread_pred_with_per_lane_mask(%ptr: !tt.ptr<f32>, %lds: !ttg.memdesc<64xf32, #shared, #smem, mutable>, %mask: tensor<64xi1, #blocked>) {
    %off = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked>
    %other = arith.constant dense<0.000000e+00> : tensor<64xf32, #blocked>
    // Even with warp-uniform threadPred, hasOther must keep branch predication.
    // CHECK: %[[PRED:.*]] = llvm.and {{.*}} : i1
    // CHECK: llvm.cond_br %[[PRED]], ^[[LOAD_BLOCK:bb[0-9]+]]
    // CHECK-NEXT: ^[[LOAD_BLOCK]]:
    // CHECK: rocdl.raw.ptr.buffer.load.async.lds
    %0 = amdg.buffer_load_to_local %ptr[%off] mask=%mask other=%other into %lds : <f32>[tensor<64xi32, #blocked>] tensor<64xf32, #blocked> -> <64xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_warp_uniform_thread_pred_with_per_lane_mask`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_warp_uniform_thread_pred_with_per_lane_mask`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on buffer operations.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 缓冲区操作。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--check-prefixes=COMMON,GFX950`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--verify-diagnostics`, `--check-prefixes=COMMON,GFX942`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--check-prefixes=COMMON,GFX950`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--verify-diagnostics`, `--check-prefixes=COMMON,GFX942` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `llvm` (LLVM dialect IR).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`llvm`（LLVM 方言 IR）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `llvm`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`llvm`。
