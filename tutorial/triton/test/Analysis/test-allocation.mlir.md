# test-allocation.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Analysis/test-allocation.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises allocation behavior in Triton's analysis coverage. **CN:** 该MLIR 测试用于覆盖 Triton 分析中的分配行为相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -allow-unregistered-dialect -test-print-allocation -verify-diagnostics -o /dev/null`<br>`triton-opt %s -allow-unregistered-dialect -test-print-allocation="get-scratch-size-function=ValidConstant" 2>&1 | FileCheck %s --check-prefix=CHECK-128` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×4, CHECK-NEXT×2; diagnostics annotations: remark×305. **CN:** FileCheck 标记为 CHECK×4, CHECK-NEXT×2；诊断标注为 remark×305。
- **Validation / 验证内容:** **EN:** The file mainly validates that allocation behavior emits the expected analysis remarks/notes. **CN:** 该文件主要验证 分配行为 会发出预期的分析备注/说明。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s -allow-unregistered-dialect -test-print-allocation -verify-diagnostics -o /dev/null
// RUN: triton-opt %s -allow-unregistered-dialect -test-print-allocation="get-scratch-size-function=ValidConstant" 2>&1 | FileCheck %s --check-prefix=CHECK-128
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -allow-unregistered-dialect -test-print-allocation -verify-diagnostics -o /dev/null; triton-opt %s -allow-unregistered-dialect -test-print-allocation="get-scratch-size-function=ValidConstant" 2>&1 | FileCheck %s --check-prefix=CHECK-128` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -allow-unregistered-dialect -test-print-allocation -verify-diagnostics -o /dev/null; triton-opt %s -allow-unregistered-dialect -test-print-allocation="get-scratch-size-function=ValidConstant" 2>&1 | FileCheck %s --check-prefix=CHECK-128`，从而以预期的 pass 顺序和校验模式执行测试。

### Line 4
```mlir
// Check there are no lines with a size different to 128 and we have at least a line with size 128.
```
**EN:** This comment block provides context for the surrounding test logic: `// Check there are no lines with a size different to 128 and we have at least a line with size 128.`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Check there are no lines with a size different to 128 and we have at least a line with size 128.`。

### Lines 6-8
```mlir
// CHECK-128-NOT: scratch offset = {{.*}}, size = {{^(128)}}
// CHECK-128: scratch offset = {{.*}}, size = 128
// CHECK-128-NOT: scratch offset = {{.*}}, size = {{^(128)}}
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 10-24
```mlir
#AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#sliceAd0 = #ttg.slice<{dim = 0, parent = #AL}>
#BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#A_SHARED = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#A_SHARED_1D = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [0]}>
#A_SHARED_T = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [0, 1]}>
#B_SHARED = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
#A_DOT = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth = 2}>
#B_DOT = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth = 2}>
#NVMMA_SHARED_0 = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 16}>
#NVMMA_SHARED_32 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16}>
#NVMMA_SHARED_64 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16}>
#NVMMA_SHARED_128 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#NVMMA_SHARED_FP4PADDED = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8, fp4Padded = true}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#AL`, `#sliceAd0`, `#BL`, `#A_SHARED`, `#A_SHARED_1D`, `#A_SHARED_T`, `#B_SHARED`, `#C`, ... (+7). They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.slice`, `#ttg.swizzled_shared`, `#ttg.nvidia_mma`, `#ttg.dot_op`, `#ttg.nvmma_shared`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#AL`, `#sliceAd0`, `#BL`, `#A_SHARED`, `#A_SHARED_1D`, `#A_SHARED_T`, `#B_SHARED`, `#C`, ... (+7)。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.slice`, `#ttg.swizzled_shared`, `#ttg.nvidia_mma`, `#ttg.dot_op`, `#ttg.nvmma_shared`。

### Lines 26-29
```mlir
#PADDED_SHARED_0_1x256 = #ttg.padded_shared<[256:+8] {order = [1, 0], shape = [1, 256]}>
#PADDED_SHARED_0_1x512 = #ttg.padded_shared<[256:+8] {order = [1, 0], shape = [1, 512]}>
#PADDED_SHARED_0_16x16 = #ttg.padded_shared<[256:+8] {order = [1, 0], shape = [16, 16]}>
#PADDED_SHARED_0_16x32 = #ttg.padded_shared<[256:+8] {order = [1, 0], shape = [16, 32]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#PADDED_SHARED_0_1x256`, `#PADDED_SHARED_0_1x512`, `#PADDED_SHARED_0_16x16`, `#PADDED_SHARED_0_16x32`. They parameterize later tests with compact names for `#ttg.padded_shared`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#PADDED_SHARED_0_1x256`, `#PADDED_SHARED_0_1x512`, `#PADDED_SHARED_0_16x16`, `#PADDED_SHARED_0_16x32`。它们为后续测试提供紧凑名称，用来表示 `#ttg.padded_shared`。

### Lines 31-32
```mlir
#PADDED_SHARED_1_16x256 = #ttg.padded_shared<[128:+4, 256:+8] {order = [1, 0], shape = [16, 256]}>
#PADDED_SHARED_2_16x256 = #ttg.padded_shared<[64:+2, 128:+4, 256:+8] {order = [1, 0], shape = [16, 256]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#PADDED_SHARED_1_16x256`, `#PADDED_SHARED_2_16x256`. They parameterize later tests with compact names for `#ttg.padded_shared`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#PADDED_SHARED_1_16x256`, `#PADDED_SHARED_2_16x256`。它们为后续测试提供紧凑名称，用来表示 `#ttg.padded_shared`。

### Lines 34-36
```mlir
// PartitionedSharedEncoding attributes for testing
#PARTITIONED_SHARED_SWIZZLE = #ttg.partitioned_shared<{numPartitions = 2, numGroups = 2, partitionDim = 0, partitionLayout = #A_SHARED}>
#PARTITIONED_SHARED_PADDED = #ttg.partitioned_shared<{numPartitions = 4, numGroups = 1, partitionDim = 1, partitionLayout = #PADDED_SHARED_0_16x32}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 38
```mlir
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#smem`. They parameterize later tests with compact names for `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.shared_memory`。

### Line 40
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.num-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.num-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 42-48
```mlir
// expected-remark @below {{empty}}
// expected-remark @below {{size = 0}}
tt.func @empty(%A : !tt.ptr<f16>) {
  %cst_2 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
  %0 = ttg.convert_layout %cst_2 : tensor<16x32xf16, #AL> -> tensor<16x32xf16, #AL>
  tt.return
}
```
**EN:** This block defines `empty` and exercises layout conversion. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `empty`，并覆盖 布局转换。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Lines 50-54
```mlir
// expected-remark @below {{matmul_loop}}
// expected-remark @below {{size = 8192}}
tt.func @matmul_loop(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>) {
  %a_ptr_init = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
  %b_ptr_init = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
```
**EN:** This block defines `matmul_loop` and exercises operations such as `tt.func`, `tt.ptr`, `tt.splat`. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `matmul_loop`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.splat` 这样的操作。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Lines 56-60
```mlir
  %a_mask = arith.constant dense<true> : tensor<128x32xi1, #AL>
  %a_other = arith.constant dense<0.00e+00> : tensor<128x32xf16, #AL>
  %b_mask = arith.constant dense<true> : tensor<32x128xi1, #BL>
  %b_other = arith.constant dense<0.00e+00> : tensor<32x128xf16, #BL>
  %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
```
**EN:** This block contributes intermediate IR built from `arith.constant`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `arith.constant` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 62-63
```mlir
  %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
  %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
```
**EN:** This block contributes intermediate IR built from `arith.constant`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `arith.constant` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 65-71
```mlir
  scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
    %a_ = tt.load %a_ptr, %a_mask, %a_other : tensor<128x32x!tt.ptr<f16>, #AL>
    // expected-remark @below {{scratch offset = 0, size = 8192}}
    %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A_DOT>
    %b_ = tt.load %b_ptr, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
    // expected-remark @below {{scratch offset = 0, size = 8192}}
    %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B_DOT>
```
**EN:** This block defines the test function(s) and exercises layout conversion, tensor loads, loop-carried state. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 the test function(s)，并覆盖 布局转换、张量加载、循环携带状态。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Line 73
```mlir
    %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A_DOT> * tensor<32x128xf16, #B_DOT> -> tensor<128x128xf32, #C>
```
**EN:** This block contributes intermediate IR built from `tt.dot`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 75-80
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

### Lines 82-90
```mlir
// Shared memory is available after a tensor's liveness range ends
// expected-remark @below {{reusable}}
// expected-remark @below {{size = 8192}}
tt.func @reusable(%A : !tt.ptr<f16>) {
  %cst1 = arith.constant dense<true> : tensor<128x32xi1, #AL>
  %cst2 = arith.constant dense<0.000000e+00> : tensor<128x32xf16, #AL>
  %cst3 = arith.constant dense<true> : tensor<32x128xi1, #AL>
  %cst4 = arith.constant dense<0.000000e+00> : tensor<32x128xf16, #AL>
  %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
```
**EN:** This block defines `reusable` and exercises operations such as `tt.func`, `tt.ptr`, `arith.constant`. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `reusable`，并覆盖 如 `tt.func`、`tt.ptr`、`arith.constant` 这样的操作。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Lines 92-109
```mlir
  %a_ptr = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
  %b_ptr = tt.splat %A : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #AL>
  %a1_ = tt.load %a_ptr, %cst1, %cst2 : tensor<128x32x!tt.ptr<f16>, #AL>
  // expected-remark @below {{scratch offset = 0, size = 8192}}
  %a1 = ttg.convert_layout %a1_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A_DOT>
  %a2_ = tt.load %b_ptr, %cst3, %cst4 : tensor<32x128x!tt.ptr<f16>, #AL>
  // expected-remark @below {{scratch offset = 0, size = 8192}}
  %a2 = ttg.convert_layout %a2_ : tensor<32x128xf16, #AL> -> tensor<32x128xf16, #B_DOT>
  %a3_ = tt.load %a_ptr, %cst1, %cst2 : tensor<128x32x!tt.ptr<f16>, #AL>
  // expected-remark @below {{scratch offset = 0, size = 8192}}
  %a3 = ttg.convert_layout %a3_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A_DOT>
  %c = tt.dot %a1, %a2, %c_init : tensor<128x32xf16, #A_DOT> * tensor<32x128xf16, #B_DOT> -> tensor<128x128xf32, #C>
  %a4_ = tt.load %b_ptr, %cst3, %cst4 : tensor<32x128x!tt.ptr<f16>, #AL>
  // expected-remark @below {{scratch offset = 0, size = 8192}}
  %a4 = ttg.convert_layout %a4_ : tensor<32x128xf16, #AL> -> tensor<32x128xf16, #B_DOT>
  %c1 = tt.dot %a3, %a4, %c : tensor<128x32xf16, #A_DOT> * tensor<32x128xf16, #B_DOT> -> tensor<128x128xf32, #C>
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises layout conversion, dot products / matmul, tensor loads. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 the test function(s)，并覆盖 布局转换、点积/矩阵乘、张量加载。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 111-127
```mlir
// A tensor's shared memory offset is larger than it needs to accommodate further tensors
// %cst0->%c
// %cst1->%cst4
// %cst3->%g->%h->%i
// expected-remark @below {{preallocate}}
// expected-remark @below {{size = 12288}}
tt.func @preallocate(%A : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 2048, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 3072, size = 512}}
  %cst1 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 3584, size = 512}}
  %cst2 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 1024}}
  %a = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 1024, size = 1024}}
  %b = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block defines `preallocate` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×7) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `preallocate`，并覆盖 共享内存分配。 期望诊断标注（remark×7）用于捕获 pass 应当发出的分析备注/说明。

### Lines 129-131
```mlir
  ttg.local_dealloc %cst0 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 2048, size = 1024}}
  %c = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Lines 133-134
```mlir
  ttg.local_dealloc %cst1 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst2 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block contributes intermediate IR built from `ttg.local_dealloc`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_dealloc`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 136-161
```mlir
  // expected-remark @below {{offset = 3072, size = 1024}}
  %cst4 = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 4096, size = 2048}}
  %e = ttg.local_alloc : () -> !ttg.memdesc<64x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %a : !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 6144, size = 2048}}
  %d = ttg.local_alloc : () -> !ttg.memdesc<64x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %b : !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 8192, size = 2048}}
  %f = ttg.local_alloc : () -> !ttg.memdesc<64x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst4 : !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %c : !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 10240, size = 2048}}
  %cst5 = ttg.local_alloc : () -> !ttg.memdesc<64x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 4096}}
  %g = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %e : !ttg.memdesc<64x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 4096}}
  %h = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %d : !ttg.memdesc<64x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 4096}}
  %i = ttg.local_alloc : () -> !ttg.memdesc<128x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %f : !ttg.memdesc<64x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst5 : !ttg.memdesc<64x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation. Expected-diagnostic annotations (remark×8) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配。 期望诊断标注（remark×8）用于捕获 pass 应当发出的分析备注/说明。

### Lines 163-173
```mlir
// expected-remark @below {{memdesc_ptr}}
// expected-remark @below {{size = 6144}}
tt.func @memdesc_ptr() {
  // expected-remark @below {{offset = 0, size = 4096}}
  %a0 = ttg.local_alloc : () -> !ttg.memdesc<32x16x!tt.ptr<f16>, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 4096, size = 2048}}
  %a1 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16x!tt.ptr<f16>, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %a0 : !ttg.memdesc<32x16x!tt.ptr<f16>, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %a1 : !ttg.memdesc<1x16x16x!tt.ptr<f16>, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `memdesc_ptr` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `memdesc_ptr`，并覆盖 共享内存分配。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 175-187
```mlir
// Unused tensors are immediately released
// expected-remark @below {{unused}}
// expected-remark @below {{size = 1024}}
tt.func @unused(%A : !tt.ptr<f16>) {
  %cst = arith.constant dense<0.000000e+00> : tensor<32x16xf16, #AL>
  // expected-remark @below {{0, size = 1024}}
  %cst0 = ttg.local_alloc %cst : (tensor<32x16xf16, #AL>) -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory>
  // expected-remark @below {{offset = 0, size = 512}}
  %cst1 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 512}}
  %cst2 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `unused` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `unused`，并覆盖 共享内存分配。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 189-202
```mlir
// cst0 is alive through the entire function, it cannot be released before the end of the function
// expected-remark @below {{longlive}}
// expected-remark @below {{size = 2560}}
tt.func @longlive(%A : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 2048, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 1024, size = 512}}
  %cst1 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 1536, size = 512}}
  %cst2 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 1024}}
  %a = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst1 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst2 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
```
**EN:** This block defines `longlive` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `longlive`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 204-222
```mlir
  // expected-remark @below {{offset = 1024, size = 512}}
  %cst3 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 1536, size = 512}}
  %cst4 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 1024}}
  %b = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 512}}
  %cst5 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 512}}
  %cst6 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 1024}}
  %c = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst3 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst4 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 1024}}
  %d = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst0 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises shared-memory allocation. Expected-diagnostic annotations (remark×7) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 the test function(s)，并覆盖 共享内存分配。 期望诊断标注（remark×7）用于捕获 pass 应当发出的分析备注/说明。

### Lines 224-269
```mlir
// This example triggers graph coloring with > 1 colors.
// expected-remark @below {{multi_color}}
// expected-remark @below {{size = 1376}}
tt.func @multi_color(%A : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 1024, size = 64}}
  %cst = ttg.local_alloc : () -> !ttg.memdesc<4x8xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 1344, size = 32}}
  %cst_0 = ttg.local_alloc : () -> !ttg.memdesc<4x4xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 1088, size = 128}}
  %cst_1 = ttg.local_alloc : () -> !ttg.memdesc<16x4xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %cst_2 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
  // expected-remark @below {{scratch offset = 0, size = 1024}}
  %0 = ttg.convert_layout %cst_2 : tensor<16x32xf16, #AL> -> tensor<16x32xf16, #BL>
  %1 = ttg.local_load %cst : !ttg.memdesc<4x8xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<4x8xf16, #AL>
  // expected-remark @below {{offset = 0, size = 128}}
  %cst_3 = ttg.local_alloc : () -> !ttg.memdesc<4x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %2 = ttg.local_load %cst_0 : !ttg.memdesc<4x4xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<4x4xf16, #AL>
  // expected-remark @below {{scratch offset = 0, size = 1024}}
  %3 = ttg.convert_layout %cst_2 : tensor<16x32xf16, #AL> -> tensor<16x32xf16, #BL>
  // expected-remark @below {{offset = 512, size = 256}}
  %cst_4 = ttg.local_alloc : () -> !ttg.memdesc<4x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 64}}
  %cst_5 = ttg.local_alloc : () -> !ttg.memdesc<4x8xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %4 = ttg.local_load %cst_5 : !ttg.memdesc<4x8xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<4x8xf16, #AL>
  %5 = ttg.local_load %cst_5 : !ttg.memdesc<4x8xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<4x8xf16, #AL>
  // expected-remark @below {{offset = 0, size = 512}}
  %cst_6 = ttg.local_alloc : () -> !ttg.memdesc<8x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 1216, size = 128}}
  %cst_7 = ttg.local_alloc : () -> !ttg.memdesc<2x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %6 = ttg.local_load %cst_0 : !ttg.memdesc<4x4xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<4x4xf16, #AL>
  // expected-remark @below {{offset = 0, size = 512}}
  %cst_8 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 32}}
  %cst_9 = ttg.local_alloc : () -> !ttg.memdesc<4x4xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 512}}
  %cst_10 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %7 = ttg.local_load %cst_1 : !ttg.memdesc<16x4xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x4xf16, #AL>
  %8 = ttg.local_load %cst_4 : !ttg.memdesc<4x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<4x32xf16, #AL>
  // expected-remark @below {{scratch offset = 0, size = 1024}}
  %9 = ttg.convert_layout %cst_2 : tensor<16x32xf16, #AL> -> tensor<16x32xf16, #BL>
  %cst_11 = arith.constant dense<0.000000e+00> : tensor<4x4xf16, #AL>
  %10 = ttg.local_load %cst_7 : !ttg.memdesc<2x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<2x32xf16, #AL>
  %cst_12 = arith.constant dense<0.000000e+00> : tensor<4x16xf16, #AL>
  %cst_13 = arith.constant dense<0.000000e+00> : tensor<8x32xf16, #AL>
  tt.return
}
```
**EN:** This block defines `multi_color` and exercises shared-memory allocation, layout conversion. Expected-diagnostic annotations (remark×16) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `multi_color`，并覆盖 共享内存分配、布局转换。 期望诊断标注（remark×16）用于捕获 pass 应当发出的分析备注/说明。

### Lines 271-296
```mlir
// This example triggers graph coloring with multiple rounds
// expected-remark @below {{multi_color_multi_rounds}}
// expected-remark @below {{size = 9376}}
tt.func @multi_color_multi_rounds(%arg0: !tt.ptr<f16>) {
  // expected-remark @below {{offset = 9344, size = 32}}
  %cst = ttg.local_alloc : () -> !ttg.memdesc<4x4xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 9216, size = 128}}
  %cst_0 = ttg.local_alloc : () -> !ttg.memdesc<16x4xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 8192}}
  %cst_1 = ttg.local_alloc : () -> !ttg.memdesc<1024x4xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %cst_2 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
  // expected-remark @below {{scratch offset = 8192, size = 1024}}
  %0 = ttg.convert_layout %cst_2 : tensor<16x32xf16, #AL> -> tensor<16x32xf16, #BL>
  %1 = ttg.local_load %cst : !ttg.memdesc<4x4xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<4x4xf16, #AL>
  // expected-remark @below {{offset = 8704, size = 128}}
  %cst_3 = ttg.local_alloc : () -> !ttg.memdesc<2x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %2 = ttg.local_load %cst : !ttg.memdesc<4x4xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<4x4xf16, #AL>
  // expected-remark @below {{offset = 8192, size = 512}}
  %cst_4 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %3 = ttg.local_load %cst_0 : !ttg.memdesc<16x4xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<16x4xf16, #AL>
  %4 = ttg.local_load %cst_1 : !ttg.memdesc<1024x4xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<1024x4xf16, #AL>
  // expected-remark @below {{scratch offset = 0, size = 1024}}
  %5 = ttg.convert_layout %cst_2 : tensor<16x32xf16, #AL> -> tensor<16x32xf16, #BL>
  %6 = ttg.local_load %cst_3 : !ttg.memdesc<2x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> tensor<2x32xf16, #AL>
  tt.return
}
```
**EN:** This block defines `multi_color_multi_rounds` and exercises shared-memory allocation, layout conversion. Expected-diagnostic annotations (remark×9) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `multi_color_multi_rounds`，并覆盖 共享内存分配、布局转换。 期望诊断标注（remark×9）用于捕获 pass 应当发出的分析备注/说明。

### Lines 299-308
```mlir
// expected-remark @below {{alloc_ptr}}
// expected-remark @below {{size = 512}}
tt.func @alloc_ptr(%A : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 0, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %cst1 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
  // expected-remark @below {{offset = 0, size = 512}}
  %cst2 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `alloc_ptr` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `alloc_ptr`，并覆盖 共享内存分配。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 311-320
```mlir
// expected-remark @below {{dealloc}}
// expected-remark @below {{size = 2048}}
tt.func @dealloc(%A : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 0, size = 1024}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 1024, size = 1024}}
  %cst1 = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst0 : !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `dealloc` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `dealloc`，并覆盖 共享内存分配。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 322-333
```mlir
// expected-remark @below {{scratch}}
// expected-remark @below {{size = 128}}
tt.func @scratch() {
  %cst0 = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  // expected-remark @below {{scratch offset = 0, size = 128}}
  %b = "tt.reduce" (%cst0) ({
  ^bb0(%arg0: f16, %arg1: f16):
    %add = arith.addf %arg0, %arg1 : f16
    tt.reduce.return %add : f16
  }) {axis = 0 : i32} : (tensor<16x16xf16, #AL>) -> tensor<16xf16, #sliceAd0>
  tt.return
}
```
**EN:** This block defines `scratch` and exercises operations such as `tt.func`, `arith.constant`, `tt.reduce`, `arith.addf`, `tt.reduce.return`. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `scratch`，并覆盖 如 `tt.func`、`arith.constant`、`tt.reduce`、`arith.addf`、`tt.reduce.return` 这样的操作。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Lines 335-342
```mlir
// expected-remark @below {{trans}}
// expected-remark @below {{size = 1024}}
tt.func @trans(%A : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 0, size = 1024}}
  %tensor = ttg.local_alloc : () -> !ttg.memdesc<16x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %b = ttg.memdesc_trans %tensor {order=array<i32: 1,0>} : !ttg.memdesc<16x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<32x16xf16, #A_SHARED_T, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `trans` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `trans`，并覆盖 共享内存分配。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Lines 345-353
```mlir
// expected-remark @below {{extract_slice}}
// expected-remark @below {{size = 512}}
tt.func @extract_slice(%A : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 0, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %index = arith.constant 0 : i32
  %cst1 = ttg.memdesc_index %cst0[%index] : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `extract_slice` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `extract_slice`，并覆盖 共享内存分配。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Lines 355-366
```mlir
// expected-remark @below {{atomic_scalar}}
// expected-remark @below {{size = 8196}}
tt.func @atomic_scalar(%arg3: !tt.ptr<i32>) -> i32 {
  %c0_i32 = arith.constant 0 : i32
  %1 = arith.constant dense<1.0> : tensor<128x32xf16, #AL>
  // expected-remark @below {{offset = 0, size = 8192}}
  %2 = ttg.local_alloc %1 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  // expected-remark @below {{scratch offset = 8192, size = 4}}
  %4 = tt.atomic_cas acq_rel, gpu, %arg3, %c0_i32, %c0_i32 : (!tt.ptr<i32>, i32, i32) -> i32
  %3 = ttg.local_load %2 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  tt.return %4 : i32
}
```
**EN:** This block defines `atomic_scalar` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `atomic_scalar`，并覆盖 共享内存分配。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 368-378
```mlir
// expected-remark @below {{atomic_scalar_no_use}}
// expected-remark @below {{size = 8192}}
tt.func @atomic_scalar_no_use(%arg3: !tt.ptr<i32>) {
  %c0_i32 = arith.constant 0 : i32
  %1 = arith.constant dense<1.0> : tensor<128x32xf16, #AL>
  // expected-remark @below {{offset = 0, size = 8192}}
  %2 = ttg.local_alloc %1 : (tensor<128x32xf16, #AL>) -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory>
  %4 = tt.atomic_cas acq_rel, gpu, %arg3, %c0_i32, %c0_i32 : (!tt.ptr<i32>, i32, i32) -> i32
  %3 = ttg.local_load %2 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory> -> tensor<128x32xf16, #AL>
  tt.return
}
```
**EN:** This block defines `atomic_scalar_no_use` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `atomic_scalar_no_use`，并覆盖 共享内存分配。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Lines 380-406
```mlir
// B0 -> (B1) -> B0
// Memory used by B1 can be reused by B0.
// expected-remark @below {{if}}
// expected-remark @below {{size = 2048}}
tt.func @if(%i1 : i1) {
  // expected-remark @below {{offset = 1024, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 1536, size = 512}}
  %cst1 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  scf.if %i1 {
    // expected-remark @below {{offset = 0, size = 1024}}
    %a = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    // expected-remark @below {{offset = 0, size = 1024}}
    %b = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    ttg.local_dealloc %cst0 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    ttg.local_dealloc %cst1 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  }
  // expected-remark @below {{offset = 1024, size = 512}}
  %cst2 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 1536, size = 512}}
  %cst3 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 1024}}
  %a = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst2 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst3 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `if` and exercises shared-memory allocation, control-flow joins. Expected-diagnostic annotations (remark×9) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `if`，并覆盖 共享内存分配、控制流汇合。 期望诊断标注（remark×9）用于捕获 pass 应当发出的分析备注/说明。

### Lines 408-437
```mlir
// B0 -> (B1) -> (B2) -> B0
// Memory used by B0 cannot be reused by B1 or B2.
// expected-remark @below {{if_else}}
// expected-remark @below {{size = 3072}}
tt.func @if_else(%i1 : i1) {
  // expected-remark @below {{offset = 1536, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 2048, size = 512}}
  %cst1 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  scf.if %i1 {
    // expected-remark @below {{offset = 0, size = 1024}}
    %a = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    // expected-remark @below {{offset = 0, size = 1024}}
    %b = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  } else {
    // expected-remark @below {{offset = 1024, size = 512}}
    %cst2 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    // expected-remark @below {{offset = 2560, size = 512}}
    %cst3 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    // expected-remark @below {{offset = 0, size = 1024}}
    %a = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    ttg.local_dealloc %cst2 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    ttg.local_dealloc %cst3 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  }
  // expected-remark @below {{offset = 0, size = 1024}}
  %a = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst0 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst1 : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `if_else` and exercises shared-memory allocation, control-flow joins. Expected-diagnostic annotations (remark×10) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `if_else`，并覆盖 共享内存分配、控制流汇合。 期望诊断标注（remark×10）用于捕获 pass 应当发出的分析备注/说明。

### Lines 439-455
```mlir
// Block arguments and yields are memory aliases that do not trigger a new
// allocation.
// expected-remark @below {{for}}
// expected-remark @below {{size = 24576}}
tt.func @for(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 0, size = 8192}}
  %a_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 8192, size = 8192}}
  %b_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 16384, size = 8192}}
  %c_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init, %c_shared = %c_shared_init) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>) {
    scf.yield %b_shared, %a_shared, %a_shared : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  }
  tt.return
  // CHECK-NEXT: size = 24576
}
```
**EN:** This block defines `for` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `for`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 457-476
```mlir
// expected-remark @below {{for_if_slice}}
// expected-remark @below {{size = 24576}}
tt.func @for_if_slice(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>, %i1 : i1) {
  // expected-remark @below {{offset = 0, size = 8192}}
  %a_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 8192, size = 8192}}
  %b_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 16384, size = 8192}}
  %c_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init, %c_shared = %c_shared_init) -> (!ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>) {
    scf.if %i1 {
      %zero = arith.constant 0 : i32
      %index = arith.constant 8 : i32
      %cst0 = ttg.memdesc_index %a_shared[%index] : !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable> -> !ttg.memdesc<32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
      scf.yield
    }
    scf.yield %b_shared, %a_shared, %a_shared : !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
  }
  tt.return
}
```
**EN:** This block defines `for_if_slice` and exercises shared-memory allocation, loop-carried state, control-flow joins. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `for_if_slice`，并覆盖 共享内存分配、循环携带状态、控制流汇合。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 478-495
```mlir
// c0 cannot be released in the loop
// expected-remark @below {{for_use_ancestor}}
// expected-remark @below {{size = 32768}}
tt.func @for_use_ancestor(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>, %i1 : i1) {
  // expected-remark @below {{offset = 0, size = 8192}}
  %a_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 8192, size = 8192}}
  %b_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 16384, size = 8192}}
  %c_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %a_shared, %b_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>) {
    %c0 = ttg.memdesc_trans %c_shared_init {order=array<i32: 1,0>} : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<32x128xf16, #A_SHARED_T, #ttg.shared_memory, mutable>
    // expected-remark @below {{offset = 24576, size = 8192}}
    %c1 = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
    scf.yield %b_shared, %a_shared: !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  }
  tt.return
}
```
**EN:** This block defines `for_use_ancestor` and exercises shared-memory allocation, loop-carried state. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `for_use_ancestor`，并覆盖 共享内存分配、循环携带状态。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 497-526
```mlir
// a_shared_init, b_shared_init, and c_shared_init's liveness ranges are span over the entire function before cst2.
// So they cannot be reused by cst0 and cst1, but can be reused by cst2.
// expected-remark @below {{for_for_if}}
// expected-remark @below {{size = 40960}}
tt.func @for_for_if(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>, %i1 : i1) {
  // expected-remark @below {{offset = 0, size = 8192}}
  %a_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 8192, size = 8192}}
  %b_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 16384, size = 8192}}
  %c_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init, %c_shared = %c_shared_init) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>) {
    %c_shared_next = scf.for %jv = %lb to %ub step %step iter_args(%c_shared_next = %c_shared) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>) {
      %c_shared_next_next = scf.if %i1 -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable> {
        // expected-remark @below {{offset = 24576, size = 8192}}
        %cst0 = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
        scf.yield %cst0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
      } else {
        // expected-remark @below {{offset = 32768, size = 8192}}
        %cst1 = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
        scf.yield %cst1 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
      }
      scf.yield %c_shared_next_next : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
    }
    scf.yield %a_shared, %b_shared, %c_shared_next : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  }
  // expected-remark @below {{offset = 0, size = 8192}}
  %cst2 = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `for_for_if` and exercises shared-memory allocation, loop-carried state, control-flow joins. Expected-diagnostic annotations (remark×8) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `for_for_if`，并覆盖 共享内存分配、循环携带状态、控制流汇合。 期望诊断标注（remark×8）用于捕获 pass 应当发出的分析备注/说明。

### Lines 528-534
```mlir
// expected-remark @below {{alloc1}}
// expected-remark @below {{size = 512}}
tt.func @alloc1(%A : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 0, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `alloc1` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `alloc1`，并覆盖 共享内存分配。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Lines 536-542
```mlir
// expected-remark @below {{alloc2}}
// expected-remark @below {{size = 1024}}
tt.func @alloc2(%A : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 0, size = 1024}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `alloc2` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `alloc2`，并覆盖 共享内存分配。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Lines 544-555
```mlir
// expected-remark @below {{alloc3}}
// expected-remark @below {{size = 1024}}
tt.func @alloc3(%cond : i1) {
  scf.if %cond {
    // expected-remark @below {{offset = 0, size = 512}}
    %cst0 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  } else {
    // expected-remark @below {{offset = 0, size = 1024}}
    %cst0 = ttg.local_alloc : () -> !ttg.memdesc<16x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  }
  tt.return
}
```
**EN:** This block defines `alloc3` and exercises shared-memory allocation, control-flow joins. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `alloc3`，并覆盖 共享内存分配、控制流汇合。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 557-568
```mlir
// expected-remark @below {{alloc4}}
// expected-remark @below {{size = 1024}}
tt.func @alloc4(%A : !tt.ptr<f16>, %cond : i1) {
  scf.if %cond {
    // expected-remark @below {{virtual offset = 0, size = 1024}}
    tt.call @alloc3(%cond) : (i1) -> ()
  } else {
    // expected-remark @below {{virtual offset = 0, size = 512}}
    tt.call @alloc1(%A) : (!tt.ptr<f16>) -> ()
  }
  tt.return
}
```
**EN:** This block defines `alloc4`, `alloc3`, `alloc1` and exercises control-flow joins. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `alloc4`, `alloc3`, `alloc1`，并覆盖 控制流汇合。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 570-579
```mlir
// expected-remark @below {{single_call}}
// expected-remark @below {{size = 512}}
tt.func @single_call(%A : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 0, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %cst1 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
  // expected-remark @below {{virtual offset = 0, size = 512}}
  tt.call @alloc1(%A) : (!tt.ptr<f16>) -> ()
  tt.return
}
```
**EN:** This block defines `single_call`, `alloc1` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `single_call`, `alloc1`，并覆盖 共享内存分配。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 581-592
```mlir
// expected-remark @below {{multiple_calls}}
// expected-remark @below {{size = 1024}}
tt.func @multiple_calls(%A : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 0, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{virtual offset = 0, size = 512}}
  tt.call @alloc1(%A) : (!tt.ptr<f16>) -> ()
  %cst1 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
  // expected-remark @below {{virtual offset = 0, size = 1024}}
  tt.call @alloc2(%A) : (!tt.ptr<f16>) -> ()
  tt.return
}
```
**EN:** This block defines `multiple_calls`, `alloc1`, `alloc2` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `multiple_calls`, `alloc1`, `alloc2`，并覆盖 共享内存分配。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 594-611
```mlir
// expected-remark @below {{if_else_calls}}
// expected-remark @below {{size = 1024}}
tt.func @if_else_calls(%A : !tt.ptr<f16>, %cond : i1) {
  %cst = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
  scf.if %cond {
    // expected-remark @below {{offset = 0, size = 512}}
    %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
    // expected-remark @below {{offset = 0, size = 1024}}
    %cst1 = ttg.local_alloc %cst : (tensor<16x32xf16, #AL>) -> !ttg.memdesc<16x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
    // expected-remark @below {{virtual offset = 0, size = 512}}
    tt.call @alloc1(%A) : (!tt.ptr<f16>) -> ()
  } else {
    %cst0 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
    // expected-remark @below {{virtual offset = 0, size = 1024}}
    tt.call @alloc2(%A) : (!tt.ptr<f16>) -> ()
  }
  tt.return
}
```
**EN:** This block defines `if_else_calls`, `alloc1`, `alloc2` and exercises shared-memory allocation, control-flow joins. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `if_else_calls`, `alloc1`, `alloc2`，并覆盖 共享内存分配、控制流汇合。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 613-628
```mlir
// expected-remark @below {{for_calls}}
// expected-remark @below {{size = 512}}
tt.func @for_calls(%A : !tt.ptr<f16>, %cond : i1) {
  // expected-remark @below {{offset = 0, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %cst1 = arith.constant dense<0.000000e+00> : tensor<16x32xf16, #AL>
  %lb = arith.constant 0 : index
  %ub = arith.constant 10 : index
  %step = arith.constant 1 : index
  scf.for %iv = %lb to %ub step %step {
    // expected-remark @below {{virtual offset = 0, size = 512}}
    tt.call @alloc1(%A) : (!tt.ptr<f16>) -> ()
  }
  tt.return
  // CHECK-NEXT: size = 512
}
```
**EN:** This block defines `for_calls`, `alloc1` and exercises shared-memory allocation, loop-carried state. Embedded check comments (CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `for_calls`, `alloc1`，并覆盖 共享内存分配、循环携带状态。 其中嵌入的检查注释（CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 630-638
```mlir
// expected-remark @below {{call_graph_1}}
// expected-remark @below {{size = 1024}}
tt.func @call_graph_1(%A : !tt.ptr<f16>, %cond : i1) {
  // expected-remark @below {{offset = 0, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{virtual offset = 0, size = 1024}}
  tt.call @alloc3(%cond) : (i1) -> ()
  tt.return
}
```
**EN:** This block defines `call_graph_1`, `alloc3` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `call_graph_1`, `alloc3`，并覆盖 共享内存分配。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 640-648
```mlir
// expected-remark @below {{call_graph_2}}
// expected-remark @below {{size = 1024}}
tt.func @call_graph_2(%A : !tt.ptr<f16>, %cond : i1) {
  // expected-remark @below {{offset = 0, size = 512}}
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{virtual offset = 0, size = 1024}}
  tt.call @alloc4(%A, %cond) : (!tt.ptr<f16>, i1) -> ()
  tt.return
}
```
**EN:** This block defines `call_graph_2`, `alloc4` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `call_graph_2`, `alloc4`，并覆盖 共享内存分配。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 650-660
```mlir
// expected-remark @below {{scan_alloc}}
// expected-remark @below {{size = 128}}
tt.func @scan_alloc(%x : tensor<8x16xf32, #AL>) {
  // expected-remark @below {{offset = 0, size = 128}}
  %a = "tt.scan"(%x) <{axis = 0 : i32, reverse = false}>({
  ^bb0(%arg0: f32, %arg1: f32):
    %add = arith.addf %arg0, %arg1 : f32
    tt.scan.return %add : f32
  }) : (tensor<8x16xf32, #AL>) -> tensor<8x16xf32, #AL>
  tt.return
}
```
**EN:** This block defines `scan_alloc` and exercises operations such as `tt.func`, `tt.scan`, `arith.addf`, `tt.scan.return`, `tt.return`. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `scan_alloc`，并覆盖 如 `tt.func`、`tt.scan`、`arith.addf`、`tt.scan.return`、`tt.return` 这样的操作。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Lines 662-677
```mlir
// expected-remark @below {{warp_specialize_default_region}}
// expected-remark @below {{size = 33}}
// expected-remark @below {{offset = 32, size = 1}}
tt.func @warp_specialize_default_region() {
  // expected-remark @below {{offset = 0, size = 16}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  ttg.warp_specialize()
  default {
    // expected-remark @below {{offset = 16, size = 16}}
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    ttg.warp_yield
  }
  partition0() num_warps(1) {
    ttg.warp_return
  } : () -> ()
  "use"(%0) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
```
**EN:** This block defines `warp_specialize_default_region` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `warp_specialize_default_region`，并覆盖 共享内存分配。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 679-680
```mlir
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 682-701
```mlir
// expected-remark @below {{nonoverlapping_liveness_in_default_region}}
// expected-remark @below {{size = 33}}
// expected-remark @below {{offset = 32, size = 1}}
tt.func @nonoverlapping_liveness_in_default_region() {
  // expected-remark @below {{offset = 0, size = 16}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  ttg.warp_specialize()
  default {
    // expected-remark @below {{offset = 16, size = 16}}
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    "use"(%1) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    // expected-remark @below {{offset = 16, size = 16}}
    %2 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    "use"(%2) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    ttg.warp_yield
  }
  partition0() num_warps(1) {
    ttg.warp_return
  } : () -> ()
  "use"(%0) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
```
**EN:** This block defines `nonoverlapping_liveness_in_default_region` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `nonoverlapping_liveness_in_default_region`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 703-704
```mlir
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 706-725
```mlir
// expected-remark @below {{overlapping_liveness_in_default_region}}
// expected-remark @below {{size = 49}}
// expected-remark @below {{offset = 48, size = 1}}
tt.func @overlapping_liveness_in_default_region() {
  // expected-remark @below {{offset = 0, size = 16}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  ttg.warp_specialize()
  default {
    // expected-remark @below {{offset = 16, size = 16}}
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    // expected-remark @below {{offset = 32, size = 16}}
    %2 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    "use"(%1) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    "use"(%2) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    ttg.warp_yield
  }
  partition0() num_warps(1) {
    ttg.warp_return
  } : () -> ()
  "use"(%0) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
```
**EN:** This block defines `overlapping_liveness_in_default_region` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `overlapping_liveness_in_default_region`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 727-728
```mlir
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 730-747
```mlir
// expected-remark @below {{alias_through_default_outputs}}
// expected-remark @below {{size = 33}}
// expected-remark @below {{offset = 32, size = 1}}
tt.func @alias_through_default_outputs() {
  // expected-remark @below {{offset = 0, size = 16}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  %1 = ttg.warp_specialize()
  default {
    ttg.warp_yield %0 : !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  }
  partition0() num_warps(1) {
    ttg.warp_return
  } : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  // expected-remark @below {{offset = 16, size = 16}}
  %2 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  "use"(%1) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
  tt.return
}
```
**EN:** This block defines `alias_through_default_outputs` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `alias_through_default_outputs`，并覆盖 共享内存分配。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 749-766
```mlir
// expected-remark @below {{implicit_capture_liveness}}
// expected-remark @below {{size = 33}}
// expected-remark @below {{offset = 32, size = 1}}
tt.func @implicit_capture_liveness() {
  // expected-remark @below {{offset = 0, size = 16}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  ttg.warp_specialize()
  default {
    // expected-remark @below {{offset = 16, size = 16}}
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    "use"(%0) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    ttg.warp_yield
  }
  partition0() num_warps(1) {
    ttg.warp_return
  } : () -> ()
  tt.return
}
```
**EN:** This block defines `implicit_capture_liveness` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `implicit_capture_liveness`，并覆盖 共享内存分配。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 768-786
```mlir
// expected-remark @below {{implicit_and_explicit_capture_liveness}}
// expected-remark @below {{size = 45}}
// expected-remark @below {{offset = 44, size = 1}}
tt.func @implicit_and_explicit_capture_liveness() {
  // expected-remark @below {{offset = 0, size = 16}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  // expected-remark @below {{offset = 16, size = 16}}
  %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  // expected-remark @below {{offset = 32, size = 12}}
  ttg.warp_specialize(%1)
  default {
    "use"(%0) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    ttg.warp_yield
  }
  partition0(%arg0: !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) num_warps(1) {
    ttg.warp_return
  } : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
  tt.return
}
```
**EN:** This block defines `implicit_and_explicit_capture_liveness` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `implicit_and_explicit_capture_liveness`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 788-805
```mlir
// expected-remark @below {{explicit_capture_liveness}}
// expected-remark @below {{size = 33}}
// expected-remark @below {{offset = 32, size = 1}}
tt.func @explicit_capture_liveness() {
  // expected-remark @below {{offset = 0, size = 16}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  // expected-remark @below {{offset = 16, size = 12}}
  ttg.warp_specialize(%0)
  default {
    // expected-remark @below {{offset = 16, size = 16}}
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    ttg.warp_yield
  }
  partition0(%arg0: !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) num_warps(1) {
    ttg.warp_return
  } : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
  tt.return
}
```
**EN:** This block defines `explicit_capture_liveness` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `explicit_capture_liveness`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 807-826
```mlir
// expected-remark @below {{implicit_capture_liveness_default}}
// expected-remark @below {{size = 33}}
// expected-remark @below {{offset = 32, size = 1}}
tt.func @implicit_capture_liveness_default() {
  // expected-remark @below {{offset = 0, size = 16}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  ttg.warp_specialize()
  default {
    // FIXME: This is correct, but not optimal. The memory for `%0` should be
    // reused for the next allocation. The same problem happens with `scf.if`.
    "use"(%0) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    // expected-remark @below {{offset = 16, size = 16}}
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    ttg.warp_yield
  }
  partition0() num_warps(1) {
    ttg.warp_return
  } : () -> ()
  tt.return
}
```
**EN:** This block defines `implicit_capture_liveness_default` and exercises shared-memory allocation, control-flow joins. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `implicit_capture_liveness_default`，并覆盖 共享内存分配、控制流汇合。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 828-845
```mlir
// expected-remark @below {{liveness_in_partition}}
// expected-remark @below {{size = 36}}
// expected-remark @below {{offset = 32, size = 4}}
tt.func @liveness_in_partition() {
  ttg.warp_specialize()
  default {
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    // expected-remark @below {{offset = 0, size = 16}}
    %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    // expected-remark @below {{offset = 16, size = 16}}
    %2 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    "use"(%0) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    ttg.warp_return
  } : () -> ()
  tt.return
}
```
**EN:** This block defines `liveness_in_partition` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `liveness_in_partition`，并覆盖 共享内存分配。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 847-866
```mlir
// expected-remark @below {{aliasing_in_partition}}
// expected-remark @below {{size = 36}}
// expected-remark @below {{offset = 32, size = 4}}
tt.func @aliasing_in_partition() {
  ttg.warp_specialize()
  default {
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    // expected-remark @below {{offset = 0, size = 16}}
    %0 = ttg.local_alloc : () -> !ttg.memdesc<2x1xi64, #A_SHARED_1D, #smem, mutable>
    %c0_i32 = arith.constant 0 : i32
    %1 = ttg.memdesc_index %0[%c0_i32] : !ttg.memdesc<2x1xi64, #A_SHARED_1D, #smem, mutable> -> !ttg.memdesc<1xi64, #A_SHARED_1D, #smem, mutable>
    // expected-remark @below {{offset = 16, size = 16}}
    %2 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    "use"(%1) : (!ttg.memdesc<1xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    ttg.warp_return
  } : () -> ()
  tt.return
}
```
**EN:** This block defines `aliasing_in_partition` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `aliasing_in_partition`，并覆盖 共享内存分配。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 868-897
```mlir
// expected-remark @below {{partition_region_interference}}
// expected-remark @below {{size = 88}}
// expected-remark @below {{offset = 80, size = 8}}
tt.func @partition_region_interference() {
  // expected-remark @below {{offset = 0, size = 16}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  ttg.warp_specialize()
  default {
    // expected-remark @below {{offset = 16, size = 16}}
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    // expected-remark @below {{offset = 32, size = 16}}
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    // expected-remark @below {{offset = 48, size = 16}}
    %2 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    "use"(%1) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    ttg.warp_return
  }
  partition1() num_warps(4) {
    // expected-remark @below {{offset = 64, size = 16}}
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    // expected-remark @below {{offset = 64, size = 16}}
    %2 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    ttg.warp_return
  } : () -> ()
  "use"(%0) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
  tt.return
}
```
**EN:** This block defines `partition_region_interference` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×9) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `partition_region_interference`，并覆盖 共享内存分配。 期望诊断标注（remark×9）用于捕获 pass 应当发出的分析备注/说明。

### Lines 899-922
```mlir
// expected-remark @below {{two_different_ws}}
// expected-remark @below {{size = 17}}
// expected-remark @below {{offset = 16, size = 1}}
tt.func @two_different_ws() {
  ttg.warp_specialize()
  default {
    // expected-remark @below {{offset = 0, size = 16}}
    ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    ttg.warp_yield
  }
  partition0() num_warps(1) {
    ttg.warp_return
  } : () -> ()
  ttg.warp_specialize()
  default {
    ttg.warp_yield
  }
  partition0() num_warps(1) {
    // expected-remark @below {{offset = 0, size = 16}}
    ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    ttg.warp_return
  } : () -> ()
  tt.return
}
```
**EN:** This block defines `two_different_ws` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `two_different_ws`，并覆盖 共享内存分配。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 924-945
```mlir
// expected-remark @below {{default_partition_outside_alloc_interference}}
// expected-remark @below {{size = 36}}
// expected-remark @below {{offset = 32, size = 4}}
tt.func @default_partition_outside_alloc_interference() {
  // expected-remark @below {{offset = 0, size = 16}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  // expected-remark @below {{offset = 16, size = 12}}
  ttg.warp_specialize(%0)
  default {
    // Ensure that we do not reuse the memory for %0 even though we are done
    // with it in this partition.
    // expected-remark @below {{offset = 16, size = 16}}
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    "use"(%1) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    ttg.warp_yield
  }
  partition0(%arg0: !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) num_warps(4) {
    "use"(%arg0) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    ttg.warp_return
  } : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
  tt.return
}
```
**EN:** This block defines `default_partition_outside_alloc_interference` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `default_partition_outside_alloc_interference`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 947-972
```mlir
// expected-remark @below {{partition_outside_alloc_interference}}
// expected-remark @below {{size = 36}}
// expected-remark @below {{offset = 32, size = 4}}
tt.func @partition_outside_alloc_interference() {
  // expected-remark @below {{offset = 0, size = 16}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
  // expected-remark @below {{offset = 16, size = 12}}
  ttg.warp_specialize(%0)
  default {
    ttg.warp_yield
  }
  partition0(%arg0: !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) num_warps(2) {
    "use"(%arg0) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    ttg.warp_return
  }
  partition1(%arg1: !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) num_warps(2) {
    "use"(%arg1) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    // Ensure that we do not reuse the memory for %0 even though we are done
    // with it in this partition.
    // expected-remark @below {{offset = 16, size = 16}}
    %1 = ttg.local_alloc : () -> !ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>
    "use"(%1) : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
    ttg.warp_return
  } : (!ttg.memdesc<2xi64, #A_SHARED_1D, #smem, mutable>) -> ()
  tt.return
}
```
**EN:** This block defines `partition_outside_alloc_interference` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `partition_outside_alloc_interference`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 974-983
```mlir
// expected-remark @below {{ptr_allocation_datalayout}}
// expected-remark @below {{size = 8}}
tt.func @ptr_allocation_datalayout(%arg0: !tt.ptr<i32>) {
  // expected-remark @below {{offset = 0, size = 8}}
  ttg.warp_specialize(%arg0)
  default {
    ttg.warp_yield
  } : (!tt.ptr<i32>) -> ()
  tt.return
}
```
**EN:** This block defines `ptr_allocation_datalayout` and exercises operations such as `tt.func`, `tt.ptr`, `ttg.warp_specialize`, `ttg.warp_yield`, `tt.return`. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `ptr_allocation_datalayout`，并覆盖 如 `tt.func`、`tt.ptr`、`ttg.warp_specialize`、`ttg.warp_yield`、`tt.return` 这样的操作。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Lines 985-1009
```mlir
// expected-remark @below {{tightly_packed_captures}}
// expected-remark @below {{size = 9}}
tt.func @tightly_packed_captures(%arg0: i8, %arg1: i64) {
  // expected-remark @below {{offset = 0, size = 9}}
  ttg.warp_specialize(%arg0, %arg1)
  default {
    ttg.warp_yield
  } : (i8, i64) -> ()
  tt.return
}
// expected-remark @below {{nvmma_alignment}}
// expected-remark @below {{size = 1088}}
tt.func @nvmma_alignment(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>) {
  // expected-remark @below {{offset = 0, size = 256}}
  %fp4 = ttg.local_alloc : () -> !ttg.memdesc<1x128xi8, #NVMMA_SHARED_FP4PADDED, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 0, size = 64}}
  %a = ttg.local_alloc : () -> !ttg.memdesc<32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 128, size = 64}}
  %b = ttg.local_alloc : () -> !ttg.memdesc<8x8xi8, #NVMMA_SHARED_0, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 256, size = 64}}
  %c = ttg.local_alloc : () -> !ttg.memdesc<4x16xi8, #NVMMA_SHARED_32, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 512, size = 64}}
  %d = ttg.local_alloc : () -> !ttg.memdesc<2x32xi8, #NVMMA_SHARED_64, #ttg.shared_memory, mutable>
  // expected-remark @below {{offset = 1024, size = 64}}
  %e = ttg.local_alloc : () -> !ttg.memdesc<1x64xi8, #NVMMA_SHARED_128, #ttg.shared_memory, mutable>
```
**EN:** This block defines `tightly_packed_captures`, `nvmma_alignment` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×11) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `tightly_packed_captures`, `nvmma_alignment`，并覆盖 共享内存分配。 期望诊断标注（remark×11）用于捕获 pass 应当发出的分析备注/说明。

### Lines 1011-1013
```mlir
  ttg.local_dealloc %a : !ttg.memdesc<32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `ttg.local_dealloc`, `ttg.memdesc`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_dealloc`, `ttg.memdesc`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 1016-1032
```mlir
// expected-remark @below {{padded_shared_layout_size}}
// expected-remark @below {{size = 1040}}
tt.func @padded_shared_layout_size() {
  // expected-remark @+2 {{offset = 0, size = 512}}
  // 256 * 2B = 512B
  %alloc0 = ttg.local_alloc : () -> !ttg.memdesc<1x256xf16, #PADDED_SHARED_0_1x256, #ttg.shared_memory, mutable>
  // expected-remark @+2 {{offset = 0, size = 1040}}
  // (512 + 8 * 1) * 2B = 1040B
  %alloc4 = ttg.local_alloc : () -> !ttg.memdesc<1x512xf16, #PADDED_SHARED_0_1x512, #ttg.shared_memory, mutable>
  // expected-remark @+2 {{offset = 0, size = 512}}
  // 16 * 16 * 2B = 512B
  %alloc6 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #PADDED_SHARED_0_16x16, #ttg.shared_memory, mutable>
  // expected-remark @+2 {{offset = 0, size = 1040}}
  // (16 * 32 + 8 * 1) * 2B = 1040B
  %alloc7 = ttg.local_alloc : () -> !ttg.memdesc<16x32xf16, #PADDED_SHARED_0_16x32, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `padded_shared_layout_size` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `padded_shared_layout_size`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 1034-1047
```mlir
// expected-remark @below {{padded_shared_layout_element_type}}
// expected-remark @below {{size = 2080}}
tt.func @padded_shared_layout_element_type() {
  // expected-remark @+2 {{offset = 0, size = 520}}
  // (16 * 32 + 8 * 1) * 1B = 520B
  %alloc0 = ttg.local_alloc : () -> !ttg.memdesc<16x32xi8, #PADDED_SHARED_0_16x32, #ttg.shared_memory, mutable>
  // expected-remark @+2 {{offset = 0, size = 1040}}
  // (16 * 256 + 8 * 15) * 2B = 1040B
  %alloc1 = ttg.local_alloc : () -> !ttg.memdesc<16x32xf16, #PADDED_SHARED_0_16x32, #ttg.shared_memory, mutable>
  // expected-remark @+2 {{offset = 0, size = 2080}}
  // (16 * 256 + 8 * 15) * 4B = 2080B
  %alloc2 = ttg.local_alloc : () -> !ttg.memdesc<16x32xf32, #PADDED_SHARED_0_16x32, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `padded_shared_layout_element_type` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `padded_shared_layout_element_type`，并覆盖 共享内存分配。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 1049-1059
```mlir
// expected-remark @below {{padded_shared_layout_multi_tier}}
// expected-remark @below {{size = 4466}}
tt.func @padded_shared_layout_multi_tier() {
  // expected-remark @+2 {{offset = 0, size = 4340}}
  // (16 * 256 + 4 * 31 + 8 * 15) * 1B = 4340B
  %alloc0 = ttg.local_alloc : () -> !ttg.memdesc<16x256xi8, #PADDED_SHARED_1_16x256, #ttg.shared_memory, mutable>
  // expected-remark @+2 {{offset = 0, size = 4466}}
  // (16 * 256 + 2 * 63 + 4 * 31 + 8 * 15) * 1B = 4466B
  %alloc1 = ttg.local_alloc : () -> !ttg.memdesc<16x256xi8, #PADDED_SHARED_2_16x256, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `padded_shared_layout_multi_tier` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `padded_shared_layout_multi_tier`，并覆盖 共享内存分配。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 1061-1071
```mlir
// PartitionedSharedEncoding with swizzled inner layout: 64x32xf16, 2 partitions, 2 groups each
// Without partition-size, pieces are placed consecutively
// expected-remark @below {{partitioned_shared_swizzle_alloc}}
// expected-remark @below {{size = 4096}}
tt.func @partitioned_shared_swizzle_alloc() {
  // 2 partition buffers, each containing 2 groups = 2048 bytes each
  // expected-remark @below {{offset = 0, size = 2048}}
  // expected-remark @below {{offset = 2048, size = 2048}}
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #PARTITIONED_SHARED_SWIZZLE, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `partitioned_shared_swizzle_alloc` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `partitioned_shared_swizzle_alloc`，并覆盖 共享内存分配。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 1073-1086
```mlir
// PartitionedSharedEncoding with padded inner layout: 64x32xf16, 4 partitions, 1 group each
// Without partition-size, pieces are placed consecutively
// expected-remark @below {{partitioned_shared_padded_alloc}}
// expected-remark @below {{size = 4220}}
tt.func @partitioned_shared_padded_alloc() {
  // 4 partition buffers, each containing 1 group = 1052 bytes each
  // expected-remark @below {{offset = 0, size = 1052}}
  // expected-remark @below {{offset = 1056, size = 1052}}
  // expected-remark @below {{offset = 2112, size = 1052}}
  // expected-remark @below {{offset = 3168, size = 1052}}
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #PARTITIONED_SHARED_PADDED, #ttg.shared_memory, mutable>
  tt.return
}
}
```
**EN:** This block defines `partitioned_shared_padded_alloc` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `partitioned_shared_padded_alloc`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's analysis coverage and focuses on allocation behavior.  
  **CN:** 该文件属于 Triton 的 分析 测试覆盖，关注点是 分配行为。
- **EN:** The `RUN` pipeline drives tools/passes such as `-allow-unregistered-dialect`, `-test-print-allocation`, `-test-print-allocation="get-scratch-size-function=ValidConstant"`, `--check-prefix=CHECK-128`.  
  **CN:** `RUN` 流水线会驱动 `-allow-unregistered-dialect`, `-test-print-allocation`, `-test-print-allocation="get-scratch-size-function=ValidConstant"`, `--check-prefix=CHECK-128` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `scf` (structured control flow).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`scf`（结构化控制流）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。
- **EN:** Expected remarks/notes are used to pin down the exact analysis output produced by the pass.  
  **CN:** 这里的期望备注/说明用于固定 pass 产生的精确分析输出。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `scf`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`scf`。
