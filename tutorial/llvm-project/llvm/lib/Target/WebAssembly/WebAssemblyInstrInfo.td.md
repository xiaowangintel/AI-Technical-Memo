# WebAssemblyInstrInfo.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyInstrInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: WebAssembly Instruction definitions. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyInstrInfo.td`，主要负责 WebAssembly 后端的指令信息与 lowering 辅助逻辑。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
// WebAssemblyInstrInfo.td-Describe the WebAssembly Instructions-*- tablegen -*-
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```tablegen
//===----------------------------------------------------------------------===//
///
/// \file
/// WebAssembly Instruction definitions.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。

### Lines 12-13

```tablegen
//===----------------------------------------------------------------------===//
```
- **EN**: Continues the WebAssembly backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 14-15

```tablegen
//===----------------------------------------------------------------------===//
// WebAssembly Instruction Predicate Definitions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly Instruction Predicate Definitions.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly Instruction Predicate Definitions.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 16-22

```tablegen
//===----------------------------------------------------------------------===//

def IsPIC     : Predicate<"TM.isPositionIndependent()">;
def IsNotPIC  : Predicate<"!TM.isPositionIndependent()">;

def HasAddr32 : Predicate<"!Subtarget->hasAddr64()">;
```
- **EN**: Adds declarative TableGen records such as `IsPIC`, `IsNotPIC`, `HasAddr32` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IsPIC`, `IsNotPIC`, `HasAddr32`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 23-32

```tablegen
def HasAddr64 : Predicate<"Subtarget->hasAddr64()">;

def HasAtomics :
    Predicate<"Subtarget->hasAtomics()">,
    AssemblerPredicate<(all_of FeatureAtomics), "atomics">;

def HasBulkMemory :
    Predicate<"Subtarget->hasBulkMemory()">,
    AssemblerPredicate<(all_of FeatureBulkMemory), "bulk-memory">;
```
- **EN**: Adds declarative TableGen records such as `HasAddr64`, `HasAtomics`, `HasBulkMemory` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasAddr64`, `HasAtomics`, `HasBulkMemory`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 33-40

```tablegen
def HasBulkMemoryOpt :
    Predicate<"Subtarget->hasBulkMemoryOpt()">,
    AssemblerPredicate<(all_of FeatureBulkMemoryOpt), "bulk-memory-opt">;

def HasCallIndirectOverlong :
    Predicate<"Subtarget->hasCallIndirectOverlong()">,
    AssemblerPredicate<(all_of FeatureCallIndirectOverlong), "call-indirect-overlong">;
```
- **EN**: Adds declarative TableGen records such as `HasBulkMemoryOpt`, `HasCallIndirectOverlong` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasBulkMemoryOpt`, `HasCallIndirectOverlong`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 41-48

```tablegen
def HasExceptionHandling :
    Predicate<"Subtarget->hasExceptionHandling()">,
    AssemblerPredicate<(all_of FeatureExceptionHandling), "exception-handling">;

def HasExtendedConst :
    Predicate<"Subtarget->hasExtendedConst()">,
    AssemblerPredicate<(all_of FeatureExtendedConst), "extended-const">;
```
- **EN**: Adds declarative TableGen records such as `HasExceptionHandling`, `HasExtendedConst` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasExceptionHandling`, `HasExtendedConst`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 49-55

```tablegen
def HasFP16 :
    Predicate<"Subtarget->hasFP16()">,
    AssemblerPredicate<(all_of FeatureFP16), "fp16">;

def HasGC : Predicate<"Subtarget->hasGC()">,
            AssemblerPredicate<(all_of FeatureGC), "gc">;
```
- **EN**: Adds declarative TableGen records such as `HasFP16`, `HasGC` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasFP16`, `HasGC`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 56-63

```tablegen
def HasMultiMemory :
    Predicate<"Subtarget->hasMultiMemory()">,
    AssemblerPredicate<(all_of FeatureMultiMemory), "multimemory">;

def HasMultivalue :
    Predicate<"Subtarget->hasMultivalue()">,
    AssemblerPredicate<(all_of FeatureMultivalue), "multivalue">;
```
- **EN**: Adds declarative TableGen records such as `HasMultiMemory`, `HasMultivalue` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasMultiMemory`, `HasMultivalue`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 64-72

```tablegen
def HasMutableGlobals:
    Predicate<"Subtarget->hasMutableGlobals()">,
    AssemblerPredicate<(all_of FeatureMutableGlobals), "mutable-globals">;

def HasNontrappingFPToInt :
    Predicate<"Subtarget->hasNontrappingFPToInt()">,
    AssemblerPredicate<(all_of FeatureNontrappingFPToInt),
                       "nontrapping-fptoint">;
```
- **EN**: Adds declarative TableGen records such as `HasMutableGlobals`, `HasNontrappingFPToInt` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasMutableGlobals`, `HasNontrappingFPToInt`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 73-81

```tablegen
def NotHasNontrappingFPToInt :
    Predicate<"!Subtarget->hasNontrappingFPToInt()">,
    AssemblerPredicate<(all_of (not FeatureNontrappingFPToInt)),
                       "nontrapping-fptoint">;

def HasReferenceTypes :
    Predicate<"Subtarget->hasReferenceTypes()">,
    AssemblerPredicate<(all_of FeatureReferenceTypes), "reference-types">;
```
- **EN**: Adds declarative TableGen records such as `NotHasNontrappingFPToInt`, `HasReferenceTypes` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NotHasNontrappingFPToInt`, `HasReferenceTypes`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 82-89

```tablegen
def HasRelaxedAtomics :
    Predicate<"Subtarget->hasRelaxedAtomics()">,
    AssemblerPredicate<(all_of FeatureRelaxedAtomics), "relaxed-atomics">;

def HasRelaxedSIMD :
    Predicate<"Subtarget->hasRelaxedSIMD()">,
    AssemblerPredicate<(all_of FeatureRelaxedSIMD), "relaxed-simd">;
```
- **EN**: Adds declarative TableGen records such as `HasRelaxedAtomics`, `HasRelaxedSIMD` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasRelaxedAtomics`, `HasRelaxedSIMD`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 90-97

```tablegen
def HasSignExt :
    Predicate<"Subtarget->hasSignExt()">,
    AssemblerPredicate<(all_of FeatureSignExt), "sign-ext">;

def HasSIMD128 :
    Predicate<"Subtarget->hasSIMD128()">,
    AssemblerPredicate<(any_of FeatureSIMD128, FeatureRelaxedSIMD), "simd128">;
```
- **EN**: Adds declarative TableGen records such as `HasSignExt`, `HasSIMD128` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasSignExt`, `HasSIMD128`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 98-105

```tablegen
def HasTailCall :
    Predicate<"Subtarget->hasTailCall()">,
    AssemblerPredicate<(all_of FeatureTailCall), "tail-call">;

def HasWideArithmetic :
    Predicate<"Subtarget->hasWideArithmetic()">,
    AssemblerPredicate<(all_of FeatureWideArithmetic), "wide-arithmetic">;
```
- **EN**: Adds declarative TableGen records such as `HasTailCall`, `HasWideArithmetic` that LLVM later expands into generated tables or helper code. Subtarget feature gating influences the behavior here.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HasTailCall`, `HasWideArithmetic`，LLVM 随后会把它们展开成生成表或辅助代码。 子目标特性裁剪会影响这里的行为。

### Lines 106-107

```tablegen
//===----------------------------------------------------------------------===//
// WebAssembly-specific DAG Node Types.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly-specific DAG Node Types.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly-specific DAG Node Types.”。

### Lines 108-114

```tablegen
//===----------------------------------------------------------------------===//

def SDT_WebAssemblyCallSeqStart : SDCallSeqStart<[SDTCisVT<0, iPTR>,
                                                  SDTCisVT<1, iPTR>]>;
def SDT_WebAssemblyCallSeqEnd :
    SDCallSeqEnd<[SDTCisVT<0, iPTR>, SDTCisVT<1, iPTR>]>;
def SDT_WebAssemblyBrTable    : SDTypeProfile<0, -1, [SDTCisPtrTy<0>]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_WebAssemblyCallSeqStart`, `SDT_WebAssemblyCallSeqEnd`, `SDT_WebAssemblyBrTable` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_WebAssemblyCallSeqStart`, `SDT_WebAssemblyCallSeqEnd`, `SDT_WebAssemblyBrTable`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 115-121

```tablegen
def SDT_WebAssemblyArgument   : SDTypeProfile<1, 1, [SDTCisVT<1, i32>]>;
def SDT_WebAssemblyLocalGet   : SDTypeProfile<1, 1, [SDTCisVT<1, i32>]>;
def SDT_WebAssemblyLocalSet   : SDTypeProfile<0, 2, [SDTCisVT<0, i32>]>;
def SDT_WebAssemblyReturn     : SDTypeProfile<0, -1, []>;
def SDT_WebAssemblyWrapper    : SDTypeProfile<1, 1, [SDTCisSameAs<0, 1>,
                                                     SDTCisPtrTy<0>]>;
def SDT_WebAssemblyGlobalGet  : SDTypeProfile<1, 1, [SDTCisPtrTy<1>]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_WebAssemblyArgument`, `SDT_WebAssemblyLocalGet`, `SDT_WebAssemblyLocalSet` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_WebAssemblyArgument`, `SDT_WebAssemblyLocalGet`, `SDT_WebAssemblyLocalSet`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 122-123

```tablegen
def SDT_WebAssemblyGlobalSet  : SDTypeProfile<0, 2, [SDTCisPtrTy<1>]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_WebAssemblyGlobalSet` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_WebAssemblyGlobalSet`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 124-125

```tablegen
//===----------------------------------------------------------------------===//
// WebAssembly-specific DAG Nodes.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly-specific DAG Nodes.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly-specific DAG Nodes.”。

### Lines 126-133

```tablegen
//===----------------------------------------------------------------------===//

def WebAssemblycallseq_start :
    SDNode<"ISD::CALLSEQ_START", SDT_WebAssemblyCallSeqStart,
           [SDNPHasChain, SDNPOutGlue]>;
def WebAssemblycallseq_end :
    SDNode<"ISD::CALLSEQ_END", SDT_WebAssemblyCallSeqEnd,
           [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;
```
- **EN**: Adds declarative TableGen records such as `WebAssemblycallseq_start`, `WebAssemblycallseq_end` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WebAssemblycallseq_start`, `WebAssemblycallseq_end`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 134-141

```tablegen
def WebAssemblybr_table : SDNode<"WebAssemblyISD::BR_TABLE",
                                 SDT_WebAssemblyBrTable,
                                 [SDNPHasChain, SDNPVariadic]>;
def WebAssemblyargument : SDNode<"WebAssemblyISD::ARGUMENT",
                                 SDT_WebAssemblyArgument>;
def WebAssemblyreturn   : SDNode<"WebAssemblyISD::RETURN",
                                 SDT_WebAssemblyReturn,
                                 [SDNPHasChain, SDNPVariadic]>;
```
- **EN**: Adds declarative TableGen records such as `WebAssemblybr_table`, `WebAssemblyargument`, `WebAssemblyreturn` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WebAssemblybr_table`, `WebAssemblyargument`, `WebAssemblyreturn`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 142-148

```tablegen
def WebAssemblyWrapper  : SDNode<"WebAssemblyISD::Wrapper",
                                 SDT_WebAssemblyWrapper>;
def WebAssemblyWrapperREL  : SDNode<"WebAssemblyISD::WrapperREL",
                                     SDT_WebAssemblyWrapper>;
def WebAssemblyglobal_get :
    SDNode<"WebAssemblyISD::GLOBAL_GET", SDT_WebAssemblyGlobalGet,
           [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;
```
- **EN**: Adds declarative TableGen records such as `WebAssemblyWrapper`, `WebAssemblyWrapperREL`, `WebAssemblyglobal_get` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WebAssemblyWrapper`, `WebAssemblyWrapperREL`, `WebAssemblyglobal_get`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 149-158

```tablegen
def WebAssemblyglobal_set :
    SDNode<"WebAssemblyISD::GLOBAL_SET", SDT_WebAssemblyGlobalSet,
           [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
def WebAssemblylocal_get :
    SDNode<"WebAssemblyISD::LOCAL_GET", SDT_WebAssemblyLocalGet,
           [SDNPHasChain, SDNPMayLoad]>;
def WebAssemblylocal_set :
    SDNode<"WebAssemblyISD::LOCAL_SET", SDT_WebAssemblyLocalSet,
           [SDNPHasChain, SDNPMayStore]>;
```
- **EN**: Adds declarative TableGen records such as `WebAssemblyglobal_set`, `WebAssemblylocal_get`, `WebAssemblylocal_set` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WebAssemblyglobal_set`, `WebAssemblylocal_get`, `WebAssemblylocal_set`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 159-160

```tablegen
//===----------------------------------------------------------------------===//
// WebAssembly-specific Operands.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly-specific Operands.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly-specific Operands.”。

### Lines 161-169

```tablegen
//===----------------------------------------------------------------------===//

// Default Operand has AsmOperandClass "Imm" which is for integers (and
// symbols), so specialize one for floats:
class FPImmAsmOperand<ValueType ty> : AsmOperandClass {
  let Name = "FPImm" # ty;
  let PredicateMethod = "isFPImm";
}
```
- **EN**: Declares a backend-facing type `FPImmAsmOperand` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `FPImmAsmOperand`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 170-176

```tablegen
class FPOperand<ValueType ty> : Operand<ty> {
  AsmOperandClass ParserMatchClass = FPImmAsmOperand<ty>;
}

let OperandNamespace = "WebAssembly" in {

let OperandType = "OPERAND_BASIC_BLOCK" in
```
- **EN**: Declares a backend-facing type `FPOperand` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `FPOperand`，并勾勒出周边代码会依赖的接口或状态。

### Lines 177-183

```tablegen
def bb_op : Operand<OtherVT>;

let OperandType = "OPERAND_LOCAL" in
def local_op : Operand<i32>;

let OperandType = "OPERAND_GLOBAL" in {
  // The operand to global instructions is always a 32-bit index.
```
- **EN**: Adds declarative TableGen records such as `bb_op`, `local_op` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `bb_op`, `local_op`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 184-190

```tablegen
  def global_op32 : Operand<i32>;
  // In PIC mode however, we temporarily represent this index as an external
  // symbol, which to LLVM is a pointer, so in wasm64 mode it is easiest to
  // pretend we use a 64-bit index for it.
  def global_op64 : Operand<i64>;
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "In PIC mode however, we temporarily represent this index as an external".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“In PIC mode however, we temporarily represent this index as an external”。

### Lines 191-197

```tablegen
let OperandType = "OPERAND_I32IMM" in
def i32imm_op : Operand<i32>;

let OperandType = "OPERAND_I64IMM" in
def i64imm_op : Operand<i64>;

let OperandType = "OPERAND_F32IMM" in
```
- **EN**: Adds declarative TableGen records such as `i32imm_op`, `i64imm_op` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `i32imm_op`, `i64imm_op`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 198-205

```tablegen
def f32imm_op : FPOperand<f32>;

let OperandType = "OPERAND_F64IMM" in
def f64imm_op : FPOperand<f64>;

let OperandType = "OPERAND_VEC_I8IMM" in
def vec_i8imm_op : Operand<i32>;
```
- **EN**: Adds declarative TableGen records such as `f32imm_op`, `f64imm_op`, `vec_i8imm_op` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `f32imm_op`, `f64imm_op`, `vec_i8imm_op`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 206-212

```tablegen
let OperandType = "OPERAND_VEC_I16IMM" in
def vec_i16imm_op : Operand<i32>;

let OperandType = "OPERAND_VEC_I32IMM" in
def vec_i32imm_op : Operand<i32>;

let OperandType = "OPERAND_VEC_I64IMM" in
```
- **EN**: Adds declarative TableGen records such as `vec_i16imm_op`, `vec_i32imm_op` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vec_i16imm_op`, `vec_i32imm_op`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 213-220

```tablegen
def vec_i64imm_op : Operand<i64>;

let OperandType = "OPERAND_FUNCTION32" in
def function32_op : Operand<i32>;

let OperandType = "OPERAND_TABLE" in
def table32_op : Operand<i32>;
```
- **EN**: Adds declarative TableGen records such as `vec_i64imm_op`, `function32_op`, `table32_op` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `vec_i64imm_op`, `function32_op`, `table32_op`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 221-227

```tablegen
let OperandType = "OPERAND_OFFSET32" in
def offset32_op : Operand<i32>;

let OperandType = "OPERAND_OFFSET64" in
def offset64_op : Operand<i64>;

let OperandType = "OPERAND_P2ALIGN" in {
```
- **EN**: Adds declarative TableGen records such as `offset32_op`, `offset64_op` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `offset32_op`, `offset64_op`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 228-236

```tablegen
def P2Align : Operand<i32> {
  let PrintMethod = "printWebAssemblyP2AlignOperand";
}

let OperandType = "OPERAND_TAG" in
def tag_op : Operand<i32>;

} // OperandType = "OPERAND_P2ALIGN"
```
- **EN**: Adds declarative TableGen records such as `P2Align`, `tag_op` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `P2Align`, `tag_op`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 237-243

```tablegen
let OperandType = "OPERAND_MEMORDER" in {
def MemOrder : Operand<i32> {
  let PrintMethod = "printWebAssemblyMemOrderOperand";
}
} // OperandType = "OPERAND_MEMORDER"

let OperandType = "OPERAND_SIGNATURE" in
```
- **EN**: Adds declarative TableGen records such as `MemOrder` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MemOrder`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 244-253

```tablegen
def Signature : Operand<i32> {
  let PrintMethod = "printWebAssemblySignatureOperand";
}

let OperandType = "OPERAND_TYPEINDEX" in
def TypeIndex : Operand<i32>;

} // OperandNamespace = "WebAssembly"

// TODO: Find more places to use this.
```
- **EN**: Adds declarative TableGen records such as `Signature`, `TypeIndex` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Signature`, `TypeIndex`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 254-262

```tablegen
def bool_node : PatLeaf<(i32 I32:$cond), [{
  return CurDAG->computeKnownBits(Op).countMinLeadingZeros() == 31;
}]>;

/// Floating point constants
def fpimm0    : PatLeaf<(fpimm), [{
  return N->isExactlyValue(+0.0);
}]>;
```
- **EN**: Adds declarative TableGen records such as `bool_node`, `fpimm0` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `bool_node`, `fpimm0`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 263-264

```tablegen
//===----------------------------------------------------------------------===//
// WebAssembly Register to Stack instruction mapping
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly Register to Stack instruction mapping". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly Register to Stack instruction mapping”。 该区间与栈帧布局或栈访问相关。

### Lines 265-271

```tablegen
//===----------------------------------------------------------------------===//

class StackRel;
def getStackOpcode : InstrMapping {
  let FilterClass = "StackRel";
  let RowFields = ["BaseName"];
  let ColFields = ["StackBased"];
```
- **EN**: Declares a backend-facing type `StackRel`, `getStackOpcode` and outlines the API or state that nearby code will rely on. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明面向后端的类型 `StackRel`, `getStackOpcode`，并勾勒出周边代码会依赖的接口或状态。 该区间与栈帧布局或栈访问相关。

### Lines 272-275

```tablegen
  let KeyCol = ["0"];
  let ValueCols = [["1"]];
}
```
- **EN**: Continues the WebAssembly backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 276-277

```tablegen
//===----------------------------------------------------------------------===//
// WebAssembly Stack to Register instruction mapping
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly Stack to Register instruction mapping". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly Stack to Register instruction mapping”。 该区间与栈帧布局或栈访问相关。

### Lines 278-284

```tablegen
//===----------------------------------------------------------------------===//

class RegisterRel;
def getRegisterOpcode : InstrMapping {
  let FilterClass = "RegisterRel";
  let RowFields = ["BaseName"];
  let ColFields = ["StackBased"];
```
- **EN**: Declares a backend-facing type `RegisterRel`, `getRegisterOpcode` and outlines the API or state that nearby code will rely on. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里声明面向后端的类型 `RegisterRel`, `getRegisterOpcode`，并勾勒出周边代码会依赖的接口或状态。 该区间与栈帧布局或栈访问相关。

### Lines 285-288

```tablegen
  let KeyCol = ["1"];
  let ValueCols = [["0"]];
}
```
- **EN**: Continues the WebAssembly backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 289-290

```tablegen
//===----------------------------------------------------------------------===//
// WebAssembly 32 to 64-bit instruction mapping
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly 32 to 64-bit instruction mapping".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly 32 to 64-bit instruction mapping”。

### Lines 291-297

```tablegen
//===----------------------------------------------------------------------===//

class Wasm64Rel;
def getWasm64Opcode : InstrMapping {
  let FilterClass = "Wasm64Rel";
  let RowFields = ["Wasm32Name"];
  let ColFields = ["IsWasm64"];
```
- **EN**: Declares a backend-facing type `Wasm64Rel`, `getWasm64Opcode` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `Wasm64Rel`, `getWasm64Opcode`，并勾勒出周边代码会依赖的接口或状态。

### Lines 298-301

```tablegen
  let KeyCol = ["0"];
  let ValueCols = [["1"]];
}
```
- **EN**: Continues the WebAssembly backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 302-303

```tablegen
//===----------------------------------------------------------------------===//
// WebAssembly Instruction Format Definitions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "WebAssembly Instruction Format Definitions.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“WebAssembly Instruction Format Definitions.”。

### Lines 304-307

```tablegen
//===----------------------------------------------------------------------===//

include "WebAssemblyInstrFormats.td"
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 308-309

```tablegen
//===----------------------------------------------------------------------===//
// Additional instructions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Additional instructions.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Additional instructions.”。

### Lines 310-318

```tablegen
//===----------------------------------------------------------------------===//

multiclass ARGUMENT<WebAssemblyRegClass rc, ValueType vt> {
  let hasSideEffects = 1, isCodeGenOnly = 1, Defs = []<Register>,
      Uses = [ARGUMENTS] in
  defm ARGUMENT_#vt :
    I<(outs rc:$res), (ins i32imm:$argno), (outs), (ins i32imm:$argno),
      [(set (vt rc:$res), (WebAssemblyargument timm:$argno))]>;
}
```
- **EN**: Adds declarative TableGen records such as `ARGUMENT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ARGUMENT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 319-328

```tablegen
defm "": ARGUMENT<I32, i32>;
defm "": ARGUMENT<I64, i64>;
defm "": ARGUMENT<F32, f32>;
defm "": ARGUMENT<F64, f64>;
defm "": ARGUMENT<FUNCREF, funcref>;
defm "": ARGUMENT<EXTERNREF, externref>;
defm "": ARGUMENT<EXNREF, exnref>;

// local.get and local.set are not generated by instruction selection; they
// are implied by virtual register uses and defs.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 329-335

```tablegen
multiclass LOCAL<WebAssemblyRegClass rc, Operand global_op> {
  let hasSideEffects = 0 in {
  // COPY is not an actual instruction in wasm, but since we allow local.get and
  // local.set to be implicit during most of codegen, we can have a COPY which
  // is actually a no-op because all the work is done in the implied local.get
  // and local.set. COPYs are eliminated (and replaced with
  // local.get/local.set) in the ExplicitLocals pass.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "COPY is not an actual instruction in wasm, but since we allow local.get and". Notable symbols in this range include `LOCAL`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“COPY is not an actual instruction in wasm, but since we allow local.get and”。 该区间中较显眼的符号包括 `LOCAL`。

### Lines 336-342

```tablegen
  let isAsCheapAsAMove = 1, isCodeGenOnly = 1 in
  defm COPY_#rc : I<(outs rc:$res), (ins rc:$src), (outs), (ins), [],
                    "local.copy\t$res, $src", "local.copy">;

  // TEE is similar to COPY, but writes two copies of its result. Typically
  // this would be used to stackify one result and write the other result to a
  // local.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "TEE is similar to COPY, but writes two copies of its result. Typically".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“TEE is similar to COPY, but writes two copies of its result. Typically”。

### Lines 343-349

```tablegen
  let isAsCheapAsAMove = 1, isCodeGenOnly = 1 in
  defm TEE_#rc : I<(outs rc:$res, rc:$also), (ins rc:$src), (outs), (ins), [],
                   "local.tee\t$res, $also, $src", "local.tee">;

  // This is the actual local.get instruction in wasm. These are made explicit
  // by the ExplicitLocals pass. It has mayLoad because it reads from a wasm
  // local, which is a side effect not otherwise modeled in LLVM.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is the actual local.get instruction in wasm. These are made explicit".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is the actual local.get instruction in wasm. These are made explicit”。

### Lines 350-357

```tablegen
  let mayLoad = 1, isAsCheapAsAMove = 1 in
  defm LOCAL_GET_#rc : I<(outs rc:$res), (ins local_op:$local),
                         (outs), (ins local_op:$local), [],
                         "local.get\t$res, $local", "local.get\t$local", 0x20>;

  // This is the actual local.set instruction in wasm. These are made explicit
  // by the ExplicitLocals pass. It has mayStore because it writes to a wasm
  // local, which is a side effect not otherwise modeled in LLVM.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is the actual local.set instruction in wasm. These are made explicit".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is the actual local.set instruction in wasm. These are made explicit”。

### Lines 358-365

```tablegen
  let mayStore = 1, isAsCheapAsAMove = 1 in
  defm LOCAL_SET_#rc : I<(outs), (ins local_op:$local, rc:$src),
                         (outs), (ins local_op:$local), [],
                         "local.set\t$local, $src", "local.set\t$local", 0x21>;

  // This is the actual local.tee instruction in wasm. TEEs are turned into
  // LOCAL_TEEs by the ExplicitLocals pass. It has mayStore for the same reason
  // as LOCAL_SET.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is the actual local.tee instruction in wasm. TEEs are turned into".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is the actual local.tee instruction in wasm. TEEs are turned into”。

### Lines 366-372

```tablegen
  let mayStore = 1, isAsCheapAsAMove = 1 in
  defm LOCAL_TEE_#rc : I<(outs rc:$res), (ins local_op:$local, rc:$src),
                         (outs), (ins local_op:$local), [],
                         "local.tee\t$res, $local, $src", "local.tee\t$local",
                         0x22>;

  // Unused values must be dropped in some contexts.
```
- **EN**: Continues the WebAssembly backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 373-381

```tablegen
  defm DROP_#rc : I<(outs), (ins rc:$src), (outs), (ins), [],
                    "drop\t$src", "drop", 0x1a>;

  let mayLoad = 1 in
  defm GLOBAL_GET_#rc : I<(outs rc:$res), (ins global_op:$addr),
                          (outs), (ins global_op:$addr), [],
                           "global.get\t$res, $addr", "global.get\t$addr",
                          0x23>;
```
- **EN**: Continues the WebAssembly backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 382-388

```tablegen
  let mayStore = 1 in
  defm GLOBAL_SET_#rc : I<(outs), (ins global_op:$addr, rc:$src),
                          (outs), (ins global_op:$addr), [],
                          "global.set\t$addr, $src", "global.set\t$addr",
                          0x24>;

  } // hasSideEffects = 0
```
- **EN**: Continues the WebAssembly backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 WebAssembly 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 389-395

```tablegen
  foreach vt = rc.RegTypes in {
    def : Pat<(vt (WebAssemblyglobal_get
                   (WebAssemblyWrapper tglobaladdr:$addr))),
              (!cast<NI>("GLOBAL_GET_" # rc) tglobaladdr:$addr)>;
    def : Pat<(WebAssemblyglobal_set
               vt:$src, (WebAssemblyWrapper tglobaladdr:$addr)),
              (!cast<NI>("GLOBAL_SET_" # rc) tglobaladdr:$addr, vt:$src)>;
```
- **EN**: Implements helper routine(s) for this portion of the WebAssembly backend instruction information and lowering helpers.
- **CN**: 这里实现了 WebAssembly 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 396-402

```tablegen
    def : Pat<(vt (WebAssemblylocal_get (i32 timm:$local))),
              (!cast<NI>("LOCAL_GET_" # rc) timm:$local)>;
    def : Pat<(WebAssemblylocal_set timm:$local, vt:$src),
              (!cast<NI>("LOCAL_SET_" # rc) timm:$local, vt:$src)>;
  }
}
defm "" : LOCAL<I32, global_op32>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 403-411

```tablegen
defm "" : LOCAL<I64, global_op64>;  // 64-bit only needed for pointers.
defm "" : LOCAL<F32, global_op32>;
defm "" : LOCAL<F64, global_op32>;
defm "" : LOCAL<V128, global_op32>, Requires<[HasSIMD128]>;
defm "" : LOCAL<FUNCREF, global_op32>, Requires<[HasReferenceTypes]>;
defm "" : LOCAL<EXTERNREF, global_op32>, Requires<[HasReferenceTypes]>;
defm "" : LOCAL<EXNREF, global_op32>,
          Requires<[HasReferenceTypes, HasExceptionHandling]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 412-420

```tablegen
let isMoveImm = 1, isAsCheapAsAMove = 1, isReMaterializable = 1 in {
defm CONST_I32 : I<(outs I32:$res), (ins i32imm_op:$imm),
                   (outs), (ins i32imm_op:$imm),
                   [(set I32:$res, imm:$imm)],
                   "i32.const\t$res, $imm", "i32.const\t$imm", 0x41>;
defm CONST_I64 : I<(outs I64:$res), (ins i64imm_op:$imm),
                   (outs), (ins i64imm_op:$imm),
                   [(set I64:$res, imm:$imm)],
                   "i64.const\t$res, $imm", "i64.const\t$imm", 0x42>;
```
- **EN**: Adds declarative TableGen records such as `CONST_I32`, `CONST_I64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CONST_I32`, `CONST_I64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 421-430

```tablegen
defm CONST_F32 : I<(outs F32:$res), (ins f32imm_op:$imm),
                   (outs), (ins f32imm_op:$imm),
                   [(set F32:$res, fpimm:$imm)],
                   "f32.const\t$res, $imm", "f32.const\t$imm", 0x43>;
defm CONST_F64 : I<(outs F64:$res), (ins f64imm_op:$imm),
                   (outs), (ins f64imm_op:$imm),
                   [(set F64:$res, fpimm:$imm)],
                   "f64.const\t$res, $imm", "f64.const\t$imm", 0x44>;
} // isMoveImm = 1, isAsCheapAsAMove = 1, isReMaterializable = 1
```
- **EN**: Adds declarative TableGen records such as `CONST_F32`, `CONST_F64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CONST_F32`, `CONST_F64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 431-437

```tablegen
def : Pat<(i32 (WebAssemblyWrapper tglobaladdr:$addr)),
          (CONST_I32 tglobaladdr:$addr)>, Requires<[IsNotPIC, HasAddr32]>;
def : Pat<(i64 (WebAssemblyWrapper tglobaladdr:$addr)),
          (CONST_I64 tglobaladdr:$addr)>, Requires<[IsNotPIC, HasAddr64]>;

def : Pat<(i32 (WebAssemblyWrapper tglobaladdr:$addr)),
          (GLOBAL_GET_I32 tglobaladdr:$addr)>, Requires<[IsPIC, HasAddr32]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 438-445

```tablegen
def : Pat<(i64 (WebAssemblyWrapper tglobaladdr:$addr)),
          (GLOBAL_GET_I64 tglobaladdr:$addr)>, Requires<[IsPIC, HasAddr64]>;

def : Pat<(i32 (WebAssemblyWrapperREL tglobaladdr:$addr)),
          (CONST_I32 tglobaladdr:$addr)>, Requires<[IsPIC, HasAddr32]>;
def : Pat<(i64 (WebAssemblyWrapperREL tglobaladdr:$addr)),
          (CONST_I64 tglobaladdr:$addr)>, Requires<[IsPIC, HasAddr64]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 446-452

```tablegen
def : Pat<(i32 (WebAssemblyWrapperREL tglobaltlsaddr:$addr)),
          (CONST_I32 tglobaltlsaddr:$addr)>, Requires<[HasAddr32]>;
def : Pat<(i64 (WebAssemblyWrapperREL tglobaltlsaddr:$addr)),
          (CONST_I64 tglobaltlsaddr:$addr)>, Requires<[HasAddr64]>;

def : Pat<(i32 (WebAssemblyWrapper tglobaltlsaddr:$addr)),
          (GLOBAL_GET_I32 tglobaltlsaddr:$addr)>, Requires<[HasAddr32]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 453-460

```tablegen
def : Pat<(i64 (WebAssemblyWrapper tglobaltlsaddr:$addr)),
          (GLOBAL_GET_I64 tglobaltlsaddr:$addr)>, Requires<[HasAddr64]>;

def : Pat<(i32 (WebAssemblyWrapper texternalsym:$addr)),
          (GLOBAL_GET_I32 texternalsym:$addr)>, Requires<[IsPIC, HasAddr32]>;
def : Pat<(i64 (WebAssemblyWrapper texternalsym:$addr)),
          (GLOBAL_GET_I64 texternalsym:$addr)>, Requires<[IsPIC, HasAddr64]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 461-467

```tablegen
def : Pat<(i32 (WebAssemblyWrapper texternalsym:$addr)),
          (CONST_I32 texternalsym:$addr)>, Requires<[IsNotPIC, HasAddr32]>;
def : Pat<(i64 (WebAssemblyWrapper texternalsym:$addr)),
          (CONST_I64 texternalsym:$addr)>, Requires<[IsNotPIC, HasAddr64]>;

def : Pat<(i32 (WebAssemblyWrapperREL texternalsym:$addr)),
          (CONST_I32 texternalsym:$addr)>, Requires<[IsPIC, HasAddr32]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 468-470

```tablegen
def : Pat<(i64 (WebAssemblyWrapperREL texternalsym:$addr)),
          (CONST_I64 texternalsym:$addr)>, Requires<[IsPIC, HasAddr64]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 471-472

```tablegen
//===----------------------------------------------------------------------===//
// Additional sets of instructions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Additional sets of instructions.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Additional sets of instructions.”。

### Lines 473-479

```tablegen
//===----------------------------------------------------------------------===//

include "WebAssemblyInstrMemory.td"
include "WebAssemblyInstrCall.td"
include "WebAssemblyInstrControl.td"
include "WebAssemblyInstrConv.td"
include "WebAssemblyInstrInteger.td"
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 480-485

```tablegen
include "WebAssemblyInstrFloat.td"
include "WebAssemblyInstrAtomics.td"
include "WebAssemblyInstrSIMD.td"
include "WebAssemblyInstrRef.td"
include "WebAssemblyInstrBulkMemory.td"
include "WebAssemblyInstrTable.td"
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Instruction information and lowering helpers / 指令信息与 lowering 辅助逻辑
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyInstrFormats.td`
- `WebAssemblyInstrMemory.td`
- `WebAssemblyInstrCall.td`
- `WebAssemblyInstrControl.td`
- `WebAssemblyInstrConv.td`
- `WebAssemblyInstrInteger.td`
- `WebAssemblyInstrFloat.td`
- `WebAssemblyInstrAtomics.td`
- `WebAssemblyInstrSIMD.td`
- `WebAssemblyInstrRef.td`
- `WebAssemblyInstrBulkMemory.td`
- `WebAssemblyInstrTable.td`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
