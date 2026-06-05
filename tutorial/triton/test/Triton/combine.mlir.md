# combine.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Triton/combine.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises combining patterns in Triton's Triton dialect canonicalization coverage. **CN:** 该MLIR 测试用于覆盖 Triton Triton 方言规范化中的合并模式相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -canonicalize -triton-combine | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×37, CHECK-DAG×24, CHECK-LABEL×30, CHECK-NEXT×26; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×37, CHECK-DAG×24, CHECK-LABEL×30, CHECK-NEXT×26；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that combining patterns produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 合并模式 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -canonicalize -triton-combine | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -canonicalize -triton-combine | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -canonicalize -triton-combine | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-12
```mlir
// We don't combine if the dot result is used by more than one op.
// CHECK-LABEL: @test_combine_dot_add_invalid_pattern
tt.func @test_combine_dot_add_invalid_pattern() -> (tensor<128x128xf32>, tensor<128x128xf32>) {
    // CHECK-DAG: %[[d:.*]] = arith.constant dense<3.000000e+00> : tensor<128x128xf32>
    // CHECK-DAG: %[[e:.*]] = arith.constant dense<4.000000e+00> : tensor<128x128xf32>
    %a = arith.constant dense<1.0> : tensor<128x128xf32>
    %b = arith.constant dense<2.0> : tensor<128x128xf32>
    %zero = arith.constant dense<0.0> : tensor<128x128xf32>
    %d = arith.constant dense<3.0> : tensor<128x128xf32>
    %e = arith.constant dense<4.0> : tensor<128x128xf32>
```
**EN:** This block defines `test_combine_dot_add_invalid_pattern` and exercises operations such as `tt.func`, `arith.constant`. Embedded check comments (CHECK-DAG×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_dot_add_invalid_pattern`，并覆盖 如 `tt.func`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 14
```mlir
    %dot_out = tt.dot %a, %b, %zero : tensor<128x128xf32> * tensor<128x128xf32> -> tensor<128x128xf32>
```
**EN:** This block contributes intermediate IR built from `tt.dot`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 16-17
```mlir
    // CHECK: arith.addf %{{.*}}, %[[d]] : tensor<128x128xf32>
    %res0 = arith.addf %dot_out, %d : tensor<128x128xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `arith.addf`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `arith.addf` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 19-20
```mlir
    // CHECK-NEXT: arith.addf %{{.*}}, %[[e]]  : tensor<128x128xf32>
    %res1 = arith.addf %dot_out, %e : tensor<128x128xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `arith.addf`. Embedded check comments (CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `arith.addf` 这样的操作。 其中嵌入的检查注释（CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 22-23
```mlir
    tt.return %res0, %res1 : tensor<128x128xf32>, tensor<128x128xf32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 26-34
```mlir
// CHECK-LABEL: @test_combine_dot_add_pattern
tt.func @test_combine_dot_add_pattern() -> (tensor<128x128xf32>) {
    // CHECK-DAG: %[[d:.*]] = arith.constant dense<3.000000e+00> : tensor<128x128xf32>
    // CHECK-DAG: %[[b:.*]] = arith.constant dense<2.000000e+00> : tensor<128x128xf32>
    // CHECK-DAG: %[[a:.*]] = arith.constant dense<1.000000e+00> : tensor<128x128xf32>
    %a = arith.constant dense<1.0> : tensor<128x128xf32>
    %b = arith.constant dense<2.0> : tensor<128x128xf32>
    %zero = arith.constant dense<0.0> : tensor<128x128xf32>
    %d = arith.constant dense<3.0> : tensor<128x128xf32>
```
**EN:** This block defines `test_combine_dot_add_pattern` and exercises operations such as `tt.func`, `arith.constant`. Embedded check comments (CHECK-DAG×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_dot_add_pattern`，并覆盖 如 `tt.func`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 36
```mlir
    %dot_out = tt.dot %a, %b, %zero : tensor<128x128xf32> * tensor<128x128xf32> -> tensor<128x128xf32>
```
**EN:** This block contributes intermediate IR built from `tt.dot`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 38-40
```mlir
    // CHECK-NEXT: %[[res:.*]] = tt.dot %[[a]], %[[b]], %[[d]] : tensor<128x128xf32> * tensor<128x128xf32> -> tensor<128x128xf32>
    // CHECK-NEXT: tt.return %[[res]] : tensor<128x128xf32>
    %res = arith.addf %dot_out, %d : tensor<128x128xf32>
```
**EN:** This block defines the test function(s) and exercises dot products / matmul. Embedded check comments (CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 点积/矩阵乘。 其中嵌入的检查注释（CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 42-43
```mlir
    tt.return %res : tensor<128x128xf32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 46-58
```mlir
// CHECK-LABEL: @test_combine_scale_dot_add_pattern
tt.func @test_combine_scale_dot_add_pattern() -> (tensor<128x128xf32>) {
    // CHECK-DAG: %[[a:.*]] = arith.constant dense<1.000000e+00> : tensor<128x128xf8E5M2>
    // CHECK-DAG: %[[sa:.*]] = arith.constant dense<1> : tensor<128x4xi8>
    // CHECK-DAG: %[[b:.*]] = arith.constant dense<2.000000e+00> : tensor<128x128xf8E5M2>
    // CHECK-DAG: %[[sb:.*]] = arith.constant dense<2> : tensor<128x4xi8>
    // CHECK-DAG: %[[d:.*]] = arith.constant dense<3.000000e+00> : tensor<128x128xf32>
    %a = arith.constant dense<1.0> : tensor<128x128xf8E5M2>
    %sa = arith.constant dense<1> : tensor<128x4xi8>
    %b = arith.constant dense<2.0> : tensor<128x128xf8E5M2>
    %sb = arith.constant dense<2> : tensor<128x4xi8>
    %zero = arith.constant dense<0.0> : tensor<128x128xf32>
    %d = arith.constant dense<3.0> : tensor<128x128xf32>
```
**EN:** This block defines `test_combine_scale_dot_add_pattern` and exercises operations such as `tt.func`, `arith.constant`. Embedded check comments (CHECK-DAG×5, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_scale_dot_add_pattern`，并覆盖 如 `tt.func`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×5, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 60-61
```mlir
    %dot_out = tt.dot_scaled %a scale %sa, %b scale %sb, %zero lhs = e5m2 rhs = e5m2 {fastMath = false}
      : tensor<128x128xf8E5M2>, tensor<128x4xi8> * tensor<128x128xf8E5M2>, tensor<128x4xi8> -> tensor<128x128xf32>
```
**EN:** This block contributes intermediate IR built from `tt.dot_scaled`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot_scaled` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 63-67
```mlir
    // CHECK-NEXT: %[[res:.*]] = tt.dot_scaled %[[a]] scale %[[sa]], %[[b]] scale %[[sb]], %[[d]] lhs = e5m2 rhs = e5m2 {fastMath = false} : tensor<128x128xf8E5M2>, tensor<128x4xi8> * tensor<128x128xf8E5M2>, tensor<128x4xi8> -> tensor<128x128xf32>
    // CHECK-NEXT: tt.return %[[res]] : tensor<128x128xf32>
    %res = arith.addf %dot_out, %d : tensor<128x128xf32>
    tt.return %res : tensor<128x128xf32>
}
```
**EN:** This block defines the test function(s) and exercises dot products / matmul. Embedded check comments (CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 点积/矩阵乘。 其中嵌入的检查注释（CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 70-78
```mlir
// CHECK-LABEL: @test_combine_dot_add_rev_pattern
tt.func @test_combine_dot_add_rev_pattern() -> (tensor<128x128xf32>) {
    // CHECK-DAG: %[[d:.*]] = arith.constant dense<3.000000e+00> : tensor<128x128xf32>
    // CHECK-DAG: %[[b:.*]] = arith.constant dense<2.000000e+00> : tensor<128x128xf32>
    // CHECK-DAG: %[[a:.*]] = arith.constant dense<1.000000e+00> : tensor<128x128xf32>
    %a = arith.constant dense<1.0> : tensor<128x128xf32>
    %b = arith.constant dense<2.0> : tensor<128x128xf32>
    %zero = arith.constant dense<0.0> : tensor<128x128xf32>
    %d = arith.constant dense<3.0> : tensor<128x128xf32>
```
**EN:** This block defines `test_combine_dot_add_rev_pattern` and exercises operations such as `tt.func`, `arith.constant`. Embedded check comments (CHECK-DAG×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_dot_add_rev_pattern`，并覆盖 如 `tt.func`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 80
```mlir
    %dot_out = tt.dot %a, %b, %zero : tensor<128x128xf32> * tensor<128x128xf32> -> tensor<128x128xf32>
```
**EN:** This block contributes intermediate IR built from `tt.dot`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 82-84
```mlir
    // CHECK-NEXT: %[[res:.*]] = tt.dot %[[a]], %[[b]], %[[d]] : tensor<128x128xf32> * tensor<128x128xf32> -> tensor<128x128xf32>
    // CHECK-NEXT: tt.return %[[res]] : tensor<128x128xf32>
    %res = arith.addf %d, %dot_out : tensor<128x128xf32>
```
**EN:** This block defines the test function(s) and exercises dot products / matmul. Embedded check comments (CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 点积/矩阵乘。 其中嵌入的检查注释（CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 86-87
```mlir
    tt.return %res : tensor<128x128xf32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 90-93
```mlir
// CHECK-LABEL: @test_combine_addptr_pattern
tt.func @test_combine_addptr_pattern(%base: !tt.ptr<f32>) -> tensor<8x!tt.ptr<f32>> {
    %off0 = arith.constant 10 : i32
    %off1 = arith.constant 15 : i32
```
**EN:** This block defines `test_combine_addptr_pattern` and exercises operations such as `tt.func`, `tt.ptr`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_addptr_pattern`，并覆盖 如 `tt.func`、`tt.ptr`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 95
```mlir
    // CHECK-NEXT: %[[cst:.*]] = arith.constant dense<25> : tensor<8xi32>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Line 97
```mlir
    %base_ = tt.splat %base : !tt.ptr<f32> -> tensor<8x!tt.ptr<f32>>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 99
```mlir
    // CHECK-NEXT: %[[tmp0:.*]] = tt.splat %{{.*}} : !tt.ptr<f32> -> tensor<8x!tt.ptr<f32>>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 101-102
```mlir
    %idx0 = tt.splat %off0 : i32 -> tensor<8xi32>
    %idx1 = tt.splat %off1 : i32 -> tensor<8xi32>
```
**EN:** This block contributes intermediate IR built from `tt.splat`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 104-106
```mlir
    // CHECK-NEXT: %1 = tt.addptr %[[tmp0]], %[[cst]] : tensor<8x!tt.ptr<f32>>, tensor<8xi32>
    %ptr0 = tt.addptr %base_, %idx0 : tensor<8x!tt.ptr<f32>>, tensor<8xi32>
    %ptr1 = tt.addptr %ptr0, %idx1 : tensor<8x!tt.ptr<f32>>, tensor<8xi32>
```
**EN:** This block defines the test function(s) and exercises pointer arithmetic. Embedded check comments (CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 指针运算。 其中嵌入的检查注释（CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 108-109
```mlir
    tt.return %ptr1 : tensor<8x!tt.ptr<f32>>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 111-118
```mlir
// CHECK-LABEL: @test_combine_addptr_pattern_discardableattrs
tt.func @test_combine_addptr_pattern_discardableattrs(%base: !tt.ptr<f32>) -> !tt.ptr<f32> {
    %off0 = arith.constant 8 : i32
    %off1 = arith.constant 4 : i32
    // CHECK-NEXT: %[[cst:.*]] = arith.constant 12 : i32
    // CHECK-NEXT: %0 = tt.addptr %{{.*}}, %[[cst]] {tt.constancy = 8 : i32, tt.contiguity = 512 : i32, tt.divisibility = 16 : i32} : !tt.ptr<f32>, i32
    %ptr0 = tt.addptr %base, %off0 : !tt.ptr<f32>, i32
    %ptr1 = tt.addptr %ptr0, %off1 {tt.divisibility = 16 : i32, tt.constancy = 8 : i32, tt.contiguity = 512 : i32} : !tt.ptr<f32>, i32
```
**EN:** This block defines `test_combine_addptr_pattern_discardableattrs` and exercises pointer arithmetic. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_addptr_pattern_discardableattrs`，并覆盖 指针运算。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 120-121
```mlir
    tt.return %ptr1 : !tt.ptr<f32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 123-130
```mlir
// CHECK-LABEL: @test_combine_addptr_pattern_discardableattrs_disallowed
tt.func @test_combine_addptr_pattern_discardableattrs_disallowed(%base: !tt.ptr<f32>) -> !tt.ptr<f32> {
    %off0 = arith.constant 8 : i32
    %off1 = arith.constant 4 : i32
    // CHECK-NEXT: %[[cst:.*]] = arith.constant 12 : i32
    // CHECK-NEXT: %0 = tt.addptr %{{.*}}, %[[cst]] {tt.divisibility = 16 : i32} : !tt.ptr<f32>, i32
    %ptr0 = tt.addptr %base, %off0 : !tt.ptr<f32>, i32
    %ptr1 = tt.addptr %ptr0, %off1 {tt.divisibility = 16 : i32, tt.disallowed = 8 : i32} : !tt.ptr<f32>, i32
```
**EN:** This block defines `test_combine_addptr_pattern_discardableattrs_disallowed` and exercises pointer arithmetic. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_addptr_pattern_discardableattrs_disallowed`，并覆盖 指针运算。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 132-137
```mlir
    tt.return %ptr1 : !tt.ptr<f32>
}
// CHECK-LABEL: @test_combine_addptr_pattern_i64
tt.func @test_combine_addptr_pattern_i64(%base: !tt.ptr<f32>) -> tensor<8x!tt.ptr<f32>> {
    %off0 = arith.constant 10 : i64
    %off1 = arith.constant dense<15> : tensor<8xi64>
```
**EN:** This block defines `test_combine_addptr_pattern_i64` and exercises operations such as `tt.return`, `tt.ptr`, `tt.func`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_addptr_pattern_i64`，并覆盖 如 `tt.return`、`tt.ptr`、`tt.func`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 139
```mlir
    // CHECK-NEXT: %[[cst:.*]] = arith.constant dense<25> : tensor<8xi64>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Line 141
```mlir
    %base_ = tt.splat %base : !tt.ptr<f32> -> tensor<8x!tt.ptr<f32>>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 143
```mlir
    // CHECK-NEXT: %[[tmp0:.*]] = tt.splat %{{.*}} : !tt.ptr<f32> -> tensor<8x!tt.ptr<f32>>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Line 145
```mlir
    %idx0 = tt.splat %off0 : i64 -> tensor<8xi64>
```
**EN:** This block contributes intermediate IR built from `tt.splat`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 147-149
```mlir
    // CHECK-NEXT: %1 = tt.addptr %[[tmp0]], %[[cst]] : tensor<8x!tt.ptr<f32>>, tensor<8xi64>
    %ptr0 = tt.addptr %base_, %idx0 : tensor<8x!tt.ptr<f32>>, tensor<8xi64>
    %ptr1 = tt.addptr %ptr0, %off1 : tensor<8x!tt.ptr<f32>>, tensor<8xi64>
```
**EN:** This block defines the test function(s) and exercises pointer arithmetic. Embedded check comments (CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 指针运算。 其中嵌入的检查注释（CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 151-152
```mlir
    tt.return %ptr1 : tensor<8x!tt.ptr<f32>>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 154-157
```mlir
// CHECK-LABEL: @test_combine_addptr_pattern_scalar
tt.func @test_combine_addptr_pattern_scalar(%base: !tt.ptr<f32>) -> !tt.ptr<f32> {
    %off0 = arith.constant 10 : i32
    %off1 = arith.constant 15 : i32
```
**EN:** This block defines `test_combine_addptr_pattern_scalar` and exercises operations such as `tt.func`, `tt.ptr`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_addptr_pattern_scalar`，并覆盖 如 `tt.func`、`tt.ptr`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 159-162
```mlir
    // CHECK-NEXT: %[[cst:.*]] = arith.constant 25 : i32
    // CHECK-NEXT: %0 = tt.addptr %{{.*}}, %[[cst]] : !tt.ptr<f32>, i32
    %ptr0 = tt.addptr %base, %off0 : !tt.ptr<f32>, i32
    %ptr1 = tt.addptr %ptr0, %off1 : !tt.ptr<f32>, i32
```
**EN:** This block defines the test function(s) and exercises pointer arithmetic. Embedded check comments (CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 指针运算。 其中嵌入的检查注释（CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 164-165
```mlir
    tt.return %ptr1 : !tt.ptr<f32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 167-169
```mlir
// CHECK-LABEL: @test_not_combine_addptr_pattern_1
tt.func @test_not_combine_addptr_pattern_1(%base: !tt.ptr<f32>, %idx0: tensor<8xi32>) -> tensor<8x!tt.ptr<f32>> {
    %off1 = arith.constant 15 : i32
```
**EN:** This block defines `test_not_combine_addptr_pattern_1` and exercises operations such as `tt.func`, `tt.ptr`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_not_combine_addptr_pattern_1`，并覆盖 如 `tt.func`、`tt.ptr`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 171-172
```mlir
    %base_ = tt.splat %base : !tt.ptr<f32> -> tensor<8x!tt.ptr<f32>>
    %idx1 = tt.splat %off1 : i32 -> tensor<8xi32>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 174-179
```mlir
    // CHECK: tt.addptr
    // CHECK-NEXT: tt.addptr
    %ptr0 = tt.addptr %base_, %idx0 : tensor<8x!tt.ptr<f32>>, tensor<8xi32>
    %ptr1 = tt.addptr %ptr0, %idx1 : tensor<8x!tt.ptr<f32>>, tensor<8xi32>
    tt.return %ptr1 : tensor<8x!tt.ptr<f32>>
}
```
**EN:** This block defines the test function(s) and exercises pointer arithmetic. Embedded check comments (CHECK×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 指针运算。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 181-184
```mlir
// CHECK-LABEL: @test_not_combine_addptr_pattern
tt.func @test_not_combine_addptr_pattern(%base: !tt.ptr<f32>) -> tensor<8x!tt.ptr<f32>> {
    %off0 = arith.constant 10 : i16
    %off1 = arith.constant 15 : i32
```
**EN:** This block defines `test_not_combine_addptr_pattern` and exercises operations such as `tt.func`, `tt.ptr`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_not_combine_addptr_pattern`，并覆盖 如 `tt.func`、`tt.ptr`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 186-187
```mlir
    // CHECK-DAG: %[[cst:.*]] = arith.constant dense<10> : tensor<8xi16>
    // CHECK-DAG: %[[cst1:.*]] = arith.constant dense<15> : tensor<8xi32>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-DAG`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-DAG`），从而保证后端/代码生成结果稳定。

### Line 189
```mlir
    %base_ = tt.splat %base : !tt.ptr<f32> -> tensor<8x!tt.ptr<f32>>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 191-192
```mlir
    %idx0 = tt.splat %off0 : i16 -> tensor<8xi16>
    %idx1 = tt.splat %off1 : i32 -> tensor<8xi32>
```
**EN:** This block contributes intermediate IR built from `tt.splat`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 194-195
```mlir
    %ptr0 = tt.addptr %base_, %idx0 : tensor<8x!tt.ptr<f32>>, tensor<8xi16>
    %ptr1 = tt.addptr %ptr0, %idx1 : tensor<8x!tt.ptr<f32>>, tensor<8xi32>
```
**EN:** This block contributes intermediate IR built from `tt.addptr`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.addptr`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 197-198
```mlir
    tt.return %ptr1 : tensor<8x!tt.ptr<f32>>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 200-203
```mlir
// CHECK-LABEL: @test_not_combine_addptr_pattern_overflow
tt.func @test_not_combine_addptr_pattern_overflow(%base: !tt.ptr<f32>) -> tensor<8x!tt.ptr<f32>> {
    %off0 = arith.constant 127 : i8
    %off1 = arith.constant 1 : i8
```
**EN:** This block defines `test_not_combine_addptr_pattern_overflow` and exercises operations such as `tt.func`, `tt.ptr`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_not_combine_addptr_pattern_overflow`，并覆盖 如 `tt.func`、`tt.ptr`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 205-206
```mlir
    // CHECK-DAG: %[[cst:.*]] = arith.constant dense<127> : tensor<8xi8>
    // CHECK-DAG: %[[cst1:.*]] = arith.constant dense<1> : tensor<8xi8>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-DAG`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-DAG`），从而保证后端/代码生成结果稳定。

### Line 208
```mlir
    %base_ = tt.splat %base : !tt.ptr<f32> -> tensor<8x!tt.ptr<f32>>
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 210-211
```mlir
    %idx0 = tt.splat %off0 : i8 -> tensor<8xi8>
    %idx1 = tt.splat %off1 : i8 -> tensor<8xi8>
```
**EN:** This block contributes intermediate IR built from `tt.splat`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 213-214
```mlir
    %ptr0 = tt.addptr %base_, %idx0 : tensor<8x!tt.ptr<f32>>, tensor<8xi8>
    %ptr1 = tt.addptr %ptr0, %idx1 : tensor<8x!tt.ptr<f32>>, tensor<8xi8>
```
**EN:** This block contributes intermediate IR built from `tt.addptr`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.addptr`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 216-217
```mlir
    tt.return %ptr1 : tensor<8x!tt.ptr<f32>>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 219-222
```mlir
// CHECK-LABEL: @test_combine_select_masked_load_pattern
tt.func @test_combine_select_masked_load_pattern(%ptr: tensor<8x!tt.ptr<f32>>, %cond: i1) -> (tensor<8xf32>, tensor<8xf32>) {
    %mask = tt.splat %cond : i1 -> tensor<8xi1>
    %false_val = arith.constant dense<0.0> : tensor<8xf32>
```
**EN:** This block defines `test_combine_select_masked_load_pattern` and exercises operations such as `tt.func`, `tt.ptr`, `tt.splat`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_select_masked_load_pattern`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.splat`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 224-226
```mlir
    // CHECK: %[[res1:.*]] = tt.load %{{.*}}, %{{.*}}, %{{.*}} : tensor<8x!tt.ptr<f32>>
    %x = tt.load %ptr, %mask, %false_val : tensor<8x!tt.ptr<f32>>
    %0 = arith.select %cond, %x, %false_val : tensor<8xf32>
```
**EN:** This block defines the test function(s) and exercises tensor loads, predicated selection. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量加载、条件选择。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 228-230
```mlir
    // CHECK: %[[res2:.*]] = tt.load %{{.*}}, %{{.*}}, %{{.*}} : tensor<8x!tt.ptr<f32>>
    %y = tt.load %ptr, %mask, %false_val : tensor<8x!tt.ptr<f32>>
    %1 = arith.select %cond, %y, %false_val : tensor<8xf32>
```
**EN:** This block defines the test function(s) and exercises tensor loads, predicated selection. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量加载、条件选择。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 232-234
```mlir
    // CHECK: tt.return %[[res1]], %[[res2]] : tensor<8xf32>, tensor<8xf32>
    tt.return %0, %1 : tensor<8xf32>, tensor<8xf32>
}
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.return`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 236-238
```mlir
// CHECK-LABEL: @test_combine_select_masked_load_fail_pattern
tt.func @test_combine_select_masked_load_fail_pattern(%ptr: tensor<8x!tt.ptr<f32>>, %dummy_load: tensor<8xf32>, %dummy_broadcast: tensor<8xi1>, %cond0: i1, %cond1: i1) -> (tensor<8xf32>, tensor<8xf32>, tensor<8xf32>) {
    %false_val = arith.constant dense<0.0> : tensor<8xf32>
```
**EN:** This block defines `test_combine_select_masked_load_fail_pattern` and exercises operations such as `tt.func`, `tt.ptr`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_select_masked_load_fail_pattern`，并覆盖 如 `tt.func`、`tt.ptr`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 240-242
```mlir
    // Case 1: value at the "load" position is not an "op".  Select should not be canonicalized.
    // CHECK: %{{.*}} = arith.select %{{.*}}, %{{.*}}, %{{.*}} : tensor<8xf32>
    %0 = arith.select %cond0, %dummy_load, %false_val : tensor<8xf32>
```
**EN:** This block defines the test function(s) and exercises predicated selection. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 条件选择。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 244-247
```mlir
    // Case 2: value at the "broadcast" position is not an "op".  Select should not be canonicalized.
    %real_load0 = tt.load %ptr, %dummy_broadcast, %false_val : tensor<8x!tt.ptr<f32>>
    // CHECK: %{{.*}} = arith.select %{{.*}}, %{{.*}}, %{{.*}} : tensor<8xf32>
    %1 = arith.select %cond0, %real_load0, %false_val : tensor<8xf32>
```
**EN:** This block defines the test function(s) and exercises tensor loads, predicated selection. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量加载、条件选择。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 249-253
```mlir
    // Case 3: condition of "broadcast" is not the same as the condition of "select".  Select should not be canonicalized.
    %cond0_ = tt.splat %cond0 : i1 -> tensor<8xi1>
    %real_load1 = tt.load %ptr, %cond0_, %false_val : tensor<8x!tt.ptr<f32>>
    // CHECK: %{{.*}} = arith.select %{{.*}}, %{{.*}}, %{{.*}} : tensor<8xf32>
    %2 = arith.select %cond1, %real_load1, %false_val : tensor<8xf32>
```
**EN:** This block defines the test function(s) and exercises tensor loads, predicated selection. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量加载、条件选择。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 255-256
```mlir
    tt.return %0, %1, %2 : tensor<8xf32>, tensor<8xf32>, tensor<8xf32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 258-262
```mlir
// CHECK-LABEL: @test_canonicalize_masked_load_pattern
tt.func @test_canonicalize_masked_load_pattern(%ptr: tensor<8x!tt.ptr<f32>>) -> (tensor<8xf32>, tensor<8xf32>, tensor<8xf32>) {
    %true_mask = arith.constant dense<true> : tensor<8xi1>
    %false_mask = arith.constant dense<false> : tensor<8xi1>
    %other_val = arith.constant dense<0.0> : tensor<8xf32>
```
**EN:** This block defines `test_canonicalize_masked_load_pattern` and exercises operations such as `tt.func`, `tt.ptr`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_canonicalize_masked_load_pattern`，并覆盖 如 `tt.func`、`tt.ptr`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 264-266
```mlir
    // true_mask with other
    // CHECK: %[[res1:.*]] = tt.load %{{.*}} : tensor<8x!tt.ptr<f32>>
    %x = tt.load %ptr, %true_mask : tensor<8x!tt.ptr<f32>>
```
**EN:** This block defines the test function(s) and exercises tensor loads. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量加载。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 268-270
```mlir
    // true_mask without other
    // CHECK: %[[res2:.*]] = tt.load %{{.*}} : tensor<8x!tt.ptr<f32>>
    %y = tt.load %ptr, %true_mask, %other_val : tensor<8x!tt.ptr<f32>>
```
**EN:** This block defines the test function(s) and exercises tensor loads. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量加载。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 272-273
```mlir
    // false_mask with other. It should become "other" (i.e., %y)
    %z = tt.load %ptr, %false_mask, %y : tensor<8x!tt.ptr<f32>>
```
**EN:** This block contributes intermediate IR built from `i.e`, `tt.load`, `tt.ptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `i.e`, `tt.load`, `tt.ptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 275-277
```mlir
    // CHECK: tt.return %[[res1]], %[[res2]], %[[res2]] : tensor<8xf32>, tensor<8xf32>, tensor<8xf32>
    tt.return %x, %y, %z: tensor<8xf32>, tensor<8xf32>, tensor<8xf32>
}
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.return`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 279-281
```mlir
// CHECK-LABEL: @test_canonicalize_masked_load_fail_pattern
tt.func @test_canonicalize_masked_load_fail_pattern(%ptr: tensor<8x!tt.ptr<f32>>, %mask: tensor<8xi1>) -> (tensor<8xf32>, tensor<8xf32>) {
    %other_val = arith.constant dense<0.0> : tensor<8xf32>
```
**EN:** This block defines `test_canonicalize_masked_load_fail_pattern` and exercises operations such as `tt.func`, `tt.ptr`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_canonicalize_masked_load_fail_pattern`，并覆盖 如 `tt.func`、`tt.ptr`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 283-287
```mlir
    // Case: value at the "mask" position is not an "op".  Load should not be canonicalized.
    // CHECK: %[[res1:.*]] = tt.load %{{.*}}, %{{.*}} : tensor<8x!tt.ptr<f32>>
    %x = tt.load %ptr, %mask : tensor<8x!tt.ptr<f32>>
    // CHECK: %[[res1:.*]] = tt.load %{{.*}}, %{{.*}}, %{{.*}} : tensor<8x!tt.ptr<f32>>
    %y = tt.load %ptr, %mask, %other_val : tensor<8x!tt.ptr<f32>>
```
**EN:** This block defines the test function(s) and exercises tensor loads. Embedded check comments (CHECK×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量加载。 其中嵌入的检查注释（CHECK×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 289-290
```mlir
    tt.return %x, %y: tensor<8xf32>, tensor<8xf32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 292-295
```mlir
// CHECK-LABEL: @test_canonicalize_masked_store_pattern
tt.func @test_canonicalize_masked_store_pattern(%ptr: tensor<8x!tt.ptr<f32>>, %val: tensor<8xf32>) {
    %true_mask = arith.constant dense<true> : tensor<8xi1>
    %false_mask = arith.constant dense<false> : tensor<8xi1>
```
**EN:** This block defines `test_canonicalize_masked_store_pattern` and exercises operations such as `tt.func`, `tt.ptr`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_canonicalize_masked_store_pattern`，并覆盖 如 `tt.func`、`tt.ptr`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 297-298
```mlir
    // CHECK: tt.store %{{.*}}, %{{.*}} : tensor<8x!tt.ptr<f32>>
    tt.store %ptr, %val, %true_mask : tensor<8x!tt.ptr<f32>>
```
**EN:** This block defines the test function(s) and exercises tensor stores. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 300-304
```mlir
    // The following store should disappear.
    // CHECK-NEXT: tt.return
    tt.store %ptr, %val, %false_mask : tensor<8x!tt.ptr<f32>>
    tt.return
}
```
**EN:** This block defines the test function(s) and exercises tensor stores. Embedded check comments (CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量存储。 其中嵌入的检查注释（CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 306-312
```mlir
// CHECK-LABEL: @test_canonicalize_masked_store_fail_pattern
tt.func @test_canonicalize_masked_store_fail_pattern(%ptr: tensor<8x!tt.ptr<f32>>, %val: tensor<8xf32>, %mask: tensor<8xi1>) {
    // Case: value at the "mask" position is not an "op".  Store should not be canonicalized.
    // CHECK: tt.store %{{.*}}, %{{.*}}, %{{.*}} : tensor<8x!tt.ptr<f32>>
    tt.store %ptr, %val, %mask : tensor<8x!tt.ptr<f32>>
    tt.return
}
```
**EN:** This block defines `test_canonicalize_masked_store_fail_pattern` and exercises tensor stores. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_canonicalize_masked_store_fail_pattern`，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 314-318
```mlir
// CHECK-LABEL: @test_canonicalize_expand_dims
tt.func @test_canonicalize_expand_dims(%arg0: tensor<f32>, %arg1: tensor<1xf32>) -> (tensor<1x8xf32>, tensor<8x8xf32>) {
    %splat = tt.splat %arg0 : tensor<f32> -> tensor<8xf32>
    // CHECK: %{{.*}} = tt.splat %arg0 : tensor<f32> -> tensor<1x8xf32>
    %ed = tt.expand_dims %splat {axis = 0 : i32} : tensor<8xf32> -> tensor<1x8xf32>
```
**EN:** This block defines `test_canonicalize_expand_dims` and exercises operations such as `tt.func`, `tt.splat`, `tt.expand_dims`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_canonicalize_expand_dims`，并覆盖 如 `tt.func`、`tt.splat`、`tt.expand_dims` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 320-324
```mlir
    // CHECK-NEXT: %[[ed2:.*]] = tt.expand_dims %arg1 {axis = 0 : i32} : tensor<1xf32> -> tensor<1x1xf32>
    // CHECK-NEXT: %{{.*}} = tt.broadcast %[[ed2]] : tensor<1x1xf32> -> tensor<8x8xf32>
    %bc = tt.broadcast %arg1 : tensor<1xf32> -> tensor<8xf32>
    %ed2 = tt.expand_dims %bc {axis = 0 : i32} : tensor<8xf32> -> tensor<1x8xf32>
    %bc2 = tt.broadcast %ed2 : tensor<1x8xf32> -> tensor<8x8xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.expand_dims`, `tt.broadcast`. Embedded check comments (CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.expand_dims`、`tt.broadcast` 这样的操作。 其中嵌入的检查注释（CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 326-327
```mlir
    tt.return %ed, %bc2 : tensor<1x8xf32>, tensor<8x8xf32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 329-333
```mlir
// CHECK-LABEL: @test_canonicalize_view
tt.func @test_canonicalize_view(%arg0: tensor<8xf32>, %arg1: tensor<f32>) -> (tensor<4x2xf32>, tensor<2x2x2xf32>, tensor<8xf32>, tensor<2x2x2xf32>) {
    %view0 = tt.reshape %arg0 allow_reorder : tensor<8xf32> -> tensor<2x4xf32>
    // CHECK: %{{.*}} = tt.reshape %arg0 allow_reorder : tensor<8xf32> -> tensor<4x2xf32>
    %view1 = tt.reshape %view0 allow_reorder : tensor<2x4xf32> -> tensor<4x2xf32>
```
**EN:** This block defines `test_canonicalize_view` and exercises operations such as `tt.func`, `tt.reshape`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_canonicalize_view`，并覆盖 如 `tt.func`、`tt.reshape` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 335-337
```mlir
    %splat = tt.splat %arg1 : tensor<f32> -> tensor<8xf32>
    // CHECK: %{{.*}} = tt.splat %arg1 : tensor<f32> -> tensor<2x2x2xf32>
    %view2 = tt.reshape %splat allow_reorder : tensor<8xf32> -> tensor<2x2x2xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.splat`, `tt.reshape`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.splat`、`tt.reshape` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 339-341
```mlir
    %view3 = tt.reshape %arg0 : tensor<8xf32> -> tensor<8xf32>
    // CHECK: %{{.*}} = arith.addf %arg0, %arg0 : tensor<8xf32>
    %add = arith.addf %view3, %arg0 : tensor<8xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.reshape`, `arith.addf`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.reshape`、`arith.addf` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 343-344
```mlir
    // CHECK: %{{.*}} = tt.reshape %arg0 allow_reorder : tensor<8xf32> -> tensor<2x2x2xf32>
    %reshape = tt.reshape %view0 : tensor<2x4xf32> -> tensor<2x2x2xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.reshape`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.reshape` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 346-347
```mlir
    tt.return %view1, %view2, %add, %reshape : tensor<4x2xf32>, tensor<2x2x2xf32>, tensor<8xf32>, tensor<2x2x2xf32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 349-353
```mlir
// CHECK-LABEL: @test_canonicalize_reshape
tt.func @test_canonicalize_reshape(%arg0: tensor<8xf32>, %arg1: tensor<f32>) -> (tensor<4x2xf32>, tensor<2x2x2xf32>, tensor<8xf32>, tensor<2x2x2xf32>) {
    %reshape0 = tt.reshape %arg0 : tensor<8xf32> -> tensor<2x4xf32>
    // CHECK: %{{.*}} = tt.reshape %arg0 : tensor<8xf32> -> tensor<4x2xf32>
    %reshape1 = tt.reshape %reshape0 : tensor<2x4xf32> -> tensor<4x2xf32>
```
**EN:** This block defines `test_canonicalize_reshape` and exercises operations such as `tt.func`, `tt.reshape`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_canonicalize_reshape`，并覆盖 如 `tt.func`、`tt.reshape` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 355-357
```mlir
    %splat = tt.splat %arg1 : tensor<f32> -> tensor<8xf32>
    // CHECK: %{{.*}} = tt.splat %arg1 : tensor<f32> -> tensor<2x2x2xf32>
    %reshape2 = tt.reshape %splat : tensor<8xf32> -> tensor<2x2x2xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.splat`, `tt.reshape`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.splat`、`tt.reshape` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 359-361
```mlir
    %reshape3 = tt.reshape %arg0 : tensor<8xf32> -> tensor<8xf32>
    // CHECK: %{{.*}} = arith.addf %arg0, %arg0 : tensor<8xf32>
    %add = arith.addf %reshape3, %arg0 : tensor<8xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.reshape`, `arith.addf`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.reshape`、`arith.addf` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 363-364
```mlir
    // CHECK: %{{.*}} = tt.reshape %arg0 allow_reorder : tensor<8xf32> -> tensor<2x2x2xf32>
    %view = tt.reshape %reshape0 allow_reorder : tensor<2x4xf32> -> tensor<2x2x2xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.reshape`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.reshape` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 366-367
```mlir
    tt.return %reshape1, %reshape2, %add, %view : tensor<4x2xf32>, tensor<2x2x2xf32>, tensor<8xf32>, tensor<2x2x2xf32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 369-373
```mlir
// CHECK-LABEL: @test_canonicalize_broadcast
tt.func @test_canonicalize_broadcast(%arg0: tensor<1x1x8xf32>, %arg1: tensor<f32>) -> (tensor<4x2x8xf32>, tensor<8x8xf32>, tensor<1x1x8xf32>) {
    %broadcast0 = tt.broadcast %arg0 : tensor<1x1x8xf32> -> tensor<1x2x8xf32>
    // CHECK: %{{.*}} = tt.broadcast %arg0 : tensor<1x1x8xf32> -> tensor<4x2x8xf32>
    %broadcast1 = tt.broadcast %broadcast0 : tensor<1x2x8xf32> -> tensor<4x2x8xf32>
```
**EN:** This block defines `test_canonicalize_broadcast` and exercises operations such as `tt.func`, `tt.broadcast`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_canonicalize_broadcast`，并覆盖 如 `tt.func`、`tt.broadcast` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 375-377
```mlir
    %splat = tt.splat %arg1 : tensor<f32> -> tensor<1x8xf32>
    // CHECK: %{{.*}} = tt.splat %arg1 : tensor<f32> -> tensor<8x8xf32>
    %broadcast2 = tt.broadcast %splat : tensor<1x8xf32> -> tensor<8x8xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.splat`, `tt.broadcast`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.splat`、`tt.broadcast` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 379-381
```mlir
    %broadcast3 = tt.broadcast %arg0 : tensor<1x1x8xf32> -> tensor<1x1x8xf32>
    // CHECK: %{{.*}} = arith.addf %arg0, %arg0 : tensor<1x1x8xf32>
    %add = arith.addf %broadcast3, %arg0 : tensor<1x1x8xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `tt.broadcast`, `arith.addf`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `tt.broadcast`、`arith.addf` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 383-384
```mlir
    tt.return %broadcast1, %broadcast2, %add : tensor<4x2x8xf32>, tensor<8x8xf32>, tensor<1x1x8xf32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 386-388
```mlir
// CHECK-LABEL: @test_fold_views
tt.func @test_fold_views() -> (tensor<16x8xf32>, tensor<16x128xf32>, tensor<1x1x128xf32>) {
    %a = arith.constant dense<1.0> : tensor<1x128xf32>
```
**EN:** This block defines `test_fold_views` and exercises operations such as `tt.func`, `arith.constant`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_fold_views`，并覆盖 如 `tt.func`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 390-391
```mlir
    // CHECK-DAG: %{{.*}} = arith.constant dense<1.{{.*}}> : tensor<16x8xf32>
    %b = tt.reshape %a allow_reorder : tensor<1x128xf32> -> tensor<16x8xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `arith.constant`, `tt.reshape`. Embedded check comments (CHECK-DAG×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `arith.constant`、`tt.reshape` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 393-394
```mlir
    // CHECK-DAG: %{{.*}} = arith.constant dense<1.{{.*}}> : tensor<16x128xf32>
    %c = tt.broadcast %a : tensor<1x128xf32> -> tensor<16x128xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `arith.constant`, `tt.broadcast`. Embedded check comments (CHECK-DAG×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `arith.constant`、`tt.broadcast` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 396-397
```mlir
    // CHECK-DAG: %{{.*}} = arith.constant dense<1.{{.*}}> : tensor<1x1x128xf32>
    %d = tt.expand_dims %a {axis = 0: i32} : tensor<1x128xf32> -> tensor<1x1x128xf32>
```
**EN:** This block defines the test function(s) and exercises operations such as `arith.constant`, `tt.expand_dims`. Embedded check comments (CHECK-DAG×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `arith.constant`、`tt.expand_dims` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 399-400
```mlir
    tt.return %b, %c, %d : tensor<16x8xf32>, tensor<16x128xf32>, tensor<1x1x128xf32>
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 402-407
```mlir
// CHECK-LABEL: @test_nop_transpose
tt.func @test_nop_transpose(%arg0: tensor<2x4xf32>) -> (tensor<2x4xf32>) {
    %a = tt.trans %arg0 {order = array<i32: 0, 1>} : tensor<2x4xf32> -> tensor<2x4xf32>
    // CHECK: tt.return %arg0
    tt.return %a : tensor<2x4xf32>
}
```
**EN:** This block defines `test_nop_transpose` and exercises operations such as `tt.func`, `tt.trans`, `tt.return`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_nop_transpose`，并覆盖 如 `tt.func`、`tt.trans`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 409-416
```mlir
// CHECK-LABEL: @test_nested_transpose
tt.func @test_nested_transpose(%arg0: tensor<2x4x8xf32>) -> (tensor<8x2x4xf32>) {
    %a = tt.trans %arg0 {order = array<i32: 1, 0, 2>} : tensor<2x4x8xf32> -> tensor<4x2x8xf32>
    %b = tt.trans %a {order = array<i32: 2, 1, 0>} : tensor<4x2x8xf32> -> tensor<8x2x4xf32>
    // CHECK: %[[res:.*]] = tt.trans %arg0 {order = array<i32: 2, 0, 1>}
    // CHECK: tt.return %[[res]]
    tt.return %b : tensor<8x2x4xf32>
}
```
**EN:** This block defines `test_nested_transpose` and exercises operations such as `tt.func`, `tt.trans`, `tt.return`. Embedded check comments (CHECK×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_nested_transpose`，并覆盖 如 `tt.func`、`tt.trans`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 418-429
```mlir
// CHECK-LABEL: test_reshape_reduce
tt.func @test_reshape_reduce(%0: tensor<32x4x2xi32>) -> (i32, tensor<16xi32>) {
  // CHECK: tt.reshape %{{.+}} allow_reorder : tensor<32x4x2xi32> -> tensor<256xi32>
  %1 = tt.reshape %0 : tensor<32x4x2xi32> -> tensor<256xi32>
  %2 = "tt.reduce" (%1) ({
    ^bb0(%arg7: i32, %arg8: i32):
      %add = arith.addi %arg7, %arg8 : i32
      tt.reduce.return %add : i32
    }) {axis = 0 : i32} : (tensor<256xi32>) -> i32
  %3 = tt.histogram %1 : tensor<256xi32> -> tensor<16xi32>
  tt.return %2, %3 : i32, tensor<16xi32>
}
```
**EN:** This block defines `test_reshape_reduce` and exercises operations such as `tt.func`, `tt.reshape`, `tt.reduce`, `arith.addi`, `tt.reduce.return`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_reshape_reduce`，并覆盖 如 `tt.func`、`tt.reshape`、`tt.reduce`、`arith.addi`、`tt.reduce.return` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 431-439
```mlir
// CHECK-LABEL: test_rank_reduce_desc_load
tt.func @test_rank_reduce_desc_load(%0: !tt.tensordesc<1x128x64xf16>) -> (tensor<128x64xf16>) {
  %c0 = arith.constant 0 : i32
  // CHECK: %[[R:.+]] = tt.descriptor_load {{.*}} : !tt.tensordesc<1x128x64xf16> -> tensor<128x64xf16>
  // CHECK: tt.return %[[R]]
  %l = tt.descriptor_load %0[%c0, %c0, %c0] : !tt.tensordesc<1x128x64xf16> -> tensor<1x128x64xf16>
  %r = tt.reshape %l : tensor<1x128x64xf16> -> tensor<128x64xf16>
  tt.return %r :  tensor<128x64xf16>
}
```
**EN:** This block defines `test_rank_reduce_desc_load` and exercises operations such as `tt.func`, `tt.tensordesc`, `arith.constant`, `tt.descriptor_load`, `tt.return`. Embedded check comments (CHECK×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_rank_reduce_desc_load`，并覆盖 如 `tt.func`、`tt.tensordesc`、`arith.constant`、`tt.descriptor_load`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 441-447
```mlir
// CHECK-LABEL: @test_combine_dot_add_no_fold_when_imprecise_allowed
tt.func @test_combine_dot_add_no_fold_when_imprecise_allowed() -> (tensor<128x128xf32>) {
    // CHECK-DAG: %[[D:.*]] = arith.constant dense<3.000000e+00> : tensor<128x128xf32>
    %a    = arith.constant dense<1.0> : tensor<128x128xf32>
    %b    = arith.constant dense<2.0> : tensor<128x128xf32>
    %zero = arith.constant dense<0.0> : tensor<128x128xf32>
    %d    = arith.constant dense<3.0> : tensor<128x128xf32>
```
**EN:** This block defines `test_combine_dot_add_no_fold_when_imprecise_allowed` and exercises operations such as `tt.func`, `arith.constant`. Embedded check comments (CHECK-DAG×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_dot_add_no_fold_when_imprecise_allowed`，并覆盖 如 `tt.func`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 449-450
```mlir
    %dot_out = tt.dot %a, %b, %zero {maxNumImpreciseAcc = 1 : i32}
               : tensor<128x128xf32> * tensor<128x128xf32> -> tensor<128x128xf32>
```
**EN:** This block contributes intermediate IR built from `tt.dot`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 452-456
```mlir
    // CHECK: arith.addf %{{.*}}, %[[D]] : tensor<128x128xf32>
    // CHECK-NEXT: tt.return %{{.*}} : tensor<128x128xf32>
    %res = arith.addf %dot_out, %d : tensor<128x128xf32>
    tt.return %res : tensor<128x128xf32>
}
```
**EN:** This block defines the test function(s) and exercises operations such as `arith.addf`, `tt.return`. Embedded check comments (CHECK×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `arith.addf`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 458-466
```mlir
// CHECK-LABEL: @test_combine_dot_add_fold_when_precise_required
tt.func @test_combine_dot_add_fold_when_precise_required() -> (tensor<128x128xf32>) {
    // CHECK-DAG: %[[D:.*]] = arith.constant dense<3.000000e+00> : tensor<128x128xf32>
    // CHECK-DAG: %[[B:.*]] = arith.constant dense<2.000000e+00> : tensor<128x128xf32>
    // CHECK-DAG: %[[A:.*]] = arith.constant dense<1.000000e+00> : tensor<128x128xf32>
    %a    = arith.constant dense<1.0> : tensor<128x128xf32>
    %b    = arith.constant dense<2.0> : tensor<128x128xf32>
    %zero = arith.constant dense<0.0> : tensor<128x128xf32>
    %d    = arith.constant dense<3.0> : tensor<128x128xf32>
```
**EN:** This block defines `test_combine_dot_add_fold_when_precise_required` and exercises operations such as `tt.func`, `arith.constant`. Embedded check comments (CHECK-DAG×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_dot_add_fold_when_precise_required`，并覆盖 如 `tt.func`、`arith.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 468-469
```mlir
    %dot_out = tt.dot %a, %b, %zero {maxNumImpreciseAcc = 0 : i32}
               : tensor<128x128xf32> * tensor<128x128xf32> -> tensor<128x128xf32>
```
**EN:** This block contributes intermediate IR built from `tt.dot`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 471-475
```mlir
    // CHECK-NEXT: %[[RES:.*]] = tt.dot %[[A]], %[[B]], %[[D]] : tensor<128x128xf32> * tensor<128x128xf32> -> tensor<128x128xf32>
    // CHECK-NEXT: tt.return %[[RES]] : tensor<128x128xf32>
    %res = arith.addf %dot_out, %d : tensor<128x128xf32>
    tt.return %res : tensor<128x128xf32>
}
```
**EN:** This block defines the test function(s) and exercises dot products / matmul. Embedded check comments (CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 点积/矩阵乘。 其中嵌入的检查注释（CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 477-493
```mlir
// CHECK-LABEL: @test_combine_broadcast_mul_reduce
tt.func @test_combine_broadcast_mul_reduce(%arg0: tensor<32x16xf32>, %arg1: tensor<16x32xf32>) -> tensor<32x32xf32> {
    // CHECK: %[[CST:.*]] = arith.constant dense<0.000000e+00> : tensor<32x32xf32>
    // CHECK: %[[RES:.*]] = tt.dot %{{.*}}, %{{.*}}, %[[CST]] : tensor<32x16xf32> * tensor<16x32xf32> -> tensor<32x32xf32>
    // CHECK: tt.return %[[RES]] : tensor<32x32xf32>
    %0 = tt.expand_dims %arg0 {axis = 2 : i32} : tensor<32x16xf32> -> tensor<32x16x1xf32>
    %1 = tt.broadcast %0 : tensor<32x16x1xf32> -> tensor<32x16x32xf32>
    %2 = tt.expand_dims %arg1 {axis = 0 : i32} : tensor<16x32xf32> -> tensor<1x16x32xf32>
    %3 = tt.broadcast %2 : tensor<1x16x32xf32> -> tensor<32x16x32xf32>
    %4 = arith.mulf %1, %3 : tensor<32x16x32xf32>
    %5 = "tt.reduce"(%4) <{axis = 1 : i32}> ({
    ^bb0(%arg2: f32, %arg3: f32):
        %6 = arith.addf %arg2, %arg3 : f32
        tt.reduce.return %6 : f32
    }) : (tensor<32x16x32xf32>) -> tensor<32x32xf32>
    tt.return %5 : tensor<32x32xf32>
}
```
**EN:** This block defines `test_combine_broadcast_mul_reduce` and exercises dot products / matmul. Embedded check comments (CHECK×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_combine_broadcast_mul_reduce`，并覆盖 点积/矩阵乘。 其中嵌入的检查注释（CHECK×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's Triton dialect canonicalization coverage and focuses on combining patterns.  
  **CN:** 该文件属于 Triton 的 Triton 方言规范化 测试覆盖，关注点是 合并模式。
- **EN:** The `RUN` pipeline drives tools/passes such as `-canonicalize`, `-triton-combine`.  
  **CN:** `RUN` 流水线会驱动 `-canonicalize`, `-triton-combine` 等工具/pass。
- **EN:** The IR mixes `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `tt`, `arith`.  
  **CN:** IR 方言依赖：`tt`、`arith`。
