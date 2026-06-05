# atomic_cas.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/atomic_cas.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises atomic operations in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的原子操作相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file -convert-triton-amdgpu-to-llvm="gfx-arch=gfx942" -cse | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×16, CHECK-DAG×4, CHECK-LABEL×5; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×16, CHECK-DAG×4, CHECK-LABEL×5；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that atomic operations produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 原子操作 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file -convert-triton-amdgpu-to-llvm="gfx-arch=gfx942" -cse | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file -convert-triton-amdgpu-to-llvm="gfx-arch=gfx942" -cse | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file -convert-triton-amdgpu-to-llvm="gfx-arch=gfx942" -cse | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-14
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @atomic_cas_0(%arg3: !tt.ptr<i32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    // CHECK-LABEL: @atomic_cas_0
    %c64_i32 = arith.constant 64 : i32
    %c32_i32 = arith.constant 32 : i32
    // CHECK: %[[C64:.*]] = llvm.mlir.constant(64 : i32) : i32
    // CHECK: %[[C32:.*]] = llvm.mlir.constant(32 : i32) : i32
    // CHECK: llvm.cmpxchg %{{.*}}, %[[C32]], %[[C64]] syncscope("agent") acquire monotonic
    %0 = tt.atomic_cas acquire, gpu, %arg3, %c32_i32, %c64_i32 : (!tt.ptr<i32>, i32, i32) -> i32
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_cas_0`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_cas_0`。

### Line 16
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 18-29
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @atomic_cas_1(%arg3: !tt.ptr<i32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    // CHECK-LABEL: @atomic_cas_1
    %c64_i32 = arith.constant 64 : i32
    %c32_i32 = arith.constant 32 : i32
    // CHECK: %[[C64:.*]] = llvm.mlir.constant(64 : i32) : i32
    // CHECK: %[[C32:.*]] = llvm.mlir.constant(32 : i32) : i32
    // CHECK: llvm.cmpxchg %{{.*}}, %[[C32]], %[[C64]] syncscope("agent") monotonic monotonic
    %0 = tt.atomic_cas relaxed, gpu, %arg3, %c32_i32, %c64_i32 : (!tt.ptr<i32>, i32, i32) -> i32
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_cas_1`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_cas_1`。

### Line 31
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 33-44
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @atomic_cas_2(%arg3: !tt.ptr<i32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    // CHECK-LABEL: @atomic_cas_2
    %c64_i32 = arith.constant 64 : i32
    %c32_i32 = arith.constant 32 : i32
    // CHECK: %[[C64:.*]] = llvm.mlir.constant(64 : i32) : i32
    // CHECK: %[[C32:.*]] = llvm.mlir.constant(32 : i32) : i32
    // CHECK: llvm.cmpxchg %{{.*}}, %[[C32]], %[[C64]] syncscope("agent") acq_rel monotonic
    %0 = tt.atomic_cas acq_rel, gpu, %arg3, %c32_i32, %c64_i32 : (!tt.ptr<i32>, i32, i32) -> i32
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_cas_2`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_cas_2`。

### Line 46
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 48-59
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @atomic_cas_3(%arg3: !tt.ptr<i32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    // CHECK-LABEL: @atomic_cas_3
    %c64_i32 = arith.constant 64 : i32
    %c32_i32 = arith.constant 32 : i32
    // CHECK: %[[C64:.*]] = llvm.mlir.constant(64 : i32) : i32
    // CHECK: %[[C32:.*]] = llvm.mlir.constant(32 : i32) : i32
    // CHECK: llvm.cmpxchg %{{.*}}, %[[C32]], %[[C64]] acquire monotonic
    %0 = tt.atomic_cas acquire, sys, %arg3, %c32_i32, %c64_i32 : (!tt.ptr<i32>, i32, i32) -> i32
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_cas_3`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_cas_3`。

### Line 61
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 63-80
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx942", "ttg.threads-per-warp" = 64 : i32} {
  tt.func public @atomic_cas_f32(%arg3: !tt.ptr<f32> {tt.divisibility = 16 : i32}) attributes {noinline = false} {
    // CHECK-LABEL: @atomic_cas_f32
    %c64_f32 = arith.constant 64. : f32
    %c32_f32 = arith.constant 32. : f32
    // CHECK-DAG: %[[C64:.*]] = llvm.mlir.constant(6.400000e+01 : f32) : f32
    // CHECK-DAG: %[[C32:.*]] = llvm.mlir.constant(3.200000e+01 : f32) : f32
    // CHECK-DAG: %[[C64I:.*]] = llvm.bitcast %[[C64]] : f32 to i32
    // CHECK-DAG: %[[C32I:.*]] = llvm.bitcast %[[C32]] : f32 to i32
    // CHECK: %[[CMPXCHG:.*]] = llvm.cmpxchg %{{.*}}, %[[C32I]], %[[C64I]] acquire monotonic
    // CHECK: %[[RESI:.*]] = llvm.extractvalue %[[CMPXCHG]][0] : !llvm.struct<(i32, i1)>
    // CHECK: %[[RES:.*]] = llvm.bitcast %[[RESI]] : i32 to f32
    // CHECK: llvm.store %[[RES]], %{{.*}} : f32, !llvm.ptr<3>
    %0 = tt.atomic_cas acquire, sys, %arg3, %c32_f32, %c64_f32 { allocation.offset = 0 : i32 }: (!tt.ptr<f32>, f32, f32) -> f32
    tt.print "some print" {hex = false, isSigned = array<i32: 0>} : %0: f32
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx942`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_cas_f32`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx942`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_cas_f32`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on atomic operations.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 原子操作。
- **EN:** The `RUN` pipeline drives tools/passes such as `-convert-triton-amdgpu-to-llvm="gfx-arch=gfx942"`, `-cse`.  
  **CN:** `RUN` 流水线会驱动 `-convert-triton-amdgpu-to-llvm="gfx-arch=gfx942"`, `-cse` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `llvm` (LLVM dialect IR).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`llvm`（LLVM 方言 IR）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `llvm`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`llvm`。
