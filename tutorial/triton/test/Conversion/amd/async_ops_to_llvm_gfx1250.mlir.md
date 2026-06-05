# async_ops_to_llvm_gfx1250.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/async_ops_to_llvm_gfx1250.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises asynchronous copy/lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的异步拷贝/降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×24, CHECK-COUNT×10, CHECK-LABEL×16, CHECK-NEXT×4, CHECK-NOT×12; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×24, CHECK-COUNT×10, CHECK-LABEL×16, CHECK-NEXT×4, CHECK-NOT×12；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that asynchronous copy/lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 异步拷贝/降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-18
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 8, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_copy_with_swizzle
  tt.func public @async_copy_with_swizzle(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg2: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    // Each thread needs to load 8 elements and we load 1 (sizePerThread) per global.load.lds
    // CHECK-COUNT-8: rocdl.global.load.async.to.lds.b32
    // CHECK-NOT: rocdl.global.load.async.to.lds
    %2 = ttg.async_copy_global_to_local %1, %arg2 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_with_swizzle`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_with_swizzle`。

### Line 20
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 22-35
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_load_strided_into_lds_with_swizzle
  tt.func public @async_load_strided_into_lds_with_swizzle(%arg0: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>},
                                %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // Each thread loads 256 contiguous bits so we split into 2 128bit loads. This was not possible on GFX9
    // CHECK-COUNT-2: rocdl.global.load.async.to.lds.b128
    // CHECK-NOT: rocdl.global.load.async.to.lds
    %6 = ttg.async_copy_global_to_local %arg0, %arg1 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_load_strided_into_lds_with_swizzle`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_load_strided_into_lds_with_swizzle`。

### Line 37
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 39-54
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 8, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_copy_with_swizzle
  tt.func public @async_copy_with_swizzle(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg2: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    // Each thread needs to load 8 elements and we load 1 (sizePerThread) per global.load.lds
    // CHECK-COUNT-8: rocdl.global.load.async.to.lds.b32
    // CHECK-NOT: rocdl.global.load.async.to.lds
    %2 = ttg.async_copy_global_to_local %1, %arg2 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_with_swizzle`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_with_swizzle`。

### Line 56
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 58-67
```mlir
// Broadcast to all CTAs so we should just see 15 (0b1111) as the broadcast mask since we have 4 CTAs per CGA
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[0, 0], [0, 0]]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0], CGALayout = [[0, 0], [0, 0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_load_multicast_to_all_ctas
  tt.func public @async_load_multicast_to_all_ctas(%arg0: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>},
                                %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // CHECK: %[[GROUP_MASK:.*]] = llvm.mlir.constant(15 : i32) : i32
    // CHECK: rocdl.cluster.load.async.to.lds{{.*}}, {{.*}}, {{.*}}, {{.*}}, %[[GROUP_MASK]]
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_load_multicast_to_all_ctas`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_load_multicast_to_all_ctas`。

### Lines 69-72
```mlir
    %6 = ttg.async_copy_global_to_local %arg0, %arg1 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `ttg.async_copy_global_to_local`, `tt.ptr`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.async_copy_global_to_local`, `tt.ptr`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 74
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 76-93
```mlir
// 8 CTAs, 2 multicast groups of 4 CTAs each. Each group is strided by 1 so the base mask should be 0b1010101 (85) and the non free mask is -7 (~0b110)
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[1, 0], [0, 0], [0, 0]]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0], CGALayout = [[1, 0], [0, 0], [0, 0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_load_multicast_to_half_ctas
  tt.func public @async_load_multicast_to_half_ctas(%arg0: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>},
                                %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // CHECK: %[[CTA_ID:.*]] = rocdl.cluster.workgroup.id.x
    // CHECK: %[[NON_FREE_BITS:.*]] = llvm.mlir.constant(-7 : i32) : i32
    // CHECK: %[[SHIFT_AMOUNT:.*]] = llvm.and %[[CTA_ID]], %[[NON_FREE_BITS]]
    // CHECK: %[[GROUP_MASK:.*]] = llvm.mlir.constant(85 : i32) : i32
    // CHECK: %[[CTA_MASK:.*]] = llvm.shl %[[GROUP_MASK]], %[[SHIFT_AMOUNT]]
    // CHECK: rocdl.cluster.load.async.to.lds{{.*}}, {{.*}}, {{.*}}, {{.*}}, %[[CTA_MASK]]
    %6 = ttg.async_copy_global_to_local %arg0, %arg1 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_load_multicast_to_half_ctas`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_load_multicast_to_half_ctas`。

### Line 95
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 97-115
```mlir
// 16 CTAs, 8 multicast groups of 2 CTAs each, each group is strided by 8 so the base mask should be 0b100000001 (257) and the non free mask is -9 (~0b1000)
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[0, 1], [0, 2], [0, 4], [0, 0]]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0], CGALayout = [[0, 1], [0, 2], [0, 4], [0, 0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 16 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_load_multicast_group_of_2_strided_by_8
  tt.func public @async_load_multicast_group_of_2_strided_by_8(%arg0: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>},
                                %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // Skip the first cluster id because it's emitted for address calculation
    // CHECK: %[[CTA_ID:.*]] = rocdl.cluster.workgroup.id.x
    // CHECK: %[[NON_FREE_BITS:.*]] = llvm.mlir.constant(-9 : i32) : i32
    // CHECK: %[[SHIFT_AMOUNT:.*]] = llvm.and %[[CTA_ID]], %[[NON_FREE_BITS]]
    // CHECK: %[[GROUP_MASK:.*]] = llvm.mlir.constant(257 : i32) : i32
    // CHECK: %[[CTA_MASK:.*]] = llvm.shl %[[GROUP_MASK]], %[[SHIFT_AMOUNT]]
    // CHECK: rocdl.cluster.load.async.to.lds{{.*}}, {{.*}}, {{.*}}, {{.*}}, %[[CTA_MASK]]
    %6 = ttg.async_copy_global_to_local %arg0, %arg1 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_load_multicast_group_of_2_strided_by_8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_load_multicast_group_of_2_strided_by_8`。

### Line 117
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 119-133
```mlir
// 16 CTAs split into 16 multicast groups so we should not emit cluster load since we do not share any data
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[0, 1], [0, 2], [0, 4], [0, 8]]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0], CGALayout = [[0, 1], [0, 2], [0, 4], [0, 8]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 16 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_load_multi_cta_but_not_data_sharing
  tt.func public @async_load_multi_cta_but_not_data_sharing(%arg0: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>},
                                %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // CHECK-NOT: rocdl.cluster.load.async.to.lds
    // CHECK: rocdl.global.load.async.to.lds.b64
    // CHECK-NOT: rocdl.cluster.load.async.to.lds
    %6 = ttg.async_copy_global_to_local %arg0, %arg1 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_load_multi_cta_but_not_data_sharing`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_load_multi_cta_but_not_data_sharing`。

### Line 135
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 137-156
```mlir
// Test with linear layout as src layout
// 16 CTAs, 8 multicast groups of 2 CTAs each, each group is strided by 8 so the base mask should be 0b100000001 (257) and the non free mask is -9 (~0b1000)
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[0, 0], [0, 0], [1, 0], [2, 0], [4, 0]], warp = [[8, 0], [16, 0]], block = [[0, 4], [0, 8], [0, 16], [0, 0]], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0], CGALayout = [[0, 1], [0, 2], [0, 4], [0, 0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 16 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_load_multi_cta_linear_layout
  tt.func public @async_load_multi_cta_linear_layout(%arg0: tensor<32x32x!tt.ptr<f32>, #linear> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>},
                                %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // Skip the first cluster id because it's emitted for address calculation
    // CHECK: %[[CTA_ID:.*]] = rocdl.cluster.workgroup.id.x
    // CHECK: %[[NON_FREE_BITS:.*]] = llvm.mlir.constant(-9 : i32) : i32
    // CHECK: %[[SHIFT_AMOUNT:.*]] = llvm.and %[[CTA_ID]], %[[NON_FREE_BITS]]
    // CHECK: %[[GROUP_MASK:.*]] = llvm.mlir.constant(257 : i32) : i32
    // CHECK: %[[CTA_MASK:.*]] = llvm.shl %[[GROUP_MASK]], %[[SHIFT_AMOUNT]]
    // CHECK: rocdl.cluster.load.async.to.lds{{.*}}, {{.*}}, {{.*}}, {{.*}}, %[[CTA_MASK]]
    %6 = ttg.async_copy_global_to_local %arg0, %arg1 : tensor<32x32x!tt.ptr<f32>, #linear> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_load_multi_cta_linear_layout`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_load_multi_cta_linear_layout`。

### Line 158
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 160-175
```mlir
// Test async_copy_local_to_global - basic case
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_copy_local_to_global_basic
  tt.func public @async_copy_local_to_global_basic(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                                   %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    // Each thread stores 8 elements with 32-bit stores
    // CHECK-COUNT-8: llvm.amdgcn.global.store.async.from.lds.b32
    // CHECK-NOT: llvm.amdgcn.global.store.async.from.lds
    %2 = amdg.async_copy_local_to_global %arg1, %1 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_local_to_global_basic`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_local_to_global_basic`。

### Line 177
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 179-193
```mlir
// Test async_copy_local_to_global with larger vector size
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_copy_local_to_global_vec128
  tt.func public @async_copy_local_to_global_vec128(%arg0: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>},
                                                    %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // Each thread stores 8 elements (256 bits), split into 2 128-bit stores
    // CHECK-COUNT-2: llvm.amdgcn.global.store.async.from.lds.b128
    // CHECK-NOT: llvm.amdgcn.global.store.async.from.lds
    %2 = amdg.async_copy_local_to_global %arg1, %arg0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_local_to_global_vec128`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_local_to_global_vec128`。

### Line 195
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 197-212
```mlir
// Test async_copy_global_to_local with padded shared layout
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[8:+4] {order = [1, 0], shape = [32, 32]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_copy_global_to_local_padded
  tt.func public @async_copy_global_to_local_padded(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                                    %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    // Each thread loads 8 elements with 32-bit loads
    // CHECK-COUNT-8: rocdl.global.load.async.to.lds.b32
    // CHECK-NOT: rocdl.global.load.async.to.lds
    %2 = ttg.async_copy_global_to_local %1, %arg1 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_global_to_local_padded`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_global_to_local_padded`。

### Line 214
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 216-231
```mlir
// Test async_copy_local_to_global with padded shared layout
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[8:+4] {order = [1, 0], shape = [32, 32]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_copy_local_to_global_padded
  tt.func public @async_copy_local_to_global_padded(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                                    %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    // Each thread stores 8 elements with 32-bit stores
    // CHECK-COUNT-8: llvm.amdgcn.global.store.async.from.lds.b32
    // CHECK-NOT: llvm.amdgcn.global.store.async.from.lds
    %2 = amdg.async_copy_local_to_global %arg1, %1 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_local_to_global_padded`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_local_to_global_padded`。

### Line 233
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 235-254
```mlir
// Test that minInterval limits vectorization for async_copy_global_to_local
// sizePerThread = [1, 4] would normally allow 128-bit (4 x f32) loads,
// but minInterval = 2 limits to 64-bit (2 x f32) loads
// Layout covers 32x16, tensor is 32x32, so 2 repetitions in dim1
// Each thread handles 1*4*1*2 = 8 elements -> 4 x 64-bit loads
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[2:+2] {order = [1, 0], shape = [32, 32]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_copy_global_to_local_padded_limited_vec
  tt.func public @async_copy_global_to_local_padded_limited_vec(%arg0: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>},
                                                                %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // minInterval=2 limits vectorization to 2 elements (64 bits)
    // Each thread handles 8 elements -> 4 x 64-bit loads
    // CHECK-COUNT-4: rocdl.global.load.async.to.lds.b64
    // CHECK-NOT: rocdl.global.load.async.to.lds
    %2 = ttg.async_copy_global_to_local %arg0, %arg1 : tensor<32x32x!tt.ptr<f32>, #blocked> -> <32x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_global_to_local_padded_limited_vec`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_global_to_local_padded_limited_vec`。

### Line 256
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 258-277
```mlir
// Test that minInterval limits vectorization for async_copy_local_to_global
// sizePerThread = [1, 4] would normally allow 128-bit (4 x f32) stores,
// but minInterval = 2 limits to 64-bit (2 x f32) stores
// Layout covers 32x16, tensor is 32x32, so 2 repetitions in dim1
// Each thread handles 1*4*1*2 = 8 elements -> 4 x 64-bit stores
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.padded_shared<[2:+2] {order = [1, 0], shape = [32, 32]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: async_copy_local_to_global_padded_limited_vec
  tt.func public @async_copy_local_to_global_padded_limited_vec(%arg0: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>},
                                                                %arg1: !ttg.memdesc<32x32xf32, #shared, #smem, mutable>) {
    // minInterval=2 limits vectorization to 2 elements (64 bits)
    // Each thread handles 8 elements -> 4 x 64-bit stores
    // CHECK-COUNT-4: llvm.amdgcn.global.store.async.from.lds.b64
    // CHECK-NOT: llvm.amdgcn.global.store.async.from.lds
    %2 = amdg.async_copy_local_to_global %arg1, %arg0 : !ttg.memdesc<32x32xf32, #shared, #smem, mutable> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_local_to_global_padded_limited_vec`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_local_to_global_padded_limited_vec`。

### Line 279
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 281-298
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: async_copy_masking
  tt.func public @async_copy_masking(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32},
                                %arg1: i32 {tt.divisibility = 16 : i32},
                                %arg2: !ttg.memdesc<4x32xf32, #shared, #smem, mutable>,
                                %arg3: i32 {tt.divisibility = 16 : i32}) {
    // We need the splat to allow the AxisAnalysis to work during lowering
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<4x32xf32, #blocked>
    %c0_i32 = arith.constant 0 : i32
    %c32_i32 = arith.constant 32 : i32
    %c31_i32 = arith.constant 31 : i32
    %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<4x32x!tt.ptr<f32>, #blocked>
    %29 = arith.addi %arg3, %c31_i32 : i32
    %30 = arith.divsi %29, %c32_i32 : i32
    %31 = arith.cmpi sgt, %30, %c0_i32 : i32
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_masking`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_masking`。

### Lines 300-304
```mlir
    %51 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %52 = tt.expand_dims %51 {axis = 1 : i32} : tensor<4xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<4x1xi32, #blocked>
    %65 = tt.splat %arg3 : i32 -> tensor<4x1xi32, #blocked>
    %66 = arith.cmpi slt, %52, %65 : tensor<4x1xi32, #blocked>
    %67 = tt.broadcast %66 : tensor<4x1xi1, #blocked> -> tensor<4x32xi1, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.make_range`, `tt.expand_dims`, `tt.splat`, `arith.cmpi`, `tt.broadcast`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.make_range`, `tt.expand_dims`, `tt.splat`, `arith.cmpi`, `tt.broadcast` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 306-307
```mlir
    %70 = tt.splat %31 : i1 -> tensor<4x32xi1, #blocked>
    %71 = arith.andi %70, %67 : tensor<4x32xi1, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `arith.andi`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `arith.andi` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 309-319
```mlir
    // CHECK: %[[NEG1:.*]] = llvm.mlir.constant(2147483647 : i32)
    // CHECK-NEXT: %[[OOB_LDS:.*]] = llvm.inttoptr %[[NEG1]] :
    // CHECK-NEXT: %[[LDS_PTR:.*]] = llvm.select %{{.*}}, %{{.*}}, %[[OOB_LDS]]
    // CHECK-NEXT: rocdl.global.load.async.to.lds{{.*}} %{{.*}}, %[[LDS_PTR]],
    // CHECK: llvm.cond_br
    // CHECK: llvm.store
    // CHECK-NEXT: llvm.br
    %2 = ttg.async_copy_global_to_local %1, %arg2 mask %67 other %cst_0 : tensor<4x32x!tt.ptr<f32>, #blocked> -> <4x32xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies. Embedded check comments (CHECK×3, CHECK-NEXT×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝。 其中嵌入的检查注释（CHECK×3, CHECK-NEXT×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 321
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 323-339
```mlir
// Test async_copy_local_to_global with 16-bit data type.
// On gfx1250, 16-bit stores are not directly supported, so they should be split into two 8-bit stores.
// sizePerThread = [1] with f16 (16-bit) should generate two b8 stores per element.
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {ttg.target = "hip:gfx1250", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 2048 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: async_copy_local_to_global_16bit_split
  tt.func public @async_copy_local_to_global_16bit_split(%dst: tensor<128x!tt.ptr<f16>, #blocked>,
                                                          %smem: !ttg.memdesc<128xf16, #shared, #smem, mutable>) {
    // Each thread stores 1 element (16 bits), split into 2 x 8-bit stores
    // CHECK-COUNT-2: llvm.amdgcn.global.store.async.from.lds.b8
    // CHECK-NOT: llvm.amdgcn.global.store.async.from.lds
    %0 = amdg.async_copy_local_to_global %smem, %dst : !ttg.memdesc<128xf16, #shared, #smem, mutable> -> tensor<128x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_local_to_global_16bit_split`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_local_to_global_16bit_split`。

### Line 341
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on asynchronous copy/lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 异步拷贝/降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `llvm` (LLVM dialect IR), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `llvm`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`llvm`、`arith`。
