# invalid_concat_op.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/invalid_concat_op.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises AMD-specific lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的AMD 专用降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt -split-input-file %s --convert-triton-amdgpu-to-llvm='gfx-arch=gfx942' -verify-diagnostics` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: none; diagnostics annotations: error×8. **CN:** FileCheck 标记为 none；诊断标注为 error×8。
- **Validation / 验证内容:** **EN:** The file mainly validates that AMD-specific lowering rejects invalid inputs or reports the expected diagnostics. **CN:** 该文件主要验证 AMD 专用降级 在遇到非法输入时会拒绝它们，或给出预期的诊断信息。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt -split-input-file %s --convert-triton-amdgpu-to-llvm='gfx-arch=gfx942' -verify-diagnostics
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt -split-input-file %s --convert-triton-amdgpu-to-llvm='gfx-arch=gfx942' -verify-diagnostics` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt -split-input-file %s --convert-triton-amdgpu-to-llvm='gfx-arch=gfx942' -verify-diagnostics`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-15
```mlir
// Invalid ranks
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_concat(
    %arg0: tensor<32x64xf32, #blocked>,
    %arg1: tensor<32x64xf32, #blocked>,
    %arg2: tensor<32x64xf32, #blocked>,
    %arg3: tensor<32x64xf32, #blocked>,
    %arg4: tensor<32x64xf32, #blocked>,
    %arg5: tensor<32x64xf32, #blocked>,
    %arg6: tensor<32x64xf32, #blocked>,
    %arg7: tensor<32x64xf32, #blocked>) {
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_concat`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_concat`。

### Lines 17-22
```mlir
    // expected-error @+1 {{Source and destination tensors must have the same rank.}}
    %1 = amdg.concat %arg0, %arg1, %arg2, %arg3, %arg4, %arg5, %arg6, %arg7:
    tensor<32x64xf32, #blocked>,tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked> -> tensor<256xf32, #blocked>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `amdg.concat`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `amdg.concat`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 24
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 26-37
```mlir
// Invalid shapes 1
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_concat(
    %arg0: tensor<32x64xf32, #blocked>,
    %arg1: tensor<32x64xf32, #blocked>,
    %arg2: tensor<32x64xf32, #blocked>,
    %arg3: tensor<32x64xf32, #blocked>,
    %arg4: tensor<32x64xf32, #blocked>,
    %arg5: tensor<32x64xf32, #blocked>,
    %arg6: tensor<32x64xf32, #blocked>,
    %arg7: tensor<32x64xf32, #blocked>) {
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_concat`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_concat`。

### Lines 39-44
```mlir
    // expected-error @+1 {{Source and destination tensor shapes don't match.}}
    %1 = amdg.concat %arg0, %arg1, %arg2, %arg3, %arg4, %arg5, %arg6, %arg7:
    tensor<32x64xf32, #blocked>,tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked> -> tensor<257x128xf32, #blocked>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `amdg.concat`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `amdg.concat`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 46
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 48-59
```mlir
// Invalid shapes 2
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_concat(
    %arg0: tensor<32x64xf32, #blocked>,
    %arg1: tensor<32x64xf32, #blocked>,
    %arg2: tensor<32x64xf32, #blocked>,
    %arg3: tensor<32x64xf32, #blocked>,
    %arg4: tensor<32x64xf32, #blocked>,
    %arg5: tensor<32x64xf32, #blocked>,
    %arg6: tensor<32x64xf32, #blocked>,
    %arg7: tensor<32x64xf32, #blocked>) {
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_concat`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_concat`。

### Lines 61-66
```mlir
    // expected-error @+1 {{Number of source tiles (8) doesn't match required count (16).}}
    %1 = amdg.concat %arg0, %arg1, %arg2, %arg3, %arg4, %arg5, %arg6, %arg7:
    tensor<32x64xf32, #blocked>,tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked> -> tensor<256x128xf32, #blocked>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `amdg.concat`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `amdg.concat`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 69
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 71-83
```mlir
// Invalid shapes 3
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_concat(
    %arg0: tensor<32x64xf32, #blocked>,
    %arg1: tensor<32x64xf32, #blocked>,
    %arg2: tensor<32x64xf32, #blocked>,
    %arg3: tensor<32x64xf32, #blocked>,
    %arg4: tensor<32x64xf32, #blocked>,
    %arg5: tensor<32x64xf32, #blocked>,
    %arg6: tensor<32x64xf32, #blocked>,
    %arg7: tensor<32x64xf32, #blocked>) {
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_concat`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_concat`。

### Lines 85-90
```mlir
    // expected-error @+1 {{No source register holds the element for destination index [16, 0]}}
    %1 = amdg.concat %arg0, %arg1, %arg2, %arg3, %arg4, %arg5, %arg6, %arg7:
    tensor<32x64xf32, #blocked>,tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked> -> tensor<128x128xf32, #blocked1>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `amdg.concat`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `amdg.concat`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 92
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 94-106
```mlir
// Different types
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 16], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_concat(
    %arg0: tensor<32x64xf32, #blocked1>,
    %arg1: tensor<32x64xf32, #blocked>,
    %arg2: tensor<32x64xf32, #blocked>,
    %arg3: tensor<32x64xf32, #blocked>,
    %arg4: tensor<32x64xf32, #blocked>,
    %arg5: tensor<32x64xf32, #blocked>,
    %arg6: tensor<32x64xf32, #blocked>,
    %arg7: tensor<32x64xf32, #blocked>) {
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_concat`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_concat`。

### Lines 108-113
```mlir
    // expected-error @+1 {{All sources must have identical tensor types.}}
    %1 = amdg.concat %arg0, %arg1, %arg2, %arg3, %arg4, %arg5, %arg6, %arg7:
    tensor<32x64xf32, #blocked1>,tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked> -> tensor<128x128xf32, #blocked>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `amdg.concat`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `amdg.concat`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 115
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 117-128
```mlir
// Invalid element types
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_concat(
    %arg0: tensor<32x64xf32, #blocked>,
    %arg1: tensor<32x64xf32, #blocked>,
    %arg2: tensor<32x64xf32, #blocked>,
    %arg3: tensor<32x64xf32, #blocked>,
    %arg4: tensor<32x64xf32, #blocked>,
    %arg5: tensor<32x64xf32, #blocked>,
    %arg6: tensor<32x64xf32, #blocked>,
    %arg7: tensor<32x64xf32, #blocked>) {
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_concat`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_concat`。

### Lines 130-135
```mlir
    // expected-error @+1 {{Element types of sources and destination must match.}}
    %1 = amdg.concat %arg0, %arg1, %arg2, %arg3, %arg4, %arg5, %arg6, %arg7:
    tensor<32x64xf32, #blocked>,tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked>, tensor<32x64xf32, #blocked> -> tensor<256x64xf16, #blocked>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `amdg.concat`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `amdg.concat`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 138
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 140-148
```mlir
// Different layouts 1
#src_layout = #ttg.linear<{register=[[0, 1], [0, 2], [0, 8], [0, 16], [0, 64], [64, 0]], lane=[[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 4]], warp=[[0, 32], [32, 0]], block=[]}>
#dst_layout = #ttg.linear<{register=[[0, 1], [0, 2], [0, 8], [0, 16], [0, 64], [0, 128], [64, 0], [128, 0]], lane=[[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 4], [0, 0]], warp=[[0, 32], [32, 0]], block=[]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_concat(
    %arg0: tensor<128x128xf32, #src_layout>,
    %arg1: tensor<128x128xf32, #src_layout>,
    %arg2: tensor<128x128xf32, #src_layout>,
    %arg3: tensor<128x128xf32, #src_layout>) {
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_concat`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_concat`。

### Lines 150-155
```mlir
    // expected-error @+1 {{Lane and warp dim basis must match between source and destination layout.}}
    %1 = amdg.concat %arg0, %arg1, %arg2, %arg3:
    tensor<128x128xf32, #src_layout>, tensor<128x128xf32, #src_layout>, tensor<128x128xf32, #src_layout>, tensor<128x128xf32, #src_layout> -> tensor<256x256xf32, #dst_layout>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `amdg.concat`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `amdg.concat`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

### Line 157
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 159-168
```mlir
// Different layouts 2
// Case when src and dst layouts have same CTA tile shape, but different number of registers
#src_layout = #ttg.linear<{register=[[1, 0], [2, 0]], lane=[[4, 0], [8, 0], [16, 0], [0, 1], [0, 2], [0, 4]], warp=[[0, 0], [0, 8]], block=[]}>
#dst_layout = #ttg.linear<{register=[[1, 0]], lane=[[4, 0], [8, 0], [16, 0], [0, 1], [0, 2], [0, 4]], warp=[[2, 0], [0, 8]], block=[]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_concat(
    %arg0: tensor<32x16xf32, #src_layout>,
    %arg1: tensor<32x16xf32, #src_layout>,
    %arg2: tensor<32x16xf32, #src_layout>,
    %arg3: tensor<32x16xf32, #src_layout>) {
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_concat`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_concat`。

### Lines 170-175
```mlir
    // expected-error @+1 {{Lane and warp dim basis must match between source and destination layout.}}
    %1 = amdg.concat %arg0, %arg1, %arg2, %arg3:
    tensor<32x16xf32, #src_layout>, tensor<32x16xf32, #src_layout>, tensor<32x16xf32, #src_layout>, tensor<32x16xf32, #src_layout> -> tensor<64x32xf32, #dst_layout>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `amdg.concat`, `tt.return`. Expected-diagnostic annotations (error×1) ensure invalid or unsupported situations fail in a controlled way.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `amdg.concat`、`tt.return` 这样的操作。 期望诊断标注（error×1）保证非法或不支持的情况会以可控方式失败。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on AMD-specific lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 AMD 专用降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--convert-triton-amdgpu-to-llvm='gfx-arch=gfx942'`.  
  **CN:** `RUN` 流水线会驱动 `--convert-triton-amdgpu-to-llvm='gfx-arch=gfx942'` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）。
- **EN:** Expected-diagnostic annotations make negative tests precise and reproducible.  
  **CN:** 期望诊断标注让负向测试更精确、可复现。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`.  
  **CN:** 外部测试工具：`triton-opt`。
- **EN:** IR dialect dependencies: `ttg`, `tt`.  
  **CN:** IR 方言依赖：`ttg`、`tt`。
