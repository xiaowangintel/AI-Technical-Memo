# allocate_shared_memory.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Proton/allocate_shared_memory.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises memory allocation in Triton's Proton pipeline coverage. **CN:** 该MLIR 测试用于覆盖 Triton Proton 流水线中的内存分配相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt --split-input-file -allocate-shared-memory -convert-proton-to-protongpu="max-shared-mem-size=4096" -allocate-proton-shared-memory %s | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×6, CHECK-LABEL×3, CHECK-NOT×1; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×6, CHECK-LABEL×3, CHECK-NOT×1；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that memory allocation produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 内存分配 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt --split-input-file -allocate-shared-memory -convert-proton-to-protongpu="max-shared-mem-size=4096" -allocate-proton-shared-memory %s | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt --split-input-file -allocate-shared-memory -convert-proton-to-protongpu="max-shared-mem-size=4096" -allocate-proton-shared-memory %s | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt --split-input-file -allocate-shared-memory -convert-proton-to-protongpu="max-shared-mem-size=4096" -allocate-proton-shared-memory %s | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-19
```mlir
#A_SHARED = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
// CHECK: ttg.shared = 1664 : i32
module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
  // CHECK-LABEL: allocate_aligned
  tt.func @allocate_aligned(%A : !tt.ptr<f16>) {
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  proton.record start "name0"
  %cst1 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %cst2 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst1 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  proton.record end "name0"
  ttg.local_dealloc %cst2 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // CHECK: ttg.local_alloc  {allocation.offset = 1536 : i32}
  tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.num-ctas`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `allocate_aligned`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.num-ctas`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `allocate_aligned`。

### Line 21
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 23-39
```mlir
#A_SHARED = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0], CGALayout = [[1, 0]]}>
// CHECK: ttg.shared = 832 : i32
module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 2 : i32} {
  // CHECK-LABEL: allocate_aligned
  tt.func @allocate_aligned(%A : !tt.ptr<f16>) {
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  proton.record start "name0"
  %cst1 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  %cst2 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst0 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst1 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  proton.record end "name0"
  ttg.local_dealloc %cst2 : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // CHECK: ttg.local_alloc  {allocation.offset = 768 : i32}
  tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.num-ctas`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `allocate_aligned`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.num-ctas`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `allocate_aligned`。

### Line 41
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 43-54
```mlir
#A_SHARED = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
// CHECK: ttg.shared = 64 : i32
module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
  // CHECK-LABEL: no_proton
  tt.func @no_proton(%A : !tt.ptr<f16>) {
  %cst0 = ttg.local_alloc : () -> !ttg.memdesc<1x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.local_dealloc %cst0 : !ttg.memdesc<1x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // CHECK: ttg.local_alloc
  // CHECK-NOT: ttg.local_alloc
  tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.num-ctas`, `ttg.shared`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `no_proton`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.num-ctas`, `ttg.shared`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `no_proton`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's Proton pipeline coverage and focuses on memory allocation.  
  **CN:** 该文件属于 Triton 的 Proton 流水线 测试覆盖，关注点是 内存分配。
- **EN:** The `RUN` pipeline drives tools/passes such as `--split-input-file`, `-allocate-shared-memory`, `-convert-proton-to-protongpu="max-shared-mem-size=4096"`, `-allocate-proton-shared-memory`.  
  **CN:** `RUN` 流水线会驱动 `--split-input-file`, `-allocate-shared-memory`, `-convert-proton-to-protongpu="max-shared-mem-size=4096"`, `-allocate-proton-shared-memory` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `proton` (Proton dialect ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`proton`（Proton 方言操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `proton`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`proton`。
