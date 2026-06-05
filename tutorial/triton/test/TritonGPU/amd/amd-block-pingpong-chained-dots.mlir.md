# amd-block-pingpong-chained-dots.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/TritonGPU/amd/amd-block-pingpong-chained-dots.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises dot-product lowering in Triton's TritonGPU transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton TritonGPU 变换中的点积降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --tritonamdgpu-block-pingpong="num-stages=4" | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×26, CHECK-LABEL×4, CHECK-NEXT×28, CHECK-NOT×5; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×26, CHECK-LABEL×4, CHECK-NEXT×28, CHECK-NOT×5；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that dot-product lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 点积降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --tritonamdgpu-block-pingpong="num-stages=4" | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --tritonamdgpu-block-pingpong="num-stages=4" | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --tritonamdgpu-block-pingpong="num-stages=4" | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-7
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 8, order = [0, 1]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 9
```mlir
  // CHECK-LABEL: chained_dots_async_loads
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-LABEL`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-LABEL`），从而保证后端/代码生成结果稳定。

### Lines 11-42
```mlir
  // CHECK: scf.for
  // CHECK-NEXT: rocdl.s.barrier
  // CHECK-NEXT: rocdl.sched.barrier 0
  // Compute Cluster1
  // CHECK: tt.dot
  // CHECK: rocdl.sched.barrier 0
  // CHECK-NEXT: ttg.async_wait
  // CHECK-NEXT: rocdl.s.setprio 1
  // CHECK-NEXT: rocdl.sched.barrier 0
  // Memory Cluster1
  // CHECK: ttg.local_load
  // CHECK: ttg.async_copy_global_to_local
  // CHECK: ttg.async_commit_group
  // CHECK: rocdl.sched.barrier 0
  // CHECK-NEXT: rocdl.s.setprio 0
  // CHECK-NEXT: amdg.memory_counter_wait ds(0)
  // CHECK-NEXT: rocdl.s.barrier
  // CHECK-NEXT: rocdl.sched.barrier 0
  // Compute Cluster2
  // CHECK: tt.dot
  // CHECK: rocdl.sched.barrier 0
  // CHECK: ttg.async_wait
  // CHECK-NEXT: rocdl.s.setprio 1
  // CHECK-NEXT: rocdl.sched.barrier 0
  // Memory Cluster2
  // CHECK: ttg.local_load
  // CHECK: ttg.async_copy_global_to_local
  // CHECK: ttg.async_commit_group
  // CHECK: rocdl.sched.barrier 0
  // CHECK-NEXT: rocdl.s.setprio 0
  // CHECK-NEXT: amdg.memory_counter_wait ds(0)
  // CHECK-NEXT: scf.yield
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 44-71
```mlir
  tt.func @chained_dots_async_loads(%arg0: tensor<64x16x!tt.ptr<f16>, #blocked>, %arg1: i32, %arg2: i32, %arg3: !ttg.async.token, %arg4: tensor<128x16xf32, #mma>, %arg5: tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>, %arg6: i32, %arg7: tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, %arg8: tensor<128x16xf32, #mma>, %arg9: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg10: tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>, %arg11: i32, %arg12: i32, %arg13: tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>) -> tensor<128x16xf32, #mma> {
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    %0 = ttg.local_alloc : () -> !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    %2 = ttg.memdesc_index %1[%c0_i32] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
    %3 = ttg.memdesc_index %0[%c0_i32] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
    %4 = ttg.memdesc_index %1[%c1_i32] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
    %5:9 = scf.for %arg14 = %c0_i32 to %arg1 step %arg2 iter_args(%arg15 = %arg4, %arg16 = %arg4, %arg17 = %arg7, %arg18 = %arg3, %arg19 = %arg3, %arg20 = %2, %arg21 = %4, %arg22 = %arg3, %arg23 = %3) -> (tensor<128x16xf32, #mma>, tensor<128x16xf32, #mma>, tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, !ttg.async.token, !ttg.async.token, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.async.token, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>)  : i32 {
      %6 = tt.dot %arg10, %arg17, %arg15 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
      %7 = ttg.async_wait %arg18 {num = 0 : i32}
      %8 = ttg.local_load %arg20 token %7 : !ttg.memdesc<64x16xf16, #shared, #smem, mutable> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
      %9 = ttg.memdesc_index %0[%arg6] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
      %10 = ttg.async_copy_global_to_local %arg0, %9 : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
      %11 = ttg.async_commit_group tokens %10
      %12 = tt.dot %arg10, %8, %arg16 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
      %13 = ttg.async_wait %arg22 {num = 0 : i32}
      %14 = ttg.local_load %arg23 token %13 : !ttg.memdesc<64x16xf16, #shared, #smem, mutable> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
      %15 = ttg.memdesc_index %1[%arg6] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
      %16 = ttg.async_copy_global_to_local %arg0, %15 : tensor<64x16x!tt.ptr<f16>, #blocked> -> <64x16xf16, #shared, #smem, mutable>
      %17 = ttg.async_commit_group tokens %16
      scf.yield %12, %6, %14, %arg19, %17, %arg21, %15, %11, %9 : tensor<128x16xf32, #mma>, tensor<128x16xf32, #mma>, tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, !ttg.async.token, !ttg.async.token, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.async.token, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
    }
    ttg.local_dealloc %1 : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    ttg.local_dealloc %0 : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    tt.return %5#0 : tensor<128x16xf32, #mma>
  }
}
```
**EN:** This block defines `chained_dots_async_loads` and exercises async global-to-shared copies, shared-memory allocation, dot products / matmul, loop-carried state.
**CN:** 这一块定义了 `chained_dots_async_loads`，并覆盖 全局到共享内存的异步拷贝、共享内存分配、点积/矩阵乘、循环携带状态。

### Line 73
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 75-79
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 8, order = [0, 1]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 81
```mlir
  // CHECK-LABEL: chained_dots_tt_loads
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-LABEL`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-LABEL`），从而保证后端/代码生成结果稳定。

### Lines 83-113
```mlir
  // CHECK-NOT: rocdl.s
  // CHECK: scf.for
  // CHECK: rocdl.s.barrier
  // CHECK-NEXT: rocdl.sched.barrier 0
  // Compute Cluster1
  // CHECK: tt.dot
  // CHECK: rocdl.sched.barrier 0
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NEXT: rocdl.s.setprio 1
  // Memory Cluster1
  // CHECK: ttg.local_store
  // CHECK: ttg.local_load
  // CHECK: tt.load
  // CHECK-NEXT: rocdl.sched.barrier 0
  // CHECK-NEXT: rocdl.s.setprio 0
  // CHECK-NEXT: amdg.memory_counter_wait ds(0)
  // CHECK-NEXT: rocdl.s.barrier
  // CHECK-NEXT: rocdl.sched.barrier 0
  // Compute Cluster2
  // CHECK: tt.dot
  // CHECK: rocdl.sched.barrier 0
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NEXT: rocdl.s.setprio 1
  // Memory Cluster2
  // CHECK: ttg.local_store
  // CHECK: ttg.local_load
  // CHECK: tt.load
  // CHECK-NEXT: rocdl.sched.barrier 0
  // CHECK-NEXT: rocdl.s.setprio 0
  // CHECK-NEXT: amdg.memory_counter_wait ds(0)
  // CHECK-NEXT: scf.yield
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`, `CHECK-NOT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`, `CHECK-NOT`），从而保证后端/代码生成结果稳定。

### Lines 115-140
```mlir
  tt.func @chained_dots_tt_loads(%arg0: tensor<64x16xf16, #blocked>, %arg1: tensor<64x16x!tt.ptr<f16>, #blocked>, %arg2: i32, %arg3: i32, %arg4: tensor<128x16xf32, #mma>, %arg5: tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>, %arg6: i32, %arg7: tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, %arg8: tensor<128x16xf32, #mma>, %arg9: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg10: tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>, %arg11: i32, %arg12: i32, %arg13: tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>) -> tensor<128x16xf32, #mma> {
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    %0 = ttg.local_alloc : () -> !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    %2 = ttg.memdesc_index %1[%c0_i32] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
    %3 = ttg.memdesc_index %0[%c0_i32] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
    %4 = ttg.memdesc_index %1[%c1_i32] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
    %5:8 = scf.for %arg14 = %c0_i32 to %arg2 step %arg3 iter_args(%arg15 = %arg4, %arg16 = %arg4, %arg17 = %arg7, %arg18 = %2, %arg19 = %4, %arg20 = %3, %arg21 = %arg0, %arg22 = %arg0) -> (tensor<128x16xf32, #mma>, tensor<128x16xf32, #mma>, tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, tensor<64x16xf16, #blocked>, tensor<64x16xf16, #blocked>)  : i32 {
      %6 = tt.dot %arg10, %arg17, %arg15 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
      ttg.local_store %arg21, %arg18 : tensor<64x16xf16, #blocked> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
      %7 = ttg.local_load %arg18 : !ttg.memdesc<64x16xf16, #shared, #smem, mutable> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
      %8 = ttg.memdesc_index %0[%arg6] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
      %9 = tt.load %arg1 : tensor<64x16x!tt.ptr<f16>, #blocked>
      %10 = tt.dot %arg10, %7, %arg16 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
      ttg.local_store %arg22, %arg20 : tensor<64x16xf16, #blocked> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
      %11 = ttg.local_load %arg20 : !ttg.memdesc<64x16xf16, #shared, #smem, mutable> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
      %12 = ttg.memdesc_index %1[%arg6] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
      %13 = tt.load %arg1 : tensor<64x16x!tt.ptr<f16>, #blocked>
      scf.yield %10, %6, %11, %arg19, %12, %8, %9, %13 : tensor<128x16xf32, #mma>, tensor<128x16xf32, #mma>, tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, tensor<64x16xf16, #blocked>, tensor<64x16xf16, #blocked>
    }
    ttg.local_dealloc %1 : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    ttg.local_dealloc %0 : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    tt.return %5#0 : tensor<128x16xf32, #mma>
  }
}
```
**EN:** This block defines `chained_dots_tt_loads` and exercises shared-memory allocation, dot products / matmul, tensor loads, loop-carried state.
**CN:** 这一块定义了 `chained_dots_tt_loads`，并覆盖 共享内存分配、点积/矩阵乘、张量加载、循环携带状态。

### Line 142
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 144-148
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 8, order = [0, 1]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 150
```mlir
  // CHECK-LABEL: reject_chained_dots_empty_mem_cluster_1
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-LABEL`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-LABEL`），从而保证后端/代码生成结果稳定。

### Lines 152-153
```mlir
  // CHECK-NOT: setprio
  // CHECK-NOT: barrier
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NOT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NOT`），从而保证后端/代码生成结果稳定。

### Lines 155-176
```mlir
  tt.func @reject_chained_dots_empty_mem_cluster_1(%arg0: tensor<64x16xf16, #blocked>, %arg1: tensor<64x16x!tt.ptr<f16>, #blocked>, %arg2: i32, %arg3: i32, %arg4: tensor<128x16xf32, #mma>, %arg5: tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>, %arg6: i32, %arg7: tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, %arg8: tensor<128x16xf32, #mma>, %arg9: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg10: tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>, %arg11: i32, %arg12: i32, %arg13: tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>) -> tensor<128x16xf32, #mma> {
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    %0 = ttg.local_alloc : () -> !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    %2 = ttg.memdesc_index %1[%c0_i32] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
    %3 = ttg.memdesc_index %0[%c0_i32] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
    %4 = ttg.memdesc_index %1[%c1_i32] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
    %5:8 = scf.for %arg14 = %c0_i32 to %arg2 step %arg3 iter_args(%arg15 = %arg4, %arg16 = %arg4, %arg17 = %arg7, %arg18 = %2, %arg19 = %4, %arg20 = %3, %arg21 = %arg0, %arg22 = %arg0) -> (tensor<128x16xf32, #mma>, tensor<128x16xf32, #mma>, tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, tensor<64x16xf16, #blocked>, tensor<64x16xf16, #blocked>)  : i32 {
      %6 = tt.dot %arg10, %arg17, %arg15 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
      %10 = tt.dot %arg10, %arg17, %arg16 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
      ttg.local_store %arg22, %arg20 : tensor<64x16xf16, #blocked> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
      %11 = ttg.local_load %arg20 : !ttg.memdesc<64x16xf16, #shared, #smem, mutable> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
      %12 = ttg.memdesc_index %1[%arg6] : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable>
      %13 = tt.load %arg1 : tensor<64x16x!tt.ptr<f16>, #blocked>
      scf.yield %10, %6, %11, %arg19, %12, %12, %13, %13 : tensor<128x16xf32, #mma>, tensor<128x16xf32, #mma>, tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable>, tensor<64x16xf16, #blocked>, tensor<64x16xf16, #blocked>
    }
    ttg.local_dealloc %1 : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    ttg.local_dealloc %0 : !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>
    tt.return %5#0 : tensor<128x16xf32, #mma>
  }
}
```
**EN:** This block defines `reject_chained_dots_empty_mem_cluster_1` and exercises shared-memory allocation, dot products / matmul, tensor loads, loop-carried state.
**CN:** 这一块定义了 `reject_chained_dots_empty_mem_cluster_1`，并覆盖 共享内存分配、点积/矩阵乘、张量加载、循环携带状态。

### Line 178
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 180-184
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [4, 1], instrShape = [32, 32, 16], isTransposed = true}>
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 8, order = [0, 1]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 186
```mlir
  // CHECK-LABEL: reject_chained_dots_empty_mem_cluster_2
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-LABEL`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-LABEL`），从而保证后端/代码生成结果稳定。

### Lines 188-189
```mlir
  // CHECK-NOT: setprio
  // CHECK-NOT: barrier
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NOT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NOT`），从而保证后端/代码生成结果稳定。

### Lines 191-202
```mlir
  tt.func @reject_chained_dots_empty_mem_cluster_2(%memdesc1: !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 2x64x16>, %memdesc2: !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 2x64x16>, %alloc1: !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>, %alloc2: !ttg.memdesc<2x64x16xf16, #shared, #smem, mutable>, %arg0: tensor<64x16xf16, #blocked>, %arg1: tensor<64x16x!tt.ptr<f16>, #blocked>, %arg2: i32, %arg3: i32, %arg4: tensor<128x16xf32, #mma>, %arg5: tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>, %arg6: i32, %arg7: tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, %arg8: tensor<128x16xf32, #mma>, %arg9: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg10: tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>, %arg11: i32, %arg12: i32, %arg13: tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>) -> tensor<128x16xf32, #mma> {
    %5:8 = scf.for %arg14 = %arg3 to %arg2 step %arg3 iter_args(%arg15 = %arg4, %arg16 = %arg4, %arg17 = %arg7, %arg18 = %memdesc1, %arg19 = %memdesc1, %arg20 = %memdesc2, %arg21 = %arg0, %arg22 = %arg0) -> (tensor<128x16xf32, #mma>, tensor<128x16xf32, #mma>, tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 2x64x16>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 2x64x16>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 2x64x16>, tensor<64x16xf16, #blocked>, tensor<64x16xf16, #blocked>)  : i32 {
      %6 = tt.dot %arg10, %arg17, %arg15 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
      ttg.local_store %arg22, %arg20 : tensor<64x16xf16, #blocked> -> !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 2x64x16>
      %11 = ttg.local_load %arg20 : !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 2x64x16> -> tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
      %13 = tt.load %arg1 : tensor<64x16x!tt.ptr<f16>, #blocked>
      %10 = tt.dot %arg10, %arg17, %arg16 : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<128x16xf32, #mma>
      scf.yield %10, %6, %11, %arg19, %arg20, %arg20, %13, %13 : tensor<128x16xf32, #mma>, tensor<128x16xf32, #mma>, tensor<64x16xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 2x64x16>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 2x64x16>, !ttg.memdesc<64x16xf16, #shared, #smem, mutable, 2x64x16>, tensor<64x16xf16, #blocked>, tensor<64x16xf16, #blocked>
    }
    tt.return %5#0 : tensor<128x16xf32, #mma>
  }
}
```
**EN:** This block defines `reject_chained_dots_empty_mem_cluster_2` and exercises dot products / matmul, tensor loads, loop-carried state.
**CN:** 这一块定义了 `reject_chained_dots_empty_mem_cluster_2`，并覆盖 点积/矩阵乘、张量加载、循环携带状态。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's TritonGPU transforms coverage and focuses on dot-product lowering.  
  **CN:** 该文件属于 Triton 的 TritonGPU 变换 测试覆盖，关注点是 点积降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--tritonamdgpu-block-pingpong="num-stages=4"`.  
  **CN:** `RUN` 流水线会驱动 `--tritonamdgpu-block-pingpong="num-stages=4"` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `scf` (structured control flow), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`scf`（结构化控制流）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `scf`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`scf`、`tt`、`arith`。
