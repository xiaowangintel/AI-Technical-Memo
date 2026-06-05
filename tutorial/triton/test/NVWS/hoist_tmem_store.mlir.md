# hoist_tmem_store.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/NVWS/hoist_tmem_store.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises memory stores in Triton's NVWS transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton NVWS 变换中的内存写入相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allow-unregistered-dialect --nvws-hoist-tmem-store | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×5, CHECK-COUNT×1, CHECK-NEXT×4, CHECK-NOT×2; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×5, CHECK-COUNT×1, CHECK-NEXT×4, CHECK-NOT×2；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that memory stores produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 内存写入 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --allow-unregistered-dialect --nvws-hoist-tmem-store | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allow-unregistered-dialect --nvws-hoist-tmem-store | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allow-unregistered-dialect --nvws-hoist-tmem-store | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-60
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
#smem = #ttg.shared_memory
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @matmul_nested_persistent_ws_kernel(%arg0: !tt.tensordesc<128x128xf8E4M3FN, #shared>, %arg1: !tt.tensordesc<128x128xf8E4M3FN, #shared>, %arg2: !tt.tensordesc<128x128xf8E4M3FN, #shared>, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    %false = arith.constant false
    %true = arith.constant true
    %c128_i32 = arith.constant 128 : i32
    %c148_i32 = arith.constant 148 : i32
    %c0_i32 = arith.constant 0 : i32
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %0 = tt.get_program_id x : i32
    %1 = arith.divsi %arg3, %c128_i32 : i32
    %2 = arith.divsi %arg4, %c128_i32 : i32
    %3 = arith.divsi %arg5, %c128_i32 : i32
    %4 = arith.muli %1, %2 : i32
    %5 = arith.muli %2, %c8_i32 : i32
    // There is llvm.intr.assume on the inner-loop upper bound, the tmem store can be hoisted to the top level
    // CHECK: {{.*}}, [[TOKEN:%.*]] = ttng.tmem_alloc {{.*}} : (tensor<128x128xf32, #blocked>)
    // CHECK-NOT: tmem_store
    // CHECK: scf.for {{.*}}iter_args([[TOKEN_ARG:%.*]] = [[TOKEN]])
    scf.for %arg6 = %0 to %4 step %c148_i32  : i32 {
      %6 = arith.divsi %arg6, %5 {ttg.partition = array<i32: 0, 2>} : i32
      %7 = arith.muli %6, %c8_i32 {ttg.partition = array<i32: 0, 2>} : i32
      %8 = arith.subi %1, %7 {ttg.partition = array<i32: 0, 2>} : i32
      %9 = arith.minsi %8, %c8_i32 {ttg.partition = array<i32: 0, 2>} : i32
      %10 = arith.remsi %arg6, %9 {ttg.partition = array<i32: 0, 2>} : i32
      %11 = arith.addi %7, %10 {ttg.partition = array<i32: 0, 2>} : i32
      %12 = arith.remsi %arg6, %5 {ttg.partition = array<i32: 0, 2>} : i32
      %13 = arith.divsi %12, %9 {ttg.partition = array<i32: 0, 2>} : i32
      // CHECK-COUNT-3: arith.muli
      // CHECK-NEXT: arith.addi
      // CHECK-NEXT: arith.cmpi
      // CHECK-NEXT: llvm.intr.assume
      // CHECK-NEXT: scf.for {{.*}}iter_args({{.*}} = {{.*}}, {{.*}} = [[TOKEN_ARG]])
      %14 = arith.muli %11, %c128_i32 {ttg.partition = array<i32: 0, 2>} : i32
      %15 = arith.muli %13, %c128_i32 {ttg.partition = array<i32: 0, 2>} : i32
      %result, %token = ttng.tmem_alloc {ttg.partition = array<i32: 0, 1>} : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
      %16 = ttng.tmem_store %cst, %result[%token], %true {ttg.partition = array<i32: 0>} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      %17 = arith.addi %3, %arg6 {ttg.partition = array<i32: 1, 2>} : i32
      %18 = arith.cmpi sgt, %17, %c0_i32 {ttg.partition = array<i32: 1, 2>} : i32
      llvm.intr.assume %18 : i1 {ttg.partition = array<i32: 1, 2>}
      %19:2 = scf.for %arg7 = %c0_i32 to %17 step %c1_i32 iter_args(%arg8 = %false, %arg9 = %16) -> (i1, !ttg.async.token)  : i32 {
        %22 = arith.muli %arg7, %c128_i32 {loop.cluster = 2 : i32, loop.stage = 0 : i32, ttg.partition = array<i32: 2>} : i32
        %23 = tt.descriptor_load %arg0[%14, %22] {loop.cluster = 2 : i32, loop.stage = 0 : i32, ttg.partition = array<i32: 2>} : !tt.tensordesc<128x128xf8E4M3FN, #shared> -> tensor<128x128xf8E4M3FN, #blocked1>
        %24 = ttg.local_alloc %23 {loop.cluster = 0 : i32, loop.stage = 2 : i32, ttg.partition = array<i32: 2>} : (tensor<128x128xf8E4M3FN, #blocked1>) -> !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>
        %25 = tt.descriptor_load %arg1[%15, %22] {loop.cluster = 2 : i32, loop.stage = 0 : i32, ttg.partition = array<i32: 2>} : !tt.tensordesc<128x128xf8E4M3FN, #shared> -> tensor<128x128xf8E4M3FN, #blocked1>
        %26 = ttg.local_alloc %25 {loop.cluster = 0 : i32, loop.stage = 2 : i32, ttg.partition = array<i32: 2>} : (tensor<128x128xf8E4M3FN, #blocked1>) -> !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>
        %27 = ttg.memdesc_trans %26 {loop.cluster = 0 : i32, loop.stage = 2 : i32, order = array<i32: 1, 0>, ttg.partition = array<i32: 1>} : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem> -> !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem>
        %28 = ttng.tc_gen5_mma %24, %27, %result[%arg9], %arg8, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32, ttg.partition = array<i32: 1>} : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>, !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
        scf.yield {ttg.partition = array<i32: 1, 2>} %true, %28 : i1, !ttg.async.token
      } {tt.scheduled_max_stage = 2 : i32, ttg.partition = array<i32: 1, 2>, ttg.partition.outputs = [array<i32: 1, 2>, array<i32: 1>]}
    } {tt.num_stages = 3 : i32, tt.warp_specialize, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 0 : i32}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`, `ttg.partition`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `matmul_nested_persistent_ws_kernel`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`, `ttg.partition`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `matmul_nested_persistent_ws_kernel`。

### Lines 62-93
```mlir
    // There is no llvm.intr.assume in this case
    // CHECK: scf.for
    scf.for %arg6 = %0 to %4 step %c148_i32  : i32 {
      %6 = arith.divsi %arg6, %5 {ttg.partition = array<i32: 0, 2>} : i32
      %7 = arith.muli %6, %c8_i32 {ttg.partition = array<i32: 0, 2>} : i32
      %8 = arith.subi %1, %7 {ttg.partition = array<i32: 0, 2>} : i32
      %9 = arith.minsi %8, %c8_i32 {ttg.partition = array<i32: 0, 2>} : i32
      %10 = arith.remsi %arg6, %9 {ttg.partition = array<i32: 0, 2>} : i32
      %11 = arith.addi %7, %10 {ttg.partition = array<i32: 0, 2>} : i32
      %12 = arith.remsi %arg6, %5 {ttg.partition = array<i32: 0, 2>} : i32
      %13 = arith.divsi %12, %9 {ttg.partition = array<i32: 0, 2>} : i32
      %14 = arith.muli %11, %c128_i32 {ttg.partition = array<i32: 0, 2>} : i32
      %15 = arith.muli %13, %c128_i32 {ttg.partition = array<i32: 0, 2>} : i32
      // CHECK: {{.*}}, [[TOKEN:%.*]] = ttng.tmem_alloc {{.*}} {ttg.partition = array<i32: 1>}
      // CHECK-NOT: tmem_store
      // CHECK: scf.for {{.*}}iter_args({{.*}} = {{.*}}, {{.*}} = [[TOKEN]])
      %result, %token = ttng.tmem_alloc {ttg.partition = array<i32: 0, 1>} : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
      %16 = ttng.tmem_store %cst, %result[%token], %true {ttg.partition = array<i32: 0>} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      %17 = arith.addi %3, %arg6 {ttg.partition = array<i32: 1, 2>} : i32
      %19:2 = scf.for %arg7 = %c0_i32 to %17 step %c1_i32 iter_args(%arg8 = %false, %arg9 = %16) -> (i1, !ttg.async.token)  : i32 {
        %22 = arith.muli %arg7, %c128_i32 {loop.cluster = 2 : i32, loop.stage = 0 : i32, ttg.partition = array<i32: 2>} : i32
        %23 = tt.descriptor_load %arg0[%14, %22] {loop.cluster = 2 : i32, loop.stage = 0 : i32, ttg.partition = array<i32: 2>} : !tt.tensordesc<128x128xf8E4M3FN, #shared> -> tensor<128x128xf8E4M3FN, #blocked1>
        %24 = ttg.local_alloc %23 {loop.cluster = 0 : i32, loop.stage = 2 : i32, ttg.partition = array<i32: 2>} : (tensor<128x128xf8E4M3FN, #blocked1>) -> !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>
        %25 = tt.descriptor_load %arg1[%15, %22] {loop.cluster = 2 : i32, loop.stage = 0 : i32, ttg.partition = array<i32: 2>} : !tt.tensordesc<128x128xf8E4M3FN, #shared> -> tensor<128x128xf8E4M3FN, #blocked1>
        %26 = ttg.local_alloc %25 {loop.cluster = 0 : i32, loop.stage = 2 : i32, ttg.partition = array<i32: 2>} : (tensor<128x128xf8E4M3FN, #blocked1>) -> !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>
        %27 = ttg.memdesc_trans %26 {loop.cluster = 0 : i32, loop.stage = 2 : i32, order = array<i32: 1, 0>, ttg.partition = array<i32: 1>} : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem> -> !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem>
        %28 = ttng.tc_gen5_mma %24, %27, %result[%arg9], %arg8, %true {loop.cluster = 0 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32, ttg.partition = array<i32: 1>} : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem>, !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
        scf.yield {ttg.partition = array<i32: 1, 2>} %true, %28 : i1, !ttg.async.token
      } {tt.scheduled_max_stage = 2 : i32, ttg.partition = array<i32: 1, 2>, ttg.partition.outputs = [array<i32: 1, 2>, array<i32: 1>]}
    } {tt.num_stages = 3 : i32, tt.warp_specialize, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 0 : i32}
    tt.return
  }
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×3, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×3, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 95
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's NVWS transforms coverage and focuses on memory stores.  
  **CN:** 该文件属于 Triton 的 NVWS 变换 测试覆盖，关注点是 内存写入。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allow-unregistered-dialect`, `--nvws-hoist-tmem-store`.  
  **CN:** `RUN` 流水线会驱动 `--allow-unregistered-dialect`, `--nvws-hoist-tmem-store` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `llvm` (LLVM dialect IR), `ttng` (next-gen TritonGPU ops), `scf` (structured control flow).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`llvm`（LLVM 方言 IR）、`ttng`（新一代 TritonGPU 操作）、`scf`（结构化控制流）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `llvm`, `ttng`, `scf`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`llvm`、`ttng`、`scf`。
