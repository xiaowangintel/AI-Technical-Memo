# auto_encoding.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Gluon/auto_encoding.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises encoding inference in Triton's Gluon pipeline coverage. **CN:** 该MLIR 测试用于覆盖 Triton Gluon 流水线中的编码推断相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --gluon-resolve-auto-encodings | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×36, CHECK-DAG×9, CHECK-LABEL×2, CHECK-NOT×2; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×36, CHECK-DAG×9, CHECK-LABEL×2, CHECK-NOT×2；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that encoding inference produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 编码推断 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --gluon-resolve-auto-encodings | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --gluon-resolve-auto-encodings | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --gluon-resolve-auto-encodings | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Line 3
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 5-18
```mlir
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @infer_simple() -> tensor<8x16xi32, #blocked> {
    // CHECK-DAG: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [4, 4], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
    // CHECK: [[CST:%.*]] = arith.constant dense<7> : tensor<16xi32, #ttg.slice<{dim = 0, parent = [[BLOCKED]]}>>
    // CHECK: [[SLICE:%.*]] = tt.expand_dims [[CST]] {axis = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = [[BLOCKED]]}>> -> tensor<1x16xi32, [[BLOCKED]]>
    // CHECK: [[BROADCAST:%.*]] = tt.broadcast [[SLICE]] : tensor<1x16xi32, [[BLOCKED]]> -> tensor<8x16xi32, [[BLOCKED]]>
    // CHECK: tt.return [[BROADCAST]] : tensor<8x16xi32, [[BLOCKED]]>
    %x_1d = arith.constant dense<7> : tensor<16xi32, #gluon.auto_encoding>
    %x_slice = tt.expand_dims %x_1d {axis = 0 : i32} : tensor<16xi32, #gluon.auto_encoding> -> tensor<1x16xi32, #gluon.auto_encoding>
    %x_2d = tt.broadcast %x_slice : tensor<1x16xi32, #gluon.auto_encoding> -> tensor<8x16xi32, #gluon.auto_encoding>
    %cvt = gluon.set_auto_layout %x_2d : tensor<8x16xi32, #gluon.auto_encoding> -> tensor<8x16xi32, #blocked>
    tt.return %cvt : tensor<8x16xi32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `infer_simple`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `infer_simple`。

### Line 20
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 22-23
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#blocked1`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#blocked1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 25-39
```mlir
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @infer_with_convert() -> tensor<16xi32, #blocked1> {
    // CHECK-DAG: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    // CHECK-DAG: [[BLOCKED1:#.*]] = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    // CHECK: [[CST:%.*]] = arith.constant dense<7> : tensor<16xi32, [[BLOCKED]]>
    // CHECK: [[CVT1:%.*]] = ttg.convert_layout [[CST]] : tensor<16xi32, [[BLOCKED]]> -> tensor<16xi32, [[BLOCKED1]]>
    // CHECK: [[ADD:%.*]] = arith.addi [[CVT1]], [[CVT1]] : tensor<16xi32, [[BLOCKED1]]>
    // CHECK: tt.return [[ADD]] : tensor<16xi32, [[BLOCKED1]]>
    %0 = arith.constant dense<7> : tensor<16xi32, #blocked>
    %cvt1 = ttg.convert_layout %0 : tensor<16xi32, #blocked> -> tensor<16xi32, #gluon.auto_encoding>
    %add = arith.addi %cvt1, %cvt1 : tensor<16xi32, #gluon.auto_encoding>
    %cvt2 = gluon.set_auto_layout %add : tensor<16xi32, #gluon.auto_encoding> -> tensor<16xi32, #blocked1>
    tt.return %cvt2 : tensor<16xi32, #blocked1>
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `infer_with_convert`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `infer_with_convert`。

### Line 42
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 44
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 46-67
```mlir
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @infer_if(%arg0 : i1) -> tensor<16xi32, #blocked> {
    // CHECK-DAG: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    // CHECK: [[C1:%.*]] = arith.constant dense<1> : tensor<16xi32, [[BLOCKED]]>
    // CHECK: [[C2:%.*]] = arith.constant dense<2> : tensor<16xi32, [[BLOCKED]]>
    // CHECK: [[IF:%.*]] = scf.if %arg0 -> (tensor<16xi32, [[BLOCKED]]>) {
    // CHECK:   scf.yield [[C1]] : tensor<16xi32, [[BLOCKED]]>
    // CHECK: } else {
    // CHECK:   scf.yield [[C2]] : tensor<16xi32, [[BLOCKED]]>
    // CHECK: }
    // CHECK: tt.return [[IF]] : tensor<16xi32, [[BLOCKED]]>
    %c1 = arith.constant dense<1> : tensor<16xi32, #gluon.auto_encoding>
    %c2 = arith.constant dense<2> : tensor<16xi32, #gluon.auto_encoding>
    %z = scf.if %arg0 -> tensor<16xi32, #gluon.auto_encoding> {
      scf.yield %c1 : tensor<16xi32, #gluon.auto_encoding>
    } else {
      scf.yield %c2 : tensor<16xi32, #gluon.auto_encoding>
    }
    %cvt = gluon.set_auto_layout %z : tensor<16xi32, #gluon.auto_encoding> -> tensor<16xi32, #blocked>
    tt.return %cvt : tensor<16xi32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `infer_if`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `infer_if`。

### Line 69
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 71
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 73-94
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.num-ctas" = 1 : i32} {
  tt.func public @infer_for(%arg0: i32) -> tensor<32xi32, #blocked> {
    // CHECK-DAG: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    // CHECK: [[RANGE:%.*]] = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, [[BLOCKED]]>
    // CHECK: [[IF:%.*]] = scf.for {{%.*}} = %c0_i32 to %arg0 step %c1_i32 iter_args([[ITER_ARG:%.*]] = [[RANGE]]) -> (tensor<32xi32, [[BLOCKED]]>) : i32 {
    // CHECK:   [[CST:%.*]] = arith.constant dense<2> : tensor<32xi32, [[BLOCKED]]>
    // CHECK:   [[MUL:%.*]] = arith.muli [[ITER_ARG]], [[CST]] : tensor<32xi32, [[BLOCKED]]>
    // CHECK:   scf.yield [[MUL]] : tensor<32xi32, [[BLOCKED]]>
    // CHECK: }
    // CHECK: tt.return [[IF]] : tensor<32xi32, [[BLOCKED]]>
    %c0_i32 = arith.constant 0 : i32
    %c1_i32 = arith.constant 1 : i32
    %0 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #gluon.auto_encoding>
    %1 = scf.for %arg1 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg2 = %0) -> (tensor<32xi32, #gluon.auto_encoding>) : i32 {
      %cst = arith.constant dense<2> : tensor<32xi32, #gluon.auto_encoding>
      %2 = arith.muli %arg2, %cst : tensor<32xi32, #gluon.auto_encoding>
      scf.yield %2 : tensor<32xi32, #gluon.auto_encoding>
    }
    %cvt = gluon.set_auto_layout %1 : tensor<32xi32, #gluon.auto_encoding> -> tensor<32xi32, #blocked>
    tt.return %cvt : tensor<32xi32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.num-ctas`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `infer_for`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.num-ctas`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `infer_for`。

### Line 97
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 100
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 102-113
```mlir
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @infer_make_range() -> tensor<16xi32, #blocked> {
    // CHECK-DAG: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
    // CHECK: [[CST:%.*]] = arith.constant 0 : i32
    // CHECK: [[SPLAT: %.*]] = tt.splat [[CST]] : i32 -> tensor<16xi32, [[BLOCKED]]>
    // CHECK: tt.return [[RANGE]] : tensor<16xi32, [[BLOCKED]]>
    %cst = arith.constant 0 : i32
    %0 = tt.splat %cst : i32 -> tensor<16xi32, #gluon.auto_encoding>
    %cvt = gluon.set_auto_layout %0 : tensor<16xi32, #gluon.auto_encoding> -> tensor<16xi32, #blocked>
    tt.return %cvt : tensor<16xi32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `infer_make_range`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `infer_make_range`。

### Line 116
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 118
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 120-133
```mlir
module attributes {ttg.maxnreg = 128 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
  tt.func private @infer_with_downstream_ops() -> tensor<128x128xi32, #blocked> {
    // CHECK-DAG: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
    // CHECK: [[RANGE:%.*]] = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = [[BLOCKED]]}>>
    // CHECK: [[EXPAND:%.*]] = tt.expand_dims [[RANGE]] {axis = 0 : i32} : tensor<128xi32, #ttg.slice<{dim = 0, parent = [[BLOCKED]]}>> -> tensor<1x128xi32, [[BLOCKED]]>
    // CHECK: [[BROADCAST:%.*]] = tt.broadcast [[EXPAND]] : tensor<1x128xi32, [[BLOCKED]]> -> tensor<128x128xi32, [[BLOCKED]]>
    // CHECK: tt.return [[BROADCAST]] : tensor<128x128xi32, [[BLOCKED]]>
    %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #gluon.auto_encoding>
    %1 = tt.expand_dims %0 {axis = 0 : i32} : tensor<128xi32, #gluon.auto_encoding> -> tensor<1x128xi32, #gluon.auto_encoding>
    %2 = gluon.set_auto_layout %1 : tensor<1x128xi32, #gluon.auto_encoding> -> tensor<1x128xi32, #blocked>
    %3 = tt.broadcast %2 : tensor<1x128xi32, #blocked> -> tensor<128x128xi32, #blocked>
    tt.return %3 : tensor<128x128xi32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.maxnreg`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `infer_with_downstream_ops`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.maxnreg`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `infer_with_downstream_ops`。

### Line 135
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 137-150
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [0, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @_tmem_col_slice_load(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) -> tensor<64x128xi32, #blocked> {
    // CHECK-DAG: [[BLOCKED:#.*]] = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [0, 1]}>
    // CHECK-DAG: [[LINEAR:#.*]] = #ttg.linear
    // CHECK: [[RANGE:%.*]] = tt.make_range {end = 8192 : i32, start = 0 : i32} : tensor<8192xi32, [[LINEAR]]>
    // CHECK: [[RESHAPE:%.*]] = tt.reshape [[RANGE]] : tensor<8192xi32, [[LINEAR]]> -> tensor<64x128xi32, [[BLOCKED]]>
    // CHECK: tt.return [[RESHAPE]] : tensor<64x128xi32, [[BLOCKED]]>
    %0 = tt.make_range {end = 8192 : i32, start = 0 : i32} : tensor<8192xi32, #gluon.auto_encoding>
    %1 = tt.reshape %0 : tensor<8192xi32, #gluon.auto_encoding> -> tensor<64x128xi32, #gluon.auto_encoding>
    %2 = gluon.set_auto_layout %1 : tensor<64x128xi32, #gluon.auto_encoding> -> tensor<64x128xi32, #blocked>
    tt.return %2 : tensor<64x128xi32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `_tmem_col_slice_load`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `_tmem_col_slice_load`。

### Line 152
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 154-155
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#blocked1`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#blocked1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 157-180
```mlir
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @infer_if_yield_propagation
  tt.func public @infer_if_yield_propagation(%cond: i1) -> tensor<16xi32, #blocked> {
    // The scf.if has two results. Result #0 is resolved from outside (via
    // set_auto_layout on %z#0). Result #1 can only be resolved via forward
    // propagation from the yield operands inside the if body.
    //
    // CHECK: %[[IF:.*]]:2 = scf.if {{.*}} -> (tensor<16xi32, #{{.*}}>, tensor<16xi32, #{{.*}}>)
    // CHECK-NOT: auto_encoding
    %c1 = arith.constant dense<1> : tensor<16xi32, #gluon.auto_encoding>
    %c2 = arith.constant dense<2> : tensor<16xi32, #gluon.auto_encoding>
    %z:2 = scf.if %cond -> (tensor<16xi32, #gluon.auto_encoding>, tensor<16xi32, #gluon.auto_encoding>) {
      %range = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #gluon.auto_encoding>
      %resolved = gluon.set_auto_layout %range : tensor<16xi32, #gluon.auto_encoding> -> tensor<16xi32, #blocked1>
      scf.yield %c1, %range : tensor<16xi32, #gluon.auto_encoding>, tensor<16xi32, #gluon.auto_encoding>
    } else {
      %cst = arith.constant dense<0> : tensor<16xi32, #gluon.auto_encoding>
      %resolved2 = gluon.set_auto_layout %cst : tensor<16xi32, #gluon.auto_encoding> -> tensor<16xi32, #blocked1>
      scf.yield %c2, %cst : tensor<16xi32, #gluon.auto_encoding>, tensor<16xi32, #gluon.auto_encoding>
    }
    %out = gluon.set_auto_layout %z#0 : tensor<16xi32, #gluon.auto_encoding> -> tensor<16xi32, #blocked>
    tt.return %out : tensor<16xi32, #blocked>
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `infer_if_yield_propagation`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `infer_if_yield_propagation`。

### Line 182
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 184
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 186-201
```mlir
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @infer_reduce_to_scalar
  // CHECK-NOT: auto_encoding
  // CHECK: "tt.reduce"
  // CHECK: tt.return
  tt.func public @infer_reduce_to_scalar() -> i32 {
    %0 = arith.constant dense<1> : tensor<16xi32, #gluon.auto_encoding>
    %1 = gluon.set_auto_layout %0 : tensor<16xi32, #gluon.auto_encoding> -> tensor<16xi32, #blocked>
    %2 = "tt.reduce"(%0) <{axis = 0 : i32}> ({
    ^bb0(%lhs: i32, %rhs: i32):
      %3 = arith.addi %lhs, %rhs : i32
      tt.reduce.return %3 : i32
    }) : (tensor<16xi32, #gluon.auto_encoding>) -> i32
    tt.return %2 : i32
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `infer_reduce_to_scalar`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `infer_reduce_to_scalar`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's Gluon pipeline coverage and focuses on encoding inference.  
  **CN:** 该文件属于 Triton 的 Gluon 流水线 测试覆盖，关注点是 编码推断。
- **EN:** The `RUN` pipeline drives tools/passes such as `--gluon-resolve-auto-encodings`.  
  **CN:** `RUN` 流水线会驱动 `--gluon-resolve-auto-encodings` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `scf` (structured control flow).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`scf`（结构化控制流）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `scf`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`scf`。
