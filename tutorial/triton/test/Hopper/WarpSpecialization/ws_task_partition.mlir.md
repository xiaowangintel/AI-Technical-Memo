# ws_task_partition.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Hopper/WarpSpecialization/ws_task_partition.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises warp-level behavior in Triton's Hopper warp-specialization coverage. **CN:** 该MLIR 测试用于覆盖 Triton Hopper warp-specialization中的warp 级行为相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --nvgpu-test-ws-task-partition=num-warp-groups=3 | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×6, CHECK-LABEL×1; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×6, CHECK-LABEL×1；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that warp-level behavior produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 warp 级行为 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --nvgpu-test-ws-task-partition=num-warp-groups=3 | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --nvgpu-test-ws-task-partition=num-warp-groups=3 | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --nvgpu-test-ws-task-partition=num-warp-groups=3 | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-9
```mlir
// CHECK-LABEL: @matmul_persistent_tma_ws_cooperative_kernel
// CHECK: %[[#GA:]] = tt.descriptor_load {{.*}} {async_task_id = array<i32: 0>}
// CHECK: %[[#LA:]] = ttg.local_alloc %[[#GA]]
// CHECK: %[[#GB:]] = tt.descriptor_load {{.*}} {async_task_id = array<i32: 0>}
// CHECK: %[[#LB:]] = ttg.local_alloc %[[#GB]]
// CHECK: %[[#C:]] = ttng.warp_group_dot %[[#LA]], %[[#LB]], {{.*}} {async_task_id = array<i32: 1, 2>
// CHECK: tt.descriptor_store {{.*}} {async_task_id = array<i32: 1, 2>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-LABEL`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-LABEL`），从而保证后端/代码生成结果稳定。

### Lines 11-40
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [2, 2], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 256, 16]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @matmul_persistent_tma_ws_cooperative_kernel(%arg0: !tt.tensordesc<128x64xf16>, %arg1: !tt.tensordesc<64x256xf16>, %arg2: !tt.tensordesc<128x256xf16>, %arg3: i32 {tt.divisibility = 16 : i32}, %arg4: i32 {tt.divisibility = 16 : i32}, %arg5: i32 {tt.divisibility = 16 : i32}) {
    %c0_i32 = arith.constant 0 : i32
    %c1_i32 = arith.constant 1 : i32
    %c64_i32 = arith.constant 64 : i32
    %cst = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #mma>
    %0 = tt.get_program_id x : i32
    %1 = tt.get_num_programs x : i32
    scf.for %arg6 = %0 to %arg3 step %1  : i32 {
      %2:2 = scf.for %arg7 = %c0_i32 to %arg5 step %c1_i32 iter_args(%arg8 = %cst, %arg9 = %c0_i32) -> (tensor<128x256xf32, #mma>, i32)  : i32 {
        %5 = tt.descriptor_load %arg0[%arg6, %arg9] : !tt.tensordesc<128x64xf16> -> tensor<128x64xf16, #blocked>
        %6 = ttg.local_alloc %5 : (tensor<128x64xf16, #blocked>) -> !ttg.memdesc<128x64xf16, #shared, #smem>
        %7 = tt.descriptor_load %arg1[%arg9, %arg6] : !tt.tensordesc<64x256xf16> -> tensor<64x256xf16, #blocked1>
        %8 = ttg.local_alloc %7 : (tensor<64x256xf16, #blocked1>) -> !ttg.memdesc<64x256xf16, #shared, #smem>
        %9 = ttng.warp_group_dot %6, %8, %arg8 {inputPrecision = 0 : i32} : !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x256xf16, #shared, #smem> -> tensor<128x256xf32, #mma>
        %10 = arith.addi %arg9, %c64_i32 : i32
        scf.yield %9, %10 : tensor<128x256xf32, #mma>, i32
      }
      %3 = arith.truncf %2#0 : tensor<128x256xf32, #mma> to tensor<128x256xf16, #mma>
      %4 = ttg.convert_layout %3 : tensor<128x256xf16, #mma> -> tensor<128x256xf16, #blocked1>
      tt.descriptor_store %arg2[%arg6, %arg6], %4 : !tt.tensordesc<128x256xf16>, tensor<128x256xf16, #blocked1>
    }
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `matmul_persistent_tma_ws_cooperative_kernel`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `matmul_persistent_tma_ws_cooperative_kernel`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's Hopper warp-specialization coverage and focuses on warp-level behavior.  
  **CN:** 该文件属于 Triton 的 Hopper warp-specialization 测试覆盖，关注点是 warp 级行为。
- **EN:** The `RUN` pipeline drives tools/passes such as `--nvgpu-test-ws-task-partition=num-warp-groups=3`.  
  **CN:** `RUN` 流水线会驱动 `--nvgpu-test-ws-task-partition=num-warp-groups=3` 等工具/pass。
- **EN:** The IR mixes `tt` (Triton core ops), `ttg` (TritonGPU ops/layouts), `ttng` (next-gen TritonGPU ops), `arith` (scalar/tensor arithmetic), `scf` (structured control flow).  
  **CN:** 该 IR 混合使用了 `tt`（Triton 核心操作）、`ttg`（TritonGPU 操作/布局）、`ttng`（新一代 TritonGPU 操作）、`arith`（标量/张量算术）、`scf`（结构化控制流）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `tt`, `ttg`, `ttng`, `arith`, `scf`.  
  **CN:** IR 方言依赖：`tt`、`ttg`、`ttng`、`arith`、`scf`。
