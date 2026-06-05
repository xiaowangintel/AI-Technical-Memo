# atomic_ldst.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/atomic_ldst.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises atomic operations in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的原子操作相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --allocate-shared-memory-nv=compute-capability=90 --convert-triton-gpu-to-llvm=compute-capability=90 2>&1 | FileCheck %s --check-prefix=CHECK-TTG2NVGPU`<br>`triton-opt %s --allocate-shared-memory-nv=compute-capability=90 --convert-triton-gpu-to-llvm=compute-capability=90 --convert-nv-gpu-to-llvm 2>&1 | FileCheck %s --check-prefix=CHECK-NVGPU2LLVM` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×8; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×8；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that atomic operations produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 原子操作 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```mlir
// RUN: triton-opt %s --allocate-shared-memory-nv=compute-capability=90 --convert-triton-gpu-to-llvm=compute-capability=90 2>&1 | FileCheck %s --check-prefix=CHECK-TTG2NVGPU
// RUN: triton-opt %s --allocate-shared-memory-nv=compute-capability=90 --convert-triton-gpu-to-llvm=compute-capability=90 --convert-nv-gpu-to-llvm 2>&1 | FileCheck %s --check-prefix=CHECK-NVGPU2LLVM
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @kernel_r(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant 0.000000e+00 : f32
    %true = arith.constant true
    %c128_i32 = arith.constant 128 : i32
    %c512_i32 = arith.constant 512 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c128_i32 : i32
    %2 = arith.cmpi slt, %1, %c512_i32 : i32
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `kernel_r`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `kernel_r`。

### Lines 13-16
```mlir
    // CHECK-TTG2NVGPU: nvg.ld_acquire acquire, gpu
    // CHECK-NVGPU2LLVM: ld.global.gpu.acquire.b32
    %3 = tt.atomic_rmw fadd, acquire, gpu, %arg0, %cst, %2 : (!tt.ptr<f32>, f32, i1) -> f32
    tt.store %arg0, %3 : !tt.ptr<f32>
```
**EN:** This block defines the test function(s) and exercises tensor stores. Embedded check comments (CHECK×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 18-21
```mlir
    // CHECK-TTG2NVGPU: nvg.ld_acquire acquire, cta
    // CHECK-NVGPU2LLVM: ld.global.cta.acquire.b32
    %4 = tt.atomic_rmw fadd, acquire, cta, %arg0, %cst, %true : (!tt.ptr<f32>, f32, i1) -> f32
    tt.store %arg0, %4 : !tt.ptr<f32>
```
**EN:** This block defines the test function(s) and exercises tensor stores. Embedded check comments (CHECK×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 23-29
```mlir
    // CHECK-TTG2NVGPU: nvg.ld_acquire acquire, sys
    // CHECK-NVGPU2LLVM: ld.global.sys.acquire.b32
    %5 = tt.atomic_rmw fadd, acquire, sys, %arg0, %cst, %2 : (!tt.ptr<f32>, f32, i1) -> f32
    tt.store %arg0, %5 : !tt.ptr<f32>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises tensor stores. Embedded check comments (CHECK×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on atomic operations.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 原子操作。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory-nv=compute-capability=90`, `--convert-triton-gpu-to-llvm=compute-capability=90`, `--check-prefix=CHECK-TTG2NVGPU`, `--convert-nv-gpu-to-llvm`, `--check-prefix=CHECK-NVGPU2LLVM`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory-nv=compute-capability=90`, `--convert-triton-gpu-to-llvm=compute-capability=90`, `--check-prefix=CHECK-TTG2NVGPU`, `--convert-nv-gpu-to-llvm`, `--check-prefix=CHECK-NVGPU2LLVM` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`。
