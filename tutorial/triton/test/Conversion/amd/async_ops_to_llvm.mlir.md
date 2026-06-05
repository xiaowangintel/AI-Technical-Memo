# async_ops_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/async_ops_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises asynchronous copy/lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的异步拷贝/降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefix=GFX950`<br>`triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×62, CHECK-COUNT×3, CHECK-LABEL×13, CHECK-NEXT×11, CHECK-NOT×3; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×62, CHECK-COUNT×3, CHECK-LABEL×13, CHECK-NEXT×11, CHECK-NOT×3；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that asynchronous copy/lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 异步拷贝/降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefix=GFX950
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefix=GFX950; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefix=GFX950; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-21
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_copy
  tt.func public @async_copy(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<32x64xf32, #shared, #smem, mutable>) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x64x!tt.ptr<f32>, #blocked>
    // Each thread needs to load 8 elements and we load 1 (sizePerThread) per load.
    // CDNA3/CDNA4 use the async variant so LLVM tracks via asyncmark.
    // CHECK-COUNT-8: rocdl.global.load.async.lds
    // CHECK-NOT: rocdl.global.load.async.lds
    %2 = ttg.async_copy_global_to_local %1, %arg2 : tensor<32x64x!tt.ptr<f32>, #blocked> -> <32x64xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy`。

### Line 23
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 25-41
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[64:+4] {order = [1, 0], shape = [32, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_copy_padded
  tt.func public @async_copy_padded(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<32x64xf32, #shared, #smem, mutable>) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x64x!tt.ptr<f32>, #blocked>
    // Each thread needs to load 8 elements and we load 1 () per load
    // CHECK-COUNT-8: rocdl.global.load.async.lds
    // CHECK-NOT: rocdl.global.load.async.lds
    %2 = ttg.async_copy_global_to_local %1, %arg2 : tensor<32x64x!tt.ptr<f32>, #blocked> -> <32x64xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_padded`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_padded`。

### Line 43
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 45-58
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_copy_vectorized_2xf16
  tt.func public @async_copy_vectorized_2xf16(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<32x64xf16, #shared, #smem, mutable>) {
    // We need the index calculation so AxisAnalysis sees that we can vectorize the load
    %1 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %2 = tt.expand_dims %1 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
    %3 = tt.broadcast %2 : tensor<1x64xi32, #blocked> -> tensor<32x64xi32, #blocked>
    %4 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<32x64x!tt.ptr<f16>, #blocked>
    %5 = tt.addptr %4, %3 : tensor<32x64x!tt.ptr<f16>, #blocked>, tensor<32x64xi32, #blocked>
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_vectorized_2xf16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_vectorized_2xf16`。

### Lines 60-66
```mlir
    // Each thread needs to load 8 elements and we load 2 (sizePerThread) per load
    // CHECK-COUNT-4: rocdl.global.load.async.lds
    // CHECK-NOT: rocdl.global.load.async.lds
    %6 = ttg.async_copy_global_to_local %5, %arg2 : tensor<32x64x!tt.ptr<f16>, #blocked> -> <32x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies. Embedded check comments (CHECK-COUNT×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 68
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 70-100
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_wait
  // GFX950-LABEL: async_wait
  tt.func public @async_wait(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                             %arg1: i32 {tt.divisibility = 16 : i32},
                             %arg2: !ttg.memdesc<32x64xf16, #shared, #smem, mutable>) {
    // CDNA3/CDNA4 lower ttg.async_wait directly to wait_asyncmark.
    // The commit group count is passed through without clamping since
    // LLVM will compute the final waitcnt.
    // CHECK: rocdl.wait.asyncmark 0
    // GFX950: rocdl.wait.asyncmark 0
    ttg.async_wait {num = 0 : i32}
    // CHECK: rocdl.wait.asyncmark 1
    // GFX950: rocdl.wait.asyncmark 1
    ttg.async_wait {num = 1 : i32}
    // CHECK: rocdl.wait.asyncmark 62
    // GFX950: rocdl.wait.asyncmark 62
    ttg.async_wait {num = 62 : i32}
    // CHECK: rocdl.wait.asyncmark 63
    // GFX950: rocdl.wait.asyncmark 63
    ttg.async_wait {num = 63 : i32}
    // No clamping — LLVM handles it based on instruction count
    // CHECK: rocdl.wait.asyncmark 64
    // GFX950: rocdl.wait.asyncmark 64
    ttg.async_wait {num = 64 : i32}
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_wait`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_wait`。

### Line 102
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 104-123
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_commit_group
  // GFX950-LABEL: async_commit_group
  tt.func public @async_commit_group(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                     %arg1: i32 {tt.divisibility = 16 : i32},
                                     %arg2: !ttg.memdesc<32x64xf16, #shared, #smem, mutable>) {
    // CDNA3/CDNA4 emit asyncmark for async group tracking
    // CHECK: rocdl.asyncmark
    // CHECK: llvm.mlir.constant(0 : i32) : i32
    // CHECK-NEXT: llvm.return
    // GFX950: rocdl.asyncmark
    // GFX950: llvm.mlir.constant(0 : i32) : i32
    // GFX950-NEXT: llvm.return
    ttg.async_commit_group
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_commit_group`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_commit_group`。

### Line 125
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 127-144
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_copy_mask_other
  tt.func public @async_copy_mask_other(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>,
                                %arg3: i32 {tt.divisibility = 16 : i32}) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    %c0_i32 = arith.constant 0 : i32
    %c32_i32 = arith.constant 32 : i32
    %c31_i32 = arith.constant 31 : i32
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    %29 = arith.addi %arg3, %c31_i32 : i32
    %30 = arith.divsi %29, %c32_i32 : i32
    %31 = arith.cmpi sgt, %30, %c0_i32 : i32
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_mask_other`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_mask_other`。

### Lines 146-150
```mlir
    %51 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %52 = tt.expand_dims %51 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xi32, #blocked>
    %65 = tt.splat %arg3 : i32 -> tensor<32x1xi32, #blocked>
    %66 = arith.cmpi slt, %52, %65 : tensor<32x1xi32, #blocked>
    %67 = tt.broadcast %66 : tensor<32x1xi1, #blocked> -> tensor<32x32xi1, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.make_range`, `tt.expand_dims`, `tt.splat`, `arith.cmpi`, `tt.broadcast`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.make_range`, `tt.expand_dims`, `tt.splat`, `arith.cmpi`, `tt.broadcast` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 152-153
```mlir
    %70 = tt.splat %31 : i1 -> tensor<32x32xi1, #blocked>
    %71 = arith.andi %70, %67 : tensor<32x32xi1, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `arith.andi`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `arith.andi` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 155-156
```mlir
    // Each thread needs to load 4 elements and we load 1 (sizePerThread) per global.load.lds
    // Note that mask/other alignment is 1 so we need 4 conditionals
```
**EN:** This comment block provides context for the surrounding test logic: `// Each thread needs to load 4 elements and we load 1 (sizePerThread) per global.load.lds`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Each thread needs to load 4 elements and we load 1 (sizePerThread) per global.load.lds`。

### Lines 158-162
```mlir
    // CHECK: llvm.cond_br
    // CHECK: rocdl.global.load.async.lds
    // CHECK-NEXT: llvm.br
    // CHECK: llvm.cond_br
    // CHECK: llvm.store
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 164-168
```mlir
    // CHECK: llvm.cond_br
    // CHECK: rocdl.global.load.async.lds
    // CHECK-NEXT: llvm.br
    // CHECK: llvm.cond_br
    // CHECK: llvm.store
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 170-174
```mlir
    // CHECK: llvm.cond_br
    // CHECK: rocdl.global.load.async.lds
    // CHECK-NEXT: llvm.br
    // CHECK: llvm.cond_br
    // CHECK: llvm.store
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 176-180
```mlir
    // CHECK: llvm.cond_br
    // CHECK: rocdl.global.load.async.lds
    // CHECK-NEXT: llvm.br
    // CHECK: llvm.cond_br
    // CHECK: llvm.store
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 182-185
```mlir
    %2 = ttg.async_copy_global_to_local %1, %arg2 mask %67 other %cst_0 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `ttg.async_copy_global_to_local`, `tt.ptr`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.async_copy_global_to_local`, `tt.ptr`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 187
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 189-206
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_copy_swizzled_mask_other
  tt.func public @async_copy_swizzled_mask_other(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>,
                                %arg3: i32 {tt.divisibility = 16 : i32}) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    %c0_i32 = arith.constant 0 : i32
    %c32_i32 = arith.constant 32 : i32
    %c31_i32 = arith.constant 31 : i32
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    %29 = arith.addi %arg3, %c31_i32 : i32
    %30 = arith.divsi %29, %c32_i32 : i32
    %31 = arith.cmpi sgt, %30, %c0_i32 : i32
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_swizzled_mask_other`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_swizzled_mask_other`。

### Lines 208-212
```mlir
    %51 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %52 = tt.expand_dims %51 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xi32, #blocked>
    %65 = tt.splat %arg3 : i32 -> tensor<32x1xi32, #blocked>
    %66 = arith.cmpi slt, %52, %65 : tensor<32x1xi32, #blocked>
    %67 = tt.broadcast %66 : tensor<32x1xi1, #blocked> -> tensor<32x32xi1, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.make_range`, `tt.expand_dims`, `tt.splat`, `arith.cmpi`, `tt.broadcast`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.make_range`, `tt.expand_dims`, `tt.splat`, `arith.cmpi`, `tt.broadcast` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 214-215
```mlir
    %70 = tt.splat %31 : i1 -> tensor<32x32xi1, #blocked>
    %71 = arith.andi %70, %67 : tensor<32x32xi1, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `arith.andi`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `arith.andi` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 217-218
```mlir
    // Each thread needs to load 4 elements and we load 1 (sizePerThread) per global.load.lds
    // Note that mask/other alignment is 1 so we need 4 conditionals
```
**EN:** This comment block provides context for the surrounding test logic: `// Each thread needs to load 4 elements and we load 1 (sizePerThread) per global.load.lds`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Each thread needs to load 4 elements and we load 1 (sizePerThread) per global.load.lds`。

### Lines 220-226
```mlir
    // CHECK: rocdl.ds_bpermute
    // CHECK: rocdl.ballot
    // CHECK: llvm.cond_br
    // CHECK: rocdl.global.load.async.lds
    // CHECK-NEXT: llvm.br
    // CHECK: llvm.cond_br
    // CHECK: llvm.store
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 228-234
```mlir
    // CHECK: rocdl.ds_bpermute
    // CHECK: rocdl.ballot
    // CHECK: llvm.cond_br
    // CHECK: rocdl.global.load.async.lds
    // CHECK-NEXT: llvm.br
    // CHECK: llvm.cond_br
    // CHECK: llvm.store
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 236-242
```mlir
    // CHECK: rocdl.ds_bpermute
    // CHECK: rocdl.ballot
    // CHECK: llvm.cond_br
    // CHECK: rocdl.global.load.async.lds
    // CHECK-NEXT: llvm.br
    // CHECK: llvm.cond_br
    // CHECK: llvm.store
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 244-250
```mlir
    // CHECK: rocdl.ds_bpermute
    // CHECK: rocdl.ballot
    // CHECK: llvm.cond_br
    // CHECK: rocdl.global.load.async.lds
    // CHECK-NEXT: llvm.br
    // CHECK: llvm.cond_br
    // CHECK: llvm.store
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 252-255
```mlir
    %2 = ttg.async_copy_global_to_local %1, %arg2 mask %67 other %cst_0 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `ttg.async_copy_global_to_local`, `tt.ptr`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.async_copy_global_to_local`, `tt.ptr`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 257
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 259-269
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 32], warpsPerCTA = [16, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 16 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_copy_cache_mods
  tt.func public @async_copy_cache_mods(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    // Each thread needs to load 1 element and we load 1 (sizePerThread) per global.load.lds
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_cache_mods`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_cache_mods`。

### Lines 271-282
```mlir
    // CHECK: llvm.getelementptr
    // CHECK: rocdl.global.load.async.lds {{.*}}, {{.*}}, 4, 0, 0
    %2 = ttg.async_copy_global_to_local %1, %arg2 cacheModifier = ca: tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    // CHECK: llvm.getelementptr
    // CHECK: rocdl.global.load.async.lds {{.*}}, {{.*}}, 4, 0, 3
    %3 = ttg.async_copy_global_to_local %1, %arg2 cacheModifier = cg: tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    // CHECK: llvm.getelementptr
    // CHECK: rocdl.global.load.async.lds {{.*}}, {{.*}}, 4, 0, 17
    %4 = ttg.async_copy_global_to_local %1, %arg2 cacheModifier = cv: tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies. Embedded check comments (CHECK×6) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝。 其中嵌入的检查注释（CHECK×6）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 284
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 286-297
```mlir
#blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
#shared1D = #ttg.swizzled_shared<{vec = 2, perPhase = 1, maxPhase = 8, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_copy_contiguity_hint
  tt.func @async_copy_contiguity_hint(%v: tensor<256x!tt.ptr<f16>, #blocked>, %smem: !ttg.memdesc<256xf16, #shared1D, #smem, mutable>) {
    // Check we load 4 bytes at a time
    // CHECK: rocdl.global.load.async.lds {{.*}}, {{.*}}, 4
    %0 = ttg.async_copy_global_to_local %v, %smem {contiguity = 2 : i32} : tensor<256x!tt.ptr<f16>, #blocked> -> !ttg.memdesc<256xf16, #shared1D, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_contiguity_hint`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_contiguity_hint`。

### Line 299
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 301-316
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_copy_one_row_into_subslice
  tt.func public @async_copy_one_row_into_subslice(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<32x128xf32, #shared, #smem, mutable>) {
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x64x!tt.ptr<f32>, #blocked>
    %2 = ttg.memdesc_subslice %arg2 [0, 0]  : !ttg.memdesc<32x128xf32, #shared, #smem, mutable> -> !ttg.memdesc<32x64xf32, #shared, #smem, mutable, 32x128>
    // We slice in the fastest dim but each warp loads one row, therefore we can write coalesced into LDS
    // CHECK: rocdl.global.load.async.lds
    %3 = ttg.async_copy_global_to_local %1, %2 : tensor<32x64x!tt.ptr<f32>, #blocked> -> <32x64xf32, #shared, #smem, mutable, 32x128>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_one_row_into_subslice`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_one_row_into_subslice`。

### Line 318
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 320-335
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_copy_into_slowest_dim_subslice
  tt.func public @async_copy_into_slowest_dim_subslice(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<64x32xf32, #shared, #smem, mutable>) {
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    %2 = ttg.memdesc_subslice %arg2 [0, 0]  : !ttg.memdesc<64x32xf32, #shared, #smem, mutable> -> !ttg.memdesc<32x32xf32, #shared, #smem, mutable, 64x32>
    // We slice into the slowest dim which does not break coalesced writes into LDS
    // CHECK: rocdl.global.load.async.lds
    %3 = ttg.async_copy_global_to_local %1, %2 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable, 64x32>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_into_slowest_dim_subslice`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_into_slowest_dim_subslice`。

### Line 337
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 339-354
```mlir
// Load 64 elements with 4 warps so we end up with redundant warps which should be masked out via LDS out-of-range address since we do not have masking
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: @async_copy_warp_uniform_thread_pred
  tt.func @async_copy_warp_uniform_thread_pred(%ptr: !tt.ptr<f32>, %lds: !ttg.memdesc<64xf32, #shared, #smem, mutable>) {
    %src = tt.splat %ptr : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>, #blocked>
    // CHECK: %[[OOB_I32:.*]] = llvm.mlir.constant(2147483647 : i32) : i32
    // CHECK: %[[OOB_PTR:.*]] = llvm.inttoptr %[[OOB_I32]] : i32 to !llvm.ptr<3>
    // CHECK: %[[PRED_ADDR:.*]] = llvm.select {{.*}}, {{.*}}, %[[OOB_PTR]] : i1, !llvm.ptr<3>
    // CHECK: rocdl.global.load.async.lds {{.*}}, %[[PRED_ADDR]], {{.*}}
    %0 = ttg.async_copy_global_to_local %src, %lds : tensor<64x!tt.ptr<f32>, #blocked> -> <64xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_warp_uniform_thread_pred`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_warp_uniform_thread_pred`。

### Line 356
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 358-374
```mlir
// Load 64 elements with 4 warps to get warp-uniform threadPred but we should still see the branch because of the explicit mask.
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: @async_copy_warp_uniform_thread_pred_with_per_lane_mask
  tt.func @async_copy_warp_uniform_thread_pred_with_per_lane_mask(%ptr: !tt.ptr<f32>, %lds: !ttg.memdesc<64xf32, #shared, #smem, mutable>, %mask: tensor<64xi1, #blocked>) {
    %src = tt.splat %ptr : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>, #blocked>
    %other = arith.constant dense<0.000000e+00> : tensor<64xf32, #blocked>
    // CHECK: %[[PRED:.*]] = llvm.and {{.*}} : i1
    // CHECK: llvm.cond_br %[[PRED]], ^[[LOAD_BLOCK:bb[0-9]+]]
    // CHECK-NEXT: ^[[LOAD_BLOCK]]:
    // CHECK-NEXT: rocdl.global.load.async.lds
    %0 = ttg.async_copy_global_to_local %src, %lds mask %mask other %other : tensor<64x!tt.ptr<f32>, #blocked> -> <64xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_warp_uniform_thread_pred_with_per_lane_mask`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_warp_uniform_thread_pred_with_per_lane_mask`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on asynchronous copy/lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 异步拷贝/降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--check-prefix=GFX950`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--check-prefix=GFX950`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `llvm` (LLVM dialect IR), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `llvm`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`llvm`、`arith`。
