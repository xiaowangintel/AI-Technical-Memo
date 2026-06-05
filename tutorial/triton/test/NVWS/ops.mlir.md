# ops.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/NVWS/ops.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises MMA lowering in Triton's NVWS transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton NVWS 变换中的MMA 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt --split-input-file %s | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×10, CHECK-LABEL×7, CHECK-NEXT×12; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×10, CHECK-LABEL×7, CHECK-NEXT×12；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that MMA lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 MMA 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt --split-input-file %s | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt --split-input-file %s | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt --split-input-file %s | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-11
```mlir
#shared0 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "cuda:0", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: aref_create_single
  // CHECK: nvws.aref.create
  tt.func @aref_create_single(%d : !ttg.memdesc<1x64x16xf16, #shared0, #smem>, %e : !ttg.memdesc<1x16x32xf16, #shared0, #smem>) {
    %0 = nvws.aref.create %d, %e : !nvws.aref<[!ttg.memdesc<1x64x16xf16, #shared0, #smem>, !ttg.memdesc<1x16x32xf16, #shared0, #smem>]>
    tt.return
  }
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `aref_create_single`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `aref_create_single`。

### Line 13
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 15
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 17-30
```mlir
#shared0 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "cuda:0", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: aref_get
  // CHECK: nvws.aref.get.enter
  // CHECK: nvws.aref.get.exit
  tt.func @aref_get(%d : !ttg.memdesc<1x64x16xf16, #shared0, #smem>, %e : !ttg.memdesc<1x16x32xf16, #shared0, #smem>) {
    %c0_i32 = arith.constant {ttg.partition = array<i32: 0, 1>} 0 : i32
    %0 = nvws.aref.create %d, %e : !nvws.aref<[!ttg.memdesc<1x64x16xf16, #shared0, #smem>, !ttg.memdesc<1x16x32xf16, #shared0, #smem>]>
    %1:3 = nvws.aref.get.enter %0[%c0_i32, %c0_i32] : !nvws.aref<[!ttg.memdesc<1x64x16xf16, #shared0, #smem>, !ttg.memdesc<1x16x32xf16, #shared0, #smem>]> -> !ttg.memdesc<64x16xf16, #shared0, #smem>, !ttg.memdesc<16x32xf16, #shared0, #smem>, !ttg.async.token
    nvws.aref.get.exit %0[%c0_i32], %1#2 [#nvws.async_op<none>] : !nvws.aref<[!ttg.memdesc<1x64x16xf16, #shared0, #smem>, !ttg.memdesc<1x16x32xf16, #shared0, #smem>]>, !ttg.async.token
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.partition`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `aref_get`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.partition`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `aref_get`。

### Line 32
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 34-47
```mlir
#shared0 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "cuda:0", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: aref_put
  // CHECK: nvws.aref.put.enter
  // CHECK: nvws.aref.put.exit
  tt.func @aref_put(%d : !ttg.memdesc<1x64x16xf16, #shared0, #smem>, %e : !ttg.memdesc<1x16x32xf16, #shared0, #smem>) {
    %c0_i32 = arith.constant {ttg.partition = array<i32: 0, 1>} 0 : i32
    %0 = nvws.aref.create %d, %e : !nvws.aref<[!ttg.memdesc<1x64x16xf16, #shared0, #smem>, !ttg.memdesc<1x16x32xf16, #shared0, #smem>]>
    %1:3 = nvws.aref.put.enter %0[%c0_i32, %c0_i32] : !nvws.aref<[!ttg.memdesc<1x64x16xf16, #shared0, #smem>, !ttg.memdesc<1x16x32xf16, #shared0, #smem>]> -> !ttg.memdesc<64x16xf16, #shared0, #smem>, !ttg.memdesc<16x32xf16, #shared0, #smem>, !ttg.async.token
    nvws.aref.put.exit %0[%c0_i32], %1#2 [#nvws.async_op<tc5mma>] : !nvws.aref<[!ttg.memdesc<1x64x16xf16, #shared0, #smem>, !ttg.memdesc<1x16x32xf16, #shared0, #smem>]>, !ttg.async.token
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.partition`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `aref_put`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.partition`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `aref_put`。

### Line 49
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 52-57
```mlir
// CHECK-LABEL: @warp_group_nothing
tt.func @warp_group_nothing() {
  // CHECK-NEXT: nvws.warp_group
  nvws.warp_group
  tt.return
}
```
**EN:** This block defines `warp_group_nothing` and exercises operations such as `tt.func`, `nvws.warp_group`, `tt.return`. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warp_group_nothing`，并覆盖 如 `tt.func`、`nvws.warp_group`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 59-70
```mlir
// CHECK-LABEL: @warp_1_partition
tt.func @warp_1_partition() {
  // CHECK-NEXT: nvws.warp_group
  nvws.warp_group
  // CHECK-NEXT:  num_warps(4) {
  partition0  num_warps(4) {
  // CHECK-NEXT: nvws.warp_group.return
    nvws.warp_group.return
  // CHECK-NEXT: }
  }
  tt.return
}
```
**EN:** This block defines `warp_1_partition` and exercises operations such as `tt.func`, `nvws.warp_group`, `nvws.warp_group.return`, `tt.return`. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warp_1_partition`，并覆盖 如 `tt.func`、`nvws.warp_group`、`nvws.warp_group.return`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 72-89
```mlir
// CHECK-LABEL: @warp_2_partition
tt.func @warp_2_partition() {
  // CHECK-NEXT: nvws.warp_group
  nvws.warp_group
  // CHECK-NEXT: partition0  num_warps(8) {
  partition0  num_warps(8) {
  // CHECK-NEXT: nvws.warp_group.return
    nvws.warp_group.return
  // CHECK-NEXT: }
  }
  // CHECK-NEXT: partition1 num_warps(4) {
  partition1 num_warps(4) {
  // CHECK-NEXT:   nvws.warp_group.return
    nvws.warp_group.return
  // CHECK-NEXT: }
  }
  tt.return
}
```
**EN:** This block defines `warp_2_partition` and exercises operations such as `tt.func`, `nvws.warp_group`, `nvws.warp_group.return`, `tt.return`. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×7) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warp_2_partition`，并覆盖 如 `tt.func`、`nvws.warp_group`、`nvws.warp_group.return`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×7）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 91-92
```mlir
// CHECK-LABEL: @token_producer_consumer
tt.func @token_producer_consumer() {
```
**EN:** This block defines `token_producer_consumer` and exercises operations such as `tt.func`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `token_producer_consumer`，并覆盖 如 `tt.func` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 94-98
```mlir
  // CHECK: nvws.create_token
  // CHECK: nvws.producer_acquire
  // CHECK: nvws.producer_commit
  // CHECK: nvws.consumer_wait
  // CHECK: nvws.consumer_release
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Line 100
```mlir
  %0 = nvws.create_token {loadType = 1 : i32, numBuffers = 3 : i32} : tensor<3x!nvws.token>
```
**EN:** This block contributes intermediate IR built from `nvws.create_token`, `nvws.token`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `nvws.create_token`, `nvws.token` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 102-103
```mlir
  %c0_i32 = arith.constant {async_task_id = dense<0> : vector<1xi32>} 0 : i32
  %false = arith.constant {async_task_id = dense<0> : vector<1xi32>} false
```
**EN:** This block contributes intermediate IR built from `arith.constant`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `arith.constant` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 105-110
```mlir
  nvws.producer_acquire %0, %c0_i32, %false {async_task_id = dense<0> : vector<1xi32>} : tensor<3x!nvws.token>, i32, i1
  nvws.producer_commit %0, %c0_i32 {async_task_id = dense<0> : vector<1xi32>} : tensor<3x!nvws.token>, i32
  nvws.consumer_wait %0, %c0_i32, %false {async_task_id = dense<1> : vector<1xi32>} : tensor<3x!nvws.token>, i32, i1
  nvws.consumer_release %0, %c0_i32 {async_task_id = dense<1> : vector<1xi32>} : tensor<3x!nvws.token>, i32
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `nvws.producer_acquire`, `nvws.token`, `nvws.producer_commit`, `nvws.consumer_wait`, `nvws.consumer_release`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `nvws.producer_acquire`, `nvws.token`, `nvws.producer_commit`, `nvws.consumer_wait`, `nvws.consumer_release`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's NVWS transforms coverage and focuses on MMA lowering.  
  **CN:** 该文件属于 Triton 的 NVWS 变换 测试覆盖，关注点是 MMA 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--split-input-file`.  
  **CN:** `RUN` 流水线会驱动 `--split-input-file` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`。
