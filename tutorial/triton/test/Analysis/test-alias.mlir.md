# test-alias.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Analysis/test-alias.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises alias analysis in Triton's analysis coverage. **CN:** 该MLIR 测试用于覆盖 Triton 分析中的别名分析相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -mlir-disable-threading -test-print-alias -verify-diagnostics -o /dev/null` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: none; diagnostics annotations: remark×55. **CN:** FileCheck 标记为 none；诊断标注为 remark×55。
- **Validation / 验证内容:** **EN:** The file mainly validates that alias analysis emits the expected analysis remarks/notes. **CN:** 该文件主要验证 别名分析 会发出预期的分析备注/说明。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -mlir-disable-threading -test-print-alias -verify-diagnostics -o /dev/null
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -mlir-disable-threading -test-print-alias -verify-diagnostics -o /dev/null` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -mlir-disable-threading -test-print-alias -verify-diagnostics -o /dev/null`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-11
```mlir
#AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#BL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [4, 1], order = [1, 0]}>
#A_SHARED_1D = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [0]}>
#A_SHARED = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#A_SHARED_T = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [0, 1]}>
#B_SHARED = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#C = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
#A_DOT = #ttg.dot_op<{opIdx = 0, parent = #C, kWidth=2}>
#B_DOT = #ttg.dot_op<{opIdx = 1, parent = #C, kWidth=2}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#AL`, `#BL`, `#A_SHARED_1D`, `#A_SHARED`, `#A_SHARED_T`, `#B_SHARED`, `#C`, `#A_DOT`, ... (+1). They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.swizzled_shared`, `#ttg.nvidia_mma`, `#ttg.dot_op`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#AL`, `#BL`, `#A_SHARED_1D`, `#A_SHARED`, `#A_SHARED_T`, `#B_SHARED`, `#C`, `#A_DOT`, ... (+1)。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.swizzled_shared`, `#ttg.nvidia_mma`, `#ttg.dot_op`。

### Line 13
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 15-31
```mlir
// There shouldn't be any aliasing with the dot op encoding.
tt.func @matmul_loop(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>) {
  %a_ptr_init = tt.splat %A : !tt.ptr<f16> -> tensor<128x32x!tt.ptr<f16>, #AL>
  %b_ptr_init = tt.splat %B : !tt.ptr<f16> -> tensor<32x128x!tt.ptr<f16>, #BL>
  %a_mask = arith.constant dense<true> : tensor<128x32xi1, #AL>
  %a_other = arith.constant dense<0.00e+00> : tensor<128x32xf16, #AL>
  %b_mask = arith.constant dense<true> : tensor<32x128xi1, #BL>
  %b_other = arith.constant dense<0.00e+00> : tensor<32x128xf16, #BL>
  %c_init = arith.constant dense<0.00e+00> : tensor<128x128xf32, #C>
  %a_off = arith.constant dense<4> : tensor<128x32xi32, #AL>
  %b_off = arith.constant dense<4> : tensor<32x128xi32, #BL>
  scf.for %iv = %lb to %ub step %step iter_args(%a_ptr = %a_ptr_init, %b_ptr = %b_ptr_init, %prev_c = %c_init) -> (tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>) {
    %a_ = tt.load %a_ptr, %a_mask, %a_other : tensor<128x32x!tt.ptr<f16>, #AL>
    %a = ttg.convert_layout %a_ : tensor<128x32xf16, #AL> -> tensor<128x32xf16, #A_DOT>
    %b_ = tt.load %b_ptr, %b_mask, %b_other : tensor<32x128x!tt.ptr<f16>, #BL>
    %b = ttg.convert_layout %b_ : tensor<32x128xf16, #BL> -> tensor<32x128xf16, #B_DOT>
    %c = tt.dot %a, %b, %prev_c : tensor<128x32xf16, #A_DOT> * tensor<32x128xf16, #B_DOT> -> tensor<128x128xf32, #C>
```
**EN:** This block defines `matmul_loop` and exercises layout conversion, dot products / matmul, tensor loads, loop-carried state.
**CN:** 这一块定义了 `matmul_loop`，并覆盖 布局转换、点积/矩阵乘、张量加载、循环携带状态。

### Lines 33-38
```mlir
    %next_a_ptr = tt.addptr %a_ptr, %a_off : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<128x32xi32, #AL>
    %next_b_ptr = tt.addptr %b_ptr, %b_off : tensor<32x128x!tt.ptr<f16>, #BL>, tensor<32x128xi32, #BL>
    scf.yield %next_a_ptr, %next_b_ptr, %c : tensor<128x32x!tt.ptr<f16>, #AL>, tensor<32x128x!tt.ptr<f16>, #BL>, tensor<128x128xf32, #C>
  }
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `tt.addptr`, `tt.ptr`, `scf.yield`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.addptr`, `tt.ptr`, `scf.yield`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 40-44
```mlir
tt.func @alloc(%A : !tt.ptr<f16>) {
  // expected-remark @below {{%0 -> %0}}
  %cst2 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `alloc` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `alloc`，并覆盖 共享内存分配。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Lines 46-51
```mlir
tt.func @alloc_init(%A : !tt.ptr<f16>) {
  %cst0 = arith.constant dense<0.000000e+00> : tensor<16x16xf16, #AL>
  // expected-remark @below {{%0 -> %0}}
  %cst1 = ttg.local_alloc %cst0 : (tensor<16x16xf16, #AL>) -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory>
  tt.return
}
```
**EN:** This block defines `alloc_init` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×1) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `alloc_init`，并覆盖 共享内存分配。 期望诊断标注（remark×1）用于捕获 pass 应当发出的分析备注/说明。

### Lines 53-59
```mlir
tt.func @trans(%A : !tt.ptr<f16>) {
  // expected-remark @below {{%0 -> %0}}
  %tensor = ttg.local_alloc : () -> !ttg.memdesc<16x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%1 -> %0}}
  %b = ttg.memdesc_trans %tensor {order=array<i32: 1,0>} : !ttg.memdesc<16x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<32x16xf16, #A_SHARED_T, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `trans` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `trans`，并覆盖 共享内存分配。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Lines 61-68
```mlir
tt.func @subview(%A : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory>) {
  %index = arith.constant 0 : i32
  // expected-remark @below {{%0 -> %0}}
  %a = ttg.local_alloc : () -> !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%1 -> %0}}
  %cst1 = ttg.memdesc_index %a[%index] : !ttg.memdesc<1x16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `subview` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `subview`，并覆盖 共享内存分配。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Lines 70-82
```mlir
tt.func @if_alias(%i1 : i1) {
  // expected-remark @below {{%0 -> %0}}
  %a = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%1 -> %1}}
  %b = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%2 -> %0,%1}}
  %cst2 = scf.if %i1 -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable> {
    scf.yield %a : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  } else {
    scf.yield %b : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  }
  tt.return
}
```
**EN:** This block defines `if_alias` and exercises shared-memory allocation, control-flow joins. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `if_alias`，并覆盖 共享内存分配、控制流汇合。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Lines 84-92
```mlir
tt.func @select_alias(%i1 : i1) {
  // expected-remark @below {{%0 -> %0}}
  %a = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%1 -> %1}}
  %b = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%2 -> %0,%1}}
  %c = arith.select %i1, %a, %b : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `select_alias` and exercises shared-memory allocation, predicated selection. Expected-diagnostic annotations (remark×3) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `select_alias`，并覆盖 共享内存分配、条件选择。 期望诊断标注（remark×3）用于捕获 pass 应当发出的分析备注/说明。

### Lines 94-112
```mlir
tt.func @for(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>) {
  // expected-remark @below {{%0 -> %0}}
  %a = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%1 -> %1}}
  %b = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%2 -> %2}}
  %c = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%arg6 -> %0}}
  // expected-remark @below {{%arg7 -> %1}}
  // expected-remark @below {{%arg8 -> %2}}
  // expected-remark @below {{%3#0 -> %0,%1}}
  // expected-remark @below {{%3#1 -> %0,%1}}
  // expected-remark @below {{%3#2 -> %0,%1,%2}}
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a, %b_shared = %b, %c_shared = %c) ->
  (!ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>) {
    scf.yield %b_shared, %a_shared, %a_shared : !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<16x16xf16, #A_SHARED, #ttg.shared_memory, mutable>
  }
  tt.return
}
```
**EN:** This block defines `for` and exercises shared-memory allocation, loop-carried state. Expected-diagnostic annotations (remark×9) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `for`，并覆盖 共享内存分配、循环携带状态。 期望诊断标注（remark×9）用于捕获 pass 应当发出的分析备注/说明。

### Lines 114-139
```mlir
tt.func @for_if(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>, %i1 : i1) {
  // expected-remark @below {{%0 -> %0}}
  %a_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
  // expected-remark @below {{%1 -> %1}}
  %b_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
  // expected-remark @below {{%2 -> %2}}
  %c_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
  // expected-remark @below {{%arg7 -> %0}}
  // expected-remark @below {{%arg8 -> %1}}
  // expected-remark @below {{%arg9 -> %2}}
  // expected-remark @below {{%3#0 -> %0,%1}}
  // expected-remark @below {{%3#1 -> %0,%1}}
  // expected-remark @below {{%3#2 -> %0,%1,%2}}
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init, %c_shared = %c_shared_init) ->
  (!ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>) {
    scf.if %i1 {
      %zero = arith.constant 0 : i32
      %index = arith.constant 8 : i32
      // expected-remark @below {{%4 -> %0,%1}}
      %cst0 = ttg.memdesc_index %a_shared[%index] : !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable> -> !ttg.memdesc<32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
      scf.yield
    }
    scf.yield %b_shared, %a_shared, %a_shared : !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED_1D, #ttg.shared_memory, mutable>
  }
  tt.return
}
```
**EN:** This block defines `for_if` and exercises shared-memory allocation, loop-carried state, control-flow joins. Expected-diagnostic annotations (remark×10) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `for_if`，并覆盖 共享内存分配、循环携带状态、控制流汇合。 期望诊断标注（remark×10）用于捕获 pass 应当发出的分析备注/说明。

### Lines 141-174
```mlir
tt.func @for_for_if(%lb : index, %ub : index, %step : index, %A : !tt.ptr<f16>, %B : !tt.ptr<f16>, %i1 : i1) {
  // expected-remark @below {{%0 -> %0}}
  %a_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%1 -> %1}}
  %b_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%2 -> %2}}
  %c_shared_init = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%arg7 -> %0}}
  // expected-remark @below {{%arg8 -> %1}}
  // expected-remark @below {{%arg9 -> %2}}
  // expected-remark @below {{%3#0 -> %0}}
  // expected-remark @below {{%3#1 -> %1}}
  // expected-remark @below {{%3#2 -> %2,%6,%6}}
  %a_shared, %b_shared, %c_shared = scf.for %iv = %lb to %ub step %step iter_args(%a_shared = %a_shared_init, %b_shared = %b_shared_init, %c_shared = %c_shared_init) ->
  (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>) {
    // expected-remark @below {{%arg11 -> %2,%6,%6}}
    // expected-remark @below {{%4 -> %2,%6,%6}}
    %c_shared_next = scf.for %jv = %lb to %ub step %step iter_args(%c_shared_next = %c_shared) -> (!ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>) {
      // expected-remark @below {{%5 -> %6,%6}}
      %c_shared_next_next = scf.if %i1 -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable> {
        // expected-remark @below {{%6 -> %6}}
        %cst0 = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
        scf.yield %cst0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
      } else {
        // expected-remark @below {{%6 -> %6}}
        %cst0 = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
        scf.yield %cst0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
      }
      scf.yield %c_shared_next_next : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
    }
    scf.yield %a_shared, %b_shared, %c_shared_next : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  }
  tt.return
}
```
**EN:** This block defines `for_for_if` and exercises shared-memory allocation, loop-carried state, control-flow joins. Expected-diagnostic annotations (remark×14) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `for_for_if`，并覆盖 共享内存分配、循环携带状态、控制流汇合。 期望诊断标注（remark×14）用于捕获 pass 应当发出的分析备注/说明。

### Lines 176-202
```mlir
tt.func @cf_for(%arg0: index, %arg1: index, %arg2: index, %arg3: !tt.ptr<f16>, %arg4: !tt.ptr<f16>) {
  // expected-remark @below {{%0 -> %0}}
  %cst = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%1 -> %1}}
  %cst_0 = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  // expected-remark @below {{%2 -> %0}}
  %0 = ttg.memdesc_subslice %cst [0, 0] : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  ttg.barrier local
  // expected-remark @below {{%3 -> %3}}
  %cst_1 = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  cf.br ^bb1(%arg0, %cst, %cst_0, %cst_1 : index, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>)
^bb1(%1: index, %2: !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, %3: !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, %4: !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>):  // 2 preds: ^bb0, ^bb2
  %5 = arith.cmpi slt, %1, %arg1 : index
  // expected-remark @below {{%5 -> %0,%1,%3}}
  // expected-remark @below {{%6 -> %0,%1,%3}}
  // expected-remark @below {{%7 -> %0,%1,%3}}
  cf.cond_br %5, ^bb2, ^bb3
^bb2:  // pred: ^bb1
  ttg.barrier local
  %8 = arith.addi %1, %arg2 : index
  cf.br ^bb1(%8, %4, %2, %3 : index, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>, !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>)
^bb3:  // pred: ^bb1
  ttg.barrier local
  // expected-remark @below {{%10 -> %0}}
  %9 = ttg.memdesc_subslice %0 [0, 0] : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `cf_for` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×8) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `cf_for`，并覆盖 共享内存分配。 期望诊断标注（remark×8）用于捕获 pass 应当发出的分析备注/说明。

### Lines 204-215
```mlir
tt.func @poison_memdesc(%arg0: i1) {
  // expected-remark @below {{%0 -> %0}}
  %0 = ttg.local_alloc : () -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  cf.cond_br %arg0, ^bb1, ^bb2(%0 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>)
^bb1:
  %1 = ub.poison : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  cf.br ^bb2(%1 : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>)
^bb2(%2: !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>):
  // expected-remark @below {{%3 -> %0}}
  %3 = ttg.memdesc_subslice %2 [0, 0]  : !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable> -> !ttg.memdesc<128x32xf16, #A_SHARED, #ttg.shared_memory, mutable>
  tt.return
}
```
**EN:** This block defines `poison_memdesc` and exercises shared-memory allocation. Expected-diagnostic annotations (remark×2) capture the analysis remarks/notes that the pass should emit.
**CN:** 这一块定义了 `poison_memdesc`，并覆盖 共享内存分配。 期望诊断标注（remark×2）用于捕获 pass 应当发出的分析备注/说明。

### Line 217
```mlir
}  // module
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's analysis coverage and focuses on alias analysis.  
  **CN:** 该文件属于 Triton 的 分析 测试覆盖，关注点是 别名分析。
- **EN:** The `RUN` pipeline drives tools/passes such as `-test-print-alias`.  
  **CN:** `RUN` 流水线会驱动 `-test-print-alias` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `scf` (structured control flow), `cf` (control-flow ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`scf`（结构化控制流）、`cf`（控制流操作）。
- **EN:** Expected remarks/notes are used to pin down the exact analysis output produced by the pass.  
  **CN:** 这里的期望备注/说明用于固定 pass 产生的精确分析输出。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`.  
  **CN:** 外部测试工具：`triton-opt`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `scf`, `cf`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`scf`、`cf`。
- **EN:** This test relies on the custom testing pass implemented in `test/lib/Analysis/TestAlias.cpp`.  
  **CN:** 该测试依赖 `test/lib/Analysis/TestAlias.cpp` 中实现的自定义测试 pass。
