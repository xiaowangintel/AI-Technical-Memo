# amdgpu_membar.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/amdgpu_membar.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises memory barrier semantics in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的内存屏障语义相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --convert-scf-to-cf --allocate-shared-memory -test-tritonamdgpu-membar | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×33, CHECK-LABEL×10, CHECK-NEXT×4, CHECK-NOT×9; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×33, CHECK-LABEL×10, CHECK-NEXT×4, CHECK-NOT×9；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that memory barrier semantics produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 内存屏障语义 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --convert-scf-to-cf --allocate-shared-memory -test-tritonamdgpu-membar | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --convert-scf-to-cf --allocate-shared-memory -test-tritonamdgpu-membar | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --convert-scf-to-cf --allocate-shared-memory -test-tritonamdgpu-membar | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-4
```mlir
#AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#A_SHARED = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#AL`, `#A_SHARED`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.swizzled_shared`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#AL`, `#A_SHARED`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.swizzled_shared`。

### Lines 6-63
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
// Check that we only get a single barrier when using AsyncWait
// CHECK-LABEL: pipelined_async_copy_local_to_global
tt.func @pipelined_async_copy_local_to_global(%A: !tt.ptr<f16>) {
  %index_0 = arith.constant 0 : i32
  %index_1 = arith.constant 1 : i32
  %a_ptr = tt.splat %A : !tt.ptr<f16> -> tensor<16x16x!tt.ptr<f16>, #AL>
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %tile_a = ttg.memdesc_index %alloc[%index_0] : !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %tile_b = ttg.memdesc_index %alloc[%index_1] : !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // Load TileA
  %1 = ttg.async_copy_global_to_local %a_ptr, %tile_a: tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // Wait for TileA
  %2 = ttg.async_wait %1 {num = 4 : i32}
  // Read TileA
  %4 = ttg.local_load %tile_a token %2 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
  // Load into TileB
  %3 = ttg.async_copy_global_to_local %a_ptr, %tile_b : tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // There should be a single barrier after async_wait
  // CHECK-NOT: ttg.barrier local
  // CHECK: ttg.async_wait
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NOT: ttg.barrier local
  // CHECK: tt.return
  tt.return
}
// Same as above but different order of ops
// CHECK-LABEL: pipelined_async_copy_local_to_global_2
tt.func @pipelined_async_copy_local_to_global_2(%A: !tt.ptr<f16>) {
  %index_0 = arith.constant 0 : i32
  %index_1 = arith.constant 1 : i32
  %a_ptr = tt.splat %A : !tt.ptr<f16> -> tensor<16x16x!tt.ptr<f16>, #AL>
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %tile_a = ttg.memdesc_index %alloc[%index_0] : !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %tile_b = ttg.memdesc_index %alloc[%index_1] : !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // Load Tile
  %1 = ttg.async_copy_global_to_local %a_ptr, %tile_a: tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // Wait for TileA
  %2 = ttg.async_wait %1 {num = 4 : i32}
  // Load into TileB
  %3 = ttg.async_copy_global_to_local %a_ptr, %tile_b : tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // Read TileA
  %4 = ttg.local_load %tile_a token %2 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
  // There should be a single barrier after async_wait
  // CHECK-NOT: ttg.barrier local
  // CHECK: ttg.async_wait
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NOT: ttg.barrier local
  // CHECK: tt.return
  tt.return
}
// Check that multiple LocalLoads waiting on the same AsyncWait produce one barrier
// CHECK-LABEL: pipelined_async_copy_local_to_global_3
tt.func @pipelined_async_copy_local_to_global_3(%A: !tt.ptr<f16>, %B: !tt.ptr<f16>) {
  %index_0 = arith.constant 0 : i32
  %index_1 = arith.constant 1 : i32
  %a_ptr = tt.splat %A : !tt.ptr<f16> -> tensor<16x16x!tt.ptr<f16>, #AL>
  %b_ptr = tt.splat %B : !tt.ptr<f16> -> tensor<16x16x!tt.ptr<f16>, #AL>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.num-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `pipelined_async_copy_local_to_global`, `pipelined_async_copy_local_to_global_2`, `pipelined_async_copy_local_to_global_3`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.num-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `pipelined_async_copy_local_to_global`, `pipelined_async_copy_local_to_global_2`, `pipelined_async_copy_local_to_global_3`。

### Lines 65-67
```mlir
  %alloc_a = ttg.local_alloc : () -> !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %tile_a_1 = ttg.memdesc_index %alloc_a[%index_0] : !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %tile_a_2 = ttg.memdesc_index %alloc_a[%index_1] : !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block contributes intermediate IR built from `ttg.local_alloc`, `ttg.memdesc`, `ttg.memdesc_index`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_alloc`, `ttg.memdesc`, `ttg.memdesc_index` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 69-71
```mlir
  %alloc_b = ttg.local_alloc : () -> !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %tile_b_1 = ttg.memdesc_index %alloc_b[%index_0] : !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %tile_b_2 = ttg.memdesc_index %alloc_b[%index_1] : !ttg.memdesc<2x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block contributes intermediate IR built from `ttg.local_alloc`, `ttg.memdesc`, `ttg.memdesc_index`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_alloc`, `ttg.memdesc`, `ttg.memdesc_index` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 73-86
```mlir
  // Load TileA_1
  %1 = ttg.async_copy_global_to_local %a_ptr, %tile_a_1: tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // Load TileB_1
  %2 = ttg.async_copy_global_to_local %b_ptr, %tile_b_1: tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // Wait for TileA
  %3 = ttg.async_wait %1, %2 {num = 4 : i32}
  // Read TileA_1
  %4 = ttg.local_load %tile_a_1 token %3 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
  // Read TileB_1
  %5 = ttg.local_load %tile_b_1 token %3 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
  // Load into TileA_2
  %6 = ttg.async_copy_global_to_local %a_ptr, %tile_a_2 : tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // Load into TileB_2
  %7 = ttg.async_copy_global_to_local %b_ptr, %tile_b_2 : tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block contributes intermediate IR built from `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.memdesc`, `ttg.async_wait`, `ttg.local_load`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.memdesc`, `ttg.async_wait`, `ttg.local_load` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 88-95
```mlir
  // There should be a single barrier after async_wait
  // CHECK-NOT: ttg.barrier local
  // CHECK: ttg.async_wait
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NOT: ttg.barrier local
  // CHECK: tt.return
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises operations such as `ttg.barrier`, `ttg.async_wait`, `tt.return`. Embedded check comments (CHECK×2, CHECK-NEXT×1, CHECK-NOT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `ttg.barrier`、`ttg.async_wait`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-NEXT×1, CHECK-NOT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 97-102
```mlir
// Check that we do not get a barrier for LocalLoad if the token comes from a previous loop iteration
// CHECK-LABEL: async_wait_in_previous_loop_iteration
tt.func @async_wait_in_previous_loop_iteration(%a_ptr: tensor<16x16x!tt.ptr<f16>, #AL>, %loopIterCount: i32) {
  %c0_i32 = arith.constant 0 : i32
  %c1_i32 = arith.constant 1 : i32
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block defines `async_wait_in_previous_loop_iteration` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `async_wait_in_previous_loop_iteration`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 104-105
```mlir
  %1 = ttg.async_copy_global_to_local %a_ptr, %alloc: tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %2 = ttg.async_wait %1 {num = 4 : i32}
```
**EN:** This block contributes intermediate IR built from `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.memdesc`, `ttg.async_wait`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.memdesc`, `ttg.async_wait` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 107-110
```mlir
  // CHECK: cf.br
  %loop_result:1 = scf.for %arg14 = %c0_i32 to %loopIterCount step %c1_i32 iter_args(%arg10 = %2) -> (!ttg.async.token)  : i32 {
    %6 = ttg.local_load %alloc token %arg10 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
    %7 = ttg.async_copy_global_to_local %a_ptr, %alloc : tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies, loop-carried state. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝、循环携带状态。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 112-121
```mlir
    // CHECK-NOT: ttg.barrier local
    // CHECK: ttg.async_wait
    %8 = ttg.async_wait %7 {num = 4 : i32}
    // CHECK: ttg.barrier local
    // CHECK-NOT: ttg.barrier local
    scf.yield %8: !ttg.async.token
  }
  // CHECK: tt.return
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises operations such as `ttg.barrier`, `ttg.async_wait`, `scf.yield`, `ttg.async.token`, `tt.return`. Embedded check comments (CHECK×3, CHECK-NOT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `ttg.barrier`、`ttg.async_wait`、`scf.yield`、`ttg.async.token`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×3, CHECK-NOT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 123-128
```mlir
// Check we do get a barrier for LocalLoad if the initial loop token does not come from AsyncWait
// CHECK-LABEL: intial_loop_token_is_not_from_async_wait
tt.func @intial_loop_token_is_not_from_async_wait(%a_ptr: tensor<16x16x!tt.ptr<f16>, #AL>, %loopIterCount: i32) {
  %c0_i32 = arith.constant 0 : i32
  %c1_i32 = arith.constant 1 : i32
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block defines `intial_loop_token_is_not_from_async_wait` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `intial_loop_token_is_not_from_async_wait`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 130-142
```mlir
  %1 = ttg.async_copy_global_to_local %a_ptr, %alloc: tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %loop_result:1 = scf.for %arg14 = %c0_i32 to %loopIterCount step %c1_i32 iter_args(%arg10 = %1) -> (!ttg.async.token)  : i32 {
    %6 = ttg.local_load %alloc token %arg10 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
    // CHECK: ttg.local_load
    // CHECK: ttg.barrier local
    // CHECK: ttg.async_copy_global_to_local
    %7 = ttg.async_copy_global_to_local %a_ptr, %alloc : tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    %8 = ttg.async_wait %7 {num = 4 : i32}
    scf.yield %8: !ttg.async.token
  }
  // CHECK: tt.return
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies, loop-carried state. Embedded check comments (CHECK×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝、循环携带状态。 其中嵌入的检查注释（CHECK×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 144-149
```mlir
// Same as above but the loop carried token does not come from AsyncWait
// CHECK-LABEL: loop_carried_token_not_from_async_wait
tt.func @loop_carried_token_not_from_async_wait(%a_ptr: tensor<16x16x!tt.ptr<f16>, #AL>, %loopIterCount: i32) {
  %c0_i32 = arith.constant 0 : i32
  %c1_i32 = arith.constant 1 : i32
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block defines `loop_carried_token_not_from_async_wait` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `loop_carried_token_not_from_async_wait`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 151-163
```mlir
  %1 = ttg.async_copy_global_to_local %a_ptr, %alloc: tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %2 = ttg.async_wait %1 {num = 4 : i32}
  %loop_result:1 = scf.for %arg14 = %c0_i32 to %loopIterCount step %c1_i32 iter_args(%arg10 = %2) -> (!ttg.async.token)  : i32 {
    %6 = ttg.local_load %alloc token %arg10 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
    // CHECK: ttg.local_load
    // CHECK: ttg.barrier local
    // CHECK: ttg.async_copy_global_to_local
    %7 = ttg.async_copy_global_to_local %a_ptr, %alloc : tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    scf.yield %7: !ttg.async.token
  }
  // CHECK: tt.return
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies, loop-carried state. Embedded check comments (CHECK×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝、循环携带状态。 其中嵌入的检查注释（CHECK×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 166-171
```mlir
// Check that we do not get a barrier for an if where both branches yield an AsyncToken from AsyncWait
// CHECK-LABEL: async_wait_inside_if
tt.func @async_wait_inside_if(%cond: i1, %a_ptr: tensor<16x16x!tt.ptr<f16>, #AL>, %loopIterCount: i32) {
  %c0_i32 = arith.constant 0 : i32
  %c1_i32 = arith.constant 1 : i32
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block defines `async_wait_inside_if` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `async_wait_inside_if`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 173-174
```mlir
  %1 = ttg.async_copy_global_to_local %a_ptr, %alloc: tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %2 = ttg.async_wait %1 {num = 4 : i32}
```
**EN:** This block contributes intermediate IR built from `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.memdesc`, `ttg.async_wait`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.memdesc`, `ttg.async_wait` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 176-193
```mlir
  %loop_result:1 = scf.for %arg14 = %c0_i32 to %loopIterCount step %c1_i32 iter_args(%arg10 = %2) -> (!ttg.async.token)  : i32 {
    %6 = ttg.local_load %alloc token %arg10 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
    // CHECK: ttg.local_load
    // CHECK-NOT: ttg.barrier local
    // CHECK: ttg.async_copy_global_to_local
    %7 = ttg.async_copy_global_to_local %a_ptr, %alloc : tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    %103 = scf.if %cond -> (!ttg.async.token) {
      %8 = ttg.async_wait %7 {num = 4 : i32}
      scf.yield %8 : !ttg.async.token
    } else {
      %9 = ttg.async_wait %7 {num = 4 : i32}
      scf.yield %9 : !ttg.async.token
    }
    scf.yield %103: !ttg.async.token
  }
  // CHECK: tt.return
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies, loop-carried state, control-flow joins. Embedded check comments (CHECK×3, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝、循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×3, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 195-200
```mlir
// Check that we do get a barrier for an if where one branch does not yield an token from AsyncWait
// CHECK-LABEL: non_async_wait_token_from_then
tt.func @non_async_wait_token_from_then(%cond: i1, %a_ptr: tensor<16x16x!tt.ptr<f16>, #AL>, %loopIterCount: i32) {
  %c0_i32 = arith.constant 0 : i32
  %c1_i32 = arith.constant 1 : i32
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block defines `non_async_wait_token_from_then` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `non_async_wait_token_from_then`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 202-203
```mlir
  %1 = ttg.async_copy_global_to_local %a_ptr, %alloc: tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %2 = ttg.async_wait %1 {num = 4 : i32}
```
**EN:** This block contributes intermediate IR built from `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.memdesc`, `ttg.async_wait`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.memdesc`, `ttg.async_wait` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 205-222
```mlir
  %loop_result:1 = scf.for %arg14 = %c0_i32 to %loopIterCount step %c1_i32 iter_args(%arg10 = %2) -> (!ttg.async.token)  : i32 {
    %6 = ttg.local_load %alloc token %arg10 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
    // We should get a barrier because the then branch does not yield an token from AsyncWait
    // CHECK: ttg.local_load
    // CHECK: ttg.barrier local
    // CHECK: ttg.async_copy_global_to_local
    %7 = ttg.async_copy_global_to_local %a_ptr, %alloc : tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    %103 = scf.if %cond -> (!ttg.async.token) {
      scf.yield %7 : !ttg.async.token
    } else {
      %8 = ttg.async_wait %7 {num = 4 : i32}
      scf.yield %8 : !ttg.async.token
    }
    scf.yield %103: !ttg.async.token
  }
  // CHECK: tt.return
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies, loop-carried state, control-flow joins. Embedded check comments (CHECK×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝、循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 224-229
```mlir
// See above
// CHECK-LABEL: non_async_wait_token_from_else
tt.func @non_async_wait_token_from_else(%cond: i1, %a_ptr: tensor<16x16x!tt.ptr<f16>, #AL>, %loopIterCount: i32) {
  %c0_i32 = arith.constant 0 : i32
  %c1_i32 = arith.constant 1 : i32
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block defines `non_async_wait_token_from_else` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `non_async_wait_token_from_else`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 231-232
```mlir
  %1 = ttg.async_copy_global_to_local %a_ptr, %alloc: tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %2 = ttg.async_wait %1 {num = 4 : i32}
```
**EN:** This block contributes intermediate IR built from `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.memdesc`, `ttg.async_wait`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.async_copy_global_to_local`, `tt.ptr`, `ttg.memdesc`, `ttg.async_wait` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 234-252
```mlir
  %loop_result:1 = scf.for %arg14 = %c0_i32 to %loopIterCount step %c1_i32 iter_args(%arg10 = %2) -> (!ttg.async.token)  : i32 {
    %6 = ttg.local_load %alloc token %arg10 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
    // We should get a barrier because the else branch does not yield an token from AsyncWait
    // CHECK: ttg.local_load
    // CHECK: ttg.barrier local
    // CHECK: ttg.async_copy_global_to_local
    %7 = ttg.async_copy_global_to_local %a_ptr, %alloc : tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    %103 = scf.if %cond -> (!ttg.async.token) {
      %8 = ttg.async_wait %7 {num = 4 : i32}
      scf.yield %8 : !ttg.async.token
    } else {
      %9 = ttg.async_copy_global_to_local %a_ptr, %alloc: tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
      scf.yield %9 : !ttg.async.token
    }
    scf.yield %103: !ttg.async.token
  }
  // CHECK: tt.return
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies, loop-carried state, control-flow joins. Embedded check comments (CHECK×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝、循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 254-257
```mlir
// CHECK-LABEL: missing_barrier_reused_allocation
tt.func @missing_barrier_reused_allocation(%A: !tt.ptr<f16>, %B: !tt.ptr<f16>) {
  %c0_i32 = arith.constant 0 : i32
  %alloc1 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<2x128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block defines `missing_barrier_reused_allocation` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `missing_barrier_reused_allocation`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 259
```mlir
  %offset = arith.constant dense<0> : tensor<128x32xi32, #AL>
```
**EN:** This block contributes intermediate IR built from `arith.constant`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `arith.constant` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 261-279
```mlir
  %slice1_0 = ttg.memdesc_index %alloc1[%c0_i32] : !ttg.memdesc<2x128x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %async1 = amdg.buffer_load_to_local %A[%offset] into %slice1_0 : <f16>[tensor<128x32xi32, #AL>] -> <128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %token1 = ttg.async_commit_group tokens %async1
  %wait1 = amdg.async_wait %token1 {num_inst = 0 : i32}
  // CHECK: ttg.barrier local
  // CHECK: ttg.local_load
  %local_load = ttg.local_load %slice1_0 token %wait1 {ttg.amdg.syncedViaAsyncWait = true} : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<128x32xf16, #AL>
  ttg.local_dealloc %alloc1 : !ttg.memdesc<2x128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %alloc2 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<2x128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %slice2_0 = ttg.memdesc_index %alloc2[%c0_i32] : !ttg.memdesc<2x128x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // op2: Async load into alloc2 (overlapping with the dealloc'd alloc1 that is still being local_load'd from)
  // CHECK: ttg.barrier local
  // CHECK-NEXT: amdg.buffer_load_to_local
  %async2 = amdg.buffer_load_to_local %B[%offset] into %slice2_0 : <f16>[tensor<128x32xi32, #AL>] -> <128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %token2 = ttg.async_commit_group tokens %async2
  %wait2 = amdg.async_wait %token2 {num_inst = 0 : i32}
  // CHECK: ttg.barrier local
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation. Embedded check comments (CHECK×4, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×4, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 281
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on memory barrier semantics.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 内存屏障语义。
- **EN:** The `RUN` pipeline drives tools/passes such as `--convert-scf-to-cf`, `--allocate-shared-memory`, `-test-tritonamdgpu-membar`.  
  **CN:** `RUN` 流水线会驱动 `--convert-scf-to-cf`, `--allocate-shared-memory`, `-test-tritonamdgpu-membar` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `cf` (control-flow ops), `scf` (structured control flow).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`cf`（控制流操作）、`scf`（结构化控制流）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `cf`, `scf`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`cf`、`scf`。
