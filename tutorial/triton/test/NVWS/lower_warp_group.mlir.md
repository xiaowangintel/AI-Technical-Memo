# lower_warp_group.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/NVWS/lower_warp_group.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises warp-level behavior in Triton's NVWS transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton NVWS 变换中的warp 级行为相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt --split-input-file --nvws-lower-warp-group %s | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×5, CHECK-LABEL×3, CHECK-NEXT×14, CHECK-NOT×3; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×5, CHECK-LABEL×3, CHECK-NEXT×14, CHECK-NOT×3；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that warp-level behavior produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 warp 级行为 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt --split-input-file --nvws-lower-warp-group %s | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt --split-input-file --nvws-lower-warp-group %s | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt --split-input-file --nvws-lower-warp-group %s | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-8
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 2>
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 10-35
```mlir
  // CHECK-LABEL: @warp_group
  //       CHECK-NOT: nvws.warp_group
  //       CHECK:   ttg.warp_specialize
  //       CHECK-NEXT:   default
  //       CHECK:   partition0
  //       CHECK-NEXT:   arith.constant
  //       CHECK-NEXT:   ttng.tc_gen5_mma
  tt.func @warp_group(%a: !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>,
                  %b: !ttg.memdesc<128x256xf8E5M2, #shared1, #ttg.shared_memory>,
                  %c: !ttg.memdesc<128x256xf16, #tmem, #ttng.tensor_memory, mutable>,
                  %accUse: i1,
                  %pred: i1,
                  %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>) {
    %false = arith.constant false
    nvws.warp_group
    partition0  num_warps(8) {
      ttng.tc_gen5_mma %a, %b, %c, %accUse, %pred, %barrier[%false] {is_async} :
        !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>,
         !ttg.memdesc<128x256xf8E5M2, #shared1, #ttg.shared_memory>,
         !ttg.memdesc<128x256xf16, #tmem, #ttng.tensor_memory, mutable>,
         !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
        nvws.warp_group.return
      }
    tt.return
  }
}
```
**EN:** This block defines `warp_group` and exercises operations such as `nvws.warp_group`, `ttg.warp_specialize`, `arith.constant`, `ttng.tc_gen5_mma`, `tt.func`. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×3, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warp_group`，并覆盖 如 `nvws.warp_group`、`ttg.warp_specialize`、`arith.constant`、`ttng.tc_gen5_mma`、`tt.func` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×3, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 37
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 39-44
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 2>
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 46-69
```mlir
  // CHECK-LABEL: @warp_default
  //       CHECK-NOT: nvws.warp_group
  //       CHECK:   ttg.warp_specialize
  //       CHECK-NEXT:   default
  //       CHECK-NEXT:   ttng.tc_gen5_mma
  tt.func @warp_default(%a: !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>,
                  %b: !ttg.memdesc<128x256xf8E5M2, #shared1, #ttg.shared_memory>,
                  %c: !ttg.memdesc<128x256xf16, #tmem, #ttng.tensor_memory, mutable>,
                  %accUse: i1,
                  %pred: i1,
                  %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>) {
    %false = arith.constant false
    nvws.warp_group
    partition0  num_warps(4) {
      ttng.tc_gen5_mma %a, %b, %c, %accUse, %pred, %barrier[%false] {is_async} :
         !ttg.memdesc<128x128xf8E5M2, #shared, #ttg.shared_memory>,
         !ttg.memdesc<128x256xf8E5M2, #shared1, #ttg.shared_memory>,
         !ttg.memdesc<128x256xf16, #tmem, #ttng.tensor_memory, mutable>,
         !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
        nvws.warp_group.return
      }
    tt.return
  }
}
```
**EN:** This block defines `warp_default` and exercises operations such as `nvws.warp_group`, `ttg.warp_specialize`, `ttng.tc_gen5_mma`, `tt.func`, `ttg.memdesc`. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warp_default`，并覆盖 如 `nvws.warp_group`、`ttg.warp_specialize`、`ttng.tc_gen5_mma`、`tt.func`、`ttg.memdesc` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 71
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 73-78
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
#shared2 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#acc_tmem = #ttng.tensor_memory_encoding<blockM = 128, blockN = 256, colStride = 2>
#blocked = #ttg.blocked<{sizePerThread = [1, 256], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 80-119
```mlir
  // CHECK-LABEL: @warp_multiple_group
  //       CHECK-NOT: nvws.warp_group
  //       CHECK:   ttg.warp_specialize(%
  //       CHECK-NEXT:   default
  //       CHECK-NEXT:   ttng.tc_gen5_mma
  //       CHECK:   partition0(%
  //       CHECK-NEXT:   arith.constant
  //       CHECK-NEXT:   ttg.local_load
  //       CHECK-NEXT:   ttng.wait_barrier
  //       CHECK-NEXT:   ttng.tmem_load
  //       CHECK-NEXT:   tt.store
  //       CHECK-NEXT:   ttg.warp_return
  //       CHECK-NEXT:   }
  tt.func @warp_multiple_group(%a: !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>,
                  %b: !ttg.memdesc<128x256xf16, #shared1, #ttg.shared_memory>,
                  %c: !ttg.memdesc<128x256xf16, #acc_tmem, #ttng.tensor_memory, mutable>,
                  %d: tensor<128x256x!tt.ptr<f16>, #blocked>,
                  %accUse: i1,
                  %pred: i1,
                  %barrier: !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>) {
    %false = arith.constant false
    %c0 = arith.constant 0 : i32
    nvws.warp_group
    partition0  num_warps(4) {
      ttng.tc_gen5_mma %a, %b, %c, %accUse, %pred, %barrier[%false] {is_async} :
         !ttg.memdesc<128x128xf16, #shared, #ttg.shared_memory>,
         !ttg.memdesc<128x256xf16, #shared1, #ttg.shared_memory>,
         !ttg.memdesc<128x256xf16, #acc_tmem, #ttng.tensor_memory, mutable>,
         !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
        nvws.warp_group.return
      }
    partition1 num_warps(4) {
      ttng.wait_barrier %barrier, %c0 : !ttg.memdesc<1xi64, #shared2, #ttg.shared_memory, mutable>
      %c_reg = ttng.tmem_load %c : !ttg.memdesc<128x256xf16, #acc_tmem, #ttng.tensor_memory, mutable> -> tensor<128x256xf16, #blocked>
      tt.store %d, %c_reg : tensor<128x256x!tt.ptr<f16>, #blocked>
      nvws.warp_group.return
    }
    tt.return
  }
}
```
**EN:** This block defines `warp_multiple_group` and exercises tensor stores. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×9, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warp_multiple_group`，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×9, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's NVWS transforms coverage and focuses on warp-level behavior.  
  **CN:** 该文件属于 Triton 的 NVWS 变换 测试覆盖，关注点是 warp 级行为。
- **EN:** The `RUN` pipeline drives tools/passes such as `--split-input-file`, `--nvws-lower-warp-group`.  
  **CN:** `RUN` 流水线会驱动 `--split-input-file`, `--nvws-lower-warp-group` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `arith` (scalar/tensor arithmetic), `ttng` (next-gen TritonGPU ops), `tt` (Triton core ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`arith`（标量/张量算术）、`ttng`（新一代 TritonGPU 操作）、`tt`（Triton 核心操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `arith`, `ttng`, `tt`.  
  **CN:** IR 方言依赖：`ttg`、`arith`、`ttng`、`tt`。
