# cluster_load.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/cluster_load.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises memory loads in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的内存读取相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×7, CHECK-COUNT×3, CHECK-LABEL×7, CHECK-NOT×6; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×7, CHECK-COUNT×3, CHECK-LABEL×7, CHECK-NOT×6；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that memory loads produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 内存读取 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-12
```mlir
// CGA layout has no broadcasting so we should not emit cluster loads
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[1, 0], [2, 0], [4, 0]]}>
module attributes {"ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: load_multi_cta_but_no_broadcast
  tt.func public @load_multi_cta_but_no_broadcast(%arg0: tensor<32x32x!tt.ptr<f32>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>}) {
    // CHECK-NOT: llvm.amdgcn.cluster.load.b128
    %6 = tt.load %arg0 : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `load_multi_cta_but_no_broadcast`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `load_multi_cta_but_no_broadcast`。

### Line 14
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 16-31
```mlir
// 8 CTAs, 2 multicast groups of 4 CTAs each. Each group is strided by 1 so the base mask should be 0b1010101 (85) and the non free mask is -7 (~0b110)
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[1, 0], [0, 0], [0, 0]]}>
module attributes {"ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: cluster_load_b128
  tt.func public @cluster_load_b128(%arg0: tensor<32x32x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>}) {
    // CHECK: %[[CTA_ID:.*]] = rocdl.cluster.workgroup.id.x
    // CHECK: %[[NON_FREE_BITS:.*]] = llvm.mlir.constant(-7 : i32) : i32
    // CHECK: %[[SHIFT_AMOUNT:.*]] = llvm.and %[[CTA_ID]], %[[NON_FREE_BITS]]
    // CHECK: %[[GROUP_MASK:.*]] = llvm.mlir.constant(85 : i32) : i32
    // CHECK: %[[CTA_MASK:.*]] = llvm.shl %[[GROUP_MASK]], %[[SHIFT_AMOUNT]]
    // CHECK: llvm.amdgcn.cluster.load.b128{{.*}}, {{.*}}, %[[CTA_MASK]]
    // CHECK-NOT: llvm.amdgcn.cluster.load
    %6 = tt.load %arg0 : tensor<32x32x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cluster_load_b128`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cluster_load_b128`。

### Line 33
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 35-45
```mlir
// Note that we already check the correct multicast mask in previous tests, so we only check the cluster load instruction here
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[1, 0], [0, 0], [0, 0]]}>
module attributes {"ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: cluster_load_b64
  tt.func public @cluster_load_b64(%arg0: tensor<32x32x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>}) {
    // CHECK-COUNT-2: llvm.amdgcn.cluster.load.b64
    // CHECK-NOT: llvm.amdgcn.cluster.load
    %6 = tt.load %arg0 : tensor<32x32x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cluster_load_b64`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cluster_load_b64`。

### Line 47
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 49-59
```mlir
// Note that we already check the correct multicast mask in previous tests, so we only check the cluster load instruction here
#blocked = #ttg.blocked<{sizePerThread = [1, 2], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[1, 0], [0, 0], [0, 0]]}>
module attributes {"ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: cluster_load_b32
  tt.func public @cluster_load_b32(%arg0: tensor<32x32x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>}) {
    // CHECK-COUNT-4: llvm.amdgcn.cluster.load.b32
    // CHECK-NOT: llvm.amdgcn.cluster.load
    %6 = tt.load %arg0 : tensor<32x32x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cluster_load_b32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cluster_load_b32`。

### Line 61
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 63-72
```mlir
// Smaller vector size than 2 (32bit) should not produce cluster loads
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[1, 0], [0, 0], [0, 0]]}>
module attributes {"ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: not_cluster_load_for_b16
  tt.func public @not_cluster_load_for_b16(%arg0: tensor<32x32x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>}) {
    // CHECK-NOT: llvm.amdgcn.cluster.load
    %6 = tt.load %arg0 : tensor<32x32x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `not_cluster_load_for_b16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `not_cluster_load_for_b16`。

### Line 74
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 76-87
```mlir
// Check that we break sizePerThread > 4 (>128bit) into multiple cluster loads b128
// Note that we already check the correct multicast mask in previous tests, so we only check the cluster load instruction here
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0], CGALayout = [[1, 0], [0, 0], [0, 0]]}>
module attributes {"ttg.num-ctas" = 8 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 8192 : i32} {
  // CHECK-LABEL: cluster_load_2_b128
  tt.func public @cluster_load_2_b128(%arg0: tensor<32x32x!tt.ptr<f16>, #blocked> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>, tt.contiguity = dense<[16, 16]> : tensor<2xi32>, tt.constancy = dense<[1, 1]> : tensor<2xi32>}) {
    // CHECK-COUNT-2: llvm.amdgcn.cluster.load.b128
    // CHECK-NOT: llvm.amdgcn.cluster.load
    %6 = tt.load %arg0 : tensor<32x32x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cluster_load_2_b128`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cluster_load_2_b128`。

### Line 89
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 91-107
```mlir
// Check that scalar loads works without emitting cluster load
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: scalar_load_gfx1250
  tt.func public @scalar_load_gfx1250(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: !tt.ptr<i32> {tt.divisibility = 16 : i32}) {
    %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked>
    // Scalar load should produce a regular llvm.load, not a cluster load
    // CHECK: llvm.load %{{.*}} : !llvm.ptr<1> -> vector<1xi16>
    %1 = tt.load %arg1 : !tt.ptr<i16>
    %2 = amdg.buffer_load %arg2[%0] : tensor<128xi32, #blocked>
    %3 = arith.extsi %1 : i16 to i32
    %4 = tt.splat %3 : i32 -> tensor<128xi32, #blocked>
    %5 = arith.ori %4, %2 : tensor<128xi32, #blocked>
    amdg.buffer_store %5, %arg0[%0] : tensor<128xi32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `scalar_load_gfx1250`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `scalar_load_gfx1250`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on memory loads.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 内存读取。
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
