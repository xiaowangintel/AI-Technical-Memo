# tritongpu_to_llvm_gfx1250.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/tritongpu_to_llvm_gfx1250.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises AMD-specific lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的AMD 专用降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx1250" | FileCheck %s --check-prefix=GFX1250` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Validation / 验证内容:** **EN:** The file validates that AMD-specific lowering runs successfully under the specified pass pipeline. **CN:** 该文件验证 AMD 专用降级 能在指定的 pass 流水线下成功运行。

## Line-by-Line Analysis / 逐行分析

### Lines 1-14
```mlir
// RUN:  triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch="gfx1250" | FileCheck %s --check-prefix=GFX1250
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 8], [0, 16]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 4]], warp = [[16, 0]], block = []}>
#mma = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[1, 0]]}, isTranspose = true, instrShape = [16, 16, 32]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // GFX1250-LABEL: wmma_permlane16_swap
  tt.func @wmma_permlane16_swap(%arg0: tensor<32x32xf16, #mma>) {
    // GFX1250-NOT: store
    // GFX1250-NOT: load
    // GFX1250-COUNT-4: llvm.call_intrinsic "llvm.amdgcn.permlane16.swap"
    // GFX1250-NOT: llvm.call_intrinsic "llvm.amdgcn.permlane16.swap"
    %0 = ttg.convert_layout %arg0 : tensor<32x32xf16, #mma> -> tensor<32x32xf16, #linear>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_permlane16_swap`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_permlane16_swap`。

### Line 16
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 18-29
```mlir
#mma = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[1, 0], [2, 0]]}, isTranspose = true, instrShape = [16, 16, 32]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // GFX1250-LABEL: reduce_16x16
  tt.func @reduce_16x16(%input: tensor<128x128xf32, #mma>) {
    // GFX1250-COUNT-2: llvm.call_intrinsic "llvm.amdgcn.permlane16.swap"
    %0 = "tt.reduce"(%input) <{axis = 1 : i32}> ({
      ^bb0(%arg1: f32 , %arg2: f32):
      %2 = "arith.maxnumf"(%arg1, %arg2) : (f32, f32) -> f32
      tt.reduce.return %2 : f32 }) : (tensor<128x128xf32, #mma>) -> tensor<128xf32, #ttg.slice<{dim = 1, parent = #mma}>>
   tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_16x16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_16x16`。

### Line 31
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 33-46
```mlir
// Test lowering of operations with PartitionedSharedEncodingAttr using padded_shared layout
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [2, 1], order = [1, 0]}>
#inner_padded = #ttg.padded_shared<[128:+4] {order = [1, 0], shape = [16, 16]}>
#partitioned = #ttg.partitioned_shared<{numPartitions = 2, numGroups = 2, partitionDim = 0, partitionLayout = #inner_padded}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // GFX1250-LABEL: partitioned_shared_padded_local_alloc
  tt.func @partitioned_shared_padded_local_alloc(%arg0: tensor<16x16xf16, #blocked>) {
    // GFX1250: llvm.mlir.addressof @global_smem
    // GFX1250-COUNT-4: llvm.store {{.*}} : vector<{{[0-9]+}}xf16>, !llvm.ptr<3>
    %0 = ttg.local_alloc %arg0 : (tensor<16x16xf16, #blocked>) -> !ttg.memdesc<16x16xf16, #partitioned, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `partitioned_shared_padded_local_alloc`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `partitioned_shared_padded_local_alloc`。

### Line 48
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 50-64
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [2, 1], order = [1, 0]}>
#inner_padded = #ttg.padded_shared<[128:+4] {order = [1, 0], shape = [16, 16]}>
#partitioned = #ttg.partitioned_shared<{numPartitions = 2, numGroups = 2, partitionDim = 0, partitionLayout = #inner_padded}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // GFX1250-LABEL: partitioned_shared_padded_local_load
  tt.func @partitioned_shared_padded_local_load() -> tensor<16x16xf16, #blocked> {
    // Allocate and then load from partitioned shared memory
    // GFX1250: llvm.mlir.addressof @global_smem
    // GFX1250-COUNT-4: llvm.load {{.*}} : !llvm.ptr<3> -> vector<{{[0-9]+}}xf16>
    %0 = ttg.local_alloc {allocation.offset = [0 : i32, 65536 : i32, 128 : i32, 65664 : i32]} : () -> !ttg.memdesc<16x16xf16, #partitioned, #smem, mutable>
    %1 = ttg.local_load %0 : !ttg.memdesc<16x16xf16, #partitioned, #smem, mutable> -> tensor<16x16xf16, #blocked>
    tt.return %1 : tensor<16x16xf16, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `partitioned_shared_padded_local_load`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `partitioned_shared_padded_local_load`。

### Line 66
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 68-82
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [2, 1], order = [1, 0]}>
#inner_padded = #ttg.padded_shared<[128:+4] {order = [1, 0], shape = [16, 16]}>
#partitioned = #ttg.partitioned_shared<{numPartitions = 2, numGroups = 2, partitionDim = 0, partitionLayout = #inner_padded}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // GFX1250-LABEL: partitioned_shared_padded_local_store
  tt.func @partitioned_shared_padded_local_store(%arg0: tensor<16x16xf16, #blocked>) {
    // Allocate and then store to partitioned shared memory
    // GFX1250: llvm.mlir.addressof @global_smem
    // GFX1250-COUNT-4: llvm.store {{.*}} : vector<{{[0-9]+}}xf16>, !llvm.ptr<3>
    %0 = ttg.local_alloc {allocation.offset = [0 : i32, 65536 : i32, 128 : i32, 65664 : i32]} : () -> !ttg.memdesc<16x16xf16, #partitioned, #smem, mutable>
    ttg.local_store %arg0, %0 : tensor<16x16xf16, #blocked> -> !ttg.memdesc<16x16xf16, #partitioned, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `partitioned_shared_padded_local_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `partitioned_shared_padded_local_store`。

### Line 84
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 86-95
```mlir
#blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // GFX1250-LABEL: @bf16_mulf
  tt.func @bf16_mulf(%arg0: tensor<64xbf16, #blocked>, %arg1: tensor<64xf8E4M3FN, #blocked>) -> tensor<64xbf16, #blocked> {
    // GFX1250: rocdl.cvt.scale.pk8.bf16.fp8
    // GFX1250: llvm.fmul {{.*}} : vector<2xbf16>
    %0 = tt.fp_to_fp %arg1 : tensor<64xf8E4M3FN, #blocked> -> tensor<64xbf16, #blocked>
    %1 = arith.mulf %arg0, %0 : tensor<64xbf16, #blocked>
    tt.return %1 : tensor<64xbf16, #blocked>
  }
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `bf16_mulf`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `bf16_mulf`。

### Lines 97-103
```mlir
  // GFX1250-LABEL: @bf16_addf
  tt.func @bf16_addf(%arg0: tensor<64xbf16, #blocked>, %arg1: tensor<64xbf16, #blocked>) -> tensor<64xbf16, #blocked> {
    // GFX1250-NOT: llvm.fadd {{.*}} : f32
    // GFX1250: llvm.fadd {{.*}} : vector<2xbf16>
    %0 = arith.addf %arg0, %arg1 : tensor<64xbf16, #blocked>
    tt.return %0 : tensor<64xbf16, #blocked>
  }
```
**EN:** This block defines `bf16_addf` and exercises operations such as `tt.func`, `llvm.fadd`, `arith.addf`, `tt.return`.
**CN:** 这一块定义了 `bf16_addf`，并覆盖 如 `tt.func`、`llvm.fadd`、`arith.addf`、`tt.return` 这样的操作。

### Lines 105-112
```mlir
  // GFX1250-LABEL: @bf16_subf
  tt.func @bf16_subf(%arg0: tensor<64xbf16, #blocked>, %arg1: tensor<64xbf16, #blocked>) -> tensor<64xbf16, #blocked> {
    // GFX1250-NOT: llvm.fsub {{.*}} : f32
    // GFX1250: llvm.fsub {{.*}} : vector<2xbf16>
    %0 = arith.subf %arg0, %arg1 : tensor<64xbf16, #blocked>
    tt.return %0 : tensor<64xbf16, #blocked>
  }
}
```
**EN:** This block defines `bf16_subf` and exercises operations such as `tt.func`, `llvm.fsub`, `arith.subf`, `tt.return`.
**CN:** 这一块定义了 `bf16_subf`，并覆盖 如 `tt.func`、`llvm.fsub`、`arith.subf`、`tt.return` 这样的操作。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on AMD-specific lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 AMD 专用降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch="gfx1250"`, `--check-prefix=GFX1250`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch="gfx1250"`, `--check-prefix=GFX1250` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `llvm` (LLVM dialect IR), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）、`arith`（标量/张量算术）。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `llvm`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`llvm`、`arith`。
