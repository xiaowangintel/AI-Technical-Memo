# allocate_shared_memory.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/allocate_shared_memory.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises memory allocation in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的内存分配相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-amdgpu-shared-memory | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×5, CHECK-LABEL×3, CHECK-NEXT×1; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×5, CHECK-LABEL×3, CHECK-NEXT×1；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that memory allocation produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 内存分配 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --allocate-amdgpu-shared-memory | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-amdgpu-shared-memory | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-amdgpu-shared-memory | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-5
```mlir
#blocked1 = #ttg.blocked<{sizePerThread = [8, 4], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked1`, `#blocked2`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked1`, `#blocked2`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 7-13
```mlir
// This test checks swizzling based converter.
//
// Swizzling converter tries to find swizzling pattern, which provides widest load and store instructions and avoids as much back conflicts as possible.
// Current converter implementation decides that best swizzling patter requires allocation of tile with shape [256, 128], which takes 256*128*4(size of one element) = 131072 bytes
//
// For implementation see mlir::triton::getNumScratchElemsSwizzledCvt function,
// in particular mlir::triton::gpu::optimalSwizzling to get shape of repeat tile.
```
**EN:** This comment block provides context for the surrounding test logic: `// This test checks swizzling based converter.`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// This test checks swizzling based converter.`。

### Lines 15-16
```mlir
// CHECK: ttg.shared = 131072 : i32
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 18-23
```mlir
// CHECK-LABEL: @convert_layout_swizzled
tt.func @convert_layout_swizzled(%arg0: tensor<256x256xi32, #blocked1>) {
  // CHECK-NEXT: allocation.offset = 0 : i32
  %0 = ttg.convert_layout %arg0 : tensor<256x256xi32, #blocked1> -> tensor<256x256xi32, #blocked2>
  tt.return
}
```
**EN:** This block defines `convert_layout_swizzled` and exercises layout conversion. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `convert_layout_swizzled`，并覆盖 布局转换。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 25
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 27
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 29-33
```mlir
// WarpSpecialize with a 2D TensorDesc capture.
// 2D TensorDesc = 12 dwords = 48 bytes.
// Capture size + Warp state buffer size = 48 + 4 = 52 bytes.
// CHECK: ttg.shared = 52 : i32
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.total-num-warps`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.total-num-warps`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 35-46
```mlir
// CHECK-LABEL: @ws_tensordesc_2d_capture
// CHECK: allocation.offset = 48 : i32
tt.func @ws_tensordesc_2d_capture(%desc: !tt.tensordesc<64x64xf16>) {
  ttg.warp_specialize(%desc) attributes {warpGroupStartIds = array<i32: 4>}
  default {
    ttg.warp_yield
  }
  partition0(%arg0: !tt.tensordesc<64x64xf16>) num_warps(4) {
    ttg.warp_return
  } : (!tt.tensordesc<64x64xf16>) -> ()
  tt.return
}
```
**EN:** This block defines `ws_tensordesc_2d_capture` and exercises operations such as `allocation.offset`, `tt.func`, `tt.tensordesc`, `ttg.warp_specialize`, `ttg.warp_yield`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ws_tensordesc_2d_capture`，并覆盖 如 `allocation.offset`、`tt.func`、`tt.tensordesc`、`ttg.warp_specialize`、`ttg.warp_yield` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 48
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 50
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 52-56
```mlir
// WarpSpecialize with a 5D TensorDesc capture.
// 5D TensorDesc = 20 dwords = 80 bytes.
// Capture size + Warp state buffer size = 80 + 4 = 84 bytes.
// CHECK: ttg.shared = 84 : i32
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.total-num-warps`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.total-num-warps`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 58-69
```mlir
// CHECK-LABEL: @ws_tensordesc_5d_capture
// CHECK: allocation.offset = 80 : i32
tt.func @ws_tensordesc_5d_capture(%desc: !tt.tensordesc<8x8x8x16x16xf16>) {
  ttg.warp_specialize(%desc) attributes {warpGroupStartIds = array<i32: 4>}
  default {
    ttg.warp_yield
  }
  partition0(%arg0: !tt.tensordesc<8x8x8x16x16xf16>) num_warps(4) {
    ttg.warp_return
  } : (!tt.tensordesc<8x8x8x16x16xf16>) -> ()
  tt.return
}
```
**EN:** This block defines `ws_tensordesc_5d_capture` and exercises operations such as `allocation.offset`, `tt.func`, `tt.tensordesc`, `ttg.warp_specialize`, `ttg.warp_yield`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ws_tensordesc_5d_capture`，并覆盖 如 `allocation.offset`、`tt.func`、`tt.tensordesc`、`ttg.warp_specialize`、`ttg.warp_yield` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 71
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on memory allocation.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 内存分配。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-amdgpu-shared-memory`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-amdgpu-shared-memory` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`.  
  **CN:** IR 方言依赖：`ttg`、`tt`。
