# proton_to_protongpu.mlir — Code Analysis / 代码分析

## Source / 来源

- **Path / 路径:** `test/Proton/proton_to_protongpu.mlir`
- **Purpose / 目的:** **EN:** This mlir test exercises Proton dialect lowering in Triton's Proton pipeline coverage. **CN:** 该MLIR 测试用于覆盖 Triton Proton 流水线中的Proton 方言降级相关场景。
- **Pass pipeline / Pass 流水线:** **EN:** `triton-opt --split-input-file -convert-proton-to-protongpu="max-shared-mem-size=32768" -canonicalize -cse %s | FileCheck %s`<br>`triton-opt --split-input-file -convert-proton-to-protongpu="buffer-type=global buffer-size=1024" -canonicalize -cse %s | FileCheck --check-prefix=CHECK-GMEM %s` **CN:** 以上命令定义了该 MLIR 测试实际运行的 pass/工具流水线。
- **Checks / 检查方式:** **EN:** FileCheck markers: CHECK×82, CHECK-GMEM×12, CHECK-LABEL×5; diagnostics annotations: none. **CN:** FileCheck 标记为 CHECK×82, CHECK-GMEM×12, CHECK-LABEL×5；诊断标注为 none。
- **Validation / 验证内容:** **EN:** The file validates that Proton dialect lowering produces the expected printed IR, LLVM/NVVM/AMDGPU fragments, or pass-specific structure matched by FileCheck. **CN:** 该文件验证 Proton 方言降级 会生成由 FileCheck 匹配到的预期打印 IR、LLVM/NVVM/AMDGPU 片段或 pass 专属结构。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```mlir
// RUN: triton-opt --split-input-file -convert-proton-to-protongpu="max-shared-mem-size=32768" -canonicalize -cse %s | FileCheck %s
// RUN: triton-opt --split-input-file -convert-proton-to-protongpu="buffer-type=global buffer-size=1024" -canonicalize -cse %s | FileCheck --check-prefix=CHECK-GMEM %s
```
**EN:** This block defines lit execution commands. The pipeline runs `triton-opt --split-input-file -convert-proton-to-protongpu="max-shared-mem-size=32768" -canonicalize -cse %s | FileCheck %s; triton-opt --split-input-file -convert-proton-to-protongpu="buffer-type=global buffer-size=1024" -canonicalize -cse %s | FileCheck --check-prefix=CHECK-GMEM %s` so the test is executed with the intended pass sequence and verification mode.
**CN:** 这一块定义了 lit 的执行命令。流水线会运行 `triton-opt --split-input-file -convert-proton-to-protongpu="max-shared-mem-size=32768" -canonicalize -cse %s | FileCheck %s; triton-opt --split-input-file -convert-proton-to-protongpu="buffer-type=global buffer-size=1024" -canonicalize -cse %s | FileCheck --check-prefix=CHECK-GMEM %s`，从而以预期的 pass 顺序和校验模式执行测试。

### Lines 4-10
```mlir
module {
  // CHECK-LABEL: no_record
  tt.func @no_record() {
    // CHECK: tt.return
    tt.return
  }
}
```
**EN:** This block defines `no_record` and exercises operations such as `tt.func`, `tt.return`. Embedded check comments (CHECK×1, CHECK-LABEL×1) describe what the lowering, verifier, or printed IR should look like.
**CN:** 这一块定义了 `no_record`，并覆盖 如 `tt.func`、`tt.return` 这样的操作。 其中嵌入的检查注释（CHECK×1, CHECK-LABEL×1）说明了降级结果、验证器输出或打印 IR 应该呈现的样子。

### Line 12
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 14-32
```mlir
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: simple_record
  // CHECK: %[[SCRATCH:.*]] = ttg.global_scratch_alloc {alignment = 128 : i32, nbytes = 1152 : i32, third_party_allocation} : !tt.ptr<i32>
  // CHECK: proton_gpu.initialize %[[SCRATCH]] : !tt.ptr<i32>
  // CHECK: %[[BUF:.*]] = ttg.local_alloc  : () -> !ttg.memdesc<256xi32, #shared, #smem, mutable>
  // CHECK: %[[SEGMENT:.*]] = proton_gpu.segment_alloc %[[BUF]]
  // CHECK: %[[START:.*]] = proton_gpu.read_counter : i32
  // CHECK: proton_gpu.circular_store start %[[SEGMENT]], %[[START]] {scopeId = 0 : i32} : !proton_gpu.segment<1024, #smem, warp>, i32
  // CHECK: %[[END:.*]] = proton_gpu.read_counter : i32
  // CHECK: proton_gpu.circular_store end %[[SEGMENT]], %[[END]] {scopeId = 0 : i32} : !proton_gpu.segment<1024, #smem, warp>, i32
  // CHECK: ttg.barrier local|global_read|global_write
  // CHECK: proton_gpu.finalize %[[SEGMENT]], %[[SCRATCH]] : !proton_gpu.segment<1024, #smem, warp>, !tt.ptr<i32>
  // CHECK: tt.return
  tt.func @simple_record() {
    proton.record start "name0"
    proton.record end "name0"
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `simple_record`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `simple_record`。

### Line 34
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 36-66
```mlir
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: scf_record
  tt.func @scf_record() {
    %i = arith.constant 0 : index
    %c1 = arith.constant 1 : index
    %c4 = arith.constant 4 : index
    // CHECK: %[[SCRATCH:.*]] = ttg.global_scratch_alloc
    // CHECK: proton_gpu.initialize %[[SCRATCH]] : !tt.ptr<i32>
    // CHECK: %[[BUF:.*]] = ttg.local_alloc
    // CHECK: %[[SEGMENT:.*]] = proton_gpu.segment_alloc %[[BUF]]
    // CHECK: %[[START0:.*]] = proton_gpu.read_counter : i32
    // CHECK: proton_gpu.circular_store start %[[SEGMENT]], %[[START0]] {scopeId = 0 : i32}
    // CHECK: scf.for
    // CHECK: %[[START1:.*]] = proton_gpu.read_counter : i32
    // CHECK: proton_gpu.circular_store start %[[SEGMENT]], %[[START1]] {scopeId = 1 : i32}
    // CHECK: %[[END1:.*]] = proton_gpu.read_counter : i32
    // CHECK: proton_gpu.circular_store end %[[SEGMENT]], %[[END1]] {scopeId = 1 : i32}
    // CHECK: }
    // CHECK: %[[END0:.*]] = proton_gpu.read_counter : i32
    // CHECK: proton_gpu.circular_store end %[[SEGMENT]], %[[END0]] {scopeId = 0 : i32}
    // CHECK: ttg.barrier local|global_read|global_write
    // CHECK: proton_gpu.finalize %[[SEGMENT]], %[[SCRATCH]]
    proton.record start "name1"
    scf.for %arg0 = %i to %c4 step %c1 {
      proton.record start "name0"
      proton.record end "name0"
    }
    proton.record end "name1"
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `scf_record`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `scf_record`。

### Line 68
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 70-108
```mlir
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: nested_record
  tt.func @nested_record() {
    %i = arith.constant 0 : index
    %c1 = arith.constant 1 : index
    %c4 = arith.constant 4 : index
    // CHECK: %[[SCRATCH:.*]] = ttg.global_scratch_alloc
    // CHECK: proton_gpu.initialize %[[SCRATCH]] : !tt.ptr<i32>
    // CHECK: %[[BUF:.*]] = ttg.local_alloc
    // CHECK: %[[SEGMENT:.*]] = proton_gpu.segment_alloc %[[BUF]]
    // CHECK: %[[START0:.*]] = proton_gpu.read_counter : i32
    // CHECK: proton_gpu.circular_store start %[[SEGMENT]], %[[START0]] {scopeId = 0 : i32}
    // CHECK: scf.for
    // CHECK: %[[START1:.*]] = proton_gpu.read_counter : i32
    // CHECK: proton_gpu.circular_store start %[[SEGMENT]], %[[START1]] {scopeId = 1 : i32}
    // CHECK: %[[END1:.*]] = proton_gpu.read_counter : i32
    // CHECK: proton_gpu.circular_store end %[[SEGMENT]], %[[END1]] {scopeId = 1 : i32}
    // CHECK: }
    // CHECK: %[[END0:.*]] = proton_gpu.read_counter : i32
    // CHECK: proton_gpu.circular_store end %[[SEGMENT]], %[[END0]] {scopeId = 0 : i32}
    // CHECK: %[[START2:.*]] = proton_gpu.read_counter : i32
    // CHECK: proton_gpu.circular_store start %[[SEGMENT]], %[[START2]] {scopeId = 2 : i32}
    // CHECK: %[[END2:.*]] = proton_gpu.read_counter : i32
    // CHECK: proton_gpu.circular_store end %[[SEGMENT]], %[[END2]] {scopeId = 2 : i32}
    // CHECK: ttg.barrier local|global_read|global_write
    // CHECK: proton_gpu.finalize %[[SEGMENT]], %[[SCRATCH]]
    proton.record start "name1"
    scf.for %arg0 = %i to %c4 step %c1 {
      proton.record start "name0"
      scf.for %arg1 = %i to %c4 step %c1 {
      }
      proton.record end "name0"
    }
    proton.record end "name1"
    proton.record start "name2"
    proton.record end "name2"
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `nested_record`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `nested_record`。

### Line 110
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 112-163
```mlir
// CHECK: #shared = #ttg.swizzled_shared<{vec = 1, perPhase = 1, maxPhase = 1, order = [0]}>
// CHECK: #smem = #ttg.shared_memory
// CHECK: module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 8 : i32} {
// CHECK:   tt.func @convert_warp_specialize() {
// CHECK:     %[[SCRATCH:.*]] = ttg.global_scratch_alloc {alignment = 128 : i32, nbytes = 1152 : i32, third_party_allocation} : !tt.ptr<i32>
// CHECK:     %[[MEMDESC:.*]] = ttg.local_alloc : () -> !ttg.memdesc<256xi32, #shared, #smem, mutable>
// CHECK:     %[[SEGMENT:.*]] = proton_gpu.segment_alloc %[[MEMDESC]] : !ttg.memdesc<256xi32, #shared, #smem, mutable> -> <1024, #smem, warp>
// CHECK:     proton_gpu.init_ctx %[[SCRATCH]] : !tt.ptr<i32>
// CHECK:     %[[COUNTER1:.*]] = proton_gpu.read_counter : i32
// CHECK:     proton_gpu.circular_store start %[[SEGMENT]], %[[COUNTER1]] {scopeId = 0 : i32} : !proton_gpu.segment<1024, #smem, warp>, i32
// CHECK:     ttg.warp_specialize(%[[MEMDESC]], %[[SCRATCH]])
// CHECK:     default {
// CHECK:       %[[COUNTER2:.*]] = proton_gpu.read_counter : i32
// CHECK:       proton_gpu.circular_store start %[[SEGMENT]], %[[COUNTER2]] {scopeId = 1 : i32} : !proton_gpu.segment<1024, #smem, warp>, i32
// CHECK:       %[[COUNTER3:.*]] = proton_gpu.read_counter : i32
// CHECK:       proton_gpu.circular_store end %[[SEGMENT]], %[[COUNTER3]] {scopeId = 1 : i32} : !proton_gpu.segment<1024, #smem, warp>, i32
// CHECK:       ttg.warp_yield
// CHECK:     }
// CHECK:     partition0(%[[ARG0:.*]]: !ttg.memdesc<256xi32, #shared, #smem, mutable>, %[[ARG1:.*]]: !tt.ptr<i32>) num_warps(1) {
// CHECK:       %[[SEGMENT2:.*]] = proton_gpu.segment_alloc %[[ARG0]] : !ttg.memdesc<256xi32, #shared, #smem, mutable> -> <1024, #smem, warp>
// CHECK:       proton_gpu.restore_ctx %[[SEGMENT2]], %[[ARG1]] : !proton_gpu.segment<1024, #smem, warp>, !tt.ptr<i32>
// CHECK:       %[[COUNTER4:.*]] = proton_gpu.read_counter : i32
// CHECK:       proton_gpu.circular_store start %[[SEGMENT2]], %[[COUNTER4]] {scopeId = 2 : i32} : !proton_gpu.segment<1024, #smem, warp>, i32
// CHECK:       %[[COUNTER5:.*]] = proton_gpu.read_counter : i32
// CHECK:       proton_gpu.circular_store end %[[SEGMENT2]], %[[COUNTER5]] {scopeId = 2 : i32} : !proton_gpu.segment<1024, #smem, warp>, i32
// CHECK:       ttg.warp_return
// CHECK:     } : (!ttg.memdesc<256xi32, #shared, #smem, mutable>, !tt.ptr<i32>) -> ()
// CHECK:     %[[COUNTER6:.*]] = proton_gpu.read_counter : i32
// CHECK:     proton_gpu.circular_store end %[[SEGMENT]], %[[COUNTER6]] {scopeId = 0 : i32} : !proton_gpu.segment<1024, #smem, warp>, i32
// CHECK: ttg.barrier local|global_read|global_write
// CHECK:     proton_gpu.finalize %[[SEGMENT]], %[[SCRATCH]] : !proton_gpu.segment<1024, #smem, warp>, !tt.ptr<i32>
// CHECK:     tt.return
// CHECK:   }
// CHECK: }
module attributes {"ttg.num-warps" = 4 : i32, "ttg.total-num-warps" = 8 : i32} {
  tt.func @convert_warp_specialize() {
    proton.record start "kernel"
    ttg.warp_specialize()
    default {
      proton.record start "default"
      proton.record end "default"
      ttg.warp_yield
    }
    partition0() num_warps(1) {
      proton.record start "partition0"
      proton.record end "partition0"
      ttg.warp_return
    } : () -> ()
    proton.record end "kernel"
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`, `ttg.total-num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `convert_warp_specialize`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`, `ttg.total-num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `convert_warp_specialize`。

### Line 165
```mlir
// -----
```
**EN:** This separator works with `-split-input-file`, telling lit and `triton-opt` that the next chunk is an independent test case.
**CN:** 这个分隔符配合 `-split-input-file` 使用，表示后面的内容是一个独立的测试用例。

### Lines 167-185
```mlir
module attributes {"ttg.num-warps" = 8 : i32} {
  // CHECK-LABEL: global_mem_buffer
// CHECK-GMEM: %[[SCRATCH:.*]] = ttg.global_scratch_alloc {alignment = 128 : i32, nbytes = 1152 : i32, third_party_allocation} : !tt.ptr<i32>
  // CHECK-GMEM: proton_gpu.initialize %[[SCRATCH]] : !tt.ptr<i32>
  // CHECK-GMEM: %[[PTR:.*]] = tt.addptr %[[SCRATCH]]
  // CHECK-GMEM: %[[SEGMENT:.*]] = proton_gpu.segment_alloc %[[PTR]] : !tt.ptr<i32> -> <1024, #proton_gpu.global_memory, warp>
  // CHECK-GMEM: %[[START:.*]] = proton_gpu.read_counter : i32
  // CHECK-GMEM: proton_gpu.circular_store start %[[SEGMENT]], %[[START]] {scopeId = 0 : i32} : !proton_gpu.segment<1024, #proton_gpu.global_memory, warp>, i32
  // CHECK-GMEM: %[[END:.*]] = proton_gpu.read_counter : i32
  // CHECK-GMEM: proton_gpu.circular_store end %[[SEGMENT]], %[[END]] {scopeId = 0 : i32} : !proton_gpu.segment<1024, #proton_gpu.global_memory, warp>, i32
  // CHECK-GMEM: ttg.barrier local|global_read|global_write
  // CHECK-GMEM: proton_gpu.finalize %[[SEGMENT]], %[[SCRATCH]] : !proton_gpu.segment<1024, #proton_gpu.global_memory, warp>, !tt.ptr<i32>
  // CHECK-GMEM: tt.return
  tt.func @global_mem_buffer() {
    proton.record start "name0"
    proton.record end "name0"
    tt.return
  }
}
```
**EN:** This module sets kernel-level metadata, including `ttg.num-warps`. The enclosed IR is interpreted under those launch and backend assumptions before the tested pass runs. It contains functions such as `global_mem_buffer`.
**CN:** 这个 module 设置了内核级元数据，其中包括 `ttg.num-warps`。被包裹的 IR 会在这些启动配置和后端假设下进入目标 pass。 其中包含的函数如 `global_mem_buffer`。

## Key Concepts / 关键概念

- **EN:** This file belongs to Triton's Proton pipeline coverage and focuses on Proton dialect lowering.  
  **CN:** 该文件属于 Triton 的 Proton 流水线 测试覆盖，关注点是 Proton 方言降级。
- **EN:** The `RUN` pipeline drives tools/passes such as `--split-input-file`, `-convert-proton-to-protongpu="max-shared-mem-size=32768"`, `-canonicalize`, `-cse`, `-convert-proton-to-protongpu="buffer-type=global`, `--check-prefix=CHECK-GMEM`.  
  **CN:** `RUN` 流水线会驱动 `--split-input-file`, `-convert-proton-to-protongpu="max-shared-mem-size=32768"`, `-canonicalize`, `-cse`, `-convert-proton-to-protongpu="buffer-type=global`, `--check-prefix=CHECK-GMEM` 等工具/pass。
- **EN:** The IR mixes `tt` (Triton core ops), `ttg` (TritonGPU ops/layouts), `proton` (Proton dialect ops), `arith` (scalar/tensor arithmetic), `scf` (structured control flow).  
  **CN:** 该 IR 混合使用了 `tt`（Triton 核心操作）、`ttg`（TritonGPU 操作/布局）、`proton`（Proton 方言操作）、`arith`（标量/张量算术）、`scf`（结构化控制流）。
- **EN:** FileCheck directives are used to pin down structural or backend-specific output patterns.  
  **CN:** FileCheck 指令用于固定结构性或后端专用的输出模式。

## Dependencies / 依赖关系

- **EN:** External test tools: `triton-opt`, `FileCheck`.  
  **CN:** 外部测试工具：`triton-opt`、`FileCheck`。
- **EN:** IR dialect dependencies: `tt`, `ttg`, `proton`, `arith`, `scf`.  
  **CN:** IR 方言依赖：`tt`、`ttg`、`proton`、`arith`、`scf`。
