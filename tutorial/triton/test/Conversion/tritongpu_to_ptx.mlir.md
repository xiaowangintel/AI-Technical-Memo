# tritongpu_to_ptx.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/tritongpu_to_ptx.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises PTX emission in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的PTX 生成相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --allocate-shared-memory-nv='compute-capability=90 ptx-version=83' --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=83' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_90 -mattr=+ptx83 | FileCheck --check-prefixes CHECK,SM90 --dump-input-context=20 %s`<br>`triton-opt %s --allocate-shared-memory-nv='compute-capability=80 ptx-version=83' --convert-triton-gpu-to-llvm='compute-capability=80 ptx-version=83' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_80 -mattr=+ptx83 | FileCheck --check-prefixes CHECK,SM80 --dump-input-context=20 %s`<br>`triton-opt %s --allocate-shared-memory-nv='compute-capability=100 ptx-version=87' --convert-triton-gpu-to-llvm='compute-capability=100 ptx-version=87' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_100 -mattr=+ptx87 | FileCheck --check-prefixes CHECK,SM100 --dump-input-context=20 %s`<br>`triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=80 ptx-version=83' -cse | FileCheck --check-prefix=VEC80 --dump-input-context=20 %s`<br>`triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=83' -cse | FileCheck --check-prefix=VEC90 --dump-input-context=20 %s`<br>`triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=100 ptx-version=87' -cse | FileCheck --check-prefix=VEC100 --dump-input-context=20 %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×3, CHECK-COUNT×4, CHECK-LABEL×9; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×3, CHECK-COUNT×4, CHECK-LABEL×9；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that PTX emission produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 PTX 生成 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6
```mlir
// RUN: triton-opt %s --allocate-shared-memory-nv='compute-capability=90 ptx-version=83' --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=83' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_90 -mattr=+ptx83 | FileCheck --check-prefixes CHECK,SM90 --dump-input-context=20 %s
// RUN: triton-opt %s --allocate-shared-memory-nv='compute-capability=80 ptx-version=83' --convert-triton-gpu-to-llvm='compute-capability=80 ptx-version=83' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_80 -mattr=+ptx83 | FileCheck --check-prefixes CHECK,SM80 --dump-input-context=20 %s
// RUN: triton-opt %s --allocate-shared-memory-nv='compute-capability=100 ptx-version=87' --convert-triton-gpu-to-llvm='compute-capability=100 ptx-version=87' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_100 -mattr=+ptx87 | FileCheck --check-prefixes CHECK,SM100 --dump-input-context=20 %s
// RUN: triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=80 ptx-version=83' -cse | FileCheck --check-prefix=VEC80 --dump-input-context=20 %s
// RUN: triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=83' -cse | FileCheck --check-prefix=VEC90 --dump-input-context=20 %s
// RUN: triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=100 ptx-version=87' -cse | FileCheck --check-prefix=VEC100 --dump-input-context=20 %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s --allocate-shared-memory-nv='compute-capability=90 ptx-version=83' --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=83' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_90 -mattr=+ptx83 | FileCheck --check-prefixes CHECK,SM90 --dump-input-context=20 %s; triton-opt %s --allocate-shared-memory-nv='compute-capability=80 ptx-version=83' --convert-triton-gpu-to-llvm='compute-capability=80 ptx-version=83' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_80 -mattr=+ptx83 | FileCheck --check-prefixes CHECK,SM80 --dump-input-context=20 %s; triton-opt %s --allocate-shared-memory-nv='compute-capability=100 ptx-version=87' --convert-triton-gpu-to-llvm='compute-capability=100 ptx-version=87' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_100 -mattr=+ptx87 | FileCheck --check-prefixes CHECK,SM100 --dump-input-context=20 %s; triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=80 ptx-version=83' -cse | FileCheck --check-prefix=VEC80 --dump-input-context=20 %s; triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=83' -cse | FileCheck --check-prefix=VEC90 --dump-input-context=20 %s; triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=100 ptx-version=87' -cse | FileCheck --check-prefix=VEC100 --dump-input-context=20 %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s --allocate-shared-memory-nv='compute-capability=90 ptx-version=83' --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=83' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_90 -mattr=+ptx83 | FileCheck --check-prefixes CHECK,SM90 --dump-input-context=20 %s; triton-opt %s --allocate-shared-memory-nv='compute-capability=80 ptx-version=83' --convert-triton-gpu-to-llvm='compute-capability=80 ptx-version=83' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_80 -mattr=+ptx83 | FileCheck --check-prefixes CHECK,SM80 --dump-input-context=20 %s; triton-opt %s --allocate-shared-memory-nv='compute-capability=100 ptx-version=87' --convert-triton-gpu-to-llvm='compute-capability=100 ptx-version=87' --convert-nv-gpu-to-llvm | mlir-translate --mlir-to-llvmir | opt -O3 -S | llc -mtriple nvptx64-nvidia-cuda -mcpu=sm_100 -mattr=+ptx87 | FileCheck --check-prefixes CHECK,SM100 --dump-input-context=20 %s; triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=80 ptx-version=83' -cse | FileCheck --check-prefix=VEC80 --dump-input-context=20 %s; triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=83' -cse | FileCheck --check-prefix=VEC90 --dump-input-context=20 %s; triton-opt %s --convert-triton-gpu-to-llvm='compute-capability=100 ptx-version=87' -cse | FileCheck --check-prefix=VEC100 --dump-input-context=20 %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 9-22
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
#blocked_reduce = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [1, 2], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @add_bf16(%ptr: !tt.ptr<bf16> {tt.divisibility = 16 : i32}, %arg0: tensor<256xbf16, #blocked>, %arg1: tensor<256xbf16, #blocked>) {
    // CHECK-LABEL: add_bf16
    // SM80-COUNT-4: fma.rn.bf16x2
    // SM90-COUNT-4: add.rn.bf16x2
    %0 = arith.addf %arg0, %arg1 : tensor<256xbf16, #blocked>
    %1 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked>
    %2 = tt.splat %ptr : !tt.ptr<bf16> -> tensor<256x!tt.ptr<bf16>, #blocked>
    %3 = tt.addptr %2, %1 : tensor<256x!tt.ptr<bf16>, #blocked>, tensor<256xi32, #blocked>
    tt.store %3, %0 : tensor<256x!tt.ptr<bf16>, #blocked>
    tt.return
  }
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `add_bf16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `add_bf16`。

### Lines 24-34
```mlir
  tt.func public @sub_bf16(%ptr: !tt.ptr<bf16> {tt.divisibility = 16 : i32}, %arg0: tensor<256xbf16, #blocked>, %arg1: tensor<256xbf16, #blocked>) {
    // CHECK-LABEL: sub_bf16
    // SM80-COUNT-4: fma.rn.bf16x2
    // SM90-COUNT-4: sub.rn.bf16x2
    %0 = arith.subf %arg0, %arg1 : tensor<256xbf16, #blocked>
    %1 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked>
    %2 = tt.splat %ptr : !tt.ptr<bf16> -> tensor<256x!tt.ptr<bf16>, #blocked>
    %3 = tt.addptr %2, %1 : tensor<256x!tt.ptr<bf16>, #blocked>, tensor<256xi32, #blocked>
    tt.store %3, %0 : tensor<256x!tt.ptr<bf16>, #blocked>
    tt.return
  }
```
**EN:** This block defines `sub_bf16` and exercises tensor stores, pointer arithmetic, range generation. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `sub_bf16`，并覆盖 张量存储、指针运算、范围生成。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 36-46
```mlir
  tt.func public @mul_bf16(%ptr: !tt.ptr<bf16> {tt.divisibility = 16 : i32}, %arg0: tensor<256xbf16, #blocked>, %arg1: tensor<256xbf16, #blocked>) {
    // CHECK-LABEL: mul_bf16
    // SM80-COUNT-4: fma.rn.bf16x2
    // SM90-COUNT-4: mul.rn.bf16x2
    %0 = arith.mulf %arg0, %arg1 : tensor<256xbf16, #blocked>
    %1 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked>
    %2 = tt.splat %ptr : !tt.ptr<bf16> -> tensor<256x!tt.ptr<bf16>, #blocked>
    %3 = tt.addptr %2, %1 : tensor<256x!tt.ptr<bf16>, #blocked>, tensor<256xi32, #blocked>
    tt.store %3, %0 : tensor<256x!tt.ptr<bf16>, #blocked>
    tt.return
  }
```
**EN:** This block defines `mul_bf16` and exercises tensor stores, pointer arithmetic, range generation. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `mul_bf16`，并覆盖 张量存储、指针运算、范围生成。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 48-57
```mlir
  tt.func public @extf_bf16(%ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg0: tensor<256xbf16, #blocked>) {
    // CHECK-LABEL: extf_bf16
    // CHECK-COUNT-8: cvt.f32.bf16
    %0 = arith.extf %arg0 : tensor<256xbf16, #blocked> to tensor<256xf32, #blocked>
    %1 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked>
    %2 = tt.splat %ptr : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked>
    %3 = tt.addptr %2, %1 : tensor<256x!tt.ptr<f32>, #blocked>, tensor<256xi32, #blocked>
    tt.store %3, %0 : tensor<256x!tt.ptr<f32>, #blocked>
    tt.return
  }
```
**EN:** This block defines `extf_bf16` and exercises tensor stores, pointer arithmetic, range generation. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `extf_bf16`，并覆盖 张量存储、指针运算、范围生成。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 59-68
```mlir
  tt.func public @truncf_bf16(%ptr: !tt.ptr<bf16> {tt.divisibility = 16 : i32}, %arg0: tensor<256xf32, #blocked>) {
    // CHECK-LABEL: truncf_bf16
    // CHECK-COUNT-4: cvt.rn.bf16x2.f32
    %0 = arith.truncf %arg0 : tensor<256xf32, #blocked> to tensor<256xbf16, #blocked>
    %1 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked>
    %2 = tt.splat %ptr : !tt.ptr<bf16> -> tensor<256x!tt.ptr<bf16>, #blocked>
    %3 = tt.addptr %2, %1 : tensor<256x!tt.ptr<bf16>, #blocked>, tensor<256xi32, #blocked>
    tt.store %3, %0 : tensor<256x!tt.ptr<bf16>, #blocked>
    tt.return
  }
```
**EN:** This block defines `truncf_bf16` and exercises tensor stores, pointer arithmetic, range generation. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `truncf_bf16`，并覆盖 张量存储、指针运算、范围生成。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 70-79
```mlir
  tt.func public @extf_f16(%ptr: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg0: tensor<256xf16, #blocked>) {
    // CHECK-LABEL: extf_f16
    // CHECK-COUNT-8: cvt.f32.f16
    %0 = arith.extf %arg0 : tensor<256xf16, #blocked> to tensor<256xf32, #blocked>
    %1 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked>
    %2 = tt.splat %ptr : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked>
    %3 = tt.addptr %2, %1 : tensor<256x!tt.ptr<f32>, #blocked>, tensor<256xi32, #blocked>
    tt.store %3, %0 : tensor<256x!tt.ptr<f32>, #blocked>
    tt.return
  }
```
**EN:** This block defines `extf_f16` and exercises tensor stores, pointer arithmetic, range generation. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `extf_f16`，并覆盖 张量存储、指针运算、范围生成。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 81-90
```mlir
  tt.func public @truncf_f16(%ptr: !tt.ptr<f16> {tt.divisibility = 16 : i32}, %arg0: tensor<256xf32, #blocked>) {
    // CHECK-LABEL: truncf_f16
    // CHECK-COUNT-4: cvt.rn.f16x2.f32
    %0 = arith.truncf %arg0 : tensor<256xf32, #blocked> to tensor<256xf16, #blocked>
    %1 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked>
    %2 = tt.splat %ptr : !tt.ptr<f16> -> tensor<256x!tt.ptr<f16>, #blocked>
    %3 = tt.addptr %2, %1 : tensor<256x!tt.ptr<f16>, #blocked>, tensor<256xi32, #blocked>
    tt.store %3, %0 : tensor<256x!tt.ptr<f16>, #blocked>
    tt.return
  }
```
**EN:** This block defines `truncf_f16` and exercises tensor stores, pointer arithmetic, range generation. Embedded check comments (CHECK-COUNT×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `truncf_f16`，并覆盖 张量存储、指针运算、范围生成。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 92-111
```mlir
  // CHECK-LABEL: reduce_f16_store
  // SM80-NOT: add.rn.f16x2
  // SM90: add.rn.f16x2
  // SM100: add.rn.f16x2
  // VEC80-LABEL: llvm.func @reduce_f16_store
  // VEC80-NOT: llvm.fadd {{.*}} : vector<2xf16>
  // VEC90-LABEL: llvm.func @reduce_f16_store
  // VEC90: llvm.fadd {{.*}} : vector<2xf16>
  // VEC100-LABEL: llvm.func @reduce_f16_store
  // VEC100: llvm.fadd {{.*}} : vector<2xf16>
  tt.func public @reduce_f16_store(%out: !tt.ptr<f16>, %arg0: tensor<1x256xf16, #blocked_reduce>) {
    %r = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f16, %b: f16):
      %sum = arith.addf %a, %b : f16
      tt.reduce.return %sum : f16
    }) {allocation.offset = 0 : i32} : (tensor<1x256xf16, #blocked_reduce>) -> tensor<1xf16, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    %ptr = tt.splat %out : !tt.ptr<f16> -> tensor<1x!tt.ptr<f16>, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.store %ptr, %r : tensor<1x!tt.ptr<f16>, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.return
  }
```
**EN:** This block defines `reduce_f16_store` and exercises tensor stores. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `reduce_f16_store`，并覆盖 张量存储。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 113-130
```mlir
  // CHECK-LABEL: reduce_f32_store
  // VEC80-LABEL: llvm.func @reduce_f32_store
  // VEC80-NOT: llvm.fadd {{.*}} : vector<2xf32>
  // VEC90-LABEL: llvm.func @reduce_f32_store
  // VEC90-NOT: llvm.fadd {{.*}} : vector<2xf32>
  // VEC100-LABEL: llvm.func @reduce_f32_store
  // VEC100: llvm.fadd {{.*}} : vector<2xf32>
  tt.func public @reduce_f32_store(%out: !tt.ptr<f32>, %arg0: tensor<1x256xf32, #blocked_reduce>) {
    %r = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f32, %b: f32):
      %sum = arith.addf %a, %b : f32
      tt.reduce.return %sum : f32
    }) {allocation.offset = 0 : i32} : (tensor<1x256xf32, #blocked_reduce>) -> tensor<1xf32, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    %ptr = tt.splat %out : !tt.ptr<f32> -> tensor<1x!tt.ptr<f32>, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.store %ptr, %r : tensor<1x!tt.ptr<f32>, #ttg.slice<{dim = 1, parent = #blocked_reduce}>>
    tt.return
  }
}
```
**EN:** This block defines `reduce_f32_store` and exercises tensor stores. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `reduce_f32_store`，并覆盖 张量存储。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on PTX emission.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 PTX 生成。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory-nv='compute-capability=90`, `--convert-triton-gpu-to-llvm='compute-capability=90`, `--convert-nv-gpu-to-llvm`, `--mlir-to-llvmir`, `-O3`, `-S`, `-mtriple`, `-mcpu=sm_90`, `-mattr=+ptx83`, `--check-prefixes`, ... (+11).  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory-nv='compute-capability=90`, `--convert-triton-gpu-to-llvm='compute-capability=90`, `--convert-nv-gpu-to-llvm`, `--mlir-to-llvmir`, `-O3`, `-S`, `-mtriple`, `-mcpu=sm_90`, `-mattr=+ptx83`, `--check-prefixes`, ... (+11) 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `llvm` (LLVM dialect IR).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`llvm`（LLVM 方言 IR）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `mlir-translate`, `llc`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`mlir-translate`、`llc`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `llvm`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`llvm`。
