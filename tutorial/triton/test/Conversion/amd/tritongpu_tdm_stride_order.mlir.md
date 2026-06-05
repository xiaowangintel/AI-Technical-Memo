# tritongpu_tdm_stride_order.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/tritongpu_tdm_stride_order.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises AMD-specific lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的AMD 专用降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --verify-diagnostics | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK-LABEL×3; diagnostics annotations: error×10. **CN:** FileCheck 标记为 CHECK-LABEL×3；诊断标注为 error×10。
- **Validation / 验证内容:** **EN:** The file mainly validates that AMD-specific lowering rejects invalid inputs or reports the expected diagnostics. **CN:** 该文件主要验证 AMD 专用降级 在遇到非法输入时会拒绝它们，或给出预期的诊断信息。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s --split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --verify-diagnostics | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s --split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --verify-diagnostics | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s --split-input-file --allocate-shared-memory --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --verify-diagnostics | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-12
```mlir
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [64, 64]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tdm_load_runtime_strides(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %stride0: i64, %stride1: i64) {
    %c_shape = arith.constant 128 : i32
    // expected-error @+2 {{last dimension must have stride 1}}
    // expected-error @+1 {{failed to legalize operation}}
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%stride0, %stride1] : <f16>, <64x64xf16, #shared>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_load_runtime_strides`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_load_runtime_strides`。

### Line 14
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 16-25
```mlir
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [64, 64]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_1x1_tensor
  tt.func public @tdm_1x1_tensor(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %stride1: i64) {
    %c_shape = arith.constant 1 : i32
    %c_stride1 = arith.constant 1 : i64
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%c_stride1, %c_stride1] : <f16>, <64x64xf16, #shared>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_1x1_tensor`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_1x1_tensor`。

### Line 27
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 29-39
```mlir
#shared = #ttg.padded_shared<[32:+4] {order = [1, 0], shape = [64, 64]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tdm_wrong_stride_order(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %runtime_stride: i64) {
    %c_shape = arith.constant 128 : i32
    %c_stride1 = arith.constant 1 : i64
    // expected-error @+2 {{last dimension must have stride 1}}
    // expected-error @+1 {{failed to legalize operation}}
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%c_stride1, %runtime_stride] : <f16>, <64x64xf16, #shared>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_wrong_stride_order`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_wrong_stride_order`。

### Line 41
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 43-53
```mlir
#shared = #ttg.padded_shared<[32:+4] {order = [0, 1], shape = [64, 64]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tdm_wrong_smem_order(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %runtime_stride: i64) {
    %c_shape = arith.constant 128 : i32
    %c_stride1 = arith.constant 1 : i64
    // expected-error @+2 {{requires shared order [rank-1, rank-2, ..., 0]}}
    // expected-error @+1 {{failed to legalize operation}}
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape], [%runtime_stride, %c_stride1] : <f16>, <64x64xf16, #shared>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_wrong_smem_order`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_wrong_smem_order`。

### Line 55
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 57-67
```mlir
// Positive test case for 1x1x1 tensor
#shared = #ttg.padded_shared<[32:+4] {order = [0, 1, 2], shape = [1, 1, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_1x1x1
  tt.func public @tdm_1x1x1(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) {
    %c_stride1 = arith.constant 1 : i64
    %c_shape = arith.constant 1 : i32
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape, %c_shape], [%c_stride1, %c_stride1, %c_stride1] : <f16>, <1x1x1xf16, #shared>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_1x1x1`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_1x1x1`。

### Line 69
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 71-82
```mlir
// Positive test case for Xx1x1 tensor
#shared = #ttg.padded_shared<[32:+4] {order = [0, 1, 2], shape = [1, 1, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: tdm_Xx1x1
  tt.func public @tdm_Xx1x1(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %runtime_stride: i64) {
    %c_stride1 = arith.constant 1 : i64
    %c_shape = arith.constant 1 : i32
    %c_shape2 = arith.constant 128 : i32
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape2, %c_shape, %c_shape], [%runtime_stride, %c_stride1, %c_stride1] : <f16>, <1x1x1xf16, #shared>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_Xx1x1`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_Xx1x1`。

### Line 84
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 86-97
```mlir
#shared = #ttg.padded_shared<[32:+4] {order = [0, 1, 2], shape = [1, 1, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tdm_1xix1(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %runtime_stride: i64) {
    %c_stride1 = arith.constant 1 : i64
    %c_shape = arith.constant 1 : i32
    %c_shape2 = arith.constant 128 : i32
    // expected-error @+2 {{requires all stride 1 dimensions to be consecutive starting from the last dimension}}
    // expected-error @+1 {{failed to legalize operation}}
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape2, %c_shape], [%c_stride1, %runtime_stride, %c_stride1] : <f16>, <1x1x1xf16, #shared>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_1xix1`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_1xix1`。

### Line 99
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 101-112
```mlir
#shared = #ttg.padded_shared<[32:+4] {order = [0, 1, 2], shape = [1, 1, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @tdm_1xix1(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %runtime_stride: i64) {
    %c_stride1 = arith.constant 1 : i64
    %c_shape = arith.constant 1 : i32
    %c_shape2 = arith.constant 128 : i32
    // expected-error @+2 {{requires all stride 1 dimensions to be consecutive starting from the last dimension}}
    // expected-error @+1 {{failed to legalize operation}}
    %0 = tt.make_tensor_descriptor %arg0, [%c_shape, %c_shape, %c_shape2], [%c_stride1, %runtime_stride, %c_stride1] : <f16>, <1x1x1xf16, #shared>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tdm_1xix1`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tdm_1xix1`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on AMD-specific lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 AMD 专用降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--split-input-file`, `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250`, `--verify-diagnostics`.  
  **CN:** `RUN` 流水线会驱动 `--split-input-file`, `--allocate-shared-memory`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250`, `--verify-diagnostics` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。
- **EN:** Expected-diagnostic annotations make negative tests precise and reproducible.  
  **CN:** 期望诊断标注让负向测试更精确、可复现。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`。
