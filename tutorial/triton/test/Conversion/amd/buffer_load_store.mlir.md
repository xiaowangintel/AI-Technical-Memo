# buffer_load_store.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/amd/buffer_load_store.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises buffer operations in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的缓冲区操作相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck %s`<br>`triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×36, CHECK-COUNT×14, CHECK-LABEL×18, CHECK-NOT×3; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×36, CHECK-COUNT×14, CHECK-LABEL×18, CHECK-NOT×3；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that buffer operations produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 缓冲区操作 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck %s
// RUN: triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck %s; triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx942 | FileCheck %s; triton-opt %s -split-input-file --convert-triton-amdgpu-to-llvm=gfx-arch=gfx950 | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-15
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
    // CHECK-LABEL: buffer_load
    tt.func @buffer_load(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %offset : tensor<128xi32, #blocked0>{tt.divisibility=16:i32}) {
        // CHECK: %[[c_mask:.*]] = llvm.mlir.constant(true) : i1
        // CHECK: %[[offset:.*]] = llvm.select %[[c_mask]]
        // CHECK: %[[aux:.*]] = llvm.mlir.constant(3 : i32) : i32
        // CHECK: rocdl.raw.ptr.buffer.load {{.*}}, %[[offset]], {{.*}}, %[[aux]]
        %ret = amdg.buffer_load %arg0[%offset] cacheModifier = cs : tensor<128xf32, #blocked0>
        tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load`。

### Line 17
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 19-37
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
    // CHECK-LABEL: buffer_load_mask
    tt.func @buffer_load_mask(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %offset : tensor<128xi32, #blocked0> {tt.divisibility=16:i32}, %N : i32 {tt.divisibility = 16 : i32}) {
        %c256_i32 = arith.constant 256 : i32
        %0 = tt.get_program_id x : i32
        %1 = arith.muli %0, %c256_i32 : i32
        %2 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked0>
        %3 = tt.splat %1 : i32 -> tensor<128xi32, #blocked0>
        %4 = arith.addi %3, %2 : tensor<128xi32, #blocked0>
        %5 = tt.splat %N: i32 -> tensor<128xi32, #blocked0>
        %7 = arith.cmpi slt, %4, %5: tensor<128xi32, #blocked0>
        // CHECK: %[[mask:.*]] = llvm.extractvalue %{{.*}} : !llvm.struct<(i1, i1, i1, i1)>
        // CHECK: %[[offset:.*]] = llvm.select %[[mask]]
        // CHECK: rocdl.raw.ptr.buffer.load {{.*}}, %[[offset]]
        %ret = amdg.buffer_load %arg0[%offset], %7 stride = %c256_i32 : tensor<128xf32, #blocked0>
        tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_mask`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_mask`。

### Line 39
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 41-61
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
    // CHECK-LABEL: buffer_load_mask_other
    tt.func @buffer_load_mask_other(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %offset : tensor<128xi32, #blocked0> {tt.divisibility=16:i32}, %N : i32 {tt.divisibility = 16 : i32}) {
        %c256_i32 = arith.constant 256 : i32
        %0 = tt.get_program_id x : i32
        %1 = arith.muli %0, %c256_i32 : i32
        %2 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked0>
        %3 = tt.splat %1 : i32 -> tensor<128xi32, #blocked0>
        %4 = arith.addi %3, %2 : tensor<128xi32, #blocked0>
        %5 = tt.splat %N: i32 -> tensor<128xi32, #blocked0>
        %7 = arith.cmpi slt, %4, %5: tensor<128xi32, #blocked0>
        %other = arith.constant dense<0.00e+00> : tensor<128xf32, #blocked0>
        // CHECK: %[[mask:.*]] = llvm.extractvalue %{{.*}} : !llvm.struct<(i1, i1, i1, i1)>
        // CHECK: %[[offset:.*]] = llvm.select %[[mask]]
        // CHECK: rocdl.raw.ptr.buffer.load {{.*}}, %[[offset]]
        // CHECK: llvm.select
        %ret = amdg.buffer_load %arg0[%offset], %7, %other stride = %c256_i32: tensor<128xf32, #blocked0>
        tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_mask_other`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_mask_other`。

### Line 63
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 65-77
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
    // CHECK-LABEL: buffer_store
    tt.func @buffer_store(%value : tensor<128xf32, #blocked0>, %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %offset : tensor<128xi32, #blocked0>{tt.divisibility=16:i32}) {
        // CHECK: %[[mask:.*]] = llvm.mlir.constant(true) : i1
        // CHECK: %[[offset:.*]] = llvm.select %[[mask]]
        // CHECK: %[[aux:.*]] = llvm.mlir.constant(3 : i32) : i32
        // CHECK: rocdl.raw.ptr.buffer.store {{.*}}, {{.*}}, %[[offset]], {{.*}}, %[[aux]]
        %c256_i32 = arith.constant 256 : i32
        amdg.buffer_store %value, %arg0[%offset] cacheModifier = cs stride = %c256_i32 : tensor<128xf32, #blocked0>
        tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_store`。

### Line 79
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 81-101
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
    // CHECK-LABEL: buffer_store_mask
    tt.func @buffer_store_mask(%value : tensor<128xf32, #blocked0>, %arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %offset : tensor<128xi32, #blocked0> {tt.divisibility=16:i32}, %N : i32 {tt.divisibility = 16 : i32}) {
        %c256_i32 = arith.constant 256 : i32
        %0 = tt.get_program_id x : i32
        %1 = arith.muli %0, %c256_i32 : i32
        %2 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked0>
        %3 = tt.splat %1 : i32 -> tensor<128xi32, #blocked0>
        %4 = arith.addi %3, %2 : tensor<128xi32, #blocked0>
        %5 = tt.splat %N: i32 -> tensor<128xi32, #blocked0>
        %7 = arith.cmpi slt, %4, %5: tensor<128xi32, #blocked0>
        // CHECK: %[[mask0:.*]] = llvm.extractvalue %{{.*}} : !llvm.struct<(i1, i1, i1, i1)>
        // CHECK: %[[mask1:.*]] = llvm.mlir.constant(true) : i1
        // CHECK: %[[mask2:.*]] = llvm.and %[[mask1]], %[[mask0]]
        // CHECK: %[[offset:.*]] = llvm.select %[[mask2]]
        // CHECK: rocdl.raw.ptr.buffer.store {{.*}}, {{.*}}, %[[offset]]
        amdg.buffer_store %value, %arg0[%offset], %7 stride = %N : tensor<128xf32, #blocked0>
        tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_store_mask`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_store_mask`。

### Line 103
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 105-127
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: buffer_load_store_vec4
    tt.func @buffer_load_store_vec4(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32) {
        %c256_i32 = arith.constant 256 : i32
        %0 = tt.get_program_id x : i32
        %1 = arith.muli %0, %c256_i32 : i32
        %2 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
        %3 = tt.splat %1 : i32 -> tensor<256xi32, #blocked0>
        %4 = arith.addi %3, %2 : tensor<256xi32, #blocked0>
        // Load 8 elements from A with two vectorized load instructions
        // CHECK-COUNT-2: rocdl.raw.ptr.buffer.load {{.*}} : vector<4xf32>
        %9 = amdg.buffer_load %arg0[%4] stride = %arg3 : tensor<256xf32, #blocked0>
        // Load 8 elements from B with two vectorized load instructions
        // CHECK-COUNT-2: rocdl.raw.ptr.buffer.load {{.*}} : vector<4xf32>
        %10 = amdg.buffer_load %arg1[%4] stride = %arg3 : tensor<256xf32, #blocked0>
        %11 = arith.addf %9, %10 : tensor<256xf32, #blocked0>
        // Store 8 elements into C with two vectorized store instructions
        // CHECK-COUNT-2: rocdl.raw.ptr.buffer.store {{.*}} : vector<4xf32>
        amdg.buffer_store %11, %arg2[%4] stride = %arg3 : tensor<256xf32, #blocked0>
        tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_store_vec4`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_store_vec4`。

### Line 129
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 131-147
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 8], warpsPerCTA = [8, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: buffer_load_8xf16
  tt.func public @buffer_load_8xf16(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg2: i32 {tt.divisibility = 16 : i32}, %arg8: i32 {tt.divisibility = 16 : i32}) {
    %0 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %1 = tt.splat %arg2 : i32 -> tensor<256x64xi32, #blocked>
    %2 = tt.expand_dims %0 {axis = 0 : i32} : tensor<64xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x64xi32, #blocked>
    %3 = tt.broadcast %2 : tensor<1x64xi32, #blocked> -> tensor<256x64xi32, #blocked>
    %4 = arith.addi %3, %1 : tensor<256x64xi32, #blocked>
    // Load 16 f16 elements check for correct vector size of instruction (4xi32 = 8xf16)
    // CHECK-COUNT-4: rocdl.raw.ptr.buffer.load {{.*}} : vector<4xi32>
    %5 = amdg.buffer_load %arg0[%4] : tensor<256x64xf16, #blocked>
    // CHECK-COUNT-4: rocdl.raw.ptr.buffer.store {{.*}} : vector<4xi32>
    amdg.buffer_store %5, %arg0[%4] : tensor<256x64xf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_8xf16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_8xf16`。

### Line 149
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 151-175
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: buffer_load_store_vec1
    tt.func @buffer_load_store_vec1(%arg0: !tt.ptr<f32> , %arg1: !tt.ptr<f32>, %arg2: !tt.ptr<f32>, %arg3: i32) {
        %c256_i32 = arith.constant 256 : i32
        %0 = tt.get_program_id x : i32
        %1 = arith.muli %0, %c256_i32 : i32
        %2 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
        %3 = tt.splat %1 : i32 -> tensor<256xi32, #blocked0>
        %4 = arith.addi %3, %2 : tensor<256xi32, #blocked0>
        %5 = tt.splat %arg3 : i32 -> tensor<256xi32, #blocked0>
        %7 = arith.cmpi slt, %4, %5: tensor<256xi32, #blocked0>
        // Load 8 elements from A with eight scalar load instructions
        // CHECK-COUNT-8: rocdl.raw.ptr.buffer.load {{.*}} : f32
        %9 = amdg.buffer_load %arg0[%4], %7 stride = %arg3 : tensor<256xf32, #blocked0>
        // Load 8 elements from B with two scalar load instructions
        // CHECK-COUNT-8: rocdl.raw.ptr.buffer.load {{.*}} : f32
        %10 = amdg.buffer_load %arg1[%4], %7 stride = %arg3 : tensor<256xf32, #blocked0>
        %11 = arith.addf %9, %10 : tensor<256xf32, #blocked0>
        // Store 8 elements into C with two scalar store instructions
        // CHECK-COUNT-8: rocdl.raw.ptr.buffer.store {{.*}} : f32
        amdg.buffer_store %11, %arg2[%4], %7 stride = %arg3 : tensor<256xf32, #blocked0>
        tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_store_vec1`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_store_vec1`。

### Line 177
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 179-203
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
    // CHECK-LABEL: buffer_load_store_vec2
    tt.func @buffer_load_store_vec2(%arg0: !tt.ptr<f16> {tt.divisibility = 4 : i32}, %arg1: !tt.ptr<f16>{tt.divisibility = 4 : i32}, %arg2: !tt.ptr<f16>{tt.divisibility = 4: i32}, %arg3: i32{tt.divisibility = 4: i32}) {
        %c256_i32 = arith.constant 256 : i32
        %0 = tt.get_program_id x : i32
        %1 = arith.muli %0, %c256_i32 : i32
        %2 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
        %3 = tt.splat %1 : i32 -> tensor<256xi32, #blocked0>
        %4 = arith.addi %3, %2 : tensor<256xi32, #blocked0>
        %5 = tt.splat %arg3 : i32 -> tensor<256xi32, #blocked0>
        %7 = arith.cmpi slt, %4, %5: tensor<256xi32, #blocked0>
        // Load 8 fp16 elements from A with four i32 scalar load instructions
        // CHECK-COUNT-4: rocdl.raw.ptr.buffer.load {{.*}} : i32
        %9 = amdg.buffer_load %arg0[%4], %7 stride = %arg3 : tensor<256xf16, #blocked0>
        // Load 8 fp16 elements from B with four i32 scalar load instructions
        // CHECK-COUNT-4: rocdl.raw.ptr.buffer.load {{.*}} : i32
        %10 = amdg.buffer_load %arg1[%4], %7 stride = %arg3 : tensor<256xf16, #blocked0>
        %11 = arith.addf %9, %10 : tensor<256xf16, #blocked0>
        // Store 8 fp16 elements into C with four i32 scalar store instructionss
        // CHECK-COUNT-4: rocdl.raw.ptr.buffer.store {{.*}} : i32
        amdg.buffer_store %11, %arg2[%4], %7 stride = %arg3 : tensor<256xf16, #blocked0>
        tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_store_vec2`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_store_vec2`。

### Line 205
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 207-224
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
    // CHECK-LABEL: buffer_atomic
    tt.func @buffer_atomic_rmw_fadd(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %offset : tensor<128xi32, #blocked0>{tt.divisibility=16:i32}, %N: i32, %values : tensor<128xf32, #blocked0>, %stride: i32 {tt.divisibility=16:i32}) {
        %c128_i32 = arith.constant 128 : i32
        %0 = tt.get_program_id x : i32
        %1 = arith.muli %0, %c128_i32 : i32
        %2 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #blocked0>
        %3 = tt.splat %1 : i32 -> tensor<128xi32, #blocked0>
        %4 = arith.addi %3, %2 : tensor<128xi32, #blocked0>
        %5 = tt.splat %N: i32 -> tensor<128xi32, #blocked0>
        %mask = arith.cmpi slt, %4, %5: tensor<128xi32, #blocked0>
        // CHECK: %[[mask0:.*]] = llvm.extractvalue %{{.*}} : !llvm.struct<(i1, i1, i1, i1)>
        // There should be a single release fence before any atomics
        // CHECK: llvm.fence syncscope("agent") release
        // CHECK: %[[mask1:.*]] = llvm.mlir.constant(true) : i1
        // CHECK: %[[mask2:.*]] = llvm.and %[[mask1]], %[[mask0]]
        // CHECK: %[[offset:.*]] = llvm.select %[[mask2]]
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_atomic_rmw_fadd`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_atomic_rmw_fadd`。

### Lines 226-228
```mlir
        // We will have 4 calls to fadd, since the sizePerThread is 4. Scope/ordering instructions will be
        // generated by the lowering of llvm.fence
        %ret = amdg.buffer_atomic_rmw fadd, acq_rel, gpu, %values, %arg0[%offset], %mask stride = %stride : tensor<128xf32, #blocked0>
```
**EN:** This block contributes intermediate IR built from `llvm.fence`, `amdg.buffer_atomic_rmw`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `llvm.fence`, `amdg.buffer_atomic_rmw` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 230-233
```mlir
        // CHECK: %[[result:.*]] = llvm.call_intrinsic "llvm.amdgcn.raw.ptr.buffer.atomic.fadd"({{.*}}, {{.*}}, %[[mask1:.*]], {{.*}}, {{.*}}) : (f32, !llvm.ptr<8>, i32, i32, i32) -> f32
        // CHECK: %[[result:.*]] = llvm.call_intrinsic "llvm.amdgcn.raw.ptr.buffer.atomic.fadd"({{.*}}, {{.*}}, %[[mask1:.*]], {{.*}}, {{.*}}) : (f32, !llvm.ptr<8>, i32, i32, i32) -> f32
        // CHECK: %[[result:.*]] = llvm.call_intrinsic "llvm.amdgcn.raw.ptr.buffer.atomic.fadd"({{.*}}, {{.*}}, %[[mask1:.*]], {{.*}}, {{.*}}) : (f32, !llvm.ptr<8>, i32, i32, i32) -> f32
        // CHECK: %[[result:.*]] = llvm.call_intrinsic "llvm.amdgcn.raw.ptr.buffer.atomic.fadd"({{.*}}, {{.*}}, %[[mask1:.*]], {{.*}}, {{.*}}) : (f32, !llvm.ptr<8>, i32, i32, i32) -> f32
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 235-239
```mlir
        // There should be a single acquire fence after all of the atomics
        // CHECK: llvm.fence syncscope("agent") acquire
        tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises operations such as `llvm.fence`, `tt.return`. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 如 `llvm.fence`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 241
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 243-251
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
    // CHECK-LABEL: buffer_atomic_rmw_fmax_f32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.raw.ptr.buffer.atomic.fmax"({{.*}}) : (f32, !llvm.ptr<8>, i32, i32, i32) -> f32
    tt.func public @buffer_atomic_rmw_fmax_f32(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %offsets : tensor<256xi32, #blocked0>{tt.divisibility=16:i32}, %values : tensor<256xf32, #blocked0>) {
        %ret = amdg.buffer_atomic_rmw max, acq_rel, gpu, %values, %arg0[%offsets] : tensor<256xf32, #blocked0>
        tt.return
    }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_atomic_rmw_fmax_f32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_atomic_rmw_fmax_f32`。

### Line 253
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 255-263
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
    // CHECK-LABEL: buffer_atomic_rmw_fmin_f32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.raw.ptr.buffer.atomic.fmin"({{.*}}) : (f32, !llvm.ptr<8>, i32, i32, i32) -> f32
    tt.func public @buffer_atomic_rmw_fmin_f32(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %offsets : tensor<256xi32, #blocked0>{tt.divisibility=16:i32}, %values : tensor<256xf32, #blocked0>) {
        %ret = amdg.buffer_atomic_rmw min, acq_rel, gpu, %values, %arg0[%offsets] : tensor<256xf32, #blocked0>
        tt.return
    }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_atomic_rmw_fmin_f32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_atomic_rmw_fmin_f32`。

### Line 265
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 267-275
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
    // CHECK-LABEL: buffer_atomic_rmw_smax_i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.raw.ptr.buffer.atomic.smax"({{.*}}) : (i32, !llvm.ptr<8>, i32, i32, i32) -> i32
    tt.func public @buffer_atomic_rmw_smax_i32(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %offsets : tensor<256xi32, #blocked0>{tt.divisibility=16:i32}, %values : tensor<256xi32, #blocked0>) {
        %ret = amdg.buffer_atomic_rmw max, acq_rel, gpu, %values, %arg0[%offsets] : tensor<256xi32, #blocked0>
        tt.return
    }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_atomic_rmw_smax_i32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_atomic_rmw_smax_i32`。

### Line 277
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 279-287
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
    // CHECK-LABEL: buffer_atomic_rmw_smin_i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.raw.ptr.buffer.atomic.smin"({{.*}}) : (i32, !llvm.ptr<8>, i32, i32, i32) -> i32
    tt.func public @buffer_atomic_rmw_smin_i32(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %offsets : tensor<256xi32, #blocked0>{tt.divisibility=16:i32}, %values : tensor<256xi32, #blocked0>) {
        %ret = amdg.buffer_atomic_rmw min, acq_rel, gpu, %values, %arg0[%offsets] : tensor<256xi32, #blocked0>
        tt.return
    }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_atomic_rmw_smin_i32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_atomic_rmw_smin_i32`。

### Line 289
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 291-299
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
    // CHECK-LABEL: buffer_atomic_rmw_umax_i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.raw.ptr.buffer.atomic.umax"({{.*}}) : (i32, !llvm.ptr<8>, i32, i32, i32) -> i32
    tt.func public @buffer_atomic_rmw_umax_i32(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %offsets : tensor<256xi32, #blocked0>{tt.divisibility=16:i32}, %values : tensor<256xi32, #blocked0>) {
        %ret = amdg.buffer_atomic_rmw umax, acq_rel, gpu, %values, %arg0[%offsets] : tensor<256xi32, #blocked0>
        tt.return
    }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_atomic_rmw_umax_i32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_atomic_rmw_umax_i32`。

### Line 301
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 303-311
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
    // CHECK-LABEL: buffer_atomic_rmw_umin_i32
    // CHECK: llvm.call_intrinsic "llvm.amdgcn.raw.ptr.buffer.atomic.umin"({{.*}}) : (i32, !llvm.ptr<8>, i32, i32, i32) -> i32
    tt.func public @buffer_atomic_rmw_umin_i32(%arg0: !tt.ptr<i32> {tt.divisibility = 16 : i32}, %offsets : tensor<256xi32, #blocked0>{tt.divisibility=16:i32}, %values : tensor<256xi32, #blocked0>) {
        %ret = amdg.buffer_atomic_rmw umin, acq_rel, gpu, %values, %arg0[%offsets] : tensor<256xi32, #blocked0>
        tt.return
    }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_atomic_rmw_umin_i32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_atomic_rmw_umin_i32`。

### Line 313
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 315-330
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 64], warpsPerCTA = [1, 4], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
    // CHECK-LABEL: buffer_load_layout_vectorization
    tt.func public @buffer_load_layout_vectorization(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
        %c1_i32 = arith.constant 1 : i32
        %21 = tt.splat %c1_i32 : i32 -> tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
        %22 = tt.expand_dims %21 {axis = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x16xi32, #blocked>
        %23 = tt.broadcast %22 : tensor<1x16xi32, #blocked> -> tensor<8x16xi32, #blocked>
        // Each thread has to load 8xi16
        // We expect vector size == 1 (i16) for the generated loads as sizePerThread = [1, 1]
        // CHECK-COUNT-8: rocdl.raw.ptr.buffer.load {{.*}}, {{.*}}, {{.*}}, {{.*}} : i16
        // CHECK-NOT: rocdl.raw.ptr.buffer.load
        %24 = amdg.buffer_load %arg0[%23] : tensor<8x16xf16, #blocked>
        tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `buffer_load_layout_vectorization`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `buffer_load_layout_vectorization`。

### Line 332
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 334-349
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [64], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 64 : i32} {
  // CHECK-LABEL: strided_buffer_load_and_store
  tt.func public @strided_buffer_load_and_store(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32, tt.pointer_range = 32 : i32}) {
    %cst = arith.constant dense<2> : tensor<1024xi32, #blocked>
    %0 = tt.make_range {end = 1024 : i32, start = 0 : i32} : tensor<1024xi32, #blocked>
    %1 = arith.muli %0, %cst : tensor<1024xi32, #blocked>
    // CHECK-COUNT-4: rocdl.raw.ptr.buffer.load {{.*}}, {{.*}}, {{.*}}, {{.*}} : f32
    // CHECK-NOT: rocdl.raw.ptr.buffer.load
    %2 = amdg.buffer_load %arg0[%1] : tensor<1024xf32, #blocked>
    // CHECK-COUNT-4: rocdl.raw.ptr.buffer.store {{.*}}, {{.*}}, {{.*}}, {{.*}}, {{.*}} : f32
    // CHECK-NOT: rocdl.raw.ptr.buffer.store
    amdg.buffer_store %2, %arg1[%1] : tensor<1024xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `strided_buffer_load_and_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `strided_buffer_load_and_store`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on buffer operations.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 缓冲区操作。
- **EN:** The `RUN` pipeline drives tools/passes such as `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950`.  
  **CN:** `RUN` 流水线会驱动 `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx942`, `--convert-triton-amdgpu-to-llvm=gfx-arch=gfx950` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `tt` (Triton core ops), `llvm` (LLVM dialect IR), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`tt`（Triton 核心操作）、`llvm`（LLVM 方言 IR）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `tt`, `llvm`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`tt`、`llvm`、`arith`。
