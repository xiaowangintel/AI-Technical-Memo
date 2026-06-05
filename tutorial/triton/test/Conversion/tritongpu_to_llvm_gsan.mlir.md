# tritongpu_to_llvm_gsan.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/tritongpu_to_llvm_gsan.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises LLVM lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的LLVM 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file -tritoninstrument-global-sanitizer --allocate-shared-memory-nv --convert-triton-gpu-to-llvm | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×19, CHECK-LABEL×5; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×19, CHECK-LABEL×5；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that LLVM lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 LLVM 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file -tritoninstrument-global-sanitizer --allocate-shared-memory-nv --convert-triton-gpu-to-llvm | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file -tritoninstrument-global-sanitizer --allocate-shared-memory-nv --convert-triton-gpu-to-llvm | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file -tritoninstrument-global-sanitizer --allocate-shared-memory-nv --convert-triton-gpu-to-llvm | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-19
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.instrumentation_mode" = "gsan", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: llvm.func @load_store
  // CHECK: llvm.call @__triton_gsan_init(%{{.*}}, %{{.*}}, %{{.*}}) : (!llvm.ptr, !llvm.ptr, i32) -> ()
  // CHECK: nvvm.barrier0
  // CHECK: llvm.store %{{.*}} : i64, !llvm.ptr
  // CHECK: llvm.store %{{.*}} : i8, !llvm.ptr
  // CHECK: llvm.call @__triton_gsan_load_tensor(%{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}) : (!llvm.ptr, !llvm.ptr, i32, i32, !llvm.ptr, i32) -> ()
  // CHECK-2: ld.global
  // CHECK: llvm.call @__triton_gsan_store_tensor(%{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}) : (!llvm.ptr, !llvm.ptr, i32, i32, !llvm.ptr, i32) -> ()
  // CHECK-2: st.global
  tt.func @load_store(%ptrs: tensor<256x!tt.ptr<f32>, #blocked>, %mask: tensor<256xi1, #blocked>,
                      %other: tensor<256xf32, #blocked>, %vals: tensor<256xf32, #blocked>) {
    %loaded = tt.load %ptrs, %mask, %other : tensor<256x!tt.ptr<f32>, #blocked>
    tt.store %ptrs, %vals, %mask : tensor<256x!tt.ptr<f32>, #blocked>
    tt.return
  }
```
**EN:** This module sets kernel-level metadata, including `ttg.instrumentation_mode`, `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `load_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.instrumentation_mode`, `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `load_store`。

### Lines 21-26
```mlir
  // CHECK-LABEL: llvm.func @unmasked_store
  // CHECK: llvm.call @__triton_gsan_store_tensor(%{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}) : (!llvm.ptr, !llvm.ptr, i32, i32, !llvm.ptr, i32) -> ()
  tt.func @unmasked_store(%ptrs: tensor<128x!tt.ptr<i32>, #blocked>, %vals: tensor<128xi32, #blocked>) {
    tt.store %ptrs, %vals : tensor<128x!tt.ptr<i32>, #blocked>
    tt.return
  }
```
**EN:** This block defines `unmasked_store` and exercises tensor stores. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `unmasked_store`，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 28-35
```mlir
  // CHECK-LABEL: llvm.func @unmasked_atomic_add
  // CHECK: llvm.call @__triton_gsan_atomic_begin_scalar
  // CHECK: llvm.call @__triton_gsan_atomic_end_scalar
  tt.func @unmasked_atomic_add(%ptr: !tt.ptr<i32>, %val: i32) {
    %0 = tt.atomic_rmw add, relaxed, gpu, %ptr, %val : (!tt.ptr<i32>, i32) -> i32
    tt.return
  }
}
```
**EN:** This block defines `unmasked_atomic_add` and exercises operations such as `llvm.func`, `llvm.call`, `tt.func`, `tt.ptr`, `tt.atomic_rmw`. Embedded check comments (CHECK×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `unmasked_atomic_add`，并覆盖 如 `llvm.func`、`llvm.call`、`tt.func`、`tt.ptr`、`tt.atomic_rmw` 这样的操作。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 37
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 39-41
```mlir
#shared_f16 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#bar = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared_f16`, `#bar`, `#smem`. They parameterize later tests with compact names for `#ttg.nvmma_shared`, `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared_f16`, `#bar`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvmma_shared`, `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 43-57
```mlir
module attributes {"ttg.instrumentation_mode" = "gsan", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: llvm.func @tma_f16_gsan_merge
  tt.func @tma_f16_gsan_merge(%desc: !tt.tensordesc<32x64xf16, #shared_f16>) {
    %true = arith.constant true
    %c0_i32 = arith.constant 0 : i32
    %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x64xf16, #shared_f16, #smem, mutable>
    %barrier = ttg.local_alloc {allocation.offset = 4096 : i32} : () -> !ttg.memdesc<1xi64, #bar, #smem, mutable>
    // CHECK: llvm.alloca %{{.*}} x !llvm.struct<(array<32 x i64>, array<32 x i8>)>
    // CHECK: %[[COUNT:.*]] = llvm.mlir.constant(32 : i32) : i32
    // CHECK: %[[BYTES:.*]] = llvm.mlir.constant(4 : i32) : i32
    // CHECK: llvm.call @__triton_gsan_load_tensor(%{{.*}}, %{{.*}}, %[[COUNT]], %[[BYTES]], %{{.*}}, %{{.*}})
    ttng.async_tma_copy_global_to_local %desc[%c0_i32, %c0_i32] %buf, %barrier, %true : !tt.tensordesc<32x64xf16, #shared_f16>, !ttg.memdesc<1xi64, #bar, #smem, mutable> -> !ttg.memdesc<32x64xf16, #shared_f16, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.instrumentation_mode`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_f16_gsan_merge`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.instrumentation_mode`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_f16_gsan_merge`。

### Line 59
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 61-63
```mlir
#shared_f16 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#bar = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared_f16`, `#bar`, `#smem`. They parameterize later tests with compact names for `#ttg.nvmma_shared`, `#ttg.swizzled_shared`, `#ttg.shared_memory`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared_f16`, `#bar`, `#smem`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvmma_shared`, `#ttg.swizzled_shared`, `#ttg.shared_memory`。

### Lines 65-79
```mlir
module attributes {"ttg.instrumentation_mode" = "gsan", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: llvm.func @tma_f16_gsan_merge_4warps
  tt.func @tma_f16_gsan_merge_4warps(%desc: !tt.tensordesc<128x64xf16, #shared_f16>) {
    %true = arith.constant true
    %c0_i32 = arith.constant 0 : i32
    %buf = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x64xf16, #shared_f16, #smem, mutable>
    %barrier = ttg.local_alloc {allocation.offset = 16384 : i32} : () -> !ttg.memdesc<1xi64, #bar, #smem, mutable>
    // CHECK: llvm.alloca %{{.*}} x !llvm.struct<(array<32 x i64>, array<32 x i8>)>
    // CHECK: %[[COUNT_4W:.*]] = llvm.mlir.constant(32 : i32) : i32
    // CHECK: %[[BYTES_4W:.*]] = llvm.mlir.constant(4 : i32) : i32
    // CHECK: llvm.call @__triton_gsan_load_tensor(%{{.*}}, %{{.*}}, %[[COUNT_4W]], %[[BYTES_4W]], %{{.*}}, %{{.*}})
    ttng.async_tma_copy_global_to_local %desc[%c0_i32, %c0_i32] %buf, %barrier, %true : !tt.tensordesc<128x64xf16, #shared_f16>, !ttg.memdesc<1xi64, #bar, #smem, mutable> -> !ttg.memdesc<128x64xf16, #shared_f16, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.instrumentation_mode`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `tma_f16_gsan_merge_4warps`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.instrumentation_mode`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `tma_f16_gsan_merge_4warps`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on LLVM lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 LLVM 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `-tritoninstrument-global-sanitizer`, `--allocate-shared-memory-nv`, `--convert-triton-gpu-to-llvm`.  
  **CN:** `RUN` 流水线会驱动 `-tritoninstrument-global-sanitizer`, `--allocate-shared-memory-nv`, `--convert-triton-gpu-to-llvm` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `llvm` (LLVM dialect IR), `nvvm` (NVVM backend intrinsics), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `ttng` (next-gen TritonGPU ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`llvm`（LLVM 方言 IR）、`nvvm`（NVVM 后端内建）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`ttng`（新一代 TritonGPU 操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `llvm`, `nvvm`, `tt`, `arith`, `ttng`.  
  **CN:** IR 方言依赖：`ttg`、`llvm`、`nvvm`、`tt`、`arith`、`ttng`。
