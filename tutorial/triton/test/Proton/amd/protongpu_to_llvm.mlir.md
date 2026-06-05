# protongpu_to_llvm.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Proton/amd/protongpu_to_llvm.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises Proton dialect lowering in Triton's Proton pipeline coverage. **CN:** 该MLIR 测试用于覆盖 Triton Proton 流水线中的Proton 方言降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt %s -split-input-file -convert-proton-amd-gpu-to-llvm="gfx-arch=gfx942" --verify-diagnostics | FileCheck %s --check-prefix=CHECK`<br>`triton-opt %s -split-input-file -convert-proton-amd-gpu-to-llvm="gfx-arch=gfx942" --convert-builtin-func-to-llvm --verify-diagnostics | FileCheck -allow-unused-prefixes --check-prefix=CONVERT-BUILTIN %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×9, CHECK-DAG×38, CHECK-LABEL×7; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×9, CHECK-DAG×38, CHECK-LABEL×7；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that Proton dialect lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 Proton 方言降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt %s -split-input-file -convert-proton-amd-gpu-to-llvm="gfx-arch=gfx942" --verify-diagnostics | FileCheck %s --check-prefix=CHECK
// RUN: triton-opt %s -split-input-file -convert-proton-amd-gpu-to-llvm="gfx-arch=gfx942" --convert-builtin-func-to-llvm --verify-diagnostics | FileCheck -allow-unused-prefixes --check-prefix=CONVERT-BUILTIN %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt %s -split-input-file -convert-proton-amd-gpu-to-llvm="gfx-arch=gfx942" --verify-diagnostics | FileCheck %s --check-prefix=CHECK; triton-opt %s -split-input-file -convert-proton-amd-gpu-to-llvm="gfx-arch=gfx942" --convert-builtin-func-to-llvm --verify-diagnostics | FileCheck -allow-unused-prefixes --check-prefix=CONVERT-BUILTIN %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt %s -split-input-file -convert-proton-amd-gpu-to-llvm="gfx-arch=gfx942" --verify-diagnostics | FileCheck %s --check-prefix=CHECK; triton-opt %s -split-input-file -convert-proton-amd-gpu-to-llvm="gfx-arch=gfx942" --convert-builtin-func-to-llvm --verify-diagnostics | FileCheck -allow-unused-prefixes --check-prefix=CONVERT-BUILTIN %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-14
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: no_conversion
  llvm.func @no_conversion() {
    //CHECK: ttg.barrier local|global_read|global_write
    %0 = ttg.local_alloc : () -> !ttg.memdesc<256xi32, #shared, #smem, mutable>
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

### Lines 18-28
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: convert_read_counter
  llvm.func @convert_read_counter() -> i32 {
    //CHECK: llvm.call_intrinsic "llvm.amdgcn.s.memtime"() : () -> i64
    //CHECK: llvm.trunc %{{.*}} : i64 to i32
    %1 = proton_gpu.read_counter : i32
    llvm.return %1 : i32
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_read_counter`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_read_counter`。

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
    // CHECK-DAG: rocdl.workitem.id.x
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

### Lines 53-71
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: convert_circular_store_smem
  llvm.func @convert_circular_store_smem() {
    // CHECK-DAG: rocdl.workitem.id.x
    // CHECK-DAG: %[[WARPID:.*]] = llvm.udiv
    // CHECK-DAG: %[[P1:.*]] = llvm.icmp "eq" %[[WARPID]], %{{.*}}
    // CHECK-DAG: %[[ADDR1:.*]] = llvm.select %[[P1]]
    // CHECK-DAG: %[[P2:.*]] = llvm.icmp "eq" %[[WARPID]], %{{.*}}
    // CHECK-DAG: %[[ADDR2:.*]] = llvm.select %[[P2]], %{{.*}}, %[[ADDR1]]
  	// CHECK-DAG: %[[CYCLE1:.*]] = llvm.call_intrinsic "llvm.amdgcn.s.memtime"()
    %0 = ttg.local_alloc : () -> !ttg.memdesc<512xi32, #shared, #smem, mutable>
    %3 = proton_gpu.segment_alloc %0 : !ttg.memdesc<512xi32, #shared, #smem, mutable> -> !proton_gpu.segment<2048, #smem, warp, [0, 1]>
    %8 = proton_gpu.read_counter : i32
    proton_gpu.circular_store start %3, %8 {scopeId = 1 : i32} : !proton_gpu.segment<2048, #smem, warp, [0, 1]>, i32
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_circular_store_smem`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_circular_store_smem`。

### Line 73
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 74-79
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32, ttg.profile_scratch_memory_alignment = 128 : i32, ttg.profile_scratch_memory_size = 384 : i32} {
  // CHECK-LABEL: convert_smem_initialize
  // CHECK: llvm.cond_br %{{.*}}, ^bb1, ^bb2
  // CHECK: ^bb1:
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。

### Lines 81-84
```mlir
  // CHECK-DAG: %[[PREAMBLE:.*]] = llvm.mlir.constant(-559038737 : i32)
  // CHECK-DAG: %[[PREAMBLE_OFFSET:.*]] = llvm.mlir.constant(0 : i32) : i32
  // CHECK-DAG: %[[PREAMBLE_PTR:.*]] = llvm.getelementptr %{{.*}}[%[[PREAMBLE_OFFSET]]] : (!llvm.ptr<1>, i32) -> !llvm.ptr<1>, i32
  // CHECK-DAG: llvm.store %[[PREAMBLE]], %{{.*}} : i32, !llvm.ptr<1>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-DAG`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-DAG`），从而保证后端/代码生成结果稳定。

### Lines 86-88
```mlir
  // CHECK-DAG: %[[PID_OFFSET:.*]] = llvm.mlir.constant(1 : i32) : i32
  // CHECK-DAG: %[[PID_PTR:.*]] = llvm.getelementptr %{{.*}}[%[[PID_OFFSET]]] : (!llvm.ptr<1>, i32) -> !llvm.ptr<1>
  // CHECK-DAG: llvm.store %{{.*}}, %[[PID_PTR]] : i32, !llvm.ptr<1>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-DAG`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-DAG`），从而保证后端/代码生成结果稳定。

### Lines 90-95
```mlir
  // CHECK-DAG: llvm.inline_asm asm_dialect = att operand_attrs = [] "s_getreg_b32 $0, hwreg(HW_REG_XCC_ID, 0, 4)", "=s"  : () -> i32
  // CHECK-DAG: llvm.inline_asm asm_dialect = att operand_attrs = [] "s_getreg_b32 $0, hwreg(HW_REG_HW_ID, 8, 4)", "=s"  : () -> i32
  // CHECK-DAG: llvm.inline_asm asm_dialect = att operand_attrs = [] "s_getreg_b32 $0, hwreg(HW_REG_HW_ID, 13, 3)", "=s"  : () -> i32
  // CHECK-DAG: %[[SMID_OFFSET:.*]] = llvm.mlir.constant(2 : i32) : i32
  // CHECK-DAG: %[[SMID_PTR:.*]] = llvm.getelementptr %{{.*}}[%[[SMID_OFFSET]]] : (!llvm.ptr<1>, i32) -> !llvm.ptr<1>
  // CHECK-DAG: llvm.store %{{.*}}, %[[SMID_PTR]] : i32, !llvm.ptr<1>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-DAG`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-DAG`），从而保证后端/代码生成结果稳定。

### Lines 97-102
```mlir
  // CHECK-DAG: %[[INIT_TIME_RAW:.*]] = llvm.call_intrinsic "llvm.amdgcn.s.memrealtime"() : () -> i64
  // CHECK-DAG: %[[TEN:.*]] = llvm.mlir.constant(10 : i64) : i64
  // CHECK-DAG: %[[INIT_TIME:.*]] = llvm.mul %[[INIT_TIME_RAW]], %[[TEN]] : i64
  // CHECK-DAG: %[[INIT_TIME_OFFSET:.*]] = llvm.mlir.constant(4 : i32) : i32
  // CHECK-DAG: %[[INIT_TIME_PTR:.*]] = llvm.getelementptr %{{.*}}[%[[INIT_TIME_OFFSET]]] : (!llvm.ptr<1>, i32) -> !llvm.ptr<1>
  // CHECK-DAG: llvm.store %[[INIT_TIME]], %[[INIT_TIME_PTR]] : i64, !llvm.ptr<1>
```
**EN:** These comments are FileCheck expectations. They constrain the lowered output to contain the expected patterns (`CHECK-DAG`), so the backend/codegen result stays stable.
**CN:** 这些注释是 FileCheck 期望。它们要求降级结果包含预期模式（`CHECK-DAG`），从而保证后端/代码生成结果稳定。

### Lines 104-111
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

### Line 113
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 115-133
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32, ttg.profile_scratch_memory_alignment = 128 : i32, ttg.profile_scratch_memory_size = 384 : i32} {
  // CHECK-LABEL: convert_smem_finalize
  // CONVERT-BUILTIN: llvm.call_intrinsic "llvm.amdgcn.s.memrealtime"() : () -> i64
  // CONVERT-BUILTIN: llvm.store %{{.*}}, %{{.*}} : i64, !llvm.ptr<1>
  // CONVERT-BUILTIN: llvm.cond_br %{{.*}}, ^bb{{.*}}, ^bb{{.*}}
  // CONVERT-BUILTIN: llvm.call_intrinsic "llvm.amdgcn.s.memrealtime"() : () -> i64
  // CONVERT-BUILTIN: llvm.store %{{.*}}, %{{.*}} : i64, !llvm.ptr<1>
  // CONVERT-BUILTIN: llvm.br ^bb{{.*}}
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

### Line 135
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 137-152
```mlir
#shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
#smem = #ttg.shared_memory
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: use_clock64
  llvm.func @use_clock64() {
    // CHECK-DAG: %[[CYCLE:.*]] = llvm.call_intrinsic "llvm.amdgcn.s.memtime"()
    // CHECK-DAG: %[[CYCLE64:.*]] = llvm.bitcast %[[CYCLE]] : i64 to vector<2xi32>
    // CHECK-DAG: llvm.extractelement %[[CYCLE64]]
    // CHECK-DAG: llvm.extractelement %[[CYCLE64]]
    %0 = ttg.local_alloc : () -> !ttg.memdesc<512xi32, #shared, #smem, mutable>
    %3 = proton_gpu.segment_alloc %0 : !ttg.memdesc<512xi32, #shared, #smem, mutable> -> !proton_gpu.segment<2048, #smem, warp, [0, 1]>
    %8 = proton_gpu.read_counter : i64
    proton_gpu.circular_store start %3, %8 {scopeId = 1 : i32} : !proton_gpu.segment<2048, #smem, warp, [0, 1]>, i64
    llvm.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `use_clock64`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `use_clock64`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's Proton pipeline coverage and focuses on Proton dialect lowering.  
  **CN:** 该文件属于 Triton 的 Proton 流水线 测试覆盖，关注点是 Proton 方言降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `-convert-proton-amd-gpu-to-llvm="gfx-arch=gfx942"`, `--verify-diagnostics`, `--check-prefix=CHECK`, `--convert-builtin-func-to-llvm`, `-allow-unused-prefixes`, `--check-prefix=CONVERT-BUILTIN`.  
  **CN:** `RUN` 流水线会驱动 `-convert-proton-amd-gpu-to-llvm="gfx-arch=gfx942"`, `--verify-diagnostics`, `--check-prefix=CHECK`, `--convert-builtin-func-to-llvm`, `-allow-unused-prefixes`, `--check-prefix=CONVERT-BUILTIN` 等工具/pass。
- **EN:** The IR mixes `ttg` (TritonGPU ops/layouts), `llvm` (LLVM dialect IR), `tt` (Triton core ops), `nvvm` (NVVM backend intrinsics).  
  **CN:** 该 IR 混合使用了 `ttg`（TritonGPU 操作/布局）、`llvm`（LLVM 方言 IR）、`tt`（Triton 核心操作）、`nvvm`（NVVM 后端内建）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `ttg`, `llvm`, `tt`, `nvvm`.  
  **CN:** IR 方言依赖：`ttg`、`llvm`、`tt`、`nvvm`。
