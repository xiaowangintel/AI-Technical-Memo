# protongpu_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Proton/nvidia/protongpu_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises Proton dialect lowering in Triton's Proton pipeline coverage. **CN:** 该MLIR 测试用于覆盖 Triton Proton 流水线中的Proton 方言降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file -convert-proton-nvidia-gpu-to-llvm -cse --verify-diagnostics | FileCheck %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×38, CHECK-DAG×50, CHECK-LABEL×8; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×38, CHECK-DAG×50, CHECK-LABEL×8；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that Proton dialect lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 Proton 方言降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Line 1
```mlir
// RUN: triton-opt %s -split-input-file -convert-proton-nvidia-gpu-to-llvm -cse --verify-diagnostics | FileCheck %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file -convert-proton-nvidia-gpu-to-llvm -cse --verify-diagnostics | FileCheck %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file -convert-proton-nvidia-gpu-to-llvm -cse --verify-diagnostics | FileCheck %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 3-13
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: no_conversion
  llvm.func @no_conversion() {
    // CHECK: ttg.barrier local|global_read|global_write
    %0 = ttg.local_alloc  : () -> !ttg.memdesc<256xi32, #shared, #smem, mutable>
    ttg.barrier local|global_read|global_write
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `no_conversion`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `no_conversion`。

### Line 16
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 18-27
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: convert_read_counter
  llvm.func @convert_read_counter() {
    // CHECK: llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "mov.u32 $0, %clock;", "=r"  : () -> i32
    %1 = proton_gpu.read_counter : i32
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `, `. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_read_counter`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `, `。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_read_counter`。

### Line 30
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 32-49
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: convert_smem_segment_setup
   tt.func @convert_smem_segment_setup() -> !proton_gpu.segment<384, #smem, warp, [0, 1, 2]> {
    // CHECK-DAG: nvvm.read.ptx.sreg.tid.x
    // CHECK-DAG: %[[WARPID:.*]] = llvm.udiv
    // CHECK-DAG: %[[P1:.*]] = llvm.icmp "eq" %[[WARPID]], %{{.*}}
    // CHECK-DAG: %[[ADDR1:.*]] = llvm.select %[[P1]]
    // CHECK-DAG: %[[P2:.*]] = llvm.icmp "eq" %[[WARPID]], %{{.*}}
    // CHECK-DAG: %[[ADDR2:.*]] = llvm.select %[[P2]], %{{.*}}, %[[ADDR1]]
    // CHECK-DAG: %[[P3:.*]] = llvm.icmp "eq" %[[WARPID]], %{{.*}}
    // CHECK-DAG: %[[ADDR3:.*]] = llvm.select %[[P3]], %{{.*}}, %[[ADDR2]]
    %0 = ttg.local_alloc : () -> !ttg.memdesc<96xi32, #shared, #smem, mutable>
    %3 = proton_gpu.segment_alloc %0 : !ttg.memdesc<96xi32, #shared, #smem, mutable> -> !proton_gpu.segment<384, #smem, warp, [0, 1, 2]>
    tt.return %3 : !proton_gpu.segment<384, #smem, warp, [0, 1, 2]>
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_smem_segment_setup`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_smem_segment_setup`。

### Line 51
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 53-85
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: convert_circular_smem_store_nested
  llvm.func @convert_circular_smem_store_nested() {
    // CHECK-DAG: nvvm.read.ptx.sreg.tid.x
    // CHECK-DAG: %[[WARPID:.*]] = llvm.udiv
    // CHECK-DAG: %[[P1:.*]] = llvm.icmp "eq" %[[WARPID]], %{{.*}}
    // CHECK-DAG: %[[ADDR1:.*]] = llvm.select %[[P1]]
    // CHECK-DAG: %[[P2:.*]] = llvm.icmp "eq" %[[WARPID]], %{{.*}}
    // CHECK-DAG: %[[ADDR2:.*]] = llvm.select %[[P2]], %{{.*}}, %[[ADDR1]]
    // CHECK-DAG: scf.for
    // CHECK-DAG: scf.for
    // CHECK-DAG: %[[CYCLE1:.*]] = llvm.inline_asm has_side_effects{{.*}}%clock
    // CHECK-DAG: %[[INDEX:.*]] = llvm.urem
    // CHECK-DAG: %[[SMEM_OFFSET:.*]] = llvm.add {{.*}}, %[[INDEX]]
    // CHECK-DAG: %[[SMEM_PTR:.*]] = llvm.getelementptr %{{.*}}[%[[SMEM_OFFSET]]] : (!llvm.ptr<3>, i32) -> !llvm.ptr<3>, i32
    // CHECK-DAG: llvm.inline_asm has_side_effects{{.*}}st.shared::cta.v2.b32{{.*}}%[[SMEM_PTR]], %{{.*}}, %{{.*}}, %{{.*}}
    // CHECK-DAG: llvm.extractvalue {{.*}}[0] : !llvm.struct<(ptr<3>, i32)>
    %c4 = arith.constant 4 : index
    %c1 = arith.constant 1 : index
    %c0 = arith.constant 0 : index
    %0 = ttg.local_alloc : () -> !ttg.memdesc<512xi32, #shared, #smem, mutable>
    %3 = proton_gpu.segment_alloc %0 : !ttg.memdesc<512xi32, #shared, #smem, mutable> -> !proton_gpu.segment<2048, #smem, warp, [0, 1]>
    scf.for %arg0 = %c0 to %c4 step %c1 {
      scf.for %arg1 = %c0 to %c4 step %c1 {
        %8 = proton_gpu.read_counter : i32
        proton_gpu.circular_store start %3, %8 {scopeId = 1 : i32} : !proton_gpu.segment<2048, #smem, warp, [0, 1]>, i32
      }
    }
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_circular_smem_store_nested`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_circular_smem_store_nested`。

### Line 87
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 89-111
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: convert_circular_smem_store_flat
  llvm.func @convert_circular_smem_store_flat() {
    // CHECK-DAG: nvvm.read.ptx.sreg.tid.x
    // CHECK-DAG: %[[WARPID:.*]] = llvm.udiv
    // CHECK-DAG: %[[P1:.*]] = llvm.icmp "eq" %[[WARPID]], %{{.*}}
    // CHECK-DAG: %[[ADDR1:.*]] = llvm.select %[[P1]]
    // CHECK-DAG: %[[P2:.*]] = llvm.icmp "eq" %[[WARPID]], %{{.*}}
    // CHECK-DAG: %[[ADDR2:.*]] = llvm.select %[[P2]], %{{.*}}, %[[ADDR1]]
    // CHECK-DAG: %[[CYCLE1:.*]] = llvm.inline_asm has_side_effects{{.*}}%clock
    // CHECK-DAG: %[[INDEX:.*]] = llvm.urem
    // CHECK-DAG: %[[SMEM_OFFSET:.*]] = llvm.add %{{.*}} %[[INDEX]]
    // CHECK-DAG: %[[SMEM_PTR:.*]] = llvm.getelementptr %{{.*}}[%[[SMEM_OFFSET]]] : (!llvm.ptr<3>, i32) -> !llvm.ptr<3>, i32
    // CHECK-DAG: llvm.inline_asm has_side_effects{{.*}}st.shared::cta.v2.b32{{.*}}%[[SMEM_PTR]], %{{.*}}, %{{.*}}, %{{.*}}
    %0 = ttg.local_alloc : () -> !ttg.memdesc<512xi32, #shared, #smem, mutable>
    %3 = proton_gpu.segment_alloc %0 : !ttg.memdesc<512xi32, #shared, #smem, mutable> -> !proton_gpu.segment<2048, #smem, warp, [0, 1]>
    %8 = proton_gpu.read_counter : i32
    proton_gpu.circular_store start %3, %8 {scopeId = 1 : i32} : !proton_gpu.segment<2048, #smem, warp, [0, 1]>, i32
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_circular_smem_store_flat`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_circular_smem_store_flat`。

### Line 113
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 115-120
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32, ttg.profile_scratch_memory_alignment = 128 : i32, ttg.profile_scratch_memory_size = 384 : i32} {
  // CHECK-LABEL: convert_smem_initialize
  // CHECK-DAG: llvm.cond_br %{{.*}}, ^bb1, ^bb2
  // CHECK-DAG: ^bb1:
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 122-125
```mlir
  // CHECK-DAG: %[[PREAMBLE:.*]] = llvm.mlir.constant(-559038737 : i32)
  // CHECK-DAG: %[[PREAMBLE_OFFSET:.*]] = llvm.mlir.constant(0 : i32) : i32
  // CHECK-DAG: %[[PREAMBLE_PTR:.*]] = llvm.getelementptr %{{.*}}[%[[PREAMBLE_OFFSET]]] : (!llvm.ptr<1>, i32) -> !llvm.ptr<1>, i32
  // CHECK-DAG: llvm.store %[[PREAMBLE]], %[[PREAMBLE_PTR]] : i32, !llvm.ptr<1>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-DAG`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-DAG`），从而保证后端/代码生成结果稳定。

### Lines 127-129
```mlir
  // CHECK-DAG: %[[PID_OFFSET:.*]] = llvm.mlir.constant(1 : i32) : i32
  // CHECK-DAG: %[[PID_PTR:.*]] = llvm.getelementptr %{{.*}}[%[[PID_OFFSET]]] : (!llvm.ptr<1>, i32) -> !llvm.ptr<1>
  // CHECK-DAG: llvm.store %{{.*}}, %[[PID_PTR]] : i32, !llvm.ptr<1>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-DAG`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-DAG`），从而保证后端/代码生成结果稳定。

### Lines 131-134
```mlir
  // CHECK-DAG: %[[SMID:.*]] = nvvm.read.ptx.sreg.smid
  // CHECK-DAG: %[[SMID_OFFSET:.*]] = llvm.mlir.constant(2 : i32) : i32
  // CHECK-DAG: %[[SMID_PTR:.*]] = llvm.getelementptr %{{.*}}[%[[SMID_OFFSET]]] : (!llvm.ptr<1>, i32) -> !llvm.ptr<1>
  // CHECK-DAG: llvm.store %[[SMID]], %[[SMID_PTR]] : i32, !llvm.ptr<1>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-DAG`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-DAG`），从而保证后端/代码生成结果稳定。

### Lines 136-139
```mlir
  // CHECK-DAG: %[[INIT_TIME:.*]] = llvm.call_intrinsic "llvm.nvvm.read.ptx.sreg.globaltimer"() : () -> i64
  // CHECK-DAG: %[[INIT_TIME_OFFSET:.*]] = llvm.mlir.constant(4 : i32) : i32
  // CHECK-DAG: %[[INIT_TIME_PTR:.*]] = llvm.getelementptr %{{.*}}[%[[INIT_TIME_OFFSET]]] : (!llvm.ptr<1>, i32) -> !llvm.ptr<1>
  // CHECK-DAG: llvm.store %[[INIT_TIME]], %[[INIT_TIME_PTR]] : i64, !llvm.ptr<1>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-DAG`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-DAG`），从而保证后端/代码生成结果稳定。

### Lines 141-148
```mlir
  // CHECK: ^bb2:
  // CHECK: llvm.return
  llvm.func @convert_smem_initialize(%arg: !llvm.ptr<1>) attributes {noinline = false, nvvm.kernel = 1 : ui1} {
    %0 = ttg.global_scratch_alloc {alignment = 128 : i32, third_party_allocation, nbytes = 384 : i32, ttg.global_scratch_memory_offset = 0 : i32} : !tt.ptr<i32>
    proton_gpu.initialize %0 : !tt.ptr<i32>
    llvm.return
  }
}
```
**EN:** This block defines `convert_smem_initialize` and exercises NVIDIA-specific codegen. Embedded check comments (CHECK×2) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `convert_smem_initialize`，并覆盖 NVIDIA 专用代码生成。 其中嵌入的检查注释（CHECK×2）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 150
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 152-194
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32, ttg.profile_scratch_memory_alignment = 128 : i32, ttg.profile_scratch_memory_size = 384 : i32} {
  // CHECK-LABEL: convert_smem_finalize
  // CHECK: llvm.extractvalue %{{.*}}[0] : !llvm.struct<(ptr<3>, i32)>
  // CHECK: llvm.store
  // CHECK: llvm.cond_br %{{.*}}, ^bb1, ^bb2
  // CHECK: ^bb1: // pred: ^bb0
  // CHECK: llvm.store %{{.*}}, %{{.*}} : i32, !llvm.ptr<1>
  // CHECK: llvm.call_intrinsic "llvm.nvvm.read.ptx.sreg.globaltimer"() : () -> i64
  // CHECK: llvm.store %{{.*}}, %{{.*}} : i64, !llvm.ptr<1>
  // CHECK: llvm.br ^bb2
  // CHECK: ^bb2: // 2 preds: ^bb0, ^bb1
  // CHECK: llvm.cond_br %{{.*}}, ^bb3, ^bb4
  // CHECK: ^bb3: // pred: ^bb2
  // CHECK: llvm.store %{{.*}}, %{{.*}} : i32, !llvm.ptr<1>
  // CHECK: llvm.br ^bb4
  // CHECK: ^bb4: // 2 preds: ^bb2, ^bb3
  // CHECK: llvm.cond_br %{{.*}}, ^[[LOOP_HEAD:bb[0-9]+]](%{{.*}} : i32), ^[[EXIT:bb[0-9]+]]
  // CHECK: ^[[LOOP_HEAD]](%{{.*}}: i32):
  // CHECK: llvm.cond_br %{{.*}}, ^[[LOOP_BODY:bb[0-9]+]](%{{.*}} : i32), ^[[EXIT]]
  // CHECK: ^[[LOOP_BODY]](%{{.*}}: i32):
  // CHECK: llvm.getelementptr
  // CHECK: llvm.store
  // CHECK: llvm.store
  // CHECK: ^[[EXIT]]:
  // CHECK: llvm.cond_br %{{.*}}, ^[[POST:bb[0-9]+]], ^[[RET:bb[0-9]+]]
  // CHECK: ^[[POST]]:
  // CHECK: %{{.*}} = llvm.mlir.constant(8 : i32) : i32
  // CHECK: %[[POST_FINAL_TIME_PTR:.*]] = llvm.getelementptr %{{.*}}{{\[}}%{{.*}}{{\]}} : (!llvm.ptr<1>, i32) -> !llvm.ptr<1>, i32
  // CHECK: %[[POST_FINAL_TIME:.*]] = llvm.call_intrinsic "llvm.nvvm.read.ptx.sreg.globaltimer"() : () -> i64
  // CHECK: llvm.store %[[POST_FINAL_TIME]], %[[POST_FINAL_TIME_PTR]] : i64, !llvm.ptr<1>
  // CHECK: llvm.br ^[[RET]]
  // CHECK: ^[[RET]]:
  // CHECK: llvm.return
  llvm.func @convert_smem_finalize(%arg: !llvm.ptr<1>) attributes {noinline = false, nvvm.kernel = 1 : ui1} {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<512xi32, #shared, #smem, mutable>
    %1 = ttg.global_scratch_alloc {alignment = 128 : i32, third_party_allocation, nbytes = 384 : i32, ttg.global_scratch_memory_offset = 0 : i32} : !tt.ptr<i32>
    %2 = proton_gpu.segment_alloc %0 : !ttg.memdesc<512xi32, #shared, #smem, mutable> -> !proton_gpu.segment<2048, #smem, warp>
    proton_gpu.finalize %2, %1 : !proton_gpu.segment<2048, #smem, warp>, !tt.ptr<i32>
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_smem_finalize`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_smem_finalize`。

### Line 196
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 198-212
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: use_clock64
  // CHECK: llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "mov.u32 $0, %clock;", "=r"  : () -> i32
  // CHECK: llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "mov.u32 $0, %clock_hi;", "=r"  : () -> i32
  // CHECK: llvm.inline_asm has_side_effects asm_dialect = att operand_attrs = [] "@$3 st.shared::cta.v2.b32{{.*}}(!llvm.ptr<3>, i32, i32, i1)
  llvm.func @use_clock64() {
    %0 = ttg.local_alloc : () -> !ttg.memdesc<512xi32, #shared, #smem, mutable>
    %3 = proton_gpu.segment_alloc %0 : !ttg.memdesc<512xi32, #shared, #smem, mutable> -> !proton_gpu.segment<2048, #smem, warp, [0, 1]>
    %8 = proton_gpu.read_counter : i64
    proton_gpu.circular_store start %3, %8 {scopeId = 1 : i32} : !proton_gpu.segment<2048, #smem, warp, [0, 1]>, i64
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `, `. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `use_clock64`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `, `。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `use_clock64`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's Proton pipeline coverage and focuses on Proton dialect lowering.  
  **CN:** 该文件属于 Triton 的 Proton 流水线 测试覆盖，关注点是 Proton 方言降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `-convert-proton-nvidia-gpu-to-llvm`, `-cse`, `--verify-diagnostics`.  
  **CN:** `RUN` 流水线会驱动 `-convert-proton-nvidia-gpu-to-llvm`, `-cse`, `--verify-diagnostics` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `llvm` (LLVM dialect IR), `tt` (Triton core ops), `nvvm` (NVVM backend intrinsics), `scf` (structured control flow), `arith` (scalar/tensor arithmetic).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`llvm`（LLVM 方言 IR）、`tt`（Triton 核心操作）、`nvvm`（NVVM 后端内建）、`scf`（结构化控制流）、`arith`（标量/张量算术）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `llvm`, `tt`, `nvvm`, `scf`, `arith`.  
  **CN:** IR 方言依赖：`ttg`、`llvm`、`tt`、`nvvm`、`scf`、`arith`。
