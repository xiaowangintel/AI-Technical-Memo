# tritongpu_to_llvm_blackwell.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/tritongpu_to_llvm_blackwell.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises LLVM lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的LLVM 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --convert-triton-gpu-to-llvm=compute-capability=100 -cse | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×222, CHECK-COUNT×25, CHECK-DAG×3, CHECK-LABEL×63, CHECK-NEXT×4, CHECK-NOT×17, CHECK-SAME×1; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×222, CHECK-COUNT×25, CHECK-DAG×3, CHECK-LABEL×63, CHECK-NEXT×4, CHECK-NOT×17, CHECK-SAME×1；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that LLVM lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 LLVM 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --convert-triton-gpu-to-llvm=compute-capability=100 -cse | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --convert-triton-gpu-to-llvm=compute-capability=100 -cse | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --convert-triton-gpu-to-llvm=compute-capability=100 -cse | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-34
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 32]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 16}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @tc_gen5_mma
  // CHECK: %[[WID:.+]] = ttg.warp_id
  // CHECK: %[[C0:.+]] = llvm.mlir.constant(0 : i32) : i32
  // CHECK: %[[P0:.+]] = llvm.icmp "eq" %[[WID]], %[[C0]] : i32
  // CHECK: %[[P1:.+]] = llvm.and %{{.*}}, %[[P0]]  : i1
  // CHECK: nvvm.barrier0
  // CHECK-NEXT: llvm.cond_br %[[P1]]
  // CHECK: %[[E:.+]] = nvvm.elect.sync -> i1
  // CHECK-COUNT-8: @$5 tcgen05.mma.cta_group::1.kind::f16 [ $0 + 0 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %{{.+}}, %{{.+}}, %{{.+}}, %{{.+}}, %{{.+}}, %[[E]]
  // CHECK-NEXT: %[[PRED:.+]] = llvm.and %arg6, %[[E]]
  // CHECK: @$0 tcgen05.commit.cta_group::1.mbarrier::arrive::one.shared::cluster.b64 [$1];", "b,r" %[[PRED]]
  tt.func @tc_gen5_mma(%a: !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>,
                       %b: !ttg.memdesc<128x128xf16, #shared1, #ttg.shared_memory>,
                       %c: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
                       %useAcc: i1,
                       %pred: i1,
                       %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>,
                       %barrierPred: i1) {
    ttng.tc_gen5_mma %a, %b, %c, %useAcc, %pred, %barrier[%barrierPred] {is_async} :
       !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>,
       !ttg.memdesc<128x128xf16, #shared1, #ttg.shared_memory>,
       !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
       !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tc_gen5_mma`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tc_gen5_mma`。

### Line 36
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 38-50
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 32]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @tc_gen5_int8_unsigned_mma
  // CHECK: %[[WID:.+]] = ttg.warp_id
  // CHECK: %[[C0:.+]] = llvm.mlir.constant(0 : i32) : i32
  // CHECK: %[[P0:.+]] = llvm.icmp "eq" %[[WID]], %[[C0]] : i32
  // CHECK: %[[P1:.+]] = llvm.and %{{.*}}, %[[P0]]  : i1
  // CHECK: llvm.cond_br %[[P1]]
  // CHECK: %[[E:.+]] = nvvm.elect.sync -> i1
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 52-53
```mlir
  // Verify descriptor is expected value for i8 unsigned.
  // CHECK:llvm.mlir.constant(136314912 : i32) : i32
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 55-72
```mlir
  // CHECK-COUNT-4: @$5 tcgen05.mma.cta_group::1.kind::i8 [ $0 + 0 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %{{.+}}, %{{.+}}, %{{.+}}, %{{.+}}, %{{.+}}, %[[E]]
  // CHECK: %[[PRED:.+]] = llvm.and %arg6, %[[E]]
  // CHECK: @$0 tcgen05.commit.cta_group::1.mbarrier::arrive::one.shared::cluster.b64 [$1];", "b,r" %[[PRED]]
  tt.func @tc_gen5_int8_unsigned_mma(%a: !ttg.memdesc<128x128xi8, #shared, #ttg.shared_memory>,
                       %b: !ttg.memdesc<128x128xi8, #shared1, #ttg.shared_memory>,
                       %c: !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable>,
                       %useAcc: i1,
                       %pred: i1,
                       %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>,
                       %barrierPred: i1) {
    ttng.tc_gen5_mma %a, %b, %c, %useAcc, %pred, %barrier[%barrierPred] {is_async, is_unsigned} :
       !ttg.memdesc<128x128xi8, #shared, #ttg.shared_memory>,
       !ttg.memdesc<128x128xi8, #shared1, #ttg.shared_memory>,
       !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable>,
       !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>
    tt.return
  }
}
```
**EN:** This block defines `tc_gen5_int8_unsigned_mma` and exercises operations such as `tcgen05.mma.cta_group`, `llvm.and`, `tcgen05.commit.cta_group`, `one.shared`, `cluster.b64`. Embedded check comments (CHECK×2, CHECK-COUNT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `tc_gen5_int8_unsigned_mma`，并覆盖 如 `tcgen05.mma.cta_group`、`llvm.and`、`tcgen05.commit.cta_group`、`one.shared`、`cluster.b64` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-COUNT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 74
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 76-88
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], CTAsPerCGA = [1, 1], CTASplitNum = [1, 1], CTAOrder = [1, 0], instrShape = [16, 256, 32]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @tc_gen5_int8_signed_mma
  // CHECK: %[[WID:.+]] = ttg.warp_id
  // CHECK: %[[C0:.+]] = llvm.mlir.constant(0 : i32) : i32
  // CHECK: %[[P0:.+]] = llvm.icmp "eq" %[[WID]], %[[C0]] : i32
  // CHECK: %[[P1:.+]] = llvm.and %{{.*}}, %[[P0]]  : i1
  // CHECK: llvm.cond_br %[[P1]]
  // CHECK: %[[E:.+]] = nvvm.elect.sync -> i1
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 90-91
```mlir
  // Verify descriptor is expected value for i8 signed.
  // CHECK:llvm.mlir.constant(136316064 : i32) : i32
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 93-110
```mlir
  // CHECK-COUNT-4: @$5 tcgen05.mma.cta_group::1.kind::i8 [ $0 + 0 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %{{.+}}, %{{.+}}, %{{.+}}, %{{.+}}, %{{.+}}, %[[E]]
  // CHECK: %[[PRED:.+]] = llvm.and %arg6, %[[E]]
  // CHECK: @$0 tcgen05.commit.cta_group::1.mbarrier::arrive::one.shared::cluster.b64 [$1];", "b,r" %[[PRED]]
  tt.func @tc_gen5_int8_signed_mma(%a: !ttg.memdesc<128x128xi8, #shared, #ttg.shared_memory>,
                       %b: !ttg.memdesc<128x128xi8, #shared1, #ttg.shared_memory>,
                       %c: !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable>,
                       %useAcc: i1,
                       %pred: i1,
                       %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>,
                       %barrierPred: i1) {
    ttng.tc_gen5_mma %a, %b, %c, %useAcc, %pred, %barrier[%barrierPred] {is_async} :
       !ttg.memdesc<128x128xi8, #shared, #ttg.shared_memory>,
       !ttg.memdesc<128x128xi8, #shared1, #ttg.shared_memory>,
       !ttg.memdesc<128x128xi32, #tmem, #ttng.tensor_memory, mutable>,
       !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>
    tt.return
  }
}
```
**EN:** This block defines `tc_gen5_int8_signed_mma` and exercises operations such as `tcgen05.mma.cta_group`, `llvm.and`, `tcgen05.commit.cta_group`, `one.shared`, `cluster.b64`. Embedded check comments (CHECK×2, CHECK-COUNT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `tc_gen5_int8_signed_mma`，并覆盖 如 `tcgen05.mma.cta_group`、`llvm.and`、`tcgen05.commit.cta_group`、`one.shared`、`cluster.b64` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-COUNT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 112
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 114-127
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 32]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 16}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @tc_gen5_mma_multi_m_n
  // CHECK: %[[TMEM_BASE:.+]] = llvm.ptrtoint %arg2{{.*}} : !llvm.ptr<3> to i32
  // CHECK: @$5 tcgen05.mma.cta_group::1.kind::f16 [ $0 + 0 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %[[TMEM_BASE]]
  // CHECK: @$5 tcgen05.mma.cta_group::1.kind::f16 [ $0 + 64 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %[[TMEM_BASE]]
  // 1048576 = row << 16 + col = 16 << 16 + 0
  // CHECK: @$5 tcgen05.mma.cta_group::1.kind::f16 [ $0 + 1048576 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %[[TMEM_BASE]]
  // 1048640 = row << 16 + col = 16 << 16 + 64
  // CHECK: @$5 tcgen05.mma.cta_group::1.kind::f16 [ $0 + 1048640 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %[[TMEM_BASE]]
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 129-143
```mlir
  tt.func @tc_gen5_mma_multi_m_n(%a: !ttg.memdesc<128x16xf16, #shared, #ttg.shared_memory>,
                       %b: !ttg.memdesc<16x128xf16, #shared1, #ttg.shared_memory>,
                       %c: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
                       %useAcc: i1,
                       %pred: i1,
                       %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>,
                       %barrierPred: i1) {
    ttng.tc_gen5_mma %a, %b, %c, %useAcc, %pred, %barrier[%barrierPred] {is_async} :
       !ttg.memdesc<128x16xf16, #shared, #ttg.shared_memory>,
       !ttg.memdesc<16x128xf16, #shared1, #ttg.shared_memory>,
       !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
       !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>
    tt.return
  }
}
```
**EN:** This block defines `tc_gen5_mma_multi_m_n` and exercises operations such as `tt.func`, `ttg.memdesc`, `ttng.tc_gen5_mma`, `tt.return`.
**CN:** 这一块定义了 `tc_gen5_mma_multi_m_n`，并覆盖 如 `tt.func`、`ttg.memdesc`、`ttng.tc_gen5_mma`、`tt.return` 这样的操作。

### Line 145
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 147-160
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], CGALayout = [[0, 0]], instrShape = [16, 256, 32]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 16, CGALayout = [[0, 0]]}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 16, CGALayout = [[0, 1]]}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 32, colStride = 1, CGALayout = [[0, 1]]>
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @tc_gen5_mma_multi_ctas
  // CHECK: %[[TMEM_BASE:.+]] = llvm.ptrtoint %arg2{{.*}} : !llvm.ptr<3> to i32
  // CHECK: @$5 tcgen05.mma.cta_group::1.kind::f16 [ $0 + 0 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %[[TMEM_BASE]]
  // CHECK: @$5 tcgen05.mma.cta_group::1.kind::f16 [ $0 + 32 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %[[TMEM_BASE]]
  // 1048576 = row << 16 + col = 16 << 16 + 0
  // CHECK: @$5 tcgen05.mma.cta_group::1.kind::f16 [ $0 + 1048576 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %[[TMEM_BASE]]
  // 1048640 = row << 16 + col = 16 << 16 + 32
  // CHECK: @$5 tcgen05.mma.cta_group::1.kind::f16 [ $0 + 1048608 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %[[TMEM_BASE]]
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 162-176
```mlir
  tt.func @tc_gen5_mma_multi_ctas(%a: !ttg.memdesc<128x16xf16, #shared, #ttg.shared_memory>,
                       %b: !ttg.memdesc<16x128xf16, #shared1, #ttg.shared_memory>,
                       %c: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
                       %useAcc: i1,
                       %pred: i1,
                       %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>,
                       %barrierPred: i1) {
    ttng.tc_gen5_mma %a, %b, %c, %useAcc, %pred, %barrier[%barrierPred] {is_async} :
       !ttg.memdesc<128x16xf16, #shared, #ttg.shared_memory>,
       !ttg.memdesc<16x128xf16, #shared1, #ttg.shared_memory>,
       !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
       !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>
    tt.return
  }
}
```
**EN:** This block defines `tc_gen5_mma_multi_ctas` and exercises operations such as `tt.func`, `ttg.memdesc`, `ttng.tc_gen5_mma`, `tt.return`.
**CN:** 这一块定义了 `tc_gen5_mma_multi_ctas`，并覆盖 如 `tt.func`、`ttg.memdesc`、`ttng.tc_gen5_mma`、`tt.return` 这样的操作。

### Line 178
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 180-205
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: @tc_gen5_mma_subslice_acc
  // CHECK-DAG: %[[TMEM_SUBSLICE_OFFSET:.+]] = llvm.mlir.constant(64 : i32) : i32
  // CHECK-DAG: %[[TMEM_BASE:.+]] = llvm.ptrtoint %arg2{{.*}} : !llvm.ptr<3> to i32
  // CHECK: %[[TMEM_SUBSLICE_BASE_INT:.+]] = llvm.add %[[TMEM_BASE]], %[[TMEM_SUBSLICE_OFFSET]] : i32
  // CHECK: %[[TMEM_SUBSLICE_PTR:.+]] = llvm.inttoptr %[[TMEM_SUBSLICE_BASE_INT]] : i32 to !llvm.ptr<3>
  // CHECK: %[[TMEM_SUBSLICE_BASE:.+]] = llvm.ptrtoint %[[TMEM_SUBSLICE_PTR]] : !llvm.ptr<3> to i32
  // CHECK-COUNT-4: @$5 tcgen05.mma.cta_group::1.kind::f16 [ $0 + 0 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %[[TMEM_SUBSLICE_BASE]]
  // CHECK-COUNT-4: @$5 tcgen05.mma.cta_group::1.kind::f16 [ $0 + 128 ], $1, $2, $3, $4;", "r,l,l,r,b,b" %[[TMEM_SUBSLICE_BASE]]
  // CHECK-NOT: nvvm.barrier0
  tt.func @tc_gen5_mma_subslice_acc(%a: !ttg.memdesc<256x64xf16, #shared, #ttg.shared_memory>,
                                    %b: !ttg.memdesc<64x64xf16, #shared1, #ttg.shared_memory>,
                                    %c: !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory, mutable>) {
    %false = arith.constant false
    %true = arith.constant true
    %sub = ttng.tmem_subslice %c {N = 64 : i32} : !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory, mutable> -> !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable, 256x128>
    ttng.tc_gen5_mma %a, %b, %sub, %false, %true :
       !ttg.memdesc<256x64xf16, #shared, #ttg.shared_memory>,
       !ttg.memdesc<64x64xf16, #shared1, #ttg.shared_memory>,
       !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable, 256x128>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tc_gen5_mma_subslice_acc`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tc_gen5_mma_subslice_acc`。

### Line 207
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 209-225
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:100", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld
  // CHECK: nvg.tensor_memory_base
  // CHECK: tcgen05.st.sync.aligned.32x32b.x128.b32
  // CHECK: nvvm.tcgen05.wait <store>
  // CHECK: tcgen05.ld.sync.aligned.32x32b.x128.b32
  // CHECK: nvvm.tcgen05.wait <load>
  tt.func public @tensor_memory_ld(%arg0: !tt.ptr<f16>, %arg1: !tt.ptr<f16>, %arg2: !tt.ptr<f16>) {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %20 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld`。

### Line 227
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 229-243
```mlir
#linear = #ttg.linear<{register = [[0, 1], [8, 0], [0, 8], [0, 16], [0, 32], [0, 64], [16, 0]], lane = [[0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], warp = [[32, 0], [64, 0]], block = []}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:100", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_16x256
  // CHECK: tcgen05.st.sync.aligned.16x256b.x16.b32
  // CHECK: tcgen05.st.sync.aligned.16x256b.x16.b32
  // CHECK: tcgen05.ld.sync.aligned.16x256b.x16.b32
  // CHECK: tcgen05.ld.sync.aligned.16x256b.x16.b32
  tt.func public @tensor_memory_ld_16x256(%arg0: !tt.ptr<f16>, %arg1: !tt.ptr<f16>, %arg2: !tt.ptr<f16>) {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #linear>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #linear>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %20 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #linear>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_16x256`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_16x256`。

### Line 245
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 247-255
```mlir
#tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:100", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_allocation
  // CHECK: llvm.mlir.constant(4194306 : i32) : i32
  tt.func public @tensor_memory_allocation() {
    %0 = ttng.tmem_alloc {tensor_memory_col_offset = 2 : i32, tensor_memory_row_offset = 64 : i32} : () -> !ttg.memdesc<64x128xf32, #tmem, #ttng.tensor_memory, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_allocation`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_allocation`。

### Line 257
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 259-274
```mlir
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32], [0, 64]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [64, 0]], warp = [[16, 0], [32, 0]], block = []}>
#tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:100", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_m64
  // CHECK: nvg.tensor_memory_base
  // CHECK: tcgen05.st.sync.aligned.32x32b.x128.b32
  // CHECK: nvvm.tcgen05.wait <store>
  // CHECK: tcgen05.ld.sync.aligned.32x32b.x128.b32
  // CHECK: nvvm.tcgen05.wait <load>
  tt.func public @tensor_memory_ld_m64(%arg0: !tt.ptr<f16>, %arg1: !tt.ptr<f16>, %arg2: !tt.ptr<f16>) {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #linear>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #linear>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %20 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #linear>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_m64`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_m64`。

### Line 276
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 278-294
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:100", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_unpack_f16
  // CHECK: nvg.tensor_memory_base
  // CHECK: tcgen05.st.sync.aligned.32x32b.x64.unpack::16b.b32
  // CHECK: nvvm.tcgen05.wait <store>
  // CHECK: tcgen05.ld.sync.aligned.32x32b.x64.pack::16b.b32
  // CHECK: nvvm.tcgen05.wait <load>
  tt.func public @tensor_memory_unpack_f16() {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf16, #blocked1>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf16, #blocked1>) -> !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable>
    %20 = ttng.tmem_load %0 : !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf16, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_unpack_f16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_unpack_f16`。

### Line 296
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 298-334
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
#tmem_scales = #ttng.tensor_memory_scales_encoding<>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: @tc_gen5_mma_block_scale
  // CHECK: %[[TMEM_BASE:.+]] = llvm.ptrtoint %arg2 : !llvm.ptr<3> to i32
  // CHECK: %[[WID:.+]] = ttg.warp_id
  // CHECK: %[[C0:.+]] = llvm.mlir.constant(0 : i32) : i32
  // CHECK: %[[P0:.+]] = llvm.icmp "eq" %[[WID]], %[[C0]] : i32
  // CHECK: %[[P1:.+]] = llvm.and %{{.*}}, %[[P0]]  : i1
  // CHECK: llvm.cond_br %[[P1]]
  // CHECK: %[[DESC0:.+]] = llvm.mlir.constant(144708608 : i32) : i32
  // CHECK: @$7 tcgen05.mma.cta_group::1.kind::mxf8f6f4.block_scale.scale_vec::1X [ $0 + 0 ], $1, $2, $3, [ $4 + 0 ], [ $5 + 0 ], $6;", "r,l,l,r,r,r,b,b" %[[TMEM_BASE]], %{{.+}}, %{{.+}}, %[[DESC0]], %{{.+}}, %{{.+}}, %arg5
  // CHECK: %[[TRUE:.+]] = llvm.mlir.constant(true) : i1
  // CHECK: %[[DESC1:.+]] = llvm.mlir.constant(681579536 : i32) : i32
  // CHECK: @$7 tcgen05.mma.cta_group::1.kind::mxf8f6f4.block_scale.scale_vec::1X [ $0 + 0 ], $1, $2, $3, [ $4 + 0 ], [ $5 + 0 ], $6;", "r,l,l,r,r,r,b,b" %[[TMEM_BASE]], %{{.+}}, %{{.+}}, %[[DESC1]], %{{.+}}, %{{.+}}, %[[TRUE]]
  tt.func @tc_gen5_mma_block_scale(%a: !ttg.memdesc<128x64xf8E4M3FN, #shared, #ttg.shared_memory>,
                       %b: !ttg.memdesc<32x128xi8, #shared1, #ttg.shared_memory>,
                       %c: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
                       %scale_a: !ttg.memdesc<128x2xi8, #tmem_scales, #ttng.tensor_memory>,
                       %scale_b: !ttg.memdesc<128x2xi8, #tmem_scales, #ttng.tensor_memory>,
                       %useAcc: i1,
                       %pred: i1,
                       %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>,
                       %barrierPred: i1) {
    ttng.tc_gen5_mma_scaled %a, %b, %c, %scale_a, %scale_b, %useAcc, %pred lhs = e4m3 rhs = e2m1, %barrier[%barrierPred] {is_async} :
    !ttg.memdesc<128x64xf8E4M3FN, #shared, #ttg.shared_memory>,
    !ttg.memdesc<32x128xi8, #shared1, #ttg.shared_memory>,
    !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
    !ttg.memdesc<128x2xi8, #tmem_scales, #ttng.tensor_memory>,
    !ttg.memdesc<128x2xi8, #tmem_scales, #ttng.tensor_memory>,
    !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tc_gen5_mma_block_scale`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tc_gen5_mma_block_scale`。

### Line 336
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 338-371
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
#tmem_scales = #ttng.tensor_memory_scales_encoding<>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: @tc_gen5_mma_block_scale_fp4_a
  // CHECK: %[[DESC0:.+]] = llvm.mlir.constant(144769664 : i32) : i32
  // CHECK: @$7 tcgen05.mma.cta_group::1.kind::mxf8f6f4.block_scale.scale_vec::1X [ $0 + 0 ], $1, $2, $3, [ $4 + 0 ], [ $5 + 0 ], $6;", "r,l,l,r,r,r,b,b" %{{.+}}, %{{.+}}, %{{.+}}, %[[DESC0]]
  // CHECK: %[[DESC1:.+]] = llvm.mlir.constant(681640592 : i32) : i32
  // CHECK: @$7 tcgen05.mma.cta_group::1.kind::mxf8f6f4.block_scale.scale_vec::1X [ $0 + 0 ], $1, $2, $3, [ $4 + 0 ], [ $5 + 0 ], $6;", "r,l,l,r,r,r,b,b" %{{.+}}, %{{.+}}, %{{.+}}, %[[DESC1]]
  // CHECK: %[[DESC2:.+]] = llvm.mlir.constant(1218511520 : i32) : i32
  // CHECK: @$7 tcgen05.mma.cta_group::1.kind::mxf8f6f4.block_scale.scale_vec::1X [ $0 + 0 ], $1, $2, $3, [ $4 + 0 ], [ $5 + 0 ], $6;", "r,l,l,r,r,r,b,b" %{{.+}}, %{{.+}}, %{{.+}}, %[[DESC2]]
  // CHECK: %[[DESC3:.+]] = llvm.mlir.constant(1755382448 : i32) : i32
  // CHECK: @$7 tcgen05.mma.cta_group::1.kind::mxf8f6f4.block_scale.scale_vec::1X [ $0 + 0 ], $1, $2, $3, [ $4 + 0 ], [ $5 + 0 ], $6;", "r,l,l,r,r,r,b,b" %{{.+}}, %{{.+}}, %{{.+}}, %[[DESC3]]
  tt.func @tc_gen5_mma_block_scale_fp4_a(%a: !ttg.memdesc<128x64xi8, #shared1, #ttg.shared_memory>,
                       %b: !ttg.memdesc<128x128xi8, #shared, #ttg.shared_memory>,
                       %c: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
                       %scale_a: !ttg.memdesc<128x2xi8, #tmem_scales, #ttng.tensor_memory>,
                       %scale_b: !ttg.memdesc<128x2xi8, #tmem_scales, #ttng.tensor_memory>,
                       %useAcc: i1,
                       %pred: i1,
                       %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>,
                       %barrierPred: i1) {
    ttng.tc_gen5_mma_scaled %a, %b, %c, %scale_a, %scale_b, %useAcc, %pred lhs = e2m1 rhs = e4m3, %barrier[%barrierPred] {is_async} :
    !ttg.memdesc<128x64xi8, #shared1, #ttg.shared_memory>,
    !ttg.memdesc<128x128xi8, #shared, #ttg.shared_memory>,
    !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>,
    !ttg.memdesc<128x2xi8, #tmem_scales, #ttng.tensor_memory>,
    !ttg.memdesc<128x2xi8, #tmem_scales, #ttng.tensor_memory>,
    !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tc_gen5_mma_block_scale_fp4_a`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tc_gen5_mma_block_scale_fp4_a`。

### Line 373
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 375-398
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, CGALayout = [[1, 0]]}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 16, CGALayout = [[0, 1]]}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[1]]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1, CGALayout = [[1, 0]], twoCTAs = true>
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 8 : i32, "ttng.two-ctas" = true} {
  // CHECK-LABEL: @tc_gen5_mma_2ctas
  tt.func @tc_gen5_mma_2ctas(%a: !ttg.memdesc<256x32xf16, #shared, #ttg.shared_memory>,
                       %b: !ttg.memdesc<32x128xf16, #shared1, #ttg.shared_memory>,
                       %c: !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory, mutable>,
                       %useAcc: i1,
                       %pred: i1,
                       %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>,
                       %barrierPred: i1) {
    // CHECK: tcgen05.mma.cta_group::2.kind::f16
    // CHECK: tcgen05.mma.cta_group::2.kind::f16
    // CHECK: tcgen05.commit.cta_group::2.mbarrier::arrive::one.shared::cluster.multicast::cluster.b64
    ttng.tc_gen5_mma %a, %b, %c, %useAcc, %pred, %barrier[%barrierPred] {is_async, two_ctas} :
       !ttg.memdesc<256x32xf16, #shared, #ttg.shared_memory>,
       !ttg.memdesc<32x128xf16, #shared1, #ttg.shared_memory>,
       !ttg.memdesc<256x128xf32, #tmem, #ttng.tensor_memory, mutable>,
       !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttng.two-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tc_gen5_mma_2ctas`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttng.two-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tc_gen5_mma_2ctas`。

### Line 400
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 403-408
```mlir
#blocked = #ttg.blocked<{sizePerThread=[1, 4], threadsPerWarp=[32, 1], warpsPerCTA=[4, 1], order=[0, 1]}>
#shared = #ttg.shared_linear<{offset = [[0, 1], [0, 2], [32, 0], [64, 0], [1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 4], [0, 8], [0, 16]]}, alignment = 16>
#shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#shared2 = #ttg.shared_linear<{offset = [[0, 1], [0, 2], [32, 0], [64, 0], [1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 4], [0, 8], [0, 16], [128, 0], [256, 0]]}, alignment = 16>
#shared3 = #ttg.shared_linear<{offset = [[0, 1], [0, 2], [32, 0], [64, 0], [1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [128, 0]]}, alignment = 128>
#tmem_scales = #ttng.tensor_memory_scales_encoding<>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#shared`, `#shared1`, `#shared2`, `#shared3`, `#tmem_scales`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.shared_linear`, `#ttg.swizzled_shared`, `#ttng.tensor_memory_scales_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#shared`, `#shared1`, `#shared2`, `#shared3`, `#tmem_scales`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.shared_linear`, `#ttg.swizzled_shared`, `#ttng.tensor_memory_scales_encoding`。

### Line 410
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.num-ctas`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.num-ctas`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 412-423
```mlir
// CHECK-LABEL: @tmem_copy_2d
tt.func public @tmem_copy_2d(%src: !ttg.memdesc<128x32xi8, #shared, #ttg.shared_memory>,
                             %dst: !ttg.memdesc<128x32xi8, #tmem_scales, #ttng.tensor_memory, mutable>) {
  // CHECK: [[ZERO:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK: [[IS_WARP_0:%.*]] = llvm.icmp "eq" {{.*}}, [[ZERO]] : i32
  // CHECK: [[ELECT:%.*]] = nvvm.elect.sync
  // CHECK: [[WARP_PRED:%.*]] = llvm.and [[IS_WARP_0]], [[ELECT]]
  // CHECK-COUNT-8: tcgen05.cp.cta_group::1.warpx4.32x128b
  // CHECK-NOT: tcgen05.commit
  ttng.tmem_copy %src, %dst : !ttg.memdesc<128x32xi8, #shared, #ttg.shared_memory>, !ttg.memdesc<128x32xi8, #tmem_scales, #ttng.tensor_memory, mutable>
  tt.return
}
```
**EN:** This block defines `tmem_copy_2d` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×4, CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `tmem_copy_2d`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×4, CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 425-438
```mlir
// CHECK-LABEL: @tmem_copy_2d_256
tt.func public @tmem_copy_2d_256(%src: !ttg.memdesc<256x4xi8, #shared3, #ttg.shared_memory>,
                                 %dst: !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory, mutable>) {
  // CHECK: [[C0:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK: [[C4:%.*]] = llvm.mlir.constant(4 : i32)
  // CHECK: [[BASE:%.*]] = llvm.ptrtoint %arg1
  // CHECK: [[OFFS0:%.*]] = llvm.add [[BASE]], [[C0]]
  // CHECK: tcgen05.cp.cta_group::1.warpx4.32x128b {{.*}} "r,l,b" [[OFFS0]]
  // CHECK: [[OFFS1:%.*]] = llvm.add [[BASE]], [[C4]]
  // CHECK: tcgen05.cp.cta_group::1.warpx4.32x128b {{.*}} "r,l,b" [[OFFS1]]
  // CHECK-NOT: tcgen05.cp
  ttng.tmem_copy %src, %dst : !ttg.memdesc<256x4xi8, #shared3, #ttg.shared_memory>, !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory, mutable>
  tt.return
}
```
**EN:** This block defines `tmem_copy_2d_256` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.mlir.constant`, `llvm.ptrtoint`, `llvm.add`. Embedded check comments (CHECK×7, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `tmem_copy_2d_256`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.mlir.constant`、`llvm.ptrtoint`、`llvm.add` 这样的操作。 其中嵌入的检查注释（CHECK×7, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 440-448
```mlir
// CHECK-LABEL: @tmem_copy_2d_slice
tt.func public @tmem_copy_2d_slice(%src: !ttg.memdesc<128x32xi8, #shared2, #ttg.shared_memory, 512x32>,
                                   %dst: !ttg.memdesc<128x32xi8, #tmem_scales, #ttng.tensor_memory, mutable>) {
  // CHECK: [[OFF0:%.*]] = llvm.extractvalue %arg0[1]
  // CHECK: [[OFF1:%.*]] = llvm.extractvalue %arg0[2]
  // CHECK-COUNT-8: tcgen05.cp.cta_group::1.warpx4.32x128b
  ttng.tmem_copy %src, %dst : !ttg.memdesc<128x32xi8, #shared2, #ttg.shared_memory, 512x32>, !ttg.memdesc<128x32xi8, #tmem_scales, #ttng.tensor_memory, mutable>
  tt.return
}
```
**EN:** This block defines `tmem_copy_2d_slice` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.extractvalue`, `tcgen05.cp.cta_group`, `ttng.tmem_copy`. Embedded check comments (CHECK×2, CHECK-COUNT×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `tmem_copy_2d_slice`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.extractvalue`、`tcgen05.cp.cta_group`、`ttng.tmem_copy` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-COUNT×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 450
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 452
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 454-459
```mlir
#blocked = #ttg.blocked<{sizePerThread=[1, 4], threadsPerWarp=[32, 1], warpsPerCTA=[4, 1], order=[0, 1], CGALayout = [[1, 0]]}>
#shared = #ttg.shared_linear<{offset = [[0, 1], [0, 2], [32, 0], [64, 0], [1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 4], [0, 8], [0, 16]], block = [[0, 0]]}, alignment = 16>
#shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0], CGALayout = [[0]]}>
#shared2 = #ttg.shared_linear<{offset = [[0, 1], [0, 2], [32, 0], [64, 0], [1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 4], [0, 8], [0, 16], [128, 0], [256, 0]], block = [[0, 0]]}, alignment = 16>
#shared3 = #ttg.shared_linear<{offset = [[0, 1], [0, 2], [32, 0], [64, 0], [1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [128, 0]], block = [[0, 0]]}, alignment = 128>
#tmem_scales = #ttng.tensor_memory_scales_encoding<CGALayout = [[0, 0]]>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#shared`, `#shared1`, `#shared2`, `#shared3`, `#tmem_scales`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.shared_linear`, `#ttg.swizzled_shared`, `#ttng.tensor_memory_scales_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#shared`, `#shared1`, `#shared2`, `#shared3`, `#tmem_scales`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.shared_linear`, `#ttg.swizzled_shared`, `#ttng.tensor_memory_scales_encoding`。

### Line 461
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 2 : i32, "ttg.threads-per-warp" = 32 : i32, "ttng.two-ctas" = true} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.num-ctas`, `ttg.threads-per-warp`, `ttng.two-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.num-ctas`, `ttg.threads-per-warp`, `ttng.two-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 463-479
```mlir
// CHECK-LABEL: @tmem_copy_2d_2cta
tt.func public @tmem_copy_2d_2cta(%src: !ttg.memdesc<128x32xi8, #shared, #ttg.shared_memory>,
                             %dst: !ttg.memdesc<128x32xi8, #tmem_scales, #ttng.tensor_memory, mutable>) {
  // CHECK: [[ZERO:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK: [[IS_WARP_0:%.*]] = llvm.icmp "eq" {{.*}}, [[ZERO]] : i32
  // CHECK: [[ELECT:%.*]] = nvvm.elect.sync
  // CHECK: [[WARP_PRED:%.*]] = llvm.and [[IS_WARP_0]], [[ELECT]]
  // CHECK: nvg.cluster_id
  // CHECK: llvm.and {{.*}}, {{.*}} : i32
  // CHECK: [[IS_CLUSTER_0:%.*]] = llvm.icmp "eq" {{.*}}, [[ZERO]]
  // CHECK: [[LEAD_PRED:%.*]] = llvm.and [[WARP_PRED]], [[IS_CLUSTER_0]]
  // CHECK-COUNT-8: tcgen05.cp.cta_group::2.warpx4.32x128b
  // CHECK-NOT: tcgen05.commit
  ttng.tmem_copy %src, %dst : !ttg.memdesc<128x32xi8, #shared, #ttg.shared_memory>, !ttg.memdesc<128x32xi8, #tmem_scales, #ttng.tensor_memory, mutable>
  tt.return
  }
}
```
**EN:** This block defines `tmem_copy_2d_2cta` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×8, CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `tmem_copy_2d_2cta`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×8, CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 481
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 483-502
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0], CGALayout = [[0, 0]]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16, CGALayout = [[0, 0]]}>
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 4 : i32, "ttng.two-ctas" = true} {
  // CHECK-LABEL: @tma_scatter_broadcast_two_ctas
  // CHECK: %[[CTA:.+]] = nvg.cluster_id
  // CHECK: %[[MASK:.+]] = llvm.mlir.constant(1 : i32) : i32
  // CHECK: %[[CTA_IN_GROUP:.+]] = llvm.and %[[CTA]], %[[MASK]] : i32
  // CHECK: %[[ZERO:.+]] = llvm.mlir.constant(0 : i32) : i32
  // CHECK: %[[IS_LEADER:.+]] = llvm.icmp "eq" %[[CTA_IN_GROUP]], %[[ZERO]] : i32
  // CHECK: %[[WARP_PRED:.+]] = llvm.icmp "eq" {{.*}} : i32
  // CHECK: %[[LEADER_WARP_PRED:.+]] = llvm.and %[[IS_LEADER]], %[[WARP_PRED]] : i1
  // CHECK: %[[ELECT:.+]] = nvvm.elect.sync -> i1
  // CHECK: %[[PRED:.+]] = llvm.and %[[LEADER_WARP_PRED]], %[[ELECT]] : i1
  // CHECK: @$0 cp.async.bulk.tensor.2d.tile::scatter4.global.shared::cta.bulk_group
  // CHECK-SAME: "b,l,r,r,r,r,r,r" %[[PRED]],
  tt.func @tma_scatter_broadcast_two_ctas(%desc: !tt.tensordesc<1x128xbf16, #shared>, %x_offsets: tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>, %y_offset: i32, %src: !ttg.memdesc<32x128xbf16, #shared, #ttg.shared_memory, mutable>) {
    ttng.async_tma_scatter %desc[%x_offsets, %y_offset] %src : !tt.tensordesc<1x128xbf16, #shared>, tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>, i32, !ttg.memdesc<32x128xbf16, #shared, #ttg.shared_memory, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttng.two-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_scatter_broadcast_two_ctas`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttng.two-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_scatter_broadcast_two_ctas`。

### Line 504
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 506-535
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 8}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
#tmem_scales = #ttng.tensor_memory_scales_encoding<>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: @tc_gen5_mma_block_scale_nvfp4
  // CHECK: %[[TMEM_BASE:.+]] = llvm.ptrtoint %{{.*}} : !llvm.ptr<3> to i32
  // CHECK: %[[DESC0:.+]] = llvm.mlir.constant(138413184 : i32) : i32
  // CHECK: @$7 tcgen05.mma.cta_group::1.kind::mxf4nvf4.block_scale.scale_vec::4X [ $0 + 0 ], $1, $2, $3, [ $4 + 0 ], [ $5 + 0 ], $6;", "r,l,l,r,r,r,b,b" %[[TMEM_BASE]], %{{.+}}, %{{.+}}, %[[DESC0]]
  // CHECK: @$7 tcgen05.mma.cta_group::1.kind::mxf4nvf4.block_scale.scale_vec::4X [ $0 + 0 ], $1, $2, $3, [ $4 + 0 ], [ $5 + 0 ], $6;", "r,l,l,r,r,r,b,b" %[[TMEM_BASE]], %{{.+}}, %{{.+}}, %[[DESC0]]
  tt.func @tc_gen5_mma_block_scale_nvfp4(%a: !ttg.memdesc<128x64xi8, #shared, #ttg.shared_memory>,
                       %b: !ttg.memdesc<64x256xi8, #shared1, #ttg.shared_memory>,
                       %c: !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>,
                       %scale_a: !ttg.memdesc<128x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>,
                       %scale_b: !ttg.memdesc<256x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>,
                       %useAcc: i1,
                       %pred: i1,
                       %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>,
                       %barrierPred: i1) {
    ttng.tc_gen5_mma_scaled %a, %b, %c, %scale_a, %scale_b, %useAcc, %pred lhs = e2m1 rhs = e2m1, %barrier[%barrierPred] {is_async} :
    !ttg.memdesc<128x64xi8, #shared, #ttg.shared_memory>,
    !ttg.memdesc<64x256xi8, #shared1, #ttg.shared_memory>,
    !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>,
    !ttg.memdesc<128x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>,
    !ttg.memdesc<256x8xf8E4M3FN, #tmem_scales, #ttng.tensor_memory>,
    !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tc_gen5_mma_block_scale_nvfp4`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tc_gen5_mma_block_scale_nvfp4`。

### Line 537
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 539-569
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = true, elementBitWidth = 8}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
#tmem_scales = #ttng.tensor_memory_scales_encoding<>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: @tc_gen5_mma_block_scale_mxfp4
  // CHECK-DAG: %[[TMEM_BASE:.+]] = llvm.ptrtoint %{{.*}} : !llvm.ptr<3> to i32
  // CHECK: %[[DESC0:.+]] = llvm.mlir.constant(146801792 : i32) : i32
  // CHECK: @$7 tcgen05.mma.cta_group::1.kind::mxf4.block_scale.scale_vec::2X [ $0 + 0 ], $1, $2, $3, [ $4 + 0 ], [ $5 + 0 ], $6;", "r,l,l,r,r,r,b,b" %[[TMEM_BASE]], %{{.+}}, %{{.+}}, %[[DESC0]]
  // CHECK: %[[DESC1:.+]] = llvm.mlir.constant(1220543648 : i32) : i32
  // CHECK: @$7 tcgen05.mma.cta_group::1.kind::mxf4.block_scale.scale_vec::2X [ $0 + 0 ], $1, $2, $3, [ $4 + 0 ], [ $5 + 0 ], $6;", "r,l,l,r,r,r,b,b" %[[TMEM_BASE]], %{{.+}}, %{{.+}}, %[[DESC1]]
  tt.func @tc_gen5_mma_block_scale_mxfp4(%a: !ttg.memdesc<128x64xi8, #shared, #ttg.shared_memory>,
                       %b: !ttg.memdesc<64x256xi8, #shared1, #ttg.shared_memory>,
                       %c: !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>,
                       %scale_a: !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>,
                       %scale_b: !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory>,
                       %useAcc: i1,
                       %pred: i1,
                       %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>,
                       %barrierPred: i1) {
    ttng.tc_gen5_mma_scaled %a, %b, %c, %scale_a, %scale_b, %useAcc, %pred lhs = e2m1 rhs = e2m1, %barrier[%barrierPred] {is_async} :
    !ttg.memdesc<128x64xi8, #shared, #ttg.shared_memory>,
    !ttg.memdesc<64x256xi8, #shared1, #ttg.shared_memory>,
    !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>,
    !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>,
    !ttg.memdesc<256x4xi8, #tmem_scales, #ttng.tensor_memory>,
    !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tc_gen5_mma_block_scale_mxfp4`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tc_gen5_mma_block_scale_mxfp4`。

### Line 571
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 573-574
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 256], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#tmem`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#tmem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttng.tensor_memory_encoding`。

### Lines 576-590
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:100", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_128x256
  // CHECK-COUNT-4: tcgen05.st.sync.aligned.32x32b.x64.b32
  // CHECK-NOT: tcgen05.st
  // CHECK: nvvm.tcgen05.wait <store>
  // CHECK-COUNT-4: tcgen05.ld.sync.aligned.32x32b.x64.b32
  // CHECK-NOT: tcgen05.ld
  // CHECK: nvvm.tcgen05.wait <load>
  tt.func public @tensor_memory_ld_128x256(%arg0: !tt.ptr<f16>, %arg1: !tt.ptr<f16>, %arg2: !tt.ptr<f16>) {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x256xf32, #blocked>) -> !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
    %20 = ttng.tmem_load %0 : !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x256xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_128x256`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_128x256`。

### Line 592
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 594-595
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 2], order = [0, 1]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#tmem`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#tmem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttng.tensor_memory_encoding`。

### Lines 597-609
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:100", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_128x256_8_warps
  // CHECK: tcgen05.st.sync.aligned.32x32b.x128.b32
  // CHECK: nvvm.tcgen05.wait <store>
  // CHECK: tcgen05.ld.sync.aligned.32x32b.x128.b32
  // CHECK: nvvm.tcgen05.wait <load>
  tt.func public @tensor_memory_ld_128x256_8_warps(%arg0: !tt.ptr<f16>, %arg1: !tt.ptr<f16>, %arg2: !tt.ptr<f16>) {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x256xf32, #blocked>) -> !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable>
    %20 = ttng.tmem_load %0 : !ttg.memdesc<128x256xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x256xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_128x256_8_warps`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_128x256_8_warps`。

### Line 611
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 613-614
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [8, 1], order = [0, 1]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#tmem`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#tmem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttng.tensor_memory_encoding`。

### Lines 616-624
```mlir
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_256x64_8_warps_blocked
  tt.func public @tensor_memory_ld_256x64_8_warps_blocked(%tmem: !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>) {
    // CHECK-COUNT-1: tcgen05.ld.sync.aligned.32x32b.x64.b32
    // CHECK-NOT: tcgen05.ld
    %result = ttng.tmem_load %tmem : !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<256x64xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_256x64_8_warps_blocked`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_256x64_8_warps_blocked`。

### Line 626
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 628-629
```mlir
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0], [128, 0]], block = []}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#linear`, `#tmem`. They parameterize later tests with compact names for `#ttg.linear`, `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#linear`, `#tmem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.linear`, `#ttng.tensor_memory_encoding`。

### Lines 631-639
```mlir
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_256x64_8_warps_splitM
  tt.func public @tensor_memory_ld_256x64_8_warps_splitM(%tmem: !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable>) {
    // CHECK: tcgen05.ld.sync.aligned.32x32b.x64.b32
    // CHECK-NOT: tcgen05.ld
    %result = ttng.tmem_load %tmem : !ttg.memdesc<256x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<256x64xf32, #linear>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_256x64_8_warps_splitM`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_256x64_8_warps_splitM`。

### Line 641
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 643-644
```mlir
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16], [0, 32]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0], [0, 64]], block = []}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#linear`, `#tmem`. They parameterize later tests with compact names for `#ttg.linear`, `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#linear`, `#tmem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.linear`, `#ttng.tensor_memory_encoding`。

### Lines 646-654
```mlir
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_128x128_8_warps_splitM
  tt.func public @tensor_memory_ld_128x128_8_warps_splitM(%tmem: !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>) {
    // CHECK-COUNT-1: tcgen05.ld.sync.aligned.32x32b.x64.b32
    // CHECK-NOT: tcgen05.ld
    %result = ttng.tmem_load %tmem : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #linear>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_128x128_8_warps_splitM`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_128x128_8_warps_splitM`。

### Line 656
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 658-659
```mlir
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [64, 0], [0, 32]], block = []}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#linear`, `#tmem`. They parameterize later tests with compact names for `#ttg.linear`, `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#linear`, `#tmem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.linear`, `#ttng.tensor_memory_encoding`。

### Lines 661-669
```mlir
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_128x64_8_warps_splitM
  tt.func public @tensor_memory_ld_128x64_8_warps_splitM(%tmem: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable>) {
    // CHECK-COUNT-1: tcgen05.ld.sync.aligned.32x32b.x32.b32
    // CHECK-NOT: tcgen05.ld
    %result = ttng.tmem_load %tmem : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x64xf32, #linear>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_128x64_8_warps_splitM`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_128x64_8_warps_splitM`。

### Line 671
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 673-674
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#tmem`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#tmem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttng.tensor_memory_encoding`。

### Line 676
```mlir
module attributes {"ttg.num-warps" = 4 : i32, ttg.maxnreg = 80 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:100", ttg.tensor_memory_size = 128 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.maxnreg`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.maxnreg`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 678-685
```mlir
// CHECK-LABEL: @tmem_message_maxnreg_80
tt.func public @tmem_message_maxnreg_80(%desc: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>) {
  // CHECK: tcgen05.ld.sync.aligned.32x32b.x32.b32 {{.*}} [$32 + 0]
  // CHECK: tcgen05.ld.sync.aligned.32x32b.x32.b32 {{.*}} [$32 + 32]
  // CHECK-NOT: tcgen05.ld
  ttng.tmem_load %desc : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory> -> tensor<128x64xf32, #blocked>
  tt.return
}
```
**EN:** This block defines `tmem_message_maxnreg_80` and exercises operations such as `tt.func`, `ttg.memdesc`, `tcgen05.ld.sync.aligned`, `x32.b32`, `tcgen05.ld`. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `tmem_message_maxnreg_80`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tcgen05.ld.sync.aligned`、`x32.b32`、`tcgen05.ld` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 687-705
```mlir
// CHECK-LABEL: @module_constraint_supercedes_local
tt.func public @module_constraint_supercedes_local(%desc: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>) {
  ttg.warp_specialize(%desc) attributes {actualRegisters = array<i32: 256, 256>}
  default {
    // CHECK-COUNT-2: tcgen05.ld.sync.aligned.32x32b.x32.b32
    // CHECK-NOT: tcgen05.ld
    // CHECK: ttg.warp_yield
    ttng.tmem_load %desc : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory> -> tensor<128x64xf32, #blocked>
    ttg.warp_yield
  }
  partition0(%arg0: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>) num_warps(4) {
    // CHECK-COUNT-2: tcgen05.ld.sync.aligned.32x32b.x32.b32
    // CHECK-NOT: tcgen05.ld
    // CHECK: ttg.warp_return
    ttng.tmem_load %arg0 : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory> -> tensor<128x64xf32, #blocked>
    ttg.warp_return
  } : (!ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>) -> ()
  tt.return
}
```
**EN:** This block defines `module_constraint_supercedes_local` and exercises operations such as `tt.func`, `ttg.memdesc`, `ttg.warp_specialize`, `tcgen05.ld.sync.aligned`, `x32.b32`. Embedded check comments (CHECK×2, CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `module_constraint_supercedes_local`，并覆盖 如 `tt.func`、`ttg.memdesc`、`ttg.warp_specialize`、`tcgen05.ld.sync.aligned`、`x32.b32` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 707
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 709
```mlir
module attributes {"ttg.num-warps" = 4 : i32, ttg.maxnreg = 256 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:100", ttg.tensor_memory_size = 128 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.maxnreg`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.maxnreg`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 711-733
```mlir
// CHECK-LABEL: @tmem_message_local_constraint
tt.func public @tmem_message_local_constraint(%desc: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>) {
  ttg.warp_specialize(%desc) attributes {actualRegisters = array<i32: 80, 48>}
  default {
    // CHECK: tcgen05.ld.sync.aligned.32x32b.x32.b32 {{.*}} [$32 + 0]
    // CHECK: tcgen05.ld.sync.aligned.32x32b.x32.b32 {{.*}} [$32 + 32]
    // CHECK-NOT: tcgen05.ld
    // CHECK: ttg.warp_yield
    ttng.tmem_load %desc : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory> -> tensor<128x64xf32, #blocked>
    ttg.warp_yield
  }
  partition0(%arg0: !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>) num_warps(4) {
    // CHECK: tcgen05.ld.sync.aligned.32x32b.x16.b32 {{.*}} [$16 + 0]
    // CHECK: tcgen05.ld.sync.aligned.32x32b.x16.b32 {{.*}} [$16 + 16]
    // CHECK: tcgen05.ld.sync.aligned.32x32b.x16.b32 {{.*}} [$16 + 32]
    // CHECK: tcgen05.ld.sync.aligned.32x32b.x16.b32 {{.*}} [$16 + 48]
    // CHECK-NOT: tcgen05.ld
    // CHECK: ttg.warp_return
    ttng.tmem_load %arg0 : !ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory> -> tensor<128x64xf32, #blocked>
    ttg.warp_return
  } : (!ttg.memdesc<128x64xf32, #tmem, #ttng.tensor_memory>) -> ()
  tt.return
}
```
**EN:** This block defines `tmem_message_local_constraint` and exercises operations such as `tt.func`, `ttg.memdesc`, `ttg.warp_specialize`, `tcgen05.ld.sync.aligned`, `x32.b32`. Embedded check comments (CHECK×8, CHECK-LABEL×1, CHECK-NOT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `tmem_message_local_constraint`，并覆盖 如 `tt.func`、`ttg.memdesc`、`ttg.warp_specialize`、`tcgen05.ld.sync.aligned`、`x32.b32` 这样的操作。 其中嵌入的检查注释（CHECK×8, CHECK-LABEL×1, CHECK-NOT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 735
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 737
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 739-740
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#packed_b16 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#packed_b16`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#packed_b16`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttng.tensor_memory_encoding`。

### Lines 742-751
```mlir
module attributes {ttg.target = "cuda:100", "ttg.num-warps" = 4 : i32, ttg.maxnreg = 128 : i32} {
// CHECK-LABEL: @store_packedb16_2x64xf16
tt.func @store_packedb16_2x64xf16(%arg0: !ttg.memdesc<128x128xf16, #packed_b16, #ttng.tensor_memory, mutable, 1x128x128>, %arg1: tensor<128x128xf16, #blocked>) {
  %true = arith.constant true
  // CHECK: tcgen05.st.sync.aligned.32x32b.x64.b32
  // CHECK-NOT: tcgen05.st
  ttng.tmem_store %arg1, %arg0, %true : tensor<128x128xf16, #blocked> -> !ttg.memdesc<128x128xf16, #packed_b16, #ttng.tensor_memory, mutable, 1x128x128>
  tt.return
}
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.target`, `ttg.maxnreg`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `store_packedb16_2x64xf16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.target`, `ttg.maxnreg`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `store_packedb16_2x64xf16`。

### Lines 753-763
```mlir
module attributes {ttg.target = "cuda:100", "ttg.num-warps" = 4 : i32, ttg.maxnreg = 80 : i32} {
// CHECK-LABEL: @store_packedb16_4x32xf16
tt.func @store_packedb16_4x32xf16(%arg0: !ttg.memdesc<128x128xf16, #packed_b16, #ttng.tensor_memory, mutable, 1x128x128>, %arg1: tensor<128x128xf16, #blocked>) {
  %true = arith.constant true
  // CHECK: tcgen05.st.sync.aligned.32x32b.x32.b32 [$1 + 0]
  // CHECK: tcgen05.st.sync.aligned.32x32b.x32.b32 [$1 + 32]
  // CHECK-NOT: tcgen05.st
  ttng.tmem_store %arg1, %arg0, %true : tensor<128x128xf16, #blocked> -> !ttg.memdesc<128x128xf16, #packed_b16, #ttng.tensor_memory, mutable, 1x128x128>
  tt.return
}
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.target`, `ttg.maxnreg`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `store_packedb16_4x32xf16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.target`, `ttg.maxnreg`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `store_packedb16_4x32xf16`。

### Line 765
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 767-783
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 16}>
#shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 32, colStride = 1>
#tmem1 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  tt.func @tc_gen5_mma_lhs_tmem(%arg0: !ttg.memdesc<128x32xf16, #tmem, #ttng.tensor_memory>, %arg1: !ttg.memdesc<32x128xf16, #shared, #smem>, %arg2: !ttg.memdesc<128x128xf32, #tmem1, #ttng.tensor_memory, mutable>, %arg3: i1, %arg4: i1, %arg5: !ttg.memdesc<1xi64, #shared1, #smem>, %barrierPred: i1) {
    // CHECK-LABEL: tc_gen5_mma_lhs_tmem
    //       CHECK: tcgen05.mma.cta_group::1.kind::f16
    ttng.tc_gen5_mma %arg0, %arg1, %arg2, %arg3, %arg4, %arg5[%barrierPred] {is_async} :
      !ttg.memdesc<128x32xf16, #tmem, #ttng.tensor_memory>,
      !ttg.memdesc<32x128xf16, #shared, #smem>,
      !ttg.memdesc<128x128xf32, #tmem1, #ttng.tensor_memory, mutable>,
      !ttg.memdesc<1xi64, #shared1, #smem>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tc_gen5_mma_lhs_tmem`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tc_gen5_mma_lhs_tmem`。

### Line 785
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 787-788
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`, `#smem`. They parameterize later tests with compact names for `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 790-803
```mlir
module attributes {"ttg.num-warps" = 1 : i32} {
// CHECK-LABEL: @tc_gen5_commit
tt.func @tc_gen5_commit(%arg0: !ttg.memdesc<1xi64, #shared, #smem, mutable>, %pred: i1) {
  // CHECK-NEXT: nvvm.barrier0
  // CHECK: [[ZERO:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK: [[IS_WARP_0:%.*]] = llvm.icmp "eq" [[ZERO]], [[ZERO]]
  // CHECK: [[ELECT:%.*]] = nvvm.elect.sync
  // CHECK: [[WARP_PRED:%.*]] = llvm.and [[IS_WARP_0]], [[ELECT]]
  // CHECK: [[PRED:%.*]] = llvm.and %arg1, [[WARP_PRED]]
  // CHECK: @$0 tcgen05.commit.cta_group::1.mbarrier::arrive::one.shared::cluster.b64 [$1];", "b,r" [[PRED]]
  ttng.tc_gen5_commit %arg0, %pred : !ttg.memdesc<1xi64, #shared, #smem, mutable>
  tt.return
}
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tc_gen5_commit`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tc_gen5_commit`。

### Line 805
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 807-808
```mlir
#tmem_f32 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 16, colStride = 1>
#tmem_f16 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 16, colStride = 2>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#tmem_f32`, `#tmem_f16`. They parameterize later tests with compact names for `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#tmem_f32`, `#tmem_f16`。它们为后续测试提供紧凑名称，用来表示 `#ttng.tensor_memory_encoding`。

### Line 810
```mlir
module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 812-817
```mlir
// CHECK-LABEL: @reinterpret
tt.func private @reinterpret(%arg0: !ttg.memdesc<128x32xf32, #tmem_f32, #ttng.tensor_memory>) -> !ttg.memdesc<128x32xf16, #tmem_f16, #ttng.tensor_memory> {
  %0 = ttg.memdesc_reinterpret %arg0 : !ttg.memdesc<128x32xf32, #tmem_f32, #ttng.tensor_memory> -> !ttg.memdesc<128x32xf16, #tmem_f16, #ttng.tensor_memory>
  // CHECK-NEXT: return %arg0
  tt.return %0 : !ttg.memdesc<128x32xf16, #tmem_f16, #ttng.tensor_memory>
}
```
**EN:** This block defines `reinterpret` and exercises operations such as `tt.func`, `ttg.memdesc`, `ttg.memdesc_reinterpret`, `tt.return`. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `reinterpret`，并覆盖 如 `tt.func`、`ttg.memdesc`、`ttg.memdesc_reinterpret`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 819
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 821
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 823-826
```mlir
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
#tmem_unpacked = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 2>
#tmem_x1 = #ttng.tensor_memory_encoding<blockM = 128, blockN = 2, colStride = 1>
#tmem_x1_unpacked = #ttng.tensor_memory_encoding<blockM = 128, blockN = 2, colStride = 2>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#tmem`, `#tmem_unpacked`, `#tmem_x1`, `#tmem_x1_unpacked`. They parameterize later tests with compact names for `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#tmem`, `#tmem_unpacked`, `#tmem_x1`, `#tmem_x1_unpacked`。它们为后续测试提供紧凑名称，用来表示 `#ttng.tensor_memory_encoding`。

### Line 828
```mlir
#blocked_x1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked_x1`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked_x1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Line 830
```mlir
module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 832-839
```mlir
// CHECK-LABEL: @subslice_unpacked
tt.func private @subslice_unpacked(%arg0: !ttg.memdesc<128x128xf16, #tmem_unpacked, #ttng.tensor_memory>) -> !ttg.memdesc<128x64xf16, #tmem_unpacked, #ttng.tensor_memory, 128x128> {
  // CHECK: [[OFFSET:%.*]] = llvm.mlir.constant(64 : i32)
  // CHECK: [[PTR:%.*]] = llvm.ptrtoint
  // CHECK: llvm.add [[PTR]], [[OFFSET]] : i32
  %0 = ttng.tmem_subslice %arg0 {N = 64 : i32} : !ttg.memdesc<128x128xf16, #tmem_unpacked, #ttng.tensor_memory> -> !ttg.memdesc<128x64xf16, #tmem_unpacked, #ttng.tensor_memory, 128x128>
  tt.return %0 : !ttg.memdesc<128x64xf16, #tmem_unpacked, #ttng.tensor_memory, 128x128>
}
```
**EN:** This block defines `subslice_unpacked` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.mlir.constant`, `llvm.ptrtoint`, `llvm.add`. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `subslice_unpacked`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.mlir.constant`、`llvm.ptrtoint`、`llvm.add` 这样的操作。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 842-849
```mlir
// CHECK-LABEL: @subslice_packed
tt.func private @subslice_packed(%arg0: !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory>) -> !ttg.memdesc<128x64xf16, #tmem, #ttng.tensor_memory, 128x128> {
  // CHECK: [[OFFSET:%.*]] = llvm.mlir.constant(32 : i32)
  // CHECK: [[PTR:%.*]] = llvm.ptrtoint
  // CHECK: llvm.add [[PTR]], [[OFFSET]] : i32
  %0 = ttng.tmem_subslice %arg0 {N = 64 : i32} : !ttg.memdesc<128x128xf16, #tmem, #ttng.tensor_memory> -> !ttg.memdesc<128x64xf16, #tmem, #ttng.tensor_memory, 128x128>
  tt.return %0 : !ttg.memdesc<128x64xf16, #tmem, #ttng.tensor_memory, 128x128>
}
```
**EN:** This block defines `subslice_packed` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.mlir.constant`, `llvm.ptrtoint`, `llvm.add`. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `subslice_packed`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.mlir.constant`、`llvm.ptrtoint`、`llvm.add` 这样的操作。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 851-865
```mlir
// CHECK-LABEL: @load_store_x1
tt.func @load_store_x1(%arg0: !ttg.memdesc<128x2xf16, #tmem_x1, #ttng.tensor_memory, mutable>) {
  %true = arith.constant true
  // CHECK: [[V:%.*]] = llvm.inline_asm {{.*}}tcgen05.ld.sync{{.*}} (i32) -> i32
  // CHECK: [[V1:%.*]] = llvm.bitcast [[V]] : i32 to i32
  // CHECK: [[F:%.*]] = llvm.bitcast [[V1]] : i32 to vector<2xf16>
  // CHECK: [[E0:%.*]] = llvm.extractelement [[F]]{{.*}} : vector<2xf16>
  // CHECK: [[E1:%.*]] = llvm.extractelement [[F]]{{.*}} : vector<2xf16>
  // CHECK: [[U:%.*]] = llvm.mlir.undef : !llvm.struct<(f16, f16)>
  // CHECK: [[I0:%.*]] = llvm.insertvalue [[E0]], [[U]][0] : !llvm.struct<(f16, f16)>
  // CHECK: [[I1:%.*]] = llvm.insertvalue [[E1]], [[I0]][1] : !llvm.struct<(f16, f16)>
  %0 = ttng.tmem_load %arg0 : !ttg.memdesc<128x2xf16, #tmem_x1, #ttng.tensor_memory, mutable> -> tensor<128x2xf16, #blocked_x1>
  ttng.tmem_store %0, %arg0, %true : tensor<128x2xf16, #blocked_x1> -> !ttg.memdesc<128x2xf16, #tmem_x1, #ttng.tensor_memory, mutable>
  tt.return
}
```
**EN:** This block defines `load_store_x1` and exercises backend inline assembly. Embedded check comments (CHECK×8, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `load_store_x1`，并覆盖 后端内联汇编。 其中嵌入的检查注释（CHECK×8, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 867-880
```mlir
// CHECK-LABEL: @load_store_x1_unpacked
tt.func @load_store_x1_unpacked(%arg0: !ttg.memdesc<128x2xf16, #tmem_x1_unpacked, #ttng.tensor_memory, mutable>) {
  %true = arith.constant true
  // CHECK: [[V:%.*]] = llvm.inline_asm {{.*}}tcgen05.ld.sync{{.*}} (i32) -> i32
  // CHECK: [[V1:%.*]] = llvm.bitcast [[V]] : i32 to i32
  // CHECK: [[F:%.*]] = llvm.bitcast [[V1]] : i32 to vector<2xf16>
  // CHECK: [[C0:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK: extractelement [[F]][[[C0]] : i32]
  // CHECK: [[C1:%.*]] = llvm.mlir.constant(1 : i32)
  // CHECK: extractelement [[F]][[[C1]] : i32]
  %0 = ttng.tmem_load %arg0 : !ttg.memdesc<128x2xf16, #tmem_x1_unpacked, #ttng.tensor_memory, mutable> -> tensor<128x2xf16, #blocked_x1>
  ttng.tmem_store %0, %arg0, %true : tensor<128x2xf16, #blocked_x1> -> !ttg.memdesc<128x2xf16, #tmem_x1_unpacked, #ttng.tensor_memory, mutable>
  tt.return
}
```
**EN:** This block defines `load_store_x1_unpacked` and exercises backend inline assembly. Embedded check comments (CHECK×7, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `load_store_x1_unpacked`，并覆盖 后端内联汇编。 其中嵌入的检查注释（CHECK×7, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 882
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 884
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 886-902
```mlir
// CHECK-LABEL: max_reduction
//       CHECK:  %[[M:.+]] = llvm.mlir.constant(-1 : i32) : i32
//       CHECK:   nvvm.redux.sync  fmax %{{.*}}, %[[M]] {nan = true} : f32 -> f32
//       CHECK:   nvvm.barrier0
//       CHECK:   nvvm.shfl.sync bfly
//       CHECK:   nvvm.shfl.sync bfly
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
module attributes {"ttg.target" = "cuda:100", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @max_reduction(%arg0: tensor<1x1024xf32, #blocked>) {
    %11 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%arg2: f32, %arg3: f32):
      %15 = arith.maximumf %arg2, %arg3 : f32
      tt.reduce.return %15 : f32
    }) {allocation.offset = 0 : i32} : (tensor<1x1024xf32, #blocked>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `max_reduction`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `max_reduction`。

### Line 904
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 906-922
```mlir
// CHECK-LABEL: maxnum_reduction
//       CHECK:  %[[M:.+]] = llvm.mlir.constant(-1 : i32) : i32
//       CHECK:   nvvm.redux.sync  fmax %{{.*}}, %[[M]] : f32 -> f32
//       CHECK:   nvvm.barrier0
//       CHECK:   nvvm.shfl.sync bfly
//       CHECK:   nvvm.shfl.sync bfly
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
module attributes {"ttg.target" = "cuda:100", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @maxnum_reduction(%arg0: tensor<1x1024xf32, #blocked>) {
    %11 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%arg2: f32, %arg3: f32):
      %15 = arith.maxnumf %arg2, %arg3 : f32
      tt.reduce.return %15 : f32
    }) {allocation.offset = 0 : i32} : (tensor<1x1024xf32, #blocked>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `maxnum_reduction`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `maxnum_reduction`。

### Line 924
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 926-936
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 1], instrShape = [16, 8]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 64, transposed = false, elementBitWidth = 8}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
  // CHECK-LABEL: lower_ldmatrix_trans_b8
  tt.func @lower_ldmatrix_trans_b8(%A: !ttg.memdesc<128x64xf8E4M3FN, #shared, #smem, mutable, 1x128x64>) {
    %0 = ttg.local_load %A : !ttg.memdesc<128x64xf8E4M3FN, #shared, #smem, mutable, 1x128x64> -> tensor<128x64xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
    // CHECK-COUNT-16: nvvm.ldmatrix %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b8>, layout = #nvvm.mma_layout<col>{{.*}}} : (!llvm.ptr<3>) -> !llvm.struct<(i32, i32, i32, i32)>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `lower_ldmatrix_trans_b8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `lower_ldmatrix_trans_b8`。

### Line 938
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 940-951
```mlir
#linear3 = #ttg.linear<{register = [[0, 0, 0, 1, 0], [0, 0, 0, 0, 8], [0, 0, 0, 8, 0], [0, 0, 0, 0, 16], [0, 0, 0, 0, 128]], lane = [[0, 0, 0, 2, 0], [0, 0, 0, 4, 0], [0, 0, 0, 0, 1], [0, 0, 0, 0, 2], [0, 0, 0, 0, 4]], warp = [[0, 0, 0, 0, 32], [0, 0, 0, 0, 64]], block = []}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8, rank = 5}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @stmatrix_b8_trans_linear
  tt.func public @stmatrix_b8_trans_linear(%data: tensor<1x1x1x16x256xf8E4M3FN, #linear3>) {
    // CHECK-COUNT-2: nvvm.stmatrix %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b8>, layout = #nvvm.mma_layout<col>{{.*}}} : !llvm.ptr<3>, i32, i32, i32, i32
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1x1x1x16x256xf8E4M3FN, #shared, #smem, mutable>
    ttg.local_store %data, %0 : tensor<1x1x1x16x256xf8E4M3FN, #linear3> -> !ttg.memdesc<1x1x1x16x256xf8E4M3FN, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `stmatrix_b8_trans_linear`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `stmatrix_b8_trans_linear`。

### Line 953
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 955-956
```mlir
#bm64_bn128 = #ttng.tensor_memory_encoding<blockM = 64, blockN = 128, colStride = 1>
#bm64_bn64 = #ttng.tensor_memory_encoding<blockM = 64, blockN = 64, colStride = 1>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#bm64_bn128`, `#bm64_bn64`. They parameterize later tests with compact names for `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#bm64_bn128`, `#bm64_bn64`。它们为后续测试提供紧凑名称，用来表示 `#ttng.tensor_memory_encoding`。

### Lines 958-959
```mlir
#bm64_bn32 = #ttng.tensor_memory_encoding<blockM = 64, blockN = 32, colStride = 1>
#bm64_bn16 = #ttng.tensor_memory_encoding<blockM = 64, blockN = 16, colStride = 1>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#bm64_bn32`, `#bm64_bn16`. They parameterize later tests with compact names for `#ttng.tensor_memory_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#bm64_bn32`, `#bm64_bn16`。它们为后续测试提供紧凑名称，用来表示 `#ttng.tensor_memory_encoding`。

### Line 961
```mlir
#tmem = #ttng.tensor_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#tmem`. They parameterize later tests with compact names for `#ttng.tensor_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#tmem`。它们为后续测试提供紧凑名称，用来表示 `#ttng.tensor_memory`。

### Line 963
```mlir
module attributes {"ttg.target" = "cuda:100", "ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 965-972
```mlir
// CHECK-LABEL: @subslice_16x32bx2
tt.func private @subslice_16x32bx2(%arg0: !ttg.memdesc<64x128xf32, #bm64_bn128, #tmem>) -> !ttg.memdesc<64x64xf32, #bm64_bn128, #tmem, 64x128> {
  // CHECK: [[OFFSET:%.*]] = llvm.mlir.constant(64 : i32)
  // CHECK: [[PTR:%.*]] = llvm.ptrtoint
  // CHECK: llvm.add [[PTR]], [[OFFSET]] : i32
  %0 = ttng.tmem_subslice %arg0 {N = 64 : i32} : !ttg.memdesc<64x128xf32, #bm64_bn128, #tmem> -> !ttg.memdesc<64x64xf32, #bm64_bn128, #tmem, 64x128>
  tt.return %0 : !ttg.memdesc<64x64xf32, #bm64_bn128, #tmem, 64x128>
}
```
**EN:** This block defines `subslice_16x32bx2` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.mlir.constant`, `llvm.ptrtoint`, `llvm.add`. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `subslice_16x32bx2`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.mlir.constant`、`llvm.ptrtoint`、`llvm.add` 这样的操作。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 974-981
```mlir
// CHECK-LABEL: @subslice_16x32bx2_packed
tt.func private @subslice_16x32bx2_packed(%arg0: !ttg.memdesc<64x128xf16, #bm64_bn128, #tmem>) -> !ttg.memdesc<64x64xf16, #bm64_bn128, #tmem, 64x128> {
  // CHECK: [[OFFSET:%.*]] = llvm.mlir.constant(32 : i32)
  // CHECK: [[PTR:%.*]] = llvm.ptrtoint
  // CHECK: llvm.add [[PTR]], [[OFFSET]] : i32
  %0 = ttng.tmem_subslice %arg0 {N = 64 : i32} : !ttg.memdesc<64x128xf16, #bm64_bn128, #tmem> -> !ttg.memdesc<64x64xf16, #bm64_bn128, #tmem, 64x128>
  tt.return %0 : !ttg.memdesc<64x64xf16, #bm64_bn128, #tmem, 64x128>
}
```
**EN:** This block defines `subslice_16x32bx2_packed` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.mlir.constant`, `llvm.ptrtoint`, `llvm.add`. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `subslice_16x32bx2_packed`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.mlir.constant`、`llvm.ptrtoint`、`llvm.add` 这样的操作。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 983-991
```mlir
// CHECK-LABEL: @subslice_16x32bx2_interleaved_block1
tt.func private @subslice_16x32bx2_interleaved_block1(%arg0: !ttg.memdesc<64x128xf32, #bm64_bn32, #tmem>) -> !ttg.memdesc<64x32xf32, #bm64_bn32, #tmem, 64x128> {
  // 16 << 16 => 1048576
  // CHECK: [[OFFSET:%.*]] = llvm.mlir.constant(1048576 : i32)
  // CHECK: [[PTR:%.*]] = llvm.ptrtoint
  // CHECK: llvm.add [[PTR]], [[OFFSET]] : i32
  %0 = ttng.tmem_subslice %arg0 {N = 32 : i32} : !ttg.memdesc<64x128xf32, #bm64_bn32, #tmem> -> !ttg.memdesc<64x32xf32, #bm64_bn32, #tmem, 64x128>
  tt.return %0 : !ttg.memdesc<64x32xf32, #bm64_bn32, #tmem, 64x128>
}
```
**EN:** This block defines `subslice_16x32bx2_interleaved_block1` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.mlir.constant`, `llvm.ptrtoint`, `llvm.add`. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `subslice_16x32bx2_interleaved_block1`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.mlir.constant`、`llvm.ptrtoint`、`llvm.add` 这样的操作。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 993-1000
```mlir
// CHECK-LABEL: @subslice_16x32bx2_interleaved_block0
tt.func private @subslice_16x32bx2_interleaved_block0(%arg0: !ttg.memdesc<64x128xf32, #bm64_bn32, #tmem>) -> !ttg.memdesc<64x16xf32, #bm64_bn32, #tmem, 64x128> {
  // CHECK: [[OFFSET:%.*]] = llvm.mlir.constant(16 : i32)
  // CHECK: [[PTR:%.*]] = llvm.ptrtoint
  // CHECK: llvm.add [[PTR]], [[OFFSET]] : i32
  %0 = ttng.tmem_subslice %arg0 {N = 16 : i32} : !ttg.memdesc<64x128xf32, #bm64_bn32, #tmem> -> !ttg.memdesc<64x16xf32, #bm64_bn32, #tmem, 64x128>
  tt.return %0 : !ttg.memdesc<64x16xf32, #bm64_bn32, #tmem, 64x128>
}
```
**EN:** This block defines `subslice_16x32bx2_interleaved_block0` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.mlir.constant`, `llvm.ptrtoint`, `llvm.add`. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `subslice_16x32bx2_interleaved_block0`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.mlir.constant`、`llvm.ptrtoint`、`llvm.add` 这样的操作。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 1002-1010
```mlir
// CHECK-LABEL: @subslice_16x32bx2_interleaved_block0_offset
tt.func private @subslice_16x32bx2_interleaved_block0_offset(%arg0: !ttg.memdesc<64x128xf32, #bm64_bn32, #tmem>) -> !ttg.memdesc<64x16xf32, #bm64_bn32, #tmem, 64x128> {
  // (16 << 16) | 16 => 1048592
  // CHECK: [[OFFSET:%.*]] = llvm.mlir.constant(1048592 : i32)
  // CHECK: [[PTR:%.*]] = llvm.ptrtoint
  // CHECK: llvm.add [[PTR]], [[OFFSET]] : i32
  %0 = ttng.tmem_subslice %arg0 {N = 48 : i32} : !ttg.memdesc<64x128xf32, #bm64_bn32, #tmem> -> !ttg.memdesc<64x16xf32, #bm64_bn32, #tmem, 64x128>
  tt.return %0 : !ttg.memdesc<64x16xf32, #bm64_bn32, #tmem, 64x128>
}
```
**EN:** This block defines `subslice_16x32bx2_interleaved_block0_offset` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.mlir.constant`, `llvm.ptrtoint`, `llvm.add`. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `subslice_16x32bx2_interleaved_block0_offset`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.mlir.constant`、`llvm.ptrtoint`、`llvm.add` 这样的操作。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 1012-1019
```mlir
// CHECK-LABEL: @subslice_16x32bx2_interleaved_block4_offset
tt.func private @subslice_16x32bx2_interleaved_block4_offset(%arg0: !ttg.memdesc<64x128xf32, #bm64_bn32, #tmem>) -> !ttg.memdesc<64x16xf32, #bm64_bn32, #tmem, 64x128> {
  // CHECK: [[OFFSET:%.*]] = llvm.mlir.constant(48 : i32)
  // CHECK: [[PTR:%.*]] = llvm.ptrtoint
  // CHECK: llvm.add [[PTR]], [[OFFSET]] : i32
  %0 = ttng.tmem_subslice %arg0 {N = 80 : i32} : !ttg.memdesc<64x128xf32, #bm64_bn32, #tmem> -> !ttg.memdesc<64x16xf32, #bm64_bn32, #tmem, 64x128>
  tt.return %0 : !ttg.memdesc<64x16xf32, #bm64_bn32, #tmem, 64x128>
}
```
**EN:** This block defines `subslice_16x32bx2_interleaved_block4_offset` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.mlir.constant`, `llvm.ptrtoint`, `llvm.add`. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `subslice_16x32bx2_interleaved_block4_offset`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.mlir.constant`、`llvm.ptrtoint`、`llvm.add` 这样的操作。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1021
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 1023
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1025-1026
```mlir
#bm128_bn256_cga = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1, CGALayout = [[1, 0], [2, 0]]>
#tmem = #ttng.tensor_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#bm128_bn256_cga`, `#tmem`. They parameterize later tests with compact names for `#ttng.tensor_memory_encoding`, `#ttng.tensor_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#bm128_bn256_cga`, `#tmem`。它们为后续测试提供紧凑名称，用来表示 `#ttng.tensor_memory_encoding`, `#ttng.tensor_memory`。

### Line 1028
```mlir
module attributes {"ttg.target" = "cuda:100", "ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-warps`, `ttg.num-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-warps`, `ttg.num-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 1030-1037
```mlir
// CHECK-LABEL: @subslice_cga_allocshape_preserved
tt.func private @subslice_cga_allocshape_preserved(%arg0: !ttg.memdesc<512x256xf32, #bm128_bn256_cga, #tmem>) -> !ttg.memdesc<512x32xf32, #bm128_bn256_cga, #tmem, 512x256> {
  // CHECK: [[OFFSET:%.*]] = llvm.mlir.constant(32 : i32)
  // CHECK: [[PTR:%.*]] = llvm.ptrtoint
  // CHECK: llvm.add [[PTR]], [[OFFSET]] : i32
  %0 = ttng.tmem_subslice %arg0 {N = 32 : i32} : !ttg.memdesc<512x256xf32, #bm128_bn256_cga, #tmem> -> !ttg.memdesc<512x32xf32, #bm128_bn256_cga, #tmem, 512x256>
  tt.return %0 : !ttg.memdesc<512x32xf32, #bm128_bn256_cga, #tmem, 512x256>
}
```
**EN:** This block defines `subslice_cga_allocshape_preserved` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.mlir.constant`, `llvm.ptrtoint`, `llvm.add`. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `subslice_cga_allocshape_preserved`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.mlir.constant`、`llvm.ptrtoint`、`llvm.add` 这样的操作。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1039
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 1041
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1043-1045
```mlir
#tmem = #ttng.tensor_memory_encoding<blockM = 64, blockN = 1, colStride = 1>
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [0, 0], [0, 4]], block = []}>
#tmem_scales = #ttng.tensor_memory_scales_encoding<>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#tmem`, `#linear`, `#tmem_scales`. They parameterize later tests with compact names for `#ttng.tensor_memory_encoding`, `#ttg.linear`, `#ttng.tensor_memory_scales_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#tmem`, `#linear`, `#tmem_scales`。它们为后续测试提供紧凑名称，用来表示 `#ttng.tensor_memory_encoding`, `#ttg.linear`, `#ttng.tensor_memory_scales_encoding`。

### Lines 1047-1055
```mlir
module attributes {"ttg.num-warps" = 8 : i32} {
// CHECK-LABEL: @load_store_16x32bx1_broadcast
tt.func private @load_store_16x32bx1_broadcast(%arg0: !ttg.memdesc<16x8xi8, #tmem_scales, #ttng.tensor_memory, mutable>, %arg1: tensor<16x8xi8, #linear>) {
  %true = arith.constant true
  // CHECK: @$0 tcgen05.st.sync.aligned.16x32bx2.x1.b32 [$1 + 0], 0, {$2}
  ttng.tmem_store %arg1, %arg0, %true : tensor<16x8xi8, #linear> -> !ttg.memdesc<16x8xi8, #tmem_scales, #ttng.tensor_memory, mutable>
  tt.return
}
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `load_store_16x32bx1_broadcast`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `load_store_16x32bx1_broadcast`。

### Line 1057
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1059-1060
```mlir
#linear = #ttg.linear<{register = [[0, 1]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [0, 0]], block = []}>
#tmem_scales = #ttng.tensor_memory_scales_encoding<>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#linear`, `#tmem_scales`. They parameterize later tests with compact names for `#ttg.linear`, `#ttng.tensor_memory_scales_encoding`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#linear`, `#tmem_scales`。它们为后续测试提供紧凑名称，用来表示 `#ttg.linear`, `#ttng.tensor_memory_scales_encoding`。

### Lines 1062-1071
```mlir
module attributes {"ttg.num-warps" = 4 : i32} {
// CHECK-LABEL: @load_store_16x2_scales_uses_zero_second_half
tt.func private @load_store_16x2_scales_uses_zero_second_half(%arg0: !ttg.memdesc<16x2xi8, #tmem_scales, #ttng.tensor_memory, mutable>, %arg1: tensor<16x2xi8, #linear>) {
  %true = arith.constant true
  // CHECK: @$0 tcgen05.st.sync.aligned.16x32bx2.x1.unpack::16b.b32 [$1 + 0], 0, {$2}
  // CHECK: llvm.return
  ttng.tmem_store %arg1, %arg0, %true : tensor<16x2xi8, #linear> -> !ttg.memdesc<16x2xi8, #tmem_scales, #ttng.tensor_memory, mutable>
  tt.return
}
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `load_store_16x2_scales_uses_zero_second_half`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `load_store_16x2_scales_uses_zero_second_half`。

### Line 1072
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1073-1088
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:100", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_st
  // CHECK: nvg.tensor_memory_base
  // CHECK: tcgen05.st.sync.aligned.32x32b.x128.b32
  // CHECK: nvvm.tcgen05.wait <store>
  tt.func public @tensor_memory_st(%arg0: !tt.ptr<f16>, %arg1: !tt.ptr<f16>, %arg2: !tt.ptr<f16>) {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
    %0 = ttng.tmem_alloc {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : () -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %true = arith.constant true
    ttng.tmem_store %cst_0, %0, %true : tensor<128x128xf32, #blocked1> -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_st`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_st`。

### Line 1090
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1092-1107
```mlir
// Test basic reduction with min
// The reduction output has 1 value per thread per message
#blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked_red = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:103", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_red_min
  // CHECK: tcgen05.ld.red.sync.aligned.32x32b.{{x[0-9]+}}.min.f32
  // CHECK: tcgen05.wait <load>
  tt.func public @tensor_memory_ld_red_min() {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<min>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>, tensor<128xf32, #blocked_red>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:103`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_red_min`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:103`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_red_min`。

### Line 1109
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1111-1125
```mlir
// Test basic reduction with max
#blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked_red = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:103", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_red_max
  // CHECK: tcgen05.ld.red.sync.aligned.32x32b.{{x[0-9]+}}.max.f32
  // CHECK: tcgen05.wait <load>
  tt.func public @tensor_memory_ld_red_max() {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<max>} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>, tensor<128xf32, #blocked_red>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:103`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_red_max`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:103`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_red_max`。

### Line 1127
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1129-1143
```mlir
// Test reduction with abs min
#blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked_red = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:103", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_red_min_abs
  // CHECK: tcgen05.ld.red.sync.aligned.32x32b.{{x[0-9]+}}.min.abs.f32
  // CHECK: tcgen05.wait <load>
  tt.func public @tensor_memory_ld_red_min_abs() {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<min>, abs = true} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>, tensor<128xf32, #blocked_red>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:103`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_red_min_abs`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:103`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_red_min_abs`。

### Line 1145
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1147-1161
```mlir
// Test reduction with NaN max
#blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked_red = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:103", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_red_max_nan
  // CHECK: tcgen05.ld.red.sync.aligned.32x32b.{{x[0-9]+}}.max.NaN.f32
  // CHECK: tcgen05.wait <load>
  tt.func public @tensor_memory_ld_red_max_nan() {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<max>, NaN = true} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>, tensor<128xf32, #blocked_red>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:103`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_red_max_nan`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:103`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_red_max_nan`。

### Line 1163
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1165-1179
```mlir
// Test reduction with abs and NaN max
#blocked1 = #ttg.blocked<{sizePerThread = [1, 128], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked_red = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 128, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:103", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_red_max_abs_nan
  // CHECK: tcgen05.ld.red.sync.aligned.32x32b.{{x[0-9]+}}.max.abs.NaN.f32
  // CHECK: tcgen05.wait <load>
  tt.func public @tensor_memory_ld_red_max_abs_nan() {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #blocked1>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x128xf32, #blocked1>) -> !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable>
    %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<max>, abs = true, NaN = true} : !ttg.memdesc<128x128xf32, #tmem, #ttng.tensor_memory, mutable> -> tensor<128x128xf32, #blocked1>, tensor<128xf32, #blocked_red>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:103`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_red_max_abs_nan`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:103`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_red_max_abs_nan`。

### Line 1181
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1183-1199
```mlir
// Test reduction with 8 warps using 256x64 shape (all warps contribute to M)
// With 8 warps on 256x64: 8 warps cover 256 rows (32 each), each thread handles 64 columns
// Reduction produces 256 values - 8 warps * 32 threads = 256 elements, 1 per thread
#blocked_8w = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [8, 1], order = [0, 1]}>
#blocked_red_8w = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
#tmem_8w = #ttng.tensor_memory_encoding<blockM = 128, blockN = 64, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:103", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_red_min_8_warps
  // CHECK: tcgen05.ld.red.sync.aligned.32x32b.{{x[0-9]+}}.min.f32
  // CHECK: tcgen05.wait <load>
  tt.func public @tensor_memory_ld_red_min_8_warps() {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<256x64xf32, #blocked_8w>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<256x64xf32, #blocked_8w>) -> !ttg.memdesc<256x64xf32, #tmem_8w, #ttng.tensor_memory, mutable>
    %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<min>} : !ttg.memdesc<256x64xf32, #tmem_8w, #ttng.tensor_memory, mutable> -> tensor<256x64xf32, #blocked_8w>, tensor<256xf32, #blocked_red_8w>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:103`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_red_min_8_warps`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:103`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_red_min_8_warps`。

### Line 1201
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1203-1219
```mlir
// Test reduction with blockM=128, blockN=256, 4 warps
// Each thread handles 256 columns -> 4 messages (x64 each) -> 4 partial reductions combined
// Uses llvm.minnum.f32 to combine partial reductions (ignores NaN)
#blocked_256N_4w = #ttg.blocked<{sizePerThread = [1, 256], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked_red_256N_4w = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#tmem_256N = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:103", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_red_min_128x256_4_warps
  // CHECK-COUNT-4: tcgen05.ld.red.sync.aligned.32x32b.x64.min.f32
  // CHECK: tcgen05.wait <load>
  // CHECK-COUNT-3: llvm.intr.minnum
  tt.func public @tensor_memory_ld_red_min_128x256_4_warps() {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked_256N_4w>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x256xf32, #blocked_256N_4w>) -> !ttg.memdesc<128x256xf32, #tmem_256N, #ttng.tensor_memory, mutable>
    %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<min>} : !ttg.memdesc<128x256xf32, #tmem_256N, #ttng.tensor_memory, mutable> -> tensor<128x256xf32, #blocked_256N_4w>, tensor<128xf32, #blocked_red_256N_4w>
    tt.return
  }
```
**EN:** This module sets kernel-level metadata targeting `cuda:103`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_red_min_128x256_4_warps`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:103`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_red_min_128x256_4_warps`。

### Lines 1221-1231
```mlir
  // CHECK-LABEL: @tensor_memory_ld_red_max_128x256_4_warps
  // CHECK-COUNT-4: tcgen05.ld.red.sync.aligned.32x32b.x64.max.f32
  // CHECK: tcgen05.wait <load>
  // CHECK-COUNT-3: llvm.intr.maxnum
  tt.func public @tensor_memory_ld_red_max_128x256_4_warps() {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked_256N_4w>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x256xf32, #blocked_256N_4w>) -> !ttg.memdesc<128x256xf32, #tmem_256N, #ttng.tensor_memory, mutable>
    %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<max>} : !ttg.memdesc<128x256xf32, #tmem_256N, #ttng.tensor_memory, mutable> -> tensor<128x256xf32, #blocked_256N_4w>, tensor<128xf32, #blocked_red_256N_4w>
    tt.return
  }
}
```
**EN:** This block defines `tensor_memory_ld_red_max_128x256_4_warps` and exercises operations such as `tcgen05.ld.red.sync.aligned`, `x64.max.f32`, `tcgen05.wait`, `llvm.intr.maxnum`, `tt.func`. Embedded check comments (CHECK×1, CHECK-COUNT×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `tensor_memory_ld_red_max_128x256_4_warps`，并覆盖 如 `tcgen05.ld.red.sync.aligned`、`x64.max.f32`、`tcgen05.wait`、`llvm.intr.maxnum`、`tt.func` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-COUNT×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1233
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1235-1250
```mlir
// Test reduction with blockM=128, blockN=256, 4 warps WITH NaN propagation
// Uses llvm.minimum.f32 to combine partial reductions (propagates NaN)
#blocked_256N_4w_nan = #ttg.blocked<{sizePerThread = [1, 256], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
#blocked_red_256N_4w_nan = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#tmem_256N_nan = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 1>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 65544 : i32, ttg.target = "cuda:103", ttg.tensor_memory_size = 128 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @tensor_memory_ld_red_min_128x256_4_warps_nan
  // CHECK-COUNT-4: tcgen05.ld.red.sync.aligned.32x32b.x64.min.NaN.f32
  // CHECK: tcgen05.wait <load>
  // CHECK-COUNT-3: llvm.intr.minimum
  tt.func public @tensor_memory_ld_red_min_128x256_4_warps_nan() {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked_256N_4w_nan>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x256xf32, #blocked_256N_4w_nan>) -> !ttg.memdesc<128x256xf32, #tmem_256N_nan, #ttng.tensor_memory, mutable>
    %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<min>, NaN = true} : !ttg.memdesc<128x256xf32, #tmem_256N_nan, #ttng.tensor_memory, mutable> -> tensor<128x256xf32, #blocked_256N_4w_nan>, tensor<128xf32, #blocked_red_256N_4w_nan>
    tt.return
  }
```
**EN:** This module sets kernel-level metadata targeting `cuda:103`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tensor_memory_ld_red_min_128x256_4_warps_nan`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:103`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tensor_memory_ld_red_min_128x256_4_warps_nan`。

### Lines 1252-1262
```mlir
  // CHECK-LABEL: @tensor_memory_ld_red_max_128x256_4_warps_nan
  // CHECK-COUNT-4: tcgen05.ld.red.sync.aligned.32x32b.x64.max.NaN.f32
  // CHECK: tcgen05.wait <load>
  // CHECK-COUNT-3: llvm.intr.maximum
  tt.func public @tensor_memory_ld_red_max_128x256_4_warps_nan() {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #blocked_256N_4w_nan>
    %0 = ttng.tmem_alloc %cst_0 {tensor_memory_col_offset = 0 : i32, tensor_memory_row_offset = 0 : i32} : (tensor<128x256xf32, #blocked_256N_4w_nan>) -> !ttg.memdesc<128x256xf32, #tmem_256N_nan, #ttng.tensor_memory, mutable>
    %result, %red = ttng.tmem_load %0 {redOp = #ttng.redOp<max>, NaN = true} : !ttg.memdesc<128x256xf32, #tmem_256N_nan, #ttng.tensor_memory, mutable> -> tensor<128x256xf32, #blocked_256N_4w_nan>, tensor<128xf32, #blocked_red_256N_4w_nan>
    tt.return
  }
}
```
**EN:** This block defines `tensor_memory_ld_red_max_128x256_4_warps_nan` and exercises operations such as `tcgen05.ld.red.sync.aligned`, `x64.max.NaN.f32`, `tcgen05.wait`, `llvm.intr.maximum`, `tt.func`. Embedded check comments (CHECK×1, CHECK-COUNT×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `tensor_memory_ld_red_max_128x256_4_warps_nan`，并覆盖 如 `tcgen05.ld.red.sync.aligned`、`x64.max.NaN.f32`、`tcgen05.wait`、`llvm.intr.maximum`、`tt.func` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-COUNT×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on LLVM lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 LLVM 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--convert-triton-gpu-to-llvm=compute-capability=100`, `-cse`.  
  **CN:** `RUN` 流水线会驱动 `--convert-triton-gpu-to-llvm=compute-capability=100`, `-cse` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `llvm` (LLVM dialect IR), `nvvm` (NVVM backend intrinsics), `tt` (Triton core ops), `ttng` (next-gen TritonGPU ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`llvm`（LLVM 方言 IR）、`nvvm`（NVVM 后端内建）、`tt`（Triton 核心操作）、`ttng`（新一代 TritonGPU 操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `llvm`, `nvvm`, `tt`, `ttng`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`llvm`、`nvvm`、`tt`、`ttng`、`arith`。
