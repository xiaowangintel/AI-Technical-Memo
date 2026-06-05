# amd-block-pingpong-asyncmark-multi-token.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/TritonGPU/amd/amd-block-pingpong-asyncmark-multi-token.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises asynchronous copy/lowering in Triton's TritonGPU transforms coverage. **CN:** 该MLIR 测试用于覆盖 Triton TritonGPU 变换中的异步拷贝/降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --tritonamdgpu-block-pingpong=num-stages=3 | FileCheck %s`<br>`triton-opt %s --tritonamdgpu-block-pingpong=num-stages=3 --tritonamdgpu-update-async-wait-count=gfx-arch=gfx950 | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×2, CHECK-LABEL×1; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×2, CHECK-LABEL×1；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that asynchronous copy/lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 异步拷贝/降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s --tritonamdgpu-block-pingpong=num-stages=3 | FileCheck %s
// RUN: triton-opt %s --tritonamdgpu-block-pingpong=num-stages=3 --tritonamdgpu-update-async-wait-count=gfx-arch=gfx950 | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s --tritonamdgpu-block-pingpong=num-stages=3 | FileCheck %s; triton-opt %s --tritonamdgpu-block-pingpong=num-stages=3 --tritonamdgpu-update-async-wait-count=gfx-arch=gfx950 | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s --tritonamdgpu-block-pingpong=num-stages=3 | FileCheck %s; triton-opt %s --tritonamdgpu-block-pingpong=num-stages=3 --tritonamdgpu-update-async-wait-count=gfx-arch=gfx950 | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-13
```mlir
// BlockPingpong's transformTwoClusterWithLocalLoadAndAll combines per-operand
// ttg.async_waits into a single multi-token wait, then reorders async copies
// and commits around it. After the reorder the pass re-runs updateWaits, so
// the merged wait's `num` is derived via minNumInterleavedCommitOps against
// the post-reorder IR (with multi-token support added in the same patch).
//
// On asyncmark targets (CDNA3/CDNA4) this `num` lowers straight to
// rocdl.wait.asyncmark(N), and UpdateAsyncWaitCount is a no-op since PR #9883
// - so whatever num BlockPingpong writes is what reaches the hardware. The
// second RUN line confirms UpdateAsyncWaitCount leaves the wait untouched.
```
**EN:** This comment block provides context for the surrounding test logic: `// BlockPingpong's transformTwoClusterWithLocalLoadAndAll combines per-operand`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// BlockPingpong's transformTwoClusterWithLocalLoadAndAll combines per-operand`。

### Lines 15-73
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [32, 2], warpsPerCTA = [1, 8], order = [0, 1]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [2, 32], warpsPerCTA = [8, 1], order = [1, 0]}>
#mma = #ttg.amd_mfma<{version = 4, warpsPerCTA = [2, 4], instrShape = [16, 16, 32], isTransposed = true}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [0, 1]}>
#shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 16, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "hip:gfx950", "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: async_ns3_gemm_pingpong_multi_token
  // The two per-operand waits in the input carry num=2 and num=1; pingpong
  // fuses them into a single multi-token wait whose num is recomputed against
  // the post-reorder IR. In this synthetic test the tokens come from tt.func
  // args (no prologue commit chain), so minNumInterleavedCommitOps bails to
  // its conservative N=0; in a real kernel the chains terminate at prologue
  // async_commit_groups and yield a tighter bound (e.g. 1 for a real
  // gfx950 simple_persistent_matmul kernel).
  // CHECK: scf.for
  // CHECK: ttg.async_wait %{{[^,]+}}, %{{[^,]+}} {num = 0 : i32}
  tt.func public @async_ns3_gemm_pingpong_multi_token(
      %arg0: i32,
      %arg1: tensor<256x32x!tt.ptr<bf16>, #blocked>,
      %arg2: tensor<32x256x!tt.ptr<bf16>, #blocked1>,
      %arg3: !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>,
      %arg4: !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>,
      %arg5: !ttg.async.token,
      %arg6: !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>,
      %arg7: !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>,
      %arg8: !ttg.async.token,
      %arg9: !ttg.async.token,
      %arg10: !ttg.async.token,
      %arg11: tensor<256x32xi32, #blocked>,
      %arg12: tensor<32x256xi32, #blocked1>,
      %arg13: !ttg.memdesc<3x256x32xbf16, #shared, #smem, mutable>,
      %arg14: !ttg.memdesc<3x32x256xbf16, #shared1, #smem, mutable>) {
    %c3_i32 = arith.constant 3 : i32
    %c0_i32 = arith.constant 0 : i32
    %c1_i32 = arith.constant 1 : i32
    %cst = arith.constant dense<0.000000e+00> : tensor<256x256xf32, #mma>
    %0:12 = scf.for %arg15 = %c0_i32 to %arg0 step %c1_i32 iter_args(%arg16 = %cst, %arg17 = %arg1, %arg18 = %arg2, %arg19 = %c1_i32, %arg20 = %arg3, %arg21 = %arg4, %arg22 = %arg5, %arg23 = %arg6, %arg24 = %arg7, %arg25 = %arg8, %arg26 = %arg9, %arg27 = %arg10) -> (tensor<256x256xf32, #mma>, tensor<256x32x!tt.ptr<bf16>, #blocked>, tensor<32x256x!tt.ptr<bf16>, #blocked1>, i32, !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>, !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>, !ttg.async.token, !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>, !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>, !ttg.async.token, !ttg.async.token, !ttg.async.token)  : i32 {
      %1 = tt.addptr %arg17, %arg11 : tensor<256x32x!tt.ptr<bf16>, #blocked>, tensor<256x32xi32, #blocked>
      %2 = tt.addptr %arg18, %arg12 : tensor<32x256x!tt.ptr<bf16>, #blocked1>, tensor<32x256xi32, #blocked1>
      %3 = arith.addi %arg19, %c1_i32 : i32
      %4 = arith.cmpi slt, %3, %c3_i32 : i32
      %5 = arith.select %4, %3, %c0_i32 : i32
      %6 = ttg.memdesc_index %arg13[%5] : !ttg.memdesc<3x256x32xbf16, #shared, #smem, mutable> -> !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>
      %7 = ttg.async_copy_global_to_local %1, %6 : tensor<256x32x!tt.ptr<bf16>, #blocked> -> <256x32xbf16, #shared, #smem, mutable>
      %8 = ttg.async_commit_group tokens %7
      %9 = ttg.local_load %arg20 token %arg22 : !ttg.memdesc<256x32xbf16, #shared, #smem, mutable> -> tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
      %10 = ttg.memdesc_index %arg14[%5] : !ttg.memdesc<3x32x256xbf16, #shared1, #smem, mutable> -> !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>
      %11 = ttg.async_copy_global_to_local %2, %10 : tensor<32x256x!tt.ptr<bf16>, #blocked1> -> <32x256xbf16, #shared1, #smem, mutable>
      %12 = ttg.async_commit_group tokens %11
      %13 = ttg.local_load %arg23 token %arg25 : !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable> -> tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>
      %14 = tt.dot %9, %13, %arg16 : tensor<256x32xbf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> * tensor<32x256xbf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>> -> tensor<256x256xf32, #mma>
      %15 = ttg.async_wait %arg26 {num = 2 : i32}
      %16 = ttg.async_wait %arg27 {num = 1 : i32}
      scf.yield %14, %1, %2, %5, %arg21, %6, %15, %arg24, %10, %16, %8, %12 : tensor<256x256xf32, #mma>, tensor<256x32x!tt.ptr<bf16>, #blocked>, tensor<32x256x!tt.ptr<bf16>, #blocked1>, i32, !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>, !ttg.memdesc<256x32xbf16, #shared, #smem, mutable>, !ttg.async.token, !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>, !ttg.memdesc<32x256xbf16, #shared1, #smem, mutable>, !ttg.async.token, !ttg.async.token, !ttg.async.token
    }
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx950`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_ns3_gemm_pingpong_multi_token`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx950`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_ns3_gemm_pingpong_multi_token`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's TritonGPU transforms coverage and focuses on asynchronous copy/lowering.  
  **CN:** 该文件属于 Triton 的 TritonGPU 变换 测试覆盖，关注点是 异步拷贝/降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--tritonamdgpu-block-pingpong=num-stages=3`, `--tritonamdgpu-update-async-wait-count=gfx-arch=gfx950`.  
  **CN:** `RUN` 流水线会驱动 `--tritonamdgpu-block-pingpong=num-stages=3`, `--tritonamdgpu-update-async-wait-count=gfx-arch=gfx950` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `scf` (structured control flow), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`scf`（结构化控制流）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `scf`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`scf`、`arith`。
