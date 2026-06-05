# tritongpu_wmma_dot_scaled_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/tritongpu_wmma_dot_scaled_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises WMMA lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的WMMA 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --convert-builtin-func-to-llvm | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×41, CHECK-COUNT×89, CHECK-LABEL×8, CHECK-NOT×2; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×41, CHECK-COUNT×89, CHECK-LABEL×8, CHECK-NOT×2；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that WMMA lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 WMMA 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --convert-builtin-func-to-llvm | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --convert-builtin-func-to-llvm | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s --split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250 --convert-builtin-func-to-llvm | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-6
```mlir
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
#linear1 = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
#mma = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, isTranspose = true, instrShape=[16, 16, 128]}>
#mma1 = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, isTranspose = true, instrShape=[16, 16, 64]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#linear`, `#linear1`, `#mma`, `#mma1`. They parameterize later tests with compact names for `#ttg.linear`, `#ttg.amd_wmma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#linear`, `#linear1`, `#mma`, `#mma1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.linear`, `#ttg.amd_wmma`。

### Lines 8-38
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  //  CHECK-LABEL: wmma_scaled_dot_fp4
  tt.func @wmma_scaled_dot_fp4(%arg0: tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 16}>>, %arg1: tensor<32x4xi8, #linear>, %arg2: tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 16}>>, %arg3: tensor<32x4xi8, #linear1>, %out0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
    // Matrix C
    // CHECK-COUNT-8:  llvm.insertelement {{.*}} : vector<8xf32>
    // Matrix A
    // CHECK-COUNT-32: llvm.extractvalue {{.*}} :  !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-32: llvm.insertelement {{.*}} : vector<32xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<32xi8> to vector<8xi32>
    // Matrix B
    // CHECK-COUNT-32: llvm.extractvalue {{.*}} :  !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-32: llvm.insertelement {{.*}} : vector<32xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<32xi8> to vector<8xi32>
    // Scale A
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // Scale B
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.wmma.scale.f32.16x16x128.f8f6f4"{{.*}} : (i32, vector<8xi32>, i32, vector<8xi32>, i16, vector<8xf32>, i32, i32, i32, i32, i32, i32, i1, i1) -> vector<8xf32>
    %c = tt.dot_scaled %arg0 scale %arg1, %arg2 scale %arg3, %cst lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 16}>>, tensor<32x4xi8, #linear> * tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 16}>>, tensor<32x4xi8, #linear1> -> tensor<32x32xf32, #mma>
    // CHECK-COUNT-8: llvm.extractelement {{.*}} : vector<8xf32>
    // CHECK-COUNT-8: llvm.insertelement {{.*}} : vector<1xf32>
    %ptr0 = tt.splat %out0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #mma>
    tt.store %ptr0, %c : tensor<32x32x!tt.ptr<f32>, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_scaled_dot_fp4`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_scaled_dot_fp4`。

### Line 40
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 42-45
```mlir
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
#linear1 = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
#mma = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, isTranspose = true, instrShape=[16, 16, 128]}>
#mma1 = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, isTranspose = true, instrShape=[16, 16, 64]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#linear`, `#linear1`, `#mma`, `#mma1`. They parameterize later tests with compact names for `#ttg.linear`, `#ttg.amd_wmma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#linear`, `#linear1`, `#mma`, `#mma1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.linear`, `#ttg.amd_wmma`。

### Lines 47-77
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: wmma_scaled_dot_fp4_fp8
  tt.func @wmma_scaled_dot_fp4_fp8(%arg0: tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 16}>>, %arg1: tensor<32x4xi8, #linear>, %arg2: tensor<128x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, %arg3: tensor<32x4xi8, #linear1>, %out0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
    // Matrix C
    // CHECK-COUNT-8:  llvm.insertelement {{.*}} : vector<8xf32>
    // Matrix A
    // CHECK-COUNT-32: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-32: llvm.insertelement {{.*}} : vector<32xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<32xi8> to vector<8xi32>
    // Matrix B
    // CHECK-COUNT-64: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8,  i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-64: llvm.insertelement {{.*}} : vector<64xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<64xi8> to vector<16xi32>
    // Scale A
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // Scale B
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.wmma.scale.f32.16x16x128.f8f6f4"{{.*}} : (i32, vector<16xi32>, i32, vector<8xi32>, i16, vector<8xf32>, i32, i32, i32, i32, i32, i32, i1, i1) -> vector<8xf32>
    %c = tt.dot_scaled %arg0 scale %arg1, %arg2 scale %arg3, %cst lhs = e2m1 rhs = e4m3 {fastMath = false} : tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 16}>>, tensor<32x4xi8, #linear> * tensor<128x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<32x4xi8, #linear1> -> tensor<32x32xf32, #mma>
    // CHECK-COUNT-8: llvm.extractelement {{.*}} : vector<8xf32>
    // CHECK-COUNT-8: llvm.insertelement {{.*}} : vector<1xf32>
    %ptr0 = tt.splat %out0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #mma>
    tt.store %ptr0, %c : tensor<32x32x!tt.ptr<f32>, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_scaled_dot_fp4_fp8`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_scaled_dot_fp4_fp8`。

### Line 79
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 81-83
```mlir
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
#linear1 = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
#mma = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, isTranspose = true, instrShape=[16, 16, 128]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#linear`, `#linear1`, `#mma`. They parameterize later tests with compact names for `#ttg.linear`, `#ttg.amd_wmma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#linear`, `#linear1`, `#mma`。它们为后续测试提供紧凑名称，用来表示 `#ttg.linear`, `#ttg.amd_wmma`。

### Lines 85-115
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: wmma_scaled_dot_fp8
  tt.func @wmma_scaled_dot_fp8(%arg0: tensor<32x128xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, %arg1: tensor<32x4xi8, #linear>, %arg2: tensor<128x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, %arg3: tensor<32x4xi8, #linear1>, %out0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
    // Matrix C
    // CHECK-COUNT-8:  llvm.insertelement {{.*}} : vector<8xf32>
    // Matrix A
    // CHECK-COUNT-64: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8,  i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-64: llvm.insertelement {{.*}} : vector<64xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<64xi8> to vector<16xi32>
    // Matrix B
    // CHECK-COUNT-64: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8,  i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-64: llvm.insertelement {{.*}} : vector<64xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<64xi8> to vector<16xi32>
    // Scale A
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // Scale B
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.wmma.scale.f32.16x16x128.f8f6f4"{{.*}} : (i32, vector<16xi32>, i32, vector<16xi32>, i16, vector<8xf32>, i32, i32, i32, i32, i32, i32, i1, i1) -> vector<8xf32>
    %c = tt.dot_scaled %arg0 scale %arg1, %arg2 scale %arg3, %cst lhs = e4m3 rhs = e4m3 {fastMath = false} : tensor<32x128xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<32x4xi8, #linear> * tensor<128x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<32x4xi8, #linear1> -> tensor<32x32xf32, #mma>
    // CHECK-COUNT-8: llvm.extractelement {{.*}} : vector<8xf32>
    // CHECK-COUNT-8: llvm.insertelement {{.*}} : vector<1xf32>
    %ptr0 = tt.splat %out0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #mma>
    tt.store %ptr0, %c : tensor<32x32x!tt.ptr<f32>, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_scaled_dot_fp8`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_scaled_dot_fp8`。

### Line 117
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 119-121
```mlir
#linear = #ttg.linear<{register = [[0, 1], [0, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
#linear1 = #ttg.linear<{register = [[0, 1], [0, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
#mma = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, isTranspose = true, instrShape=[16, 16, 128]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#linear`, `#linear1`, `#mma`. They parameterize later tests with compact names for `#ttg.linear`, `#ttg.amd_wmma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#linear`, `#linear1`, `#mma`。它们为后续测试提供紧凑名称，用来表示 `#ttg.linear`, `#ttg.amd_wmma`。

### Lines 123-157
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: wmma_scaled_dot_fp8_k64
  tt.func @wmma_scaled_dot_fp8_k64(%arg0: tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, %arg1: tensor<32x2xi8, #linear>, %arg2: tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, %arg3: tensor<32x2xi8, #linear1>, %out0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
    // Adjust for acc
    // CHECK: %[[ZERO:.*]] = llvm.mlir.constant(0 : i8) : i8
    // Matrix C
    // CHECK-COUNT-8:  llvm.insertelement {{.*}} : vector<8xf32>
    // Matrix A
    // CHECK-COUNT-32: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8,  i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-32: llvm.insertelement {{.*}} : vector<64xi8>
    // CHECK-COUNT-32: llvm.insertelement %[[ZERO]], {{.*}} : vector<64xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<64xi8> to vector<16xi32>
    // Matrix B
    // CHECK-COUNT-32: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8,  i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-32: llvm.insertelement {{.*}} : vector<64xi8>
    // CHECK-COUNT-32: llvm.insertelement %[[ZERO]], {{.*}} : vector<64xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<64xi8> to vector<16xi32>
    // Scale A
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // Scale B
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.wmma.scale.f32.16x16x128.f8f6f4"{{.*}} : (i32, vector<16xi32>, i32, vector<16xi32>, i16, vector<8xf32>, i32, i32, i32, i32, i32, i32, i1, i1) -> vector<8xf32>
    %c = tt.dot_scaled %arg0 scale %arg1, %arg2 scale %arg3, %cst lhs = e4m3 rhs = e4m3 {fastMath = false} : tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<32x2xi8, #linear> * tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<32x2xi8, #linear1> -> tensor<32x32xf32, #mma>
    // CHECK-COUNT-8: llvm.extractelement {{.*}} : vector<8xf32>
    // CHECK-COUNT-8: llvm.insertelement {{.*}} : vector<1xf32>
    %ptr0 = tt.splat %out0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #mma>
    tt.store %ptr0, %c : tensor<32x32x!tt.ptr<f32>, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_scaled_dot_fp8_k64`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_scaled_dot_fp8_k64`。

### Line 159
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 161-163
```mlir
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
#linear1 = #ttg.linear<{register = [[0, 1], [0, 2], [0, 4]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
#mma = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, isTranspose = true, instrShape=[16, 16, 128]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#linear`, `#linear1`, `#mma`. They parameterize later tests with compact names for `#ttg.linear`, `#ttg.amd_wmma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#linear`, `#linear1`, `#mma`。它们为后续测试提供紧凑名称，用来表示 `#ttg.linear`, `#ttg.amd_wmma`。

### Lines 165-212
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: wmma_scaled_dot_fp8_repeat_k
  tt.func @wmma_scaled_dot_fp8_repeat_k(%arg0: tensor<32x256xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, %arg1: tensor<32x8xi8, #linear>, %arg2: tensor<256x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, %arg3: tensor<32x8xi8, #linear1>, %out0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
    // Matrix C
    // CHECK-COUNT-8:  llvm.insertelement {{.*}} : vector<8xf32>
    // Matrix A
    // CHECK-COUNT-64: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-64: llvm.insertelement {{.*}} : vector<64xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<64xi8> to vector<16xi32>
    // Matrix B
    // CHECK-COUNT-64: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-64: llvm.insertelement {{.*}} : vector<64xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<64xi8> to vector<16xi32>
    // Scale A
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // Scale B
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.wmma.scale.f32.16x16x128.f8f6f4"{{.*}} : (i32, vector<16xi32>, i32, vector<16xi32>, i16, vector<8xf32>, i32, i32, i32, i32, i32, i32, i1, i1) -> vector<8xf32>
    // Matrix A
    // CHECK-COUNT-64: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-64: llvm.insertelement {{.*}} : vector<64xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<64xi8> to vector<16xi32>
    // Matrix B
    // CHECK-COUNT-64: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-64: llvm.insertelement {{.*}} : vector<64xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<64xi8> to vector<16xi32>
    // Scale A
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // Scale B
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.wmma.scale.f32.16x16x128.f8f6f4"{{.*}} : (i32, vector<16xi32>, i32, vector<16xi32>, i16, vector<8xf32>, i32, i32, i32, i32, i32, i32, i1, i1) -> vector<8xf32>
    %c = tt.dot_scaled %arg0 scale %arg1, %arg2 scale %arg3, %cst lhs = e4m3 rhs = e4m3 {fastMath = false} : tensor<32x256xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<32x8xi8, #linear> * tensor<256x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<32x8xi8, #linear1> -> tensor<32x32xf32, #mma>
    // CHECK-COUNT-8: llvm.extractelement {{.*}} : vector<8xf32>
    // CHECK-COUNT-8: llvm.insertelement {{.*}} : vector<1xf32>
    %ptr0 = tt.splat %out0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #mma>
    tt.store %ptr0, %c : tensor<32x32x!tt.ptr<f32>, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_scaled_dot_fp8_repeat_k`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_scaled_dot_fp8_repeat_k`。

### Line 214
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 216-218
```mlir
#linear = #ttg.linear<{register = [[0, 1], [0, 2], [64, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [32, 0]], block = []}>
#linear1 = #ttg.linear<{register = [[0, 1], [0, 2], [16, 0], [32, 0], [64, 0]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [0, 0]], block = []}>
#mma = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[1, 0], [2, 0]]}, isTranspose = true, instrShape=[16, 16, 128]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#linear`, `#linear1`, `#mma`. They parameterize later tests with compact names for `#ttg.linear`, `#ttg.amd_wmma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#linear`, `#linear1`, `#mma`。它们为后续测试提供紧凑名称，用来表示 `#ttg.linear`, `#ttg.amd_wmma`。

### Lines 220-238
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: wmma_scaled_dot_fp8_chained
  tt.func @wmma_scaled_dot_fp8_chained(%arg0: tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, %arg2: tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, %arg3: tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>, %out0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x128xf32, #mma>
    %scale0 = arith.constant dense<127> :  tensor<128x4xi8, #linear>
    %scale1 = arith.constant dense<127> :  tensor<128x4xi8, #linear1>
    // CHECK-COUNT-16: llvm.call_intrinsic "llvm.amdgcn.wmma.scale.f32.16x16x128.f8f6f4"{{.*}} : (i32, vector<16xi32>, i32, vector<16xi32>, i16, vector<8xf32>, i32, i32, i32, i32, i32, i32, i1, i1) -> vector<8xf32>
    %mm0 = tt.dot_scaled %arg0 scale %scale0, %arg2 scale %scale1, %cst lhs = e4m3 rhs = e4m3 {fastMath = false} : tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 16}>>, tensor<128x4xi8, #linear> * tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 16}>>, tensor<128x4xi8, #linear1> -> tensor<128x128xf32, #mma>
    // CHECK-NOT: rocdl.ds_swizzle
    // CHECK-NOT: llvm.call_intrinsic "llvm.amdgcn.permlane16.swap"
    %op0 = ttg.convert_layout %mm0 : tensor<128x128xf32, #mma> -> tensor<128x128xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    %op1 = tt.fp_to_fp %op0, rounding = rtne : tensor<128x128xf32, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>> -> tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>
    // CHECK-COUNT-16: llvm.call_intrinsic "llvm.amdgcn.wmma.scale.f32.16x16x128.f8f6f4"{{.*}} : (i32, vector<16xi32>, i32, vector<16xi32>, i16, vector<8xf32>, i32, i32, i32, i32, i32, i32, i1, i1) -> vector<8xf32>
    %mm1 = tt.dot_scaled %op1 scale %scale0, %arg3 scale %scale1, %cst lhs = e4m3 rhs = e4m3 {fastMath = false} : tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 8}>>, tensor<128x4xi8, #linear> * tensor<128x128xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 8}>>, tensor<128x4xi8, #linear1> -> tensor<128x128xf32, #mma>
    %ptr0 = tt.splat %out0 : !tt.ptr<f32> -> tensor<128x128x!tt.ptr<f32>, #mma>
    tt.store %ptr0, %mm1 : tensor<128x128x!tt.ptr<f32>, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_scaled_dot_fp8_chained`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_scaled_dot_fp8_chained`。

### Line 240
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 242-247
```mlir
// Scale A: 32x16 WMMA — all 32 lanes cover M (depth=1), no broadcast
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[0, 0], [32, 0]], block = []}>
// Scale B: 16 lanes cover N (depth=2), 1 broadcast bit
#linear1 = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[16, 0], [0, 0]], block = []}>
#mma = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, isTranspose = false, instrShape=[32, 16, 128]}>
#mma1 = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, isTranspose = false, instrShape=[32, 16, 64]}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 249-279
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: wmma_scaled_dot_fp4_32x16
  tt.func @wmma_scaled_dot_fp4_32x16(%arg0: tensor<64x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 16}>>, %arg1: tensor<64x4xi8, #linear>, %arg2: tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 16}>>, %arg3: tensor<32x4xi8, #linear1>, %out0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant dense<0.000000e+00> : tensor<64x32xf32, #mma>
    // Matrix C
    // CHECK-COUNT-16: llvm.insertelement {{.*}} : vector<16xf32>
    // Matrix A
    // CHECK-COUNT-64: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-64: llvm.insertelement {{.*}} : vector<64xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<64xi8> to vector<16xi32>
    // Matrix B
    // CHECK-COUNT-32: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-32: llvm.insertelement {{.*}} : vector<32xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<32xi8> to vector<8xi32>
    // Scale A
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // Scale B
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.wmma.scale.f32.32x16x128.f4"{{.*}} : (vector<16xi32>, vector<8xi32>, i16, vector<16xf32>, i32, i32, i32, i32, i32, i32, i1, i1) -> vector<16xf32>
    %c = tt.dot_scaled %arg0 scale %arg1, %arg2 scale %arg3, %cst lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<64x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 16}>>, tensor<64x4xi8, #linear> * tensor<64x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 16}>>, tensor<32x4xi8, #linear1> -> tensor<64x32xf32, #mma>
    // CHECK-COUNT-16: llvm.extractelement {{.*}} : vector<16xf32>
    // CHECK-COUNT-16: llvm.insertelement {{.*}} : vector<1xf32>
    %ptr0 = tt.splat %out0 : !tt.ptr<f32> -> tensor<64x32x!tt.ptr<f32>, #mma>
    tt.store %ptr0, %c : tensor<64x32x!tt.ptr<f32>, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_scaled_dot_fp4_32x16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_scaled_dot_fp4_32x16`。

### Line 281
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 283-289
```mlir
// 32x16 WMMA with isTranspose=true
// Scale A 16 lanes cover M, 1 broadcast bit.
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [0, 0]], warp = [[0, 0], [16, 0]], block = []}>
// Scale B 32 lanes cover N, no broadcast.
#linear1 = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[1, 0], [2, 0], [4, 0], [8, 0], [16, 0]], warp = [[32, 0], [0, 0]], block = []}>
#mma = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, isTranspose = true, instrShape=[32, 16, 128]}>
#mma1 = #ttg.amd_wmma<{version = 3, ctaLayout = {warp = [[0, 1], [1, 0]]}, isTranspose = true, instrShape=[32, 16, 64]}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 291-321
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "hip:gfx1250", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: wmma_scaled_dot_fp4_32x16_transposed
  tt.func @wmma_scaled_dot_fp4_32x16_transposed(%arg0: tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 16}>>, %arg1: tensor<32x4xi8, #linear>, %arg2: tensor<64x64xi8, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 16}>>, %arg3: tensor<64x4xi8, #linear1>, %out0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x64xf32, #mma>
    // Matrix C
    // CHECK-COUNT-16: llvm.insertelement {{.*}} : vector<16xf32>
    // Matrix A (frontend A -> slot-2, kBase=32 i8 after swap; vector<32xi8>)
    // CHECK-COUNT-32: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-32: llvm.insertelement {{.*}} : vector<32xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<32xi8> to vector<8xi32>
    // Matrix B (frontend B -> slot-1, kBase=64 i8 after swap; vector<64xi8>)
    // CHECK-COUNT-64: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8, i8)>
    // CHECK-COUNT-64: llvm.insertelement {{.*}} : vector<64xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<64xi8> to vector<16xi32>
    // Scale A
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // Scale B
    // CHECK-COUNT-4: llvm.extractvalue {{.*}} : !llvm.struct<(i8, i8, i8, i8)>
    // CHECK-COUNT-4: llvm.insertelement {{.*}} : vector<4xi8>
    // CHECK: llvm.bitcast {{.*}} : vector<4xi8> to i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.wmma.scale.f32.32x16x128.f4"{{.*}} : (vector<16xi32>, vector<8xi32>, i16, vector<16xf32>, i32, i32, i32, i32, i32, i32, i1, i1) -> vector<16xf32>
    %c = tt.dot_scaled %arg0 scale %arg1, %arg2 scale %arg3, %cst lhs = e2m1 rhs = e2m1 {fastMath = false} : tensor<32x64xi8, #ttg.dot_op<{opIdx = 0, parent = #mma1, kWidth = 16}>>, tensor<32x4xi8, #linear> * tensor<64x64xi8, #ttg.dot_op<{opIdx = 1, parent = #mma1, kWidth = 16}>>, tensor<64x4xi8, #linear1> -> tensor<32x64xf32, #mma>
    // CHECK-COUNT-16: llvm.extractelement {{.*}} : vector<16xf32>
    // CHECK-COUNT-16: llvm.insertelement {{.*}} : vector<1xf32>
    %ptr0 = tt.splat %out0 : !tt.ptr<f32> -> tensor<32x64x!tt.ptr<f32>, #mma>
    tt.store %ptr0, %c : tensor<32x64x!tt.ptr<f32>, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `hip:gfx1250`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `wmma_scaled_dot_fp4_32x16_transposed`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `hip:gfx1250`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `wmma_scaled_dot_fp4_32x16_transposed`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on WMMA lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 WMMA 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--split-input-file`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250`, `--convert-builtin-func-to-llvm`.  
  **CN:** `RUN` 流水线会驱动 `--split-input-file`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx1250`, `--convert-builtin-func-to-llvm` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `arith` (scalar/tensor arithmetic), `llvm` (LLVM dialect IR).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`arith`（标量/张量算术）、`llvm`（LLVM 方言 IR）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `arith`, `llvm`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`arith`、`llvm`。
