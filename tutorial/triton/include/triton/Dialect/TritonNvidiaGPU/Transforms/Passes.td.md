# Passes.td — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonNvidiaGPU/Transforms/Passes.td`
- **EN:** Defines TableGen pass records, options, and registration metadata for this subsystem.
- **CN:** 定义该子系统的 TableGen pass 记录、选项与注册元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20
```tablegen
   1: // Copyright (c) 2023 NVIDIA Corporation & Affiliates. All rights reserved.
   2: //
   3: // Permission is hereby granted, free of charge, to any person obtaining
   4: // a copy of this software and associated documentation files
   5: // (the "Software"), to deal in the Software without restriction,
   6: // including without limitation the rights to use, copy, modify, merge,
   7: // publish, distribute, sublicense, and/or sell copies of the Software,
   8: // and to permit persons to whom the Software is furnished to do so,
   9: // subject to the following conditions:
  10: //
  11: // The above copyright notice and this permission notice shall be
  12: // included in all copies or substantial portions of the Software.
  13: //
  14: // THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
  15: // EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
  16: // MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
  17: // IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
  18: // CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
  19: // TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
  20: // SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```
**EN:** This comment block records the intent and constraints of the surrounding code: Copyright (c) 2023 NVIDIA Corporation & Affiliates. All rights reserved. Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associ....
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 22-23
```tablegen
  22: #ifndef TRITONNVIDIAGPU_PASSES
  23: #define TRITONNVIDIAGPU_PASSES
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 25-25
```tablegen
  25: include "mlir/Pass/PassBase.td"
```
**EN:** This block imports the direct dependencies needed here, including mlir/Pass/PassBase.td.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/Pass/PassBase.td。

### Lines 27-28
```tablegen
  27: def TritonGPUPlanCTAPass : Pass<"triton-nvidia-gpu-plan-cta", "mlir::ModuleOp"> {
  28:   let summary = "plan CTA";
```
**EN:** This TableGen def record defines `TritonGPUPlanCTAPass` with the summary “plan CTA”. It is specialized from `Pass<"triton-nvidia-gpu-plan-cta", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonGPUPlanCTAPass`，其摘要为“plan CTA”。 它基于 `Pass<"triton-nvidia-gpu-plan-cta", "mlir::ModuleOp">` 进一步特化。

### Lines 30-33
```tablegen
  30:   let description = [{
  31:     This pass computes and applies "optimized" CTA tilings to DotOp, ReduceOp
  32:     and StoreLikeOps operations.
  33:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 35-35
```tablegen
  35:   let constructor = "mlir::triton::nvidia_gpu::createTritonNvidiaGPUPlanCTAPass()";
```
**EN:** This block declares or defines callable APIs such as createTritonNvidiaGPUPlanCTAPass, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 createTritonNvidiaGPUPlanCTAPass 等可调用 API，用来封装这里提供的核心行为。

### Lines 37-41
```tablegen
  37:   let dependentDialects = [
  38:     "mlir::triton::gpu::TritonGPUDialect",
  39:     "mlir::triton::nvidia_gpu::TritonNvidiaGPUDialect"
  40:   ];
  41: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 43-44
```tablegen
  43: def TritonGPUFenceInsertion : Pass<"triton-nvidia-gpu-fence-insertion", "mlir::ModuleOp"> {
  44:   let summary = "Insert fences across generic and async proxy.";
```
**EN:** This TableGen def record defines `TritonGPUFenceInsertion` with the summary “Insert fences across generic and async proxy.”. It is specialized from `Pass<"triton-nvidia-gpu-fence-insertion", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonGPUFenceInsertion`，其摘要为“Insert fences across generic and async proxy.”。 它基于 `Pass<"triton-nvidia-gpu-fence-insertion", "mlir::ModuleOp">` 进一步特化。

### Lines 46-51
```tablegen
  46:   let description = [{
  47:     This pass is to insert memory fences to ensure that memory operations are
  48:     properly ordered across generic and async operations.
  49:     This pass inserts fences at optimized location.
  50:     There is a pass later to handle all the functional requirements
  51:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 53-56
```tablegen
  53:   let dependentDialects = [
  54:     "mlir::triton::gpu::TritonGPUDialect",
  55:     "mlir::triton::nvidia_gpu::TritonNvidiaGPUDialect"
  56:   ];
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 58-63
```tablegen
  58:   let options = [
  59:     Option<"computeCapability", "compute-capability",
  60:            "int32_t", /*default*/"90",
  61:            "device compute capability">
  62:   ];
  63: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 65-66
```tablegen
  65: def TritonGPUProxyFenceInsertion : Pass<"triton-nvidia-gpu-proxy-fence-insertion", "mlir::ModuleOp"> {
  66:   let summary = "Insert fences across generic and async proxy";
```
**EN:** This TableGen def record defines `TritonGPUProxyFenceInsertion` with the summary “Insert fences across generic and async proxy”. It is specialized from `Pass<"triton-nvidia-gpu-proxy-fence-insertion", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonGPUProxyFenceInsertion`，其摘要为“Insert fences across generic and async proxy”。 它基于 `Pass<"triton-nvidia-gpu-proxy-fence-insertion", "mlir::ModuleOp">` 进一步特化。

### Lines 68-71
```tablegen
  68:   let description = [{
  69:     This pass is to insert memory fences to ensure that memory operations are
  70:     properly ordered across generic and async operations.
  71:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 73-76
```tablegen
  73:   let dependentDialects = [
  74:     "mlir::triton::gpu::TritonGPUDialect",
  75:     "mlir::triton::nvidia_gpu::TritonNvidiaGPUDialect"
  76:   ];
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 78-83
```tablegen
  78:   let options = [
  79:     Option<"computeCapability", "compute-capability",
  80:            "int32_t", /*default*/"90",
  81:            "device compute capability">
  82:   ];
  83: }
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 85-86
```tablegen
  85: def TritonNvidiaGPUTMemBarrierInsertionPass : Pass<"triton-nvidia-gpu-tmem-barrier-insertion", "mlir::ModuleOp"> {
  86:   let summary = "Insert barriers to order tensor-memory reuse";
```
**EN:** This TableGen def record defines `TritonNvidiaGPUTMemBarrierInsertionPass` with the summary “Insert barriers to order tensor-memory reuse”. It is specialized from `Pass<"triton-nvidia-gpu-tmem-barrier-insertion", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonNvidiaGPUTMemBarrierInsertionPass`，其摘要为“Insert barriers to order tensor-memory reuse”。 它基于 `Pass<"triton-nvidia-gpu-tmem-barrier-insertion", "mlir::ModuleOp">` 进一步特化。

### Lines 88-92
```tablegen
  88:   let description = [{
  89:     This pass inserts CTA barriers between tensor-memory accesses that reuse
  90:     aliasing physical tensor-memory storage and require ordering across logical
  91:     allocation lifetimes.
  92:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 94-98
```tablegen
  94:   let dependentDialects = [
  95:     "mlir::triton::gpu::TritonGPUDialect",
  96:     "mlir::triton::nvidia_gpu::TritonNvidiaGPUDialect"
  97:   ];
  98: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 100-101
```tablegen
 100: def TritonNvidiaGPUTMALoweringPass : Pass<"triton-nvidia-tma-lowering", "mlir::ModuleOp"> {
 101:   let summary = "lower to TMA load/store operations";
```
**EN:** This TableGen def record defines `TritonNvidiaGPUTMALoweringPass` with the summary “lower to TMA load/store operations”. It is specialized from `Pass<"triton-nvidia-tma-lowering", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonNvidiaGPUTMALoweringPass`，其摘要为“lower to TMA load/store operations”。 它基于 `Pass<"triton-nvidia-tma-lowering", "mlir::ModuleOp">` 进一步特化。

### Lines 103-105
```tablegen
 103:   let description = [{
 104:     Lower Triton descriptor load to TMA load/store operations in TritonNvidiaGPUDialect.
 105:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 107-110
```tablegen
 107:   let dependentDialects = [
 108:     "mlir::triton::nvidia_gpu::TritonNvidiaGPUDialect"
 109:   ];
 110: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 112-113
```tablegen
 112: def TritonTensorMemoryAllocationPass : Pass<"triton-tensor-memory-allocation", "mlir::ModuleOp"> {
 113:   let summary = "Assign tensor memory allocation";
```
**EN:** This TableGen def record defines `TritonTensorMemoryAllocationPass` with the summary “Assign tensor memory allocation”. It is specialized from `Pass<"triton-tensor-memory-allocation", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonTensorMemoryAllocationPass`，其摘要为“Assign tensor memory allocation”。 它基于 `Pass<"triton-tensor-memory-allocation", "mlir::ModuleOp">` 进一步特化。

### Lines 115-117
```tablegen
 115:   let description = [{
 116:     Decide on tensor memory allocation and assign attributes to each allocation.
 117:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 119-122
```tablegen
 119:   let dependentDialects = [
 120:     "mlir::triton::nvidia_gpu::TritonNvidiaGPUDialect"
 121:   ];
 122: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 124-125
```tablegen
 124: def TritonNvidiaGPUMMALoweringPass : Pass<"triton-nvidia-mma-lowering", "mlir::ModuleOp"> {
 125:   let summary = "lower mma operations if needed";
```
**EN:** This TableGen def record defines `TritonNvidiaGPUMMALoweringPass` with the summary “lower mma operations if needed”. It is specialized from `Pass<"triton-nvidia-mma-lowering", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonNvidiaGPUMMALoweringPass`，其摘要为“lower mma operations if needed”。 它基于 `Pass<"triton-nvidia-mma-lowering", "mlir::ModuleOp">` 进一步特化。

### Lines 127-129
```tablegen
 127:   let description = [{
 128:     Lower MMA ops to prepare for conversion to LLVM.
 129:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 131-134
```tablegen
 131:   let dependentDialects = [
 132:     "mlir::triton::nvidia_gpu::TritonNvidiaGPUDialect"
 133:   ];
 134: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 136-137
```tablegen
 136: def TritonNvidiaGPUPromoteLHSToTMemPass : Pass<"tritongpu-promote-lhs-to-tmem", "mlir::ModuleOp"> {
 137:   let summary = "Promote LHS operand of MMAv5 op to Tensor Memory";
```
**EN:** This TableGen def record defines `TritonNvidiaGPUPromoteLHSToTMemPass` with the summary “Promote LHS operand of MMAv5 op to Tensor Memory”. It is specialized from `Pass<"tritongpu-promote-lhs-to-tmem", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonNvidiaGPUPromoteLHSToTMemPass`，其摘要为“Promote LHS operand of MMAv5 op to Tensor Memory”。 它基于 `Pass<"tritongpu-promote-lhs-to-tmem", "mlir::ModuleOp">` 进一步特化。

### Lines 139-141
```tablegen
 139:   let description = [{
 140:     Promote LHS operand of MMAv5 op to Tensor Memory.
 141:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 143-146
```tablegen
 143:   let dependentDialects = ["mlir::triton::gpu::TritonGPUDialect",
 144:                            "mlir::triton::nvidia_gpu::TritonNvidiaGPUDialect",
 145:                            "mlir::triton::TritonDialect"];
 146: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 148-149
```tablegen
 148: def TritonNvidiaGPUOptimizeDescriptorEncodingPass : Pass<"triton-nvidia-optimize-descriptor-encoding", "mlir::ModuleOp"> {
 149:   let summary = "Set encodings on tensor descriptor types";
```
**EN:** This TableGen def record defines `TritonNvidiaGPUOptimizeDescriptorEncodingPass` with the summary “Set encodings on tensor descriptor types”. It is specialized from `Pass<"triton-nvidia-optimize-descriptor-encoding", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonNvidiaGPUOptimizeDescriptorEncodingPass`，其摘要为“Set encodings on tensor descriptor types”。 它基于 `Pass<"triton-nvidia-optimize-descriptor-encoding", "mlir::ModuleOp">` 进一步特化。

### Lines 151-153
```tablegen
 151:   let description = [{
 152:     Set shared memory encoding on tensor descriptors, which decides the swizzling mode and message size of the tma descriptor.
 153:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 155-158
```tablegen
 155:   let dependentDialects = ["mlir::triton::gpu::TritonGPUDialect",
 156:                            "mlir::triton::nvidia_gpu::TritonNvidiaGPUDialect",
 157:                            "mlir::triton::TritonDialect"];
 158: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 160-161
```tablegen
 160: def TritonNvidiaGPUOptimizeTMemLayoutsPass : Pass<"triton-nvidia-optimize-tmem-layouts", "mlir::ModuleOp"> {
 161:   let summary = "Optimize TMEM layouts.";
```
**EN:** This TableGen def record defines `TritonNvidiaGPUOptimizeTMemLayoutsPass` with the summary “Optimize TMEM layouts.”. It is specialized from `Pass<"triton-nvidia-optimize-tmem-layouts", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonNvidiaGPUOptimizeTMemLayoutsPass`，其摘要为“Optimize TMEM layouts.”。 它基于 `Pass<"triton-nvidia-optimize-tmem-layouts", "mlir::ModuleOp">` 进一步特化。

### Lines 163-166
```tablegen
 163:   let description = [{
 164:     Optimize TMEM layouts by selecting a layouts to enable better subtiling,
 165:     reduction performance, etc.
 166:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 168-171
```tablegen
 168:   let dependentDialects = ["mlir::triton::gpu::TritonGPUDialect",
 169:                            "mlir::triton::nvidia_gpu::TritonNvidiaGPUDialect",
 170:                            "mlir::triton::TritonDialect"];
 171: }
```
**EN:** This block lists dialect dependencies that must be loaded before the generated pass or dialect can operate correctly.
**CN:** 该代码块列出了生成 pass 或方言运行前必须加载的依赖方言。

### Lines 173-174
```tablegen
 173: def TritonNvidiaGPUInterleaveTMemPass : Pass<"triton-nvidia-interleave-tmem", "mlir::ModuleOp"> {
 174:   let summary = "Interleave TMEM loads/stores.";
```
**EN:** This TableGen def record defines `TritonNvidiaGPUInterleaveTMemPass` with the summary “Interleave TMEM loads/stores.”. It is specialized from `Pass<"triton-nvidia-interleave-tmem", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonNvidiaGPUInterleaveTMemPass`，其摘要为“Interleave TMEM loads/stores.”。 它基于 `Pass<"triton-nvidia-interleave-tmem", "mlir::ModuleOp">` 进一步特化。

### Lines 176-181
```tablegen
 176:   let description = [{
 177:     The `triton-nvidia-interleave-tmem` pass attempts to sink TMEM loads and
 178:     hoist TMEM stores, and potentially interleave them, to reduce register
 179:     pressure.
 180:   }];
 181: }
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 183-184
```tablegen
 183: def TritonNvidiaGPURemoveTMEMTokensPass : Pass<"triton-nvidia-gpu-remove-tmem-tokens", "mlir::ModuleOp"> {
 184:   let summary = "remove TMEM tokens";
```
**EN:** This TableGen def record defines `TritonNvidiaGPURemoveTMEMTokensPass` with the summary “remove TMEM tokens”. It is specialized from `Pass<"triton-nvidia-gpu-remove-tmem-tokens", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonNvidiaGPURemoveTMEMTokensPass`，其摘要为“remove TMEM tokens”。 它基于 `Pass<"triton-nvidia-gpu-remove-tmem-tokens", "mlir::ModuleOp">` 进一步特化。

### Lines 186-190
```tablegen
 186:   let description = [{
 187:     The `triton-nvidia-gpu-remove-tmem-tokens` pass removes TMEM memory
 188:     dependency tokens from the IR, after they are no longer needed.
 189:   }];
 190: }
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 192-193
```tablegen
 192: def TritonNvidiaGPUCheckMatmulTwoCTAPass : Pass<"triton-nvidia-check-matmul-two-cta", "mlir::ModuleOp"> {
 193:   let summary = "Verify consistent two_ctas usage across matmuls";
```
**EN:** This TableGen def record defines `TritonNvidiaGPUCheckMatmulTwoCTAPass` with the summary “Verify consistent two_ctas usage across matmuls”. It is specialized from `Pass<"triton-nvidia-check-matmul-two-cta", "mlir::ModuleOp">`.
**CN:** 该 TableGen def 记录定义了 `TritonNvidiaGPUCheckMatmulTwoCTAPass`，其摘要为“Verify consistent two_ctas usage across matmuls”。 它基于 `Pass<"triton-nvidia-check-matmul-two-cta", "mlir::ModuleOp">` 进一步特化。

### Lines 195-200
```tablegen
 195:   let description = [{
 196:     Inspect all matmul operations and ensure they agree on the `two_ctas`
 197:     setting. Propagate the chosen value to the module so later lowering steps
 198:     can access it. Compilation fails if mixed configurations are detected.
 199:   }];
 200: }
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 202-202
```tablegen
 202: #endif
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** shared-memory allocation  
  **CN:** 共享内存分配
- **EN:** matrix-multiply acceleration  
  **CN:** 矩阵乘加加速
- **EN:** dot-product lowering  
  **CN:** 点积降级
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** compiler passes  
  **CN:** 编译 pass
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** attributes  
  **CN:** 属性

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/Pass/PassBase.td`
