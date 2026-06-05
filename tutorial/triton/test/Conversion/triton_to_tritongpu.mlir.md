# triton_to_tritongpu.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/triton_to_tritongpu.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises warp-level behavior in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的warp 级行为相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file -convert-triton-to-tritongpu='target=cuda:80 num-warps=2' | FileCheck %s`<br>`triton-opt %s -split-input-file -convert-triton-to-tritongpu='target=cuda:80 num-warps=2 num-ctas=2' | FileCheck %s --check-prefixes=CHECK-TWO-CTAS` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×25, CHECK-LABEL×9, CHECK-NEXT×2, CHECK-SAME×1, CHECK-TWO×3; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×25, CHECK-LABEL×9, CHECK-NEXT×2, CHECK-SAME×1, CHECK-TWO×3；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that warp-level behavior produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 warp 级行为 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s -split-input-file -convert-triton-to-tritongpu='target=cuda:80 num-warps=2' | FileCheck %s
// RUN: triton-opt %s -split-input-file -convert-triton-to-tritongpu='target=cuda:80 num-warps=2 num-ctas=2' | FileCheck %s --check-prefixes=CHECK-TWO-CTAS
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file -convert-triton-to-tritongpu='target=cuda:80 num-warps=2' | FileCheck %s; triton-opt %s -split-input-file -convert-triton-to-tritongpu='target=cuda:80 num-warps=2 num-ctas=2' | FileCheck %s --check-prefixes=CHECK-TWO-CTAS` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file -convert-triton-to-tritongpu='target=cuda:80 num-warps=2' | FileCheck %s; triton-opt %s -split-input-file -convert-triton-to-tritongpu='target=cuda:80 num-warps=2 num-ctas=2' | FileCheck %s --check-prefixes=CHECK-TWO-CTAS`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-13
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
tt.func @ops() {
  // CHECK: module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {{.*}}
  %a = arith.constant dense<1.00e+00> : tensor<128x32xf16>
  %b = arith.constant dense<2.00e+00> : tensor<32x128xf16>
  %c = arith.constant dense<3.00e+00> : tensor<128x128xf32>
  %0 = tt.dot %a, %b, %c : tensor<128x32xf16> * tensor<32x128xf16> -> tensor<128x128xf32>
  tt.return
}
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:80`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `ops`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:80`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `ops`。

### Line 15
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 17-34
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
tt.func @load_ops(%ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
  // Test if LoadOp is lowered properly (see #771)
  %ptrs = tt.splat %ptr : !tt.ptr<f32> -> tensor<128x!tt.ptr<f32>>
  %mask = arith.constant dense<true> : tensor<128xi1>
  %other = arith.constant dense<0.0e+0> : tensor<128xf32>
  // CHECK: %{{.*}} = tt.load %{{.*}} : {{.*}}
  %a = tt.load %ptrs : tensor<128x!tt.ptr<f32>>
  // CHECK: %{{.*}} = tt.load %{{.*}}, %{{.*}} : {{.*}}
  %b = tt.load %ptrs, %mask : tensor<128x!tt.ptr<f32>>
  // CHECK: %{{.*}} = tt.load %{{.*}}, %{{.*}}, %{{.*}} : {{.*}}
  %c = tt.load %ptrs, %mask, %other : tensor<128x!tt.ptr<f32>>
  tt.store %ptrs, %a : tensor<128x!tt.ptr<f32>>
  tt.store %ptrs, %b : tensor<128x!tt.ptr<f32>>
  tt.store %ptrs, %c : tensor<128x!tt.ptr<f32>>
  tt.return
}
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `load_ops`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `load_ops`。

### Line 36
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 38-72
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
tt.func @reduce_ops(%ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
  // Test if the total number of threadsPerWarp is 32
  // Test if the total number of warps is 2
  // CHECK: #[[blocked0:.*]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [2, 1], order = [1, 0]}>
  // CHECK: #[[blocked1:.*]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [2, 1], order = [1, 0]}>
  // CHECK: #[[blocked2:.*]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [2, 16], warpsPerCTA = [2, 1], order = [1, 0]}>
  // CHECK: module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {{.*}}
  %c0 = arith.constant dense<1.00e+00> : tensor<4x4xf32>
  %c1 = arith.constant dense<2.00e+00> : tensor<8x2xf32>
  %c2 = arith.constant dense<3.00e+00> : tensor<16x16xf32>
  // CHECK: (tensor<4x4xf32, #[[blocked0]]>) -> tensor<4xf32, #ttg.slice<{dim = 0, parent = #[[blocked0]]}>>
  %c0_ = "tt.reduce" (%c0) ({
  ^bb0(%arg1: f32, %arg2: f32):
    %add = arith.addf %arg1, %arg2 : f32
    tt.reduce.return %add : f32
  }) {axis = 0 : i32} : (tensor<4x4xf32>) -> tensor<4xf32>
  // CHECK: (tensor<8x2xf32, #[[blocked1]]>) -> tensor<2xf32, #ttg.slice<{dim = 0, parent = #[[blocked1]]}>
  %c1_ = "tt.reduce" (%c1) ({
  ^bb0(%arg3: f32, %arg4: f32):
    %add = arith.addf %arg3, %arg4 : f32
    tt.reduce.return %add : f32
  }) {axis = 0 : i32} : (tensor<8x2xf32>) -> tensor<2xf32>
  // CHECK: (tensor<8x2xf32, #[[blocked1]]>) -> tensor<8xf32, #ttg.slice<{dim = 1, parent = #[[blocked1]]}>>
  %c2_ = "tt.reduce" (%c1) ({
  ^bb0(%arg5: f32, %arg6: f32):
    %add = arith.addf %arg5, %arg6 : f32
    tt.reduce.return %add : f32
  }) {axis = 1 : i32} : (tensor<8x2xf32>) -> tensor<8xf32>
  // CHECK: (tensor<16x16xf32, #[[blocked2]]>) -> tensor<16xf32, #ttg.slice<{dim = 0, parent = #[[blocked2]]}>>
  %c3_ = "tt.reduce" (%c2) ({
  ^bb0(%arg7: f32, %arg8: f32):
    %add = arith.addf %arg7, %arg8 : f32
    tt.reduce.return %add : f32
  }) {axis = 0 : i32} : (tensor<16x16xf32>) -> tensor<16xf32>
```
**EN:** This module sets kernel-level metadata targeting `cuda:80`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_ops`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:80`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_ops`。

### Lines 74-76
```mlir
  tt.return
}
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 79
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 81-88
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
tt.func public @select_op(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: i1) {
  // CHECK-LABEL: select_op
  %cst = arith.constant dense<0.000000e+00> : tensor<128xf32>
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  %1 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<128x!tt.ptr<f32>>
  %2 = tt.addptr %1, %0 : tensor<128x!tt.ptr<f32>>, tensor<128xi32>
  %3 = tt.load %2 : tensor<128x!tt.ptr<f32>>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `select_op`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `select_op`。

### Lines 90-91
```mlir
  // CHECK: %{{.*}} = arith.select %arg2, %{{.*}}, %{{.*}} : tensor<128xf32, #blocked>
  %4 = arith.select %arg2, %cst, %3 : tensor<128xf32>
```
**EN:** This block defines the test function(s) and exercises predicated selection. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 条件选择。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 93-98
```mlir
  %5 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<128x!tt.ptr<f32>>
  %6 = tt.addptr %5, %0 : tensor<128x!tt.ptr<f32>>, tensor<128xi32>
  tt.store %6, %4 : tensor<128x!tt.ptr<f32>>
  tt.return
}
}
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.addptr`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.addptr`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 100
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 102-104
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
tt.func @arith_splat_bool(%ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
  // CHECK-LABEL: arith_splat_bool
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `arith_splat_bool`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `arith_splat_bool`。

### Lines 106-111
```mlir
  // Test arith.constant with splatted bool.
  // CHECK-NEXT: arith.constant dense<true> : tensor<128xi1, #{{.*}}>
  %mask = arith.constant dense<true> : tensor<128xi1>
  tt.return
}
}
```
**EN:** This block defines the test function(s) and exercises operations such as `arith.constant`, `tt.return`. Embedded check comments (CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `arith.constant`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 113
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 115-122
```mlir
// CHECK-LABEL: gather_op
tt.func @gather_op() {
  %cst = arith.constant dense<1.0> : tensor<128x4xf32>
  %cst_0 = arith.constant dense<1> : tensor<256x4xi32>
  // CHECK: tt.gather %{{.*}}[%{{.*}}] {axis = 0 : i32} : (tensor<128x4xf32, #blocked>, tensor<256x4xi32, #blocked>) -> tensor<256x4xf32, #blocked>
  %0 = tt.gather %cst[%cst_0] {axis = 0 : i32} : (tensor<128x4xf32>, tensor<256x4xi32>) -> tensor<256x4xf32>
  tt.return
}
```
**EN:** This block defines `gather_op` and exercises operations such as `tt.func`, `arith.constant`, `tt.gather`, `tt.return`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `gather_op`，并覆盖 如 `tt.func`、`arith.constant`、`tt.gather`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 124
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 126-127
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
#bar_layout = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`, `#bar_layout`. They parameterize later tests with compact names for `#ttg.nvmma_shared`, `#ttg.swizzled_shared`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`, `#bar_layout`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvmma_shared`, `#ttg.swizzled_shared`。

### Line 129
```mlir
// CHECK: [[SLICE_PARENT:#.*]] = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 2], order = [1, 0]}>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 131-139
```mlir
// CHECK: @gather4_layout
tt.func @gather4_layout(%arg0: !tt.tensordesc<1x128xf32>, %arg1: i32, %arg2: !tt.ptr<f32>) {
  %cst = arith.constant dense<1> : tensor<32xi32>
  // CHECK: [[IDX:%.*]] = ttg.convert_layout %cst : tensor<32xi32, #{{.*}}> -> tensor<32xi32, #ttg.slice<{dim = 0, parent = [[SLICE_PARENT]]}>>
  %0 = tt.descriptor_gather %arg0[%cst, %arg1] : (!tt.tensordesc<1x128xf32>, tensor<32xi32>, i32) -> tensor<32x128xf32>
  %1 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<32x128x!tt.ptr<f32>>
  tt.store %1, %0 : tensor<32x128x!tt.ptr<f32>>
  tt.return
}
```
**EN:** This block defines `gather4_layout` and exercises layout conversion, tensor stores. Embedded check comments (CHECK×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `gather4_layout`，并覆盖 布局转换、张量存储。 其中嵌入的检查注释（CHECK×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 141-149
```mlir
// CHECK: @scatter4_layout
tt.func @scatter4_layout(%arg0: !tt.tensordesc<1x128xf32>, %arg1: i32, %arg2: !tt.ptr<f32>) {
  %cst = arith.constant dense<1> : tensor<32xi32>
  %0 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<32x128x!tt.ptr<f32>>
  %1 = tt.load %0 : tensor<32x128x!tt.ptr<f32>>
  // CHECK: [[IDX:%.*]] = ttg.convert_layout %cst : tensor<32xi32, #{{.*}}> -> tensor<32xi32, #ttg.slice<{dim = 0, parent = [[SLICE_PARENT]]}>>
  tt.descriptor_scatter %arg0[%cst, %arg1], %1 : !tt.tensordesc<1x128xf32>, tensor<32xi32>, i32, tensor<32x128xf32>
  tt.return
}
```
**EN:** This block defines `scatter4_layout` and exercises layout conversion, tensor loads. Embedded check comments (CHECK×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `scatter4_layout`，并覆盖 布局转换、张量加载。 其中嵌入的检查注释（CHECK×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 151
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 153-158
```mlir
// CHECK-LABEL: @ub_poison
tt.func @ub_poison() {
  // CHECK-NEXT: ub.poison : tensor<128x64xf16, #blocked>
  %0 = ub.poison : tensor<128x64xf16>
  tt.return
}
```
**EN:** This block defines `ub_poison` and exercises operations such as `tt.func`, `ub.poison`, `tt.return`. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ub_poison`，并覆盖 如 `tt.func`、`ub.poison`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 160
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 162-171
```mlir
// CHECK-LABEL: @cf_br
tt.func @cf_br(%ptr: !tt.ptr<i32>) {
  %cst = arith.constant dense<1> : tensor<128xi32>
  // cf.br ^bb1(%{{.+}} : tensor<128xi32, #{{.+}}>)
  cf.br ^bb1(%cst : tensor<128xi32>)
^bb1(%arg0: tensor<128xi32>):
  %ptrs = tt.splat %ptr : !tt.ptr<i32> -> tensor<128x!tt.ptr<i32>>
  tt.store %ptrs, %arg0 : tensor<128x!tt.ptr<i32>>
  tt.return
}
```
**EN:** This block defines `cf_br` and exercises tensor stores. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `cf_br`，并覆盖 张量存储。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 173
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 175-184
```mlir
tt.func @split_op(%arg0: !tt.ptr<f32>, %arg1: !tt.ptr<f32>) {
  // CHECK-TWO-CTAS-LABEL: split_op
  // CHECK-TWO-CTAS: tt.split
  %0 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<64x2x!tt.ptr<f32>>
  %1 = tt.load %0 : tensor<64x2x!tt.ptr<f32>>
  %res1, %res2 = tt.split %1 : tensor<64x2xf32> -> tensor<64xf32>
  %2 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>>
  tt.store %2, %res1 : tensor<64x!tt.ptr<f32>>
  tt.return
}
```
**EN:** This block defines `split_op` and exercises tensor loads, tensor stores. Embedded check comments (CHECK-TWO×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `split_op`，并覆盖 张量加载、张量存储。 其中嵌入的检查注释（CHECK-TWO×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 186
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 188-194
```mlir
// CHECK-LABEL: tt.func private @callee
// CHECK-SAME: (%{{.*}}: !tt.ptr<i32>) -> tensor<128xi32, #{{.*}}>
tt.func private @callee(%arg0: !tt.ptr<i32>) -> tensor<128xi32> {
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // CHECK: tt.return %{{.*}} : tensor<128xi32, #{{.*}}>
  tt.return %0 : tensor<128xi32>
}
```
**EN:** This block defines `callee` and exercises range generation. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-SAME×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `callee`，并覆盖 范围生成。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-SAME×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 196-203
```mlir
// CHECK-LABEL: tt.func @caller
tt.func @caller(%ptr: !tt.ptr<i32>) {
  // CHECK: %{{.*}} = tt.call @callee(%{{.*}}) : (!tt.ptr<i32>) -> tensor<128xi32, #{{.*}}>
  %v = tt.call @callee(%ptr) : (!tt.ptr<i32>) -> tensor<128xi32>
  %ptrs = tt.splat %ptr : !tt.ptr<i32> -> tensor<128x!tt.ptr<i32>>
  tt.store %ptrs, %v : tensor<128x!tt.ptr<i32>>
  tt.return
}
```
**EN:** This block defines `caller`, `callee` and exercises tensor stores. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `caller`, `callee`，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 205
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 207-209
```mlir
// When a callee returns a tensor whose default encoding doesn't match what
// the caller's consumer wants, a ttg.convert_layout should be auto-inserted
// at the call boundary.
```
**EN:** This comment block provides context for the surrounding test logic: `// When a callee returns a tensor whose default encoding doesn't match what`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// When a callee returns a tensor whose default encoding doesn't match what`。

### Lines 211-216
```mlir
// CHECK-LABEL: tt.func private @make_a
// CHECK: tt.return %{{.*}} : tensor<128x32xf16, #[[$BLOCKED:[^,>]+]]>
tt.func private @make_a() -> tensor<128x32xf16> {
  %a = arith.constant dense<1.0> : tensor<128x32xf16>
  tt.return %a : tensor<128x32xf16>
}
```
**EN:** This block defines `make_a` and exercises operations such as `tt.func`, `tt.return`, `arith.constant`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `make_a`，并覆盖 如 `tt.func`、`tt.return`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 218-227
```mlir
// CHECK-LABEL: tt.func @call_into_dot
// CHECK: %[[V:.*]] = tt.call @make_a() : () -> tensor<128x32xf16, #[[$BLOCKED]]>
// CHECK: ttg.convert_layout %[[V]] : tensor<128x32xf16, #[[$BLOCKED]]> -> tensor<128x32xf16, #ttg.dot_op<{{.*}}>>
// CHECK: tt.dot
tt.func @call_into_dot(%b: tensor<32x128xf16>) {
  %a = tt.call @make_a() : () -> tensor<128x32xf16>
  %c = arith.constant dense<0.0> : tensor<128x128xf32>
  %0 = tt.dot %a, %b, %c : tensor<128x32xf16> * tensor<32x128xf16> -> tensor<128x128xf32>
  tt.return
}
```
**EN:** This block defines `call_into_dot`, `make_a` and exercises layout conversion, dot products / matmul. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `call_into_dot`, `make_a`，并覆盖 布局转换、点积/矩阵乘。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on warp-level behavior.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 warp 级行为。
- **EN:** The `RUN` pipeline drives tools/passes such as `-convert-triton-to-tritongpu='target=cuda:80`, `--check-prefixes=CHECK-TWO-CTAS`.  
  **CN:** `RUN` 流水线会驱动 `-convert-triton-to-tritongpu='target=cuda:80`, `--check-prefixes=CHECK-TWO-CTAS` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `cf` (control-flow ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`cf`（控制流操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `cf`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`cf`。
