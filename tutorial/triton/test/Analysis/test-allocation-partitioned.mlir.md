# test-allocation-partitioned.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Analysis/test-allocation-partitioned.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises allocation behavior in Triton's analysis coverage. **CN:** 该MLIR 测试用于覆盖 Triton 分析中的分配行为相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -allow-unregistered-dialect -test-print-allocation="partition-size=65536" -verify-diagnostics -o /dev/null` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: none; diagnostics annotations: remark×37. **CN:** FileCheck 标记为 none；诊断标注为 remark×37。
- **Validation / 验证内容:** **EN:** The file mainly validates that allocation behavior emits the expected analysis remarks/notes. **CN:** 该文件主要验证 分配行为 会发出预期的分析备注/说明。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -allow-unregistered-dialect -test-print-allocation="partition-size=65536" -verify-diagnostics -o /dev/null
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -allow-unregistered-dialect -test-print-allocation="partition-size=65536" -verify-diagnostics -o /dev/null` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -allow-unregistered-dialect -test-print-allocation="partition-size=65536" -verify-diagnostics -o /dev/null`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-4
```mlir
// Test allocation with shared memory partitioning enabled (64KB partition size like for AMD GFX1250)
// With partition-size=65536, partition buffers should be placed in different 64KB physical partitions
```
**EN:** This comment block provides context for the surrounding test logic: `// Test allocation with shared memory partitioning enabled (64KB partition size like for AMD GFX1250`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Test allocation with shared memory partitioning enabled (64KB partition size like for AMD GFX1250`。

### Lines 6-7
```mlir
#A_SHARED = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#PADDED_SHARED = #ttg.padded_shared<[256:+8] {order = [1, 0], shape = [16, 32]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#A_SHARED`, `#PADDED_SHARED`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.padded_shared`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#A_SHARED`, `#PADDED_SHARED`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.padded_shared`。

### Lines 9-11
```mlir
// 2 partitions, 2 groups each
// Each piece is 1052 bytes, so each partition buffer is 1052 * 2 = 2104 bytes
#PARTITIONED_2P_2G = #ttg.partitioned_shared<{numPartitions = 2, numGroups = 2, partitionDim = 0, partitionLayout = #PADDED_SHARED}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 13-15
```mlir
// 4 partitions, 1 group each
// Each piece is 1052 bytes, each partition buffer is 1052 * 1 = 1052 bytes
#PARTITIONED_4P_1G = #ttg.partitioned_shared<{numPartitions = 4, numGroups = 1, partitionDim = 0, partitionLayout = #PADDED_SHARED}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 17-19
```mlir
// 2 partitions, 4 groups each (using swizzled layout)
// 64x32xf16 = 4096 bytes total, 8 pieces = 512 bytes each, partition buffer = 512 * 4 = 2048 bytes
#PARTITIONED_2P_4G = #ttg.partitioned_shared<{numPartitions = 2, numGroups = 4, partitionDim = 0, partitionLayout = #A_SHARED}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 21-23
```mlir
// 4 partitions, 2 groups each (using swizzled layout)
// 64x32xf16 = 4096 bytes total, 8 pieces = 512 bytes each, partition buffer = 512 * 2 = 1024 bytes
#PARTITIONED_4P_2G = #ttg.partitioned_shared<{numPartitions = 4, numGroups = 2, partitionDim = 0, partitionLayout = #A_SHARED}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 25
```mlir
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#smem`. They parameterize later tests with compact names for `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.shared_memory`。

### Line 27
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.num-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.num-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 29-40
```mlir
// Test basic 2 partitions, 2 groups allocation
// expected-remark @below {{partitioned_2p_2g}}
// expected-remark @below {{size = 67640}}
tt.func @partitioned_2p_2g() {
  // 2 partition buffers: one for partition 0 (contains groups 0,1) and one for partition 1 (contains groups 0,1)
  // Each partition buffer is 2104 bytes (1052 bytes per piece * 2 groups)
  // With 64KB partition size, the two partition buffers are placed in different 64KB physical partitions
  // expected-remark @below {{offset = 0, size = 2104}}
  // expected-remark @below {{offset = 65536, size = 2104}}
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #PARTITIONED_2P_2G, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `partitioned_2p_2g` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `partitioned_2p_2g`，并覆盖 共享内存分配。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 42-54
```mlir
// Test 4 partitions, 1 group each - each partition is a neighbor to all others
// expected-remark @below {{partitioned_4p_1g}}
// expected-remark @below {{size = 197660}}
tt.func @partitioned_4p_1g() {
  // 4 partition buffers, each containing 1 group = 1052 bytes each
  // All 4 partitions are neighbors, so they must be in different 64KB physical partitions
  // expected-remark @below {{offset = 0, size = 1052}}
  // expected-remark @below {{offset = 65536, size = 1052}}
  // expected-remark @below {{offset = 131072, size = 1052}}
  // expected-remark @below {{offset = 196608, size = 1052}}
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #PARTITIONED_4P_1G, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `partitioned_4p_1g` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `partitioned_4p_1g`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 56-66
```mlir
// Test 2 partitions, 4 groups each with swizzled layout
// expected-remark @below {{partitioned_2p_4g}}
// expected-remark @below {{size = 67584}}
tt.func @partitioned_2p_4g() {
  // 2 partition buffers, each containing 4 groups concatenated = 2048 bytes each
  // With 64KB partition size, the two partition buffers are placed in different 64KB physical partitions
  // expected-remark @below {{offset = 0, size = 2048}}
  // expected-remark @below {{offset = 65536, size = 2048}}
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #PARTITIONED_2P_4G, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `partitioned_2p_4g` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `partitioned_2p_4g`，并覆盖 共享内存分配。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Lines 68-80
```mlir
// Test 4 partitions, 2 groups each - four partitions need four different physical partitions
// expected-remark @below {{partitioned_4p_2g}}
// expected-remark @below {{size = 197632}}
tt.func @partitioned_4p_2g() {
  // 4 partition buffers, each containing 2 groups concatenated = 1024 bytes each
  // All 4 partitions are neighbors, so they must be in different 64KB physical partitions
  // expected-remark @below {{offset = 0, size = 1024}}
  // expected-remark @below {{offset = 65536, size = 1024}}
  // expected-remark @below {{offset = 131072, size = 1024}}
  // expected-remark @below {{offset = 196608, size = 1024}}
  %alloc = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #PARTITIONED_4P_2G, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `partitioned_4p_2g` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `partitioned_4p_2g`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 82-97
```mlir
// Test partitioned allocation alongside non-partitioned allocation
// expected-remark @below {{partitioned_with_regular}}
// expected-remark @below {{size = 67640}}
tt.func @partitioned_with_regular() {
  // Non-partitioned allocation: 1024 bytes (placed after the partitioned buffer in partition 0)
  // expected-remark @below {{offset = 4224, size = 1024}}
  %regular = ttg.local_alloc : () -> !ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // Partitioned allocation: 2 partition buffers of 2104 bytes each
  // expected-remark @below {{offset = 0, size = 2104}}
  // expected-remark @below {{offset = 65536, size = 2104}}
  %partitioned = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #PARTITIONED_2P_2G, #ttg.shared_memory, mutable>
  // Use both allocations so they overlap in liveness
  "use"(%regular) : (!ttg.memdesc<32x16xf16, #A_SHARED, #ttg.shared_memory, mutable>) -> ()
  "use"(%partitioned) : (!ttg.memdesc<64x32xf16, #PARTITIONED_2P_2G, #ttg.shared_memory, mutable>) -> ()
  tt.return
}
```
**EN:** This block defines `partitioned_with_regular` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `partitioned_with_regular`，并覆盖 共享内存分配。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Lines 99-116
```mlir
// Test multiple partitioned allocations (both live at the same time)
// expected-remark @below {{multiple_partitioned}}
// expected-remark @below {{size = 69696}}
tt.func @multiple_partitioned() {
  // First partitioned allocation: 2 partition buffers of 2104 bytes each
  // expected-remark @below {{offset = 0, size = 2104}}
  // expected-remark @below {{offset = 65536, size = 2104}}
  %alloc1 = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #PARTITIONED_2P_2G, #ttg.shared_memory, mutable>
  // Second partitioned allocation: 2 partition buffers of 2048 bytes each
  // Both allocations are live, so they must be at different offsets within each physical partition
  // expected-remark @below {{offset = 4224, size = 2048}}
  // expected-remark @below {{offset = 67648, size = 2048}}
  %alloc2 = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #PARTITIONED_2P_4G, #ttg.shared_memory, mutable>
  // Use both allocations so they overlap in liveness
  "use"(%alloc1) : (!ttg.memdesc<64x32xf16, #PARTITIONED_2P_2G, #ttg.shared_memory, mutable>) -> ()
  "use"(%alloc2) : (!ttg.memdesc<64x32xf16, #PARTITIONED_2P_4G, #ttg.shared_memory, mutable>) -> ()
  tt.return
}
```
**EN:** This block defines `multiple_partitioned` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `multiple_partitioned`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 118-133
```mlir
// Test liveness/reuse of partitioned buffers
// expected-remark @below {{partitioned_reuse}}
// expected-remark @below {{size = 67640}}
tt.func @partitioned_reuse() {
  // First allocation: 2 partition buffers of 2104 bytes each
  // expected-remark @below {{offset = 0, size = 2104}}
  // expected-remark @below {{offset = 65536, size = 2104}}
  %alloc1 = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #PARTITIONED_2P_2G, #ttg.shared_memory, mutable>
  ttg.local_dealloc %alloc1 : !ttg.memdesc<64x32xf16, #PARTITIONED_2P_2G, #ttg.shared_memory, mutable>
  // Second allocation after dealloc: should reuse the same memory
  // expected-remark @below {{offset = 0, size = 2048}}
  // expected-remark @below {{offset = 65536, size = 2048}}
  %alloc2 = ttg.local_alloc : () -> !ttg.memdesc<64x32xf16, #PARTITIONED_2P_4G, #ttg.shared_memory, mutable>
  tt.return
}
}
```
**EN:** This block defines `partitioned_reuse` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `partitioned_reuse`，并覆盖 共享内存分配。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's analysis coverage and focuses on allocation behavior.  
  **CN:** 该文件属于 Triton 的 分析 测试覆盖，关注点是 分配行为。
- **EN:** The `RUN` pipeline drives tools/passes such as `-allow-unregistered-dialect`, `-test-print-allocation="partition-size=65536"`.  
  **CN:** `RUN` 流水线会驱动 `-allow-unregistered-dialect`, `-test-print-allocation="partition-size=65536"` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）。
- **EN:** Expected remarks/notes are used to pin down the exact analysis output produced by the pass.  
  **CN:** 这里的期望备注/说明用于固定 pass 产生的精确分析输出。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`.  
  **CN:** 外部测试工具：`triton-opt`。
- **EN:** IR dialect dependencies: `ttg`, `tt`.  
  **CN:** IR 方言依赖：`ttg`、`tt`。
