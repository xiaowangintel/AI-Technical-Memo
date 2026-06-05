# tritongpu_to_llvm_hopper.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/tritongpu_to_llvm_hopper.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises LLVM lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的LLVM 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory-nv='compute-capability=90 ptx-version=81' --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=81' | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×81, CHECK-COUNT×43, CHECK-LABEL×36, CHECK-NOT×8, CHECK-SAME×2; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×81, CHECK-COUNT×43, CHECK-LABEL×36, CHECK-NOT×8, CHECK-SAME×2；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that LLVM lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 LLVM 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --allocate-shared-memory-nv='compute-capability=90 ptx-version=81' --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=81' | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-shared-memory-nv='compute-capability=90 ptx-version=81' --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=81' | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-shared-memory-nv='compute-capability=90 ptx-version=81' --convert-triton-gpu-to-llvm='compute-capability=90 ptx-version=81' | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-11
```mlir
module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: @test_cluster_attr
  // CHECK: nvvm.cluster_dim = array<i32: 4>
  // CHECK: nvvm.kernel = 1 : ui1
  // CHECK: nvvm.reqntid = array<i32: 128>
  tt.func @test_cluster_attr(%lb : index, %A : !tt.ptr<f16>) {
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_cluster_attr`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_cluster_attr`。

### Line 13
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 15-35
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 32]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @dot_high_precision_acc
  tt.func @dot_high_precision_acc(%a: !ttg.memdesc<128x128xf8E5M2, #shared, #smem>, %b: !ttg.memdesc<128x256xf8E5M2, #shared1, #smem>, %c: tensor<128x256xf32, #mma>) {
    // CHECK: nvg.wgmma
    // CHECK-COUNT-128: llvm.fadd
    // CHECK: nvg.wgmma
    // CHECK-COUNT-128: llvm.fadd
    // CHECK: nvg.wgmma
    // CHECK-COUNT-128: llvm.fadd
    // CHECK: nvg.wgmma
    // CHECK-COUNT-128: llvm.fadd
    %m = ttng.warp_group_dot %a, %b, %c
      {maxNumImpreciseAcc = 32 : i32, inputPrecision = 0 : i32} :
      !ttg.memdesc<128x128xf8E5M2, #shared, #smem> * !ttg.memdesc<128x256xf8E5M2, #shared1, #smem> -> tensor<128x256xf32, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `dot_high_precision_acc`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `dot_high_precision_acc`。

### Line 37
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 39-60
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 32]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @dot_low_precision_acc
  tt.func @dot_low_precision_acc(%a: !ttg.memdesc<128x128xf8E5M2, #shared, #smem>, %b: !ttg.memdesc<128x256xf8E5M2, #shared1, #smem>, %c: tensor<128x256xf32, #mma>) {
    // CHECK: nvg.wgmma
    // CHECK-NOT: llvm.fadd
    // CHECK: nvg.wgmma
    // CHECK-NOT: llvm.fadd
    // CHECK: nvg.wgmma
    // CHECK-NOT: llvm.fadd
    // CHECK: nvg.wgmma
    // CHECK-NOT: llvm.fadd
    // CHECK: llvm.return
    %m = ttng.warp_group_dot %a, %b, %c
      {maxNumImpreciseAcc = 129 : i32, inputPrecision = 0 : i32} :
      !ttg.memdesc<128x128xf8E5M2, #shared, #smem> * !ttg.memdesc<128x256xf8E5M2, #shared1, #smem> -> tensor<128x256xf32, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `dot_low_precision_acc`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `dot_low_precision_acc`。

### Line 62
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 64-85
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 32]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 32, transposed = true, elementBitWidth = 8}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @dot_mix_precision_acc
  tt.func @dot_mix_precision_acc(%a: !ttg.memdesc<128x128xf8E5M2, #shared, #smem>, %b: !ttg.memdesc<128x256xf8E5M2, #shared1, #smem>, %c: tensor<128x256xf32, #mma>) {
    // CHECK: nvg.wgmma
    // CHECK-NOT: llvm.fadd
    // CHECK: nvg.wgmma
    // CHECK-COUNT-128: llvm.fadd
    // CHECK: nvg.wgmma
    // CHECK-NOT: llvm.fadd
    // CHECK: nvg.wgmma
    // CHECK-COUNT-128: llvm.fadd
    // CHECK: llvm.return
    %m = ttng.warp_group_dot %a, %b, %c
      {maxNumImpreciseAcc = 64 : i32, inputPrecision = 0 : i32} :
      !ttg.memdesc<128x128xf8E5M2, #shared, #smem> * !ttg.memdesc<128x256xf8E5M2, #shared1, #smem> -> tensor<128x256xf32, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `dot_mix_precision_acc`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `dot_mix_precision_acc`。

### Line 87
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 89-103
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [16, 2], instrShape = [16, 256, 16]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 32 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @warp_group_dot_bf16_32_warps
  tt.func @warp_group_dot_bf16_32_warps(
      %a: !ttg.memdesc<256x128xbf16, #shared, #smem>,
      %b: !ttg.memdesc<128x512xbf16, #shared, #smem>,
      %acc: tensor<256x512xf32, #mma>) {
    %res = ttng.warp_group_dot %a, %b, %acc {inputPrecision = 0 : i32, isAsync = true} :
      !ttg.memdesc<256x128xbf16, #shared, #smem> * !ttg.memdesc<128x512xbf16, #shared, #smem> -> tensor<256x512xf32, #mma>
    // CHECK: nvg.wgmma {{.*}} k = 16 : i32, layoutA = 1 : i32, layoutB = 1 : i32, m = 64 : i32, n = 256 : i32}
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `warp_group_dot_bf16_32_warps`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `warp_group_dot_bf16_32_warps`。

### Line 105
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 107-120
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: @dot_zero_acc
  // Generate a wgmma with 2 sources.
  // CHECK: nvg.wgmma %{{.*}}, %{{.*}} {
  tt.func @dot_zero_acc(%a: !ttg.memdesc<128x64xf16, #shared, #smem>, %b: !ttg.memdesc<64x64xf16, #shared1, #smem>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
    %m = ttng.warp_group_dot %a, %b, %cst {inputPrecision = 0 : i32, maxNumImpreciseAcc = 0 : i32} :
      !ttg.memdesc<128x64xf16, #shared, #smem> * !ttg.memdesc<64x64xf16, #shared1, #smem> -> tensor<128x64xf32, #mma>
    tt.return
  }
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `dot_zero_acc`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `dot_zero_acc`。

### Lines 122-129
```mlir
  // CHECK-LABEL: @wgmma_on_subtile
  // CHECK: nvg.wgmma %{{.*}}, %{{.*}}
  tt.func @wgmma_on_subtile(%a: tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>, %b:  !ttg.memdesc<16x256xf16, #shared1, #smem, mutable, 3x64x256>){
    %cst = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #mma>
    %m = ttng.warp_group_dot %a, %b, %cst {inputPrecision = 0 : i32, isAsync = true} : tensor<128x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<16x256xf16, #shared1, #smem, mutable, 3x64x256> -> tensor<128x256xf32, #mma>
    tt.return
  }
}
```
**EN:** This block defines `wgmma_on_subtile` and exercises operations such as `nvg.wgmma`, `tt.func`, `ttg.memdesc`, `arith.constant`, `ttng.warp_group_dot`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `wgmma_on_subtile`，并覆盖 如 `nvg.wgmma`、`tt.func`、`ttg.memdesc`、`arith.constant`、`ttng.warp_group_dot` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 131
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 133-158
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: @dot_reg_operand_A
  // Generate a wgmma where the first operand is a struct.
  // CHECK: %[[A_MOV0:.*]] = llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "mov.b32 $0, $1;", "=r,r" %{{.*}} : (i32) -> i32
  // CHECK: %[[A_MOV1:.*]] = llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "mov.b32 $0, $1;", "=r,r" %{{.*}} : (i32) -> i32
  // CHECK: %[[A_MOV2:.*]] = llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "mov.b32 $0, $1;", "=r,r" %{{.*}} : (i32) -> i32
  // CHECK: %[[A_MOV3:.*]] = llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "mov.b32 $0, $1;", "=r,r" %{{.*}} : (i32) -> i32
  // CHECK: nvvm.wgmma.fence.aligned
  // CHECK: %[[A_PACK0:.*]] = llvm.insertvalue %[[A_MOV0]], %{{.*}}[0] : !llvm.struct<(i32, i32, i32, i32)>
  // CHECK: %[[A_PACK1:.*]] = llvm.insertvalue %[[A_MOV1]], %[[A_PACK0]][1] : !llvm.struct<(i32, i32, i32, i32)>
  // CHECK: %[[A_PACK2:.*]] = llvm.insertvalue %[[A_MOV2]], %[[A_PACK1]][2] : !llvm.struct<(i32, i32, i32, i32)>
  // CHECK: %[[A_PACK3:.*]] = llvm.insertvalue %[[A_MOV3]], %[[A_PACK2]][3] : !llvm.struct<(i32, i32, i32, i32)>
  // CHECK: nvg.wgmma %[[A_PACK3]], %{{.*}}
  // CHECK: nvg.wgmma {{.*}} : (!llvm.struct<(i32, i32, i32, i32)>, i64, i1) -> !llvm.struct<(f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32)>
  // CHECK: nvg.wgmma_wait_group %{{.*}} {pendings = 0 : i32} : !llvm.struct<(f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32)>
  tt.func @dot_reg_operand_A(%a: tensor<128x64xf16, #mma>, %b: !ttg.memdesc<64x64xf16, #shared, #smem>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x64xf32, #mma>
    %opA = ttg.convert_layout %a : tensor<128x64xf16, #mma> -> tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
    %m = ttng.warp_group_dot %opA, %b, %cst { inputPrecision = 0 : i32 }:
      tensor<128x64xf16,  #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<64x64xf16, #shared, #smem> -> tensor<128x64xf32, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `, `. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `dot_reg_operand_A`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `, `。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `dot_reg_operand_A`。

### Line 160
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 162-177
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 128, 32]}>
#mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 32]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: @dot_reg_operand_A_fp8
  // Generate a wgmma where the first operand is a struct.
  // CHECK: nvg.wgmma {{.*}} : (!llvm.struct<(i32, i32, i32, i32)>, i64, i1) -> !llvm.struct<(f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32, f32)>
  // CHECK: nvg.wgmma_wait_group %{{.*}} {pendings = 0 : i32}
  tt.func @dot_reg_operand_A_fp8(%a: tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %b: !ttg.memdesc<128x256xf8E5M2, #shared, #smem>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #mma1>
    %m = ttng.warp_group_dot %a, %b, %cst { maxNumImpreciseAcc = 1073741824 : i32, inputPrecision = 0 : i32 } :
      tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * !ttg.memdesc<128x256xf8E5M2, #shared, #smem> -> tensor<128x256xf32, #mma1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `dot_reg_operand_A_fp8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `dot_reg_operand_A_fp8`。

### Line 179
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 181-194
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [1, 0]}>
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 16}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: dot_reg_operand_upcast
  tt.func @dot_reg_operand_upcast(%a_desc: !ttg.memdesc<128x64xi8, #shared, #smem>, %b: !ttg.memdesc<64x64xf16, #shared1, #smem>, %acc: tensor<128x64xf32, #mma>) {
    %a_dotop = ttg.local_load %a_desc : !ttg.memdesc<128x64xi8, #shared, #smem> -> tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
    %a_casted = arith.sitofp %a_dotop : tensor<128x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> to tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
    %res = ttng.warp_group_dot %a_casted, %b, %acc : tensor<128x64xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * !ttg.memdesc<64x64xf16, #shared1, #smem> -> tensor<128x64xf32, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `dot_reg_operand_upcast`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `dot_reg_operand_upcast`。

### Line 196
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 198-210
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: test_fp8_to_f16_conversion
  tt.func @test_fp8_to_f16_conversion(
    %in0: tensor<128xf8E5M2, #blocked>, %in1: tensor<128xf8E4M3FN, #blocked>,
    %in2: tensor<128xf16, #blocked>, %in3: tensor<128xf32, #blocked>) {
    // CHECK-COUNT-2: cvt.rn.f16x2.e5m2x2 {{.*}} "=r,h" %{{.*}} : (i16) -> vector<2xf16>
    %out0 = tt.fp_to_fp %in0 : tensor<128xf8E5M2, #blocked> -> tensor<128xf16, #blocked>
    // CHECK-COUNT-2: cvt.rn.f16x2.e4m3x2 {{.*}} "=r,h" %{{.*}} : (i16) -> vector<2xf16>
    %out1 = tt.fp_to_fp %in1 : tensor<128xf8E4M3FN, #blocked> -> tensor<128xf16, #blocked>
    // CHECK-COUNT-2: mul.rn.bf16x2
    %out2 = tt.fp_to_fp %in0 : tensor<128xf8E5M2, #blocked> -> tensor<128xbf16, #blocked>
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, ` %{{.*}} : (i16) -> vector<2xf16>
    %out0 = tt.fp_to_fp %in0 : tensor<128xf8E5M2, #blocked> -> tensor<128xf16, #blocked>
    // CHECK-COUNT-2: cvt.rn.f16x2.e4m3x2 {{.*}} `. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_fp8_to_f16_conversion`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, ` %{{.*}} : (i16) -> vector<2xf16>
    %out0 = tt.fp_to_fp %in0 : tensor<128xf8E5M2, #blocked> -> tensor<128xf16, #blocked>
    // CHECK-COUNT-2: cvt.rn.f16x2.e4m3x2 {{.*}} `。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_fp8_to_f16_conversion`。

### Lines 212-215
```mlir
    // CHECK-COUNT-2: cvt.rn.satfinite.e5m2x2.f16x2 {{.*}} "=h,r" %{{.*}} : (i32) -> vector<2xi8>
    %out3 = tt.fp_to_fp %in2, rounding = rtne : tensor<128xf16, #blocked> -> tensor<128xf8E5M2, #blocked>
    // CHECK-COUNT-2: cvt.rn.satfinite.e4m3x2.f16x2 {{.*}} "=h,r" %{{.*}} : (i32) -> vector<2xi8>
    %out4 = tt.fp_to_fp %in2, rounding = rtne : tensor<128xf16, #blocked> -> tensor<128xf8E4M3FN, #blocked>
```
**EN:** This block defines the test function(s) and exercises operations such as `cvt.rn.satfinite.e5m2x2.f16x2`, `tt.fp_to_fp`, `cvt.rn.satfinite.e4m3x2.f16x2`. Embedded check comments (CHECK-COUNT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `cvt.rn.satfinite.e5m2x2.f16x2`、`tt.fp_to_fp`、`cvt.rn.satfinite.e4m3x2.f16x2` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 217-223
```mlir
    // CHECK-COUNT-2: cvt.rn.satfinite.e5m2x2.f32 {{.*}} "=h,r,r" %{{.*}}, %{{.*}} : (i32, i32) -> vector<2xi8>
    %out5 = tt.fp_to_fp %in3, rounding = rtne : tensor<128xf32, #blocked> -> tensor<128xf8E5M2, #blocked>
    // CHECK-COUNT-2: cvt.rn.satfinite.e4m3x2.f32 {{.*}} "=h,r,r" %{{.*}}, %{{.*}} : (i32, i32) -> vector<2xi8>
    %out6 = tt.fp_to_fp %in3, rounding = rtne : tensor<128xf32, #blocked> -> tensor<128xf8E4M3FN, #blocked>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `cvt.rn.satfinite.e5m2x2.f32`, `tt.fp_to_fp`, `cvt.rn.satfinite.e4m3x2.f32`, `tt.return`. Embedded check comments (CHECK-COUNT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `cvt.rn.satfinite.e5m2x2.f32`、`tt.fp_to_fp`、`cvt.rn.satfinite.e4m3x2.f32`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 225
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 227-232
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
// CHECK-LABEL: clamp
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @clamp(%x : tensor<1024xf32, #blocked>, %limit : tensor<1024xf32, #blocked>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<1024xf32, #blocked>
    %neg_limit = arith.subf %cst, %limit : tensor<1024xf32, #blocked>
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `clamp`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `clamp`。

### Lines 234-238
```mlir
    // CHECK-COUNT-8: nvvm.fmin.xorsign.abs.f
    %12 = tt.clampf %x, %neg_limit, %limit, propagateNan = none : tensor<1024xf32, #blocked>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises NVIDIA-specific codegen. Embedded check comments (CHECK-COUNT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK-COUNT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 240
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 242-248
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
// CHECK-LABEL: clamp_splat_negf
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @clamp_splat_negf(%x : tensor<1024xf32, #blocked>, %limit : f32) {
    %neg_limit = arith.negf %limit : f32
    %lower = tt.splat %neg_limit : f32 -> tensor<1024xf32, #blocked>
    %upper = tt.splat %limit : f32 -> tensor<1024xf32, #blocked>
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `clamp_splat_negf`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `clamp_splat_negf`。

### Lines 250-254
```mlir
    // CHECK-COUNT-8: nvvm.fmin.xorsign.abs.f
    %12 = tt.clampf %x, %lower, %upper, propagateNan = none : tensor<1024xf32, #blocked>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises NVIDIA-specific codegen. Embedded check comments (CHECK-COUNT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK-COUNT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 256
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 258-262
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
// CHECK-LABEL: clamp_negf
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @clamp_negf(%x : tensor<1024xf32, #blocked>, %limit : tensor<1024xf32, #blocked>) {
    %neg_limit = arith.negf %limit : tensor<1024xf32, #blocked>
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `clamp_negf`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `clamp_negf`。

### Lines 264-268
```mlir
    // CHECK-COUNT-8: nvvm.fmin.xorsign.abs.f
    %12 = tt.clampf %x, %neg_limit, %limit, propagateNan = none : tensor<1024xf32, #blocked>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises NVIDIA-specific codegen. Embedded check comments (CHECK-COUNT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK-COUNT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 270
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 272-275
```mlir
// CHECK-LABEL: clamp_negf_scalar
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @clamp_negf_scalar(%x : f32, %limit : f32) {
    %neg_limit = arith.negf %limit : f32
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `clamp_negf_scalar`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `clamp_negf_scalar`。

### Lines 277-281
```mlir
    // CHECK: nvvm.fmin.xorsign.abs.f
    %12 = tt.clampf %x, %neg_limit, %limit, propagateNan = none : f32
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 283
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 285-289
```mlir
// CHECK-LABEL: clamp_scalar
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @clamp_scalar(%x : f32, %limit : f32) {
    %cst = arith.constant 0.000000e+00 : f32
    %neg_limit = arith.subf %cst, %limit : f32
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `clamp_scalar`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `clamp_scalar`。

### Lines 291-295
```mlir
    // CHECK: nvvm.fmin.xorsign.abs.f
    %12 = tt.clampf %x, %neg_limit, %limit, propagateNan = none : f32
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 297
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 299-314
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [1, 32], warpsPerCTA = [8, 1], order = [0, 1]}>
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 16]}>
// CHECK-LABEL: convert_mma_to_blocked
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @convert_mma_to_blocked(%a: tensor<128x256xf16, #mma>) {
    // CHECK-COUNT-8: llvm.store
    //          CHECK: nvvm.barrier0
    // CHECK-COUNT-8: nvvm.ldmatrix
    //          CHECK: nvvm.barrier0
    // CHECK-COUNT-8: llvm.store
    //          CHECK: nvvm.barrier0
    // CHECK-COUNT-8: nvvm.ldmatrix
    %c = ttg.convert_layout %a : tensor<128x256xf16, #mma> -> tensor<128x256xf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_mma_to_blocked`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_mma_to_blocked`。

### Line 316
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 318-342
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [4, 2], order = [0, 1]}>
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 32]], warp = [[32, 0], [64, 0], [16, 0]], block = []}>
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @convert_mma_to_blocked(%a: tensor<128x64xbf16, #linear>) {
    // CHECK: llvm.store {{.*}} : vector<4xi32>
    // CHECK: nvvm.barrier0
    // CHECK: llvm.load {{.*}} -> vector<4xi32>
    // CHECK: nvvm.barrier0
    // CHECK: llvm.store {{.*}} : vector<4xi32>
    // CHECK: nvvm.barrier0
    // CHECK: llvm.load {{.*}} -> vector<4xi32>
    // CHECK: nvvm.barrier0
    // CHECK: llvm.store {{.*}} : vector<4xi32>
    // CHECK: nvvm.barrier0
    // CHECK: llvm.load {{.*}} -> vector<4xi32>
    // CHECK: nvvm.barrier0
    // CHECK: llvm.store {{.*}} : vector<4xi32>
    // CHECK: nvvm.barrier0
    // CHECK: llvm.load {{.*}} -> vector<4xi32>
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %b = ttg.convert_layout %a: tensor<128x64xbf16, #linear> -> tensor<128x64xbf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_mma_to_blocked`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_mma_to_blocked`。

### Line 344
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 346-370
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 8]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // There are x4 the ldmatrix as there is broadcasting at a warp level
  // CHECK-LABEL: convert_blocked_to_dot_rhs
  tt.func @convert_blocked_to_dot_rhs(%a: tensor<64x64xf16, #blocked>) {
    // CHECK-COUNT-1: llvm.store
    //          CHECK: nvvm.barrier0
    // CHECK-COUNT-4: nvvm.ldmatrix
    //          CHECK: nvvm.barrier0
    // CHECK-COUNT-1: llvm.store
    //          CHECK: nvvm.barrier0
    // CHECK-COUNT-4: nvvm.ldmatrix
    //          CHECK: nvvm.barrier0
    // CHECK-COUNT-1: llvm.store
    //          CHECK: nvvm.barrier0
    // CHECK-COUNT-4: nvvm.ldmatrix
    //          CHECK: nvvm.barrier0
    // CHECK-COUNT-1: llvm.store
    //          CHECK: nvvm.barrier0
    // CHECK-COUNT-4: nvvm.ldmatrix
    %b = ttg.convert_layout %a  : tensor<64x64xf16, #blocked> -> tensor<64x64xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_blocked_to_dot_rhs`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_blocked_to_dot_rhs`。

### Line 372
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 374-385
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
// CHECK-LABEL: cvt_mma_to_dot_fp8
// CHECK-COUNT-16: llvm.select
// CHECK-COUNT-16: nvvm.shfl.sync
// CHECK-COUNT-16: llvm.select
  tt.func @cvt_mma_to_dot_fp8(%a: tensor<128x64xf8E5M2, #mma>) {
    %opA = ttg.convert_layout %a : tensor<128x64xf8E5M2, #mma> -> tensor<128x64xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `cvt_mma_to_dot_fp8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `cvt_mma_to_dot_fp8`。

### Line 387
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 389-402
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 32]}>
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 8}>
#shared1 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = true, elementBitWidth = 8}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
// CHECK-LABEL: dot_zero_acc_operand
// CHECK-COUNT-128: llvm.fadd
  tt.func @dot_zero_acc_operand(%a: !ttg.memdesc<128x128xf8E5M2, #shared, #smem>, %b: !ttg.memdesc<128x128xf8E5M2, #shared1, #smem>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
    %m = ttng.warp_group_dot %a, %b, %cst {maxNumImpreciseAcc = 64 : i32, inputPrecision = 0 : i32} :
      !ttg.memdesc<128x128xf8E5M2, #shared, #smem> * !ttg.memdesc<128x128xf8E5M2, #shared1, #smem> -> tensor<128x128xf32, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `dot_zero_acc_operand`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `dot_zero_acc_operand`。

### Line 405
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 407-418
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 16]}>
#smem = #ttg.shared_memory
// CHECK-LABEL: distribute_to_shared_st_matrix
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @distribute_to_shared_st_matrix(%a: tensor<128x128xf16, #mma>) {
    // CHECK-COUNT-16: nvvm.stmatrix
    //          CHECK: llvm.return
    %b = ttg.local_alloc %a {allocation.offset = 0 : i32} : (tensor<128x128xf16, #mma>) -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `distribute_to_shared_st_matrix`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `distribute_to_shared_st_matrix`。

### Line 420
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 422-434
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 16]}>
#smem = #ttg.shared_memory
// CHECK-LABEL: distribute_to_shared_st_matrix_local_store
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @distribute_to_shared_st_matrix_local_store(%a: tensor<128x128xf16, #mma>) {
    // CHECK-COUNT-16: nvvm.stmatrix
    //          CHECK: llvm.return
    %b = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    ttg.local_store %a, %b : tensor<128x128xf16, #mma> -> !ttg.memdesc<128x128xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `distribute_to_shared_st_matrix_local_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `distribute_to_shared_st_matrix_local_store`。

### Line 436
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 438-450
```mlir
#shared = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#linear = #ttg.linear<{register = [[1, 0], [0, 8], [8, 0], [16, 0], [32, 0], [0, 16]], lane = [[2, 0], [4, 0], [0, 1], [0, 2], [0, 4]], warp = [[0, 32], [0, 64]], block = []}>
#smem = #ttg.shared_memory
// CHECK-LABEL: distribute_to_shared_st_matrix_local_store
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @distribute_to_shared_st_matrix_local_store(%a: tensor<64x128xf16, #linear>) {
    // CHECK-COUNT-8: nvvm.stmatrix %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<col>, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>}
    //          CHECK: llvm.return
    %b = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<64x128xf16, #shared, #smem, mutable>
    ttg.local_store %a, %b : tensor<64x128xf16, #linear> -> !ttg.memdesc<64x128xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `distribute_to_shared_st_matrix_local_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `distribute_to_shared_st_matrix_local_store`。

### Line 452
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 454-466
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 32, 16]}>
#smem = #ttg.shared_memory
// CHECK-LABEL: distribute_to_swizzled_st_matrix_local_store
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @distribute_to_swizzled_st_matrix_local_store(%a: tensor<8x64xf16, #mma>) {
    // CHECK-COUNT-2: nvvm.stmatrix
    //          CHECK: llvm.return
    %b = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<8x64xf16, #shared, #smem, mutable>
    ttg.local_store %a, %b : tensor<8x64xf16, #mma> -> !ttg.memdesc<8x64xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `distribute_to_swizzled_st_matrix_local_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `distribute_to_swizzled_st_matrix_local_store`。

### Line 468
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 470-482
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#linear = #ttg.linear<{register = [[0, 1], [8, 0], [0, 8], [0, 16]], lane = [[0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], warp = [[16, 0], [32, 0]], block = []}>
#smem = #ttg.shared_memory
// CHECK-LABEL: linear_to_swizzled_st_matrix_local_store
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @linear_to_swizzled_st_matrix_local_store(%a: tensor<64x32xf16, #linear>) {
    // CHECK-COUNT-2: nvvm.stmatrix %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<row>, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>}
    //          CHECK: llvm.return
    %b = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<64x32xf16, #shared, #smem, mutable>
    ttg.local_store %a, %b : tensor<64x32xf16, #linear> -> !ttg.memdesc<64x32xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `linear_to_swizzled_st_matrix_local_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `linear_to_swizzled_st_matrix_local_store`。

### Line 484
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 486-504
```mlir
// Stretching a bit the lowering. Feel free to kill this test if we restrain
// the lowering a bit later on.
// These layouts will have plenty of bank conflicts, so it'd make sense not to
// lower them via stmatrix.
// It is of course possible to design a shared memory layout that makes the lowering
// via stmatrix not have any bank conflicts, but yeah.
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#linear = #ttg.linear<{register = [[0, 1], [4, 0], [0, 0], [0, 16], [2, 0]], lane = [[0, 2], [0, 4], [0, 0], [8, 0], [0, 8]], warp = [[1, 0], [16, 0]], block = []}>
#smem = #ttg.shared_memory
// CHECK-LABEL: linear_to_swizzled_st_matrix_local_store
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @linear_to_swizzled_st_matrix_local_store(%a: tensor<32x32xf16, #linear>) {
    // CHECK-COUNT-2: nvvm.stmatrix %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<row>, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>}
    //          CHECK: llvm.return
    %b = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable>
    ttg.local_store %a, %b : tensor<32x32xf16, #linear> -> !ttg.memdesc<32x32xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `linear_to_swizzled_st_matrix_local_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `linear_to_swizzled_st_matrix_local_store`。

### Line 506
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 508-520
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [8, 0]], lane = [[0, 4], [0, 8], [1, 0], [2, 0], [4, 0]], warp = [[16, 0], [32, 0]], block = []}>
#smem = #ttg.shared_memory
// CHECK-LABEL: linear_to_swizzled_st_matrix_x2_local_store_fp8
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @linear_to_swizzled_st_matrix_x2_local_store_fp8(%a: tensor<64x16xf8E4M3FNUZ, #linear>) {
    // CHECK-COUNT-1: nvvm.stmatrix %{{.*}}, %{{.*}}, %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<row>, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>}
    //          CHECK: llvm.return
    %b = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<64x16xf8E4M3FNUZ, #shared, #smem, mutable>
    ttg.local_store %a, %b : tensor<64x16xf8E4M3FNUZ, #linear> -> !ttg.memdesc<64x16xf8E4M3FNUZ, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `linear_to_swizzled_st_matrix_x2_local_store_fp8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `linear_to_swizzled_st_matrix_x2_local_store_fp8`。

### Line 522
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 524-536
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#linear = #ttg.linear<{register = [[8, 0], [0, 4], [0, 8]], lane = [[0, 1], [0, 2], [1, 0], [2, 0], [4, 0]], warp = [[16, 0], [32, 0]], block = []}>
#smem = #ttg.shared_memory
// CHECK-LABEL: linear_to_swizzled_st_matrix_local_store_fp32
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @linear_to_swizzled_st_matrix_local_store_fp32(%a: tensor<64x16xf32, #linear>) {
    // CHECK-COUNT-2: nvvm.stmatrix %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<row>, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>}
    //          CHECK: llvm.return
    %b = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<64x16xf32, #shared, #smem, mutable>
    ttg.local_store %a, %b : tensor<64x16xf32, #linear> -> !ttg.memdesc<64x16xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `linear_to_swizzled_st_matrix_local_store_fp32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `linear_to_swizzled_st_matrix_local_store_fp32`。

### Line 539
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 541-553
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1]}>
#linear = #ttg.linear<{register = [[0, 1], [8, 0], [0, 8], [0, 16]], lane = [[0, 2], [0, 4], [1, 0], [2, 0], [4, 0]], warp = [[16, 0], [32, 0]], block = []}>
#smem = #ttg.shared_memory
// CHECK-LABEL: linear_to_swizzled_st_matrix_trans_local_store
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @linear_to_swizzled_st_matrix_trans_local_store(%a: tensor<64x32xf16, #linear>) {
    // CHECK-COUNT-2: nvvm.stmatrix %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<col>, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>}
    //          CHECK: llvm.return
    %b = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<64x32xf16, #shared, #smem, mutable>
    ttg.local_store %a, %b : tensor<64x32xf16, #linear> -> !ttg.memdesc<64x32xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `linear_to_swizzled_st_matrix_trans_local_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `linear_to_swizzled_st_matrix_trans_local_store`。

### Line 555
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 557-575
```mlir
// Stretching a bit the lowering. Feel free to kill this test if we restrain
// the lowering a bit later on.
// These layouts will have plenty of bank conflicts, so it'd make sense not to
// lower them via stmatrix.
// It is of course possible to design a shared memory layout that makes the lowering
// via stmatrix not have any bank conflicts, but yeah.
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0, 1]}>
#linear = #ttg.linear<{register = [[0, 2], [0, 8], [0, 0], [0, 16], [0, 1]], lane = [[0, 0], [0, 4], [1, 0], [2, 0], [4, 0]], warp = [[0, 0], [8, 0]], block = []}>
#smem = #ttg.shared_memory
// CHECK-LABEL: linear_to_swizzled_st_matrix_trans_local_store
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @linear_to_swizzled_st_matrix_trans_local_store(%a: tensor<16x32xf16, #linear>) {
    // CHECK-COUNT-2: nvvm.stmatrix %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}}, %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<col>, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>}
    //          CHECK: llvm.return
    %b = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<16x32xf16, #shared, #smem, mutable>
    ttg.local_store %a, %b : tensor<16x32xf16, #linear> -> !ttg.memdesc<16x32xf16, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `linear_to_swizzled_st_matrix_trans_local_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `linear_to_swizzled_st_matrix_trans_local_store`。

### Line 577
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 579-588
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @fp8_const(%arg0: tensor<1024xi1, #blocked>, %arg1: tensor<1024xf8E4M3FNUZ, #blocked>) {
    // CHECK-LABEL: @fp8_const
    // CHECK: llvm.mlir.constant(0.000000e+00 : f8E4M3FNUZ) : i8
    %cst = arith.constant dense<0.000000e+00> : tensor<1024xf8E4M3FNUZ, #blocked>
    %a = arith.select %arg0, %arg1, %cst : tensor<1024xi1, #blocked>, tensor<1024xf8E4M3FNUZ, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fp8_const`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fp8_const`。

### Line 590
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 592-600
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @atomic_add_f32_nomask(%dest_ptrs: tensor<256x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %data: tensor<256xf32, #blocked>) {
    // CHECK-LABEL: atomic_add_f32_nomask
    // CHECK: atom.global.gpu.acq_rel.add.v4.f32
    %0 = tt.atomic_rmw fadd, acq_rel, gpu, %dest_ptrs, %data : (tensor<256x!tt.ptr<f32>, #blocked>, tensor<256xf32, #blocked>) -> tensor<256xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f32_nomask`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f32_nomask`。

### Line 602
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 604-613
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @atomic_add_f32_withmask(%dest_ptrs: tensor<256x!tt.ptr<f32>, #blocked> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %data: tensor<256xf32, #blocked>, %mask: tensor<256xi1, #blocked> {tt.constancy = 2 : i32}) {
    // CHECK-LABEL: atomic_add_f32_withmask
    // CHECK: atom.global.gpu.acq_rel.add.v2.f32
    // CHECK: atom.global.gpu.acq_rel.add.v2.f32
    %0 = tt.atomic_rmw fadd, acq_rel, gpu, %dest_ptrs, %data, %mask : (tensor<256x!tt.ptr<f32>, #blocked>, tensor<256xf32, #blocked>, tensor<256xi1, #blocked>) -> tensor<256xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f32_withmask`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f32_withmask`。

### Line 615
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 617-626
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @atomic_add_f16_withmask(%dest_ptrs: tensor<256x!tt.ptr<f16>, #blocked> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %data: tensor<256xf16, #blocked>, %mask: tensor<256xi1, #blocked> {tt.constancy = 4 : i32}) {
    // CHECK-LABEL: atomic_add_f16_withmask
    // CHECK: atom.global.gpu.acq_rel.add.noftz.v4.f16
    // CHECK: atom.global.gpu.acq_rel.add.noftz.v4.f16
    %0 = tt.atomic_rmw fadd, acq_rel, gpu, %dest_ptrs, %data, %mask : (tensor<256x!tt.ptr<f16>, #blocked>, tensor<256xf16, #blocked>, tensor<256xi1, #blocked>) -> tensor<256xf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f16_withmask`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f16_withmask`。

### Line 628
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 630-638
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 16]}>
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: test_fp8_to_fp16_dot_operand
  // CHECK-COUNT-16: cvt.rn.f16x2.e5m2x2
  tt.func @test_fp8_to_fp16_dot_operand(%arg: tensor<128x32xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>) {
    %r = tt.fp_to_fp %arg : tensor<128x32xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> -> tensor<128x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_fp8_to_fp16_dot_operand`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_fp8_to_fp16_dot_operand`。

### Line 640
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 642-649
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
#shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 4, order = [1, 0]}>
#shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 2, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {ttg.global_scratch_memory_alignment = 1 : i32, ttg.global_scratch_memory_size = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 4096 : i32, ttg.target = "cuda:90", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
  tt.func public @hopper_f64_mma_cvt() {
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<16x16xf64, #shared, #smem, mutable>
    %1 = ttg.local_alloc {allocation.offset = 2048 : i32} : () -> !ttg.memdesc<16x16xf64, #shared1, #smem, mutable>
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.total-num-warps`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `hopper_f64_mma_cvt`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.total-num-warps`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `hopper_f64_mma_cvt`。

### Line 651
```mlir
    %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf64, #mma>
```
**EN:** This block contributes intermediate IR built from `arith.constant`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `arith.constant` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 653
```mlir
    %2 = ttg.local_load %0 : !ttg.memdesc<16x16xf64, #shared, #smem, mutable> -> tensor<16x16xf64, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
```
**EN:** This block contributes intermediate IR built from `ttg.local_load`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_load`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 655
```mlir
    %3 = ttg.local_load %1 : !ttg.memdesc<16x16xf64, #shared1, #smem, mutable> -> tensor<16x16xf64, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
```
**EN:** This block contributes intermediate IR built from `ttg.local_load`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_load`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 657-660
```mlir
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m8n8k4.row.col.f64.f64.f64.f64
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m8n8k4.row.col.f64.f64.f64.f64
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-SAME`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-SAME`），从而保证后端/代码生成结果稳定。

### Line 662
```mlir
    %out = tt.dot %2, %3, %cst, inputPrecision = tf32 : tensor<16x16xf64, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<16x16xf64, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x16xf64, #mma>
```
**EN:** This block contributes intermediate IR built from `tt.dot`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 664-666
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 668
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 670
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Line 672
```mlir
module attributes {"ttg.target" = "cuda:90", "ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 674-679
```mlir
// CHECK-LABEL: @warpgroup_dot_wait_1_input
tt.func @warpgroup_dot_wait_1_input(%arg0: tensor<128xf32, #blocked>) {
  // CHECK: nvg.wgmma_wait_group
  ttng.warp_group_dot_wait %arg0 {pendings = 0 : i32} : tensor<128xf32, #blocked>
  tt.return
}
```
**EN:** This block defines `warpgroup_dot_wait_1_input` and exercises operations such as `tt.func`, `nvg.wgmma_wait_group`, `ttng.warp_group_dot_wait`, `tt.return`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warpgroup_dot_wait_1_input`，并覆盖 如 `tt.func`、`nvg.wgmma_wait_group`、`ttng.warp_group_dot_wait`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 681-685
```mlir
tt.func @warpgroup_dot_wait_2_inputs(%arg0: tensor<128xf32, #blocked>, %arg1: tensor<128xf32, #blocked>) {
  // CHECK: nvg.wgmma_wait_group
  ttng.warp_group_dot_wait %arg0, %arg1 {pendings = 0 : i32} : tensor<128xf32, #blocked>, tensor<128xf32, #blocked>
  tt.return
}
```
**EN:** This block defines `warpgroup_dot_wait_2_inputs` and exercises operations such as `tt.func`, `nvg.wgmma_wait_group`, `ttng.warp_group_dot_wait`, `tt.return`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `warpgroup_dot_wait_2_inputs`，并覆盖 如 `tt.func`、`nvg.wgmma_wait_group`、`ttng.warp_group_dot_wait`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 687
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on LLVM lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 LLVM 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory-nv='compute-capability=90`, `--convert-triton-gpu-to-llvm='compute-capability=90`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory-nv='compute-capability=90`, `--convert-triton-gpu-to-llvm='compute-capability=90` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `nvvm` (NVVM backend intrinsics), `tt` (Triton core ops), `llvm` (LLVM dialect IR), `ttng` (next-gen TritonGPU ops), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`nvvm`（NVVM 后端内建）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）、`ttng`（新一代 TritonGPU 操作）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `nvvm`, `tt`, `llvm`, `ttng`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`nvvm`、`tt`、`llvm`、`ttng`、`arith`。
