# warp_specialize_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/warp_specialize_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises warp-level behavior in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的warp 级行为相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file -mlir-print-local-scope -allow-unregistered-dialect -convert-warp-specialize-to-llvm -canonicalize=region-simplify=disabled | FileCheck %s --check-prefixes=COMMON,CHECK`<br>`triton-opt %s -split-input-file -mlir-print-local-scope -allow-unregistered-dialect -triton-amdgpu-convert-warp-specialize-to-llvm=gfx-arch=gfx1250 -canonicalize=region-simplify=disabled | FileCheck %s --check-prefixes=COMMON,AMD` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×98, CHECK-DAG×26, CHECK-LABEL×5, CHECK-NEXT×224, CHECK-NOT×1; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×98, CHECK-DAG×26, CHECK-LABEL×5, CHECK-NEXT×224, CHECK-NOT×1；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that warp-level behavior produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 warp 级行为 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s -split-input-file -mlir-print-local-scope -allow-unregistered-dialect -convert-warp-specialize-to-llvm -canonicalize=region-simplify=disabled | FileCheck %s --check-prefixes=COMMON,CHECK
// RUN: triton-opt %s -split-input-file -mlir-print-local-scope -allow-unregistered-dialect -triton-amdgpu-convert-warp-specialize-to-llvm=gfx-arch=gfx1250 -canonicalize=region-simplify=disabled | FileCheck %s --check-prefixes=COMMON,AMD
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file -mlir-print-local-scope -allow-unregistered-dialect -convert-warp-specialize-to-llvm -canonicalize=region-simplify=disabled | FileCheck %s --check-prefixes=COMMON,CHECK; triton-opt %s -split-input-file -mlir-print-local-scope -allow-unregistered-dialect -triton-amdgpu-convert-warp-specialize-to-llvm=gfx-arch=gfx1250 -canonicalize=region-simplify=disabled | FileCheck %s --check-prefixes=COMMON,AMD` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file -mlir-print-local-scope -allow-unregistered-dialect -convert-warp-specialize-to-llvm -canonicalize=region-simplify=disabled | FileCheck %s --check-prefixes=COMMON,CHECK; triton-opt %s -split-input-file -mlir-print-local-scope -allow-unregistered-dialect -triton-amdgpu-convert-warp-specialize-to-llvm=gfx-arch=gfx1250 -canonicalize=region-simplify=disabled | FileCheck %s --check-prefixes=COMMON,AMD`，从而以预期的 pass 顺序和校验模式执行测试。

### Line 4
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 11 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 6
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 8-15
```mlir
// CHECK-LABEL: @rewrite_barriers
llvm.func @rewrite_barriers() attributes {allocation.offset = 32 : i32} {
  // CHECK-DAG: [[C0:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK-DAG: [[C1:%.*]] = llvm.mlir.constant(1 : i32)
  // CHECK-DAG: [[C2:%.*]] = llvm.mlir.constant(2 : i32)
  // CHECK-DAG: [[C3:%.*]] = llvm.mlir.constant(3 : i32)
  // CHECK-DAG: [[C64:%.*]] = llvm.mlir.constant(64 : i32)
  // CHECK-DAG: [[C128:%.*]] = llvm.mlir.constant(128 : i32)
```
**EN:** This block defines `rewrite_barriers` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.constant`. Embedded check comments (CHECK-DAG×6, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `rewrite_barriers`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×6, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 17-19
```mlir
  // CHECK: nvvm.barrier id = [[C2]] number_of_threads = [[C128]]
  // CHECK: nvvm.barrier id = [[C3]] number_of_threads = [[C64]]
  // CHECK: bar.warp.sync
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 21-45
```mlir
  // CHECK: bb{{[0-9]+}}:
  // CHECK-NEXT: nvvm.barrier id = [[C0]] number_of_threads = [[C128]]
  nvvm.barrier0
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4, 8, 10>}
  default {
    // CHECK: nvvm.barrier id = [[C0]] number_of_threads = [[C128]]
    nvvm.barrier0
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    nvvm.barrier0
    ttg.warp_return
  }
  partition1() num_warps(2) {
    nvvm.barrier0
    ttg.warp_return
  }
  partition2() num_warps(1) {
    nvvm.barrier0
    ttg.warp_return
  } : () -> ()
  // CHECK: nvvm.barrier id = [[C0]] number_of_threads = [[C128]]
  nvvm.barrier0
  llvm.return
}
```
**EN:** This block defines the test function(s) and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×3, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×3, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 47
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 49
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 51
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 11 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.target" = "hip:gfx1250"} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.total-num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.total-num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 53
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 55-59
```mlir
// AMD-LABEL: @rewrite_barriers
// AMD-DAG: llvm.mlir.global internal @nbar1
// AMD-DAG: llvm.mlir.global internal @nbar2
// AMD-DAG: llvm.mlir.global internal @nbar3
// AMD-DAG: llvm.mlir.global internal @nbar4
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD-LABEL: @rewrite_barriers`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD-LABEL: @rewrite_barriers`。

### Lines 61-63
```mlir
llvm.func @rewrite_barriers() attributes {allocation.offset = 32 : i32} {
  // AMD: bb{{[0-9]+}}:
  // AMD-NEXT: rocdl.barrier
```
**EN:** This block defines `rewrite_barriers` and exercises operations such as `llvm.func`, `allocation.offset`, `rocdl.barrier`.
**CN:** 这一块定义了 `rewrite_barriers`，并覆盖 如 `llvm.func`、`allocation.offset`、`rocdl.barrier` 这样的操作。

### Lines 65-70
```mlir
  // Check that named barriers are used and that we have the correct counts:
  // AMD-DAG-COUNT-6: rocdl.s.barrier.join
  // AMD-DAG-COUNT-4: rocdl.s.barrier.signal.var {{.*}}, 4
  // AMD-DAG-COUNT-1: rocdl.s.barrier.signal.var {{.*}}, 2
  // AMD-DAG-COUNT-1: rocdl.s.barrier.signal.var {{.*}}, 1
  // AMD-DAG-COUNT-6: rocdl.s.barrier.wait 1
```
**EN:** This comment block provides context for the surrounding test logic: `// Check that named barriers are used and that we have the correct counts:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// Check that named barriers are used and that we have the correct counts:`。

### Lines 72-92
```mlir
  rocdl.barrier
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4, 8, 10>}
  default {
    rocdl.barrier
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    rocdl.barrier
    ttg.warp_return
  }
  partition1() num_warps(2) {
    rocdl.barrier
    ttg.warp_return
  }
  partition2() num_warps(1) {
    rocdl.barrier
    ttg.warp_return
  } : () -> ()
  rocdl.barrier
  llvm.return
}
```
**EN:** This block contributes intermediate IR built from `rocdl.barrier`, `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `rocdl.barrier`, `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 94
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 96
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 98
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 11 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 100
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 102-110
```mlir
// CHECK: llvm.func internal @inner_func_nw4_ws(%arg0: i32)
llvm.func internal @inner_func_nw4() attributes {"ws_num_warps" = 4 : i32} {
  // CHECK: [[C128:%.*]] = llvm.mlir.constant(128 : i32)
  // CHECK: nvvm.barrier id = %arg0 number_of_threads = [[C128]]
  // CHECK: llvm.call @inner_func_nw4_ws(%arg0) : (i32) -> ()
  nvvm.barrier0
  llvm.call @inner_func_nw4() : () -> ()
  llvm.return
}
```
**EN:** This block defines `inner_func_nw4` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `inner_func_nw4`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 112-118
```mlir
// CHECK: llvm.func internal @inner_func_nw2_ws(%arg0: f32 {some.attr = "some_value"}, %arg1: i32)
llvm.func internal @inner_func_nw2(%arg0: f32 {some.attr = "some_value"}) attributes {"ws_num_warps" = 2 : i32} {
  // CHECK: [[C64:%.*]] = llvm.mlir.constant(64 : i32)
  // CHECK: nvvm.barrier id = %arg1 number_of_threads = [[C64]]
  nvvm.barrier0
  llvm.return
}
```
**EN:** This block defines `inner_func_nw2` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `inner_func_nw2`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 120-125
```mlir
// CHECK: llvm.func internal @inner_func_nw1_ws(%arg0: i32)
llvm.func internal @inner_func_nw1() attributes {"ws_num_warps" = 1 : i32} {
  // CHECK: nvvm.bar.warp.sync
  nvvm.barrier0
  llvm.return
}
```
**EN:** This block defines `inner_func_nw1` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `inner_func_nw1`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 127
```mlir
llvm.func @__libdevice_function()
```
**EN:** This block defines `__libdevice_function` and exercises operations such as `llvm.func`.
**CN:** 这一块定义了 `__libdevice_function`，并覆盖 如 `llvm.func` 这样的操作。

### Lines 129-159
```mlir
// CHECK: llvm.func @rewrite_barriers()
llvm.func @rewrite_barriers() attributes {allocation.offset = 32 : i32} {
  // CHECK-DAG: [[CST:%.*]] = llvm.mlir.constant({{.*}}) : f32
  // CHECK-DAG: [[C0:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK-DAG: [[C2:%.*]] = llvm.mlir.constant(2 : i32)
  // CHECK-DAG: [[C3:%.*]] = llvm.mlir.constant(3 : i32)
  // CHECK-DAG: [[C4:%.*]] = llvm.mlir.constant(4 : i32)
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4, 8, 10>}
  default {
    llvm.call @inner_func_nw4() : () -> ()
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    // CHECK: call @inner_func_nw4_ws([[C2]]) : (i32) -> ()
    llvm.call @inner_func_nw4() : () -> ()
    // CHEC: call @__libdevice_function() : () -> ()
    llvm.call @__libdevice_function() : () -> ()
    ttg.warp_return
  }
  partition1() num_warps(2) {
    // CHECK: call @inner_func_nw2_ws([[CST]], [[C3]]) : (f32, i32) -> ()
    %cst = llvm.mlir.constant(4.2 : f32) : f32
    llvm.call @inner_func_nw2(%cst) : (f32) -> ()
    ttg.warp_return
  }
  partition2() num_warps(1) {
    // CHECK: call @inner_func_nw1_ws([[C4]]) : (i32) -> ()
    llvm.call @inner_func_nw1() : () -> ()
    ttg.warp_return
  } : () -> ()
  // CHECK: call @inner_func_nw4_ws([[C0]]) : (i32) -> ()
```
**EN:** This block defines `rewrite_barriers`, `inner_func_nw4`, `__libdevice_function`, `inner_func_nw2`, `inner_func_nw1` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.constant`, `ttg.warp_specialize`, `llvm.call`. Embedded check comments (CHECK×5, CHECK-DAG×5) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `rewrite_barriers`, `inner_func_nw4`, `__libdevice_function`, `inner_func_nw2`, `inner_func_nw1`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.constant`、`ttg.warp_specialize`、`llvm.call` 这样的操作。 其中嵌入的检查注释（CHECK×5, CHECK-DAG×5）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 161-164
```mlir
  // CHECK: call @inner_func_nw4_ws([[C0]]) : (i32) -> ()
  llvm.call @inner_func_nw4() : () -> ()
  llvm.return
}
```
**EN:** This block defines `inner_func_nw4` and exercises operations such as `llvm.call`, `llvm.return`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `inner_func_nw4`，并覆盖 如 `llvm.call`、`llvm.return` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 166
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 168
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 170
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 11 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 172
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 174-181
```mlir
// COMMON-LABEL: @generate_switch_loop
llvm.func @generate_switch_loop() attributes {allocation.offset = 32 : i32} {
  // CHECK-DAG: [[CNEG1:%.*]] = llvm.mlir.constant(-1 : i32)
  // CHECK-DAG: [[C0:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK-DAG: [[C1:%.*]] = llvm.mlir.constant(1 : i32)
  // COMMON-DAG: [[C4:%.*]] = llvm.mlir.constant(4 : i32)
  // CHECK-DAG: [[C31:%.*]] = llvm.mlir.constant(31 : i32)
  // CHECK-DAG: [[C32:%.*]] = llvm.mlir.constant(32 : i32)
```
**EN:** This block defines `generate_switch_loop` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.constant`. Embedded check comments (CHECK-DAG×5) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `generate_switch_loop`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×5）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 183-186
```mlir
  // COMMON-DAG: [[C0_i8:%.*]] = llvm.mlir.constant(0 : i8)
  // COMMON-DAG: [[C1_i8:%.*]] = llvm.mlir.constant(1 : i8)
  // COMMON-DAG: [[C2_i8:%.*]] = llvm.mlir.constant(2 : i8)
  // COMMON-DAG: [[C3_i8:%.*]] = llvm.mlir.constant(3 : i8)
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON-DAG: [[C0_i8:%.*]] = llvm.mlir.constant(0 : i8)`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON-DAG: [[C0_i8:%.*]] = llvm.mlir.constant(0 : i8)`。

### Line 188
```mlir
  // COMMON-DAG: [[SMEM_ADDR:%.*]] = llvm.mlir.addressof @global_smem
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON-DAG: [[SMEM_ADDR:%.*]] = llvm.mlir.addressof @global_smem`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON-DAG: [[SMEM_ADDR:%.*]] = llvm.mlir.addressof @global_smem`。

### Lines 190-194
```mlir
  // CHECK-NEXT: [[TIDX:%.*]] = nvvm.read.ptx.sreg.tid.x
  // CHECK-NEXT: [[WID:%.*]] = llvm.udiv [[TIDX]], [[C32]]
  // CHECK-NEXT: [[WARP_ID:%.*]] = nvvm.shfl.sync idx [[CNEG1]], [[WID]], [[C0]], [[C31]]
  // CHECK-NEXT: [[IS_DEFAULT:%.*]] = llvm.icmp "ult" [[WARP_ID]], [[C4]]
  // CHECK-NEXT: llvm.cond_br [[IS_DEFAULT]], [[BODY:\^.*]], [[SWITCH_LOOP:\^.*]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 196-199
```mlir
  // CHECK: [[SWITCH_LOOP]]:
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: [[SMEM_BASE:%.*]] = llvm.getelementptr [[SMEM_ADDR]][32] : (!llvm.ptr<3>) -> !llvm.ptr<3>, i8
  // CHECK-NEXT: [[REL_WID:%.*]] = llvm.sub [[WARP_ID]], [[C4]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 201-207
```mlir
  // CHECK-NEXT: [[STATE_PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][[[REL_WID]]]
  // CHECK-NEXT: [[STATE:%.*]] = llvm.load [[STATE_PTR]]
  // CHECK-NEXT: llvm.switch [[STATE]] : i8, [[DEFAULT:\^.*]] [
  // CHECK-NEXT: 0: [[PARTITION0:\^.*]],
  // CHECK-NEXT: 1: [[PARTITION1:\^.*]],
  // CHECK-NEXT: 2: [[PARTITION2:\^.*]],
  // CHECK-NEXT: 3: [[EXIT:\^.*]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 209-212
```mlir
  // CHECK: [[DEFAULT]]:
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: llvm.br [[SWITCH_LOOP]] {loop_annotation = #llvm.loop_annotation<licm = <disable = true>>}
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 214-215
```mlir
  // CHECK: [[EXIT]]:
  // CHECK-NEXT: llvm.return
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 217-221
```mlir
  // CHECK: [[PARTITION0]]:
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "partition0"
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: llvm.br [[SWITCH_LOOP]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 223-227
```mlir
  // CHECK: [[PARTITION1]]:
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "partition1"
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: llvm.br [[SWITCH_LOOP]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 229-233
```mlir
  // CHECK: [[PARTITION2]]:
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "partition2"
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: llvm.br [[SWITCH_LOOP]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 235-237
```mlir
  // CHECK: [[BODY]]:
  // CHECK-NEXT: "before"
  // CHECK-NEXT: [[SMEM_BASE:%.*]] = llvm.getelementptr [[SMEM_ADDR]][32]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 239-245
```mlir
  // CHECK-NEXT: llvm.store [[C0_i8]], [[SMEM_BASE]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][1]
  // CHECK-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][2]
  // CHECK-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][3]
  // CHECK-NEXT: llvm.store [[C0_i8]], [[PTR]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 247-250
```mlir
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][4]
  // CHECK-NEXT: llvm.store [[C1_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][5]
  // CHECK-NEXT: llvm.store [[C1_i8]], [[PTR]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 252-253
```mlir
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][6]
  // CHECK-NEXT: llvm.store [[C2_i8]], [[PTR]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 255-261
```mlir
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: llvm.br [[DEFAULT_PARTITION:\^.*]]
  // CHECK: [[DEFAULT_PARTITION]]:
  // CHECK-NEXT: "default"
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: llvm.br [[AFTER:\^.*]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 263-265
```mlir
  // AMD: [[WID:%.*]] = llvm.call_intrinsic "llvm.amdgcn.wave.id"
  // AMD-NEXT: [[IS_DEFAULT:%.*]] = llvm.icmp "ult" [[WID]], [[C4]]
  // AMD-NEXT: llvm.cond_br [[IS_DEFAULT]], [[BODY:\^bb[0-9]+]], [[SWITCH_LOOP:\^bb[0-9]+]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: [[WID:%.*]] = llvm.call_intrinsic "llvm.amdgcn.wave.id"`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: [[WID:%.*]] = llvm.call_intrinsic "llvm.amdgcn.wave.id"`。

### Lines 267-270
```mlir
  // AMD: [[SWITCH_LOOP]]:
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: [[SMEM_BASE:%.*]] = llvm.getelementptr [[SMEM_ADDR]][32] : (!llvm.ptr<3>) -> !llvm.ptr<3>, i8
  // AMD-NEXT: [[REL_WID:%.*]] = llvm.sub [[WID]], [[C4]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: [[SWITCH_LOOP]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: [[SWITCH_LOOP]]:`。

### Lines 272-278
```mlir
  // AMD-NEXT: [[STATE_PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][[[REL_WID]]]
  // AMD-NEXT: [[STATE:%.*]] = llvm.load [[STATE_PTR]]
  // AMD-NEXT: llvm.switch [[STATE]] : i8, [[DEFAULT:\^bb[0-9]+]] [
  // AMD-NEXT: 0: [[PARTITION0:\^bb[0-9]+]],
  // AMD-NEXT: 1: [[PARTITION1:\^bb[0-9]+]],
  // AMD-NEXT: 2: [[PARTITION2:\^bb[0-9]+]],
  // AMD-NEXT: 3: [[EXIT:\^bb[0-9]+]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD-NEXT: [[STATE_PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][[[REL_WID]]]`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD-NEXT: [[STATE_PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][[[REL_WID]]]`。

### Lines 280-283
```mlir
  // AMD: [[DEFAULT]]:
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: llvm.br [[SWITCH_LOOP]] {loop_annotation = #llvm.loop_annotation<licm = <disable = true>>}
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: [[DEFAULT]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: [[DEFAULT]]:`。

### Lines 285-286
```mlir
  // AMD: [[EXIT]]:
  // AMD-NEXT: llvm.return
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: [[EXIT]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: [[EXIT]]:`。

### Lines 288-292
```mlir
  // AMD: [[PARTITION0]]:
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: "partition0"
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: llvm.br [[SWITCH_LOOP]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: [[PARTITION0]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: [[PARTITION0]]:`。

### Lines 294-298
```mlir
  // AMD: [[PARTITION1]]:
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: "partition1"
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: llvm.br [[SWITCH_LOOP]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: [[PARTITION1]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: [[PARTITION1]]:`。

### Lines 300-304
```mlir
  // AMD: [[PARTITION2]]:
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: "partition2"
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: llvm.br [[SWITCH_LOOP]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: [[PARTITION2]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: [[PARTITION2]]:`。

### Lines 306-308
```mlir
  // AMD: [[BODY]]:
  // AMD-NEXT: "before"
  // AMD-NEXT: [[SMEM_BASE:%.*]] = llvm.getelementptr [[SMEM_ADDR]][32]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: [[BODY]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: [[BODY]]:`。

### Lines 310-316
```mlir
  // AMD-NEXT: llvm.store [[C0_i8]], [[SMEM_BASE]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][1]
  // AMD-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][2]
  // AMD-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][3]
  // AMD-NEXT: llvm.store [[C0_i8]], [[PTR]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD-NEXT: llvm.store [[C0_i8]], [[SMEM_BASE]]`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD-NEXT: llvm.store [[C0_i8]], [[SMEM_BASE]]`。

### Lines 318-321
```mlir
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][4]
  // AMD-NEXT: llvm.store [[C1_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][5]
  // AMD-NEXT: llvm.store [[C1_i8]], [[PTR]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][4]`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][4]`。

### Lines 323-324
```mlir
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][6]
  // AMD-NEXT: llvm.store [[C2_i8]], [[PTR]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][6]`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][6]`。

### Lines 326-332
```mlir
  // AMD: rocdl.barrier
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: llvm.br [[DEFAULT_PARTITION:\^bb[0-9]+]]
  // AMD: [[DEFAULT_PARTITION]]:
  // AMD-NEXT: "default"
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: llvm.br [[AFTER:\^bb[0-9]+]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: rocdl.barrier`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: rocdl.barrier`。

### Lines 334-353
```mlir
  "before"() : () -> ()
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4, 8, 10>}
  default {
    "default"() : () -> ()
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    "partition0"() : () -> ()
    ttg.warp_return
  }
  partition1() num_warps(2) {
    "partition1"() : () -> ()
    ttg.warp_return
  }
  partition2() num_warps(1) {
    "partition2"() : () -> ()
    ttg.warp_return
  } : () -> ()
  // CHECK: [[AFTER]]:
  // CHECK-NEXT: "after"
```
**EN:** This block defines the test function(s) and exercises operations such as `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`. Embedded check comments (CHECK×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `ttg.warp_specialize`、`allocation.offset`、`ttg.warp_yield`、`ttg.warp_return` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 355
```mlir
  // CHECK-NEXT: [[SMEM_BASE:%.*]] = llvm.getelementptr [[SMEM_ADDR]][32]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 357-371
```mlir
  // CHECK-NEXT: llvm.store [[C3_i8]], [[SMEM_BASE]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][1]
  // CHECK-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][2]
  // CHECK-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][3]
  // CHECK-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][4]
  // CHECK-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][5]
  // CHECK-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][6]
  // CHECK-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: llvm.return
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 373-374
```mlir
  // AMD: [[AFTER:\^bb[0-9]+]]:
  // AMD-NEXT: "after"
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: [[AFTER:\^bb[0-9]+]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: [[AFTER:\^bb[0-9]+]]:`。

### Line 376
```mlir
  // AMD-NEXT: [[SMEM_BASE:%.*]] = llvm.getelementptr [[SMEM_ADDR]][32]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD-NEXT: [[SMEM_BASE:%.*]] = llvm.getelementptr [[SMEM_ADDR]][32]`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD-NEXT: [[SMEM_BASE:%.*]] = llvm.getelementptr [[SMEM_ADDR]][32]`。

### Lines 378-392
```mlir
  // AMD-NEXT: llvm.store [[C3_i8]], [[SMEM_BASE]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][1]
  // AMD-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][2]
  // AMD-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][3]
  // AMD-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][4]
  // AMD-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][5]
  // AMD-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][6]
  // AMD-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: llvm.return
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD-NEXT: llvm.store [[C3_i8]], [[SMEM_BASE]]`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD-NEXT: llvm.store [[C3_i8]], [[SMEM_BASE]]`。

### Lines 394-396
```mlir
  "after"() : () -> ()
  llvm.return
}
```
**EN:** This block contributes intermediate IR built from `llvm.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 398
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 400
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 402
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 404
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 406-410
```mlir
// COMMON-LABEL: @pass_captures
llvm.func @pass_captures() attributes {allocation.offset = 32 : i32} {
  // CHECK-DAG: [[C0:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK-DAG: [[C1:%.*]] = llvm.mlir.constant(1 : i32)
  // COMMON-DAG: [[SMEM_ADDR:%.*]] = llvm.mlir.addressof @global_smem
```
**EN:** This block defines `pass_captures` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.constant`, `llvm.mlir.addressof`. Embedded check comments (CHECK-DAG×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `pass_captures`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.constant`、`llvm.mlir.addressof` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 412-419
```mlir
  // CHECK: ^bb4:
  // CHECK-NEXT: [[ARG0_PTR:%.*]] = llvm.getelementptr [[SMEM_ADDR]][0, 0] : (!llvm.ptr<3>) -> !llvm.ptr<3>, !llvm.struct<packed (i32, i64)>
  // CHECK-NEXT: [[ARG0:%.*]] = llvm.load [[ARG0_PTR]] {alignment = 1 : i64}
  // CHECK-NEXT: [[ARG1_PTR:%.*]] = llvm.getelementptr [[SMEM_ADDR]][0, 1] : (!llvm.ptr<3>) -> !llvm.ptr<3>, !llvm.struct<packed (i32, i64)>
  // CHECK-NEXT: [[ARG1:%.*]] = llvm.load [[ARG1_PTR]] {alignment = 1 : i64}
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "use"([[ARG0]], [[ARG1]])
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 421-428
```mlir
  // CHECK: ^bb5:
  // CHECK: [[INS:%.*]]:2 = "produce"()
  // CHECK: [[ARG0_PTR:%.*]] = llvm.getelementptr [[SMEM_ADDR]][0, 0] : (!llvm.ptr<3>) -> !llvm.ptr<3>, !llvm.struct<packed (i32, i64)>
  // CHECK-NEXT: llvm.store [[INS]]#0, [[ARG0_PTR]] {alignment = 1 : i64}
  // CHECK-NEXT: [[ARG1_PTR:%.*]] = llvm.getelementptr [[SMEM_ADDR]][0, 1] : (!llvm.ptr<3>) -> !llvm.ptr<3>, !llvm.struct<packed (i32, i64)>
  // CHECK-NEXT: llvm.store [[INS]]#1, [[ARG1_PTR]] {alignment = 1 : i64}
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 430-437
```mlir
  // AMD: ^bb4:
  // AMD-NEXT: [[ARG0_PTR:%.*]] = llvm.getelementptr [[SMEM_ADDR]][0, 0] : (!llvm.ptr<3>) -> !llvm.ptr<3>, !llvm.struct<packed (i32, i64)>
  // AMD-NEXT: [[ARG0:%.*]] = llvm.load [[ARG0_PTR]] {alignment = 1 : i64}
  // AMD-NEXT: [[ARG1_PTR:%.*]] = llvm.getelementptr [[SMEM_ADDR]][0, 1] : (!llvm.ptr<3>) -> !llvm.ptr<3>, !llvm.struct<packed (i32, i64)>
  // AMD-NEXT: [[ARG1:%.*]] = llvm.load [[ARG1_PTR]] {alignment = 1 : i64}
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: "use"([[ARG0]], [[ARG1]])
  // AMD-NEXT: rocdl.barrier
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: ^bb4:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: ^bb4:`。

### Lines 439-446
```mlir
  // AMD: ^bb5:
  // AMD: [[INS:%.*]]:2 = "produce"()
  // AMD: [[ARG0_PTR:%.*]] = llvm.getelementptr [[SMEM_ADDR]][0, 0] : (!llvm.ptr<3>) -> !llvm.ptr<3>, !llvm.struct<packed (i32, i64)>
  // AMD-NEXT: llvm.store [[INS]]#0, [[ARG0_PTR]] {alignment = 1 : i64}
  // AMD-NEXT: [[ARG1_PTR:%.*]] = llvm.getelementptr [[SMEM_ADDR]][0, 1] : (!llvm.ptr<3>) -> !llvm.ptr<3>, !llvm.struct<packed (i32, i64)>
  // AMD-NEXT: llvm.store [[INS]]#1, [[ARG1_PTR]] {alignment = 1 : i64}
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: rocdl.barrier
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: ^bb5:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: ^bb5:`。

### Lines 448-458
```mlir
  %ins:2 = "produce"() : () -> (i32, i64)
  ttg.warp_specialize(%ins#0, %ins#1) attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4>}
  default {
    ttg.warp_yield
  }
  partition0(%arg2: i32, %arg3: i64) num_warps(4) {
    "use"(%arg2, %arg3) : (i32, i64) -> ()
    ttg.warp_return
  } : (i32, i64) -> ()
  llvm.return
}
```
**EN:** This block contributes intermediate IR built from `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 460
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 462
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 464
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 18 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 466
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 468-473
```mlir
// COMMON-LABEL: @partition_warpid_order
llvm.func @partition_warpid_order() attributes {allocation.offset = 32 : i32} {
  // COMMON-DAG: [[SMEM_ADDR:%.*]] = llvm.mlir.addressof @global_smem
  // COMMON-DAG: [[C0_i8:%.*]] = llvm.mlir.constant(0 : i8)
  // COMMON-DAG: [[C1_i8:%.*]] = llvm.mlir.constant(1 : i8)
  // COMMON-DAG: [[C2_i8:%.*]] = llvm.mlir.constant(2 : i8)
```
**EN:** This block defines `partition_warpid_order` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.addressof`, `llvm.mlir.constant`.
**CN:** 这一块定义了 `partition_warpid_order`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.addressof`、`llvm.mlir.constant` 这样的操作。

### Lines 475-479
```mlir
  // COMMON: llvm.switch
  // COMMON-NEXT: 0: [[PARTITION0:\^.*]],
  // COMMON-NEXT: 1: [[PARTITION1:\^.*]],
  // COMMON-NEXT: 2: [[PARTITION2:\^.*]],
  // COMMON-NEXT: 3: [[EXIT:\^.*]]
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: llvm.switch`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: llvm.switch`。

### Lines 481-486
```mlir
  // COMMON: [[PARTITION0]]:
  // COMMON: "ws0_partition0"
  // COMMON: [[PARTITION1]]:
  // COMMON: "ws0_partition1"
  // COMMON: [[PARTITION2]]:
  // COMMON: "ws0_partition2"
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: [[PARTITION0]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: [[PARTITION0]]:`。

### Line 488
```mlir
  // COMMON: [[SMEM_BASE:%.*]] = llvm.getelementptr [[SMEM_ADDR]]
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: [[SMEM_BASE:%.*]] = llvm.getelementptr [[SMEM_ADDR]]`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: [[SMEM_BASE:%.*]] = llvm.getelementptr [[SMEM_ADDR]]`。

### Lines 490-492
```mlir
  // COMMON-NEXT: llvm.store [[C1_i8]], [[SMEM_BASE]]
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[1]
  // COMMON-NEXT: llvm.store [[C1_i8]], [[PTR]]
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON-NEXT: llvm.store [[C1_i8]], [[SMEM_BASE]]`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON-NEXT: llvm.store [[C1_i8]], [[SMEM_BASE]]`。

### Lines 494-501
```mlir
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[2]
  // COMMON-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[3]
  // COMMON-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[4]
  // COMMON-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[5]
  // COMMON-NEXT: llvm.store [[C0_i8]], [[PTR]]
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[2]`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[2]`。

### Lines 503-537
```mlir
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[6]
  // COMMON-NEXT: llvm.store [[C2_i8]], [[PTR]]
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[7]
  // COMMON-NEXT: llvm.store [[C2_i8]], [[PTR]]
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[8]
  // COMMON-NEXT: llvm.store [[C2_i8]], [[PTR]]
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[9]
  // COMMON-NEXT: llvm.store [[C2_i8]], [[PTR]]
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[10]
  // COMMON-NEXT: llvm.store [[C2_i8]], [[PTR]]
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[11]
  // COMMON-NEXT: llvm.store [[C2_i8]], [[PTR]]
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[12]
  // COMMON-NEXT: llvm.store [[C2_i8]], [[PTR]]
  // COMMON-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[13]
  // COMMON-NEXT: llvm.store [[C2_i8]], [[PTR]]
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 6, 4, 10>}
  default {
    "ws0_default"() : () -> ()
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    "ws0_partition0"() : () -> ()
    ttg.warp_return
  }
  partition1() num_warps(2) {
    "ws0_partition1"() : () -> ()
    ttg.warp_return
  }
  partition2() num_warps(8) {
    "ws0_partition2"() : () -> ()
    ttg.warp_return
  } : () -> ()
  llvm.return
}
```
**EN:** This block contributes intermediate IR built from `llvm.getelementptr`, `llvm.store`, `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.getelementptr`, `llvm.store`, `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 539
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 541
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 543
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 18 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 545
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 547-550
```mlir
// CHECK-LABEL: @warpid_warp_specialize
llvm.func @warpid_warp_specialize() attributes {allocation.offset = 32 : i32} {
  // CHECK-DAG: [[C4:%.*]] = llvm.mlir.constant(4 : i32)
  // CHECK-DAG: [[C6:%.*]] = llvm.mlir.constant(6 : i32)
```
**EN:** This block defines `warpid_warp_specialize` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.constant`. Embedded check comments (CHECK-DAG×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warpid_warp_specialize`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 552-556
```mlir
  // Partition warp IDs are rewritten to be relative in this pass, while
  // keeping ttg.warp_id for NVGPUToLLVM to lower later.
  // CHECK: %{{.*}} = ttg.warp_id
  // CHECK-NEXT: [[REL0:%.*]] = llvm.sub %{{.*}}, [[C6]] : i32
  // CHECK-NEXT: "use"([[REL0]]) : (i32) -> ()
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 558-560
```mlir
  // CHECK: %{{.*}} = ttg.warp_id
  // CHECK-NEXT: [[REL1:%.*]] = llvm.sub %{{.*}}, [[C4]] : i32
  // CHECK-NEXT: "use"([[REL1]]) : (i32) -> ()
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 562-563
```mlir
  %0 = ttg.warp_id
  "use"(%0) : (i32) -> ()
```
**EN:** This block contributes intermediate IR built from `ttg.warp_id`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.warp_id` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 565-582
```mlir
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 6, 4>}
  default {
    %1 = ttg.warp_id
    "use"(%1) : (i32) -> ()
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    %1 = ttg.warp_id
    "use"(%1) : (i32) -> ()
    ttg.warp_return
  }
  partition1() num_warps(2) {
    %1 = ttg.warp_id
    "use"(%1) : (i32) -> ()
    ttg.warp_return
  } : () -> ()
  llvm.return
}
```
**EN:** This block contributes intermediate IR built from `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_id`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_id`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 584
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 586
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 588
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 12 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 590
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 592-602
```mlir
// COMMON-LABEL: @multiple_specialize
llvm.func @multiple_specialize() attributes {allocation.offset = 32 : i32} {
  // COMMON-DAG: llvm.mlir.addressof @global_smem
  // CHECK-DAG: [[C1:%.*]] = llvm.mlir.constant(1 : i32)
  // COMMON-DAG: [[C0_i8:%.*]] = llvm.mlir.constant(0 : i8)
  // COMMON-DAG: [[C1_i8:%.*]] = llvm.mlir.constant(1 : i8)
  // COMMON-DAG: [[C2_i8:%.*]] = llvm.mlir.constant(2 : i8)
  // COMMON-DAG: [[C3_i8:%.*]] = llvm.mlir.constant(3 : i8)
  // COMMON-DAG: [[C4_i8:%.*]] = llvm.mlir.constant(4 : i8)
  // COMMON-DAG: [[C5_i8:%.*]] = llvm.mlir.constant(5 : i8)
  // COMMON-DAG: [[Cn1_i8:%.*]] = llvm.mlir.constant(-1 : i8)
```
**EN:** This block defines `multiple_specialize` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.addressof`, `llvm.mlir.constant`. Embedded check comments (CHECK-DAG×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `multiple_specialize`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.addressof`、`llvm.mlir.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 604-611
```mlir
  // CHECK: llvm.switch
  // CHECK-NEXT: 0: [[WS0_PARTITION0:\^.*]],
  // CHECK-NEXT: 1: [[WS0_PARTITION1:\^.*]],
  // CHECK-NEXT: 2: [[WS0_PARTITION2:\^.*]],
  // CHECK-NEXT: 3: [[WS1_PARTITION0:\^.*]],
  // CHECK-NEXT: 4: [[WS1_PARTITION1:\^.*]],
  // CHECK-NEXT: 5: [[WS3_PARTITION0:\^.*]],
  // CHECK-NEXT: 6: [[EXIT:\^.*]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 613-624
```mlir
  // CHECK: [[WS0_PARTITION0]]:
  // CHECK: "ws0_partition0"
  // CHECK: [[WS0_PARTITION1]]:
  // CHECK: "ws0_partition1"
  // CHECK: [[WS0_PARTITION2]]:
  // CHECK: "ws0_partition2"
  // CHECK: [[WS1_PARTITION0]]:
  // CHECK: "ws1_partition0"
  // CHECK: [[WS1_PARTITION1]]:
  // CHECK: "ws1_partition1"
  // CHECK: [[WS3_PARTITION0]]:
  // CHECK: "ws3_partition0"
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 626-643
```mlir
  // CHECK: getelementptr
  // CHECK-NEXT: llvm.store [[C0_i8]], [[SMEM_BASE:%[0-9]+]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][1]
  // CHECK-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[2]
  // CHECK-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[3]
  // CHECK-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[4]
  // CHECK-NEXT: llvm.store [[C1_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[5]
  // CHECK-NEXT: llvm.store [[C1_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[6]
  // CHECK-NEXT: llvm.store [[C2_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[7]
  // CHECK-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK: "ws0_default"
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 645-652
```mlir
  // AMD: llvm.switch
  // AMD-NEXT: 0: [[WS0_PARTITION0:\^bb[0-9]+]],
  // AMD-NEXT: 1: [[WS0_PARTITION1:\^bb[0-9]+]],
  // AMD-NEXT: 2: [[WS0_PARTITION2:\^bb[0-9]+]],
  // AMD-NEXT: 3: [[WS1_PARTITION0:\^bb[0-9]+]],
  // AMD-NEXT: 4: [[WS1_PARTITION1:\^bb[0-9]+]],
  // AMD-NEXT: 5: [[WS3_PARTITION0:\^bb[0-9]+]],
  // AMD-NEXT: 6: [[EXIT:\^bb[0-9]+]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: llvm.switch`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: llvm.switch`。

### Lines 654-665
```mlir
  // AMD: [[WS0_PARTITION0]]:
  // AMD: "ws0_partition0"
  // AMD: [[WS0_PARTITION1]]:
  // AMD: "ws0_partition1"
  // AMD: [[WS0_PARTITION2]]:
  // AMD: "ws0_partition2"
  // AMD: [[WS1_PARTITION0]]:
  // AMD: "ws1_partition0"
  // AMD: [[WS1_PARTITION1]]:
  // AMD: "ws1_partition1"
  // AMD: [[WS3_PARTITION0]]:
  // AMD: "ws3_partition0"
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: [[WS0_PARTITION0]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: [[WS0_PARTITION0]]:`。

### Lines 667-684
```mlir
  // AMD: getelementptr
  // AMD-NEXT: llvm.store [[C0_i8]], [[SMEM_BASE:%[0-9]+]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][1]
  // AMD-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[2]
  // AMD-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[3]
  // AMD-NEXT: llvm.store [[C0_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[4]
  // AMD-NEXT: llvm.store [[C1_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[5]
  // AMD-NEXT: llvm.store [[C1_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[6]
  // AMD-NEXT: llvm.store [[C2_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[7]
  // AMD-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // AMD: rocdl.barrier
  // AMD: "ws0_default"
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: getelementptr`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: getelementptr`。

### Lines 686-702
```mlir
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4, 8, 10>}
  default {
    "ws0_default"() : () -> ()
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    "ws0_partition0"() : () -> ()
    ttg.warp_return
  }
  partition1() num_warps(2) {
    "ws0_partition1"() : () -> ()
    ttg.warp_return
  }
  partition2() num_warps(1) {
    "ws0_partition2"() : () -> ()
    ttg.warp_return
  } : () -> ()
```
**EN:** This block contributes intermediate IR built from `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 704-721
```mlir
  // CHECK: getelementptr
  // CHECK-NEXT: llvm.store [[C4_i8]], [[SMEM_BASE:%[0-9]+]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][1]
  // CHECK-NEXT: llvm.store [[C4_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[2]
  // CHECK-NEXT: llvm.store [[C4_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[3]
  // CHECK-NEXT: llvm.store [[C4_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[4]
  // CHECK-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[5]
  // CHECK-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[6]
  // CHECK-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[7]
  // CHECK-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK: "ws1_default"
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 723-740
```mlir
  // AMD: getelementptr
  // AMD-NEXT: llvm.store [[C4_i8]], [[SMEM_BASE:%[0-9]+]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][1]
  // AMD-NEXT: llvm.store [[C4_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[2]
  // AMD-NEXT: llvm.store [[C4_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[3]
  // AMD-NEXT: llvm.store [[C4_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[4]
  // AMD-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[5]
  // AMD-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[6]
  // AMD-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[7]
  // AMD-NEXT: llvm.store [[C3_i8]], [[PTR]]
  // AMD: rocdl.barrier
  // AMD: "ws1_default"
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: getelementptr`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: getelementptr`。

### Lines 742-754
```mlir
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 8, 4>}
  default {
    "ws1_default"() : () -> ()
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    "ws1_partition0"() : () -> ()
    ttg.warp_return
  }
  partition1() num_warps(4) {
    "ws1_partition1"() : () -> ()
    ttg.warp_return
  } : () -> ()
```
**EN:** This block contributes intermediate IR built from `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 756-773
```mlir
  // CHECK: getelementptr
  // CHECK-NEXT: llvm.store [[Cn1_i8]], [[SMEM_BASE:%[0-9]+]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[1]
  // CHECK-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[2]
  // CHECK-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[3]
  // CHECK-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[4]
  // CHECK-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[5]
  // CHECK-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[6]
  // CHECK-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[7]
  // CHECK-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK: "ws2_default"
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 775-792
```mlir
  // AMD: getelementptr
  // AMD-NEXT: llvm.store [[Cn1_i8]], [[SMEM_BASE:%[0-9]+]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[1]
  // AMD-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[2]
  // AMD-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[3]
  // AMD-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[4]
  // AMD-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[5]
  // AMD-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[6]
  // AMD-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[7]
  // AMD-NEXT: llvm.store [[Cn1_i8]], [[PTR]]
  // AMD: rocdl.barrier
  // AMD: "ws2_default"
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: getelementptr`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: getelementptr`。

### Lines 794-798
```mlir
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32>}
  default {
    "ws2_default"() : () -> ()
    ttg.warp_yield
  } : () -> ()
```
**EN:** This block contributes intermediate IR built from `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 800-817
```mlir
  // CHECK: getelementptr
  // CHECK-NEXT: llvm.store [[C5_i8]], [[SMEM_BASE:%[0-9]+]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][1]
  // CHECK-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[2]
  // CHECK-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[3]
  // CHECK-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[4]
  // CHECK-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[5]
  // CHECK-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[6]
  // CHECK-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[7]
  // CHECK-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK: "ws3_default"
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 819-836
```mlir
  // AMD: getelementptr
  // AMD-NEXT: llvm.store [[C5_i8]], [[SMEM_BASE:%[0-9]+]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM_BASE]][1]
  // AMD-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[2]
  // AMD-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[3]
  // AMD-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[4]
  // AMD-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[5]
  // AMD-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[6]
  // AMD-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr %{{[0-9]+}}[7]
  // AMD-NEXT: llvm.store [[C5_i8]], [[PTR]]
  // AMD: rocdl.barrier
  // AMD: "ws3_default"
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: getelementptr`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: getelementptr`。

### Lines 838-848
```mlir
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4>}
  default {
    "ws3_default"() : () -> ()
    ttg.warp_yield
  }
  partition0() num_warps(8) {
    "ws3_partition0"() : () -> ()
    ttg.warp_return
  }: () -> ()
  llvm.return
}
```
**EN:** This block contributes intermediate IR built from `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 850
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 852
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 854
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 856
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 858-860
```mlir
// COMMON-LABEL: @cfg
llvm.func @cfg() attributes {allocation.offset = 32 : i32} {
  // CHECK-DAG: [[C1:%.*]] = llvm.mlir.constant(1 : i32)
```
**EN:** This block defines `cfg` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.constant`. Embedded check comments (CHECK-DAG×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `cfg`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 862-865
```mlir
  // COMMON: [[SWITCH_LOOP:\^bb1]]:
  // COMMON: llvm.switch
  // COMMON-NEXT: 0: [[PARTITION:\^.*]],
  // COMMON-NEXT: 1: [[EXIT:\^.*]]
```
**EN:** This comment block provides context for the surrounding test logic: `// COMMON: [[SWITCH_LOOP:\^bb1]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// COMMON: [[SWITCH_LOOP:\^bb1]]:`。

### Lines 867-877
```mlir
  // CHECK: [[PARTITION]]:
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "something"()[[[A:\^.*]], [[B:\^.*]]]
  // CHECK: [[A]]:
  // CHECK-NEXT: "A"
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: llvm.br [[SWITCH_LOOP]]
  // CHECK: [[B]]:
  // CHECK-NEXT: "B"
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: llvm.br [[SWITCH_LOOP]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 879-891
```mlir
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK: llvm.br [[DEFAULT:\^.*]]
  // CHECK: [[DEFAULT]]:
  // CHECK-NEXT: "something"()[[[A:\^.*]], [[B:\^.*]]]
  // CHECK: [[A]]:
  // CHECK-NEXT: "A"
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: llvm.br [[AFTER:\^.*]]
  // CHECK: [[B]]:
  // CHECK-NEXT: "B"
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: llvm.br [[AFTER]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 893-903
```mlir
  // AMD: [[PARTITION]]:
  // AMD: rocdl.barrier
  // AMD-NEXT: "something"()[[[A:\^bb[0-9]+]], [[B:\^bb[0-9]+]]]
  // AMD: [[A]]:
  // AMD-NEXT: "A"
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: llvm.br [[SWITCH_LOOP]]
  // AMD: [[B]]:
  // AMD-NEXT: "B"
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: llvm.br [[SWITCH_LOOP]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: [[PARTITION]]:`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: [[PARTITION]]:`。

### Lines 905-917
```mlir
  // AMD: rocdl.barrier
  // AMD-NEXT: rocdl.barrier
  // AMD: llvm.br [[DEFAULT:\^bb[0-9]+]]
  // AMD: [[DEFAULT]]:
  // AMD-NEXT: "something"()[[[A:\^bb[0-9]+]], [[B:\^bb[0-9]+]]]
  // AMD: [[A]]:
  // AMD-NEXT: "A"
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: llvm.br [[AFTER:\^bb[0-9]+]]
  // AMD: [[B]]:
  // AMD-NEXT: "B"
  // AMD-NEXT: rocdl.barrier
  // AMD-NEXT: llvm.br [[AFTER]]
```
**EN:** This comment block provides context for the surrounding test logic: `// AMD: rocdl.barrier`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// AMD: rocdl.barrier`。

### Lines 919-939
```mlir
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4>}
  default {
    "something"()[^A, ^B] : () -> ()
  ^A:
   "A"() : () -> ()
    ttg.warp_yield
  ^B:
   "B"() : () -> ()
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    "something"()[^A, ^B] : () -> ()
  ^A:
   "A"() : () -> ()
    ttg.warp_return
  ^B:
   "B"() : () -> ()
    ttg.warp_return
  } : () -> ()
  llvm.return
}
```
**EN:** This block contributes intermediate IR built from `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 941
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 943
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 945
```mlir
module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 947
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 949-959
```mlir
// COMMON-LABEL: @no_captures
llvm.func @no_captures() attributes {allocation.offset = 0 : i32} {
  ttg.warp_specialize() attributes {warpGroupStartIds = array<i32: 4>}
  default {
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    ttg.warp_return
  } : () -> ()
  llvm.return
}
```
**EN:** This block defines `no_captures` and exercises operations such as `llvm.func`, `allocation.offset`, `ttg.warp_specialize`, `ttg.warp_yield`, `ttg.warp_return`.
**CN:** 这一块定义了 `no_captures`，并覆盖 如 `llvm.func`、`allocation.offset`、`ttg.warp_specialize`、`ttg.warp_yield`、`ttg.warp_return` 这样的操作。

### Line 961
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 963
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 965
```mlir
module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.total-num-warps" = 6 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.total-num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.total-num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 967
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 969-991
```mlir
// COMMON-LABEL: @type_conversion_results
// COMMON-NOT: !tt.ptr<i32>
// COMMON-NOT: unrealized_conversion_cast
llvm.func @type_conversion_results() attributes {allocation.offset = 0 : i32} {
  // COMMON: [[CAP:%.*]] = "produce"
  %cap = "produce"() : () -> !llvm.ptr<1>
  %0 = builtin.unrealized_conversion_cast %cap : !llvm.ptr<1> to !tt.ptr<i32>
  %1 = ttg.warp_specialize(%0) attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4>}
  default {
    // COMMON: llvm.br [[AFTER:\^.*]]([[CAP]] : !llvm.ptr<1>)
    ttg.warp_yield %0 : !tt.ptr<i32>
  }
  partition0(%arg1: !tt.ptr<i32>) num_warps(2) {
    %3 = builtin.unrealized_conversion_cast %arg1 : !tt.ptr<i32> to !llvm.ptr<1>
    %4 = llvm.load %3 : !llvm.ptr<1> -> i32
    ttg.warp_return
  } : (!tt.ptr<i32>) -> !tt.ptr<i32>
  // COMMON: [[AFTER]]([[OUT:%.*]]: !llvm.ptr<1>):
  %2 = builtin.unrealized_conversion_cast %1 : !tt.ptr<i32> to !llvm.ptr<1>
  // COMMON-NEXT: "use"([[OUT]])
  "use"(%2) : (!llvm.ptr<1>) -> ()
  llvm.return
}
```
**EN:** This block defines `type_conversion_results` and exercises operations such as `tt.ptr`, `llvm.func`, `allocation.offset`, `llvm.ptr`, `builtin.unrealized_conversion_cast`.
**CN:** 这一块定义了 `type_conversion_results`，并覆盖 如 `tt.ptr`、`llvm.func`、`allocation.offset`、`llvm.ptr`、`builtin.unrealized_conversion_cast` 这样的操作。

### Line 993
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 995
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 997
```mlir
module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.total-num-warps" = 6 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.total-num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.total-num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 999
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 1001-1013
```mlir
// COMMON-LABEL: @capture_function_arg
llvm.func @capture_function_arg(%arg0: i32) attributes {allocation.offset = 0 : i32} {
  ttg.warp_specialize(%arg0) attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4>}
  default {
    ttg.warp_yield
  }
  partition0(%arg1: i32) num_warps(1) {
    // COMMON: "use"(%arg0)
    "use"(%arg1) : (i32) -> ()
    ttg.warp_return
  } : (i32) -> ()
  llvm.return
}
```
**EN:** This block defines `capture_function_arg` and exercises operations such as `llvm.func`, `allocation.offset`, `ttg.warp_specialize`, `ttg.warp_yield`, `ttg.warp_return`.
**CN:** 这一块定义了 `capture_function_arg`，并覆盖 如 `llvm.func`、`allocation.offset`、`ttg.warp_specialize`、`ttg.warp_yield`、`ttg.warp_return` 这样的操作。

### Lines 1015-1029
```mlir
// COMMON-LABEL: @type_conversion_func_arg
llvm.func @type_conversion_func_arg(%arg0: !llvm.ptr<1>) attributes {allocation.offset = 0 : i32} {
  %0 = builtin.unrealized_conversion_cast %arg0 : !llvm.ptr<1> to !tt.ptr<i32>
  ttg.warp_specialize(%0) attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4>}
  default {
    ttg.warp_yield
  }
  partition0(%arg1: !tt.ptr<i32>) num_warps(1) {
    %1 = builtin.unrealized_conversion_cast %arg1 : !tt.ptr<i32> to !llvm.ptr<1>
    // COMMON: "use"(%arg0)
    "use"(%1) : (!llvm.ptr<1>) -> ()
    ttg.warp_return
  } : (!tt.ptr<i32>) -> ()
  llvm.return
}
```
**EN:** This block defines `type_conversion_func_arg` and exercises operations such as `llvm.func`, `llvm.ptr`, `allocation.offset`, `builtin.unrealized_conversion_cast`, `tt.ptr`.
**CN:** 这一块定义了 `type_conversion_func_arg`，并覆盖 如 `llvm.func`、`llvm.ptr`、`allocation.offset`、`builtin.unrealized_conversion_cast`、`tt.ptr` 这样的操作。

### Lines 1031-1035
```mlir
// COMMON-LABEL: @trivial_remat
llvm.func @trivial_remat() attributes {allocation.offset = 0 : i32} {
  // CHECK-DAG: [[C1:%.*]] = llvm.mlir.constant(1 : i32)
  // COMMON-DAG: [[CAP0:%.*]] = llvm.mlir.constant(0 : i32)
  // COMMON-DAG: [[CAP1:%.*]] = llvm.mlir.addressof @global_smem : !llvm.ptr<3>
```
**EN:** This block defines `trivial_remat` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.constant`, `llvm.mlir.addressof`, `llvm.ptr`. Embedded check comments (CHECK-DAG×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `trivial_remat`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.constant`、`llvm.mlir.addressof`、`llvm.ptr` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 1037-1056
```mlir
  %0 = llvm.mlir.constant(0 : i32) : i32
  %1 = llvm.mlir.addressof @global_smem : !llvm.ptr<3>
  ttg.warp_specialize(%0, %1) attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4>}
  default {
    ttg.warp_yield
  }
  partition0(%arg0: i32, %arg1: !llvm.ptr<3>) num_warps(1) {
  // CHECK: ^bb4:
    // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
    // CHECK-NEXT: "use"([[CAP0]], [[CAP1]])
  // AMD: ^bb4:
    // AMD-NEXT: rocdl.barrier
    // AMD-NEXT: "use"([[CAP0]], [[CAP1]])
    "use"(%arg0, %arg1) : (i32, !llvm.ptr<3>) -> ()
    // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
    // AMD-NEXT: rocdl.barrier
    ttg.warp_return
  } : (i32, !llvm.ptr<3>) -> ()
  llvm.return
}
```
**EN:** This block defines `global_smem` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×1, CHECK-NEXT×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `global_smem`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 1058-1061
```mlir
// COMMON-LABEL: @remat_subgraph
llvm.func @remat_subgraph(%arg0: i32, %arg1: i32) attributes {allocation.offset = 0 : i32} {
  // CHECK-DAG: [[C1:%.*]] = llvm.mlir.constant(1 : i32)
  // COMMON-DAG: [[ADDR:%.*]] = llvm.mlir.addressof @global_smem : !llvm.ptr<3>
```
**EN:** This block defines `remat_subgraph` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.constant`, `llvm.mlir.addressof`, `llvm.ptr`. Embedded check comments (CHECK-DAG×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `remat_subgraph`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.constant`、`llvm.mlir.addressof`、`llvm.ptr` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 1063-1093
```mlir
  %0 = llvm.mlir.addressof @global_smem : !llvm.ptr<3>
  %1 = llvm.getelementptr %0[%arg0] : (!llvm.ptr<3>, i32) -> !llvm.ptr<3>, i32
  %2 = llvm.add %arg0, %arg1 : i32
  %3 = llvm.mul %2, %arg1 : i32
  %4 = llvm.urem %2, %3 : i32
  ttg.warp_specialize(%1, %4) attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4>}
  default {
    ttg.warp_yield
  }
  partition0(%arg2: !llvm.ptr<3>, %arg3: i32) num_warps(1) {
  // CHECK: ^bb4:
    // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
    // CHECK-NEXT: [[ADD:%.*]] = llvm.add %arg0, %arg1 : i32
    // CHECK-NEXT: [[MUL:%.*]] = llvm.mul [[ADD]], %arg1 : i32
    // CHECK-NEXT: [[UREM:%.*]] = llvm.urem [[ADD]], [[MUL]] : i32
    // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[ADDR]][%arg0]
    // CHECK-NEXT: "use"([[PTR]], [[UREM]])
  // AMD: ^bb4:
    // AMD-NEXT: rocdl.barrier
    // AMD-NEXT: [[ADD:%.*]] = llvm.add %arg0, %arg1 : i32
    // AMD-NEXT: [[MUL:%.*]] = llvm.mul [[ADD]], %arg1 : i32
    // AMD-NEXT: [[UREM:%.*]] = llvm.urem [[ADD]], [[MUL]] : i32
    // AMD-NEXT: [[PTR:%.*]] = llvm.getelementptr [[ADDR]][%arg0]
    // AMD-NEXT: "use"([[PTR]], [[UREM]])
    "use"(%arg2, %arg3) : (!llvm.ptr<3>, i32) -> ()
    // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
    // AMD-NEXT: rocdl.barrier
    ttg.warp_return
  } : (!llvm.ptr<3>, i32) -> ()
  llvm.return
}
```
**EN:** This block defines `global_smem` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×1, CHECK-NEXT×7) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `global_smem`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×1, CHECK-NEXT×7）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1095
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 1097
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 1099
```mlir
module attributes {ttg.maxnreg = 80 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.total-num-warps" = 16 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.total-num-warps`, `ttg.maxnreg`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.total-num-warps`, `ttg.maxnreg`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 1101
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 1103-1105
```mlir
// CHECK-LABEL: @dynamic_register_reallocation
llvm.func @dynamic_register_reallocation() attributes {allocation.offset = 0 : i32} {
  // CHECK-DAG: [[C1:%.*]] = llvm.mlir.constant(1 : i32)
```
**EN:** This block defines `dynamic_register_reallocation` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.constant`. Embedded check comments (CHECK-DAG×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `dynamic_register_reallocation`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1107
```mlir
  // CHECK: cond_br %{{.*}}, [[ENTRY:\^.*]], [[SWITCH_LOOP:\^.*]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 1109-1116
```mlir
  // CHECK: [[SWITCH_LOOP]]:
  // CHECK-NEXT: nvvm.setmaxregister decrease 24
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK: llvm.switch
  // CHECK-NEXT: 0: [[PARTITION0:\^.*]],
  // CHECK-NEXT: 1: [[PARTITION1:\^.*]],
  // CHECK-NEXT: 2: [[PARTITION2:\^.*]],
  // CHECK-NEXT: 3: [[EXIT:\^.*]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1118-1123
```mlir
  // CHECK: [[PARTITION0]]:
  // CHECK-NEXT: nvvm.setmaxregister increase 80
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "partition0"()
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: nvvm.setmaxregister decrease 24
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1125-1130
```mlir
  // CHECK: [[PARTITION1]]:
  // CHECK-NEXT: nvvm.setmaxregister increase 48
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "partition1"()
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: nvvm.setmaxregister decrease 24
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1132-1137
```mlir
  // CHECK: [[PARTITION2]]:
  // CHECK-NEXT: nvvm.setmaxregister increase 128
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "partition2"()
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: nvvm.setmaxregister decrease 24
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1139-1140
```mlir
  // CHECK: [[ENTRY]]:
  // CHECK-NEXT: nvvm.setmaxregister increase 248
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1142-1147
```mlir
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: setmaxregister decrease 152
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK: "default"
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: setmaxregister increase 248
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1149-1167
```mlir
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4, 8, 12>, actualRegisters = array<i32: 152, 80, 48, 128>}
  default {
    "default"() : () -> ()
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    "partition0"() : () -> ()
    ttg.warp_return
  }
  partition1() num_warps(4) {
    "partition1"() : () -> ()
    ttg.warp_return
  }
  partition2() num_warps(4) {
    "partition2"() : () -> ()
    ttg.warp_return
  } : () -> ()
  llvm.return
}
```
**EN:** This block contributes intermediate IR built from `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1169
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 1171
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 1173
```mlir
module attributes {ttg.maxnreg = 128 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.total-num-warps" = 16 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.total-num-warps`, `ttg.maxnreg`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.total-num-warps`, `ttg.maxnreg`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 1175
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 1177-1179
```mlir
// CHECK-LABEL: @dynamic_register_reallocation
llvm.func @dynamic_register_reallocation_overalloc() attributes {allocation.offset = 0 : i32} {
  // CHECK-DAG: [[C1:%.*]] = llvm.mlir.constant(1 : i32)
```
**EN:** This block defines `dynamic_register_reallocation_overalloc` and exercises operations such as `llvm.func`, `allocation.offset`, `llvm.mlir.constant`. Embedded check comments (CHECK-DAG×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `dynamic_register_reallocation_overalloc`，并覆盖 如 `llvm.func`、`allocation.offset`、`llvm.mlir.constant` 这样的操作。 其中嵌入的检查注释（CHECK-DAG×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1181
```mlir
  // CHECK: cond_br %{{.*}}, [[ENTRY:\^.*]], [[SWITCH_LOOP:\^.*]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 1183-1190
```mlir
  // CHECK: [[SWITCH_LOOP]]:
  // CHECK-NEXT: nvvm.setmaxregister decrease 80
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK: llvm.switch
  // CHECK-NEXT: 0: [[PARTITION0:\^.*]],
  // CHECK-NEXT: 1: [[PARTITION1:\^.*]],
  // CHECK-NEXT: 2: [[PARTITION2:\^.*]],
  // CHECK-NEXT: 3: [[EXIT:\^.*]]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1192-1197
```mlir
  // CHECK: [[PARTITION0]]:
  // CHECK-NEXT: nvvm.setmaxregister decrease 24
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "partition0"()
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: nvvm.setmaxregister increase 80
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1199-1204
```mlir
  // CHECK: [[PARTITION1]]:
  // CHECK-NEXT: nvvm.setmaxregister increase 192
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "partition1"()
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: nvvm.setmaxregister decrease 80
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1206-1211
```mlir
  // CHECK: [[PARTITION2]]:
  // CHECK-NEXT: nvvm.setmaxregister increase 192
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: "partition2"()
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: nvvm.setmaxregister decrease 80
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1213-1214
```mlir
  // CHECK: [[ENTRY]]:
  // CHECK-NEXT: nvvm.setmaxregister increase 256
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1216-1221
```mlir
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: setmaxregister decrease 104
  // CHECK-NEXT: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK: "default"
  // CHECK: "llvm.nvvm.barrier.cta.sync.all"([[C1]])
  // CHECK-NEXT: setmaxregister increase 256
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Lines 1223-1241
```mlir
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4, 8, 12>, actualRegisters = array<i32: 104, 24, 192, 192>}
  default {
    "default"() : () -> ()
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    "partition0"() : () -> ()
    ttg.warp_return
  }
  partition1() num_warps(4) {
    "partition1"() : () -> ()
    ttg.warp_return
  }
  partition2() num_warps(4) {
    "partition2"() : () -> ()
    ttg.warp_return
  } : () -> ()
  llvm.return
}
```
**EN:** This block contributes intermediate IR built from `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.warp_specialize`, `allocation.offset`, `ttg.warp_yield`, `ttg.warp_return`, `llvm.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1243
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 1245
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 1247
```mlir
module attributes {ttg.maxnreg = 80 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100", "ttg.total-num-warps" = 8 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-warps`, `ttg.total-num-warps`, `ttg.maxnreg`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-warps`, `ttg.total-num-warps`, `ttg.maxnreg`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Line 1249
```mlir
llvm.mlir.global external @global_smem() {addr_space = 3 : i32, alignment = 16 : i64} : !llvm.array<0 x i8>
```
**EN:** This block contributes intermediate IR built from `llvm.mlir.global`, `llvm.array`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.mlir.global`, `llvm.array` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 1251-1265
```mlir
// CHECK-LABEL: @gsan_dynamic_register_reallocation_disabled
llvm.func @gsan_dynamic_register_reallocation_disabled() attributes {allocation.offset = 0 : i32} {
  // CHECK-NOT: nvvm.setmaxregister
  // CHECK: llvm.return
  ttg.warp_specialize() attributes {allocation.offset = 0 : i32, warpGroupStartIds = array<i32: 4>, actualRegisters = array<i32: 152, 80>, "tti.disable_setmaxregister"}
  default {
    "default"() : () -> ()
    ttg.warp_yield
  }
  partition0() num_warps(4) {
    "partition0"() : () -> ()
    ttg.warp_return
  } : () -> ()
  llvm.return
}
```
**EN:** This block defines `gsan_dynamic_register_reallocation_disabled` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `gsan_dynamic_register_reallocation_disabled`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 1267
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on warp-level behavior.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 warp 级行为。
- **EN:** The `RUN` pipeline drives tools/passes such as `-mlir-print-local-scope`, `-allow-unregistered-dialect`, `-convert-warp-specialize-to-llvm`, `-canonicalize=region-simplify=disabled`, `--check-prefixes=COMMON,CHECK`, `-triton-amdgpu-convert-warp-specialize-to-llvm=gfx-arch=gfx1250`, `--check-prefixes=COMMON,AMD`.  
  **CN:** `RUN` 流水线会驱动 `-mlir-print-local-scope`, `-allow-unregistered-dialect`, `-convert-warp-specialize-to-llvm`, `-canonicalize=region-simplify=disabled`, `--check-prefixes=COMMON,CHECK`, `-triton-amdgpu-convert-warp-specialize-to-llvm=gfx-arch=gfx1250`, `--check-prefixes=COMMON,AMD` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `llvm` (LLVM dialect IR), `nvvm` (NVVM backend intrinsics), `tt` (Triton core ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`llvm`（LLVM 方言 IR）、`nvvm`（NVVM 后端内建）、`tt`（Triton 核心操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `llvm`, `nvvm`, `tt`.  
  **CN:** IR 方言依赖：`ttg`、`llvm`、`nvvm`、`tt`。
