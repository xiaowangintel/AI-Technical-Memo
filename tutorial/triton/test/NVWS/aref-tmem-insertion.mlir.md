# aref-tmem-insertion.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/NVWS/aref-tmem-insertion.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises IR behavior under test in Triton's NVWS transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton NVWS 变换中的待测试的 IR 行为相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file -allow-unregistered-dialect -nvws-insert-tmem-aref -cse | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×91, CHECK-LABEL×20, CHECK-NEXT×236, CHECK-NOT×1; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×91, CHECK-LABEL×20, CHECK-NEXT×236, CHECK-NOT×1；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that IR behavior under test produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 待测试的 IR 行为 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file -allow-unregistered-dialect -nvws-insert-tmem-aref -cse | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file -allow-unregistered-dialect -nvws-insert-tmem-aref -cse | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file -allow-unregistered-dialect -nvws-insert-tmem-aref -cse | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-18
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0], [0, 4]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
#linear1 = #ttg.linear<{register = [[1, 0], [2, 0], [0, 32], [0, 64], [4, 0]], lane = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], warp = [[0, 0], [0, 0]], block = []}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
#shared2 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
#shared3 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [4, 3, 2, 1, 0]}>
#shared4 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 8}>
#shared5 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8, fp4Padded = true, rank = 3}>
#smem = #ttg.shared_memory
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
#tmem1 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
#tmem_scales = #ttng.tensor_memory_scales_encoding<>
module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 20-21
```mlir
  // CHECK-LABEL: @warp_specialize_tma_matmul
  tt.func @warp_specialize_tma_matmul(%arg0: i32, %arg1: i32, %arg2: i32, %arg3: !tt.tensordesc<128x64xf16, #shared>, %arg4: !tt.tensordesc<128x64xf16, #shared>) {
```
**EN:** This block defines `warp_specialize_tma_matmul` and exercises operations such as `tt.func`, `tt.tensordesc`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warp_specialize_tma_matmul`，并覆盖 如 `tt.func`、`tt.tensordesc` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 23-56
```mlir
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %c64_i32 = arith.constant 64 : i32
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    %true = arith.constant true
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    // CHECK: [[ABUF:%.*]] = ttng.tmem_alloc : () -> !ttg.memdesc<1x128x128xf32,
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create [[ABUF]]
    // CHECK-NEXT: {{.*}}, [[ATOK:%.*]] = nvws.aref.put.enter [[AREF]]
    // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[ATOK]]
    // CHECK-NEXT: tmem_store {{.*}}, [[BUF]]
    %0 = ttng.tmem_store %cst, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK-NEXT: [[TOK2:%.*]] = scf.for {{.*}} iter_args([[TOK:%.*]] = [[ATOK]])
    %1 = scf.for %arg5 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg6 = %0) -> (!ttg.async.token)  : i32 {
      %2 = arith.muli %arg5, %c64_i32 {ttg.partition = array<i32: 2>} : i32
      %3 = tt.descriptor_load %arg3[%arg1, %2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %4 = tt.descriptor_load %arg4[%arg2, %2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %5 = ttg.local_alloc %3 {ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %6 = ttg.local_alloc %4 {ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %7 = ttg.memdesc_trans %6 {order = array<i32: 1, 0>, ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem> -> !ttg.memdesc<64x128xf16, #shared1, #smem>
      // CHECK: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[TOK]]
      // CHECK-NEXT: tc_gen5_mma {{.*}}, {{.*}}, [[BUF]]
      %8 = ttng.tc_gen5_mma %5, %7, %result[%arg6], %true, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      scf.yield {ttg.partition = array<i32: 0, 1, 2>} %8 : !ttg.async.token
    } {tt.num_stages = 2 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>]}
    // CHECK: nvws.aref.put.exit [[AREF]], [[TOK2]] [#nvws.async_op<tc5mma>]
    // CHECK-NEXT: {{.*}}, [[ATOK:%.*]] = nvws.aref.get.enter [[AREF]]
    // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[ATOK]]
    // CHECK-NEXT: tmem_load [[BUF]]
    // CHECK-NEXT: nvws.aref.get.exit [[AREF]], [[ATOK]] [#nvws.async_op<none>]
    %result_0, %token_1 = ttng.tmem_load %result[%1] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
    "use"(%result_0) : (tensor<128x128xf32, #blocked>) -> ()
    tt.return
  }
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×3, CHECK-NEXT×10) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×3, CHECK-NEXT×10）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 58-83
```mlir
// CHECK-LABEL: @matmul_tma_acc_with_unconditional_user
  tt.func @matmul_tma_acc_with_unconditional_user(%arg0: !tt.tensordesc<128x64xf16, #shared>, %arg1: !tt.tensordesc<64x128xf16, #shared>) {
    %c32_i32 = arith.constant 32 : i32
    %cst = arith.constant dense<1.000000e+00> : tensor<128x128xf32, #blocked>
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %true = arith.constant true
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    // CHECK: [[ABUF:%.*]] = ttng.tmem_alloc : () -> !ttg.memdesc<2x128x128xf32,
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create [[ABUF]]
    // CHECK-NEXT: {{.*}}, [[ATOK:%.*]] = nvws.aref.put.enter [[AREF]]
    // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[ATOK]]
    // CHECK-NEXT: tmem_store {{.*}}, [[BUF]]
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %0 = ttng.tmem_store %cst_0, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK: [[TOK1:%.]] = scf.for [[I:%.*]] = [[UB:%.*]] to [[LB:%.*]] step [[STEP:%.*]] iter_args([[TOK:%.*]] = [[ATOK]])
    %1 = scf.for %arg2 = %c0_i32 to %c32_i32 step %c1_i32 iter_args(%arg3 = %0) -> (!ttg.async.token)  : i32 {
      %2:3 = "get_offsets"(%arg2) {ttg.partition = array<i32: 2>} : (i32) -> (i32, i32, i32)
      %3 = tt.descriptor_load %arg0[%2#0, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %4 = tt.descriptor_load %arg1[%2#1, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<64x128xf16, #shared> -> tensor<64x128xf16, #blocked1>
      %5 = ttg.local_alloc %3 {ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %6 = ttg.local_alloc %4 {ttg.partition = array<i32: 2>} : (tensor<64x128xf16, #blocked1>) -> !ttg.memdesc<64x128xf16, #shared, #smem>
      // CHECK: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[TOK]] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: ttng.tc_gen5_mma {{.*}}, {{.*}}, [[BUF]]
      // CHECK-NEXT: nvws.aref.put.exit [[AREF]], [[TOK]] [#nvws.async_op<tc5mma>] {ttg.partition = array<i32: 1>}
      %7 = ttng.tc_gen5_mma %5, %6, %result[%arg3], %true, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This block defines `matmul_tma_acc_with_unconditional_user` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×6) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `matmul_tma_acc_with_unconditional_user`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×6）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 85-89
```mlir
      // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.get.enter [[AREF]] {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[TOK]]
      // CHECK-NEXT: tmem_load [[BUF]]
      // CHECK-NEXT: nvws.aref.get.exit [[AREF]], [[TOK]] [#nvws.async_op<none>] {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: "acc_user"
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 91-102
```mlir
      // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.put.enter [[AREF]] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[TOK]]
      // CHECK-NEXT: tmem_store {{.*}}, [[BUF]]
      // CHECK-NEXT: yield {ttg.partition = array<i32: 0, 1, 2>} [[TOK]]
      %result_1, %token_2 = ttng.tmem_load %result[%7] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
      "acc_user"(%result_1) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
      %8 = ttng.tmem_store %cst, %result[%token_2], %true {ttg.partition = array<i32: 1>} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      scf.yield {ttg.partition = array<i32: 0, 1, 2>} %8 : !ttg.async.token
    } {tt.num_stages = 2 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 4 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>]}
    // CHECK: nvws.aref.put.exit [[AREF]], [[TOK1]] [#nvws.async_op<none>]
    tt.return
  }
```
**EN:** This block defines the test function(s) and exercises operations such as `nvws.aref.put.enter`, `ttg.partition`, `nvws.aref.buffer`, `ttng.tmem_load`, `ttg.memdesc`. Embedded check comments (CHECK×1, CHECK-NEXT×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `nvws.aref.put.enter`、`ttg.partition`、`nvws.aref.buffer`、`ttng.tmem_load`、`ttg.memdesc` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 104-111
```mlir
  // CHECK-LABEL: @matmul_tma_acc_with_conditional_user
  tt.func @matmul_tma_acc_with_conditional_user(%arg0: !tt.tensordesc<128x64xf16, #shared>, %arg1: !tt.tensordesc<64x128xf16, #shared>) {
    %c32_i32 = arith.constant 32 : i32
    %cst = arith.constant dense<1.000000e+00> : tensor<128x128xf32, #blocked>
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %true = arith.constant true
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
```
**EN:** This block defines `matmul_tma_acc_with_conditional_user` and exercises operations such as `tt.func`, `tt.tensordesc`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `matmul_tma_acc_with_conditional_user`，并覆盖 如 `tt.func`、`tt.tensordesc`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 113-137
```mlir
    // CHECK: [[ABUF:%.*]] = ttng.tmem_alloc : () -> !ttg.memdesc<2x128x128xf32,
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create [[ABUF]]
    // CHECK-NEXT: {{.*}}, [[ATOK:%.*]] = nvws.aref.put.enter [[AREF]]
    // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[ATOK]]
    // CHECK-NEXT: tmem_store {{.*}}, [[BUF]]
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %0 = ttng.tmem_store %cst_0, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK: [[TOK2:%.*]] = scf.for [[I:%.*]] = [[UB:%.*]] to [[LB:%.*]] step [[STEP:%.*]] iter_args([[TOK:%.*]] = [[ATOK]])
    %1 = scf.for %arg2 = %c0_i32 to %c32_i32 step %c1_i32 iter_args(%arg3 = %0) -> (!ttg.async.token)  : i32 {
      %2:3 = "get_offsets"(%arg2) {ttg.partition = array<i32: 2>} : (i32) -> (i32, i32, i32)
      %3 = tt.descriptor_load %arg0[%2#0, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %4 = tt.descriptor_load %arg1[%2#1, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<64x128xf16, #shared> -> tensor<64x128xf16, #blocked1>
      %5 = ttg.local_alloc %3 {ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %6 = ttg.local_alloc %4 {ttg.partition = array<i32: 2>} : (tensor<64x128xf16, #blocked1>) -> !ttg.memdesc<64x128xf16, #shared, #smem>
      %7 = ttng.tc_gen5_mma %5, %6, %result[%arg3], %true, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      %8 = arith.cmpi eq, %arg2, %c0_i32 {ttg.partition = array<i32: 0, 1>}: i32
      // CHECK: scf.if
      %9 = scf.if %8 -> (!ttg.async.token) {
        // CHECK-NEXT:  nvws.aref.put.exit [[AREF]], [[TOK]] [#nvws.async_op<tc5mma>] {ttg.partition = array<i32: 1>}
      // CHECK: scf.if
        // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.get.enter [[AREF]] {ttg.partition = array<i32: 0>}
        // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[TOK]]
        // CHECK-NEXT: tmem_load [[BUF]]
        // CHECK-NEXT: nvws.aref.get.exit [[AREF]], [[TOK]] [#nvws.async_op<none>] {ttg.partition = array<i32: 0>}
        // CHECK-NEXT: "acc_user"
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation, loop-carried state, control-flow joins. Embedded check comments (CHECK×4, CHECK-NEXT×10) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配、循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×4, CHECK-NEXT×10）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 139-157
```mlir
      // CHECK: [[TOK1:%.*]] = scf.if
        // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.put.enter [[AREF]] {ttg.partition = array<i32: 1>}
        // CHECK-NEXT: yield {ttg.partition = array<i32: 1>} [[TOK]]
        %result_1, %token_2 = ttng.tmem_load %result[%7] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
        "acc_user"(%result_1) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
        scf.yield %token_2 : !ttg.async.token
      } else {
        scf.yield %7 : !ttg.async.token
      } {ttg.partition = array<i32: 0, 1>, ttg.partition.outputs = [array<i32: 1>]}
      // CHECK: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[TOK1]]
      // CHECK-NEXT: tmem_store {{.*}}, [[BUF]]
      %10 = ttng.tmem_store %cst, %result[%9], %true {ttg.partition = array<i32: 1>} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      scf.yield %10 : !ttg.async.token
    } {tt.num_stages = 2 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 5 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs= [array<i32: 1>]}
    // CHECK: nvws.aref.put.exit [[AREF]], [[TOK2]] [#nvws.async_op<none>] {ttg.partition = array<i32: 1>, ttg.warp_specialize.tag = 5 : i32}
    // CHECK-NEXT: [[BUF:%.*]], [[TOK:%.*]] = nvws.aref.get.enter [[AREF]] {ttg.partition = array<i32: 0>, ttg.warp_specialize.tag = 5 : i32}
    // CHECK-NEXT: nvws.aref.get.exit [[AREF]], [[TOK]] [#nvws.async_op<none>] {ttg.partition = array<i32: 0>, ttg.warp_specialize.tag = 5 : i32}
    tt.return
  }
```
**EN:** This block defines the test function(s) and exercises control-flow joins. Embedded check comments (CHECK×3, CHECK-NEXT×5) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 控制流汇合。 其中嵌入的检查注释（CHECK×3, CHECK-NEXT×5）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 159-192
```mlir
  // CHECK-LABEL: @matmul_tma_acc_with_conditional_def
  tt.func @matmul_tma_acc_with_conditional_def(%arg0: !tt.tensordesc<128x64xf16, #shared>, %arg1: !tt.tensordesc<64x128xf16, #shared>) {
    %c32_i32 = arith.constant 32 : i32
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %true = arith.constant true
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    // CHECK: [[AREF:%.*]] = nvws.aref.create {{.*}}
    // CHECK-NEXT: {{.*}}, [[ATOK:%.*]] = nvws.aref.put.enter [[AREF]]
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %0 = ttng.tmem_store %cst, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %1 = scf.for %arg2 = %c0_i32 to %c32_i32 step %c1_i32 iter_args(%arg3 = %0) -> (!ttg.async.token)  : i32 {
      %2:3 = "get_offsets"(%arg2) : (i32) -> (i32, i32, i32)
      %3 = tt.descriptor_load %arg0[%2#0, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %4 = tt.descriptor_load %arg1[%2#1, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<64x128xf16, #shared> -> tensor<64x128xf16, #blocked1>
      %5 = ttg.local_alloc %3 {ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %6 = ttg.local_alloc %4 {ttg.partition = array<i32: 2>} : (tensor<64x128xf16, #blocked1>) -> !ttg.memdesc<64x128xf16, #shared, #smem>
      %7 = ttng.tc_gen5_mma %5, %6, %result[%arg3], %true, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      // CHECK: tc_gen5_mma
      // CHECK-NEXT: nvws.aref.put.exit
      // CHECK: nvws.aref.get.enter
      // CHECK-NEXT: nvws.aref.buffer
      // CHECK-NEXT: tmem_load
      // CHECK-NEXT: nvws.aref.get.exit
      // CHECK-NEXT: acc_user
      %8 = arith.cmpi eq, %arg2, %c0_i32 : i32
      %result_0, %token_1 = ttng.tmem_load %result[%7] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
      "acc_user"(%result_0) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
      // CHECK-NEXT: nvws.aref.put.enter
      %9 = ttng.tmem_store %cst, %result[%token_1], %8 {ttg.partition = array<i32: 1>} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      scf.yield %9 : !ttg.async.token
    } {tt.num_stages = 2 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 6 : i32}
    tt.return
  }
```
**EN:** This block defines `matmul_tma_acc_with_conditional_def` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×7) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `matmul_tma_acc_with_conditional_def`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×7）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 194-239
```mlir
  // CHECK-LABEL: @matmul_tma_acc_with_conditional_def_and_use
  tt.func @matmul_tma_acc_with_conditional_def_and_use(%arg0: !tt.tensordesc<128x64xf16, #shared>, %arg1: !tt.tensordesc<64x128xf16, #shared>) {
    %c32_i32 = arith.constant 32 : i32
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %true = arith.constant true
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    // CHECK: [[AREF:%.*]] = nvws.aref.create
    // CHECK-NEXT: {{.*}}, [[ATOK:%.*]] = nvws.aref.put.enter [[AREF]]
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %0 = ttng.tmem_store %cst, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK: [[TOK2:%.*]] = scf.for [[I:%.*]] = [[UB:%.*]] to [[LB:%.*]] step [[STEP:%.*]] iter_args([[TOK:%.*]] = [[ATOK]])
    %1 = scf.for %arg2 = %c0_i32 to %c32_i32 step %c1_i32 iter_args(%arg3 = %0) -> (!ttg.async.token)  : i32 {
      %2:3 = "get_offsets"(%arg2) {ttg.partition = array<i32: 2>} : (i32) -> (i32, i32, i32)
      %3 = tt.descriptor_load %arg0[%2#0, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %4 = tt.descriptor_load %arg1[%2#1, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<64x128xf16, #shared> -> tensor<64x128xf16, #blocked1>
      %5 = ttg.local_alloc %3 {ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %6 = ttg.local_alloc %4 {ttg.partition = array<i32: 2>} : (tensor<64x128xf16, #blocked1>) -> !ttg.memdesc<64x128xf16, #shared, #smem>
      %7 = ttng.tc_gen5_mma %5, %6, %result[%arg3], %true, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      %8 = arith.cmpi eq, %arg2, %c0_i32 {ttg.partition = array<i32: 0, 1>}: i32
      // CHECK: scf.if
      %9 = scf.if %8 -> (!ttg.async.token) {
        // CHECK-NEXT: nvws.aref.put.exit [[AREF]], [[TOK]]
      //CHECK: scf.if
        // CHECK-NEXT: nvws.aref.get.enter
        // CHECK-NEXT: nvws.aref.buffer
        // CHECK-NEXT: tmem_load
        // CHECK-NEXT: nvws.aref.get.exit
        // CHECK-NEXT: acc_user
      // CHECK: [[TOK1:%.*]] = scf.if
        // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.put.enter [[AREF]]
        // CHECK-NEXT: yield {ttg.partition = array<i32: 1>} [[TOK]]
        %result_0, %token_1 = ttng.tmem_load %result[%7] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
        "acc_user"(%result_0) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
        scf.yield %token_1 : !ttg.async.token
      } else {
        scf.yield %7 : !ttg.async.token
      } {ttg.partition = array<i32: 0, 1>, ttg.partition.outputs = [array<i32: 1>]}
      // CHECK: nvws.aref.buffer [[AREF]], [[TOK1]]
      // CHECK-NEXT: tmem_store
      // CHECK-NEXT: scf.yield [[TOK1]]
      %10 = ttng.tmem_store %cst, %result[%9], %8 {ttg.partition = array<i32: 1>} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      scf.yield %10 : !ttg.async.token
    } {tt.num_stages = 2 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 7 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>]}
    tt.return
  }
```
**EN:** This block defines `matmul_tma_acc_with_conditional_def_and_use` and exercises shared-memory allocation, loop-carried state, control-flow joins. Embedded check comments (CHECK×6, CHECK-LABEL×1, CHECK-NEXT×11) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `matmul_tma_acc_with_conditional_def_and_use`，并覆盖 共享内存分配、循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×6, CHECK-LABEL×1, CHECK-NEXT×11）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 241-271
```mlir
  // CHECK-LABEL: @matmul_tma_acc_with_conditional_def_and_use_no_multibuf_flag
  tt.func @matmul_tma_acc_with_conditional_def_and_use_no_multibuf_flag(%arg0: !tt.tensordesc<128x64xf16, #shared>, %arg1: !tt.tensordesc<64x128xf16, #shared>) {
    %c32_i32 = arith.constant 32 : i32
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %true = arith.constant true
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    // CHECK: ttng.tmem_alloc : () -> !ttg.memdesc<1x128x128xf32
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %0 = ttng.tmem_store %cst, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %1:2 = scf.for %arg2 = %c0_i32 to %c32_i32 step %c1_i32 iter_args(%arg3 = %true, %arg4 = %0) -> (i1, !ttg.async.token)  : i32 {
      %2:3 = "get_offsets"(%arg2) {ttg.partition = array<i32: 2>} : (i32) -> (i32, i32, i32)
      %3 = tt.descriptor_load %arg0[%2#0, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %4 = tt.descriptor_load %arg1[%2#1, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<64x128xf16, #shared> -> tensor<64x128xf16, #blocked1>
      %5 = ttg.local_alloc %3 {ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %6 = ttg.local_alloc %4 {ttg.partition = array<i32: 2>} : (tensor<64x128xf16, #blocked1>) -> !ttg.memdesc<64x128xf16, #shared, #smem>
      %7 = ttng.tc_gen5_mma %5, %6, %result[%arg4], %arg3, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      %8 = arith.cmpi eq, %arg2, %c0_i32 {ttg.partition = array<i32: 0, 1>}: i32
      %9 = arith.cmpi ne, %arg2, %c0_i32 {ttg.partition = array<i32: 1>} : i32
      %10 = scf.if %8 -> (!ttg.async.token) {
        "some_op"() {ttg.partition = array<i32: 0>} : () -> ()
        %result_0, %token_1 = ttng.tmem_load %result[%7] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
        "acc_user"(%result_0) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
        scf.yield %token_1 : !ttg.async.token
      } else {
        scf.yield %7 : !ttg.async.token
      } {ttg.partition = array<i32: 0, 1>, ttg.partition.outputs = [array<i32: 1>]}
      scf.yield %9, %10 : i1, !ttg.async.token
    } {ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>, array<i32: 1>], tt.disallow_acc_multi_buffer, tt.num_stages = 2 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 8 : i32}
    tt.return
  }
```
**EN:** This block defines `matmul_tma_acc_with_conditional_def_and_use_no_multibuf_flag` and exercises shared-memory allocation, loop-carried state, control-flow joins. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `matmul_tma_acc_with_conditional_def_and_use_no_multibuf_flag`，并覆盖 共享内存分配、循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 273-284
```mlir
  // CHECK-LABEL: @matmul_scaled_rhs_scales_tma
  tt.func @matmul_scaled_rhs_scales_tma(%arg0: i32, %arg1: i32, %arg2: i32, %arg3: !tt.tensordesc<128x64xf8E4M3FN, #shared2>, %arg4: !tt.tensordesc<128x64xf8E4M3FN, #shared2>, %arg5: !tt.tensordesc<128x8xi8, #shared3>) {
    // CHECK: [[CST:%.*]] = arith.constant dense<127> : tensor<128x8xi8
    // CHECK: [[CST_0:%.*]] = arith.constant dense<{{.*}}> : tensor<128x128xf32
    %cst = arith.constant dense<127> : tensor<128x8xi8, #linear>
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %c64_i32 = arith.constant 64 : i32
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    %true = arith.constant true
    // CHECK: [[LHS_SCALES_BUF:%.*]] = ttng.tmem_alloc [[CST]] : (tensor<128x8xi8, #linear>) -> !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
    %result = ttng.tmem_alloc %cst : (tensor<128x8xi8, #linear>) -> !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
```
**EN:** This block defines `matmul_scaled_rhs_scales_tma` and exercises operations such as `tt.func`, `tt.tensordesc`, `arith.constant`, `ttng.tmem_alloc`, `ttg.memdesc`. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `matmul_scaled_rhs_scales_tma`，并覆盖 如 `tt.func`、`tt.tensordesc`、`arith.constant`、`ttng.tmem_alloc`、`ttg.memdesc` 这样的操作。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 286-292
```mlir
    // CHECK-NEXT: [[ABUF:%.*]] = ttng.tmem_alloc
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create [[ABUF]]
    // CHECK-NEXT: {{.*}}, [[ATOK:%.*]] = nvws.aref.put.enter [[AREF]]
    // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[ATOK]]
    // CHECK-NEXT: tmem_store [[CST_0]], [[BUF]]
    %result_1, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %0 = ttng.tmem_store %cst_0, %result_1[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This block defines the test function(s) and exercises operations such as `ttng.tmem_alloc`, `nvws.aref.create`, `nvws.aref.put.enter`, `nvws.aref.buffer`, `ttg.memdesc`. Embedded check comments (CHECK-NEXT×5) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `ttng.tmem_alloc`、`nvws.aref.create`、`nvws.aref.put.enter`、`nvws.aref.buffer`、`ttg.memdesc` 这样的操作。 其中嵌入的检查注释（CHECK-NEXT×5）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 294-315
```mlir
    // CHECK-NEXT: [[RHS_SCALES_BUF:%.*]] = ttng.tmem_alloc : () -> !ttg.memdesc<128x8xi8,
    // CHECK-NEXT: [[RHS_SCALES_AREF:%.*]] = nvws.aref.create [[RHS_SCALES_BUF]]
    // CHECK-NEXT: [[TOK1:%.*]] = scf.for {{.*}} iter_args([[TOK:%.*]] = [[ATOK]])
    %1 = scf.for %arg6 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg7 = %0) -> (!ttg.async.token)  : i32 {
      // CHECK: [[LHS:%.]] = tt.descriptor_load
      // CHECK-NEXT: [[RHS:%.*]] = tt.descriptor_load
      // CHECK-NEXT: [[RHS_SCALES:%.*]] = tt.descriptor_load
      // CHECK-NEXT: local_alloc [[LHS]]
      // CHECK-NEXT: local_alloc [[RHS]]
      %2 = arith.muli %arg6, %c64_i32 {ttg.partition = array<i32: 2>} : i32
      %3 = tt.descriptor_load %arg3[%arg1, %2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf8E4M3FN, #shared2> -> tensor<128x64xf8E4M3FN, #blocked1>
      %4 = tt.descriptor_load %arg4[%arg2, %2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf8E4M3FN, #shared2> -> tensor<128x64xf8E4M3FN, #blocked1>
      %5 = tt.descriptor_load %arg5[%arg1, %c0_i32] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x8xi8, #shared3> -> tensor<128x8xi8, #linear>
      %6 = ttg.local_alloc %3 {ttg.partition = array<i32: 2>} : (tensor<128x64xf8E4M3FN, #blocked1>) -> !ttg.memdesc<128x64xf8E4M3FN, #shared2, #smem>
      %7 = ttg.local_alloc %4 {ttg.partition = array<i32: 2>} : (tensor<128x64xf8E4M3FN, #blocked1>) -> !ttg.memdesc<128x64xf8E4M3FN, #shared2, #smem>
      %8 = ttg.memdesc_trans %7 {order = array<i32: 1, 0>, ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf8E4M3FN, #shared2, #smem> -> !ttg.memdesc<64x128xf8E4M3FN, #shared4, #smem>
      // CHECK: {{.*}}, [[RHS_SCALES_TOK:%.*]] = nvws.aref.put.enter [[RHS_SCALES_AREF]]
      // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer [[RHS_SCALES_AREF]], [[RHS_SCALES_TOK]]
      // CHECK-NEXT: arith.constant {ttg.partition = array<i32: 2>}
      // CHECK-NEXT: tmem_store [[RHS_SCALES]], [[BUF]]
      // CHECK-NEXT: nvws.aref.put.exit [[RHS_SCALES_AREF]], [[RHS_SCALES_TOK]] [#nvws.async_op<none>]
      %result_2 = ttng.tmem_alloc %5 {ttg.partition = array<i32: 2>} : (tensor<128x8xi8, #linear>) -> !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×2, CHECK-NEXT×11) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×2, CHECK-NEXT×11）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 317-334
```mlir
      // CHECK-NEXT: [[BUF_ACC:%.*]] = nvws.aref.buffer [[AREF]], [[TOK]]
      // CHECK-NEXT: {{.*}}, [[RHS_TOK:%.*]] = nvws.aref.get.enter [[RHS_SCALES_AREF]]
      // CHECK-NEXT: [[RHS_SCALES_BUF:%.*]] = nvws.aref.buffer [[RHS_SCALES_AREF]], [[RHS_TOK]]
      // CHECK-NEXT: tc_gen5_mma_scaled {{.*}}, {{.*}}, [[BUF_ACC]][], [[LHS_SCALES_BUF]], [[RHS_SCALES_BUF]]
      // CHECK-NEXT: nvws.aref.get.exit [[RHS_SCALES_AREF]], [[RHS_TOK]] [#nvws.async_op<tc5mma>]
      %9 = ttng.tc_gen5_mma_scaled %6, %8, %result_1[%arg7], %result, %result_2, %true, %true lhs = e4m3 rhs = e4m3 {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf8E4M3FN, #shared2, #smem>, !ttg.memdesc<64x128xf8E4M3FN, #shared4, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>, !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
      scf.yield {ttg.partition = array<i32: 0, 1, 2>} %9 : !ttg.async.token
    } {tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 9 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>]}
    %val, %tok = ttng.tmem_load %result_1[%1] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
    "use"(%val) : (tensor<128x128xf32, #blocked>) -> ()
    // CHECK: nvws.aref.put.exit [[AREF]], [[TOK1]] [#nvws.async_op<tc5mma>]
    // CHECK-NEXT: aref.get.enter
    // CHECK-NEXT: aref.buffer
    // CHECK-NEXT: tmem_load
    // CHECK-NEXT: aref.get.exit
    // CHECK-NEXT: use
    tt.return
  }
```
**EN:** This block defines the test function(s) and exercises operations such as `nvws.aref.buffer`, `nvws.aref.get.enter`, `nvws.aref.get.exit`, `ttng.tc_gen5_mma_scaled`, `ttg.partition`. Embedded check comments (CHECK×1, CHECK-NEXT×10) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `nvws.aref.buffer`、`nvws.aref.get.enter`、`nvws.aref.get.exit`、`ttng.tc_gen5_mma_scaled`、`ttg.partition` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×10）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 336-375
```mlir
  // CHECK-LABEL: @user_partition_has_cycle
  tt.func @user_partition_has_cycle(%arg0: i32, %arg1: i32, %arg2: i32, %arg3: !tt.tensordesc<128x64xf16, #shared>, %arg4: !tt.tensordesc<128x64xf16, #shared>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %c64_i32 = arith.constant 64 : i32
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    %false = arith.constant false
    %true = arith.constant true
    %0 = tt.descriptor_load %arg3[%c0_i32, %c0_i32] : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
    // CHECK: [[BUF:%.*]] = ttng.tmem_alloc
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create [[BUF]]
    // CHECK-NEXT: {{.*}}, [[ATOK:%.*]] = nvws.aref.put.enter [[AREF]] :
    // CHECK-NEXT: scf.for {{.*}} iter_args({{.*}}, [[TOK:%.*]] = [[ATOK]])
    %1 = ttg.local_alloc %0 : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %2:2 = scf.for %arg5 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg6 = %cst, %arg7 = %token) -> (tensor<128x128xf32, #blocked>, !ttg.async.token)  : i32 {
      %3 = arith.muli %arg5, %c64_i32 {ttg.partition = array<i32: 2>} : i32
      %4 = tt.descriptor_load %arg4[%arg2, %3] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %5 = ttg.local_alloc %4 {ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %6 = ttg.memdesc_trans %5 {order = array<i32: 1, 0>, ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem> -> !ttg.memdesc<64x128xf16, #shared1, #smem>
      // CHECK: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[TOK]] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: tc_gen5_mma {{.*}} [[BUF]]
      // CHECK-NEXT: nvws.aref.put.exit [[AREF]], [[TOK]] [#nvws.async_op<tc5mma>] {ttg.partition = array<i32: 1>}
      %7 = ttng.tc_gen5_mma %1, %6, %result[%arg7], %false, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      // CHECK-NEXT: arith.addf
      %8 = arith.addf %arg6, %arg6 {ttg.partition = array<i32: 0>} : tensor<128x128xf32, #blocked>
      // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.get.enter [[AREF]] {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[TOK]] {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: tmem_load [[BUF]][]
      // CHECK-NEXT: nvws.aref.get.exit [[AREF]], [[TOK]] [#nvws.async_op<none>] {ttg.partition = array<i32: 0>}
      %result_0, %token_1 = ttng.tmem_load %result[%7] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
      // CHECK-NEXT: arith.mulf
      %9 = arith.mulf %8, %result_0 {ttg.partition = array<i32: 0>} : tensor<128x128xf32, #blocked>
      // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.put.enter [[AREF]] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: scf.yield {{.*}}, [[TOK]]
      scf.yield %9, %token_1 : tensor<128x128xf32, #blocked>, !ttg.async.token
    } {tt.num_stages = 2 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 11 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 0>, array<i32: 1>]}
    "use"(%2#0) : (tensor<128x128xf32, #blocked>) -> ()
    tt.return
  }
```
**EN:** This block defines `user_partition_has_cycle` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×13) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `user_partition_has_cycle`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×13）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 377-426
```mlir
  // CHECK-LABEL: @matmul_tma_acc_with_conditional_def_and_use_flag
  tt.func @matmul_tma_acc_with_conditional_def_and_use_flag(%arg0: !tt.tensordesc<128x64xf16, #shared>, %arg1: !tt.tensordesc<64x128xf16, #shared>) {
    %c32_i32 = arith.constant 32 : i32
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %true = arith.constant true
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    // CHECK: [[AREF:%.*]] = nvws.aref.create
    // CHECK-NEXT: {{.*}}, [[ATOK:%.*]] = nvws.aref.put.enter [[AREF]] :
    // CHECK-NEXT: aref.buffer
    // CHECK-NEXT: tmem_store
    %0 = ttng.tmem_store %cst, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK-NEXT: scf.for {{.*}} iter_args({{.*}}, [[TOK:%.*]] = [[ATOK]])
    %1:2 = scf.for %arg2 = %c0_i32 to %c32_i32 step %c1_i32 iter_args(%arg3 = %true, %arg4 = %0) -> (i1, !ttg.async.token)  : i32 {
      %2:3 = "get_offsets"(%arg2) {ttg.partition = array<i32: 2>} : (i32) -> (i32, i32, i32)
      %3 = tt.descriptor_load %arg0[%2#0, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %4 = tt.descriptor_load %arg1[%2#1, %2#2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<64x128xf16, #shared> -> tensor<64x128xf16, #blocked1>
      %5 = ttg.local_alloc %3 {ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %6 = ttg.local_alloc %4 {ttg.partition = array<i32: 2>} : (tensor<64x128xf16, #blocked1>) -> !ttg.memdesc<64x128xf16, #shared, #smem>
      // CHECK: aref.buffer [[AREF]], [[TOK]] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: tc_gen5_mma
      %7 = ttng.tc_gen5_mma %5, %6, %result[%arg4], %arg3, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      %8 = arith.cmpi eq, %arg2, %c0_i32 {ttg.partition = array<i32: 0, 1>} : i32
      %9 = arith.cmpi ne, %arg2, %c0_i32 {ttg.partition = array<i32: 0, 1>} : i32
      // CHECK: scf.if
      %10 = scf.if %8 -> (!ttg.async.token) {
        // CHECK-NEXT: aref.put.exit [[AREF]], [[TOK]] [#nvws.async_op<tc5mma>] {ttg.partition = array<i32: 1>}
      // CHECK: scf.if
        // CHECK-NEXT: some_op
        "some_op"() {ttg.partition = array<i32: 0>} : () -> ()
        // CHECK-NEXT: aref.get.enter [[AREF]] {ttg.partition = array<i32: 0>}
        // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer {{.*}} {ttg.partition = array<i32: 0>}
        // CHECK-NEXT: tmem_load [[BUF]]
        // CHECK-NEXT: aref.get.exit
        %result_0, %token_1 = ttng.tmem_load %result[%7] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
        // CHECK-NEXT: acc_user
        "acc_user"(%result_0) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
      // CHECK: [[TOK1:%.*]] = scf.if
        // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.put.enter [[AREF]] {ttg.partition = array<i32: 1>}
        // CHECK-NEXT: scf.yield {ttg.partition = array<i32: 1>} [[TOK]]
        scf.yield %token_1 : !ttg.async.token
      } else {
        scf.yield %7 : !ttg.async.token
      } {ttg.partition = array<i32: 0, 1>, ttg.partition.outputs = [array<i32: 1>]}
      // CHECK: scf.yield {{.*}}, [[TOK1]]
      scf.yield %9, %10 : i1, !ttg.async.token
    } {tt.num_stages = 4 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 12 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 0, 1>, array<i32: 1>]}
    tt.return
  }
```
**EN:** This block defines `matmul_tma_acc_with_conditional_def_and_use_flag` and exercises shared-memory allocation, loop-carried state, control-flow joins. Embedded check comments (CHECK×6, CHECK-LABEL×1, CHECK-NEXT×14) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `matmul_tma_acc_with_conditional_def_and_use_flag`，并覆盖 共享内存分配、循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×6, CHECK-LABEL×1, CHECK-NEXT×14）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 428-465
```mlir
  // CHECK-LABEL: @specialize_mma_only
  tt.func @specialize_mma_only(%arg0: !tt.tensordesc<64x128xf16, #shared>, %arg1: !ttg.memdesc<128x64xf16, #shared, #smem>, %arg2: i32) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %true = arith.constant true
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    // CHECK: tmem_alloc
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create
    // CHECK-NEXT: aref.put.enter
    // CHECK-NEXT: aref.buffer
    // CHECK-NEXT: tmem_store
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %0 = ttng.tmem_store %cst, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK-NEXT: [[TOK:%.*]] = scf.for {{.*}} iter_args([[TOK:%.*]] = {{.*}})
    %1 = scf.for %arg3 = %c0_i32 to %arg2 step %c1_i32 iter_args(%arg4 = %0) -> (!ttg.async.token)  : i32 {
      %2 = tt.descriptor_load %arg0[%arg3, %arg3] {ttg.partition = array<i32: 2>} : !tt.tensordesc<64x128xf16, #shared> -> tensor<64x128xf16, #blocked1>
      // CHECK: [[BUF:%.*]] = nvws.aref.buffer [[AREF]], [[TOK]]
      // CHECK-NEXT: tmem_load [[BUF]]
      %result_2, %token_3 = ttng.tmem_load %result[%arg4] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
      %3:2 = "some_producer"(%2, %result_2) {ttg.partition = array<i32: 0>} : (tensor<64x128xf16, #blocked1>, tensor<128x128xf32, #blocked>) -> (tensor<128x64xf16, #blocked1>, tensor<128x128xf32, #blocked>)
      %4 = ttg.local_alloc %3#0 {ttg.partition = array<i32: 0>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %5 = ttg.memdesc_trans %4 {order = array<i32: 1, 0>, ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem> -> !ttg.memdesc<64x128xf16, #shared1, #smem>
      // CHECK: tmem_store {{.*}}, [[BUF]]
      // CHECK-NEXT: aref.put.exit [[AREF]], [[TOK]]
      %6 = ttng.tmem_store %3#1, %result[%token_3], %true {ttg.partition = array<i32: 0>} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      // CHECK-NEXT: aref.get.enter {{.*}} {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: aref.buffer {{.*}} {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: tc_gen5_mma
      // CHECK-NEXT: aref.get.exit {{.*}} {ttg.partition = array<i32: 1>}
      %7 = ttng.tc_gen5_mma %arg1, %5, %result[%6], %true, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.put.enter [[AREF]] {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: scf.yield [[TOK]]
      scf.yield %7 : !ttg.async.token
    } {tt.num_stages = 3 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 15 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 0>]}
    %result_0, %token_1 = ttng.tmem_load %result[%1] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
    "use"(%result_0) : (tensor<128x128xf32, #blocked>) -> ()
    tt.return
  }
```
**EN:** This block defines `specialize_mma_only` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×13) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `specialize_mma_only`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×13）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 467-498
```mlir
  // CHECK-LABEL: @load_scale_mma_user
  tt.func @load_scale_mma_user(%arg0: !ttg.memdesc<128x64xf16, #shared, #smem>, %arg1: !ttg.memdesc<64x128xf16, #shared, #smem>, %arg2: !tt.tensordesc<8x128xi8, #shared>, %arg3: !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>, %arg4: i32) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %true = arith.constant true
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    // CHECK: tmem_alloc {{.*}} !ttg.memdesc<1x128x128xf32
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create
    // CHECK-NEXT: aref.put.enter [[AREF]]
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %0 = ttng.tmem_store %cst, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK: tmem_alloc {{.*}} !ttg.memdesc<128x8xi8
    // CHECK-NEXT: [[SCALE_AREF:%.*]] = nvws.aref.create
    // CHECK-NEXT: [[TOK1:%.*]] = scf.for
    %1 = scf.for %arg5 = %c0_i32 to %arg4 step %c1_i32 iter_args(%arg6 = %0) -> (!ttg.async.token)  : i32 {
      %2 = tt.descriptor_load %arg2[%arg5, %arg5] {ttg.partition = array<i32: 2>} : !tt.tensordesc<8x128xi8, #shared> -> tensor<8x128xi8, #blocked1>
      %3 = ttg.local_alloc %2 {ttg.partition = array<i32: 2>} : (tensor<8x128xi8, #blocked1>) -> !ttg.memdesc<8x128xi8, #shared, #smem>
      %4 = ttg.local_load %3 {ttg.partition = array<i32: 0>} : !ttg.memdesc<8x128xi8, #shared, #smem> -> tensor<8x128xi8, #linear1>
      %5 = tt.trans %4 {order = array<i32: 1, 0>, ttg.partition = array<i32: 0>} : tensor<8x128xi8, #linear1> -> tensor<128x8xi8, #linear>
      // CHECK: put.enter [[SCALE_AREF]]
      // CHECK-NEXT: aref.buffer [[SCALE_AREF]]
      // CHECK-NEXT: arith.constant
      // CHECK-NEXT: tmem_store
      // CHECK-NEXT: put.exit [[SCALE_AREF]]
      %result_2 = ttng.tmem_alloc %5 {ttg.partition = array<i32: 0>} : (tensor<128x8xi8, #linear>) -> !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
      // CHECK-NEXT: aref.buffer [[AREF]]
      // CHECK-NEXT: get.enter [[SCALE_AREF]]
      // CHECK-NEXT: aref.buffer [[SCALE_AREF]]
      // CHECK-NEXT: tc_gen5_mma_scaled
      // CHECK-NEXT: get.exit [[SCALE_AREF]]
      // CHECK-NEXT: put.exit [[AREF]]
      %6 = ttng.tc_gen5_mma_scaled %arg0, %arg1, %result[%arg6], %result_2, %arg3, %true, %true lhs = e4m3 rhs = e4m3 {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>, !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
```
**EN:** This block defines `load_scale_mma_user` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×14) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `load_scale_mma_user`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×14）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 500-520
```mlir
      // CHECK-NEXT: get.enter [[AREF]]
      // CHECK-NEXT: aref.buffer [[AREF]]
      // CHECK-NEXT: tmem_load
      // CHECK-NEXT: get.exit [[AREF]]
      %result_3, %token_4 = ttng.tmem_load %result[%6] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
      // CHECK-NEXT: user
      "user"(%result_3) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
      // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.put.enter [[AREF]]
      // CHECK-NEXT: scf.yield [[TOK]]
      scf.yield %token_4 : !ttg.async.token
      // CHECK-NEXT: }
    } {tt.num_stages = 3 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 16 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>]}
    // CHECK-NEXT: put.exit [[AREF]], [[TOK1]] [#nvws.async_op<tc5mma>] {ttg.partition = array<i32: 1>, ttg.warp_specialize.tag = 16 : i32}
    // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.get.enter [[AREF]] :
    // CHECK-NEXT: aref.buffer [[AREF]], [[TOK]] :
    // CHECK-NEXT: tmem_load
    // CHECK-NEXT: get.exit [[AREF]], [[TOK]] [#nvws.async_op<none>] :
    %result_0, %token_1 = ttng.tmem_load %result[%1] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
    "use"(%result_0) : (tensor<128x128xf32, #blocked>) -> ()
    tt.return
  }
```
**EN:** This block defines the test function(s) and exercises operations such as `get.enter`, `aref.buffer`, `get.exit`, `ttng.tmem_load`, `ttg.partition`. Embedded check comments (CHECK-NEXT×13) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `get.enter`、`aref.buffer`、`get.exit`、`ttng.tmem_load`、`ttg.partition` 这样的操作。 其中嵌入的检查注释（CHECK-NEXT×13）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 522-545
```mlir
  // CHECK-LABEL: @store_mma_load
  tt.func @store_mma_load(%arg0: i32, %arg1: !tt.tensordesc<128x64xf16, #shared>, %arg2: !ttg.memdesc<64x128xf16, #shared, #smem>) {
    %true = arith.constant true
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    // CHECK: tmem_alloc
    // CHECK-NEXT: aref.create
    // CHECK-NEXT: aref.put.enter
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %0 = scf.for %arg3 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg4 = %token) -> (!ttg.async.token)  : i32 {
      %1 = tt.descriptor_load %arg1[%arg3, %arg3] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %2 = arith.addf %1, %1 {ttg.partition = array<i32: 0>} : tensor<128x64xf16, #blocked1>
      %3 = ttg.local_alloc %2 {ttg.partition = array<i32: 0>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      // CHECK: make_acc
      %4 = "make_acc"() {ttg.partition = array<i32: 0>} : () -> tensor<128x128xf32, #blocked>
      // CHECK-NEXT: aref.buffer {{.*}} {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: tmem_store
      // CHECK-NEXT: aref.put.exit {{.*}} {ttg.partition = array<i32: 0>}
      %5 = ttng.tmem_store %4, %result[%arg4], %true {ttg.partition = array<i32: 0>} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      // CHECK-NEXT: aref.get.enter {{.*}} {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: aref.buffer {{.*}} {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: tc_gen5_mma
      // CHECK-NEXT: get.exit {{.*}} {ttg.partition = array<i32: 1>}
      %6 = ttng.tc_gen5_mma %3, %arg2, %result[%5], %true, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This block defines `store_mma_load` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×9) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `store_mma_load`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×9）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 547-557
```mlir
      // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.put.enter {{.*}} {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: aref.buffer {{.*}} {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: tmem_load
      %result_0, %token_1 = ttng.tmem_load %result[%6] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
      // CHECK-NEXT: use
      "use"(%result_0) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
      // CHECK-NEXT: scf.yield [[TOK]]
      scf.yield %token_1 : !ttg.async.token
    } {tt.disallow_acc_multi_buffer, tt.num_stages = 2 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 17 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 0>]}
    tt.return
  }
```
**EN:** This block defines the test function(s) and exercises operations such as `nvws.aref.put.enter`, `ttg.partition`, `aref.buffer`, `ttng.tmem_load`, `ttg.memdesc`. Embedded check comments (CHECK-NEXT×5) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `nvws.aref.put.enter`、`ttg.partition`、`aref.buffer`、`ttng.tmem_load`、`ttg.memdesc` 这样的操作。 其中嵌入的检查注释（CHECK-NEXT×5）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 559-580
```mlir
  // CHECK-LABEL: @local_alloc_into_mma
  tt.func @local_alloc_into_mma(%arg0: i32, %arg1: tensor<128x64xf16, #blocked1>, %arg2: !tt.tensordesc<64x128xf16, #shared>) {
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    %true = arith.constant true
    // CHECK: tmem_alloc
    // CHECK-NEXT: aref.create
    // CHECK-NEXT: aref.put.enter
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %5 = scf.for %arg3 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg4 = %token) -> (!ttg.async.token)  : i32 {
      %0 = ttg.local_alloc %arg1 {ttg.partition = array<i32: 0>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %1 = tt.descriptor_load %arg2[%arg3, %arg3] {ttg.partition = array<i32: 2>} : !tt.tensordesc<64x128xf16, #shared> -> tensor<64x128xf16, #blocked1>
      %2 = arith.addf %1, %1 {ttg.partition = array<i32: 0>} : tensor<64x128xf16, #blocked1>
      %3 = ttg.local_alloc %2 {ttg.partition = array<i32: 0>} : (tensor<64x128xf16, #blocked1>) -> !ttg.memdesc<64x128xf16, #shared, #smem>
      // CHECK: aref.buffer
      %4 = ttng.tc_gen5_mma %0, %3, %result[%arg4], %true, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      scf.yield %4 : !ttg.async.token
    } {ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>], tt.num_stages = 2 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 18 : i32}
    // CHECK: aref.put.exit
    ttng.tmem_load %result[%5] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
    tt.return
  }
```
**EN:** This block defines `local_alloc_into_mma` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `local_alloc_into_mma`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 582-628
```mlir
  tt.func @shmem_sink_iterator_invalidation(%arg0: i32, %arg1: i32, %arg2: i32, %arg3: !tt.tensordesc<128x64xf16, #shared>, %arg4: !tt.tensordesc<128x64xf16, #shared>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %c64_i32 = arith.constant 64 : i32
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    %true = arith.constant true
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    // CHECK: tmem_alloc {{.*}} !ttg.memdesc<1x128x128xf32
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create
    // CHECK-NEXT: {{.*}}, [[ATOK:%.*]] = nvws.aref.put.enter [[AREF]]
    %0 = ttng.tmem_store %cst, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK: tmem_alloc {{.*}} !ttg.memdesc<1x128x64xf16
    // CHECK-NEXT: [[LHS_AREF:%.*]] = nvws.aref.create
    // CHECK-NEXT: [[TOK1:%.*]] = scf.for {{.*}} iter_args([[TOK2:%.*]] = [[ATOK]])
    %1 = scf.for %arg5 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg6 = %0) -> (!ttg.async.token)  : i32 {
      %2 = arith.muli %arg5, %c64_i32 {ttg.partition = array<i32: 2>} : i32
      %3 = tt.descriptor_load %arg4[%arg2, %2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %4 = tt.descriptor_load %arg3[%arg1, %2] {ttg.partition = array<i32: 2>} : !tt.tensordesc<128x64xf16, #shared> -> tensor<128x64xf16, #blocked1>
      %5 = ttg.local_alloc %4 {ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %6 = ttg.local_load %5 {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x64xf16, #shared, #smem> -> tensor<128x64xf16, #blocked2>
      %7 = ttg.local_alloc %3 {ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %8 = ttg.memdesc_trans %7 {order = array<i32: 1, 0>, ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem> -> !ttg.memdesc<64x128xf16, #shared1, #smem>
      // CHECK: {{.*}}, [[TOK:%.*]] = nvws.aref.put.enter [[LHS_AREF]]
      // CHECK-NEXT: aref.buffer [[LHS_AREF]], [[TOK]]
      // CHECK-NEXT: arith.constant
      // CHECK-NEXT: tmem_store
      // CHECK-NEXT: aref.put.exit
      %result_2 = ttng.tmem_alloc %6 {ttg.partition = array<i32: 0>} : (tensor<128x64xf16, #blocked2>) -> !ttg.memdesc<128x64xf16, #tmem1, #ttng.tensor_memory>
      // CHECK-NEXT: aref.buffer [[AREF]], [[TOK2]]
      // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.get.enter [[LHS_AREF]]
      // CHECK-NEXT: aref.buffer [[LHS_AREF]], [[TOK]]
      // CHECK-NEXT: tc_gen5_mma
      // CHECK-NEXT: get.exit [[LHS_AREF]]
      %9 = ttng.tc_gen5_mma %result_2, %8, %result[%arg6], %true, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #tmem1, #ttng.tensor_memory>, !ttg.memdesc<64x128xf16, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      scf.yield %9 : !ttg.async.token
    } {tt.num_stages = 2 : i32, tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 19 : i32, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>]}
    // CHECK: aref.put.exit [[AREF]], [[TOK1]]
    // CHECK-NEXT: {{.*}}, [[TOK:%.*]] = nvws.aref.get.enter [[AREF]] :
    // CHECK-NEXT: aref.buffer [[AREF]], [[TOK]]
    // CHECK-NEXT: tmem_load
    // CHECK-NEXT: aref.get.exit [[AREF]]
    // CHECK-NEXT: use
    %result_0, %token_1 = ttng.tmem_load %result[%1] : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
    "use"(%result_0) : (tensor<128x128xf32, #blocked>) -> ()
    tt.return
  }
}
```
**EN:** This block defines `shmem_sink_iterator_invalidation` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK×4, CHECK-NEXT×18) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `shmem_sink_iterator_invalidation`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK×4, CHECK-NEXT×18）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 630
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 631-668
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
#smem = #ttg.shared_memory
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
#tmem1 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
  tt.func public @attention_forward(%arg0: !ttg.memdesc<256x64xf16, #shared, #smem>, %arg1: !tt.tensordesc<64x64xf16, #shared>, %arg2: !tt.tensordesc<64x64xf16, #shared>, %arg3: f32, %arg4: i32) {
    %cst = arith.constant dense<1.000000e+00> : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<256x64xf32, #blocked>
    %cst_1 = arith.constant dense<0xFF800000> : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %c64_i32 = arith.constant 64 : i32
    %c0_i32 = arith.constant 0 : i32
    %false = arith.constant false
    %true = arith.constant true
    // CHECK: tmem_alloc {{.*}} !ttg.memdesc<2x256x64xf32
    // CHECK-NEXT: [[AREF_S:%.*]] = nvws.aref.create
    // CHECK-NEXT: {{.*}}, [[TOK_S:%.*]] = nvws.aref.put.enter [[AREF_S]]
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    // CHECK: tmem_alloc {{.*}} !ttg.memdesc<1x256x64xf32
    // CHECK-NEXT: [[AREF_O:%.*]] = nvws.aref.create
    // CHECK-NEXT: {{.*}}, [[TOK_O:%.*]] = nvws.aref.put.enter [[AREF_O]]
    // CHECK-NEXT: [[BUF_O:%.*]] = nvws.aref.buffer [[AREF_O]], [[TOK_O]]
    // CHECK-NEXT: tmem_store {{.*}}, [[BUF_O]]
    %result_2, %token_3 = ttng.tmem_alloc : () -> (!ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %0 = ttng.tmem_store %cst_0, %result_2[%token_3], %true : tensor<256x64xf32, #blocked> -> !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK: tmem_alloc {{.*}} !ttg.memdesc<1x256x64xf16
    // CHECK-NEXT: [[AREF_P:%.*]] = nvws.aref.create
    // CHECK-NEXT: [[RET:%.*]]:4 = scf.for {{.*}} iter_args([[A1:%.*]] = {{.*}}, [[A2:%.*]] = {{.*}}, [[TOKS:%.*]] = [[TOK_S]], [[TOKO:%.*]] = [[TOK_O]])
    %1:4 = scf.for %arg5 = %c0_i32 to %arg4 step %c64_i32 iter_args(%arg6 = %cst, %arg7 = %cst_1, %arg8 = %token, %arg9 = %0) -> (tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, !ttg.async.token, !ttg.async.token)  : i32 {
      %2 = tt.descriptor_load %arg1[%arg5, %c0_i32] {ttg.partition = array<i32: 2>} : !tt.tensordesc<64x64xf16, #shared> -> tensor<64x64xf16, #blocked1>
      %3 = ttg.local_alloc %2 {ttg.partition = array<i32: 2>} : (tensor<64x64xf16, #blocked1>) -> !ttg.memdesc<64x64xf16, #shared, #smem>
      %4 = ttg.memdesc_trans %3 {order = array<i32: 1, 0>, ttg.partition = array<i32: 1>} : !ttg.memdesc<64x64xf16, #shared, #smem> -> !ttg.memdesc<64x64xf16, #shared1, #smem>
      // CHECK: [[BUF:%.*]] = nvws.aref.buffer [[AREF_S]], [[TOKS]] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: tc_gen5_mma {{.*}}, {{.*}}, [[BUF]]
      // CHECK-NEXT: put.exit [[AREF_S]], [[TOKS]] [#nvws.async_op<tc5mma>] {ttg.partition = array<i32: 1>}
      %5 = ttng.tc_gen5_mma %arg0, %4, %result[%arg8], %false, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<256x64xf16, #shared, #smem>, !ttg.memdesc<64x64xf16, #shared1, #smem>, !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.target`, `ttg.partition`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `attention_forward`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.target`, `ttg.partition`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `attention_forward`。

### Lines 670-674
```mlir
      // CHECK: {{.*}}, [[TOKS:%.*]] = nvws.aref.get.enter [[AREF_S]] {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: [[BUF:%.*]] = nvws.aref.buffer [[AREF_S]], [[TOKS]] {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: tmem_load [[BUF]]
      // CHECK-NEXT: get.exit [[AREF_S]], [[TOKS]] [#nvws.async_op<none>] {ttg.partition = array<i32: 0>}
      %result_6, %token_7 = ttng.tmem_load %result[%5] {ttg.partition = array<i32: 0>} : !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<256x64xf32, #blocked>
```
**EN:** This block defines the test function(s) and exercises operations such as `nvws.aref.get.enter`, `ttg.partition`, `nvws.aref.buffer`, `get.exit`, `ttng.tmem_load`. Embedded check comments (CHECK×1, CHECK-NEXT×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `nvws.aref.get.enter`、`ttg.partition`、`nvws.aref.buffer`、`get.exit`、`ttng.tmem_load` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 676-691
```mlir
      %6 = "compute_row_max"(%result_6, %arg3) {ttg.partition = array<i32: 0>} : (tensor<256x64xf32, #blocked>, f32) -> tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
      %7 = "sub_row_max"(%result_6, %6, %arg3) {ttg.partition = array<i32: 0>} : (tensor<256x64xf32, #blocked>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, f32) -> tensor<256x64xf32, #blocked>
      %8 = math.exp2 %7 {ttg.partition = array<i32: 0>} : tensor<256x64xf32, #blocked>
      %9 = arith.subf %arg7, %6 {ttg.partition = array<i32: 3>} : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
      %10 = arith.subf %arg7, %6 {ttg.partition = array<i32: 0>} : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
      %11 = math.exp2 %9 {ttg.partition = array<i32: 3>} : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
      %12 = math.exp2 %10 {ttg.partition = array<i32: 0>} : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
      %13 = "tt.reduce"(%8) <{axis = 1 : i32}> ({
      ^bb0(%arg10: f32, %arg11: f32):
        %24 = arith.addf %arg10, %arg11 {ttg.partition = array<i32: 0>}: f32
        tt.reduce.return %24 {ttg.partition = array<i32: 0>} : f32
      }) {ttg.partition = array<i32: 0>, ttg.partition.outputs = [array<i32: 0>]} : (tensor<256x64xf32, #blocked>) -> tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
      %14 = arith.mulf %arg6, %12 {ttg.partition = array<i32: 0>} : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
      %15 = arith.addf %14, %13 {ttg.partition = array<i32: 0>} : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
      %16 = tt.expand_dims %11 {axis = 1 : i32, ttg.partition = array<i32: 3>} : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<256x1xf32, #blocked>
      %17 = tt.broadcast %16 {ttg.partition = array<i32: 3>} : tensor<256x1xf32, #blocked> -> tensor<256x64xf32, #blocked>
```
**EN:** This block contributes intermediate IR built from `ttg.partition`, `math.exp2`, `arith.subf`, `tt.reduce`, `arith.addf`, `tt.reduce.return`, `ttg.partition.outputs`, `arith.mulf`, ... (+2). It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.partition`, `math.exp2`, `arith.subf`, `tt.reduce`, `arith.addf`, `tt.reduce.return`, `ttg.partition.outputs`, `arith.mulf`, ... (+2) 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 693-695
```mlir
      // CHECK: [[BUF:%.*]] = nvws.aref.buffer [[AREF_O]], [[TOKO]] {ttg.partition = array<i32: 3>}
      // CHECK-NEXT: tmem_load [[BUF]]
      %result_8, %token_9 = ttng.tmem_load %result_2[%arg9] {ttg.partition = array<i32: 3>} : !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<256x64xf32, #blocked>
```
**EN:** This block defines the test function(s) and exercises operations such as `nvws.aref.buffer`, `ttg.partition`, `ttng.tmem_load`, `ttg.memdesc`. Embedded check comments (CHECK×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `nvws.aref.buffer`、`ttg.partition`、`ttng.tmem_load`、`ttg.memdesc` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 697-706
```mlir
      %18 = arith.mulf %result_8, %17 {ttg.partition = array<i32: 3>} : tensor<256x64xf32, #blocked>
      %19 = tt.descriptor_load %arg2[%arg5, %c0_i32] {ttg.partition = array<i32: 2>} : !tt.tensordesc<64x64xf16, #shared> -> tensor<64x64xf16, #blocked1>
      %20 = ttg.local_alloc %19 {ttg.partition = array<i32: 2>} : (tensor<64x64xf16, #blocked1>) -> !ttg.memdesc<64x64xf16, #shared, #smem>
      %21 = arith.truncf %8 {ttg.partition = array<i32: 0>} : tensor<256x64xf32, #blocked> to tensor<256x64xf16, #blocked>
      // CHECK: {{.*}}, [[TOKP:%.*]] = nvws.aref.put.enter [[AREF_P]] {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: [[BUFP:%.*]] = nvws.aref.buffer [[AREF_P]], [[TOKP]] {ttg.partition = array<i32: 0>}
      // CHECK-NEXT: arith.constant
      // CHECK-NEXT: tmem_store {{.*}}, [[BUFP]]
      // CHECK-NEXT: aref.put.exit [[AREF_P]], [[TOKP]] [#nvws.async_op<none>] {ttg.partition = array<i32: 0>}
      %result_10 = ttng.tmem_alloc %21 {ttg.partition = array<i32: 0>} : (tensor<256x64xf16, #blocked>) -> !ttg.memdesc<256x64xf16, #tmem1, #ttng.tensor_memory>
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation. Embedded check comments (CHECK×1, CHECK-NEXT×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 708-710
```mlir
      // CHECK: tmem_store {{.*}}, [[BUF]]
      // CHECK-NEXT: aref.put.exit [[AREF_O]], [[TOKO]] [#nvws.async_op<none>] {ttg.partition = array<i32: 3>}
      %22 = ttng.tmem_store %18, %result_2[%token_9], %true {ttg.partition = array<i32: 3>} : tensor<256x64xf32, #blocked> -> !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This block defines the test function(s) and exercises operations such as `aref.put.exit`, `ttg.partition`, `ttng.tmem_store`, `ttg.memdesc`. Embedded check comments (CHECK×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `aref.put.exit`、`ttg.partition`、`ttng.tmem_store`、`ttg.memdesc` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 712-719
```mlir
      // CHECK: {{.*}}, [[TOKO:%.*]] = nvws.aref.get.enter [[AREF_O]] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: [[BUFO:%.*]] = nvws.aref.buffer [[AREF_O]], [[TOKO]] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: {{.*}}, [[TOKP:%.*]] = nvws.aref.get.enter [[AREF_P]] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: [[BUFP:%.*]] = nvws.aref.buffer [[AREF_P]], [[TOKP]] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: tc_gen5_mma [[BUFP]], {{.*}}, [[BUFO]]
      // CHECK-NEXT: get.exit [[AREF_P]], [[TOKP]] [#nvws.async_op<tc5mma>] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: get.exit [[AREF_O]], [[TOKO]] [#nvws.async_op<tc5mma>] {ttg.partition = array<i32: 1>}
      %23 = ttng.tc_gen5_mma %result_10, %20, %result_2[%22], %true, %true {ttg.partition = array<i32: 1>} : !ttg.memdesc<256x64xf16, #tmem1, #ttng.tensor_memory>, !ttg.memdesc<64x64xf16, #shared, #smem>, !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>
```
**EN:** This block defines the test function(s) and exercises operations such as `nvws.aref.get.enter`, `ttg.partition`, `nvws.aref.buffer`, `get.exit`, `ttng.tc_gen5_mma`. Embedded check comments (CHECK×1, CHECK-NEXT×6) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `nvws.aref.get.enter`、`ttg.partition`、`nvws.aref.buffer`、`get.exit`、`ttng.tc_gen5_mma` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×6）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 721-738
```mlir
      // CHECK: {{.*}}, [[TOKS:%.*]] = nvws.aref.put.enter [[AREF_S]] {ttg.partition = array<i32: 1>}
      // CHECK-NEXT: {{.*}}, [[TOKO:%.*]] = nvws.aref.put.enter [[AREF_O]] {ttg.partition = array<i32: 3>}
      // CHECK-NEXT: scf.yield {{.*}}, {{.*}}, [[TOKS]], [[TOKO]]
      scf.yield %15, %6, %token_7, %23 : tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, !ttg.async.token, !ttg.async.token
      // CHECK-NEXT: } {
    } {tt.warp_specialize, ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32, 1 : i32], ttg.warp_specialize.tag = 0 : i32, ttg.partition = array<i32: 0, 1, 2, 3>, ttg.partition.outputs = [array<i32: 0>, array<i32: 0>, array<i32: 1>, array<i32: 3>]}
    // CHECK: aref.put.exit [[AREF_O]], [[RET]]#3 [#nvws.async_op<none>] {ttg.partition = array<i32: 3>, ttg.warp_specialize.tag = 0 : i32}
    // CHECK-NEXT: aref.put.exit [[AREF_S]], [[RET]]#2 [#nvws.async_op<tc5mma>] {ttg.partition = array<i32: 1>, ttg.warp_specialize.tag = 0 : i32}
    // CHECK-NEXT: aref.get.enter [[AREF_S]] {ttg.partition = array<i32: 0>, ttg.warp_specialize.tag = 0 : i32}
    // CHECK-NEXT: aref.get.exit [[AREF_S]], {{.*}} [{{.*}}] {ttg.partition = array<i32: 0>, ttg.warp_specialize.tag = 0 : i32}
    // CHECK-NEXT: aref.get.enter [[AREF_O]] :
    // CHECK-NEXT: aref.buffer [[AREF_O]], {{.*}} :
    // CHECK-NEXT: tmem_load
    // CHECK-NEXT: aref.get.exit [[AREF_O]], {{.*}} [{{.*}}] :
    %result_4, %token_5 = ttng.tmem_load %result_2[%1#3] : !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<256x64xf32, #blocked>
    "use"(%1#0, %result_4, %1#1) : (tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>, tensor<256x64xf32, #blocked>, tensor<256xf32, #ttg.slice<{dim = 1, parent = #blocked}>>) -> ()
    tt.return
  }
```
**EN:** This block defines the test function(s) and exercises operations such as `nvws.aref.put.enter`, `ttg.partition`, `scf.yield`, `ttg.async.token`, `tt.warp_specialize`. Embedded check comments (CHECK×2, CHECK-NEXT×10) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `nvws.aref.put.enter`、`ttg.partition`、`scf.yield`、`ttg.async.token`、`tt.warp_specialize` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-NEXT×10）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 740
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 742
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 744-790
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#smem = #ttg.shared_memory
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
  // CHECK-LABEL: @hoisted_alloc
  tt.func @hoisted_alloc(%lb: i32, %ub: i32, %step: i32, %ptr0: !tt.ptr<i32>) {
    %true = arith.constant true
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    // CHECK: tmem_alloc
    // CHECK-NEXT: aref.create
    // CHECK-NEXT: put.enter
    // CHECK-NEXT: aref.buffer
    // CHECK-NEXT: tmem_store
    %res, %tok = ttng.tmem_alloc %cst : (tensor<128x128xf32, #blocked>) -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    // CHECK: scf.for
    %tok0 = scf.for %iv0 = %lb to %ub step %step iter_args(%tok1 = %tok) -> (!ttg.async.token) : i32 {
      %ptrub = tt.addptr %ptr0, %iv0 {ttg.partition = array<i32: 1, 2>} : !tt.ptr<i32>, i32
      %ub1 = tt.load %ptrub {ttg.partition = array<i32: 1, 2>} : !tt.ptr<i32>
      %lb1 = "lb1"(%iv0) {ttg.partition = array<i32: 1, 2>} : (i32) -> i32
      %step1 = "step1"(%iv0) {ttg.partition = array<i32: 1, 2>} : (i32) -> i32
    // CHECK: scf.for
      %tok4 = scf.for %iv = %lb1 to %ub1 step %step1 iter_args(%tok2 = %tok1) -> (!ttg.async.token)  : i32 {
        %sA = "load1"(%iv) {ttg.partition = array<i32: 1>} : (i32) -> !ttg.memdesc<128x64xf32, #shared, #smem>
        %sB = "load2"(%iv) {ttg.partition = array<i32: 1>} : (i32) -> !ttg.memdesc<64x128xf32, #shared, #smem>
        %tok3 = ttng.tc_gen5_mma %sA, %sB, %res[%tok2], %true, %true {ttg.partition = array<i32: 2>} : !ttg.memdesc<128x64xf32, #shared, #smem>, !ttg.memdesc<64x128xf32, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
        // CHECK: scf.yield
        scf.yield {ttg.partition = array<i32: 1, 2>} %tok3 : !ttg.async.token
      } {ttg.partition = array<i32: 1, 2>, ttg.partition.outputs = [array<i32: 2>]}
      // CHECK: put.exit
      // CHECK-NEXT: get.enter
      // CHECK-NEXT: aref.buffer
      // CHECK-NEXT: tmem_load
      // CHECK-NEXT: get.exit
      // CHECK-NEXT: use
      %val, %tok5 = ttng.tmem_load %res[%tok4] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
      "use"(%val) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
      // CHECK: scf.yield
      scf.yield {ttg.partition = array<i32: 0, 1, 2>} %tok5 : !ttg.async.token
    } {tt.warp_specialize, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 2>], ttg.warp_specialize.tag = 0 : i32}
    // CHECK: put.exit
    // CHECK-NEXT: get.enter
    // CHECK-NEXT: get.exit
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.target`, `ttg.partition`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `hoisted_alloc`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.target`, `ttg.partition`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `hoisted_alloc`。

### Line 792
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 794-844
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#blocked3 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#smem = #ttg.shared_memory
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
  // CHECK-LABEL: @if_split_workaround
  tt.func @if_split_workaround(%arg0: !tt.tensordesc<1x64xf16, #shared>, %arg1: tensor<64x128x!tt.ptr<f16>, #blocked3> {tt.contiguity = dense<[1, 64]> : tensor<2xi32>, tt.divisibility = dense<16> : tensor<2xi32>}) {
    %c0_i32 = arith.constant 0 : i32
    %c1_i32 = arith.constant 1 : i32
    %true = arith.constant true
    %false = arith.constant false
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    %c32_i32 = arith.constant 32 : i32
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %0 = ttng.tmem_store %cst, %result[%token], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK: scf.for
    %1:3 = scf.for %arg2 = %c0_i32 to %c32_i32 step %c1_i32 iter_args(%arg3 = %true, %arg4 = %arg1, %arg5 = %0) -> (i1, tensor<64x128x!tt.ptr<f16>, #blocked3>, !ttg.async.token)  : i32 {
      %2:3 = "get_offsets"(%arg2) {loop.cluster = 3 : i32, loop.stage = 0 : i32, ttg.partition = array<i32: 1, 2>} : (i32) -> (i32, tensor<64x128xi32, #blocked3>, i32)
      %3 = tt.splat %2#0 {loop.cluster = 3 : i32, loop.stage = 0 : i32, ttg.partition = array<i32: 2>} : i32 -> tensor<128xi32, #blocked2>
      %4 = tt.descriptor_gather %arg0[%3, %2#2] {loop.cluster = 3 : i32, loop.stage = 0 : i32, ttg.partition = array<i32: 2>} : (!tt.tensordesc<1x64xf16, #shared>, tensor<128xi32, #blocked2>, i32) -> tensor<128x64xf16, #blocked1>
      %5 = tt.addptr %arg4, %2#1 {loop.cluster = 3 : i32, loop.stage = 1 : i32, tt.constancy = dense<1> : tensor<2xi32>, tt.contiguity = dense<[1, 64]> : tensor<2xi32>, tt.divisibility = dense<16> : tensor<2xi32>, ttg.partition = array<i32: 1>} : tensor<64x128x!tt.ptr<f16>, #blocked3>, tensor<64x128xi32, #blocked3>
      %6 = tt.load %5 {loop.cluster = 3 : i32, loop.stage = 1 : i32, ttg.partition = array<i32: 1>} : tensor<64x128x!tt.ptr<f16>, #blocked3>
      %7 = ttg.local_alloc %4 {loop.cluster = 2 : i32, loop.stage = 2 : i32, ttg.partition = array<i32: 2>} : (tensor<128x64xf16, #blocked1>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
      %8 = ttg.local_alloc %6 {loop.cluster = 2 : i32, loop.stage = 2 : i32, ttg.partition = array<i32: 1>} : (tensor<64x128xf16, #blocked3>) -> !ttg.memdesc<64x128xf16, #shared, #smem>
      // CHECK: tc_gen5_mma {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32
      %9 = ttng.tc_gen5_mma %7, %8, %result[%arg5], %arg3, %true {loop.cluster = 2 : i32, loop.stage = 2 : i32, tt.self_latency = 1 : i32, ttg.partition = array<i32: 1>} : !ttg.memdesc<128x64xf16, #shared, #smem>, !ttg.memdesc<64x128xf16, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      %10 = arith.cmpi eq, %arg2, %c0_i32 {loop.cluster = 1 : i32, loop.stage = 3 : i32, ttg.partition = array<i32: 0, 1>} : i32
      %11 = arith.select %10, %false, %true {loop.cluster = 1 : i32, loop.stage = 3 : i32, ttg.partition = array<i32: 1>} : i1
      // CHECK: scf.if
      // CHECK-NEXT: put.exit {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32
      // CHECK} {loop.cluster = 2 : i32, loop.stage = 2 : i32
      // CHECK: scf.if
      // CHECK: } {loop.cluster = 4 : i32, loop.stage = 3 : i32
      // CHECK: scf.if
      // CKECK-NEXT: put.enter {{.*}} {loop.cluster = 2 : i32, loop.stage = 2 : i32
      // CHECK: } {loop.cluster = 2 : i32, loop.stage = 2 : i32
      %12 = scf.if %10 -> (!ttg.async.token) {
        %result_0, %token_1 = ttng.tmem_load %result[%9] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
        "acc_user"(%result_0) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
        scf.yield {ttg.partition = array<i32: 0, 1>} %token_1 : !ttg.async.token
      } else {
        scf.yield {ttg.partition = array<i32: 0, 1>} %9 : !ttg.async.token
      } {loop.cluster = 4 : i32, loop.stage = 3 : i32, ttg.partition = array<i32: 0, 1>, ttg.partition.outputs = [array<i32: 1>]}
      scf.yield {ttg.partition = array<i32: 0, 1, 2>} %11, %5, %12 : i1, tensor<64x128x!tt.ptr<f16>, #blocked3>, !ttg.async.token
    } {tt.disallow_acc_multi_buffer, tt.num_stages = 3 : i32, tt.scheduled_max_stage = 3 : i32, tt.warp_specialize, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 1>, array<i32: 1>, array<i32: 1>], ttg.partition.stages = [0 : i32, 1 : i32, 0 : i32], ttg.warp_specialize.tag = 2 : i32}
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.target`, `ttg.partition`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `if_split_workaround`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.target`, `ttg.partition`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `if_split_workaround`。

### Line 846
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 848-880
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#smem = #ttg.shared_memory
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0], [0, 4]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
#tmem_scales = #ttng.tensor_memory_scales_encoding<>
module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
  // CHECK-LABEL: @nested_loop_yes_double_buffer
  tt.func @nested_loop_yes_double_buffer(%lb: i32, %ub: i32, %step: i32, %ptr0: !tt.ptr<i32>) {
    %true = arith.constant true
    %false = arith.constant false
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    // CHECK: [[BUF:%.*]] = ttng.tmem_alloc : () -> !ttg.memdesc<2x128x128xf32, #tmem,
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create [[BUF]]
    %res, %tok = ttng.tmem_alloc : () ->(!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %toka = ttng.tmem_store %cst, %res[%tok], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK: scf.for
    %tok0 = scf.for %iv0 = %lb to %ub step %step iter_args(%tok1 = %toka) -> (!ttg.async.token) : i32 {
      %tok1a = ttng.tmem_store %cst, %res[%tok1], %true {ttg.partition = array<i32: 2>} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      // CHECK: scf.for
      %useD, %tok4 = scf.for %iv = %lb to %ub step %step iter_args(%useD = %false, %tok2 = %tok1a) -> (i1, !ttg.async.token)  : i32 {
        %sA = "load1"(%iv) {ttg.partition = array<i32: 1>} : (i32) -> !ttg.memdesc<128x64xf32, #shared, #smem>
        %sB = "load2"(%iv) {ttg.partition = array<i32: 1>} : (i32) -> !ttg.memdesc<64x128xf32, #shared, #smem>
        %tok3 = ttng.tc_gen5_mma %sA, %sB, %res[%tok2], %useD, %true {ttg.partition = array<i32: 2>} : !ttg.memdesc<128x64xf32, #shared, #smem>, !ttg.memdesc<64x128xf32, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
        scf.yield {ttg.partition = array<i32: 1, 2>} %true, %tok3 : i1, !ttg.async.token
      } {ttg.partition = array<i32: 1, 2>, ttg.partition.outputs = [array<i32: 2>, array<i32: 2>]}
      %val, %tok5 = ttng.tmem_load %res[%tok4] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
      "use"(%val) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
      scf.yield {ttg.partition = array<i32: 0, 1, 2>} %tok5 : !ttg.async.token
    } {tt.warp_specialize, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 2>], ttg.warp_specialize.tag = 0 : i32}
    tt.return
  }
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.target`, `ttg.partition`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `nested_loop_yes_double_buffer`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.target`, `ttg.partition`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `nested_loop_yes_double_buffer`。

### Lines 882-906
```mlir
  // CHECK-LABEL: @nested_loop_no_double_buffer
  tt.func @nested_loop_no_double_buffer(%lb: i32, %ub: i32, %step: i32, %ptr0: !tt.ptr<i32>) {
    %true = arith.constant true
    %false = arith.constant false
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    // CHECK: [[BUF:%.*]] = ttng.tmem_alloc : () -> !ttg.memdesc<1x128x128xf32, #tmem,
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create [[BUF]]
    %res, %tok = ttng.tmem_alloc : () ->(!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %toka = ttng.tmem_store %cst, %res[%tok], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    // CHECK: scf.for
    %tok0 = scf.for %iv0 = %lb to %ub step %step iter_args(%tok1 = %toka) -> (!ttg.async.token) : i32 {
      %tok1a = ttng.tmem_store %cst, %res[%tok1], %true {ttg.partition = array<i32: 0>} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      // CHECK: scf.for
      %useD, %tok4 = scf.for %iv = %lb to %ub step %step iter_args(%useD = %false, %tok2 = %tok1a) -> (i1, !ttg.async.token)  : i32 {
        %sA = "load1"(%iv) {ttg.partition = array<i32: 1>} : (i32) -> !ttg.memdesc<128x64xf32, #shared, #smem>
        %sB = "load2"(%iv) {ttg.partition = array<i32: 1>} : (i32) -> !ttg.memdesc<64x128xf32, #shared, #smem>
        %tok3 = ttng.tc_gen5_mma %sA, %sB, %res[%tok2], %useD, %true {ttg.partition = array<i32: 2>} : !ttg.memdesc<128x64xf32, #shared, #smem>, !ttg.memdesc<64x128xf32, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
        scf.yield {ttg.partition = array<i32: 1, 2>} %true, %tok3 : i1, !ttg.async.token
      } {ttg.partition = array<i32: 1, 2>, ttg.partition.outputs = [array<i32: 2>, array<i32: 2>]}
      %val, %tok5 = ttng.tmem_load %res[%tok4] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
      "use"(%val) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
      scf.yield {ttg.partition = array<i32: 0, 1, 2>} %tok5 : !ttg.async.token
    } {tt.warp_specialize, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 0>], ttg.warp_specialize.tag = 0 : i32}
    tt.return
  }
```
**EN:** This block defines `nested_loop_no_double_buffer` and exercises loop-carried state. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `nested_loop_no_double_buffer`，并覆盖 循环携带状态。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 908-935
```mlir
  // CHECK-LABEL: @nested_loop_yes_double_buffer_scaled
  tt.func @nested_loop_yes_double_buffer_scaled(%lb: i32, %ub: i32, %step: i32, %ptr0: !tt.ptr<i32>,
    %scalesA: tensor<128x8xi8, #linear>, %scalesB: tensor<128x8xi8, #linear>) {
    %true = arith.constant true
    %false = arith.constant false
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked>
    // CHECK: [[BUF:%.*]] = ttng.tmem_alloc : () -> !ttg.memdesc<2x128x128xf32, #tmem,
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create [[BUF]]
    %res, %tok = ttng.tmem_alloc : () ->(!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %toka = ttng.tmem_store %cst, %res[%tok], %true : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %lhs_scales = ttng.tmem_alloc %scalesA: (tensor<128x8xi8, #linear>) -> !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
    %rhs_scales = ttng.tmem_alloc %scalesB : (tensor<128x8xi8, #linear>) -> !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
    // CHECK: scf.for
    %tok0 = scf.for %iv0 = %lb to %ub step %step iter_args(%tok1 = %toka) -> (!ttg.async.token) : i32 {
      %tok1a = ttng.tmem_store %cst, %res[%tok1], %true {ttg.partition = array<i32: 2>} : tensor<128x128xf32, #blocked> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
      // CHECK: scf.for
      %useD, %tok4 = scf.for %iv = %lb to %ub step %step iter_args(%useD = %false, %tok2 = %tok1a) -> (i1, !ttg.async.token)  : i32 {
        %sA = "load1"(%iv) {ttg.partition = array<i32: 1>} : (i32) -> !ttg.memdesc<128x64xf32, #shared, #smem>
        %sB = "load2"(%iv) {ttg.partition = array<i32: 1>} : (i32) -> !ttg.memdesc<64x128xf32, #shared, #smem>
        %tok3 = ttng.tc_gen5_mma_scaled %sA, %sB, %res[%tok2], %lhs_scales, %rhs_scales, %useD, %true lhs = e4m3 rhs = e4m3 {ttg.partition = array<i32: 2>} : !ttg.memdesc<128x64xf32, #shared, #smem>, !ttg.memdesc<64x128xf32, #shared, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>, !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
        scf.yield {ttg.partition = array<i32: 1, 2>} %true, %tok3 : i1, !ttg.async.token
      } {ttg.partition = array<i32: 1, 2>, ttg.partition.outputs = [array<i32: 2>, array<i32: 2>]}
      %val, %tok5 = ttng.tmem_load %res[%tok4] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked>
      "use"(%val) {ttg.partition = array<i32: 0>} : (tensor<128x128xf32, #blocked>) -> ()
      scf.yield {ttg.partition = array<i32: 0, 1, 2>} %tok5 : !ttg.async.token
    } {tt.warp_specialize, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 2>], ttg.warp_specialize.tag = 0 : i32}
    tt.return
  }
```
**EN:** This block defines `nested_loop_yes_double_buffer_scaled` and exercises loop-carried state. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `nested_loop_yes_double_buffer_scaled`，并覆盖 循环携带状态。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 937-965
```mlir
  // CHECK-LABEL: @nested_loop_no_double_buffer_scaled
  tt.func @nested_loop_no_double_buffer_scaled(%lb: i32, %ub: i32, %step: i32, %ptr0: !tt.ptr<i32>,
    %scalesA: tensor<128x8xi8, #linear>, %scalesB: tensor<128x8xi8, #linear>) {
    %true = arith.constant true
    %false = arith.constant false
    %cst = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked>
    // CHECK: [[BUF:%.*]] = ttng.tmem_alloc : () -> !ttg.memdesc<1x128x256xf32, #tmem,
    // CHECK-NEXT: [[AREF:%.*]] = nvws.aref.create [[BUF]]
    %res, %tok = ttng.tmem_alloc : () ->(!ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %toka = ttng.tmem_store %cst, %res[%tok], %true : tensor<128x256xf32, #blocked> -> !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
    %lhs_scales = ttng.tmem_alloc %scalesA : (tensor<128x8xi8, #linear>) -> !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
    %rhs_scales = ttng.tmem_alloc %scalesB : (tensor<128x8xi8, #linear>) -> !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
    // CHECK: scf.for
    %tok0 = scf.for %iv0 = %lb to %ub step %step iter_args(%tok1 = %toka) -> (!ttg.async.token) : i32 {
      %tok1a = ttng.tmem_store %cst, %res[%tok1], %true {ttg.partition = array<i32: 2>} : tensor<128x256xf32, #blocked> -> !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
      // CHECK: scf.for
      %useD, %tok4 = scf.for %iv = %lb to %ub step %step iter_args(%useD = %false, %tok2 = %tok1a) -> (i1, !ttg.async.token)  : i32 {
        %sA = "load1"(%iv) {ttg.partition = array<i32: 1>} : (i32) -> !ttg.memdesc<128x64xf32, #shared, #smem>
        %sB = "load2"(%iv) {ttg.partition = array<i32: 1>} : (i32) -> !ttg.memdesc<64x256xf32, #shared, #smem>
        %tok3 = ttng.tc_gen5_mma_scaled %sA, %sB, %res[%tok2], %lhs_scales, %rhs_scales, %useD, %true lhs = e4m3 rhs = e4m3 {ttg.partition = array<i32: 2>} : !ttg.memdesc<128x64xf32, #shared, #smem>, !ttg.memdesc<64x256xf32, #shared, #smem>, !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>, !ttg.memdesc<128x8xi8, #tmem_scales, #ttng.tensor_memory>
        scf.yield {ttg.partition = array<i32: 1, 2>} %true, %tok3 : i1, !ttg.async.token
      } {ttg.partition = array<i32: 1, 2>, ttg.partition.outputs = [array<i32: 2>, array<i32: 2>]}
      %val, %tok5 = ttng.tmem_load %res[%tok4] {ttg.partition = array<i32: 0>} : !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x256xf32, #blocked>
      "use"(%val) {ttg.partition = array<i32: 0>} : (tensor<128x256xf32, #blocked>) -> ()
      scf.yield {ttg.partition = array<i32: 0, 1, 2>} %tok5 : !ttg.async.token
    } {tt.warp_specialize, ttg.partition = array<i32: 0, 1, 2>, ttg.partition.outputs = [array<i32: 2>], ttg.warp_specialize.tag = 0 : i32}
    tt.return
  }
}
```
**EN:** This block defines `nested_loop_no_double_buffer_scaled` and exercises loop-carried state. Embedded check comments (CHECK×3, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `nested_loop_no_double_buffer_scaled`，并覆盖 循环携带状态。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 967
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 969-970
```mlir
// Test that tmem allocations in functions that do not use warp specialization
// do not trigger an assert if they have multiple uses.
```
**EN:** This comment block provides context for the surrounding test logic: `// Test that tmem allocations in functions that do not use warp specialization`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Test that tmem allocations in functions that do not use warp specialization`。

### Lines 972-992
```mlir
// CHECK-LABEL: @test_tmem_no_ws
// CHECK-NOT: nvws.aref.create
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [32, 0], [64, 0], [0, 4], [0, 8]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
#smem = #ttg.shared_memory
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
#tmem_scales = #ttng.tensor_memory_scales_encoding<>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @test_tmem_no_ws(%arg0: !ttg.memdesc<128x128xi8, #shared, #smem>, %arg1: !ttg.memdesc<128x128xi8, #shared1, #smem>, %arg2: !ttg.memdesc<128x128xi8, #shared1, #smem>, %arg3: tensor<128x16xf8E4M3FN, #linear>, %arg4: tensor<128x16xf8E4M3FN, #linear>, %arg5: tensor<128x16xf8E4M3FN, #linear>) {
    %true = arith.constant true
    %result, %token = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %result_0, %token_1 = ttng.tmem_alloc : () -> (!ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.async.token)
    %result_2 = ttng.tmem_alloc %arg3 : (tensor<128x16xf8E4M3FN, #linear>) -> !ttg.memdesc<128x16xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>
    %result_3 = ttng.tmem_alloc %arg4 : (tensor<128x16xf8E4M3FN, #linear>) -> !ttg.memdesc<128x16xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>
    %result_4 = ttng.tmem_alloc %arg5 : (tensor<128x16xf8E4M3FN, #linear>) -> !ttg.memdesc<128x16xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>
    %0 = ttng.tc_gen5_mma_scaled %arg0, %arg1, %result[%token], %result_2, %result_3, %true, %true lhs = e2m1 rhs = e2m1 : !ttg.memdesc<128x128xi8, #shared, #smem>, !ttg.memdesc<128x128xi8, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x16xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>, !ttg.memdesc<128x16xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>
    %1 = ttng.tc_gen5_mma_scaled %arg0, %arg2, %result_0[%token_1], %result_2, %result_4, %true, %true lhs = e2m1 rhs = e2m1 : !ttg.memdesc<128x128xi8, #shared, #smem>, !ttg.memdesc<128x128xi8, #shared1, #smem>, !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>, !ttg.memdesc<128x16xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>, !ttg.memdesc<128x16xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_tmem_no_ws`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_tmem_no_ws`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's NVWS transforms coverage and focuses on IR behavior under test.  
  **CN:** 该文件属于 Triton 的 NVWS 变换 测试覆盖，关注点是 待测试的 IR 行为。
- **EN:** The `RUN` pipeline drives tools/passes such as `-allow-unregistered-dialect`, `-nvws-insert-tmem-aref`, `-cse`.  
  **CN:** `RUN` 流水线会驱动 `-allow-unregistered-dialect`, `-nvws-insert-tmem-aref`, `-cse` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `ttng` (next-gen TritonGPU ops), `scf` (structured control flow), `math` (math library ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`ttng`（新一代 TritonGPU 操作）、`scf`（结构化控制流）、`math`（数学库操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `ttng`, `scf`, `math`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`ttng`、`scf`、`math`。
