# add_sched_barriers.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Proton/amd/add_sched_barriers.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises Proton dialect lowering in Triton's Proton pipeline coverage. **CN:** 该MLIR 测试用于覆盖 Triton Proton 流水线中的Proton 方言降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file -add-sched-barriers --verify-diagnostics | FileCheck --check-prefix=CHECK %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×33, CHECK-LABEL×3; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×33, CHECK-LABEL×3；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that Proton dialect lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 Proton 方言降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file -add-sched-barriers --verify-diagnostics | FileCheck --check-prefix=CHECK %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file -add-sched-barriers --verify-diagnostics | FileCheck --check-prefix=CHECK %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file -add-sched-barriers --verify-diagnostics | FileCheck --check-prefix=CHECK %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-12
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: convert_read_counter
  llvm.func @convert_read_counter() -> i32 {
    // CHECK: rocdl.sched.barrier 0
    %1 = proton_gpu.read_counter : i32
    llvm.return %1 : i32
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_read_counter`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_read_counter`。

### Line 14
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 16-79
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32, ttg.profile_scratch_memory_alignment = 128 : i32, ttg.profile_scratch_memory_size = 384 : i32} {
  // CHECK-LABEL: nested_record
  llvm.func @nested_record(%arg: !llvm.ptr<1>) attributes {noinline = false, nvvm.kernel = 1 : ui1} {
  // CHECK: proton_gpu.initialize
  // CHECK: rocdl.sched.barrier 0
  // CHECK: proton_gpu.read_counter
  // CHECK: proton_gpu.circular_store
  // CHECK: rocdl.sched.barrier 0
  // CHECK: scf.for
  // CHECK:   rocdl.sched.barrier 0
  // CHECK:   proton_gpu.read_counter
  // CHECK:   proton_gpu.circular_store
  // CHECK:   rocdl.sched.barrier 0
  // CHECK:   scf.for
  // CHECK:     rocdl.sched.barrier 0
  // CHECK:     proton_gpu.read_counter
  // CHECK:     proton_gpu.circular_store
  // CHECK:     rocdl.sched.barrier 0
  // CHECK:   }
  // CHECK:   rocdl.sched.barrier 0
  // CHECK:   proton_gpu.read_counter
  // CHECK:   proton_gpu.circular_store
  // CHECK:   rocdl.sched.barrier 0
  // CHECK: }
  // CHECK: rocdl.sched.barrier 0
  // CHECK: proton_gpu.read_counter
  // CHECK: proton_gpu.circular_store
  // CHECK: rocdl.sched.barrier 0
  // CHECK: proton_gpu.read_counter
  // CHECK: proton_gpu.circular_store
  // CHECK: rocdl.sched.barrier 0
  // CHECK: ttg.barrier local|global_read|global_write
  // CHECK: proton_gpu.finalize
  // CHECK: llvm.return
    %c4 = arith.constant 4 : index
    %c1 = arith.constant 1 : index
    %c0 = arith.constant 0 : index
    %0 = ttg.local_alloc : () -> !ttg.memdesc<512xi32, #shared, #smem, mutable>
    %1 = ttg.global_scratch_alloc {alignment = 128 : i32, third_party_allocation, nbytes = 384 : i32, ttg.global_scratch_memory_offset = 0 : i32} : !tt.ptr<i32>
    proton_gpu.initialize %1 : !tt.ptr<i32>
    %2 = proton_gpu.segment_alloc %0 : !ttg.memdesc<512xi32, #shared, #smem, mutable> -> !proton_gpu.segment<2048, #smem, warp>
    %3 = proton_gpu.read_counter : i32
    proton_gpu.circular_store start %2, %3 {scopeId = 0 : i32} : !proton_gpu.segment<2048, #smem, warp>, i32
    scf.for %arg0 = %c0 to %c4 step %c1 {
      %7 = proton_gpu.read_counter : i32
      proton_gpu.circular_store start %2, %7 {scopeId = 0 : i32} : !proton_gpu.segment<2048, #smem, warp>, i32
      scf.for %arg1 = %c0 to %c4 step %c1 {
        %9 = proton_gpu.read_counter : i32
        proton_gpu.circular_store start %2, %9 {scopeId = 0 : i32} : !proton_gpu.segment<2048, #smem, warp>, i32
      }
      %8 = proton_gpu.read_counter : i32
      proton_gpu.circular_store start %2, %8 {scopeId = 0 : i32} : !proton_gpu.segment<2048, #smem, warp>, i32
    }
    %5 = proton_gpu.read_counter : i32
    proton_gpu.circular_store start %2, %5 {scopeId = 0 : i32} : !proton_gpu.segment<2048, #smem, warp>, i32
    %6 = proton_gpu.read_counter : i32
    proton_gpu.circular_store start %2, %6 {scopeId = 0 : i32} : !proton_gpu.segment<2048, #smem, warp>, i32
    ttg.barrier local|global_read|global_write
    proton_gpu.finalize %2, %1 : !proton_gpu.segment<2048, #smem, warp>, !tt.ptr<i32>
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `nested_record`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `nested_record`。

### Line 81
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 83-90
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 3072 : i32, ttg.target = "hip:gfx90a", "ttg.threads-per-warp" = 64 : i32} {
  llvm.func @llvm.exp2.f32(f32) -> f32 attributes {libname = "", libpath = ""}
  // CHECK-LABEL: two_functions
  llvm.func @two_functions(%arg: f32) -> f32 {
    %1 = llvm.call @llvm.exp2.f32(%arg) : (f32) -> f32
    llvm.return %1 : f32
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx90a`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `llvm.exp2.f32`, `two_functions`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx90a`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `llvm.exp2.f32`, `two_functions`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's Proton pipeline coverage and focuses on Proton dialect lowering.  
  **CN:** 该文件属于 Triton 的 Proton 流水线 测试覆盖，关注点是 Proton 方言降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `-add-sched-barriers`, `--verify-diagnostics`, `--check-prefix=CHECK`.  
  **CN:** `RUN` 流水线会驱动 `-add-sched-barriers`, `--verify-diagnostics`, `--check-prefix=CHECK` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `llvm` (LLVM dialect IR), `nvvm` (NVVM backend intrinsics), `scf` (structured control flow), `arith` (scalar/tensor arithmetic), `tt` (Triton core ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`llvm`（LLVM 方言 IR）、`nvvm`（NVVM 后端内建）、`scf`（结构化控制流）、`arith`（标量/张量算术）、`tt`（Triton 核心操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `llvm`, `nvvm`, `scf`, `arith`, `tt`.  
  **CN:** IR 方言依赖：`ttg`、`llvm`、`nvvm`、`scf`、`arith`、`tt`。
