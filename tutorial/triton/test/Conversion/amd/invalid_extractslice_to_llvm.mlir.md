# invalid_extractslice_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/invalid_extractslice_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises AMD-specific lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的AMD 专用降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt -split-input-file %s --convert-triton-amdgpu-to-llvm='gfx-arch=gfx942' -verify-diagnostics` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: none; diagnostics annotations: error×11. **CN:** FileCheck 标记为 none；诊断标注为 error×11。
- **Validation / 验证内容:** **EN:** The file mainly validates that AMD-specific lowering rejects invalid inputs or reports the expected diagnostics. **CN:** 该文件主要验证 AMD 专用降级 在遇到非法输入时会拒绝它们，或给出预期的诊断信息。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt -split-input-file %s --convert-triton-amdgpu-to-llvm='gfx-arch=gfx942' -verify-diagnostics
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt -split-input-file %s --convert-triton-amdgpu-to-llvm='gfx-arch=gfx942' -verify-diagnostics` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt -split-input-file %s --convert-triton-amdgpu-to-llvm='gfx-arch=gfx942' -verify-diagnostics`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-11
```mlir
// Invalid size
#blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_size_input(%arg0: tensor<256x128xi32, #blocked1> {tt.divisibility = 16 : i32}) {
    // expected-error @+1 {{Lane and warp dim basis must match between source and destination layout.}}
    %1 = amdg.extract_slice %arg0 [0,0] : tensor<256x128xi32, #blocked1> to tensor<256x2xi32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_size_input`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_size_input`。

### Line 13
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 15-23
```mlir
// Invalid offset, not multiple of shapePerTile
#blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_offset_input(%arg0: tensor<256x128xi32, #blocked1> {tt.divisibility = 16 : i32}) {
    // expected-error @+1 {{No source register holds the element for destination index [0, 5]}}
    %1 = amdg.extract_slice %arg0 [0,5] : tensor<256x128xi32, #blocked1> to tensor<256x16xi32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_offset_input`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_offset_input`。

### Line 24
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 26-34
```mlir
// Invalid offset, out of bounds for dimension
#blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_offset_input(%arg0: tensor<256x128xi32, #blocked1> {tt.divisibility = 16 : i32}) {
    // expected-error @+1 {{invalid offset at dimension 1}}
    %1 = amdg.extract_slice %arg0 [0,128] : tensor<256x128xi32, #blocked1> to tensor<256x16xi32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_offset_input`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_offset_input`。

### Line 36
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 38-47
```mlir
// Invalid result layout
#blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_result_layout(%arg0: tensor<256x128xi32, #blocked1> {tt.divisibility = 16 : i32}) {
    // expected-error @+1 {{No source register holds the element for destination index [128, 0]}}
    %1 = amdg.extract_slice %arg0 [0,0] : tensor<256x128xi32, #blocked1> to tensor<256x16xi32, #blocked2>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_result_layout`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_result_layout`。

### Line 49
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 51-59
```mlir
// Invalid result element type
#blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_result_element_type(%arg0: tensor<256x128xi32, #blocked1> {tt.divisibility = 16 : i32}) {
    // expected-error @+1 {{result element type must match source element type}}
    %1 = amdg.extract_slice %arg0 [0,0] : tensor<256x128xi32, #blocked1> to tensor<256x16xi64, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_result_element_type`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_result_element_type`。

### Line 61
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 63-71
```mlir
// Invalid result rank
#blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_result_rank(%arg0: tensor<256x128xi32, #blocked1> {tt.divisibility = 16 : i32}) {
    // expected-error @+1 {{result rank must be equal to source rank}}
    %1 = amdg.extract_slice %arg0 [0,0] : tensor<256x128xi32, #blocked1> to tensor<256x16x2xi32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_result_rank`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_result_rank`。

### Line 73
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 75-83
```mlir
// Invalid result shape
#blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_result_rank(%arg0: tensor<256x128xi32, #blocked1> {tt.divisibility = 16 : i32}) {
    // expected-error @+1 {{result shape cannot exceed source shape at dimension 1}}
    %1 = amdg.extract_slice %arg0 [0,0] : tensor<256x128xi32, #blocked1> to tensor<256x256xi32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_result_rank`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_result_rank`。

### Line 85
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 87-96
```mlir
// Invalid non static offset
#blocked1 = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [4, 16], warpsPerCTA = [8, 1], order = [1, 0]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_non_static_offset(%arg0: tensor<256x128xi32, #blocked1> {tt.divisibility = 16 : i32}, %arg1: i32) {
    // expected-error @+2 {{expected ']'}}
    // expected-error @+1 {{expected integer value}}
    %2 = amdg.extract_slice %arg0 [%arg1, 0] : tensor<256x128xi32, #blocked1> to tensor<256x16xi32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_non_static_offset`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_non_static_offset`。

### Line 98
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 100-109
```mlir
// Invalid layout 1
#dst_layout = #ttg.linear<{register=[[0, 1], [0, 2], [0, 8], [0, 16], [0, 64], [64, 0]], lane=[[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 4]], warp=[[0, 32], [32, 0]], block=[]}>
#src_layout = #ttg.linear<{register=[[0, 1], [0, 2], [0, 8], [0, 16], [0, 64], [0, 128], [64, 0], [128, 0]], lane=[[1, 0], [2, 0], [4, 0], [8, 0], [16, 0], [0, 4], [0, 0]], warp=[[0, 32], [32, 0]], block=[]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_lane_warp_basis(%arg0: tensor<256x256xi32, #src_layout> {tt.divisibility = 16 : i32}) {
    // expected-error @+1 {{Lane and warp dim basis must match between source and destination layout}}
    %2 = amdg.extract_slice %arg0 [0, 0] : tensor<256x256xi32, #src_layout> to tensor<128x128xi32, #dst_layout>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_lane_warp_basis`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_lane_warp_basis`。

### Line 111
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 113-123
```mlir
// Invalid layout 2
// Case when src and dst layouts have same CTA tile shape, but different number of registers
#src_layout = #ttg.linear<{register=[[1, 0], [2, 0]], lane=[[4, 0], [8, 0], [16, 0], [0, 1], [0, 2], [0, 4]], warp=[[0, 0], [0, 8]], block=[]}>
#dst_layout = #ttg.linear<{register=[[1, 0]], lane=[[4, 0], [8, 0], [16, 0], [0, 1], [0, 2], [0, 4]], warp=[[2, 0], [0, 8]], block=[]}>
module attributes {"ttg.compute-capability" = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @invalid_concat(%arg0: tensor<64x32xi32, #src_layout>) {
    // expected-error @+1 {{Lane and warp dim basis must match between source and destination layout}}
    %1 = amdg.extract_slice %arg0 [0, 0] : tensor<64x32xi32, #src_layout> to tensor<32x16xi32, #dst_layout>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `invalid_concat`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.compute-capability`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `invalid_concat`。

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
