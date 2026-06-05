# canonicalize.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Triton/canonicalize.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises canonicalization patterns in Triton's Triton dialect canonicalization coverage. **CN:** 该MLIR 测试用于覆盖 Triton Triton 方言规范化中的规范化模式相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file -canonicalize | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×13, CHECK-DAG×3, CHECK-LABEL×19, CHECK-NEXT×24, CHECK-NOT×10; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×13, CHECK-DAG×3, CHECK-LABEL×19, CHECK-NEXT×24, CHECK-NOT×10；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that canonicalization patterns produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 规范化模式 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file -canonicalize | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file -canonicalize | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file -canonicalize | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-12
```mlir
// CHECK-LABEL: dead_load
tt.func @dead_load(%ptr: tensor<32x128x!tt.ptr<f16>>) {
  %mask = arith.constant dense<true> : tensor<32x128xi1>
  %other = arith.constant dense<0.00e+00> : tensor<32x128xf16>
  // CHECK-NOT: tt.load {{.*}}isVolatile = false
  //     CHECK: tt.load {{.*}}isVolatile = true
  %a = tt.load %ptr, %mask, %other : tensor<32x128x!tt.ptr<f16>>
  %b = tt.load %ptr, %mask, %other {isVolatile = true} : tensor<32x128x!tt.ptr<f16>>
  tt.return
}
```
**EN:** This block defines `dead_load` and exercises tensor loads. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `dead_load`，并覆盖 张量加载。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 14
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 16-21
```mlir
// CHECK-LABEL: make_range
tt.func @make_range() -> (tensor<128x1xi32>, tensor<1xi32>) {
  // CHECK-DAG: %[[c:.*]] = arith.constant dense<0> : tensor<128x1xi32>
  %a = tt.make_range {end = 1 : i32, start = 0 : i32} : tensor<1xi32>
  %b = tt.expand_dims %a {axis = 1 : i32} : tensor<1xi32> -> tensor<1x1xi32>
  %c = tt.broadcast %b : tensor<1x1xi32> -> tensor<128x1xi32>
```
**EN:** This block defines `make_range` and exercises range generation. Embedded check comments (CHECK-DAG×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `make_range`，并覆盖 范围生成。 其中嵌入的检查注释（CHECK-DAG×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 23-24
```mlir
  // CHECK-DAG: %[[d:.*]] = arith.constant dense<1> : tensor<1xi32>
  %d = tt.make_range {end = 2 : i32, start = 1 : i32} : tensor<1xi32>
```
**EN:** This block defines the test function(s) and exercises range generation. Embedded check comments (CHECK-DAG×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 范围生成。 其中嵌入的检查注释（CHECK-DAG×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 26-28
```mlir
  // CHECK-DAG: tt.return %[[c]], %[[d]] : tensor<128x1xi32>, tensor<1xi32>
  tt.return %c, %d : tensor<128x1xi32>, tensor<1xi32>
}
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.return`. Embedded check comments (CHECK-DAG×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.return` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 30
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 32-40
```mlir
// CHECK-LABEL: fold_addptr
tt.func @fold_addptr(%arg: tensor<64x64x!tt.ptr<f16>>) -> (tensor<64x64x!tt.ptr<f16>>) {
  // CHECK-NOT: tt.addptr
  // CHECK-NOT: arith.constant
  //     CHECK: tt.return %arg
  %c0_i32 = arith.constant dense<0> : tensor<64x64xi32>
  %0 = tt.addptr %arg, %c0_i32 : tensor<64x64x!tt.ptr<f16>>, tensor<64x64xi32>
  tt.return %0 : tensor<64x64x!tt.ptr<f16>>
}
```
**EN:** This block defines `fold_addptr` and exercises pointer arithmetic. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NOT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `fold_addptr`，并覆盖 指针运算。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NOT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 42
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 44-52
```mlir
// CHECK-LABEL: fold_addptr_scalar
tt.func @fold_addptr_scalar(%arg: !tt.ptr<f16>) -> (!tt.ptr<f16>) {
  // CHECK-NOT: tt.addptr
  // CHECK-NOT: arith.constant
  //     CHECK: tt.return %arg
  %c0_i32 = arith.constant 0 : i32
  %0 = tt.addptr %arg, %c0_i32 : !tt.ptr<f16>, i32
  tt.return %0 : !tt.ptr<f16>
}
```
**EN:** This block defines `fold_addptr_scalar` and exercises pointer arithmetic. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NOT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `fold_addptr_scalar`，并覆盖 指针运算。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NOT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 54
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 56-57
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
#sliced0 = #ttg.slice<{dim = 1, parent = #blocked0}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked0`, `#sliced0`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.slice`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked0`, `#sliced0`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.slice`。

### Lines 59-68
```mlir
// CHECK-LABEL: fn
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
tt.func @fn(%arg0: tensor<1xf32, #sliced0>) -> (tensor<32x1xf32, #blocked0>){
  // CHECK: %[[a:.*]] = tt.expand_dims
  // CHECK: tt.broadcast %[[a]]
  %a = tt.broadcast %arg0 : tensor<1xf32, #sliced0> -> tensor<32xf32, #sliced0>
  %b = tt.expand_dims %a {axis = 1 : i32} : tensor<32xf32, #sliced0> -> tensor<32x1xf32, #blocked0>
  tt.return %b : tensor<32x1xf32, #blocked0>
}
}  // end module
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fn`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fn`。

### Line 70
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 72-82
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  tt.func @fp_to_fp_pos_zero_fold() -> tensor<32x128xf8E4M3FNUZ, #blocked> {
    // CHECK-LABEL: fp_to_fp_pos_zero_fold
    // CHECK-NEXT: %[[cst_folded:.+]] = arith.constant dense<0.000000e+00> : tensor<32x128xf8E4M3FNUZ, #blocked>
    // CHECK-NEXT: tt.return %[[cst_folded]]
    %cst = arith.constant dense<0.00e+00> : tensor<32x128xf32, #blocked>
    %cst_converted = tt.fp_to_fp %cst, rounding = rtne : tensor<32x128xf32, #blocked> -> tensor<32x128xf8E4M3FNUZ, #blocked>
    tt.return %cst_converted : tensor<32x128xf8E4M3FNUZ, #blocked>
  }
}  // end module
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fp_to_fp_pos_zero_fold`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fp_to_fp_pos_zero_fold`。

### Line 84
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 86-95
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  tt.func @fp_to_fp_pos_zero_fold_scalar() -> f8E4M3FNUZ {
    // CHECK-LABEL: fp_to_fp_pos_zero_fold_scalar
    // CHECK-NEXT: %[[cst_folded:.+]] = arith.constant 0.000000e+00 : f8E4M3FNUZ
    // CHECK-NEXT: tt.return %[[cst_folded]]
    %cst = arith.constant 0.00e+00 : f32
    %cst_converted = tt.fp_to_fp %cst, rounding = rtne : f32 -> f8E4M3FNUZ
    tt.return %cst_converted : f8E4M3FNUZ
  }
}  // end module
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fp_to_fp_pos_zero_fold_scalar`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fp_to_fp_pos_zero_fold_scalar`。

### Line 97
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 99-109
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  tt.func @fp_to_fp_neg_zero_fold() -> tensor<32x128xf8E4M3FN, #blocked> {
    // CHECK-LABEL: fp_to_fp_neg_zero_fold
    // CHECK-NEXT: %[[cst_folded:.+]] = arith.constant dense<-0.000000e+00> : tensor<32x128xf8E4M3FN, #blocked>
    // CHECK-NEXT: tt.return %[[cst_folded]]
    %cst = arith.constant dense<-0.00e+00> : tensor<32x128xf32, #blocked>
    %cst_converted = tt.fp_to_fp %cst, rounding = rtne : tensor<32x128xf32, #blocked> -> tensor<32x128xf8E4M3FN, #blocked>
    tt.return %cst_converted : tensor<32x128xf8E4M3FN, #blocked>
  }
}  // end module
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fp_to_fp_neg_zero_fold`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fp_to_fp_neg_zero_fold`。

### Line 111
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 113-124
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  tt.func @fp_to_fp_neg_zero_fold() -> tensor<32x128xf8E4M3FNUZ, #blocked> {
    // CHECK-LABEL: fp_to_fp_neg_zero_fold
    // We fold to the positive zero here given by definition f8E4M3FNUZ does not have negative zero encoding.
    // CHECK-NEXT: %[[cst_folded:.+]] = arith.constant dense<0.000000e+00> : tensor<32x128xf8E4M3FNUZ, #blocked>
    // CHECK-NEXT: tt.return %[[cst_folded]]
    %cst = arith.constant dense<-0.00e+00> : tensor<32x128xf32, #blocked>
    %cst_converted = tt.fp_to_fp %cst, rounding = rtne : tensor<32x128xf32, #blocked> -> tensor<32x128xf8E4M3FNUZ, #blocked>
    tt.return %cst_converted : tensor<32x128xf8E4M3FNUZ, #blocked>
  }
}  // end module
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fp_to_fp_neg_zero_fold`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fp_to_fp_neg_zero_fold`。

### Line 126
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 128-139
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  tt.func @fold_fp_to_fp_non_zero_nofold() -> tensor<32x128xf8E4M3FNUZ, #blocked> {
    // CHECK-LABEL: fold_fp_to_fp_non_zero_nofold
    // CHECK-NEXT: %[[cst:.+]] = arith.constant dense<0xFF800000> : tensor<32x128xf32, #blocked>
    // CHECK-NEXT: %[[cst_cvt:.+]] = tt.fp_to_fp %[[cst]]
    // CHECK-NEXT: tt.return %[[cst_cvt]]
    %cst = arith.constant dense<0xFF800000> : tensor<32x128xf32, #blocked>
    %cst_converted = tt.fp_to_fp %cst, rounding = rtne : tensor<32x128xf32, #blocked> -> tensor<32x128xf8E4M3FNUZ, #blocked>
    tt.return %cst_converted : tensor<32x128xf8E4M3FNUZ, #blocked>
  }
}  // end module
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fold_fp_to_fp_non_zero_nofold`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fold_fp_to_fp_non_zero_nofold`。

### Line 141
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 143-152
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  tt.func @fold_fp_to_fp_non_constant_nofold(%arg0: tensor<32x128xf32, #blocked>) -> tensor<32x128xf8E4M3FNUZ, #blocked> {
    // CHECK-LABEL: fold_fp_to_fp_non_constant_nofold
    // CHECK-NEXT: %[[arg_cvt:.+]] = tt.fp_to_fp %arg0
    // CHECK-NEXT: tt.return %[[arg_cvt]]
    %cst_converted = tt.fp_to_fp %arg0, rounding = rtne : tensor<32x128xf32, #blocked> -> tensor<32x128xf8E4M3FNUZ, #blocked>
    tt.return %cst_converted : tensor<32x128xf8E4M3FNUZ, #blocked>
  }
}  // end module
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fold_fp_to_fp_non_constant_nofold`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fold_fp_to_fp_non_constant_nofold`。

### Line 154
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 156-160
```mlir
// CHECK-LABEL: @fold_broadcast_constant_pattern
tt.func @fold_broadcast_constant_pattern(%cst : f32) -> tensor<8x2xf32> {
    // CHECK: %[[cst:.*]] = arith.constant dense<1.000000e+00> : tensor<8x2xf32>
    %const = arith.constant dense<1.0> : tensor<8x1xf32>
    %bst_out = tt.broadcast %const : tensor<8x1xf32> -> tensor<8x2xf32>
```
**EN:** This block defines `fold_broadcast_constant_pattern` and exercises operations such as `tt.func`, `arith.constant`, `tt.broadcast`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `fold_broadcast_constant_pattern`，并覆盖 如 `tt.func`、`arith.constant`、`tt.broadcast` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 162-164
```mlir
    // CHECK-NEXT: tt.return %[[cst]] : tensor<8x2xf32>
    tt.return %bst_out : tensor<8x2xf32>
}
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.return`. Embedded check comments (CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.return` 这样的操作。 其中嵌入的检查注释（CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 166
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 168-175
```mlir
// CHECK-LABEL: @fold_transpose_constant
tt.func @fold_transpose_constant() -> tensor<128x16xf32> {
    // CHECK: %[[cst:.*]] = arith.constant dense<1.000000e+00> : tensor<128x16xf32>
    %cst = arith.constant dense<1.0> : tensor<16x128xf32>
    %r = tt.trans %cst {order = array<i32: 1, 0>} : tensor<16x128xf32> -> tensor<128x16xf32>
    // CHECK-NEXT: tt.return %[[cst]] : tensor<128x16xf32>
    tt.return %r : tensor<128x16xf32>
}
```
**EN:** This block defines `fold_transpose_constant` and exercises operations such as `tt.func`, `arith.constant`, `tt.trans`, `tt.return`. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `fold_transpose_constant`，并覆盖 如 `tt.func`、`arith.constant`、`tt.trans`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 176
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 178-188
```mlir
// CHECK-LABEL: @canonicalize_int_to_ptr_of_ptr_to_int
// Test: int_to_ptr(ptr_to_int(ptr)) -> ptr (round-trip elimination)
tt.func @canonicalize_int_to_ptr_of_ptr_to_int(%ptr: tensor<64x!tt.ptr<f32>>) -> tensor<64x!tt.ptr<f32>> {
  // CHECK-NOT: tt.ptr_to_int
  // CHECK-NOT: tt.int_to_ptr
  // CHECK-NOT: tt.bitcast
  // CHECK: tt.return %{{.*}} : tensor<64x!tt.ptr<f32>>
  %int = tt.ptr_to_int %ptr : tensor<64x!tt.ptr<f32>> -> tensor<64xi64>
  %result = tt.int_to_ptr %int : tensor<64xi64> -> tensor<64x!tt.ptr<f32>>
  tt.return %result : tensor<64x!tt.ptr<f32>>
}
```
**EN:** This block defines `canonicalize_int_to_ptr_of_ptr_to_int` and exercises operations such as `tt.func`, `tt.ptr`, `tt.ptr_to_int`, `tt.int_to_ptr`, `tt.bitcast`. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NOT×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `canonicalize_int_to_ptr_of_ptr_to_int`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.ptr_to_int`、`tt.int_to_ptr`、`tt.bitcast` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NOT×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 190
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 192-201
```mlir
// CHECK-LABEL: @canonicalize_int_to_ptr_of_ptr_to_int_with_different_ptr_type
tt.func @canonicalize_int_to_ptr_of_ptr_to_int_with_different_ptr_type(%ptr: !tt.ptr<f32>) -> !tt.ptr<f16> {
  // CHECK-NOT: tt.ptr_to_int
  // CHECK-NOT: tt.int_to_ptr
  // CHECK: %[[RESULT:.*]] = tt.bitcast %{{.*}} : !tt.ptr<f32> -> !tt.ptr<f16>
  %int = tt.ptr_to_int %ptr : !tt.ptr<f32> -> i64
  %result = tt.int_to_ptr %int : i64 -> !tt.ptr<f16>
  // CHECK-NEXT: tt.return %[[RESULT]] : !tt.ptr<f16>
  tt.return %result : !tt.ptr<f16>
}
```
**EN:** This block defines `canonicalize_int_to_ptr_of_ptr_to_int_with_different_ptr_type` and exercises operations such as `tt.func`, `tt.ptr`, `tt.ptr_to_int`, `tt.int_to_ptr`, `tt.bitcast`. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1, CHECK-NOT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `canonicalize_int_to_ptr_of_ptr_to_int_with_different_ptr_type`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.ptr_to_int`、`tt.int_to_ptr`、`tt.bitcast` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×1, CHECK-NOT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 203
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 205-217
```mlir
// CHECK-LABEL: @canonicalize_int_to_ptr_with_constant_offset_f32
// Test: int_to_ptr(addi(ptr_to_int(ptr), constant)) -> addptr(ptr, element_offset)
// For f32 (4 bytes): 16 bytes = 4 elements
tt.func @canonicalize_int_to_ptr_with_constant_offset_f32(%base: tensor<128x!tt.ptr<f32>>) -> tensor<128x!tt.ptr<f32>> {
  // CHECK: %[[OFFSET:.*]] = arith.constant dense<4> : tensor<128xi64>
  // CHECK-NEXT: %[[RESULT:.*]] = tt.addptr %{{.*}}, %[[OFFSET]] : tensor<128x!tt.ptr<f32>>, tensor<128xi64>
  %byte_offset = arith.constant dense<16> : tensor<128xi64>
  %ptr_as_int = tt.ptr_to_int %base : tensor<128x!tt.ptr<f32>> -> tensor<128xi64>
  %offset_ptr_int = arith.addi %ptr_as_int, %byte_offset : tensor<128xi64>
  %result = tt.int_to_ptr %offset_ptr_int : tensor<128xi64> -> tensor<128x!tt.ptr<f32>>
  // CHECK-NEXT: tt.return %[[RESULT]] : tensor<128x!tt.ptr<f32>>
  tt.return %result : tensor<128x!tt.ptr<f32>>
}
```
**EN:** This block defines `canonicalize_int_to_ptr_with_constant_offset_f32` and exercises pointer arithmetic. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `canonicalize_int_to_ptr_with_constant_offset_f32`，并覆盖 指针运算。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 219
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 221-232
```mlir
// CHECK-LABEL: @canonicalize_int_to_ptr_with_constant_offset_f16
// Test: For f16 (2 bytes): 32 bytes = 16 elements
tt.func @canonicalize_int_to_ptr_with_constant_offset_f16(%base: tensor<1024x!tt.ptr<f16>>) -> tensor<1024x!tt.ptr<f16>> {
  // CHECK: %[[OFFSET:.*]] = arith.constant dense<16> : tensor<1024xi64>
  // CHECK-NEXT: %[[RESULT:.*]] = tt.addptr %{{.*}}, %[[OFFSET]] : tensor<1024x!tt.ptr<f16>>, tensor<1024xi64>
  %byte_offset = arith.constant dense<32> : tensor<1024xi64>
  %ptr_as_int = tt.ptr_to_int %base : tensor<1024x!tt.ptr<f16>> -> tensor<1024xi64>
  %offset_ptr_int = arith.addi %ptr_as_int, %byte_offset : tensor<1024xi64>
  %result = tt.int_to_ptr %offset_ptr_int : tensor<1024xi64> -> tensor<1024x!tt.ptr<f16>>
  // CHECK-NEXT: tt.return %[[RESULT]] : tensor<1024x!tt.ptr<f16>>
  tt.return %result : tensor<1024x!tt.ptr<f16>>
}
```
**EN:** This block defines `canonicalize_int_to_ptr_with_constant_offset_f16` and exercises pointer arithmetic. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `canonicalize_int_to_ptr_with_constant_offset_f16`，并覆盖 指针运算。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 234
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 236-246
```mlir
// CHECK-LABEL: @no_canonicalize_non_constant_offset
// Test: Non-constant offsets should not be canonicalized
tt.func @no_canonicalize_non_constant_offset(%base: tensor<128x!tt.ptr<f32>>, %offset: tensor<128xi64>) -> tensor<128x!tt.ptr<f32>> {
  // CHECK: tt.ptr_to_int
  // CHECK-NEXT: arith.addi
  // CHECK-NEXT: tt.int_to_ptr
  %ptr_as_int = tt.ptr_to_int %base : tensor<128x!tt.ptr<f32>> -> tensor<128xi64>
  %offset_ptr_int = arith.addi %ptr_as_int, %offset : tensor<128xi64>
  %result = tt.int_to_ptr %offset_ptr_int : tensor<128xi64> -> tensor<128x!tt.ptr<f32>>
  tt.return %result : tensor<128x!tt.ptr<f32>>
}
```
**EN:** This block defines `no_canonicalize_non_constant_offset` and exercises operations such as `tt.func`, `tt.ptr`, `tt.ptr_to_int`, `arith.addi`, `tt.int_to_ptr`. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `no_canonicalize_non_constant_offset`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.ptr_to_int`、`arith.addi`、`tt.int_to_ptr` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 248
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 250-262
```mlir
// CHECK-LABEL: @no_canonicalize_indivisible_offset
// Test: Offset not divisible by element size should not be canonicalized
tt.func @no_canonicalize_indivisible_offset(%base: tensor<128x!tt.ptr<f32>>) -> tensor<128x!tt.ptr<f32>> {
  // 7 bytes is not divisible by 4 (size of f32)
  // CHECK: tt.ptr_to_int
  // CHECK-NEXT: arith.addi
  // CHECK-NEXT: tt.int_to_ptr
  %byte_offset = arith.constant dense<7> : tensor<128xi64>
  %ptr_as_int = tt.ptr_to_int %base : tensor<128x!tt.ptr<f32>> -> tensor<128xi64>
  %offset_ptr_int = arith.addi %ptr_as_int, %byte_offset : tensor<128xi64>
  %result = tt.int_to_ptr %offset_ptr_int : tensor<128xi64> -> tensor<128x!tt.ptr<f32>>
  tt.return %result : tensor<128x!tt.ptr<f32>>
}
```
**EN:** This block defines `no_canonicalize_indivisible_offset` and exercises operations such as `tt.func`, `tt.ptr`, `tt.ptr_to_int`, `arith.addi`, `tt.int_to_ptr`. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `no_canonicalize_indivisible_offset`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.ptr_to_int`、`arith.addi`、`tt.int_to_ptr` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's Triton dialect canonicalization coverage and focuses on canonicalization patterns.  
  **CN:** 该文件属于 Triton 的 Triton 方言规范化 测试覆盖，关注点是 规范化模式。
- **EN:** The `RUN` pipeline drives tools/passes such as `-canonicalize`.  
  **CN:** `RUN` 流水线会驱动 `-canonicalize` 等工具/pass。
- **EN:** The IR mixes `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `ttg` (TritonGPU ops/layouts).  
  **CN:** 该 IR 混合使用了 `tt`（Triton 核心操作）、`arith`（标量/张量算术）、`ttg`（TritonGPU 操作/布局）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `tt`, `arith`, `ttg`.  
  **CN:** IR 方言依赖：`tt`、`arith`、`ttg`。
