# TritonInstrumentOps.td — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Dialect/TritonInstrument/IR/TritonInstrumentOps.td`
- **EN:** Defines TableGen operation records, summaries, operands, results, and assembly syntax.
- **CN:** 定义 TableGen 操作记录，以及摘要、操作数、结果和汇编语法。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```tablegen
   1: #ifndef TRITONINSTRUMENT_OPS
   2: #define TRITONINSTRUMENT_OPS
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-11
```tablegen
   4: include "triton/Dialect/TritonInstrument/IR/TritonInstrumentDialect.td"
   5: include "triton/Dialect/TritonGPU/IR/TritonGPUTypes.td"
   6: include "triton/Dialect/Triton/IR/TritonTypes.td"
   7: include "triton/Dialect/Triton/IR/TritonInterfaces.td"
   8: include "triton/Dialect/Triton/IR/TritonAttrDefs.td"
   9: include "mlir/IR/OpBase.td"
  10: include "mlir/Interfaces/SideEffectInterfaces.td"
  11: include "triton/Dialect/TritonInstrument/IR/TritonInstrumentAttrDefs.td"
```
**EN:** This block imports the direct dependencies needed here, including triton/Dialect/TritonInstrument/IR/TritonInstrumentDialect.td, triton/Dialect/TritonGPU/IR/TritonGPUTypes.td, triton/Dialect/Triton/IR/TritonTypes.td, triton/Dialect/Triton/IR/TritonInterfaces.td, triton/Dialect/Triton/IR/TritonAttrDefs.td, and mlir/IR/OpBase.td.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 triton/Dialect/TritonInstrument/IR/TritonInstrumentDialect.td, triton/Dialect/TritonGPU/IR/TritonGPUTypes.td, triton/Dialect/Triton/IR/TritonTypes.td, triton/Dialect/Triton/IR/TritonInterfaces.td, triton/Dialect/Triton/IR/TritonAttrDefs.td, and mlir/IR/OpBase.td。

### Lines 13-16
```tablegen
  13: //
  14: // Interfaces
  15: //
  16: def GlobalMemory : Resource<"::mlir::triton::GlobalMemory">;
```
**EN:** This TableGen def record defines `GlobalMemory`. It is specialized from `Resource<"::mlir::triton::GlobalMemory">;`.
**CN:** 该 TableGen def 记录定义了 `GlobalMemory`。 它基于 `Resource<"::mlir::triton::GlobalMemory">;` 进一步特化。

### Lines 18-20
```tablegen
  18: //
  19: // Ops
  20: //
```
**EN:** This comment block records the intent and constraints of the surrounding code: Ops.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 22-24
```tablegen
  22: class TTI_Op<string mnemonic, list<Trait> traits = []> :
  23:     Op<TritonInstrument_Dialect, mnemonic, traits> {
  24: }
```
**EN:** This block introduces `TTI_Op`, the main class/struct defined here. It also inherits behavior from a base type.
**CN:** 该代码块引入了此文件的核心类/结构体 `TTI_Op`。 它还通过继承复用基类能力。

### Lines 26-35
```tablegen
  26: def TTI_ExperimentalAssertUniformOp : TTI_Op<"experimental_assert_uniform", [MemoryEffects<[MemWrite<GlobalMemory>]>]> {
  27:   let summary = "assert the uniform condition";
  28:   let description = [{
  29:     Assert that the condition is true given all threads in the warp group have
  30:     the same value, so only one thread needs to evaluate the assert and print
  31:     the message.
  32:   }];
  33:   let arguments = (ins I1:$condition, StrAttr:$message);
  34:   let assemblyFormat = "$condition `,` $message attr-dict-with-keyword";
  35: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalAssertUniformOp` with the summary “assert the uniform condition”. It is specialized from `TTI_Op<"experimental_assert_uniform", [MemoryEffects<[MemWrite<GlobalMemory>]>]>`.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalAssertUniformOp`，其摘要为“assert the uniform condition”。 它基于 `TTI_Op<"experimental_assert_uniform", [MemoryEffects<[MemWrite<GlobalMemory>]>]>` 进一步特化。

### Lines 37-50
```tablegen
  37: def TTI_ExperimentalBufferDescriptorsOp
  38:     : TTI_Op<"experimental_buffer_descriptors", [Pure]> {
  39:   let summary = "define an array of buffer descriptors";
  40:   let description = [{
  41:     Create a tensor of buffer descriptors packing 32-bit pointer offsets and
  42:     32-bit lengths into 64-bit elements.
  43:   }];
  44:   let arguments = (ins DenseI32ArrayAttr:$offsets, DenseI32ArrayAttr:$lengths,
  45:                    TT_MemTypeAttr:$memType);
  46:   let results = (outs TT_Tensor:$result);
  47:   let assemblyFormat = [{
  48:     $offsets `,` $lengths `,` $memType attr-dict `:` type($result)
  49:   }];
  50: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalBufferDescriptorsOp` with the summary “define an array of buffer descriptors”.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalBufferDescriptorsOp`，其摘要为“define an array of buffer descriptors”。

### Lines 52-62
```tablegen
  52: def TTI_ExperimentalMemDescToI32Op : TTI_Op<"experimental_memdesc_to_i32", [Pure]> {
  53:   let summary = "Convert a memdesc into its base pointer as i32";
  54:   let description = [{
  55:     Extract the base pointer from the given memdesc and return it as a 32-bit
  56:     integer. This can be used to compare the memdesc against tensors of barrier
  57:     pointers maintained by the concurrency sanitizer.
  58:   }];
  59:   let arguments = (ins TTG_MemDescType:$memdesc);
  60:   let results = (outs I32:$result);
  61:   let assemblyFormat = "$memdesc attr-dict `:` type($memdesc)";
  62: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalMemDescToI32Op` with the summary “Convert a memdesc into its base pointer as i32”. It is specialized from `TTI_Op<"experimental_memdesc_to_i32", [Pure]>`.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalMemDescToI32Op`，其摘要为“Convert a memdesc into its base pointer as i32”。 它基于 `TTI_Op<"experimental_memdesc_to_i32", [Pure]>` 进一步特化。

### Lines 64-78
```tablegen
  64: def TTI_ExperimentalClusterCTAIdOp
  65:     : TTI_Op<"experimental_cluster_cta_id", [Pure]> {
  66:   let summary = "Get the CTA id within the current cluster";
  67:   let description = [{
  68:     Return the cluster-local CTA id used to index ConSan's multi-CTA scratch
  69:     slabs. For single-CTA kernels this is always zero.
  70:   }];
  71:   let results = (outs I32:$result);
  72:   let builders = [
  73:     OpBuilder<(ins), [{
  74:       build($_builder, $_state, $_builder.getI32Type());
  75:     }]>
  76:   ];
  77:   let assemblyFormat = "attr-dict `:` type($result)";
  78: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalClusterCTAIdOp` with the summary “Get the CTA id within the current cluster”.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalClusterCTAIdOp`，其摘要为“Get the CTA id within the current cluster”。

### Lines 80-84
```tablegen
  80: def TTI_ExperimentalGSanInitOp
  81:     : TTI_Op<"experimental_gsan_init"> {
  82:   let summary = "Initialize GSan thread";
  83:   let arguments = (ins);
  84: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalGSanInitOp` with the summary “Initialize GSan thread”.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalGSanInitOp`，其摘要为“Initialize GSan thread”。

### Lines 86-98
```tablegen
  86: def TTI_ExperimentalGSanTensorDescInfoOp
  87:     : TTI_Op<"experimental_gsan_tensordesc_info", [Pure]> {
  88:   let summary = "Decode GSan descriptor metadata from a native tensor descriptor";
  89:   let description = [{
  90:     Decodes a native tensor descriptor into the underlying
  91:     base pointer, shape and stride values.
  92:   }];
  93:   let arguments = (ins TT_AnyTensorDescType:$desc);
  94:   let results = (outs Variadic<AnyType>:$result);
  95:   let assemblyFormat = [{
  96:     $desc attr-dict `:` qualified(type($desc)) `->` type($result)
  97:   }];
  98: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalGSanTensorDescInfoOp` with the summary “Decode GSan descriptor metadata from a native tensor descriptor”.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalGSanTensorDescInfoOp`，其摘要为“Decode GSan descriptor metadata from a native tensor descriptor”。

### Lines 100-115
```tablegen
 100: def TTI_ExperimentalGSanTensorAccessOp
 101:     : TTI_Op<"experimental_gsan_tensor_access",
 102:              [TypesMatchWith<"mask type matches ptr type", "ptr", "mask",
 103:                              "getI1SameShape(getPointeeType($_self))",
 104:                              "($_op.getOperands().size() <= 1) || std::equal_to<>()">]> {
 105:   let summary = "Instrument a tensor load/store access for GSan";
 106:   let description = [{
 107:     Emits runtime instrumentation for a tensor pointer access. The pointer and
 108:     optional mask are consumed by the GSan runtime.
 109:   }];
 110:   let arguments = (ins Arg<TT_PtrLike, "", [MemWrite<GlobalMemory>, MemRead<GlobalMemory>]>:$ptr,
 111:                    Optional<TT_BoolLike>:$mask, BoolAttr:$isStore);
 112:   let assemblyFormat = [{
 113:     $ptr `,` $isStore (`,` $mask^)? attr-dict `:` type($ptr)
 114:   }];
 115: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalGSanTensorAccessOp` with the summary “Instrument a tensor load/store access for GSan”.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalGSanTensorAccessOp`，其摘要为“Instrument a tensor load/store access for GSan”。

### Lines 117-132
```tablegen
 117: def TTI_ExperimentalGSanAtomicTensorAccessOp
 118:     : TTI_Op<"experimental_gsan_atomic_tensor_access",
 119:              [TypesMatchWith<"mask type matches ptr type", "ptr", "mask",
 120:                              "getI1SameShape(getPointeeType($_self))",
 121:                              "($_op.getOperands().size() <= 1) || std::equal_to<>()">]> {
 122:   let summary = "Instrument a tensor atomic access for GSan";
 123:   let description = [{
 124:     Emits runtime instrumentation for a tensor pointer access whose individual
 125:     elements are atomic read-modify-write operations.
 126:   }];
 127:   let arguments = (ins Arg<TT_PtrLike, "", [MemWrite<GlobalMemory>, MemRead<GlobalMemory>]>:$ptr,
 128:                    Optional<TT_BoolLike>:$mask, TT_MemSemanticAttr:$sem, TT_MemSyncScopeAttr:$scope);
 129:   let assemblyFormat = [{
 130:     $sem `,` $scope `,` $ptr (`,` $mask^)? attr-dict `:` type($ptr)
 131:   }];
 132: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalGSanAtomicTensorAccessOp` with the summary “Instrument a tensor atomic access for GSan”.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalGSanAtomicTensorAccessOp`，其摘要为“Instrument a tensor atomic access for GSan”。

### Lines 134-157
```tablegen
 134: def TTI_ExperimentalGSanAtomicRMWOp : TTI_Op<"experimental_gsan_atomic_rmw", [
 135:   SameOperandsAndResultShape,
 136:   SameOperandsAndResultEncoding,
 137:   TypesMatchWith<"ptr type matches value type", "val", "ptr",
 138:                  "getPointerTypeSameShape($_self)">,
 139:   TypesMatchWith<"mask type matches value type",
 140:                  "val", "mask", "getI1SameShape($_self)",
 141:                  "($_op.getOperands().size() <= 2) || std::equal_to<>()">
 142: ]> {
 143:   let summary = "Lower a GSan-instrumented atomic rmw";
 144:   let arguments = (ins
 145:     TT_AtomicRMWAttr:$atomic_rmw_op,
 146:     Arg<TT_PtrLike, "", [MemRead<GlobalMemory>, MemWrite<GlobalMemory>]>:$ptr,
 147:     TT_Type:$val,
 148:     Optional<TT_BoolLike>:$mask,
 149:     TT_MemSemanticAttr:$sem,
 150:     TT_MemSyncScopeAttr:$scope
 151:   );
 152:   let results = (outs TT_Type:$result);
 153:   let assemblyFormat = [{
 154:     $atomic_rmw_op `,` $sem `,` $scope `,` $ptr `,` $val (`,` $mask^)? attr-dict `:`
 155:     functional-type(operands, $result)
 156:   }];
 157: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalGSanAtomicRMWOp` with the summary “Lower a GSan-instrumented atomic rmw”. It is specialized from `TTI_Op<"experimental_gsan_atomic_rmw", [`.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalGSanAtomicRMWOp`，其摘要为“Lower a GSan-instrumented atomic rmw”。 它基于 `TTI_Op<"experimental_gsan_atomic_rmw", [` 进一步特化。

### Lines 159-180
```tablegen
 159: def TTI_ExperimentalGSanAtomicCASOp : TTI_Op<"experimental_gsan_atomic_cas", [
 160:   SameOperandsAndResultShape,
 161:   SameOperandsAndResultEncoding,
 162:   TypesMatchWith<"ptr type matches cmp type", "cmp", "ptr",
 163:                  "getPointerTypeSameShape($_self)">,
 164:   TypesMatchWith<"ptr type matches value type", "val", "ptr",
 165:                  "getPointerTypeSameShape($_self)">
 166: ]> {
 167:   let summary = "Lower a GSan-instrumented atomic cas";
 168:   let arguments = (ins
 169:     Arg<TT_PtrLike, "", [MemRead<GlobalMemory>, MemWrite<GlobalMemory>]>:$ptr,
 170:     TT_Type:$cmp,
 171:     TT_Type:$val,
 172:     TT_MemSemanticAttr:$sem,
 173:     TT_MemSyncScopeAttr:$scope
 174:   );
 175:   let results = (outs TT_Type:$result);
 176:   let assemblyFormat = [{
 177:     $sem `,` $scope `,` $ptr `,` $cmp `,` $val attr-dict `:`
 178:     functional-type(operands, $result)
 179:   }];
 180: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalGSanAtomicCASOp` with the summary “Lower a GSan-instrumented atomic cas”. It is specialized from `TTI_Op<"experimental_gsan_atomic_cas", [`.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalGSanAtomicCASOp`，其摘要为“Lower a GSan-instrumented atomic cas”。 它基于 `TTI_Op<"experimental_gsan_atomic_cas", [` 进一步特化。

### Lines 183-183
```tablegen
 183: // ===== Critical section lock ops =====
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===== Critical section lock ops =====.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 186-195
```tablegen
 186: def TTI_ExperimentalLockAcquireOp : TTI_Op<"experimental_lock_acquire", [MemoryEffects<[MemWrite<GlobalMemory>]>]> {
 187:   let summary = "Acquire a lock.";
 188:   let description = [{
 189:     Enter a critical section by acquiring a lock with single thread.
 190:   }];
 191:   let arguments = (ins TT_PtrLike:$lock, Optional<I1>:$pred);
 192:   let assemblyFormat = [{
 193:     $lock (`,` $pred^)? attr-dict `:` type($lock)
 194:   }];
 195: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalLockAcquireOp` with the summary “Acquire a lock.”. It is specialized from `TTI_Op<"experimental_lock_acquire", [MemoryEffects<[MemWrite<GlobalMemory>]>]>`.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalLockAcquireOp`，其摘要为“Acquire a lock.”。 它基于 `TTI_Op<"experimental_lock_acquire", [MemoryEffects<[MemWrite<GlobalMemory>]>]>` 进一步特化。

### Lines 198-207
```tablegen
 198: def TTI_ExperimentalLockReleaseOp : TTI_Op<"experimental_lock_release", [MemoryEffects<[MemWrite<GlobalMemory>]>]> {
 199:   let summary = "Release a lock.";
 200:   let description = [{
 201:     Leave a critical section by releasing a lock with single thread.
 202:   }];
 203:   let arguments = (ins TT_PtrLike:$lock, Optional<I1>:$pred);
 204:   let assemblyFormat = [{
 205:     $lock (`,` $pred^)? attr-dict `:` type($lock)
 206:   }];
 207: }
```
**EN:** This TableGen def record defines `TTI_ExperimentalLockReleaseOp` with the summary “Release a lock.”. It is specialized from `TTI_Op<"experimental_lock_release", [MemoryEffects<[MemWrite<GlobalMemory>]>]>`.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalLockReleaseOp`，其摘要为“Release a lock.”。 它基于 `TTI_Op<"experimental_lock_release", [MemoryEffects<[MemWrite<GlobalMemory>]>]>` 进一步特化。

### Lines 210-210
```tablegen
 210: // ===== FPSan ops =====
```
**EN:** This comment block records the intent and constraints of the surrounding code: ===== FPSan ops =====.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 213-224
```tablegen
 213: def TTI_ExperimentalFPSanEmbedOp : TTI_Op<"experimental_fpsan_embed", [
 214:   Pure,
 215:   Elementwise,
 216:   SameOperandsAndResultShape,
 217:   SameOperandsAndResultEncoding
 218: ]> {
 219:   let summary = "Embed float value in the fpsan integer ring";
 220:   let arguments = (ins TT_FloatLike:$val);
 221:   let results = (outs TT_IntLike:$result);
 222:   let assemblyFormat = [{
 223:     $val attr-dict `:` functional-type(operands, $result)
 224:   }];
```
**EN:** This TableGen def record defines `TTI_ExperimentalFPSanEmbedOp` with the summary “Embed float value in the fpsan integer ring”. It is specialized from `TTI_Op<"experimental_fpsan_embed", [`.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalFPSanEmbedOp`，其摘要为“Embed float value in the fpsan integer ring”。 它基于 `TTI_Op<"experimental_fpsan_embed", [` 进一步特化。

### Lines 226-228
```tablegen
 226:   let hasCanonicalizer = true;
 227:   let hasFolder = true;
 228: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 230-241
```tablegen
 230: def TTI_ExperimentalFPSanUnembedOp : TTI_Op<"experimental_fpsan_unembed", [
 231:   Pure,
 232:   Elementwise,
 233:   SameOperandsAndResultShape,
 234:   SameOperandsAndResultEncoding
 235: ]> {
 236:   let summary = "Unembed fpsan integer payload as a float value";
 237:   let arguments = (ins TT_IntLike:$val);
 238:   let results = (outs TT_FloatLike:$result);
 239:   let assemblyFormat = [{
 240:     $val attr-dict `:` functional-type(operands, $result)
 241:   }];
```
**EN:** This TableGen def record defines `TTI_ExperimentalFPSanUnembedOp` with the summary “Unembed fpsan integer payload as a float value”. It is specialized from `TTI_Op<"experimental_fpsan_unembed", [`.
**CN:** 该 TableGen def 记录定义了 `TTI_ExperimentalFPSanUnembedOp`，其摘要为“Unembed fpsan integer payload as a float value”。 它基于 `TTI_Op<"experimental_fpsan_unembed", [` 进一步特化。

### Lines 243-245
```tablegen
 243:   let hasCanonicalizer = true;
 244:   let hasFolder = true;
 245: }
```
**EN:** This block toggles generated hooks such as verifier, folder, canonicalizer, or custom property handling.
**CN:** 该代码块开启生成层面的钩子，例如 verifier、folder、canonicalizer 或自定义属性处理。

### Lines 247-247
```tablegen
 247: #endif // TRITONINSTRUMENT_OPS
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
  - `triton/Dialect/TritonInstrument/IR/TritonInstrumentDialect.td`
  - `triton/Dialect/TritonGPU/IR/TritonGPUTypes.td`
  - `triton/Dialect/Triton/IR/TritonTypes.td`
  - `triton/Dialect/Triton/IR/TritonInterfaces.td`
  - `triton/Dialect/Triton/IR/TritonAttrDefs.td`
  - `mlir/IR/OpBase.td`
  - `mlir/Interfaces/SideEffectInterfaces.td`
  - `triton/Dialect/TritonInstrument/IR/TritonInstrumentAttrDefs.td`
