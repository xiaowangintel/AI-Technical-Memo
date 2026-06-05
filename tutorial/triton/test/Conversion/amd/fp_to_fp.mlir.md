# fp_to_fp.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/fp_to_fp.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises AMD-specific lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的AMD 专用降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck --check-prefixes=COMMON,GFX942 %s`<br>`triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck --check-prefixes=COMMON,GFX950 %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK-LABEL×9; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK-LABEL×9；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that AMD-specific lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 AMD 专用降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck --check-prefixes=COMMON,GFX942 %s
// RUN: triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck --check-prefixes=COMMON,GFX950 %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck --check-prefixes=COMMON,GFX942 %s; triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck --check-prefixes=COMMON,GFX950 %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck --check-prefixes=COMMON,GFX942 %s; triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck --check-prefixes=COMMON,GFX950 %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-12
```mlir
//  CHECK-LABEL: f16_to_f32
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @f16_to_f32(%arg0: tensor<8x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>) {
    // GFX942-COUNT-8: llvm.fpext %{{.+}} : f16 to f32
    %0 = tt.fp_to_fp %arg0 : tensor<8x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked}>> -> tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `f16_to_f32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `f16_to_f32`。

### Line 14
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 16-24
```mlir
//  CHECK-LABEL: bf16_to_f32
#blocked2 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @bf16_to_f32(%arg0: tensor<8x8xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>) {
    // GFX942-COUNT-8: llvm.bitcast
    %0 = tt.fp_to_fp %arg0 : tensor<8x8xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `bf16_to_f32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `bf16_to_f32`。

### Line 26
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 28-39
```mlir
//  CHECK-LABEL: f32_to_f16
#blocked2 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @f32_to_f16(%arg0: tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>) {
    // GFX942-COUNT-8: llvm.fptrunc %{{.+}} : f32 to f16
    // GFX950-COUNT-4: llvm.fptrunc %{{.+}} : vector<2xf32> to vector<2xf16>
    %0 = tt.fp_to_fp %arg0, rounding = rtne : tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
    // COMMON-COUNT-4: rocdl.cvt.pkrtz
    %1 = tt.fp_to_fp %arg0, rounding = rtz : tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `f32_to_f16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `f32_to_f16`。

### Line 41
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 43-55
```mlir
//  CHECK-LABEL: f32_to_f16_single_value
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  tt.func @f32_to_f16_single_value(%arg0: tensor<1x128xf32, #blocked>) {
    // COMMON: llvm.fptrunc %{{.+}} : f32 to f16
    // COMMON-NOT: llvm.fptrunc
    %0 = tt.fp_to_fp %arg0, rounding = rtne : tensor<1x128xf32, #blocked> -> tensor<1x128xf16, #blocked>
    // COMMON: rocdl.cvt.pkrtz
    // COMMON-NOT: rocdl.cvt.pkrtz
    %1 = tt.fp_to_fp %arg0, rounding = rtz : tensor<1x128xf32, #blocked> -> tensor<1x128xf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `f32_to_f16_single_value`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `f32_to_f16_single_value`。

### Line 57
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 59-69
```mlir
//  CHECK-LABEL: downcast_to_f8
#blocked2 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @downcast_to_f8(%arg0: tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>,
                     %arg1: tensor<8x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>,
                     %arg2: tensor<8x8xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>) {
    // GFX950: rocdl.cvt.scalef32.pk.bf8.f32  %{{.*}}, %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.bf8.f32  %{{.*}}, %{{.*}}, %{{.*}} -> %{{.*}}[true]
    // GFX950: rocdl.cvt.scalef32.pk.bf8.f32  %{{.*}}, %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.bf8.f32  %{{.*}}, %{{.*}}, %{{.*}} -> %{{.*}}[true]
    %0 = tt.fp_to_fp %arg0, rounding = rtne : tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `downcast_to_f8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `downcast_to_f8`。

### Lines 71-75
```mlir
    // GFX950: rocdl.cvt.scalef32.pk.bf8.f16 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.bf8.f16 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    // GFX950: rocdl.cvt.scalef32.pk.bf8.f16 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.bf8.f16 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    %1 = tt.fp_to_fp %arg1, rounding = rtne : tensor<8x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.scalef32.pk.bf8.f16`, `tt.fp_to_fp`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.scalef32.pk.bf8.f16`, `tt.fp_to_fp` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 77-81
```mlir
    // GFX950: rocdl.cvt.scalef32.pk.bf8.bf16 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.bf8.bf16 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    // GFX950: rocdl.cvt.scalef32.pk.bf8.bf16 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.bf8.bf16 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    %2 = tt.fp_to_fp %arg2, rounding = rtne : tensor<8x8xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.scalef32.pk.bf8.bf16`, `tt.fp_to_fp`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.scalef32.pk.bf8.bf16`, `tt.fp_to_fp` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 83-87
```mlir
    // GFX950: rocdl.cvt.scalef32.pk.fp8.f32 %{{.*}}, %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.fp8.f32 %{{.*}}, %{{.*}}, %{{.*}} -> %{{.*}}[true]
    // GFX950: rocdl.cvt.scalef32.pk.fp8.f32 %{{.*}}, %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.fp8.f32 %{{.*}}, %{{.*}}, %{{.*}} -> %{{.*}}[true]
    %3 = tt.fp_to_fp %arg0, rounding = rtne : tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.scalef32.pk.fp8.f32`, `tt.fp_to_fp`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.scalef32.pk.fp8.f32`, `tt.fp_to_fp` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 89-93
```mlir
    // GFX950: rocdl.cvt.scalef32.pk.fp8.f16 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.fp8.f16 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    // GFX950: rocdl.cvt.scalef32.pk.fp8.f16 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.fp8.f16 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    %4 = tt.fp_to_fp %arg1, rounding = rtne : tensor<8x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.scalef32.pk.fp8.f16`, `tt.fp_to_fp`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.scalef32.pk.fp8.f16`, `tt.fp_to_fp` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 95-102
```mlir
    // GFX950: rocdl.cvt.scalef32.pk.fp8.bf16 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.fp8.bf16 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    // GFX950: rocdl.cvt.scalef32.pk.fp8.bf16 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX950: rocdl.cvt.scalef32.pk.fp8.bf16 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    %5 = tt.fp_to_fp %arg2, rounding = rtne : tensor<8x8xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.scalef32.pk.fp8.bf16`, `tt.fp_to_fp`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.scalef32.pk.fp8.bf16`, `tt.fp_to_fp`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 104
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 106-118
```mlir
// CHECK-LABEL: f32_to_bf8
#blocked2 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @downcast_to_bf8(%arg0: tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>) {
    // GFX942: rocdl.cvt.pk.bf8.f32 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX942: rocdl.cvt.pk.bf8.f32 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    // GFX942: rocdl.cvt.pk.bf8.f32 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX942: rocdl.cvt.pk.bf8.f32 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    // GFX950-COUNT-16: llvm.trunc %{{.+}} : i32 to i8
    %6 = tt.fp_to_fp %arg0, rounding = rtne : tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf8E5M2FNUZ, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `downcast_to_bf8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `downcast_to_bf8`。

### Line 120
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 122-134
```mlir
// CHECK-LABEL: f32_to_f8
#blocked2 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @f32_to_f8(%arg0: tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>) {
    // GFX942: rocdl.cvt.pk.fp8.f32 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX942: rocdl.cvt.pk.fp8.f32 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    // GFX942: rocdl.cvt.pk.fp8.f32 %{{.*}}, %{{.*}} -> %{{.*}}[false]
    // GFX942: rocdl.cvt.pk.fp8.f32 %{{.*}}, %{{.*}} -> %{{.*}}[true]
    // GFX950-COUNT-16: llvm.trunc %{{.+}} : i32 to i8
    %7 = tt.fp_to_fp %arg0, rounding = rtne : tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf8E4M3FNUZ, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `f32_to_f8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `f32_to_f8`。

### Line 136
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 138-149
```mlir
//  CHECK-LABEL: upcast_from_f8
#blocked2 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @upcast_from_f8(%arg0: tensor<8x8xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>,
                     %arg1: tensor<8x8xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>,
                     %arg2: tensor<8x8xf8E5M2FNUZ, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>,
                     %arg3: tensor<8x8xf8E4M3FNUZ, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>) {
    // GFX950: rocdl.cvt.scalef32.pk.f32.bf8 %[[VR1:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.f32.bf8 %[[VR1]][true]
    // GFX950: rocdl.cvt.scalef32.pk.f32.bf8 %[[VR2:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.f32.bf8 %[[VR2]][true]
    %0 = tt.fp_to_fp %arg0 : tensor<8x8xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `upcast_from_f8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `upcast_from_f8`。

### Lines 151-155
```mlir
    // GFX950: rocdl.cvt.scalef32.pk.f16.bf8 %[[VR3:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.f16.bf8 %[[VR3]][true]
    // GFX950: rocdl.cvt.scalef32.pk.f16.bf8 %[[VR4:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.f16.bf8 %[[VR4]][true]
    %1 = tt.fp_to_fp %arg0 : tensor<8x8xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.scalef32.pk.f16.bf8`, `tt.fp_to_fp`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.scalef32.pk.f16.bf8`, `tt.fp_to_fp` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 157-161
```mlir
    // GFX950: rocdl.cvt.scalef32.pk.bf16.bf8 %[[VR5:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.bf16.bf8 %[[VR5]][true]
    // GFX950: rocdl.cvt.scalef32.pk.bf16.bf8 %[[VR6:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.bf16.bf8 %[[VR6]][true]
    %2 = tt.fp_to_fp %arg0 : tensor<8x8xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.scalef32.pk.bf16.bf8`, `tt.fp_to_fp`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.scalef32.pk.bf16.bf8`, `tt.fp_to_fp` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 163-167
```mlir
    // GFX950: rocdl.cvt.scalef32.pk.f32.fp8 %[[VR7:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.f32.fp8 %[[VR7]][true]
    // GFX950: rocdl.cvt.scalef32.pk.f32.fp8 %[[VR8:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.f32.fp8 %[[VR8]][true]
    %3 = tt.fp_to_fp %arg1 : tensor<8x8xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.scalef32.pk.f32.fp8`, `tt.fp_to_fp`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.scalef32.pk.f32.fp8`, `tt.fp_to_fp` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 169-173
```mlir
    // GFX950: rocdl.cvt.scalef32.pk.f16.fp8 %[[VR9:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.f16.fp8 %[[VR9]][true]
    // GFX950: rocdl.cvt.scalef32.pk.f16.fp8 %[[VR10:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.f16.fp8 %[[VR10]][true]
    %4 = tt.fp_to_fp %arg1 : tensor<8x8xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.scalef32.pk.f16.fp8`, `tt.fp_to_fp`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.scalef32.pk.f16.fp8`, `tt.fp_to_fp` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 175-179
```mlir
    // GFX950: rocdl.cvt.scalef32.pk.bf16.fp8 %[[VR11:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.bf16.fp8 %[[VR11]][true]
    // GFX950: rocdl.cvt.scalef32.pk.bf16.fp8 %[[VR12:.*]][false]
    // GFX950: rocdl.cvt.scalef32.pk.bf16.fp8 %[[VR12]][true]
    %5 = tt.fp_to_fp %arg1 : tensor<8x8xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xbf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.scalef32.pk.bf16.fp8`, `tt.fp_to_fp`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.scalef32.pk.bf16.fp8`, `tt.fp_to_fp` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 181-185
```mlir
    // GFX942: rocdl.cvt.pk.f32.bf8 %[[VR13:.*]][false]
    // GFX942: rocdl.cvt.pk.f32.bf8 %[[VR13]][true]
    // GFX942: rocdl.cvt.pk.f32.bf8 %[[VR14:.*]][false]
    // GFX942: rocdl.cvt.pk.f32.bf8 %[[VR14]][true]
    %6 = tt.fp_to_fp %arg2 : tensor<8x8xf8E5M2FNUZ, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.pk.f32.bf8`, `tt.fp_to_fp`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.pk.f32.bf8`, `tt.fp_to_fp` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 187-194
```mlir
    // GFX942: rocdl.cvt.pk.f32.fp8 %[[VR15:.*]][false]
    // GFX942: rocdl.cvt.pk.f32.fp8 %[[VR15]][true]
    // GFX942: rocdl.cvt.pk.f32.fp8 %[[VR16:.*]][false]
    // GFX942: rocdl.cvt.pk.f32.fp8 %[[VR16]][true]
    %7 = tt.fp_to_fp %arg3 : tensor<8x8xf8E4M3FNUZ, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `rocdl.cvt.pk.f32.fp8`, `tt.fp_to_fp`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.cvt.pk.f32.fp8`, `tt.fp_to_fp`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 196
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 198-210
```mlir
//  CHECK-LABEL: f8_rtz
#blocked2 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @f8_rtz(%arg0: tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>,
                     %arg1: tensor<8x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>) {
    // GFX950-NOT: rocdl.cvt.scalef32.pk.f32.bf8
    // GFX950-COUNT-4: rocdl.cvt.pkrtz
    %1 = tt.fp_to_fp %arg0, rounding = rtz : tensor<8x8xf32, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
    // GFX950-NOT: rocdl.cvt.scalef32.pk.f16.bf8
    %2 = tt.fp_to_fp %arg1, rounding = rtz : tensor<8x8xf16, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>> -> tensor<8x8xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #blocked2}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `f8_rtz`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `f8_rtz`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on AMD-specific lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 AMD 专用降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--split-input-file`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--check-prefixes=COMMON,GFX942`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--check-prefixes=COMMON,GFX950`.  
  **CN:** `RUN` 流水线会驱动 `--split-input-file`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--check-prefixes=COMMON,GFX942`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`, `--check-prefixes=COMMON,GFX950` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `llvm` (LLVM dialect IR).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `llvm`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`llvm`。
