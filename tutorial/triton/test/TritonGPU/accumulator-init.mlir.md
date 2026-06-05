# accumulator-init.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/TritonGPU/accumulator-init.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises warp-level behavior in Triton's TritonGPU transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton TritonGPU 变换中的warp 级行为相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file -tritongpu-optimize-accumulator-init | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×81, CHECK-DAG×25, CHECK-LABEL×15, CHECK-NOT×2; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×81, CHECK-DAG×25, CHECK-LABEL×15, CHECK-NOT×2；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that warp-level behavior produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 warp 级行为 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file -tritongpu-optimize-accumulator-init | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file -tritongpu-optimize-accumulator-init | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file -tritongpu-optimize-accumulator-init | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-10
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 16, 16]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 12-26
```mlir
// CHECK-LABEL: @constant_init
// CHECK-DAG: %[[FALSE:.+]] = arith.constant false
// CHECK: ttng.warp_group_dot {{.*}}, {{.*}}, {{.*}}, %[[FALSE]]
  tt.func @constant_init(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc = ttng.warp_group_dot %A, %B, %cst_2 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      scf.yield %acc: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `constant_init` and exercises loop-carried state. Embedded check comments (CHECK×1, CHECK-DAG×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `constant_init`，并覆盖 循环携带状态。 其中嵌入的检查注释（CHECK×1, CHECK-DAG×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 28-42
```mlir
// CHECK-LABEL: @constant_init_integer
// CHECK-DAG: %[[FALSE:.+]] = arith.constant false
// CHECK: ttng.warp_group_dot {{.*}}, {{.*}}, {{.*}}, %[[FALSE]]
  tt.func @constant_init_integer(%A: !ttg.memdesc<128x64xi8, #shared, #smem>, %B: !ttg.memdesc<64x16xi8, #shared1, #smem>, %arg0: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xi32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0> : tensor<128x16xi32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xi32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc = ttng.warp_group_dot %A, %B, %cst_2 : !ttg.memdesc<128x64xi8, #shared, #smem> * !ttg.memdesc<64x16xi8, #shared1, #smem> -> tensor<128x16xi32, #mma1>
      scf.yield %acc: tensor<128x16xi32, #mma1>
    }
    tt.return %17 : tensor<128x16xi32, #mma1>
  }
```
**EN:** This block defines `constant_init_integer` and exercises loop-carried state. Embedded check comments (CHECK×1, CHECK-DAG×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `constant_init_integer`，并覆盖 循环携带状态。 其中嵌入的检查注释（CHECK×1, CHECK-DAG×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 44-73
```mlir
// CHECK-LABEL: @if_after_mma
// CHECK-DAG: %[[C0_TENSOR:.+]] = arith.constant dense<0.000000e+00>
// CHECK-DAG: %[[TRUE:.+]] = arith.constant true
// CHECK-DAG: %[[FALSE:.+]] = arith.constant false
// CHECK: scf.for {{.*}} iter_args(%[[ACC:.+]] = %[[C0_TENSOR]], %[[USE_ACC:.+]] = %[[FALSE]])
// CHECK: %[[CND:.+]] = arith.cmpi
// CHECK: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, %[[ACC]], %[[USE_ACC]]
// CHECK: %[[USE_ACC_NEXT:.*]] = arith.select %[[CND]], %[[FALSE]], %[[TRUE]]
// CHECK: scf.if %[[CND]]
// CHECK: scf.yield %[[ACC_NEXT]]
// CHECK: else
// CHECK: scf.yield %[[ACC_NEXT]]
// CHECK: scf.yield {{.*}}, %[[USE_ACC_NEXT]]
  tt.func @if_after_mma(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc = ttng.warp_group_dot %A, %B, %arg4 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      %acc_ = scf.if %cnd -> (tensor<128x16xf32, #mma1>) {
        scf.yield %cst_2 : tensor<128x16xf32, #mma1>
      } else {
        scf.yield %acc : tensor<128x16xf32, #mma1>
      }
      scf.yield %acc_: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `if_after_mma` and exercises loop-carried state, control-flow joins, predicated selection. Embedded check comments (CHECK×9, CHECK-DAG×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `if_after_mma`，并覆盖 循环携带状态、控制流汇合、条件选择。 其中嵌入的检查注释（CHECK×9, CHECK-DAG×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 75-102
```mlir
// CHECK-LABEL: @if_after_mma_invert
// CHECK-DAG: %[[C0_TENSOR:.+]] = arith.constant dense<0.000000e+00>
// CHECK-DAG: %[[FALSE:.+]] = arith.constant false
// CHECK: scf.for {{.*}} iter_args(%[[ACC:.+]] = %[[C0_TENSOR]], %[[USE_ACC:.+]] = %[[FALSE]])
// CHECK: %[[CND:.+]] = arith.cmpi
// CHECK: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, %[[ACC]], %[[USE_ACC]]
// CHECK: scf.if %[[CND]]
// CHECK: scf.yield %[[ACC_NEXT]]
// CHECK: else
// CHECK: scf.yield %[[ACC_NEXT]]
// CHECK: scf.yield {{.*}}, %[[CND]]
  tt.func @if_after_mma_invert(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc = ttng.warp_group_dot %A, %B, %arg4 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      %acc_ = scf.if %cnd -> (tensor<128x16xf32, #mma1>) {
        scf.yield %acc : tensor<128x16xf32, #mma1>
      } else {
        scf.yield %cst_2 : tensor<128x16xf32, #mma1>
      }
      scf.yield %acc_: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `if_after_mma_invert` and exercises loop-carried state, control-flow joins. Embedded check comments (CHECK×8, CHECK-DAG×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `if_after_mma_invert`，并覆盖 循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×8, CHECK-DAG×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 104-133
```mlir
// CHECK-LABEL: @if_before_mma
// CHECK-DAG: %[[C0_TENSOR:.+]] = arith.constant dense<0.000000e+00>
// CHECK-DAG: %[[TRUE:.+]] = arith.constant true
// CHECK-DAG: %[[FALSE:.+]] = arith.constant false
// CHECK: scf.for {{.*}} iter_args(%[[ACC:.+]] = %[[C0_TENSOR]], %[[USE_ACC:.+]] = %[[TRUE]])
// CHECK: %[[CND:.+]] = arith.cmpi
// CHECK: %[[USE_ACC_NEXT:.*]] = arith.select %[[CND]], %[[FALSE]], %[[USE_ACC]]
// CHECK: %[[ACC_CND:.+]] = scf.if %[[CND]]
// CHECK: scf.yield %[[ACC]]
// CHECK: else
// CHECK: scf.yield %[[ACC]]
// CHECK: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, %[[ACC_CND]], %[[USE_ACC_NEXT]]
// CHECK: scf.yield {{.*}}, %[[TRUE]]
  tt.func @if_before_mma(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc_ = scf.if %cnd -> (tensor<128x16xf32, #mma1>) {
        scf.yield %cst_2 : tensor<128x16xf32, #mma1>
      } else {
        scf.yield %arg4 : tensor<128x16xf32, #mma1>
      }
      %acc = ttng.warp_group_dot %A, %B, %acc_ : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      scf.yield %acc: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `if_before_mma` and exercises loop-carried state, control-flow joins, predicated selection. Embedded check comments (CHECK×9, CHECK-DAG×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `if_before_mma`，并覆盖 循环携带状态、控制流汇合、条件选择。 其中嵌入的检查注释（CHECK×9, CHECK-DAG×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 135-164
```mlir
// CHECK-LABEL: @if_before_mma_invert
// CHECK-DAG: %[[C0_TENSOR:.+]] = arith.constant dense<0.000000e+00>
// CHECK-DAG: %[[TRUE:.+]] = arith.constant true
// CHECK-DAG: %[[FALSE:.+]] = arith.constant false
// CHECK: scf.for {{.*}} iter_args(%[[ACC:.+]] = %[[C0_TENSOR]], %[[USE_ACC:.+]] = %[[TRUE]])
// CHECK: %[[CND:.+]] = arith.cmpi
// CHECK: %[[USE_ACC_NEXT:.*]] = arith.select %[[CND]], %[[USE_ACC]], %[[FALSE]]
// CHECK: %[[ACC_CND:.+]] = scf.if %[[CND]]
// CHECK: scf.yield %[[ACC]]
// CHECK: else
// CHECK: scf.yield %[[ACC]]
// CHECK: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, %[[ACC_CND]], %[[USE_ACC_NEXT]]
// CHECK: scf.yield {{.*}}, %[[TRUE]]
  tt.func @if_before_mma_invert(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc_ = scf.if %cnd -> (tensor<128x16xf32, #mma1>) {
        scf.yield %arg4 : tensor<128x16xf32, #mma1>
      } else {
        scf.yield %cst_2 : tensor<128x16xf32, #mma1>
      }
      %acc = ttng.warp_group_dot %A, %B, %acc_ : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      scf.yield %acc: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `if_before_mma_invert` and exercises loop-carried state, control-flow joins, predicated selection. Embedded check comments (CHECK×9, CHECK-DAG×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `if_before_mma_invert`，并覆盖 循环携带状态、控制流汇合、条件选择。 其中嵌入的检查注释（CHECK×9, CHECK-DAG×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 166-187
```mlir
// CHECK-LABEL: @sel_after_mma
// CHECK-DAG: %[[C0_TENSOR:.+]] = arith.constant dense<0.000000e+00>
// CHECK-DAG: %[[TRUE:.+]] = arith.constant true
// CHECK-DAG: %[[FALSE:.+]] = arith.constant false
// CHECK: scf.for {{.*}} iter_args(%[[ACC:.+]] = %[[C0_TENSOR]], %[[USE_ACC:.+]] = %[[FALSE]])
// CHECK: %[[CND:.+]] = arith.cmpi
// CHECK: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, %[[ACC]], %[[USE_ACC]]
// CHECK: %[[USE_ACC_NEXT:.*]] = arith.select %[[CND]], %[[FALSE]], %[[TRUE]]
// CHECK: scf.yield {{.*}}, %[[USE_ACC_NEXT]]
  tt.func @sel_after_mma(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc = ttng.warp_group_dot %A, %B, %arg4 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      %acc_ = arith.select %cnd, %cst_2, %acc : tensor<128x16xf32, #mma1>
      scf.yield %acc_: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `sel_after_mma` and exercises loop-carried state, predicated selection. Embedded check comments (CHECK×5, CHECK-DAG×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `sel_after_mma`，并覆盖 循环携带状态、条件选择。 其中嵌入的检查注释（CHECK×5, CHECK-DAG×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 189-210
```mlir
// CHECK-LABEL: @sel_before_mma
// CHECK-DAG: %[[C0_TENSOR:.+]] = arith.constant dense<0.000000e+00>
// CHECK-DAG: %[[TRUE:.+]] = arith.constant true
// CHECK-DAG: %[[FALSE:.+]] = arith.constant false
// CHECK: scf.for {{.*}} iter_args(%[[ACC:.+]] = %[[C0_TENSOR]], %[[USE_ACC:.+]] = %[[TRUE]])
// CHECK: %[[CND:.+]] = arith.cmpi
// CHECK: %[[USE_ACC_NEXT:.*]] = arith.select %[[CND]], %[[FALSE]], %[[USE_ACC]]
// CHECK: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, %[[ACC]], %[[USE_ACC_NEXT]]
// CHECK: scf.yield {{.*}}, %[[TRUE]]
  tt.func @sel_before_mma(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc_ = arith.select %cnd, %cst_2, %arg4 : tensor<128x16xf32, #mma1>
      %acc = ttng.warp_group_dot %A, %B, %acc_ : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      scf.yield %acc: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `sel_before_mma` and exercises loop-carried state, predicated selection. Embedded check comments (CHECK×5, CHECK-DAG×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `sel_before_mma`，并覆盖 循环携带状态、条件选择。 其中嵌入的检查注释（CHECK×5, CHECK-DAG×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 213
```mlir
// Check that we look only at the zeroing directly preceding the mma
```
**EN:** This comment block provides context for the surrounding test logic: `// Check that we look only at the zeroing directly preceding the mma`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Check that we look only at the zeroing directly preceding the mma`。

### Lines 215-253
```mlir
// CHECK-LABEL: @if_before_and_after_mma
// CHECK-DAG: %[[C0_TENSOR:.+]] = arith.constant dense<0.000000e+00>
// CHECK-DAG: %[[TRUE:.+]] = arith.constant true
// CHECK-DAG: %[[FALSE:.+]] = arith.constant false
// CHECK: scf.for {{.*}} iter_args(%[[ACC:.+]] = %[[C0_TENSOR]], %[[USE_ACC:.+]] = %[[TRUE]])
// CHECK: %[[CND:.+]] = arith.cmpi
// CHECK: %[[USE_ACC_NEXT:.*]] = arith.select %[[CND]], %[[FALSE]], %[[USE_ACC]]
// CHECK: %[[ACC_CND:.+]] = scf.if %[[CND]]
// CHECK: scf.yield %[[ACC]]
// CHECK: else
// CHECK: scf.yield %[[ACC]]
// CHECK: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, %[[ACC_CND]], %[[USE_ACC_NEXT]]
// CHECK: scf.if %[[CND]]
// CHECK: scf.yield %[[C0_TENSOR]]
// CHECK: else
// CHECK: scf.yield %[[ACC_NEXT]]
// CHECK: scf.yield {{.*}}, %[[TRUE]]
  tt.func @if_before_and_after_mma(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc_0 = scf.if %cnd -> (tensor<128x16xf32, #mma1>) {
        scf.yield %cst_2 : tensor<128x16xf32, #mma1>
      } else {
        scf.yield %arg4 : tensor<128x16xf32, #mma1>
      }
      %acc = ttng.warp_group_dot %A, %B, %acc_0 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      %acc_1 = scf.if %cnd -> (tensor<128x16xf32, #mma1>) {
        scf.yield %cst_2 : tensor<128x16xf32, #mma1>
      } else {
        scf.yield %acc : tensor<128x16xf32, #mma1>
      }
      scf.yield %acc_1: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `if_before_and_after_mma` and exercises loop-carried state, control-flow joins, predicated selection. Embedded check comments (CHECK×13, CHECK-DAG×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `if_before_and_after_mma`，并覆盖 循环携带状态、控制流汇合、条件选择。 其中嵌入的检查注释（CHECK×13, CHECK-DAG×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 255-293
```mlir
// CHECK-LABEL: @two_ifs_after_mma
// CHECK-DAG: %[[C0_TENSOR:.+]] = arith.constant dense<0.000000e+00>
// CHECK-DAG: %[[TRUE:.+]] = arith.constant true
// CHECK-DAG: %[[FALSE:.+]] = arith.constant false
// CHECK: scf.for {{.*}} iter_args(%[[ACC:.+]] = %[[C0_TENSOR]], %[[USE_ACC:.+]] = %[[FALSE]])
// CHECK: %[[CND:.+]] = arith.cmpi
// CHECK: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, %[[ACC]], %[[USE_ACC]]
// CHECK: %[[ACC_CND:.+]] = scf.if %[[CND]]
// CHECK: scf.yield %[[C0_TENSOR]]
// CHECK: else
// CHECK: scf.yield %[[ACC_NEXT]]
// CHECK: %[[USE_ACC_NEXT:.*]] = arith.select %[[CND]], %[[FALSE]], %[[TRUE]]
// CHECK: scf.if %[[CND]]
// CHECK: scf.yield %[[ACC_CND]]
// CHECK: else
// CHECK: scf.yield %[[ACC_CND]]
// CHECK: scf.yield {{.*}}, %[[USE_ACC_NEXT]]
  tt.func @two_ifs_after_mma(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc = ttng.warp_group_dot %A, %B, %arg4 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      %acc_0 = scf.if %cnd -> (tensor<128x16xf32, #mma1>) {
        scf.yield %cst_2 : tensor<128x16xf32, #mma1>
      } else {
        scf.yield %acc : tensor<128x16xf32, #mma1>
      }
      %acc_1 = scf.if %cnd -> (tensor<128x16xf32, #mma1>) {
        scf.yield %cst_2 : tensor<128x16xf32, #mma1>
      } else {
        scf.yield %acc_0 : tensor<128x16xf32, #mma1>
      }
      scf.yield %acc_1: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `two_ifs_after_mma` and exercises loop-carried state, control-flow joins, predicated selection. Embedded check comments (CHECK×13, CHECK-DAG×3, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `two_ifs_after_mma`，并覆盖 循环携带状态、控制流汇合、条件选择。 其中嵌入的检查注释（CHECK×13, CHECK-DAG×3, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 295-312
```mlir
  // CHECK-LABEL: @zero_init_dist_2
  tt.func @zero_init_dist_2(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    // CHECK: %[[CST:.*]] = arith.constant dense<0.000000e+00>
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    // CHECK: scf.for {{.*}} = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg{{[1-9]+}} = %{{.*}}, %[[ACC:.*]] = %[[CST]], %[[INIT_FLAG:.*]] = %false)
    %17:2 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2, %arg5 = %cst_2) -> (tensor<128x16xf32, #mma1>, tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      // CHECK: %2 = ttng.warp_group_dot {{.*}}, {{.*}}, %[[ACC]], %[[INIT_FLAG]]
      %acc = ttng.warp_group_dot %A, %B, %arg5 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      %acc_ = arith.select %cnd, %cst_2, %acc : tensor<128x16xf32, #mma1>
      // CHECK: scf.yield {{.*}}, {{.*}}, %true
      scf.yield %acc_, %arg4: tensor<128x16xf32, #mma1>, tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `zero_init_dist_2` and exercises loop-carried state, predicated selection. Embedded check comments (CHECK×4, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `zero_init_dist_2`，并覆盖 循环携带状态、条件选择。 其中嵌入的检查注释（CHECK×4, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 314-335
```mlir
// CHECK-LABEL: @if_defines_alternative
// CHECK: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, {{.*}}, %arg{{.*}} : !ttg.memdesc
  tt.func @if_defines_alternative(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %cst_3 = arith.constant dense<1.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc = ttng.warp_group_dot %A, %B, %arg4 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      %acc_ = scf.if %cnd -> (tensor<128x16xf32, #mma1>) {
        scf.yield %cst_2 : tensor<128x16xf32, #mma1>
      } else {
        %acc_alt = arith.addf %acc, %cst_3 : tensor<128x16xf32, #mma1>
        scf.yield %acc_alt : tensor<128x16xf32, #mma1>
      }
      // CHECK: scf.yield {{.*}}, %true
      scf.yield %acc_: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `if_defines_alternative` and exercises loop-carried state, control-flow joins. Embedded check comments (CHECK×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `if_defines_alternative`，并覆盖 循环携带状态、控制流汇合。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 337-352
```mlir
// CHECK-LABEL: @non_cond_override
// CHECK: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, {{.*}}, %arg{{.*}} : !ttg.memdesc
  tt.func @non_cond_override(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %cst_3 = arith.constant dense<1.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %acc = ttng.warp_group_dot %A, %B, %arg4 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      %acc_ = arith.addf %acc, %cst_3 : tensor<128x16xf32, #mma1>
      // CHECK: scf.yield {{.*}}, %true
      scf.yield %acc_: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `non_cond_override` and exercises loop-carried state. Embedded check comments (CHECK×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `non_cond_override`，并覆盖 循环携带状态。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 355
```mlir
// Check that we bail out in unsupported cases
```
**EN:** This comment block provides context for the surrounding test logic: `// Check that we bail out in unsupported cases`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Check that we bail out in unsupported cases`。

### Lines 357-371
```mlir
// CHECK-LABEL: @non_zero_init
// CHECK-NOT: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, {{.*}}, {{.*}} : !ttg.memdesc
  tt.func @non_zero_init(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %ext: i32, %inc: tensor<64x16xi32, #blocked> {tt.divisibility = 16 : i32}) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<1.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %cnd = arith.cmpi slt, %arg3, %ext : i32
      %acc = ttng.warp_group_dot %A, %B, %arg4 : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      %acc_ = arith.select %cnd, %cst_2, %acc : tensor<128x16xf32, #mma1>
      scf.yield %acc_: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
```
**EN:** This block defines `non_zero_init` and exercises loop-carried state, predicated selection. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `non_zero_init`，并覆盖 循环携带状态、条件选择。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 373-388
```mlir
// If the condition is a tensor skip the optimization.
// CHECK-LABEL: @negative_sel_tensor
// CHECK-NOT: %[[ACC_NEXT:.+]] = ttng.warp_group_dot {{.*}}, {{.*}}, {{.*}}, {{.*}} : !ttg.memdesc
  tt.func @negative_sel_tensor(%A: !ttg.memdesc<128x64xf16, #shared, #smem>, %B: !ttg.memdesc<64x16xf16, #shared1, #smem>, %cnd: tensor<128x16xi1, #mma1>) -> tensor<128x16xf32, #mma1> {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<0.000000e+00> : tensor<128x16xf32, #mma1>
    %c1_i32 = arith.constant 1 : i32
    %c8_i32 = arith.constant 8 : i32
    %17 = scf.for %arg3 = %c0_i32 to %c8_i32 step %c1_i32 iter_args(%arg4 = %cst_2) -> (tensor<128x16xf32, #mma1>)  : i32 {
      %acc_ = arith.select %cnd, %cst_2, %arg4 : tensor<128x16xi1, #mma1>, tensor<128x16xf32, #mma1>
      %acc = ttng.warp_group_dot %A, %B, %acc_ : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x16xf16, #shared1, #smem> -> tensor<128x16xf32, #mma1>
      scf.yield %acc: tensor<128x16xf32, #mma1>
    }
    tt.return %17 : tensor<128x16xf32, #mma1>
  }
}
```
**EN:** This block defines `negative_sel_tensor` and exercises loop-carried state, predicated selection. Embedded check comments (CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `negative_sel_tensor`，并覆盖 循环携带状态、条件选择。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's TritonGPU transforms coverage and focuses on warp-level behavior.  
  **CN:** 该文件属于 Triton 的 TritonGPU 变换 测试覆盖，关注点是 warp 级行为。
- **EN:** The `RUN` pipeline drives tools/passes such as `-tritongpu-optimize-accumulator-init`.  
  **CN:** `RUN` 流水线会驱动 `-tritongpu-optimize-accumulator-init` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `arith` (scalar/tensor arithmetic), `ttng` (next-gen TritonGPU ops), `tt` (Triton core ops), `scf` (structured control flow).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`arith`（标量/张量算术）、`ttng`（新一代 TritonGPU 操作）、`tt`（Triton 核心操作）、`scf`（结构化控制流）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `arith`, `ttng`, `tt`, `scf`.  
  **CN:** IR 方言依赖：`ttg`、`arith`、`ttng`、`tt`、`scf`。
