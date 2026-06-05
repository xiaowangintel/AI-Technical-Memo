# reduce_tree_vectorize.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/reduce_tree_vectorize.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises AMD-specific lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的AMD 专用降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx90a -cse | FileCheck %s --check-prefix=GFX90A`<br>`triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 -cse | FileCheck %s --check-prefix=GFX942`<br>`triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 -cse | FileCheck %s --check-prefix=GFX950`<br>`triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 -cse | FileCheck %s --check-prefix=GFX1250` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Validation / 验证内容:** **EN:** The file validates that AMD-specific lowering runs successfully under the specified pass pipeline. **CN:** 该文件验证 AMD 专用降级 能在指定的 pass 流水线下成功运行。

## Line-by-Line Analysis / 逐行分析

### Lines 1-4
```mlir
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx90a -cse | FileCheck %s --check-prefix=GFX90A
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 -cse | FileCheck %s --check-prefix=GFX942
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 -cse | FileCheck %s --check-prefix=GFX950
// RUN: triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 -cse | FileCheck %s --check-prefix=GFX1250
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx90a -cse | FileCheck %s --check-prefix=GFX90A; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 -cse | FileCheck %s --check-prefix=GFX942; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 -cse | FileCheck %s --check-prefix=GFX950; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 -cse | FileCheck %s --check-prefix=GFX1250` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx90a -cse | FileCheck %s --check-prefix=GFX90A; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 -cse | FileCheck %s --check-prefix=GFX942; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 -cse | FileCheck %s --check-prefix=GFX950; triton-opt %s -split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 -cse | FileCheck %s --check-prefix=GFX1250`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 6-19
```mlir
#blocked_reduce = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 64], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // GFX942-LABEL: reduce_f16
  // GFX942: llvm.fadd {{.*}} : vector<2xf16>
  // GFX950-LABEL: reduce_f16
  // GFX950: llvm.fadd {{.*}} : vector<2xf16>
  tt.func public @reduce_f16(%arg0: tensor<1x256xf16, #blocked_reduce>) {
    %0 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f16, %b: f16):
      %sum = arith.addf %a, %b : f16
      tt.reduce.return %sum : f16
    }) : (tensor<1x256xf16, #blocked_reduce>) -> tensor<1xf16, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.return
  }
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_f16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_f16`。

### Lines 21-35
```mlir
  // GFX90A-LABEL: reduce_f32
  // GFX90A: llvm.fadd {{.*}} : vector<2xf32>
  // GFX942-LABEL: reduce_f32
  // GFX942: llvm.fadd {{.*}} : vector<2xf32>
  // GFX950-LABEL: reduce_f32
  // GFX950: llvm.fadd {{.*}} : vector<2xf32>
  tt.func public @reduce_f32(%arg0: tensor<1x256xf32, #blocked_reduce>) {
    %0 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f32, %b: f32):
      %sum = arith.addf %a, %b : f32
      tt.reduce.return %sum : f32
    }) : (tensor<1x256xf32, #blocked_reduce>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.return
  }
}
```
**EN:** This block defines `reduce_f32` and exercises operations such as `llvm.fadd`, `tt.func`, `tt.reduce`, `arith.addf`, `tt.reduce.return`.
**CN:** 这一块定义了 `reduce_f32`，并覆盖 如 `llvm.fadd`、`tt.func`、`tt.reduce`、`arith.addf`、`tt.reduce.return` 这样的操作。

### Line 37
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 39-50
```mlir
#blocked_reduce = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // GFX1250-LABEL: reduce_f16_tree_vectorize
  // GFX1250: llvm.fadd {{.*}} : vector<2xf16>
  tt.func public @reduce_f16_tree_vectorize(%arg0: tensor<1x128xf16, #blocked_reduce>) {
    %0 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f16, %b: f16):
      %sum = arith.addf %a, %b : f16
      tt.reduce.return %sum : f16
    }) : (tensor<1x128xf16, #blocked_reduce>) -> tensor<1xf16, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.return
  }
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_f16_tree_vectorize`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_f16_tree_vectorize`。

### Lines 52-61
```mlir
  // GFX1250-LABEL: reduce_f32_tree_vectorize
  // GFX1250: llvm.fadd {{.*}} : vector<2xf32>
  tt.func public @reduce_f32_tree_vectorize(%arg0: tensor<1x128xf32, #blocked_reduce>) {
    %0 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f32, %b: f32):
      %sum = arith.addf %a, %b : f32
      tt.reduce.return %sum : f32
    }) : (tensor<1x128xf32, #blocked_reduce>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.return
  }
```
**EN:** This block defines `reduce_f32_tree_vectorize` and exercises operations such as `llvm.fadd`, `tt.func`, `tt.reduce`, `arith.addf`, `tt.reduce.return`.
**CN:** 这一块定义了 `reduce_f32_tree_vectorize`，并覆盖 如 `llvm.fadd`、`tt.func`、`tt.reduce`、`arith.addf`、`tt.reduce.return` 这样的操作。

### Lines 63-64
```mlir
  // Ternary tree reduction for max/min: generates a chain of 3 dependent ops
  // per group so LLVM can fold into v_maximum3/v_minimum3/v_max3/v_min3.
```
**EN:** This comment block provides context for the surrounding test logic: `// Ternary tree reduction for max/min: generates a chain of 3 dependent ops`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Ternary tree reduction for max/min: generates a chain of 3 dependent ops`。

### Lines 66-77
```mlir
  // GFX1250-LABEL: reduce_maximum_f32_ternary
  // GFX1250: %[[A:.*]] = llvm.intr.maximum(%{{.*}}, %{{.*}}) : (f32, f32) -> f32
  // GFX1250-NEXT: %[[B:.*]] = llvm.intr.maximum(%[[A]], %{{.*}}) : (f32, f32) -> f32
  // GFX1250-NEXT: %[[C:.*]] = llvm.intr.maximum(%[[B]], %{{.*}}) : (f32, f32) -> f32
  tt.func public @reduce_maximum_f32_ternary(%arg0: tensor<1x128xf32, #blocked_reduce>) {
    %0 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f32, %b: f32):
      %max = arith.maximumf %a, %b : f32
      tt.reduce.return %max : f32
    }) : (tensor<1x128xf32, #blocked_reduce>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.return
  }
```
**EN:** This block defines `reduce_maximum_f32_ternary` and exercises operations such as `llvm.intr.maximum`, `tt.func`, `tt.reduce`, `arith.maximumf`, `tt.reduce.return`.
**CN:** 这一块定义了 `reduce_maximum_f32_ternary`，并覆盖 如 `llvm.intr.maximum`、`tt.func`、`tt.reduce`、`arith.maximumf`、`tt.reduce.return` 这样的操作。

### Lines 79-90
```mlir
  // GFX1250-LABEL: reduce_minimum_f32_ternary
  // GFX1250: %[[A:.*]] = llvm.intr.minimum(%{{.*}}, %{{.*}}) : (f32, f32) -> f32
  // GFX1250-NEXT: %[[B:.*]] = llvm.intr.minimum(%[[A]], %{{.*}}) : (f32, f32) -> f32
  // GFX1250-NEXT: %[[C:.*]] = llvm.intr.minimum(%[[B]], %{{.*}}) : (f32, f32) -> f32
  tt.func public @reduce_minimum_f32_ternary(%arg0: tensor<1x128xf32, #blocked_reduce>) {
    %0 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f32, %b: f32):
      %min = arith.minimumf %a, %b : f32
      tt.reduce.return %min : f32
    }) : (tensor<1x128xf32, #blocked_reduce>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.return
  }
```
**EN:** This block defines `reduce_minimum_f32_ternary` and exercises operations such as `llvm.intr.minimum`, `tt.func`, `tt.reduce`, `arith.minimumf`, `tt.reduce.return`.
**CN:** 这一块定义了 `reduce_minimum_f32_ternary`，并覆盖 如 `llvm.intr.minimum`、`tt.func`、`tt.reduce`、`arith.minimumf`、`tt.reduce.return` 这样的操作。

### Lines 92-103
```mlir
  // GFX1250-LABEL: reduce_maxnum_f32_ternary
  // GFX1250: %[[A:.*]] = llvm.intr.maxnum(%{{.*}}, %{{.*}}) : (f32, f32) -> f32
  // GFX1250-NEXT: %[[B:.*]] = llvm.intr.maxnum(%[[A]], %{{.*}}) : (f32, f32) -> f32
  // GFX1250-NEXT: %[[C:.*]] = llvm.intr.maxnum(%[[B]], %{{.*}}) : (f32, f32) -> f32
  tt.func public @reduce_maxnum_f32_ternary(%arg0: tensor<1x128xf32, #blocked_reduce>) {
    %0 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f32, %b: f32):
      %max = arith.maxnumf %a, %b : f32
      tt.reduce.return %max : f32
    }) : (tensor<1x128xf32, #blocked_reduce>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.return
  }
```
**EN:** This block defines `reduce_maxnum_f32_ternary` and exercises operations such as `llvm.intr.maxnum`, `tt.func`, `tt.reduce`, `arith.maxnumf`, `tt.reduce.return`.
**CN:** 这一块定义了 `reduce_maxnum_f32_ternary`，并覆盖 如 `llvm.intr.maxnum`、`tt.func`、`tt.reduce`、`arith.maxnumf`、`tt.reduce.return` 这样的操作。

### Lines 105-117
```mlir
  // GFX1250-LABEL: reduce_minnum_f32_ternary
  // GFX1250: %[[A:.*]] = llvm.intr.minnum(%{{.*}}, %{{.*}}) : (f32, f32) -> f32
  // GFX1250-NEXT: %[[B:.*]] = llvm.intr.minnum(%[[A]], %{{.*}}) : (f32, f32) -> f32
  // GFX1250-NEXT: %[[C:.*]] = llvm.intr.minnum(%[[B]], %{{.*}}) : (f32, f32) -> f32
  tt.func public @reduce_minnum_f32_ternary(%arg0: tensor<1x128xf32, #blocked_reduce>) {
    %0 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f32, %b: f32):
      %min = arith.minnumf %a, %b : f32
      tt.reduce.return %min : f32
    }) : (tensor<1x128xf32, #blocked_reduce>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.return
  }
}
```
**EN:** This block defines `reduce_minnum_f32_ternary` and exercises operations such as `llvm.intr.minnum`, `tt.func`, `tt.reduce`, `arith.minnumf`, `tt.reduce.return`.
**CN:** 这一块定义了 `reduce_minnum_f32_ternary`，并覆盖 如 `llvm.intr.minnum`、`tt.func`、`tt.reduce`、`arith.minnumf`、`tt.reduce.return` 这样的操作。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on AMD-specific lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 AMD 专用降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx90a`, `-cse`, `--check-prefix=GFX90A`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--check-prefix=GFX942`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--check-prefix=GFX950`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250`, `--check-prefix=GFX1250`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx90a`, `-cse`, `--check-prefix=GFX90A`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--check-prefix=GFX942`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--check-prefix=GFX950`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250`, `--check-prefix=GFX1250` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `llvm` (LLVM dialect IR), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`llvm`（LLVM 方言 IR）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `llvm`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`llvm`、`tt`、`arith`。
