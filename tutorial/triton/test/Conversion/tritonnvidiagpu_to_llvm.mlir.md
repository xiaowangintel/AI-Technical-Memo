# tritonnvidiagpu_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/tritonnvidiagpu_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises NVIDIA-specific lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的NVIDIA 专用降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --convert-triton-gpu-to-llvm=compute-capability=90 -reconcile-unrealized-casts | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×134, CHECK-DAG×2, CHECK-LABEL×33, CHECK-NEXT×35, CHECK-NOT×18; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×134, CHECK-DAG×2, CHECK-LABEL×33, CHECK-NEXT×35, CHECK-NOT×18；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that NVIDIA-specific lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 NVIDIA 专用降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --convert-triton-gpu-to-llvm=compute-capability=90 -reconcile-unrealized-casts | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --convert-triton-gpu-to-llvm=compute-capability=90 -reconcile-unrealized-casts | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --convert-triton-gpu-to-llvm=compute-capability=90 -reconcile-unrealized-casts | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-12
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: init_barrier
  tt.func @init_barrier(%alloc: !ttg.memdesc<1xi64, #shared0, #smem>) {
    // CHECK: "@$0 mbarrier.init.shared::cta.b64 [$1], 1;", "b,r" %{{.*}}, %{{.*}} : (i1, !llvm.ptr<3>) -> !llvm.void
    ttng.init_barrier %alloc, 1 : !ttg.memdesc<1xi64, #shared0, #smem>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `init_barrier`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `init_barrier`。

### Line 14
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 16-28
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: init_barrier_cluster_broadcast
  tt.func @init_barrier_cluster_broadcast() {
    %alloc = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xi64, #shared0, #smem, mutable>
    // CHECK: nvg.cluster_id
    // CHECK: @$0 mbarrier.init.shared::cta.b64 [$1], 2;
    ttng.init_barrier %alloc, 1 : !ttg.memdesc<1xi64, #shared0, #smem, mutable>
    ttng.inval_barrier %alloc : !ttg.memdesc<1xi64, #shared0, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `init_barrier_cluster_broadcast`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `init_barrier_cluster_broadcast`。

### Line 30
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 32-45
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: inval_barrier_cluster_broadcast
  tt.func @inval_barrier_cluster_broadcast(%alloc: !ttg.memdesc<1xi64, #shared0, #smem, mutable>) {
    // CHECK: nvg.cluster_id
    // CHECK: llvm.ptrtoint
    // CHECK: llvm.and
    // CHECK: llvm.inttoptr
    // CHECK: @$0 mbarrier.inval.shared::cta.b64 [$1];
    ttng.inval_barrier %alloc : !ttg.memdesc<1xi64, #shared0, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `inval_barrier_cluster_broadcast`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `inval_barrier_cluster_broadcast`。

### Line 47
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 49-60
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: wait_barrier
  tt.func @wait_barrier(%alloc: !ttg.memdesc<1xi64, #shared0, #smem>, %phase: i32, %pred: i1) {
    // CHECK: waitLoop:
    // CHECK: mbarrier.try_wait.parity.shared::cta.b64
    // CHECK: @!complete bra.uni waitLoop
    // CHECK-NOT: skipWait
    // CHECK: %{{[0-9]+}}, %arg1 :
    ttng.wait_barrier %alloc, %phase : !ttg.memdesc<1xi64, #shared0, #smem>
    %true = arith.constant true
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wait_barrier`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wait_barrier`。

### Lines 62-67
```mlir
    // CHECK: waitLoop:
    // CHECK: mbarrier.try_wait.parity.shared::cta.b64
    // CHECK: @!complete bra.uni waitLoop
    // CHECK-NOT: skipWait
    // CHECK: %{{[0-9]+}}, %arg1 :
    ttng.wait_barrier %alloc, %phase, %true : !ttg.memdesc<1xi64, #shared0, #smem>
```
**EN:** This block defines the test function(s) and exercises operations such as `mbarrier.try_wait.parity.shared`, `cta.b64`, `bra.uni`, `ttng.wait_barrier`, `ttg.memdesc`. Embedded check comments (CHECK×4, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `mbarrier.try_wait.parity.shared`、`cta.b64`、`bra.uni`、`ttng.wait_barrier`、`ttg.memdesc` 这样的操作。 其中嵌入的检查注释（CHECK×4, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 69-77
```mlir
    // CHECK: @!$2 bra.uni skipWait
    // CHECK: waitLoop:
    // CHECK: mbarrier.try_wait.parity.shared::cta.b64
    // CHECK: @!complete bra.uni waitLoop
    // CHECK: skipWait:
    // CHECK: %{{[0-9]+}}, %arg1, %arg2 :
    ttng.wait_barrier %alloc, %phase, %pred : !ttg.memdesc<1xi64, #shared0, #smem>
    tt.return
  }
```
**EN:** This block defines the test function(s) and exercises operations such as `bra.uni`, `mbarrier.try_wait.parity.shared`, `cta.b64`, `ttng.wait_barrier`, `ttg.memdesc`. Embedded check comments (CHECK×6) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `bra.uni`、`mbarrier.try_wait.parity.shared`、`cta.b64`、`ttng.wait_barrier`、`ttg.memdesc` 这样的操作。 其中嵌入的检查注释（CHECK×6）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 79-92
```mlir
  // CHECK-LABEL: arrive_barrier
  tt.func @arrive_barrier(%alloc: !ttg.memdesc<1xi64, #shared0, #smem>) {
    // CHECK-NEXT: [[BASE:%.*]] = llvm.extractvalue %arg0[0] : !llvm.struct<(ptr<3>, i32)>
    // CHECK-NEXT: llvm.extractvalue %arg0[1] : !llvm.struct<(ptr<3>, i32)>
    // CHECK-NEXT: nvvm.barrier0
    // CHECK-NEXT: [[TID:%.*]] = nvvm.read.ptx.sreg.tid.x
    // CHECK-NEXT: [[C127:%.*]] = llvm.mlir.constant(127 : i32)
    // CHECK-NEXT: [[RTID:%.*]] = llvm.and [[TID]], [[C127]]
    // CHECK-NEXT: [[C0:%.*]] = llvm.mlir.constant(0 : i32)
    // CHECK-NEXT: [[IS_ZERO:%.*]] = llvm.icmp "eq" [[RTID]], [[C0]]
    // CHECK-NEXT: "@$0 mbarrier.arrive.shared::cta.b64 _, [$1], 2;", "b,r" [[IS_ZERO]], [[BASE]]
    ttng.arrive_barrier %alloc, 2 : !ttg.memdesc<1xi64, #shared0, #smem>
    tt.return
  }
```
**EN:** This block defines `arrive_barrier` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×9) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `arrive_barrier`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×9）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 94-109
```mlir
  // CHECK-LABEL: arrive_barrier_pred
  tt.func @arrive_barrier_pred(%alloc: !ttg.memdesc<1xi64, #shared0, #smem>, %pred: i1) {
    // CHECK-NEXT: [[BASE:%.*]] = llvm.extractvalue %arg0[0] : !llvm.struct<(ptr<3>, i32)>
    // CHECK-NEXT: llvm.extractvalue %arg0[1] : !llvm.struct<(ptr<3>, i32)>
    // CHECK-NEXT: nvvm.barrier0
    // CHECK-NEXT: [[TID:%.*]] = nvvm.read.ptx.sreg.tid.x
    // CHECK-NEXT: [[C127:%.*]] = llvm.mlir.constant(127 : i32)
    // CHECK-NEXT: [[RTID:%.*]] = llvm.and [[TID]], [[C127]]
    // CHECK-NEXT: [[C0:%.*]] = llvm.mlir.constant(0 : i32)
    // CHECK-NEXT: [[IS_ZERO:%.*]] = llvm.icmp "eq" [[RTID]], [[C0]]
    // CHECK-NEXT: [[PRED:%.*]] = llvm.and [[IS_ZERO]], %arg1
    // CHECK-NEXT: "@$0 mbarrier.arrive.shared::cta.b64 _, [$1], 2;", "b,r" [[PRED]], [[BASE]]
    ttng.arrive_barrier %alloc, 2, %pred : !ttg.memdesc<1xi64, #shared0, #smem>
    tt.return
  }
}
```
**EN:** This block defines `arrive_barrier_pred` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×10) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `arrive_barrier_pred`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×10）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 111
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 113-128
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: arrive_barrier_cluster_broadcast
  tt.func @arrive_barrier_cluster_broadcast(%alloc: !ttg.memdesc<1xi64, #shared0, #smem>) {
    // CHECK: nvvm.barrier0
    // CHECK-NOT: nvg.cluster_id
    // CHECK: llvm.ptrtoint
    // CHECK: llvm.and
    // CHECK: llvm.inttoptr
    // CHECK: mbarrier.arrive.shared::cluster.b64
    // CHECK-NOT: mbarrier.arrive.shared::cta.b64
    ttng.arrive_barrier %alloc, 1 : !ttg.memdesc<1xi64, #shared0, #smem>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `arrive_barrier_cluster_broadcast`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `arrive_barrier_cluster_broadcast`。

### Line 131
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 133-146
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: tma_copy_global_to_local
  // CHECK: elect.sync
  // CHECK: "@$0 cp.async.bulk.tensor.2d.shared::cta.global.mbarrier::complete_tx::bytes [$1], [$2, {$3, $4}], [$5];", "b,r,l,r,r,r" {{.*}} : (i1, !llvm.ptr<3>, !llvm.ptr, i32, i32, !llvm.ptr<3>) -> !llvm.void
  // CHECK-NOT: cp.async.bulk.tensor.2d.shared
  // CHECK: return
  tt.func @tma_copy_global_to_local(%tma: !tt.tensordesc<128x128xf32, #shared1>, %alloc: !ttg.memdesc<128x128xf32, #shared1, #smem, mutable>, %x: i32, %barrier: !ttg.memdesc<1xi64, #shared0, #smem>, %pred: i1) {
    ttng.async_tma_copy_global_to_local %tma[%x, %x] %alloc, %barrier, %pred : !tt.tensordesc<128x128xf32, #shared1>, !ttg.memdesc<1xi64, #shared0, #smem> -> !ttg.memdesc<128x128xf32, #shared1, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_copy_global_to_local`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_copy_global_to_local`。

### Line 148
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 150-162
```mlir
// TMA copy with barrier mask zero: barrier has no CGALayout -> shared::cta
#shared0_cta = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: tma_copy_barrier_mask_zero
  // CHECK: cp.async.bulk.tensor.2d.shared::cta.global.mbarrier::complete_tx::bytes
  // CHECK-NOT: cp.async.bulk.tensor.2d.shared::cluster.global.mbarrier
  tt.func @tma_copy_barrier_mask_zero(%tma: !tt.tensordesc<128x128xf32, #shared1>, %alloc: !ttg.memdesc<128x128xf32, #shared1, #smem, mutable>, %x: i32, %barrier: !ttg.memdesc<1xi64, #shared0_cta, #smem>, %pred: i1) {
    ttng.async_tma_copy_global_to_local %tma[%x, %x] %alloc, %barrier, %pred : !tt.tensordesc<128x128xf32, #shared1>, !ttg.memdesc<1xi64, #shared0_cta, #smem> -> !ttg.memdesc<128x128xf32, #shared1, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_copy_barrier_mask_zero`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_copy_barrier_mask_zero`。

### Line 164
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 166-183
```mlir
// TMA copy with barrier mask non-zero: barrier has CGALayout
#shared0_cluster = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
#shared1_cga = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8, CGALayout = [[0, 0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttng.two-ctas" = true} {
  // CHECK-LABEL: tma_copy_barrier_mask_nonzero
  // Barrier pointer is modified when barrier mask != 0
  // CHECK: llvm.ptrtoint
  // CHECK: llvm.and
  // CHECK: llvm.inttoptr
  // TMA uses shared::cluster when barrier mask is non-zero
  // CHECK: cp.async.bulk.tensor.2d.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes
  // CHECK-NOT: cp.async.bulk.tensor.2d.shared::cta.global.mbarrier
  tt.func @tma_copy_barrier_mask_nonzero(%tma: !tt.tensordesc<128x128xf32, #shared1_cga>, %alloc: !ttg.memdesc<128x128xf32, #shared1_cga, #smem, mutable>, %x: i32, %barrier: !ttg.memdesc<1xi64, #shared0_cluster, #smem>, %pred: i1) {
    ttng.async_tma_copy_global_to_local %tma[%x, %x] %alloc, %barrier, %pred : !tt.tensordesc<128x128xf32, #shared1_cga>, !ttg.memdesc<1xi64, #shared0_cluster, #smem> -> !ttg.memdesc<128x128xf32, #shared1_cga, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttng.two-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_copy_barrier_mask_nonzero`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttng.two-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_copy_barrier_mask_nonzero`。

### Line 185
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 187-198
```mlir
#shared0_cta = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
#shared1_cga = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8, CGALayout = [[0, 0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttng.two-ctas" = true} {
  // CHECK-LABEL: tma_copy_single_cta_barrier_in_two_cta_kernel
  // CHECK: cp.async.bulk.tensor.2d.cta_group::1.shared::cta.global.mbarrier::complete_tx::bytes
  // CHECK-NOT: cp.async.bulk.tensor.2d.cta_group::2
  tt.func @tma_copy_single_cta_barrier_in_two_cta_kernel(%tma: !tt.tensordesc<128x128xf32, #shared1_cga>, %alloc: !ttg.memdesc<128x128xf32, #shared1_cga, #smem, mutable>, %x: i32, %barrier: !ttg.memdesc<2xi64, #shared0_cta, #smem>, %pred: i1) {
    ttng.async_tma_copy_global_to_local %tma[%x, %x] %alloc, %barrier, %pred : !tt.tensordesc<128x128xf32, #shared1_cga>, !ttg.memdesc<2xi64, #shared0_cta, #smem> -> !ttg.memdesc<128x128xf32, #shared1_cga, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttng.two-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_copy_single_cta_barrier_in_two_cta_kernel`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttng.two-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_copy_single_cta_barrier_in_two_cta_kernel`。

### Line 200
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 202-215
```mlir
#blocked_offsets = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0], CGALayout = [[0, 1]]}>
#shared0_cluster = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
#shared1_split = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttng.two-ctas" = true} {
  // CHECK-LABEL: tma_gather_barrier_mask_nonzero
  // Gather uses shared::cluster when barrier mask is non-zero
  // CHECK: cp.async.bulk.tensor.2d.tile::gather4.cta_group::2.shared::cluster.global.mbarrier::complete_tx::bytes
  // CHECK-NOT: cp.async.bulk.tensor.2d.tile::gather4.cta_group::1
  tt.func @tma_gather_barrier_mask_nonzero(%tma: !tt.tensordesc<1x128xbf16, #shared1_split>, %alloc: !ttg.memdesc<32x128xbf16, #shared1_split, #smem, mutable>, %x_offsets: tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked_offsets}>>, %y: i32, %barrier: !ttg.memdesc<1xi64, #shared0_cluster, #smem>, %pred: i1) {
    ttng.async_tma_gather %tma[%x_offsets, %y] %alloc, %barrier, %pred : !tt.tensordesc<1x128xbf16, #shared1_split>, tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked_offsets}>>, i32, !ttg.memdesc<1xi64, #shared0_cluster, #smem>, !ttg.memdesc<32x128xbf16, #shared1_split, #smem, mutable>, i1
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttng.two-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_gather_barrier_mask_nonzero`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttng.two-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_gather_barrier_mask_nonzero`。

### Line 217
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 219-234
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 32}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: tma_copy_global_to_local_im2col
  // CHECK: elect.sync
  // CHECK: cp.async.bulk.tensor.4d.shared::cta.global.im2col.mbarrier::complete_tx::bytes
  // CHECK-NOT: cp.async.bulk.tensor.4d.shared::cta.global.mbarrier
  // CHECK: return
  tt.func @tma_copy_global_to_local_im2col(%tma: !ttng.tensordesc_im2col<16x64xf32, #shared1>, %alloc: !ttg.memdesc<16x64xf32, #shared1, #smem, mutable>, %x: i32, %barrier: !ttg.memdesc<1xi64, #shared0, #smem>, %pred: i1) {
    %off_w = arith.constant 1 : i16
    %off_h = arith.constant 2 : i16
    ttng.async_tma_copy_global_to_local %tma[%x, %x, %x, %x] offsets = [%off_w, %off_h] %alloc, %barrier, %pred : !ttng.tensordesc_im2col<16x64xf32, #shared1>, !ttg.memdesc<1xi64, #shared0, #smem> -> !ttg.memdesc<16x64xf32, #shared1, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_copy_global_to_local_im2col`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_copy_global_to_local_im2col`。

### Line 236
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 238-269
```mlir
// Test im2col with multiple TMA messages in the channel dimension (no swizzle).
// Channel dim = 1024 exceeds max 256, requiring 1024/256 = 4 messages.
// With num-warps = 1, the loop iterates 4 times, generating 4 TMA instructions.
// Channel offsets: 0, 256, 512, 768 (computed as copyIdx << 8).
// Pixel offset is always 0 for im2col mode.
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#shared2 = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 32}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: tma_copy_global_to_local_im2col_multi_msg
  // CHECK: elect.sync
  // Verify 4 TMA messages are generated with offsets computed via shift-left by 8 (multiply by 256)
  // CHECK-DAG: llvm.mlir.constant(8 : i32)
  // Message 1 (copyIdx=0): offset = 0 << 8 = 0
  // CHECK: cp.async.bulk.tensor.4d.shared::cta.global.im2col.mbarrier::complete_tx::bytes
  // Message 2 (copyIdx=1): offset = 1 << 8 = 256
  // CHECK: llvm.mlir.constant(1 : i32)
  // CHECK: cp.async.bulk.tensor.4d.shared::cta.global.im2col.mbarrier::complete_tx::bytes
  // Message 3 (copyIdx=2): offset = 2 << 8 = 512
  // CHECK: llvm.mlir.constant(2 : i32)
  // CHECK: cp.async.bulk.tensor.4d.shared::cta.global.im2col.mbarrier::complete_tx::bytes
  // Message 4 (copyIdx=3): offset = 3 << 8 = 768
  // CHECK: llvm.mlir.constant(3 : i32)
  // CHECK: cp.async.bulk.tensor.4d.shared::cta.global.im2col.mbarrier::complete_tx::bytes
  // CHECK: return
  tt.func @tma_copy_global_to_local_im2col_multi_msg(%tma: !ttng.tensordesc_im2col<64x1024xf32, #shared2>, %alloc: !ttg.memdesc<64x1024xf32, #shared2, #smem, mutable>, %x: i32, %barrier: !ttg.memdesc<1xi64, #shared0, #smem>, %pred: i1) {
    %off_w = arith.constant 1 : i16
    %off_h = arith.constant 2 : i16
    ttng.async_tma_copy_global_to_local %tma[%x, %x, %x, %x] offsets = [%off_w, %off_h] %alloc, %barrier, %pred : !ttng.tensordesc_im2col<64x1024xf32, #shared2>, !ttg.memdesc<1xi64, #shared0, #smem> -> !ttg.memdesc<64x1024xf32, #shared2, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_copy_global_to_local_im2col_multi_msg`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_copy_global_to_local_im2col_multi_msg`。

### Line 271
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 273-304
```mlir
// Test im2col with multiple TMA messages with swizzle enabled.
// swizzlingByteWidth=128, f16 (16-bit) -> block size = (8 * 128) / 16 = 64 elements.
// Channel dim = 256 requires 256/64 = 4 messages.
// Channel offsets: 0, 64, 128, 192 (computed as copyIdx << 6).
// Pixel offset is always 0 for im2col mode.
#shared0_swz = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#shared_swz = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#smem_swz = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: tma_copy_global_to_local_im2col_multi_msg_swizzle
  // CHECK: elect.sync
  // Verify 4 TMA messages are generated with offsets computed via shift-left by 6 (multiply by 64)
  // CHECK-DAG: llvm.mlir.constant(6 : i32)
  // Message 1 (copyIdx=0): offset = 0 << 6 = 0
  // CHECK: cp.async.bulk.tensor.4d.shared::cta.global.im2col.mbarrier::complete_tx::bytes
  // Message 2 (copyIdx=1): offset = 1 << 6 = 64
  // CHECK: llvm.mlir.constant(1 : i32)
  // CHECK: cp.async.bulk.tensor.4d.shared::cta.global.im2col.mbarrier::complete_tx::bytes
  // Message 3 (copyIdx=2): offset = 2 << 6 = 128
  // CHECK: llvm.mlir.constant(2 : i32)
  // CHECK: cp.async.bulk.tensor.4d.shared::cta.global.im2col.mbarrier::complete_tx::bytes
  // Message 4 (copyIdx=3): offset = 3 << 6 = 192
  // CHECK: llvm.mlir.constant(3 : i32)
  // CHECK: cp.async.bulk.tensor.4d.shared::cta.global.im2col.mbarrier::complete_tx::bytes
  // CHECK: return
  tt.func @tma_copy_global_to_local_im2col_multi_msg_swizzle(%tma: !ttng.tensordesc_im2col<64x256xf16, #shared_swz>, %alloc: !ttg.memdesc<64x256xf16, #shared_swz, #smem_swz, mutable>, %x: i32, %barrier: !ttg.memdesc<1xi64, #shared0_swz, #smem_swz>, %pred: i1) {
    %off_w = arith.constant 1 : i16
    %off_h = arith.constant 2 : i16
    ttng.async_tma_copy_global_to_local %tma[%x, %x, %x, %x] offsets = [%off_w, %off_h] %alloc, %barrier, %pred : !ttng.tensordesc_im2col<64x256xf16, #shared_swz>, !ttg.memdesc<1xi64, #shared0_swz, #smem_swz> -> !ttg.memdesc<64x256xf16, #shared_swz, #smem_swz, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_copy_global_to_local_im2col_multi_msg_swizzle`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_copy_global_to_local_im2col_multi_msg_swizzle`。

### Line 306
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 308-320
```mlir
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: tma_copy_local_to_global
  // CHECK: elect.sync
  // CHECK: "@$0 cp.async.bulk.tensor.2d.global.shared::cta.bulk_group [$1, {$2, $3}], [$4];", "b,l,r,r,r" {{.*}} : (i1, !llvm.ptr, i32, i32, !llvm.ptr<3>) -> !llvm.void
  // CHECK-NOT: cp.async.bulk.tensor.2d.global.shared::cta.bulk_group
  // CHECK: nvvm.cp.async.bulk.commit.group
  tt.func @tma_copy_local_to_global(%tma: !tt.tensordesc<128x128xf32, #shared1>, %alloc: !ttg.memdesc<128x128xf32, #shared1, #smem>, %x: i32) {
    ttng.async_tma_copy_local_to_global %tma[%x, %x] %alloc : !tt.tensordesc<128x128xf32, #shared1>, !ttg.memdesc<128x128xf32, #shared1, #smem>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_copy_local_to_global`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_copy_local_to_global`。

### Line 322
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 324-336
```mlir
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 32}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: tma_reduce
  // CHECK: elect.sync
  // CHECK: "@$0 cp.reduce.async.bulk.tensor.2d.global.shared::cta.add.bulk_group [$1, {$2, $3}], [$4];", "b,l,r,r,r" {{.*}} : (i1, !llvm.ptr, i32, i32, !llvm.ptr<3>) -> !llvm.void
  // CHECK-NOT: cp.reduce.async.bulk.tensor.2d.global.shared::cta.add.bulk_group
  // CHECK: nvvm.cp.async.bulk.commit.group
  tt.func @tma_reduce(%tma: !tt.tensordesc<128x128xf32, #shared1>, %alloc: !ttg.memdesc<128x128xf32, #shared1, #smem>, %x: i32) {
    ttng.async_tma_reduce add, %tma[%x, %x] %alloc : !tt.tensordesc<128x128xf32, #shared1>, !ttg.memdesc<128x128xf32, #shared1, #smem>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_reduce`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_reduce`。

### Line 338
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 340-355
```mlir
#shared1_broadcast = #ttg.nvmma_shared<{swizzlingByteWidth = 0, transposed = false, elementBitWidth = 8, CGALayout = [[0, 0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: tma_copy_local_to_global_broadcast
  // CHECK: elect.sync
  // CHECK: nvg.cluster_id
  // CHECK: llvm.and
  // CHECK: llvm.icmp "eq"
  // CHECK: "@$0 cp.async.bulk.tensor.2d.global.shared::cta.bulk_group [$1, {$2, $3}], [$4];", "b,l,r,r,r" {{.*}} : (i1, !llvm.ptr, i32, i32, !llvm.ptr<3>) -> !llvm.void
  // CHECK-NOT: shared::cluster
  // CHECK: nvvm.cp.async.bulk.commit.group
  tt.func @tma_copy_local_to_global_broadcast(%tma: !tt.tensordesc<128x128xf32, #shared1_broadcast>, %alloc: !ttg.memdesc<128x128xf32, #shared1_broadcast, #smem>, %x: i32) {
    ttng.async_tma_copy_local_to_global %tma[%x, %x] %alloc : !tt.tensordesc<128x128xf32, #shared1_broadcast>, !ttg.memdesc<128x128xf32, #shared1_broadcast, #smem>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_copy_local_to_global_broadcast`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_copy_local_to_global_broadcast`。

### Line 357
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 359-366
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: async_tma_store_wait
  // CHECK: nvvm.cp.async.bulk.wait_group 0 {read}
  tt.func @async_tma_store_wait() {
    ttng.async_tma_store_wait {pendings = 0 : i32}
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_tma_store_wait`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_tma_store_wait`。

### Line 368
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 370-386
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: expect_barrier
  // CHECK: nvvm.barrier0
  // CHECK: [[TID:%.*]] = nvvm.read.ptx.sreg.tid.x
  // CHECK: [[C127:%.*]] = llvm.mlir.constant(127 : i32)
  // CHECK: [[RTID:%.*]] = llvm.and [[TID]], [[C127]]
  // CHECK: [[C0:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK: [[IS_ZERO:%.*]] = llvm.icmp "eq" [[RTID]], [[C0]]
  // CHECK: [[PRED:%.*]] = llvm.and [[IS_ZERO]], %arg1
  // CHECK: @$0 mbarrier.arrive.expect_tx.shared::cta.b64 _, [$1], 16384;
  tt.func @expect_barrier(%barrier: !ttg.memdesc<1xi64, #shared0, #smem, mutable>, %pred: i1) {
    ttng.barrier_expect %barrier, 16384, %pred : !ttg.memdesc<1xi64, #shared0, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `expect_barrier`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `expect_barrier`。

### Line 388
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 390-405
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: expect_barrier_cluster_broadcast
  // CHECK: nvvm.barrier0
  // CHECK-NOT: nvg.cluster_id
  // CHECK: llvm.ptrtoint
  // CHECK: llvm.and
  // CHECK: llvm.inttoptr
  // CHECK: @$0 mbarrier.arrive.expect_tx.shared::cluster.b64 _, [$1], 16384;
  // CHECK-NOT: mbarrier.arrive.shared::cluster.b64
  tt.func @expect_barrier_cluster_broadcast(%barrier: !ttg.memdesc<1xi64, #shared0, #smem, mutable>, %pred: i1) {
    ttng.barrier_expect %barrier, 16384, %pred : !ttg.memdesc<1xi64, #shared0, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `expect_barrier_cluster_broadcast`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `expect_barrier_cluster_broadcast`。

### Line 407
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 409-417
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: byval_tma_desc
  // CHECK: llvm.align = 64
  // CHECK: llvm.byval = !llvm.array<128 x i8>
  // CHECK: nvvm.grid_constant
  tt.func @byval_tma_desc(%desc: !tt.ptr<i8, 0> {tt.nv_tma_desc = 1 : i32}) {
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `byval_tma_desc`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `byval_tma_desc`。

### Line 419
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 421-442
```mlir
// CHECK-LABEL: device_tensormap_create1d
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @device_tensormap_create1d(%arg0: !tt.ptr<i16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg2: i32 {tt.divisibility = 16 : i32}) {
    %c256_i32 = arith.constant 256 : i32
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    // CHECK: st.shared::cta.b32
    // CHECK: bar.warp.sync
    // CHECK: tensormap.replace.tile.global_address.shared::cta.b1024.b64 [ $0 + 0 ], $1;
    // CHECK: tensormap.replace.tile.rank.shared::cta.b1024.b32 [ $0 + 0 ], 0x0;
    // CHECK: tensormap.replace.tile.box_dim.shared::cta.b1024.b32 [ $0 + 0 ], 0x0, $1;
    // CHECK: tensormap.replace.tile.global_dim.shared::cta.b1024.b32 [ $0 + 0 ], 0x0, $1;
    // CHECK: tensormap.replace.tile.element_stride.shared::cta.b1024.b32 [ $0 + 0 ], 0x0, $1;
    // CHECK: tensormap.replace.tile.elemtype.shared::cta.b1024.b32 [ $0 + 0 ], 0x3;
    // CHECK: tensormap.replace.tile.interleave_layout.shared::cta.b1024.b32 [ $0 + 0 ], 0x0;
    // CHECK: tensormap.replace.tile.swizzle_mode.shared::cta.b1024.b32 [ $0 + 0 ], 0x2;
    // CHECK: tensormap.replace.tile.fill_mode.shared::cta.b1024.b32 [ $0 + 0 ], 0x1;
    // CHECK: tensormap.cp_fenceproxy.global.shared::cta.tensormap::generic.release.gpu.sync.aligned [ $0 + 0 ], [ $1 + 0 ], 0x80;
    ttng.tensormap_create %arg1, %arg0, [%c256_i32], [%arg2], [], [%c1_i32] {elem_type = 3 : i32, fill_mode = 1 : i32, interleave_layout = 0 : i32, swizzle_mode = 2 : i32, allocation.offset = 0 : i32} : (!tt.ptr<i8>, !tt.ptr<i16>, i32, i32, i32) -> ()
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `device_tensormap_create1d`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `device_tensormap_create1d`。

### Line 444
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 446-472
```mlir
// CHECK-LABEL: device_tensormap_create2d
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @device_tensormap_create2d(%arg0: !tt.ptr<i16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg2: i32 {tt.divisibility = 16 : i32}) {
    %c256_i32 = arith.constant 256 : i32
    %c1_i32 = arith.constant 1 : i32
    %c0_i32 = arith.constant 0 : i32
    %c1024_i64 = arith.constant 1024 : i64
    // CHECK: st.shared::cta.b32
    // CHECK: bar.warp.sync
    // CHECK: tensormap.replace.tile.global_address.shared::cta.b1024.b64 [ $0 + 0 ], $1;
    // CHECK: tensormap.replace.tile.rank.shared::cta.b1024.b32 [ $0 + 0 ], 0x1;
    // CHECK: tensormap.replace.tile.box_dim.shared::cta.b1024.b32 [ $0 + 0 ], 0x0, $1;
    // CHECK: tensormap.replace.tile.box_dim.shared::cta.b1024.b32 [ $0 + 0 ], 0x1, $1;
    // CHECK: tensormap.replace.tile.global_dim.shared::cta.b1024.b32 [ $0 + 0 ], 0x0, $1;
    // CHECK: tensormap.replace.tile.global_dim.shared::cta.b1024.b32 [ $0 + 0 ], 0x1, $1;
    // CHECK: tensormap.replace.tile.global_stride.shared::cta.b1024.b64 [ $0 + 0 ], 0x0, $1;
    // CHECK: tensormap.replace.tile.element_stride.shared::cta.b1024.b32 [ $0 + 0 ], 0x0, $1;
    // CHECK: tensormap.replace.tile.element_stride.shared::cta.b1024.b32 [ $0 + 0 ], 0x1, $1;
    // CHECK: tensormap.replace.tile.elemtype.shared::cta.b1024.b32 [ $0 + 0 ], 0x3;
    // CHECK: tensormap.replace.tile.interleave_layout.shared::cta.b1024.b32 [ $0 + 0 ], 0x0;
    // CHECK: tensormap.replace.tile.swizzle_mode.shared::cta.b1024.b32 [ $0 + 0 ], 0x2;
    // CHECK: tensormap.replace.tile.fill_mode.shared::cta.b1024.b32 [ $0 + 0 ], 0x1;
    // CHECK: tensormap.cp_fenceproxy.global.shared::cta.tensormap::generic.release.gpu.sync.aligned [ $0 + 0 ], [ $1 + 0 ], 0x80;
    ttng.tensormap_create %arg1, %arg0, [%c256_i32, %c256_i32], [%arg2, %arg2], [%c1024_i64], [%c1_i32, %c1_i32] {elem_type = 3 : i32, fill_mode = 1 : i32, interleave_layout = 0 : i32, swizzle_mode = 2 : i32, allocation.offset = 0 : i32} : (!tt.ptr<i8>, !tt.ptr<i16>, i32, i32, i32, i32, i64, i32, i32) -> ()
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `device_tensormap_create2d`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `device_tensormap_create2d`。

### Line 474
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 476-486
```mlir
// CHECK-LABEL: tensormap_fenceproxy_acquire
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tensormap_fenceproxy_acquire(%arg0: !tt.ptr<i8> {tt.divisibility = 16 : i32}) {
    // CHECK: fence.proxy.tensormap::generic.acquire.gpu [ $0 + 0 ], 0x80;
    // ptxas missing fence workaround:
    // CHECK: cp.async.bulk.commit_group
    // CHECK: cp.async.bulk.wait_group.read 0
    ttng.tensormap_fenceproxy_acquire %arg0 : !tt.ptr<i8>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensormap_fenceproxy_acquire`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensormap_fenceproxy_acquire`。

### Line 488
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 490
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`. They parameterize later tests with compact names for `#ttg.swizzled_shared`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`。

### Lines 492-501
```mlir
// CHECK-LABEL: async_copy_mbarrier_arrive
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @async_copy_mbarrier_arrive(%arg0: !ttg.memdesc<1xi64, #shared, #ttg.shared_memory>)  attributes { noinline = false } {
    // CHECK: nvvm.cp.async.mbarrier.arrive %{{.*}} : !llvm.ptr<3>
    ttng.async_copy_mbarrier_arrive %arg0 : !ttg.memdesc<1xi64, #shared, #ttg.shared_memory>
    // CHECK: nvvm.cp.async.mbarrier.arrive %{{.*}} {noinc = true} : !llvm.ptr<3>
    ttng.async_copy_mbarrier_arrive %arg0 { noIncrement } : !ttg.memdesc<1xi64, #shared, #ttg.shared_memory>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_copy_mbarrier_arrive`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_copy_mbarrier_arrive`。

### Line 503
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 505-515
```mlir
// CHECK-LABEL: mbarrier_sync_cluster_init
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @mbarrier_sync_cluster_init() {
    // CHECK: fence.mbarrier_init.release.cluster
    // CHECK: nvvm.cluster.arrive.relaxed
    // CHECK-NEXT: nvvm.cluster.wait
    ttng.fence_mbarrier_init_release_cluster
    ttng.cluster_barrier {relaxed = 1 : i1}
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mbarrier_sync_cluster_init`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mbarrier_sync_cluster_init`。

### Line 517
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 519-527
```mlir
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 4 : i32} {
  // CHECK-LABEL: @cluster_arrive_not_warp_specialized
  llvm.func @cluster_arrive_not_warp_specialized() {
    // CHECK-NOT: ttg.warp_specialize
    // CHECK: nvvm.cluster.arrive
    ttng.cluster_arrive
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cluster_arrive_not_warp_specialized`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cluster_arrive_not_warp_specialized`。

### Line 529
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 531-544
```mlir
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 8 : i32} {
  // CHECK-LABEL: @cluster_arrive_warp_specialized
  llvm.func @cluster_arrive_warp_specialized() {
    // CHECK: ttg.warp_specialize() attributes {warpGroupStartIds = array<i32: 4>}
    // CHECK: default {
    // CHECK-NEXT: nvvm.cluster.arrive.relaxed
    // CHECK-NEXT: ttg.warp_yield
    // CHECK: partition0() num_warps(4) {
    // CHECK-NEXT: nvvm.cluster.arrive.relaxed
    // CHECK-NEXT: ttg.warp_return
    ttng.cluster_arrive {relaxed = true}
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cluster_arrive_warp_specialized`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cluster_arrive_warp_specialized`。

### Line 546
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 548-556
```mlir
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 4 : i32} {
  // CHECK-LABEL: @cluster_wait_not_warp_specialized
  llvm.func @cluster_wait_not_warp_specialized() {
    // CHECK-NOT: ttg.warp_specialize
    // CHECK: nvvm.cluster.wait
    ttng.cluster_wait
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cluster_wait_not_warp_specialized`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cluster_wait_not_warp_specialized`。

### Line 558
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 560-573
```mlir
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 8 : i32} {
  // CHECK-LABEL: @cluster_wait_warp_specialized
  llvm.func @cluster_wait_warp_specialized() {
    // CHECK: ttg.warp_specialize() attributes {warpGroupStartIds = array<i32: 4>}
    // CHECK: default {
    // CHECK-NEXT: nvvm.cluster.wait
    // CHECK-NEXT: ttg.warp_yield
    // CHECK: partition0() num_warps(4) {
    // CHECK-NEXT: nvvm.cluster.wait
    // CHECK-NEXT: ttg.warp_return
    ttng.cluster_wait
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cluster_wait_warp_specialized`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cluster_wait_warp_specialized`。

### Line 575
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 577-586
```mlir
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 4 : i32} {
  // CHECK-LABEL: @cluster_barrier_not_warp_specialized
  llvm.func @cluster_barrier_not_warp_specialized() {
    // CHECK-NOT: ttg.warp_specialize
    // CHECK: nvvm.cluster.arrive
    // CHECK-NEXT: nvvm.cluster.wait
    ttng.cluster_barrier
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cluster_barrier_not_warp_specialized`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cluster_barrier_not_warp_specialized`。

### Line 588
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 590-605
```mlir
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 8 : i32} {
  // CHECK-LABEL: @cluster_barrier_warp_specialized
  llvm.func @cluster_barrier_warp_specialized() {
    // CHECK: ttg.warp_specialize() attributes {warpGroupStartIds = array<i32: 4>}
    // CHECK: default {
    // CHECK-NEXT: nvvm.cluster.arrive.relaxed
    // CHECK-NEXT: nvvm.cluster.wait
    // CHECK-NEXT: ttg.warp_yield
    // CHECK: partition0() num_warps(4) {
    // CHECK-NEXT: nvvm.cluster.arrive.relaxed
    // CHECK-NEXT: nvvm.cluster.wait
    // CHECK-NEXT: ttg.warp_return
    ttng.cluster_barrier {relaxed = true}
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cluster_barrier_warp_specialized`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cluster_barrier_warp_specialized`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on NVIDIA-specific lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 NVIDIA 专用降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--convert-triton-gpu-to-llvm=compute-capability=90`, `-reconcile-unrealized-casts`.  
  **CN:** `RUN` 流水线会驱动 `--convert-triton-gpu-to-llvm=compute-capability=90`, `-reconcile-unrealized-casts` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `llvm` (LLVM dialect IR), `ttng` (next-gen TritonGPU ops), `arith` (scalar/tensor arithmetic), `nvvm` (NVVM backend intrinsics).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）、`ttng`（新一代 TritonGPU 操作）、`arith`（标量/张量算术）、`nvvm`（NVVM 后端内建）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `llvm`, `ttng`, `arith`, `nvvm`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`llvm`、`ttng`、`arith`、`nvvm`。
