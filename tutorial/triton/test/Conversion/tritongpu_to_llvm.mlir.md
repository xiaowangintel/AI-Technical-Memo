# tritongpu_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Conversion/tritongpu_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises LLVM lowering in Triton's conversion/lowering coverage. **CN:** 该MLIR 测试用于覆盖 Triton 转换/降级中的LLVM 降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file --allocate-shared-memory-nv --convert-triton-gpu-to-llvm -reconcile-unrealized-casts 2>/dev/null | FileCheck %s --dump-input-context 20` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×392, CHECK-COUNT×24, CHECK-DAG×3, CHECK-LABEL×137, CHECK-NEXT×12, CHECK-NOT×55, CHECK-SAME×63; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×392, CHECK-COUNT×24, CHECK-DAG×3, CHECK-LABEL×137, CHECK-NEXT×12, CHECK-NOT×55, CHECK-SAME×63；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that LLVM lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 LLVM 降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file --allocate-shared-memory-nv --convert-triton-gpu-to-llvm -reconcile-unrealized-casts 2>/dev/null | FileCheck %s --dump-input-context 20
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file --allocate-shared-memory-nv --convert-triton-gpu-to-llvm -reconcile-unrealized-casts 2>/dev/null | FileCheck %s --dump-input-context 20` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file --allocate-shared-memory-nv --convert-triton-gpu-to-llvm -reconcile-unrealized-casts 2>/dev/null | FileCheck %s --dump-input-context 20`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-11
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK: llvm.func @test_empty_kernel(%arg0: i32, %arg1: !llvm.ptr<1> {tt.pointee_type = f16}, %arg2: !llvm.ptr<1>, %arg3: !llvm.ptr<1>)
  // Here the 128 comes from the 4 in module attribute multiples 32
  // CHECK: nvvm.kernel = 1 : ui1, nvvm.reqntid = array<i32: 128>
  tt.func @test_empty_kernel(%lb : index, %A : !tt.ptr<f16>) {
    // CHECK:  llvm.return
    tt.return
  }
} // end module
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_empty_kernel`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_empty_kernel`。

### Line 13
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 15-26
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: basic_load
  tt.func @basic_load(%a_ptr_init : tensor<256x!tt.ptr<f32>, #blocked0>, %cst : tensor<256xi1, #blocked0>, %cst_0 : tensor<256xf32, #blocked0>) {
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mov.u32 $0, $1;
    // CHECK-SAME: @$3 ld.global.b32 { $0 }, [ $2 + 0 ];", "=r,r,l,b"
    // CHECK: llvm.inline_asm
    %1 = tt.load %a_ptr_init, %cst, %cst_0 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `, `. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_load`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `, `。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_load`。

### Line 28
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 30-41
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: vectorized_load
  tt.func @vectorized_load(%a_ptr_init : tensor<256x!tt.ptr<f32>, #blocked0>, %cst : tensor<256xi1, #blocked0>, %cst_0 : tensor<256xf32, #blocked0>) {
    // CHECK: llvm.inline_asm
    // CHECK-SAME: ld.global.b32
    // CHECK: llvm.inline_asm
    // CHECK-SAME: ld.global.b32
    %1 = tt.load %a_ptr_init, %cst, %cst_0 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `vectorized_load`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `vectorized_load`。

### Line 43
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 45-56
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: vectorized_load_f16
  tt.func @vectorized_load_f16(%a_ptr_init: tensor<256x!tt.ptr<f16>, #blocked0>, %cst : tensor<256xi1, #blocked0>, %cst_0 : tensor<256xf16, #blocked0>) {
    // CHECK: llvm.inline_asm
    // CHECK-SAME: ld.global.b16
    // CHECK: llvm.inline_asm
    // CHECK-SAME: ld.global.b16
    %1 = tt.load %a_ptr_init, %cst, %cst_0 : tensor<256x!tt.ptr<f16>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `vectorized_load_f16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `vectorized_load_f16`。

### Line 58
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 60-69
```mlir
// TODO: masked load with vectorization is pending on TODO
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: masked_load_const_other
  tt.func @masked_load_const_other(%a_ptr_init : tensor<256x!tt.ptr<f32>, #blocked0>, %cst : tensor<256xi1, #blocked0>) {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<256xf32, #blocked0>
    %1 = tt.load %a_ptr_init, %cst, %cst_0 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `masked_load_const_other`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `masked_load_const_other`。

### Line 71
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 73-82
```mlir
// TODO: masked load with vectorization is pending on TODO
#blocked0 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: masked_load_const_other_vec
  tt.func @masked_load_const_other_vec(%a_ptr_init : tensor<256x!tt.ptr<f32>, #blocked0>, %cst : tensor<256xi1, #blocked0>) {
    %cst_0 = arith.constant dense<0.000000e+00> : tensor<256xf32, #blocked0>
    %1 = tt.load %a_ptr_init, %cst, %cst_0 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `masked_load_const_other_vec`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `masked_load_const_other_vec`。

### Line 84
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 86-97
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: store_with_cache_attr
  tt.func @store_with_cache_attr(%a_ptr_init : tensor<256x!tt.ptr<f32>, #blocked0>, %cst : tensor<256xi1, #blocked0>, %cst_0 : tensor<256xf32, #blocked0>) {
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "mov.u64 $0, 0x0;\0A\09createpolicy.fractional.L2::evict_last.b64 $0, 1.0;"
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "@$3 st.global.L1::evict_last.L2::cache_hint.b32 [ $1 + 0 ], { $0 }, $2;"
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "mov.u64 $0, 0x0;\0A\09createpolicy.fractional.L2::evict_last.b64 $0, 1.0;"
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "@$3 st.global.L1::evict_last.L2::cache_hint.b32 [ $1 + 0 ], { $0 }, $2;"
    tt.store %a_ptr_init, %cst_0, %cst evictionPolicy = evict_last cacheModifier = ca : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `store_with_cache_attr`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `store_with_cache_attr`。

### Line 99
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 101-112
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: load_with_l2_cache_hint
  tt.func @load_with_l2_cache_hint(%a_ptr_init : tensor<256x!tt.ptr<f32>, #blocked0>, %cst : tensor<256xi1, #blocked0>, %cst_0 : tensor<256xf32, #blocked0>) {
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "mov.u64 $0, 0x0;\0A\09createpolicy.fractional.L2::evict_first.b64 $0, 1.0;"
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "mov.u32 $0, $1;\0A\09@$4 ld.global.L1::evict_first.L2::cache_hint.b32 { $0 }, [ $2 + 0 ], $3;"
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "mov.u64 $0, 0x0;\0A\09createpolicy.fractional.L2::evict_first.b64 $0, 1.0;"
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "mov.u32 $0, $1;\0A\09@$4 ld.global.L1::evict_first.L2::cache_hint.b32 { $0 }, [ $2 + 0 ], $3;"
      %1 = tt.load %a_ptr_init, %cst, %cst_0 evictionPolicy = evict_first : tensor<256x!tt.ptr<f32>, #blocked0>
      tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `load_with_l2_cache_hint`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `load_with_l2_cache_hint`。

### Line 114
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 115-126
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: store_with_l2_cache_hint
  tt.func @store_with_l2_cache_hint(%a_ptr_init : tensor<256x!tt.ptr<f32>, #blocked0>, %cst : tensor<256xi1, #blocked0>, %cst_0 : tensor<256xf32, #blocked0>) {
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "mov.u64 $0, 0x0;\0A\09createpolicy.fractional.L2::evict_last.b64 $0, 1.0;"
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "@$3 st.global.L1::evict_last.L2::cache_hint.b32 [ $1 + 0 ], { $0 }, $2;"
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "mov.u64 $0, 0x0;\0A\09createpolicy.fractional.L2::evict_last.b64 $0, 1.0;"
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att {{.*}} "@$3 st.global.L1::evict_last.L2::cache_hint.b32 [ $1 + 0 ], { $0 }, $2;"
      tt.store %a_ptr_init, %cst_0, %cst evictionPolicy = evict_last : tensor<256x!tt.ptr<f32>, #blocked0>
      tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `store_with_l2_cache_hint`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `store_with_l2_cache_hint`。

### Line 128
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 130-143
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
  // CHECK-LABEL: global_load_store_no_vec
  tt.func @global_load_store_no_vec(%arg0: !tt.ptr<f32> {tt.divisibility = 4 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 4 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 4 : i32}, %arg3: i32) {
    %c256_i32 = arith.constant 256 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c256_i32 : i32
    %2 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
    %3 = tt.splat %1 : i32 -> tensor<256xi32, #blocked0>
    %4 = arith.addi %3, %2 : tensor<256xi32, #blocked0>
    %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %6 = tt.addptr %5, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
    %7 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %8 = tt.addptr %7, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `global_load_store_no_vec`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `global_load_store_no_vec`。

### Lines 145-153
```mlir
    // Load 4 elements from vector0
    // CHECK: mov.u32 $0, 0x0
    // CHECK: ld.global.b32 { ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: mov.u32 $0, 0x0
    // CHECK: ld.global.b32 { ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: mov.u32 $0, 0x0
    // CHECK: ld.global.b32 { ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: mov.u32 $0, 0x0
    // CHECK: ld.global.b32 { ${{.*}} }, [ ${{.*}} + 0 ];
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 155-168
```mlir
    // Load 4 elements from vector1
    // CHECK: mov.u32 $0, 0x0
    // CHECK: ld.global.b32 { ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: mov.u32 $0, 0x0
    // CHECK: ld.global.b32 { ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: mov.u32 $0, 0x0
    // CHECK: ld.global.b32 { ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: mov.u32 $0, 0x0
    // CHECK: ld.global.b32 { ${{.*}} }, [ ${{.*}} + 0 ];
    %9 = tt.load %6 : tensor<256x!tt.ptr<f32>, #blocked0>
    %10 = tt.load %8 : tensor<256x!tt.ptr<f32>, #blocked0>
    %11 = arith.addf %9, %10 : tensor<256xf32, #blocked0>
    %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %13 = tt.addptr %12, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
```
**EN:** This block defines the test function(s) and exercises tensor loads, pointer arithmetic. Embedded check comments (CHECK×8) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量加载、指针运算。 其中嵌入的检查注释（CHECK×8）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 170-178
```mlir
    // Store 4 elements to global
    // CHECK: st.global.b32 [ ${{.*}} + 0 ], { ${{.*}} };
    // CHECK: st.global.b32 [ ${{.*}} + 0 ], { ${{.*}} };
    // CHECK: st.global.b32 [ ${{.*}} + 0 ], { ${{.*}} };
    // CHECK: st.global.b32 [ ${{.*}} + 0 ], { ${{.*}} };
    tt.store %13, %11 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises tensor stores. Embedded check comments (CHECK×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 180
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 182-195
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
  // CHECK-LABEL: global_load_store_vec4
  tt.func @global_load_store_vec4(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32) {
    %c256_i32 = arith.constant 256 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c256_i32 : i32
    %2 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
    %3 = tt.splat %1 : i32 -> tensor<256xi32, #blocked0>
    %4 = arith.addi %3, %2 : tensor<256xi32, #blocked0>
    %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %6 = tt.addptr %5, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
    %7 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %8 = tt.addptr %7, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `global_load_store_vec4`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `global_load_store_vec4`。

### Lines 197-198
```mlir
    // Load 4 elements from A with single one vectorized load instruction
    // CHECK: ld.global.v4.b32 { ${{.*}}, ${{.*}}, ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 200-201
```mlir
    // Load 4 elements from B with single one vectorized load instruction
    // CHECK: ld.global.v4.b32 { ${{.*}}, ${{.*}}, ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 203-207
```mlir
    %9 = tt.load %6 : tensor<256x!tt.ptr<f32>, #blocked0>
    %10 = tt.load %8 : tensor<256x!tt.ptr<f32>, #blocked0>
    %11 = arith.addf %9, %10 : tensor<256xf32, #blocked0>
    %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %13 = tt.addptr %12, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
```
**EN:** This block contributes intermediate IR built from `tt.load`, `tt.ptr`, `arith.addf`, `tt.splat`, `tt.addptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.load`, `tt.ptr`, `arith.addf`, `tt.splat`, `tt.addptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 209-214
```mlir
    // Store 4 elements to global with single one vectorized store instruction
    // CHECK: st.global.v4.b32 [ ${{.*}} + 0 ], { ${{.*}}, ${{.*}}, ${{.*}}, ${{.*}} };
    tt.store %13, %11 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises tensor stores. Embedded check comments (CHECK×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 216
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 218-245
```mlir
// This test verifies the vectorization of Load and Store Ops.
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
// Note, the %n_elements doesn't have a "tt.divisibility" hint, so Triton assumes it's divisibility is 1, this should effect the mask's alignment and further restrict the load/store ops' vector width to be 1.
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
  tt.func @vecadd_masked_vec1(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %n_elements: i32) {
    %c64_i32 = arith.constant 64 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c64_i32 : i32
    %2 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #blocked>
    %3 = tt.splat %1 : i32 -> tensor<64xi32, #blocked>
    %4 = arith.addi %3, %2 : tensor<64xi32, #blocked>
    %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>, #blocked>
    %6 = tt.addptr %5, %4 : tensor<64x!tt.ptr<f32>, #blocked>, tensor<64xi32, #blocked>
    %7 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>, #blocked>
    %8 = tt.addptr %7, %4 : tensor<64x!tt.ptr<f32>, #blocked>, tensor<64xi32, #blocked>
    %9 = tt.splat %n_elements : i32 -> tensor<64xi32, #blocked>
    %10 = arith.cmpi "slt", %4, %9 : tensor<64xi32, #blocked>
    // load op has a vector width = 1 due to the %mask's alignment
    // CHECK: ld.global.b32
    %11 = tt.load %6, %10 : tensor<64x!tt.ptr<f32>, #blocked>
    %12 = tt.load %8, %10 : tensor<64x!tt.ptr<f32>, #blocked>
    %13 = arith.addf %11, %12 : tensor<64xf32, #blocked>
    %14 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<64x!tt.ptr<f32>, #blocked>
    %15 = tt.addptr %14, %4 : tensor<64x!tt.ptr<f32>, #blocked>, tensor<64xi32, #blocked>
    tt.store %15, %13, %10 : tensor<64x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `vecadd_masked_vec1`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `vecadd_masked_vec1`。

### Line 247
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 249-262
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: global_load_store_vec2
    tt.func @global_load_store_vec2(%arg0: !tt.ptr<f32> {tt.divisibility = 8 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 8 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 8 : i32}, %arg3: i32) {
    %c256_i32 = arith.constant 256 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c256_i32 : i32
    %2 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
    %3 = tt.splat %1 : i32 -> tensor<256xi32, #blocked0>
    %4 = arith.addi %3, %2 : tensor<256xi32, #blocked0>
    %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %6 = tt.addptr %5, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
    %7 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %8 = tt.addptr %7, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `global_load_store_vec2`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `global_load_store_vec2`。

### Lines 264-268
```mlir
    // Load 8 elements from A with four vectorized load instruction
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 270-274
```mlir
    // Load 8 elements from B with four vectorized load instruction
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 276-280
```mlir
    %9 = tt.load %6 : tensor<256x!tt.ptr<f32>, #blocked0>
    %10 = tt.load %8 : tensor<256x!tt.ptr<f32>, #blocked0>
    %11 = arith.addf %9, %10 : tensor<256xf32, #blocked0>
    %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %13 = tt.addptr %12, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
```
**EN:** This block contributes intermediate IR built from `tt.load`, `tt.ptr`, `arith.addf`, `tt.splat`, `tt.addptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.load`, `tt.ptr`, `arith.addf`, `tt.splat`, `tt.addptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 282-290
```mlir
    // Store 8 elements to global with four vectorized store instruction
    // CHECK: st.global.v2.b32 [ ${{.*}} + 0 ], { ${{.*}}, ${{.*}} };
    // CHECK: st.global.v2.b32 [ ${{.*}} + 0 ], { ${{.*}}, ${{.*}} };
    // CHECK: st.global.v2.b32 [ ${{.*}} + 0 ], { ${{.*}}, ${{.*}} };
    // CHECK: st.global.v2.b32 [ ${{.*}} + 0 ], { ${{.*}}, ${{.*}} };
    tt.store %13, %11 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises tensor stores. Embedded check comments (CHECK×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 292
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 294-307
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
    // CHECK-LABEL: global_load_store_vec2
    tt.func @global_load_store_vec2(%arg0: !tt.ptr<f32> {tt.divisibility = 8 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 8 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 8 : i32}, %arg3: i32) {
    %c256_i32 = arith.constant 256 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c256_i32 : i32
    %2 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
    %3 = tt.splat %1 : i32 -> tensor<256xi32, #blocked0>
    %4 = arith.addi %3, %2 : tensor<256xi32, #blocked0>
    %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %6 = tt.addptr %5, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
    %7 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %8 = tt.addptr %7, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `global_load_store_vec2`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `global_load_store_vec2`。

### Lines 309-313
```mlir
    // Load 8 elements from A with four vectorized load instruction
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 315-319
```mlir
    // Load 8 elements from B with four vectorized load instruction
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v2.b32 { ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 321-325
```mlir
    %9 = tt.load %6 : tensor<256x!tt.ptr<f32>, #blocked0>
    %10 = tt.load %8 : tensor<256x!tt.ptr<f32>, #blocked0>
    %11 = arith.addf %9, %10 : tensor<256xf32, #blocked0>
    %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %13 = tt.addptr %12, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
```
**EN:** This block contributes intermediate IR built from `tt.load`, `tt.ptr`, `arith.addf`, `tt.splat`, `tt.addptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.load`, `tt.ptr`, `arith.addf`, `tt.splat`, `tt.addptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 327-335
```mlir
    // Store 8 elements to global with four vectorized store instruction
    // CHECK: st.global.v2.b32 [ ${{.*}} + 0 ], { ${{.*}}, ${{.*}} };
    // CHECK: st.global.v2.b32 [ ${{.*}} + 0 ], { ${{.*}}, ${{.*}} };
    // CHECK: st.global.v2.b32 [ ${{.*}} + 0 ], { ${{.*}}, ${{.*}} };
    // CHECK: st.global.v2.b32 [ ${{.*}} + 0 ], { ${{.*}}, ${{.*}} };
    tt.store %13, %11 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises tensor stores. Embedded check comments (CHECK×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 337
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 339-352
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [8], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: global_load_store_vec8
    tt.func @global_load_store_vec8(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 16 : i32}, %arg3: i32) {
    %c256_i32 = arith.constant 256 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c256_i32 : i32
    %2 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
    %3 = tt.splat %1 : i32 -> tensor<256xi32, #blocked0>
    %4 = arith.addi %3, %2 : tensor<256xi32, #blocked0>
    %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %6 = tt.addptr %5, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
    %7 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %8 = tt.addptr %7, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `global_load_store_vec8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `global_load_store_vec8`。

### Lines 354-356
```mlir
    // Load 8 elements from A with two vectorized load instruction
    // CHECK: ld.global.v4.b32 { ${{.*}}, ${{.*}}, ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v4.b32 { ${{.*}}, ${{.*}}, ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 358-360
```mlir
    // Load 8 elements from B with two vectorized load instruction
    // CHECK: ld.global.v4.b32 { ${{.*}}, ${{.*}}, ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
    // CHECK: ld.global.v4.b32 { ${{.*}}, ${{.*}}, ${{.*}}, ${{.*}} }, [ ${{.*}} + 0 ];
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 362-366
```mlir
    %9 = tt.load %6 : tensor<256x!tt.ptr<f32>, #blocked0>
    %10 = tt.load %8 : tensor<256x!tt.ptr<f32>, #blocked0>
    %11 = arith.addf %9, %10 : tensor<256xf32, #blocked0>
    %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>, #blocked0>
    %13 = tt.addptr %12, %4 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
```
**EN:** This block contributes intermediate IR built from `tt.load`, `tt.ptr`, `arith.addf`, `tt.splat`, `tt.addptr`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.load`, `tt.ptr`, `arith.addf`, `tt.splat`, `tt.addptr` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 368-374
```mlir
    // Store 8 elements to global with two vectorized store instruction
    // CHECK: st.global.v4.b32 [ ${{.*}} + 0 ], { ${{.*}}, ${{.*}}, ${{.*}}, ${{.*}} };
    // CHECK: st.global.v4.b32 [ ${{.*}} + 0 ], { ${{.*}}, ${{.*}}, ${{.*}}, ${{.*}} };
    tt.store %13, %11 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises tensor stores. Embedded check comments (CHECK×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量存储。 其中嵌入的检查注释（CHECK×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 376
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 378-380
```mlir
// Slice layout with 2 unique elements, but 8 total elements per thread
#blocked2d = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [2, 1], order = [0, 1]}>
#slice = #ttg.slice<{dim = 1, parent = #blocked2d}>
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Lines 382-394
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
  // CHECK-LABEL: global_load_store_slice
  tt.func @global_load_store_slice(%arg0: !tt.ptr<f32> {tt.divisibility = 4 : i32}, %arg1: !tt.ptr<f32> {tt.divisibility = 4 : i32}, %arg2: !tt.ptr<f32> {tt.divisibility = 4 : i32}, %arg3: i32) {
    %c128_i32 = arith.constant 128 : i32
    %0 = tt.get_program_id x : i32
    %1 = arith.muli %0, %c128_i32 : i32
    %2 = tt.make_range {end = 128 : i32, start = 0 : i32} : tensor<128xi32, #slice>
    %3 = tt.splat %1 : i32 -> tensor<128xi32, #slice>
    %4 = arith.addi %3, %2 : tensor<128xi32, #slice>
    %5 = tt.splat %arg0 : !tt.ptr<f32> -> tensor<128x!tt.ptr<f32>, #slice>
    %6 = tt.addptr %5, %4 : tensor<128x!tt.ptr<f32>, #slice>, tensor<128xi32, #slice>
    %7 = tt.splat %arg1 : !tt.ptr<f32> -> tensor<128x!tt.ptr<f32>, #slice>
    %8 = tt.addptr %7, %4 : tensor<128x!tt.ptr<f32>, #slice>, tensor<128xi32, #slice>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `global_load_store_slice`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `global_load_store_slice`。

### Lines 396-399
```mlir
    // Load 2 element from vector0 without predicate
    // CHECK: mov.u32 $0, 0x0
    // CHECK-NOT: @{{.*}} ld.global
    // CHECK-COUNT-2: ld.global.b32 { ${{.*}} }, [ ${{.*}} + 0 ];
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-COUNT`, `CHECK-NOT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-COUNT`, `CHECK-NOT`），从而保证后端/代码生成结果稳定。

### Lines 401-409
```mlir
    // Load 2 elements from vector1 without predicate
    // CHECK: mov.u32 $0, 0x0
    // CHECK-NOT: @{{.*}} ld.global
    // CHECK-COUNT-2: ld.global.b32 { ${{.*}} }, [ ${{.*}} + 0 ];
    %9 = tt.load %6 : tensor<128x!tt.ptr<f32>, #slice>
    %10 = tt.load %8 : tensor<128x!tt.ptr<f32>, #slice>
    %11 = arith.addf %9, %10 : tensor<128xf32, #slice>
    %12 = tt.splat %arg2 : !tt.ptr<f32> -> tensor<128x!tt.ptr<f32>, #slice>
    %13 = tt.addptr %12, %4 : tensor<128x!tt.ptr<f32>, #slice>, tensor<128xi32, #slice>
```
**EN:** This block defines the test function(s) and exercises tensor loads, pointer arithmetic. Embedded check comments (CHECK×1, CHECK-COUNT×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量加载、指针运算。 其中嵌入的检查注释（CHECK×1, CHECK-COUNT×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 411-417
```mlir
    // Store 2 element to global without predicate
    // CHECK-NOT: @{{.*}} st.global
    // CHECK-COUNT-2: st.global.b32 [ ${{.*}} + 0 ], { ${{.*}} };
    tt.store %13, %11 : tensor<128x!tt.ptr<f32>, #slice>
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises tensor stores. Embedded check comments (CHECK-COUNT×1, CHECK-NOT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 张量存储。 其中嵌入的检查注释（CHECK-COUNT×1, CHECK-NOT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 419-420
```mlir
// TODO: Add a testcase to verify the optimization when ptr of the LoadOp
//       is from an addptr with const idx
```
**EN:** This comment block provides context for the surrounding test logic: `// TODO: Add a testcase to verify the optimization when ptr of the LoadOp`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// TODO: Add a testcase to verify the optimization when ptr of the LoadOp`。

### Line 422
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 424-445
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: basic_view_broadcast
  tt.func @basic_view_broadcast(%arg : tensor<256xf32,#blocked0>) {
    // CHECK: llvm.mlir.undef
    // CHECK: %[[T0:.*]] = llvm.extractvalue
    // CHECK: %[[T1:.*]] = llvm.extractvalue
    %0 = tt.reshape %arg allow_reorder : tensor<256xf32, #blocked0> -> tensor<256x1xf32,#blocked2>
    // CHECK: llvm.mlir.undef
    // CHECK: llvm.insertvalue %[[T0]]
    // CHECK: llvm.insertvalue %[[T1]]
    // CHECK: llvm.insertvalue %[[T0]]
    // CHECK: llvm.insertvalue %[[T1]]
    // CHECK: llvm.insertvalue %[[T0]]
    // CHECK: llvm.insertvalue %[[T1]]
    // CHECK: llvm.insertvalue %[[T0]]
    // CHECK: llvm.insertvalue %[[T1]]
    %1 = tt.broadcast %0 : tensor<256x1xf32,#blocked2> -> tensor<256x4xf32, #blocked2>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_view_broadcast`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_view_broadcast`。

### Line 447
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 449-460
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: basic_make_range
  tt.func @basic_make_range() {
    // CHECK: nvvm.read.ptx.sreg.tid.x
    // CHECK: llvm.mlir.undef
    // CHECK: llvm.insertvalue
    // CHECK: llvm.insertvalue
    %0 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_make_range`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_make_range`。

### Line 463
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 465-478
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: sliced_layout_make_range
  tt.func @sliced_layout_make_range() {
    // CHECK: nvvm.read.ptx.sreg.tid.x
    // CHECK: llvm.mlir.undef
    // CHECK: llvm.insertvalue
    // CHECK: llvm.insertvalue
    // CHECK: llvm.insertvalue
    // CHECK: llvm.insertvalue
    %0 = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #ttg.slice<{dim = 0, parent = #blocked0}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `sliced_layout_make_range`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `sliced_layout_make_range`。

### Line 480
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 482-491
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: basic_addf
  tt.func @basic_addf(%arg0 : tensor<256xf32,#blocked0>, %arg1 : tensor<256xf32,#blocked0>) {
    // CHECK: llvm.fadd
    // CHECK: llvm.fadd
    %1 = arith.addf %arg0, %arg1 : tensor<256xf32,#blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_addf`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_addf`。

### Line 493
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 495-504
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: basic_addi
  tt.func @basic_addi(%arg0 : tensor<256xi32,#blocked0>, %arg1 : tensor<256xi32,#blocked0>) {
    // CHECK: llvm.add
    // CHECK: llvm.add
    %1 = arith.addi %arg0, %arg1 : tensor<256xi32,#blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_addi`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_addi`。

### Line 506
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 508-515
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: basic_program_id
  tt.func @basic_program_id() {
    // CHECK: nvvm.read.ptx.sreg.ctaid.x : i32
    %0 = tt.get_program_id x : i32
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_program_id`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_program_id`。

### Line 517
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 519-528
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: basic_addptr
  tt.func @basic_addptr(%arg0 : tensor<256x!tt.ptr<f32>,#blocked0>, %arg1 : tensor<256xi32,#blocked0>) {
    // CHECK: llvm.getelementptr
    // CHECK: llvm.getelementptr
    %0 = tt.addptr %arg0, %arg1 : tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xi32, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_addptr`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_addptr`。

### Line 530
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 532-543
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK: llvm.mlir.global external @global_smem
  // CHECK-LABEL: basic_alloc_tensor
  tt.func @basic_alloc_tensor() {
    // CHECK: llvm.mlir.addressof @global_smem
    // CHECK: llvm.getelementptr
    %0 = ttg.local_alloc : () -> !ttg.memdesc<16x16xf16, #shared0, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_alloc_tensor`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_alloc_tensor`。

### Line 545
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 547-567
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK: llvm.mlir.global external @global_smem
  // CHECK-LABEL: rank_reducing_subview
  tt.func @rank_reducing_subview() {
    // CHECK: llvm.mlir.addressof @global_smem
    // CHECK: llvm.mlir.constant(512 : i32) : i32
    // CHECK-NEXT: llvm.mul
    // CHECK-NEXT: llvm.extractvalue
    // CHECK-NEXT: llvm.extractvalue
    // CHECK-NEXT: llvm.extractvalue
    // CHECK-NEXT: llvm.extractvalue
    // CHECK-NEXT: llvm.getelementptr
    %index = arith.constant 1 : i32
    %zero = arith.constant 0 : i32
    %0 = ttg.local_alloc : () -> !ttg.memdesc<128x16x32xf32, #shared0, #smem, mutable>
    %1 = ttg.memdesc_index %0[%index] : !ttg.memdesc<128x16x32xf32, #shared0, #smem, mutable> -> !ttg.memdesc<16x32xf32, #shared0, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `rank_reducing_subview`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `rank_reducing_subview`。

### Line 569
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 571-578
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: basic_async_wait
  tt.func @basic_async_wait() {
    // CHECK: nvvm.cp.async.wait.group 4
    ttg.async_wait {num = 4: i32}
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_async_wait`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_async_wait`。

### Line 580
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 582-614
```mlir
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 8], order = [0, 1]}>
#slice1d0 = #ttg.slice<{dim = 0, parent = #blocked1}>
#shared1D = #ttg.swizzled_shared<{vec = 2, perPhase = 1, maxPhase = 8, order = [0]}>
#shared2D = #ttg.swizzled_shared<{vec = 2, perPhase = 1, maxPhase = 8, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: basic_insert_slice_async_1d
  tt.func @basic_insert_slice_async_1d(%arg0: !tt.ptr<i64> {tt.divisibility = 16 : i32}) {
    %c0_i32 = arith.constant 0 : i32
    %cst_2 = arith.constant dense<64> : tensor<64xi32, #slice1d0>
    %58 = tt.splat %arg0 : !tt.ptr<i64> -> tensor<64x!tt.ptr<i64>, #slice1d0>
    %24 = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #slice1d0>
    %59 = tt.addptr %58, %24 : tensor<64x!tt.ptr<i64>, #slice1d0>, tensor<64xi32, #slice1d0>
    %66 = tt.addptr %59, %cst_2 : tensor<64x!tt.ptr<i64>, #slice1d0>, tensor<64xi32, #slice1d0>
    %71 = ttg.local_alloc : () -> !ttg.memdesc<2x64xi64, #shared1D, #smem, mutable>
    %subview = ttg.memdesc_index %71[%c0_i32] :
      !ttg.memdesc<2x64xi64, #shared1D, #smem, mutable> ->
      !ttg.memdesc<64xi64, #shared1D, #smem, mutable>
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att
    // CHECK-SAME: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x8, 0x8
    // CHECK: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x8, 0x8
    // CHECK: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x8, 0x8
    // CHECK: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x8, 0x8
    // CHECK: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x8, 0x8
    // CHECK: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x8, 0x8
    // CHECK: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x8, 0x8
    // CHECK: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x8, 0x8
    // CHECK: nvvm.cp.async.commit.group
    %73 = ttg.async_copy_global_to_local %66, %subview : tensor<64x!tt.ptr<i64>, #slice1d0> -> !ttg.memdesc<64xi64, #shared1D, #smem, mutable>
    ttg.async_commit_group tokens %73
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_insert_slice_async_1d`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_insert_slice_async_1d`。

### Line 616
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 618-628
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared1D = #ttg.swizzled_shared<{vec = 2, perPhase = 1, maxPhase = 8, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: async_cp_contiguity_hint
  tt.func @async_cp_contiguity_hint(%v: tensor<256x!tt.ptr<f16>, #blocked>, %smem: !ttg.memdesc<256xf16, #shared1D, #smem, mutable>) {
    // CHECK: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x8, 0x8
    %0 = ttg.async_copy_global_to_local %v, %smem {contiguity = 4 : i32} : tensor<256x!tt.ptr<f16>, #blocked> -> !ttg.memdesc<256xf16, #shared1D, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `async_cp_contiguity_hint`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `async_cp_contiguity_hint`。

### Line 631
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 633-660
```mlir
#block0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#block1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#block2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#block3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [4, 8], warpsPerCTA = [1, 4], order = [1, 0]}>
#slice2d1 = #ttg.slice<{dim = 1, parent=#block2}>
#slice3d0 = #ttg.slice<{dim = 0, parent=#block3}>
#AL = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#A = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: basic_insert_slice_async_v4
  tt.func @basic_insert_slice_async_v4(%arg0: !tt.ptr<f32> {tt.divisibility = 32 : i32}) {
    %off0_ = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #slice2d1>
    %off1_ = tt.make_range {end = 64 : i32, start = 0 : i32} : tensor<64xi32, #slice3d0>
    %off0 = tt.expand_dims %off0_ {axis = 1 : i32} : tensor<16xi32, #slice2d1> -> tensor<16x1xi32, #block2>
    %off1 = tt.expand_dims %off1_ {axis = 0 : i32} : tensor<64xi32, #slice3d0> -> tensor<1x64xi32, #block3>
    %broadcast_off0_scalar = tt.broadcast %off0 : tensor<16x1xi32, #block2> -> tensor<16x64xi32, #block2>
    %cst_scalar = arith.constant 64 : i32
    %cst = tt.splat %cst_scalar : i32 -> tensor<16x64xi32, #block2>
    %broadcast_off0_ = arith.muli %broadcast_off0_scalar, %cst : tensor<16x64xi32, #block2>
    %broadcast_off1_ = tt.broadcast %off1 : tensor<1x64xi32, #block3> -> tensor<16x64xi32, #block3>
    %broadcast_off0 = ttg.convert_layout %broadcast_off0_ : tensor<16x64xi32, #block2> -> tensor<16x64xi32, #AL>
    %broadcast_off1 = ttg.convert_layout %broadcast_off1_ : tensor<16x64xi32, #block3> -> tensor<16x64xi32, #AL>
    %off = arith.addi %broadcast_off0, %broadcast_off1 : tensor<16x64xi32, #AL>
    %a_init = tt.splat %arg0 : !tt.ptr<f32> -> tensor<16x64x!tt.ptr<f32>, #AL>
    %a_ptr = tt.addptr %a_init, %off : tensor<16x64x!tt.ptr<f32>, #AL>, tensor<16x64xi32, #AL>
    %tensor = ttg.local_alloc : () -> !ttg.memdesc<16x64xf32, #A, #smem, mutable>
    %index = arith.constant 1 : i32
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_insert_slice_async_v4`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_insert_slice_async_v4`。

### Lines 662-669
```mlir
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "cp.async.cg.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x10, 0x10;"
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "cp.async.cg.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x10, 0x10;"
    // CHECK: nvvm.cp.async.commit.group
    %a = ttg.async_copy_global_to_local %a_ptr, %tensor : tensor<16x64x!tt.ptr<f32>, #AL> -> !ttg.memdesc<16x64xf32, #A, #smem, mutable>
    ttg.async_commit_group
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies, backend inline assembly, NVIDIA-specific codegen. Embedded check comments (CHECK×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝、后端内联汇编、NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 671
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 673-700
```mlir
#block0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#block1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#block2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#block3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [4, 8], warpsPerCTA = [1, 4], order = [1, 0]}>
#slice2d1 = #ttg.slice<{dim = 1, parent=#block2}>
#slice3d0 = #ttg.slice<{dim = 0, parent=#block3}>
#AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#A = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: basic_insert_slice_async_v1
  tt.func @basic_insert_slice_async_v1(%arg0: !tt.ptr<f32> {tt.divisibility = 4 : i32}) {
    %off0_ = tt.make_range {end = 16 : i32, start = 0 : i32} : tensor<16xi32, #slice2d1>
    %off1_ = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #slice3d0>
    %off0 = tt.expand_dims %off0_ {axis = 1 : i32} : tensor<16xi32, #slice2d1> -> tensor<16x1xi32, #block2>
    %off1 = tt.expand_dims %off1_ {axis = 0 : i32} : tensor<32xi32, #slice3d0> -> tensor<1x32xi32, #block3>
    %broadcast_off0_scalar = tt.broadcast %off0 : tensor<16x1xi32, #block2> -> tensor<16x32xi32, #block2>
    %cst_scalar = arith.constant 32 : i32
    %cst = tt.splat %cst_scalar : i32 -> tensor<16x32xi32, #block2>
    %broadcast_off0_ = arith.muli %broadcast_off0_scalar, %cst : tensor<16x32xi32, #block2>
    %broadcast_off1_ = tt.broadcast %off1 : tensor<1x32xi32, #block3> -> tensor<16x32xi32, #block3>
    %broadcast_off0 = ttg.convert_layout %broadcast_off0_ : tensor<16x32xi32, #block2> -> tensor<16x32xi32, #AL>
    %broadcast_off1 = ttg.convert_layout %broadcast_off1_ : tensor<16x32xi32, #block3> -> tensor<16x32xi32, #AL>
    %off = arith.addi %broadcast_off0, %broadcast_off1 : tensor<16x32xi32, #AL>
    %a_init = tt.splat %arg0 : !tt.ptr<f32> -> tensor<16x32x!tt.ptr<f32>, #AL>
    %a_ptr = tt.addptr %a_init, %off : tensor<16x32x!tt.ptr<f32>, #AL>, tensor<16x32xi32, #AL>
    %tensor = ttg.local_alloc : () -> !ttg.memdesc<16x32xf32, #A, #smem, mutable>
    %index = arith.constant 1 : i32
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_insert_slice_async_v1`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_insert_slice_async_v1`。

### Lines 702-715
```mlir
    // CHECK: llvm.inline_asm
    // CHECK: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4
    // CHECK: llvm.inline_asm
    // CHECK-SAME: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4
    // CHECK: llvm.inline_asm
    // CHECK-SAME: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4
    // CHECK: llvm.inline_asm
    // CHECK-SAME: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4
    // CHECK: nvvm.cp.async.commit.group
    %a = ttg.async_copy_global_to_local %a_ptr, %tensor : tensor<16x32x!tt.ptr<f32>, #AL> -> !ttg.memdesc<16x32xf32, #A, #smem, mutable>
    ttg.async_commit_group
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies, backend inline assembly, NVIDIA-specific codegen. Embedded check comments (CHECK×6, CHECK-SAME×3) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝、后端内联汇编、NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×6, CHECK-SAME×3）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 717
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 719-745
```mlir
#block0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#block2 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#block3 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [4, 8], warpsPerCTA = [1, 4], order = [1, 0]}>
#slice2d1 = #ttg.slice<{dim = 1, parent=#block2}>
#slice3d0 = #ttg.slice<{dim = 0, parent=#block3}>
#AL = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#A = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: basic_insert_slice_async_v1_multictas
  tt.func @basic_insert_slice_async_v1_multictas(%arg0: !tt.ptr<f32> {tt.divisibility = 4 : i32}) {
    %off0_ = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #slice2d1>
    %off1_ = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #slice3d0>
    %off0 = tt.expand_dims %off0_ {axis = 1 : i32} : tensor<32xi32, #slice2d1> -> tensor<32x1xi32, #block2>
    %off1 = tt.expand_dims %off1_ {axis = 0 : i32} : tensor<32xi32, #slice3d0> -> tensor<1x32xi32, #block3>
    %broadcast_off0_scalar = tt.broadcast %off0 : tensor<32x1xi32, #block2> -> tensor<32x32xi32, #block2>
    %cst_scalar = arith.constant 32 : i32
    %cst = tt.splat %cst_scalar : i32 -> tensor<32x32xi32, #block2>
    %broadcast_off0_ = arith.muli %broadcast_off0_scalar, %cst : tensor<32x32xi32, #block2>
    %broadcast_off1_ = tt.broadcast %off1 : tensor<1x32xi32, #block3> -> tensor<32x32xi32, #block3>
    %broadcast_off0 = ttg.convert_layout %broadcast_off0_ : tensor<32x32xi32, #block2> -> tensor<32x32xi32, #AL>
    %broadcast_off1 = ttg.convert_layout %broadcast_off1_ : tensor<32x32xi32, #block3> -> tensor<32x32xi32, #AL>
    %off = arith.addi %broadcast_off0, %broadcast_off1 : tensor<32x32xi32, #AL>
    %a_init = tt.splat %arg0 : !tt.ptr<f32> -> tensor<32x32x!tt.ptr<f32>, #AL>
    %a_ptr = tt.addptr %a_init, %off : tensor<32x32x!tt.ptr<f32>, #AL>, tensor<32x32xi32, #AL>
    %tensor = ttg.local_alloc : () -> !ttg.memdesc<32x32xf32, #A, #smem, mutable>
    %index = arith.constant 1 : i32
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_insert_slice_async_v1_multictas`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_insert_slice_async_v1_multictas`。

### Lines 747-767
```mlir
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4;"
    // CHECK: llvm.inline_asm
    // CHECK-SAME: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4
    // CHECK: llvm.inline_asm
    // CHECK-SAME: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4
    // CHECK: llvm.inline_asm
    // CHECK-SAME: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4
    // CHECK: llvm.inline_asm
    // CHECK-SAME: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4
    // CHECK: llvm.inline_asm
    // CHECK-SAME: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4
    // CHECK: llvm.inline_asm
    // CHECK-SAME: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4
    // CHECK: llvm.inline_asm
    // CHECK-SAME: cp.async.ca.shared.global [ ${{.*}} + 0 ], [ ${{.*}} + 0 ], 0x4, 0x4
    // CHECK: nvvm.cp.async.commit.group
    %a = ttg.async_copy_global_to_local %a_ptr, %tensor : tensor<32x32x!tt.ptr<f32>, #AL> -> !ttg.memdesc<32x32xf32, #A, #smem, mutable>
    ttg.async_commit_group
    tt.return
  }
}
```
**EN:** This block defines the test function(s) and exercises async global-to-shared copies, backend inline assembly, NVIDIA-specific codegen. Embedded check comments (CHECK×9, CHECK-SAME×7) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 全局到共享内存的异步拷贝、后端内联汇编、NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×9, CHECK-SAME×7）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 769
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 771-781
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK: basic_splat
  tt.func @basic_splat(%ptr: !tt.ptr<f32>) {
    // CHECK: llvm.mlir.undef
    // CHECK: llvm.insertvalue
    // CHECK: llvm.insertvalue
    %0 = tt.splat %ptr : !tt.ptr<f32> -> tensor<256x!tt.ptr<f32>,#blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_splat`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_splat`。

### Line 783
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 785-796
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: basic_store
  tt.func @basic_store(%ptrs: tensor<256x!tt.ptr<f32>, #blocked0>, %vals: tensor<256xf32, #blocked0>, %mask: tensor<256xi1, #blocked0>) {
    // CHECK: llvm.inline_asm
    // CHECK-SAME: st.global.b32 [ ${{.*}} + 0 ], { ${{.*}} };
    // CHECK: llvm.inline_asm
    // CHECK-SAME: st.global.b32 [ ${{.*}} + 0 ], { ${{.*}} };
    tt.store %ptrs, %vals, %mask : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `basic_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `basic_store`。

### Line 798
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 800-811
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [32, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [16, 2], threadsPerWarp = [2, 16], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  //CHECK-LABEL: @convert_layout_blocked_blocked_shuffle_swap
  tt.func @convert_layout_blocked_blocked_shuffle_swap(%arg0: tensor<32x32xi32, #blocked0>) {
    //CHECK-COUNT-32: llvm.select
    //CHECK-COUNT-32: nvvm.shfl.sync
    //CHECK-COUNT-32: llvm.select
    %0 = ttg.convert_layout %arg0 : tensor<32x32xi32, #blocked0> -> tensor<32x32xi32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_blocked_blocked_shuffle_swap`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_blocked_blocked_shuffle_swap`。

### Line 813
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 815-824
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [32, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 1], order = [0, 1]}>
#blocked1 = #ttg.blocked<{sizePerThread = [16, 2], threadsPerWarp = [2, 16], warpsPerCTA = [1, 1], order = [0, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  //CHECK-LABEL: @convert_layout_blocked_blocked_shuffle_ship
  tt.func @convert_layout_blocked_blocked_shuffle_ship(%arg0: tensor<32x32xi32, #blocked0>) {
    //CHECK-COUNT-16: nvvm.shfl.sync
    %0 = ttg.convert_layout %arg0 : tensor<32x32xi32, #blocked0> -> tensor<32x32xi32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_blocked_blocked_shuffle_ship`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_blocked_blocked_shuffle_ship`。

### Line 826
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 828-839
```mlir
#linear0 = #ttg.linear<{register=[[1, 0], [2, 0], [4, 0]], lane=[[0, 1], [0, 2], [0, 4], [0, 8], [0, 16]], warp=[], block=[]}>
#linear1 = #ttg.linear<{register=[[1, 0], [2, 0], [0, 1]], lane=[[4, 0], [0, 2], [0, 4], [0, 8], [0, 16]], warp=[], block=[]}>
module attributes {"ttg.num-warps" = 1 : i32} {
  //CHECK-LABEL: @convert_layout_shuffle_packed_4xi1
  tt.func @convert_layout_shuffle_packed_4xi1(%arg0: tensor<8x32xi1, #linear0>) {
    //CHECK: llvm.select
    //CHECK: nvvm.shfl.sync
    //CHECK-COUNT-2: llvm.select
    %0 = ttg.convert_layout %arg0 : tensor<8x32xi1, #linear0> -> tensor<8x32xi1, #linear1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_shuffle_packed_4xi1`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_shuffle_packed_4xi1`。

### Line 841
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 843-856
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [2, 2], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [4, 1], threadsPerWarp = [4, 8], warpsPerCTA = [2, 2], order = [0, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK: llvm.mlir.global external @global_smem
  // CHECK-LABEL: convert_layout_blocked_blocked
  tt.func @convert_layout_blocked_blocked(%arg0: tensor<32x32xf32, #blocked0>) {
    // CHECK: llvm.mlir.addressof @global_smem
    // CHECK-COUNT-8: llvm.store
    // CHECK-: nvvm.barrier0
    // CHECK-COUNT-8: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<32x32xf32, #blocked0> -> tensor<32x32xf32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_blocked_blocked`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_blocked_blocked`。

### Line 858
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 860-875
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [2, 2], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [16, 2], warpsPerCTA = [2, 2], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK: llvm.mlir.global external @global_smem
  // CHECK-LABEL: convert_layout_blocked_blocked_vec
  tt.func @convert_layout_blocked_blocked_vec(%arg0: tensor<32x32xf32, #blocked0>) {
    // CHECK: llvm.mlir.addressof @global_smem
    // CHECK: llvm.store
    // CHECK: llvm.store
    // CHECK: nvvm.barrier0
    // CHECK: llvm.load
    // CHECK: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<32x32xf32, #blocked0> -> tensor<32x32xf32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_blocked_blocked_vec`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_blocked_blocked_vec`。

### Line 877
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 879-880
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
#blocked2 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [16, 2], warpsPerCTA = [1, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked0`, `#blocked2`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked0`, `#blocked2`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Line 882
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 884-890
```mlir
// CHECK-LABEL: convert_layout_ptr_element
tt.func @convert_layout_ptr_element(%arg0: tensor<16x16x!tt.ptr<i32>, #blocked0>) {
  // CHECK: llvm.ptrtoint
  // CHECK: llvm.inttoptr
  %0 = ttg.convert_layout %arg0 : tensor<16x16x!tt.ptr<i32>, #blocked0> -> tensor<16x16x!tt.ptr<i32>, #blocked2>
  tt.return
}
```
**EN:** This block defines `convert_layout_ptr_element` and exercises layout conversion. Embedded check comments (CHECK×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `convert_layout_ptr_element`，并覆盖 布局转换。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 892
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 894
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 896-913
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 32], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [4, 8], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK: llvm.mlir.global external @global_smem
  // CHECK-LABEL: convert_layout_blocked_blocked_multi_rep
  tt.func @convert_layout_blocked_blocked_multi_rep(%arg0: tensor<32x32xf32, #blocked0>) {
    // CHECK: llvm.mlir.addressof @global_smem
    // CHECK: llvm.store {{.*}} vector<4xi32>
    // CHECK: nvvm.bar.warp.sync
    // CHECK: nvvm.ldmatrix %{{.*}} : (!llvm.ptr<3>) -> !llvm.struct<(i32, i32, i32, i32)>
    // CHECK: nvvm.bar.warp.sync
    // CHECK: llvm.store {{.*}} vector<4xi32>
    // CHECK: nvvm.bar.warp.sync
    // CHECK: nvvm.ldmatrix %{{.*}} : (!llvm.ptr<3>) -> !llvm.struct<(i32, i32, i32, i32)>
    %0 = ttg.convert_layout %arg0 : tensor<32x32xf32, #blocked0> -> tensor<32x32xf32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_blocked_blocked_multi_rep`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_blocked_blocked_multi_rep`。

### Line 915
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 917-933
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase=1, maxPhase=1, order = [1, 0]}>
#mma0 = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
#dot_operand_a = #ttg.dot_op<{opIdx=0, parent=#mma0, kWidth=2}>
#dot_operand_b = #ttg.dot_op<{opIdx=1, parent=#mma0, kWidth=2}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: convert_dot_ldmatrix
  tt.func @convert_dot_ldmatrix(%A: tensor<16x16xf16, #blocked0>, %B: tensor<16x16xf16, #blocked0>) {
    %AA = ttg.local_alloc %A : (tensor<16x16xf16, #blocked0>) -> !ttg.memdesc<16x16xf16, #shared0, #smem>
    %BB = ttg.local_alloc %B : (tensor<16x16xf16, #blocked0>) -> !ttg.memdesc<16x16xf16, #shared0, #smem>
    // CHECK: nvvm.ldmatrix %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<row>, num = 4 : i32, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>} : (!llvm.ptr<3>) -> !llvm.struct<(i32, i32, i32, i32)>
    // CHECK: nvvm.ldmatrix %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<col>, num = 4 : i32, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>} : (!llvm.ptr<3>) -> !llvm.struct<(i32, i32, i32, i32)>
    // CHECK-NOT: nvvm.ldmatrix
    %AA_DOT = ttg.local_load %AA : !ttg.memdesc<16x16xf16, #shared0, #smem> -> tensor<16x16xf16, #dot_operand_a>
    %BB_DOT = ttg.local_load %BB : !ttg.memdesc<16x16xf16, #shared0, #smem> -> tensor<16x16xf16, #dot_operand_b>
    %cst0 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_dot_ldmatrix`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_dot_ldmatrix`。

### Lines 935-939
```mlir
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32
    %D = tt.dot %AA_DOT, %BB_DOT, %cst0 : tensor<16x16xf16, #dot_operand_a> * tensor<16x16xf16, #dot_operand_b> -> tensor<16x16xf32, #mma0>
```
**EN:** This block defines the test function(s) and exercises dot products / matmul, backend inline assembly. Embedded check comments (CHECK×2, CHECK-SAME×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 点积/矩阵乘、后端内联汇编。 其中嵌入的检查注释（CHECK×2, CHECK-SAME×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 941-943
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 945
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 947-963
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
#shared0 = #ttg.swizzled_shared<{vec = 8, perPhase=1, maxPhase=8, order = [1, 0]}>
#mma0 = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
#dot_operand_a = #ttg.dot_op<{opIdx=0, parent=#mma0, kWidth=2}>
#dot_operand_b = #ttg.dot_op<{opIdx=1, parent=#mma0, kWidth=2}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: convert_dot
  tt.func @convert_dot_ldmatrix_swizzle(%A: tensor<16x16xf16, #blocked0>, %B: tensor<16x16xf16, #blocked0>) {
    %AA = ttg.local_alloc %A : (tensor<16x16xf16, #blocked0>) -> !ttg.memdesc<16x16xf16, #shared0, #smem>
    %BB = ttg.local_alloc %B : (tensor<16x16xf16, #blocked0>) -> !ttg.memdesc<16x16xf16, #shared0, #smem>
    // CHECK: nvvm.ldmatrix %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<row>, num = 4 : i32, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>} : (!llvm.ptr<3>) -> !llvm.struct<(i32, i32, i32, i32)>
    // CHECK: nvvm.ldmatrix %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<col>, num = 4 : i32, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>} : (!llvm.ptr<3>) -> !llvm.struct<(i32, i32, i32, i32)>
    // CHECK-NOT: nvvm.ldmatrix
    %AA_DOT = ttg.local_load %AA : !ttg.memdesc<16x16xf16, #shared0, #smem> -> tensor<16x16xf16, #dot_operand_a>
    %BB_DOT = ttg.local_load %BB : !ttg.memdesc<16x16xf16, #shared0, #smem> -> tensor<16x16xf16, #dot_operand_b>
    %cst0 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_dot_ldmatrix_swizzle`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_dot_ldmatrix_swizzle`。

### Lines 965-969
```mlir
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32
    %D = tt.dot %AA_DOT, %BB_DOT, %cst0 : tensor<16x16xf16, #dot_operand_a> * tensor<16x16xf16, #dot_operand_b> -> tensor<16x16xf32, #mma0>
```
**EN:** This block defines the test function(s) and exercises dot products / matmul, backend inline assembly. Embedded check comments (CHECK×2, CHECK-SAME×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 点积/矩阵乘、后端内联汇编。 其中嵌入的检查注释（CHECK×2, CHECK-SAME×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 971-973
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 975
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 977-991
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase=1, maxPhase=8, order = [1, 0]}>
#mma0 = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
#dot_operand_a = #ttg.dot_op<{opIdx=0, parent=#mma0, kWidth=2}>
#dot_operand_b = #ttg.dot_op<{opIdx=1, parent=#mma0, kWidth=2}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: convert_dot
  tt.func @convert_dot(%A: tensor<16x16xf16, #blocked0>, %B: tensor<16x16xf16, #blocked0>) {
    %AA = ttg.local_alloc %A : (tensor<16x16xf16, #blocked0>) -> !ttg.memdesc<16x16xf16, #shared0, #smem>
    %BB = ttg.local_alloc %B : (tensor<16x16xf16, #blocked0>) -> !ttg.memdesc<16x16xf16, #shared0, #smem>
    // CHECK-NOT: nvvm.ldmatrix
    %AA_DOT = ttg.local_load %AA : !ttg.memdesc<16x16xf16, #shared0, #smem> -> tensor<16x16xf16, #dot_operand_a>
    %BB_DOT = ttg.local_load %BB : !ttg.memdesc<16x16xf16, #shared0, #smem> -> tensor<16x16xf16, #dot_operand_b>
    %cst0 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_dot`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_dot`。

### Lines 993-997
```mlir
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32
    %D = tt.dot %AA_DOT, %BB_DOT, %cst0 : tensor<16x16xf16, #dot_operand_a> * tensor<16x16xf16, #dot_operand_b> -> tensor<16x16xf32, #mma0>
```
**EN:** This block defines the test function(s) and exercises dot products / matmul, backend inline assembly. Embedded check comments (CHECK×2, CHECK-SAME×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 点积/矩阵乘、后端内联汇编。 其中嵌入的检查注释（CHECK×2, CHECK-SAME×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 999-1001
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1003
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1005-1019
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
#shared0 = #ttg.nvmma_shared<{swizzlingByteWidth = 128, transposed = false, elementBitWidth = 16}>
#mma0 = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
#dot_operand_a = #ttg.dot_op<{opIdx=0, parent=#mma0, kWidth=2}>
#dot_operand_b = #ttg.dot_op<{opIdx=1, parent=#mma0, kWidth=2}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: convert_dot_mmav3_shared
  tt.func @convert_dot_mmav3_shared(%A: tensor<64x64xf16, #blocked0>, %B: tensor<64x64xf16, #blocked0>) {
    %AA = ttg.local_alloc %A : (tensor<64x64xf16, #blocked0>) -> !ttg.memdesc<64x64xf16, #shared0, #smem>
    %BB = ttg.local_alloc %B : (tensor<64x64xf16, #blocked0>) -> !ttg.memdesc<64x64xf16, #shared0, #smem>
    // CHECK-COUNT-32: nvvm.ldmatrix %{{.*}} : (!llvm.ptr<3>) -> !llvm.struct<(i32, i32, i32, i32)>
    %AA_DOT = ttg.local_load %AA : !ttg.memdesc<64x64xf16, #shared0, #smem> -> tensor<64x64xf16, #dot_operand_a>
    %BB_DOT = ttg.local_load %BB : !ttg.memdesc<64x64xf16, #shared0, #smem> -> tensor<64x64xf16, #dot_operand_b>
    %cst0 = arith.constant dense<0.000000e+00> : tensor<64x64xf32, #mma0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_dot_mmav3_shared`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_dot_mmav3_shared`。

### Line 1021
```mlir
    %D = tt.dot %AA_DOT, %BB_DOT, %cst0 : tensor<64x64xf16, #dot_operand_a> * tensor<64x64xf16, #dot_operand_b> -> tensor<64x64xf32, #mma0>
```
**EN:** This block contributes intermediate IR built from `tt.dot`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 1023-1025
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1027
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1029-1044
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
#shared0 = #ttg.swizzled_shared<{vec = 16, perPhase=1, maxPhase=8, order = [1, 0]}>
#mma0 = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
#dot_operand_a = #ttg.dot_op<{opIdx=0, parent=#mma0, kWidth=4}>
#dot_operand_b = #ttg.dot_op<{opIdx=1, parent=#mma0, kWidth=4}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: convert_dot_fp8
  tt.func @convert_dot_fp8(%A: tensor<16x16xf8E5M2, #blocked0>, %B: tensor<16x16xf8E5M2, #blocked0>) {
    %AA = ttg.local_alloc %A : (tensor<16x16xf8E5M2, #blocked0>) -> !ttg.memdesc<16x16xf8E5M2, #shared0, #smem>
    %BB = ttg.local_alloc %B : (tensor<16x16xf8E5M2, #blocked0>) -> !ttg.memdesc<16x16xf8E5M2, #shared0, #smem>
    // CHECK: nvvm.ldmatrix %{{.*}} {eltType = #nvvm.ld_st_matrix_elt_type<b16>, layout = #nvvm.mma_layout<row>, num = 2 : i32, shape = #nvvm.ld_st_matrix_shape<m = 8, n = 8>} : (!llvm.ptr<3>) -> !llvm.struct<(i32, i32)>
    // CHECK-NOT: nvvm.ldmatrix
    %AA_DOT = ttg.local_load %AA : !ttg.memdesc<16x16xf8E5M2, #shared0, #smem> -> tensor<16x16xf8E5M2, #dot_operand_a>
    %BB_DOT = ttg.local_load %BB : !ttg.memdesc<16x16xf8E5M2, #shared0, #smem> -> tensor<16x16xf8E5M2, #dot_operand_b>
    %cst0 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_dot_fp8`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_dot_fp8`。

### Lines 1046-1050
```mlir
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k32.row.col.f32.e5m2.e5m2.f32
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k32.row.col.f32.e5m2.e5m2.f32
    %D = tt.dot %AA_DOT, %BB_DOT, %cst0 : tensor<16x16xf8E5M2, #dot_operand_a> * tensor<16x16xf8E5M2, #dot_operand_b> -> tensor<16x16xf32, #mma0>
```
**EN:** This block defines the test function(s) and exercises dot products / matmul, backend inline assembly. Embedded check comments (CHECK×2, CHECK-SAME×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 点积/矩阵乘、后端内联汇编。 其中嵌入的检查注释（CHECK×2, CHECK-SAME×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 1052-1054
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1056
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1058-1070
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [4, 8], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 4], order = [0, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK: llvm.mlir.global external @global_smem
  // CHECK-LABEL: convert_layout_transpose
  tt.func @convert_layout_transpose(%arg0: tensor<128x128xf8E5M2, #blocked>) {
    // CHECK-COUNT-128: llvm.store {{.*}} vector<1xi8>
    // CHECK: nvvm.barrier0
    // CHECK-COUNT-32: llvm.load {{.*}} vector<4xi8>
    %0 = ttg.convert_layout %arg0 : tensor<128x128xf8E5M2, #blocked> -> tensor<128x128xf8E5M2, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_transpose`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_transpose`。

### Line 1072
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1074-1087
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [1, 0]}>
#mma = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK: llvm.mlir.global external @global_smem
  // CHECK-LABEL: convert_layout_mmav2_block
  tt.func @convert_layout_mmav2_blocked(%arg0: tensor<32x16xf32, #mma>) {
    // CHECK: llvm.store
    // CHECK: llvm.store
    // CHECK: nvvm.barrier0
    // CHECK: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<32x16xf32, #mma> -> tensor<32x16xf32, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav2_blocked`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav2_blocked`。

### Line 1089
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1091-1101
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
#dot1 = #ttg.dot_op<{opIdx=0, parent=#mma, kWidth=2}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: convert_layout_mmav2_dot_reg
  tt.func @convert_layout_mmav2_dot_reg(%arg0: tensor<16x16xf16, #mma>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<16x16xf16, #mma> -> tensor<16x16xf16, #dot1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav2_dot_reg`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav2_dot_reg`。

### Line 1103
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1105-1115
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
#dot1 = #ttg.dot_op<{opIdx=0, parent=#mma, kWidth=2}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: convert_layout_mmav2_dot_reg
  tt.func @convert_layout_mmav2_dot_reg(%arg0: tensor<1x16xf16, #mma>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<1x16xf16, #mma> -> tensor<1x16xf16, #dot1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav2_dot_reg`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav2_dot_reg`。

### Line 1117
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1119-1121
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#slice = #ttg.slice<{dim = 0, parent = #mma}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma`, `#blocked`, `#slice`. They parameterize later tests with compact names for `#ttg.nvidia_mma`, `#ttg.blocked`, `#ttg.slice`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma`, `#blocked`, `#slice`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`, `#ttg.blocked`, `#ttg.slice`。

### Lines 1123-1131
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: convert_layout_slice_mmav2_blocked_reg
  tt.func @convert_layout_slice_mmav2_blocked_reg(%arg0: tensor<1xf16, #slice>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<1xf16, #slice> -> tensor<1xf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_slice_mmav2_blocked_reg`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_slice_mmav2_blocked_reg`。

### Line 1133
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1135-1136
```mlir
#mma0 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 16]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma0`, `#mma1`. They parameterize later tests with compact names for `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma0`, `#mma1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`。

### Lines 1138-1146
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: convert_layout_mmav3_mmav3_0
  tt.func @convert_layout_mmav3_mmav3_0(%arg0: tensor<64x64xf16, #mma0>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<64x64xf16, #mma0> -> tensor<64x64xf16, #mma1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav3_mmav3_0`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav3_mmav3_0`。

### Line 1148
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1150-1151
```mlir
#mma0 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 16]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma0`, `#mma1`. They parameterize later tests with compact names for `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma0`, `#mma1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`。

### Lines 1153-1161
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: convert_layout_mmav3_mmav3_1
  tt.func @convert_layout_mmav3_mmav3_1(%arg0: tensor<64x64xf16, #mma1>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<64x64xf16, #mma1> -> tensor<64x64xf16, #mma0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav3_mmav3_1`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav3_mmav3_1`。

### Line 1163
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1165-1166
```mlir
#mma0 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 16]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma0`, `#mma1`. They parameterize later tests with compact names for `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma0`, `#mma1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`。

### Lines 1168-1176
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: convert_layout_mmav3_mmav3_2
  tt.func @convert_layout_mmav3_mmav3_2(%arg0: tensor<16x16xf16, #mma1>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<16x16xf16, #mma1> -> tensor<16x16xf16, #mma0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav3_mmav3_2`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav3_mmav3_2`。

### Line 1178
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1180-1181
```mlir
#mma0 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 16]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma0`, `#mma1`. They parameterize later tests with compact names for `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma0`, `#mma1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`。

### Lines 1183-1191
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: convert_layout_mmav3_mmav3_3
  tt.func @convert_layout_mmav3_mmav3_3(%arg0: tensor<1x64xf16, #mma1>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<1x64xf16, #mma1> -> tensor<1x64xf16, #mma0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav3_mmav3_3`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav3_mmav3_3`。

### Line 1193
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1195-1205
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
#dot1 = #ttg.dot_op<{opIdx=0, parent=#mma, kWidth=2}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: convert_layout_mmav2_dot_reg
  tt.func @convert_layout_mmav2_dot_reg(%arg0: tensor<16x16xf16, #mma>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<16x16xf16, #mma> -> tensor<16x16xf16, #dot1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav2_dot_reg`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav2_dot_reg`。

### Line 1207
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1209-1210
```mlir
#mma0 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 16]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma0`, `#mma1`. They parameterize later tests with compact names for `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma0`, `#mma1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`。

### Lines 1212-1220
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: convert_layout_mmav3_mmav3_0
  tt.func @convert_layout_mmav3_mmav3_0(%arg0: tensor<64x64xf16, #mma0>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<64x64xf16, #mma0> -> tensor<64x64xf16, #mma1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav3_mmav3_0`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav3_mmav3_0`。

### Line 1222
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1224-1225
```mlir
#mma0 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 16]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma0`, `#mma1`. They parameterize later tests with compact names for `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma0`, `#mma1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`。

### Lines 1227-1235
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: convert_layout_mmav3_mmav3_1
  tt.func @convert_layout_mmav3_mmav3_1(%arg0: tensor<64x64xf16, #mma1>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<64x64xf16, #mma1> -> tensor<64x64xf16, #mma0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav3_mmav3_1`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav3_mmav3_1`。

### Line 1237
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1239-1240
```mlir
#mma0 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 16]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma0`, `#mma1`. They parameterize later tests with compact names for `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma0`, `#mma1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`。

### Lines 1242-1250
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: convert_layout_mmav3_mmav3_2
  tt.func @convert_layout_mmav3_mmav3_2(%arg0: tensor<16x16xf16, #mma1>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<16x16xf16, #mma1> -> tensor<16x16xf16, #mma0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav3_mmav3_2`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav3_mmav3_2`。

### Line 1252
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1254-1255
```mlir
#mma0 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 64, 16]}>
#mma1 = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [4, 1], instrShape = [16, 128, 16]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma0`, `#mma1`. They parameterize later tests with compact names for `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma0`, `#mma1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`。

### Lines 1257-1265
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: convert_layout_mmav3_mmav3_3
  tt.func @convert_layout_mmav3_mmav3_3(%arg0: tensor<1x64xf16, #mma1>) {
    // CHECK-NOT: llvm.store
    // CHECK-NOT: llvm.load
    %0 = ttg.convert_layout %arg0 : tensor<1x64xf16, #mma1> -> tensor<1x64xf16, #mma0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav3_mmav3_3`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav3_mmav3_3`。

### Line 1267
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1269-1280
```mlir
#blocked = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [8, 4], warpsPerCTA = [1, 8], order = [0, 1]}>
#mma = #ttg.nvidia_mma<{versionMajor = 3, versionMinor = 0, warpsPerCTA = [8, 1], instrShape = [16, 256, 32]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK: llvm.mlir.global external @global_smem
  // CHECK-LABEL: convert_layout_mmav3_transpose
  tt.func @convert_layout_mmav3_transpose(%arg0: tensor<128x256xf8E5M2, #mma>) {
    // CHECK-COUNT-8: llvm.store {{.*}} : vector<4xi32>
    // CHECK: nvvm.barrier0
    %0 = ttg.convert_layout %arg0 : tensor<128x256xf8E5M2, #mma> -> tensor<128x256xf8E5M2, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_mmav3_transpose`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_mmav3_transpose`。

### Line 1282
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1283-1297
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
#shared0 = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK: llvm.mlir.global external @global_smem
  // CHECK-LABEL: convert_layout_blocked_shared
  tt.func @convert_layout_blocked_shared(%arg0: tensor<128x32xf32, #blocked0>) {
    // CHECK: llvm.store
    // CHECK-SAME: !llvm.ptr<3>
    // CHECK: llvm.store
    // CHECK-SAME: !llvm.ptr<3>
    %0 = ttg.local_alloc %arg0 : (tensor<128x32xf32, #blocked0>) -> !ttg.memdesc<128x32xf32, #shared0, #smem>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_layout_blocked_shared`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_layout_blocked_shared`。

### Line 1299
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1301-1312
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: convert_blocked1d_to_slice0
  tt.func @convert_blocked1d_to_slice0(%src:tensor<32xi32, #blocked0>) {
    // CHECK: llvm.store {{.*}} : vector<1xi32>
    // CHECK: nvvm.bar.warp.sync
    // CHECK-COUNT-1: llvm.load {{.*}} -> vector<4xi32>
    %cvt = ttg.convert_layout %src : tensor<32xi32, #blocked0> -> tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked1}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_blocked1d_to_slice0`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_blocked1d_to_slice0`。

### Line 1314
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1316-1325
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [4, 8], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: convert_blocked1d_to_slice1
  tt.func @convert_blocked1d_to_slice1(%src:tensor<32xi32, #blocked0>) {
    // CHECK-COUNT-2: llvm.load {{.*}} -> vector<4xi32>
    %cvt = ttg.convert_layout %src : tensor<32xi32, #blocked0> -> tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked1}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_blocked1d_to_slice1`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_blocked1d_to_slice1`。

### Line 1327
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1329-1342
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: convert_blocked_to_blocked_ptr
  tt.func @convert_blocked_to_blocked_ptr(%src:tensor<32x!tt.ptr<f32>, #blocked0>) {
    // CHECK: llvm.ptrtoint
    // CHECK: llvm.store
    // CHECK: nvvm.bar.warp.sync
    // CHECK: llvm.inttoptr
    // CHECK-COUNT-4: llvm.insertvalue
    %cvt = ttg.convert_layout %src : tensor<32x!tt.ptr<f32>, #blocked0> -> tensor<32x!tt.ptr<f32>, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_blocked_to_blocked_ptr`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_blocked_to_blocked_ptr`。

### Line 1344
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1346-1361
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0], CGALayout = [[0]]}>
#blocked1 = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [1], order = [0], CGALayout = [[0]]}>
module attributes {"ttg.num-ctas" = 2 : i32, "ttg.num-warps" = 1 : i32} {
  // Broadcast-only CGA bases should not force a cluster barrier.
  // CHECK-LABEL: convert_blocked_to_blocked_broadcast_cga
  tt.func @convert_blocked_to_blocked_broadcast_cga(%src: tensor<32xf32, #blocked0>) {
    // CHECK: llvm.store
    // CHECK: nvvm.bar.warp.sync
    // CHECK: llvm.load
    // CHECK-NOT: nvvm.cluster.arrive
    // CHECK-NOT: nvvm.cluster.wait
    // CHECK: llvm.return
    %cvt = ttg.convert_layout %src : tensor<32xf32, #blocked0> -> tensor<32xf32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_blocked_to_blocked_broadcast_cga`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_blocked_to_blocked_broadcast_cga`。

### Line 1363
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1365-1377
```mlir
// Regression test for https://github.com/triton-lang/triton/issues/5745
#linear = #ttg.linear<{register = [[0, 1], [0, 2]], lane = [[0, 0], [0, 0], [0, 0], [0, 0], [0, 0]], warp = [[1, 0], [2, 0], [4, 0]], block = []}>
#linear1 = #ttg.linear<{register = [[0, 2]], lane = [[0, 0], [0, 0], [0, 0], [0, 0], [1, 0]], warp = [[2, 0], [4, 0], [0, 1]], block = []}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: linear_layout_with_multiple_iterations
  tt.func @linear_layout_with_multiple_iterations(%src: tensor<8x4xbf16, #linear>) {
    %cvt = ttg.convert_layout %src : tensor<8x4xbf16, #linear> -> tensor<8x4xbf16, #linear1>
    // CHECK-COUNT-1: llvm.store {{.*}} : vector<4xi16>
    // CHECK: nvvm.barrier0
    // CHECK-COUNT: llvm.load{{.*}}->vector<2xi16>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `linear_layout_with_multiple_iterations`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `linear_layout_with_multiple_iterations`。

### Line 1379
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1381-1393
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 8, order = [1, 0]}>
#mma = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
#dot_operand_a = #ttg.dot_op<{opIdx=0, parent=#mma, kWidth=2}>
#dot_operand_b = #ttg.dot_op<{opIdx=1, parent=#mma, kWidth=2}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  tt.func @matmul_kernel_dot_operand_layout(%ptr:!tt.ptr<f32> {tt.divisibility = 16 : i32},
  %a:!ttg.memdesc<128x32xf16, #shared, #smem>, %b:!ttg.memdesc<32x256xf16, #shared, #smem>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<128x256xf32, #mma>
    // CHECK: nvvm.ldmatrix
    %a_mat = ttg.local_load %a : !ttg.memdesc<128x32xf16, #shared, #smem> -> tensor<128x32xf16, #dot_operand_a>
    %b_mat = ttg.local_load %b : !ttg.memdesc<32x256xf16, #shared, #smem> -> tensor<32x256xf16, #dot_operand_b>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `matmul_kernel_dot_operand_layout`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `matmul_kernel_dot_operand_layout`。

### Lines 1395-1396
```mlir
    %28 = tt.dot %a_mat, %b_mat, %cst : tensor<128x32xf16, #dot_operand_a> * tensor<32x256xf16, #dot_operand_b> -> tensor<128x256xf32, #mma>
    %38 = ttg.convert_layout %28 : tensor<128x256xf32, #mma> -> tensor<128x256xf32, #blocked>
```
**EN:** This block contributes intermediate IR built from `tt.dot`, `ttg.convert_layout`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot`, `ttg.convert_layout` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 1398-1403
```mlir
    %30 = tt.splat %ptr : !tt.ptr<f32> -> tensor<128x1x!tt.ptr<f32>, #blocked>
    %36 = tt.broadcast %30 : tensor<128x1x!tt.ptr<f32>, #blocked> -> tensor<128x256x!tt.ptr<f32>, #blocked>
    tt.store %36, %38 : tensor<128x256x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.broadcast`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.broadcast`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1405
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1407-1418
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#dot_operand_a = #ttg.dot_op<{opIdx=0, parent=#blocked}>
#dot_operand_b = #ttg.dot_op<{opIdx=1, parent=#blocked}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  tt.func @matmul_fmadot(%ptr:!tt.ptr<f32> {tt.divisibility = 16 : i32},
  %a:!ttg.memdesc<32x16xf32, #shared, #smem>, %b:!ttg.memdesc<16x32xf32, #shared, #smem>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #blocked>
    // CHECK: llvm.intr.fmuladd
    %a_mat = ttg.local_load %a : !ttg.memdesc<32x16xf32, #shared, #smem> -> tensor<32x16xf32, #dot_operand_a>
    %b_mat = ttg.local_load %b : !ttg.memdesc<16x32xf32, #shared, #smem> -> tensor<16x32xf32, #dot_operand_b>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `matmul_fmadot`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `matmul_fmadot`。

### Lines 1420-1426
```mlir
    %28 = tt.dot %a_mat, %b_mat, %cst, inputPrecision = ieee : tensor<32x16xf32, #dot_operand_a> * tensor<16x32xf32, #dot_operand_b> -> tensor<32x32xf32, #blocked>
    %30 = tt.splat %ptr : !tt.ptr<f32> -> tensor<32x1x!tt.ptr<f32>, #blocked>
    %36 = tt.broadcast %30 : tensor<32x1x!tt.ptr<f32>, #blocked> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.store %36, %28 : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.dot`, `tt.splat`, `tt.ptr`, `tt.broadcast`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot`, `tt.splat`, `tt.ptr`, `tt.broadcast`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1428
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1430-1444
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#dot_operand_a = #ttg.dot_op<{opIdx=0, parent=#blocked}>
#dot_operand_b = #ttg.dot_op<{opIdx=1, parent=#blocked}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "cuda:70", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: matmul_fmadot_integer
  tt.func @matmul_fmadot_integer(%ptr:!tt.ptr<i32> {tt.divisibility = 16 : i32},
  %a:!ttg.memdesc<32x16xi32, #shared, #smem>, %b:!ttg.memdesc<16x32xi32, #shared, #smem>) {
    %cst = arith.constant dense<0> : tensor<32x32xi32, #blocked>
    // CHECK-NOT: llvm.intr.fmuladd
    // CHECK: llvm.mul
    // CHECK: llvm.add
    %a_mat = ttg.local_load %a : !ttg.memdesc<32x16xi32, #shared, #smem> -> tensor<32x16xi32, #dot_operand_a>
    %b_mat = ttg.local_load %b : !ttg.memdesc<16x32xi32, #shared, #smem> -> tensor<16x32xi32, #dot_operand_b>
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `matmul_fmadot_integer`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `matmul_fmadot_integer`。

### Lines 1446-1452
```mlir
    %28 = tt.dot %a_mat, %b_mat, %cst, inputPrecision = ieee : tensor<32x16xi32, #dot_operand_a> * tensor<16x32xi32, #dot_operand_b> -> tensor<32x32xi32, #blocked>
    %30 = tt.splat %ptr : !tt.ptr<i32> -> tensor<32x1x!tt.ptr<i32>, #blocked>
    %36 = tt.broadcast %30 : tensor<32x1x!tt.ptr<i32>, #blocked> -> tensor<32x32x!tt.ptr<i32>, #blocked>
    tt.store %36, %28 : tensor<32x32x!tt.ptr<i32>, #blocked>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.dot`, `tt.splat`, `tt.ptr`, `tt.broadcast`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot`, `tt.splat`, `tt.ptr`, `tt.broadcast`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1454
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1456-1472
```mlir
#mma = #ttg.nvidia_mma<{versionMajor=2, warpsPerCTA=[2, 2], instrShape = [16, 8]}>
#shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 4, order = [1, 0]}>
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
#dot_operand_a = #ttg.dot_op<{opIdx=0, parent=#mma, kWidth=1}>
#dot_operand_b = #ttg.dot_op<{opIdx=1, parent=#mma, kWidth=1}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: matmul_tf32dot
  tt.func @matmul_tf32dot(%ptr:!tt.ptr<f32> {tt.divisibility = 16 : i32},
  %a:!ttg.memdesc<32x16xf32, #shared, #smem>, %b:!ttg.memdesc<16x32xf32, #shared, #smem>) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
    // CHECK: nvvm.ldmatrix
    // CHECK-SAME: (i32, i32, i32, i32)
    // CHECK: nvvm.ldmatrix
    // CHECK-SAME: (i32, i32, i32, i32)
    %a_mat = ttg.local_load %a : !ttg.memdesc<32x16xf32, #shared, #smem> -> tensor<32x16xf32, #dot_operand_a>
    %b_mat = ttg.local_load %b : !ttg.memdesc<16x32xf32, #shared, #smem> -> tensor<16x32xf32, #dot_operand_b>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `matmul_tf32dot`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `matmul_tf32dot`。

### Lines 1474-1483
```mlir
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k8.row.col.f32.tf32.tf32.f32
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k8.row.col.f32.tf32.tf32.f32
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k8.row.col.f32.tf32.tf32.f32
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k8.row.col.f32.tf32.tf32.f32
    %28 = tt.dot %a_mat, %b_mat, %cst, inputPrecision = tf32 : tensor<32x16xf32, #dot_operand_a> * tensor<16x32xf32, #dot_operand_b> -> tensor<32x32xf32, #mma>
    %38 = ttg.convert_layout %28 : tensor<32x32xf32, #mma> -> tensor<32x32xf32, #blocked>
```
**EN:** This block defines the test function(s) and exercises layout conversion, dot products / matmul, backend inline assembly. Embedded check comments (CHECK×4, CHECK-SAME×4) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 the test function(s)，并覆盖 布局转换、点积/矩阵乘、后端内联汇编。 其中嵌入的检查注释（CHECK×4, CHECK-SAME×4）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 1485-1490
```mlir
    %30 = tt.splat %ptr : !tt.ptr<f32> -> tensor<32x1x!tt.ptr<f32>, #blocked>
    %36 = tt.broadcast %30 : tensor<32x1x!tt.ptr<f32>, #blocked> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.store %36, %38 : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.splat`, `tt.ptr`, `tt.broadcast`, `tt.store`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.splat`, `tt.ptr`, `tt.broadcast`, `tt.store`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1492
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1494-1505
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
  // CHECK-LABEL: atomic_add_f32
  tt.func @atomic_add_f32(%arg0 : tensor<256x!tt.ptr<f32>, #blocked0>, %arg1 : tensor<256xi1, #blocked0>, %arg2 : tensor<256xf32, #blocked0>) {
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "mov.u32 $0, 0x0;
    // CHECK-SAME: @$3 atom.global.gpu.relaxed.add.f32
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "mov.u32 $0, 0x0;
    // CHECK-SAME: @$3 atom.global.gpu.relaxed.add.f32
    %0 = tt.atomic_rmw fadd, relaxed, gpu, %arg0, %arg2, %arg1 : (tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xf32, #blocked0>, tensor<256xi1, #blocked0>) -> tensor<256xf32, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f32`。

### Line 1507
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1509-1518
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
  // CHECK-LABEL: atomic_add_f32_scalar
  tt.func @atomic_add_f32_scalar(%arg0 : !tt.ptr<f32>, %arg1 : i1, %arg2 : f32) {
    // CHECK: llvm.icmp "eq"
    // CHECK: llvm.inline_asm
    // CHECK-SAME: @$3 atom.global.gpu.relaxed.add.f32
    %0 = tt.atomic_rmw fadd, relaxed, gpu, %arg0, %arg2, %arg1 : (!tt.ptr<f32>, f32, i1) -> f32
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f32_scalar`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f32_scalar`。

### Line 1520
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1522-1533
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
  // CHECK-LABEL: atomic_add_f32
  tt.func @atomic_add_f32_sys_scope(%arg0 : tensor<256x!tt.ptr<f32>, #blocked0>, %arg1 : tensor<256xi1, #blocked0>, %arg2 : tensor<256xf32, #blocked0>) {
    // CHECK: llvm.inline_asm
    // CHECK-SAME: @$3 atom.global.sys.relaxed.add.f32
    // CHECK: llvm.inline_asm
    // CHECK-SAME: @$3 atom.global.sys.relaxed.add.f32
    %0 = tt.atomic_rmw fadd, relaxed, sys, %arg0, %arg2, %arg1 : (tensor<256x!tt.ptr<f32>, #blocked0>, tensor<256xf32, #blocked0>, tensor<256xi1, #blocked0>) -> tensor<256xf32, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f32_sys_scope`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f32_sys_scope`。

### Line 1535
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1537-1548
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
  // CHECK-LABEL: atomic_add_use_result_broadcasting
  tt.func @atomic_add_use_result_broadcasting(%arg0 : tensor<16x!tt.ptr<f32>, #blocked0>, %arg1 : tensor<16xi1, #blocked0>, %arg2 : tensor<16xf32, #blocked0>) {
    %0 = tt.atomic_rmw fadd, relaxed, sys, %arg0, %arg2, %arg1 : (tensor<16x!tt.ptr<f32>, #blocked0>, tensor<16xf32, #blocked0>, tensor<16xi1, #blocked0>) -> tensor<16xf32, #blocked0>
    // CHECK: st.shared
    // CHECK: nvvm.barrier0
    // CHECK: llvm.load
    tt.store %arg0, %0 : tensor<16x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_use_result_broadcasting`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_use_result_broadcasting`。

### Line 1550
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1552-1563
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.target" = "cuda:80"} {
  // CHECK-LABEL: atomic_add_use_result_no_broadcasting
  tt.func @atomic_add_use_result_no_broadcasting(%arg0 : tensor<128x!tt.ptr<f32>, #blocked0>, %arg1 : tensor<128xi1, #blocked0>, %arg2 : tensor<128xf32, #blocked0>) {
    %0 = tt.atomic_rmw fadd, relaxed, sys, %arg0, %arg2, %arg1 : (tensor<128x!tt.ptr<f32>, #blocked0>, tensor<128xf32, #blocked0>, tensor<128xi1, #blocked0>) -> tensor<128xf32, #blocked0>
    // CHECK-NOT: st.shared
    // CHECK-NOT: nvvm.barrier0
    // CHECK-NOT: llvm.load
    tt.store %arg0, %0 : tensor<128x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_use_result_no_broadcasting`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_use_result_no_broadcasting`。

### Line 1565
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1567-1576
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @atomic_add_f16_nomask(%dest_ptrs: tensor<256x!tt.ptr<f16>, #blocked> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %data: tensor<256xf16, #blocked>) {
    // CHECK-LABEL: atomic_add_f16_nomask
    // CHECK: atom.global.gpu.acq_rel.add.noftz.f16x2
    // CHECK: atom.global.gpu.acq_rel.add.noftz.f16x2
    %0 = tt.atomic_rmw fadd, acq_rel, gpu, %dest_ptrs, %data : (tensor<256x!tt.ptr<f16>, #blocked>, tensor<256xf16, #blocked>) -> tensor<256xf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:80`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f16_nomask`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:80`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f16_nomask`。

### Line 1578
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1580-1591
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @atomic_add_f16_withmask(%dest_ptrs: tensor<256x!tt.ptr<f16>, #blocked> {tt.divisibility = 16 : i32, tt.contiguity = 16 : i32}, %data: tensor<256xf16, #blocked>, %mask: tensor<256xi1, #blocked>) {
    // CHECK-LABEL: atomic_add_f16_withmask
    // CHECK: atom.global.gpu.acq_rel.add.noftz.f16
    // CHECK: atom.global.gpu.acq_rel.add.noftz.f16
    // CHECK: atom.global.gpu.acq_rel.add.noftz.f16
    // CHECK: atom.global.gpu.acq_rel.add.noftz.f16
    %0 = tt.atomic_rmw fadd, acq_rel, gpu, %dest_ptrs, %data, %mask : (tensor<256x!tt.ptr<f16>, #blocked>, tensor<256xf16, #blocked>, tensor<256xi1, #blocked>) -> tensor<256xf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:80`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `atomic_add_f16_withmask`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:80`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `atomic_add_f16_withmask`。

### Line 1593
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1595-1606
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: store_f32
  tt.func @store_f32(%arg0 : tensor<256x!tt.ptr<f32>, #blocked0>, %arg1 : tensor<256xf32, #blocked0>) {
    // CHECK: llvm.inline_asm
    // CHECK-SAME: st.global.b32
    // CHECK: llvm.inline_asm
    // CHECK-SAME: st.global.b32
    tt.store %arg0, %arg1 : tensor<256x!tt.ptr<f32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `store_f32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `store_f32`。

### Line 1608
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1610-1619
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: store_f32_scalar
  tt.func @store_f32_scalar(%arg0 : !tt.ptr<f32>, %arg1 : f32) {
    // CHECK: llvm.icmp "eq"
    // CHECK: llvm.inline_asm
    // CHECK-SAME: @$2 st.global.b32
    tt.store %arg0, %arg1 : !tt.ptr<f32>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `store_f32_scalar`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `store_f32_scalar`。

### Line 1621
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1623-1636
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
// CHECK-LABEL: test_get_program_id
tt.func @test_get_program_id(%a: tensor<32x!tt.ptr<i32>, #blocked0>) {
  %blockidx = tt.get_program_id x: i32
  %blockidy = tt.get_program_id y: i32
  %blockidz = tt.get_program_id z: i32
  // CHECK: ctaid.x
  // CHECK: ctaid.y
  // CHECK: ctaid.z
  %v0 = arith.addi %blockidx, %blockidy : i32
  %v1 = arith.addi %v0, %blockidz : i32
  %0 = tt.splat %v1 : i32 -> tensor<32xi32, #blocked0>
  tt.store %a, %0 : tensor<32x!tt.ptr<i32>, #blocked0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_get_program_id`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_get_program_id`。

### Lines 1638-1639
```mlir
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1641
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 1643
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1645-1658
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0], CGALayout = [[0], [0]]}>
module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32} {
// CHECK-LABEL: test_get_program_id
tt.func @test_get_program_id(%a: tensor<32x!tt.ptr<i32>, #blocked0>) {
  %blockidx = tt.get_program_id x: i32
  %blockidy = tt.get_program_id y: i32
  %blockidz = tt.get_program_id z : i32
  // CHECK: clusterid.x
  // CHECK: clusterid.y
  // CHECK: clusterid.z
  %v0 = arith.addi %blockidx, %blockidy : i32
  %v1 = arith.addi %v0, %blockidz : i32
  %0 = tt.splat %v1 : i32 -> tensor<32xi32, #blocked0>
  tt.store %a, %0 : tensor<32x!tt.ptr<i32>, #blocked0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_get_program_id`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_get_program_id`。

### Lines 1660-1661
```mlir
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1663
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 1665
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1667-1680
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: test_get_num_program
  tt.func @test_get_num_program(%a: tensor<32x!tt.ptr<i32>, #blocked0>) {
    %blockdimx = tt.get_num_programs x : i32
    %blockdimy = tt.get_num_programs y : i32
    %blockdimz = tt.get_num_programs z : i32
    // CHECK: nctaid.x
    // CHECK: nctaid.y
    // CHECK: nctaid.z
    %v0 = arith.addi %blockdimx, %blockdimy : i32
    %v1 = arith.addi %v0, %blockdimz : i32
    %0 = tt.splat %v1 : i32 -> tensor<32xi32, #blocked0>
    tt.store %a, %0 : tensor<32x!tt.ptr<i32>, #blocked0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_get_num_program`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_get_num_program`。

### Lines 1682-1684
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1686
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1688-1700
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0], CGALayout = [[0], [0]]}>
module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32} {
  tt.func @test_get_num_program(%a: tensor<32x!tt.ptr<i32>, #blocked0>) {
    %blockdimx = tt.get_num_programs x : i32
    %blockdimy = tt.get_num_programs y : i32
    %blockdimz = tt.get_num_programs z : i32
    // CHECK: nclusterid.x
    // CHECK: nclusterid.y
    // CHECK: nclusterid.z
    %v0 = arith.addi %blockdimx, %blockdimy : i32
    %v1 = arith.addi %v0, %blockdimz : i32
    %0 = tt.splat %v1 : i32 -> tensor<32xi32, #blocked0>
    tt.store %a, %0 : tensor<32x!tt.ptr<i32>, #blocked0>
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_get_num_program`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_get_num_program`。

### Lines 1702-1704
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 1706
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1707-1716
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: test_index_cache
  tt.func @test_index_cache() {
    // CHECK: nvvm.read.ptx.sreg.tid.x
    %0 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
    %1 = tt.make_range {end = 256 : i32, start = 0 : i32} : tensor<256xi32, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_index_cache`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_index_cache`。

### Line 1718
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1719-1730
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
#shared0 = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: test_base_index_cache
  tt.func @test_base_index_cache(%arg0: tensor<128x32xf32, #blocked0>) {
    // CHECK: nvvm.read.ptx.sreg.tid.x
    %0 = ttg.local_alloc %arg0 : (tensor<128x32xf32, #blocked0>) -> !ttg.memdesc<128x32xf32, #shared0, #smem>
    %1 = ttg.local_alloc %arg0 : (tensor<128x32xf32, #blocked0>) -> !ttg.memdesc<128x32xf32, #shared0, #smem>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_base_index_cache`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_base_index_cache`。

### Line 1732
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1733-1748
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
#shared0 = #ttg.swizzled_shared<{vec = 8, perPhase = 2, maxPhase = 4, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: test_index_cache_different_block
  tt.func @test_index_cache_different_block(%arg0: tensor<128x32xf32, #blocked0>, %arg1: i1) {
    // CHECK: nvvm.read.ptx.sreg.tid.x
    %0 = ttg.local_alloc %arg0 : (tensor<128x32xf32, #blocked0>) -> !ttg.memdesc<128x32xf32, #shared0, #smem>
    cf.cond_br %arg1, ^bb1, ^bb2
    ^bb1:  // pred: ^bb0
      %1 = ttg.local_alloc %arg0 : (tensor<128x32xf32, #blocked0>) -> !ttg.memdesc<128x32xf32, #shared0, #smem>
      cf.br ^bb2
    ^bb2:  // 2 preds: ^bb0, ^bb1
      tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_index_cache_different_block`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_index_cache_different_block`。

### Line 1750
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1752-1773
```mlir
#mma = #ttg.nvidia_mma<{versionMajor=2, warpsPerCTA=[2, 2], instrShape = [16, 8]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [2, 16], warpsPerCTA = [1, 4], order = [1, 0]}>
#dot_operand_a = #ttg.dot_op<{opIdx=0, parent=#mma, kWidth=1}>
#dot_operand_b = #ttg.dot_op<{opIdx=1, parent=#mma, kWidth=1}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: matmul_tf32_cst_b
  tt.func @matmul_tf32_cst_b(%ptr:!tt.ptr<f32> {tt.divisibility = 16 : i32},
  %a: tensor<32x16xf32, #dot_operand_a>, %c: tensor<32x32xf32, #mma>) {
  // CHECK: %[[CST:.+]] = llvm.mlir.constant(1.000000e+00 : f32) : f32
  // CHECK: %[[BC:.+]] = llvm.bitcast %[[CST]] : f32 to f32
  // CHECK: %[[SI:.+]] = llvm.mlir.undef : !llvm.struct<(f32, f32, f32, f32, f32, f32, f32, f32)>
  // CHECK: llvm.insertvalue %[[BC]], %[[SI]][0] : !llvm.struct<(f32, f32, f32, f32, f32, f32, f32, f32)>
    %b_mat = arith.constant dense<1.000000e+00> : tensor<16x32xf32, #dot_operand_b>
    %28 = tt.dot %a, %b_mat, %c, inputPrecision = tf32 : tensor<32x16xf32, #dot_operand_a> * tensor<16x32xf32, #dot_operand_b> -> tensor<32x32xf32, #mma>
    %38 = ttg.convert_layout %28 : tensor<32x32xf32, #mma> -> tensor<32x32xf32, #blocked>
    %30 = tt.splat %ptr : !tt.ptr<f32> -> tensor<32x1x!tt.ptr<f32>, #blocked>
    %36 = tt.broadcast %30 : tensor<32x1x!tt.ptr<f32>, #blocked> -> tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.store %36, %38 : tensor<32x32x!tt.ptr<f32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `matmul_tf32_cst_b`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `matmul_tf32_cst_b`。

### Line 1775
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1777-1808
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: matmul_f16_cst_operands
  tt.func public @matmul_f16_cst_operands(%arg0: !tt.ptr<f16> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x32xf32, #mma>
  // CHECK: %[[U:.+]] = llvm.mlir.undef : vector<2xf16>
  // CHECK: %[[C0:.+]] = llvm.mlir.constant(0 : i32) : i32
  // CHECK: %[[V0:.+]] = llvm.insertelement %{{.*}}, %[[U]][%[[C0]] : i32] : vector<2xf16>
  // CHECK: %[[C1:.+]] = llvm.mlir.constant(1 : i32) : i32
  // CHECK: %[[V1:.+]] = llvm.insertelement %{{.*}}, %[[V0]][%[[C1]] : i32] : vector<2xf16>
  // CHECK: %[[BC:.+]] = llvm.bitcast %[[V1]] : vector<2xf16> to i32
    %cst_0 = arith.constant dense<1.000000e+00> : tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>>
    %cst_1 = arith.constant dense<1.000000e+00> : tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>>
    %cst_2 = arith.constant dense<32> : tensor<32x1xi32, #blocked>
    %0 = tt.dot %cst_0, %cst_1, %cst : tensor<32x32xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>> * tensor<32x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>> -> tensor<32x32xf32, #mma>
    %1 = ttg.convert_layout %0 : tensor<32x32xf32, #mma> -> tensor<32x32xf32, #blocked>
    %2 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    %3 = tt.expand_dims %2 {axis = 1 : i32} : tensor<32xi32, #ttg.slice<{dim = 1, parent = #blocked}>> -> tensor<32x1xi32, #blocked>
    %4 = arith.muli %3, %cst_2 : tensor<32x1xi32, #blocked>
    %5 = tt.splat %arg0 : !tt.ptr<f16> -> tensor<32x1x!tt.ptr<f16>, #blocked>
    %6 = tt.addptr %5, %4 : tensor<32x1x!tt.ptr<f16>, #blocked>, tensor<32x1xi32, #blocked>
    %7 = tt.make_range {end = 32 : i32, start = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>>
    %8 = tt.expand_dims %7 {axis = 0 : i32} : tensor<32xi32, #ttg.slice<{dim = 0, parent = #blocked}>> -> tensor<1x32xi32, #blocked>
    %9 = tt.broadcast %6 : tensor<32x1x!tt.ptr<f16>, #blocked> -> tensor<32x32x!tt.ptr<f16>, #blocked>
    %10 = tt.broadcast %8 : tensor<1x32xi32, #blocked> -> tensor<32x32xi32, #blocked>
    %11 = tt.addptr %9, %10 : tensor<32x32x!tt.ptr<f16>, #blocked>, tensor<32x32xi32, #blocked>
    %12 = arith.truncf %1 : tensor<32x32xf32, #blocked> to tensor<32x32xf16, #blocked>
    tt.store %11, %12 : tensor<32x32x!tt.ptr<f16>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `matmul_f16_cst_operands`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `matmul_f16_cst_operands`。

### Line 1810
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1811-1822
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [1], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: test_s8_to_bf16_conversion
  tt.func @test_s8_to_bf16_conversion(%in: tensor<32xi8, #blocked>) {
    // We can't vectorize if we only process
    // CHECK-NOT: llvm.inline_asm
    // CHECK: llvm.sitofp
    // CHECK-NOT: llvm.sitofp
    %out = arith.sitofp %in : tensor<32xi8, #blocked> to tensor<32xbf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_s8_to_bf16_conversion`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_s8_to_bf16_conversion`。

### Line 1824
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1825-1838
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
#dot = #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32} {
  // CHECK-LABEL: test_s8_to_bf16_vectorized_conversion
  tt.func @test_s8_to_bf16_vectorized_conversion(%in: tensor<16x16xi8, #mma>) {
    // CHECK-NOT: llvm.sitofp
    // 8 elements per thread => we should process 2 vectors of 4
    // CHECK: llvm.inline_asm
    // CHECK: llvm.inline_asm
    // CHECK-NOT: llvm.inline_asm
    %out = arith.sitofp %in : tensor<16x16xi8, #mma> to tensor<16x16xbf16, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_s8_to_bf16_vectorized_conversion`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_s8_to_bf16_vectorized_conversion`。

### Line 1840
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1842-1858
```mlir
// CHECK-LABEL: sum_reduction
//       CHECK:  %[[M:.+]] = llvm.mlir.constant(-1 : i32) : i32
//       CHECK:   nvvm.redux.sync  add %{{.*}}, %[[M]]
//       CHECK:   nvvm.barrier0
//       CHECK:   nvvm.shfl.sync bfly
//       CHECK:   nvvm.shfl.sync bfly
#blocked = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @sum_reduction(%arg0: tensor<1x1024xi32, #blocked>) {
    %11 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%arg2: i32, %arg3: i32):
      %15 = arith.addi %arg2, %arg3 : i32
      tt.reduce.return %15 : i32
    }) : (tensor<1x1024xi32, #blocked>) -> tensor<1xi32, #ttg.slice<{dim = 1, parent = #blocked}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `sum_reduction`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `sum_reduction`。

### Line 1860
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1861-1875
```mlir
// CHECK-LABEL: @reduce_vec8_axisonly
// CHECK-COUNT-7: llvm.fadd {{.*}} : f16
// CHECK-NOT: llvm.fadd {{.*}} : vector<2xf16>
// CHECK: llvm.return
#blocked_vec8_axisonly = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [1, 32], warpsPerCTA = [1, 1], order = [1, 0]}>
module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @reduce_vec8_axisonly(%arg0: tensor<1x8xf16, #blocked_vec8_axisonly>) {
    %0 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f16, %b: f16):
      %c = arith.addf %a, %b : f16
      tt.reduce.return %c : f16
    }) : (tensor<1x8xf16, #blocked_vec8_axisonly>) -> tensor<1xf16, #ttg.slice<{dim = 1, parent = #blocked_vec8_axisonly}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_vec8_axisonly`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_vec8_axisonly`。

### Line 1877
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1878-1892
```mlir
// CHECK-LABEL: @reduce_vec8_firstbasis_nonaxis
// CHECK-COUNT-14: llvm.fadd {{.*}} : f16
// CHECK-NOT: llvm.fadd {{.*}} : vector<2xf16>
// CHECK: llvm.return
#blocked_vec8_firstbasis_nonaxis = #ttg.blocked<{sizePerThread = [2, 8], threadsPerWarp = [32, 1], warpsPerCTA = [1, 1], order = [0, 1]}>
module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @reduce_vec8_firstbasis_nonaxis(%arg0: tensor<64x8xf16, #blocked_vec8_firstbasis_nonaxis>) {
    %0 = "tt.reduce"(%arg0) <{axis = 1 : i32}> ({
    ^bb0(%a: f16, %b: f16):
      %c = arith.addf %a, %b : f16
      tt.reduce.return %c : f16
    }) : (tensor<64x8xf16, #blocked_vec8_firstbasis_nonaxis>) -> tensor<64xf16, #ttg.slice<{dim = 1, parent = #blocked_vec8_firstbasis_nonaxis}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_vec8_firstbasis_nonaxis`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_vec8_firstbasis_nonaxis`。

### Line 1894
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1895-1908
```mlir
#blocked = #ttg.blocked<{sizePerThread = [8, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 2], order = [1, 0]}>
#slice = #ttg.slice<{dim = 1, parent = #blocked}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 2 : i32} {
  // CHECK-LABEL: reduce_bools
  tt.func public @reduce_bools(%arg: tensor<256x2xi1, #blocked>) {
    // CHECK: llvm.mlir.addressof @global_smem
    %24 = "tt.reduce"(%arg) <{axis = 1 : i32}> ({
    ^bb0(%arg4: i1, %arg5: i1):
      %48 = arith.ori %arg4, %arg5 : i1
      tt.reduce.return %48 : i1
    }) : (tensor<256x2xi1, #blocked>) -> tensor<256xi1, #slice>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_bools`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_bools`。

### Line 1910
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1912-1927
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: inline_asm
  tt.func public @inline_asm(%arg0: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i8> {tt.divisibility = 16 : i32}) {
    %0 = tt.make_range {end = 512 : i32, start = 0 : i32} : tensor<512xi32, #blocked>
    %1 = tt.splat %arg0 : !tt.ptr<i8> -> tensor<512x!tt.ptr<i8>, #blocked>
    %2 = tt.addptr %1, %0 : tensor<512x!tt.ptr<i8>, #blocked>, tensor<512xi32, #blocked>
    %3 = tt.load %2 : tensor<512x!tt.ptr<i8>, #blocked>
// CHECK: %{{.*}} = llvm.inline_asm asm_dialect = att "shl.b32 $0, $0, 3;", "=r,r" %{{.*}} : (vector<4xi8>) -> vector<4xi8>
    %4 = tt.elementwise_inline_asm "shl.b32 $0, $0, 3;" {constraints = "=r,r", packed_element = 4 : i32, pure = true} %3 : tensor<512xi8, #blocked> -> tensor<512xi8, #blocked>
    %5 = tt.splat %arg1 : !tt.ptr<i8> -> tensor<512x!tt.ptr<i8>, #blocked>
    %6 = tt.addptr %5, %0 : tensor<512x!tt.ptr<i8>, #blocked>, tensor<512xi32, #blocked>
    tt.store %6, %4 : tensor<512x!tt.ptr<i8>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `, `, ` {constraints = `. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `inline_asm`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `, `, ` {constraints = `。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `inline_asm`。

### Line 1929
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1931-1946
```mlir
#blocked = #ttg.blocked<{sizePerThread = [4], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: inline_asm_pack_16bit
  tt.func public @inline_asm_pack_16bit(%arg0: !tt.ptr<i8> {tt.divisibility = 16 : i32}, %arg1: !tt.ptr<i8> {tt.divisibility = 16 : i32}) {
    %0 = tt.make_range {end = 512 : i32, start = 0 : i32} : tensor<512xi32, #blocked>
    %1 = tt.splat %arg0 : !tt.ptr<i8> -> tensor<512x!tt.ptr<i8>, #blocked>
    %2 = tt.addptr %1, %0 : tensor<512x!tt.ptr<i8>, #blocked>, tensor<512xi32, #blocked>
    %3 = tt.load %2 : tensor<512x!tt.ptr<i8>, #blocked>
// CHECK: %{{.*}} = llvm.inline_asm asm_dialect = att "shl.b16 $0, $0, 3;", "=h,h" %{{.*}} : (vector<2xi8>) -> vector<2xi8>
    %4 = tt.elementwise_inline_asm "shl.b16 $0, $0, 3;" {constraints = "=h,h", packed_element = 2 : i32, pure = true} %3 : tensor<512xi8, #blocked> -> tensor<512xi8, #blocked>
    %5 = tt.splat %arg1 : !tt.ptr<i8> -> tensor<512x!tt.ptr<i8>, #blocked>
    %6 = tt.addptr %5, %0 : tensor<512x!tt.ptr<i8>, #blocked>, tensor<512xi32, #blocked>
    tt.store %6, %4 : tensor<512x!tt.ptr<i8>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `, `, ` {constraints = `. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `inline_asm_pack_16bit`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `, `, ` {constraints = `。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `inline_asm_pack_16bit`。

### Line 1948
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1950-1965
```mlir
//  CHECK-LABEL: reduce_slice
//  CHECK-NOT: st.shared
//  CHECK-NOT: ld.shared
#blocked = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [4, 4, 2], warpsPerCTA = [2, 4, 2], order = [2, 0, 1]}>
#sliced2 = #ttg.slice<{dim = 2, parent = #blocked}>
module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 16 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @reduce_slice() {
    %cst = arith.constant dense<true> : tensor<4x1xi1, #sliced2>
    %0 = "tt.reduce"(%cst) <{axis = 1 : i32}> ({
    ^bb0(%arg0: i1, %arg1: i1):
      %1 = arith.ori %arg0, %arg1 : i1
      tt.reduce.return %1 : i1
    }) : (tensor<4x1xi1, #sliced2>) -> tensor<4xi1, #ttg.slice<{dim = 1, parent = #sliced2}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_slice`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_slice`。

### Line 1967
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1969-1984
```mlir
//  CHECK-LABEL: reduce_md_slice
//  CHECK: llvm.store {{.*}} vector<2xi32>
//  CHECK: llvm.load {{.*}} vector<2xi32>
#blocked = #ttg.blocked<{sizePerThread = [1, 1, 1], threadsPerWarp = [1, 1, 32], warpsPerCTA = [1, 2, 2], order = [2, 1, 0]}>
#sliced = #ttg.slice<{dim = 2, parent = #blocked}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @reduce_md_slice(%arg0: !tt.ptr<f32> {tt.divisibility = 16 : i32}) {
    %cst = arith.constant dense<0.000000e+00> : tensor<2x128xf32, #ttg.slice<{dim = 2, parent = #blocked}>>
    %0 = "tt.reduce"(%cst) <{axis = 1 : i32}> ({
    ^bb0(%arg1: f32, %arg2: f32):
      %18 = arith.maxnumf %arg1, %arg2 : f32
      tt.reduce.return %18 : f32
    }) {allocation.offset = 0 : i32} : (tensor<2x128xf32, #sliced>) -> tensor<2xf32, #ttg.slice<{dim = 1, parent = #sliced}>>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:80`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `reduce_md_slice`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:80`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `reduce_md_slice`。

### Line 1986
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 1988-1996
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1, 4], threadsPerWarp = [8, 4], warpsPerCTA = [1, 1], order = [1, 0]}>
#shared0 = #ttg.swizzled_shared<{vec = 8, perPhase=1, maxPhase=8, order = [1, 0]}>
#mma = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [1, 1], instrShape = [16, 8]}>
#dot_operand_a = #ttg.dot_op<{opIdx=0, parent=#mma, kWidth=2}>
#dot_operand_b = #ttg.dot_op<{opIdx=1, parent=#mma, kWidth=2}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, "ttg.threads-per-warp" = 32 : i32} {
  tt.func @i16_mma_layout(%f16_inp: tensor<16x16xf16, #blocked0>, %i16_inp: tensor<16x16xi16, #blocked0>) {
    // CHECK-LABEL: @i16_mma_layout
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `i16_mma_layout`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `i16_mma_layout`。

### Lines 1998-1999
```mlir
    %f16_shared = ttg.local_alloc %f16_inp : (tensor<16x16xf16, #blocked0>) -> !ttg.memdesc<16x16xf16, #shared0, #smem>
    %i16_shared = ttg.local_alloc %i16_inp : (tensor<16x16xi16, #blocked0>) -> !ttg.memdesc<16x16xi16, #shared0, #smem>
```
**EN:** This block contributes intermediate IR built from `ttg.local_alloc`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_alloc`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 2001-2002
```mlir
    // CHECK: nvvm.ldmatrix
    // CHECK: nvvm.ldmatrix
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 2004-2005
```mlir
    %f16_dot = ttg.local_load %f16_shared : !ttg.memdesc<16x16xf16, #shared0, #smem> -> tensor<16x16xf16, #dot_operand_a>
    %i16_dot = ttg.local_load %i16_shared : !ttg.memdesc<16x16xi16, #shared0, #smem> -> tensor<16x16xi16, #dot_operand_b>
```
**EN:** This block contributes intermediate IR built from `ttg.local_load`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_load`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 2007
```mlir
    // CHECK: llvm.sitofp %{{.*}} : i16 to f16
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 2009-2010
```mlir
    %converted_i16 = arith.sitofp %i16_dot : tensor<16x16xi16, #dot_operand_b> to tensor<16x16xf16, #dot_operand_b>
    %cst0 = arith.constant dense<0.000000e+00> : tensor<16x16xf32, #mma>
```
**EN:** This block contributes intermediate IR built from `arith.sitofp`, `arith.constant`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `arith.sitofp`, `arith.constant` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 2012-2015
```mlir
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-SAME`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-SAME`），从而保证后端/代码生成结果稳定。

### Line 2017
```mlir
    %out = tt.dot %f16_dot, %converted_i16, %cst0 : tensor<16x16xf16, #dot_operand_a> * tensor<16x16xf16, #dot_operand_b> -> tensor<16x16xf32, #mma>
```
**EN:** This block contributes intermediate IR built from `tt.dot`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 2019-2021
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 2023
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2025-2032
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [1, 1], instrShape = [8, 8]}>
#shared = #ttg.swizzled_shared<{vec = 4, perPhase = 1, maxPhase = 4, order = [1, 0]}>
#shared1 = #ttg.swizzled_shared<{vec = 8, perPhase = 1, maxPhase = 2, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {ttg.global_scratch_memory_alignment = 1 : i32, ttg.global_scratch_memory_size = 0 : i32, "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 1 : i32, ttg.shared = 4096 : i32, ttg.target = "cuda:80", ttg.tensor_memory_size = 0 : i32, "ttg.threads-per-warp" = 32 : i32, "ttg.total-num-warps" = 1 : i32} {
  tt.func public @f64_mma_cvt() {
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<16x16xf64, #shared, #smem, mutable>
    %1 = ttg.local_alloc {allocation.offset = 2048 : i32} : () -> !ttg.memdesc<16x16xf64, #shared1, #smem, mutable>
```
**EN:** This module sets kernel-level metadata targeting `cuda:80`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.total-num-warps`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `f64_mma_cvt`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:80`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.total-num-warps`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `f64_mma_cvt`。

### Line 2034
```mlir
    %cst = arith.constant dense<0.000000e+00> : tensor<16x16xf64, #mma>
```
**EN:** This block contributes intermediate IR built from `arith.constant`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `arith.constant` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 2036
```mlir
    %2 = ttg.local_load %0 : !ttg.memdesc<16x16xf64, #shared, #smem, mutable> -> tensor<16x16xf64, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>>
```
**EN:** This block contributes intermediate IR built from `ttg.local_load`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_load`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 2038
```mlir
    %3 = ttg.local_load %1 : !ttg.memdesc<16x16xf64, #shared1, #smem, mutable> -> tensor<16x16xf64, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>>
```
**EN:** This block contributes intermediate IR built from `ttg.local_load`, `ttg.memdesc`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `ttg.local_load`, `ttg.memdesc` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 2040-2043
```mlir
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m8n8k4.row.col.f64.f64.f64.f64
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m8n8k4.row.col.f64.f64.f64.f64
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-SAME`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-SAME`），从而保证后端/代码生成结果稳定。

### Line 2045
```mlir
    %out = tt.dot %2, %3, %cst, inputPrecision = tf32 : tensor<16x16xf64, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 1}>> * tensor<16x16xf64, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 1}>> -> tensor<16x16xf64, #mma>
```
**EN:** This block contributes intermediate IR built from `tt.dot`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.dot` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Lines 2047-2049
```mlir
    tt.return
  }
}
```
**EN:** This block contributes intermediate IR built from `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 2052
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2054-2066
```mlir
#blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
module attributes {"ttg.target" = "cuda:75", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: convert_single_element
  // CHECK-NOT: llvm.store
  // CHECK-NOT: llvm.load
  // CHECK: llvm.return
  tt.func public @convert_single_element() {
    %cst = arith.constant dense<1.000000e+03> : tensor<1xf32, #blocked1>
    %0 = ttg.convert_layout %cst : tensor<1xf32, #blocked1> -> tensor<1xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_single_element`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_single_element`。

### Line 2068
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2070-2085
```mlir
#blocked = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
module attributes {"ttg.target" = "cuda:75", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: convert_single_element_and_add
  // CHECK-NOT: llvm.store
  // CHECK-NOT: llvm.load
  // CHECK: llvm.insertvalue
  // CHECK: llvm.extractvalue
  tt.func public @convert_single_element_and_add() {
    %cst = arith.constant dense<1.000000e+03> : tensor<1xf32, #blocked1>
    %cst2 = arith.constant dense<1.000000e+03> : tensor<1xf32, #blocked>
    %0 = ttg.convert_layout %cst : tensor<1xf32, #blocked1> -> tensor<1xf32, #blocked>
    %1 = arith.addf %0, %cst2 : tensor<1xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_single_element_and_add`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_single_element_and_add`。

### Line 2087
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2089-2101
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @vectorize_shmem_load
  // CHECK: llvm.load
  // CHECK-SAME: {alignment = 8 : i64} : !llvm.ptr<3> -> vector<2xi32>
  // CHECK-NOT: llvm.load
  tt.func public @vectorize_shmem_load(%shmem : !ttg.memdesc<16x16xi8, #shared, #smem>) {
    %0 = ttg.local_load %shmem : !ttg.memdesc<16x16xi8, #shared, #smem> -> tensor<16x16xi8, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `vectorize_shmem_load`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `vectorize_shmem_load`。

### Line 2103
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2105-2115
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#blocked = #ttg.blocked<{sizePerThread = [1, 16], threadsPerWarp = [8, 4], warpsPerCTA = [8, 1], order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "cuda:90", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @vectorize_shmem_store
  // CHECK-COUNT-4:  llvm.store {{.*}} {alignment = 16 : i64} : vector<4xi32>, !llvm.ptr<3>
  tt.func public @vectorize_shmem_store(%block : tensor<64x64xi32, #blocked>) {
    %0 = ttg.local_alloc %block : (tensor<64x64xi32, #blocked>) -> !ttg.memdesc<64x64xi32, #shared, #smem>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `vectorize_shmem_store`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `vectorize_shmem_store`。

### Line 2117
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2119-2127
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [2], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: abs_is_int_min_poison
  // CHECK: %{{.*}} = "llvm.intr.abs"(%{{.*}}) <{is_int_min_poison = false}> : (i32) -> i32
  tt.func @abs_is_int_min_poison(%arg0 : tensor<256xi32, #blocked0>) {
    %abs = math.absi %arg0 : tensor<256xi32, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `abs_is_int_min_poison`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `abs_is_int_min_poison`。

### Line 2129
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2130-2144
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 8], threadsPerWarp = [1, 32], warpsPerCTA = [1, 8], order = [1, 0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.target" = "cuda:80", "ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: test_local_load_bf16
  // CHECK: llvm.extractelement {{.*}} : vector<8xbf16>
  tt.func public @test_local_load_bf16() {
    %c0_i32 = arith.constant 0 : i32
    %19 = ttg.local_alloc : () -> !ttg.memdesc<1x1x2048xbf16, #shared, #smem, mutable>
    %22 = ttg.memdesc_index %19[%c0_i32] : !ttg.memdesc<1x1x2048xbf16, #shared, #smem, mutable> -> !ttg.memdesc<1x2048xbf16, #shared, #smem, mutable>
    %39 = ttg.local_load %22 : !ttg.memdesc<1x2048xbf16, #shared, #smem, mutable> -> tensor<1x2048xbf16, #blocked>
    %40 = arith.extf %39 : tensor<1x2048xbf16, #blocked> to tensor<1x2048xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_local_load_bf16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.target`, `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_local_load_bf16`。

### Line 2146
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2147-2159
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: test_local_store
  // CHECK: llvm.store
  tt.func public @test_local_store(%arg0: tensor<1xf32, #blocked>) {
    %c0_i32 = arith.constant 0 : i32
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xf32, #shared, #smem, mutable>
    ttg.local_store %arg0, %0 : tensor<1xf32, #blocked> -> !ttg.memdesc<1xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_local_store`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_local_store`。

### Line 2161
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2163-2176
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: test_local_store_subview
  // CHECK: llvm.store
  tt.func public @test_local_store_subview(%arg0: tensor<1xf32, #blocked>) {
    %c0_i32 = arith.constant 0 : i32
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1x1xf32, #shared, #smem, mutable>
    %sv = ttg.memdesc_index %0[%c0_i32] : !ttg.memdesc<1x1xf32, #shared, #smem, mutable> -> !ttg.memdesc<1xf32, #shared, #smem, mutable>
    ttg.local_store %arg0, %sv : tensor<1xf32, #blocked> -> !ttg.memdesc<1xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_local_store_subview`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_local_store_subview`。

### Line 2178
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2180-2195
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: test_local_atomic_scatter_add
  // CHECK: llvm.inline_asm has_side_effects asm_dialect = att
  // CHECK-SAME: atom.shared.cta.relaxed.add.u32
  // CHECK-SAME: "=r,r,r,b"
  tt.func public @test_local_atomic_scatter_add(%arg0: tensor<1xi32, #blocked>, %arg1: tensor<1xi32, #blocked>) {
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    %1 = ttg.local_atomic_scatter_rmw add, %0[%arg0], %arg1 {axis = 0 : i32} : (!ttg.memdesc<1xi32, #shared, #smem, mutable>, tensor<1xi32, #blocked>, tensor<1xi32, #blocked>) -> tensor<1xi32, #blocked>
    %2 = ttg.local_alloc {allocation.offset = 4 : i32} : () -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    ttg.local_store %1, %2 : tensor<1xi32, #blocked> -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_local_atomic_scatter_add`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_local_atomic_scatter_add`。

### Line 2197
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2199-2215
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: test_local_atomic_inc_masked
  // CHECK: llvm.inline_asm has_side_effects asm_dialect = att
  // CHECK-SAME: atom.shared.cta.relaxed.inc.u32
  // CHECK-SAME: "=r,r,b"
  tt.func public @test_local_atomic_inc_masked(%arg0: tensor<1xi32, #blocked>, %mask: tensor<1xi1, #blocked>) {
    %c1 = arith.constant dense<1> : tensor<1xi32, #blocked>
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    %1 = ttg.local_atomic_scatter_rmw add, %0[%arg0], %c1, %mask {axis = 0 : i32} : (!ttg.memdesc<1xi32, #shared, #smem, mutable>, tensor<1xi32, #blocked>, tensor<1xi32, #blocked>, tensor<1xi1, #blocked>) -> tensor<1xi32, #blocked>
    %2 = ttg.local_alloc {allocation.offset = 4 : i32} : () -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    ttg.local_store %1, %2 : tensor<1xi32, #blocked> -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_local_atomic_inc_masked`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_local_atomic_inc_masked`。

### Line 2217
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2219-2234
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: test_local_atomic_scatter_add_i32_masked
  // CHECK: llvm.inline_asm has_side_effects asm_dialect = att
  // CHECK-SAME: atom.shared.cta.relaxed.add.u32
  // CHECK-SAME: "=r,r,r,b"
  tt.func public @test_local_atomic_scatter_add_i32_masked(%arg0: tensor<1xi32, #blocked>, %arg1: tensor<1xi32, #blocked>, %mask: tensor<1xi1, #blocked>) {
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    %1 = ttg.local_atomic_scatter_rmw add, %0[%arg0], %arg1, %mask {axis = 0 : i32} : (!ttg.memdesc<1xi32, #shared, #smem, mutable>, tensor<1xi32, #blocked>, tensor<1xi32, #blocked>, tensor<1xi1, #blocked>) -> tensor<1xi32, #blocked>
    %2 = ttg.local_alloc {allocation.offset = 4 : i32} : () -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    ttg.local_store %1, %2 : tensor<1xi32, #blocked> -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_local_atomic_scatter_add_i32_masked`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_local_atomic_scatter_add_i32_masked`。

### Line 2236
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2238-2253
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: test_local_atomic_scatter_rmw_max_i32
  // CHECK: llvm.inline_asm has_side_effects asm_dialect = att
  // CHECK-SAME: atom.shared.cta.relaxed.max.s32
  // CHECK-SAME: "=r,r,r,b"
  tt.func public @test_local_atomic_scatter_rmw_max_i32(%arg0: tensor<1xi32, #blocked>, %arg1: tensor<1xi32, #blocked>, %mask: tensor<1xi1, #blocked>) {
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    %1 = ttg.local_atomic_scatter_rmw max, %0[%arg0], %arg1, %mask {axis = 0 : i32} : (!ttg.memdesc<1xi32, #shared, #smem, mutable>, tensor<1xi32, #blocked>, tensor<1xi32, #blocked>, tensor<1xi1, #blocked>) -> tensor<1xi32, #blocked>
    %2 = ttg.local_alloc {allocation.offset = 4 : i32} : () -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    ttg.local_store %1, %2 : tensor<1xi32, #blocked> -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_local_atomic_scatter_rmw_max_i32`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_local_atomic_scatter_rmw_max_i32`。

### Line 2255
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2257-2272
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: test_local_atomic_scatter_add_f32_masked
  // CHECK: llvm.inline_asm has_side_effects asm_dialect = att
  // CHECK-SAME: atom.shared.cta.relaxed.add.f32
  // CHECK-SAME: "=r,r,r,b"
  tt.func public @test_local_atomic_scatter_add_f32_masked(%arg0: tensor<1xi32, #blocked>, %arg1: tensor<1xf32, #blocked>, %mask: tensor<1xi1, #blocked>) {
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xf32, #shared, #smem, mutable>
    %1 = ttg.local_atomic_scatter_rmw fadd, %0[%arg0], %arg1, %mask {axis = 0 : i32} : (!ttg.memdesc<1xf32, #shared, #smem, mutable>, tensor<1xf32, #blocked>, tensor<1xi32, #blocked>, tensor<1xi1, #blocked>) -> tensor<1xf32, #blocked>
    %2 = ttg.local_alloc {allocation.offset = 4 : i32} : () -> !ttg.memdesc<1xf32, #shared, #smem, mutable>
    ttg.local_store %1, %2 : tensor<1xf32, #blocked> -> !ttg.memdesc<1xf32, #shared, #smem, mutable>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_local_atomic_scatter_add_f32_masked`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_local_atomic_scatter_add_f32_masked`。

### Line 2274
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2276-2289
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: test_local_atomic_inc_dead
  // CHECK: llvm.inline_asm has_side_effects asm_dialect = att
  // CHECK-SAME: red.shared.cta.relaxed.inc.u32
  // CHECK-SAME: "r,b"
  tt.func public @test_local_atomic_inc_dead(%arg0: tensor<1xi32, #blocked>) {
    %c1 = arith.constant dense<1> : tensor<1xi32, #blocked>
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    %1 = ttg.local_atomic_scatter_rmw add, %0[%arg0], %c1 {axis = 0 : i32} : (!ttg.memdesc<1xi32, #shared, #smem, mutable>, tensor<1xi32, #blocked>, tensor<1xi32, #blocked>) -> tensor<1xi32, #blocked>
    tt.return
  }
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_local_atomic_inc_dead`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_local_atomic_inc_dead`。

### Lines 2291-2301
```mlir
  // CHECK-LABEL: test_local_atomic_add_dead
  // CHECK: llvm.inline_asm has_side_effects asm_dialect = att
  // CHECK-SAME: red.shared.cta.relaxed.add.u32
  // CHECK-SAME: "r,r,b"
  tt.func public @test_local_atomic_add_dead(%arg0: tensor<1xi32, #blocked>) {
    %c2 = arith.constant dense<2> : tensor<1xi32, #blocked>
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xi32, #shared, #smem, mutable>
    %1 = ttg.local_atomic_scatter_rmw add, %0[%arg0], %c2 {axis = 0 : i32} : (!ttg.memdesc<1xi32, #shared, #smem, mutable>, tensor<1xi32, #blocked>, tensor<1xi32, #blocked>) -> tensor<1xi32, #blocked>
    tt.return
  }
}
```
**EN:** This block defines `test_local_atomic_add_dead` and exercises shared-memory allocation, backend inline assembly. Embedded check comments (CHECK×1, CHECK-LABEL×1, CHECK-SAME×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `test_local_atomic_add_dead`，并覆盖 共享内存分配、后端内联汇编。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1, CHECK-SAME×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 2303
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2305-2313
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: print_ptr
  // CHECK: llvm.call @vprintf(%{{.*}}, %{{.*}}) : (!llvm.ptr, !llvm.ptr) -> i32
  tt.func @print_ptr(%arg0 : tensor<256x!tt.ptr<i32>, #blocked0>) {
    tt.print "ptr: " {hex = false, isSigned = array<i32: 0>} : %arg0 : tensor<256x!tt.ptr<i32>, #blocked0>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `print_ptr`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `print_ptr`。

### Line 2315
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2316-2326
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // Test that %u format specifier is used if isSigned is false
  // CHECK: llvm.mlir.global internal constant @printfFormat_0("{{.*}}int32 tensor: %u{{.*}}")
  // CHECK-LABEL: print_int32_tensor_issigned_off
  // CHECK: llvm.call @vprintf(%{{.*}}, %{{.*}}) : (!llvm.ptr, !llvm.ptr) -> i32
  tt.func @print_int32_tensor_issigned_off(%arg0 : i32) {
    tt.print "int32 tensor: " {hex = false, isSigned = array<i32: 0>} : %arg0 : i32
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `print_int32_tensor_issigned_off`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `print_int32_tensor_issigned_off`。

### Line 2328
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2329-2339
```mlir
#blocked0 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // Test that %i format specifier is used if isSigned is true
  // CHECK: llvm.mlir.global internal constant @printfFormat_0("{{.*}}int32 tensor: %i{{.*}}")
  // CHECK-LABEL: print_int32_tensor_issigned_on
  // CHECK: llvm.call @vprintf(%{{.*}}, %{{.*}}) : (!llvm.ptr, !llvm.ptr) -> i32
  tt.func @print_int32_tensor_issigned_on(%arg0 : i32) {
    tt.print "int32 tensor: " {hex = false, isSigned = array<i32: 1>} : %arg0 : i32
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `print_int32_tensor_issigned_on`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `print_int32_tensor_issigned_on`。

### Line 2341
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2343-2351
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  tt.func @int32_to_bf16(%arg0: tensor<256xi32, #blocked>) {
    // CHECK-LABEL: @int32_to_bf16
    // CHECK: llvm.sitofp %{{.*}} : i32 to bf16
    %a = arith.sitofp %arg0 : tensor<256xi32, #blocked> to tensor<256xbf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `int32_to_bf16`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `int32_to_bf16`。

### Line 2353
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2355-2363
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  tt.func @bf16_to_int32(%arg0: tensor<256xbf16, #blocked>) {
    // CHECK-LABEL: @bf16_to_int32
    // CHECK: llvm.fptosi %{{.*}} : bf16 to i32
    %a = arith.fptosi %arg0 : tensor<256xbf16, #blocked> to tensor<256xi32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `bf16_to_int32`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `bf16_to_int32`。

### Line 2365
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2367-2383
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
// CHECK-DAG: llvm.mlir.global internal constant @assertFunc_0("unknown\00") {addr_space = 0 : i32}
// CHECK-DAG: llvm.mlir.global internal constant @assertFile_0("inner_call\00") {addr_space = 0 : i32}
// CHECK-DAG: llvm.mlir.global internal constant @assertMessage_0("assert text\00") {addr_space = 0 : i32}
// CHECK: llvm.call @__assertfail
// CHECK: nvvm.barrier0
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @add_kernel(%arg0: tensor<1xi1, #blocked>) {
    tt.assert %arg0, "assert text" : tensor<1xi1, #blocked> loc(#loc5)
    tt.return
  }
}
#loc1 = loc("outer_call":33:8)
#loc2 = loc("top_func":47:8)
#loc3 = loc("inner_call":29:28)
#loc4 = loc(callsite(#loc3 at #loc1))
#loc5 = loc(callsite(#loc4 at #loc2))
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `add_kernel`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `add_kernel`。

### Line 2385
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2387-2401
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [32, 1], warpsPerCTA = [1, 4], order = [0, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @log1pf_scan(%39: tensor<32x16xf32, #blocked>) {
    // CHECK: log1pf_scan
    // non-speculatable ops will introduce a cond_br; extern_elementwise with pure = true should be considered speculatable.
    // CHECK-NOT: llvm.cond_br
    %40 = "tt.scan"(%39) <{axis = 1 : i32, reverse = false}> ({
    ^bb0(%arg5: f32, %arg6: f32):
      %43 = tt.extern_elementwise %arg5 {libname = "", libpath = "", pure = true, symbol = "__nv_log1pf"} : (f32) -> f32
      %44 = arith.addf %43, %43 : f32
      tt.scan.return %44 : f32
    }) : (tensor<32x16xf32, #blocked>) -> tensor<32x16xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `log1pf_scan`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `log1pf_scan`。

### Line 2403
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2405-2414
```mlir
// CHECK: inline_asm_pack
#blocked = #ttg.blocked<{sizePerThread = [16, 1], threadsPerWarp = [4, 8], warpsPerCTA = [1, 4], order = [0, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  // check specifically for the case where asm has two results, pack > 1, and the result bitwidth is < 32
  tt.func public @inline_asm_pack(%80: tensor<64x64xi8, #blocked>) {
    // CHECK: llvm.inline_asm asm_dialect {{.*}} (vector<4xi8>) -> !llvm.struct<(vector<2xbf16>, vector<2xbf16>, vector<2xbf16>, vector<2xbf16>)>
    %83:2 = tt.elementwise_inline_asm "" {constraints = "=r,=r,=r,=r,r", packed_element = 4 : i32, pure = true} %80 : tensor<64x64xi8, #blocked> -> tensor<64x64xbf16, #blocked>, tensor<64x64xbf16, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, ` {constraints = `, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `inline_asm_pack`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, ` {constraints = `, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `inline_asm_pack`。

### Line 2416
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2418-2419
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [8, 4], warpsPerCTA = [4, 1], order = [1, 0]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#blocked1`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#blocked1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Line 2421
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 2423-2428
```mlir
tt.func @gather_in_shared(%arg0: tensor<16x4xi32, #blocked1>, %arg1: tensor<8x4xf32, #blocked>) {
  // CHECK-LABEL: gather_in_shared
  // CHECK: [[SMEM_BASE:%.*]] = llvm.mlir.addressof @global_smem
  // CHECK-NEXT: [[SMEM:%.*]] = llvm.getelementptr [[SMEM_BASE]]
  // CHECK: store
  // CHECK-NEXT: nvvm.barrier0
```
**EN:** This block defines `gather_in_shared` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `gather_in_shared`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×2, CHECK-LABEL×1, CHECK-NEXT×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 2430
```mlir
  // CHECK: [[I0:%.*]] = llvm.extractvalue %arg0[0]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 2432-2437
```mlir
  // CHECK: [[IDX:%.*]] = llvm.add {{.*}}, [[I0]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM]][[[IDX]]]
  // CHECK: llvm.load [[PTR]]
  // CHECK: llvm.load
  // CHECK-NOT: llvm.load
  // CHECK: return
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`, `CHECK-NOT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`, `CHECK-NOT`），从而保证后端/代码生成结果稳定。

### Lines 2439-2441
```mlir
  %0 = tt.gather %arg1[%arg0] {axis = 0 : i32} : (tensor<8x4xf32, #blocked>, tensor<16x4xi32, #blocked1>) -> tensor<16x4xf32, #blocked1>
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `tt.gather`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.gather`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 2443
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 2445
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2447-2449
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, warpsPerCTA = [4, 1], instrShape = [8, 8]}>
#dot = #ttg.dot_op<{opIdx=0, parent=#mma, kWidth=1}>
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [16, 2], warpsPerCTA = [4, 1], order = [1, 0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma`, `#dot`, `#blocked`. They parameterize later tests with compact names for `#ttg.nvidia_mma`, `#ttg.dot_op`, `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma`, `#dot`, `#blocked`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`, `#ttg.dot_op`, `#ttg.blocked`。

### Line 2451
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 2453-2454
```mlir
tt.func @gather_in_shared_dot_input(%arg0: tensor<16x4xi32, #blocked>, %arg1: tensor<8x4xf32, #dot>) {
  // CHECK-LABEL: gather_in_shared_dot_input
```
**EN:** This block defines `gather_in_shared_dot_input` and exercises operations such as `tt.func`. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `gather_in_shared_dot_input`，并覆盖 如 `tt.func` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 2456
```mlir
  // CHECK: [[S0:%.*]] = llvm.extractvalue %arg1[0]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 2458-2461
```mlir
  // CHECK: [[SMEM_BASE:%.*]] = llvm.mlir.addressof @global_smem
  // CHECK-NEXT: [[SMEM:%.*]] = llvm.getelementptr [[SMEM_BASE]]
  // CHECK: insertelement [[S0]]
  // CHECK: nvvm.barrier0
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`），从而保证后端/代码生成结果稳定。

### Line 2463
```mlir
  // CHECK: [[I0:%.*]] = llvm.extractvalue %arg0[0]
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`），从而保证后端/代码生成结果稳定。

### Lines 2465-2470
```mlir
  // CHECK: [[IDX:%.*]] = llvm.add {{.*}}, [[I0]]
  // CHECK-NEXT: [[PTR:%.*]] = llvm.getelementptr [[SMEM]][[[IDX]]]
  // CHECK: llvm.load [[PTR]]
  // CHECK: llvm.load
  // CHECK-NOT: llvm.load
  // CHECK: return
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK`, `CHECK-NEXT`, `CHECK-NOT`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK`, `CHECK-NEXT`, `CHECK-NOT`），从而保证后端/代码生成结果稳定。

### Lines 2472-2474
```mlir
  %0 = tt.gather %arg1[%arg0] {axis = 0 : i32} : (tensor<8x4xf32, #dot>, tensor<16x4xi32, #blocked>) -> tensor<16x4xf32, #blocked>
  tt.return
}
```
**EN:** This block contributes intermediate IR built from `tt.gather`, `tt.return`. It prepares data, layout, or control-flow state for the surrounding test scenario.
**CN:** 这一块提供了由 `tt.gather`, `tt.return` 构成的中间 IR，用于为周围测试场景准备数据、布局或控制流状态。

### Line 2476
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 2478
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 2480
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma`. They parameterize later tests with compact names for `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`。

### Line 2482
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 3072 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:80`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:80`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 2484-2489
```mlir
  tt.func public @ampere_s8_to_fp16_conversion_opIdx1(%1 : tensor<16x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>) {
    // CHECK-LABEL: ampere_s8_to_fp16_conversion_opIdx1
    // CHECK: llvm.sitofp %{{.*}} : i8 to f16
    %2 = arith.sitofp %1 : tensor<16x32xi8, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> to tensor<16x32xf16, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>
    tt.return
}
```
**EN:** This block defines `ampere_s8_to_fp16_conversion_opIdx1` and exercises operations such as `tt.func`, `llvm.sitofp`, `arith.sitofp`, `tt.return`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `ampere_s8_to_fp16_conversion_opIdx1`，并覆盖 如 `tt.func`、`llvm.sitofp`、`arith.sitofp`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 2491
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 2493
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 2495
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 2], instrShape = [16, 8]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma`. They parameterize later tests with compact names for `#ttg.nvidia_mma`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`。

### Lines 2497-2503
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.shared = 3072 : i32, ttg.target = "cuda:80", "ttg.threads-per-warp" = 32 : i32} {
  tt.func public @ampere_s8_to_fp16_conversion_opIdx0(%1 : tensor<32x16xi8, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>) {
    // CHECK-LABEL: @ampere_s8_to_fp16_conversion_opIdx0
    // CHECK: llvm.sitofp %{{.*}} : i8 to f16
    %2 = arith.sitofp %1 : tensor<32x16xi8, #ttg.dot_op<{opIdx = 0 , parent = #mma, kWidth = 4}>> to tensor<32x16xf16, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>
    tt.return
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:80`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `ampere_s8_to_fp16_conversion_opIdx0`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:80`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.shared`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `ampere_s8_to_fp16_conversion_opIdx0`。

### Line 2505
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 2507
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2509-2516
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [2, 4], instrShape = [16, 8]}>
module attributes {"ttg.num-warps" = 8 : i32, ttg.target = "cuda:120"} {
  // CHECK-LABEL: mmav2_e5m2_e5m2_fp16
  tt.func public @mmav2_e5m2_e5m2_fp16(%arg0: tensor<32x32xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg1: tensor<32x32xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>, %arg2: tensor<32x32xf16, #mma>) {
    // CHECK: mma.{{.*}}.col.f16.e5m2.e5m2.f16
    %0 = tt.dot %arg0, %arg1, %arg2 {maxNumImpreciseAcc = 1073741824 : i32} : tensor<32x32xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x32xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<32x32xf16, #mma>
    tt.return
  }
```
**EN:** This module sets kernel-level metadata targeting `cuda:120`, including `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `mmav2_e5m2_e5m2_fp16`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:120`，其中包括 `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `mmav2_e5m2_e5m2_fp16`。

### Lines 2518-2523
```mlir
  // CHECK-LABEL: mmav2_e5m2_e4m3_fp16
  tt.func public @mmav2_e5m2_e4m3_fp16(%arg0: tensor<32x32xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg1: tensor<32x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>, %arg2: tensor<32x32xf16, #mma>) {
    // CHECK: mma.{{.*}}.col.f16.e5m2.e4m3.f16
    %0 = tt.dot %arg0, %arg1, %arg2 {maxNumImpreciseAcc = 1073741824 : i32} : tensor<32x32xf8E5M2, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<32x32xf16, #mma>
    tt.return
  }
```
**EN:** This block defines `mmav2_e5m2_e4m3_fp16` and exercises dot products / matmul. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `mmav2_e5m2_e4m3_fp16`，并覆盖 点积/矩阵乘。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 2525-2530
```mlir
  // CHECK-LABEL: mmav2_e4m3_e5m2_fp16
  tt.func public @mmav2_e4m3_e5m2_fp16(%arg0: tensor<32x32xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg1: tensor<32x32xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>, %arg2: tensor<32x32xf16, #mma>) {
    // CHECK: mma.{{.*}}.col.f16.e4m3.e5m2.f16
    %0 = tt.dot %arg0, %arg1, %arg2 {maxNumImpreciseAcc = 1073741824 : i32} : tensor<32x32xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x32xf8E5M2, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<32x32xf16, #mma>
    tt.return
  }
```
**EN:** This block defines `mmav2_e4m3_e5m2_fp16` and exercises dot products / matmul. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `mmav2_e4m3_e5m2_fp16`，并覆盖 点积/矩阵乘。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 2532-2538
```mlir
  // CHECK-LABEL: mmav2_e4m3_e4m3_fp16
  tt.func public @mmav2_e4m3_e4m3_fp16(%arg0: tensor<32x32xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>>, %arg1: tensor<32x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>>, %arg2: tensor<32x32xf16, #mma>) {
    // CHECK: mma.{{.*}}.col.f16.e4m3.e4m3.f16
    %0 = tt.dot %arg0, %arg1, %arg2 {maxNumImpreciseAcc = 1073741824 : i32} : tensor<32x32xf8E4M3FN, #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 4}>> * tensor<32x32xf8E4M3FN, #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 4}>> -> tensor<32x32xf16, #mma>
    tt.return
  }
}
```
**EN:** This block defines `mmav2_e4m3_e4m3_fp16` and exercises dot products / matmul. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `mmav2_e4m3_e4m3_fp16`，并覆盖 点积/矩阵乘。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 2540
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2542-2543
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1, 16], threadsPerWarp = [4, 4, 2], warpsPerCTA = [8, 1, 1], order = [2, 1, 0]}>
#linear = #ttg.linear<{register = [[0, 0], [0, 0], [0, 0], [0, 0]], lane = [[0, 0], [0, 1], [0, 2], [1, 0], [2, 0]], warp = [[4, 0], [8, 0], [16, 0]], block = []}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`, `#linear`. They parameterize later tests with compact names for `#ttg.blocked`, `#ttg.linear`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`, `#linear`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`, `#ttg.linear`。

### Line 2545
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 2547-2553
```mlir
// CHECK-LABEL: expand_dims_linear_layout
tt.func private @expand_dims_linear_layout() -> tensor<1x4xi32, #linear> {
  %0 = tt.make_range {end = 4 : i32, start = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #linear}>>
  %1 = tt.expand_dims %0 {axis = 0 : i32} : tensor<4xi32, #ttg.slice<{dim = 0, parent = #linear}>> -> tensor<1x4xi32, #linear>
  // CHECK: return %{{.*}} : !llvm.struct<(i32, i32, i32, i32, i32, i32, i32, i32, i32, i32, i32, i32, i32, i32, i32, i32)>
  tt.return %1 : tensor<1x4xi32, #linear>
}
```
**EN:** This block defines `expand_dims_linear_layout` and exercises range generation. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `expand_dims_linear_layout`，并覆盖 范围生成。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Lines 2555-2561
```mlir
// CHECK-LABEL: reshape_linear_layout_broadcasting
tt.func private @reshape_linear_layout_broadcasting(%arg0: tensor<32x4xbf16, #linear>) -> tensor<32x4x1xbf16, #blocked> {
  // CHECK-COUNT-16: extractvalue
  // CHECK-COUNT-16: insertvalue
  %0 = tt.reshape %arg0 : tensor<32x4xbf16, #linear> -> tensor<32x4x1xbf16, #blocked>
  tt.return %0 : tensor<32x4x1xbf16, #blocked>
}
```
**EN:** This block defines `reshape_linear_layout_broadcasting` and exercises operations such as `tt.func`, `tt.reshape`, `tt.return`. Embedded check comments (CHECK-COUNT×2, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `reshape_linear_layout_broadcasting`，并覆盖 如 `tt.func`、`tt.reshape`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK-COUNT×2, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 2563
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 2566
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2568-2584
```mlir
#linear1 = #ttg.linear<{register = [[0, 0, 0, 1], [0, 0, 1, 0], [0, 1, 0, 0], [16, 0, 0, 0], [32, 0, 0, 0], [64, 0, 0, 0]], lane = [[0, 0, 0, 0], [0, 0, 0, 0], [0, 0, 0, 0], [1, 0, 0, 0], [2, 0, 0, 0]], warp = [[4, 0, 0, 0], [8, 0, 0, 0]], block = []}>
#linear2 = #ttg.linear<{register = [[0, 0, 1], [0, 1, 0], [16, 0, 0], [32, 0, 0], [64, 0, 0]], lane = [[0, 0, 0], [0, 0, 0], [0, 0, 0], [1, 0, 0], [2, 0, 0]], warp = [[4, 0, 0], [8, 0, 0]], block = []}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
// CHECK-LABEL: split_linear
tt.func @split_linear(%arg : tensor<128x2x2x2xf32, #linear1>) {
  // CHECK: %[[E0:.+]] = llvm.extractvalue %{{.*}}[0]
  // CHECK: %[[E1:.+]] = llvm.extractvalue %{{.*}}[1]
  // CHECK: %[[E2:.+]] = llvm.extractvalue %{{.*}}[2]
  // CHECK: %[[E3:.+]] = llvm.extractvalue %{{.*}}[3]
  // CHECK: llvm.insertvalue %[[E0]], %{{.*}}[0]
  // CHECK: llvm.insertvalue %[[E2]], %{{.*}}[1]
  // CHECK: llvm.insertvalue %[[E1]], %{{.*}}[0]
  // CHECK: llvm.insertvalue %[[E3]], %{{.*}}[1]
  %outLHS, %outRHS = tt.split %arg : tensor<128x2x2x2xf32, #linear1> -> tensor<128x2x2xf32, #linear2>
  tt.return
}
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `split_linear`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `split_linear`。

### Line 2586
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2588-2604
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 64, 2], threadsPerWarp = [32, 1, 1], warpsPerCTA = [4, 1, 1], order = [0, 1, 2]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: split_stride
  tt.func public @split_stride(%arg0: tensor<128x64x2xf32, #blocked>) {
  // CHECK: %[[E0:.+]] = llvm.extractvalue %{{.*}}[0]
  // CHECK: %[[E1:.+]] = llvm.extractvalue %{{.*}}[1]
  // CHECK: %[[E64:.+]] = llvm.extractvalue %{{.*}}[64]
  // CHECK: %[[E65:.+]] = llvm.extractvalue %{{.*}}[65]
  // CHECK: llvm.insertvalue %[[E0]], %{{.*}}[0]
  // CHECK: llvm.insertvalue %[[E1]], %{{.*}}[1]
  // CHECK: llvm.insertvalue %[[E64]], %{{.*}}[0]
  // CHECK: llvm.insertvalue %[[E65]], %{{.*}}[1]
    %outLHS, %outRHS = tt.split %arg0 : tensor<128x64x2xf32, #blocked> -> tensor<128x64xf32, #blocked1>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `split_stride`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `split_stride`。

### Line 2606
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2608-2624
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 64, 2], threadsPerWarp = [32, 1, 1], warpsPerCTA = [4, 1, 1], order = [0, 1, 2]}>
#blocked1 = #ttg.blocked<{sizePerThread = [1, 64], threadsPerWarp = [32, 1], warpsPerCTA = [4, 1], order = [0, 1]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: join_stride
  tt.func public @join_stride(%arg0: tensor<128x64xf32, #blocked1>, %arg1: tensor<128x64xf32, #blocked1>) {
  // CHECK: %[[A0:.+]] = llvm.extractvalue %{{.*}}[0]
  // CHECK: %[[A1:.+]] = llvm.extractvalue %{{.*}}[1]
  // CHECK: %[[B0:.+]] = llvm.extractvalue %{{.*}}[0]
  // CHECK: %[[B1:.+]] = llvm.extractvalue %{{.*}}[1]
  // CHECK: llvm.insertvalue %[[A0]], %{{.*}}[0]
  // CHECK: llvm.insertvalue %[[A1]], %{{.*}}[1]
  // CHECK: llvm.insertvalue %[[B0]], %{{.*}}[64]
  // CHECK: llvm.insertvalue %[[B1]], %{{.*}}[65]
    %r = tt.join %arg0, %arg1 : tensor<128x64xf32, #blocked1> -> tensor<128x64x2xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `join_stride`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `join_stride`。

### Line 2626
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 2628
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared`. They parameterize later tests with compact names for `#ttg.swizzled_shared`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`。

### Line 2630
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:100"} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 2632-2637
```mlir
// CHECK-LABEL: @reinterpret_tensor_descriptor
tt.func private @reinterpret_tensor_descriptor(%arg0: !tt.ptr<i8, 0>) -> !tt.tensordesc<128x64xf16, #shared> {
  // CHECK-NEXT: llvm.addrspacecast %arg0 : !llvm.ptr to !llvm.ptr
  %0 = ttng.reinterpret_tensor_descriptor %arg0 : !tt.ptr<i8, 0> to !tt.tensordesc<128x64xf16, #shared>
  tt.return %0 : !tt.tensordesc<128x64xf16, #shared>
}
```
**EN:** This block defines `reinterpret_tensor_descriptor` and exercises operations such as `tt.func`, `tt.ptr`, `tt.tensordesc`, `llvm.addrspacecast`, `llvm.ptr`. Embedded check comments (CHECK-LABEL×1, CHECK-NEXT×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `reinterpret_tensor_descriptor`，并覆盖 如 `tt.func`、`tt.ptr`、`tt.tensordesc`、`llvm.addrspacecast`、`llvm.ptr` 这样的操作。 其中嵌入的检查注释（CHECK-LABEL×1, CHECK-NEXT×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 2639
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 2641
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 2643
```mlir
#blocked2 = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [2], order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked2`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked2`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Line 2645
```mlir
module attributes {"ttg.num-warps" = 4 : i32} {
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 2647-2659
```mlir
// CHECK-LABEL: @partition_axis_info
tt.func @partition_axis_info(%arg0: !tt.ptr<i32>, %arg1: !tt.ptr<i32>) {
  ttg.warp_specialize(%arg0)
  default {
    ttg.warp_yield
  }
  partition0(%arg2: !tt.ptr<i32>) num_warps(2) {
    %splatted = tt.splat %arg2 : !tt.ptr<i32> -> tensor<256x!tt.ptr<i32>, #blocked2>
    %input = tt.load %splatted : tensor<256x!tt.ptr<i32>, #blocked2>
    ttg.warp_return
  } : (!tt.ptr<i32>) -> ()
  tt.return
}
```
**EN:** This block defines `partition_axis_info` and exercises tensor loads. Embedded check comments (CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `partition_axis_info`，并覆盖 张量加载。 其中嵌入的检查注释（CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 2661
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 2663
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2665-2682
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, ttg.target = "cuda:90", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: test_call_without_smem
  tt.func public @test_call_without_smem() attributes {allocation.offset = 0 : i32} {
    %cst = arith.constant dense<0.000000e+00> : tensor<1xf32, #blocked>
    %0 = ttg.local_alloc {allocation.offset = 0 : i32} : () -> !ttg.memdesc<1xf32, #shared, #smem, mutable>
    ttg.local_store %cst, %0 : tensor<1xf32, #blocked> -> !ttg.memdesc<1xf32, #shared, #smem, mutable>
    // CHECK: llvm.call @call_no_smem_usage(%{{.+}}, %{{.+}}, %{{.+}}) : (!llvm.ptr<3>, !llvm.ptr<1>, !llvm.ptr<1>) -> ()
    tt.call @call_no_smem_usage() : () -> ()
    tt.return
  }
  // CHECK: llvm.func internal @call_no_smem_usage(%arg0: !llvm.ptr<3>, %arg1: !llvm.ptr<1>, %arg2: !llvm.ptr<1>)
  tt.func private @call_no_smem_usage() {
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `test_call_without_smem`, `call_no_smem_usage`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `test_call_without_smem`, `call_no_smem_usage`。

### Line 2684
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2686-2687
```mlir
#shared0 = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [1, 0]}>
#shared1 = #ttg.swizzled_shared<{vec = 2, perPhase = 2, maxPhase = 1, order = [1, 0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#shared0`, `#shared1`. They parameterize later tests with compact names for `#ttg.swizzled_shared`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#shared0`, `#shared1`。它们为后续测试提供紧凑名称，用来表示 `#ttg.swizzled_shared`。

### Line 2689
```mlir
module attributes {"ttg.num-warps" = 4 : i32, ttg.target = "cuda:90"} {
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 2691-2704
```mlir
// CHECK-LABEL: @memdesc_reinterpret
tt.func private @memdesc_reinterpret(%arg0: !ttg.memdesc<4x1024xi64, #shared0, #ttg.shared_memory, mutable>) {
  // CHECK: [[BASE_PTR:%.*]] = llvm.extractvalue %arg0[0]
  // CHECK: [[C0:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK: [[PTR:%.*]] = llvm.getelementptr [[BASE_PTR]][[[C0]]] : (!llvm.ptr<3>, i32) -> !llvm.ptr<3>, i64
  ttg.memdesc_reinterpret %arg0 : !ttg.memdesc<4x1024xi64, #shared0, #ttg.shared_memory, mutable> -> !ttg.memdesc<4x1x2048xi32, #shared1, #ttg.shared_memory, mutable>
  // CHECK: [[C0:%.*]] = llvm.mlir.constant(0 : i32)
  // CHECK: [[S0:%.*]] = llvm.mlir.undef
  // CHECK: [[S1:%.*]] = llvm.insertvalue [[PTR]], [[S0]][0]
  // CHECK: [[S2:%.*]] = llvm.insertvalue [[C0]], [[S1]][1]
  // CHECK: [[S3:%.*]] = llvm.insertvalue [[C0]], [[S2]][2]
  // CHECK: [[S4:%.*]] = llvm.insertvalue [[C0]], [[S3]][3]
  tt.return
}
```
**EN:** This block defines `memdesc_reinterpret` and exercises operations such as `tt.func`, `ttg.memdesc`, `llvm.extractvalue`, `llvm.mlir.constant`, `llvm.getelementptr`. Embedded check comments (CHECK×9, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `memdesc_reinterpret`，并覆盖 如 `tt.func`、`ttg.memdesc`、`llvm.extractvalue`、`llvm.mlir.constant`、`llvm.getelementptr` 这样的操作。 其中嵌入的检查注释（CHECK×9, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 2706
```mlir
}
```
**EN:** This block carries supporting MLIR syntax needed by the surrounding test case.
**CN:** 这一块包含周围测试用例所需的辅助 MLIR 语法。

### Line 2708
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2710-2721
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1, 1], threadsPerWarp = [1, 32], warpsPerCTA = [1, 4], order = [1, 0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32} {
  // CHECK-LABEL: load_br
  tt.func @load_br(%arg0: tensor<16x4x!tt.ptr<i8>, #blocked>) {
    // CHECK: llvm.br
    cf.br ^bb1(%arg0 : tensor<16x4x!tt.ptr<i8>, #blocked>)
    ^bb1(%arg1: tensor<16x4x!tt.ptr<i8>, #blocked>):
    // CHECK: ld.global.b8
      %0 = tt.load %arg1 : tensor<16x4x!tt.ptr<i8>, #blocked>
      tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `load_br`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `load_br`。

### Line 2723
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2726-2728
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-warps" = 2 : i32, ttg.target = "cuda:90"} {
// CHECK: llvm.mlir.global internal constant @tensor_constant_0([0 : i32, 1 : i32, {{.*}}, 15 : i32]) {addr_space = 0 : i32} : !llvm.array<16 x i32>
```
**EN:** This module sets kernel-level metadata targeting `cuda:90`, including `ttg.num-warps`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:90`，其中包括 `ttg.num-warps`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 2730-2740
```mlir
// CHECK-LABEL: @arith_constant_array
tt.func private @arith_constant_array() {
  %0 = arith.constant dense<[0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15]> : tensor<16xi32, #blocked>
  // CHECK: nvvm.read.ptx.sreg.tid.x
  // CHECK: [[ADDR:%.*]] = llvm.mlir.addressof @tensor_constant_0
  // CHECK: [[OFFSET:%.*]] = llvm.getelementptr [[ADDR]]
  // CHECK: [[VALUE:%.*]] = llvm.load [[OFFSET]]
  // CHECK: llvm.insertvalue [[VALUE]], %{{.*}}[0]
  tt.return
}
}
```
**EN:** This block defines `arith_constant_array` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×5, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `arith_constant_array`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×5, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 2742
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 2744
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 2746-2753
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:75", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: fp16_to_fp32
  tt.func public @fp16_to_fp32(%arg0 : tensor<256xf16, #blocked>) {
    // CHECK: llvm.fpext %{{.*}} : f16 to f32
    %0 = tt.fp_to_fp %arg0 : tensor<256xf16, #blocked> -> tensor<256xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:75`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `fp16_to_fp32`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:75`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `fp16_to_fp32`。

### Line 2755
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Line 2757
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [8], order = [0]}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#blocked`. They parameterize later tests with compact names for `#ttg.blocked`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#blocked`。它们为后续测试提供紧凑名称，用来表示 `#ttg.blocked`。

### Lines 2759-2768
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 8 : i32, ttg.target = "cuda:75", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: precise_math
  tt.func public @precise_math(%arg0 : tensor<256xf32, #blocked>, %arg1 : tensor<256xf32, #blocked>) {
    // CHECK: llvm.call_intrinsic "llvm.nvvm.div.rn.f"
    %0 = tt.precise_divf %arg0, %arg1 : tensor<256xf32, #blocked>
    // CHECK: llvm.call_intrinsic "llvm.nvvm.sqrt.rn.f"
    %1 = tt.precise_sqrt %arg0 : tensor<256xf32, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:75`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `precise_math`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:75`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `precise_math`。

### Line 2770
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2772-2774
```mlir
// We had a bug where DotOp lowering treated any input where shape[1] == 1 as an
// outer product and rejected it. This was incorrect in 3D tensors, since
// the dimension to look at would have been shape[2].
```
**EN:** This comment block provides context for the surrounding test logic: `// We had a bug where DotOp lowering treated any input where shape[1] == 1 as an`.
**CN:** 这一注释块为周围测试逻辑提供上下文：`// We had a bug where DotOp lowering treated any input where shape[1] == 1 as an`。

### Lines 2776-2778
```mlir
#mma = #ttg.nvidia_mma<{versionMajor = 2, versionMinor = 0, warpsPerCTA = [32, 1, 1], instrShape = [1, 16, 8]}>
#dot_operand_a = #ttg.dot_op<{opIdx = 0, parent = #mma, kWidth = 2}>
#dot_operand_b = #ttg.dot_op<{opIdx = 1, parent = #mma, kWidth = 2}>
```
**EN:** These lines declare reusable attribute/layout aliases such as `#mma`, `#dot_operand_a`, `#dot_operand_b`. They parameterize later tests with compact names for `#ttg.nvidia_mma`, `#ttg.dot_op`.
**CN:** 这些行声明了可复用的属性/布局别名，例如 `#mma`, `#dot_operand_a`, `#dot_operand_b`。它们为后续测试提供紧凑名称，用来表示 `#ttg.nvidia_mma`, `#ttg.dot_op`。

### Lines 2780-2793
```mlir
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 32 : i32, ttg.target = "cuda:100", "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: batched_dot_3d
  tt.func public @batched_dot_3d(
    %arg0: tensor<32x1x32xf16, #dot_operand_a>,
    %arg1: tensor<32x32x32xf16, #dot_operand_b>
  ) {
    %cst = arith.constant dense<0.000000e+00> : tensor<32x1x32xf32, #mma>
    // CHECK: llvm.inline_asm
    // CHECK-SAME: mma.sync.aligned.m16n8k16.row.col.f32.f16.f16.f32
    %result = tt.dot %arg0, %arg1, %cst, inputPrecision = tf32 :
      tensor<32x1x32xf16, #dot_operand_a> * tensor<32x32x32xf16, #dot_operand_b> -> tensor<32x1x32xf32, #mma>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata targeting `cuda:100`, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `batched_dot_3d`.
**CN:** 这个 module 设置了内核级元数据，目标后端为 `cuda:100`，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`, `ttg.target`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `batched_dot_3d`。

### Line 2795
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2797-2809
```mlir
#blocked = #ttg.blocked<{sizePerThread = [1], threadsPerWarp = [32], warpsPerCTA = [4], order = [0]}>
module attributes {"ttg.num-ctas" = 1 : i32, "ttg.num-warps" = 4 : i32, "ttg.threads-per-warp" = 32 : i32} {
  // CHECK-LABEL: @histogram_atomic
  // CHECK: llvm.icmp "ult"
  // CHECK: llvm.atomicrmw add
  // CHECK-NOT: ballot
  // CHECK-NOT: ctpop
  tt.func @histogram_atomic(%src: tensor<256xi32, #blocked>, %mask: tensor<256xi1, #blocked>, %out_ptr: tensor<8x!tt.ptr<i32>, #blocked>) {
    %hist = tt.histogram %src, %mask : tensor<256xi32, #blocked> -> tensor<8xi32, #blocked>
    tt.store %out_ptr, %hist : tensor<8x!tt.ptr<i32>, #blocked>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `histogram_atomic`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`, `ttg.threads-per-warp`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `histogram_atomic`。

### Line 2811
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 2813-2830
```mlir
module attributes {"ttg.num-ctas" = 4 : i32, "ttg.num-warps" = 4 : i32, ttg.profile_scratch_memory_alignment = 128 : i32, ttg.profile_scratch_memory_size = 2304 : i32} {
  // CHECK-LABEL: @profile_scratch_ptr_uses_i64
  // CHECK: %[[CLUSTER_Z:.*]] = nvvm.read.ptx.sreg.clusterid.z : i32
  // CHECK: %[[NCLUSTER_Y:.*]] = nvvm.read.ptx.sreg.nclusterid.y : i32
  // CHECK: %[[CLUSTER_Z_I64:.*]] = llvm.zext %[[CLUSTER_Z]] : i32 to i64
  // CHECK: %[[NCLUSTER_Y_I64:.*]] = llvm.zext %[[NCLUSTER_Y]] : i32 to i64
  // CHECK: %[[LINEAR_Z:.*]] = llvm.mul %[[CLUSTER_Z_I64]], %[[NCLUSTER_Y_I64]] : i64
  // CHECK: %[[NUM_CTAS:.*]] = llvm.mlir.constant(4 : i64) : i64
  // CHECK: %[[CTA_OFFSET:.*]] = llvm.mul %{{.*}}, %[[NUM_CTAS]] : i64
  // CHECK: %[[PROFILE_SIZE:.*]] = llvm.mlir.constant(2304 : i64) : i64
  // CHECK: %[[BYTE_OFFSET:.*]] = llvm.mul %{{.*}}, %[[PROFILE_SIZE]] : i64
  // CHECK: %[[TOTAL_OFFSET:.*]] = llvm.add %[[BYTE_OFFSET]], %{{.*}} : i64
  // CHECK: llvm.getelementptr %arg1[%[[TOTAL_OFFSET]]] : (!llvm.ptr<1>, i64) -> !llvm.ptr<1>, i8
  tt.func @profile_scratch_ptr_uses_i64() {
    %0 = ttg.global_scratch_alloc {alignment = 128 : i32, third_party_allocation, nbytes = 2304 : i32, ttg.global_scratch_memory_offset = 0 : i32} : !tt.ptr<i32>
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-ctas`, `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `profile_scratch_ptr_uses_i64`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-ctas`, `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `profile_scratch_ptr_uses_i64`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's conversion/lowering coverage and focuses on LLVM lowering.  
  **CN:** 该文件属于 Triton 的 转换/降级 测试覆盖，关注点是 LLVM 降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--allocate-shared-memory-nv`, `--convert-triton-gpu-to-llvm`, `-reconcile-unrealized-casts`.  
  **CN:** `RUN` 流水线会驱动 `--allocate-shared-memory-nv`, `--convert-triton-gpu-to-llvm`, `-reconcile-unrealized-casts` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `llvm` (LLVM dialect IR), `tt` (Triton core ops), `nvvm` (NVVM backend intrinsics), `arith` (scalar/tensor arithmetic), `cf` (control-flow ops).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`llvm`（LLVM 方言 IR）、`tt`（Triton 核心操作）、`nvvm`（NVVM 后端内建）、`arith`（标量/张量算术）、`cf`（控制流操作）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `llvm`, `tt`, `nvvm`, `arith`, `cf`, `math`, `ttng`.  
  **CN:** IR 方言依赖：`ttg`、`llvm`、`tt`、`nvvm`、`arith`、`cf`、`math`、`ttng`。
