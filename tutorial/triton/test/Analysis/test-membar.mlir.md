# test-membar.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Analysis/test-membar.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises memory barrier semantics in Triton's analysis coverage. **CN:** 该MLIR 测试用于覆盖 Triton 分析中的内存屏障语义相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory -test-print-membar | FileCheck %s`<br>`triton-opt %s -split-input-file --allocate-shared-memory -test-tritonamdgpu-membar | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×104, CHECK-LABEL×54, CHECK-NEXT×161, CHECK-NOT×13; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×104, CHECK-LABEL×54, CHECK-NEXT×161, CHECK-NOT×13；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that memory barrier semantics produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 内存屏障语义 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s -split-input-file --allocate-shared-memory -test-print-membar | FileCheck %s
// RUN: triton-opt %s -split-input-file --allocate-shared-memory -test-tritonamdgpu-membar | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-shared-memory -test-print-membar | FileCheck %s; triton-opt %s -split-input-file --allocate-shared-memory -test-tritonamdgpu-membar | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-shared-memory -test-print-membar | FileCheck %s; triton-opt %s -split-input-file --allocate-shared-memory -test-tritonamdgpu-membar | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-11
```mlir
#AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#sliceAd0 = #ttg.slice<{dim = 0, parent = #AL}>
#BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#A_SHARED = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#A_SHARED_T = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [0, 1]}>
#C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
#A_DOT = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 2}>
#B_DOT = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 2}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#AL`, `#sliceAd0`, `#BL`, `#A_SHARED`, `#A_SHARED_T`, `#C`, `#A_DOT`, `#B_DOT`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.slice`, `#ttg.swizzled_shared`, `#ttg.nvidia_mma`, `#ttg.dot_op`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#AL`, `#sliceAd0`, `#BL`, `#A_SHARED`, `#A_SHARED_T`, `#C`, `#A_DOT`, `#B_DOT`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.slice`, `#ttg.swizzled_shared`, `#ttg.nvidia_mma`, `#ttg.dot_op`。

### Line 13
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.num-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.num-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 15-19
```mlir
// CHECK-LABEL: matmul_loop
// There shouldn't be any membar with the dot op encoding.
tt.func @matmul_loop(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>) {
  %a_ptr_init = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
  %b_ptr_init = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
```
**EN:** This block defines `matmul_loop` and exercises operations such as `tt.func`, `tt.ptr`, `tt.splat`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `matmul_loop`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.splat` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 21-25
```mlir
  %a_mask = arith.constant dense<true> : tensor<128x32xi1, #AL>
  %a_other = arith.constant dense<0.00e+00> : tensor<128x32xf16, #AL>
  %b_mask = arith.constant dense<true> : tensor<32x128xi1, #BL>
  %b_other = arith.constant dense<0.00e+00> : tensor<32x128xf16, #BL>
  %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
```
**EN:** This block contributes intermediate IR built from `arith.constant`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `arith.constant` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 27-28
```mlir
  %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
  %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
```
**EN:** This block contributes intermediate IR built from `arith.constant`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `arith.constant` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 30-35
```mlir
  scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
    %a_ = tt.load %a_ptr, %a_mask, %a_other : tensor<128x32x!tt.ptr<f16>, #AL>
    %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A_DOT>
    %b_ = tt.load %b_ptr, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
    %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B_DOT>
    %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A_DOT> * tensor<32x128xf16, #B_DOT> -> tensor<128x128xf32, #C>
```
**EN:** This block contributes intermediate IR built from `scf.for`, `tt.ptr`, `tt.load`, `ttg.convert_layout`, `tt.dot`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `scf.for`, `tt.ptr`, `tt.load`, `ttg.convert_layout`, `tt.dot` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 37-42
```mlir
    %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
    %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
    scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
  }
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `tt.addptr`, `tt.ptr`, `scf.yield`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.addptr`, `tt.ptr`, `scf.yield`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 44-55
```mlir
// CHECK-LABEL: raw_single_block
tt.func @raw_single_block(%A : !tt.ptr<f16>) {
  %cst1 = arith.constant dense<true> : tensor<128x32xi1, #AL>
  %cst2 = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #AL>
  %0 = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
  %1 = tt.load %0, %cst1, %cst2 : tensor<128x32x!tt.ptr<f16>, #AL>
  %2 = ttg.local_alloc %1 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %3 = ttg.local_load %2 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  tt.return
}
```
**EN:** This block defines `raw_single_block` and exercises shared-memory allocation, tensor loads. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `raw_single_block`，并覆盖 共享内存分配、张量加载。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 57-72
```mlir
// CHECK-LABEL: war_single_block
tt.func @war_single_block(%A : !tt.ptr<f16>) {
  %cst1 = arith.constant dense<true> : tensor<128x32xi1, #AL>
  %cst2 = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #AL>
  %0 = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
  %1 = tt.load %0, %cst1, %cst2 : tensor<128x32x!tt.ptr<f16>, #AL>
  %2 = ttg.local_alloc %1 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  // CHECK: ttg.local_alloc
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %3 = ttg.local_load %2 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: %4 = ttg.local_alloc
  %4 = ttg.local_alloc %1 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  tt.return
}
```
**EN:** This block defines `war_single_block` and exercises shared-memory allocation, tensor loads. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `war_single_block`，并覆盖 共享内存分配、张量加载。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 74-89
```mlir
// CHECK-LABEL: war_single_block_local_store
tt.func @war_single_block_local_store(%A : !tt.ptr<f16>) {
  %cst1 = arith.constant dense<true> : tensor<128x32xi1, #AL>
  %cst2 = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #AL>
  %0 = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
  %1 = tt.load %0, %cst1, %cst2 : tensor<128x32x!tt.ptr<f16>, #AL>
  %2 = ttg.local_alloc %1 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // CHECK: ttg.local_alloc
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %3 = ttg.local_load %2 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<128x32xf16, #AL>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_store
  ttg.local_store %1, %2 : tensor<128x32xf16, #AL> -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `war_single_block_local_store` and exercises shared-memory allocation, tensor loads. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `war_single_block_local_store`，并覆盖 共享内存分配、张量加载。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 91-105
```mlir
// CHECK-LABEL: scratch
tt.func @scratch(%arg: tensor<16x16xf16, #AL>) {
  %cst0 = ttg.local_alloc %arg : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  // CHECK: ttg.barrier local
  // CHECK: tt.reduce
  %1 = ttg.local_load %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  %2 = "tt.reduce" (%1) ({
  ^bb0(%arg1: f16, %arg2: f16):
    %add = arith.addf %arg1, %arg2 : f16
    tt.reduce.return %add : f16
  }) {axis = 0 : i32} : (tensor<16x16xf16, #AL>) -> tensor<16xf16, #sliceAd0>
  tt.return
}
```
**EN:** This block defines `scratch` and exercises shared-memory allocation. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `scratch`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 107-116
```mlir
// CHECK-LABEL: async_wait
tt.func @async_wait(%arg: tensor<32x16xf16, #AL>) {
  %cst0 = ttg.local_alloc %arg : (tensor<32x16xf16, #AL>) -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory>
  // CHECK: ttg.async_wait
  ttg.async_wait {num = 4 : i32}
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %1 = ttg.local_load %cst0 : !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<32x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `async_wait` and exercises shared-memory allocation. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `async_wait`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 118-130
```mlir
// CHECK-LABEL: subview
tt.func @subview() {
  %cst0 = arith.constant dense<0.000000e+00> : tensor<32x16xf16, #AL>
  %a = ttg.local_alloc %cst0 : (tensor<32x16xf16, #AL>) -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory>
  %0 = ttg.memdesc_subslice %a [0, 0] : !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %1 = ttg.local_load %0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_alloc
  %2 = ttg.local_alloc %1 : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  tt.return
}
```
**EN:** This block defines `subview` and exercises shared-memory allocation. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `subview`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 132-137
```mlir
// CHECK-LABEL: trans
tt.func @trans(%a: !ttg.memdesc<16x32xf16, #A_SHARED, #ttg.shared_memory>) {
  // CHECK-NOT: ttg.barrier local
  %b = ttg.memdesc_trans %a {order=array<i32: 1,0>} : !ttg.memdesc<16x32xf16, #A_SHARED, #ttg.shared_memory> -> !ttg.memdesc<32x16xf16, #A_SHARED_T, #ttg.shared_memory>
  tt.return
}
```
**EN:** This block defines `trans` and exercises operations such as `tt.func`, `ttg.memdesc`, `ttg.barrier`, `ttg.memdesc_trans`, `tt.return`. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `trans`，并覆盖 如 `tt.func`、`ttg.memdesc`、`ttg.barrier`、`ttg.memdesc_trans`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 139-171
```mlir
// CHECK-LABEL: async_copy_global_to_local
tt.func @async_copy_global_to_local(%A : !tt.ptr<f16>, %i1 : i1) {
  %index = arith.constant 0 : i32
  %a_ptr = tt.splat %A : !tt.ptr<f16> -> tensor<16x16x!tt.ptr<f16>, #AL>
  %mask = tt.splat %i1 : i1 -> tensor<16x16xi1, #AL>
  %other = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %subview = ttg.memdesc_index %alloc[%index] : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %1 = ttg.async_copy_global_to_local %a_ptr, %subview : tensor<16x16x!tt.ptr<f16>, #AL> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %4 = ttg.local_load %subview : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
  tt.return
}
// If branch inserted a barrier for %cst0, but else didn't, then the barrier should be inserted in the parent region
// CHECK-LABEL: multi_blocks
tt.func @multi_blocks(%i1 : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %cst0 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  scf.if %i1 {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %0 = ttg.local_load %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
    scf.yield
  } else {
    %cst1 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
    scf.yield
  }
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %2 = ttg.local_load %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `async_copy_global_to_local`, `multi_blocks` and exercises async global-to-shared copies, shared-memory allocation, control-flow joins. Embedded check comments (CHECK×3, CHECK-LABEL×2, CHECK-NEXT×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `async_copy_global_to_local`, `multi_blocks`，并覆盖 全局到共享内存的异步拷贝、共享内存分配、控制流汇合。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×2, CHECK-NEXT×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 173-193
```mlir
// Both branches inserted a barrier for %cst0 and %cst1, then the barrier doesn't need to be inserted in the parent region
// CHECK-LABEL: multi_blocks_join_barrier
tt.func @multi_blocks_join_barrier(%i1 : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %cst0 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  scf.if %i1 {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %0 = ttg.local_load %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
    scf.yield
  } else {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %1 = ttg.local_load %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
    scf.yield
  }
  // CHECK-NOT: ttg.barrier local
  // CHECK: tt.return
  %a_ = ttg.local_load %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `multi_blocks_join_barrier` and exercises shared-memory allocation, control-flow joins. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×2, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `multi_blocks_join_barrier`，并覆盖 共享内存分配、控制流汇合。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×2, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 195-219
```mlir
// Read yielded tensor requires a barrier
// CHECK-LABEL: multi_blocks_yield
tt.func @multi_blocks_yield(%i1 : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %cst0 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  %a = scf.if %i1 -> (!ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>) {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %0 = ttg.local_load %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
    %1 = ttg.local_alloc %0 : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
    scf.yield %1 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  } else {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %2 = ttg.local_load %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
    %3 = ttg.local_alloc %2 : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
    scf.yield %3 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  }
  %a_ = ttg.local_load %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  // CHECK: ttg.local_load
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %4 = ttg.local_load %a : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `multi_blocks_yield` and exercises shared-memory allocation, control-flow joins. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `multi_blocks_yield`，并覆盖 共享内存分配、控制流汇合。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 221-247
```mlir
// Even though the entry block doesn't have a barrier, the successors should have barriers
// CHECK-LABEL: multi_blocks_entry_no_shared
tt.func @multi_blocks_entry_no_shared(%i1 : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %cst0 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  %a = scf.if %i1 -> (!ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>) {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_alloc
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: ttg.local_alloc
    %cst1 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
    %0 = ttg.local_load %cst1 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
    %1 = ttg.local_alloc %0 : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
    scf.yield %1 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  } else {
    // CHECK-NOT: ttg.barrier local
    // CHECK: ttg.local_alloc
    %cst1 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
    scf.yield %cst1 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  }
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %2 = ttg.local_load %a : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `multi_blocks_entry_no_shared` and exercises shared-memory allocation, control-flow joins. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×6, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `multi_blocks_entry_no_shared`，并覆盖 共享内存分配、控制流汇合。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×6, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 249-264
```mlir
// Conservatively add a barrier as if the branch (%i1) is never taken
// CHECK-LABEL: multi_blocks_noelse
tt.func @multi_blocks_noelse(%i1 : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %cst0 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  scf.if %i1 {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %0 = ttg.local_load %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
    scf.yield
  }
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %1 = ttg.local_load %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `multi_blocks_noelse` and exercises shared-memory allocation, control-flow joins. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `multi_blocks_noelse`，并覆盖 共享内存分配、控制流汇合。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 266-289
```mlir
// Conservatively add a barrier as if the branch (%i2) is never taken
// CHECK-LABEL: multi_blocks_nested_scf
tt.func @multi_blocks_nested_scf(%i1 : i1, %i2 : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #AL>
  %cst0 = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  scf.if %i1 {
    scf.if %i2 {
      // CHECK: ttg.barrier local
      // CHECK-NEXT: ttg.local_load
      %0 = ttg.local_load %cst0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
      scf.yield
    }
    scf.yield
  } else {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %1 = ttg.local_load %cst0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    scf.yield
  }
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %2 = ttg.local_load %cst0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  tt.return
}
```
**EN:** This block defines `multi_blocks_nested_scf` and exercises shared-memory allocation, control-flow joins. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `multi_blocks_nested_scf`，并覆盖 共享内存分配、控制流汇合。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 291-305
```mlir
// CHECK-LABEL: for
tt.func @for(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>) {
  %cst = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #AL>
  %a_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %b_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %c_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init, %c_shared = %c_shared_init) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>) {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %a0 = ttg.local_load %a_shared : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    %b0 = ttg.local_load %b_shared : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    scf.yield %b_shared, %a_shared, %a_shared : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  }
  tt.return
}
```
**EN:** This block defines `for` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `for`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 307-331
```mlir
// Although a_shared and b_shared are synced before entering the loop,
// they are reassociated with aliases (c_shared) and thus require a barrier.
// CHECK-LABEL: for_alias
tt.func @for_alias(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>) {
  %cst = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #AL>
  %a_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %b_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %a0 = ttg.local_load %a_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  %b0 = ttg.local_load %b_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  %0 = ttg.local_alloc %a0 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %c_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init, %c_shared = %c_shared_init) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>) {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %a1 = ttg.local_load %a_shared : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    %b1 = ttg.local_load %b_shared : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    scf.yield %c_shared, %a_shared, %b_shared : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  }
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %r = ttg.local_load %0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  tt.return
}
```
**EN:** This block defines `for_alias` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `for_alias`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 333-363
```mlir
// Although cst2 is not an argument of scf.yield, its memory is reused by cst1.
// So we need a barrier both before and after cst1
// CHECK-LABEL: for_reuse
tt.func @for_reuse(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>) {
  %cst = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #AL>
  %a_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %b_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %a0 = ttg.local_load %a_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  %b0 = ttg.local_load %b_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  %0 = ttg.local_alloc %a0 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %c_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init, %c_shared = %c_shared_init) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>) {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_alloc
    %a1 = ttg.local_load %a_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    %b1 = ttg.local_load %b_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    %1 = ttg.local_alloc %a1 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_alloc
    %a2 = ttg.local_load %a_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    %b2 = ttg.local_load %b_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    %2 = ttg.local_alloc %a1 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
    scf.yield %c_shared, %a_shared, %b_shared : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  }
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %r = ttg.local_load %0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  tt.return
}
```
**EN:** This block defines `for_reuse` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×4, CHECK-LABEL×1, CHECK-NEXT×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `for_reuse`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×4, CHECK-LABEL×1, CHECK-NEXT×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 365-396
```mlir
// CHECK-LABEL: for_reuse_nested
tt.func @for_reuse_nested(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>) {
  %cst = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #AL>
  %a_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %b_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %a0 = ttg.local_load %a_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  %b0 = ttg.local_load %b_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  %0 = ttg.local_alloc %a0 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %c_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init, %c_shared = %c_shared_init) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>) {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_alloc
    %a1 = ttg.local_load %a_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    %b1 = ttg.local_load %b_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    %1 = ttg.local_alloc %a1 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
    %a_shared_next, %b_shared_next, %c_shared_next = scf.for %ivv = %lb to %ub step %step iter_args(%a_shared_nested = %a_shared_init, %b_shared_nested = %b_shared_init, %c_shared_nested = %c_shared_init) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>) {
      // CHECK: ttg.barrier local
      // CHECK-NEXT:  ttg.local_alloc
      %a2 = ttg.local_load %a_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
      %b2 = ttg.local_load %b_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
      %2 = ttg.local_alloc %a2 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
      scf.yield %c_shared_nested, %a_shared_nested, %b_shared_nested : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
    }
    scf.yield %c_shared, %a_shared, %b_shared : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  }
  // CHECK: ttg.barrier local
  // CHECK-NEXT:  ttg.local_load
  %r = ttg.local_load %0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  tt.return
}
```
**EN:** This block defines `for_reuse_nested` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×4, CHECK-LABEL×1, CHECK-NEXT×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `for_reuse_nested`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×4, CHECK-LABEL×1, CHECK-NEXT×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 398-423
```mlir
// repeatedly write to the same shared memory addresses
// CHECK-LABEL: for_for_if
tt.func @for_for_if(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>, %i1 : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #AL>
  %a_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %b_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %c_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init, %c_shared = %c_shared_init) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>) {
    %c_shared_next = scf.for %jv = %lb to %ub step %step iter_args(%c_shared_next = %c_shared) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>) {
      %c_shared_next_next = scf.if %i1 -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> {
        // CHECK: ttg.barrier local
        // CHECK-NEXT: ttg.local_alloc
        %cst0 = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
        scf.yield %cst0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
      } else {
        // CHECK: ttg.barrier local
        // CHECK-NEXT: ttg.local_alloc
        %cst0 = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
        scf.yield %cst0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
      }
      scf.yield %c_shared_next_next : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
    }
    scf.yield %a_shared, %b_shared, %c_shared_next : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  }
  tt.return
}
```
**EN:** This block defines `for_for_if` and exercises shared-memory allocation, loop-carried state, control-flow joins. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `for_for_if`，并覆盖 共享内存分配、循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 425-433
```mlir
// c_block_next can either be converted from c_shared_init or c_shared_next_next
// CHECK-LABEL: for_if_for
tt.func @for_if_for(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>, %i1 : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #AL>
  %a_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %b_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %c_shared_init = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  // CHECK: ttg.barrier local
  %c_blocked = ttg.local_load %c_shared_init : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
```
**EN:** This block defines `for_if_for` and exercises shared-memory allocation. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `for_if_for`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 435-455
```mlir
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init, %c_shared = %c_shared_init) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>) {
    %c_shared_next_next = scf.if %i1 -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> {
      // CHECK: ttg.barrier local
      // CHECK-NEXT: ttg.local_alloc
      %cst0 = ttg.local_alloc %cst : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
      scf.yield %cst0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
    } else {
      %c_shared_ = scf.for %jv = %lb to %ub step %step iter_args(%c_shared_next = %c_shared) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>) {
        // CHECK: ttg.barrier local
        // CHECK-NEXT: ttg.local_load
        %c_blocked_next = ttg.local_load %c_shared_next : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
        scf.yield %c_shared : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
      }
      scf.yield %c_shared_ : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
    }
    // CHECK-NOT: ttg.barrier local
    %b_blocked_next = ttg.local_load %b_shared: !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
    scf.yield %a_shared, %b_shared, %c_shared_next_next : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  }
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation, loop-carried state, control-flow joins. Embedded check comments (CHECK×2, CHECK-NEXT×2, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配、循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×2, CHECK-NEXT×2, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 457-472
```mlir
// CHECK-LABEL: cf_if
tt.func @cf_if(%i1 : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %a = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  cf.cond_br %i1, ^bb1, ^bb2
^bb1:  // pred: ^bb0
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %0 = ttg.local_load %a : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  cf.br ^bb2
^bb2:  // 2 preds: ^bb0, ^bb1
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %1 = ttg.local_load %a : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `cf_if` and exercises shared-memory allocation. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `cf_if`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 474-500
```mlir
// CHECK-LABEL: cf_if_else
tt.func @cf_if_else(%i1 : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %a = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  cf.cond_br %i1, ^bb1, ^bb2
^bb1:  // pred: ^bb0
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %0 = ttg.local_load %a : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  %1 = ttg.local_alloc %0 : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  cf.br ^bb3(%1 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>)
^bb2:  // pred: ^bb0
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %2 = ttg.local_load %a : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  %3 = ttg.local_alloc %2 : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  cf.br ^bb3(%3 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>)
^bb3(%arg: !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>):  // 2 preds: ^bb1, ^bb2
  cf.br ^bb4
^bb4:  // pred: ^bb3
  // CHECK: ttg.local_load
  %4 = ttg.local_load %a : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %5 = ttg.local_load %arg : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `cf_if_else` and exercises shared-memory allocation. Embedded check comments (CHECK×4, CHECK-LABEL×1, CHECK-NEXT×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `cf_if_else`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×4, CHECK-LABEL×1, CHECK-NEXT×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 502-520
```mlir
// CHECK-LABEL: cf_if_else_return
tt.func @cf_if_else_return(%i1 : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %a = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  %b = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  cf.cond_br %i1, ^bb1, ^bb2
^bb1:  // pred: ^bb0
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %0 = ttg.local_load %a : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  %1 = ttg.local_load %b : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  tt.return
^bb2:  // pred: ^bb0
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %2 = ttg.local_load %a : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  %3 = ttg.local_load %b : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory> -> tensor<16x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `cf_if_else_return` and exercises shared-memory allocation. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `cf_if_else_return`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 522-531
```mlir
// CHECK-LABEL: atomic_scalar
tt.func @atomic_scalar(%arg3: !tt.ptr<i32>) -> i32 {
  // CHECK-NOT: ttg.barrier local
  %c0_i32 = arith.constant 0 : i32
  %1 = arith.constant dense<1.0> : tensor<128x32xf16, #AL>
  %2 = ttg.local_alloc %1 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %4 = tt.atomic_cas acq_rel, gpu, %arg3, %c0_i32, %c0_i32 : (!tt.ptr<i32>, i32, i32) -> i32
  %3 = ttg.local_load %2 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  tt.return %4 : i32
}
```
**EN:** This block defines `atomic_scalar` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `atomic_scalar`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 533-543
```mlir
// CHECK-LABEL: atomic_scalar_no_use
tt.func @atomic_scalar_no_use(%arg3: !tt.ptr<i32>) {
  %c0_i32 = arith.constant 0 : i32
  %1 = arith.constant dense<1.0> : tensor<128x32xf16, #AL>
  %2 = ttg.local_alloc %1 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %4 = tt.atomic_cas acq_rel, gpu, %arg3, %c0_i32, %c0_i32 : (!tt.ptr<i32>, i32, i32) -> i32
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %3 = ttg.local_load %2 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  tt.return
}
```
**EN:** This block defines `atomic_scalar_no_use` and exercises shared-memory allocation. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `atomic_scalar_no_use`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 545
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 547
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.num-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.num-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 549-555
```mlir
// CHECK-LABEL: convert_layout1
tt.func @convert_layout1(%A : !tt.ptr<f16>) {
  // CHECK-NOT: ttg.barrier local
  %0 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %1 = ttg.local_load %0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `convert_layout1` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `convert_layout1`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 557-568
```mlir
// CHECK-LABEL: convert_layout2
tt.func @convert_layout2(%A : !tt.ptr<f16>) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %0 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %1 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // CHECK: ttg.local_load
  // CHECK-NEXT: ttg.barrier local
  // CHECK: ttg.local_load
  %3 = ttg.local_load %0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
  %4 = ttg.local_load %1 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `convert_layout2` and exercises shared-memory allocation. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `convert_layout2`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 570-586
```mlir
// CHECK-LABEL: convert_layout3
tt.func @convert_layout3(%cond : i1) {
  scf.if %cond {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<16x64xf16, #A_SHARED, #ttg.shared_memory, mutable>
    // CHECK: ttg.local_load
    // CHECK-NOT: ttg.barrier local
    %1 = ttg.local_load %0 : !ttg.memdesc<16x64xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x64xf16, #AL>
  } else {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    // CHECK: ttg.local_load
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: ttg.local_alloc
    %1 = ttg.local_load %0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
    %2 = ttg.local_alloc %1 : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  }
  tt.return
}
```
**EN:** This block defines `convert_layout3` and exercises shared-memory allocation, control-flow joins. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `convert_layout3`，并覆盖 共享内存分配、控制流汇合。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 588-597
```mlir
// CHEKC-LABEL: convert_layout4
tt.func @convert_layout4(%A : !tt.ptr<f16>, %cond : i1) {
  // CHECK-NOT: ttg.barrier local
  scf.if %cond {
    tt.call @convert_layout3(%cond) : (i1) -> ()
  } else {
    tt.call @convert_layout2(%A) : (!tt.ptr<f16>) -> ()
  }
  tt.return
}
```
**EN:** This block defines `convert_layout4`, `convert_layout3`, `convert_layout2` and exercises control-flow joins. Embedded check comments (CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `convert_layout4`, `convert_layout3`, `convert_layout2`，并覆盖 控制流汇合。 其中嵌入的检查注释（CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 599-610
```mlir
// CHECK-LABEL: convert_layout5
tt.func @convert_layout5(%A : !tt.ptr<f16>) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %0 = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %1 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // CHECK: ttg.local_load
  // CHECK-NEXT: ttg.barrier local
  // CHECK: ttg.local_load
  %3 = ttg.local_load %0 : !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<32x16xf16, #AL>
  %4 = ttg.local_load %1 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x16xf16, #AL>
  tt.return
}
```
**EN:** This block defines `convert_layout5` and exercises shared-memory allocation. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `convert_layout5`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 612-620
```mlir
// CHECK-LABEL: single_call_sync
tt.func @single_call_sync(%A : !tt.ptr<f16>) {
  %0 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
  // CHECK: tt.call
  // CHECK-NEXT: ttg.barrier local
  tt.call @convert_layout1(%A) : (!tt.ptr<f16>) -> ()
  %1 = ttg.convert_layout %0 : tensor<16x32xf16, #AL> -> tensor<16x32xf16, #BL>
  tt.return
}
```
**EN:** This block defines `single_call_sync`, `convert_layout1` and exercises layout conversion. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `single_call_sync`, `convert_layout1`，并覆盖 布局转换。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 622-630
```mlir
// CHECK-LABEL: single_call_no_sync
// %1 can reuse %0 in convert_layout2, which has been synced
tt.func @single_call_no_sync(%A : !tt.ptr<f16>) {
  // CHECK-NOT: ttg.barrier local
  %0 = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  tt.call @convert_layout5(%A) : (!tt.ptr<f16>) -> ()
  %1 = ttg.convert_layout %0 : tensor<16x16xf16, #AL> -> tensor<16x16xf16, #BL>
  tt.return
}
```
**EN:** This block defines `single_call_no_sync`, `convert_layout5` and exercises layout conversion. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `single_call_no_sync`, `convert_layout5`，并覆盖 布局转换。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 632-640
```mlir
// CHECK-LABEL: multiple_calls
tt.func @multiple_calls(%A : !tt.ptr<f16>) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %cst0 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  tt.call @convert_layout1(%A) : (!tt.ptr<f16>) -> ()
  %cst1 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
  tt.call @convert_layout2(%A) : (!tt.ptr<f16>) -> ()
  tt.return
}
```
**EN:** This block defines `multiple_calls`, `convert_layout1`, `convert_layout2` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `multiple_calls`, `convert_layout1`, `convert_layout2`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 642-660
```mlir
// CHECK-LABEL: if_else_calls
tt.func @if_else_calls(%A : !tt.ptr<f16>, %cond : i1) {
  scf.if %cond {
    %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
    %cst_ = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
    %cst0 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
    // CHECK: ttg.barrier local
    // CHECK-NEXT: tt.call
    // CHECK-NEXT: ttg.barrier local
    tt.call @convert_layout1(%A) : (!tt.ptr<f16>) -> ()
    %cst1 = ttg.local_alloc %cst_ : (tensor<16x32xf16, #AL>) -> !ttg.memdesc<16x32xf16, #A_SHARED, #ttg.shared_memory>
  } else {
    %cst0 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
    // CHECK: tt.call
    // CHECK-NOT: ttg.barrier local
    tt.call @convert_layout2(%A) : (!tt.ptr<f16>) -> ()
  }
  tt.return
}
```
**EN:** This block defines `if_else_calls`, `convert_layout1`, `convert_layout2` and exercises shared-memory allocation, control-flow joins. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `if_else_calls`, `convert_layout1`, `convert_layout2`，并覆盖 共享内存分配、控制流汇合。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 662-676
```mlir
// CHECK-LABEL: for_calls
tt.func @for_calls(%A : !tt.ptr<f16>, %cond : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %cst0 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  %cst1 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
  %lb = arith.constant 0 : index
  %ub = arith.constant 10 : index
  %step = arith.constant 1 : index
  scf.for %iv = %lb to %ub step %step {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: tt.call
    tt.call @convert_layout1(%A) : (!tt.ptr<f16>) -> ()
  }
  tt.return
}
```
**EN:** This block defines `for_calls`, `convert_layout1` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `for_calls`, `convert_layout1`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 678-685
```mlir
// CHECK-LABEL: call_graph_1
tt.func @call_graph_1(%A : !tt.ptr<f16>, %cond : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  %cst0 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>  // CHECK: ttg.barrier local
  // CHECK-NEXT: tt.call
  tt.call @convert_layout3(%cond) : (i1) -> ()
  tt.return
}
```
**EN:** This block defines `call_graph_1`, `convert_layout3` and exercises shared-memory allocation. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `call_graph_1`, `convert_layout3`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 687-695
```mlir
// CHECK-LABEL: call_graph_2
tt.func @call_graph_2(%A : !tt.ptr<f16>, %cond : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  tt.call @convert_layout4(%A, %cond) : (!tt.ptr<f16>, i1) -> ()
  // CHECK: tt.call
  // CHECK-NEXT: ttg.barrier local
  %cst0 = ttg.local_alloc %cst : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  tt.return
}
```
**EN:** This block defines `call_graph_2`, `convert_layout4` and exercises shared-memory allocation. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `call_graph_2`, `convert_layout4`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 697
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 699
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 701-718
```mlir
#block0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
#block1 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: @barrier_between_warp_sync_convert_and_read
  tt.func @barrier_between_warp_sync_convert_and_read(%src: tensor<32x!tt.ptr<f32>, #block0>) {
    %alloc = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>
    %c = arith.constant dense<0.0> : tensor<16x16xf16>
    // CHECK: ttg.local_store
    ttg.local_store %c, %alloc : tensor<16x16xf16> -> !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>
    // CHECK-NEXT: ttg.convert_layout
    %cvt = ttg.convert_layout %src : tensor<32x!tt.ptr<f32>, #block0> -> tensor<32x!tt.ptr<f32>, #block1>
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %ld = ttg.local_load %alloc : !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable> -> tensor<16x16xf16>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `barrier_between_warp_sync_convert_and_read`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `barrier_between_warp_sync_convert_and_read`。

### Line 720
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 722-749
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 4], instrShape = [16, 8]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 8, order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 18944 : i32} {
  tt.func public @kernel(%arg3: !tt.ptr<i32>, %arg4: !tt.ptr<f16>, %arg12: tensor<32x128xf16, #blocked>, %arg13: tensor<32x128xf32, #blocked>, %arg14: tensor<32x32xf16, #blocked1>) {
    %c0_i32 = arith.constant 0 : i32
    %cst = arith.constant dense<0.000000e+00> : tensor<32x128xf32, #blocked>
    %37 = ttg.local_alloc %arg14 {allocation.offset = 0 : i32} : (tensor<32x32xf16, #blocked1>) -> !ttg.memdesc<32x32xf16, #shared, #ttg.shared_memory>
    %58 = ttg.local_alloc %arg12 : (tensor<32x128xf16, #blocked>) -> !ttg.memdesc<32x128xf16, #shared1, #ttg.shared_memory>
    cf.br ^bb1
  ^bb1:  // 2 preds: ^bb0, ^bb1
    %59 = tt.atomic_cas acq_rel, gpu, %arg3, %c0_i32, %c0_i32 : (!tt.ptr<i32>, i32, i32) -> i32
    %60 = arith.cmpi eq, %59, %c0_i32 : i32
    cf.cond_br %60, ^bb1, ^bb2
  ^bb2:  // pred: ^bb1
    %72 = ttg.convert_layout %arg13 : tensor<32x128xf32, #blocked> -> tensor<32x128xf32, #mma>
    %73 = ttg.local_load %37 : !ttg.memdesc<32x32xf16, #shared, #ttg.shared_memory> -> tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
    %74 = ttg.local_load %58 : !ttg.memdesc<32x128xf16, #shared1, #ttg.shared_memory> -> tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
    %75 = tt.dot %73, %74, %72, inputPrecision = tf32 : tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<32x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<32x128xf32, #mma>
    %76 = ttg.convert_layout %75 {allocation.offset = 0 : i32} : tensor<32x128xf32, #mma> -> tensor<32x128xf32, #blocked>
    %77 = arith.truncf %76 : tensor<32x128xf32, #blocked> to tensor<32x128xf16, #blocked>
    %78 = tt.splat %arg4 : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #blocked>
    tt.store %78, %77 : tensor<32x128x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `kernel`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `kernel`。

### Line 751
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 753-754
```mlir
#layout = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [0]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#layout`, `#smem`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#layout`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 756-764
```mlir
// CHECK-LABEL: @warp_specialize_isolated_regions
tt.func @warp_specialize_isolated_regions(%arg0: tensor<1xi64>) {
  // CHECK-NEXT: local_alloc
  %0 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
  // CHECK-NEXT: local_store
  ttg.local_store %arg0, %0 : tensor<1xi64> -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NEXT: local_load
  ttg.local_load %0 : !ttg.memdesc<1xi64, #layout, #smem, mutable> -> tensor<1xi64>
```
**EN:** This block defines `warp_specialize_isolated_regions` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warp_specialize_isolated_regions`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 766-783
```mlir
  // CHECK-NEXT: warp_specialize
  ttg.warp_specialize()
  default {
    ttg.warp_yield
  }
  // CHECK: partition0
  partition0() num_warps(4) {
    %cst = arith.constant dense<0> : tensor<1xi64>
    // CHECK: local_alloc
    %1 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
    // CHECK-NEXT: local_store
    ttg.local_store %cst, %1 : tensor<1xi64> -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: local_load
    ttg.local_load %1 : !ttg.memdesc<1xi64, #layout, #smem, mutable> -> tensor<1xi64>
    // CHECK-NEXT: warp_return
    ttg.warp_return
  } : () -> ()
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation. Embedded check comments (CHECK×2, CHECK-NEXT×5) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×2, CHECK-NEXT×5）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 785-786
```mlir
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 788-810
```mlir
// CHECK-LABEL: @warp_specialize_into_default
tt.func @warp_specialize_into_default(%arg0: tensor<1xi64>) {
  // CHECK-NEXT: local_alloc
  %0 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
  // CHECK-NEXT: local_store
  ttg.local_store %arg0, %0 : tensor<1xi64> -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
  // CHECK-NEXT: warp_specialize
  ttg.warp_specialize()
  // CHECK-NEXT: default
  default {
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: local_load
    ttg.local_load %0 : !ttg.memdesc<1xi64, #layout, #smem, mutable> -> tensor<1xi64>
    // CHECK-NEXT: ttg.barrier local
    ttg.barrier local
    // CHECK-NEXT: warp_yield
    ttg.warp_yield
  // CHECK-NEXT: () -> ()
  } : () -> ()
  // CHECK-NEXT: local_store
  ttg.local_store %arg0, %0 : tensor<1xi64> -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
  tt.return
}
```
**EN:** This block defines `warp_specialize_into_default` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×10) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warp_specialize_into_default`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×10）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 812-843
```mlir
// CHECK-LABEL: @default_region_cfg
tt.func @default_region_cfg(%arg0: tensor<1xi64>, %arg1: i1) {
  // CHECK-NEXT: local_alloc
  %0 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
  // CHECK-NEXT: local_store
  ttg.local_store %arg0, %0 : tensor<1xi64> -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
  // CHECK-NEXT: warp_specialize
  ttg.warp_specialize()
  // CHECK-NEXT: default
  default {
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: local_load
    ttg.local_load %0 : !ttg.memdesc<1xi64, #layout, #smem, mutable> -> tensor<1xi64>
    cf.cond_br %arg1, ^bb1, ^bb2
  // CHECK: ^bb1:
  ^bb1:
    // CHECK-NEXT: ttg.barrier local
    ttg.barrier local
    cf.br ^bb3
  ^bb2:
    cf.br ^bb3
  // CHECK: ^bb3:
  ^bb3:
    // CHECK-NEXT: warp_yield
    ttg.warp_yield
  // CHECK-NEXT: () -> ()
  } : () -> ()
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NEXT: local_store
  ttg.local_store %arg0, %0 : tensor<1xi64> -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
  tt.return
}
```
**EN:** This block defines `default_region_cfg` and exercises shared-memory allocation. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×11) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `default_region_cfg`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×11）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 845
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 847-849
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#shared`, `#smem`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#shared`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Line 851
```mlir
module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 853-882
```mlir
// CHECK-LABEL: @direct_backedge_within_loop
tt.func @direct_backedge_within_loop(%arg0: index, %arg1: index, %arg2: index, %arg3: !tt.ptr<f16>, %arg4: !tt.ptr<f16>, %arg5: i1) {
  // CHECK-NEXT: constant
  %cst = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #blocked>
  // CHECK-NEXT: local_alloc
  %0 = ttg.local_alloc %cst : (tensor<128x32xf16, #blocked>) -> !ttg.memdesc<128x32xf16, #shared, #smem>
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NEXT: local_load
  %1 = ttg.local_load %0 : !ttg.memdesc<128x32xf16, #shared, #smem> -> tensor<128x32xf16, #blocked>
  // CHECK-NEXT: br
  cf.br ^bb1(%arg0, %0 : index, !ttg.memdesc<128x32xf16, #shared, #smem>)
^bb1(%2: index, %3: !ttg.memdesc<128x32xf16, #shared, #smem>):
  cf.cond_br %arg5, ^bb2, ^bb3
// CHECK: ^bb2:
^bb2:
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NEXT: local_alloc
  %4 = ttg.local_alloc %cst : (tensor<128x32xf16, #blocked>) -> !ttg.memdesc<128x32xf16, #shared, #smem>
  // CHECK-NEXT: br
  cf.br ^bb1(%arg1, %4 : index, !ttg.memdesc<128x32xf16, #shared, #smem>)
// CHECK: ^bb3
^bb3:
  // CHECK-NEXT: ttg.barrier local
  // CHECK-NEXT: local_load
  %5 = ttg.local_load %3 : !ttg.memdesc<128x32xf16, #shared, #smem> -> tensor<128x32xf16, #blocked>
  // CHECK-NEXT: cond_br
  cf.cond_br %arg5, ^bb3, ^bb4
^bb4:
  tt.return
}
```
**EN:** This block defines `direct_backedge_within_loop` and exercises shared-memory allocation. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×11) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `direct_backedge_within_loop`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×11）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 884
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 886
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 888
```mlir
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`. They parameterize later tests with compact names for `#ttg.swizzled_shared`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`。

### Line 890
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 892-924
```mlir
// CHECK-LABEL: @membar_alias_through_warp_specialize
tt.func @membar_alias_through_warp_specialize() {
  %0 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>
  ttg.warp_specialize(%0)
  default {
    ttg.warp_yield
  }
  // CHECK: partition0
  partition0(%arg0: !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>) num_warps(2) {
    %c0 = arith.constant 0 : i32
    %1 = ttg.memdesc_subslice %arg0 [0, 0]  : !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>
    %c = arith.constant dense<0.0> : tensor<16x16xf16>
    // CHECK: local_store
    ttg.local_store %c, %1 : tensor<16x16xf16> -> !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: local_store
    ttg.local_store %c, %1 : tensor<16x16xf16> -> !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>
    ttg.warp_return
  }
  // CHECK: partition1
  partition1(%arg0: !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>) num_warps(2) {
    %c0 = arith.constant 0 : i32
    %1 = ttg.memdesc_subslice %arg0 [0, 0]  : !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>
    %c = arith.constant dense<0.0> : tensor<16x16xf16>
    // CHECK: local_store
    ttg.local_store %c, %1 : tensor<16x16xf16> -> !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: local_store
    ttg.local_store %c, %1 : tensor<16x16xf16> -> !ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>
    ttg.warp_return
  } : (!ttg.memdesc<16x16xf16, #shared, #ttg.shared_memory, mutable>) -> ()
  tt.return
}
```
**EN:** This block defines `membar_alias_through_warp_specialize` and exercises shared-memory allocation. Embedded check comments (CHECK×4, CHECK-LABEL×1, CHECK-NEXT×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `membar_alias_through_warp_specialize`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×4, CHECK-LABEL×1, CHECK-NEXT×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 926
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 928
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 930-931
```mlir
#layout = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [0]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#layout`, `#smem`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#layout`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 933-956
```mlir
// CHECK-LABEL: @check_barrier_no_duplication
tt.func @check_barrier_no_duplication(%arg0: tensor<1xi64>) {
  // CHECK-NEXT: local_alloc
  %0 = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
  // CHECK-NEXT: local_store
  ttg.local_store %arg0, %0 : tensor<1xi64> -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
  // CHECK-NEXT: warp_specialize
  ttg.warp_specialize()
  // CHECK-NEXT: default
  default {
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: local_load
    ttg.local_load %0 : !ttg.memdesc<1xi64, #layout, #smem, mutable> -> tensor<1xi64>
    // CHECK-NEXT: ttg.barrier
    // CHECK-NOT: ttg.barrier
    ttg.barrier local
    // CHECK-NEXT: warp_yield
    ttg.warp_yield
  // CHECK-NEXT: () -> ()
  } : () -> ()
  // CHECK-NEXT: local_store
  ttg.local_store %arg0, %0 : tensor<1xi64> -> !ttg.memdesc<1xi64, #layout, #smem, mutable>
  tt.return
}
```
**EN:** This block defines `check_barrier_no_duplication` and exercises shared-memory allocation. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×10, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `check_barrier_no_duplication`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×10, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 958
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 959-960
```mlir
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 8, order = [1, 0]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`, `#smem`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 962-1008
```mlir
// CHECK-LABEL: @subslice_aliasing
tt.func public @subslice_aliasing(%data: tensor<128x128xf16>) {
    // CHECK: ttg.local_alloc
    %alloc = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    // CHECK-NEXT: ttg.memdesc_subslice
    %view0 = ttg.memdesc_subslice %alloc[0, 0] : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128>
    // CHECK-NEXT: ttg.memdesc_subslice
    %view1 = ttg.memdesc_subslice %alloc[0, 64] : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128>
    // CHECK-NEXT: ttg.memdesc_subslice
    %view2 = ttg.memdesc_subslice %alloc[64, 0] : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128>
    // CHECK-NEXT: ttg.memdesc_subslice
    %view3 = ttg.memdesc_subslice %alloc[64, 64] : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128>
    // CHECK-NEXT: ttg.local_store
    ttg.local_store %data, %alloc : tensor<128x128xf16> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    // RAW between 128x128 store and %data0 local_load, both access part of %view0
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %data0 = ttg.local_load %view0 : !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128> -> tensor<64x64xf16>
    // WAR between %data0 load and the store, both access %view0
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: ttg.local_store
    ttg.local_store %data0, %view0 : tensor<64x64xf16> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128>
    // CHECK-NEXT: ttg.local_load
    %data1 = ttg.local_load %view1 : !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128> -> tensor<64x64xf16>
    // WAR between %data1 load and the store, both access %view1
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: ttg.local_store
    ttg.local_store %data1, %view1 : tensor<64x64xf16> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128>
    // CHECK-NEXT: ttg.local_load
    %data2 = ttg.local_load %view2 : !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128> -> tensor<64x64xf16>
    // WAR between %data2 load and the store, both access %view2
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: ttg.local_store
    ttg.local_store %data2, %view2 : tensor<64x64xf16> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128>
    // CHECK-NEXT: ttg.local_load
    %data3 = ttg.local_load %view3 : !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128> -> tensor<64x64xf16>
    // WAR between %data3 load and the store, both access %view3
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: ttg.local_store
    ttg.local_store %data3, %view3 : tensor<64x64xf16> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128>
    // RAW between %view3 store and %all_res load, both access part of %view3
    // CHECK-NEXT: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %all_res = ttg.local_load %alloc : !ttg.memdesc<128x128xf16, #shared, #smem, mutable, 128x128> -> tensor<128x128xf16>
    // CHECK-NEXT: return
    tt.return
}
```
**EN:** This block defines `subslice_aliasing` and exercises shared-memory allocation. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×21) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `subslice_aliasing`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×21）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1010
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1011-1013
```mlir
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#sharedT = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [0, 1]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`, `#sharedT`, `#smem`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`, `#sharedT`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 1015-1028
```mlir
// CHECK-LABEL: layout_changed_reinterpret
tt.func @layout_changed_reinterpret() {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf16>
  %alloc = ttg.local_alloc %cst : (tensor<16x16xf16>) -> !ttg.memdesc<16x16xf16, #shared, #smem>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %0 = ttg.local_load %alloc : !ttg.memdesc<16x16xf16, #shared, #smem> -> tensor<16x16xf16>
  // CHECK-NEXT: ttg.memdesc_reinterpret
  %reinterpreted = ttg.memdesc_reinterpret %alloc : !ttg.memdesc<16x16xf16, #shared, #smem> -> !ttg.memdesc<16x16xf16, #sharedT, #smem>
  // CHECK-NOT: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %1 = ttg.local_load %reinterpreted : !ttg.memdesc<16x16xf16, #sharedT, #smem> -> tensor<16x16xf16>
  tt.return
}
```
**EN:** This block defines `layout_changed_reinterpret` and exercises shared-memory allocation. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×3, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `layout_changed_reinterpret`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×3, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1030
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1031-1033
```mlir
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#sharedT = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [0, 1]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`, `#sharedT`, `#smem`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`, `#sharedT`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 1035-1053
```mlir
// CHECK-LABEL: layout_changed_reinterpret_subslice
tt.func @layout_changed_reinterpret_subslice() {
  %cst_alloc = arith.constant dense<0.000000e+00> : tensor<32x16xf16>
  %cst_store = arith.constant dense<0.000000e+00> : tensor<16x16xf16>
  %alloc = ttg.local_alloc %cst_alloc : (tensor<32x16xf16>) -> !ttg.memdesc<32x16xf16, #shared, #smem, mutable>
  %subslice1 = ttg.memdesc_subslice %alloc [0, 0] : !ttg.memdesc<32x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %0 = ttg.local_load %subslice1 : !ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16> -> tensor<16x16xf16>
  %reinterpreted_parent = ttg.memdesc_reinterpret %alloc : !ttg.memdesc<32x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<32x16xf16, #sharedT, #smem, mutable>
  %reinterpreted = ttg.memdesc_subslice %reinterpreted_parent [16, 0] : !ttg.memdesc<32x16xf16, #sharedT, #smem, mutable> -> !ttg.memdesc<16x16xf16, #sharedT, #smem, mutable, 32x16>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_store
  ttg.local_store %cst_store, %reinterpreted : tensor<16x16xf16> -> !ttg.memdesc<16x16xf16, #sharedT, #smem, mutable, 32x16>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %1 = ttg.local_load %subslice1 : !ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16> -> tensor<16x16xf16>
  tt.return
}
```
**EN:** This block defines `layout_changed_reinterpret_subslice` and exercises shared-memory allocation. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `layout_changed_reinterpret_subslice`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1055
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1056-1058
```mlir
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#sharedT = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [0, 1]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`, `#sharedT`, `#smem`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`, `#sharedT`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 1060-1073
```mlir
// CHECK-LABEL: reinterpret_then_multiple_loads
tt.func @reinterpret_then_multiple_loads() {
  %cst_f16 = arith.constant dense<0.000000e+00> : tensor<16x16xf16>
  %cst_f32 = arith.constant dense<0.000000e+00> : tensor<16x8xf32>
  %alloc = ttg.local_alloc %cst_f16 : (tensor<16x16xf16>) -> !ttg.memdesc<16x16xf16, #shared, #smem, mutable>
  %reinterpreted = ttg.memdesc_reinterpret %alloc : !ttg.memdesc<16x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<16x8xf32, #sharedT, #smem, mutable>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %0 = ttg.local_load %reinterpreted : !ttg.memdesc<16x8xf32, #sharedT, #smem, mutable> -> tensor<16x8xf32>
  // CHECK-NOT: ttg.barrier local
  // CHECK: ttg.local_load
  %1 = ttg.local_load %reinterpreted : !ttg.memdesc<16x8xf32, #sharedT, #smem, mutable> -> tensor<16x8xf32>
  tt.return
}
```
**EN:** This block defines `reinterpret_then_multiple_loads` and exercises shared-memory allocation. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `reinterpret_then_multiple_loads`，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1075
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1076-1077
```mlir
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`, `#smem`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 1079-1103
```mlir
// CHECK-LABEL: loop_with_indexed_memdesc
// Test that a loop carried memdesc_index is conservatively
// marked as overlapping.
tt.func @loop_with_indexed_memdesc(%lb : index, %ub : index) {
  %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf16>
  %step = arith.constant 1 : index
  %c0_i32 = arith.constant 0 : i32
  %c2_i32 = arith.constant 2 : i32
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<2x128x128xf16, #shared, #smem, mutable>
  %view0 = ttg.memdesc_index %alloc[%c0_i32] : !ttg.memdesc<2x128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
  ttg.local_store %cst, %view0 : tensor<128x128xf16> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
  %result = scf.for %iv = %lb to %ub step %step iter_args(%iter_view = %view0) -> (!ttg.memdesc<128x128xf16, #shared, #smem, mutable>) {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %load = ttg.local_load %iter_view : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16>
    %iv_i32 = arith.index_cast %iv : index to i32
    %next_idx = arith.remui %iv_i32, %c2_i32 : i32
    %next_view = ttg.memdesc_index %alloc[%next_idx] : !ttg.memdesc<2x128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_store
    ttg.local_store %load, %next_view : tensor<128x128xf16> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    scf.yield %next_view : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
  }
  tt.return
}
```
**EN:** This block defines `loop_with_indexed_memdesc` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `loop_with_indexed_memdesc`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1105
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1106-1107
```mlir
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`, `#smem`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 1109-1138
```mlir
// CHECK-LABEL: loop_subslice_iterarg
// Test that a loop carried memdesc_subslice is conservatively
// marked as overlapping.
tt.func @loop_subslice_iterarg() {
  %cst = arith.constant dense<0.000000e+00> : tensor<32x16xf16>
  %c0 = arith.constant 0 : index
  %c1 = arith.constant 1 : index
  %c2 = arith.constant 2 : index
  %c0_i32 = arith.constant 0 : i32
  %alloc = ttg.local_alloc %cst : (tensor<32x16xf16>) -> !ttg.memdesc<32x16xf16, #shared, #smem, mutable>
  %subA = ttg.memdesc_subslice %alloc[0, 0] : !ttg.memdesc<32x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16>
  %subB = ttg.memdesc_subslice %alloc[16, 0] : !ttg.memdesc<32x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16>
  %result = scf.for %iv = %c0 to %c2 step %c1 iter_args(%cur = %subA) -> (!ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16>) {
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_load
    %val = ttg.local_load %cur : !ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16> -> tensor<16x16xf16>
    %iv_i32 = arith.index_cast %iv : index to i32
    %isZero = arith.cmpi eq, %iv_i32, %c0_i32 : i32
    %next = scf.if %isZero -> !ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16> {
      scf.yield %subB : !ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16>
    } else {
      scf.yield %subA : !ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16>
    }
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_store
    ttg.local_store %val, %next : tensor<16x16xf16> -> !ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16>
    scf.yield %next : !ttg.memdesc<16x16xf16, #shared, #smem, mutable, 32x16>
  }
  tt.return
}
```
**EN:** This block defines `loop_subslice_iterarg` and exercises shared-memory allocation, loop-carried state, control-flow joins. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `loop_subslice_iterarg`，并覆盖 共享内存分配、循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1140
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1141-1142
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`, `#smem`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 1144-1171
```mlir
// CHECK-LABEL: two_subslices_with_if
// Test that a subslice with partly unknown offsets is treated conservatively.
tt.func @two_subslices_with_if() {
  %cst_dummy = arith.constant dense<1.000000e+00> : tensor<16x16xf16>
  %cst_store = arith.constant dense<2.000000e+00> : tensor<8x8xf16>
  %c1 = arith.constant 1 : i1
  %alloc = ttg.local_alloc %cst_dummy : (tensor<16x16xf16>) -> !ttg.memdesc<16x16xf16, #shared, #smem, mutable>
  // CHECK: ttg.local_store
  ttg.local_store %cst_dummy, %alloc : tensor<16x16xf16> -> !ttg.memdesc<16x16xf16, #shared, #smem, mutable>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_load
  %loaded = ttg.local_load %alloc : !ttg.memdesc<16x16xf16, #shared, #smem, mutable> -> tensor<16x16xf16>
  %subsliceA = ttg.memdesc_subslice %alloc[8, 8] : !ttg.memdesc<16x16xf16, #shared, #smem, mutable> -> !ttg.memdesc<8x8xf16, #shared, #smem, mutable, 16x16>
  %subsliceA1 = scf.if %c1 -> !ttg.memdesc<8x8xf16, #shared, #smem, mutable, 16x16> {
    scf.yield %subsliceA : !ttg.memdesc<8x8xf16, #shared, #smem, mutable, 16x16>
  } else {
    scf.yield %subsliceA : !ttg.memdesc<8x8xf16, #shared, #smem, mutable, 16x16>
  }
  %cst_store_4x4 = arith.constant dense<2.000000e+00> : tensor<4x4xf16>
  %subsliceA2 = ttg.memdesc_subslice %subsliceA1[0, 0] : !ttg.memdesc<8x8xf16, #shared, #smem, mutable, 16x16> -> !ttg.memdesc<4x4xf16, #shared, #smem, mutable, 16x16>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_store
  ttg.local_store %cst_store_4x4, %subsliceA2 : tensor<4x4xf16> -> !ttg.memdesc<4x4xf16, #shared, #smem, mutable, 16x16>
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.local_store
  ttg.local_store %cst_store, %subsliceA : tensor<8x8xf16> -> !ttg.memdesc<8x8xf16, #shared, #smem, mutable, 16x16>
  tt.return
}
```
**EN:** This block defines `two_subslices_with_if` and exercises shared-memory allocation, control-flow joins. Embedded check comments (CHECK×4, CHECK-LABEL×1, CHECK-NEXT×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `two_subslices_with_if`，并覆盖 共享内存分配、控制流汇合。 其中嵌入的检查注释（CHECK×4, CHECK-LABEL×1, CHECK-NEXT×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1173
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1174-1175
```mlir
#shared = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`, `#smem`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 1177-1206
```mlir
// CHECK-LABEL: loop_memindex_subslice
tt.func @loop_memindex_subslice(%arg0: tensor<2x128x128xf16>) {
  %c0 = arith.constant 0 : index
  %c1 = arith.constant 1 : index
  %c2 = arith.constant 2 : index
  %c0_i32 = arith.constant 0 : i32
  %c1_i32 = arith.constant 1 : i32
  // CHECK: ttg.local_alloc
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<2x128x128xf16, #shared, #smem, mutable>
  // CHECK: ttg.memdesc_index
  %base = ttg.memdesc_index %alloc[%c0_i32] : !ttg.memdesc<2x128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
  %result = scf.for %iv = %c0 to %c2 step %c1 iter_args(%cur = %base) -> (!ttg.memdesc<128x128xf16, #shared, #smem, mutable>) {
    // CHECK: ttg.memdesc_subslice
    %top_left = ttg.memdesc_subslice %cur[0, 0] : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128>
    // CHECK: ttg.memdesc_subslice
    %bottom_right = ttg.memdesc_subslice %cur[64, 64] : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128>
    // CHECK-NEXT: ttg.local_load
    %tile = ttg.local_load %top_left : !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128> -> tensor<64x64xf16>
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.local_store
    ttg.local_store %tile, %bottom_right : tensor<64x64xf16> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable, 128x128>
    %iv_i32 = arith.index_cast %iv : index to i32
    %next = arith.addi %iv_i32, %c1_i32 : i32
    // CHECK: ttg.memdesc_index
    %next_view = ttg.memdesc_index %alloc[%next] : !ttg.memdesc<2x128x128xf16, #shared, #smem, mutable> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    scf.yield %next_view : !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
  }
  // CHECK: return
  tt.return
}
```
**EN:** This block defines `loop_memindex_subslice` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×7, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `loop_memindex_subslice`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×7, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1208
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1209-1213
```mlir
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#shared3 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
#smem = #ttg.shared_memory
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 32]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared1`, `#shared2`, `#shared3`, `#smem`, `#mma`. They parameterize later tests with compact names for `#ttg.nvmma_shared`, `#ttg.swizzled_shared`, `#ttg.shared_memory`, `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared1`, `#shared2`, `#shared3`, `#smem`, `#mma`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvmma_shared`, `#ttg.swizzled_shared`, `#ttg.shared_memory`, `#ttg.nvidia_mma`。

### Lines 1215-1217
```mlir
module attributes {ttg.target = "cuda:90", "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: warp_dot_multi_read
  tt.func @warp_dot_multi_read(%arg0: !tt.tensordesc<1x256x128xf8E5M2, #shared1>, %arg1: tensor<128x128x!tt.ptr<f8E5M2>>, %arg2: i32, %arg3: i1, %arg4: tensor<128x256xf32, #mma>, %arg5: tensor<128x128xi1>) {
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `warp_dot_multi_read`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `warp_dot_multi_read`。

### Lines 1219-1221
```mlir
    %a_tile = ttg.local_alloc : () -> !ttg.memdesc<128x128xf8E5M2, #shared1, #smem, mutable>
    %b_tile = ttg.local_alloc : () -> !ttg.memdesc<256x128xf8E5M2, #shared1, #smem, mutable>
    %barrier = ttg.local_alloc : () -> !ttg.memdesc<1xi64, #shared2, #smem, mutable>
```
**EN:** This block contributes intermediate IR built from `ttg.local_alloc`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_alloc`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1223
```mlir
    %b_trans = ttg.memdesc_trans %b_tile {order = array<i32: 1, 0>} : !ttg.memdesc<256x128xf8E5M2, #shared1, #smem, mutable> -> !ttg.memdesc<128x256xf8E5M2, #shared3, #smem, mutable>
```
**EN:** This block contributes intermediate IR built from `ttg.memdesc_trans`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.memdesc_trans`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 1225-1226
```mlir
    %dot = ttng.warp_group_dot %a_tile, %b_trans, %arg4 {inputPrecision = 0 : i32, isAsync = true, maxNumImpreciseAcc = 1073741824 : i32} : !ttg.memdesc<128x128xf8E5M2, #shared1, #smem, mutable> * !ttg.memdesc<128x256xf8E5M2, #shared3, #smem, mutable> -> tensor<128x256xf32, #mma>
    %0:3 = ttng.warp_group_dot_wait %dot, %a_tile, %b_trans {pendings = 1 : i32} : tensor<128x256xf32, #mma>, !ttg.memdesc<128x128xf8E5M2, #shared1, #smem, mutable>, !ttg.memdesc<128x256xf8E5M2, #shared3, #smem, mutable>
```
**EN:** This block contributes intermediate IR built from `ttng.warp_group_dot`, `ttg.memdesc`, `ttng.warp_group_dot_wait`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttng.warp_group_dot`, `ttg.memdesc`, `ttng.warp_group_dot_wait` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 1228-1234
```mlir
    // CHECK: ttg.barrier local
    // CHECK-NEXT: ttg.async_copy_global_to_local
    ttg.async_copy_global_to_local %arg1, %a_tile mask %arg5 {contiguity = 16 : i32} : tensor<128x128x!tt.ptr<f8E5M2>> -> <128x128xf8E5M2, #shared1, #smem, mutable>
    ttng.async_tma_copy_global_to_local %arg0[%arg2, %arg2, %arg2] %b_tile, %barrier, %arg3 : !tt.tensordesc<1x256x128xf8E5M2, #shared1>, !ttg.memdesc<1xi64, #shared2, #smem, mutable> -> !ttg.memdesc<256x128xf8E5M2, #shared1, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies. Embedded check comments (CHECK×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1236
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1238-1243
```mlir
#blockedLarge = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#sharedLarge = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#blockedCallSrc = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#mmaCall = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
#blockedCallDst = #ttg.dot_op<{opIdx = 0, parent = #mmaCall, kWidth = 2}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blockedLarge`, `#sharedLarge`, `#blockedCallSrc`, `#mmaCall`, `#blockedCallDst`, `#smem`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.swizzled_shared`, `#ttg.nvidia_mma`, `#ttg.dot_op`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blockedLarge`, `#sharedLarge`, `#blockedCallSrc`, `#mmaCall`, `#blockedCallDst`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.swizzled_shared`, `#ttg.nvidia_mma`, `#ttg.dot_op`, `#ttg.shared_memory`。

### Lines 1245-1250
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  tt.func private @callee_call_offset_membar() -> tensor<128x32xf16, #blockedCallDst> {
    %cst = arith.constant dense<0.0> : tensor<128x32xf16, #blockedCallSrc>
    %cvt = ttg.convert_layout %cst : tensor<128x32xf16, #blockedCallSrc> -> tensor<128x32xf16, #blockedCallDst>
    tt.return %cvt : tensor<128x32xf16, #blockedCallDst>
  }
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `callee_call_offset_membar`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `callee_call_offset_membar`。

### Lines 1252-1261
```mlir
  // The call's virtual buffer is offset by the large allocation. The
  // subsequent scratch op should alias at the same offset and require a membar.
  // CHECK-LABEL: @caller_call_offset_membar
  // CHECK: tt.call @callee_call_offset_membar{{.*}}allocation.offset = [[CALL_OFFSET:[1-9][0-9]*]]
  // CHECK: ttg.barrier local
  // CHECK-NEXT: ttg.convert_layout{{.*}}allocation.offset = [[CALL_OFFSET]]
  tt.func @caller_call_offset_membar() -> tensor<128x32xf16, #blockedCallDst> {
    %large = arith.constant dense<0> : tensor<65536xi8, #blockedLarge>
    %buf = ttg.local_alloc : () -> !ttg.memdesc<65536xi8, #sharedLarge, #smem, mutable>
    ttg.local_store %large, %buf : tensor<65536xi8, #blockedLarge> -> !ttg.memdesc<65536xi8, #sharedLarge, #smem, mutable>
```
**EN:** This block defines `caller_call_offset_membar` and exercises shared-memory allocation, layout conversion. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `caller_call_offset_membar`，并覆盖 共享内存分配、布局转换。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1263
```mlir
    %call = tt.call @callee_call_offset_membar() : () -> tensor<128x32xf16, #blockedCallDst>
```
**EN:** This block contributes intermediate IR built from `tt.call`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.call` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 1265-1267
```mlir
    %cst = arith.constant dense<0.0> : tensor<128x32xf16, #blockedCallSrc>
    %cvt = ttg.convert_layout %cst : tensor<128x32xf16, #blockedCallSrc> -> tensor<128x32xf16, #blockedCallDst>
    %sum = arith.addf %call, %cvt : tensor<128x32xf16, #blockedCallDst>
```
**EN:** This block contributes intermediate IR built from `arith.constant`, `ttg.convert_layout`, `arith.addf`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `arith.constant`, `ttg.convert_layout`, `arith.addf` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 1269-1273
```mlir
    %ld = ttg.local_load %buf : !ttg.memdesc<65536xi8, #sharedLarge, #smem, mutable> -> tensor<65536xi8, #blockedLarge>
    ttg.local_dealloc %buf : !ttg.memdesc<65536xi8, #sharedLarge, #smem, mutable>
    tt.return %sum : tensor<128x32xf16, #blockedCallDst>
  }
}
```
**EN:** This block contributes intermediate IR built from `ttg.local_load`, `ttg.memdesc`, `ttg.local_dealloc`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_load`, `ttg.memdesc`, `ttg.local_dealloc`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's analysis coverage and focuses on memory barrier semantics.  
  **CN:** 该文件属于 Triton 的 分析 测试覆盖，关注点是 内存屏障语义。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory`, `-test-print-membar`, `-test-tritonamdgpu-membar`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory`, `-test-print-membar`, `-test-tritonamdgpu-membar` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `scf` (structured control flow), `cf` (control-flow ops), `ttng` (next-gen TritonGPU ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`scf`（结构化控制流）、`cf`（控制流操作）、`ttng`（新一代 TritonGPU 操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `scf`, `cf`, `ttng`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`scf`、`cf`、`ttng`。
