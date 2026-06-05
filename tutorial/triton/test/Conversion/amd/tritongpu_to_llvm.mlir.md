# tritongpu_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/tritongpu_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises AMD-specific lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的AMD 专用降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s --check-prefixes=CHECK,COMMON`<br>`triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefixes=GFX950,COMMON`<br>`triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s --check-prefixes=GFX1250,COMMON`<br>`triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx906 | FileCheck %s --check-prefixes=GFX906,COMMON` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×119, CHECK-COUNT×20, CHECK-DAG×5, CHECK-LABEL×25, CHECK-NEXT×34, CHECK-NOT×17, CHECK-SAME×11; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×119, CHECK-COUNT×20, CHECK-DAG×5, CHECK-LABEL×25, CHECK-NEXT×34, CHECK-NOT×17, CHECK-SAME×11；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that AMD-specific lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 AMD 专用降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4
```mlir
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s --check-prefixes=CHECK,COMMON
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefixes=GFX950,COMMON
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s --check-prefixes=GFX1250,COMMON
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx906 | FileCheck %s --check-prefixes=GFX906,COMMON
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s --check-prefixes=CHECK,COMMON; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefixes=GFX950,COMMON; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s --check-prefixes=GFX1250,COMMON; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx906 | FileCheck %s --check-prefixes=GFX906,COMMON` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 --convert-builtin-func-to-llvm | FileCheck %s --check-prefixes=CHECK,COMMON; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s --check-prefixes=GFX950,COMMON; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s --check-prefixes=GFX1250,COMMON; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx906 | FileCheck %s --check-prefixes=GFX906,COMMON`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 6-25
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: atomic_add_f32_scalar
  // GFX1250-LABEL: atomic_add_f32_scalar
  tt.func @atomic_add_f32_scalar(%arg0 : !tt.ptr<f32>, %arg1 : i1, %arg2 : f32) {
    // Scalar atomics should not have the compiler fence (no tensorTy)
    // GFX1250-NOT: llvm.inline_asm
    // CHECK: llvm.cond_br
    // GFX1250: llvm.cond_br
    // CHECK: llvm.atomicrmw
    // CHECK: llvm.store
    // CHECK: llvm.br
    // CHECK: rocdl.s.waitcnt 49279
    // CHECK: rocdl.s.barrier
    // CHECK: llvm.load
    // CHECK: llvm.store
    %0 = tt.atomic_rmw fadd, relaxed, gpu, %arg0, %arg2, %arg1 : (!tt.ptr<f32>, f32, i1) -> f32
    tt.store %arg0, %0 : !tt.ptr<f32>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f32_scalar`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f32_scalar`。

### Line 27
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 29-47
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: atomic_add_f32
  // GFX1250-LABEL: atomic_add_f32
  tt.func @atomic_add_f32(%arg0 : tensor<256x!tt.ptr<f32>, #blocked0>, %arg1 : tensor<256xi1, #blocked0>, %arg2 : tensor<256xf32, #blocked0>) {
    // Tensor atomics on gfx1250 should have a compiler fence before the cond_br
    // to prevent MachineSink from sinking LDS loads past barriers.
    // GFX1250: llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "", "~{memory}"
    // GFX1250: llvm.cond_br
    // CHECK: llvm.cond_br
    // CHECK: llvm.atomicrmw
    // CHECK: llvm.atomicrmw
    // CHECK: llvm.store
    // CHECK: llvm.store
    %0 = tt.atomic_rmw fadd, relaxed, gpu, %arg0, %arg2, %arg1 : (tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xf32, #blocked0>, tensor<256xi1, #blocked0>) -> tensor<256xf32, #blocked0>
    tt.store %arg0, %0 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f32`。

### Line 49
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 51-69
```mlir
// Smoke test to check that mfma 32 and dot operand layouts can work with small tensors, for example with shape 16x16
#mfma = #ttg.amd_mfma<{version = 2, warpsPerCTA = [2, 2], instrShape = [32, 32, 8], isTransposed = true}>
#dotop0 = #ttg.dot_op<{opIdx = 0, parent = #mfma, kWidth=4}>
#dotop1 = #ttg.dot_op<{opIdx = 1, parent = #mfma, kWidth=4}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: small_mfma_tensor_conversions
  tt.func public @small_mfma_tensor_conversions(%arg0: tensor<16x16xf16, #mfma>, %arg1: tensor<16x16x!tt.ptr<f32>, #mfma>) {
    // CHECK-NOT: ttg.convert_layout
    %0 = ttg.local_alloc %arg0 : (tensor<16x16xf16, #mfma>) -> !ttg.memdesc<16x16xf16, #shared, #smem>
    // CHECK-4: store {{.*}} vector<4xf16>
    %1 = ttg.local_load %0 : !ttg.memdesc<16x16xf16, #shared, #smem> -> tensor<16x16xf16, #dotop0>
    // CHECK-2: load {{.*}} vector<4xf16>
    %2 = ttg.local_load %0 : !ttg.memdesc<16x16xf16, #shared, #smem> -> tensor<16x16xf16, #dotop1>
    // CHECK-8: load {{.*}} vector<1xf16>
    %3 = ttg.local_load %0 : !ttg.memdesc<16x16xf16, #shared, #smem> -> tensor<16x16xf16, #mfma>
    // CHECK-4: load {{.*}} vector<4xf16>
    %4 = tt.fp_to_fp %3 : tensor<16x16xf16, #mfma> -> tensor<16x16xf32, #mfma>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `small_mfma_tensor_conversions`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `small_mfma_tensor_conversions`。

### Lines 71-76
```mlir
    %5 = tt.dot %1, %2, %4 : tensor<16x16xf16, #dotop0> * tensor<16x16xf16, #dotop1> -> tensor<16x16xf32, #mfma>
    // Store result to prevent DCE from removing all conversion related code
    %6 = ttg.local_alloc %5 : (tensor<16x16xf32, #mfma>) -> !ttg.memdesc<16x16xf32, #shared, #smem>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.dot`, `ttg.local_alloc`, `ttg.memdesc`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot`, `ttg.local_alloc`, `ttg.memdesc`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 78
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 80-94
```mlir
#blocked1 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: atomic_add_f16x2
  tt.func @atomic_add_f16x2(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1 : tensor<256xi1, #blocked1> {tt.constancy = 2 : i32}, %arg2 : tensor<256xf16, #blocked1>) {
    %range = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked1>
    %base_ptr = tt.splat %arg0 : !tt.ptr<f16> -> tensor<256x!tt.ptr<f16>, #blocked1>
    %ptr = tt.addptr %base_ptr, %range : tensor<256x!tt.ptr<f16>, #blocked1>, tensor<256xi32, #blocked1>
    // CHECK: llvm.cond_br
    // CHECK-NOT: rocdl.update.dpp
    // CHECK: llvm.atomicrmw fadd {{.*}} vector<2xf16>
    // CHECK-NOT: rocdl.update.dpp
    %0 =  tt.atomic_rmw fadd, relaxed, gpu, %ptr, %arg2, %arg1 : (tensor<256x!tt.ptr<f16>, #blocked1>, tensor<256xf16, #blocked1>, tensor<256xi1, #blocked1>) -> tensor<256xf16, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f16x2`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f16x2`。

### Line 96
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 98-112
```mlir
#blocked2 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: atomic_add_bf16x2
  tt.func @atomic_add_bf16x2(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32}, %arg1 : tensor<256xi1, #blocked2> {tt.constancy = 2 : i32}, %arg2 : tensor<256xbf16, #blocked2>) {
    %range = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked2>
    %base_ptr = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<256x!tt.ptr<bf16>, #blocked2>
    %ptr = tt.addptr %base_ptr, %range : tensor<256x!tt.ptr<bf16>, #blocked2>, tensor<256xi32, #blocked2>
    // CHECK: llvm.cond_br
    // CHECK-NOT: rocdl.update.dpp
    // CHECK: llvm.atomicrmw fadd {{.*}} vector<2xbf16>
    // CHECK-NOT: rocdl.update.dpp
    %0 =  tt.atomic_rmw fadd, relaxed, gpu, %ptr, %arg2, %arg1 : (tensor<256x!tt.ptr<bf16>, #blocked2>, tensor<256xbf16, #blocked2>, tensor<256xi1, #blocked2>) -> tensor<256xbf16, #blocked2>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_bf16x2`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_bf16x2`。

### Line 114
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 116-130
```mlir
#blocked1 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: atomic_add_f16_mask_not_aligned
  tt.func @atomic_add_f16_mask_not_aligned(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1 : tensor<256xi1, #blocked1>, %arg2 : tensor<256xf16, #blocked1>) {
    %range = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked1>
    %base_ptr = tt.splat %arg0 : !tt.ptr<f16> -> tensor<256x!tt.ptr<f16>, #blocked1>
    %ptr = tt.addptr %base_ptr, %range : tensor<256x!tt.ptr<f16>, #blocked1>, tensor<256xi32, #blocked1>
    // CHECK: llvm.cond_br
    // CHECK: rocdl.update.dpp
    // CHECK: llvm.atomicrmw fadd {{.*}} vector<2xf16>
    // CHECK: rocdl.update.dpp
    %0 =  tt.atomic_rmw fadd, relaxed, gpu, %ptr, %arg2, %arg1 : (tensor<256x!tt.ptr<f16>, #blocked1>, tensor<256xf16, #blocked1>, tensor<256xi1, #blocked1>) -> tensor<256xf16, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f16_mask_not_aligned`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f16_mask_not_aligned`。

### Line 132
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 134-148
```mlir
#blocked1 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: atomic_add_bf16_mask_not_aligned
  tt.func @atomic_add_bf16_mask_not_aligned(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32}, %arg1 : tensor<256xi1, #blocked1>, %arg2 : tensor<256xbf16, #blocked1>) {
    %range = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked1>
    %base_ptr = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<256x!tt.ptr<bf16>, #blocked1>
    %ptr = tt.addptr %base_ptr, %range : tensor<256x!tt.ptr<bf16>, #blocked1>, tensor<256xi32, #blocked1>
    // CHECK: llvm.cond_br
    // CHECK: rocdl.update.dpp
    // CHECK: llvm.atomicrmw fadd {{.*}} vector<2xbf16>
    // CHECK: rocdl.update.dpp
    %0 =  tt.atomic_rmw fadd, relaxed, gpu, %ptr, %arg2, %arg1 : (tensor<256x!tt.ptr<bf16>, #blocked1>, tensor<256xbf16, #blocked1>, tensor<256xi1, #blocked1>) -> tensor<256xbf16, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_bf16_mask_not_aligned`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_bf16_mask_not_aligned`。

### Line 150
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 152-166
```mlir
#blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: atomic_add_f16_dpp
  tt.func @atomic_add_f16_dpp(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1 : tensor<256xi1, #blocked1>, %arg2 : tensor<256xf16, #blocked1>) {
    %range = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked1>
    %base_ptr = tt.splat %arg0 : !tt.ptr<f16> -> tensor<256x!tt.ptr<f16>, #blocked1>
    %ptr = tt.addptr %base_ptr, %range : tensor<256x!tt.ptr<f16>, #blocked1>, tensor<256xi32, #blocked1>
    // CHECK: llvm.cond_br
    // CHECK: rocdl.update.dpp
    // CHECK: llvm.atomicrmw fadd {{.*}} vector<2xf16>
    // CHECK: rocdl.update.dpp
    %0 =  tt.atomic_rmw fadd, relaxed, gpu, %ptr, %arg2, %arg1 : (tensor<256x!tt.ptr<f16>, #blocked1>, tensor<256xf16, #blocked1>, tensor<256xi1, #blocked1>) -> tensor<256xf16, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f16_dpp`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f16_dpp`。

### Line 168
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 170-184
```mlir
#blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: atomic_add_bf16_dpp
  tt.func @atomic_add_bf16_dpp(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32}, %arg1 : tensor<256xi1, #blocked2>, %arg2 : tensor<256xbf16, #blocked2>) {
    %range = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked2>
    %base_ptr = tt.splat %arg0 : !tt.ptr<bf16> -> tensor<256x!tt.ptr<bf16>, #blocked2>
    %ptr = tt.addptr %base_ptr, %range : tensor<256x!tt.ptr<bf16>, #blocked2>, tensor<256xi32, #blocked2>
    // CHECK: llvm.cond_br
    // CHECK: rocdl.update.dpp
    // CHECK: llvm.atomicrmw fadd {{.*}} vector<2xbf16>
    // CHECK: rocdl.update.dpp
    %0 =  tt.atomic_rmw fadd, relaxed, gpu, %ptr, %arg2, %arg1 : (tensor<256x!tt.ptr<bf16>, #blocked2>, tensor<256xbf16, #blocked2>, tensor<256xi1, #blocked2>) -> tensor<256xbf16, #blocked2>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_bf16_dpp`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_bf16_dpp`。

### Line 186
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 188-194
```mlir
#blocked3 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: reduce_dpp_max
  tt.func @reduce_dpp_max(%arg0: tensor<64xf32, #blocked3>) {
    // CHECK: rocdl.update.dpp
    // CHECK-SAME: with 280, 15, 15, true : f32
    // CHECK-NEXT: llvm.intr.maxnum
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_dpp_max`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_dpp_max`。

### Lines 196-198
```mlir
    // CHECK-NEXT: rocdl.update.dpp
    // CHECK-SAME: with 276, 15, 15, true : f32
    // CHECK-NEXT: llvm.intr.maxnum
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`, `CHECK-SAME`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`, `CHECK-SAME`），从而保证后端/代码生成结果稳定。

### Lines 200-202
```mlir
    // CHECK-NEXT: rocdl.update.dpp
    // CHECK-SAME: with 274, 15, 15, true : f32
    // CHECK-NEXT: llvm.intr.maxnum
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`, `CHECK-SAME`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`, `CHECK-SAME`），从而保证后端/代码生成结果稳定。

### Lines 204-206
```mlir
    // CHECK-NEXT: rocdl.update.dpp
    // CHECK-SAME: with 273, 15, 15, true : f32
    // CHECK-NEXT: llvm.intr.maxnum
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`, `CHECK-SAME`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`, `CHECK-SAME`），从而保证后端/代码生成结果稳定。

### Lines 208-210
```mlir
    // CHECK-NEXT: rocdl.update.dpp
    // CHECK-SAME: with 322, 10, 15, true : f32
    // CHECK-NEXT: llvm.intr.maxnum
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`, `CHECK-SAME`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`, `CHECK-SAME`），从而保证后端/代码生成结果稳定。

### Lines 212-214
```mlir
    // CHECK-NEXT: rocdl.update.dpp
    // CHECK-SAME: with 323, 15, 15, true : f32
    // CHECK-NEXT: llvm.intr.maxnum
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`, `CHECK-SAME`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`, `CHECK-SAME`），从而保证后端/代码生成结果稳定。

### Lines 216-224
```mlir
    // CHECK: rocdl.readlane
    %0 = "tt.reduce"(%arg0) <{axis = 0 : i32}> ({
    ^bb0(%arg1: f32, %arg2: f32):
      %1 = arith.maxnumf %arg1, %arg2 : f32
      tt.reduce.return %1 : f32
    }) : (tensor<64xf32, #blocked3>) -> f32
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `rocdl.readlane`, `tt.reduce`, `arith.maxnumf`, `tt.reduce.return`, `tt.return`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `rocdl.readlane`、`tt.reduce`、`arith.maxnumf`、`tt.reduce.return`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 226
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 228-233
```mlir
#blocked4 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: reduce_xor_max
  tt.func @reduce_xor_max(%arg0: tensor<32xf32, #blocked4>) {
    // stride 16: CDNA fallback to bpermute
    // CHECK: rocdl.ds_bpermute
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_xor_max`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_xor_max`。

### Lines 235-238
```mlir
    // stride 8: ROW_ROR:8 (0x128 = 296)
    // CHECK: rocdl.update.dpp
    // CHECK-SAME: with 296, 15, 15, false : i32
    // CHECK: llvm.intr.maxnum
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-SAME`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-SAME`），从而保证后端/代码生成结果稳定。

### Lines 240-245
```mlir
    // stride 4: ROW_HALF_MIRROR (0x141 = 321) + quad_perm xor 3 (27)
    // CHECK: rocdl.update.dpp
    // CHECK-SAME: with 321, 15, 15, false : i32
    // CHECK: rocdl.update.dpp
    // CHECK-SAME: with 27, 15, 15, false : i32
    // CHECK: llvm.intr.maxnum
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-SAME`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-SAME`），从而保证后端/代码生成结果稳定。

### Lines 247-250
```mlir
    // stride 2: quad_perm xor 2 (78)
    // CHECK: rocdl.update.dpp
    // CHECK-SAME: with 78, 15, 15, false : i32
    // CHECK: llvm.intr.maxnum
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-SAME`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-SAME`），从而保证后端/代码生成结果稳定。

### Lines 252-262
```mlir
    // stride 1: quad_perm xor 1 (177)
    // CHECK: rocdl.update.dpp
    // CHECK-SAME: with 177, 15, 15, false : i32
    %0 = "tt.reduce"(%arg0) <{axis = 0 : i32}> ({
    ^bb0(%arg1: f32, %arg2: f32):
      %1 = arith.maxnumf %arg1, %arg2 : f32
      tt.reduce.return %1 : f32
    }) : (tensor<32xf32, #blocked4>) -> f32
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `rocdl.update.dpp`, `tt.reduce`, `arith.maxnumf`, `tt.reduce.return`, `tt.return`. Embedded check comments (CHECK×1, CHECK-SAME×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `rocdl.update.dpp`、`tt.reduce`、`arith.maxnumf`、`tt.reduce.return`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-SAME×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 264
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 266-273
```mlir
#linear = #ttg.linear<{register = [[0, 1]], lane = [[1, 0], [0, 0], [2, 0], [4, 0], [8, 0]], warp = [], block = []}>
#slice = #ttg.slice<{dim = 0, parent = #linear}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // GFX1250-LABEL: reduce_xor_row_xmask
  tt.func @reduce_xor_row_xmask(%arg0: tensor<16x2xf32, #linear>) {
    // stride 16
    // GFX1250-NOT: rocdl.ds_bpermute
    // GFX1250: rocdl.permlanex16
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_xor_row_xmask`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_xor_row_xmask`。

### Lines 275-277
```mlir
    // stride 8: ROW_XMASK:8
    // GFX1250: rocdl.update.dpp
    // GFX1250-SAME: with 360, 15, 15, false
```
**EN:** This comment block provides context for the surrounding test logic: `// stride 8: ROW_XMASK:8`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// stride 8: ROW_XMASK:8`。

### Lines 279-281
```mlir
    // stride 4: ROW_XMASK:4
    // GFX1250: rocdl.update.dpp
    // GFX1250-SAME: with 356, 15, 15, false
```
**EN:** This comment block provides context for the surrounding test logic: `// stride 4: ROW_XMASK:4`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// stride 4: ROW_XMASK:4`。

### Lines 283-293
```mlir
    // stride 1: ROW_XMASK:1
    // GFX1250: rocdl.update.dpp
    // GFX1250-SAME: with 353, 15, 15, false
    %0 = "tt.reduce"(%arg0) <{axis = 0 : i32}> ({
    ^bb0(%arg1: f32, %arg2: f32):
      %1 = arith.maxnumf %arg1, %arg2 : f32
      tt.reduce.return %1 : f32
    }) : (tensor<16x2xf32, #linear>) -> tensor<2xf32, #slice>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `rocdl.update.dpp`, `tt.reduce`, `arith.maxnumf`, `tt.reduce.return`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.update.dpp`, `tt.reduce`, `arith.maxnumf`, `tt.reduce.return`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 295
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 297-307
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: atomicrmw_scope_memsemantics
  tt.func @atomicrmw_scope_memsemantics(%arg0 : tensor<128x!tt.ptr<f32>, #blocked0>, %arg1 : tensor<128xi1, #blocked0>, %arg2 : tensor<128xf32, #blocked0>) {
    // relaxed
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} monotonic
    %0 = tt.atomic_rmw fadd, relaxed, sys, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) monotonic
    %1 = tt.atomic_rmw fadd, relaxed, gpu, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"workgroup"}}) monotonic
    %2 = tt.atomic_rmw fadd, relaxed, cta, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomicrmw_scope_memsemantics`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomicrmw_scope_memsemantics`。

### Lines 309-315
```mlir
    // acquire
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} acquire
    %3 = tt.atomic_rmw fadd, acquire, sys, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) acquire
    %4 = tt.atomic_rmw fadd, acquire, gpu, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"workgroup"}}) acquire
    %5 = tt.atomic_rmw fadd, acquire, cta, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
```
**EN:** This block defines the test function(s) and exercises operations such as `llvm.atomicrmw`, `tt.atomic_rmw`, `tt.ptr`. Embedded check comments (CHECK×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `llvm.atomicrmw`、`tt.atomic_rmw`、`tt.ptr` 这样的操作。 其中嵌入的检查注释（CHECK×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 317-323
```mlir
    // release
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} release
    %6 = tt.atomic_rmw fadd, release, sys, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) release
    %7 = tt.atomic_rmw fadd, release, gpu, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"workgroup"}}) release
    %8 = tt.atomic_rmw fadd, release, cta, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
```
**EN:** This block defines the test function(s) and exercises operations such as `llvm.atomicrmw`, `tt.atomic_rmw`, `tt.ptr`. Embedded check comments (CHECK×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `llvm.atomicrmw`、`tt.atomic_rmw`、`tt.ptr` 这样的操作。 其中嵌入的检查注释（CHECK×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 325-331
```mlir
    // acq_rel
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} acq_rel
    %9 = tt.atomic_rmw fadd, acq_rel, sys, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) acq_rel
    %10 = tt.atomic_rmw fadd, acq_rel, gpu, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"workgroup"}}) acq_rel
    %11 = tt.atomic_rmw fadd, acq_rel, cta, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
```
**EN:** This block defines the test function(s) and exercises operations such as `llvm.atomicrmw`, `tt.atomic_rmw`, `tt.ptr`. Embedded check comments (CHECK×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `llvm.atomicrmw`、`tt.atomic_rmw`、`tt.ptr` 这样的操作。 其中嵌入的检查注释（CHECK×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 333-335
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 337
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 339-342
```mlir
#blocked5 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: atomic_runtime_lds_reduction
  tt.func @atomic_runtime_lds_reduction(%arg0 : tensor<64x!tt.ptr<f32>, #blocked5>, %arg2 : tensor<64xf32, #blocked5>) {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_runtime_lds_reduction`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_runtime_lds_reduction`。

### Lines 344-355
```mlir
    // CHECK-COUNT-7: rocdl.update.dpp
    // CHECK: llvm.bitcast
    // CHECK-COUNT: llvm.amdgcqn.ds.permute
    // CHECK: llvm.bitcast
    // CHECK: llvm.ptrtoint
    // CHECK: llvm.bitcast
    // CHECK-COUNT-2: llvm.amdgcn.ds.permute
    // CHECK: llvm.bitcast
    // CHECK: llvm.inttoptr
    // CHECK: rocdl.ballot
    // CHECK: llvm.ptrtoint
    // CHECK: rocdl.ballot
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-COUNT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-COUNT`），从而保证后端/代码生成结果稳定。

### Lines 357-363
```mlir
    // loop body:
    // CHECK: llvm.bitcast
    // CHECK-COUNT-2: llvm.amdgcn.readfirstlane
    // CHECK: llvm.bitcast
    // CHECK: rocdl.ballot
    // CHECK: rocdl.mbcnt.lo
    // CHECK: rocdl.mbcnt.hi
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-COUNT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-COUNT`），从而保证后端/代码生成结果稳定。

### Lines 365-376
```mlir
    // share info:
    // 1. address
    // CHECK: llvm.bitcast
    // CHECK-COUNT-2: llvm.amdgcn.ds.permute
    // CHECK: llvm.bitcast
    // 2. value
    // CHECK: llvm.amdgcn.ds.permute
    // CHECK: llvm.bitcast
    // 3. packed methadata
    // CHECK: llvm.bitcast
    // CHECK: llvm.amdgcn.ds.permute
    // CHECK: llvm.bitcast
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-COUNT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-COUNT`），从而保证后端/代码生成结果稳定。

### Line 378
```mlir
    // CHECK: rocdl.ballot
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 380-381
```mlir
    // reduction:
    // CHECK-COUNT-6: llvm.amdgcn.ds.bpermute
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-COUNT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-COUNT`），从而保证后端/代码生成结果稳定。

### Lines 383-388
```mlir
    // CHECK: inttoptr
    // CHECK: llvm.atomicrmw
    %0 = tt.atomic_rmw fadd, relaxed, gpu, %arg0, %arg2 {allocation.offset = 0 : i32} : (tensor<64x!tt.ptr<f32>, #blocked5>, tensor<64xf32, #blocked5>) -> tensor<64xf32, #blocked5>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `llvm.atomicrmw`, `tt.atomic_rmw`, `allocation.offset`, `tt.ptr`, `tt.return`. Embedded check comments (CHECK×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `llvm.atomicrmw`、`tt.atomic_rmw`、`allocation.offset`、`tt.ptr`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 390
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 392-400
```mlir
// CHECK-LABEL: v_dot_i8
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @v_dot_i8(%arg0: tensor<16x16xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>, %arg1: tensor<16x16xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>, %arg2: tensor<16x16xi32, #blocked>) {
    // CHECK-COUNT-4: llvm.call_intrinsic "llvm.amdgcn.sdot4"
    %0 = tt.dot %arg0, %arg1, %arg2, inputPrecision = ieee : tensor<16x16xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<16x16xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<16x16xi32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `v_dot_i8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `v_dot_i8`。

### Line 402
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 404-412
```mlir
// CHECK-LABEL: v_dot_fp16
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @v_dot_fp16(%arg0: tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>, %arg1: tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>, %arg2: tensor<16x16xf32, #blocked>) {
    // CHECK-COUNT-8: llvm.call_intrinsic "llvm.amdgcn.fdot2"
    %0 = tt.dot %arg0, %arg1, %arg2, inputPrecision = ieee : tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<16x16xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `v_dot_fp16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `v_dot_fp16`。

### Line 414
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 416-424
```mlir
// CHECK-LABEL: v_dot_fp16_fp16
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @v_dot_fp16_fp16(%arg0: tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>, %arg1: tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>, %arg2: tensor<16x16xf16, #blocked>) {
    // CHECK-COUNT-16: llvm.call_intrinsic "llvm.fmuladd.f16"
    %0 = tt.dot %arg0, %arg1, %arg2, inputPrecision = ieee : tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<16x16xf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `v_dot_fp16_fp16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `v_dot_fp16_fp16`。

### Line 426
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 428-442
```mlir
// CHECK-LABEL: amd_rotating_shared_layout
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
#shared = #ttg.amd_rotating_shared<{vec = 1, perPhase = 1, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @amd_rotating_shared_layout(%arg0: tensor<64x64xf16, #blocked>) {
    // CHECK-COUNT-16: llvm.store {{.*}} : vector<1xf16>, !llvm.ptr<3>
    %0 = ttg.local_alloc %arg0 : (tensor<64x64xf16, #blocked>) -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    // CHECK-COUNT-16: llvm.load {{.*}} : !llvm.ptr<3> -> vector<1xf16>
    %1 = ttg.local_load %0 : !ttg.memdesc<64x64xf16, #shared, #smem, mutable> -> tensor<64x64xf16, #blocked>
    // CHECK-COUNT-16: llvm.store {{.*}} : vector<1xf16>, !llvm.ptr<3>
    ttg.local_store %1, %0 : tensor<64x64xf16, #blocked> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `amd_rotating_shared_layout`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `amd_rotating_shared_layout`。

### Line 444
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 446-463
```mlir
// CHECK-LABEL: amd_rotating_subview_shared_layout
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
#shared = #ttg.amd_rotating_shared<{vec = 4, perPhase = 1, maxPhase = 16, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @amd_rotating_subview_shared_layout(%arg0: tensor<64x64xf16, #blocked>) {
    %c0_i32 = arith.constant 0 : i32
    %c16_i32 = arith.constant 16 : i32
    // CHECK-COUNT-16: llvm.store {{.*}} : vector<1xf16>, !llvm.ptr<3>
    %0 = ttg.local_alloc %arg0 : (tensor<64x64xf16, #blocked>) -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    %1 = ttg.memdesc_subslice %0 [0, 16]  : !ttg.memdesc<64x64xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 64x64>
    // CHECK-COUNT-4: llvm.load {{.*}} : !llvm.ptr<3> -> vector<1xf16>
    %2 = ttg.local_load %1 : !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 64x64> -> tensor<64x16xf16, #blocked>
    // CHECK-COUNT-4: llvm.store {{.*}} : vector<1xf16>, !llvm.ptr<3>
    ttg.local_store %2, %1 : tensor<64x16xf16, #blocked> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 64x64>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `amd_rotating_subview_shared_layout`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `amd_rotating_subview_shared_layout`。

### Line 465
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 467-477
```mlir
// CHECK-LABEL: padded_shared_layout
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
#shared = #ttg.padded_shared<[128:+4, 256:+8] {order = [1, 0], shape = [64, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @padded_shared_layout(%arg0: tensor<64x64xf16, #blocked>) {
    // CHECK-DAG: %[[CST0:.+]] = llvm.mlir.constant(0 : i32)
    // CHECK-DAG: %[[CST3:.+]] = llvm.mlir.constant(3 : i32)
    // CHECK-DAG: %[[CST4:.+]] = llvm.mlir.constant(4 : i32)
    // CHECK-DAG: %[[CST8:.+]] = llvm.mlir.constant(8 : i32)
    // CHECK-DAG: %[[CST9:.+]] = llvm.mlir.constant(9 : i32)
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `padded_shared_layout`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `padded_shared_layout`。

### Lines 479-487
```mlir
    //      CHECK: %[[SHR0:.+]] = llvm.lshr %[[ADD:.+]], %[[CST8]] : i32
    // CHECK-NEXT: %[[SHL0:.+]] = llvm.shl %[[SHR0]], %[[CST3]] : i32
    // CHECK-NEXT: %[[ADD0:.+]] = llvm.add %[[SHL0]], %[[CST0]] : i32
    // CHECK-NEXT: %[[SHR1:.+]] = llvm.lshr %[[ADD]], %[[CST9]] : i32
    // CHECK-NEXT: %[[SHL1:.+]] = llvm.shl %[[SHR1]], %[[CST4]] : i32
    // CHECK-NEXT: %[[ADD1:.+]] = llvm.add %[[ADD0]], %[[SHL1]] : i32
    // CHECK-NEXT: %[[ADD2:.+]] = llvm.add %[[ADD]], %[[ADD1]] : i32
    // CHECK-NEXT: %[[ADD3:.+]] = llvm.add %[[ADD2]], %[[CST0]] : i32
    // CHECK: llvm.getelementptr inbounds %{{.+}}[%[[ADD3]]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 489-493
```mlir
    // CHECK-COUNT-16: llvm.store {{.*}} : vector<1xf16>, !llvm.ptr<3>
    %0 = ttg.local_alloc %arg0 : (tensor<64x64xf16, #blocked>) -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation. Embedded check comments (CHECK-COUNT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-COUNT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 495
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 497-511
```mlir
// CHECK-LABEL: padded_shared_layout_with_linear_component
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
#shared = #ttg.padded_shared<[128:+4, 256:+8] {order = [1, 0], shape = [64, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @padded_shared_layout_with_linear_component(%arg0: tensor<64x64xf16, #blocked>) {
    // CHECK-COUNT-16: llvm.store {{.*}} : vector<1xf16>, !llvm.ptr<3>
    %0 = ttg.local_alloc %arg0 : (tensor<64x64xf16, #blocked>) -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    // CHECK-COUNT-16: llvm.load {{.*}} : !llvm.ptr<3> -> vector<1xf16>
    %2 = ttg.local_load %0 : !ttg.memdesc<64x64xf16, #shared, #smem, mutable> -> tensor<64x64xf16, #blocked>
    // CHECK-COUNT-16: llvm.store {{.*}} : vector<1xf16>, !llvm.ptr<3>
    ttg.local_store %2, %0 : tensor<64x64xf16, #blocked> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `padded_shared_layout_with_linear_component`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `padded_shared_layout_with_linear_component`。

### Line 513
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 515-526
```mlir
// GFX950-LABEL: padded_shared_layout_subview
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
#shared = #ttg.padded_shared<[128:+4] {order = [1, 0], shape = [64, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @padded_shared_layout_subview(%arg0: !ttg.memdesc<2x64x64xf16, #shared, #smem, mutable>) {
    %c0_i32 = arith.constant 0 : i32
    %c1_i32 = arith.constant 1 : i32
    // Skip three constants from the stride calculation
    // GFX950: llvm.mlir.constant
    // GFX950: llvm.mlir.constant
    // GFX950: llvm.mlir.constant
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `padded_shared_layout_subview`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `padded_shared_layout_subview`。

### Lines 528-530
```mlir
    // GFX950-DAG: %[[CST0:.+]] = llvm.mlir.constant(0 : i32)
    // GFX950-DAG: %[[CST7:.+]] = llvm.mlir.constant(7 : i32)
    // GFX950-DAG: %[[CST2:.+]] = llvm.mlir.constant(2 : i32)
```
**EN:** This comment block provides context for the surrounding test logic: `// GFX950-DAG: %[[CST0:.+]] = llvm.mlir.constant(0 : i32)`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// GFX950-DAG: %[[CST0:.+]] = llvm.mlir.constant(0 : i32)`。

### Lines 532-536
```mlir
    // GFX950-DAG: %[[SHR0:.+]] = llvm.lshr %[[ADD:.+]], %[[CST7]] : i32
    // GFX950-NEXT: %[[SHL0:.+]] = llvm.shl %[[SHR0]], %[[CST2]] : i32
    // GFX950-NEXT: %[[ADD1:.+]] = llvm.add %[[CST0]], %[[SHL0]] : i32
    // GFX950-NEXT: %[[ADD2:.+]] = llvm.add %[[ADD]], %[[ADD1]] : i32
    // GFX950: llvm.getelementptr %{{.+}}[%[[ADD2]]]
```
**EN:** This comment block provides context for the surrounding test logic: `// GFX950-DAG: %[[SHR0:.+]] = llvm.lshr %[[ADD:.+]], %[[CST7]] : i32`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// GFX950-DAG: %[[SHR0:.+]] = llvm.lshr %[[ADD:.+]], %[[CST7]] : i32`。

### Lines 538-541
```mlir
    %1 = ttg.memdesc_index %arg0[%c1_i32] : !ttg.memdesc<2x64x64xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `ttg.memdesc_index`, `ttg.memdesc`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.memdesc_index`, `ttg.memdesc`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 543
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 545-548
```mlir
// CHECK-LABEL: padded_shared_layout_vectorization
// CHECK-NOT: llvm.load
// CHECK: llvm.load {{.*}} !llvm.ptr<3> -> vector<8xf16>
// CHECK-NOT: llvm.load
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-LABEL`, `CHECK-NOT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-LABEL`, `CHECK-NOT`），从而保证后端/代码生成结果稳定。

### Lines 550-561
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[128:+4] {order = [1, 0], shape = [16, 32]}>
#smem = #ttg.shared_memory
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 4], instrShape = [16, 16, 32], isTransposed = true}>
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @padded_shared_layout_vectorization(%arg0: tensor<16x32xf16, #blocked>) {
    %0 = ttg.local_alloc %arg0 : (tensor<16x32xf16, #blocked>) -> !ttg.memdesc<16x32xf16, #shared, #smem, mutable>
    %1 = ttg.local_load %0: !ttg.memdesc<16x32xf16, #shared, #smem, mutable, 16x32> -> tensor<16x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    ttg.local_store %1, %0 : tensor<16x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> -> !ttg.memdesc<16x32xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `padded_shared_layout_vectorization`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `padded_shared_layout_vectorization`。

### Line 563
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 565-576
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[4:+4] {offset=[[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [1, 0], [2, 0], [4, 0], [8, 0]], block=[]}>
#smem = #ttg.shared_memory
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 4], instrShape = [16, 16, 32], isTransposed = true}>
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: padded_shared_layout_vectorization_limited_by_min_interval
  tt.func @padded_shared_layout_vectorization_limited_by_min_interval(%arg0: tensor<16x32xf16, #blocked>) {
    // CHECK-NOT: llvm.store
    // CHECK: llvm.store {{.*}} : vector<4xf16>
    // CHECK: llvm.store {{.*}} : vector<4xf16>
    // CHECK-NOT: llvm.store
    %0 = ttg.local_alloc %arg0 : (tensor<16x32xf16, #blocked>) -> !ttg.memdesc<16x32xf16, #shared, #smem, mutable>
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `padded_shared_layout_vectorization_limited_by_min_interval`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `padded_shared_layout_vectorization_limited_by_min_interval`。

### Lines 578-582
```mlir
    // CHECK-NOT: llvm.load
    // CHECK: llvm.load {{.*}} !llvm.ptr<3> -> vector<4xf16>
    // CHECK: llvm.load {{.*}} !llvm.ptr<3> -> vector<4xf16>
    // CHECK-NOT: llvm.load
    %1 = ttg.local_load %0: !ttg.memdesc<16x32xf16, #shared, #smem, mutable, 16x32> -> tensor<16x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
```
**EN:** This block defines the test function(s) and exercises operations such as `llvm.load`, `llvm.ptr`, `ttg.local_load`, `ttg.memdesc`. Embedded check comments (CHECK×2, CHECK-NOT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `llvm.load`、`llvm.ptr`、`ttg.local_load`、`ttg.memdesc` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-NOT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 584-591
```mlir
    // CHECK-NOT: llvm.store
    // CHECK: llvm.store {{.*}} : vector<4xf16>
    // CHECK: llvm.store {{.*}} : vector<4xf16>
    // CHECK-NOT: llvm.store
    ttg.local_store %1, %0 : tensor<16x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> -> !ttg.memdesc<16x32xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `llvm.store`, `ttg.local_store`, `ttg.memdesc`, `tt.return`. Embedded check comments (CHECK×2, CHECK-NOT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `llvm.store`、`ttg.local_store`、`ttg.memdesc`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-NOT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 593
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 595
```mlir
// CHECK-LABEL: padded_shared_layout_subslice_load_store
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-LABEL`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-LABEL`），从而保证后端/代码生成结果稳定。

### Lines 597-638
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [16, 4], warpsPerCTA = [2, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [32, 32]}>
#smem = #ttg.shared_memory
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 1], instrShape = [16, 16, 32], isTransposed = true}>
module attributes {"ttg.target" = "hip:gfx942", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @padded_shared_layout_subslice_load_store(%arg0: tensor<32x32xf16, #blocked>) {
    // CHECK: %[[SUBSLICE_CST16:.+]] = llvm.mlir.constant(16 : i32)
    // CHECK: %[[SUBSLICE_CST3:.+]] = llvm.mlir.constant(3 : i32)
    // CHECK: %[[SUBSLICE_CST2:.+]] = llvm.mlir.constant(2 : i32)
    // CHECK: %[[SUBSLICE_CST6:.+]] = llvm.mlir.constant(6 : i32)
    // CHECK: %[[SUBSLICE_CST0:.+]] = llvm.mlir.constant(0 : i32)
    // CHECK: llvm.store {{.*}} : vector<8xf16>, !llvm.ptr<3>
    // CHECK-NOT: llvm.store
    %0 = ttg.local_alloc %arg0 : (tensor<32x32xf16, #blocked>) -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable>
    %1 = ttg.memdesc_subslice %0 [16, 0]  : !ttg.memdesc<32x32xf16, #shared, #smem, mutable> -> !ttg.memdesc<16x32xf16, #shared, #smem, mutable, 32x32>
    // CHECK: rocdl.s.waitcnt
    // CHECK-NEXT: rocdl.s.barrier
    // CHECK: %[[AFF_I8:.+]] = llvm.mul %{{.+}}, %[[SUBSLICE_CST2]] : i32
    // CHECK-NEXT: %[[AFF_SHR:.+]] = llvm.lshr %[[AFF_I8]], %[[SUBSLICE_CST6]] : i32
    // CHECK-NEXT: %[[AFF_SHL:.+]] = llvm.shl %[[AFF_SHR]], %[[SUBSLICE_CST3]] : i32
    // CHECK-NEXT: %[[AFF_PAD0:.+]] = llvm.add %[[AFF_SHL]], %[[SUBSLICE_CST0]] : i32
    // CHECK-NEXT: %[[PAD_AFF:.+]] = llvm.add %[[AFF_I8]], %[[AFF_PAD0]] : i32
    // CHECK: %[[DYN_BASE:.+]] = llvm.xor %{{.+}}, %[[SUBSLICE_CST0]] : i32
    // CHECK-NEXT: %[[DYN_SHR:.+]] = llvm.lshr %[[DYN_BASE]], %[[SUBSLICE_CST6]] : i32
    // CHECK-NEXT: %[[DYN_SHL:.+]] = llvm.shl %[[DYN_SHR]], %[[SUBSLICE_CST3]] : i32
    // CHECK-NEXT: %[[DYN_PAD0:.+]] = llvm.add %[[DYN_SHL]], %[[SUBSLICE_CST0]] : i32
    // CHECK-NEXT: %[[PAD_DYN:.+]] = llvm.add %[[DYN_BASE]], %[[DYN_PAD0]] : i32
    // CHECK-NEXT: %[[PADDED_OFF:.+]] = llvm.add %[[PAD_DYN]], %[[PAD_AFF]] : i32
    // CHECK-NEXT: %[[INNER_OFF0:.+]] = llvm.add %[[PADDED_OFF]], %[[SUBSLICE_CST0]] : i32
    // CHECK-NEXT: %[[LOAD_PTR0:.+]] = llvm.getelementptr inbounds %{{.+}}[%[[INNER_OFF0]]] : (!llvm.ptr<3>, i32) -> !llvm.ptr<3>, i8
    // CHECK-NEXT: llvm.load %[[LOAD_PTR0]] : !llvm.ptr<3> -> vector<4xf16>
    // CHECK: %[[INNER_OFF1:.+]] = llvm.add %[[PADDED_OFF]], %{{.+}} : i32
    // CHECK-NEXT: %[[LOAD_PTR1:.+]] = llvm.getelementptr inbounds %{{.+}}[%[[INNER_OFF1]]] : (!llvm.ptr<3>, i32) -> !llvm.ptr<3>, i8
    // CHECK-NEXT: llvm.load %[[LOAD_PTR1]] : !llvm.ptr<3> -> vector<4xf16>
    // CHECK-NOT: llvm.load
    %2 = ttg.local_load %1: !ttg.memdesc<16x32xf16, #shared, #smem, mutable, 32x32> -> tensor<16x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
    // CHECK-COUNT-2: llvm.store {{.*}} : vector<4xf16>, !llvm.ptr<3>
    // CHECK-NOT: llvm.store
    ttg.local_store %2, %1 : tensor<16x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> -> !ttg.memdesc<16x32xf16, #shared, #smem, mutable, 32x32>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `padded_shared_layout_subslice_load_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `padded_shared_layout_subslice_load_store`。

### Line 640
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 642-653
```mlir
// GFX950-LABEL: reduce_32x32
// GFX950: llvm.call_intrinsic "llvm.amdgcn.permlane32.swap"
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @reduce_32x32(%arg0: tensor<64x32xf32, #ttg.amd_mfma<{versionMajor = 4, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>>) {
%3101 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
^bb0(%arg24: f32, %arg25: f32):
  %3166 = "arith.maxnumf"(%arg24, %arg25) <{fastmath = #arith.fastmath<none>}> : (f32, f32) -> f32
  "tt.reduce.return"(%3166) : (f32) -> ()
}) : (tensor<64x32xf32, #ttg.amd_mfma<{versionMajor = 4, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>>) -> tensor<64xf32, #ttg.slice<{dim = 1, parent = #ttg.amd_mfma<{versionMajor = 4, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [32, 32, 8], isTransposed = true}>}>>
  tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_32x32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_32x32`。

### Line 655
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 657-669
```mlir
// GFX950-LABEL: reduce_16x16
// GFX950: llvm.call_intrinsic "llvm.amdgcn.permlane32.swap"
// GFX950: llvm.call_intrinsic "llvm.amdgcn.permlane16.swap"
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @reduce_16x16(%arg0: tensor<64x16xf32, #ttg.amd_mfma<{versionMajor = 4, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true}>>){
%1 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
^bb0(%arg24: f32, %arg25: f32):
  %3166 = "arith.maxnumf"(%arg24, %arg25) <{fastmath = #arith.fastmath<none>}> : (f32, f32) -> f32
  "tt.reduce.return"(%3166) : (f32) -> ()
}) : (tensor<64x16xf32, #ttg.amd_mfma<{versionMajor = 4, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true}>>) -> tensor<64xf32, #ttg.slice<{dim = 1, parent = #ttg.amd_mfma<{versionMajor = 4, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16], isTransposed = true}>}>>
  tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_16x16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_16x16`。

### Line 671
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 672-689
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @atomic_kernel_bf16(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) release
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) acquire
    %cst = arith.constant dense<true> : tensor<1024xi1, #blocked>
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<1024xbf16, #blocked>
    %c1024_i32 = arith.constant 1024 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c1024_i32 : i32
    %2 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
    %3 = tt.addptr %arg0, %1 : !tt.ptr<bf16>, i32
    %4 = tt.splat %3 : !tt.ptr<bf16> -> tensor<1024x!tt.ptr<bf16>, #blocked>
    %5 = tt.addptr %4, %2 : tensor<1024x!tt.ptr<bf16>, #blocked>, tensor<1024xi32, #blocked>
    %6 = tt.atomic_rmw fadd, acq_rel, gpu, %5, %cst_0, %cst : (tensor<1024x!tt.ptr<bf16>, #blocked>, tensor<1024xbf16, #blocked>, tensor<1024xi1, #blocked>) -> tensor<1024xbf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_kernel_bf16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_kernel_bf16`。

### Line 691
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 693-712
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @atomic_kernel_bf16(%arg0: !tt.ptr<bf16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) release
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) monotonic
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) monotonic
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) acquire
    %cst = arith.constant dense<true> : tensor<1024xi1, #blocked>
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<1024xbf16, #blocked>
    %c1024_i32 = arith.constant 1024 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c1024_i32 : i32
    %2 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
    %3 = tt.addptr %arg0, %1 : !tt.ptr<bf16>, i32
    %4 = tt.splat %3 : !tt.ptr<bf16> -> tensor<1024x!tt.ptr<bf16>, #blocked>
    %5 = tt.addptr %4, %2 : tensor<1024x!tt.ptr<bf16>, #blocked>, tensor<1024xi32, #blocked>
    %6 = tt.atomic_rmw fadd, acq_rel, gpu, %5, %cst_0, %cst : (tensor<1024x!tt.ptr<bf16>, #blocked>, tensor<1024xbf16, #blocked>, tensor<1024xi1, #blocked>) -> tensor<1024xbf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_kernel_bf16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_kernel_bf16`。

### Line 714
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 716-740
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @atomic_kernel_fp32(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) release
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) monotonic
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) monotonic
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) monotonic
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) monotonic
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) monotonic
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) monotonic
    // CHECK: llvm.atomicrmw {{.*}}, {{.*}} syncscope({{"agent"}}) acquire
    %cst = arith.constant dense<true> : tensor<1024xi1, #blocked>
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<1024xf32, #blocked>
    %c1024_i32 = arith.constant 1024 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c1024_i32 : i32
    %2 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
    %3 = tt.splat %1 : i32 -> tensor<1024xi32, #blocked>
    %4 = arith.addi %3, %2 : tensor<1024xi32, #blocked>
    %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>, #blocked>
    %6 = tt.addptr %5, %4 : tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xi32, #blocked>
    %7 = tt.atomic_rmw fadd, acq_rel, gpu, %6, %cst_0, %cst : (tensor<1024x!tt.ptr<f32>, #blocked>, tensor<1024xf32, #blocked>, tensor<1024xi1, #blocked>) -> tensor<1024xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_kernel_fp32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_kernel_fp32`。

### Line 742
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 744-751
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // Make sure there is no attribute attached to the function.
  // CHECK-LABEL: func_attr({{.*}}) {
  // CHECK-NEXT: llvm.return
  tt.func @func_attr() {
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `func_attr`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `func_attr`。

### Line 753
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 755-766
```mlir
#linear = #ttg.linear<{register=[[64]], lane=[[1], [2], [4], [8], [16], [32]], warp=[], block=[]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
    // COMMON-LABEL: clampf_f32
    tt.func public @clampf_f32(%x : tensor<128xf32, #linear>, %ptr : tensor<128x!tt.ptr<f32>, #linear>) {
        %min = arith.constant dense<-1.00000e+00> : tensor<128xf32, #linear>
        %max = arith.constant dense<1.000000e+00> : tensor<128xf32, #linear>
        // COMMON: rocdl.fmed3
        %0 = tt.clampf %x, %min, %max, propagateNan = none : tensor<128xf32, #linear>
        // COMMON-NOT: llvm.select
        tt.store %ptr, %0 : tensor<128x!tt.ptr<f32>, #linear>
        tt.return
    }
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `clampf_f32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `clampf_f32`。

### Lines 768-777
```mlir
    // COMMON-LABEL: clampf_f16
    tt.func public @clampf_f16(%x : tensor<128xf16, #linear>, %ptr : tensor<128x!tt.ptr<f16>, #linear>) {
        %min = arith.constant dense<-1.00000e+00> : tensor<128xf16, #linear>
        %max = arith.constant dense<1.000000e+00> : tensor<128xf16, #linear>
        // COMMON: rocdl.fmed3
        %0 = tt.clampf %x, %min, %max, propagateNan = none : tensor<128xf16, #linear>
        // COMMON-NOT: llvm.select
        tt.store %ptr, %0 : tensor<128x!tt.ptr<f16>, #linear>
        tt.return
    }
```
**EN:** This block defines `clampf_f16` and exercises tensor stores.
**CN:** 这一块定义了 `clampf_f16`，并覆盖 张量存储。

### Lines 779-789
```mlir
    // COMMON-LABEL: clampf_f32_propagateNan
    tt.func public @clampf_f32_propagateNan(%x : tensor<128xf32, #linear>, %ptr : tensor<128x!tt.ptr<f32>, #linear>) {
        %min = arith.constant dense<-1.00000e+00> : tensor<128xf32, #linear>
        %max = arith.constant dense<1.000000e+00> : tensor<128xf32, #linear>
        // COMMON: [[MED:%.*]] = rocdl.fmed3 [[X:%.*]], %{{.*}}, %{{.*}} : f32
        %0 = tt.clampf %x, %min, %max, propagateNan = all : tensor<128xf32, #linear>
        // COMMON: [[ISNAN:%.*]] = llvm.fcmp "une" [[X]], [[X]] : f32
        // COMMON: llvm.select [[ISNAN]], [[X]], [[MED]] : i1, f32
        tt.store %ptr, %0 : tensor<128x!tt.ptr<f32>, #linear>
        tt.return
    }
```
**EN:** This block defines `clampf_f32_propagateNan` and exercises tensor stores.
**CN:** 这一块定义了 `clampf_f32_propagateNan`，并覆盖 张量存储。

### Lines 791-802
```mlir
    // COMMON-LABEL: clampf_bf16
    tt.func public @clampf_bf16(%x : tensor<128xbf16, #linear>, %ptr : tensor<128x!tt.ptr<bf16>, #linear>) {
        %min = arith.constant dense<-1.00000e+00> : tensor<128xbf16, #linear>
        %max = arith.constant dense<1.000000e+00> : tensor<128xbf16, #linear>
        // COMMON-NOT: rodcl.fmed3
        // COMMON: llvm.intr.maxnum
        // COMMON: llvm.intr.minnum
        %0 = tt.clampf %x, %min, %max, propagateNan = none : tensor<128xbf16, #linear>
        tt.store %ptr, %0 : tensor<128x!tt.ptr<bf16>, #linear>
        tt.return
    }
}
```
**EN:** This block defines `clampf_bf16` and exercises tensor stores.
**CN:** 这一块定义了 `clampf_bf16`，并覆盖 张量存储。

### Line 804
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 806-818
```mlir
// Make sure there is no rocdl.grid.dim.* generated when global_scratch_memory_size is 0.
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32, ttg.global_scratch_memory_size = 0 : i32, ttg.global_scratch_memory_alignment = 1 : i32} {
  // CHECK-LABEL: @test_call_zero_scratch_no_grid_ops
  // CHECK-NOT: rocdl.grid.dim
  // CHECK: llvm.call @callee_zero_scratch
  tt.func public @test_call_zero_scratch_no_grid_ops() attributes {noinline = false} {
    tt.call @callee_zero_scratch() : () -> ()
    tt.return
  }
  tt.func private @callee_zero_scratch() attributes {noinline = true} {
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_call_zero_scratch_no_grid_ops`, `callee_zero_scratch`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_call_zero_scratch_no_grid_ops`, `callee_zero_scratch`。

### Line 820
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 822-823
```mlir
/// gfx906 has the same dot intrinsics as gfx908+ (HasDot1Insts) but not
/// the compact VOP2 encoding (HasDot2Insts); LLVM handles this transparently.
```
**EN:** This comment block provides context for the surrounding test logic: `/// gfx906 has the same dot intrinsics as gfx908+ (HasDot1Insts) but not`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`/// gfx906 has the same dot intrinsics as gfx908+ (HasDot1Insts) but not`。

### Lines 825-833
```mlir
// GFX906-LABEL: v_dot_fp16_gfx906
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.target" = "hip:gfx906", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @v_dot_fp16_gfx906(%arg0: tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>, %arg1: tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>, %arg2: tensor<16x16xf32, #blocked>) {
    // GFX906-COUNT-8: llvm.call_intrinsic "llvm.amdgcn.fdot2"
    %0 = tt.dot %arg0, %arg1, %arg2, inputPrecision = ieee : tensor<16x16xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<16x16xf16, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<16x16xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `v_dot_fp16_gfx906`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `v_dot_fp16_gfx906`。

### Line 835
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 837-845
```mlir
// GFX906-LABEL: v_dot_i8_gfx906
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.target" = "hip:gfx906", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @v_dot_i8_gfx906(%arg0: tensor<16x16xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>, %arg1: tensor<16x16xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>>, %arg2: tensor<16x16xi32, #blocked>) {
    // GFX906-COUNT-4: llvm.call_intrinsic "llvm.amdgcn.sdot4"
    %0 = tt.dot %arg0, %arg1, %arg2, inputPrecision = ieee : tensor<16x16xi8, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> * tensor<16x16xi8, #ttg.dot_op<{opIdx = 1, parent = #blocked}>> -> tensor<16x16xi32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `v_dot_i8_gfx906`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `v_dot_i8_gfx906`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on AMD-specific lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 AMD 专用降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--convert-builtin-func-to-llvm`, `--check-prefixes=CHECK,COMMON`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--check-prefixes=GFX950,COMMON`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250`, `--check-prefixes=GFX1250,COMMON`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx906`, `--check-prefixes=GFX906,COMMON`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--convert-builtin-func-to-llvm`, `--check-prefixes=CHECK,COMMON`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--check-prefixes=GFX950,COMMON`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250`, `--check-prefixes=GFX1250,COMMON`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx906`, `--check-prefixes=GFX906,COMMON` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `llvm` (LLVM dialect IR), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `llvm`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`llvm`、`arith`。
