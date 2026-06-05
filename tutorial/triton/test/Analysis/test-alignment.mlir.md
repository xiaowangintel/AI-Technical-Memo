# test-alignment.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Analysis/test-alignment.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises alignment and axis information in Triton's analysis coverage. **CN:** 该MLIR 测试用于覆盖 Triton 分析中的对齐与轴信息相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -test-print-alignment -split-input-file -verify-diagnostics=only-expected -o /dev/null` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: none; diagnostics annotations: remark×346. **CN:** FileCheck 标记为 none；诊断标注为 remark×346。
- **Validation / 验证内容:** **EN:** The file mainly validates that alignment and axis information emits the expected analysis remarks/notes. **CN:** 该文件主要验证 对齐与轴信息 会发出预期的分析备注/说明。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -test-print-alignment -split-input-file -verify-diagnostics=only-expected -o /dev/null
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -test-print-alignment -split-input-file -verify-diagnostics=only-expected -o /dev/null` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -test-print-alignment -split-input-file -verify-diagnostics=only-expected -o /dev/null`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-14
```mlir
tt.func @cast() {
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = 1}}
  %cst = arith.constant 1 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = 1}}
  %0 = arith.extsi %cst : i32 to i64
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = 1}}
  %cst_tensor = arith.constant dense<1> : tensor<128xi32>
  // Bitcast preserves axis info for same-width types.
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = 1}}
  %1 = tt.bitcast %cst_tensor : tensor<128xi32> -> tensor<128xf32>
  tt.return
}
```
**EN:** This block defines `cast` and exercises operations such as `tt.func`, `arith.constant`, `arith.extsi`, `tt.bitcast`, `tt.return`. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `cast`，并覆盖 如 `tt.func`、`arith.constant`、`arith.extsi`、`tt.bitcast`、`tt.return` 这样的操作。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Line 16
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 18-54
```mlir
tt.func @add(%arg0: tensor<128xi32> {tt.contiguity = 1 : i32, tt.divisibility = 4 : i32, tt.constancy = 2: i32}, %arg1: tensor<128xi32> {tt.contiguity = 4 : i32, tt.divisibility = 4 : i32, tt.constancy = 1: i32}) {
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = 1}}
  %1 = arith.constant dense<1> : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [1], constancy = [1], constant_value = <none>}}
  %2 = arith.addi %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = 127}}
  %3 = arith.constant dense<127> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [128], constancy = [128], constant_value = 128}}
  %4 = arith.addi %1, %3 : tensor<128xi32>
  // Contiguous + contiguous can overestimate divisibility if we only take
  // gcd(divisibility) (divisibility is defined on contiguity-group bases).
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [1], constant_value = <none>}}
  %5 = arith.addi %0, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [1], constancy = [1], constant_value = <none>}}
  %odd = tt.make_range {end = 129 : i32, start = 1 : i32} : tensor<128xi32>
  // Contiguous + contiguous with different base parity yields an odd sequence.
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %even_plus_odd = arith.addi %0, %odd : tensor<128xi32>
  // Contiguous + contiguous with odd bases may yield an even sequence but we still conservatively estimate the
  // divisibility as 1.
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %odd_plus_odd = arith.addi %odd, %odd : tensor<128xi32>
  // Partial constant + contiguous. The resultant contiguity is smaller than the operands'.
  // [4, 4, 8, 8, 12, 12, ...] + [0, 1, 2, 3, 4, 5, ...]
  // expected-remark @below {{contiguity = [2], divisibility = [2], constancy = [1], constant_value = <none>}}
  %9 = arith.addi %0, %arg0 : tensor<128xi32>
  // Partial contiguous + partial contiguous can also overestimate divisibility.
  // [0, 1, 2, 3, 0, 1, 2, 3, ...] + [0, 1, 2, 3, 0, 1, 2, 3, ...]
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [1], constant_value = <none>}}
  %11 = arith.addi %arg1, %arg1 : tensor<128xi32>
  // Partial constant + partial constant.
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [2], constant_value = <none>}}
  %12 = arith.addi %arg0, %arg0 : tensor<128xi32>
  tt.return
}
```
**EN:** This block defines `add` and exercises range generation. Expected-diagnostic annotations (remark×12) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `add`，并覆盖 范围生成。 期望诊断标注（remark×12）用于捕获 pass 应当发出的分析备注/说明。

### Line 56
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 58-110
```mlir
tt.func @addptr(%arg0: !tt.ptr<i1> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<i16> {tt.divisibility = 16 : i32}, %arg3: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %arg4: !tt.ptr<i64> {tt.divisibility = 16 : i32}) {
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = 1}}
  %cst1 = arith.constant 1 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %0 = tt.addptr %arg0, %cst1 : !tt.ptr<i1>, i32
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %1 = tt.addptr %arg1, %cst1 : !tt.ptr<i8>, i32
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [1], constant_value = <none>}}
  %2 = tt.addptr %arg2, %cst1 : !tt.ptr<i16>, i32
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = <none>}}
  %3 = tt.addptr %arg3, %cst1 : !tt.ptr<i32>, i32
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [1], constant_value = <none>}}
  %4 = tt.addptr %arg4, %cst1 : !tt.ptr<i64>, i32
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = 4}}
  %cst4 = arith.constant 4 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = <none>}}
  %5 = tt.addptr %arg0, %cst4 : !tt.ptr<i1>, i32
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = <none>}}
  %6 = tt.addptr %arg1, %cst4 : !tt.ptr<i8>, i32
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [1], constant_value = <none>}}
  %7 = tt.addptr %arg2, %cst4 : !tt.ptr<i16>, i32
  // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [1], constant_value = <none>}}
  %8 = tt.addptr %arg3, %cst4 : !tt.ptr<i32>, i32
  // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [1], constant_value = <none>}}
  %9 = tt.addptr %arg4, %cst4 : !tt.ptr<i64>, i32
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %10 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1, 128], divisibility = [1, 1073741824], constancy = [1, 1], constant_value = <none>}}
  %11 = tt.expand_dims %10 {axis = 0: i32} : tensor<128xi32> -> tensor<1x128xi32>
  // expected-remark @below {{contiguity = [1, 128], divisibility = [1, 1073741824], constancy = [128, 1], constant_value = <none>}}
  %12 = tt.broadcast %11 : tensor<1x128xi32> -> tensor<128x128xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 128], constant_value = <none>}}
  %13 = tt.splat %arg0 : !tt.ptr<i1> -> tensor<128x128x!tt.ptr<i1>>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 128], constant_value = <none>}}
  %14 = tt.splat %arg1 : !tt.ptr<i8> -> tensor<128x128x!tt.ptr<i8>>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 128], constant_value = <none>}}
  %15 = tt.splat %arg2 : !tt.ptr<i16> -> tensor<128x128x!tt.ptr<i16>>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 128], constant_value = <none>}}
  %16 = tt.splat %arg3 : !tt.ptr<i32> -> tensor<128x128x!tt.ptr<i32>>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 128], constant_value = <none>}}
  %17 = tt.splat %arg4 : !tt.ptr<i64> -> tensor<128x128x!tt.ptr<i64>>
  // expected-remark @below {{contiguity = [1, 128], divisibility = [1, 16], constancy = [128, 1], constant_value = <none>}}
  %18 = tt.addptr %13, %12 : tensor<128x128x!tt.ptr<i1>>, tensor<128x128xi32>
  // expected-remark @below {{contiguity = [1, 128], divisibility = [1, 16], constancy = [128, 1], constant_value = <none>}}
  %19 = tt.addptr %14, %12 : tensor<128x128x!tt.ptr<i8>>, tensor<128x128xi32>
  // expected-remark @below {{contiguity = [1, 128], divisibility = [2, 16], constancy = [128, 1], constant_value = <none>}}
  %20 = tt.addptr %15, %12 : tensor<128x128x!tt.ptr<i16>>, tensor<128x128xi32>
  // expected-remark @below {{contiguity = [1, 128], divisibility = [4, 16], constancy = [128, 1], constant_value = <none>}}
  %21 = tt.addptr %16, %12 : tensor<128x128x!tt.ptr<i32>>, tensor<128x128xi32>
  // expected-remark @below {{contiguity = [1, 128], divisibility = [8, 16], constancy = [128, 1], constant_value = <none>}}
  %22 = tt.addptr %17, %12 : tensor<128x128x!tt.ptr<i64>>, tensor<128x128xi32>
  tt.return
}
```
**EN:** This block defines `addptr` and exercises pointer arithmetic, range generation. Expected-diagnostic annotations (remark×25) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `addptr`，并覆盖 指针运算、范围生成。 期望诊断标注（remark×25）用于捕获 pass 应当发出的分析备注/说明。

### Line 112
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 114-162
```mlir
tt.func @sub(%arg0: tensor<128xi32> {tt.contiguity = 1 : i32, tt.divisibility = 4 : i32, tt.constancy = 2: i32}, %arg1: tensor<128xi32> {tt.contiguity = 4 : i32, tt.divisibility = 4 : i32, tt.constancy = 1: i32}) {
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = 1}}
  %1 = arith.constant dense<1> : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [1], constancy = [1], constant_value = <none>}}
  %2 = arith.subi %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %3 = arith.subi %1, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = 129}}
  %4 = arith.constant dense<129> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [128], constancy = [128], constant_value = 128}}
  %5 = arith.subi %4, %1 : tensor<128xi32>
  // Result contiguity depends on RHS constancy for subi; ensure divisibility is
  // clamped when result contiguity is smaller than operand contiguity.
  // expected-remark @below {{contiguity = [128], divisibility = [16], constancy = [1], constant_value = <none>}}
  %rhs_range = tt.make_range {end = 144 : i32, start = 16 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [1], constant_value = <none>}}
  %sub_clamp = arith.subi %0, %rhs_range : tensor<128xi32>
  // Both operands are contiguous, but with different contiguity groups. In this
  // case, we conservatively infer divisibility from parity: even - even is
  // divisible by 2.
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [1], constant_value = <none>}}
  %even_even_diff_contig = arith.subi %0, %arg1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [4], divisibility = [1], constancy = [1], constant_value = <none>}}
  %arg1_plus_one = arith.addi %arg1, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %6 = arith.subi %0, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [1], constancy = [1], constant_value = <none>}}
  %odd = tt.make_range {end = 129 : i32, start = 1 : i32} : tensor<128xi32>
  // Contiguous - contiguous with different base parity yields an odd constant.
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %even_minus_odd = arith.subi %0, %odd : tensor<128xi32>
  // Contiguous - contiguous. With both only odd divisibilities.
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %odd_minus_arg1_plus_one = arith.subi %odd, %arg1_plus_one : tensor<128xi32>
  // Partial constant - contiguous. The resultant contiguity is smaller than the LHS'.
  // [0, 1, 2, 3, 4, 5, ...] - [4, 4, 8, 8, 12, 12, ...]
  // expected-remark @below {{contiguity = [2], divisibility = [2], constancy = [1], constant_value = <none>}}
  %7 = arith.subi %0, %arg0 : tensor<128xi32>
  // Same contiguity - shortcut optimization for divisibility
  // [4, 5, 6, 7, 4, 5, 6, 7, ...] - [4, 5, 6, 7, 4, 5, 6, 7, ...]
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = <none>}}
  %8 = arith.subi %arg1, %arg1 : tensor<128xi32>
  // Partial constant - partial constant.
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [2], constant_value = <none>}}
  %9 = arith.subi %arg0, %arg0 : tensor<128xi32>
  tt.return
}
```
**EN:** This block defines `sub` and exercises range generation. Expected-diagnostic annotations (remark×17) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `sub`，并覆盖 范围生成。 期望诊断标注（remark×17）用于捕获 pass 应当发出的分析备注/说明。

### Line 164
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 166-186
```mlir
tt.func @mul(%arg0: i64 {tt.divisibility = 16 : i32}) {
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = 1}}
  %1 = arith.constant dense<1> : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %2 = arith.muli %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [128], constancy = [128], constant_value = 128}}
  %3 = arith.constant dense<128> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [128], constancy = [128], constant_value = 128}}
  %4 = arith.muli %3, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [128], constant_value = 2}}
  %5 = arith.constant dense<2> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [256], constancy = [128], constant_value = 256}}
  %6 = arith.muli %4, %5 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [1], constant_value = 4611686018427387904}}
  %7 = arith.constant 4611686018427387904: i64
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [1], constant_value = <none>}}
  %8 = arith.muli %arg0, %7 : i64
  tt.return
}
```
**EN:** This block defines `mul` and exercises range generation. Expected-diagnostic annotations (remark×9) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `mul`，并覆盖 范围生成。 期望诊断标注（remark×9）用于捕获 pass 应当发出的分析备注/说明。

### Line 188
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 190-233
```mlir
tt.func @div(%arg0: i32 {tt.divisibility = 16 : i32}) {
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = 1}}
  %1 = arith.constant dense<1> : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %2 = arith.divsi %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %3 = arith.divui %1, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [128], constant_value = 64}}
  %4 = arith.constant dense<64> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [64], constant_value = <none>}}
  %5 = arith.divsi %0, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %6 = arith.divsi %4, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [128], constant_value = 64}}
  %7 = arith.divsi %4, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [128], constant_value = 66}}
  %8 = arith.constant dense<66> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [2], constant_value = <none>}}
  %9 = arith.divui %0, %8 : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [8192], constancy = [1], constant_value = <none>}}
  %10 = tt.make_range {end = 8320 : i32, start = 8192 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [64], constant_value = <none>}}
  %11 = arith.divsi %10, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [1], constant_value = 2}}
  %12 = arith.constant 2 : i32
  // dividing a scalar by a power of two should give predictable divisibility
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [1], constant_value = <none>}}
  %13 = arith.divsi %arg0, %12 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [32], constancy = [1], constant_value = 32}}
  %14 = arith.constant 32 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %15 = arith.divsi %arg0, %14 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [1], constant_value = 6}}
  %16 = arith.constant 6 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %17 = arith.divsi %arg0, %16 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [128], constant_value = 2}}
  %18 = arith.constant dense<2> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [2], constant_value = <none>}}
  %19 = arith.divsi %0, %18 : tensor<128xi32>
  tt.return
}
```
**EN:** This block defines `div` and exercises range generation. Expected-diagnostic annotations (remark×20) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `div`，并覆盖 范围生成。 期望诊断标注（remark×20）用于捕获 pass 应当发出的分析备注/说明。

### Line 236
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 238-274
```mlir
tt.func @rem() {
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = 1}}
  %1 = arith.constant dense<1> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [128], constant_value = 0}}
  %2 = arith.remsi %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %3 = arith.remui %1, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [128], constant_value = 64}}
  %4 = arith.constant dense<64> : tensor<128xi32>
  // expected-remark @below {{contiguity = [64], divisibility = [64], constancy = [1], constant_value = <none>}}
  %5 = arith.remsi %0, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %6 = arith.remsi %4, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [128], constant_value = 66}}
  %7 = arith.constant dense<66> : tensor<128xi32>
  // expected-remark @below {{contiguity = [2], divisibility = [2], constancy = [1], constant_value = <none>}}
  %8 = arith.remui %0, %7 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [128], constant_value = 192}}
  %9 = arith.constant dense<192> : tensor<128xi32>
  // expected-remark @below {{contiguity = [64], divisibility = [64], constancy = [1], constant_value = <none>}}
  %10 = arith.remsi %0, %9 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %11 = arith.remsi %9, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [32], constancy = [1], constant_value = <none>}}
  %12 = tt.make_range {end = 160 : i32, start = 32 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %13 = arith.remsi %0, %12 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %14 = arith.remsi %12, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [32], divisibility = [32], constancy = [1], constant_value = <none>}}
  %15 = arith.remsi %12, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %16 = arith.remsi %4, %12 : tensor<128xi32>
  tt.return
}
```
**EN:** This block defines `rem` and exercises range generation. Expected-diagnostic annotations (remark×17) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `rem`，并覆盖 范围生成。 期望诊断标注（remark×17）用于捕获 pass 应当发出的分析备注/说明。

### Line 276
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 278-288
```mlir
tt.func @expanddims() {
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [128], constant_value = 2}}
  %1 = arith.constant dense<2> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [1], constant_value = <none>}}
  %2 = arith.muli %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [2, 2], constancy = [1, 1], constant_value = <none>}}
  %3 = tt.expand_dims %2 {axis = 1 : i32} : tensor<128xi32> -> tensor<128x1xi32>
  tt.return
}
```
**EN:** This block defines `expanddims` and exercises range generation. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `expanddims`，并覆盖 范围生成。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Line 290
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 292-324
```mlir
tt.func @reshape(%arg0: tensor<8xi32> {tt.contiguity = 1 : i32, tt.divisibility = 8 : i32, tt.constancy = 4 : i32}) {
  // expected-remark @below {{contiguity = [4], divisibility = [4], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 8 : i32, start = 4 : i32} : tensor<4xi32>
  // expected-remark @below {{contiguity = [1, 1, 4], divisibility = [1, 1, 4], constancy = [1, 1, 1], constant_value = <none>}}
  %1 = tt.reshape %0 : tensor<4xi32> -> tensor<1x1x4xi32>
  // expected-remark @below {{contiguity = [16], divisibility = [4], constancy = [1], constant_value = <none>}}
  %2 = tt.make_range {end = 20 : i32, start = 4 : i32} : tensor<16xi32>
  // expected-remark @below {{contiguity = [1, 4], divisibility = [1, 4], constancy = [1, 1], constant_value = <none>}}
  %3 = tt.reshape %2 : tensor<16xi32> -> tensor<4x4xi32>
  // expected-remark @below {{contiguity = [4], divisibility = [4], constancy = [1], constant_value = <none>}}
  %4 = tt.reshape %3 : tensor<4x4xi32> -> tensor<16xi32>
  // expected-remark @below {{contiguity = [4, 1], divisibility = [4, 1], constancy = [1, 1], constant_value = <none>}}
  %5 = tt.trans %3 {order = array<i32: 1, 0>} : tensor<4x4xi32> -> tensor<4x4xi32>
  // expected-remark @below {{contiguity = [1, 2, 1], divisibility = [1, 2, 1], constancy = [1, 1, 1], constant_value = <none>}}
  %6 = tt.reshape %5 : tensor<4x4xi32> -> tensor<2x2x4xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %7 = tt.reshape %5 : tensor<4x4xi32> -> tensor<16xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [8, 8], constancy = [2, 2], constant_value = <none>}}
  %8 = tt.reshape %arg0 : tensor<8xi32> -> tensor<4x2xi32>
  // expected-remark @below {{contiguity = [8], divisibility = [16], constancy = [1], constant_value = <none>}}
  %9 = tt.make_range {end = 24 : i32, start = 16 : i32} : tensor<8xi32>
  // expected-remark @below {{contiguity = [1, 8], divisibility = [1, 16], constancy = [1, 1], constant_value = <none>}}
  %10 = tt.reshape %9 : tensor<8xi32> -> tensor<1x8xi32>
  // expected-remark @below {{contiguity = [4], divisibility = [1], constancy = [1], constant_value = <none>}}
  %11 = tt.make_range {end = 5 : i32, start = 1 : i32} : tensor<4xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [4], constant_value = 2}}
  %12 = arith.constant dense<2> : tensor<4xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [1], constant_value = <none>}}
  %13 = arith.muli %11, %12 : tensor<4xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [2, 2], constancy = [1, 1], constant_value = <none>}}
  %14 = tt.reshape %13 : tensor<4xi32> -> tensor<1x4xi32>
  tt.return
}
```
**EN:** This block defines `reshape` and exercises range generation. Expected-diagnostic annotations (remark×15) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `reshape`，并覆盖 范围生成。 期望诊断标注（remark×15）用于捕获 pass 应当发出的分析备注/说明。

### Line 326
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 328-336
```mlir
tt.func @reshape_refined_piece_merge(
    %arg0: tensor<4x4xi32> {tt.contiguity = dense<[1, 1]> : tensor<2xi32>, tt.divisibility = dense<[1, 1]> : tensor<2xi32>, tt.constancy = dense<[2, 4]> : tensor<2xi32>},
    %arg1: tensor<2x2x2xi32> {tt.contiguity = dense<[1, 1, 2]> : tensor<3xi32>, tt.divisibility = dense<[1, 1, 1]> : tensor<3xi32>, tt.constancy = dense<[2, 2, 1]> : tensor<3xi32>}) {
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [8], constant_value = <none>}}
  %0 = tt.reshape %arg0 : tensor<4x4xi32> -> tensor<16xi32>
  // expected-remark @below {{contiguity = [1, 2], divisibility = [1, 1], constancy = [4, 1], constant_value = <none>}}
  %1 = tt.reshape %arg1 : tensor<2x2x2xi32> -> tensor<4x2xi32>
  tt.return
}
```
**EN:** This block defines `reshape_refined_piece_merge` and exercises operations such as `tt.func`, `tt.contiguity`, `tt.divisibility`, `tt.constancy`, `tt.reshape`. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `reshape_refined_piece_merge`，并覆盖 如 `tt.func`、`tt.contiguity`、`tt.divisibility`、`tt.constancy`、`tt.reshape` 这样的操作。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Line 338
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 340-348
```mlir
tt.func @broadcast() {
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [128], constant_value = 64}}
  %0 = arith.constant dense<64> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [64, 64], constancy = [128, 1], constant_value = 64}}
  %1 = tt.expand_dims %0 {axis = 1 : i32} : tensor<128xi32> -> tensor<128x1xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [64, 64], constancy = [128, 128], constant_value = 64}}
  %2 = tt.broadcast %1 : tensor<128x1xi32> -> tensor<128x128xi32>
  tt.return
}
```
**EN:** This block defines `broadcast` and exercises operations such as `tt.func`, `arith.constant`, `tt.expand_dims`, `tt.broadcast`, `tt.return`. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `broadcast`，并覆盖 如 `tt.func`、`arith.constant`、`tt.expand_dims`、`tt.broadcast`、`tt.return` 这样的操作。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Line 350
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 352-356
```mlir
tt.func @splat(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 128], constant_value = <none>}}
  %0 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<128x128x!tt.ptr<f32>>
  tt.return
}
```
**EN:** This block defines `splat` and exercises operations such as `tt.func`, `tt.ptr`, `tt.divisibility`, `tt.splat`, `tt.return`. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `splat`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.divisibility`、`tt.splat`、`tt.return` 这样的操作。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 358
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 360-396
```mlir
tt.func @cmp_all_contiguous() {
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [128], constant_value = 0}}
  %1 = arith.constant dense<0> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %2 = arith.cmpi eq, %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %3 = arith.cmpi ne, %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = <none>}}
  %4 = arith.cmpi slt, %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %5 = arith.cmpi sle, %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = <none>}}
  %6 = arith.cmpi sge, %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %7 = arith.cmpi sgt, %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %8 = arith.cmpi eq, %1, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %9 = arith.cmpi ne, %1, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %10 = arith.cmpi slt, %1, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = <none>}}
  %11 = arith.cmpi sle, %1, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %12 = arith.cmpi sge, %1, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = <none>}}
  %13 = arith.cmpi sgt, %1, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [128], constant_value = 8}}
  %14 = arith.constant dense<8> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [8], constant_value = <none>}}
  %15 = arith.cmpi sgt, %14, %0 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = 1}}
  %16 = arith.cmpi sgt, %14, %1 : tensor<128xi32>
  tt.return
}
```
**EN:** This block defines `cmp_all_contiguous` and exercises range generation. Expected-diagnostic annotations (remark×17) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `cmp_all_contiguous`，并覆盖 范围生成。 期望诊断标注（remark×17）用于捕获 pass 应当发出的分析备注/说明。

### Lines 398-460
```mlir
tt.func @cmp_partial_contiguous() {
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [128], constant_value = 8}}
  %1 = arith.constant dense<8> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [32], constancy = [128], constant_value = 32}}
  %3 = arith.constant dense<32> : tensor<128xi32>
  // expected-remark @below {{contiguity = [32], divisibility = [32], constancy = [1], constant_value = <none>}}
  %4 = arith.remsi %0, %3 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %5 = arith.cmpi eq, %4, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %6 = arith.cmpi ne, %4, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [8], constant_value = <none>}}
  %7 = arith.cmpi slt, %4, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %8 = arith.cmpi sle, %4, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [8], constant_value = <none>}}
  %9 = arith.cmpi sge, %4, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %10 = arith.cmpi sgt, %4, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %11 = arith.cmpi eq, %1, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %12 = arith.cmpi ne, %1, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %13 = arith.cmpi slt, %1, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [8], constant_value = <none>}}
  %14 = arith.cmpi sle, %1, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %15 = arith.cmpi sge, %1, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [8], constant_value = <none>}}
  %16 = arith.cmpi sgt, %1, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [128], constant_value = 48}}
  %17 = arith.constant dense<48> : tensor<128xi32>
  // expected-remark @below {{contiguity = [16], divisibility = [16], constancy = [1], constant_value = <none>}}
  %18 = arith.remsi %0, %17 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %19 = arith.cmpi eq, %18, %3 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %20 = arith.cmpi ne, %18, %3 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [16], constant_value = <none>}}
  %21 = arith.cmpi slt, %18, %3 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %22 = arith.cmpi sle, %18, %3 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [16], constant_value = <none>}}
  %23 = arith.cmpi sge, %18, %3 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %24 = arith.cmpi sgt, %18, %3 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %25 = arith.cmpi eq, %3, %18 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %26 = arith.cmpi ne, %3, %18 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %27 = arith.cmpi slt, %3, %18 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [16], constant_value = <none>}}
  %28 = arith.cmpi sle, %3, %18 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %29 = arith.cmpi sge, %3, %18 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [16], constant_value = <none}}
  %30 = arith.cmpi sgt, %3, %18 : tensor<128xi32>
  tt.return
}
```
**EN:** This block defines `cmp_partial_contiguous` and exercises range generation. Expected-diagnostic annotations (remark×30) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `cmp_partial_contiguous`，并覆盖 范围生成。 期望诊断标注（remark×30）用于捕获 pass 应当发出的分析备注/说明。

### Line 462
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 464-488
```mlir
tt.func @logic() {
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [128], constant_value = 64}}
  %1 = arith.constant dense<64> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [64], constant_value = <none>}}
  %2 = arith.divsi %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [128], constant_value = 8}}
  %3 = arith.constant dense<8> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [8], constant_value = <none>}}
  %4 = arith.divsi %0, %3 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %5 = arith.andi %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %6 = arith.ori %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %7 = arith.xori %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [8], constant_value = <none>}}
  %8 = arith.andi %2, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [8], constant_value = <none>}}
  %9 = arith.ori %2, %4 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [8], constant_value = <none>}}
  %10 = arith.xori %2, %4 : tensor<128xi32>
  tt.return
}
```
**EN:** This block defines `logic` and exercises range generation. Expected-diagnostic annotations (remark×11) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `logic`，并覆盖 范围生成。 期望诊断标注（remark×11）用于捕获 pass 应当发出的分析备注/说明。

### Line 490
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 492-526
```mlir
tt.func @select(%arg0 : i1, %arg1 : tensor<4xi1>) {
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [128], constant_value = 0}}
  %1 = arith.constant dense<0> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %2 = arith.cmpi eq, %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [128], constant_value = <none>}}
  %3 = arith.cmpi slt, %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [1], constant_value = 0}}
  %4 = arith.constant 0 : i1
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [128], constant_value = 0}}
  %7 = tt.splat %4 : i1 -> tensor<128xi1>
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [128], constant_value = 0}}
  %5 = arith.select %4, %3, %7 : tensor<128xi1>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %8 = arith.select %7, %3, %2 : tensor<128xi1>, tensor<128xi1>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [1, 1], constant_value = <none>}}
  %9 = tt.expand_dims %2 {axis = 1 : i32} : tensor<128xi1> -> tensor<128x1xi1>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [128, 1], constant_value = <none>}}
  %10 = tt.expand_dims %3 {axis = 1 : i32} : tensor<128xi1> -> tensor<128x1xi1>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [1, 1], constant_value = <none>}}
  %11 = arith.select %arg0, %9, %10 : tensor<128x1xi1>
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [4], constant_value = 4}}
  %cst = arith.constant dense<4> : tensor<4xi32>
  // expected-remark @below {{contiguity = [4], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %12 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = <none>}}
  %13 = arith.muli %12, %cst : tensor<4xi32>
  // expected-remark @below {{contiguity = [4], divisibility = [16], constancy = [1], constant_value = <none>}}
  %14 = tt.make_range {end = 20 : i32, start = 16 : i32} : tensor<4xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %15 = arith.select %arg1, %12, %13 : tensor<4xi1>, tensor<4xi32>
  tt.return
}
```
**EN:** This block defines `select` and exercises range generation, predicated selection. Expected-diagnostic annotations (remark×16) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `select`，并覆盖 范围生成、条件选择。 期望诊断标注（remark×16）用于捕获 pass 应当发出的分析备注/说明。

### Line 528
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 530-550
```mlir
tt.func @shift(%arg0: i32 {tt.divisibility = 4 : i32}) {
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [128], constant_value = <none>}}
  %s = tt.splat %arg0 : i32 -> tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [128], constant_value = 8}}
  %1 = arith.constant dense<8> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [128], constant_value = 4}}
  %2 = arith.constant dense<4> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [256], constancy = [1], constant_value = <none>}}
  %3 = arith.shli %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %4 = arith.shrsi %0, %2 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [128], constancy = [128], constant_value = 128}}
  %5 = arith.shli %1, %2 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [128], constant_value = <none>}}
  %6 = arith.shli %1, %s : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %7 = arith.shrsi %0, %s : tensor<128xi32>
  tt.return
}
```
**EN:** This block defines `shift` and exercises range generation. Expected-diagnostic annotations (remark×9) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `shift`，并覆盖 范围生成。 期望诊断标注（remark×9）用于捕获 pass 应当发出的分析备注/说明。

### Line 552
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 554-570
```mlir
tt.func @max_min() {
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [64], constancy = [1], constant_value = <none>}}
  %1 = tt.make_range {end = 192 : i32, start = 64 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [64], constancy = [1], constant_value = <none>}}
  %2 = arith.maxsi %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [64], constancy = [1], constant_value = <none>}}
  %3 = arith.minsi %0, %1 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [128], constant_value = 8}}
  %4 = arith.constant dense<8> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [128], constant_value = 4}}
  %5 = arith.constant dense<4> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [128], constant_value = 8}}
  %6 = arith.maxsi %4, %5 : tensor<128xi32>
  tt.return
}
```
**EN:** This block defines `max_min` and exercises range generation. Expected-diagnostic annotations (remark×7) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `max_min`，并覆盖 范围生成。 期望诊断标注（remark×7）用于捕获 pass 应当发出的分析备注/说明。

### Line 572
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 574-586
```mlir
// A complicated example with different contiguity and divisibility in lhs and rhs.
// To simplify construction of the test we just pass attributes from the arguments
tt.func @contiguity_dependent_divisibility(%arg0: tensor<8xi32> {tt.contiguity = 8 : i32, tt.divisibility = 4 : i32, tt.constancy = 1 : i32}, %arg1: tensor<8xi32> {tt.contiguity = 2 : i32, tt.divisibility = 8 : i32, tt.constancy = 1 : i32}) {
  // expected-remark @below {{contiguity = [2], divisibility = [2], constancy = [1], constant_value = <none>}}
  %0 = arith.maxsi %arg0, %arg1 : tensor<8xi32>
  // expected-remark @below {{contiguity = [2], divisibility = [2], constancy = [1], constant_value = <none>}}
  %1 = arith.minsi %arg0, %arg1 : tensor<8xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [1], constant_value = 0}}
  %2 = arith.constant 0 : i1
  // expected-remark @below {{contiguity = [2], divisibility = [2], constancy = [1], constant_value = <none>}}
  %3 = arith.select %2, %0, %1 : tensor<8xi32>
  tt.return
}
```
**EN:** This block defines `contiguity_dependent_divisibility` and exercises predicated selection. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `contiguity_dependent_divisibility`，并覆盖 条件选择。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Line 588
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 590-604
```mlir
tt.func @if(%i1 : i1) {
  // expected-remark @below {{contiguity = [1, 1], divisibility = [64, 64], constancy = [128, 32], constant_value = 64}}
  %cst_64 = arith.constant dense<64> : tensor<128x32xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [128, 32], constant_value = 1}}
  %cst_1 = arith.constant dense<1> : tensor<128x32xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [64, 64], constancy = [128, 32], constant_value = 64}}
  %a = arith.muli %cst_64, %cst_1 : tensor<128x32xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [128, 32], constant_value = <none>}}
  %ret = scf.if %i1 -> tensor<128x32xi32> {
    scf.yield %a : tensor<128x32xi32>
  } else {
    scf.yield %cst_1 : tensor<128x32xi32>
  }
  tt.return
}
```
**EN:** This block defines `if` and exercises control-flow joins. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `if`，并覆盖 控制流汇合。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Line 606
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 608-630
```mlir
tt.func @for() {
  // expected-remark @below {{contiguity = [1, 1], divisibility = [4611686018427387904, 4611686018427387904], constancy = [128, 32], constant_value = 0}}
  %a_init = arith.constant dense<0> : tensor<128x32xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [128, 32], constant_value = 1}}
  %b_init = arith.constant dense<1> : tensor<128x32xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [4, 4], constancy = [128, 32], constant_value = 4}}
  %c_init = arith.constant dense<4> : tensor<128x32xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [128], constancy = [1], constant_value = 128}}
  %ub = arith.constant 128 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [1], constant_value = 0}}
  %lb = arith.constant 0 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [1], constant_value = 16}}
  %step = arith.constant 16 : i32
  %a, %b, %c = scf.for %iv = %lb to %ub step %step iter_args(%a = %a_init, %b = %b_init, %c = %c_init) -> (tensor<128x32xi32>, tensor<128x32xi32>, tensor<128x32xi32>) : i32 {
    // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [1], constant_value = <none>}}
    %t = arith.addi %iv, %lb : i32
    // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [128, 32], constant_value = <none>}}
    // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [128, 32], constant_value = <none>}}
    // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [4, 4], constancy = [128, 32], constant_value = 4}}
    scf.yield %b, %a, %c : tensor<128x32xi32>, tensor<128x32xi32>, tensor<128x32xi32>
  }
  tt.return
}
```
**EN:** This block defines `for` and exercises loop-carried state. Expected-diagnostic annotations (remark×7) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `for`，并覆盖 循环携带状态。 期望诊断标注（remark×7）用于捕获 pass 应当发出的分析备注/说明。

### Line 632
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 634-642
```mlir
tt.func @for_dynamic(%lb: i32 {tt.divisibility = 16 : i32}, %step: i32 {tt.divisibility = 8 : i32}, %ub: i32) {
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [1], constant_value = 0}}
  %c0 = arith.constant 0 : i32
  scf.for %iv = %lb to %ub step %step : i32 {
    // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [1], constant_value = <none>}}
    %t = arith.addi %iv, %c0 : i32
  }
  tt.return
}
```
**EN:** This block defines `for_dynamic` and exercises loop-carried state. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `for_dynamic`，并覆盖 循环携带状态。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Line 644
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 646-673
```mlir
tt.func @for_if(%i1: i1, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) {
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [1], constant_value = 0}}
  %c0_i32 = arith.constant 0 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = 1}}
  %c1_i32 = arith.constant 1 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [1], constant_value = 10}}
  %c10_i32 = arith.constant 10 : i32
  // expected-remark @below {{contiguity = [1, 1], divisibility = [64, 64], constancy = [128, 64], constant_value = 64}}
  %cst = arith.constant dense<64> : tensor<128x64xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 64], constant_value = <none>}}
  %1 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>>
  %2 = scf.for %arg9 = %c0_i32 to %c10_i32 step %c1_i32 iter_args(%arg1 = %1) -> (tensor<128x64x!tt.ptr<f16>>): i32 {
    // TODO-remark(this remark is wrong, needs to be fixed) @below {{scf.if}}
    // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 64], constant_value = <none>}}
    %3 = scf.if %i1 -> (tensor<128x64x!tt.ptr<f16>>) {
      scf.yield %arg1 : tensor<128x64x!tt.ptr<f16>>
    } else {
      scf.yield %arg1 : tensor<128x64x!tt.ptr<f16>>
    }
    // TODO-remark(this remark is wrong, needs to be fixed) @below {{tt.addptr}}
    // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 64], constant_value = <none>}}
    %4 = tt.addptr %3, %cst : tensor<128x64x!tt.ptr<f16>>, tensor<128x64xi32>
    // TODO-remark(this remark is wrong, needs to be fixed) @below {{scf.for}}
    // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 64], constant_value = <none>}}
    scf.yield %1 : tensor<128x64x!tt.ptr<f16>>
  }
  tt.return
}
```
**EN:** This block defines `for_if` and exercises pointer arithmetic, loop-carried state, control-flow joins. Expected-diagnostic annotations (remark×5) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `for_if`，并覆盖 指针运算、循环携带状态、控制流汇合。 期望诊断标注（remark×5）用于捕获 pass 应当发出的分析备注/说明。

### Line 675
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 677-711
```mlir
tt.func @for_if_for(%i1: i1, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 8 : i32}) {
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [1], constant_value = 0}}
  %c0_i32 = arith.constant 0 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = 1}}
  %c1_i32 = arith.constant 1 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [2], constancy = [1], constant_value = 10}}
  %c10_i32 = arith.constant 10 : i32
  // expected-remark @below {{contiguity = [1, 1], divisibility = [64, 64], constancy = [128, 64], constant_value = 64}}
  %cst = arith.constant dense<64> : tensor<128x64xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 64], constant_value = <none>}}
  %1 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [8, 8], constancy = [128, 64], constant_value = <none>}}
  %2 = tt.splat %arg1 : !tt.ptr<f16> -> tensor<128x64x!tt.ptr<f16>>
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{scf.for}}
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [8, 8], constancy = [128, 64], constant_value = <none>}}
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{scf.if}}
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [8, 8], constancy = [128, 64], constant_value = <none>}}
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{tt.addptr}}
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [8, 8], constancy = [128, 64], constant_value = <none>}}
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{scf.for}}
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 64], constant_value = <none>}}
  %3 = scf.for %arg9 = %c0_i32 to %c10_i32 step %c1_i32 iter_args(%arg2 = %1) -> (tensor<128x64x!tt.ptr<f16>>) : i32 {
    %4 = scf.if %i1 -> (tensor<128x64x!tt.ptr<f16>>) {
      %5 = scf.for %arg10 = %c0_i32 to %c10_i32 step %c1_i32 iter_args(%arg3 = %2) -> (tensor<128x64x!tt.ptr<f16>>) : i32 {
        scf.yield %arg3 : tensor<128x64x!tt.ptr<f16>>
      }
      scf.yield %5 : tensor<128x64x!tt.ptr<f16>>
    } else {
      scf.yield %arg2 : tensor<128x64x!tt.ptr<f16>>
    }
    %6 = tt.addptr %4, %cst : tensor<128x64x!tt.ptr<f16>>, tensor<128x64xi32>
    scf.yield %1 : tensor<128x64x!tt.ptr<f16>>
  }
  tt.return
}
```
**EN:** This block defines `for_if_for` and exercises pointer arithmetic, loop-carried state, control-flow joins. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `for_if_for`，并覆盖 指针运算、循环携带状态、控制流汇合。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Line 713
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 715-764
```mlir
tt.func @permute_2d(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: i32 {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32 {tt.divisibility = 16 : i32}) {
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [128, 128], constant_value = -1}}
  %cst = arith.constant dense<true> : tensor<128x128xi1>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [1, 1], constant_value = <none>}}
  %cst_0 = arith.constant dense<0.000000e+00> : tensor<128x128xf32>
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %1 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [128, 1], divisibility = [1073741824, 1], constancy = [1, 1], constant_value = <none>}}
  %2 = tt.expand_dims %0 {axis = 1 : i32} : tensor<128xi32> -> tensor<128x1xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 1], constant_value = <none>}}
  %3 = tt.splat %arg1 : i32 -> tensor<128x1xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [1, 1], constant_value = <none>}}
  %4 = arith.muli %2, %3 : tensor<128x1xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 1], constant_value = <none>}}
  %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<128x1x!tt.ptr<f32>>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [1, 1], constant_value = <none>}}
  %6 = tt.addptr %5, %4 : tensor<128x1x!tt.ptr<f32>>, tensor<128x1xi32>
  // expected-remark @below {{contiguity = [1, 128], divisibility = [1, 1073741824], constancy = [1, 1], constant_value = <none>}}
  %7 = tt.expand_dims %1 {axis = 0 : i32}: tensor<128xi32> -> tensor<1x128xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [1, 128], constant_value = <none>}}
  %8 = tt.broadcast %6 : tensor<128x1x!tt.ptr<f32>> -> tensor<128x128x!tt.ptr<f32>>
  // expected-remark @below {{contiguity = [1, 128], divisibility = [1, 1073741824], constancy = [128, 1], constant_value = <none>}}
  %9 = tt.broadcast %7 : tensor<1x128xi32> -> tensor<128x128xi32>
  // expected-remark @below {{contiguity = [1, 128], divisibility = [4, 16], constancy = [1, 1], constant_value = <none>}}
  %10 = tt.addptr %8, %9 : tensor<128x128x!tt.ptr<f32>>, tensor<128x128xi32>
  // expected-remark @below {{contiguity = [128, 1], divisibility = [1073741824, 1], constancy = [1, 1], constant_value = <none>}}
  %11 = tt.expand_dims %0 {axis = 1 : i32}: tensor<128xi32> -> tensor<128x1xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 1], constant_value = <none>}}
  %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<128x1x!tt.ptr<f32>>
  // expected-remark @below {{contiguity = [128, 1], divisibility = [16, 4], constancy = [1, 1], constant_value = <none>}}
  %13 = tt.addptr %12, %11 : tensor<128x1x!tt.ptr<f32>>, tensor<128x1xi32>
  // expected-remark @below {{contiguity = [1, 128], divisibility = [1, 1073741824], constancy = [1, 1], constant_value = <none>}}
  %14 = tt.expand_dims %1 {axis = 0 : i32} : tensor<128xi32> -> tensor<1x128xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [1, 128], constant_value = <none>}}
  %15 = tt.splat %arg3 : i32 -> tensor<1x128xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [1, 1], constant_value = <none>}}
  %16 = arith.muli %14, %15 : tensor<1x128xi32>
  // expected-remark @below {{contiguity = [128, 1], divisibility = [16, 4], constancy = [1, 128], constant_value = <none>}}
  %17 = tt.broadcast %13 : tensor<128x1x!tt.ptr<f32>> -> tensor<128x128x!tt.ptr<f32>>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [128, 1], constant_value = <none>}}
  %18 = tt.broadcast %16 : tensor<1x128xi32> -> tensor<128x128xi32>
  // expected-remark @below {{contiguity = [128, 1], divisibility = [16, 4], constancy = [1, 1], constant_value = <none>}}
  %19 = tt.addptr %17, %18 : tensor<128x128x!tt.ptr<f32>>, tensor<128x128xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [1, 1], constant_value = <none>}}
  %20 = tt.load %10, %cst, %cst_0 : tensor<128x128x!tt.ptr<f32>>
  tt.store %19, %20, %cst : tensor<128x128x!tt.ptr<f32>>
  tt.return
}
```
**EN:** This block defines `permute_2d` and exercises tensor loads, tensor stores, pointer arithmetic, range generation. Expected-diagnostic annotations (remark×23) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `permute_2d`，并覆盖 张量加载、张量存储、指针运算、范围生成。 期望诊断标注（remark×23）用于捕获 pass 应当发出的分析备注/说明。

### Line 766
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 768-792
```mlir
tt.func @load_constancy(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: i32 {tt.divisibility = 1 : i32}) {
  // expected-remark @below {{divisibility = [16]}}
  %sixteen = arith.constant dense<16> : tensor<1024xi32>
  // expected-remark @below {{divisibility = [8]}}
  %eight = arith.constant dense<8> : tensor<1024xi32>
  // expected-remark @below {{contiguity = [1024], divisibility = [1073741824], constancy = [1]}}
  %1 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32>
  // expected-remark @below {{constancy = [16]}}
  %2 = arith.divsi %1, %sixteen : tensor<1024xi32>
  // expected-remark @below {{constancy = [1024]}}
  %3 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<1024x!tt.ptr<f32>>
  // expected-remark @below {{constancy = [1024]}}
  %4 = tt.splat %arg1 : i32 -> tensor<1024xi32>
  // expected-remark @below {{constancy = [8]}}
  %5 = arith.divsi %1, %eight : tensor<1024xi32>
  // expected-remark @below {{constancy = [8]}}
  %6 = arith.cmpi slt, %5, %4 : tensor<1024xi32>
  // expected-remark @below {{constancy = [16]}}
  %7 = tt.addptr %3, %2 : tensor<1024x!tt.ptr<f32>>, tensor<1024xi32>
  // expected-remark @below {{constancy = [16]}}
  %8 = tt.load %7 : tensor<1024x!tt.ptr<f32>>
  // expected-remark @below {{constancy = [8]}}
  %9 = tt.load %7, %6 : tensor<1024x!tt.ptr<f32>>
  tt.return
}
```
**EN:** This block defines `load_constancy` and exercises tensor loads, pointer arithmetic, range generation. Expected-diagnostic annotations (remark×11) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `load_constancy`，并覆盖 张量加载、指针运算、范围生成。 期望诊断标注（remark×11）用于捕获 pass 应当发出的分析备注/说明。

### Line 794
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 796-822
```mlir
// This is a tiny test for verifying StoreOp-related alignment, It simply store a constant to a buffer.
tt.func @store_constant_align(%addr: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %n: i32 {tt.divisibility = 16 : i32}) {
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %pid = tt.get_program_id x : i32
  // expected-remark @below {{contiguity = [1], divisibility = [128], constancy = [1], constant_value = 128}}
  %c128_i32 = arith.constant 128 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [128], constancy = [1], constant_value = <none>}}
  %1 = arith.muli %pid, %c128_i32 : i32
  // expected-remark @below {{contiguity = [128], divisibility = [1073741824], constancy = [1], constant_value = <none>}}
  %2 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
 // expected-remark @below {{contiguity = [1], divisibility = [128], constancy = [128], constant_value = <none>}}
  %3 = tt.splat %1 : i32 -> tensor<128xi32>
 // expected-remark @below {{contiguity = [128], divisibility = [128], constancy = [1], constant_value = <none>}}
  %4 = arith.addi %3, %2 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [128], constant_value = <none>}}
  %5 = tt.splat %addr : !tt.ptr<f32> -> tensor<128x!tt.ptr<f32>>
  // expected-remark @below {{contiguity = [128], divisibility = [16], constancy = [1], constant_value = <none>}}
  %6 = tt.addptr %5, %4 : tensor<128x!tt.ptr<f32>>, tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [128], constant_value = <none>}}
  %9 = tt.splat %n : i32 -> tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [16], constant_value = <none>}}
  %mask = arith.cmpi slt, %4, %9 : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %cst = arith.constant dense<0.0> : tensor<128xf32>
  tt.store %5, %cst, %mask : tensor<128x!tt.ptr<f32>>
  tt.return
}
```
**EN:** This block defines `store_constant_align` and exercises tensor stores, pointer arithmetic, range generation. Expected-diagnostic annotations (remark×11) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `store_constant_align`，并覆盖 张量存储、指针运算、范围生成。 期望诊断标注（remark×11）用于捕获 pass 应当发出的分析备注/说明。

### Line 824
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 826-850
```mlir
// This IR is dumped from vecadd test.
// Note, the hint {tt.divisibility = 16 : i32} for %n_elements affects the alignment of mask.
tt.func @vecadd_mask_align_16(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %n_elements: i32 {tt.divisibility = 16 : i32}) {
  %c64_i32 = arith.constant 64 : i32
  %0 = tt.get_program_id x : i32
  %1 = arith.muli %0, %c64_i32 : i32
  %2 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32>
  %3 = tt.splat %1 : i32 -> tensor<64xi32>
  %4 = arith.addi %3, %2 : tensor<64xi32>
  %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>>
  %6 = tt.addptr %5, %4 : tensor<64x!tt.ptr<f32>>, tensor<64xi32>
  %7 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>>
  %8 = tt.addptr %7, %4 : tensor<64x!tt.ptr<f32>>, tensor<64xi32>
  %9 = tt.splat %n_elements : i32 -> tensor<64xi32>
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{arith.cmpi slt, %{{.*}} => contiguity = [1], divisibility = [1], constancy = [16], constant_value = <none>}}
  %mask = arith.cmpi slt, %4, %9 : tensor<64xi32>
  %11 = tt.load %6, %mask : tensor<64x!tt.ptr<f32>>
  %12 = tt.load %8, %mask : tensor<64x!tt.ptr<f32>>
  %13 = arith.addf %11, %12 : tensor<64xf32>
  %14 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>>
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{tt.addptr %{{.*}} => contiguity = [64], divisibility = [16], constancy = [1], constant_value = <none>}}
  %15 = tt.addptr %14, %4 : tensor<64x!tt.ptr<f32>>, tensor<64xi32>
  tt.store %15, %13, %mask : tensor<64x!tt.ptr<f32>>
  tt.return
}
```
**EN:** This block defines `vecadd_mask_align_16` and exercises tensor loads, tensor stores, pointer arithmetic, range generation.
**CN:** 这一块定义了 `vecadd_mask_align_16`，并覆盖 张量加载、张量存储、指针运算、范围生成。

### Line 852
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 854-877
```mlir
// This IR is dumped from vecadd test.
// Note, there is no divisibility hint for %n_elements, Triton should assume its divisibility to be 1 by default.
tt.func @vecadd_mask_align_1(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %n_elements: i32) {
  %c64_i32 = arith.constant 64 : i32
  %0 = tt.get_program_id x : i32
  %1 = arith.muli %0, %c64_i32 : i32
  %2 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32>
  %3 = tt.splat %1 : i32 -> tensor<64xi32>
  %4 = arith.addi %3, %2 : tensor<64xi32>
  %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>>
  %6 = tt.addptr %5, %4 : tensor<64x!tt.ptr<f32>>, tensor<64xi32>
  %7 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>>
  %8 = tt.addptr %7, %4 : tensor<64x!tt.ptr<f32>>, tensor<64xi32>
  %9 = tt.splat %n_elements : i32 -> tensor<64xi32>
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{arith.cmpi slt, %{{.*}} => contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %10 = arith.cmpi slt, %4, %9 : tensor<64xi32>
  %11 = tt.load %6, %10 : tensor<64x!tt.ptr<f32>>
  %12 = tt.load %8, %10 : tensor<64x!tt.ptr<f32>>
  %13 = arith.addf %11, %12 : tensor<64xf32>
  %14 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>>
  %15 = tt.addptr %14, %4 : tensor<64x!tt.ptr<f32>>, tensor<64xi32>
  tt.store %15, %13, %10 : tensor<64x!tt.ptr<f32>>
  tt.return
}
```
**EN:** This block defines `vecadd_mask_align_1` and exercises tensor loads, tensor stores, pointer arithmetic, range generation.
**CN:** 这一块定义了 `vecadd_mask_align_1`，并覆盖 张量加载、张量存储、指针运算、范围生成。

### Line 879
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 881
```mlir
module {
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 883-898
```mlir
// We don't use function cloning here, so the alignment info is the gcd of all call sites.
tt.func @addptr_hints(%arg0: !tt.ptr<i32>) {
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = 1}}
  %cst1 = arith.constant 1 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = <none>}}
  %1 = tt.addptr %arg0, %cst1 : !tt.ptr<i32>, i32
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = 4}}
  %cst4 = arith.constant 4 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = <none>}}
  %2 = tt.addptr %arg0, %cst4 : !tt.ptr<i32>, i32
  // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [1], constant_value = 16}}
  %cst16 = arith.constant 16 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = <none>}}
  %3 = tt.addptr %arg0, %cst4 : !tt.ptr<i32>, i32
  tt.return
}
```
**EN:** This block defines `addptr_hints` and exercises pointer arithmetic. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `addptr_hints`，并覆盖 指针运算。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 900-903
```mlir
tt.func @kernel_div16(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32}) {
  tt.call @addptr_hints(%arg0) : (!tt.ptr<i32>) -> ()
  tt.return
}
```
**EN:** This block defines `kernel_div16`, `addptr_hints` and exercises operations such as `tt.func`, `tt.ptr`, `tt.divisibility`, `tt.call`, `tt.return`.
**CN:** 这一块定义了 `kernel_div16`, `addptr_hints`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.divisibility`、`tt.call`、`tt.return` 这样的操作。

### Lines 905-908
```mlir
tt.func @kernel_div8(%arg0: !tt.ptr<i32> {tt.divisibility = 8 : i32}) {
  tt.call @addptr_hints(%arg0) : (!tt.ptr<i32>) -> ()
  tt.return
}
```
**EN:** This block defines `kernel_div8`, `addptr_hints` and exercises operations such as `tt.func`, `tt.ptr`, `tt.divisibility`, `tt.call`, `tt.return`.
**CN:** 这一块定义了 `kernel_div8`, `addptr_hints`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.divisibility`、`tt.call`、`tt.return` 这样的操作。

### Lines 910-913
```mlir
tt.func @kernel_div4(%arg0: !tt.ptr<i32> {tt.divisibility = 4 : i32}) {
  tt.call @addptr_hints(%arg0) : (!tt.ptr<i32>) -> ()
  tt.return
}
```
**EN:** This block defines `kernel_div4`, `addptr_hints` and exercises operations such as `tt.func`, `tt.ptr`, `tt.divisibility`, `tt.call`, `tt.return`.
**CN:** 这一块定义了 `kernel_div4`, `addptr_hints`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.divisibility`、`tt.call`、`tt.return` 这样的操作。

### Line 915
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 917
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 919
```mlir
module {
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 921-928
```mlir
// We don't use function cloning here, so the alignment info is the gcd of all call sites.
tt.func @mul(%arg0: i32) {
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = 1}}
  %cst1 = arith.constant 1 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = <none>}}
  %1 = arith.muli %arg0, %cst1 : i32
  tt.return
}
```
**EN:** This block defines `mul` and exercises operations such as `tt.func`, `arith.constant`, `arith.muli`, `tt.return`. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `mul`，并覆盖 如 `tt.func`、`arith.constant`、`arith.muli`、`tt.return` 这样的操作。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Lines 930-933
```mlir
tt.func @bar(%arg0: i32) {
  tt.call @mul(%arg0) : (i32) -> ()
  tt.return
}
```
**EN:** This block defines `bar`, `mul` and exercises operations such as `tt.func`, `tt.call`, `tt.return`.
**CN:** 这一块定义了 `bar`, `mul`，并覆盖 如 `tt.func`、`tt.call`、`tt.return` 这样的操作。

### Lines 935-938
```mlir
tt.func @foo(%arg0: i32) {
  tt.call @mul(%arg0) : (i32) -> ()
  tt.return
}
```
**EN:** This block defines `foo`, `mul` and exercises operations such as `tt.func`, `tt.call`, `tt.return`.
**CN:** 这一块定义了 `foo`, `mul`，并覆盖 如 `tt.func`、`tt.call`、`tt.return` 这样的操作。

### Lines 940-952
```mlir
tt.func @call_graph(%arg0: i32) {
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = 12}}
  %cst12 = arith.constant 12 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [4], constancy = [1], constant_value = <none>}}
  %0 = arith.muli %arg0, %cst12 : i32
  tt.call @foo(%0) : (i32) -> ()
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [1], constant_value = 8}}
  %cst8 = arith.constant 8 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [1], constant_value = <none>}}
  %1 = arith.muli %arg0, %cst8 : i32
  tt.call @bar(%1) : (i32) -> ()
  tt.return
}
```
**EN:** This block defines `call_graph`, `foo`, `bar` and exercises operations such as `tt.func`, `arith.constant`, `arith.muli`, `tt.call`, `tt.return`. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `call_graph`, `foo`, `bar`，并覆盖 如 `tt.func`、`arith.constant`、`arith.muli`、`tt.call`、`tt.return` 这样的操作。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

### Line 954
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 956
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 958-982
```mlir
tt.func public @chained_for(%8: tensor<128x64x!tt.ptr<bf16>> {tt.divisibility = dense<[16, 16]> : tensor<2xi32>}) {
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [1, 1], constant_value = <none>}}
  %cst = arith.constant dense<0.000000e+00> : tensor<128x64xbf16>
  // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [1], constant_value = 16}}
  %c16_i32 = arith.constant 16 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = 1}}
  %c1_i32 = arith.constant 1 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [1], constant_value = 0}}
  %c0_i32 = arith.constant 0 : i32
  // expected-remark @below {{contiguity = [1, 1], divisibility = [64, 64], constancy = [128, 64], constant_value = 64}}
  %cst_0 = arith.constant dense<64> : tensor<128x64xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [1, 1], constant_value = <none>}}
  %9 = scf.for %arg7 = %c0_i32 to %c16_i32 step %c1_i32 iter_args(%arg8 = %8) -> (tensor<128x64x!tt.ptr<bf16>>)  : i32 {
    %11 = tt.addptr %arg8, %cst_0 : tensor<128x64x!tt.ptr<bf16>>, tensor<128x64xi32>
    scf.yield %11 : tensor<128x64x!tt.ptr<bf16>>
  }
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [1, 1], constant_value = <none>}}
  // TODO-remark(this remark is wrong, needs to be fixed) @below {{contiguity = [1, 1], divisibility = [16, 16], constancy = [1, 1], constant_value = <none>}}
  %10 = scf.for %arg7 = %c0_i32 to %c16_i32 step %c1_i32 iter_args(%arg8 = %9) -> (tensor<128x64x!tt.ptr<bf16>>)  : i32 {
    tt.store %arg8, %cst : tensor<128x64x!tt.ptr<bf16>>
    %11 = tt.addptr %arg8, %cst_0 : tensor<128x64x!tt.ptr<bf16>>, tensor<128x64xi32>
    scf.yield %11 : tensor<128x64x!tt.ptr<bf16>>
  }
  tt.return
}
```
**EN:** This block defines `chained_for` and exercises tensor stores, pointer arithmetic, loop-carried state. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `chained_for`，并覆盖 张量存储、指针运算、循环携带状态。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Line 984
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 986-992
```mlir
module {
  tt.func @int_min_does_not_underflow_in_analysis() -> i64 {
    // expected-remark @below {{divisibility = [4611686018427387904]}}
    %int_min = arith.constant -9223372036854775808 : i64
    tt.return %int_min : i64
  }
}
```
**EN:** This block defines `int_min_does_not_underflow_in_analysis` and exercises operations such as `tt.func`, `arith.constant`, `tt.return`. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `int_min_does_not_underflow_in_analysis`，并覆盖 如 `tt.func`、`arith.constant`、`tt.return` 这样的操作。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 994
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 996-1014
```mlir
tt.func @test_warp_specialize_propagation(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: i32 {tt.divisibility = 16 : i32}) {
  ttg.warp_specialize(%arg0, %arg1)
  default {
    // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [1], constant_value = <none>}}
    tt.addptr %arg0, %arg1 : !tt.ptr<f16>, i32
    ttg.warp_yield
  }
  partition0(%arg2: !tt.ptr<f16>, %arg3: i32) num_warps(1) {
    // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [1], constant_value = <none>}}
    tt.addptr %arg2, %arg3 : !tt.ptr<f16>, i32
    ttg.warp_return
  }
  partition1(%arg2: !tt.ptr<f16>, %arg3: i32) num_warps(1) {
    // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [1], constant_value = <none>}}
    tt.addptr %arg2, %arg3 : !tt.ptr<f16>, i32
    ttg.warp_return
  } : (!tt.ptr<f16>, i32) -> ()
  tt.return
}
```
**EN:** This block defines `test_warp_specialize_propagation` and exercises pointer arithmetic. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `test_warp_specialize_propagation`，并覆盖 指针运算。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Line 1016
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1018-1033
```mlir
tt.func @if_into_for_init(%i1 : i1) {
  %c0 = arith.constant 0 : i32
  %cst_64 = arith.constant 64 : i32
  %cst128 = arith.constant 128 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [1], constant_value = <none>}}
  %ret = scf.if %i1 -> i32 {
    scf.yield %cst_64 : i32
  } else {
    scf.yield %cst128 : i32
  }
  scf.for %i = %ret to %cst128 step %cst_64 : i32 {
    // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [1], constant_value = <none>}}
    %t = arith.addi %i, %c0 : i32
  }
  tt.return
}
```
**EN:** This block defines `if_into_for_init` and exercises loop-carried state, control-flow joins. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `if_into_for_init`，并覆盖 循环携带状态、控制流汇合。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Line 1035
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1037-1052
```mlir
tt.func @if_into_for_step(%i1 : i1) {
  %c0 = arith.constant 0 : i32
  %cst_64 = arith.constant 64 : i32
  %cst128 = arith.constant 128 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [1], constant_value = <none>}}
  %ret = scf.if %i1 -> i32 {
    scf.yield %cst_64 : i32
  } else {
    scf.yield %cst128 : i32
  }
  scf.for %i = %c0 to %cst128 step %ret : i32 {
    // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [1], constant_value = <none>}}
    %t = arith.addi %i, %c0 : i32
  }
  tt.return
}
```
**EN:** This block defines `if_into_for_step` and exercises loop-carried state, control-flow joins. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `if_into_for_step`，并覆盖 循环携带状态、控制流汇合。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Line 1054
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1056-1063
```mlir
tt.func @op_annotation(%i32 : i32) {
  %c0 = arith.constant 0 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [4096], constancy = [1], constant_value = <none>}}
  %ret0 = arith.addi %c0, %i32 { tt.divisibility = 4096 : i32 } : i32
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1024, 1024], constancy = [128, 64], constant_value = <none>}}
  %ret1 = tt.splat %ret0 { tt.divisibility = dense<[1024, 1024]> : tensor<2xi32> } : i32 -> tensor<128x64xi32>
  tt.return
}
```
**EN:** This block defines `op_annotation` and exercises operations such as `tt.func`, `arith.constant`, `arith.addi`, `tt.divisibility`, `tt.splat`. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `op_annotation`，并覆盖 如 `tt.func`、`arith.constant`、`arith.addi`、`tt.divisibility`、`tt.splat` 这样的操作。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Line 1065
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1067-1073
```mlir
tt.func public @trans_4d_tensor_kernel(%arg0: tensor<32x32x32x32xi32> {tt.contiguity = dense<[32, 1, 1, 1]> : tensor<4xi32>, tt.divisibility = dense<[16, 1, 1, 1]> : tensor<4xi32>}) attributes {noinline = false} {
  // expected-remark @below {{contiguity = [1, 1, 1, 32], divisibility = [1, 1, 1, 16], constancy = [1, 1, 1, 1], constant_value = <none>}}
  %101 = tt.trans %arg0 {order = array<i32: 3, 2, 1, 0>} : tensor<32x32x32x32xi32> -> tensor<32x32x32x32xi32>
  // expected-remark @below {{contiguity = [1, 32, 1, 1], divisibility = [1, 16, 1, 1], constancy = [1, 1, 1, 1], constant_value = <none>}}
  %102 = tt.trans %arg0 {order = array<i32: 1, 0, 2, 3>} : tensor<32x32x32x32xi32> -> tensor<32x32x32x32xi32>
  tt.return
}
```
**EN:** This block defines `trans_4d_tensor_kernel` and exercises operations such as `tt.func`, `tt.contiguity`, `tt.divisibility`, `tt.trans`, `tt.return`. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `trans_4d_tensor_kernel`，并覆盖 如 `tt.func`、`tt.contiguity`、`tt.divisibility`、`tt.trans`、`tt.return` 这样的操作。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Line 1075
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1077-1083
```mlir
tt.func @unrealized_conversion_cast(%arg0: tensor<128x128xi32> {tt.contiguity = dense<[16, 32]> : tensor<2xi32>}) {
  // Case 1: AxisInfo is propagated through a sequence of
  // unrealized_conversion_cast ops.
  // expected-remark @below {{contiguity = [16, 32], divisibility = [1, 1], constancy = [1, 1], constant_value = <none>}}
  %0 = builtin.unrealized_conversion_cast %arg0 : tensor<128x128xi32> to !llvm.struct<(i32, i32, i32, i32)>
  // expected-remark @below {{contiguity = [16, 32], divisibility = [1, 1], constancy = [1, 1], constant_value = <none>}}
  %1 = builtin.unrealized_conversion_cast %0 : !llvm.struct<(i32, i32, i32, i32)> to tensor<128x128xi32>
```
**EN:** This block defines `unrealized_conversion_cast` and exercises operations such as `tt.func`, `tt.contiguity`, `builtin.unrealized_conversion_cast`, `llvm.struct`. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `unrealized_conversion_cast`，并覆盖 如 `tt.func`、`tt.contiguity`、`builtin.unrealized_conversion_cast`、`llvm.struct` 这样的操作。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Lines 1085-1094
```mlir
  // Case 2: AxisInfo is falling back to the pessimistic state if the
  // propagated AxisInfo would be invalid.
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %2 = llvm.mlir.undef : !llvm.struct<(i32, i32, i32, i32)>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [1, 1], constant_value = <none>}}
  %3 = builtin.unrealized_conversion_cast %2 : !llvm.struct<(i32, i32, i32, i32)> to tensor<128x128xi32>
  // expected-remark @below {{contiguity = [1, 1], divisibility = [1, 1], constancy = [1, 1], constant_value = <none>}}
  %4 = tt.trans %3 {order = array<i32: 1, 0>} : tensor<128x128xi32> -> tensor<128x128xi32>
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises operations such as `llvm.mlir.undef`, `llvm.struct`, `builtin.unrealized_conversion_cast`, `tt.trans`, `tt.return`. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `llvm.mlir.undef`、`llvm.struct`、`builtin.unrealized_conversion_cast`、`tt.trans`、`tt.return` 这样的操作。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Line 1096
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1098-1102
```mlir
// Axis analysis does not support multi-dimensional function arguments. Make
// sure that we don't crash.
tt.func @callee(%arg0: tensor<128x1xi32>) {
  tt.return
}
```
**EN:** This block defines `callee` and exercises operations such as `tt.func`, `tt.return`.
**CN:** 这一块定义了 `callee`，并覆盖 如 `tt.func`、`tt.return` 这样的操作。

### Lines 1104-1110
```mlir
tt.func @caller() {
  %0 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  // expected-remark @below {{contiguity = [128, 1], divisibility = [1073741824, 1], constancy = [1, 1], constant_value = <none>}}
  %1 = tt.expand_dims %0 {axis = 1: i32} : tensor<128xi32> -> tensor<128x1xi32>
  tt.call @callee(%1) : (tensor<128x1xi32>) -> ()
  tt.return
}
```
**EN:** This block defines `caller`, `callee` and exercises range generation. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `caller`, `callee`，并覆盖 范围生成。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 1112
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1114-1120
```mlir
tt.func @mul_zero_constancy() {
  %range = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32>
  %zeros = arith.constant dense<0> : tensor<128xi32>
  // expected-remark @below {{constancy = [128]}}
  %product = arith.muli %zeros, %range : tensor<128xi32>
  tt.return
}
```
**EN:** This block defines `mul_zero_constancy` and exercises range generation. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `mul_zero_constancy`，并覆盖 范围生成。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 1122
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1124-1130
```mlir
tt.func @max_constancy() {
  %c5 = arith.constant dense<5> : tensor<4xi32>
  %c7 = arith.constant dense<7> : tensor<4xi32>
  // expected-remark @below {{constancy = [4], constant_value = 7}}
  %max = arith.maxsi %c5, %c7 : tensor<4xi32>
  tt.return
}
```
**EN:** This block defines `max_constancy` and exercises operations such as `tt.func`, `arith.constant`, `arith.maxsi`, `tt.return`. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `max_constancy`，并覆盖 如 `tt.func`、`arith.constant`、`arith.maxsi`、`tt.return` 这样的操作。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 1132
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1134-1145
```mlir
tt.func @select_same_value_constancy() {
  %range = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32>
  %two = arith.constant dense<2> : tensor<4xi32>
  %mod = arith.remsi %range, %two : tensor<4xi32>
  %zero = arith.constant dense<0> : tensor<4xi32>
  %cond = arith.cmpi ne, %mod, %zero : tensor<4xi32>
  %lhs = arith.constant dense<42> : tensor<4xi32>
  %rhs = arith.constant dense<42> : tensor<4xi32>
  // expected-remark @below {{constancy = [4], constant_value = 42}}
  %sel = arith.select %cond, %lhs, %rhs : tensor<4xi1>, tensor<4xi32>
  tt.return
}
```
**EN:** This block defines `select_same_value_constancy` and exercises range generation, predicated selection. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `select_same_value_constancy`，并覆盖 范围生成、条件选择。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 1147
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1149-1156
```mlir
tt.func @cmp_after_max_constancy() {
  %c5 = arith.constant dense<5> : tensor<4xi32>
  %c7 = arith.constant dense<7> : tensor<4xi32>
  %max = arith.maxsi %c5, %c7 : tensor<4xi32>
  // expected-remark @below {{constancy = [4], constant_value = 1}}
  %cmp = arith.cmpi sgt, %max, %c5 : tensor<4xi32>
  tt.return
}
```
**EN:** This block defines `cmp_after_max_constancy` and exercises operations such as `tt.func`, `arith.constant`, `arith.maxsi`, `arith.cmpi`, `tt.return`. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `cmp_after_max_constancy`，并覆盖 如 `tt.func`、`arith.constant`、`arith.maxsi`、`arith.cmpi`、`tt.return` 这样的操作。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 1158
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1160-1172
```mlir
tt.func public @test_inductor_for() {
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [1], constant_value = 64}}
  %c64_i32 = arith.constant 64 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [1], constant_value = 0}}
  %c0_i64 = arith.constant 0 : i64
  // expected-remark @below {{contiguity = [1], divisibility = [4611686018427387904], constancy = [1], constant_value = 0}}
  %c0_i32 = arith.constant 0 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = 1}}
  %c1_i32 = arith.constant 1 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [1], constant_value = 64}}
  %c64_i64 = arith.constant 64 : i64
  // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
  %0 = arith.cmpi slt, %c0_i32, %c1_i32 : i32
```
**EN:** This block defines `test_inductor_for` and exercises operations such as `tt.func`, `arith.constant`, `arith.cmpi`. Expected-diagnostic annotations (remark×6) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `test_inductor_for`，并覆盖 如 `tt.func`、`arith.constant`、`arith.cmpi` 这样的操作。 期望诊断标注（remark×6）用于捕获 pass 应当发出的分析备注/说明。

### Lines 1174-1179
```mlir
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [1], constant_value = 64}}
  %1:2 = scf.if %0 -> (i32, i32) {
    scf.yield %c0_i32, %c64_i32 : i32, i32
  } else {
    scf.yield %c1_i32, %c64_i32 : i32, i32
  }
```
**EN:** This block defines the test function(s) and exercises control-flow joins. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 the test function(s)，并覆盖 控制流汇合。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Lines 1181-1188
```mlir
  // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [1], constant_value = <none>}}
  %2 = scf.for %arg0 = %1#0 to %1#1 step %c64_i32 iter_args(%arg1 = %c0_i64) -> (i64)  : i32 {
    // expected-remark @below {{contiguity = [1], divisibility = [64], constancy = [1], constant_value = <none>}}
    %3 = arith.addi %arg1, %c64_i64 : i64
    scf.yield %3 : i64
  }
  tt.return
}
```
**EN:** This block defines the test function(s) and exercises loop-carried state. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 the test function(s)，并覆盖 循环携带状态。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Line 1190
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1192-1203
```mlir
// Verify that if an operation is statically determined to be dead, we fall back
// to assigning it a pessimistic value, rather than skipping it entirely.
tt.func @dead_op_pessimistic() {
  %c5 = arith.constant dense<5> : tensor<4xi32>
  %c7 = arith.constant dense<7> : tensor<4xi32>
  %false = arith.constant false
  scf.if %false {
    // expected-remark @below {{contiguity = [1], divisibility = [1], constancy = [1], constant_value = <none>}}
    %add = arith.addi %c5, %c7 : tensor<4xi32>
  }
  tt.return
}
```
**EN:** This block defines `dead_op_pessimistic` and exercises control-flow joins. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `dead_op_pessimistic`，并覆盖 控制流汇合。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Line 1205
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1207-1217
```mlir
tt.func @negative_constants() {
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [1], constant_value = -8}}
  %neg8_scalar = arith.constant -8 : i32
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [128], constant_value = -8}}
  %neg8_dense = arith.constant dense<-8> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [16], constancy = [128], constant_value = 16}}
  %sixteen = arith.constant dense<16> : tensor<128xi32>
  // expected-remark @below {{contiguity = [1], divisibility = [8], constancy = [128], constant_value = 8}}
  %sum = arith.addi %neg8_dense, %sixteen : tensor<128xi32>
  tt.return
}
```
**EN:** This block defines `negative_constants` and exercises operations such as `tt.func`, `arith.constant`, `arith.addi`, `tt.return`. Expected-diagnostic annotations (remark×4) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `negative_constants`，并覆盖 如 `tt.func`、`arith.constant`、`arith.addi`、`tt.return` 这样的操作。 期望诊断标注（remark×4）用于捕获 pass 应当发出的分析备注/说明。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's analysis coverage and focuses on alignment and axis information.  
  **CN:** 该文件属于 Triton 的 分析 测试覆盖，关注点是 对齐与轴信息。
- **EN:** The `RUN` pipeline drives tools/passes such as `-test-print-alignment`, `-verify-diagnostics=only-expected`.  
  **CN:** `RUN` 流水线会驱动 `-test-print-alignment`, `-verify-diagnostics=only-expected` 等工具/pass。
- **EN:** The IR mixes `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `scf` (structured control flow), `ttg` (TritonGPU ops/layouts), `llvm` (LLVM dialect IR).  
  **CN:** 该 IR 混合使用了 `tt`（Triton 核心操作）、`arith`（标量/张量算术）、`scf`（结构化控制流）、`ttg`（TritonGPU 操作/布局）、`llvm`（LLVM 方言 IR）。
- **EN:** Expected remarks/notes are used to pin down the exact analysis output produced by the pass.  
  **CN:** 这里的期望备注/说明用于固定 pass 产生的精确分析输出。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`.  
  **CN:** 外部测试工具：`triton-opt`。
- **EN:** IR dialect dependencies: `tt`, `arith`, `scf`, `ttg`, `llvm`.  
  **CN:** IR 方言依赖：`tt`、`arith`、`scf`、`ttg`、`llvm`。
- **EN:** This test relies on the alignment/axis-info test pass declared in `test/include/Analysis/TestAxisInfo.h` and implemented in `test/lib/Analysis/TestAxisInfo.cpp`.  
  **CN:** 该测试依赖 `test/include/Analysis/TestAxisInfo.h` 中声明、`test/lib/Analysis/TestAxisInfo.cpp` 中实现的对齐/轴信息测试 pass。
