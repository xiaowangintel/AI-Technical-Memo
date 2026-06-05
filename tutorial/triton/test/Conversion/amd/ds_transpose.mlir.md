# ds_transpose.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/ds_transpose.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises transpose lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的转置降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --convert-builtin-func-to-llvm | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×1, CHECK-COUNT×72, CHECK-DAG×3, CHECK-LABEL×52, CHECK-NEXT×6, CHECK-NOT×55; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×1, CHECK-COUNT×72, CHECK-DAG×3, CHECK-LABEL×52, CHECK-NEXT×6, CHECK-NOT×55；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that transpose lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 转置降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --convert-builtin-func-to-llvm | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --convert-builtin-func-to-llvm | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 --convert-builtin-func-to-llvm | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-10
```mlir
#mma16 = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 2], instrShape = [16, 16, 32], isTransposed = true}>
#mma32 = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 2], instrShape = [32, 32, 16], isTransposed = true}>
#mma32_scaled = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 2], instrShape = [32, 32, 64], isTransposed = true}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1]}>
#shared1 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#padding = #ttg.padded_shared<[512:+16] {order = [0, 1], shape = [128, 64]}>
#padding_vec1 = #ttg.padded_shared<[1:+4] {order = [0, 1], shape = [128, 64]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma16`, `#mma32`, `#mma32_scaled`, `#shared`, `#shared1`, `#padding`, `#padding_vec1`, `#smem`. They parameterize later tests with compact names for `#ttg.amd_mfma`, `#ttg.swizzled_shared`, `#ttg.padded_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma16`, `#mma32`, `#mma32_scaled`, `#shared`, `#shared1`, `#padding`, `#padding_vec1`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.amd_mfma`, `#ttg.swizzled_shared`, `#ttg.padded_shared`, `#ttg.shared_memory`。

### Lines 12-16
```mlir
#linear_ds_tr_tile_out = #ttg.linear<{register = [[0, 1], [0, 2], [0, 8], [0, 4]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [32, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
#linear_ds_tr_tile_invalid = #ttg.linear<{register = [[0, 1], [0, 2], [0, 8], [0, 4]], lane = [[1, 0], [4, 0], [2, 0], [8, 0], [32, 0], [16, 0]], warp = [[0, 0], [0, 0]], block = []}>
#linear_ds_tr_complex_8contig = #ttg.linear<{register = [[0, 64], [16, 0], [0, 1], [32, 0], [0, 2], [0, 4], [64, 0], [0, 8]], lane = [[1, 0], [2, 0], [4, 0], [0, 16], [8, 0], [0, 32]], warp = [[0, 0], [0, 0]], block = []}>
#linear_ds_tr_complex_4contig = #ttg.linear<{register = [[0, 64], [16, 0], [0, 1], [32, 0], [0, 2], [0, 4], [64, 0], [0, 8]], lane = [[1, 0], [2, 0], [0, 16], [4, 0], [8, 0], [0, 32]], warp = [[0, 0], [0, 0]], block = []}>
#linear_ds_tr_complex_novec = #ttg.linear<{register = [[0, 64], [16, 0], [0, 1], [32, 0], [0, 2], [0, 4], [64, 0], [0, 8]], lane = [[2, 0], [1, 0], [4, 0], [0, 16], [8, 0], [0, 32]], warp = [[0, 0], [0, 0]], block = []}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#linear_ds_tr_tile_out`, `#linear_ds_tr_tile_invalid`, `#linear_ds_tr_complex_8contig`, `#linear_ds_tr_complex_4contig`, `#linear_ds_tr_complex_novec`. They parameterize later tests with compact names for `#ttg.linear`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#linear_ds_tr_tile_out`, `#linear_ds_tr_tile_invalid`, `#linear_ds_tr_complex_8contig`, `#linear_ds_tr_complex_4contig`, `#linear_ds_tr_complex_novec`。它们为后续测试提供紧凑名称，用来表示 `#ttg.linear`。

### Lines 18-24
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  //  CHECK-LABEL: ds_transpose_n_t_fp16_mfma_16
  tt.func @ds_transpose_n_t_fp16_mfma_16(%arg0: !ttg.memdesc<128x64xf16, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared1, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-32: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared1, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `ds_transpose_n_t_fp16_mfma_16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `ds_transpose_n_t_fp16_mfma_16`。

### Lines 26-31
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 33-38
```mlir
  //  CHECK-LABEL: ds_transpose_n_t_fp16_mfma_16_small_kWidth
  tt.func @ds_transpose_n_t_fp16_mfma_16_small_kWidth(%arg0: !ttg.memdesc<128x64xf16, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared1, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-32: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 4}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared1, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 4}>>
```
**EN:** This block defines `ds_transpose_n_t_fp16_mfma_16_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_t_fp16_mfma_16_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 40-45
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 4}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 4}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 4}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 4}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 47-53
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_fp16_mfma_16
  tt.func @ds_transpose_t_t_fp16_mfma_16(%arg0: !ttg.memdesc<128x64xf16, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared1, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<8xf16>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared1, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    // CHECK-COUNT-16: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared1, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_t_t_fp16_mfma_16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_fp16_mfma_16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 55-60
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 62-68
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_fp16_mfma_16_small_kWdith
  tt.func @ds_transpose_t_t_fp16_mfma_16_small_kWdith(%arg0: !ttg.memdesc<128x64xf16, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared1, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<8xf16>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared1, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    // CHECK-COUNT-16: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared1, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 4}>>
```
**EN:** This block defines `ds_transpose_t_t_fp16_mfma_16_small_kWdith` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_fp16_mfma_16_small_kWdith`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 70-75
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 4}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 4}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 77-83
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_fp16_mfma_16
  tt.func @ds_transpose_n_n_fp16_mfma_16(%arg0: !ttg.memdesc<128x64xf16, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<8xf16>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_n_n_fp16_mfma_16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_fp16_mfma_16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 85-90
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 92-98
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_fp16_mfma_16_small_kWidth
  tt.func @ds_transpose_n_n_fp16_mfma_16_small_kWidth(%arg0: !ttg.memdesc<128x64xf16, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 4}>>
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<8xf16>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_n_n_fp16_mfma_16_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_fp16_mfma_16_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 100-105
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 4}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 4}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 107-111
```mlir
  //  CHECK-LABEL: ds_transpose_t_n_fp16_mfma_16
  tt.func @ds_transpose_t_n_fp16_mfma_16(%arg0: !ttg.memdesc<128x64xf16, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-NOT: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared1, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_t_n_fp16_mfma_16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_n_fp16_mfma_16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 113-118
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 120-125
```mlir
  //  CHECK-LABEL: ds_transpose_n_t_fp16_mfma32
  tt.func @ds_transpose_n_t_fp16_mfma32(%arg0: !ttg.memdesc<128x64xf16, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared1, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-32: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared1, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_n_t_fp16_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_t_fp16_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 127-132
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 134-139
```mlir
  //  CHECK-LABEL: ds_transpose_n_t_fp16_mfma32_small_kWidth
  tt.func @ds_transpose_n_t_fp16_mfma32_small_kWidth(%arg0: !ttg.memdesc<128x64xf16, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared1, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-32: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 4}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared1, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 4}>>
```
**EN:** This block defines `ds_transpose_n_t_fp16_mfma32_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_t_fp16_mfma32_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 141-146
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 4}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 4}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 4}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 4}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 148-154
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_fp16_mfma32
  tt.func @ds_transpose_t_t_fp16_mfma32(%arg0: !ttg.memdesc<128x64xf16, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared1, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<8xf16>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared1, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    // CHECK-COUNT-16: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared1, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_t_t_fp16_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_fp16_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 156-161
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 163-169
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_fp16_mfma32_small_kWidth
  tt.func @ds_transpose_t_t_fp16_mfma32_small_kWidth(%arg0: !ttg.memdesc<128x64xf16, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared1, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<8xf16>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared1, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    // CHECK-COUNT-16: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared1, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 4}>>
```
**EN:** This block defines `ds_transpose_t_t_fp16_mfma32_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_fp16_mfma32_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 171-176
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 4}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 4}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 178-184
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_fp16_mfma32
  tt.func @ds_transpose_n_n_fp16_mfma32(%arg0: !ttg.memdesc<128x64xf16, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<8xf16>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_n_n_fp16_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_fp16_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 186-191
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 193-199
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_fp16_mfma32_small_kWidth
  tt.func @ds_transpose_n_n_fp16_mfma32_small_kWidth(%arg0: !ttg.memdesc<128x64xf16, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 4}>>
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<8xf16>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_n_n_fp16_mfma32_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_fp16_mfma32_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 201-206
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 4}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 4}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 208-212
```mlir
  //  CHECK-LABEL: ds_transpose_t_n_fp16_mfma32
  tt.func @ds_transpose_t_n_fp16_mfma32(%arg0: !ttg.memdesc<128x64xf16, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xf16, #shared, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-NOT: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #shared1, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xf16, #shared, #smem, mutable> -> tensor<64x128xf16, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_t_n_fp16_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_n_fp16_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 214-219
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 221-226
```mlir
  //  CHECK-LABEL: ds_transpose_n_t_i8_mfma_16
  tt.func @ds_transpose_n_t_i8_mfma_16(%arg0: !ttg.memdesc<128x64xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared1, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared1, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_t_i8_mfma_16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_t_i8_mfma_16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 228-233
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 235-240
```mlir
  //  CHECK-LABEL: ds_transpose_n_t_i8_mfma_16_small_kWidth
  tt.func @ds_transpose_n_t_i8_mfma_16_small_kWidth(%arg0: !ttg.memdesc<128x64xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared1, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared1, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_n_t_i8_mfma_16_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_t_i8_mfma_16_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 242-247
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 249-255
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_i8_mfma_16
  tt.func @ds_transpose_t_t_i8_mfma_16(%arg0: !ttg.memdesc<128x64xi8, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared1, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-4: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared1, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    // CHECK-COUNT-8: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared1, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_t_t_i8_mfma_16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_i8_mfma_16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 257-262
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 264-270
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_i8_mfma_16_small_kWidth
  tt.func @ds_transpose_t_t_i8_mfma_16_small_kWidth(%arg0: !ttg.memdesc<128x64xi8, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared1, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-4: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared1, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    // CHECK-COUNT-8: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared1, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_t_t_i8_mfma_16_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_i8_mfma_16_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 272-277
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 279-285
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_i8_mfma_16
  tt.func @ds_transpose_n_n_i8_mfma_16(%arg0: !ttg.memdesc<128x64xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    // CHECK-COUNT-4: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_n_i8_mfma_16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_i8_mfma_16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 287-292
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 294-300
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_i8_mfma_16_small_kWidth
  tt.func @ds_transpose_n_n_i8_mfma_16_small_kWidth(%arg0: !ttg.memdesc<128x64xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    // CHECK-COUNT-4: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_n_i8_mfma_16_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_i8_mfma_16_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 302-307
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 309-313
```mlir
  //  CHECK-LABEL: ds_transpose_t_n_i8_mfma_16
  tt.func @ds_transpose_t_n_i8_mfma_16(%arg0: !ttg.memdesc<128x64xi8, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-NOT: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared1, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_t_n_i8_mfma_16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_n_i8_mfma_16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 315-320
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 322-327
```mlir
  //  CHECK-LABEL: ds_transpose_n_t_i8_mfma32
  tt.func @ds_transpose_n_t_i8_mfma32(%arg0: !ttg.memdesc<128x64xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared1, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared1, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_t_i8_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_t_i8_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 329-334
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 336-341
```mlir
  //  CHECK-LABEL: ds_transpose_n_t_i8_mfma32_small_kWidth
  tt.func @ds_transpose_n_t_i8_mfma32_small_kWidth(%arg0: !ttg.memdesc<128x64xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared1, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared1, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_n_t_i8_mfma32_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_t_i8_mfma32_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 343-348
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 350-356
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_i8_mfma32
  tt.func @ds_transpose_t_t_i8_mfma32(%arg0: !ttg.memdesc<128x64xi8, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared1, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-4: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared1, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    // CHECK-COUNT-8: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared1, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_t_t_i8_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_i8_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 358-363
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 365-371
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_i8_mfma32_small_kWidth
  tt.func @ds_transpose_t_t_i8_mfma32_small_kWidth(%arg0: !ttg.memdesc<128x64xi8, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared1, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-4: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared1, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    // CHECK-COUNT-8: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared1, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_t_t_i8_mfma32_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_i8_mfma32_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 373-378
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 380-386
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_i8_mfma32
  tt.func @ds_transpose_n_n_i8_mfma32(%arg0: !ttg.memdesc<128x64xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    // CHECK-COUNT-4: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_n_i8_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_i8_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 388-393
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 395-401
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_i8_mfma32_small_kWidth
  tt.func @ds_transpose_n_n_i8_mfma32_small_kWidth(%arg0: !ttg.memdesc<128x64xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    // CHECK-COUNT-4: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_n_i8_mfma32_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_i8_mfma32_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 403-408
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 410-414
```mlir
  //  CHECK-LABEL: ds_transpose_t_n_i8_mfma32
  tt.func @ds_transpose_t_n_i8_mfma32(%arg0: !ttg.memdesc<128x64xi8, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<64x128xi8, #shared, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-NOT: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xi8, #shared1, #smem, mutable> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<64x128xi8, #shared, #smem, mutable> -> tensor<64x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_t_n_i8_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_n_i8_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 416-421
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<64x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 423-428
```mlir
  //  CHECK-LABEL: ds_transpose_n_t_fp8_mfma_16
  tt.func @ds_transpose_n_t_fp8_mfma_16(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-32: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_t_fp8_mfma_16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_t_fp8_mfma_16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 430-435
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 437-442
```mlir
  //  CHECK-LABEL: ds_transpose_n_t_fp8_mfma_16_small_kWidth
  tt.func @ds_transpose_n_t_fp8_mfma_16_small_kWidth(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-32: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_n_t_fp8_mfma_16_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_t_fp8_mfma_16_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 444-449
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 451-457
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_fp8_mfma_16
  tt.func @ds_transpose_t_t_fp8_mfma_16(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_t_t_fp8_mfma_16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_fp8_mfma_16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 459-464
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 466-472
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_fp8_mfma_16_small_kWidth
  tt.func @ds_transpose_t_t_fp8_mfma_16_small_kWidth(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_t_t_fp8_mfma_16_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_fp8_mfma_16_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 474-479
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 481-487
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_fp8_mfma_16
  tt.func @ds_transpose_n_n_fp8_mfma_16(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_n_fp8_mfma_16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_fp8_mfma_16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 489-494
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 496-502
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_fp8_mfma_16_small_kWidth
  tt.func @ds_transpose_n_n_fp8_mfma_16_small_kWidth(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_n_fp8_mfma_16_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_fp8_mfma_16_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 504-509
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 511-515
```mlir
  //  CHECK-LABEL: ds_transpose_t_n_fp8_mfma_16
  tt.func @ds_transpose_t_n_fp8_mfma_16(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-NOT: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_t_n_fp8_mfma_16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_n_fp8_mfma_16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 517-522
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 524-529
```mlir
  //  CHECK-LABEL: ds_transpose_n_t_fp8_mfma32
  tt.func @ds_transpose_n_t_fp8_mfma32(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-32: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_t_fp8_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_t_fp8_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 531-536
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 538-543
```mlir
  //  CHECK-LABEL: ds_transpose_n_t_fp8_mfma32_small_kWidth
  tt.func @ds_transpose_n_t_fp8_mfma32_small_kWidth(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-32: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_n_t_fp8_mfma32_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_t_fp8_mfma32_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 545-550
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 552-558
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_fp8_mfma32
  tt.func @ds_transpose_t_t_fp8_mfma32(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_t_t_fp8_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_fp8_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 560-565
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 567-573
```mlir
  //  CHECK-LABEL: ds_transpose_t_t_fp8_mfma32_small_kWidth
  tt.func @ds_transpose_t_t_fp8_mfma32_small_kWidth(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_t_t_fp8_mfma32_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_t_fp8_mfma32_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 575-580
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 582-588
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_fp8_mfma32
  tt.func @ds_transpose_n_n_fp8_mfma32(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_n_fp8_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_fp8_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 590-595
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 597-603
```mlir
  //  CHECK-LABEL: ds_transpose_n_n_fp8_mfma32_small_kWidth
  tt.func @ds_transpose_n_n_fp8_mfma32_small_kWidth(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-16: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    // CHECK-COUNT-8: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<16xi8>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_n_n_fp8_mfma32_small_kWidth` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_n_n_fp8_mfma32_small_kWidth`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 605-610
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 8}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 612-616
```mlir
  //  CHECK-LABEL: ds_transpose_t_n_fp8_mfma32
  tt.func @ds_transpose_t_n_fp8_mfma32(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg2: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-NOT: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared1, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
```
**EN:** This block defines `ds_transpose_t_n_fp8_mfma32` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_n_fp8_mfma32`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 618-623
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 625-635
```mlir
  //  CHECK-LABEL: ds_transpose_fp4_mfma_32
  tt.func @ds_transpose_fp4_mfma_32(%arg0: !ttg.memdesc<128x128xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x128xi8, #shared1, #smem, mutable>, %arg2: !ttg.memdesc<128x128xf32, #shared1, #smem, mutable>) {
    // CHECK-COUNT-32: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xi8, #shared, #smem, mutable> -> tensor<128x128xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32_scaled, kWidth = 16}>>
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xi8, #shared1, #smem, mutable> -> tensor<128x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32_scaled, kWidth = 16}>>
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma32_scaled>
    %3 = tt.dot_scaled %1, %2, %cst_2 lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<128x128xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32_scaled, kWidth = 16}>> * tensor<128x128xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32_scaled, kWidth = 16}>> -> tensor<128x128xf32, #mma32_scaled>
    ttg.local_store %3, %arg2 : tensor<128x128xf32, #mma32_scaled> -> !ttg.memdesc<128x128xf32, #shared1, #smem, mutable>
    tt.return
  }
```
**EN:** This block defines `ds_transpose_fp4_mfma_32` and exercises dot products / matmul. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_fp4_mfma_32`，并覆盖 点积/矩阵乘。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 637-648
```mlir
  //  CHECK-LABEL: ds_transpose_t_fp4_mfma32_small
  tt.func @ds_transpose_t_fp4_mfma32_small(%arg0: !ttg.memdesc<16x64xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<64x16xi8, #shared1, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-4: rocdl.ds.read.tr4.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr4.b64
    %1 = amdg.local_load_packed_tranposed %arg0 : !ttg.memdesc<16x64xi8, #shared, #smem, mutable> -> tensor<32x32xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %2 = amdg.local_load_packed_tranposed %arg1 : !ttg.memdesc<64x16xi8, #shared1, #smem, mutable> -> tensor<32x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<32x32x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<32x32x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<32x32x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<32x32x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block defines `ds_transpose_t_fp4_mfma32_small` and exercises tensor stores. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_fp4_mfma32_small`，并覆盖 张量存储。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 650-661
```mlir
  //  CHECK-LABEL: ds_transpose_t_fp4_mfma16
  tt.func @ds_transpose_t_fp4_mfma16(%arg0: !ttg.memdesc<8x128xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x8xi8, #shared1, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-4: rocdl.ds.read.tr4.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr4.b64
    %1 = amdg.local_load_packed_tranposed %arg0 : !ttg.memdesc<8x128xi8, #shared, #smem, mutable> -> tensor<16x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %2 = amdg.local_load_packed_tranposed %arg1 : !ttg.memdesc<128x8xi8, #shared1, #smem, mutable> -> tensor<64x16xi8, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<16x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<64x16x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<16x64x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<64x16x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma16, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block defines `ds_transpose_t_fp4_mfma16` and exercises tensor stores. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_fp4_mfma16`，并覆盖 张量存储。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 663-674
```mlir
  //  CHECK-LABEL: ds_transpose_t_fp4_mfma32
  tt.func @ds_transpose_t_fp4_mfma32(%arg0: !ttg.memdesc<256x256xi8, #shared, #smem, mutable>, %arg1: !ttg.memdesc<256x256xi8, #shared1, #smem, mutable>, %arg2: !tt.ptr<i8> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-128: rocdl.ds.read.tr4.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr4.b64
    %1 = amdg.local_load_packed_tranposed %arg0 : !ttg.memdesc<256x256xi8, #shared, #smem, mutable> -> tensor<512x128xi8, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %2 = amdg.local_load_packed_tranposed %arg1 : !ttg.memdesc<256x256xi8, #shared1, #smem, mutable> -> tensor<128x512xi8, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    %ptr1 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<512x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    %ptr2 = tt.splat %arg2 : !tt.ptr<i8> -> tensor<128x512x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.store %ptr1, %1 : tensor<512x128x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 0, parent = #mma32, kWidth = 16}>>
    tt.store %ptr2, %2 : tensor<128x512x!tt.ptr<i8>, #ttg.dot_op<{opIdx = 1, parent = #mma32, kWidth = 16}>>
    tt.return
  }
```
**EN:** This block defines `ds_transpose_t_fp4_mfma32` and exercises tensor stores. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_t_fp4_mfma32`，并覆盖 张量存储。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 676-680
```mlir
  //  CHECK-LABEL: ds_transpose_ll
  tt.func @ds_transpose_ll(%arg0: !ttg.memdesc<64x16xbf16, #shared, #smem>, %arg1: !tt.ptr<bf16>) {
    // CHECK-COUNT-4: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xbf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %a1 = ttg.local_load %arg0 : !ttg.memdesc<64x16xbf16, #shared, #smem> -> tensor<64x16xbf16, #linear_ds_tr_tile_out>
```
**EN:** This block defines `ds_transpose_ll` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `rocdl.ds.read.tr16.b64`, `ttg.local_load`. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_ll`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`rocdl.ds.read.tr16.b64`、`ttg.local_load` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 682-685
```mlir
    %ptr1 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<64x16x!tt.ptr<bf16>, #linear_ds_tr_tile_out>
    tt.store %ptr1, %a1 : tensor<64x16x!tt.ptr<bf16>, #linear_ds_tr_tile_out>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 687-690
```mlir
  //  CHECK-LABEL: ds_transpose_ll_invalid
  tt.func @ds_transpose_ll_invalid(%arg0: !ttg.memdesc<64x16xbf16, #shared, #smem>, %arg1: !tt.ptr<bf16>) {
    %a1 = ttg.local_load %arg0 : !ttg.memdesc<64x16xbf16, #shared, #smem> -> tensor<64x16xbf16, #linear_ds_tr_tile_invalid>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
```
**EN:** This block defines `ds_transpose_ll_invalid` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `ttg.local_load`, `rocdl.ds.read.tr16.b64`. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_ll_invalid`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`ttg.local_load`、`rocdl.ds.read.tr16.b64` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 692-695
```mlir
    %ptr1 = tt.splat %arg1 : !tt.ptr<bf16> -> tensor<64x16x!tt.ptr<bf16>, #linear_ds_tr_tile_invalid>
    tt.store %ptr1, %a1 : tensor<64x16x!tt.ptr<bf16>, #linear_ds_tr_tile_invalid>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 697-711
```mlir
  //  CHECK-LABEL: ds_transpose_with_padding
  tt.func @ds_transpose_with_padding(%arg0: !ttg.memdesc<128x64xf16, #padding, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-DAG:  [[CST0:%.*]] = llvm.mlir.constant(0 : i32) : i32
    // CHECK-DAG:  [[SHIFT_AMT0:%.*]] = llvm.mlir.constant(10 : i32) : i32
    // CHECK-DAG:  [[SHIFT_AMT1:%.*]] = llvm.mlir.constant(5 : i32) : i32
    // CHECK:      [[SHR:%.*]]  = llvm.lshr [[OFF:%.*]], [[SHIFT_AMT0]] : i32
    // CHECK-NEXT: [[SHL:%.*]]  = llvm.shl [[SHR]], [[SHIFT_AMT1]] : i32
    // CHECK-NEXT: [[ADD0:%.*]] = llvm.add [[SHL:%.*]], [[CST0:%.*]] : i32
    // CHECK-NEXT: [[ADD1:%.*]] = llvm.add [[OFF]], [[ADD0:%.*]] : i32
    // CHECK-NEXT: [[ADD2:%.*]] = llvm.add [[ADD1]], [[CST0]] : i32
    // CHECK-NEXT: [[GEP:%.*]] = llvm.getelementptr inbounds [[BASE:%.*]]{{\[}}[[ADD2]]] : (!llvm.ptr<3>, i32) -> !llvm.ptr<3>, i8
    // CHECK-NEXT: [[RESULT:%.*]] = rocdl.ds.read.tr16.b64 [[GEP]] : <3> -> vector<4xf16>
    // CHECK-COUNT-15: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #padding, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_with_padding` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK×1, CHECK-COUNT×1, CHECK-DAG×3, CHECK-LABEL×1, CHECK-NEXT×6, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_with_padding`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-COUNT×1, CHECK-DAG×3, CHECK-LABEL×1, CHECK-NEXT×6, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 713-716
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 718-721
```mlir
  //  CHECK-LABEL: ds_transpose_padding_interval_too_small
  tt.func @ds_transpose_padding_interval_too_small(%arg0: !ttg.memdesc<128x64xf16, #padding_vec1, #smem, mutable>, %arg2: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x64xf16, #padding_vec1, #smem, mutable> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
```
**EN:** This block defines `ds_transpose_padding_interval_too_small` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_padding_interval_too_small`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 723-726
```mlir
    %ptr1 = tt.splat %arg2 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.store %ptr1, %1 : tensor<128x64x!tt.ptr<f16>, #ttg.dot_op<{opIdx = 0, parent = #mma16, kWidth = 8}>>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 728-737
```mlir
  //  CHECK-LABEL: ds_transpose_complex_ll_b8
  tt.func @ds_transpose_complex_ll_b8(%arg0: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg2: !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable>, %arg3: !tt.ptr<f8E4M3FN> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-256: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<1xi8>
    // CHECK-NOT: llvm.load
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #linear_ds_tr_complex_4contig>
    // CHECK-COUNT-32: rocdl.ds.read.tr8.b64 %{{.*}} : <3> -> vector<2xi32>
    // CHECK-NOT: rocdl.ds.read.tr8.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #linear_ds_tr_complex_8contig>
    // CHECK-COUNT-128: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<1xi8>
    %3 = ttg.local_load %arg2 : !ttg.memdesc<128x128xf8E4M3FN, #shared, #smem, mutable> -> tensor<128x128xf8E4M3FN, #linear_ds_tr_complex_novec>
```
**EN:** This block defines `ds_transpose_complex_ll_b8` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×3, CHECK-LABEL×1, CHECK-NOT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_complex_ll_b8`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×3, CHECK-LABEL×1, CHECK-NOT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 739-746
```mlir
    %ptr1 = tt.splat %arg3 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #linear_ds_tr_complex_4contig>
    %ptr2 = tt.splat %arg3 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #linear_ds_tr_complex_8contig>
    %ptr3 = tt.splat %arg3 : !tt.ptr<f8E4M3FN> -> tensor<128x128x!tt.ptr<f8E4M3FN>, #linear_ds_tr_complex_novec>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f8E4M3FN>, #linear_ds_tr_complex_4contig>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f8E4M3FN>, #linear_ds_tr_complex_8contig>
    tt.store %ptr3, %3 : tensor<128x128x!tt.ptr<f8E4M3FN>, #linear_ds_tr_complex_novec>
    tt.return
  }
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 748-758
```mlir
  //  CHECK-LABEL: ds_transpose_complex_ll_b16
  tt.func @ds_transpose_complex_ll_b16(%arg0: !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, %arg1: !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, %arg2: !ttg.memdesc<128x128xf16, #shared, #smem, mutable>, %arg3: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    // CHECK-COUNT-64: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %1 = ttg.local_load %arg0 : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #linear_ds_tr_complex_4contig>
    // CHECK-COUNT-256: llvm.load %{{.*}} : !llvm.ptr<3> -> vector<1xf16>
    // CHECK-NOT: llvm.load
    %3 = ttg.local_load %arg2 : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #linear_ds_tr_complex_novec>
    // CHECK-COUNT-64: rocdl.ds.read.tr16.b64 %{{.*}} : <3> -> vector<4xf16>
    // CHECK-NOT: rocdl.ds.read.tr16.b64
    %2 = ttg.local_load %arg1 : !ttg.memdesc<128x128xf16, #shared, #smem, mutable> -> tensor<128x128xf16, #linear_ds_tr_complex_8contig>
```
**EN:** This block defines `ds_transpose_complex_ll_b16` and exercises operations such as `tt.func`, `ttg.memdesc`, `tt.ptr`, `tt.divisibility`, `tt.pointer_range`. Embedded check comments (CHECK-COUNT×3, CHECK-LABEL×1, CHECK-NOT×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ds_transpose_complex_ll_b16`，并覆盖 如 `tt.func`、`ttg.memdesc`、`tt.ptr`、`tt.divisibility`、`tt.pointer_range` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×3, CHECK-LABEL×1, CHECK-NOT×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 760-768
```mlir
    %ptr1 = tt.splat %arg3 : !tt.ptr<f16> -> tensor<128x128x!tt.ptr<f16>, #linear_ds_tr_complex_4contig>
    %ptr2 = tt.splat %arg3 : !tt.ptr<f16> -> tensor<128x128x!tt.ptr<f16>, #linear_ds_tr_complex_8contig>
    %ptr3 = tt.splat %arg3 : !tt.ptr<f16> -> tensor<128x128x!tt.ptr<f16>, #linear_ds_tr_complex_novec>
    tt.store %ptr1, %1 : tensor<128x128x!tt.ptr<f16>, #linear_ds_tr_complex_4contig>
    tt.store %ptr2, %2 : tensor<128x128x!tt.ptr<f16>, #linear_ds_tr_complex_8contig>
    tt.store %ptr3, %3 : tensor<128x128x!tt.ptr<f16>, #linear_ds_tr_complex_novec>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on transpose lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 转置降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--convert-builtin-func-to-llvm`.  
  **CN:** `RUN` 流水线会驱动 `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--convert-builtin-func-to-llvm` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `llvm` (LLVM dialect IR), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `llvm`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`llvm`、`arith`。
