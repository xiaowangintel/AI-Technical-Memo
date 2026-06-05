# tritongpu_tdm_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/tritongpu_tdm_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises AMD-specific lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的AMD 专用降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --convert-builtin-func-to-llvm | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×39, CHECK-DAG×16, CHECK-LABEL×14, CHECK-NOT×1, CHECK-SAME×2; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×39, CHECK-DAG×16, CHECK-LABEL×14, CHECK-NOT×1, CHECK-SAME×2；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that AMD-specific lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 AMD 专用降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s --split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --convert-builtin-func-to-llvm | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s --split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --convert-builtin-func-to-llvm | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s --split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --convert-builtin-func-to-llvm | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-28
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [64, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_load
  tt.func public @tdm_load(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c_shape = arith.constant 128 : i32
    %c_stride0 = arith.constant 128 : i64
    %c_stride1 = arith.constant 1 : i64
    %c_offset = arith.constant 0 : i32
    %c_pred = arith.constant 1 : i32
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%c_stride0, %c_stride1] : <f16>, <64x64xf16, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    // The descriptor is built as two vectors (<4xi32> + <8xi32>) via a
    // sequence of insertelement / extractelement ops; just check the final
    // intrinsic call gets the right operand types.
    // CHECK: llvm.insertelement{{.*}} : vector<4xi32>
    // CHECK: llvm.insertelement{{.*}} : vector<8xi32>
    // CHECK: "llvm.amdgcn.tensor.load.to.lds"({{.+}}) : (vector<4xi32>, vector<8xi32>, vector<4xi32>, vector<4xi32>, vector<8xi32>, i32) -> ()
    %2 = amdg.async_tdm_copy_global_to_local %0[%c_offset, %c_offset] into %1, pred = %c_pred : !tt.tensordesc<64x64xf16, #shared> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    // CHECK: rocdl.s.wait.tensorcnt 0
    %3 = amdg.async_tdm_intrinsic_wait  {count = 0 : i32}
    %4 = ttg.local_load %1 : !ttg.memdesc<64x64xf16, #shared, #smem, mutable> -> tensor<64x64xf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_load`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_load`。

### Line 30
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 32-57
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_store
  tt.func public @tdm_store(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c_shape = arith.constant 128 : i32
    %c_stride0 = arith.constant 128 : i64
    %c_stride1 = arith.constant 1 : i64
    %c_offset = arith.constant 0 : i32
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%c_stride0, %c_stride1] : <f16>, <64x64xf16, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    %2 = arith.constant dense<0.000000e+00> : tensor<64x64xf16, #blocked>
    ttg.local_store %2, %1 : tensor<64x64xf16, #blocked> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    // The descriptor is built as two vectors (<4xi32> + <8xi32>) via a
    // sequence of insertelement / extractelement ops; just check the final
    // intrinsic call gets the right operand types.
    // CHECK: llvm.insertelement{{.*}} : vector<4xi32>
    // CHECK: llvm.insertelement{{.*}} : vector<8xi32>
    // CHECK: "llvm.amdgcn.tensor.store.from.lds"({{.+}}) : (vector<4xi32>, vector<8xi32>, vector<4xi32>, vector<4xi32>, vector<8xi32>, i32) -> ()
    amdg.async_tdm_copy_local_to_global %0[%c_offset, %c_offset] from %1: !ttg.memdesc<64x64xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x64xf16, #shared>
    // CHECK: rocdl.s.wait.tensorcnt 0
    %3 = amdg.async_tdm_intrinsic_wait  {count = 0 : i32}
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_store`。

### Line 59
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 61-71
```mlir
// Check that CTA offsets are computed and applied to base pointer for multi-cta layouts
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0], CGALayout = [[0, 0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_load_multi_cta
  tt.func public @tdm_load_multi_cta(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c_shape = arith.constant 128 : i32
    %c_stride0 = arith.constant 128 : i64
    %c_stride1 = arith.constant 1 : i64
    %c_offset = arith.constant 0 : i32
    %c_pred = arith.constant 1 : i32
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_load_multi_cta`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_load_multi_cta`。

### Lines 73-84
```mlir
    // CHECK-DAG: %[[STRIDE0:.*]] = llvm.mlir.constant(128 : i64) : i64
    // CHECK-DAG: %[[STRIDE1:.*]] = llvm.mlir.constant(1 : i32) : i32
    // CHECK: %[[STRIDE0_TRUNC:.*]] = llvm.trunc %[[STRIDE0]] : i64 to i32
    // The stride is inserted into group1 (<8 x i32>), then the CTA-offset
    // computation re-extracts it to multiply into the per-dim offset.  The
    // per-dim offsets are summed and the total feeds `getelementptr` for
    // the load.
    // CHECK: llvm.insertelement %[[STRIDE0_TRUNC]]{{.*}}: vector<8xi32>
    // CHECK: llvm.mul
    // CHECK: llvm.getelementptr
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%c_stride0, %c_stride1] : <f16>, <64x64xf16, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation. Embedded check comments (CHECK×4, CHECK-DAG×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×4, CHECK-DAG×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 86-90
```mlir
    // CHECK: "llvm.amdgcn.tensor.load.to.lds"({{.+}}) : (vector<4xi32>, vector<8xi32>, vector<4xi32>, vector<4xi32>, vector<8xi32>, i32) -> ()
    %2 = amdg.async_tdm_copy_global_to_local %0[%c_offset, %c_offset] into %1, pred = %c_pred : !tt.tensordesc<64x64xf16, #shared> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `llvm.amdgcn.tensor.load.to.lds`, `amdg.async_tdm_copy_global_to_local`, `tt.tensordesc`, `ttg.memdesc`, `tt.return`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `llvm.amdgcn.tensor.load.to.lds`、`amdg.async_tdm_copy_global_to_local`、`tt.tensordesc`、`ttg.memdesc`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 92
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 94-104
```mlir
// Check that CTA offsets are computed and applied to base pointer for multi-cta layouts (store)
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0], CGALayout = [[0, 1]]}>
#blocked_store = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_store_multi_cta
  tt.func public @tdm_store_multi_cta(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c_shape = arith.constant 128 : i32
    %c_stride0 = arith.constant 128 : i64
    %c_stride1 = arith.constant 1 : i64
    %c_offset = arith.constant 0 : i32
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_store_multi_cta`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_store_multi_cta`。

### Lines 106-121
```mlir
    // CHECK-DAG: %[[STRIDE0:.*]] = llvm.mlir.constant(128 : i64) : i64
    // CHECK-DAG: %[[STRIDE1:.*]] = llvm.mlir.constant(1 : i32) : i32
    // CHECK-DAG: rocdl.cluster.workgroup.id.x
    // CHECK-DAG: %[[STRIDE0_TRUNC:.*]] = llvm.trunc %[[STRIDE0]] : i64 to i32
    // Same as the load case above: stride is inserted into group1, then
    // re-extracted for the CTA-offset multiply, which feeds getelementptr.
    // CHECK-DAG: llvm.insertelement %[[STRIDE0_TRUNC]]{{.*}}: vector<8xi32>
    // CHECK: llvm.mul
    // CHECK: llvm.getelementptr
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%c_stride0, %c_stride1] : <f16>, <64x64xf16, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    // CHECK: "llvm.amdgcn.tensor.store.from.lds"({{.+}}) : (vector<4xi32>, vector<8xi32>, vector<4xi32>, vector<4xi32>, vector<8xi32>, i32) -> ()
    amdg.async_tdm_copy_local_to_global %0[%c_offset, %c_offset] from %1: !ttg.memdesc<64x64xf16, #shared, #smem, mutable> -> !tt.tensordesc<64x64xf16, #shared>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation. Embedded check comments (CHECK×3, CHECK-DAG×5) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×3, CHECK-DAG×5）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 123
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 125-134
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0], CGALayout = [[0, 1], [0, 2], [0, 0], [0, 0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 16 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_load_multicast
  tt.func public @tdm_load_multicast(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c_shape = arith.constant 128 : i32
    %c_stride0 = arith.constant 128 : i64
    %c_stride1 = arith.constant 1 : i64
    %c_offset = arith.constant 0 : i32
    %c_pred = arith.constant 1 : i32
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_load_multicast`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_load_multicast`。

### Lines 136-148
```mlir
    // Check we compute the multicast mask and used it in the second group of SGPRs (vector<8xi32>)
    // CHECK-DAG: %[[GROUP_MASK:.*]] = llvm.mlir.constant(4369 : i32) : i32
    // CHECK-DAG: %[[NON_FREE_BITS:.*]] = llvm.mlir.constant(-13 : i32) : i32
    // CHECK-DAG: %[[CTA_ID:.*]] = rocdl.cluster.workgroup.id.x
    // CHECK: %[[SHIFT_AMOUNT:.*]] = llvm.and %[[CTA_ID]], %[[NON_FREE_BITS]]
    // CHECK: %[[CTA_MASK:.*]] = llvm.shl %[[GROUP_MASK]], %[[SHIFT_AMOUNT]]
    // Combine with other values
    // CHECK: %[[TMP:.*]] = llvm.or %{{.*}}, %[[CTA_MASK]]
    // CHECK: %[[TMP2:.*]] = llvm.and %[[TMP]]
    // CHECK-NOT: llvm.insertelement{{.*}} : vector<8xi32>
    // CHECK: llvm.insertelement %[[TMP2]]
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%c_stride0, %c_stride1] : <f16>, <64x64xf16, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation. Embedded check comments (CHECK×5, CHECK-DAG×3, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×5, CHECK-DAG×3, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 151-155
```mlir
    // CHECK: "llvm.amdgcn.tensor.load.to.lds"({{.+}}) : (vector<4xi32>, vector<8xi32>, vector<4xi32>, vector<4xi32>, vector<8xi32>, i32) -> ()
    %2 = amdg.async_tdm_copy_global_to_local %0[%c_offset, %c_offset] into %1, pred = %c_pred : !tt.tensordesc<64x64xf16, #shared> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `llvm.amdgcn.tensor.load.to.lds`, `amdg.async_tdm_copy_global_to_local`, `tt.tensordesc`, `ttg.memdesc`, `tt.return`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `llvm.amdgcn.tensor.load.to.lds`、`amdg.async_tdm_copy_global_to_local`、`tt.tensordesc`、`ttg.memdesc`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 157
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 159-169
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [64, 64]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_prefetch_regular
  tt.func public @tdm_prefetch_regular(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %c_shape = arith.constant 128 : i32
    %c_stride0 = arith.constant 128 : i64
    %c_stride1 = arith.constant 1 : i64
    %c_offset = arith.constant 0 : i32
    %c_pred = arith.constant true
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%c_stride0, %c_stride1] : <f16>, <64x64xf16, #shared>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_prefetch_regular`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_prefetch_regular`。

### Lines 171-172
```mlir
    // CHECK: rocdl.global.prefetch %{{.*}} scope 8
    amdg.tdm_prefetch %0[%c_offset, %c_offset], %c_pred, speculative = false : !tt.tensordesc<64x64xf16, #shared>
```
**EN:** This block defines the test function(s) and exercises operations such as `rocdl.global.prefetch`, `amdg.tdm_prefetch`, `tt.tensordesc`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `rocdl.global.prefetch`、`amdg.tdm_prefetch`、`tt.tensordesc` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 174-178
```mlir
    // CHECK: rocdl.global.prefetch %{{.*}} scope 9
    amdg.tdm_prefetch %0[%c_offset, %c_offset], %c_pred, speculative = true : !tt.tensordesc<64x64xf16, #shared>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `rocdl.global.prefetch`, `amdg.tdm_prefetch`, `tt.tensordesc`, `tt.return`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `rocdl.global.prefetch`、`amdg.tdm_prefetch`、`tt.tensordesc`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 180
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 182-189
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_wait_inst
  tt.func public @tdm_wait_inst() {
    // CHECK: rocdl.s.wait.tensorcnt 0
    %3 = amdg.async_tdm_intrinsic_wait  {count = 0 : i32}
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_wait_inst`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_wait_inst`。

### Line 191
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 193-206
```mlir
#shared = #ttg.padded_shared<[64:+4] {order = [1, 0], shape = [128, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_2d_with_padding
  tt.func public @tdm_2d_with_padding(
    %tensorDesc: !tt.tensordesc<128x64xf16>,
    %memDesc: !ttg.memdesc<128x64xf16, #shared, #smem, mutable>
  ) {
    %c0_i32 = arith.constant 0 : i32
    amdg.async_tdm_copy_local_to_global %tensorDesc[%c0_i32, %c0_i32] from %memDesc: !ttg.memdesc<128x64xf16, #shared, #smem, mutable> -> !tt.tensordesc<128x64xf16>
    // CHECK: "llvm.amdgcn.tensor.store.from.lds"({{.+}}) : (vector<4xi32>, vector<8xi32>, vector<4xi32>, vector<4xi32>, vector<8xi32>, i32) -> ()
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_2d_with_padding`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_2d_with_padding`。

### Line 208
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 210-223
```mlir
#shared_5d = #ttg.padded_shared<[16:+4] {order = [4, 3, 2, 1, 0], shape = [8, 8, 8, 16, 16]}>
#smem_5d = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_5d_with_padding
  tt.func public @tdm_5d_with_padding(
    %tensorDesc: !tt.tensordesc<8x8x8x16x16xf16>,
    %memDesc: !ttg.memdesc<8x8x8x16x16xf16, #shared_5d, #smem_5d, mutable>
  ) {
    %c0_i32 = arith.constant 0 : i32
    amdg.async_tdm_copy_local_to_global %tensorDesc[%c0_i32, %c0_i32, %c0_i32, %c0_i32, %c0_i32] from %memDesc: !ttg.memdesc<8x8x8x16x16xf16, #shared_5d, #smem_5d, mutable> -> !tt.tensordesc<8x8x8x16x16xf16>
    // CHECK: "llvm.amdgcn.tensor.store.from.lds"({{.+}}) : (vector<4xi32>, vector<8xi32>, vector<4xi32>, vector<4xi32>, vector<8xi32>, i32) -> ()
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_5d_with_padding`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_5d_with_padding`。

### Line 225
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 227-244
```mlir
// Scatter with padded shared layout: padding interval = innermost block dim.
#idx_parent = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
#idx_layout = #ttg.slice<{dim = 1, parent = #idx_parent}>
#shared_scatter_pad = #ttg.padded_shared<[64:+8] {order = [1, 0], shape = [8, 64]}>
#smem_scatter_pad = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_scatter_with_padding
  tt.func public @tdm_scatter_with_padding(
    %tensorDesc: !tt.tensordesc<8x64xf16>,
    %memDesc: !ttg.memdesc<8x64xf16, #shared_scatter_pad, #smem_scatter_pad, mutable>,
    %row_indices: tensor<8xi32, #idx_layout>
  ) {
    %c0_i32 = arith.constant 0 : i32
    amdg.async_tdm_scatter %tensorDesc[%row_indices, %c0_i32] from %memDesc : tensor<8xi32, #idx_layout>, !ttg.memdesc<8x64xf16, #shared_scatter_pad, #smem_scatter_pad, mutable> -> !tt.tensordesc<8x64xf16>
    // CHECK: "llvm.amdgcn.tensor.store.from.lds"({{.+}}) : (vector<4xi32>, vector<8xi32>, vector<4xi32>, vector<4xi32>, vector<8xi32>, i32) -> ()
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_scatter_with_padding`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_scatter_with_padding`。

### Line 246
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 248-276
```mlir
// Partial TDM copy: warp_used_hint = 0x0F picks K=4 active warps out of 8.
// The warp sublayout of the TDM LinearLayout is an identity over K=4 warps
// (low 2 bits of warpId drive offsets), so the free-variable mask for the
// "warp" input dim is 0b100 (bit 2 is redundant).  Predication is
// (warpId & 4) == 0, i.e. warps 0..3 are active and warps 4..7 issue a
// no-op TDM.
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_load_warp_used_hint_predication
  tt.func public @tdm_load_warp_used_hint_predication(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) {
    %c_shape = arith.constant 256 : i32
    %c_stride0 = arith.constant 256 : i64
    %c_stride1 = arith.constant 1 : i64
    %c_offset = arith.constant 0 : i32
    %c_pred = arith.constant 1 : i32
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%c_stride0, %c_stride1] : <f16>, <256x64xf16, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<256x64xf16, #shared, #smem, mutable>
    // CHECK-DAG: %[[FREE_MASK:.*]] = llvm.mlir.constant(4 : i32) : i32
    // CHECK-DAG: %[[ZERO:.*]] = llvm.mlir.constant(0 : i32) : i32
    // CHECK: %[[MASKED:.*]] = llvm.and %{{.*}}, %[[FREE_MASK]] : i32
    // CHECK: %[[IS_ACTIVE:.*]] = llvm.icmp "eq" %[[MASKED]], %[[ZERO]] : i32
    // CHECK: %[[LAYOUT_PRED:.*]] = llvm.select %[[IS_ACTIVE]], %{{.*}}, %{{.*}} : i1, i32
    // CHECK: llvm.and %{{.*}}, %[[LAYOUT_PRED]] : i32
    // CHECK: "llvm.amdgcn.tensor.load.to.lds"
    %2 = amdg.async_tdm_copy_global_to_local %0[%c_offset, %c_offset] into %1, pred = %c_pred {warp_used_hint = 15 : i32} : !tt.tensordesc<256x64xf16, #shared> -> !ttg.memdesc<256x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_load_warp_used_hint_predication`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_load_warp_used_hint_predication`。

### Line 278
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 280-305
```mlir
// Partial TDM copy with partitioned layout.  warp_used_hint = 0x03 selects
// K=2 active warps out of 4, matching numLogicalPieces (2) so the copy
// fits in a single TDM instruction.  The redundant warp bit is bit 1 of
// warpId (free mask = 2).
#shared_inner = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#partitioned = #ttg.partitioned_shared<{numPartitions = 2, numGroups = 1, partitionDim = 0, partitionLayout = #shared_inner}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_load_warp_used_hint_partitioned
  tt.func public @tdm_load_warp_used_hint_partitioned(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) {
    %c_shape = arith.constant 256 : i32
    %c_stride0 = arith.constant 256 : i64
    %c_stride1 = arith.constant 1 : i64
    %c_offset = arith.constant 0 : i32
    %c_pred = arith.constant 1 : i32
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%c_stride0, %c_stride1] : <f16>, <128x16xf16, #partitioned>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #partitioned, #smem, mutable>
    // CHECK-DAG: %[[FREE_MASK:.*]] = llvm.mlir.constant(2 : i32) : i32
    // CHECK-DAG: %[[ZERO:.*]] = llvm.mlir.constant(0 : i32) : i32
    // CHECK: %[[MASKED:.*]] = llvm.and %{{.*}}, %[[FREE_MASK]] : i32
    // CHECK: llvm.icmp "eq" %[[MASKED]], %[[ZERO]] : i32
    // CHECK: "llvm.amdgcn.tensor.load.to.lds"
    %2 = amdg.async_tdm_copy_global_to_local %0[%c_offset, %c_offset] into %1, pred = %c_pred {warp_used_hint = 3 : i32} : !tt.tensordesc<128x16xf16, #partitioned> -> !ttg.memdesc<128x16xf16, #partitioned, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_load_warp_used_hint_partitioned`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_load_warp_used_hint_partitioned`。

### Line 307
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 309-329
```mlir
// TDM stride slots are 48 bits wide. Verify that an i64 stride is split
// into low-32 and high-16 pieces and not silently truncated to i32.
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [64, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_load_64bit_stride
  // CHECK-SAME: %{{.*}}: !llvm.ptr<1> {{.*}}, %[[STRIDE:.*]]: i64,
  tt.func public @tdm_load_64bit_stride(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32},
                                        %stride0: i64, %shape0: i32, %shape1: i32) {
    %c_stride1 = arith.constant 1 : i64
    %c_offset = arith.constant 0 : i32
    %c_pred = arith.constant 1 : i32
    // CHECK: %[[HI64:.*]] = llvm.lshr %[[STRIDE]], %{{.*}} : i64
    // CHECK: %[[HI32:.*]] = llvm.trunc %[[HI64]] : i64 to i32
    // CHECK: llvm.insertelement %{{.*}}, %{{.*}} : vector<8xi32>
    %0 = tt.make_tensor_descriptor %arg0, [%shape0, %shape1], [%stride0, %c_stride1] : <f16>, <64x64xf16, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    %2 = amdg.async_tdm_copy_global_to_local %0[%c_offset, %c_offset] into %1, pred = %c_pred : !tt.tensordesc<64x64xf16, #shared> -> !ttg.memdesc<64x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_load_64bit_stride`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_load_64bit_stride`。

### Line 331
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 333-352
```mlir
// Same as above but for 3D. 4D and 5D share the logic so a test would be redundant.
#shared = #ttg.padded_shared<[16:+4] {order = [2, 1, 0], shape = [4, 16, 64]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_load_64bit_stride_3d
  // CHECK-SAME: %{{.*}}: !llvm.ptr<1> {{.*}}, %[[S0:.*]]: i64, %[[S1:.*]]: i64,
  tt.func public @tdm_load_64bit_stride_3d(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32},
                                           %stride0: i64, %stride1: i64,
                                           %shape0: i32, %shape1: i32, %shape2: i32) {
    %c_stride2 = arith.constant 1 : i64
    %c_offset = arith.constant 0 : i32
    %c_pred = arith.constant 1 : i32
    // CHECK-DAG: llvm.lshr %[[S0]], %{{.*}} : i64
    // CHECK-DAG: llvm.lshr %[[S1]], %{{.*}} : i64
    %0 = tt.make_tensor_descriptor %arg0, [%shape0, %shape1, %shape2], [%stride0, %stride1, %c_stride2] : <f16>, <4x16x64xf16, #shared>
    %1 = ttg.local_alloc : () -> !ttg.memdesc<4x16x64xf16, #shared, #smem, mutable>
    %2 = amdg.async_tdm_copy_global_to_local %0[%c_offset, %c_offset, %c_offset] into %1, pred = %c_pred : !tt.tensordesc<4x16x64xf16, #shared> -> !ttg.memdesc<4x16x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_load_64bit_stride_3d`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_load_64bit_stride_3d`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on AMD-specific lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 AMD 专用降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--split-input-file`, `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250`, `--convert-builtin-func-to-llvm`.  
  **CN:** `RUN` 流水线会驱动 `--split-input-file`, `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250`, `--convert-builtin-func-to-llvm` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `llvm` (LLVM dialect IR).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`llvm`（LLVM 方言 IR）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `llvm`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`llvm`。
