# TritonNvidiaGPUOps.td — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUOps.td`
- **EN:** Defines TableGen operation records, summaries, operands, results, and assembly syntax.
- **CN:** 定义 TableGen 操作记录，以及摘要、操作数、结果和汇编语法。

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
  22: #ifndef TRITONNVIDIAGPU_OPS
  23: #define TRITONNVIDIAGPU_OPS
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 25-41
```tablegen
  25: include "triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUDialect.td"
  26: include "triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUAttrDefs.td"
  27: include "triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUOpInterfaces.td"
  28: include "mlir/Dialect/Arith/IR/ArithBase.td"
  29: include "triton/Dialect/Triton/IR/TritonTypes.td"
  30: include "triton/Dialect/Triton/IR/TritonAttrDefs.td"
  31: include "triton/Dialect/Triton/IR/TritonInterfaces.td"
  32: include "triton/Dialect/Triton/IR/TritonOpInterfaces.td"
  33: include "triton/Dialect/TritonGPU/IR/TritonGPUAttrDefs.td"
  34: include "triton/Dialect/TritonGPU/IR/TritonGPUTypes.td"
  35: include "triton/Dialect/TritonGPU/IR/TritonGPUTypeInterfaces.td"
  36: include "triton/Dialect/TritonGPU/IR/TritonGPUOpInterfaces.td"
  37: include "mlir/IR/OpBase.td"
  38: include "mlir/Interfaces/SideEffectInterfaces.td" // Pure
  39: include "mlir/Interfaces/InferTypeOpInterface.td" // SameOperandsAndResultType
  40: include "mlir/Interfaces/DestinationStyleOpInterface.td"
  41: include "mlir/Interfaces/ViewLikeInterface.td"
```
**EN:** This block imports the direct dependencies needed here, including triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUDialect.td, triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUAttrDefs.td, triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUOpInterfaces.td, mlir/Dialect/Arith/IR/ArithBase.td, triton/Dialect/Triton/IR/TritonTypes.td, and triton/Dialect/Triton/IR/TritonAttrDefs.td.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUDialect.td, triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUAttrDefs.td, triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUOpInterfaces.td, mlir/Dialect/Arith/IR/ArithBase.td, triton/Dialect/Triton/IR/TritonTypes.td, and triton/Dialect/Triton/IR/TritonAttrDefs.td。

### Lines 43-45
```tablegen
  43: def GlobalMemory : Resource<"::mlir::triton::GlobalMemory">;
  44: def SharedMemory : Resource<"::mlir::triton::gpu::SharedMemory">;
  45: def TensorMemory : Resource<"::mlir::triton::nvidia_gpu::TensorMemory">;
```
**EN:** This TableGen def record defines `GlobalMemory`. It is specialized from `Resource<"::mlir::triton::GlobalMemory">;`.
**CN:** 该 TableGen def 记录定义了 `GlobalMemory`。 它基于 `Resource<"::mlir::triton::GlobalMemory">;` 进一步特化。

### Lines 47-50
```tablegen
  47: class TTNG_Op<string mnemonic, list<Trait> traits = []> :
  48:     Op<TritonNvidiaGPU_Dialect, mnemonic,
  49:        !listconcat(traits, [VerifyTensorLayoutsTrait, VerifyMemDescLayoutsTrait])> {
  50: }
```
**EN:** This block introduces `TTNG_Op`, the main class/struct defined here. Within the declaration, methods such as listconcat expose its core API. It also inherits behavior from a base type.
**CN:** 该代码块引入了此文件的核心类/结构体 `TTNG_Op`。 其中 listconcat 等方法构成了它的主要接口。 它还通过继承复用基类能力。

### Lines 52-53
```tablegen
  52: def TTNG_FenceAsyncSharedOp : TTNG_Op<"fence_async_shared"> {
  53:   let arguments = (ins BoolAttr:$bCluster);
```
**EN:** This TableGen def record defines `TTNG_FenceAsyncSharedOp`. It is specialized from `TTNG_Op<"fence_async_shared">`.
**CN:** 该 TableGen def 记录定义了 `TTNG_FenceAsyncSharedOp`。 它基于 `TTNG_Op<"fence_async_shared">` 进一步特化。

### Lines 55-55
```tablegen
  55:   let summary = "fence proxy async";
```
**EN:** This block assigns the one-line summary used to describe fence proxy async in generated documentation and diagnostics.
**CN:** 该代码块设置一行摘要，用于在生成文档和诊断信息中描述 fence proxy async。

### Lines 57-57
```tablegen
  57:   let assemblyFormat = "attr-dict";
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 59-64
```tablegen
  59:   let extraClassDeclaration = [{
  60:     static bool isSupported(int computeCapability) {
  61:       return computeCapability >= 90;
  62:     }
  63:   }];
  64: }
```
**EN:** This block injects extra C++ helpers into the generated class, such as isSupported.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 isSupported。

### Lines 66-68
```tablegen
  66: def TTNG_FenceMBarrierInitReleaseClusterOp : TTNG_Op<
  67:     "fence_mbarrier_init_release_cluster"> {
  68:   let summary = "fence mbarrier init release.cluster";
```
**EN:** This TableGen def record defines `TTNG_FenceMBarrierInitReleaseClusterOp` with the summary “fence mbarrier init release.cluster”. It is specialized from `TTNG_Op<`.
**CN:** 该 TableGen def 记录定义了 `TTNG_FenceMBarrierInitReleaseClusterOp`，其摘要为“fence mbarrier init release.cluster”。 它基于 `TTNG_Op<` 进一步特化。

### Lines 70-71
```tablegen
  70:   let assemblyFormat = "attr-dict";
  71:   let hasVerifier = 1;
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 73-79
```tablegen
  73:   let extraClassDeclaration = [{
  74:     static bool isSupported(int computeCapability) {
  75:       return ::mlir::triton::nvidia_gpu::TargetFeatures(computeCapability)
  76:           .supportClusterOps();
  77:     }
  78:   }];
  79: }
```
**EN:** This block injects extra C++ helpers into the generated class, such as isSupported and supportClusterOps.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 isSupported and supportClusterOps。

### Lines 81-85
```tablegen
  81: def TTNG_ClusterArriveOp : TTNG_Op<"cluster_arrive", []> {
  82:   let arguments = (ins DefaultValuedAttr<BoolAttr, "false">:$relaxed);
  83:   let assemblyFormat = "attr-dict";
  84:   let hasVerifier = 1;
  85: }
```
**EN:** This TableGen def record defines `TTNG_ClusterArriveOp`. It is specialized from `TTNG_Op<"cluster_arrive", []>`.
**CN:** 该 TableGen def 记录定义了 `TTNG_ClusterArriveOp`。 它基于 `TTNG_Op<"cluster_arrive", []>` 进一步特化。

### Lines 87-90
```tablegen
  87: def TTNG_ClusterWaitOp : TTNG_Op<"cluster_wait", []> {
  88:   let assemblyFormat = "attr-dict";
  89:   let hasVerifier = 1;
  90: }
```
**EN:** This TableGen def record defines `TTNG_ClusterWaitOp`. It is specialized from `TTNG_Op<"cluster_wait", []>`.
**CN:** 该 TableGen def 记录定义了 `TTNG_ClusterWaitOp`。 它基于 `TTNG_Op<"cluster_wait", []>` 进一步特化。

### Lines 92-96
```tablegen
  92: def TTNG_ClusterBarrierOp : TTNG_Op<"cluster_barrier", []> {
  93:   let arguments = (ins DefaultValuedAttr<BoolAttr, "false">:$relaxed);
  94:   let summary = "Synchronize all warps at cluster scope";
  95:   let description = [{
  96:     Lowers to a cluster arrive/wait pair.
```
**EN:** This TableGen def record defines `TTNG_ClusterBarrierOp` with the summary “Synchronize all warps at cluster scope”. It is specialized from `TTNG_Op<"cluster_barrier", []>`.
**CN:** 该 TableGen def 记录定义了 `TTNG_ClusterBarrierOp`，其摘要为“Synchronize all warps at cluster scope”。 它基于 `TTNG_Op<"cluster_barrier", []>` 进一步特化。

### Lines 98-104
```tablegen
  98:     In warp-specialized kernels, lowering wraps the barrier in a synthetic
  99:     `ttg.warp_specialize` region so worker warps also execute the barrier.
 100:     This op cannot be placed inside an existing `ttg.warp_specialize`.
 101:   }];
 102:   let assemblyFormat = "attr-dict";
 103:   let hasVerifier = 1;
 104: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 106-111
```tablegen
 106: //
 107: // Cluster Launch Control (CLC) Ops - Blackwell SM100+
 108: //
 109: def TTNG_CLCTryCancelOp : TTNG_Op<"clc_try_cancel", [
 110:     DeclareOpInterfaceMethods<MBarrierOpInterface, ["getBarrier"]>]> {
 111:   let summary = "Issue CLC try_cancel to cancel a pending cluster";
```
**EN:** This TableGen def record defines `TTNG_CLCTryCancelOp` with the summary “Issue CLC try_cancel to cancel a pending cluster”. It is specialized from `TTNG_Op<"clc_try_cancel", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_CLCTryCancelOp`，其摘要为“Issue CLC try_cancel to cancel a pending cluster”。 它基于 `TTNG_Op<"clc_try_cancel", [` 进一步特化。

### Lines 113-116
```tablegen
 113:   let description = [{
 114:     Issues a clusterlaunchcontrol.try_cancel instruction to atomically cancel
 115:     a pending cluster launch. The result is written asynchronously to the
 116:     result buffer and the mbarrier is signaled on completion.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 118-118
```tablegen
 118:     This is used for dynamic persistent kernels on Blackwell (SM100+).
```
**EN:** This block declares or defines callable APIs such as Blackwell, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 Blackwell 等可调用 API，用来封装这里提供的核心行为。

### Lines 120-122
```tablegen
 120:     The result buffer must be 16-byte aligned shared memory.
 121:     The mbarrier must be 8-byte aligned shared memory.
 122:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 124-127
```tablegen
 124:   let arguments = (ins
 125:     Arg<TTG_MemDescType, "", [MemWrite<SharedMemory>]>:$result,
 126:     Arg<TTG_MemDescType, "", [MemWrite<SharedMemory>]>:$mbarrier
 127:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 129-133
```tablegen
 129:   let assemblyFormat = [{
 130:     $result `,` $mbarrier attr-dict `:` qualified(type($result)) `,` qualified(type($mbarrier))
 131:   }];
 132:   let hasVerifier = 1;
 133: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 135-136
```tablegen
 135: def TTNG_CLCLoadResultOp : TTNG_Op<"clc_load_result", []> {
 136:   let summary = "Load CLC response from shared memory into registers";
```
**EN:** This TableGen def record defines `TTNG_CLCLoadResultOp` with the summary “Load CLC response from shared memory into registers”. It is specialized from `TTNG_Op<"clc_load_result", []>`.
**CN:** 该 TableGen def 记录定义了 `TTNG_CLCLoadResultOp`，其摘要为“Load CLC response from shared memory into registers”。 它基于 `TTNG_Op<"clc_load_result", []>` 进一步特化。

### Lines 138-142
```tablegen
 138:   let description = [{
 139:     Loads the 128-bit CLC response from shared memory into two i64 registers.
 140:     This allows subsequent is_canceled and get_first_ctaid operations to
 141:     operate on registers without re-reading shared memory.
 142:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 144-146
```tablegen
 144:   let arguments = (ins
 145:     Arg<TTG_MemDescType, "", [MemRead<SharedMemory>]>:$src
 146:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 148-148
```tablegen
 148:   let results = (outs I128:$clcResult);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 150-154
```tablegen
 150:   let assemblyFormat = [{
 151:     $src attr-dict `:` qualified(type($src)) `->` type($clcResult)
 152:   }];
 153:   let hasVerifier = 1;
 154: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 156-157
```tablegen
 156: def TTNG_CLCIsCanceledOp : TTNG_Op<"clc_is_canceled", [Pure]> {
 157:   let summary = "Check if CLC response indicates successful cancellation";
```
**EN:** This TableGen def record defines `TTNG_CLCIsCanceledOp` with the summary “Check if CLC response indicates successful cancellation”. It is specialized from `TTNG_Op<"clc_is_canceled", [Pure]>`.
**CN:** 该 TableGen def 记录定义了 `TTNG_CLCIsCanceledOp`，其摘要为“Check if CLC response indicates successful cancellation”。 它基于 `TTNG_Op<"clc_is_canceled", [Pure]>` 进一步特化。

### Lines 159-162
```tablegen
 159:   let description = [{
 160:     Decodes the CLC response to check if a cluster was successfully
 161:     canceled. Returns true if canceled, false otherwise.
 162:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 164-164
```tablegen
 164:   let arguments = (ins I128:$clcResult);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 166-166
```tablegen
 166:   let results = (outs I1:$is_canceled);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 168-171
```tablegen
 168:   let assemblyFormat = [{
 169:     $clcResult attr-dict `:` type($clcResult) `->` type($is_canceled)
 170:   }];
 171: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 173-174
```tablegen
 173: def TTNG_CLCGetProgramIdOp : TTNG_Op<"clc_get_program_id", [Pure]> {
 174:   let summary = "Get CTA ID coordinate from CLC response";
```
**EN:** This TableGen def record defines `TTNG_CLCGetProgramIdOp` with the summary “Get CTA ID coordinate from CLC response”. It is specialized from `TTNG_Op<"clc_get_program_id", [Pure]>`.
**CN:** 该 TableGen def 记录定义了 `TTNG_CLCGetProgramIdOp`，其摘要为“Get CTA ID coordinate from CLC response”。 它基于 `TTNG_Op<"clc_get_program_id", [Pure]>` 进一步特化。

### Lines 176-179
```tablegen
 176:   let description = [{
 177:     Decodes the CLC response to get the first CTA ID coordinate of the
 178:     canceled cluster. The dim attribute specifies which dimension (0=x, 1=y, 2=z).
 179:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 181-184
```tablegen
 181:   let arguments = (ins
 182:     I128:$clcResult,
 183:     TT_ProgramDim:$dim
 184:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 186-186
```tablegen
 186:   let results = (outs I32:$result);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 188-190
```tablegen
 188:   let assemblyFormat = [{
 189:     $clcResult `,` $dim attr-dict `:` type($clcResult) `->` type($result)
 190:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 192-197
```tablegen
 192:   let builders = [
 193:     OpBuilder<(ins "Value":$clcResult, "int":$axis), [{
 194:       build($_builder, $_state, clcResult, ProgramIDDim(axis));
 195:     }]>
 196:   ];
 197: }
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 199-208
```tablegen
 199: //
 200: // WarpGroupDot Op
 201: //
 202: def TTNG_WarpGroupDotOp : TTNG_Op<"warp_group_dot", [
 203:   DeclareOpInterfaceMethods<InferTypeOpInterface>,
 204:   DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
 205:   DeclareOpInterfaceMethods<DotOpInterface>,
 206:   TypesMatchWith<"result's type matches accumulator's type", "d", "c", "$_self">
 207: ]> {
 208:   let summary = "warp group dot";
```
**EN:** This TableGen def record defines `TTNG_WarpGroupDotOp` with the summary “warp group dot”. It is specialized from `TTNG_Op<"warp_group_dot", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_WarpGroupDotOp`，其摘要为“warp group dot”。 它基于 `TTNG_Op<"warp_group_dot", [` 进一步特化。

### Lines 210-212
```tablegen
 210:   let description = [{
 211:     $d = matrix_multiply($a, $b) + $c. For docs on InputPrecisionAttr, see TT_DotOp
 212:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 214-222
```tablegen
 214:   let arguments = (ins
 215:     TTG_TensorOrMemDesc:$a,
 216:     TTG_MemDescType:$b,
 217:     TT_FpIntTensor:$c,
 218:     Optional<I1>:$useC,
 219:     DefaultValuedAttr<TT_InputPrecisionAttr, "::mlir::triton::InputPrecision::IEEE">:$inputPrecision,
 220:     DefaultValuedAttr<I32Attr, "0">:$maxNumImpreciseAcc,
 221:     DefaultValuedAttr<BoolAttr, "false">:$isAsync
 222:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 224-224
```tablegen
 224:   let results = (outs TT_FpIntTensor:$d);
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 226-229
```tablegen
 226:   let assemblyFormat = [{
 227:     $a`,` $b`,` $c (`,` $useC^)? attr-dict
 228:     `:` type($a) `*` qualified(type($b)) `->` type($d)
 229:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 231-233
```tablegen
 231:   let extraClassDeclaration = [{
 232:     bool needsPartialAccumulator();
 233:   }];
```
**EN:** This block injects extra C++ helpers into the generated class, such as needsPartialAccumulator.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 needsPartialAccumulator。

### Lines 235-236
```tablegen
 235:   let hasVerifier = 1;
 236: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 238-244
```tablegen
 238: def TTNG_WarpGroupDotWaitOp : TTNG_Op<"warp_group_dot_wait", [DeclareOpInterfaceMethods<InferTypeOpInterface>,
 239:                                                               AllTypesMatch<["inputs", "outputs"]>]> {
 240:   let summary = "warp group dot wait";
 241:   let arguments = (ins Variadic<TTG_TensorOrMemDesc>:$inputs, I32Attr:$pendings);
 242:   let results = (outs Variadic<TTG_TensorOrMemDesc>:$outputs);
 243:   let description = [{
 244:     Waits until there are $pendings or fewer outstanding async dot operations.
```
**EN:** This TableGen def record defines `TTNG_WarpGroupDotWaitOp` with the summary “warp group dot wait”. It is specialized from `TTNG_Op<"warp_group_dot_wait", [DeclareOpInterfaceMethods<InferTypeOpInterface>,`.
**CN:** 该 TableGen def 记录定义了 `TTNG_WarpGroupDotWaitOp`，其摘要为“warp group dot wait”。 它基于 `TTNG_Op<"warp_group_dot_wait", [DeclareOpInterfaceMethods<InferTypeOpInterface>,` 进一步特化。

### Lines 246-249
```tablegen
 246:     $inputs must be the tensors corresponding to the async dot ops that we're
 247:     waiting on.  For example, if there are N pending async dot ops and we call
 248:     `warp_group_dot_wait 1`, then $inputs must be the result of the first dot op.
 249:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 251-253
```tablegen
 251:   let assemblyFormat = "$inputs attr-dict `:` type($inputs)";
 252:   let hasVerifier = 1;
 253: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 255-257
```tablegen
 255: def TTNG_InitBarrierOp : TTNG_Op<"init_barrier", [
 256:     DeclareOpInterfaceMethods<MBarrierOpInterface, ["getBarrier"]>]> {
 257:   let summary = "Initialize a barrier in the given shared memory allocation.";
```
**EN:** This TableGen def record defines `TTNG_InitBarrierOp` with the summary “Initialize a barrier in the given shared memory allocation.”. It is specialized from `TTNG_Op<"init_barrier", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_InitBarrierOp`，其摘要为“Initialize a barrier in the given shared memory allocation.”。 它基于 `TTNG_Op<"init_barrier", [` 进一步特化。

### Lines 259-262
```tablegen
 259:   let description = [{
 260:       Initializes a shared memory allocation with mbarrier information.
 261:       `alloc` is a descriptor to the shared memory allocation. `count` is the
 262:       number of arrives expected by the barrier.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 264-265
```tablegen
 264:       This lowers to PTX mbarrier.init.shared::cta.b64.
 265:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 267-273
```tablegen
 267:   let arguments = (ins
 268:     Arg<TTG_MemDescType, "", [MemWrite<SharedMemory>]>:$alloc,
 269:     I32Attr:$count
 270:   );
 271:   let assemblyFormat = "$alloc `,` $count attr-dict `:` qualified(type($alloc))";
 272:   let hasVerifier = 1;
 273: }
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 275-277
```tablegen
 275: def TTNG_InvalBarrierOp : TTNG_Op<"inval_barrier", [
 276:     DeclareOpInterfaceMethods<MBarrierOpInterface, ["getBarrier"]>]> {
 277:   let summary = "Invalidate a barrier allocation.";
```
**EN:** This TableGen def record defines `TTNG_InvalBarrierOp` with the summary “Invalidate a barrier allocation.”. It is specialized from `TTNG_Op<"inval_barrier", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_InvalBarrierOp`，其摘要为“Invalidate a barrier allocation.”。 它基于 `TTNG_Op<"inval_barrier", [` 进一步特化。

### Lines 279-281
```tablegen
 279:   let description = [{
 280:     Invalidate a barrier allocation so that it can be re-used. According to PTX
 281:     spec this has to be done before any reuse of the memory used by mbarrier.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 283-284
```tablegen
 283:     https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#parallel-synchronization-and-communication-instructions-mbarrier-inval
 284:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 286-289
```tablegen
 286:   let hasVerifier = 1;
 287:   let arguments = (ins Arg<TTG_MemDescType, "", [MemWrite<SharedMemory>]>:$alloc);
 288:   let assemblyFormat = "$alloc attr-dict `:` qualified(type($alloc))";
 289: }
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 291-294
```tablegen
 291: def TTNG_BarrierExpectOp : TTNG_Op<"barrier_expect", [
 292:     DeclareOpInterfaceMethods<MBarrierOpInterface, ["getBarrier"]>,
 293:     DeclareOpInterfaceMethods<PredicatedOpInterface>]> {
 294:   let summary = "Signal a barrier of an expected number of bytes to be copied.";
```
**EN:** This TableGen def record defines `TTNG_BarrierExpectOp` with the summary “Signal a barrier of an expected number of bytes to be copied.”. It is specialized from `TTNG_Op<"barrier_expect", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_BarrierExpectOp`，其摘要为“Signal a barrier of an expected number of bytes to be copied.”。 它基于 `TTNG_Op<"barrier_expect", [` 进一步特化。

### Lines 296-299
```tablegen
 296:   let description = [{
 297:     This signal the barrier that `size` bytes are expected to be copied. The
 298:     associated barrier wait will block until the expected number of bytes are copied.
 299:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 301-306
```tablegen
 301:   let hasVerifier = 1;
 302:   let arguments = (ins
 303:     Arg<TTG_MemDescType, "", [MemWrite<SharedMemory>]>:$alloc,
 304:     I32Attr:$size,
 305:     I1:$pred
 306:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 308-310
```tablegen
 308:   let assemblyFormat = [{
 309:     $alloc `,` $size attr-dict `,` $pred `:` qualified(type($alloc))
 310:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 312-312
```tablegen
 312: }
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 314-317
```tablegen
 314: def TTNG_WaitBarrierOp : TTNG_Op<"wait_barrier", [AttrSizedOperandSegments,
 315:     DeclareOpInterfaceMethods<MBarrierOpInterface, ["getBarrier"]>,
 316:     DeclareOpInterfaceMethods<PredicatedOpInterface>]> {
 317:   let summary = "wait until the mbarrier phase completes.";
```
**EN:** This TableGen def record defines `TTNG_WaitBarrierOp` with the summary “wait until the mbarrier phase completes.”. It is specialized from `TTNG_Op<"wait_barrier", [AttrSizedOperandSegments,`.
**CN:** 该 TableGen def 记录定义了 `TTNG_WaitBarrierOp`，其摘要为“wait until the mbarrier phase completes.”。 它基于 `TTNG_Op<"wait_barrier", [AttrSizedOperandSegments,` 进一步特化。

### Lines 319-321
```tablegen
 319:   let description = [{
 320:     Blocks the program progress until the mbarrier object in `alloc` completes
 321:     its current phase.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 323-324
```tablegen
 323:     This lowers a waitloop using PTX instruction
 324:     mbarrier.try_wait.parity.shared::cta.b64.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 326-327
```tablegen
 326:     Accepts optional list of memory. If present, it is assumed that any of the
 327:     dependencies may be accessed until the barrier completes.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 329-331
```tablegen
 329:     The barrier behavior is described here:
 330:     https://docs.nvidia.com/cuda/parallel-thread-execution/index.html#data-movement-and-conversion-instructions-asynchronous-copy-completion-mechanisms
 331:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 333-338
```tablegen
 333:   let arguments = (ins
 334:     Arg<TTG_MemDescType, "", [MemRead<SharedMemory>, MemWrite<SharedMemory>]>:$alloc,
 335:     I32:$phase,
 336:     Optional<I1>:$pred,
 337:     Variadic<TTG_MemDescType>:$deps
 338:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 340-353
```tablegen
 340:   let builders = [
 341:     OpBuilder<(ins "Value":$alloc, "Value":$phase),
 342:     [{
 343:     build($_builder, $_state, alloc, phase, /*pred=*/static_cast<mlir::Value>(nullptr), /*deps=*/{});
 344:     }]>,
 345:     OpBuilder<(ins "Value":$alloc, "Value":$phase, "Value":$pred),
 346:     [{
 347:     build($_builder, $_state, alloc, phase, pred, /*deps=*/{});
 348:     }]>,
 349:     OpBuilder<(ins "Value":$alloc, "Value":$phase, "ValueRange":$deps),
 350:     [{
 351:     build($_builder, $_state, alloc, phase, /*pred=*/static_cast<mlir::Value>(nullptr), deps);
 352:     }]>,
 353:   ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 355-360
```tablegen
 355:   let assemblyFormat = [{
 356:     $alloc `,` $phase (`,` $pred^)? (`deps` $deps^)?
 357:     attr-dict `:` qualified(type($alloc)) (`,` type($deps)^)?
 358:   }];
 359:   let hasVerifier = 1;
 360: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 362-370
```tablegen
 362: def TTNG_ArriveBarrierOp : TTNG_Op<"arrive_barrier", [
 363:     DeclareOpInterfaceMethods<MBarrierOpInterface, ["getBarrier"]>,
 364:     DeclareOpInterfaceMethods<PredicatedOpInterface>]> {
 365:   let summary = "perform the arrive operation on an mbarrier";
 366:   let description = [{
 367:     The `ttng.arrive_barrier` operation performs the "arrive" operation on an
 368:     mbarrier object in shared memory. The operation requires a `count` attribute
 369:     of at least 1, and decreasing the pending arrival count of the mbarrier by
 370:     the specific count.
```
**EN:** This TableGen def record defines `TTNG_ArriveBarrierOp` with the summary “perform the arrive operation on an mbarrier”. It is specialized from `TTNG_Op<"arrive_barrier", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_ArriveBarrierOp`，其摘要为“perform the arrive operation on an mbarrier”。 它基于 `TTNG_Op<"arrive_barrier", [` 进一步特化。

### Lines 372-372
```tablegen
 372:     The operation accepts an optional predicate.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 374-374
```tablegen
 374:     Example:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 376-380
```tablegen
 376:     ```mlir
 377:     ttng.arrive_barrier %barrier, 2 : !ttg.memdesc<1xi64, #shared, #smem, mutable>
 378:     ttng.arrive_barrier %barrier, 1, %pred : !ttg.memdesc<1xi64, #shared, #smem, mutable>
 379:     ```
 380:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 382-386
```tablegen
 382:   let arguments = (ins
 383:     Arg<TTG_MemDescType, "", [MemRead<SharedMemory>, MemWrite<SharedMemory>]>:$alloc,
 384:     I32Attr:$count,
 385:     Optional<I1>:$pred
 386:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 388-390
```tablegen
 388:   let assemblyFormat = [{
 389:     $alloc `,` $count (`,` $pred^)? attr-dict `:` qualified(type($alloc))
 390:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 392-396
```tablegen
 392:   let builders = [
 393:     OpBuilder<(ins "Value":$alloc, "uint32_t":$count), [{
 394:       return build($_builder, $_state, alloc, count, /*pred=*/Value());
 395:     }]>
 396:   ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 398-399
```tablegen
 398:   let hasVerifier = 1;
 399: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 401-409
```tablegen
 401: def TTNG_AsyncCopyMbarrierArriveOp : TTNG_Op<"async_copy_mbarrier_arrive", [
 402:     DeclareOpInterfaceMethods<MBarrierOpInterface>]> {
 403:   let summary = "arrive on mbarrier once all previously issued copies are completed";
 404:   let arguments = (ins
 405:     Arg<TTG_MemDescType, "", [MemWrite<SharedMemory>]>:$barrier,
 406:     UnitAttr:$noIncrement
 407:   );
 408:   let assemblyFormat = "$barrier attr-dict `:` qualified(type($barrier))";
 409: }
```
**EN:** This TableGen def record defines `TTNG_AsyncCopyMbarrierArriveOp` with the summary “arrive on mbarrier once all previously issued copies are completed”. It is specialized from `TTNG_Op<"async_copy_mbarrier_arrive", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_AsyncCopyMbarrierArriveOp`，其摘要为“arrive on mbarrier once all previously issued copies are completed”。 它基于 `TTNG_Op<"async_copy_mbarrier_arrive", [` 进一步特化。

### Lines 412-415
```tablegen
 412: def TTNG_AsyncTMACopyGlobalToLocalOp : TTNG_Op<"async_tma_copy_global_to_local", [
 413:     AttrSizedOperandSegments, DeclareOpInterfaceMethods<MBarrierOpInterface>,
 414:     DeclareOpInterfaceMethods<PredicatedOpInterface>, TMALoadLikeOpInterface]> {
 415:   let summary = "copy data based on descriptor from global memory to local memory asynchronously";
```
**EN:** This TableGen def record defines `TTNG_AsyncTMACopyGlobalToLocalOp` with the summary “copy data based on descriptor from global memory to local memory asynchronously”. It is specialized from `TTNG_Op<"async_tma_copy_global_to_local", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_AsyncTMACopyGlobalToLocalOp`，其摘要为“copy data based on descriptor from global memory to local memory asynchronously”。 它基于 `TTNG_Op<"async_tma_copy_global_to_local", [` 进一步特化。

### Lines 417-422
```tablegen
 417:   let description = [{
 418:     This operation copies data from global memory to local memory
 419:     asynchronously.  This is analogue to tt.load except the data are copied to
 420:     local memory pointed by the memory descriptor instead of a distributed
 421:     tensor. The data copied depends on the global memory descriptor pointed to
 422:     by `desc`.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 424-435
```tablegen
 424:     The tensor mode is determined by the descriptor type:
 425:     - tt.tensordesc: TILED mode - Regular tiled tensor memory access
 426:       - See: https://docs.nvidia.com/cuda/parallel-thread-execution/#tensor-tiled-mode
 427:     - ttng.tensordesc_im2col: IM2COL mode - Im2col mode for convolution-friendly access patterns
 428:       - In IM2COL mode, 'coord' is the coordinates in the input tensor
 429:         - For example, for a 4D tensor (NHWC), 'coord' is [batch_idx, channel_idx, h, w]
 430:       - In IM2COL mode, additional `offsets` must be provided (uint16 values)
 431:         - For 3D tensors (NWC): 1 offset (offset_w)
 432:         - For 4D tensors (NHWC): 2 offsets (offset_w, offset_h)
 433:         - For 5D tensors (NDHWC): 3 offsets (offset_w, offset_h, offset_d)
 434:         - General rule: number of offsets = coord.size() - 2
 435:       - See: https://docs.nvidia.com/cuda/parallel-thread-execution/#tensor-im2col-mode
```
**EN:** This block declares or defines callable APIs such as tensor, provided, tensors, offset, offsets, and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 tensor, provided, tensors, offset, offsets, and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 437-437
```tablegen
 437:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 439-451
```tablegen
 439:   let hasVerifier = 1;
 440:   let arguments = (ins
 441:     Arg<TT_AnyTensorDescType, "", [MemRead<GlobalMemory>]>:$desc,
 442:     Variadic<I32>:$coord,
 443:     Variadic<I16>:$offsets,
 444:     Arg<TTG_MemDescType, "", [MemWrite<SharedMemory>]>:$barrier,
 445:     Arg<TTG_MemDescType, "", [MemWrite<SharedMemory>]>:$result,
 446:     I1:$pred,
 447:     UnitAttr:$multicast,
 448:     DefaultValuedAttr<TT_CacheModifierAttr, "triton::CacheModifier::NONE">:$cache,
 449:     DefaultValuedAttr<TT_EvictionPolicyAttr, "triton::EvictionPolicy::NORMAL">:$evict,
 450:     DefaultValuedAttr<BoolAttr, "false">:$isVolatile
 451:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 453-467
```tablegen
 453:   let builders = [
 454:     // Builder for TILED mode (no offsets required, attributes default to standard values)
 455:     OpBuilder<(ins "Value":$desc, "ValueRange":$coord, "Value":$barrier,
 456:                    "Value":$result, "Value":$pred,
 457:                    CArg<"bool", "false">:$multicast,
 458:                    CArg<"triton::CacheModifier", "triton::CacheModifier::NONE">:$cache,
 459:                    CArg<"triton::EvictionPolicy", "triton::EvictionPolicy::NORMAL">:$evict,
 460:                    CArg<"bool", "false">:$isVolatile), [{
 461:       multicast &= ::mlir::triton::nvidia_gpu::hasCGABroadcast(
 462:                                    ::mlir::cast<::mlir::triton::gpu::MemDescType>(
 463:                                        result.getType()));
 464:       build($_builder, $_state, desc, coord, /*offsets=*/ValueRange{}, barrier,
 465:             result, pred, multicast, cache, evict, isVolatile);
 466:     }]>
 467:   ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 469-473
```tablegen
 469:   let assemblyFormat = [{
 470:     $desc `[` $coord `]` (`offsets` `=` `[` $offsets^ `]`)? $result `,` $barrier `,` $pred
 471:     oilist(`cacheModifier` `=` $cache | `evictionPolicy` `=` $evict)
 472:     attr-dict `:` qualified(type($desc)) `,` qualified(type($barrier)) `->` qualified(type($result))
 473:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 475-475
```tablegen
 475: }
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 477-479
```tablegen
 477: def TTNG_AsyncTMACopyLocalToGlobalOp : TTNG_Op<"async_tma_copy_local_to_global", [
 478:     TMAStoreLikeOpInterface]> {
 479:   let summary = "copy data based on descriptor from local memory to global memory asynchronously";
```
**EN:** This TableGen def record defines `TTNG_AsyncTMACopyLocalToGlobalOp` with the summary “copy data based on descriptor from local memory to global memory asynchronously”. It is specialized from `TTNG_Op<"async_tma_copy_local_to_global", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_AsyncTMACopyLocalToGlobalOp`，其摘要为“copy data based on descriptor from local memory to global memory asynchronously”。 它基于 `TTNG_Op<"async_tma_copy_local_to_global", [` 进一步特化。

### Lines 481-487
```tablegen
 481:   let description = [{
 482:     This operation copies data from local memory to global memory
 483:     asynchronously.  This is analogue to tt.store except the data are copied from
 484:     local memory pointed by the memory descriptor instead of a distributed
 485:     tensor. The data copied depends on the global memory descriptor pointed to
 486:     by `desc`.
 487:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 489-493
```tablegen
 489:   let arguments = (ins
 490:     Arg<TT_TensorDescType, "", [MemRead<GlobalMemory>, MemWrite<GlobalMemory>]>:$desc,
 491:     Variadic<I32>:$coord,
 492:     Arg<TTG_MemDescType, "", [MemRead<SharedMemory>]>:$src
 493:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 495-500
```tablegen
 495:   let assemblyFormat = [{
 496:     $desc `[` $coord `]` $src
 497:     attr-dict `:` qualified(type($desc)) `,` qualified(type($src))
 498:   }];
 499:   let hasVerifier = 1;
 500: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 502-503
```tablegen
 502: def TTNG_AsyncTMAReduceOp : TTNG_Op<"async_tma_reduce", [TMAStoreLikeOpInterface]> {
 503:   let summary = "reduce result in gmem based on a TMA descriptor";
```
**EN:** This TableGen def record defines `TTNG_AsyncTMAReduceOp` with the summary “reduce result in gmem based on a TMA descriptor”. It is specialized from `TTNG_Op<"async_tma_reduce", [TMAStoreLikeOpInterface]>`.
**CN:** 该 TableGen def 记录定义了 `TTNG_AsyncTMAReduceOp`，其摘要为“reduce result in gmem based on a TMA descriptor”。 它基于 `TTNG_Op<"async_tma_reduce", [TMAStoreLikeOpInterface]>` 进一步特化。

### Lines 505-510
```tablegen
 505:   let description = [{
 506:     This operation copies data from local memory to global memory
 507:     asynchronously, and atomically performs the specified reduction kind.
 508:     Atomicity is at the granularity of individual elements, and only relaxed
 509:     semantics are implied.
 510:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 512-517
```tablegen
 512:   let arguments = (ins
 513:     TT_DescriptorReduceKindAttr:$kind,
 514:     Arg<TT_TensorDescType, "", [MemRead<GlobalMemory>, MemWrite<GlobalMemory>]>:$desc,
 515:     Variadic<I32>:$coord,
 516:     Arg<TTG_MemDescType, "", [MemRead<SharedMemory>]>:$src
 517:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 519-528
```tablegen
 519:   let assemblyFormat = [{
 520:     $kind `,` $desc `[` $coord `]` $src
 521:     attr-dict `:` qualified(type($desc)) `,` qualified(type($src))
 522:   }];
 523:   let extraClassDeclaration = [{
 524:     static bool isSupportedReduceKind(::mlir::triton::DescriptorReduceKind kind,
 525:                                       ::mlir::Type elementType);
 526:   }];
 527:   let hasVerifier = 1;
 528: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 530-533
```tablegen
 530: def TTNG_AsyncTMAGatherOp : TTNG_Op<"async_tma_gather", [
 531:     DeclareOpInterfaceMethods<MBarrierOpInterface>,
 532:     DeclareOpInterfaceMethods<PredicatedOpInterface>, TMALoadLikeOpInterface]> {
 533:   let summary = "gather data based on descriptor from global memory to local memory asynchronously";
```
**EN:** This TableGen def record defines `TTNG_AsyncTMAGatherOp` with the summary “gather data based on descriptor from global memory to local memory asynchronously”. It is specialized from `TTNG_Op<"async_tma_gather", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_AsyncTMAGatherOp`，其摘要为“gather data based on descriptor from global memory to local memory asynchronously”。 它基于 `TTNG_Op<"async_tma_gather", [` 进一步特化。

### Lines 535-539
```tablegen
 535:   let description = [{
 536:     This operation gathers multiple rows of data from global memory matrix to
 537:     local memory asynchronously.  This is similar to
 538:     async_tma_copy_global_to_local except that each row is indexed independently.
 539:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 541-549
```tablegen
 541:   let arguments = (ins
 542:     Arg<TT_TensorDescType, "", [MemRead<GlobalMemory>]>:$desc,
 543:     RankedTensorOf<[I32]>:$x_offsets,
 544:     I32:$y_offset,
 545:     Arg<TTG_MemDescType, "", [MemWrite<SharedMemory>]>:$barrier,
 546:     Arg<TTG_MemDescType, "", [MemWrite<SharedMemory>]>:$result,
 547:     I1:$pred,
 548:     UnitAttr:$multicast
 549:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 551-554
```tablegen
 551:   let assemblyFormat = [{
 552:     $desc `[` $x_offsets `,` $y_offset `]` $result `,` $barrier `,` $pred
 553:     attr-dict `:` type(operands)
 554:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 556-557
```tablegen
 556:   let hasVerifier = 1;
 557: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 559-561
```tablegen
 559: def TTNG_AsyncTMAScatterOp : TTNG_Op<"async_tma_scatter", [
 560:     TMAStoreLikeOpInterface]> {
 561:   let summary = "scatter data from local memory into global memory based on a descriptor asynchronously";
```
**EN:** This TableGen def record defines `TTNG_AsyncTMAScatterOp` with the summary “scatter data from local memory into global memory based on a descriptor asynchronously”. It is specialized from `TTNG_Op<"async_tma_scatter", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_AsyncTMAScatterOp`，其摘要为“scatter data from local memory into global memory based on a descriptor asynchronously”。 它基于 `TTNG_Op<"async_tma_scatter", [` 进一步特化。

### Lines 563-569
```tablegen
 563:   let description = [{
 564:     The `ttng.async_tma_scatter` operation scatters multiple separately-indexed
 565:     rows of data from local memory into global memory asynchronously. The
 566:     operation scatters a 2D tensor in shared memory, laid out by core tensor
 567:     tiles nvmma_shared layout into separately indexed rows in global
 568:     memory at a given `y` offset.
 569:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 571-576
```tablegen
 571:   let arguments = (ins
 572:     Arg<TT_TensorDescType, "", [MemRead<GlobalMemory>, MemWrite<GlobalMemory>]>:$desc,
 573:     RankedTensorOf<[I32]>:$x_offsets,
 574:     I32:$y_offset,
 575:     Arg<TTG_MemDescType, "", [MemRead<SharedMemory>]>:$src
 576:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 578-581
```tablegen
 578:   let assemblyFormat = [{
 579:     $desc `[` $x_offsets `,` $y_offset `]` $src
 580:     attr-dict `:` type(operands)
 581:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 583-584
```tablegen
 583:   let hasVerifier = 1;
 584: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 586-592
```tablegen
 586: def TTNG_TMAStoreWaitOp : TTNG_Op<"async_tma_store_wait", [MemWaitOpTrait]> {
 587:   let summary = "wait until all the inputs are read.";
 588:   let arguments = (ins I32Attr:$pendings);
 589:   let description = [{
 590:     Wait until all the read operations are done from the associated store operations.
 591:     This is needed before the shared memory can be written to.
 592:   }];
```
**EN:** This TableGen def record defines `TTNG_TMAStoreWaitOp` with the summary “wait until all the inputs are read.”. It is specialized from `TTNG_Op<"async_tma_store_wait", [MemWaitOpTrait]>`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TMAStoreWaitOp`，其摘要为“wait until all the inputs are read.”。 它基于 `TTNG_Op<"async_tma_store_wait", [MemWaitOpTrait]>` 进一步特化。

### Lines 594-595
```tablegen
 594:   let assemblyFormat = "attr-dict";
 595: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 597-605
```tablegen
 597: def TTNG_TCGen5MMAOp : TTNG_Op<"tc_gen5_mma", [
 598:     DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
 599:     DeclareOpInterfaceMethods<DotOpInterface>,
 600:     DeclareOpInterfaceMethods<MMAv5OpInterface>,
 601:     DeclareOpInterfaceMethods<PredicatedOpInterface>,
 602:     DeclareOpInterfaceMethods<MBarrierOpInterface>,
 603:     AttrSizedOperandSegments
 604: ]> {
 605:   let summary = "block level op mapping to tensorcore gen5 mma";
```
**EN:** This TableGen def record defines `TTNG_TCGen5MMAOp` with the summary “block level op mapping to tensorcore gen5 mma”. It is specialized from `TTNG_Op<"tc_gen5_mma", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TCGen5MMAOp`，其摘要为“block level op mapping to tensorcore gen5 mma”。 它基于 `TTNG_Op<"tc_gen5_mma", [` 进一步特化。

### Lines 607-612
```tablegen
 607:   let description = [{
 608:     $d += matrix_multiply($a, $b).
 609:     if is_async is false, the op executes synchronously. The barrier operands must not be present in that case.
 610:     Otherwise, if a barrier is given, the op will trigger a commit/arrive on it. The result will be safe to read after a barrier wait.
 611:     If $two_ctas is set the op will execute a matmul across two contiguous CTAs, it will read the data distributed across the two CTAs.
 612:     and syncronize both CTAs if the op is synchronous.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 614-616
```tablegen
 614:     This operation takes and produces an optional token to indicate TMEM read
 615:     and write on its accumulator operand. When the tokens are present, they can
 616:     be used to check aliasing and modref on the accumulator memory.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 618-620
```tablegen
 618:     The `isUnsigned` attribute is only relevant when performing an integer MMA operation.
 619:     If true, the integer values are treated as unsigned, otherwise they are treated as signed.
 620:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 622-636
```tablegen
 622:   let arguments = (ins
 623:     TTG_MemDescType:$a,
 624:     TTG_MemDescType:$b,
 625:     TTG_MemDescType:$d,
 626:     Optional<TTG_AsyncToken>:$acc_dep,
 627:     I1:$useD,
 628:     I1:$pred,
 629:     Variadic<TTG_MemDescType>:$barriers,
 630:     Variadic<I1>:$barrier_preds,
 631:     UnitAttr:$is_async,
 632:     UnitAttr:$two_ctas,
 633:     UnitAttr:$multicast,
 634:     UnitAttr:$is_unsigned
 635:   );
 636:   let results = (outs Optional<TTG_AsyncToken>:$token);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 638-647
```tablegen
 638:   let builders = [
 639:     OpBuilder<(ins "Type":$token,
 640:       "Value":$a, "Value":$b, "Value":$d, "Value":$acc_dep, "Value":$useD,
 641:       "Value":$pred, CArg<"bool", "false">:$two_ctas,
 642:       CArg<"bool", "false">:$multicast,
 643:       CArg<"ValueRange", "{}">:$barriers,
 644:       CArg<"ValueRange", "{}">:$barrier_preds,
 645:       CArg<"bool", "false">:$is_async,
 646:       CArg<"bool", "false">:$is_unsigned)>
 647:   ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 649-654
```tablegen
 649:   let assemblyFormat = [{
 650:     $a `,` $b `,` $d `` custom<Token>($acc_dep, type($token)) `,` $useD`,`
 651:     $pred `` custom<BarriersAndPreds>($barriers, $barrier_preds)
 652:     attr-dict `:` qualified(type($a)) `,` qualified(type($b)) `,`
 653:     qualified(type($d)) (`,` qualified(type($barriers))^)?
 654:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 656-657
```tablegen
 656:   let hasVerifier = 1;
 657: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 659-667
```tablegen
 659: def TTNG_TCGen5MMAScaledOp : TTNG_Op<"tc_gen5_mma_scaled", [
 660:     DeclareOpInterfaceMethods<MemoryEffectsOpInterface>,
 661:     DeclareOpInterfaceMethods<DotOpInterface, ["verifyDims", "verifyOutputDims"]>,
 662:     DeclareOpInterfaceMethods<MMAv5OpInterface>,
 663:     DeclareOpInterfaceMethods<PredicatedOpInterface>,
 664:     DeclareOpInterfaceMethods<MBarrierOpInterface>,
 665:     AttrSizedOperandSegments
 666: ]> {
 667:   let summary = "block level op mapping to tensorcore gen5 mma";
```
**EN:** This TableGen def record defines `TTNG_TCGen5MMAScaledOp` with the summary “block level op mapping to tensorcore gen5 mma”. It is specialized from `TTNG_Op<"tc_gen5_mma_scaled", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TCGen5MMAScaledOp`，其摘要为“block level op mapping to tensorcore gen5 mma”。 它基于 `TTNG_Op<"tc_gen5_mma_scaled", [` 进一步特化。

### Lines 669-675
```tablegen
 669:   let description = [{
 670:     $d += matrix_multiply(scale($lhs, $lhs_scale), scale(rlhs, $rhs_scale))
 671:     If $two_ctas is set the op will execute a matmul across two contiguous CTAs, it will read the data distributed across the two CTAs
 672:     and synchronize both CTAs if the op is synchronous.
 673:     If is_async is false, the op executes synchronously. The barrier operands must not be present in that case.
 674:     Otherwise, if a barrier is given, the op will trigger a commit/arrive on it.
 675:     The result will be safe to read after a barrier wait.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 677-680
```tablegen
 677:     This operation takes and produces an optional token to indicate TMEM read
 678:     and write on its accumulator operand. When the tokens are present, they can
 679:     be used to check aliasing and modref on the accumulator memory.
 680:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 682-699
```tablegen
 682:   let arguments = (ins
 683:     TTG_MemDescType:$a,
 684:     TTG_MemDescType:$b,
 685:     TTG_MemDescType:$d,
 686:     Optional<TTG_AsyncToken>:$acc_dep,
 687:     TTG_MemDescType:$a_scale,
 688:     TTG_MemDescType:$b_scale,
 689:     TT_ScaleDotElemTypeAttr:$a_type,
 690:     TT_ScaleDotElemTypeAttr:$b_type,
 691:     I1:$useD,
 692:     I1:$pred,
 693:     Variadic<TTG_MemDescType>:$barriers,
 694:     Variadic<I1>:$barrier_preds,
 695:     UnitAttr:$two_ctas,
 696:     UnitAttr:$multicast,
 697:     UnitAttr:$is_async
 698:   );
 699:   let results = (outs Optional<TTG_AsyncToken>:$token);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 701-705
```tablegen
 701:   let extraClassDeclaration = [{
 702:     int64_t getBlockM();
 703:     int64_t getBlockN();
 704:     int64_t getBlockK();
 705:   }];
```
**EN:** This block injects extra C++ helpers into the generated class, such as getBlockM, getBlockN, and getBlockK.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getBlockM, getBlockN, and getBlockK。

### Lines 707-721
```tablegen
 707:   let builders = [
 708:     // Namespaces need to be prefixed so ODS prefers our
 709:     // custom builder signature over the default-generated one.
 710:     OpBuilder<(ins "::mlir::Type":$token,
 711:       "::mlir::Value":$a, "::mlir::Value":$b, "::mlir::Value":$d,
 712:       "::mlir::Value":$acc_dep, "::mlir::Value":$a_scale,
 713:       "::mlir::Value":$b_scale, "::mlir::triton::ScaleDotElemType":$a_type,
 714:       "::mlir::triton::ScaleDotElemType":$b_type,
 715:       "::mlir::Value":$useD, "::mlir::Value":$pred,
 716:       CArg<"::mlir::ValueRange", "{}">:$barriers,
 717:       CArg<"::mlir::ValueRange", "{}">:$barrier_preds,
 718:       CArg<"bool", "false">:$two_ctas,
 719:       CArg<"bool", "false">:$is_async,
 720:       CArg<"bool", "false">:$multicast)>
 721:   ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 723-730
```tablegen
 723:   let assemblyFormat = [{
 724:     $a `,` $b `,` $d `` custom<Token>($acc_dep, type($token)) `,` $a_scale `,`
 725:     $b_scale `,` $useD `,` $pred `lhs` `=` $a_type `rhs` `=` $b_type
 726:     `` custom<BarriersAndPreds>($barriers, $barrier_preds)
 727:     attr-dict `:` qualified(type($a)) `,` qualified(type($b)) `,`
 728:     qualified(type($d)) `,` qualified(type($a_scale)) `,`
 729:     qualified(type($b_scale)) (`,` qualified(type($barriers))^)?
 730:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 732-733
```tablegen
 732:   let hasVerifier = 1;
 733: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 735-738
```tablegen
 735: def TTNG_TCGen5CommitOp : TTNG_Op<"tc_gen5_commit", [AttrSizedOperandSegments,
 736:     DeclareOpInterfaceMethods<MBarrierOpInterface>,
 737:     DeclareOpInterfaceMethods<PredicatedOpInterface>]> {
 738:   let summary = "make an mbarrier track completion of all prior async tcgen5 ops";
```
**EN:** This TableGen def record defines `TTNG_TCGen5CommitOp` with the summary “make an mbarrier track completion of all prior async tcgen5 ops”. It is specialized from `TTNG_Op<"tc_gen5_commit", [AttrSizedOperandSegments,`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TCGen5CommitOp`，其摘要为“make an mbarrier track completion of all prior async tcgen5 ops”。 它基于 `TTNG_Op<"tc_gen5_commit", [AttrSizedOperandSegments,` 进一步特化。

### Lines 740-744
```tablegen
 740:   let description = [{
 741:     The `ttng.tc_gen5_commit` is an asynchronous operation that makes the
 742:     mbarrier object track the completion of all prior asynchronous tcgen5
 743:     operations. Upon completion of all asynchronous operations, the mbarrier
 744:     arrive operation is performed on the mbarrier with a count of 1.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 746-749
```tablegen
 746:     If `descs` are provided, the commit will be multicast across the CTA cluster
 747:     based on the shared layouts of those descriptors. This should be used when
 748:     the inputs to the tcgen5 MMA, including scaled-MMA scale inputs, come from
 749:     TMA descriptors using multicast.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 751-752
```tablegen
 751:     Note that the completion mechanisms are guaranteed to occur sequentially in
 752:     the order the commit operations were issued. This means, for example:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 754-759
```tablegen
 754:     ```mlir
 755:     ttng.tmem_copy
 756:     ttng.tc_gen5_mma
 757:     ttng.tc_gen5_commit %barrierA
 758:     ttng.tc_gen5_commit %barrierB
 759:     ```
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 761-765
```tablegen
 761:     `%barrierA` tracks the completion of the previous TMEM copy and MMA
 762:     operations, but since the commit groups are sequential, the arrive-on
 763:     operation on `%barrierA` is guaranteed to be performed before the arrive-on
 764:     operation on `%barrierB`, even though its commit group is empty.
 765:   }];
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 767-771
```tablegen
 767:   let arguments = (ins
 768:     Arg<TTG_MemDescType, "", [MemWrite<SharedMemory>]>:$barrier,
 769:     Optional<I1>:$pred,
 770:     Variadic<TTG_MemDescType>:$descs
 771:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 773-776
```tablegen
 773:   let assemblyFormat = [{
 774:     $barrier (`,` $pred^)? (`descs` $descs^)? attr-dict `:`
 775:     qualified(type($barrier)) (`,` qualified(type($descs))^)?
 776:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 778-779
```tablegen
 778:   let hasVerifier = 1;
 779: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 781-782
```tablegen
 781: def TTNG_TMEMLoadOp : TTNG_Op<"tmem_load", [AttrSizedResultSegments]> {
 782:   let summary = "Load a buffer from tensor memory into a distributed tensor";
```
**EN:** This TableGen def record defines `TTNG_TMEMLoadOp` with the summary “Load a buffer from tensor memory into a distributed tensor”. It is specialized from `TTNG_Op<"tmem_load", [AttrSizedResultSegments]>`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TMEMLoadOp`，其摘要为“Load a buffer from tensor memory into a distributed tensor”。 它基于 `TTNG_Op<"tmem_load", [AttrSizedResultSegments]>` 进一步特化。

### Lines 784-786
```tablegen
 784:   let description = [{
 785:     This is similar to ttg.local_load except the result layout is restricted to only few possibility.
 786:     Therefore we cannot combine this op with any convert layout like local_load.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 788-790
```tablegen
 788:     This operation takes and produces an optional token to indicate TMEM read
 789:     on its source operand. When the tokens are present, they can
 790:     be used to check aliasing and modref on the TMEM buffer.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 792-797
```tablegen
 792:     Optional reduction modifier:
 793:     When `redOp` is specified, the load operation additionally performs an
 794:     element-wise reduction along the N-dimension of the input and produces a
 795:     second result tensor `red`. For a input of shape `[M, N]`, the
 796:     reduced result has shape `[M]`, containing one reduced value per "slice"
 797:     of the N-dimension.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 799-799
```tablegen
 799:     Currently restricted to f32 element type.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 801-808
```tablegen
 801:     - redOp: Specifies the reduction operation (MIN or MAX) to apply along
 802:              the N-dimension. When set, the `red` result must be present.
 803:     - abs:   When true, applies absolute value to each element before performing
 804:              the reduction. Only valid when `redOp` is specified.
 805:     - NaN:   When true, the reduction propagates NaN values (if any input element
 806:              in a slice is NaN, the corresponding reduced value is NaN).
 807:              When false, NaN values are ignored during reduction.
 808:              Only valid when `redOp` is specified.
```
**EN:** This block declares or defines callable APIs such as operation and values, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 operation and values 等可调用 API，用来封装这里提供的核心行为。

### Lines 810-813
```tablegen
 810:     Example:
 811:       Input in TMEM of shape[M=2, N=4]:
 812:         [[ 1.0, 3.0, 2.0, 4.0],
 813:          [-5.0, 1.0, 8.0, 2.0]]
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 815-818
```tablegen
 815:       With redOp=MAX:
 816:         result = [[ 1.0, 3.0, 2.0, 4.0],   // unchanged
 817:                   [-5.0, 1.0, 8.0, 2.0]]
 818:         red    = [4.0, 8.0]               // max along N per row
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 820-821
```tablegen
 820:       With redOp=MIN, abs=true:
 821:         red    = [1.0, 1.0]               // min of |values| per row
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 823-837
```tablegen
 823:     This operation lowers to hardware-accelerated reduction via the PTX
 824:     tcgen05.ld.red instruction on supported architectures, e.g. Blackwell Ultra.
 825:   }];
 826:   let arguments = (ins
 827:     Arg<TTG_MemDescType, "", [MemRead<TensorMemory>]>:$src,
 828:     Optional<TTG_AsyncToken>:$dep,
 829:     OptionalAttr<TTNG_TMEMLoadReduceModifierEnum>:$redOp,
 830:     OptionalAttr<BoolAttr>:$abs,
 831:     OptionalAttr<BoolAttr>:$NaN
 832:   );
 833:   let results = (outs
 834:     TT_Tensor:$result,
 835:     Optional<TTG_AsyncToken>:$token,
 836:     Optional<TT_Tensor>:$red
 837:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 839-842
```tablegen
 839:   let assemblyFormat = [{
 840:     $src `` custom<Token>($dep, type($token))
 841:     attr-dict `:` qualified(type($src)) `->` type($result) (`,` type($red)^)?
 842:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 844-871
```tablegen
 844:   let builders = [
 845:     // Basic builder: result type, optional token type, src, optional dep
 846:     OpBuilder<(ins "Type":$result, "Type":$token, "Value":$src, "Value":$dep), [{
 847:       build($_builder, $_state, result, token, /*red=*/Type(), src, dep,
 848:             /*redOp=*/nullptr, /*abs=*/nullptr, /*NaN=*/nullptr);
 849:     }]>,
 850:     // Builder without token
 851:     OpBuilder<(ins "Type":$result, "Value":$src), [{
 852:       build($_builder, $_state, result, /*token=*/Type(), /*red=*/Type(), src,
 853:             /*dep=*/Value(), /*redOp=*/nullptr, /*abs=*/nullptr, /*NaN=*/nullptr);
 854:     }]>,
 855:     // Builder with reduction - infers red type from result type
 856:     OpBuilder<(ins "Type":$result, "Type":$token, "Value":$src, "Value":$dep,
 857:                "::mlir::triton::nvidia_gpu::TMEMLoadReduceModifierAttr":$redOp,
 858:                "BoolAttr":$abs, "BoolAttr":$NaN), [{
 859:       Type redTy;
 860:       if (redOp) {
 861:         auto tensorTy = ::mlir::cast<RankedTensorType>(result);
 862:         SmallVector<int64_t> redShape = {tensorTy.getShape()[0]};
 863:         auto parentEnc = ::mlir::cast<::mlir::triton::gpu::DistributedEncodingTrait>(
 864:             tensorTy.getEncoding());
 865:         auto sliceEnc = ::mlir::triton::gpu::SliceEncodingAttr::get(
 866:             $_builder.getContext(), 1, parentEnc);
 867:         redTy = RankedTensorType::get(redShape, tensorTy.getElementType(), sliceEnc);
 868:       }
 869:       build($_builder, $_state, result, token, redTy, src, dep, redOp, abs, NaN);
 870:     }]>,
 871:   ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 873-873
```tablegen
 873:   let hasVerifier = 1;
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 875-879
```tablegen
 875:   let extraClassDeclaration = [{
 876:     RankedTensorType getType() { return getResult().getType(); }
 877:     operator TypedValue<RankedTensorType>() { return getResult(); }
 878:   }];
 879: }
```
**EN:** This block injects extra C++ helpers into the generated class, such as getType and getResult.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getType and getResult。

### Lines 881-883
```tablegen
 881: def TTNG_TMEMStoreOp : TTNG_Op<"tmem_store", [
 882:     DeclareOpInterfaceMethods<PredicatedOpInterface>]> {
 883:   let summary = "Store a distributed tensor into a buffer in tensor memory";
```
**EN:** This TableGen def record defines `TTNG_TMEMStoreOp` with the summary “Store a distributed tensor into a buffer in tensor memory”. It is specialized from `TTNG_Op<"tmem_store", [`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TMEMStoreOp`，其摘要为“Store a distributed tensor into a buffer in tensor memory”。 它基于 `TTNG_Op<"tmem_store", [` 进一步特化。

### Lines 885-886
```tablegen
 885:   let description = [{
 886:     This is similar to ttg.local_store except the source layout is restricted to only few possibility.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 888-898
```tablegen
 888:     This operation takes and produces an optional token to indicate TMEM write
 889:     on its source operand. When the tokens are present, they can
 890:     be used to check aliasing and modref on the TMEM buffer.
 891:   }];
 892:   let arguments = (ins
 893:     Arg<TTG_MemDescType, "", [MemWrite<TensorMemory>]>:$dst,
 894:     Optional<TTG_AsyncToken>:$dep,
 895:     TT_Tensor:$src,
 896:     I1:$pred
 897:   );
 898:   let results = (outs Optional<TTG_AsyncToken>:$token);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 900-904
```tablegen
 900:   let builders = [
 901:     OpBuilder<(ins "Value":$dst, "Value":$src, "Value":$pred), [{
 902:       build($_builder, $_state, Type(), dst, Value(), src, pred);
 903:     }]>
 904:   ];
```
**EN:** This block adds convenience builders so the generated API can construct the entity with common argument patterns.
**CN:** 该代码块添加便捷 builder，使生成 API 能以常见参数组合快速构造该实体。

### Lines 906-911
```tablegen
 906:   let assemblyFormat = [{
 907:     $src `,` $dst `` custom<Token>($dep, type($token)) `,` $pred
 908:     attr-dict `:` type($src) `->` qualified(type($dst))
 909:   }];
 910:   let hasVerifier = 1;
 911: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 913-918
```tablegen
 913: def TTNG_TMEMAllocOp : TTNG_Op<"tmem_alloc", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]> {
 914:   let summary = "allocate tensor memory";
 915:   let description = [{
 916:     This operation allocates buffer in tensor memory and return a descriptor
 917:     containing the address and a view of the buffer.
 918:     This is similar to ttg.local_alloc except the buffer is allocated in tensor memory.
```
**EN:** This TableGen def record defines `TTNG_TMEMAllocOp` with the summary “allocate tensor memory”. It is specialized from `TTNG_Op<"tmem_alloc", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]>`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TMEMAllocOp`，其摘要为“allocate tensor memory”。 它基于 `TTNG_Op<"tmem_alloc", [DeclareOpInterfaceMethods<MemoryEffectsOpInterface>]>` 进一步特化。

### Lines 920-926
```tablegen
 920:     Explicitly deallocating a buffer is optional; see local_dealloc.
 921:   }];
 922:   let arguments = (ins Optional<TT_Tensor>:$src);
 923:   let results = (outs
 924:     TTG_MemDescType:$result,
 925:     Optional<TTG_AsyncToken>:$token
 926:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 928-930
```tablegen
 928:   let assemblyFormat = [{
 929:     ($src^)? attr-dict `:` functional-type(operands, results)
 930:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 932-932
```tablegen
 932:   let hasVerifier = 1;
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 934-938
```tablegen
 934:   let extraClassDeclaration = [{
 935:     triton::gpu::MemDescType getType() { return getResult().getType(); }
 936:     operator TypedValue<triton::gpu::MemDescType>() { return getResult(); }
 937:   }];
 938: }
```
**EN:** This block injects extra C++ helpers into the generated class, such as getType and getResult.
**CN:** 该代码块向生成类中注入额外的 C++ 辅助方法，例如 getType and getResult。

### Lines 940-949
```tablegen
 940: def TTNG_TMEMSubSliceOp : TTNG_Op<"tmem_subslice", [Pure,
 941:                                                     MemDescViewTrait]> {
 942:   let summary = "Take a subslice of a tensor memory allocation";
 943:   let description = [{
 944:     This operation takes a subslice of a tensor memory allocation and returns a new descriptor
 945:     containing the address and a view of the subslice.
 946:     This is similar to ttg.memdesc_subslice except we can only slice along the inner dimension
 947:     of a 2D memdesc as this is the only one we can do for TMem.
 948:   }];
 949:   let arguments = (ins TTG_MemDescType:$src, I32Attr:$N);
```
**EN:** This TableGen def record defines `TTNG_TMEMSubSliceOp` with the summary “Take a subslice of a tensor memory allocation”. It is specialized from `TTNG_Op<"tmem_subslice", [Pure,`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TMEMSubSliceOp`，其摘要为“Take a subslice of a tensor memory allocation”。 它基于 `TTNG_Op<"tmem_subslice", [Pure,` 进一步特化。

### Lines 951-953
```tablegen
 951:   let assemblyFormat = [{
 952:     $src attr-dict `:` qualified(type($src)) `->` qualified(type($result))
 953:   }];
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 955-960
```tablegen
 955:   let builders = [
 956:       OpBuilder<(ins "Value":$alloc, "int":$offset, "int":$size)>,
 957:     ];
 958:   let results = (outs TTG_MemDescType:$result);
 959:   let hasVerifier = 1;
 960: }
```
**EN:** This block declares the result values produced by the generated operation or record.
**CN:** 该代码块声明生成操作或记录会产出的结果值。

### Lines 962-963
```tablegen
 962: def TTNG_TMEMCopyOp : TTNG_Op<"tmem_copy"> {
 963:   let summary = "Initiate an asynchronous copy operation from shared memory to the Tensor Memory.";
```
**EN:** This TableGen def record defines `TTNG_TMEMCopyOp` with the summary “Initiate an asynchronous copy operation from shared memory to the Tensor Memory.”. It is specialized from `TTNG_Op<"tmem_copy">`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TMEMCopyOp`，其摘要为“Initiate an asynchronous copy operation from shared memory to the Tensor Memory.”。 它基于 `TTNG_Op<"tmem_copy">` 进一步特化。

### Lines 965-968
```tablegen
 965:   let description = [{
 966:     2D blocks stored contiguously in SMEM are copied into TMEM as specified by the destination address.
 967:     This op lowers to the PTX instruction tcgen05.cp. This supports writing either to scales tmem layout as well as default tmem layout.
 968:     Currently the semantic is different when writing to tmem scale layout.
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 970-970
```tablegen
 970:     In case of default layout the copy doesn't change the logical elements between the source and destination memdesc.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 972-974
```tablegen
 972:     In case of scale layout:
 973:     Each 32x128b block in SMEM is duplicated over 4 warps and stored into 128 rows
 974:     and 4 columns of TMEM. The primary use case of this op is to copy blocked scales from SMEM to TMEM.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 976-985
```tablegen
 976:     The shape of the input SMEM can be flexibily chosen depending on use cases. In the simplest case (e.g. unit test),
 977:     the source SMEM can be of shape (32 x num_blocks, 16), and the destination TMEM should be of shape (128, 16 x num_blocks),
 978:     for copying 8 bit values. For scaled GEMM, rep_m x rep_k copies of a 32x128b block need to be stored in SMEM, where
 979:     rep_m = BLOCK_M / 128, rep_k = BLOCK_K / scale_vec_size / 4, and scale_vec_size = 32 for MXFP.
 980:     Conceptually, the SMEM is organized in a high-dimensional layout, (rep_m, rep_k, 32, 4, 4B).
 981:     Some of axes can be flattened into one, to reduce the rank of the load. For example, the following patterns are supported:
 982:      * (rep_m, rep_k * 32 x 4 x 4B), 2D scale load with cp.async
 983:      * (rep_m, rep_k, 32, 16B), 4D scale load with TMA
 984:      * (rep_m, rep_k, 32, 4, 4B), 5D scale load with cp.async
 985:     Since rep_m blocks are not contiguous in SMEM, this axis cannot be flattened into inner ones.
```
**EN:** This block declares or defines callable APIs such as case and shape, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 case and shape 等可调用 API，用来封装这里提供的核心行为。

### Lines 987-989
```tablegen
 987:     In Triton, the TMEM memdesc for blocked scales must be of the following form:
 988:     * Its shape must be (BLOCK_MN, BLOCK_K / scale_vec_size), representing the logical shape of blocked scales.
 989:     * It must be attached with `tensor_memory_scales_encoding` to indicate the chunk-based layout and its duplication over 4 warps.
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 991-991
```tablegen
 991:     In contrast, the src SMEM must be in the explicit chunk-based layout as described above. So the IR might look like this:
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 993-994
```tablegen
 993:     %0 = ttng.tmem_alloc : () -> !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>
 994:     ttng.tmem_copy %1, %0 : (!ttg.memdesc<1x1x32x4x4xi8, #shared1, #smem>, !ttg.memdesc<128x4xi8, #tmem_scales, #ttng.tensor_memory>) -> ()
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 996-1003
```tablegen
 996:     We interpret the semantics of this copy operation as follows. The chunk-based layout in SMEM implies that
 997:     the logical shape (BLOCK_MN, BLOCK_K / scale_vec_size) in TMEM is the result of certain reshape and transpose operations.
 998:     In practice, to take an advantage of the native scale layout and the TMEM copy op,  users need to do
 999:     `scales5D.trans(0, 3, 2, 1, 4).reshape(BLOCK_M, BLOCK_K // scale_vec_size)` before feeding scales into dot_scaled.
1000:     When we use tmem_copy in the IR, such reshape and transpose operations are removed. But the change in the logical shape they have caused on
1001:     registers is now understood to be incorporated into tmem_copy itself. Ideally, we would lift reshape / transpose done on registers onto
1002:     the SMEM memdesc, making tmem_copy a straightforward 2D copy operation: (BLOCK_MN, BLOCK_K / scale_vec_size) -> (BLOCK_MN, BLOCK_K / scale_vec_size).
1003:     In the absence of such operations on memdesc, we resort to implicitly encoding the reshape/transpose semantics in tmem_copy.
```
**EN:** This block declares or defines callable APIs such as shape, trans, reshape, and operation:, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 shape, trans, reshape, and operation: 等可调用 API，用来封装这里提供的核心行为。

### Lines 1005-1009
```tablegen
1005:   }];
1006:   let arguments = (ins
1007:     Arg<TTG_MemDescType, "", [MemRead<SharedMemory>]>:$src,
1008:     Arg<TTG_MemDescType, "", [MemWrite<TensorMemory>]>:$dst
1009:   );
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 1011-1013
```tablegen
1011:   let assemblyFormat = [{$src `,` $dst attr-dict `:` qualified(type(operands))}];
1012:   let hasVerifier = 1;
1013: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 1015-1016
```tablegen
1015: def TTNG_ReinterpretTensorDescOp : TTNG_Op<"reinterpret_tensor_descriptor", [Pure]> {
1016:   let summary = "Reinterpret a pointer as a tensor descriptor";
```
**EN:** This TableGen def record defines `TTNG_ReinterpretTensorDescOp` with the summary “Reinterpret a pointer as a tensor descriptor”. It is specialized from `TTNG_Op<"reinterpret_tensor_descriptor", [Pure]>`.
**CN:** 该 TableGen def 记录定义了 `TTNG_ReinterpretTensorDescOp`，其摘要为“Reinterpret a pointer as a tensor descriptor”。 它基于 `TTNG_Op<"reinterpret_tensor_descriptor", [Pure]>` 进一步特化。

### Lines 1018-1021
```tablegen
1018:   let description = [{
1019:      This Op exists to help the transition from untyped raw TMA objects to typed Tensor descriptor objects.
1020:      Ideally, we can remove this once the APIs are fully fleshed out.
1021:   }];
```
**EN:** This block provides the longer human-readable description that explains semantics, assumptions, or usage details for the generated entity.
**CN:** 该代码块提供更详细的人类可读描述，用来说明生成实体的语义、前提条件或使用方式。

### Lines 1023-1024
```tablegen
1023:   let arguments = (ins TT_Ptr:$rawDesc);
1024:   let results = (outs TT_TensorDescType:$result);
```
**EN:** This block declares the operands and attributes accepted by the generated operation or record, which defines its input contract.
**CN:** 该代码块声明生成操作或记录所接受的操作数和属性，从而确定输入契约。

### Lines 1026-1029
```tablegen
1026:   let assemblyFormat = [{
1027:     $rawDesc attr-dict `:` qualified(type($rawDesc))  `to` qualified(type($result))
1028:   }];
1029: }
```
**EN:** This block specifies the custom assembly syntax used when printing or parsing the operation in MLIR textual form.
**CN:** 该代码块指定了该操作在 MLIR 文本形式中的自定义汇编语法，用于打印和解析。

### Lines 1031-1064
```tablegen
1031: def TTNG_TensormapCreateOp: TTNG_Op<
1032:   "tensormap_create",
1033:   [
1034:     MemoryEffects<[MemRead<GlobalMemory>, MemWrite<GlobalMemory>]>,
1035:     AttrSizedOperandSegments,
1036:   ]
1037: > {
1038:   let summary = "Create a new TMA descriptor on device";
1039:   let arguments = (
1040:       ins
1041:       TT_PtrType:$desc_ptr,
1042:       TT_PtrType:$global_address,
1043:       Variadic<I32>:$box_dim,
1044:       Variadic<I32>:$global_dim,
1045:       Variadic<I64>:$global_stride,
1046:       Variadic<I32>:$element_stride,
1047:       ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<15>]>:$elem_type,
1048:       ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<2>]>:$interleave_layout,
1049:       ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<3>]>:$swizzle_mode,
1050:       ConfinedAttr<I32Attr, [IntNonNegative, IntMaxValue<1>]>:$fill_mode
1051:   );
1052:   let extraClassDeclaration = [{
1053:       int32_t getRank() {
1054:           return getBoxDim().size();
1055:       }
1056:   }];
1057:   let assemblyFormat = [{
1058:     $desc_ptr `,` $global_address `,`
1059:     `[` $box_dim `]` `,`
1060:     `[` $global_dim `]` `,`
1061:     `[` $global_stride `]` `,`
1062:     `[` $element_stride `]`
1063:     attr-dict `:` functional-type(operands, results)
1064:   }];
```
**EN:** This TableGen def record defines `TTNG_TensormapCreateOp` with the summary “Create a new TMA descriptor on device”. It is specialized from `TTNG_Op<`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TensormapCreateOp`，其摘要为“Create a new TMA descriptor on device”。 它基于 `TTNG_Op<` 进一步特化。

### Lines 1066-1067
```tablegen
1066:   let hasVerifier = 1;
1067: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 1069-1078
```tablegen
1069: def TTNG_TensormapFenceproxyAcquireOp: TTNG_Op<
1070:   "tensormap_fenceproxy_acquire",
1071:   [MemoryEffects<[MemWrite<GlobalMemory>]>]
1072: > {
1073:   let summary = "Acquire fence on a tensormap object";
1074:   let arguments = (ins TT_PtrType:$desc_ptr);
1075:   let assemblyFormat = [{
1076:     $desc_ptr attr-dict `:` qualified(type($desc_ptr))
1077:   }];
1078: }
```
**EN:** This TableGen def record defines `TTNG_TensormapFenceproxyAcquireOp` with the summary “Acquire fence on a tensormap object”. It is specialized from `TTNG_Op<`.
**CN:** 该 TableGen def 记录定义了 `TTNG_TensormapFenceproxyAcquireOp`，其摘要为“Acquire fence on a tensormap object”。 它基于 `TTNG_Op<` 进一步特化。

### Lines 1080-1080
```tablegen
1080: #endif
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** buffer modeling  
  **CN:** 缓冲区建模
- **EN:** warp-level execution  
  **CN:** warp 级执行
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** operation definitions  
  **CN:** 操作定义
- **EN:** interfaces  
  **CN:** 接口
- **EN:** attributes  
  **CN:** 属性
- **EN:** types  
  **CN:** 类型
- **EN:** traits  
  **CN:** 特征约束

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUDialect.td`
  - `triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUAttrDefs.td`
  - `triton/Dialect/TritonNvidiaGPU/IR/TritonNvidiaGPUOpInterfaces.td`
  - `mlir/Dialect/Arith/IR/ArithBase.td`
  - `triton/Dialect/Triton/IR/TritonTypes.td`
  - `triton/Dialect/Triton/IR/TritonAttrDefs.td`
  - `triton/Dialect/Triton/IR/TritonInterfaces.td`
  - `triton/Dialect/Triton/IR/TritonOpInterfaces.td`
  - `triton/Dialect/TritonGPU/IR/TritonGPUAttrDefs.td`
  - `triton/Dialect/TritonGPU/IR/TritonGPUTypes.td`
  - `triton/Dialect/TritonGPU/IR/TritonGPUTypeInterfaces.td`
  - `triton/Dialect/TritonGPU/IR/TritonGPUOpInterfaces.td`
  - `mlir/IR/OpBase.td`
  - `mlir/Interfaces/SideEffectInterfaces.td`
  - `mlir/Interfaces/InferTypeOpInterface.td`
  - `mlir/Interfaces/DestinationStyleOpInterface.td`
  - `mlir/Interfaces/ViewLikeInterface.td`
