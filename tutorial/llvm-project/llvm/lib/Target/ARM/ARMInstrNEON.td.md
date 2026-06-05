# ARMInstrNEON.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMInstrNEON.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the ARM NEON instruction set.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMInstrNEON`，涵盖指令定义与目标操作码元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- ARMInstrNEON.td - NEON support for ARM -------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the ARM NEON instruction set.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 14-19
```tablegen
//===----------------------------------------------------------------------===//
// NEON-specific Operands.
//===----------------------------------------------------------------------===//
def nModImm : Operand<i32> {
  let PrintMethod = "printVMOVModImmOperand";
}
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 21-38
```tablegen
def nImmSplatI8AsmOperand : AsmOperandClass { let Name = "NEONi8splat"; }
def nImmSplatI8 : Operand<i32> {
  let PrintMethod = "printVMOVModImmOperand";
  let ParserMatchClass = nImmSplatI8AsmOperand;
}
def nImmSplatI16AsmOperand : AsmOperandClass { let Name = "NEONi16splat"; }
def nImmSplatI16 : Operand<i32> {
  let PrintMethod = "printVMOVModImmOperand";
  let ParserMatchClass = nImmSplatI16AsmOperand;
}
def nImmSplatI32AsmOperand : AsmOperandClass { let Name = "NEONi32splat"; }
def nImmSplatI32 : Operand<i32> {
  let PrintMethod = "printVMOVModImmOperand";
  let ParserMatchClass = nImmSplatI32AsmOperand;
}
def nImmSplatNotI16AsmOperand : AsmOperandClass { let Name = "NEONi16splatNot"; }
def nImmSplatNotI16 : Operand<i32> {
  let ParserMatchClass = nImmSplatNotI16AsmOperand;
```
- EN: Defines TableGen record `nImmSplatI8AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `nImmSplatI8AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 39-48
```tablegen
}
def nImmSplatNotI32AsmOperand : AsmOperandClass { let Name = "NEONi32splatNot"; }
def nImmSplatNotI32 : Operand<i32> {
  let ParserMatchClass = nImmSplatNotI32AsmOperand;
}
def nImmVMOVI32AsmOperand : AsmOperandClass { let Name = "NEONi32vmov"; }
def nImmVMOVI32 : Operand<i32> {
  let PrintMethod = "printVMOVModImmOperand";
  let ParserMatchClass = nImmVMOVI32AsmOperand;
}
```
- EN: Defines TableGen record `nImmSplatNotI32AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `nImmSplatNotI32AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 50-55
```tablegen
class nImmVMOVIAsmOperandReplicate<ValueType From, ValueType To>
  : AsmOperandClass {
  let Name = "NEONi" # To.Size # "vmovi" # From.Size # "Replicate";
  let PredicateMethod = "isNEONmovReplicate<" # From.Size # ", " # To.Size # ">";
  let RenderMethod = "addNEONvmovi" # From.Size # "ReplicateOperands";
}
```
- EN: Declares reusable TableGen class `nImmVMOVIAsmOperandReplicate` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `nImmVMOVIAsmOperandReplicate`，通常用于抽象共享字段、谓词或编码结构。

### Lines 57-62
```tablegen
class nImmVINVIAsmOperandReplicate<ValueType From, ValueType To>
  : AsmOperandClass {
  let Name = "NEONi" # To.Size # "invi" # From.Size # "Replicate";
  let PredicateMethod = "isNEONinvReplicate<" # From.Size # ", " # To.Size # ">";
  let RenderMethod = "addNEONinvi" # From.Size # "ReplicateOperands";
}
```
- EN: Declares reusable TableGen class `nImmVINVIAsmOperandReplicate` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `nImmVINVIAsmOperandReplicate`，通常用于抽象共享字段、谓词或编码结构。

### Lines 64-67
```tablegen
class nImmVMOVIReplicate<ValueType From, ValueType To> : Operand<i32> {
  let PrintMethod = "printVMOVModImmOperand";
  let ParserMatchClass = nImmVMOVIAsmOperandReplicate<From, To>;
}
```
- EN: Declares reusable TableGen class `nImmVMOVIReplicate` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `nImmVMOVIReplicate`，通常用于抽象共享字段、谓词或编码结构。

### Lines 69-72
```tablegen
class nImmVINVIReplicate<ValueType From, ValueType To> : Operand<i32> {
  let PrintMethod = "printVMOVModImmOperand";
  let ParserMatchClass = nImmVINVIAsmOperandReplicate<From, To>;
}
```
- EN: Declares reusable TableGen class `nImmVINVIReplicate` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `nImmVINVIReplicate`，通常用于抽象共享字段、谓词或编码结构。

### Lines 74-87
```tablegen
def nImmVMOVI32NegAsmOperand : AsmOperandClass { let Name = "NEONi32vmovNeg"; }
def nImmVMOVI32Neg : Operand<i32> {
  let PrintMethod = "printVMOVModImmOperand";
  let ParserMatchClass = nImmVMOVI32NegAsmOperand;
}
def nImmVMOVF32 : Operand<i32> {
  let PrintMethod = "printFPImmOperand";
  let ParserMatchClass = FPImmOperand;
}
def nImmSplatI64AsmOperand : AsmOperandClass { let Name = "NEONi64splat"; }
def nImmSplatI64 : Operand<i32> {
  let PrintMethod = "printVMOVModImmOperand";
  let ParserMatchClass = nImmSplatI64AsmOperand;
}
```
- EN: Defines TableGen record `nImmVMOVI32NegAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `nImmVMOVI32NegAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 89-106
```tablegen
def VectorIndex8Operand  : AsmOperandClass { let Name = "VectorIndex8"; }
def VectorIndex16Operand : AsmOperandClass { let Name = "VectorIndex16"; }
def VectorIndex32Operand : AsmOperandClass { let Name = "VectorIndex32"; }
def VectorIndex64Operand : AsmOperandClass { let Name = "VectorIndex64"; }
def VectorIndex8 : Operand<i32>, ImmLeaf<i32, [{
  return ((uint64_t)Imm) < 8;
}]> {
  let ParserMatchClass = VectorIndex8Operand;
  let PrintMethod = "printVectorIndex";
}
def VectorIndex16 : Operand<i32>, ImmLeaf<i32, [{
  return ((uint64_t)Imm) < 4;
}]> {
  let ParserMatchClass = VectorIndex16Operand;
  let PrintMethod = "printVectorIndex";
}
def VectorIndex32 : Operand<i32>, ImmLeaf<i32, [{
  return ((uint64_t)Imm) < 2;
```
- EN: Defines TableGen record `VectorIndex8Operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VectorIndex8Operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 107-116
```tablegen
}]> {
  let ParserMatchClass = VectorIndex32Operand;
  let PrintMethod = "printVectorIndex";
}
def VectorIndex64 : Operand<i32>, ImmLeaf<i32, [{
  return ((uint64_t)Imm) < 1;
}]> {
  let ParserMatchClass = VectorIndex64Operand;
  let PrintMethod = "printVectorIndex";
}
```
- EN: Defines TableGen record `VectorIndex64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VectorIndex64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 118-123
```tablegen
def VectorIndex32_Hi_XForm  : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(N->getZExtValue() - 2, SDLoc(N), MVT::i32);
}]>;
def VectorIndex32_Hi : ImmLeaf<i32, [{
  return ((uint64_t)Imm) >= 2 && ((uint64_t)Imm) < 4;
}], VectorIndex32_Hi_XForm>;
```
- EN: Defines TableGen record `VectorIndex32_Hi_XForm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `VectorIndex32_Hi_XForm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 125-142
```tablegen
// Register list of one D register.
def VecListOneDAsmOperand : AsmOperandClass {
  let Name = "VecListOneD";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListOneD : RegisterOperand<DPR, "printVectorListOne"> {
  let ParserMatchClass = VecListOneDAsmOperand;
}
// Register list of two sequential D registers.
def VecListDPairAsmOperand : AsmOperandClass {
  let Name = "VecListDPair";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListDPair : RegisterOperand<DPair, "printVectorListTwo"> {
  let ParserMatchClass = VecListDPairAsmOperand;
}
```
- EN: Defines TableGen record `VecListOneDAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListOneDAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 143-160
```tablegen
// Register list of three sequential D registers.
def VecListThreeDAsmOperand : AsmOperandClass {
  let Name = "VecListThreeD";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListThreeD : RegisterOperand<DPR, "printVectorListThree"> {
  let ParserMatchClass = VecListThreeDAsmOperand;
}
// Register list of four sequential D registers.
def VecListFourDAsmOperand : AsmOperandClass {
  let Name = "VecListFourD";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListFourD : RegisterOperand<DPR, "printVectorListFour"> {
  let ParserMatchClass = VecListFourDAsmOperand;
}
```
- EN: Defines TableGen record `VecListThreeDAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListThreeDAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 161-178
```tablegen
// Register list of two D registers spaced by 2 (two sequential Q registers).
def VecListDPairSpacedAsmOperand : AsmOperandClass {
  let Name = "VecListDPairSpaced";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListDPairSpaced : RegisterOperand<DPair, "printVectorListTwoSpaced"> {
  let ParserMatchClass = VecListDPairSpacedAsmOperand;
}
// Register list of three D registers spaced by 2 (three Q registers).
def VecListThreeQAsmOperand : AsmOperandClass {
  let Name = "VecListThreeQ";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListThreeQ : RegisterOperand<DPR, "printVectorListThreeSpaced"> {
  let ParserMatchClass = VecListThreeQAsmOperand;
}
```
- EN: Defines TableGen record `VecListDPairSpacedAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListDPairSpacedAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 179-187
```tablegen
// Register list of three D registers spaced by 2 (three Q registers).
def VecListFourQAsmOperand : AsmOperandClass {
  let Name = "VecListFourQ";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListFourQ : RegisterOperand<DPR, "printVectorListFourSpaced"> {
  let ParserMatchClass = VecListFourQAsmOperand;
}
```
- EN: Defines TableGen record `VecListFourQAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListFourQAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 189-206
```tablegen
// Register list of one D register, with "all lanes" subscripting.
def VecListOneDAllLanesAsmOperand : AsmOperandClass {
  let Name = "VecListOneDAllLanes";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListOneDAllLanes : RegisterOperand<DPR, "printVectorListOneAllLanes"> {
  let ParserMatchClass = VecListOneDAllLanesAsmOperand;
}
// Register list of two D registers, with "all lanes" subscripting.
def VecListDPairAllLanesAsmOperand : AsmOperandClass {
  let Name = "VecListDPairAllLanes";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListDPairAllLanes : RegisterOperand<DPair,
                                           "printVectorListTwoAllLanes"> {
  let ParserMatchClass = VecListDPairAllLanesAsmOperand;
```
- EN: Defines TableGen record `VecListOneDAllLanesAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListOneDAllLanesAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 207-224
```tablegen
}
// Register list of two D registers spaced by 2 (two sequential Q registers).
def VecListDPairSpacedAllLanesAsmOperand : AsmOperandClass {
  let Name = "VecListDPairSpacedAllLanes";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListDPairSpacedAllLanes : RegisterOperand<DPairSpc,
                                         "printVectorListTwoSpacedAllLanes"> {
  let ParserMatchClass = VecListDPairSpacedAllLanesAsmOperand;
}
// Register list of three D registers, with "all lanes" subscripting.
def VecListThreeDAllLanesAsmOperand : AsmOperandClass {
  let Name = "VecListThreeDAllLanes";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListThreeDAllLanes : RegisterOperand<DPR,
```
- EN: Defines TableGen record `VecListDPairSpacedAllLanesAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListDPairSpacedAllLanesAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 225-242
```tablegen
                                            "printVectorListThreeAllLanes"> {
  let ParserMatchClass = VecListThreeDAllLanesAsmOperand;
}
// Register list of three D registers spaced by 2 (three sequential Q regs).
def VecListThreeQAllLanesAsmOperand : AsmOperandClass {
  let Name = "VecListThreeQAllLanes";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListThreeQAllLanes : RegisterOperand<DPR,
                                         "printVectorListThreeSpacedAllLanes"> {
  let ParserMatchClass = VecListThreeQAllLanesAsmOperand;
}
// Register list of four D registers, with "all lanes" subscripting.
def VecListFourDAllLanesAsmOperand : AsmOperandClass {
  let Name = "VecListFourDAllLanes";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
```
- EN: Defines TableGen record `VecListThreeQAllLanesAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListThreeQAllLanesAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 243-256
```tablegen
}
def VecListFourDAllLanes : RegisterOperand<DPR, "printVectorListFourAllLanes"> {
  let ParserMatchClass = VecListFourDAllLanesAsmOperand;
}
// Register list of four D registers spaced by 2 (four sequential Q regs).
def VecListFourQAllLanesAsmOperand : AsmOperandClass {
  let Name = "VecListFourQAllLanes";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListOperands";
}
def VecListFourQAllLanes : RegisterOperand<DPR,
                                         "printVectorListFourSpacedAllLanes"> {
  let ParserMatchClass = VecListFourQAllLanesAsmOperand;
}
```
- EN: Defines TableGen record `VecListFourDAllLanes` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListFourDAllLanes`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 259-276
```tablegen
// Register list of one D register, with byte lane subscripting.
def VecListOneDByteIndexAsmOperand : AsmOperandClass {
  let Name = "VecListOneDByteIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListOneDByteIndexed : Operand<i32> {
  let ParserMatchClass = VecListOneDByteIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// ...with half-word lane subscripting.
def VecListOneDHWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListOneDHWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListOneDHWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListOneDHWordIndexAsmOperand;
```
- EN: Defines TableGen record `VecListOneDByteIndexAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListOneDByteIndexAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 277-288
```tablegen
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// ...with word lane subscripting.
def VecListOneDWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListOneDWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListOneDWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListOneDWordIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
```
- EN: Defines TableGen record `VecListOneDWordIndexAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListOneDWordIndexAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 290-307
```tablegen
// Register list of two D registers with byte lane subscripting.
def VecListTwoDByteIndexAsmOperand : AsmOperandClass {
  let Name = "VecListTwoDByteIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListTwoDByteIndexed : Operand<i32> {
  let ParserMatchClass = VecListTwoDByteIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// ...with half-word lane subscripting.
def VecListTwoDHWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListTwoDHWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListTwoDHWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListTwoDHWordIndexAsmOperand;
```
- EN: Defines TableGen record `VecListTwoDByteIndexAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListTwoDByteIndexAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 308-325
```tablegen
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// ...with word lane subscripting.
def VecListTwoDWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListTwoDWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListTwoDWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListTwoDWordIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// Register list of two Q registers with half-word lane subscripting.
def VecListTwoQHWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListTwoQHWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
```
- EN: Defines TableGen record `VecListTwoDWordIndexAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListTwoDWordIndexAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 326-339
```tablegen
def VecListTwoQHWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListTwoQHWordIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// ...with word lane subscripting.
def VecListTwoQWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListTwoQWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListTwoQWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListTwoQWordIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
```
- EN: Defines TableGen record `VecListTwoQHWordIndexed` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListTwoQHWordIndexed`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 342-359
```tablegen
// Register list of three D registers with byte lane subscripting.
def VecListThreeDByteIndexAsmOperand : AsmOperandClass {
  let Name = "VecListThreeDByteIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListThreeDByteIndexed : Operand<i32> {
  let ParserMatchClass = VecListThreeDByteIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// ...with half-word lane subscripting.
def VecListThreeDHWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListThreeDHWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListThreeDHWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListThreeDHWordIndexAsmOperand;
```
- EN: Defines TableGen record `VecListThreeDByteIndexAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListThreeDByteIndexAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 360-377
```tablegen
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// ...with word lane subscripting.
def VecListThreeDWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListThreeDWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListThreeDWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListThreeDWordIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// Register list of three Q registers with half-word lane subscripting.
def VecListThreeQHWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListThreeQHWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
```
- EN: Defines TableGen record `VecListThreeDWordIndexAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListThreeDWordIndexAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 378-391
```tablegen
def VecListThreeQHWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListThreeQHWordIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// ...with word lane subscripting.
def VecListThreeQWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListThreeQWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListThreeQWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListThreeQWordIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
```
- EN: Defines TableGen record `VecListThreeQHWordIndexed` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListThreeQHWordIndexed`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 393-410
```tablegen
// Register list of four D registers with byte lane subscripting.
def VecListFourDByteIndexAsmOperand : AsmOperandClass {
  let Name = "VecListFourDByteIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListFourDByteIndexed : Operand<i32> {
  let ParserMatchClass = VecListFourDByteIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// ...with half-word lane subscripting.
def VecListFourDHWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListFourDHWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListFourDHWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListFourDHWordIndexAsmOperand;
```
- EN: Defines TableGen record `VecListFourDByteIndexAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListFourDByteIndexAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 411-428
```tablegen
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// ...with word lane subscripting.
def VecListFourDWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListFourDWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListFourDWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListFourDWordIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// Register list of four Q registers with half-word lane subscripting.
def VecListFourQHWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListFourQHWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
```
- EN: Defines TableGen record `VecListFourDWordIndexAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListFourDWordIndexAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 429-442
```tablegen
def VecListFourQHWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListFourQHWordIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
// ...with word lane subscripting.
def VecListFourQWordIndexAsmOperand : AsmOperandClass {
  let Name = "VecListFourQWordIndexed";
  let ParserMethod = "parseVectorList";
  let RenderMethod = "addVecListIndexedOperands";
}
def VecListFourQWordIndexed : Operand<i32> {
  let ParserMatchClass = VecListFourQWordIndexAsmOperand;
  let MIOperandInfo = (ops DPR:$Vd, i32imm:$idx);
}
```
- EN: Defines TableGen record `VecListFourQHWordIndexed` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VecListFourQHWordIndexed`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 444-461
```tablegen
def dword_alignedload : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return cast<LoadSDNode>(N)->getAlign() >= 8;
}]>;
def dword_alignedstore : PatFrag<(ops node:$val, node:$ptr),
                                 (store node:$val, node:$ptr), [{
  return cast<StoreSDNode>(N)->getAlign() >= 8;
}]>;
def word_alignedload : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return cast<LoadSDNode>(N)->getAlign() == 4;
}]>;
def word_alignedstore : PatFrag<(ops node:$val, node:$ptr),
                                 (store node:$val, node:$ptr), [{
  return cast<StoreSDNode>(N)->getAlign() == 4;
}]>;
def hword_alignedload : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return cast<LoadSDNode>(N)->getAlign() == 2;
}]>;
def hword_alignedstore : PatFrag<(ops node:$val, node:$ptr),
```
- EN: Defines TableGen record `dword_alignedload` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `dword_alignedload`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 462-478
```tablegen
                                 (store node:$val, node:$ptr), [{
  return cast<StoreSDNode>(N)->getAlign() == 2;
}]>;
def byte_alignedload : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return cast<LoadSDNode>(N)->getAlign() == 1;
}]>;
def byte_alignedstore : PatFrag<(ops node:$val, node:$ptr),
                             (store node:$val, node:$ptr), [{
  return cast<StoreSDNode>(N)->getAlign() == 1;
}]>;
def non_word_alignedload : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return cast<LoadSDNode>(N)->getAlign() < 4;
}]>;
def non_word_alignedstore : PatFrag<(ops node:$val, node:$ptr),
                                    (store node:$val, node:$ptr), [{
  return cast<StoreSDNode>(N)->getAlign() < 4;
}]>;
```
- EN: Defines TableGen record `byte_alignedload` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `byte_alignedload`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 480-482
```tablegen
//===----------------------------------------------------------------------===//
// NEON-specific DAG Nodes.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 484-484
```tablegen
def SDTARMVTST    : SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisSameAs<1, 2>]>;
```
- EN: Defines TableGen record `SDTARMVTST` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVTST`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 486-487
```tablegen
// Vector test bits.
def NEONvtst      : SDNode<"ARMISD::VTST", SDTARMVTST>;
```
- EN: Defines TableGen record `NEONvtst` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONvtst`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 489-495
```tablegen
// Types for vector shift by immediates.  The "SHX" version is for long and
// narrow operations where the source and destination vectors have different
// types.  The "SHINS" version is for shift and insert operations.
def SDTARMVSHXIMM    : SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisInt<1>,
                                            SDTCisVT<2, i32>]>;
def SDTARMVSHINSIMM  : SDTypeProfile<1, 3, [SDTCisInt<0>, SDTCisSameAs<0, 1>,
                                            SDTCisSameAs<0, 2>, SDTCisVT<3, i32>]>;
```
- EN: Defines TableGen record `SDTARMVSHXIMM` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVSHXIMM`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 497-497
```tablegen
def NEONvshrnImm     : SDNode<"ARMISD::VSHRNIMM", SDTARMVSHXIMM>;
```
- EN: Defines TableGen record `NEONvshrnImm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONvshrnImm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 499-502
```tablegen
// Vector rounding shift by immediate
def NEONvrshrsImm    : SDNode<"ARMISD::VRSHRsIMM", SDTARMVSHIMM>;
def NEONvrshruImm    : SDNode<"ARMISD::VRSHRuIMM", SDTARMVSHIMM>;
def NEONvrshrnImm    : SDNode<"ARMISD::VRSHRNIMM", SDTARMVSHXIMM>;
```
- EN: Defines TableGen record `NEONvrshrsImm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONvrshrsImm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 504-510
```tablegen
// Vector saturating shift by immediate
def NEONvqshlsImm    : SDNode<"ARMISD::VQSHLsIMM", SDTARMVSHIMM>;
def NEONvqshluImm    : SDNode<"ARMISD::VQSHLuIMM", SDTARMVSHIMM>;
def NEONvqshlsuImm   : SDNode<"ARMISD::VQSHLsuIMM", SDTARMVSHIMM>;
def NEONvqshrnsImm   : SDNode<"ARMISD::VQSHRNsIMM", SDTARMVSHXIMM>;
def NEONvqshrnuImm   : SDNode<"ARMISD::VQSHRNuIMM", SDTARMVSHXIMM>;
def NEONvqshrnsuImm  : SDNode<"ARMISD::VQSHRNsuIMM", SDTARMVSHXIMM>;
```
- EN: Defines TableGen record `NEONvqshlsImm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONvqshlsImm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 512-515
```tablegen
// Vector saturating rounding shift by immediate
def NEONvqrshrnsImm  : SDNode<"ARMISD::VQRSHRNsIMM", SDTARMVSHXIMM>;
def NEONvqrshrnuImm  : SDNode<"ARMISD::VQRSHRNuIMM", SDTARMVSHXIMM>;
def NEONvqrshrnsuImm : SDNode<"ARMISD::VQRSHRNsuIMM", SDTARMVSHXIMM>;
```
- EN: Defines TableGen record `NEONvqrshrnsImm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONvqrshrnsImm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 517-519
```tablegen
// Vector shift and insert
def NEONvsliImm      : SDNode<"ARMISD::VSLIIMM", SDTARMVSHINSIMM>;
def NEONvsriImm      : SDNode<"ARMISD::VSRIIMM", SDTARMVSHINSIMM>;
```
- EN: Defines TableGen record `NEONvsliImm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONvsliImm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 521-526
```tablegen
// Pseudo vector bitwise select
def NEONvbsp      : SDNode<"ARMISD::VBSP",
                           SDTypeProfile<1, 3, [SDTCisVec<0>,
                                                SDTCisSameAs<0, 1>,
                                                SDTCisSameAs<0, 2>,
                                                SDTCisSameAs<0, 3>]>>;
```
- EN: Defines TableGen record `NEONvbsp` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONvbsp`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 528-530
```tablegen
def SDTARMVEXT    : SDTypeProfile<1, 3, [SDTCisVec<0>, SDTCisSameAs<0, 1>,
                                         SDTCisSameAs<0, 2>, SDTCisVT<3, i32>]>;
def NEONvext      : SDNode<"ARMISD::VEXT", SDTARMVEXT>;
```
- EN: Defines TableGen record `SDTARMVEXT` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVEXT`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 532-534
```tablegen
def SDTARMVSHUF2  : SDTypeProfile<2, 2, [SDTCisVec<0>, SDTCisSameAs<0, 1>,
                                         SDTCisSameAs<0, 2>,
                                         SDTCisSameAs<0, 3>]>;
```
- EN: Defines TableGen record `SDTARMVSHUF2` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVSHUF2`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 536-537
```tablegen
// zip (interleave)
def NEONzip       : SDNode<"ARMISD::VZIP", SDTARMVSHUF2>;
```
- EN: Defines TableGen record `NEONzip` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONzip`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 539-540
```tablegen
// unzip (deinterleave)
def NEONuzp       : SDNode<"ARMISD::VUZP", SDTARMVSHUF2>;
```
- EN: Defines TableGen record `NEONuzp` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONuzp`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 542-543
```tablegen
// transpose
def NEONtrn       : SDNode<"ARMISD::VTRN", SDTARMVSHUF2>;
```
- EN: Defines TableGen record `NEONtrn` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONtrn`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 545-548
```tablegen
def SDTARMVTBL1   : SDTypeProfile<1, 2, [SDTCisVT<0, v8i8>, SDTCisVT<1, v8i8>,
                                         SDTCisVT<2, v8i8>]>;
def SDTARMVTBL2   : SDTypeProfile<1, 3, [SDTCisVT<0, v8i8>, SDTCisVT<1, v8i8>,
                                         SDTCisVT<2, v8i8>, SDTCisVT<3, v8i8>]>;
```
- EN: Defines TableGen record `SDTARMVTBL1` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVTBL1`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 550-551
```tablegen
// 1-register shuffle with mask
def NEONvtbl1     : SDNode<"ARMISD::VTBL1", SDTARMVTBL1>;
```
- EN: Defines TableGen record `NEONvtbl1` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONvtbl1`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 553-554
```tablegen
// 2-register shuffle with mask
def NEONvtbl2     : SDNode<"ARMISD::VTBL2", SDTARMVTBL2>;
```
- EN: Defines TableGen record `NEONvtbl2` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `NEONvtbl2`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 557-559
```tablegen
//===----------------------------------------------------------------------===//
// NEON load / store instructions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 561-566
```tablegen
// Use VLDM to load a Q register as a D register pair.
// This is a pseudo instruction that is expanded to VLDMD after reg alloc.
def VLDMQIA
  : PseudoVFPLdStM<(outs DPair:$dst), (ins GPR:$Rn),
                    IIC_fpLoad_m, "",
                   [(set DPair:$dst, (v2f64 (word_alignedload GPR:$Rn)))]>;
```
- EN: Defines TableGen record `VLDMQIA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLDMQIA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 568-573
```tablegen
// Use VSTM to store a Q register as a D register pair.
// This is a pseudo instruction that is expanded to VSTMD after reg alloc.
def VSTMQIA
  : PseudoVFPLdStM<(outs), (ins DPair:$src, GPR:$Rn),
                    IIC_fpStore_m, "",
                   [(word_alignedstore (v2f64 DPair:$src), GPR:$Rn)]>;
```
- EN: Defines TableGen record `VSTMQIA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSTMQIA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 575-590
```tablegen
// Classes for VLD* pseudo-instructions with multi-register operands.
// These are expanded to real instructions after register allocation.
class VLDQPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QPR:$dst), (ins addrmode6:$addr), itin, "">;
class VLDQWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QPR:$dst, GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset), itin,
                "$addr.addr = $wb">;
class VLDQWBfixedPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QPR:$dst, GPR:$wb),
                (ins addrmode6:$addr), itin,
                "$addr.addr = $wb">;
class VLDQWBregisterPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QPR:$dst, GPR:$wb),
                (ins addrmode6:$addr, rGPR:$offset), itin,
                "$addr.addr = $wb">;
```
- EN: Declares reusable TableGen class `VLDQPseudo` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLDQPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 592-605
```tablegen
class VLDQQPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QQPR:$dst), (ins addrmode6:$addr), itin, "">;
class VLDQQWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QQPR:$dst, GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset), itin,
                "$addr.addr = $wb">;
class VLDQQWBfixedPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QQPR:$dst, GPR:$wb),
                (ins addrmode6:$addr), itin,
                "$addr.addr = $wb">;
class VLDQQWBregisterPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QQPR:$dst, GPR:$wb),
                (ins addrmode6:$addr, rGPR:$offset), itin,
                "$addr.addr = $wb">;
```
- EN: Declares reusable TableGen class `VLDQQPseudo` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLDQQPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 608-614
```tablegen
class VLDQQQQPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QQQQPR:$dst), (ins addrmode6:$addr, QQQQPR:$src),itin,
                "$src = $dst">;
class VLDQQQQWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QQQQPR:$dst, GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset, QQQQPR:$src), itin,
                "$addr.addr = $wb, $src = $dst">;
```
- EN: Declares reusable TableGen class `VLDQQQQPseudo` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLDQQQQPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 616-616
```tablegen
let mayLoad = 1, hasSideEffects = 0, hasExtraDefRegAllocReq = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 618-634
```tablegen
//   VLD1     : Vector Load (multiple single elements)
class VLD1D<bits<4> op7_4, string Dt, Operand AddrMode>
  : NLdSt<0,0b10,0b0111,op7_4, (outs VecListOneD:$Vd),
          (ins AddrMode:$Rn), IIC_VLD1,
          "vld1", Dt, "$Vd, $Rn", "", []>, Sched<[WriteVLD1]> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLDST1Instruction";
}
class VLD1Q<bits<4> op7_4, string Dt, Operand AddrMode>
  : NLdSt<0,0b10,0b1010,op7_4, (outs VecListDPair:$Vd),
          (ins AddrMode:$Rn), IIC_VLD1x2,
          "vld1", Dt, "$Vd, $Rn", "", []>, Sched<[WriteVLD2]> {
  let Rm = 0b1111;
  let Inst{5-4} = Rn{5-4};
  let DecoderMethod = "DecodeVLDST1Instruction";
}
```
- EN: Declares reusable TableGen class `VLD1D` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD1D`，通常用于抽象共享字段、谓词或编码结构。

### Lines 636-639
```tablegen
def  VLD1d8   : VLD1D<{0,0,0,?}, "8",  addrmode6align64>;
def  VLD1d16  : VLD1D<{0,1,0,?}, "16", addrmode6align64>;
def  VLD1d32  : VLD1D<{1,0,0,?}, "32", addrmode6align64>;
def  VLD1d64  : VLD1D<{1,1,0,?}, "64", addrmode6align64>;
```
- EN: Defines TableGen record `VLD1d8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1d8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 641-644
```tablegen
def  VLD1q8   : VLD1Q<{0,0,?,?}, "8",  addrmode6align64or128>;
def  VLD1q16  : VLD1Q<{0,1,?,?}, "16", addrmode6align64or128>;
def  VLD1q32  : VLD1Q<{1,0,?,?}, "32", addrmode6align64or128>;
def  VLD1q64  : VLD1Q<{1,1,?,?}, "64", addrmode6align64or128>;
```
- EN: Defines TableGen record `VLD1q8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1q8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 646-663
```tablegen
// ...with address register writeback:
multiclass VLD1DWB<bits<4> op7_4, string Dt, Operand AddrMode> {
  def _fixed : NLdSt<0,0b10, 0b0111,op7_4, (outs VecListOneD:$Vd, GPR:$wb),
                     (ins AddrMode:$Rn), IIC_VLD1u,
                     "vld1", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVLD1]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{4} = Rn{4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
  def _register : NLdSt<0,0b10,0b0111,op7_4, (outs VecListOneD:$Vd, GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm), IIC_VLD1u,
                        "vld1", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []>, Sched<[WriteVLD1]> {
    let Inst{4} = Rn{4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
}
```
- EN: Declares TableGen `multiclass VLD1DWB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VLD1DWB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 664-680
```tablegen
multiclass VLD1QWB<bits<4> op7_4, string Dt, Operand AddrMode> {
  def _fixed : NLdSt<0,0b10,0b1010,op7_4, (outs VecListDPair:$Vd, GPR:$wb),
                    (ins AddrMode:$Rn), IIC_VLD1x2u,
                     "vld1", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVLD2]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
  def _register : NLdSt<0,0b10,0b1010,op7_4, (outs VecListDPair:$Vd, GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm), IIC_VLD1x2u,
                        "vld1", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []>, Sched<[WriteVLD2]> {
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
}
```
- EN: Declares TableGen `multiclass VLD1QWB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VLD1QWB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 682-689
```tablegen
defm VLD1d8wb  : VLD1DWB<{0,0,0,?}, "8",  addrmode6align64>;
defm VLD1d16wb : VLD1DWB<{0,1,0,?}, "16", addrmode6align64>;
defm VLD1d32wb : VLD1DWB<{1,0,0,?}, "32", addrmode6align64>;
defm VLD1d64wb : VLD1DWB<{1,1,0,?}, "64", addrmode6align64>;
defm VLD1q8wb  : VLD1QWB<{0,0,?,?}, "8",  addrmode6align64or128>;
defm VLD1q16wb : VLD1QWB<{0,1,?,?}, "16", addrmode6align64or128>;
defm VLD1q32wb : VLD1QWB<{1,0,?,?}, "32", addrmode6align64or128>;
defm VLD1q64wb : VLD1QWB<{1,1,?,?}, "64", addrmode6align64or128>;
```
- EN: Defines TableGen record `VLD1d8wb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1d8wb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 691-708
```tablegen
// ...with 3 registers
class VLD1D3<bits<4> op7_4, string Dt, Operand AddrMode>
  : NLdSt<0,0b10,0b0110,op7_4, (outs VecListThreeD:$Vd),
          (ins AddrMode:$Rn), IIC_VLD1x3, "vld1", Dt,
          "$Vd, $Rn", "", []>, Sched<[WriteVLD3]> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLDST1Instruction";
}
multiclass VLD1D3WB<bits<4> op7_4, string Dt, Operand AddrMode> {
  def _fixed : NLdSt<0,0b10,0b0110, op7_4, (outs VecListThreeD:$Vd, GPR:$wb),
                    (ins AddrMode:$Rn), IIC_VLD1x2u,
                     "vld1", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVLD3]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{4} = Rn{4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
```
- EN: Declares TableGen `multiclass VLD1D3WB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VLD1D3WB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 709-716
```tablegen
  def _register : NLdSt<0,0b10,0b0110,op7_4, (outs VecListThreeD:$Vd, GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm), IIC_VLD1x2u,
                        "vld1", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []>, Sched<[WriteVLD3]> {
    let Inst{4} = Rn{4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
}
```
- EN: Defines TableGen record `_register` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_register`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 718-721
```tablegen
def VLD1d8T      : VLD1D3<{0,0,0,?}, "8",  addrmode6align64>;
def VLD1d16T     : VLD1D3<{0,1,0,?}, "16", addrmode6align64>;
def VLD1d32T     : VLD1D3<{1,0,0,?}, "32", addrmode6align64>;
def VLD1d64T     : VLD1D3<{1,1,0,?}, "64", addrmode6align64>;
```
- EN: Defines TableGen record `VLD1d8T` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1d8T`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 723-726
```tablegen
defm VLD1d8Twb  : VLD1D3WB<{0,0,0,?}, "8",  addrmode6align64>;
defm VLD1d16Twb : VLD1D3WB<{0,1,0,?}, "16", addrmode6align64>;
defm VLD1d32Twb : VLD1D3WB<{1,0,0,?}, "32", addrmode6align64>;
defm VLD1d64Twb : VLD1D3WB<{1,1,0,?}, "64", addrmode6align64>;
```
- EN: Defines TableGen record `VLD1d8Twb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1d8Twb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 728-739
```tablegen
def VLD1d8TPseudo             : VLDQQPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1d8TPseudoWB_fixed     : VLDQQWBfixedPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1d8TPseudoWB_register  : VLDQQWBregisterPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1d16TPseudo            : VLDQQPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1d16TPseudoWB_fixed    : VLDQQWBfixedPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1d16TPseudoWB_register : VLDQQWBregisterPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1d32TPseudo            : VLDQQPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1d32TPseudoWB_fixed    : VLDQQWBfixedPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1d32TPseudoWB_register : VLDQQWBregisterPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1d64TPseudo            : VLDQQPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1d64TPseudoWB_fixed    : VLDQQWBfixedPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1d64TPseudoWB_register : VLDQQWBregisterPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
```
- EN: Defines TableGen record `VLD1d8TPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1d8TPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 741-752
```tablegen
def VLD1q8HighTPseudo      : VLDQQQQPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1q8HighTPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1q8LowTPseudo_UPD   : VLDQQQQWBPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1q16HighTPseudo     : VLDQQQQPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1q16HighTPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1q16LowTPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1q32HighTPseudo     : VLDQQQQPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1q32HighTPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1q32LowTPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1q64HighTPseudo     : VLDQQQQPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1q64HighTPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
def VLD1q64LowTPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD1x3>, Sched<[WriteVLD3]>;
```
- EN: Defines TableGen record `VLD1q8HighTPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1q8HighTPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 754-771
```tablegen
// ...with 4 registers
class VLD1D4<bits<4> op7_4, string Dt, Operand AddrMode>
  : NLdSt<0, 0b10, 0b0010, op7_4, (outs VecListFourD:$Vd),
          (ins AddrMode:$Rn), IIC_VLD1x4, "vld1", Dt,
          "$Vd, $Rn", "", []>, Sched<[WriteVLD4]> {
  let Rm = 0b1111;
  let Inst{5-4} = Rn{5-4};
  let DecoderMethod = "DecodeVLDST1Instruction";
}
multiclass VLD1D4WB<bits<4> op7_4, string Dt, Operand AddrMode> {
  def _fixed : NLdSt<0,0b10,0b0010, op7_4, (outs VecListFourD:$Vd, GPR:$wb),
                    (ins AddrMode:$Rn), IIC_VLD1x2u,
                     "vld1", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVLD4]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
```
- EN: Declares TableGen `multiclass VLD1D4WB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VLD1D4WB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 772-779
```tablegen
  def _register : NLdSt<0,0b10,0b0010,op7_4, (outs VecListFourD:$Vd, GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm), IIC_VLD1x2u,
                        "vld1", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []>, Sched<[WriteVLD4]> {
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
}
```
- EN: Defines TableGen record `_register` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_register`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 781-784
```tablegen
def VLD1d8Q      : VLD1D4<{0,0,?,?}, "8",  addrmode6align64or128or256>;
def VLD1d16Q     : VLD1D4<{0,1,?,?}, "16", addrmode6align64or128or256>;
def VLD1d32Q     : VLD1D4<{1,0,?,?}, "32", addrmode6align64or128or256>;
def VLD1d64Q     : VLD1D4<{1,1,?,?}, "64", addrmode6align64or128or256>;
```
- EN: Defines TableGen record `VLD1d8Q` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1d8Q`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 786-789
```tablegen
defm VLD1d8Qwb   : VLD1D4WB<{0,0,?,?}, "8",  addrmode6align64or128or256>;
defm VLD1d16Qwb  : VLD1D4WB<{0,1,?,?}, "16", addrmode6align64or128or256>;
defm VLD1d32Qwb  : VLD1D4WB<{1,0,?,?}, "32", addrmode6align64or128or256>;
defm VLD1d64Qwb  : VLD1D4WB<{1,1,?,?}, "64", addrmode6align64or128or256>;
```
- EN: Defines TableGen record `VLD1d8Qwb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1d8Qwb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 791-802
```tablegen
def VLD1d8QPseudo             : VLDQQPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1d8QPseudoWB_fixed     : VLDQQWBfixedPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1d8QPseudoWB_register  : VLDQQWBregisterPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1d16QPseudo            : VLDQQPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1d16QPseudoWB_fixed    : VLDQQWBfixedPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1d16QPseudoWB_register : VLDQQWBregisterPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1d32QPseudo            : VLDQQPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1d32QPseudoWB_fixed    : VLDQQWBfixedPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1d32QPseudoWB_register : VLDQQWBregisterPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1d64QPseudo            : VLDQQPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1d64QPseudoWB_fixed    : VLDQQWBfixedPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1d64QPseudoWB_register : VLDQQWBregisterPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
```
- EN: Defines TableGen record `VLD1d8QPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1d8QPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 804-815
```tablegen
def VLD1q8LowQPseudo_UPD   : VLDQQQQWBPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1q8HighQPseudo      : VLDQQQQPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1q8HighQPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1q16LowQPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1q16HighQPseudo     : VLDQQQQPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1q16HighQPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1q32LowQPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1q32HighQPseudo     : VLDQQQQPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1q32HighQPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1q64LowQPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1q64HighQPseudo     : VLDQQQQPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
def VLD1q64HighQPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD1x4>, Sched<[WriteVLD4]>;
```
- EN: Defines TableGen record `VLD1q8LowQPseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1q8LowQPseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 817-826
```tablegen
//   VLD2     : Vector Load (multiple 2-element structures)
class VLD2<bits<4> op11_8, bits<4> op7_4, string Dt, RegisterOperand VdTy,
           InstrItinClass itin, Operand AddrMode>
  : NLdSt<0, 0b10, op11_8, op7_4, (outs VdTy:$Vd),
          (ins AddrMode:$Rn), itin,
          "vld2", Dt, "$Vd, $Rn", "", []> {
  let Rm = 0b1111;
  let Inst{5-4} = Rn{5-4};
  let DecoderMethod = "DecodeVLDST2Instruction";
}
```
- EN: Declares reusable TableGen class `VLD2` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 828-833
```tablegen
def  VLD2d8   : VLD2<0b1000, {0,0,?,?}, "8", VecListDPair, IIC_VLD2,
                     addrmode6align64or128>, Sched<[WriteVLD2]>;
def  VLD2d16  : VLD2<0b1000, {0,1,?,?}, "16", VecListDPair, IIC_VLD2,
                     addrmode6align64or128>, Sched<[WriteVLD2]>;
def  VLD2d32  : VLD2<0b1000, {1,0,?,?}, "32", VecListDPair, IIC_VLD2,
                     addrmode6align64or128>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD2d8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2d8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 835-840
```tablegen
def  VLD2q8   : VLD2<0b0011, {0,0,?,?}, "8", VecListFourD, IIC_VLD2x2,
                     addrmode6align64or128or256>, Sched<[WriteVLD4]>;
def  VLD2q16  : VLD2<0b0011, {0,1,?,?}, "16", VecListFourD, IIC_VLD2x2,
                     addrmode6align64or128or256>, Sched<[WriteVLD4]>;
def  VLD2q32  : VLD2<0b0011, {1,0,?,?}, "32", VecListFourD, IIC_VLD2x2,
                     addrmode6align64or128or256>, Sched<[WriteVLD4]>;
```
- EN: Defines TableGen record `VLD2q8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2q8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 842-844
```tablegen
def  VLD2q8Pseudo  : VLDQQPseudo<IIC_VLD2x2>, Sched<[WriteVLD4]>;
def  VLD2q16Pseudo : VLDQQPseudo<IIC_VLD2x2>, Sched<[WriteVLD4]>;
def  VLD2q32Pseudo : VLDQQPseudo<IIC_VLD2x2>, Sched<[WriteVLD4]>;
```
- EN: Defines TableGen record `VLD2q8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2q8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 846-863
```tablegen
// ...with address register writeback:
multiclass VLD2WB<bits<4> op11_8, bits<4> op7_4, string Dt,
                  RegisterOperand VdTy, InstrItinClass itin, Operand AddrMode> {
  def _fixed : NLdSt<0, 0b10, op11_8, op7_4, (outs VdTy:$Vd, GPR:$wb),
                     (ins AddrMode:$Rn), itin,
                     "vld2", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST2Instruction";
  }
  def _register : NLdSt<0, 0b10, op11_8, op7_4, (outs VdTy:$Vd, GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm), itin,
                        "vld2", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []> {
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST2Instruction";
  }
```
- EN: Declares TableGen `multiclass VLD2WB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VLD2WB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 864-864
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 866-871
```tablegen
defm VLD2d8wb  : VLD2WB<0b1000, {0,0,?,?}, "8", VecListDPair, IIC_VLD2u,
                        addrmode6align64or128>, Sched<[WriteVLD2]>;
defm VLD2d16wb : VLD2WB<0b1000, {0,1,?,?}, "16", VecListDPair, IIC_VLD2u,
                        addrmode6align64or128>, Sched<[WriteVLD2]>;
defm VLD2d32wb : VLD2WB<0b1000, {1,0,?,?}, "32", VecListDPair, IIC_VLD2u,
                        addrmode6align64or128>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD2d8wb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2d8wb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 873-878
```tablegen
defm VLD2q8wb  : VLD2WB<0b0011, {0,0,?,?}, "8", VecListFourD, IIC_VLD2x2u,
                        addrmode6align64or128or256>, Sched<[WriteVLD4]>;
defm VLD2q16wb : VLD2WB<0b0011, {0,1,?,?}, "16", VecListFourD, IIC_VLD2x2u,
                        addrmode6align64or128or256>, Sched<[WriteVLD4]>;
defm VLD2q32wb : VLD2WB<0b0011, {1,0,?,?}, "32", VecListFourD, IIC_VLD2x2u,
                        addrmode6align64or128or256>, Sched<[WriteVLD4]>;
```
- EN: Defines TableGen record `VLD2q8wb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2q8wb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 880-885
```tablegen
def VLD2q8PseudoWB_fixed     : VLDQQWBfixedPseudo<IIC_VLD2x2u>, Sched<[WriteVLD4]>;
def VLD2q16PseudoWB_fixed    : VLDQQWBfixedPseudo<IIC_VLD2x2u>, Sched<[WriteVLD4]>;
def VLD2q32PseudoWB_fixed    : VLDQQWBfixedPseudo<IIC_VLD2x2u>, Sched<[WriteVLD4]>;
def VLD2q8PseudoWB_register  : VLDQQWBregisterPseudo<IIC_VLD2x2u>, Sched<[WriteVLD4]>;
def VLD2q16PseudoWB_register : VLDQQWBregisterPseudo<IIC_VLD2x2u>, Sched<[WriteVLD4]>;
def VLD2q32PseudoWB_register : VLDQQWBregisterPseudo<IIC_VLD2x2u>, Sched<[WriteVLD4]>;
```
- EN: Defines TableGen record `VLD2q8PseudoWB_fixed` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2q8PseudoWB_fixed`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 887-899
```tablegen
// ...with double-spaced registers
def  VLD2b8    : VLD2<0b1001, {0,0,?,?}, "8", VecListDPairSpaced, IIC_VLD2,
                      addrmode6align64or128>, Sched<[WriteVLD2]>;
def  VLD2b16   : VLD2<0b1001, {0,1,?,?}, "16", VecListDPairSpaced, IIC_VLD2,
                      addrmode6align64or128>, Sched<[WriteVLD2]>;
def  VLD2b32   : VLD2<0b1001, {1,0,?,?}, "32", VecListDPairSpaced, IIC_VLD2,
                      addrmode6align64or128>, Sched<[WriteVLD2]>;
defm VLD2b8wb  : VLD2WB<0b1001, {0,0,?,?}, "8", VecListDPairSpaced, IIC_VLD2u,
                        addrmode6align64or128>, Sched<[WriteVLD2]>;
defm VLD2b16wb : VLD2WB<0b1001, {0,1,?,?}, "16", VecListDPairSpaced, IIC_VLD2u,
                        addrmode6align64or128>, Sched<[WriteVLD2]>;
defm VLD2b32wb : VLD2WB<0b1001, {1,0,?,?}, "32", VecListDPairSpaced, IIC_VLD2u,
                        addrmode6align64or128>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD2b8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2b8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 901-909
```tablegen
//   VLD3     : Vector Load (multiple 3-element structures)
class VLD3D<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdSt<0, 0b10, op11_8, op7_4, (outs DPR:$Vd, DPR:$dst2, DPR:$dst3),
          (ins addrmode6:$Rn), IIC_VLD3,
          "vld3", Dt, "\\{$Vd, $dst2, $dst3\\}, $Rn", "", []>, Sched<[WriteVLD3]> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLDST3Instruction";
}
```
- EN: Declares reusable TableGen class `VLD3D` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD3D`，通常用于抽象共享字段、谓词或编码结构。

### Lines 911-913
```tablegen
def  VLD3d8   : VLD3D<0b0100, {0,0,0,?}, "8">;
def  VLD3d16  : VLD3D<0b0100, {0,1,0,?}, "16">;
def  VLD3d32  : VLD3D<0b0100, {1,0,0,?}, "32">;
```
- EN: Defines TableGen record `VLD3d8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3d8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 915-917
```tablegen
def  VLD3d8Pseudo  : VLDQQPseudo<IIC_VLD3>, Sched<[WriteVLD3]>;
def  VLD3d16Pseudo : VLDQQPseudo<IIC_VLD3>, Sched<[WriteVLD3]>;
def  VLD3d32Pseudo : VLDQQPseudo<IIC_VLD3>, Sched<[WriteVLD3]>;
```
- EN: Defines TableGen record `VLD3d8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3d8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 919-928
```tablegen
// ...with address register writeback:
class VLD3DWB<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdSt<0, 0b10, op11_8, op7_4,
          (outs DPR:$Vd, DPR:$dst2, DPR:$dst3, GPR:$wb),
          (ins addrmode6:$Rn, am6offset:$Rm), IIC_VLD3u,
          "vld3", Dt, "\\{$Vd, $dst2, $dst3\\}, $Rn$Rm",
          "$Rn.addr = $wb", []>, Sched<[WriteVLD3]> {
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLDST3Instruction";
}
```
- EN: Declares reusable TableGen class `VLD3DWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD3DWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 930-932
```tablegen
def VLD3d8_UPD  : VLD3DWB<0b0100, {0,0,0,?}, "8">;
def VLD3d16_UPD : VLD3DWB<0b0100, {0,1,0,?}, "16">;
def VLD3d32_UPD : VLD3DWB<0b0100, {1,0,0,?}, "32">;
```
- EN: Defines TableGen record `VLD3d8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3d8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 934-936
```tablegen
def VLD3d8Pseudo_UPD  : VLDQQWBPseudo<IIC_VLD3u>, Sched<[WriteVLD3]>;
def VLD3d16Pseudo_UPD : VLDQQWBPseudo<IIC_VLD3u>, Sched<[WriteVLD3]>;
def VLD3d32Pseudo_UPD : VLDQQWBPseudo<IIC_VLD3u>, Sched<[WriteVLD3]>;
```
- EN: Defines TableGen record `VLD3d8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3d8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 938-944
```tablegen
// ...with double-spaced registers:
def VLD3q8      : VLD3D<0b0101, {0,0,0,?}, "8">;
def VLD3q16     : VLD3D<0b0101, {0,1,0,?}, "16">;
def VLD3q32     : VLD3D<0b0101, {1,0,0,?}, "32">;
def VLD3q8_UPD  : VLD3DWB<0b0101, {0,0,0,?}, "8">;
def VLD3q16_UPD : VLD3DWB<0b0101, {0,1,0,?}, "16">;
def VLD3q32_UPD : VLD3DWB<0b0101, {1,0,0,?}, "32">;
```
- EN: Defines TableGen record `VLD3q8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3q8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 946-948
```tablegen
def VLD3q8Pseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD3u>, Sched<[WriteVLD3]>;
def VLD3q16Pseudo_UPD : VLDQQQQWBPseudo<IIC_VLD3u>, Sched<[WriteVLD3]>;
def VLD3q32Pseudo_UPD : VLDQQQQWBPseudo<IIC_VLD3u>, Sched<[WriteVLD3]>;
```
- EN: Defines TableGen record `VLD3q8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3q8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 950-953
```tablegen
// ...alternate versions to be allocated odd register numbers:
def VLD3q8oddPseudo   : VLDQQQQPseudo<IIC_VLD3>, Sched<[WriteVLD3]>;
def VLD3q16oddPseudo  : VLDQQQQPseudo<IIC_VLD3>, Sched<[WriteVLD3]>;
def VLD3q32oddPseudo  : VLDQQQQPseudo<IIC_VLD3>, Sched<[WriteVLD3]>;
```
- EN: Defines TableGen record `VLD3q8oddPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3q8oddPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 955-957
```tablegen
def VLD3q8oddPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD3u>, Sched<[WriteVLD3]>;
def VLD3q16oddPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD3u>, Sched<[WriteVLD3]>;
def VLD3q32oddPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD3u>, Sched<[WriteVLD3]>;
```
- EN: Defines TableGen record `VLD3q8oddPseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3q8oddPseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 959-969
```tablegen
//   VLD4     : Vector Load (multiple 4-element structures)
class VLD4D<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdSt<0, 0b10, op11_8, op7_4,
          (outs DPR:$Vd, DPR:$dst2, DPR:$dst3, DPR:$dst4),
          (ins addrmode6:$Rn), IIC_VLD4,
          "vld4", Dt, "\\{$Vd, $dst2, $dst3, $dst4\\}, $Rn", "", []>,
    Sched<[WriteVLD4]> {
  let Rm = 0b1111;
  let Inst{5-4} = Rn{5-4};
  let DecoderMethod = "DecodeVLDST4Instruction";
}
```
- EN: Declares reusable TableGen class `VLD4D` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD4D`，通常用于抽象共享字段、谓词或编码结构。

### Lines 971-973
```tablegen
def  VLD4d8   : VLD4D<0b0000, {0,0,?,?}, "8">;
def  VLD4d16  : VLD4D<0b0000, {0,1,?,?}, "16">;
def  VLD4d32  : VLD4D<0b0000, {1,0,?,?}, "32">;
```
- EN: Defines TableGen record `VLD4d8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4d8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 975-977
```tablegen
def  VLD4d8Pseudo  : VLDQQPseudo<IIC_VLD4>, Sched<[WriteVLD4]>;
def  VLD4d16Pseudo : VLDQQPseudo<IIC_VLD4>, Sched<[WriteVLD4]>;
def  VLD4d32Pseudo : VLDQQPseudo<IIC_VLD4>, Sched<[WriteVLD4]>;
```
- EN: Defines TableGen record `VLD4d8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4d8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 979-988
```tablegen
// ...with address register writeback:
class VLD4DWB<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdSt<0, 0b10, op11_8, op7_4,
          (outs DPR:$Vd, DPR:$dst2, DPR:$dst3, DPR:$dst4, GPR:$wb),
          (ins addrmode6:$Rn, am6offset:$Rm), IIC_VLD4u,
          "vld4", Dt, "\\{$Vd, $dst2, $dst3, $dst4\\}, $Rn$Rm",
          "$Rn.addr = $wb", []>, Sched<[WriteVLD4]> {
  let Inst{5-4} = Rn{5-4};
  let DecoderMethod = "DecodeVLDST4Instruction";
}
```
- EN: Declares reusable TableGen class `VLD4DWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD4DWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 990-992
```tablegen
def VLD4d8_UPD  : VLD4DWB<0b0000, {0,0,?,?}, "8">;
def VLD4d16_UPD : VLD4DWB<0b0000, {0,1,?,?}, "16">;
def VLD4d32_UPD : VLD4DWB<0b0000, {1,0,?,?}, "32">;
```
- EN: Defines TableGen record `VLD4d8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4d8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 994-996
```tablegen
def VLD4d8Pseudo_UPD  : VLDQQWBPseudo<IIC_VLD4u>, Sched<[WriteVLD4]>;
def VLD4d16Pseudo_UPD : VLDQQWBPseudo<IIC_VLD4u>, Sched<[WriteVLD4]>;
def VLD4d32Pseudo_UPD : VLDQQWBPseudo<IIC_VLD4u>, Sched<[WriteVLD4]>;
```
- EN: Defines TableGen record `VLD4d8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4d8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 998-1004
```tablegen
// ...with double-spaced registers:
def VLD4q8      : VLD4D<0b0001, {0,0,?,?}, "8">;
def VLD4q16     : VLD4D<0b0001, {0,1,?,?}, "16">;
def VLD4q32     : VLD4D<0b0001, {1,0,?,?}, "32">;
def VLD4q8_UPD  : VLD4DWB<0b0001, {0,0,?,?}, "8">;
def VLD4q16_UPD : VLD4DWB<0b0001, {0,1,?,?}, "16">;
def VLD4q32_UPD : VLD4DWB<0b0001, {1,0,?,?}, "32">;
```
- EN: Defines TableGen record `VLD4q8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4q8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1006-1008
```tablegen
def VLD4q8Pseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD4u>, Sched<[WriteVLD4]>;
def VLD4q16Pseudo_UPD : VLDQQQQWBPseudo<IIC_VLD4u>, Sched<[WriteVLD4]>;
def VLD4q32Pseudo_UPD : VLDQQQQWBPseudo<IIC_VLD4u>, Sched<[WriteVLD4]>;
```
- EN: Defines TableGen record `VLD4q8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4q8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1010-1013
```tablegen
// ...alternate versions to be allocated odd register numbers:
def VLD4q8oddPseudo   : VLDQQQQPseudo<IIC_VLD4>, Sched<[WriteVLD4]>;
def VLD4q16oddPseudo  : VLDQQQQPseudo<IIC_VLD4>, Sched<[WriteVLD4]>;
def VLD4q32oddPseudo  : VLDQQQQPseudo<IIC_VLD4>, Sched<[WriteVLD4]>;
```
- EN: Defines TableGen record `VLD4q8oddPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4q8oddPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1015-1017
```tablegen
def VLD4q8oddPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD4u>, Sched<[WriteVLD4]>;
def VLD4q16oddPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD4u>, Sched<[WriteVLD4]>;
def VLD4q32oddPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD4u>, Sched<[WriteVLD4]>;
```
- EN: Defines TableGen record `VLD4q8oddPseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4q8oddPseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1019-1019
```tablegen
} // mayLoad = 1, hasSideEffects = 0, hasExtraDefRegAllocReq = 1
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1021-1038
```tablegen
// Classes for VLD*LN pseudo-instructions with multi-register operands.
// These are expanded to real instructions after register allocation.
class VLDQLNPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QPR:$dst),
                (ins addrmode6:$addr, QPR:$src, nohash_imm:$lane),
                itin, "$src = $dst">;
class VLDQLNWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QPR:$dst, GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset, QPR:$src,
                 nohash_imm:$lane), itin, "$addr.addr = $wb, $src = $dst">;
class VLDQQLNPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QQPR:$dst),
                (ins addrmode6:$addr, QQPR:$src, nohash_imm:$lane),
                itin, "$src = $dst">;
class VLDQQLNWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QQPR:$dst, GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset, QQPR:$src,
                 nohash_imm:$lane), itin, "$addr.addr = $wb, $src = $dst">;
```
- EN: Declares reusable TableGen class `VLDQLNPseudo` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLDQLNPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1039-1046
```tablegen
class VLDQQQQLNPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QQQQPR:$dst),
                (ins addrmode6:$addr, QQQQPR:$src, nohash_imm:$lane),
                itin, "$src = $dst">;
class VLDQQQQLNWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs QQQQPR:$dst, GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset, QQQQPR:$src,
                 nohash_imm:$lane), itin, "$addr.addr = $wb, $src = $dst">;
```
- EN: Declares reusable TableGen class `VLDQQQQLNPseudo` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLDQQQQLNPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1048-1065
```tablegen
//   VLD1LN   : Vector Load (single element to one lane)
class VLD1LN<bits<4> op11_8, bits<4> op7_4, string Dt, ValueType Ty,
             PatFrag LoadOp>
  : NLdStLn<1, 0b10, op11_8, op7_4, (outs DPR:$Vd),
          (ins addrmode6:$Rn, DPR:$src, nohash_imm:$lane),
          IIC_VLD1ln, "vld1", Dt, "\\{$Vd[$lane]\\}, $Rn",
          "$src = $Vd",
          [(set DPR:$Vd, (vector_insert (Ty DPR:$src),
                                         (i32 (LoadOp addrmode6:$Rn)),
                                         imm:$lane))]> {
  let Rm = 0b1111;
  let DecoderMethod = "DecodeVLD1LN";
}
class VLD1LN32<bits<4> op11_8, bits<4> op7_4, string Dt, ValueType Ty,
             PatFrag LoadOp>
  : NLdStLn<1, 0b10, op11_8, op7_4, (outs DPR:$Vd),
          (ins addrmode6oneL32:$Rn, DPR:$src, nohash_imm:$lane),
          IIC_VLD1ln, "vld1", Dt, "\\{$Vd[$lane]\\}, $Rn",
```
- EN: Declares reusable TableGen class `VLD1LN` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD1LN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1066-1078
```tablegen
          "$src = $Vd",
          [(set DPR:$Vd, (vector_insert (Ty DPR:$src),
                                         (i32 (LoadOp addrmode6oneL32:$Rn)),
                                         imm:$lane))]>, Sched<[WriteVLD1]> {
  let Rm = 0b1111;
  let DecoderMethod = "DecodeVLD1LN";
}
class VLD1QLNPseudo<ValueType Ty, PatFrag LoadOp> : VLDQLNPseudo<IIC_VLD1ln>,
                                                    Sched<[WriteVLD1]> {
  let Pattern = [(set QPR:$dst, (vector_insert (Ty QPR:$src),
                                               (i32 (LoadOp addrmode6:$addr)),
                                               imm:$lane))];
}
```
- EN: Declares reusable TableGen class `VLD1QLNPseudo` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD1QLNPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1080-1090
```tablegen
def VLD1LNd8  : VLD1LN<0b0000, {?,?,?,0}, "8", v8i8, extloadi8> {
  let Inst{7-5} = lane{2-0};
}
def VLD1LNd16 : VLD1LN<0b0100, {?,?,0,?}, "16", v4i16, extloadi16> {
  let Inst{7-6} = lane{1-0};
  let Inst{5-4} = Rn{5-4};
}
def VLD1LNd32 : VLD1LN32<0b1000, {?,0,?,?}, "32", v2i32, load> {
  let Inst{7} = lane{0};
  let Inst{5-4} = Rn{5-4};
}
```
- EN: Defines TableGen record `VLD1LNd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1LNd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1092-1094
```tablegen
def VLD1LNq8Pseudo  : VLD1QLNPseudo<v16i8, extloadi8>;
def VLD1LNq16Pseudo : VLD1QLNPseudo<v8i16, extloadi16>;
def VLD1LNq32Pseudo : VLD1QLNPseudo<v4i32, load>;
```
- EN: Defines TableGen record `VLD1LNq8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1LNq8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1096-1113
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(vector_insert (v4f16 DPR:$src),
                         (f16 (load addrmode6:$addr)), imm:$lane),
          (VLD1LNd16 addrmode6:$addr, DPR:$src, imm:$lane)>;
def : Pat<(vector_insert (v8f16 QPR:$src),
                         (f16 (load addrmode6:$addr)), imm:$lane),
          (VLD1LNq16Pseudo addrmode6:$addr, QPR:$src, imm:$lane)>;
def : Pat<(vector_insert (v4bf16 DPR:$src),
                         (bf16 (load addrmode6:$addr)), imm:$lane),
          (VLD1LNd16 addrmode6:$addr, DPR:$src, imm:$lane)>;
def : Pat<(vector_insert (v8bf16 QPR:$src),
                         (bf16 (load addrmode6:$addr)), imm:$lane),
          (VLD1LNq16Pseudo addrmode6:$addr, QPR:$src, imm:$lane)>;
def : Pat<(vector_insert (v2f32 DPR:$src),
                         (f32 (load addrmode6:$addr)), imm:$lane),
          (VLD1LNd32 addrmode6:$addr, DPR:$src, imm:$lane)>;
def : Pat<(vector_insert (v4f32 QPR:$src),
                         (f32 (load addrmode6:$addr)), imm:$lane),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1114-1114
```tablegen
          (VLD1LNq32Pseudo addrmode6:$addr, QPR:$src, imm:$lane)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1116-1130
```tablegen
// A 64-bit subvector insert to the first 128-bit vector position
// is a subregister copy that needs no instruction.
def : Pat<(insert_subvector undef, (v1i64 DPR:$src), (i32 0)),
          (INSERT_SUBREG (v2i64 (IMPLICIT_DEF)), DPR:$src, dsub_0)>;
def : Pat<(insert_subvector undef, (v2i32 DPR:$src), (i32 0)),
          (INSERT_SUBREG (v4i32 (IMPLICIT_DEF)), DPR:$src, dsub_0)>;
def : Pat<(insert_subvector undef, (v2f32 DPR:$src), (i32 0)),
          (INSERT_SUBREG (v4f32 (IMPLICIT_DEF)), DPR:$src, dsub_0)>;
def : Pat<(insert_subvector undef, (v4i16 DPR:$src), (i32 0)),
          (INSERT_SUBREG (v8i16 (IMPLICIT_DEF)), DPR:$src, dsub_0)>;
def : Pat<(insert_subvector undef, (v4f16 DPR:$src), (i32 0)),
          (INSERT_SUBREG (v8f16 (IMPLICIT_DEF)), DPR:$src, dsub_0)>;
def : Pat<(insert_subvector (v16i8 undef), (v8i8 DPR:$src), (i32 0)),
          (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)), DPR:$src, dsub_0)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1133-1133
```tablegen
let mayLoad = 1, hasSideEffects = 0, hasExtraDefRegAllocReq = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1135-1143
```tablegen
// ...with address register writeback:
class VLD1LNWB<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b10, op11_8, op7_4, (outs DPR:$Vd, GPR:$wb),
          (ins addrmode6:$Rn, am6offset:$Rm,
           DPR:$src, nohash_imm:$lane), IIC_VLD1lnu, "vld1", Dt,
          "\\{$Vd[$lane]\\}, $Rn$Rm",
          "$src = $Vd, $Rn.addr = $wb", []>, Sched<[WriteVLD1]> {
  let DecoderMethod = "DecodeVLD1LN";
}
```
- EN: Declares reusable TableGen class `VLD1LNWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD1LNWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1145-1156
```tablegen
def VLD1LNd8_UPD  : VLD1LNWB<0b0000, {?,?,?,0}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VLD1LNd16_UPD : VLD1LNWB<0b0100, {?,?,0,?}, "16"> {
  let Inst{7-6} = lane{1-0};
  let Inst{4}   = Rn{4};
}
def VLD1LNd32_UPD : VLD1LNWB<0b1000, {?,0,?,?}, "32"> {
  let Inst{7} = lane{0};
  let Inst{5} = Rn{4};
  let Inst{4} = Rn{4};
}
```
- EN: Defines TableGen record `VLD1LNd8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1LNd8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1158-1160
```tablegen
def VLD1LNq8Pseudo_UPD  : VLDQLNWBPseudo<IIC_VLD1lnu>, Sched<[WriteVLD1]>;
def VLD1LNq16Pseudo_UPD : VLDQLNWBPseudo<IIC_VLD1lnu>, Sched<[WriteVLD1]>;
def VLD1LNq32Pseudo_UPD : VLDQLNWBPseudo<IIC_VLD1lnu>, Sched<[WriteVLD1]>;
```
- EN: Defines TableGen record `VLD1LNq8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1LNq8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1162-1171
```tablegen
//   VLD2LN   : Vector Load (single 2-element structure to one lane)
class VLD2LN<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b10, op11_8, op7_4, (outs DPR:$Vd, DPR:$dst2),
          (ins addrmode6:$Rn, DPR:$src1, DPR:$src2, nohash_imm:$lane),
          IIC_VLD2ln, "vld2", Dt, "\\{$Vd[$lane], $dst2[$lane]\\}, $Rn",
          "$src1 = $Vd, $src2 = $dst2", []>, Sched<[WriteVLD1]> {
  let Rm = 0b1111;
  let Inst{4}   = Rn{4};
  let DecoderMethod = "DecodeVLD2LN";
}
```
- EN: Declares reusable TableGen class `VLD2LN` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD2LN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1173-1181
```tablegen
def VLD2LNd8  : VLD2LN<0b0001, {?,?,?,?}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VLD2LNd16 : VLD2LN<0b0101, {?,?,0,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD2LNd32 : VLD2LN<0b1001, {?,0,0,?}, "32"> {
  let Inst{7} = lane{0};
}
```
- EN: Defines TableGen record `VLD2LNd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1183-1185
```tablegen
def VLD2LNd8Pseudo  : VLDQLNPseudo<IIC_VLD2ln>, Sched<[WriteVLD1]>;
def VLD2LNd16Pseudo : VLDQLNPseudo<IIC_VLD2ln>, Sched<[WriteVLD1]>;
def VLD2LNd32Pseudo : VLDQLNPseudo<IIC_VLD2ln>, Sched<[WriteVLD1]>;
```
- EN: Defines TableGen record `VLD2LNd8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNd8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1187-1193
```tablegen
// ...with double-spaced registers:
def VLD2LNq16 : VLD2LN<0b0101, {?,?,1,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD2LNq32 : VLD2LN<0b1001, {?,1,0,?}, "32"> {
  let Inst{7} = lane{0};
}
```
- EN: Defines TableGen record `VLD2LNq16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNq16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1195-1196
```tablegen
def VLD2LNq16Pseudo : VLDQQLNPseudo<IIC_VLD2ln>, Sched<[WriteVLD1]>;
def VLD2LNq32Pseudo : VLDQQLNPseudo<IIC_VLD2ln>, Sched<[WriteVLD1]>;
```
- EN: Defines TableGen record `VLD2LNq16Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNq16Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1198-1207
```tablegen
// ...with address register writeback:
class VLD2LNWB<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b10, op11_8, op7_4, (outs DPR:$Vd, DPR:$dst2, GPR:$wb),
          (ins addrmode6:$Rn, am6offset:$Rm,
           DPR:$src1, DPR:$src2, nohash_imm:$lane), IIC_VLD2lnu, "vld2", Dt,
          "\\{$Vd[$lane], $dst2[$lane]\\}, $Rn$Rm",
          "$src1 = $Vd, $src2 = $dst2, $Rn.addr = $wb", []> {
  let Inst{4}   = Rn{4};
  let DecoderMethod = "DecodeVLD2LN";
}
```
- EN: Declares reusable TableGen class `VLD2LNWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD2LNWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1209-1217
```tablegen
def VLD2LNd8_UPD  : VLD2LNWB<0b0001, {?,?,?,?}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VLD2LNd16_UPD : VLD2LNWB<0b0101, {?,?,0,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD2LNd32_UPD : VLD2LNWB<0b1001, {?,0,0,?}, "32"> {
  let Inst{7} = lane{0};
}
```
- EN: Defines TableGen record `VLD2LNd8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNd8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1219-1221
```tablegen
def VLD2LNd8Pseudo_UPD  : VLDQLNWBPseudo<IIC_VLD2lnu>, Sched<[WriteVLD1]>;
def VLD2LNd16Pseudo_UPD : VLDQLNWBPseudo<IIC_VLD2lnu>, Sched<[WriteVLD1]>;
def VLD2LNd32Pseudo_UPD : VLDQLNWBPseudo<IIC_VLD2lnu>, Sched<[WriteVLD1]>;
```
- EN: Defines TableGen record `VLD2LNd8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNd8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1223-1228
```tablegen
def VLD2LNq16_UPD : VLD2LNWB<0b0101, {?,?,1,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD2LNq32_UPD : VLD2LNWB<0b1001, {?,1,0,?}, "32"> {
  let Inst{7} = lane{0};
}
```
- EN: Defines TableGen record `VLD2LNq16_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNq16_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1230-1231
```tablegen
def VLD2LNq16Pseudo_UPD : VLDQQLNWBPseudo<IIC_VLD2lnu>, Sched<[WriteVLD1]>;
def VLD2LNq32Pseudo_UPD : VLDQQLNWBPseudo<IIC_VLD2lnu>, Sched<[WriteVLD1]>;
```
- EN: Defines TableGen record `VLD2LNq16Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNq16Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1233-1242
```tablegen
//   VLD3LN   : Vector Load (single 3-element structure to one lane)
class VLD3LN<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b10, op11_8, op7_4, (outs DPR:$Vd, DPR:$dst2, DPR:$dst3),
          (ins addrmode6:$Rn, DPR:$src1, DPR:$src2, DPR:$src3,
          nohash_imm:$lane), IIC_VLD3ln, "vld3", Dt,
          "\\{$Vd[$lane], $dst2[$lane], $dst3[$lane]\\}, $Rn",
          "$src1 = $Vd, $src2 = $dst2, $src3 = $dst3", []>, Sched<[WriteVLD2]> {
  let Rm = 0b1111;
  let DecoderMethod = "DecodeVLD3LN";
}
```
- EN: Declares reusable TableGen class `VLD3LN` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD3LN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1244-1252
```tablegen
def VLD3LNd8  : VLD3LN<0b0010, {?,?,?,0}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VLD3LNd16 : VLD3LN<0b0110, {?,?,0,0}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD3LNd32 : VLD3LN<0b1010, {?,0,0,0}, "32"> {
  let Inst{7}   = lane{0};
}
```
- EN: Defines TableGen record `VLD3LNd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1254-1256
```tablegen
def VLD3LNd8Pseudo  : VLDQQLNPseudo<IIC_VLD3ln>, Sched<[WriteVLD2]>;
def VLD3LNd16Pseudo : VLDQQLNPseudo<IIC_VLD3ln>, Sched<[WriteVLD2]>;
def VLD3LNd32Pseudo : VLDQQLNPseudo<IIC_VLD3ln>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD3LNd8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNd8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1258-1264
```tablegen
// ...with double-spaced registers:
def VLD3LNq16 : VLD3LN<0b0110, {?,?,1,0}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD3LNq32 : VLD3LN<0b1010, {?,1,0,0}, "32"> {
  let Inst{7}   = lane{0};
}
```
- EN: Defines TableGen record `VLD3LNq16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNq16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1266-1267
```tablegen
def VLD3LNq16Pseudo : VLDQQQQLNPseudo<IIC_VLD3ln>, Sched<[WriteVLD2]>;
def VLD3LNq32Pseudo : VLDQQQQLNPseudo<IIC_VLD3ln>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD3LNq16Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNq16Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1269-1280
```tablegen
// ...with address register writeback:
class VLD3LNWB<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b10, op11_8, op7_4,
          (outs DPR:$Vd, DPR:$dst2, DPR:$dst3, GPR:$wb),
          (ins addrmode6:$Rn, am6offset:$Rm,
           DPR:$src1, DPR:$src2, DPR:$src3, nohash_imm:$lane),
          IIC_VLD3lnu, "vld3", Dt,
          "\\{$Vd[$lane], $dst2[$lane], $dst3[$lane]\\}, $Rn$Rm",
          "$src1 = $Vd, $src2 = $dst2, $src3 = $dst3, $Rn.addr = $wb",
          []>, Sched<[WriteVLD2]> {
  let DecoderMethod = "DecodeVLD3LN";
}
```
- EN: Declares reusable TableGen class `VLD3LNWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD3LNWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1282-1290
```tablegen
def VLD3LNd8_UPD  : VLD3LNWB<0b0010, {?,?,?,0}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VLD3LNd16_UPD : VLD3LNWB<0b0110, {?,?,0,0}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD3LNd32_UPD : VLD3LNWB<0b1010, {?,0,0,0}, "32"> {
  let Inst{7} = lane{0};
}
```
- EN: Defines TableGen record `VLD3LNd8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNd8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1292-1294
```tablegen
def VLD3LNd8Pseudo_UPD  : VLDQQLNWBPseudo<IIC_VLD3lnu>, Sched<[WriteVLD2]>;
def VLD3LNd16Pseudo_UPD : VLDQQLNWBPseudo<IIC_VLD3lnu>, Sched<[WriteVLD2]>;
def VLD3LNd32Pseudo_UPD : VLDQQLNWBPseudo<IIC_VLD3lnu>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD3LNd8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNd8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1296-1301
```tablegen
def VLD3LNq16_UPD : VLD3LNWB<0b0110, {?,?,1,0}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD3LNq32_UPD : VLD3LNWB<0b1010, {?,1,0,0}, "32"> {
  let Inst{7} = lane{0};
}
```
- EN: Defines TableGen record `VLD3LNq16_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNq16_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1303-1304
```tablegen
def VLD3LNq16Pseudo_UPD : VLDQQQQLNWBPseudo<IIC_VLD3lnu>, Sched<[WriteVLD2]>;
def VLD3LNq32Pseudo_UPD : VLDQQQQLNWBPseudo<IIC_VLD3lnu>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD3LNq16Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNq16Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1306-1318
```tablegen
//   VLD4LN   : Vector Load (single 4-element structure to one lane)
class VLD4LN<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b10, op11_8, op7_4,
          (outs DPR:$Vd, DPR:$dst2, DPR:$dst3, DPR:$dst4),
          (ins addrmode6:$Rn, DPR:$src1, DPR:$src2, DPR:$src3, DPR:$src4,
          nohash_imm:$lane), IIC_VLD4ln, "vld4", Dt,
          "\\{$Vd[$lane], $dst2[$lane], $dst3[$lane], $dst4[$lane]\\}, $Rn",
          "$src1 = $Vd, $src2 = $dst2, $src3 = $dst3, $src4 = $dst4", []>,
    Sched<[WriteVLD2]> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLD4LN";
}
```
- EN: Declares reusable TableGen class `VLD4LN` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD4LN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1320-1329
```tablegen
def VLD4LNd8  : VLD4LN<0b0011, {?,?,?,?}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VLD4LNd16 : VLD4LN<0b0111, {?,?,0,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD4LNd32 : VLD4LN<0b1011, {?,0,?,?}, "32"> {
  let Inst{7} = lane{0};
  let Inst{5} = Rn{5};
}
```
- EN: Defines TableGen record `VLD4LNd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1331-1333
```tablegen
def VLD4LNd8Pseudo  : VLDQQLNPseudo<IIC_VLD4ln>, Sched<[WriteVLD2]>;
def VLD4LNd16Pseudo : VLDQQLNPseudo<IIC_VLD4ln>, Sched<[WriteVLD2]>;
def VLD4LNd32Pseudo : VLDQQLNPseudo<IIC_VLD4ln>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD4LNd8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNd8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1335-1342
```tablegen
// ...with double-spaced registers:
def VLD4LNq16 : VLD4LN<0b0111, {?,?,1,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD4LNq32 : VLD4LN<0b1011, {?,1,?,?}, "32"> {
  let Inst{7} = lane{0};
  let Inst{5} = Rn{5};
}
```
- EN: Defines TableGen record `VLD4LNq16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNq16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1344-1345
```tablegen
def VLD4LNq16Pseudo : VLDQQQQLNPseudo<IIC_VLD4ln>, Sched<[WriteVLD2]>;
def VLD4LNq32Pseudo : VLDQQQQLNPseudo<IIC_VLD4ln>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD4LNq16Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNq16Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1347-1359
```tablegen
// ...with address register writeback:
class VLD4LNWB<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b10, op11_8, op7_4,
          (outs DPR:$Vd, DPR:$dst2, DPR:$dst3, DPR:$dst4, GPR:$wb),
          (ins addrmode6:$Rn, am6offset:$Rm,
           DPR:$src1, DPR:$src2, DPR:$src3, DPR:$src4, nohash_imm:$lane),
          IIC_VLD4lnu, "vld4", Dt,
"\\{$Vd[$lane], $dst2[$lane], $dst3[$lane], $dst4[$lane]\\}, $Rn$Rm",
"$src1 = $Vd, $src2 = $dst2, $src3 = $dst3, $src4 = $dst4, $Rn.addr = $wb",
          []> {
  let Inst{4}   = Rn{4};
  let DecoderMethod = "DecodeVLD4LN"  ;
}
```
- EN: Declares reusable TableGen class `VLD4LNWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD4LNWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1361-1370
```tablegen
def VLD4LNd8_UPD  : VLD4LNWB<0b0011, {?,?,?,?}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VLD4LNd16_UPD : VLD4LNWB<0b0111, {?,?,0,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD4LNd32_UPD : VLD4LNWB<0b1011, {?,0,?,?}, "32"> {
  let Inst{7} = lane{0};
  let Inst{5} = Rn{5};
}
```
- EN: Defines TableGen record `VLD4LNd8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNd8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1372-1374
```tablegen
def VLD4LNd8Pseudo_UPD  : VLDQQLNWBPseudo<IIC_VLD4lnu>, Sched<[WriteVLD2]>;
def VLD4LNd16Pseudo_UPD : VLDQQLNWBPseudo<IIC_VLD4lnu>, Sched<[WriteVLD2]>;
def VLD4LNd32Pseudo_UPD : VLDQQLNWBPseudo<IIC_VLD4lnu>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD4LNd8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNd8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1376-1382
```tablegen
def VLD4LNq16_UPD : VLD4LNWB<0b0111, {?,?,1,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VLD4LNq32_UPD : VLD4LNWB<0b1011, {?,1,?,?}, "32"> {
  let Inst{7} = lane{0};
  let Inst{5} = Rn{5};
}
```
- EN: Defines TableGen record `VLD4LNq16_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNq16_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1384-1385
```tablegen
def VLD4LNq16Pseudo_UPD : VLDQQQQLNWBPseudo<IIC_VLD4lnu>, Sched<[WriteVLD2]>;
def VLD4LNq32Pseudo_UPD : VLDQQQQLNWBPseudo<IIC_VLD4lnu>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD4LNq16Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNq16Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1387-1387
```tablegen
} // mayLoad = 1, hasSideEffects = 0, hasExtraDefRegAllocReq = 1
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1389-1406
```tablegen
//   VLD1DUP  : Vector Load (single element to all lanes)
class VLD1DUP<bits<4> op7_4, string Dt, ValueType Ty, PatFrag LoadOp,
              Operand AddrMode>
  : NLdSt<1, 0b10, 0b1100, op7_4, (outs VecListOneDAllLanes:$Vd),
          (ins AddrMode:$Rn),
          IIC_VLD1dup, "vld1", Dt, "$Vd, $Rn", "",
          [(set VecListOneDAllLanes:$Vd,
                (Ty (ARMvdup (i32 (LoadOp AddrMode:$Rn)))))]>,
   Sched<[WriteVLD2]> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLD1DupInstruction";
}
def VLD1DUPd8  : VLD1DUP<{0,0,0,?}, "8", v8i8, extloadi8,
                         addrmode6dupalignNone>;
def VLD1DUPd16 : VLD1DUP<{0,1,0,?}, "16", v4i16, extloadi16,
                         addrmode6dupalign16>;
def VLD1DUPd32 : VLD1DUP<{1,0,0,?}, "32", v2i32, load,
```
- EN: Declares reusable TableGen class `VLD1DUP` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD1DUP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1407-1407
```tablegen
                         addrmode6dupalign32>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1409-1412
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v2f32 (ARMvdup (f32 (load addrmode6dup:$addr)))),
          (VLD1DUPd32 addrmode6:$addr)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1414-1424
```tablegen
class VLD1QDUP<bits<4> op7_4, string Dt, ValueType Ty, PatFrag LoadOp,
               Operand AddrMode>
  : NLdSt<1, 0b10, 0b1100, op7_4, (outs VecListDPairAllLanes:$Vd),
          (ins AddrMode:$Rn), IIC_VLD1dup,
          "vld1", Dt, "$Vd, $Rn", "",
          [(set VecListDPairAllLanes:$Vd,
                (Ty (ARMvdup (i32 (LoadOp AddrMode:$Rn)))))]> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLD1DupInstruction";
}
```
- EN: Declares reusable TableGen class `VLD1QDUP` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD1QDUP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1426-1431
```tablegen
def VLD1DUPq8  : VLD1QDUP<{0,0,1,0}, "8", v16i8, extloadi8,
                          addrmode6dupalignNone>;
def VLD1DUPq16 : VLD1QDUP<{0,1,1,?}, "16", v8i16, extloadi16,
                          addrmode6dupalign16>;
def VLD1DUPq32 : VLD1QDUP<{1,0,1,?}, "32", v4i32, load,
                          addrmode6dupalign32>;
```
- EN: Defines TableGen record `VLD1DUPq8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1DUPq8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1433-1436
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v4f32 (ARMvdup (f32 (load addrmode6dup:$addr)))),
          (VLD1DUPq32 addrmode6:$addr)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1438-1455
```tablegen
let mayLoad = 1, hasSideEffects = 0, hasExtraDefRegAllocReq = 1 in {
// ...with address register writeback:
multiclass VLD1DUPWB<bits<4> op7_4, string Dt, Operand AddrMode> {
  def _fixed : NLdSt<1, 0b10, 0b1100, op7_4,
                     (outs VecListOneDAllLanes:$Vd, GPR:$wb),
                     (ins AddrMode:$Rn), IIC_VLD1dupu,
                     "vld1", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{4} = Rn{4};
    let DecoderMethod = "DecodeVLD1DupInstruction";
  }
  def _register : NLdSt<1, 0b10, 0b1100, op7_4,
                        (outs VecListOneDAllLanes:$Vd, GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm), IIC_VLD1dupu,
                        "vld1", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []> {
    let Inst{4} = Rn{4};
```
- EN: Declares TableGen `multiclass VLD1DUPWB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VLD1DUPWB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1456-1473
```tablegen
    let DecoderMethod = "DecodeVLD1DupInstruction";
  }
}
multiclass VLD1QDUPWB<bits<4> op7_4, string Dt, Operand AddrMode> {
  def _fixed : NLdSt<1, 0b10, 0b1100, op7_4,
                     (outs VecListDPairAllLanes:$Vd, GPR:$wb),
                     (ins AddrMode:$Rn), IIC_VLD1dupu,
                     "vld1", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVLD1]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{4} = Rn{4};
    let DecoderMethod = "DecodeVLD1DupInstruction";
  }
  def _register : NLdSt<1, 0b10, 0b1100, op7_4,
                        (outs VecListDPairAllLanes:$Vd, GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm), IIC_VLD1dupu,
                        "vld1", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []> {
```
- EN: Declares TableGen `multiclass VLD1QDUPWB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VLD1QDUPWB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1474-1477
```tablegen
    let Inst{4} = Rn{4};
    let DecoderMethod = "DecodeVLD1DupInstruction";
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1479-1481
```tablegen
defm VLD1DUPd8wb  : VLD1DUPWB<{0,0,0,0}, "8", addrmode6dupalignNone>;
defm VLD1DUPd16wb : VLD1DUPWB<{0,1,0,?}, "16", addrmode6dupalign16>;
defm VLD1DUPd32wb : VLD1DUPWB<{1,0,0,?}, "32", addrmode6dupalign32>;
```
- EN: Defines TableGen record `VLD1DUPd8wb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1DUPd8wb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1483-1485
```tablegen
defm VLD1DUPq8wb  : VLD1QDUPWB<{0,0,1,0}, "8", addrmode6dupalignNone>;
defm VLD1DUPq16wb : VLD1QDUPWB<{0,1,1,?}, "16", addrmode6dupalign16>;
defm VLD1DUPq32wb : VLD1QDUPWB<{1,0,1,?}, "32", addrmode6dupalign32>;
```
- EN: Defines TableGen record `VLD1DUPq8wb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1DUPq8wb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1487-1495
```tablegen
//   VLD2DUP  : Vector Load (single 2-element structure to all lanes)
class VLD2DUP<bits<4> op7_4, string Dt, RegisterOperand VdTy, Operand AddrMode>
  : NLdSt<1, 0b10, 0b1101, op7_4, (outs VdTy:$Vd),
          (ins AddrMode:$Rn), IIC_VLD2dup,
          "vld2", Dt, "$Vd, $Rn", "", []> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLD2DupInstruction";
}
```
- EN: Declares reusable TableGen class `VLD2DUP` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD2DUP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1497-1502
```tablegen
def VLD2DUPd8  : VLD2DUP<{0,0,0,?}, "8",  VecListDPairAllLanes,
                         addrmode6dupalign16>;
def VLD2DUPd16 : VLD2DUP<{0,1,0,?}, "16", VecListDPairAllLanes,
                         addrmode6dupalign32>;
def VLD2DUPd32 : VLD2DUP<{1,0,0,?}, "32", VecListDPairAllLanes,
                         addrmode6dupalign64>;
```
- EN: Defines TableGen record `VLD2DUPd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2DUPd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1504-1512
```tablegen
// HACK this one, VLD2DUPd8x2 must be changed at the same time with VLD2b8 or
// "vld2.8 {d0[], d2[]}, [r4:32]" will become "vld2.8 {d0, d2}, [r4:32]".
// ...with double-spaced registers
def VLD2DUPd8x2  : VLD2DUP<{0,0,1,?}, "8",  VecListDPairSpacedAllLanes,
                           addrmode6dupalign16>;
def VLD2DUPd16x2 : VLD2DUP<{0,1,1,?}, "16", VecListDPairSpacedAllLanes,
                           addrmode6dupalign32>;
def VLD2DUPd32x2 : VLD2DUP<{1,0,1,?}, "32", VecListDPairSpacedAllLanes,
                           addrmode6dupalign64>;
```
- EN: Defines TableGen record `VLD2DUPd8x2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2DUPd8x2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1514-1526
```tablegen
// Duplicate of VLDQQPseudo but with a constraint variable
// to ensure the odd and even lanes use the same register range 
class VLDQQPseudoInputDST<InstrItinClass itin>
  : PseudoNLdSt<(outs QQPR:$dst), (ins addrmode6:$addr, QQPR: $src), itin, 
                "$src = $dst">;
class VLDQQWBPseudoInputDST<InstrItinClass itin>
  : PseudoNLdSt<(outs QQPR:$dst, GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset, QQPR: $src), itin,
                "$addr.addr = $wb, $src = $dst">;
class VLDQQWBfixedPseudoInputDST<InstrItinClass itin>
  : PseudoNLdSt<(outs QQPR:$dst, GPR:$wb),
                (ins addrmode6:$addr, QQPR: $src), itin,
                "$addr.addr = $wb, $src = $dst">;
```
- EN: Declares reusable TableGen class `VLDQQPseudoInputDST` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLDQQPseudoInputDST`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1528-1533
```tablegen
def VLD2DUPq8EvenPseudo  : VLDQQPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
def VLD2DUPq8OddPseudo   : VLDQQPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
def VLD2DUPq16EvenPseudo : VLDQQPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
def VLD2DUPq16OddPseudo  : VLDQQPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
def VLD2DUPq32EvenPseudo : VLDQQPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
def VLD2DUPq32OddPseudo  : VLDQQPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD2DUPq8EvenPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2DUPq8EvenPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1535-1552
```tablegen
// ...with address register writeback:
multiclass VLD2DUPWB<bits<4> op7_4, string Dt, RegisterOperand VdTy,
                     Operand AddrMode> {
  def _fixed : NLdSt<1, 0b10, 0b1101, op7_4,
                     (outs VdTy:$Vd, GPR:$wb),
                     (ins AddrMode:$Rn), IIC_VLD2dupu,
                     "vld2", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVLD1]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{4} = Rn{4};
    let DecoderMethod = "DecodeVLD2DupInstruction";
  }
  def _register : NLdSt<1, 0b10, 0b1101, op7_4,
                        (outs VdTy:$Vd, GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm), IIC_VLD2dupu,
                        "vld2", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []>, Sched<[WriteVLD1]> {
    let Inst{4} = Rn{4};
```
- EN: Declares TableGen `multiclass VLD2DUPWB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VLD2DUPWB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1553-1555
```tablegen
    let DecoderMethod = "DecodeVLD2DupInstruction";
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1557-1562
```tablegen
defm VLD2DUPd8wb    : VLD2DUPWB<{0,0,0,0}, "8",  VecListDPairAllLanes,
                                addrmode6dupalign16>;
defm VLD2DUPd16wb   : VLD2DUPWB<{0,1,0,?}, "16", VecListDPairAllLanes,
                                addrmode6dupalign32>;
defm VLD2DUPd32wb   : VLD2DUPWB<{1,0,0,?}, "32", VecListDPairAllLanes,
                                addrmode6dupalign64>;
```
- EN: Defines TableGen record `VLD2DUPd8wb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2DUPd8wb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1564-1569
```tablegen
defm VLD2DUPd8x2wb  : VLD2DUPWB<{0,0,1,0}, "8",  VecListDPairSpacedAllLanes,
                                addrmode6dupalign16>;
defm VLD2DUPd16x2wb : VLD2DUPWB<{0,1,1,?}, "16", VecListDPairSpacedAllLanes,
                                addrmode6dupalign32>;
defm VLD2DUPd32x2wb : VLD2DUPWB<{1,0,1,?}, "32", VecListDPairSpacedAllLanes,
                                addrmode6dupalign64>;
```
- EN: Defines TableGen record `VLD2DUPd8x2wb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2DUPd8x2wb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1571-1576
```tablegen
def VLD2DUPq8OddPseudoWB_fixed     : VLDQQWBfixedPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
def VLD2DUPq16OddPseudoWB_fixed    : VLDQQWBfixedPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
def VLD2DUPq32OddPseudoWB_fixed    : VLDQQWBfixedPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
def VLD2DUPq8OddPseudoWB_register  : VLDQQWBPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
def VLD2DUPq16OddPseudoWB_register : VLDQQWBPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
def VLD2DUPq32OddPseudoWB_register : VLDQQWBPseudoInputDST<IIC_VLD2dup>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD2DUPq8OddPseudoWB_fixed` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2DUPq8OddPseudoWB_fixed`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1578-1587
```tablegen
//   VLD3DUP  : Vector Load (single 3-element structure to all lanes)
class VLD3DUP<bits<4> op7_4, string Dt>
  : NLdSt<1, 0b10, 0b1110, op7_4, (outs DPR:$Vd, DPR:$dst2, DPR:$dst3),
          (ins addrmode6dup:$Rn), IIC_VLD3dup,
          "vld3", Dt, "\\{$Vd[], $dst2[], $dst3[]\\}, $Rn", "", []>,
    Sched<[WriteVLD2]> {
  let Rm = 0b1111;
  let Inst{4} = 0;
  let DecoderMethod = "DecodeVLD3DupInstruction";
}
```
- EN: Declares reusable TableGen class `VLD3DUP` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD3DUP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1589-1591
```tablegen
def VLD3DUPd8  : VLD3DUP<{0,0,0,?}, "8">;
def VLD3DUPd16 : VLD3DUP<{0,1,0,?}, "16">;
def VLD3DUPd32 : VLD3DUP<{1,0,0,?}, "32">;
```
- EN: Defines TableGen record `VLD3DUPd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1593-1595
```tablegen
def VLD3DUPd8Pseudo  : VLDQQPseudo<IIC_VLD3dup>, Sched<[WriteVLD2]>;
def VLD3DUPd16Pseudo : VLDQQPseudo<IIC_VLD3dup>, Sched<[WriteVLD2]>;
def VLD3DUPd32Pseudo : VLDQQPseudo<IIC_VLD3dup>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD3DUPd8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPd8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1597-1600
```tablegen
// ...with double-spaced registers (not used for codegen):
def VLD3DUPq8  : VLD3DUP<{0,0,1,?}, "8">;
def VLD3DUPq16 : VLD3DUP<{0,1,1,?}, "16">;
def VLD3DUPq32 : VLD3DUP<{1,0,1,?}, "32">;
```
- EN: Defines TableGen record `VLD3DUPq8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPq8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1602-1607
```tablegen
def VLD3DUPq8EvenPseudo  : VLDQQQQPseudo<IIC_VLD3dup>, Sched<[WriteVLD2]>;
def VLD3DUPq8OddPseudo   : VLDQQQQPseudo<IIC_VLD3dup>, Sched<[WriteVLD2]>;
def VLD3DUPq16EvenPseudo : VLDQQQQPseudo<IIC_VLD3dup>, Sched<[WriteVLD2]>;
def VLD3DUPq16OddPseudo  : VLDQQQQPseudo<IIC_VLD3dup>, Sched<[WriteVLD2]>;
def VLD3DUPq32EvenPseudo : VLDQQQQPseudo<IIC_VLD3dup>, Sched<[WriteVLD2]>;
def VLD3DUPq32OddPseudo  : VLDQQQQPseudo<IIC_VLD3dup>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD3DUPq8EvenPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPq8EvenPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1609-1617
```tablegen
// ...with address register writeback:
class VLD3DUPWB<bits<4> op7_4, string Dt, Operand AddrMode>
  : NLdSt<1, 0b10, 0b1110, op7_4, (outs DPR:$Vd, DPR:$dst2, DPR:$dst3, GPR:$wb),
          (ins AddrMode:$Rn, am6offset:$Rm), IIC_VLD3dupu,
          "vld3", Dt, "\\{$Vd[], $dst2[], $dst3[]\\}, $Rn$Rm",
          "$Rn.addr = $wb", []>, Sched<[WriteVLD2]> {
  let Inst{4} = 0;
  let DecoderMethod = "DecodeVLD3DupInstruction";
}
```
- EN: Declares reusable TableGen class `VLD3DUPWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD3DUPWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1619-1621
```tablegen
def VLD3DUPd8_UPD  : VLD3DUPWB<{0,0,0,0}, "8",  addrmode6dupalign64>;
def VLD3DUPd16_UPD : VLD3DUPWB<{0,1,0,?}, "16", addrmode6dupalign64>;
def VLD3DUPd32_UPD : VLD3DUPWB<{1,0,0,?}, "32", addrmode6dupalign64>;
```
- EN: Defines TableGen record `VLD3DUPd8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPd8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1623-1625
```tablegen
def VLD3DUPq8_UPD  : VLD3DUPWB<{0,0,1,0}, "8",  addrmode6dupalign64>;
def VLD3DUPq16_UPD : VLD3DUPWB<{0,1,1,?}, "16", addrmode6dupalign64>;
def VLD3DUPq32_UPD : VLD3DUPWB<{1,0,1,?}, "32", addrmode6dupalign64>;
```
- EN: Defines TableGen record `VLD3DUPq8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPq8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1627-1629
```tablegen
def VLD3DUPd8Pseudo_UPD  : VLDQQWBPseudo<IIC_VLD3dupu>, Sched<[WriteVLD2]>;
def VLD3DUPd16Pseudo_UPD : VLDQQWBPseudo<IIC_VLD3dupu>, Sched<[WriteVLD2]>;
def VLD3DUPd32Pseudo_UPD : VLDQQWBPseudo<IIC_VLD3dupu>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD3DUPd8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPd8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1631-1633
```tablegen
def VLD3DUPq8OddPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD3dupu>, Sched<[WriteVLD2]>;
def VLD3DUPq16OddPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD3dupu>, Sched<[WriteVLD2]>;
def VLD3DUPq32OddPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD3dupu>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD3DUPq8OddPseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPq8OddPseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1635-1644
```tablegen
//   VLD4DUP  : Vector Load (single 4-element structure to all lanes)
class VLD4DUP<bits<4> op7_4, string Dt>
  : NLdSt<1, 0b10, 0b1111, op7_4,
          (outs DPR:$Vd, DPR:$dst2, DPR:$dst3, DPR:$dst4),
          (ins addrmode6dup:$Rn), IIC_VLD4dup,
          "vld4", Dt, "\\{$Vd[], $dst2[], $dst3[], $dst4[]\\}, $Rn", "", []> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLD4DupInstruction";
}
```
- EN: Declares reusable TableGen class `VLD4DUP` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD4DUP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1646-1648
```tablegen
def VLD4DUPd8  : VLD4DUP<{0,0,0,?}, "8">;
def VLD4DUPd16 : VLD4DUP<{0,1,0,?}, "16">;
def VLD4DUPd32 : VLD4DUP<{1,?,0,?}, "32"> { let Inst{6} = Rn{5}; }
```
- EN: Defines TableGen record `VLD4DUPd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1650-1652
```tablegen
def VLD4DUPd8Pseudo  : VLDQQPseudo<IIC_VLD4dup>, Sched<[WriteVLD2]>;
def VLD4DUPd16Pseudo : VLDQQPseudo<IIC_VLD4dup>, Sched<[WriteVLD2]>;
def VLD4DUPd32Pseudo : VLDQQPseudo<IIC_VLD4dup>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD4DUPd8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPd8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1654-1657
```tablegen
// ...with double-spaced registers (not used for codegen):
def VLD4DUPq8  : VLD4DUP<{0,0,1,?}, "8">;
def VLD4DUPq16 : VLD4DUP<{0,1,1,?}, "16">;
def VLD4DUPq32 : VLD4DUP<{1,?,1,?}, "32"> { let Inst{6} = Rn{5}; }
```
- EN: Defines TableGen record `VLD4DUPq8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPq8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1659-1664
```tablegen
def VLD4DUPq8EvenPseudo  : VLDQQQQPseudo<IIC_VLD4dup>, Sched<[WriteVLD2]>;
def VLD4DUPq8OddPseudo   : VLDQQQQPseudo<IIC_VLD4dup>, Sched<[WriteVLD2]>;
def VLD4DUPq16EvenPseudo : VLDQQQQPseudo<IIC_VLD4dup>, Sched<[WriteVLD2]>;
def VLD4DUPq16OddPseudo  : VLDQQQQPseudo<IIC_VLD4dup>, Sched<[WriteVLD2]>;
def VLD4DUPq32EvenPseudo : VLDQQQQPseudo<IIC_VLD4dup>, Sched<[WriteVLD2]>;
def VLD4DUPq32OddPseudo  : VLDQQQQPseudo<IIC_VLD4dup>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD4DUPq8EvenPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPq8EvenPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1666-1675
```tablegen
// ...with address register writeback:
class VLD4DUPWB<bits<4> op7_4, string Dt>
  : NLdSt<1, 0b10, 0b1111, op7_4,
          (outs DPR:$Vd, DPR:$dst2, DPR:$dst3, DPR:$dst4, GPR:$wb),
          (ins addrmode6dup:$Rn, am6offset:$Rm), IIC_VLD4dupu,
          "vld4", Dt, "\\{$Vd[], $dst2[], $dst3[], $dst4[]\\}, $Rn$Rm",
          "$Rn.addr = $wb", []>, Sched<[WriteVLD2]> {
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLD4DupInstruction";
}
```
- EN: Declares reusable TableGen class `VLD4DUPWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VLD4DUPWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1677-1679
```tablegen
def VLD4DUPd8_UPD  : VLD4DUPWB<{0,0,0,0}, "8">;
def VLD4DUPd16_UPD : VLD4DUPWB<{0,1,0,?}, "16">;
def VLD4DUPd32_UPD : VLD4DUPWB<{1,?,0,?}, "32"> { let Inst{6} = Rn{5}; }
```
- EN: Defines TableGen record `VLD4DUPd8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPd8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1681-1683
```tablegen
def VLD4DUPq8_UPD  : VLD4DUPWB<{0,0,1,0}, "8">;
def VLD4DUPq16_UPD : VLD4DUPWB<{0,1,1,?}, "16">;
def VLD4DUPq32_UPD : VLD4DUPWB<{1,?,1,?}, "32"> { let Inst{6} = Rn{5}; }
```
- EN: Defines TableGen record `VLD4DUPq8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPq8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1685-1687
```tablegen
def VLD4DUPd8Pseudo_UPD  : VLDQQWBPseudo<IIC_VLD4dupu>, Sched<[WriteVLD2]>;
def VLD4DUPd16Pseudo_UPD : VLDQQWBPseudo<IIC_VLD4dupu>, Sched<[WriteVLD2]>;
def VLD4DUPd32Pseudo_UPD : VLDQQWBPseudo<IIC_VLD4dupu>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD4DUPd8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPd8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1689-1691
```tablegen
def VLD4DUPq8OddPseudo_UPD  : VLDQQQQWBPseudo<IIC_VLD4dupu>, Sched<[WriteVLD2]>;
def VLD4DUPq16OddPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD4dupu>, Sched<[WriteVLD2]>;
def VLD4DUPq32OddPseudo_UPD : VLDQQQQWBPseudo<IIC_VLD4dupu>, Sched<[WriteVLD2]>;
```
- EN: Defines TableGen record `VLD4DUPq8OddPseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPq8OddPseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1693-1693
```tablegen
} // mayLoad = 1, hasSideEffects = 0, hasExtraDefRegAllocReq = 1
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1695-1695
```tablegen
let mayStore = 1, hasSideEffects = 0, hasExtraSrcRegAllocReq = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1697-1714
```tablegen
// Classes for VST* pseudo-instructions with multi-register operands.
// These are expanded to real instructions after register allocation.
class VSTQPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs), (ins addrmode6:$addr, QPR:$src), itin, "">;
class VSTQWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset, QPR:$src), itin,
                "$addr.addr = $wb">;
class VSTQWBfixedPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs GPR:$wb),
                (ins addrmode6:$addr, QPR:$src), itin,
                "$addr.addr = $wb">;
class VSTQWBregisterPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs GPR:$wb),
                (ins addrmode6:$addr, rGPR:$offset, QPR:$src), itin,
                "$addr.addr = $wb">;
class VSTQQPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs), (ins addrmode6:$addr, QQPR:$src), itin, "">;
```
- EN: Declares reusable TableGen class `VSTQPseudo` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VSTQPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1715-1726
```tablegen
class VSTQQWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset, QQPR:$src), itin,
                "$addr.addr = $wb">;
class VSTQQWBfixedPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs GPR:$wb),
                (ins addrmode6:$addr, QQPR:$src), itin,
                "$addr.addr = $wb">;
class VSTQQWBregisterPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs GPR:$wb),
                (ins addrmode6:$addr, rGPR:$offset, QQPR:$src), itin,
                "$addr.addr = $wb">;
```
- EN: Declares reusable TableGen class `VSTQQWBPseudo` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VSTQQWBPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1728-1733
```tablegen
class VSTQQQQPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs), (ins addrmode6:$addr, QQQQPR:$src), itin, "">;
class VSTQQQQWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset, QQQQPR:$src), itin,
                "$addr.addr = $wb">;
```
- EN: Declares reusable TableGen class `VSTQQQQPseudo` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VSTQQQQPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1735-1749
```tablegen
//   VST1     : Vector Store (multiple single elements)
class VST1D<bits<4> op7_4, string Dt, Operand AddrMode>
  : NLdSt<0,0b00,0b0111,op7_4, (outs), (ins AddrMode:$Rn, VecListOneD:$Vd),
          IIC_VST1, "vst1", Dt, "$Vd, $Rn", "", []>, Sched<[WriteVST1]> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLDST1Instruction";
}
class VST1Q<bits<4> op7_4, string Dt, Operand AddrMode>
  : NLdSt<0,0b00,0b1010,op7_4, (outs), (ins AddrMode:$Rn, VecListDPair:$Vd),
          IIC_VST1x2, "vst1", Dt, "$Vd, $Rn", "", []>, Sched<[WriteVST2]> {
  let Rm = 0b1111;
  let Inst{5-4} = Rn{5-4};
  let DecoderMethod = "DecodeVLDST1Instruction";
}
```
- EN: Declares reusable TableGen class `VST1D` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST1D`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1751-1754
```tablegen
def  VST1d8   : VST1D<{0,0,0,?}, "8",  addrmode6align64>;
def  VST1d16  : VST1D<{0,1,0,?}, "16", addrmode6align64>;
def  VST1d32  : VST1D<{1,0,0,?}, "32", addrmode6align64>;
def  VST1d64  : VST1D<{1,1,0,?}, "64", addrmode6align64>;
```
- EN: Defines TableGen record `VST1d8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1d8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1756-1759
```tablegen
def  VST1q8   : VST1Q<{0,0,?,?}, "8",  addrmode6align64or128>;
def  VST1q16  : VST1Q<{0,1,?,?}, "16", addrmode6align64or128>;
def  VST1q32  : VST1Q<{1,0,?,?}, "32", addrmode6align64or128>;
def  VST1q64  : VST1Q<{1,1,?,?}, "64", addrmode6align64or128>;
```
- EN: Defines TableGen record `VST1q8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1q8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1761-1778
```tablegen
// ...with address register writeback:
multiclass VST1DWB<bits<4> op7_4, string Dt, Operand AddrMode> {
  def _fixed : NLdSt<0,0b00, 0b0111,op7_4, (outs GPR:$wb),
                     (ins AddrMode:$Rn, VecListOneD:$Vd), IIC_VLD1u,
                     "vst1", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVST1]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{4} = Rn{4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
  def _register : NLdSt<0,0b00,0b0111,op7_4, (outs GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm, VecListOneD:$Vd),
                        IIC_VLD1u,
                        "vst1", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []>, Sched<[WriteVST1]> {
    let Inst{4} = Rn{4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
```
- EN: Declares TableGen `multiclass VST1DWB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VST1DWB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1779-1796
```tablegen
}
multiclass VST1QWB<bits<4> op7_4, string Dt, Operand AddrMode> {
  def _fixed : NLdSt<0,0b00,0b1010,op7_4, (outs GPR:$wb),
                    (ins AddrMode:$Rn, VecListDPair:$Vd), IIC_VLD1x2u,
                     "vst1", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVST2]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
  def _register : NLdSt<0,0b00,0b1010,op7_4, (outs GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm, VecListDPair:$Vd),
                        IIC_VLD1x2u,
                        "vst1", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []>, Sched<[WriteVST2]> {
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
```
- EN: Declares TableGen `multiclass VST1QWB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VST1QWB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1797-1797
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1799-1802
```tablegen
defm VST1d8wb  : VST1DWB<{0,0,0,?}, "8",  addrmode6align64>;
defm VST1d16wb : VST1DWB<{0,1,0,?}, "16", addrmode6align64>;
defm VST1d32wb : VST1DWB<{1,0,0,?}, "32", addrmode6align64>;
defm VST1d64wb : VST1DWB<{1,1,0,?}, "64", addrmode6align64>;
```
- EN: Defines TableGen record `VST1d8wb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1d8wb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1804-1807
```tablegen
defm VST1q8wb  : VST1QWB<{0,0,?,?}, "8",  addrmode6align64or128>;
defm VST1q16wb : VST1QWB<{0,1,?,?}, "16", addrmode6align64or128>;
defm VST1q32wb : VST1QWB<{1,0,?,?}, "32", addrmode6align64or128>;
defm VST1q64wb : VST1QWB<{1,1,?,?}, "64", addrmode6align64or128>;
```
- EN: Defines TableGen record `VST1q8wb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1q8wb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1809-1826
```tablegen
// ...with 3 registers
class VST1D3<bits<4> op7_4, string Dt, Operand AddrMode>
  : NLdSt<0, 0b00, 0b0110, op7_4, (outs),
          (ins AddrMode:$Rn, VecListThreeD:$Vd),
          IIC_VST1x3, "vst1", Dt, "$Vd, $Rn", "", []>, Sched<[WriteVST3]> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLDST1Instruction";
}
multiclass VST1D3WB<bits<4> op7_4, string Dt, Operand AddrMode> {
  def _fixed : NLdSt<0,0b00,0b0110,op7_4, (outs GPR:$wb),
                    (ins AddrMode:$Rn, VecListThreeD:$Vd), IIC_VLD1x3u,
                     "vst1", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVST3]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
```
- EN: Declares TableGen `multiclass VST1D3WB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VST1D3WB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1827-1835
```tablegen
  def _register : NLdSt<0,0b00,0b0110,op7_4, (outs GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm, VecListThreeD:$Vd),
                        IIC_VLD1x3u,
                        "vst1", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []>, Sched<[WriteVST3]> {
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
}
```
- EN: Defines TableGen record `_register` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_register`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1837-1840
```tablegen
def VST1d8T     : VST1D3<{0,0,0,?}, "8",  addrmode6align64>;
def VST1d16T    : VST1D3<{0,1,0,?}, "16", addrmode6align64>;
def VST1d32T    : VST1D3<{1,0,0,?}, "32", addrmode6align64>;
def VST1d64T    : VST1D3<{1,1,0,?}, "64", addrmode6align64>;
```
- EN: Defines TableGen record `VST1d8T` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1d8T`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1842-1845
```tablegen
defm VST1d8Twb  : VST1D3WB<{0,0,0,?}, "8",  addrmode6align64>;
defm VST1d16Twb : VST1D3WB<{0,1,0,?}, "16", addrmode6align64>;
defm VST1d32Twb : VST1D3WB<{1,0,0,?}, "32", addrmode6align64>;
defm VST1d64Twb : VST1D3WB<{1,1,0,?}, "64", addrmode6align64>;
```
- EN: Defines TableGen record `VST1d8Twb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1d8Twb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1847-1858
```tablegen
def VST1d8TPseudo             : VSTQQPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1d8TPseudoWB_fixed     : VSTQQWBfixedPseudo<IIC_VST1x3u>, Sched<[WriteVST3]>;
def VST1d8TPseudoWB_register  : VSTQQWBPseudo<IIC_VST1x3u>, Sched<[WriteVST3]>;
def VST1d16TPseudo            : VSTQQPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1d16TPseudoWB_fixed    : VSTQQWBfixedPseudo<IIC_VST1x3u>, Sched<[WriteVST3]>;
def VST1d16TPseudoWB_register : VSTQQWBPseudo<IIC_VST1x3u>, Sched<[WriteVST3]>;
def VST1d32TPseudo            : VSTQQPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1d32TPseudoWB_fixed    : VSTQQWBfixedPseudo<IIC_VST1x3u>, Sched<[WriteVST3]>;
def VST1d32TPseudoWB_register : VSTQQWBPseudo<IIC_VST1x3u>, Sched<[WriteVST3]>;
def VST1d64TPseudo            : VSTQQPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1d64TPseudoWB_fixed    : VSTQQWBfixedPseudo<IIC_VST1x3u>, Sched<[WriteVST3]>;
def VST1d64TPseudoWB_register : VSTQQWBPseudo<IIC_VST1x3u>, Sched<[WriteVST3]>;
```
- EN: Defines TableGen record `VST1d8TPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1d8TPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1860-1863
```tablegen
def VST1q8HighTPseudo     : VSTQQQQPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1q16HighTPseudo    : VSTQQQQPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1q32HighTPseudo    : VSTQQQQPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1q64HighTPseudo    : VSTQQQQPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
```
- EN: Defines TableGen record `VST1q8HighTPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1q8HighTPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1865-1868
```tablegen
def VST1q8HighTPseudo_UPD  : VSTQQQQWBPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1q16HighTPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1q32HighTPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1q64HighTPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
```
- EN: Defines TableGen record `VST1q8HighTPseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1q8HighTPseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1870-1873
```tablegen
def VST1q8LowTPseudo_UPD  : VSTQQQQWBPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1q16LowTPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1q32LowTPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
def VST1q64LowTPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x3>, Sched<[WriteVST3]>;
```
- EN: Defines TableGen record `VST1q8LowTPseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1q8LowTPseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1875-1892
```tablegen
// ...with 4 registers
class VST1D4<bits<4> op7_4, string Dt, Operand AddrMode>
  : NLdSt<0, 0b00, 0b0010, op7_4, (outs),
          (ins AddrMode:$Rn, VecListFourD:$Vd),
          IIC_VST1x4, "vst1", Dt, "$Vd, $Rn", "",
          []>, Sched<[WriteVST4]> {
  let Rm = 0b1111;
  let Inst{5-4} = Rn{5-4};
  let DecoderMethod = "DecodeVLDST1Instruction";
}
multiclass VST1D4WB<bits<4> op7_4, string Dt, Operand AddrMode> {
  def _fixed : NLdSt<0,0b00,0b0010,op7_4, (outs GPR:$wb),
                    (ins AddrMode:$Rn, VecListFourD:$Vd), IIC_VLD1x4u,
                     "vst1", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVST4]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST1Instruction";
```
- EN: Declares TableGen `multiclass VST1D4WB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VST1D4WB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1893-1902
```tablegen
  }
  def _register : NLdSt<0,0b00,0b0010,op7_4, (outs GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm, VecListFourD:$Vd),
                        IIC_VLD1x4u,
                        "vst1", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []>, Sched<[WriteVST4]> {
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST1Instruction";
  }
}
```
- EN: Defines TableGen record `_register` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_register`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1904-1907
```tablegen
def VST1d8Q     : VST1D4<{0,0,?,?}, "8",  addrmode6align64or128or256>;
def VST1d16Q    : VST1D4<{0,1,?,?}, "16", addrmode6align64or128or256>;
def VST1d32Q    : VST1D4<{1,0,?,?}, "32", addrmode6align64or128or256>;
def VST1d64Q    : VST1D4<{1,1,?,?}, "64", addrmode6align64or128or256>;
```
- EN: Defines TableGen record `VST1d8Q` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1d8Q`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1909-1912
```tablegen
defm VST1d8Qwb  : VST1D4WB<{0,0,?,?}, "8",  addrmode6align64or128or256>;
defm VST1d16Qwb : VST1D4WB<{0,1,?,?}, "16", addrmode6align64or128or256>;
defm VST1d32Qwb : VST1D4WB<{1,0,?,?}, "32", addrmode6align64or128or256>;
defm VST1d64Qwb : VST1D4WB<{1,1,?,?}, "64", addrmode6align64or128or256>;
```
- EN: Defines TableGen record `VST1d8Qwb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1d8Qwb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1914-1925
```tablegen
def VST1d8QPseudo             : VSTQQPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1d8QPseudoWB_fixed     : VSTQQWBfixedPseudo<IIC_VST1x4u>, Sched<[WriteVST4]>;
def VST1d8QPseudoWB_register  : VSTQQWBPseudo<IIC_VST1x4u>, Sched<[WriteVST4]>;
def VST1d16QPseudo            : VSTQQPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1d16QPseudoWB_fixed    : VSTQQWBfixedPseudo<IIC_VST1x4u>, Sched<[WriteVST4]>;
def VST1d16QPseudoWB_register : VSTQQWBPseudo<IIC_VST1x4u>, Sched<[WriteVST4]>;
def VST1d32QPseudo            : VSTQQPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1d32QPseudoWB_fixed    : VSTQQWBfixedPseudo<IIC_VST1x4u>, Sched<[WriteVST4]>;
def VST1d32QPseudoWB_register : VSTQQWBPseudo<IIC_VST1x4u>, Sched<[WriteVST4]>;
def VST1d64QPseudo            : VSTQQPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1d64QPseudoWB_fixed    : VSTQQWBfixedPseudo<IIC_VST1x4u>, Sched<[WriteVST4]>;
def VST1d64QPseudoWB_register : VSTQQWBPseudo<IIC_VST1x4u>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST1d8QPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1d8QPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1927-1930
```tablegen
def VST1q8HighQPseudo     : VSTQQQQPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1q16HighQPseudo    : VSTQQQQPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1q32HighQPseudo    : VSTQQQQPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1q64HighQPseudo    : VSTQQQQPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST1q8HighQPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1q8HighQPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1932-1935
```tablegen
def VST1q8HighQPseudo_UPD  : VSTQQQQWBPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1q16HighQPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1q32HighQPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1q64HighQPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST1q8HighQPseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1q8HighQPseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1937-1940
```tablegen
def VST1q8LowQPseudo_UPD  : VSTQQQQWBPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1q16LowQPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1q32LowQPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
def VST1q64LowQPseudo_UPD : VSTQQQQWBPseudo<IIC_VST1x4>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST1q8LowQPseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1q8LowQPseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1942-1950
```tablegen
//   VST2     : Vector Store (multiple 2-element structures)
class VST2<bits<4> op11_8, bits<4> op7_4, string Dt, RegisterOperand VdTy,
            InstrItinClass itin, Operand AddrMode>
  : NLdSt<0, 0b00, op11_8, op7_4, (outs), (ins AddrMode:$Rn, VdTy:$Vd),
          itin, "vst2", Dt, "$Vd, $Rn", "", []> {
  let Rm = 0b1111;
  let Inst{5-4} = Rn{5-4};
  let DecoderMethod = "DecodeVLDST2Instruction";
}
```
- EN: Declares reusable TableGen class `VST2` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1952-1957
```tablegen
def  VST2d8   : VST2<0b1000, {0,0,?,?}, "8",  VecListDPair, IIC_VST2,
                     addrmode6align64or128>, Sched<[WriteVST2]>;
def  VST2d16  : VST2<0b1000, {0,1,?,?}, "16", VecListDPair, IIC_VST2,
                     addrmode6align64or128>, Sched<[WriteVST2]>;
def  VST2d32  : VST2<0b1000, {1,0,?,?}, "32", VecListDPair, IIC_VST2,
                     addrmode6align64or128>, Sched<[WriteVST2]>;
```
- EN: Defines TableGen record `VST2d8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2d8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1959-1964
```tablegen
def  VST2q8   : VST2<0b0011, {0,0,?,?}, "8",  VecListFourD, IIC_VST2x2,
                     addrmode6align64or128or256>, Sched<[WriteVST4]>;
def  VST2q16  : VST2<0b0011, {0,1,?,?}, "16", VecListFourD, IIC_VST2x2,
                     addrmode6align64or128or256>, Sched<[WriteVST4]>;
def  VST2q32  : VST2<0b0011, {1,0,?,?}, "32", VecListFourD, IIC_VST2x2,
                     addrmode6align64or128or256>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST2q8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2q8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1966-1968
```tablegen
def  VST2q8Pseudo  : VSTQQPseudo<IIC_VST2x2>, Sched<[WriteVST4]>;
def  VST2q16Pseudo : VSTQQPseudo<IIC_VST2x2>, Sched<[WriteVST4]>;
def  VST2q32Pseudo : VSTQQPseudo<IIC_VST2x2>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST2q8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2q8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1970-1987
```tablegen
// ...with address register writeback:
multiclass VST2DWB<bits<4> op11_8, bits<4> op7_4, string Dt,
                   RegisterOperand VdTy, Operand AddrMode> {
  def _fixed : NLdSt<0, 0b00, op11_8, op7_4, (outs GPR:$wb),
                     (ins AddrMode:$Rn, VdTy:$Vd), IIC_VLD1u,
                     "vst2", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVST2]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST2Instruction";
  }
  def _register : NLdSt<0, 0b00, op11_8, op7_4, (outs GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm, VdTy:$Vd), IIC_VLD1u,
                        "vst2", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []>, Sched<[WriteVST2]> {
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST2Instruction";
  }
```
- EN: Declares TableGen `multiclass VST2DWB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VST2DWB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1988-2005
```tablegen
}
multiclass VST2QWB<bits<4> op7_4, string Dt, Operand AddrMode> {
  def _fixed : NLdSt<0, 0b00, 0b0011, op7_4, (outs GPR:$wb),
                     (ins AddrMode:$Rn, VecListFourD:$Vd), IIC_VLD1u,
                     "vst2", Dt, "$Vd, $Rn!",
                     "$Rn.addr = $wb", []>, Sched<[WriteVST4]> {
    let Rm = 0b1101; // NLdSt will assign to the right encoding bits.
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST2Instruction";
  }
  def _register : NLdSt<0, 0b00, 0b0011, op7_4, (outs GPR:$wb),
                        (ins AddrMode:$Rn, rGPR:$Rm, VecListFourD:$Vd),
                        IIC_VLD1u,
                        "vst2", Dt, "$Vd, $Rn, $Rm",
                        "$Rn.addr = $wb", []>, Sched<[WriteVST4]> {
    let Inst{5-4} = Rn{5-4};
    let DecoderMethod = "DecodeVLDST2Instruction";
  }
```
- EN: Declares TableGen `multiclass VST2QWB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VST2QWB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2006-2006
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2008-2013
```tablegen
defm VST2d8wb    : VST2DWB<0b1000, {0,0,?,?}, "8",  VecListDPair,
                           addrmode6align64or128>;
defm VST2d16wb   : VST2DWB<0b1000, {0,1,?,?}, "16", VecListDPair,
                           addrmode6align64or128>;
defm VST2d32wb   : VST2DWB<0b1000, {1,0,?,?}, "32", VecListDPair,
                           addrmode6align64or128>;
```
- EN: Defines TableGen record `VST2d8wb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2d8wb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2015-2017
```tablegen
defm VST2q8wb    : VST2QWB<{0,0,?,?}, "8", addrmode6align64or128or256>;
defm VST2q16wb   : VST2QWB<{0,1,?,?}, "16", addrmode6align64or128or256>;
defm VST2q32wb   : VST2QWB<{1,0,?,?}, "32", addrmode6align64or128or256>;
```
- EN: Defines TableGen record `VST2q8wb` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2q8wb`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2019-2024
```tablegen
def VST2q8PseudoWB_fixed     : VSTQQWBfixedPseudo<IIC_VST2x2u>, Sched<[WriteVST4]>;
def VST2q16PseudoWB_fixed    : VSTQQWBfixedPseudo<IIC_VST2x2u>, Sched<[WriteVST4]>;
def VST2q32PseudoWB_fixed    : VSTQQWBfixedPseudo<IIC_VST2x2u>, Sched<[WriteVST4]>;
def VST2q8PseudoWB_register  : VSTQQWBregisterPseudo<IIC_VST2x2u>, Sched<[WriteVST4]>;
def VST2q16PseudoWB_register : VSTQQWBregisterPseudo<IIC_VST2x2u>, Sched<[WriteVST4]>;
def VST2q32PseudoWB_register : VSTQQWBregisterPseudo<IIC_VST2x2u>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST2q8PseudoWB_fixed` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2q8PseudoWB_fixed`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2026-2038
```tablegen
// ...with double-spaced registers
def VST2b8      : VST2<0b1001, {0,0,?,?}, "8",  VecListDPairSpaced, IIC_VST2,
                      addrmode6align64or128>;
def VST2b16     : VST2<0b1001, {0,1,?,?}, "16", VecListDPairSpaced, IIC_VST2,
                      addrmode6align64or128>;
def VST2b32     : VST2<0b1001, {1,0,?,?}, "32", VecListDPairSpaced, IIC_VST2,
                      addrmode6align64or128>;
defm VST2b8wb   : VST2DWB<0b1001, {0,0,?,?}, "8",  VecListDPairSpaced,
                          addrmode6align64or128>;
defm VST2b16wb  : VST2DWB<0b1001, {0,1,?,?}, "16", VecListDPairSpaced,
                          addrmode6align64or128>;
defm VST2b32wb  : VST2DWB<0b1001, {1,0,?,?}, "32", VecListDPairSpaced,
                          addrmode6align64or128>;
```
- EN: Defines TableGen record `VST2b8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2b8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2040-2048
```tablegen
//   VST3     : Vector Store (multiple 3-element structures)
class VST3D<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdSt<0, 0b00, op11_8, op7_4, (outs),
          (ins addrmode6:$Rn, DPR:$Vd, DPR:$src2, DPR:$src3), IIC_VST3,
          "vst3", Dt, "\\{$Vd, $src2, $src3\\}, $Rn", "", []>, Sched<[WriteVST3]> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLDST3Instruction";
}
```
- EN: Declares reusable TableGen class `VST3D` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST3D`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2050-2052
```tablegen
def  VST3d8   : VST3D<0b0100, {0,0,0,?}, "8">;
def  VST3d16  : VST3D<0b0100, {0,1,0,?}, "16">;
def  VST3d32  : VST3D<0b0100, {1,0,0,?}, "32">;
```
- EN: Defines TableGen record `VST3d8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3d8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2054-2056
```tablegen
def  VST3d8Pseudo  : VSTQQPseudo<IIC_VST3>, Sched<[WriteVST3]>;
def  VST3d16Pseudo : VSTQQPseudo<IIC_VST3>, Sched<[WriteVST3]>;
def  VST3d32Pseudo : VSTQQPseudo<IIC_VST3>, Sched<[WriteVST3]>;
```
- EN: Defines TableGen record `VST3d8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3d8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2058-2067
```tablegen
// ...with address register writeback:
class VST3DWB<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdSt<0, 0b00, op11_8, op7_4, (outs GPR:$wb),
          (ins addrmode6:$Rn, am6offset:$Rm,
           DPR:$Vd, DPR:$src2, DPR:$src3), IIC_VST3u,
          "vst3", Dt, "\\{$Vd, $src2, $src3\\}, $Rn$Rm",
          "$Rn.addr = $wb", []>, Sched<[WriteVST3]> {
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVLDST3Instruction";
}
```
- EN: Declares reusable TableGen class `VST3DWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST3DWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2069-2071
```tablegen
def VST3d8_UPD  : VST3DWB<0b0100, {0,0,0,?}, "8">;
def VST3d16_UPD : VST3DWB<0b0100, {0,1,0,?}, "16">;
def VST3d32_UPD : VST3DWB<0b0100, {1,0,0,?}, "32">;
```
- EN: Defines TableGen record `VST3d8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3d8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2073-2075
```tablegen
def VST3d8Pseudo_UPD  : VSTQQWBPseudo<IIC_VST3u>, Sched<[WriteVST3]>;
def VST3d16Pseudo_UPD : VSTQQWBPseudo<IIC_VST3u>, Sched<[WriteVST3]>;
def VST3d32Pseudo_UPD : VSTQQWBPseudo<IIC_VST3u>, Sched<[WriteVST3]>;
```
- EN: Defines TableGen record `VST3d8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3d8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2077-2083
```tablegen
// ...with double-spaced registers:
def VST3q8      : VST3D<0b0101, {0,0,0,?}, "8">;
def VST3q16     : VST3D<0b0101, {0,1,0,?}, "16">;
def VST3q32     : VST3D<0b0101, {1,0,0,?}, "32">;
def VST3q8_UPD  : VST3DWB<0b0101, {0,0,0,?}, "8">;
def VST3q16_UPD : VST3DWB<0b0101, {0,1,0,?}, "16">;
def VST3q32_UPD : VST3DWB<0b0101, {1,0,0,?}, "32">;
```
- EN: Defines TableGen record `VST3q8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3q8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2085-2087
```tablegen
def VST3q8Pseudo_UPD  : VSTQQQQWBPseudo<IIC_VST3u>, Sched<[WriteVST3]>;
def VST3q16Pseudo_UPD : VSTQQQQWBPseudo<IIC_VST3u>, Sched<[WriteVST3]>;
def VST3q32Pseudo_UPD : VSTQQQQWBPseudo<IIC_VST3u>, Sched<[WriteVST3]>;
```
- EN: Defines TableGen record `VST3q8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3q8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2089-2092
```tablegen
// ...alternate versions to be allocated odd register numbers:
def VST3q8oddPseudo   : VSTQQQQPseudo<IIC_VST3>, Sched<[WriteVST3]>;
def VST3q16oddPseudo  : VSTQQQQPseudo<IIC_VST3>, Sched<[WriteVST3]>;
def VST3q32oddPseudo  : VSTQQQQPseudo<IIC_VST3>, Sched<[WriteVST3]>;
```
- EN: Defines TableGen record `VST3q8oddPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3q8oddPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2094-2096
```tablegen
def VST3q8oddPseudo_UPD  : VSTQQQQWBPseudo<IIC_VST3u>, Sched<[WriteVST3]>;
def VST3q16oddPseudo_UPD : VSTQQQQWBPseudo<IIC_VST3u>, Sched<[WriteVST3]>;
def VST3q32oddPseudo_UPD : VSTQQQQWBPseudo<IIC_VST3u>, Sched<[WriteVST3]>;
```
- EN: Defines TableGen record `VST3q8oddPseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3q8oddPseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2098-2107
```tablegen
//   VST4     : Vector Store (multiple 4-element structures)
class VST4D<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdSt<0, 0b00, op11_8, op7_4, (outs),
          (ins addrmode6:$Rn, DPR:$Vd, DPR:$src2, DPR:$src3, DPR:$src4),
          IIC_VST4, "vst4", Dt, "\\{$Vd, $src2, $src3, $src4\\}, $Rn",
          "", []>, Sched<[WriteVST4]> {
  let Rm = 0b1111;
  let Inst{5-4} = Rn{5-4};
  let DecoderMethod = "DecodeVLDST4Instruction";
}
```
- EN: Declares reusable TableGen class `VST4D` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST4D`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2109-2111
```tablegen
def  VST4d8   : VST4D<0b0000, {0,0,?,?}, "8">;
def  VST4d16  : VST4D<0b0000, {0,1,?,?}, "16">;
def  VST4d32  : VST4D<0b0000, {1,0,?,?}, "32">;
```
- EN: Defines TableGen record `VST4d8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4d8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2113-2115
```tablegen
def  VST4d8Pseudo  : VSTQQPseudo<IIC_VST4>, Sched<[WriteVST4]>;
def  VST4d16Pseudo : VSTQQPseudo<IIC_VST4>, Sched<[WriteVST4]>;
def  VST4d32Pseudo : VSTQQPseudo<IIC_VST4>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST4d8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4d8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2117-2126
```tablegen
// ...with address register writeback:
class VST4DWB<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdSt<0, 0b00, op11_8, op7_4, (outs GPR:$wb),
          (ins addrmode6:$Rn, am6offset:$Rm,
           DPR:$Vd, DPR:$src2, DPR:$src3, DPR:$src4), IIC_VST4u,
           "vst4", Dt, "\\{$Vd, $src2, $src3, $src4\\}, $Rn$Rm",
          "$Rn.addr = $wb", []>, Sched<[WriteVST4]> {
  let Inst{5-4} = Rn{5-4};
  let DecoderMethod = "DecodeVLDST4Instruction";
}
```
- EN: Declares reusable TableGen class `VST4DWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST4DWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2128-2130
```tablegen
def VST4d8_UPD  : VST4DWB<0b0000, {0,0,?,?}, "8">;
def VST4d16_UPD : VST4DWB<0b0000, {0,1,?,?}, "16">;
def VST4d32_UPD : VST4DWB<0b0000, {1,0,?,?}, "32">;
```
- EN: Defines TableGen record `VST4d8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4d8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2132-2134
```tablegen
def VST4d8Pseudo_UPD  : VSTQQWBPseudo<IIC_VST4u>, Sched<[WriteVST4]>;
def VST4d16Pseudo_UPD : VSTQQWBPseudo<IIC_VST4u>, Sched<[WriteVST4]>;
def VST4d32Pseudo_UPD : VSTQQWBPseudo<IIC_VST4u>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST4d8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4d8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2136-2142
```tablegen
// ...with double-spaced registers:
def VST4q8      : VST4D<0b0001, {0,0,?,?}, "8">;
def VST4q16     : VST4D<0b0001, {0,1,?,?}, "16">;
def VST4q32     : VST4D<0b0001, {1,0,?,?}, "32">;
def VST4q8_UPD  : VST4DWB<0b0001, {0,0,?,?}, "8">;
def VST4q16_UPD : VST4DWB<0b0001, {0,1,?,?}, "16">;
def VST4q32_UPD : VST4DWB<0b0001, {1,0,?,?}, "32">;
```
- EN: Defines TableGen record `VST4q8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4q8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2144-2146
```tablegen
def VST4q8Pseudo_UPD  : VSTQQQQWBPseudo<IIC_VST4u>, Sched<[WriteVST4]>;
def VST4q16Pseudo_UPD : VSTQQQQWBPseudo<IIC_VST4u>, Sched<[WriteVST4]>;
def VST4q32Pseudo_UPD : VSTQQQQWBPseudo<IIC_VST4u>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST4q8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4q8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2148-2151
```tablegen
// ...alternate versions to be allocated odd register numbers:
def VST4q8oddPseudo   : VSTQQQQPseudo<IIC_VST4>, Sched<[WriteVST4]>;
def VST4q16oddPseudo  : VSTQQQQPseudo<IIC_VST4>, Sched<[WriteVST4]>;
def VST4q32oddPseudo  : VSTQQQQPseudo<IIC_VST4>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST4q8oddPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4q8oddPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2153-2155
```tablegen
def VST4q8oddPseudo_UPD  : VSTQQQQWBPseudo<IIC_VST4u>, Sched<[WriteVST4]>;
def VST4q16oddPseudo_UPD : VSTQQQQWBPseudo<IIC_VST4u>, Sched<[WriteVST4]>;
def VST4q32oddPseudo_UPD : VSTQQQQWBPseudo<IIC_VST4u>, Sched<[WriteVST4]>;
```
- EN: Defines TableGen record `VST4q8oddPseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4q8oddPseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2157-2157
```tablegen
} // mayStore = 1, hasSideEffects = 0, hasExtraSrcRegAllocReq = 1
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2159-2176
```tablegen
// Classes for VST*LN pseudo-instructions with multi-register operands.
// These are expanded to real instructions after register allocation.
class VSTQLNPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs), (ins addrmode6:$addr, QPR:$src, nohash_imm:$lane),
                itin, "">;
class VSTQLNWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset, QPR:$src,
                 nohash_imm:$lane), itin, "$addr.addr = $wb">;
class VSTQQLNPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs), (ins addrmode6:$addr, QQPR:$src, nohash_imm:$lane),
                itin, "">;
class VSTQQLNWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset, QQPR:$src,
                 nohash_imm:$lane), itin, "$addr.addr = $wb">;
class VSTQQQQLNPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs), (ins addrmode6:$addr, QQQQPR:$src, nohash_imm:$lane),
```
- EN: Declares reusable TableGen class `VSTQLNPseudo` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VSTQLNPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2177-2181
```tablegen
                itin, "">;
class VSTQQQQLNWBPseudo<InstrItinClass itin>
  : PseudoNLdSt<(outs GPR:$wb),
                (ins addrmode6:$addr, am6offset:$offset, QQQQPR:$src,
                 nohash_imm:$lane), itin, "$addr.addr = $wb">;
```
- EN: Declares reusable TableGen class `VSTQQQQLNWBPseudo` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VSTQQQQLNWBPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2183-2198
```tablegen
//   VST1LN   : Vector Store (single element from one lane)
class VST1LN<bits<4> op11_8, bits<4> op7_4, string Dt, ValueType Ty,
             PatFrag StoreOp, SDNode ExtractOp, Operand AddrMode>
  : NLdStLn<1, 0b00, op11_8, op7_4, (outs),
          (ins AddrMode:$Rn, DPR:$Vd, nohash_imm:$lane),
          IIC_VST1ln, "vst1", Dt, "\\{$Vd[$lane]\\}, $Rn", "",
          [(StoreOp (ExtractOp (Ty DPR:$Vd), imm:$lane), AddrMode:$Rn)]>,
     Sched<[WriteVST1]> {
  let Rm = 0b1111;
  let DecoderMethod = "DecodeVST1LN";
}
class VST1QLNPseudo<ValueType Ty, PatFrag StoreOp, SDNode ExtractOp>
  : VSTQLNPseudo<IIC_VST1ln>, Sched<[WriteVST1]> {
  let Pattern = [(StoreOp (ExtractOp (Ty QPR:$src), imm:$lane),
                          addrmode6:$addr)];
}
```
- EN: Declares reusable TableGen class `VST1LN` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST1LN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2200-2208
```tablegen
def VST1LNd8  : VST1LN<0b0000, {?,?,?,0}, "8", v8i8, truncstorei8,
                       ARMvgetlaneu, addrmode6> {
  let Inst{7-5} = lane{2-0};
}
def VST1LNd16 : VST1LN<0b0100, {?,?,0,?}, "16", v4i16, truncstorei16,
                       ARMvgetlaneu, addrmode6> {
  let Inst{7-6} = lane{1-0};
  let Inst{4}   = Rn{4};
}
```
- EN: Defines TableGen record `VST1LNd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1LNd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2210-2214
```tablegen
def VST1LNd32 : VST1LN<0b1000, {?,0,?,?}, "32", v2i32, store, extractelt,
                       addrmode6oneL32> {
  let Inst{7}   = lane{0};
  let Inst{5-4} = Rn{5-4};
}
```
- EN: Defines TableGen record `VST1LNd32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1LNd32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2216-2218
```tablegen
def VST1LNq8Pseudo  : VST1QLNPseudo<v16i8, truncstorei8, ARMvgetlaneu>;
def VST1LNq16Pseudo : VST1QLNPseudo<v8i16, truncstorei16, ARMvgetlaneu>;
def VST1LNq32Pseudo : VST1QLNPseudo<v4i32, store, extractelt>;
```
- EN: Defines TableGen record `VST1LNq8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1LNq8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2220-2224
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(store (extractelt (v2f32 DPR:$src), imm:$lane), addrmode6:$addr),
          (VST1LNd32 addrmode6:$addr, DPR:$src, imm:$lane)>;
def : Pat<(store (extractelt (v4f32 QPR:$src), imm:$lane), addrmode6:$addr),
          (VST1LNq32Pseudo addrmode6:$addr, QPR:$src, imm:$lane)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2226-2230
```tablegen
def : Pat<(store (extractelt (v4f16 DPR:$src), imm:$lane), addrmode6:$addr),
          (VST1LNd16 addrmode6:$addr, DPR:$src, imm:$lane)>;
def : Pat<(store (extractelt (v8f16 QPR:$src), imm:$lane), addrmode6:$addr),
          (VST1LNq16Pseudo addrmode6:$addr, QPR:$src, imm:$lane)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2232-2249
```tablegen
// ...with address register writeback:
class VST1LNWB<bits<4> op11_8, bits<4> op7_4, string Dt, ValueType Ty,
               PatFrag StoreOp, SDNode ExtractOp, Operand AdrMode>
  : NLdStLn<1, 0b00, op11_8, op7_4, (outs GPR:$wb),
          (ins AdrMode:$Rn, am6offset:$Rm,
           DPR:$Vd, nohash_imm:$lane), IIC_VST1lnu, "vst1", Dt,
          "\\{$Vd[$lane]\\}, $Rn$Rm",
          "$Rn.addr = $wb",
          [(set GPR:$wb, (StoreOp (ExtractOp (Ty DPR:$Vd), imm:$lane),
                                  AdrMode:$Rn, am6offset:$Rm))]>,
    Sched<[WriteVST1]> {
  let DecoderMethod = "DecodeVST1LN";
}
class VST1QLNWBPseudo<ValueType Ty, PatFrag StoreOp, SDNode ExtractOp>
  : VSTQLNWBPseudo<IIC_VST1lnu>, Sched<[WriteVST1]> {
  let Pattern = [(set GPR:$wb, (StoreOp (ExtractOp (Ty QPR:$src), imm:$lane),
                                        addrmode6:$addr, am6offset:$offset))];
}
```
- EN: Declares reusable TableGen class `VST1LNWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST1LNWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2251-2264
```tablegen
def VST1LNd8_UPD  : VST1LNWB<0b0000, {?,?,?,0}, "8", v8i8, post_truncsti8,
                             ARMvgetlaneu, addrmode6> {
  let Inst{7-5} = lane{2-0};
}
def VST1LNd16_UPD : VST1LNWB<0b0100, {?,?,0,?}, "16", v4i16, post_truncsti16,
                             ARMvgetlaneu, addrmode6> {
  let Inst{7-6} = lane{1-0};
  let Inst{4}   = Rn{4};
}
def VST1LNd32_UPD : VST1LNWB<0b1000, {?,0,?,?}, "32", v2i32, post_store,
                             extractelt, addrmode6oneL32> {
  let Inst{7}   = lane{0};
  let Inst{5-4} = Rn{5-4};
}
```
- EN: Defines TableGen record `VST1LNd8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1LNd8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2266-2268
```tablegen
def VST1LNq8Pseudo_UPD  : VST1QLNWBPseudo<v16i8, post_truncsti8, ARMvgetlaneu>;
def VST1LNq16Pseudo_UPD : VST1QLNWBPseudo<v8i16, post_truncsti16,ARMvgetlaneu>;
def VST1LNq32Pseudo_UPD : VST1QLNWBPseudo<v4i32, post_store, extractelt>;
```
- EN: Defines TableGen record `VST1LNq8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1LNq8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2270-2270
```tablegen
let mayStore = 1, hasSideEffects = 0, hasExtraSrcRegAllocReq = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2272-2281
```tablegen
//   VST2LN   : Vector Store (single 2-element structure from one lane)
class VST2LN<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b00, op11_8, op7_4, (outs),
          (ins addrmode6:$Rn, DPR:$Vd, DPR:$src2, nohash_imm:$lane),
          IIC_VST2ln, "vst2", Dt, "\\{$Vd[$lane], $src2[$lane]\\}, $Rn",
          "", []>, Sched<[WriteVST1]> {
  let Rm = 0b1111;
  let Inst{4}   = Rn{4};
  let DecoderMethod = "DecodeVST2LN";
}
```
- EN: Declares reusable TableGen class `VST2LN` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST2LN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2283-2291
```tablegen
def VST2LNd8  : VST2LN<0b0001, {?,?,?,?}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VST2LNd16 : VST2LN<0b0101, {?,?,0,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VST2LNd32 : VST2LN<0b1001, {?,0,0,?}, "32"> {
  let Inst{7}   = lane{0};
}
```
- EN: Defines TableGen record `VST2LNd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2293-2295
```tablegen
def VST2LNd8Pseudo  : VSTQLNPseudo<IIC_VST2ln>, Sched<[WriteVST1]>;
def VST2LNd16Pseudo : VSTQLNPseudo<IIC_VST2ln>, Sched<[WriteVST1]>;
def VST2LNd32Pseudo : VSTQLNPseudo<IIC_VST2ln>, Sched<[WriteVST1]>;
```
- EN: Defines TableGen record `VST2LNd8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNd8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2297-2305
```tablegen
// ...with double-spaced registers:
def VST2LNq16 : VST2LN<0b0101, {?,?,1,?}, "16"> {
  let Inst{7-6} = lane{1-0};
  let Inst{4}   = Rn{4};
}
def VST2LNq32 : VST2LN<0b1001, {?,1,0,?}, "32"> {
  let Inst{7}   = lane{0};
  let Inst{4}   = Rn{4};
}
```
- EN: Defines TableGen record `VST2LNq16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNq16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2307-2308
```tablegen
def VST2LNq16Pseudo : VSTQQLNPseudo<IIC_VST2ln>, Sched<[WriteVST1]>;
def VST2LNq32Pseudo : VSTQQLNPseudo<IIC_VST2ln>, Sched<[WriteVST1]>;
```
- EN: Defines TableGen record `VST2LNq16Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNq16Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2310-2319
```tablegen
// ...with address register writeback:
class VST2LNWB<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b00, op11_8, op7_4, (outs GPR:$wb),
          (ins addrmode6:$Rn, am6offset:$Rm,
           DPR:$Vd, DPR:$src2, nohash_imm:$lane), IIC_VST2lnu, "vst2", Dt,
          "\\{$Vd[$lane], $src2[$lane]\\}, $Rn$Rm",
          "$Rn.addr = $wb", []> {
  let Inst{4}   = Rn{4};
  let DecoderMethod = "DecodeVST2LN";
}
```
- EN: Declares reusable TableGen class `VST2LNWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST2LNWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2321-2329
```tablegen
def VST2LNd8_UPD  : VST2LNWB<0b0001, {?,?,?,?}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VST2LNd16_UPD : VST2LNWB<0b0101, {?,?,0,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VST2LNd32_UPD : VST2LNWB<0b1001, {?,0,0,?}, "32"> {
  let Inst{7}   = lane{0};
}
```
- EN: Defines TableGen record `VST2LNd8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNd8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2331-2333
```tablegen
def VST2LNd8Pseudo_UPD  : VSTQLNWBPseudo<IIC_VST2lnu>, Sched<[WriteVST1]>;
def VST2LNd16Pseudo_UPD : VSTQLNWBPseudo<IIC_VST2lnu>, Sched<[WriteVST1]>;
def VST2LNd32Pseudo_UPD : VSTQLNWBPseudo<IIC_VST2lnu>, Sched<[WriteVST1]>;
```
- EN: Defines TableGen record `VST2LNd8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNd8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2335-2340
```tablegen
def VST2LNq16_UPD : VST2LNWB<0b0101, {?,?,1,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VST2LNq32_UPD : VST2LNWB<0b1001, {?,1,0,?}, "32"> {
  let Inst{7}   = lane{0};
}
```
- EN: Defines TableGen record `VST2LNq16_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNq16_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2342-2343
```tablegen
def VST2LNq16Pseudo_UPD : VSTQQLNWBPseudo<IIC_VST2lnu>, Sched<[WriteVST1]>;
def VST2LNq32Pseudo_UPD : VSTQQLNWBPseudo<IIC_VST2lnu>, Sched<[WriteVST1]>;
```
- EN: Defines TableGen record `VST2LNq16Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNq16Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2345-2354
```tablegen
//   VST3LN   : Vector Store (single 3-element structure from one lane)
class VST3LN<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b00, op11_8, op7_4, (outs),
          (ins addrmode6:$Rn, DPR:$Vd, DPR:$src2, DPR:$src3,
           nohash_imm:$lane), IIC_VST3ln, "vst3", Dt,
          "\\{$Vd[$lane], $src2[$lane], $src3[$lane]\\}, $Rn", "", []>,
    Sched<[WriteVST2]> {
  let Rm = 0b1111;
  let DecoderMethod = "DecodeVST3LN";
}
```
- EN: Declares reusable TableGen class `VST3LN` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST3LN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2356-2364
```tablegen
def VST3LNd8  : VST3LN<0b0010, {?,?,?,0}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VST3LNd16 : VST3LN<0b0110, {?,?,0,0}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VST3LNd32 : VST3LN<0b1010, {?,0,0,0}, "32"> {
  let Inst{7}   = lane{0};
}
```
- EN: Defines TableGen record `VST3LNd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2366-2368
```tablegen
def VST3LNd8Pseudo  : VSTQQLNPseudo<IIC_VST3ln>, Sched<[WriteVST2]>;
def VST3LNd16Pseudo : VSTQQLNPseudo<IIC_VST3ln>, Sched<[WriteVST2]>;
def VST3LNd32Pseudo : VSTQQLNPseudo<IIC_VST3ln>, Sched<[WriteVST2]>;
```
- EN: Defines TableGen record `VST3LNd8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNd8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2370-2376
```tablegen
// ...with double-spaced registers:
def VST3LNq16 : VST3LN<0b0110, {?,?,1,0}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VST3LNq32 : VST3LN<0b1010, {?,1,0,0}, "32"> {
  let Inst{7}   = lane{0};
}
```
- EN: Defines TableGen record `VST3LNq16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNq16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2378-2379
```tablegen
def VST3LNq16Pseudo : VSTQQQQLNPseudo<IIC_VST3ln>;
def VST3LNq32Pseudo : VSTQQQQLNPseudo<IIC_VST3ln>;
```
- EN: Defines TableGen record `VST3LNq16Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNq16Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2381-2390
```tablegen
// ...with address register writeback:
class VST3LNWB<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b00, op11_8, op7_4, (outs GPR:$wb),
          (ins addrmode6:$Rn, am6offset:$Rm,
           DPR:$Vd, DPR:$src2, DPR:$src3, nohash_imm:$lane),
          IIC_VST3lnu, "vst3", Dt,
          "\\{$Vd[$lane], $src2[$lane], $src3[$lane]\\}, $Rn$Rm",
          "$Rn.addr = $wb", []> {
  let DecoderMethod = "DecodeVST3LN";
}
```
- EN: Declares reusable TableGen class `VST3LNWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST3LNWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2392-2400
```tablegen
def VST3LNd8_UPD  : VST3LNWB<0b0010, {?,?,?,0}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VST3LNd16_UPD : VST3LNWB<0b0110, {?,?,0,0}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VST3LNd32_UPD : VST3LNWB<0b1010, {?,0,0,0}, "32"> {
  let Inst{7}   = lane{0};
}
```
- EN: Defines TableGen record `VST3LNd8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNd8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2402-2404
```tablegen
def VST3LNd8Pseudo_UPD  : VSTQQLNWBPseudo<IIC_VST3lnu>, Sched<[WriteVST2]>;
def VST3LNd16Pseudo_UPD : VSTQQLNWBPseudo<IIC_VST3lnu>, Sched<[WriteVST2]>;
def VST3LNd32Pseudo_UPD : VSTQQLNWBPseudo<IIC_VST3lnu>, Sched<[WriteVST2]>;
```
- EN: Defines TableGen record `VST3LNd8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNd8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2406-2411
```tablegen
def VST3LNq16_UPD : VST3LNWB<0b0110, {?,?,1,0}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VST3LNq32_UPD : VST3LNWB<0b1010, {?,1,0,0}, "32"> {
  let Inst{7}   = lane{0};
}
```
- EN: Defines TableGen record `VST3LNq16_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNq16_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2413-2414
```tablegen
def VST3LNq16Pseudo_UPD : VSTQQQQLNWBPseudo<IIC_VST3lnu>, Sched<[WriteVST2]>;
def VST3LNq32Pseudo_UPD : VSTQQQQLNWBPseudo<IIC_VST3lnu>, Sched<[WriteVST2]>;
```
- EN: Defines TableGen record `VST3LNq16Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNq16Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2416-2426
```tablegen
//   VST4LN   : Vector Store (single 4-element structure from one lane)
class VST4LN<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b00, op11_8, op7_4, (outs),
          (ins addrmode6:$Rn, DPR:$Vd, DPR:$src2, DPR:$src3, DPR:$src4,
           nohash_imm:$lane), IIC_VST4ln, "vst4", Dt,
          "\\{$Vd[$lane], $src2[$lane], $src3[$lane], $src4[$lane]\\}, $Rn",
          "", []>, Sched<[WriteVST2]> {
  let Rm = 0b1111;
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVST4LN";
}
```
- EN: Declares reusable TableGen class `VST4LN` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST4LN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2428-2437
```tablegen
def VST4LNd8  : VST4LN<0b0011, {?,?,?,?}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VST4LNd16 : VST4LN<0b0111, {?,?,0,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VST4LNd32 : VST4LN<0b1011, {?,0,?,?}, "32"> {
  let Inst{7}   = lane{0};
  let Inst{5} = Rn{5};
}
```
- EN: Defines TableGen record `VST4LNd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2439-2441
```tablegen
def VST4LNd8Pseudo  : VSTQQLNPseudo<IIC_VST4ln>, Sched<[WriteVST2]>;
def VST4LNd16Pseudo : VSTQQLNPseudo<IIC_VST4ln>, Sched<[WriteVST2]>;
def VST4LNd32Pseudo : VSTQQLNPseudo<IIC_VST4ln>, Sched<[WriteVST2]>;
```
- EN: Defines TableGen record `VST4LNd8Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNd8Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2443-2450
```tablegen
// ...with double-spaced registers:
def VST4LNq16 : VST4LN<0b0111, {?,?,1,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VST4LNq32 : VST4LN<0b1011, {?,1,?,?}, "32"> {
  let Inst{7}   = lane{0};
  let Inst{5} = Rn{5};
}
```
- EN: Defines TableGen record `VST4LNq16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNq16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2452-2453
```tablegen
def VST4LNq16Pseudo : VSTQQQQLNPseudo<IIC_VST4ln>, Sched<[WriteVST2]>;
def VST4LNq32Pseudo : VSTQQQQLNPseudo<IIC_VST4ln>, Sched<[WriteVST2]>;
```
- EN: Defines TableGen record `VST4LNq16Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNq16Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2455-2465
```tablegen
// ...with address register writeback:
class VST4LNWB<bits<4> op11_8, bits<4> op7_4, string Dt>
  : NLdStLn<1, 0b00, op11_8, op7_4, (outs GPR:$wb),
          (ins addrmode6:$Rn, am6offset:$Rm,
           DPR:$Vd, DPR:$src2, DPR:$src3, DPR:$src4, nohash_imm:$lane),
          IIC_VST4lnu, "vst4", Dt,
  "\\{$Vd[$lane], $src2[$lane], $src3[$lane], $src4[$lane]\\}, $Rn$Rm",
          "$Rn.addr = $wb", []> {
  let Inst{4} = Rn{4};
  let DecoderMethod = "DecodeVST4LN";
}
```
- EN: Declares reusable TableGen class `VST4LNWB` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VST4LNWB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2467-2476
```tablegen
def VST4LNd8_UPD  : VST4LNWB<0b0011, {?,?,?,?}, "8"> {
  let Inst{7-5} = lane{2-0};
}
def VST4LNd16_UPD : VST4LNWB<0b0111, {?,?,0,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VST4LNd32_UPD : VST4LNWB<0b1011, {?,0,?,?}, "32"> {
  let Inst{7}   = lane{0};
  let Inst{5} = Rn{5};
}
```
- EN: Defines TableGen record `VST4LNd8_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNd8_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2478-2480
```tablegen
def VST4LNd8Pseudo_UPD  : VSTQQLNWBPseudo<IIC_VST4lnu>, Sched<[WriteVST2]>;
def VST4LNd16Pseudo_UPD : VSTQQLNWBPseudo<IIC_VST4lnu>, Sched<[WriteVST2]>;
def VST4LNd32Pseudo_UPD : VSTQQLNWBPseudo<IIC_VST4lnu>, Sched<[WriteVST2]>;
```
- EN: Defines TableGen record `VST4LNd8Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNd8Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2482-2488
```tablegen
def VST4LNq16_UPD : VST4LNWB<0b0111, {?,?,1,?}, "16"> {
  let Inst{7-6} = lane{1-0};
}
def VST4LNq32_UPD : VST4LNWB<0b1011, {?,1,?,?}, "32"> {
  let Inst{7}   = lane{0};
  let Inst{5} = Rn{5};
}
```
- EN: Defines TableGen record `VST4LNq16_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNq16_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2490-2491
```tablegen
def VST4LNq16Pseudo_UPD : VSTQQQQLNWBPseudo<IIC_VST4lnu>, Sched<[WriteVST2]>;
def VST4LNq32Pseudo_UPD : VSTQQQQLNWBPseudo<IIC_VST4lnu>, Sched<[WriteVST2]>;
```
- EN: Defines TableGen record `VST4LNq16Pseudo_UPD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNq16Pseudo_UPD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2493-2493
```tablegen
} // mayStore = 1, hasSideEffects = 0, hasExtraSrcRegAllocReq = 1
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2495-2511
```tablegen
// Use vld1/vst1 for unaligned f64 load / store
let Predicates = [IsLE,HasNEON] in {
def : Pat<(f64 (hword_alignedload addrmode6:$addr)),
          (VLD1d16 addrmode6:$addr)>;
def : Pat<(hword_alignedstore (f64 DPR:$value), addrmode6:$addr),
          (VST1d16 addrmode6:$addr, DPR:$value)>;
def : Pat<(f64 (byte_alignedload addrmode6:$addr)),
          (VLD1d8 addrmode6:$addr)>;
def : Pat<(byte_alignedstore (f64 DPR:$value), addrmode6:$addr),
          (VST1d8 addrmode6:$addr, DPR:$value)>;
}
let Predicates = [IsBE,HasNEON] in {
def : Pat<(f64 (non_word_alignedload addrmode6:$addr)),
          (VLD1d64 addrmode6:$addr)>;
def : Pat<(non_word_alignedstore (f64 DPR:$value), addrmode6:$addr),
          (VST1d64 addrmode6:$addr, DPR:$value)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2513-2530
```tablegen
// Use vld1/vst1 for Q and QQ. Also use them for unaligned v2f64
// load / store if it's legal.
let Predicates = [HasNEON] in {
def : Pat<(v2f64 (dword_alignedload addrmode6:$addr)),
          (VLD1q64 addrmode6:$addr)>;
def : Pat<(dword_alignedstore (v2f64 QPR:$value), addrmode6:$addr),
          (VST1q64 addrmode6:$addr, QPR:$value)>;
}
let Predicates = [IsLE,HasNEON] in {
def : Pat<(v2f64 (word_alignedload addrmode6:$addr)),
          (VLD1q32 addrmode6:$addr)>;
def : Pat<(word_alignedstore (v2f64 QPR:$value), addrmode6:$addr),
          (VST1q32 addrmode6:$addr, QPR:$value)>;
def : Pat<(v2f64 (hword_alignedload addrmode6:$addr)),
          (VLD1q16 addrmode6:$addr)>;
def : Pat<(hword_alignedstore (v2f64 QPR:$value), addrmode6:$addr),
          (VST1q16 addrmode6:$addr, QPR:$value)>;
def : Pat<(v2f64 (byte_alignedload addrmode6:$addr)),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2531-2534
```tablegen
          (VLD1q8 addrmode6:$addr)>;
def : Pat<(byte_alignedstore (v2f64 QPR:$value), addrmode6:$addr),
          (VST1q8 addrmode6:$addr, QPR:$value)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2536-2538
```tablegen
//===----------------------------------------------------------------------===//
// Instruction Classes
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2540-2552
```tablegen
// Basic 2-register operations: double- and quad-register.
class N2VD<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
           bits<2> op17_16, bits<5> op11_7, bit op4, string OpcodeStr,
           string Dt, ValueType ResTy, ValueType OpTy, SDNode OpNode>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, 0, op4, (outs DPR:$Vd),
        (ins DPR:$Vm), IIC_VUNAD, OpcodeStr, Dt,"$Vd, $Vm", "",
        [(set DPR:$Vd, (ResTy (OpNode (OpTy DPR:$Vm))))]>;
class N2VQ<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
           bits<2> op17_16, bits<5> op11_7, bit op4, string OpcodeStr,
           string Dt, ValueType ResTy, ValueType OpTy, SDNode OpNode>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, 1, op4, (outs QPR:$Vd),
        (ins QPR:$Vm), IIC_VUNAQ, OpcodeStr, Dt,"$Vd, $Vm", "",
        [(set QPR:$Vd, (ResTy (OpNode (OpTy QPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `N2VD` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VD`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2554-2568
```tablegen
// Basic 2-register intrinsics, both double- and quad-register.
class N2VDInt<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
              bits<2> op17_16, bits<5> op11_7, bit op4,
              InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, 0, op4, (outs DPR:$Vd),
        (ins DPR:$Vm), itin, OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set DPR:$Vd, (ResTy (IntOp (OpTy DPR:$Vm))))]>;
class N2VQInt<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
              bits<2> op17_16, bits<5> op11_7, bit op4,
              InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, 1, op4, (outs QPR:$Vd),
        (ins QPR:$Vm), itin, OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set QPR:$Vd, (ResTy (IntOp (OpTy QPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `N2VDInt` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VDInt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2570-2576
```tablegen
// Same as above, but not predicated.
class N2VDIntnp<bits<2> op19_18, bits<2> op17_16, bits<3> op10_8, bit op7,
              InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N2Vnp<op19_18, op17_16, op10_8, op7, 0,  (outs DPR:$Vd), (ins DPR:$Vm),
          itin, OpcodeStr, Dt,
          [(set DPR:$Vd, (ResTy (IntOp (OpTy DPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `N2VDIntnp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VDIntnp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2578-2583
```tablegen
class N2VQIntnp<bits<2> op19_18, bits<2> op17_16, bits<3> op10_8, bit op7,
              InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N2Vnp<op19_18, op17_16, op10_8, op7, 1,  (outs QPR:$Vd), (ins QPR:$Vm),
          itin, OpcodeStr, Dt,
          [(set QPR:$Vd, (ResTy (IntOp (OpTy QPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `N2VQIntnp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VQIntnp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2585-2591
```tablegen
// Similar to NV2VQIntnp with some more encoding bits exposed (crypto).
class N2VQIntXnp<bits<2> op19_18, bits<2> op17_16, bits<3> op10_8, bit op6,
              bit op7, InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N2Vnp<op19_18, op17_16, op10_8, op7, op6,  (outs QPR:$Vd), (ins QPR:$Vm),
          itin, OpcodeStr, Dt,
          [(set QPR:$Vd, (ResTy (IntOp (OpTy QPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `N2VQIntXnp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VQIntXnp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2593-2602
```tablegen
// Same as N2VQIntXnp but with Vd as a src register.
class N2VQIntX2np<bits<2> op19_18, bits<2> op17_16, bits<3> op10_8, bit op6,
              bit op7, InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N2Vnp<op19_18, op17_16, op10_8, op7, op6,
          (outs QPR:$Vd), (ins QPR:$src, QPR:$Vm),
          itin, OpcodeStr, Dt,
          [(set QPR:$Vd, (ResTy (IntOp (OpTy QPR:$src), (OpTy QPR:$Vm))))]> {
  let Constraints = "$src = $Vd";
}
```
- EN: Declares reusable TableGen class `N2VQIntX2np` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VQIntX2np`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2604-2611
```tablegen
// Narrow 2-register operations.
class N2VN<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
           bits<2> op17_16, bits<5> op11_7, bit op6, bit op4,
           InstrItinClass itin, string OpcodeStr, string Dt,
           ValueType TyD, ValueType TyQ, SDNode OpNode>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, op6, op4, (outs DPR:$Vd),
        (ins QPR:$Vm), itin, OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set DPR:$Vd, (TyD (OpNode (TyQ QPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `N2VN` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VN`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2613-2620
```tablegen
// Narrow 2-register intrinsics.
class N2VNInt<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
              bits<2> op17_16, bits<5> op11_7, bit op6, bit op4,
              InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType TyD, ValueType TyQ, SDPatternOperator IntOp>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, op6, op4, (outs DPR:$Vd),
        (ins QPR:$Vm), itin, OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set DPR:$Vd, (TyD (IntOp (TyQ QPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `N2VNInt` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VNInt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2622-2629
```tablegen
// Long 2-register operations (currently only used for VMOVL).
class N2VL<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
           bits<2> op17_16, bits<5> op11_7, bit op6, bit op4,
           InstrItinClass itin, string OpcodeStr, string Dt,
           ValueType TyQ, ValueType TyD, SDNode OpNode>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, op6, op4, (outs QPR:$Vd),
        (ins DPR:$Vm), itin, OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set QPR:$Vd, (TyQ (OpNode (TyD DPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `N2VL` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2631-2638
```tablegen
// Long 2-register intrinsics.
class N2VLInt<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
              bits<2> op17_16, bits<5> op11_7, bit op6, bit op4,
              InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType TyQ, ValueType TyD, SDPatternOperator IntOp>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, op6, op4, (outs QPR:$Vd),
        (ins DPR:$Vm), itin, OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set QPR:$Vd, (TyQ (IntOp (TyD DPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `N2VLInt` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VLInt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2640-2652
```tablegen
// 2-register shuffles (VTRN/VZIP/VUZP), both double- and quad-register.
let hasSideEffects = 0 in {
class N2VDShuffle<bits<2> op19_18, bits<5> op11_7, string OpcodeStr, string Dt>
  : N2V<0b11, 0b11, op19_18, 0b10, op11_7, 0, 0, (outs DPR:$Vd, DPR:$Vm),
        (ins DPR:$src1, DPR:$src2), IIC_VPERMD,
        OpcodeStr, Dt, "$Vd, $Vm",
        "$src1 = $Vd, $src2 = $Vm", []>;
class N2VQShuffle<bits<2> op19_18, bits<5> op11_7,
                  InstrItinClass itin, string OpcodeStr, string Dt>
  : N2V<0b11, 0b11, op19_18, 0b10, op11_7, 1, 0, (outs QPR:$Vd, QPR:$Vm),
        (ins QPR:$src1, QPR:$src2), itin, OpcodeStr, Dt, "$Vd, $Vm",
        "$src1 = $Vd, $src2 = $Vm", []>;
}
```
- EN: Declares reusable TableGen class `N2VDShuffle` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VDShuffle`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2654-2671
```tablegen
// Basic 3-register operations: double- and quad-register.
class N3VD<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
           InstrItinClass itin, string OpcodeStr, string Dt,
           ValueType ResTy, ValueType OpTy, SDNode OpNode, bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs DPR:$Vd), (ins DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set DPR:$Vd, (ResTy (OpNode (OpTy DPR:$Vn), (OpTy DPR:$Vm))))]> {
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = Commutable;
}
// Same as N3VD but no data type.
class N3VDX<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
           InstrItinClass itin, string OpcodeStr,
           ValueType ResTy, ValueType OpTy,
           SDNode OpNode, bit Commutable>
  : N3VX<op24, op23, op21_20, op11_8, 0, op4,
```
- EN: Declares reusable TableGen class `N3VD` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VD`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2672-2678
```tablegen
         (outs DPR:$Vd), (ins DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
         OpcodeStr, "$Vd, $Vn, $Vm", "",
         [(set DPR:$Vd, (ResTy (OpNode (OpTy DPR:$Vn), (OpTy DPR:$Vm))))]>{
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = Commutable;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2680-2697
```tablegen
class N3VDSL<bits<2> op21_20, bits<4> op11_8,
             InstrItinClass itin, string OpcodeStr, string Dt,
             ValueType Ty, SDNode ShOp>
  : N3VLane32<0, 1, op21_20, op11_8, 1, 0,
        (outs DPR:$Vd), (ins DPR:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane),
        NVMulSLFrm, itin, OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "",
        [(set (Ty DPR:$Vd),
              (Ty (ShOp (Ty DPR:$Vn),
                        (Ty (ARMvduplane (Ty DPR_VFP2:$Vm),imm:$lane)))))]> {
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = 0;
}
class N3VDSL16<bits<2> op21_20, bits<4> op11_8,
               string OpcodeStr, string Dt, ValueType Ty, SDNode ShOp>
  : N3VLane16<0, 1, op21_20, op11_8, 1, 0,
        (outs DPR:$Vd), (ins DPR:$Vn, DPR_8:$Vm, VectorIndex16:$lane),
        NVMulSLFrm, IIC_VMULi16D, OpcodeStr, Dt,"$Vd, $Vn, $Vm$lane","",
```
- EN: Declares reusable TableGen class `N3VDSL` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VDSL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2698-2704
```tablegen
        [(set (Ty DPR:$Vd),
              (Ty (ShOp (Ty DPR:$Vn),
                        (Ty (ARMvduplane (Ty DPR_8:$Vm), imm:$lane)))))]> {
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2706-2723
```tablegen
class N3VQ<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
           InstrItinClass itin, string OpcodeStr, string Dt,
           ValueType ResTy, ValueType OpTy, SDNode OpNode, bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 1, op4,
        (outs QPR:$Vd), (ins QPR:$Vn, QPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set QPR:$Vd, (ResTy (OpNode (OpTy QPR:$Vn), (OpTy QPR:$Vm))))]> {
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = Commutable;
}
class N3VQX<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
           InstrItinClass itin, string OpcodeStr,
           ValueType ResTy, ValueType OpTy, SDNode OpNode, bit Commutable>
  : N3VX<op24, op23, op21_20, op11_8, 1, op4,
         (outs QPR:$Vd), (ins QPR:$Vn, QPR:$Vm), N3RegFrm, itin,
         OpcodeStr, "$Vd, $Vn, $Vm", "",
         [(set QPR:$Vd, (ResTy (OpNode (OpTy QPR:$Vn), (OpTy QPR:$Vm))))]>{
```
- EN: Declares reusable TableGen class `N3VQ` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VQ`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2724-2741
```tablegen
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = Commutable;
}
class N3VQSL<bits<2> op21_20, bits<4> op11_8,
             InstrItinClass itin, string OpcodeStr, string Dt,
             ValueType ResTy, ValueType OpTy, SDNode ShOp>
  : N3VLane32<1, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd), (ins QPR:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane),
        NVMulSLFrm, itin, OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "",
        [(set (ResTy QPR:$Vd),
              (ResTy (ShOp (ResTy QPR:$Vn),
                           (ResTy (ARMvduplane (OpTy DPR_VFP2:$Vm),
                                                imm:$lane)))))]> {
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = 0;
}
```
- EN: Declares reusable TableGen class `N3VQSL` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VQSL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2742-2754
```tablegen
class N3VQSL16<bits<2> op21_20, bits<4> op11_8, string OpcodeStr, string Dt,
               ValueType ResTy, ValueType OpTy, SDNode ShOp>
  : N3VLane16<1, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd), (ins QPR:$Vn, DPR_8:$Vm, VectorIndex16:$lane),
        NVMulSLFrm, IIC_VMULi16Q, OpcodeStr, Dt,"$Vd, $Vn, $Vm$lane", "",
        [(set (ResTy QPR:$Vd),
              (ResTy (ShOp (ResTy QPR:$Vn),
                           (ResTy (ARMvduplane (OpTy DPR_8:$Vm),
                                                imm:$lane)))))]> {
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = 0;
}
```
- EN: Declares reusable TableGen class `N3VQSL16` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VQSL16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2756-2767
```tablegen
// Basic 3-register intrinsics, both double- and quad-register.
class N3VDInt<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
              Format f, InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp, bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs DPR:$Vd), (ins DPR:$Vn, DPR:$Vm), f, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set DPR:$Vd, (ResTy (IntOp (OpTy DPR:$Vn), (OpTy DPR:$Vm))))]> {
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VDInt` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VDInt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2769-2777
```tablegen
class N3VDIntnp<bits<5> op27_23, bits<2> op21_20, bits<4> op11_8, bit op6,
                bit op4, Format f, InstrItinClass itin, string OpcodeStr,
                string Dt, ValueType ResTy, ValueType OpTy,
                SDPatternOperator IntOp, bit Commutable>
  : N3Vnp<op27_23, op21_20, op11_8, op6, op4,
          (outs DPR:$Vd), (ins DPR:$Vn, DPR:$Vm), f, itin, OpcodeStr, Dt,
          [(set DPR:$Vd, (ResTy (IntOp (OpTy DPR:$Vn), (OpTy DPR:$Vm))))]> {
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VDIntnp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VDIntnp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2780-2790
```tablegen
class N3VDIntSL<bits<2> op21_20, bits<4> op11_8, InstrItinClass itin,
                string OpcodeStr, string Dt, ValueType Ty, SDPatternOperator IntOp>
  : N3VLane32<0, 1, op21_20, op11_8, 1, 0,
        (outs DPR:$Vd), (ins DPR:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane),
        NVMulSLFrm, itin, OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "",
        [(set (Ty DPR:$Vd),
              (Ty (IntOp (Ty DPR:$Vn),
                         (Ty (ARMvduplane (Ty DPR_VFP2:$Vm),
                                           imm:$lane)))))]> {
  let isCommutable = 0;
}
```
- EN: Declares reusable TableGen class `N3VDIntSL` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VDIntSL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2792-2809
```tablegen
class N3VDIntSL16<bits<2> op21_20, bits<4> op11_8, InstrItinClass itin,
                  string OpcodeStr, string Dt, ValueType Ty, SDPatternOperator IntOp>
  : N3VLane16<0, 1, op21_20, op11_8, 1, 0,
        (outs DPR:$Vd), (ins DPR:$Vn, DPR_8:$Vm, VectorIndex16:$lane),
        NVMulSLFrm, itin, OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "",
        [(set (Ty DPR:$Vd),
              (Ty (IntOp (Ty DPR:$Vn),
                         (Ty (ARMvduplane (Ty DPR_8:$Vm), imm:$lane)))))]> {
  let isCommutable = 0;
}
class N3VDIntSh<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
              Format f, InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs DPR:$Vd), (ins DPR:$Vm, DPR:$Vn), f, itin,
        OpcodeStr, Dt, "$Vd, $Vm, $Vn", "",
        [(set DPR:$Vd, (ResTy (IntOp (OpTy DPR:$Vm), (OpTy DPR:$Vn))))]> {
  let TwoOperandAliasConstraint = "$Vm = $Vd";
```
- EN: Declares reusable TableGen class `N3VDIntSL16` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VDIntSL16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2810-2811
```tablegen
  let isCommutable = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2813-2823
```tablegen
class N3VQInt<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
              Format f, InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp, bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 1, op4,
        (outs QPR:$Vd), (ins QPR:$Vn, QPR:$Vm), f, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set QPR:$Vd, (ResTy (IntOp (OpTy QPR:$Vn), (OpTy QPR:$Vm))))]> {
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VQInt` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VQInt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2825-2833
```tablegen
class N3VQIntnp<bits<5> op27_23, bits<2> op21_20, bits<4> op11_8, bit op6,
                bit op4, Format f, InstrItinClass itin, string OpcodeStr,
                string Dt, ValueType ResTy, ValueType OpTy,
                SDPatternOperator IntOp, bit Commutable>
  : N3Vnp<op27_23, op21_20, op11_8, op6, op4,
          (outs QPR:$Vd), (ins QPR:$Vn, QPR:$Vm), f, itin, OpcodeStr, Dt,
          [(set QPR:$Vd, (ResTy (IntOp (OpTy QPR:$Vn), (OpTy QPR:$Vm))))]> {
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VQIntnp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VQIntnp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2835-2847
```tablegen
// Same as N3VQIntnp but with Vd as a src register.
class N3VQInt3np<bits<5> op27_23, bits<2> op21_20, bits<4> op11_8, bit op6,
                bit op4, Format f, InstrItinClass itin, string OpcodeStr,
                string Dt, ValueType ResTy, ValueType OpTy,
                SDPatternOperator IntOp>
  : N3Vnp<op27_23, op21_20, op11_8, op6, op4,
          (outs QPR:$Vd), (ins QPR:$src, QPR:$Vn, QPR:$Vm),
          f, itin, OpcodeStr, Dt,
          [(set QPR:$Vd, (ResTy (IntOp (OpTy QPR:$src), (OpTy QPR:$Vn),
                                       (OpTy QPR:$Vm))))]> {
  let Constraints = "$src = $Vd";
  let isCommutable = 0;
}
```
- EN: Declares reusable TableGen class `N3VQInt3np` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VQInt3np`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2849-2866
```tablegen
class N3VQIntSL<bits<2> op21_20, bits<4> op11_8, InstrItinClass itin,
                string OpcodeStr, string Dt,
                ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N3VLane32<1, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd), (ins QPR:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane),
        NVMulSLFrm, itin, OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "",
        [(set (ResTy QPR:$Vd),
              (ResTy (IntOp (ResTy QPR:$Vn),
                            (ResTy (ARMvduplane (OpTy DPR_VFP2:$Vm),
                                                 imm:$lane)))))]> {
  let isCommutable = 0;
}
class N3VQIntSL16<bits<2> op21_20, bits<4> op11_8, InstrItinClass itin,
                  string OpcodeStr, string Dt,
                  ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N3VLane16<1, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd), (ins QPR:$Vn, DPR_8:$Vm, VectorIndex16:$lane),
        NVMulSLFrm, itin, OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "",
```
- EN: Declares reusable TableGen class `N3VQIntSL` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VQIntSL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2867-2882
```tablegen
        [(set (ResTy QPR:$Vd),
              (ResTy (IntOp (ResTy QPR:$Vn),
                            (ResTy (ARMvduplane (OpTy DPR_8:$Vm),
                                                 imm:$lane)))))]> {
  let isCommutable = 0;
}
class N3VQIntSh<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
              Format f, InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N3V<op24, op23, op21_20, op11_8, 1, op4,
        (outs QPR:$Vd), (ins QPR:$Vm, QPR:$Vn), f, itin,
        OpcodeStr, Dt, "$Vd, $Vm, $Vn", "",
        [(set QPR:$Vd, (ResTy (IntOp (OpTy QPR:$Vm), (OpTy QPR:$Vn))))]> {
  let TwoOperandAliasConstraint = "$Vm = $Vd";
  let isCommutable = 0;
}
```
- EN: Declares reusable TableGen class `N3VQIntSh` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VQIntSh`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2884-2892
```tablegen
// Multiply-Add/Sub operations: double- and quad-register.
class N3VDMulOp<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
                InstrItinClass itin, string OpcodeStr, string Dt,
                ValueType Ty, SDPatternOperator MulOp, SDPatternOperator OpNode>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs DPR:$Vd), (ins DPR:$src1, DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "$src1 = $Vd",
        [(set DPR:$Vd, (Ty (OpNode DPR:$src1,
                             (Ty (MulOp DPR:$Vn, DPR:$Vm)))))]>;
```
- EN: Declares reusable TableGen class `N3VDMulOp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VDMulOp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2894-2911
```tablegen
class N3VDMulOpSL<bits<2> op21_20, bits<4> op11_8, InstrItinClass itin,
                  string OpcodeStr, string Dt,
                  ValueType Ty, SDPatternOperator MulOp, SDPatternOperator ShOp>
  : N3VLane32<0, 1, op21_20, op11_8, 1, 0,
        (outs DPR:$Vd),
        (ins DPR:$src1, DPR:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane),
        NVMulSLFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "$src1 = $Vd",
        [(set (Ty DPR:$Vd),
              (Ty (ShOp (Ty DPR:$src1),
                        (Ty (MulOp DPR:$Vn,
                                   (Ty (ARMvduplane (Ty DPR_VFP2:$Vm),
                                                     imm:$lane)))))))]>;
class N3VDMulOpSL16<bits<2> op21_20, bits<4> op11_8, InstrItinClass itin,
                    string OpcodeStr, string Dt,
                    ValueType Ty, SDPatternOperator MulOp, SDPatternOperator ShOp>
  : N3VLane16<0, 1, op21_20, op11_8, 1, 0,
        (outs DPR:$Vd),
```
- EN: Declares reusable TableGen class `N3VDMulOpSL` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VDMulOpSL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2912-2919
```tablegen
        (ins DPR:$src1, DPR:$Vn, DPR_8:$Vm, VectorIndex16:$lane),
        NVMulSLFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "$src1 = $Vd",
        [(set (Ty DPR:$Vd),
              (Ty (ShOp (Ty DPR:$src1),
                        (Ty (MulOp DPR:$Vn,
                                   (Ty (ARMvduplane (Ty DPR_8:$Vm),
                                                     imm:$lane)))))))]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2921-2938
```tablegen
class N3VQMulOp<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
                InstrItinClass itin, string OpcodeStr, string Dt, ValueType Ty,
                SDPatternOperator MulOp, SDPatternOperator OpNode>
  : N3V<op24, op23, op21_20, op11_8, 1, op4,
        (outs QPR:$Vd), (ins QPR:$src1, QPR:$Vn, QPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "$src1 = $Vd",
        [(set QPR:$Vd, (Ty (OpNode QPR:$src1,
                             (Ty (MulOp QPR:$Vn, QPR:$Vm)))))]>;
class N3VQMulOpSL<bits<2> op21_20, bits<4> op11_8, InstrItinClass itin,
                  string OpcodeStr, string Dt, ValueType ResTy, ValueType OpTy,
                  SDPatternOperator MulOp, SDPatternOperator ShOp>
  : N3VLane32<1, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd),
        (ins QPR:$src1, QPR:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane),
        NVMulSLFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "$src1 = $Vd",
        [(set (ResTy QPR:$Vd),
              (ResTy (ShOp (ResTy QPR:$src1),
```
- EN: Declares reusable TableGen class `N3VQMulOp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VQMulOp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2939-2955
```tablegen
                           (ResTy (MulOp QPR:$Vn,
                                   (ResTy (ARMvduplane (OpTy DPR_VFP2:$Vm),
                                                        imm:$lane)))))))]>;
class N3VQMulOpSL16<bits<2> op21_20, bits<4> op11_8, InstrItinClass itin,
                    string OpcodeStr, string Dt,
                    ValueType ResTy, ValueType OpTy,
                    SDPatternOperator MulOp, SDPatternOperator ShOp>
  : N3VLane16<1, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd),
        (ins QPR:$src1, QPR:$Vn, DPR_8:$Vm, VectorIndex16:$lane),
        NVMulSLFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "$src1 = $Vd",
        [(set (ResTy QPR:$Vd),
              (ResTy (ShOp (ResTy QPR:$src1),
                           (ResTy (MulOp QPR:$Vn,
                                   (ResTy (ARMvduplane (OpTy DPR_8:$Vm),
                                                        imm:$lane)))))))]>;
```
- EN: Declares reusable TableGen class `N3VQMulOpSL16` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VQMulOpSL16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2957-2973
```tablegen
// Neon Intrinsic-Op instructions (VABA): double- and quad-register.
class N3VDIntOp<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
                InstrItinClass itin, string OpcodeStr, string Dt,
                ValueType Ty, SDPatternOperator IntOp, SDNode OpNode>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs DPR:$Vd), (ins DPR:$src1, DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "$src1 = $Vd",
        [(set DPR:$Vd, (Ty (OpNode DPR:$src1,
                             (Ty (IntOp (Ty DPR:$Vn), (Ty DPR:$Vm))))))]>;
class N3VQIntOp<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
                InstrItinClass itin, string OpcodeStr, string Dt,
                ValueType Ty, SDPatternOperator IntOp, SDNode OpNode>
  : N3V<op24, op23, op21_20, op11_8, 1, op4,
        (outs QPR:$Vd), (ins QPR:$src1, QPR:$Vn, QPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "$src1 = $Vd",
        [(set QPR:$Vd, (Ty (OpNode QPR:$src1,
                             (Ty (IntOp (Ty QPR:$Vn), (Ty QPR:$Vm))))))]>;
```
- EN: Declares reusable TableGen class `N3VDIntOp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VDIntOp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2975-2992
```tablegen
// Neon 3-argument intrinsics, both double- and quad-register.
// The destination register is also used as the first source operand register.
class N3VDInt3<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
               InstrItinClass itin, string OpcodeStr, string Dt,
               ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs DPR:$Vd), (ins DPR:$src1, DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "$src1 = $Vd",
        [(set DPR:$Vd, (ResTy (IntOp (OpTy DPR:$src1),
                                      (OpTy DPR:$Vn), (OpTy DPR:$Vm))))]>;
class N3VQInt3<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
               InstrItinClass itin, string OpcodeStr, string Dt,
               ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N3V<op24, op23, op21_20, op11_8, 1, op4,
        (outs QPR:$Vd), (ins QPR:$src1, QPR:$Vn, QPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "$src1 = $Vd",
        [(set QPR:$Vd, (ResTy (IntOp (OpTy QPR:$src1),
                                      (OpTy QPR:$Vn), (OpTy QPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `N3VDInt3` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VDInt3`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2994-3011
```tablegen
// Long Multiply-Add/Sub operations.
class N3VLMulOp<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
                InstrItinClass itin, string OpcodeStr, string Dt,
                ValueType TyQ, ValueType TyD, SDNode MulOp, SDNode OpNode>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs QPR:$Vd), (ins QPR:$src1, DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "$src1 = $Vd",
        [(set QPR:$Vd, (OpNode (TyQ QPR:$src1),
                                (TyQ (MulOp (TyD DPR:$Vn),
                                            (TyD DPR:$Vm)))))]>;
class N3VLMulOpSL<bit op24, bits<2> op21_20, bits<4> op11_8,
                  InstrItinClass itin, string OpcodeStr, string Dt,
                  ValueType TyQ, ValueType TyD, SDNode MulOp, SDNode OpNode>
  : N3VLane32<op24, 1, op21_20, op11_8, 1, 0, (outs QPR:$Vd),
        (ins QPR:$src1, DPR:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane),
        NVMulSLFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "$src1 = $Vd",
        [(set QPR:$Vd,
```
- EN: Declares reusable TableGen class `N3VLMulOp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VLMulOp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3012-3027
```tablegen
          (OpNode (TyQ QPR:$src1),
                  (TyQ (MulOp (TyD DPR:$Vn),
                              (TyD (ARMvduplane (TyD DPR_VFP2:$Vm),
                                                 imm:$lane))))))]>;
class N3VLMulOpSL16<bit op24, bits<2> op21_20, bits<4> op11_8,
                    InstrItinClass itin, string OpcodeStr, string Dt,
                    ValueType TyQ, ValueType TyD, SDNode MulOp, SDNode OpNode>
  : N3VLane16<op24, 1, op21_20, op11_8, 1, 0, (outs QPR:$Vd),
        (ins QPR:$src1, DPR:$Vn, DPR_8:$Vm, VectorIndex16:$lane),
        NVMulSLFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "$src1 = $Vd",
        [(set QPR:$Vd,
          (OpNode (TyQ QPR:$src1),
                  (TyQ (MulOp (TyD DPR:$Vn),
                              (TyD (ARMvduplane (TyD DPR_8:$Vm),
                                                 imm:$lane))))))]>;
```
- EN: Declares reusable TableGen class `N3VLMulOpSL16` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VLMulOpSL16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3029-3039
```tablegen
// Long Intrinsic-Op vector operations with explicit extend (VABAL).
class N3VLIntExtOp<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
                   InstrItinClass itin, string OpcodeStr, string Dt,
                   ValueType TyQ, ValueType TyD, SDPatternOperator IntOp, SDNode ExtOp,
                   SDNode OpNode>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs QPR:$Vd), (ins QPR:$src1, DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "$src1 = $Vd",
        [(set QPR:$Vd, (OpNode (TyQ QPR:$src1),
                                (TyQ (ExtOp (TyD (IntOp (TyD DPR:$Vn),
                                                        (TyD DPR:$Vm)))))))]>;
```
- EN: Declares reusable TableGen class `N3VLIntExtOp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VLIntExtOp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3041-3058
```tablegen
// Neon Long 3-argument intrinsic.  The destination register is
// a quad-register and is also used as the first source operand register.
class N3VLInt3<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
               InstrItinClass itin, string OpcodeStr, string Dt,
               ValueType TyQ, ValueType TyD, SDPatternOperator IntOp>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs QPR:$Vd), (ins QPR:$src1, DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "$src1 = $Vd",
        [(set QPR:$Vd,
          (TyQ (IntOp (TyQ QPR:$src1), (TyD DPR:$Vn), (TyD DPR:$Vm))))]>;
class N3VLInt3SL<bit op24, bits<2> op21_20, bits<4> op11_8, InstrItinClass itin,
                 string OpcodeStr, string Dt,
                 ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N3VLane32<op24, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd),
        (ins QPR:$src1, DPR:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane),
        NVMulSLFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "$src1 = $Vd",
```
- EN: Declares reusable TableGen class `N3VLInt3` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VLInt3`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3059-3076
```tablegen
        [(set (ResTy QPR:$Vd),
              (ResTy (IntOp (ResTy QPR:$src1),
                            (OpTy DPR:$Vn),
                            (OpTy (ARMvduplane (OpTy DPR_VFP2:$Vm),
                                                imm:$lane)))))]>;
class N3VLInt3SL16<bit op24, bits<2> op21_20, bits<4> op11_8,
                   InstrItinClass itin, string OpcodeStr, string Dt,
                   ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N3VLane16<op24, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd),
        (ins QPR:$src1, DPR:$Vn, DPR_8:$Vm, VectorIndex16:$lane),
        NVMulSLFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "$src1 = $Vd",
        [(set (ResTy QPR:$Vd),
              (ResTy (IntOp (ResTy QPR:$src1),
                            (OpTy DPR:$Vn),
                            (OpTy (ARMvduplane (OpTy DPR_8:$Vm),
                                                imm:$lane)))))]>;
```
- EN: Declares reusable TableGen class `N3VLInt3SL16` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VLInt3SL16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3078-3087
```tablegen
// Narrowing 3-register intrinsics.
class N3VNInt<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
              string OpcodeStr, string Dt, ValueType TyD, ValueType TyQ,
              SDPatternOperator IntOp, bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs DPR:$Vd), (ins QPR:$Vn, QPR:$Vm), N3RegFrm, IIC_VBINi4D,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set DPR:$Vd, (TyD (IntOp (TyQ QPR:$Vn), (TyQ QPR:$Vm))))]> {
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VNInt` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VNInt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3089-3098
```tablegen
// Long 3-register operations.
class N3VL<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
           InstrItinClass itin, string OpcodeStr, string Dt,
           ValueType TyQ, ValueType TyD, SDNode OpNode, bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs QPR:$Vd), (ins DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set QPR:$Vd, (TyQ (OpNode (TyD DPR:$Vn), (TyD DPR:$Vm))))]> {
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VL` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3100-3117
```tablegen
class N3VLSL<bit op24, bits<2> op21_20, bits<4> op11_8,
             InstrItinClass itin, string OpcodeStr, string Dt,
             ValueType TyQ, ValueType TyD, SDNode OpNode>
  : N3VLane32<op24, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd), (ins DPR:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane),
        NVMulSLFrm, itin, OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "",
        [(set QPR:$Vd,
          (TyQ (OpNode (TyD DPR:$Vn),
                       (TyD (ARMvduplane (TyD DPR_VFP2:$Vm),imm:$lane)))))]>;
class N3VLSL16<bit op24, bits<2> op21_20, bits<4> op11_8,
               InstrItinClass itin, string OpcodeStr, string Dt,
               ValueType TyQ, ValueType TyD, SDNode OpNode>
  : N3VLane16<op24, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd), (ins DPR:$Vn, DPR_8:$Vm, VectorIndex16:$lane),
        NVMulSLFrm, itin, OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "",
        [(set QPR:$Vd,
          (TyQ (OpNode (TyD DPR:$Vn),
                       (TyD (ARMvduplane (TyD DPR_8:$Vm), imm:$lane)))))]>;
```
- EN: Declares reusable TableGen class `N3VLSL` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VLSL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3119-3130
```tablegen
// Long 3-register operations with explicitly extended operands.
class N3VLExt<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
              InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType TyQ, ValueType TyD, SDNode OpNode, SDPatternOperator ExtOp,
              bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs QPR:$Vd), (ins DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set QPR:$Vd, (OpNode (TyQ (ExtOp (TyD DPR:$Vn))),
                                (TyQ (ExtOp (TyD DPR:$Vm)))))]> {
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VLExt` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VLExt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3132-3143
```tablegen
// Long 3-register intrinsics with explicit extend (VABDL).
class N3VLIntExt<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
                 InstrItinClass itin, string OpcodeStr, string Dt,
                 ValueType TyQ, ValueType TyD, SDPatternOperator IntOp, SDNode ExtOp,
                 bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs QPR:$Vd), (ins DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set QPR:$Vd, (TyQ (ExtOp (TyD (IntOp (TyD DPR:$Vn),
                                                (TyD DPR:$Vm))))))]> {
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VLIntExt` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VLIntExt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3145-3154
```tablegen
// Long 3-register intrinsics.
class N3VLInt<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
              InstrItinClass itin, string OpcodeStr, string Dt,
              ValueType TyQ, ValueType TyD, SDPatternOperator IntOp, bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs QPR:$Vd), (ins DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set QPR:$Vd, (TyQ (IntOp (TyD DPR:$Vn), (TyD DPR:$Vm))))]> {
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VLInt` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VLInt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3156-3165
```tablegen
// Same as above, but not predicated.
class N3VLIntnp<bits<5> op27_23, bits<2> op21_20, bits<4> op11_8, bit op6,
                bit op4, InstrItinClass itin, string OpcodeStr,
                string Dt, ValueType ResTy, ValueType OpTy,
                SDPatternOperator IntOp, bit Commutable>
  : N3Vnp<op27_23, op21_20, op11_8, op6, op4,
          (outs QPR:$Vd), (ins DPR:$Vn, DPR:$Vm), N3RegFrm, itin, OpcodeStr, Dt,
          [(set QPR:$Vd, (ResTy (IntOp (OpTy DPR:$Vn), (OpTy DPR:$Vm))))]> {
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VLIntnp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VLIntnp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3168-3185
```tablegen
class N3VLIntSL<bit op24, bits<2> op21_20, bits<4> op11_8, InstrItinClass itin,
                string OpcodeStr, string Dt,
                ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N3VLane32<op24, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd), (ins DPR:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane),
        NVMulSLFrm, itin, OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "",
        [(set (ResTy QPR:$Vd),
              (ResTy (IntOp (OpTy DPR:$Vn),
                            (OpTy (ARMvduplane (OpTy DPR_VFP2:$Vm),
                                                imm:$lane)))))]>;
class N3VLIntSL16<bit op24, bits<2> op21_20, bits<4> op11_8,
                  InstrItinClass itin, string OpcodeStr, string Dt,
                  ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N3VLane16<op24, 1, op21_20, op11_8, 1, 0,
        (outs QPR:$Vd), (ins DPR:$Vn, DPR_8:$Vm, VectorIndex16:$lane),
        NVMulSLFrm, itin, OpcodeStr, Dt, "$Vd, $Vn, $Vm$lane", "",
        [(set (ResTy QPR:$Vd),
              (ResTy (IntOp (OpTy DPR:$Vn),
```
- EN: Declares reusable TableGen class `N3VLIntSL` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VLIntSL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3186-3187
```tablegen
                            (OpTy (ARMvduplane (OpTy DPR_8:$Vm),
                                                imm:$lane)))))]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3189-3201
```tablegen
// Wide 3-register operations.
class N3VW<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
           string OpcodeStr, string Dt, ValueType TyQ, ValueType TyD,
           SDNode OpNode, SDPatternOperator ExtOp, bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs QPR:$Vd), (ins QPR:$Vn, DPR:$Vm), N3RegFrm, IIC_VSUBiD,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set QPR:$Vd, (OpNode (TyQ QPR:$Vn),
                                (TyQ (ExtOp (TyD DPR:$Vm)))))]> {
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VW` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VW`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3203-3217
```tablegen
// Pairwise long 2-register intrinsics, both double- and quad-register.
class N2VDPLInt<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
                bits<2> op17_16, bits<5> op11_7, bit op4,
                string OpcodeStr, string Dt,
                ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, 0, op4, (outs DPR:$Vd),
        (ins DPR:$Vm), IIC_VSHLiD, OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set DPR:$Vd, (ResTy (IntOp (OpTy DPR:$Vm))))]>;
class N2VQPLInt<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
                bits<2> op17_16, bits<5> op11_7, bit op4,
                string OpcodeStr, string Dt,
                ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, 1, op4, (outs QPR:$Vd),
        (ins QPR:$Vm), IIC_VSHLiD, OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set QPR:$Vd, (ResTy (IntOp (OpTy QPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `N2VDPLInt` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VDPLInt`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3219-3236
```tablegen
// Pairwise long 2-register accumulate intrinsics,
// both double- and quad-register.
// The destination register is also used as the first source operand register.
class N2VDPLInt2<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
                 bits<2> op17_16, bits<5> op11_7, bit op4,
                 string OpcodeStr, string Dt,
                 ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, 0, op4,
        (outs DPR:$Vd), (ins DPR:$src1, DPR:$Vm), IIC_VPALiD,
        OpcodeStr, Dt, "$Vd, $Vm", "$src1 = $Vd",
        [(set DPR:$Vd, (ResTy (IntOp (ResTy DPR:$src1), (OpTy DPR:$Vm))))]>;
class N2VQPLInt2<bits<2> op24_23, bits<2> op21_20, bits<2> op19_18,
                 bits<2> op17_16, bits<5> op11_7, bit op4,
                 string OpcodeStr, string Dt,
                 ValueType ResTy, ValueType OpTy, SDPatternOperator IntOp>
  : N2V<op24_23, op21_20, op19_18, op17_16, op11_7, 1, op4,
        (outs QPR:$Vd), (ins QPR:$src1, QPR:$Vm), IIC_VPALiQ,
        OpcodeStr, Dt, "$Vd, $Vm", "$src1 = $Vd",
```
- EN: Declares reusable TableGen class `N2VDPLInt2` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VDPLInt2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3237-3237
```tablegen
        [(set QPR:$Vd, (ResTy (IntOp (ResTy QPR:$src1), (OpTy QPR:$Vm))))]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3239-3256
```tablegen
// Shift by immediate,
// both double- and quad-register.
let TwoOperandAliasConstraint = "$Vm = $Vd" in {
class N2VDSh<bit op24, bit op23, bits<4> op11_8, bit op7, bit op4,
             Format f, InstrItinClass itin, Operand ImmTy,
             string OpcodeStr, string Dt, ValueType Ty, SDNode OpNode>
  : N2VImm<op24, op23, op11_8, op7, 0, op4,
           (outs DPR:$Vd), (ins DPR:$Vm, ImmTy:$SIMM), f, itin,
           OpcodeStr, Dt, "$Vd, $Vm, $SIMM", "",
           [(set DPR:$Vd, (Ty (OpNode (Ty DPR:$Vm), (i32 imm:$SIMM))))]>;
class N2VQSh<bit op24, bit op23, bits<4> op11_8, bit op7, bit op4,
             Format f, InstrItinClass itin, Operand ImmTy,
             string OpcodeStr, string Dt, ValueType Ty, SDNode OpNode>
  : N2VImm<op24, op23, op11_8, op7, 1, op4,
           (outs QPR:$Vd), (ins QPR:$Vm, ImmTy:$SIMM), f, itin,
           OpcodeStr, Dt, "$Vd, $Vm, $SIMM", "",
           [(set QPR:$Vd, (Ty (OpNode (Ty QPR:$Vm), (i32 imm:$SIMM))))]>;
}
```
- EN: Declares reusable TableGen class `N2VDSh` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VDSh`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3258-3266
```tablegen
// Long shift by immediate.
class N2VLSh<bit op24, bit op23, bits<4> op11_8, bit op7, bit op6, bit op4,
             string OpcodeStr, string Dt,
             ValueType ResTy, ValueType OpTy, Operand ImmTy,
             SDPatternOperator OpNode>
  : N2VImm<op24, op23, op11_8, op7, op6, op4,
           (outs QPR:$Vd), (ins DPR:$Vm, ImmTy:$SIMM), N2RegVShLFrm,
           IIC_VSHLiD, OpcodeStr, Dt, "$Vd, $Vm, $SIMM", "",
           [(set QPR:$Vd, (ResTy (OpNode (OpTy DPR:$Vm), ImmTy:$SIMM)))]>;
```
- EN: Declares reusable TableGen class `N2VLSh` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VLSh`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3268-3277
```tablegen
// Narrow shift by immediate.
class N2VNSh<bit op24, bit op23, bits<4> op11_8, bit op7, bit op6, bit op4,
             InstrItinClass itin, string OpcodeStr, string Dt,
             ValueType ResTy, ValueType OpTy, Operand ImmTy,
             SDPatternOperator OpNode>
  : N2VImm<op24, op23, op11_8, op7, op6, op4,
           (outs DPR:$Vd), (ins QPR:$Vm, ImmTy:$SIMM), N2RegVShRFrm, itin,
           OpcodeStr, Dt, "$Vd, $Vm, $SIMM", "",
           [(set DPR:$Vd, (ResTy (OpNode (OpTy QPR:$Vm),
                                          (i32 ImmTy:$SIMM))))]>;
```
- EN: Declares reusable TableGen class `N2VNSh` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VNSh`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3279-3296
```tablegen
// Shift right by immediate and accumulate,
// both double- and quad-register.
let TwoOperandAliasConstraint = "$Vm = $Vd" in {
class N2VDShAdd<bit op24, bit op23, bits<4> op11_8, bit op7, bit op4,
                Operand ImmTy, string OpcodeStr, string Dt,
                ValueType Ty, SDNode ShOp>
  : N2VImm<op24, op23, op11_8, op7, 0, op4, (outs DPR:$Vd),
           (ins DPR:$src1, DPR:$Vm, ImmTy:$SIMM), N2RegVShRFrm, IIC_VPALiD,
           OpcodeStr, Dt, "$Vd, $Vm, $SIMM", "$src1 = $Vd",
           [(set DPR:$Vd, (Ty (add DPR:$src1,
                                (Ty (ShOp DPR:$Vm, (i32 imm:$SIMM))))))]>;
class N2VQShAdd<bit op24, bit op23, bits<4> op11_8, bit op7, bit op4,
                Operand ImmTy, string OpcodeStr, string Dt,
                ValueType Ty, SDNode ShOp>
  : N2VImm<op24, op23, op11_8, op7, 1, op4, (outs QPR:$Vd),
           (ins QPR:$src1, QPR:$Vm, ImmTy:$SIMM), N2RegVShRFrm, IIC_VPALiD,
           OpcodeStr, Dt, "$Vd, $Vm, $SIMM", "$src1 = $Vd",
           [(set QPR:$Vd, (Ty (add QPR:$src1,
```
- EN: Declares reusable TableGen class `N2VDShAdd` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VDShAdd`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3297-3298
```tablegen
                                (Ty (ShOp QPR:$Vm, (i32 imm:$SIMM))))))]>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3300-3317
```tablegen
// Shift by immediate and insert,
// both double- and quad-register.
let TwoOperandAliasConstraint = "$Vm = $Vd" in {
class N2VDShIns<bit op24, bit op23, bits<4> op11_8, bit op7, bit op4,
                Operand ImmTy, Format f, string OpcodeStr, string Dt,
                ValueType Ty,SDNode ShOp>
  : N2VImm<op24, op23, op11_8, op7, 0, op4, (outs DPR:$Vd),
           (ins DPR:$src1, DPR:$Vm, ImmTy:$SIMM), f, IIC_VSHLiD,
           OpcodeStr, Dt, "$Vd, $Vm, $SIMM", "$src1 = $Vd",
           [(set DPR:$Vd, (Ty (ShOp DPR:$src1, DPR:$Vm, (i32 imm:$SIMM))))]>;
class N2VQShIns<bit op24, bit op23, bits<4> op11_8, bit op7, bit op4,
                Operand ImmTy, Format f, string OpcodeStr, string Dt,
                ValueType Ty,SDNode ShOp>
  : N2VImm<op24, op23, op11_8, op7, 1, op4, (outs QPR:$Vd),
           (ins QPR:$src1, QPR:$Vm, ImmTy:$SIMM), f, IIC_VSHLiQ,
           OpcodeStr, Dt, "$Vd, $Vm, $SIMM", "$src1 = $Vd",
           [(set QPR:$Vd, (Ty (ShOp QPR:$src1, QPR:$Vm, (i32 imm:$SIMM))))]>;
}
```
- EN: Declares reusable TableGen class `N2VDShIns` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VDShIns`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3319-3334
```tablegen
// Convert, with fractional bits immediate,
// both double- and quad-register.
class N2VCvtD<bit op24, bit op23, bits<4> op11_8, bit op7, bit op4,
              string OpcodeStr, string Dt, ValueType ResTy, ValueType OpTy,
              SDPatternOperator IntOp>
  : N2VImm<op24, op23, op11_8, op7, 0, op4,
           (outs DPR:$Vd), (ins DPR:$Vm, neon_vcvt_imm32:$SIMM), NVCVTFrm,
           IIC_VUNAD, OpcodeStr, Dt, "$Vd, $Vm, $SIMM", "",
           [(set DPR:$Vd, (ResTy (IntOp (OpTy DPR:$Vm), (i32 imm:$SIMM))))]>;
class N2VCvtQ<bit op24, bit op23, bits<4> op11_8, bit op7, bit op4,
              string OpcodeStr, string Dt, ValueType ResTy, ValueType OpTy,
              SDPatternOperator IntOp>
  : N2VImm<op24, op23, op11_8, op7, 1, op4,
           (outs QPR:$Vd), (ins QPR:$Vm, neon_vcvt_imm32:$SIMM), NVCVTFrm,
           IIC_VUNAQ, OpcodeStr, Dt, "$Vd, $Vm, $SIMM", "",
           [(set QPR:$Vd, (ResTy (IntOp (OpTy QPR:$Vm), (i32 imm:$SIMM))))]>;
```
- EN: Declares reusable TableGen class `N2VCvtD` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VCvtD`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3336-3338
```tablegen
//===----------------------------------------------------------------------===//
// Multiclasses
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3340-3344
```tablegen
// Abbreviations used in multiclass suffixes:
//   Q = quarter int (8 bit) elements
//   H = half int (16 bit) elements
//   S = single int (32 bit) elements
//   D = double int (64 bit) elements
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3346-3346
```tablegen
// Neon 2-register vector operations and intrinsics.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3348-3365
```tablegen
// Neon 2-register comparisons.
//   source operand element sizes of 8, 16 and 32 bits:
multiclass N2V_QHS_cmp<bits<2> op24_23, bits<2> op21_20, bits<2> op17_16,
                       bits<5> op11_7, bit op4, string opc, string Dt,
                       string asm, PatFrag fc> {
  // 64-bit vector types.
  def v8i8  : N2V<op24_23, op21_20, 0b00, op17_16, op11_7, 0, op4,
                  (outs DPR:$Vd), (ins DPR:$Vm), NoItinerary,
                  opc, !strconcat(Dt, "8"), asm, "",
                  [(set DPR:$Vd, (v8i8 (ARMvcmpz (v8i8 DPR:$Vm), fc)))]>;
  def v4i16 : N2V<op24_23, op21_20, 0b01, op17_16, op11_7, 0, op4,
                  (outs DPR:$Vd), (ins DPR:$Vm), NoItinerary,
                  opc, !strconcat(Dt, "16"), asm, "",
                  [(set DPR:$Vd, (v4i16 (ARMvcmpz (v4i16 DPR:$Vm), fc)))]>;
  def v2i32 : N2V<op24_23, op21_20, 0b10, op17_16, op11_7, 0, op4,
                  (outs DPR:$Vd), (ins DPR:$Vm), NoItinerary,
                  opc, !strconcat(Dt, "32"), asm, "",
                  [(set DPR:$Vd, (v2i32 (ARMvcmpz (v2i32 DPR:$Vm), fc)))]>;
```
- EN: Declares TableGen `multiclass N2V_QHS_cmp`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2V_QHS_cmp`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3366-3378
```tablegen
  def v2f32 : N2V<op24_23, op21_20, 0b10, op17_16, op11_7, 0, op4,
                  (outs DPR:$Vd), (ins DPR:$Vm), NoItinerary,
                  opc, "f32", asm, "",
                  [(set DPR:$Vd, (v2i32 (ARMvcmpz (v2f32 DPR:$Vm), fc)))]> {
    let Inst{10} = 1; // overwrite F = 1
  }
  def v4f16 : N2V<op24_23, op21_20, 0b01, op17_16, op11_7, 0, op4,
                  (outs DPR:$Vd), (ins DPR:$Vm), NoItinerary,
                  opc, "f16", asm, "",
                  [(set DPR:$Vd, (v4i16 (ARMvcmpz (v4f16 DPR:$Vm), fc)))]>,
              Requires<[HasNEON,HasFullFP16]> {
    let Inst{10} = 1; // overwrite F = 1
  }
```
- EN: Defines TableGen record `v2f32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v2f32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3380-3397
```tablegen
  // 128-bit vector types.
  def v16i8 : N2V<op24_23, op21_20, 0b00, op17_16, op11_7, 1, op4,
                  (outs QPR:$Vd), (ins QPR:$Vm), NoItinerary,
                  opc, !strconcat(Dt, "8"), asm, "",
                  [(set QPR:$Vd, (v16i8 (ARMvcmpz (v16i8 QPR:$Vm), fc)))]>;
  def v8i16 : N2V<op24_23, op21_20, 0b01, op17_16, op11_7, 1, op4,
                  (outs QPR:$Vd), (ins QPR:$Vm), NoItinerary,
                  opc, !strconcat(Dt, "16"), asm, "",
                  [(set QPR:$Vd, (v8i16 (ARMvcmpz (v8i16 QPR:$Vm), fc)))]>;
  def v4i32 : N2V<op24_23, op21_20, 0b10, op17_16, op11_7, 1, op4,
                  (outs QPR:$Vd), (ins QPR:$Vm), NoItinerary,
                  opc, !strconcat(Dt, "32"), asm, "",
                  [(set QPR:$Vd, (v4i32 (ARMvcmpz (v4i32 QPR:$Vm), fc)))]>;
  def v4f32 : N2V<op24_23, op21_20, 0b10, op17_16, op11_7, 1, op4,
                  (outs QPR:$Vd), (ins QPR:$Vm), NoItinerary,
                  opc, "f32", asm, "",
                  [(set QPR:$Vd, (v4i32 (ARMvcmpz (v4f32 QPR:$Vm), fc)))]> {
    let Inst{10} = 1; // overwrite F = 1
```
- EN: Defines TableGen record `v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3398-3406
```tablegen
  }
  def v8f16 : N2V<op24_23, op21_20, 0b01, op17_16, op11_7, 1, op4,
                  (outs QPR:$Vd), (ins QPR:$Vm), NoItinerary,
                  opc, "f16", asm, "",
                  [(set QPR:$Vd, (v8i16 (ARMvcmpz (v8f16 QPR:$Vm), fc)))]>,
              Requires<[HasNEON,HasFullFP16]> {
    let Inst{10} = 1; // overwrite F = 1
  }
}
```
- EN: Defines TableGen record `v8f16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v8f16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3408-3419
```tablegen
// Neon 3-register comparisons.
class N3VQ_cmp<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
               InstrItinClass itin, string OpcodeStr, string Dt,
               ValueType ResTy, ValueType OpTy, PatFrag fc, bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 1, op4,
        (outs QPR:$Vd), (ins QPR:$Vn, QPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set QPR:$Vd, (ResTy (ARMvcmp (OpTy QPR:$Vn), (OpTy QPR:$Vm), fc)))]> {
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VQ_cmp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VQ_cmp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3421-3431
```tablegen
class N3VD_cmp<bit op24, bit op23, bits<2> op21_20, bits<4> op11_8, bit op4,
               InstrItinClass itin, string OpcodeStr, string Dt,
               ValueType ResTy, ValueType OpTy, PatFrag fc, bit Commutable>
  : N3V<op24, op23, op21_20, op11_8, 0, op4,
        (outs DPR:$Vd), (ins DPR:$Vn, DPR:$Vm), N3RegFrm, itin,
        OpcodeStr, Dt, "$Vd, $Vn, $Vm", "",
        [(set DPR:$Vd, (ResTy (ARMvcmp (OpTy DPR:$Vn), (OpTy DPR:$Vm), fc)))]> {
  // All of these have a two-operand InstAlias.
  let TwoOperandAliasConstraint = "$Vn = $Vd";
  let isCommutable = Commutable;
}
```
- EN: Declares reusable TableGen class `N3VD_cmp` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VD_cmp`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3433-3447
```tablegen
multiclass N3V_QHS_cmp<bit op24, bit op23, bits<4> op11_8, bit op4,
                       InstrItinClass itinD16, InstrItinClass itinD32,
                       InstrItinClass itinQ16, InstrItinClass itinQ32,
                       string OpcodeStr, string Dt,
                       PatFrag fc, bit Commutable = 0> {
  // 64-bit vector types.
  def v8i8  : N3VD_cmp<op24, op23, 0b00, op11_8, op4, itinD16,
                       OpcodeStr, !strconcat(Dt, "8"),
                       v8i8, v8i8, fc, Commutable>;
  def v4i16 : N3VD_cmp<op24, op23, 0b01, op11_8, op4, itinD16,
                       OpcodeStr, !strconcat(Dt, "16"),
                       v4i16, v4i16, fc, Commutable>;
  def v2i32 : N3VD_cmp<op24, op23, 0b10, op11_8, op4, itinD32,
                       OpcodeStr, !strconcat(Dt, "32"),
                       v2i32, v2i32, fc, Commutable>;
```
- EN: Declares TableGen `multiclass N3V_QHS_cmp`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3V_QHS_cmp`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3449-3459
```tablegen
  // 128-bit vector types.
  def v16i8 : N3VQ_cmp<op24, op23, 0b00, op11_8, op4, itinQ16,
                       OpcodeStr, !strconcat(Dt, "8"),
                       v16i8, v16i8, fc, Commutable>;
  def v8i16 : N3VQ_cmp<op24, op23, 0b01, op11_8, op4, itinQ16,
                       OpcodeStr, !strconcat(Dt, "16"),
                       v8i16, v8i16, fc, Commutable>;
  def v4i32 : N3VQ_cmp<op24, op23, 0b10, op11_8, op4, itinQ32,
                       OpcodeStr, !strconcat(Dt, "32"),
                       v4i32, v4i32, fc, Commutable>;
}
```
- EN: Defines TableGen record `v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3462-3474
```tablegen
// Neon 2-register vector intrinsics,
//   element sizes of 8, 16 and 32 bits:
multiclass N2VInt_QHS<bits<2> op24_23, bits<2> op21_20, bits<2> op17_16,
                      bits<5> op11_7, bit op4,
                      InstrItinClass itinD, InstrItinClass itinQ,
                      string OpcodeStr, string Dt, SDPatternOperator IntOp> {
  // 64-bit vector types.
  def v8i8  : N2VDInt<op24_23, op21_20, 0b00, op17_16, op11_7, op4,
                      itinD, OpcodeStr, !strconcat(Dt, "8"), v8i8, v8i8, IntOp>;
  def v4i16 : N2VDInt<op24_23, op21_20, 0b01, op17_16, op11_7, op4,
                      itinD, OpcodeStr, !strconcat(Dt, "16"),v4i16,v4i16,IntOp>;
  def v2i32 : N2VDInt<op24_23, op21_20, 0b10, op17_16, op11_7, op4,
                      itinD, OpcodeStr, !strconcat(Dt, "32"),v2i32,v2i32,IntOp>;
```
- EN: Declares TableGen `multiclass N2VInt_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VInt_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3476-3483
```tablegen
  // 128-bit vector types.
  def v16i8 : N2VQInt<op24_23, op21_20, 0b00, op17_16, op11_7, op4,
                      itinQ, OpcodeStr, !strconcat(Dt, "8"), v16i8,v16i8,IntOp>;
  def v8i16 : N2VQInt<op24_23, op21_20, 0b01, op17_16, op11_7, op4,
                      itinQ, OpcodeStr, !strconcat(Dt, "16"),v8i16,v8i16,IntOp>;
  def v4i32 : N2VQInt<op24_23, op21_20, 0b10, op17_16, op11_7, op4,
                      itinQ, OpcodeStr, !strconcat(Dt, "32"),v4i32,v4i32,IntOp>;
}
```
- EN: Defines TableGen record `v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3486-3501
```tablegen
// Neon Narrowing 2-register vector operations,
//   source operand element sizes of 16, 32 and 64 bits:
multiclass N2VN_HSD<bits<2> op24_23, bits<2> op21_20, bits<2> op17_16,
                    bits<5> op11_7, bit op6, bit op4,
                    InstrItinClass itin, string OpcodeStr, string Dt,
                    SDNode OpNode> {
  def v8i8  : N2VN<op24_23, op21_20, 0b00, op17_16, op11_7, op6, op4,
                   itin, OpcodeStr, !strconcat(Dt, "16"),
                   v8i8, v8i16, OpNode>;
  def v4i16 : N2VN<op24_23, op21_20, 0b01, op17_16, op11_7, op6, op4,
                   itin, OpcodeStr, !strconcat(Dt, "32"),
                   v4i16, v4i32, OpNode>;
  def v2i32 : N2VN<op24_23, op21_20, 0b10, op17_16, op11_7, op6, op4,
                   itin, OpcodeStr, !strconcat(Dt, "64"),
                   v2i32, v2i64, OpNode>;
}
```
- EN: Declares TableGen `multiclass N2VN_HSD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VN_HSD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3503-3518
```tablegen
// Neon Narrowing 2-register vector intrinsics,
//   source operand element sizes of 16, 32 and 64 bits:
multiclass N2VNInt_HSD<bits<2> op24_23, bits<2> op21_20, bits<2> op17_16,
                       bits<5> op11_7, bit op6, bit op4,
                       InstrItinClass itin, string OpcodeStr, string Dt,
                       SDPatternOperator IntOp> {
  def v8i8  : N2VNInt<op24_23, op21_20, 0b00, op17_16, op11_7, op6, op4,
                      itin, OpcodeStr, !strconcat(Dt, "16"),
                      v8i8, v8i16, IntOp>;
  def v4i16 : N2VNInt<op24_23, op21_20, 0b01, op17_16, op11_7, op6, op4,
                      itin, OpcodeStr, !strconcat(Dt, "32"),
                      v4i16, v4i32, IntOp>;
  def v2i32 : N2VNInt<op24_23, op21_20, 0b10, op17_16, op11_7, op6, op4,
                      itin, OpcodeStr, !strconcat(Dt, "64"),
                      v2i32, v2i64, IntOp>;
}
```
- EN: Declares TableGen `multiclass N2VNInt_HSD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VNInt_HSD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3521-3531
```tablegen
// Neon Lengthening 2-register vector intrinsic (currently specific to VMOVL).
//   source operand element sizes of 16, 32 and 64 bits:
multiclass N2VL_QHS<bits<2> op24_23, bits<5> op11_7, bit op6, bit op4,
                    string OpcodeStr, string Dt, SDNode OpNode> {
  def v8i16 : N2VL<op24_23, 0b00, 0b10, 0b00, op11_7, op6, op4, IIC_VQUNAiD,
                   OpcodeStr, !strconcat(Dt, "8"), v8i16, v8i8, OpNode>;
  def v4i32 : N2VL<op24_23, 0b01, 0b00, 0b00, op11_7, op6, op4, IIC_VQUNAiD,
                   OpcodeStr, !strconcat(Dt, "16"), v4i32, v4i16, OpNode>;
  def v2i64 : N2VL<op24_23, 0b10, 0b00, 0b00, op11_7, op6, op4, IIC_VQUNAiD,
                   OpcodeStr, !strconcat(Dt, "32"), v2i64, v2i32, OpNode>;
}
```
- EN: Declares TableGen `multiclass N2VL_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VL_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3534-3534
```tablegen
// Neon 3-register vector operations.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3536-3551
```tablegen
// First with only element sizes of 8, 16 and 32 bits:
multiclass N3V_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                   InstrItinClass itinD16, InstrItinClass itinD32,
                   InstrItinClass itinQ16, InstrItinClass itinQ32,
                   string OpcodeStr, string Dt,
                   SDNode OpNode, bit Commutable = 0> {
  // 64-bit vector types.
  def v8i8  : N3VD<op24, op23, 0b00, op11_8, op4, itinD16,
                   OpcodeStr, !strconcat(Dt, "8"),
                   v8i8, v8i8, OpNode, Commutable>;
  def v4i16 : N3VD<op24, op23, 0b01, op11_8, op4, itinD16,
                   OpcodeStr, !strconcat(Dt, "16"),
                   v4i16, v4i16, OpNode, Commutable>;
  def v2i32 : N3VD<op24, op23, 0b10, op11_8, op4, itinD32,
                   OpcodeStr, !strconcat(Dt, "32"),
                   v2i32, v2i32, OpNode, Commutable>;
```
- EN: Declares TableGen `multiclass N3V_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3V_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3553-3563
```tablegen
  // 128-bit vector types.
  def v16i8 : N3VQ<op24, op23, 0b00, op11_8, op4, itinQ16,
                   OpcodeStr, !strconcat(Dt, "8"),
                   v16i8, v16i8, OpNode, Commutable>;
  def v8i16 : N3VQ<op24, op23, 0b01, op11_8, op4, itinQ16,
                   OpcodeStr, !strconcat(Dt, "16"),
                   v8i16, v8i16, OpNode, Commutable>;
  def v4i32 : N3VQ<op24, op23, 0b10, op11_8, op4, itinQ32,
                   OpcodeStr, !strconcat(Dt, "32"),
                   v4i32, v4i32, OpNode, Commutable>;
}
```
- EN: Defines TableGen record `v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3565-3571
```tablegen
multiclass N3VSL_HS<bits<4> op11_8, string OpcodeStr, SDNode ShOp> {
  def v4i16 : N3VDSL16<0b01, op11_8, OpcodeStr, "i16", v4i16, ShOp>;
  def v2i32 : N3VDSL<0b10, op11_8, IIC_VMULi32D, OpcodeStr, "i32", v2i32, ShOp>;
  def v8i16 : N3VQSL16<0b01, op11_8, OpcodeStr, "i16", v8i16, v4i16, ShOp>;
  def v4i32 : N3VQSL<0b10, op11_8, IIC_VMULi32Q, OpcodeStr, "i32",
                     v4i32, v2i32, ShOp>;
}
```
- EN: Declares TableGen `multiclass N3VSL_HS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VSL_HS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3573-3586
```tablegen
// ....then also with element size 64 bits:
multiclass N3V_QHSD<bit op24, bit op23, bits<4> op11_8, bit op4,
                    InstrItinClass itinD, InstrItinClass itinQ,
                    string OpcodeStr, string Dt,
                    SDNode OpNode, bit Commutable = 0>
  : N3V_QHS<op24, op23, op11_8, op4, itinD, itinD, itinQ, itinQ,
            OpcodeStr, Dt, OpNode, Commutable> {
  def v1i64 : N3VD<op24, op23, 0b11, op11_8, op4, itinD,
                   OpcodeStr, !strconcat(Dt, "64"),
                   v1i64, v1i64, OpNode, Commutable>;
  def v2i64 : N3VQ<op24, op23, 0b11, op11_8, op4, itinQ,
                   OpcodeStr, !strconcat(Dt, "64"),
                   v2i64, v2i64, OpNode, Commutable>;
}
```
- EN: Declares TableGen `multiclass N3V_QHSD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3V_QHSD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3589-3589
```tablegen
// Neon 3-register vector intrinsics.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3591-3603
```tablegen
// First with only element sizes of 16 and 32 bits:
multiclass N3VInt_HS<bit op24, bit op23, bits<4> op11_8, bit op4, Format f,
                     InstrItinClass itinD16, InstrItinClass itinD32,
                     InstrItinClass itinQ16, InstrItinClass itinQ32,
                     string OpcodeStr, string Dt,
                     SDPatternOperator IntOp, bit Commutable = 0> {
  // 64-bit vector types.
  def v4i16 : N3VDInt<op24, op23, 0b01, op11_8, op4, f, itinD16,
                      OpcodeStr, !strconcat(Dt, "16"),
                      v4i16, v4i16, IntOp, Commutable>;
  def v2i32 : N3VDInt<op24, op23, 0b10, op11_8, op4, f, itinD32,
                      OpcodeStr, !strconcat(Dt, "32"),
                      v2i32, v2i32, IntOp, Commutable>;
```
- EN: Declares TableGen `multiclass N3VInt_HS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VInt_HS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3605-3622
```tablegen
  // 128-bit vector types.
  def v8i16 : N3VQInt<op24, op23, 0b01, op11_8, op4, f, itinQ16,
                      OpcodeStr, !strconcat(Dt, "16"),
                      v8i16, v8i16, IntOp, Commutable>;
  def v4i32 : N3VQInt<op24, op23, 0b10, op11_8, op4, f, itinQ32,
                      OpcodeStr, !strconcat(Dt, "32"),
                      v4i32, v4i32, IntOp, Commutable>;
}
multiclass N3VInt_HSSh<bit op24, bit op23, bits<4> op11_8, bit op4, Format f,
                     InstrItinClass itinD16, InstrItinClass itinD32,
                     InstrItinClass itinQ16, InstrItinClass itinQ32,
                     string OpcodeStr, string Dt,
                     SDPatternOperator IntOp> {
  // 64-bit vector types.
  def v4i16 : N3VDIntSh<op24, op23, 0b01, op11_8, op4, f, itinD16,
                      OpcodeStr, !strconcat(Dt, "16"),
                      v4i16, v4i16, IntOp>;
  def v2i32 : N3VDIntSh<op24, op23, 0b10, op11_8, op4, f, itinD32,
```
- EN: Declares TableGen `multiclass N3VInt_HSSh`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VInt_HSSh`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3623-3624
```tablegen
                      OpcodeStr, !strconcat(Dt, "32"),
                      v2i32, v2i32, IntOp>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3626-3633
```tablegen
  // 128-bit vector types.
  def v8i16 : N3VQIntSh<op24, op23, 0b01, op11_8, op4, f, itinQ16,
                      OpcodeStr, !strconcat(Dt, "16"),
                      v8i16, v8i16, IntOp>;
  def v4i32 : N3VQIntSh<op24, op23, 0b10, op11_8, op4, f, itinQ32,
                      OpcodeStr, !strconcat(Dt, "32"),
                      v4i32, v4i32, IntOp>;
}
```
- EN: Defines TableGen record `v8i16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v8i16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3635-3647
```tablegen
multiclass N3VIntSL_HS<bits<4> op11_8,
                       InstrItinClass itinD16, InstrItinClass itinD32,
                       InstrItinClass itinQ16, InstrItinClass itinQ32,
                       string OpcodeStr, string Dt, SDPatternOperator IntOp> {
  def v4i16 : N3VDIntSL16<0b01, op11_8, itinD16,
                          OpcodeStr, !strconcat(Dt, "16"), v4i16, IntOp>;
  def v2i32 : N3VDIntSL<0b10, op11_8, itinD32,
                        OpcodeStr, !strconcat(Dt, "32"), v2i32, IntOp>;
  def v8i16 : N3VQIntSL16<0b01, op11_8, itinQ16,
                          OpcodeStr, !strconcat(Dt, "16"), v8i16, v4i16, IntOp>;
  def v4i32 : N3VQIntSL<0b10, op11_8, itinQ32,
                        OpcodeStr, !strconcat(Dt, "32"), v4i32, v2i32, IntOp>;
}
```
- EN: Declares TableGen `multiclass N3VIntSL_HS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VIntSL_HS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3649-3666
```tablegen
// ....then also with element size of 8 bits:
multiclass N3VInt_QHS<bit op24, bit op23, bits<4> op11_8, bit op4, Format f,
                      InstrItinClass itinD16, InstrItinClass itinD32,
                      InstrItinClass itinQ16, InstrItinClass itinQ32,
                      string OpcodeStr, string Dt,
                      SDPatternOperator IntOp, bit Commutable = 0>
  : N3VInt_HS<op24, op23, op11_8, op4, f, itinD16, itinD32, itinQ16, itinQ32,
              OpcodeStr, Dt, IntOp, Commutable> {
  def v8i8  : N3VDInt<op24, op23, 0b00, op11_8, op4, f, itinD16,
                      OpcodeStr, !strconcat(Dt, "8"),
                      v8i8, v8i8, IntOp, Commutable>;
  def v16i8 : N3VQInt<op24, op23, 0b00, op11_8, op4, f, itinQ16,
                      OpcodeStr, !strconcat(Dt, "8"),
                      v16i8, v16i8, IntOp, Commutable>;
}
multiclass N3VInt_QHSSh<bit op24, bit op23, bits<4> op11_8, bit op4, Format f,
                      InstrItinClass itinD16, InstrItinClass itinD32,
                      InstrItinClass itinQ16, InstrItinClass itinQ32,
```
- EN: Declares TableGen `multiclass N3VInt_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VInt_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3667-3677
```tablegen
                      string OpcodeStr, string Dt,
                      SDPatternOperator IntOp>
  : N3VInt_HSSh<op24, op23, op11_8, op4, f, itinD16, itinD32, itinQ16, itinQ32,
              OpcodeStr, Dt, IntOp> {
  def v8i8  : N3VDIntSh<op24, op23, 0b00, op11_8, op4, f, itinD16,
                      OpcodeStr, !strconcat(Dt, "8"),
                      v8i8, v8i8, IntOp>;
  def v16i8 : N3VQIntSh<op24, op23, 0b00, op11_8, op4, f, itinQ16,
                      OpcodeStr, !strconcat(Dt, "8"),
                      v16i8, v16i8, IntOp>;
}
```
- EN: Defines TableGen record `v8i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v8i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3680-3697
```tablegen
// ....then also with element size of 64 bits:
multiclass N3VInt_QHSD<bit op24, bit op23, bits<4> op11_8, bit op4, Format f,
                       InstrItinClass itinD16, InstrItinClass itinD32,
                       InstrItinClass itinQ16, InstrItinClass itinQ32,
                       string OpcodeStr, string Dt,
                       SDPatternOperator IntOp, bit Commutable = 0>
  : N3VInt_QHS<op24, op23, op11_8, op4, f, itinD16, itinD32, itinQ16, itinQ32,
               OpcodeStr, Dt, IntOp, Commutable> {
  def v1i64 : N3VDInt<op24, op23, 0b11, op11_8, op4, f, itinD32,
                      OpcodeStr, !strconcat(Dt, "64"),
                      v1i64, v1i64, IntOp, Commutable>;
  def v2i64 : N3VQInt<op24, op23, 0b11, op11_8, op4, f, itinQ32,
                      OpcodeStr, !strconcat(Dt, "64"),
                      v2i64, v2i64, IntOp, Commutable>;
}
multiclass N3VInt_QHSDSh<bit op24, bit op23, bits<4> op11_8, bit op4, Format f,
                       InstrItinClass itinD16, InstrItinClass itinD32,
                       InstrItinClass itinQ16, InstrItinClass itinQ32,
```
- EN: Declares TableGen `multiclass N3VInt_QHSD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VInt_QHSD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3698-3708
```tablegen
                       string OpcodeStr, string Dt,
                       SDPatternOperator IntOp>
  : N3VInt_QHSSh<op24, op23, op11_8, op4, f, itinD16, itinD32, itinQ16, itinQ32,
               OpcodeStr, Dt, IntOp> {
  def v1i64 : N3VDIntSh<op24, op23, 0b11, op11_8, op4, f, itinD32,
                      OpcodeStr, !strconcat(Dt, "64"),
                      v1i64, v1i64, IntOp>;
  def v2i64 : N3VQIntSh<op24, op23, 0b11, op11_8, op4, f, itinQ32,
                      OpcodeStr, !strconcat(Dt, "64"),
                      v2i64, v2i64, IntOp>;
}
```
- EN: Defines TableGen record `v1i64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v1i64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3710-3724
```tablegen
// Neon Narrowing 3-register vector intrinsics,
//   source operand element sizes of 16, 32 and 64 bits:
multiclass N3VNInt_HSD<bit op24, bit op23, bits<4> op11_8, bit op4,
                       string OpcodeStr, string Dt,
                       SDPatternOperator IntOp, bit Commutable = 0> {
  def v8i8  : N3VNInt<op24, op23, 0b00, op11_8, op4,
                      OpcodeStr, !strconcat(Dt, "16"),
                      v8i8, v8i16, IntOp, Commutable>;
  def v4i16 : N3VNInt<op24, op23, 0b01, op11_8, op4,
                      OpcodeStr, !strconcat(Dt, "32"),
                      v4i16, v4i32, IntOp, Commutable>;
  def v2i32 : N3VNInt<op24, op23, 0b10, op11_8, op4,
                      OpcodeStr, !strconcat(Dt, "64"),
                      v2i32, v2i64, IntOp, Commutable>;
}
```
- EN: Declares TableGen `multiclass N3VNInt_HSD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VNInt_HSD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3727-3727
```tablegen
// Neon Long 3-register vector operations.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3729-3742
```tablegen
multiclass N3VL_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                    InstrItinClass itin16, InstrItinClass itin32,
                    string OpcodeStr, string Dt,
                    SDNode OpNode, bit Commutable = 0> {
  def v8i16 : N3VL<op24, op23, 0b00, op11_8, op4, itin16,
                   OpcodeStr, !strconcat(Dt, "8"),
                   v8i16, v8i8, OpNode, Commutable>;
  def v4i32 : N3VL<op24, op23, 0b01, op11_8, op4, itin16,
                   OpcodeStr, !strconcat(Dt, "16"),
                   v4i32, v4i16, OpNode, Commutable>;
  def v2i64 : N3VL<op24, op23, 0b10, op11_8, op4, itin32,
                   OpcodeStr, !strconcat(Dt, "32"),
                   v2i64, v2i32, OpNode, Commutable>;
}
```
- EN: Declares TableGen `multiclass N3VL_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VL_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3744-3751
```tablegen
multiclass N3VLSL_HS<bit op24, bits<4> op11_8,
                     InstrItinClass itin, string OpcodeStr, string Dt,
                     SDNode OpNode> {
  def v4i16 : N3VLSL16<op24, 0b01, op11_8, itin, OpcodeStr,
                       !strconcat(Dt, "16"), v4i32, v4i16, OpNode>;
  def v2i32 : N3VLSL<op24, 0b10, op11_8, itin, OpcodeStr,
                     !strconcat(Dt, "32"), v2i64, v2i32, OpNode>;
}
```
- EN: Declares TableGen `multiclass N3VLSL_HS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLSL_HS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3753-3766
```tablegen
multiclass N3VLExt_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                       InstrItinClass itin16, InstrItinClass itin32,
                       string OpcodeStr, string Dt,
                       SDNode OpNode, SDPatternOperator ExtOp, bit Commutable = 0> {
  def v8i16 : N3VLExt<op24, op23, 0b00, op11_8, op4, itin16,
                      OpcodeStr, !strconcat(Dt, "8"),
                      v8i16, v8i8, OpNode, ExtOp, Commutable>;
  def v4i32 : N3VLExt<op24, op23, 0b01, op11_8, op4, itin16,
                      OpcodeStr, !strconcat(Dt, "16"),
                      v4i32, v4i16, OpNode, ExtOp, Commutable>;
  def v2i64 : N3VLExt<op24, op23, 0b10, op11_8, op4, itin32,
                      OpcodeStr, !strconcat(Dt, "32"),
                      v2i64, v2i32, OpNode, ExtOp, Commutable>;
}
```
- EN: Declares TableGen `multiclass N3VLExt_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLExt_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3768-3768
```tablegen
// Neon Long 3-register vector intrinsics.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3770-3781
```tablegen
// First with only element sizes of 16 and 32 bits:
multiclass N3VLInt_HS<bit op24, bit op23, bits<4> op11_8, bit op4,
                      InstrItinClass itin16, InstrItinClass itin32,
                      string OpcodeStr, string Dt,
                      SDPatternOperator IntOp, bit Commutable = 0> {
  def v4i32 : N3VLInt<op24, op23, 0b01, op11_8, op4, itin16,
                      OpcodeStr, !strconcat(Dt, "16"),
                      v4i32, v4i16, IntOp, Commutable>;
  def v2i64 : N3VLInt<op24, op23, 0b10, op11_8, op4, itin32,
                      OpcodeStr, !strconcat(Dt, "32"),
                      v2i64, v2i32, IntOp, Commutable>;
}
```
- EN: Declares TableGen `multiclass N3VLInt_HS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLInt_HS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3783-3790
```tablegen
multiclass N3VLIntSL_HS<bit op24, bits<4> op11_8,
                        InstrItinClass itin, string OpcodeStr, string Dt,
                        SDPatternOperator IntOp> {
  def v4i16 : N3VLIntSL16<op24, 0b01, op11_8, itin,
                          OpcodeStr, !strconcat(Dt, "16"), v4i32, v4i16, IntOp>;
  def v2i32 : N3VLIntSL<op24, 0b10, op11_8, itin,
                        OpcodeStr, !strconcat(Dt, "32"), v2i64, v2i32, IntOp>;
}
```
- EN: Declares TableGen `multiclass N3VLIntSL_HS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLIntSL_HS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3792-3802
```tablegen
// ....then also with element size of 8 bits:
multiclass N3VLInt_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                       InstrItinClass itin16, InstrItinClass itin32,
                       string OpcodeStr, string Dt,
                       SDPatternOperator IntOp, bit Commutable = 0>
  : N3VLInt_HS<op24, op23, op11_8, op4, itin16, itin32, OpcodeStr, Dt,
               IntOp, Commutable> {
  def v8i16 : N3VLInt<op24, op23, 0b00, op11_8, op4, itin16,
                      OpcodeStr, !strconcat(Dt, "8"),
                      v8i16, v8i8, IntOp, Commutable>;
}
```
- EN: Declares TableGen `multiclass N3VLInt_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLInt_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3804-3817
```tablegen
// ....with explicit extend (VABDL).
multiclass N3VLIntExt_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                       InstrItinClass itin, string OpcodeStr, string Dt,
                       SDPatternOperator IntOp, SDNode ExtOp, bit Commutable = 0> {
  def v8i16 : N3VLIntExt<op24, op23, 0b00, op11_8, op4, itin,
                         OpcodeStr, !strconcat(Dt, "8"),
                         v8i16, v8i8, IntOp, ExtOp, Commutable>;
  def v4i32 : N3VLIntExt<op24, op23, 0b01, op11_8, op4, itin,
                         OpcodeStr, !strconcat(Dt, "16"),
                         v4i32, v4i16, IntOp, ExtOp, Commutable>;
  def v2i64 : N3VLIntExt<op24, op23, 0b10, op11_8, op4, itin,
                         OpcodeStr, !strconcat(Dt, "32"),
                         v2i64, v2i32, IntOp, ExtOp, Commutable>;
}
```
- EN: Declares TableGen `multiclass N3VLIntExt_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLIntExt_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3820-3834
```tablegen
// Neon Wide 3-register vector intrinsics,
//   source operand element sizes of 8, 16 and 32 bits:
multiclass N3VW_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                    string OpcodeStr, string Dt,
                    SDNode OpNode, SDPatternOperator ExtOp, bit Commutable = 0> {
  def v8i16 : N3VW<op24, op23, 0b00, op11_8, op4,
                   OpcodeStr, !strconcat(Dt, "8"),
                   v8i16, v8i8, OpNode, ExtOp, Commutable>;
  def v4i32 : N3VW<op24, op23, 0b01, op11_8, op4,
                   OpcodeStr, !strconcat(Dt, "16"),
                   v4i32, v4i16, OpNode, ExtOp, Commutable>;
  def v2i64 : N3VW<op24, op23, 0b10, op11_8, op4,
                   OpcodeStr, !strconcat(Dt, "32"),
                   v2i64, v2i32, OpNode, ExtOp, Commutable>;
}
```
- EN: Declares TableGen `multiclass N3VW_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VW_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3837-3849
```tablegen
// Neon Multiply-Op vector operations,
//   element sizes of 8, 16 and 32 bits:
multiclass N3VMulOp_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                        InstrItinClass itinD16, InstrItinClass itinD32,
                        InstrItinClass itinQ16, InstrItinClass itinQ32,
                        string OpcodeStr, string Dt, SDNode OpNode> {
  // 64-bit vector types.
  def v8i8  : N3VDMulOp<op24, op23, 0b00, op11_8, op4, itinD16,
                        OpcodeStr, !strconcat(Dt, "8"), v8i8, mul, OpNode>;
  def v4i16 : N3VDMulOp<op24, op23, 0b01, op11_8, op4, itinD16,
                        OpcodeStr, !strconcat(Dt, "16"), v4i16, mul, OpNode>;
  def v2i32 : N3VDMulOp<op24, op23, 0b10, op11_8, op4, itinD32,
                        OpcodeStr, !strconcat(Dt, "32"), v2i32, mul, OpNode>;
```
- EN: Declares TableGen `multiclass N3VMulOp_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VMulOp_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3851-3858
```tablegen
  // 128-bit vector types.
  def v16i8 : N3VQMulOp<op24, op23, 0b00, op11_8, op4, itinQ16,
                        OpcodeStr, !strconcat(Dt, "8"), v16i8, mul, OpNode>;
  def v8i16 : N3VQMulOp<op24, op23, 0b01, op11_8, op4, itinQ16,
                        OpcodeStr, !strconcat(Dt, "16"), v8i16, mul, OpNode>;
  def v4i32 : N3VQMulOp<op24, op23, 0b10, op11_8, op4, itinQ32,
                        OpcodeStr, !strconcat(Dt, "32"), v4i32, mul, OpNode>;
}
```
- EN: Defines TableGen record `v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3860-3874
```tablegen
multiclass N3VMulOpSL_HS<bits<4> op11_8,
                         InstrItinClass itinD16, InstrItinClass itinD32,
                         InstrItinClass itinQ16, InstrItinClass itinQ32,
                         string OpcodeStr, string Dt, SDPatternOperator ShOp> {
  def v4i16 : N3VDMulOpSL16<0b01, op11_8, itinD16,
                            OpcodeStr, !strconcat(Dt, "16"), v4i16, mul, ShOp>;
  def v2i32 : N3VDMulOpSL<0b10, op11_8, itinD32,
                          OpcodeStr, !strconcat(Dt, "32"), v2i32, mul, ShOp>;
  def v8i16 : N3VQMulOpSL16<0b01, op11_8, itinQ16,
                            OpcodeStr, !strconcat(Dt, "16"), v8i16, v4i16,
                            mul, ShOp>;
  def v4i32 : N3VQMulOpSL<0b10, op11_8, itinQ32,
                          OpcodeStr, !strconcat(Dt, "32"), v4i32, v2i32,
                          mul, ShOp>;
}
```
- EN: Declares TableGen `multiclass N3VMulOpSL_HS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VMulOpSL_HS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3876-3888
```tablegen
// Neon Intrinsic-Op vector operations,
//   element sizes of 8, 16 and 32 bits:
multiclass N3VIntOp_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                        InstrItinClass itinD, InstrItinClass itinQ,
                        string OpcodeStr, string Dt, SDPatternOperator IntOp,
                        SDNode OpNode> {
  // 64-bit vector types.
  def v8i8  : N3VDIntOp<op24, op23, 0b00, op11_8, op4, itinD,
                        OpcodeStr, !strconcat(Dt, "8"), v8i8, IntOp, OpNode>;
  def v4i16 : N3VDIntOp<op24, op23, 0b01, op11_8, op4, itinD,
                        OpcodeStr, !strconcat(Dt, "16"), v4i16, IntOp, OpNode>;
  def v2i32 : N3VDIntOp<op24, op23, 0b10, op11_8, op4, itinD,
                        OpcodeStr, !strconcat(Dt, "32"), v2i32, IntOp, OpNode>;
```
- EN: Declares TableGen `multiclass N3VIntOp_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VIntOp_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3890-3897
```tablegen
  // 128-bit vector types.
  def v16i8 : N3VQIntOp<op24, op23, 0b00, op11_8, op4, itinQ,
                        OpcodeStr, !strconcat(Dt, "8"), v16i8, IntOp, OpNode>;
  def v8i16 : N3VQIntOp<op24, op23, 0b01, op11_8, op4, itinQ,
                        OpcodeStr, !strconcat(Dt, "16"), v8i16, IntOp, OpNode>;
  def v4i32 : N3VQIntOp<op24, op23, 0b10, op11_8, op4, itinQ,
                        OpcodeStr, !strconcat(Dt, "32"), v4i32, IntOp, OpNode>;
}
```
- EN: Defines TableGen record `v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3899-3909
```tablegen
// Neon 3-argument intrinsics,
//   element sizes of 16 and 32 bits:
multiclass N3VInt3_HS<bit op24, bit op23, bits<4> op11_8, bit op4,
                       InstrItinClass itinD16, InstrItinClass itinD32,
                       InstrItinClass itinQ16, InstrItinClass itinQ32,
                       string OpcodeStr, string Dt, SDPatternOperator IntOp> {
  // 64-bit vector types.
  def v4i16 : N3VDInt3<op24, op23, 0b01, op11_8, op4, itinD16,
                       OpcodeStr, !strconcat(Dt, "16"), v4i16, v4i16, IntOp>;
  def v2i32 : N3VDInt3<op24, op23, 0b10, op11_8, op4, itinD32,
                       OpcodeStr, !strconcat(Dt, "32"), v2i32, v2i32, IntOp>;
```
- EN: Declares TableGen `multiclass N3VInt3_HS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VInt3_HS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3911-3916
```tablegen
  // 128-bit vector types.
  def v8i16 : N3VQInt3<op24, op23, 0b01, op11_8, op4, itinQ16,
                       OpcodeStr, !strconcat(Dt, "16"), v8i16, v8i16, IntOp>;
  def v4i32 : N3VQInt3<op24, op23, 0b10, op11_8, op4, itinQ32,
                       OpcodeStr, !strconcat(Dt, "32"), v4i32, v4i32, IntOp>;
}
```
- EN: Defines TableGen record `v8i16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v8i16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3918-3931
```tablegen
//   element sizes of 8, 16 and 32 bits:
multiclass N3VInt3_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                       InstrItinClass itinD16, InstrItinClass itinD32,
                       InstrItinClass itinQ16, InstrItinClass itinQ32,
                       string OpcodeStr, string Dt, SDPatternOperator IntOp>
           :N3VInt3_HS <op24, op23, op11_8, op4, itinD16, itinD32,
                        itinQ16, itinQ32, OpcodeStr, Dt, IntOp>{
  // 64-bit vector types.
  def v8i8  : N3VDInt3<op24, op23, 0b00, op11_8, op4, itinD16,
                       OpcodeStr, !strconcat(Dt, "8"), v8i8, v8i8, IntOp>;
  // 128-bit vector types.
  def v16i8 : N3VQInt3<op24, op23, 0b00, op11_8, op4, itinQ16,
                       OpcodeStr, !strconcat(Dt, "8"), v16i8, v16i8, IntOp>;
}
```
- EN: Declares TableGen `multiclass N3VInt3_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VInt3_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3933-3945
```tablegen
// Neon Long Multiply-Op vector operations,
//   element sizes of 8, 16 and 32 bits:
multiclass N3VLMulOp_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                         InstrItinClass itin16, InstrItinClass itin32,
                         string OpcodeStr, string Dt, SDNode MulOp,
                         SDNode OpNode> {
  def v8i16 : N3VLMulOp<op24, op23, 0b00, op11_8, op4, itin16, OpcodeStr,
                        !strconcat(Dt, "8"), v8i16, v8i8, MulOp, OpNode>;
  def v4i32 : N3VLMulOp<op24, op23, 0b01, op11_8, op4, itin16, OpcodeStr,
                        !strconcat(Dt, "16"), v4i32, v4i16, MulOp, OpNode>;
  def v2i64 : N3VLMulOp<op24, op23, 0b10, op11_8, op4, itin32, OpcodeStr,
                        !strconcat(Dt, "32"), v2i64, v2i32, MulOp, OpNode>;
}
```
- EN: Declares TableGen `multiclass N3VLMulOp_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLMulOp_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3947-3953
```tablegen
multiclass N3VLMulOpSL_HS<bit op24, bits<4> op11_8, string OpcodeStr,
                          string Dt, SDNode MulOp, SDNode OpNode> {
  def v4i16 : N3VLMulOpSL16<op24, 0b01, op11_8, IIC_VMACi16D, OpcodeStr,
                            !strconcat(Dt,"16"), v4i32, v4i16, MulOp, OpNode>;
  def v2i32 : N3VLMulOpSL<op24, 0b10, op11_8, IIC_VMACi32D, OpcodeStr,
                          !strconcat(Dt, "32"), v2i64, v2i32, MulOp, OpNode>;
}
```
- EN: Declares TableGen `multiclass N3VLMulOpSL_HS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLMulOpSL_HS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3956-3956
```tablegen
// Neon Long 3-argument intrinsics.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3958-3966
```tablegen
// First with only element sizes of 16 and 32 bits:
multiclass N3VLInt3_HS<bit op24, bit op23, bits<4> op11_8, bit op4,
                       InstrItinClass itin16, InstrItinClass itin32,
                       string OpcodeStr, string Dt, SDPatternOperator IntOp> {
  def v4i32 : N3VLInt3<op24, op23, 0b01, op11_8, op4, itin16,
                       OpcodeStr, !strconcat(Dt, "16"), v4i32, v4i16, IntOp>;
  def v2i64 : N3VLInt3<op24, op23, 0b10, op11_8, op4, itin32,
                       OpcodeStr, !strconcat(Dt, "32"), v2i64, v2i32, IntOp>;
}
```
- EN: Declares TableGen `multiclass N3VLInt3_HS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLInt3_HS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3968-3974
```tablegen
multiclass N3VLInt3SL_HS<bit op24, bits<4> op11_8,
                         string OpcodeStr, string Dt, SDPatternOperator IntOp> {
  def v4i16 : N3VLInt3SL16<op24, 0b01, op11_8, IIC_VMACi16D,
                           OpcodeStr, !strconcat(Dt,"16"), v4i32, v4i16, IntOp>;
  def v2i32 : N3VLInt3SL<op24, 0b10, op11_8, IIC_VMACi32D,
                         OpcodeStr, !strconcat(Dt, "32"), v2i64, v2i32, IntOp>;
}
```
- EN: Declares TableGen `multiclass N3VLInt3SL_HS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLInt3SL_HS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3976-3983
```tablegen
// ....then also with element size of 8 bits:
multiclass N3VLInt3_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                        InstrItinClass itin16, InstrItinClass itin32,
                        string OpcodeStr, string Dt, SDPatternOperator IntOp>
  : N3VLInt3_HS<op24, op23, op11_8, op4, itin16, itin32, OpcodeStr, Dt, IntOp> {
  def v8i16 : N3VLInt3<op24, op23, 0b00, op11_8, op4, itin16,
                       OpcodeStr, !strconcat(Dt, "8"), v8i16, v8i8, IntOp>;
}
```
- EN: Declares TableGen `multiclass N3VLInt3_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLInt3_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3985-3998
```tablegen
// ....with explicit extend (VABAL).
multiclass N3VLIntExtOp_QHS<bit op24, bit op23, bits<4> op11_8, bit op4,
                            InstrItinClass itin, string OpcodeStr, string Dt,
                            SDPatternOperator IntOp, SDNode ExtOp, SDNode OpNode> {
  def v8i16 : N3VLIntExtOp<op24, op23, 0b00, op11_8, op4, itin,
                           OpcodeStr, !strconcat(Dt, "8"), v8i16, v8i8,
                           IntOp, ExtOp, OpNode>;
  def v4i32 : N3VLIntExtOp<op24, op23, 0b01, op11_8, op4, itin,
                           OpcodeStr, !strconcat(Dt, "16"), v4i32, v4i16,
                           IntOp, ExtOp, OpNode>;
  def v2i64 : N3VLIntExtOp<op24, op23, 0b10, op11_8, op4, itin,
                           OpcodeStr, !strconcat(Dt, "32"), v2i64, v2i32,
                           IntOp, ExtOp, OpNode>;
}
```
- EN: Declares TableGen `multiclass N3VLIntExtOp_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VLIntExtOp_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4001-4012
```tablegen
// Neon Pairwise long 2-register intrinsics,
//   element sizes of 8, 16 and 32 bits:
multiclass N2VPLInt_QHS<bits<2> op24_23, bits<2> op21_20, bits<2> op17_16,
                        bits<5> op11_7, bit op4,
                        string OpcodeStr, string Dt, SDPatternOperator IntOp> {
  // 64-bit vector types.
  def v8i8  : N2VDPLInt<op24_23, op21_20, 0b00, op17_16, op11_7, op4,
                        OpcodeStr, !strconcat(Dt, "8"), v4i16, v8i8, IntOp>;
  def v4i16 : N2VDPLInt<op24_23, op21_20, 0b01, op17_16, op11_7, op4,
                        OpcodeStr, !strconcat(Dt, "16"), v2i32, v4i16, IntOp>;
  def v2i32 : N2VDPLInt<op24_23, op21_20, 0b10, op17_16, op11_7, op4,
                        OpcodeStr, !strconcat(Dt, "32"), v1i64, v2i32, IntOp>;
```
- EN: Declares TableGen `multiclass N2VPLInt_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VPLInt_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4014-4021
```tablegen
  // 128-bit vector types.
  def v16i8 : N2VQPLInt<op24_23, op21_20, 0b00, op17_16, op11_7, op4,
                        OpcodeStr, !strconcat(Dt, "8"), v8i16, v16i8, IntOp>;
  def v8i16 : N2VQPLInt<op24_23, op21_20, 0b01, op17_16, op11_7, op4,
                        OpcodeStr, !strconcat(Dt, "16"), v4i32, v8i16, IntOp>;
  def v4i32 : N2VQPLInt<op24_23, op21_20, 0b10, op17_16, op11_7, op4,
                        OpcodeStr, !strconcat(Dt, "32"), v2i64, v4i32, IntOp>;
}
```
- EN: Defines TableGen record `v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4024-4035
```tablegen
// Neon Pairwise long 2-register accumulate intrinsics,
//   element sizes of 8, 16 and 32 bits:
multiclass N2VPLInt2_QHS<bits<2> op24_23, bits<2> op21_20, bits<2> op17_16,
                         bits<5> op11_7, bit op4,
                         string OpcodeStr, string Dt, SDPatternOperator IntOp> {
  // 64-bit vector types.
  def v8i8  : N2VDPLInt2<op24_23, op21_20, 0b00, op17_16, op11_7, op4,
                         OpcodeStr, !strconcat(Dt, "8"), v4i16, v8i8, IntOp>;
  def v4i16 : N2VDPLInt2<op24_23, op21_20, 0b01, op17_16, op11_7, op4,
                         OpcodeStr, !strconcat(Dt, "16"), v2i32, v4i16, IntOp>;
  def v2i32 : N2VDPLInt2<op24_23, op21_20, 0b10, op17_16, op11_7, op4,
                         OpcodeStr, !strconcat(Dt, "32"), v1i64, v2i32, IntOp>;
```
- EN: Declares TableGen `multiclass N2VPLInt2_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VPLInt2_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4037-4044
```tablegen
  // 128-bit vector types.
  def v16i8 : N2VQPLInt2<op24_23, op21_20, 0b00, op17_16, op11_7, op4,
                         OpcodeStr, !strconcat(Dt, "8"), v8i16, v16i8, IntOp>;
  def v8i16 : N2VQPLInt2<op24_23, op21_20, 0b01, op17_16, op11_7, op4,
                         OpcodeStr, !strconcat(Dt, "16"), v4i32, v8i16, IntOp>;
  def v4i32 : N2VQPLInt2<op24_23, op21_20, 0b10, op17_16, op11_7, op4,
                         OpcodeStr, !strconcat(Dt, "32"), v2i64, v4i32, IntOp>;
}
```
- EN: Defines TableGen record `v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4047-4064
```tablegen
// Neon 2-register vector shift by immediate,
//   with f of either N2RegVShLFrm or N2RegVShRFrm
//   element sizes of 8, 16, 32 and 64 bits:
multiclass N2VShL_QHSD<bit op24, bit op23, bits<4> op11_8, bit op4,
                       InstrItinClass itin, string OpcodeStr, string Dt,
                       SDNode OpNode> {
  // 64-bit vector types.
  def v8i8  : N2VDSh<op24, op23, op11_8, 0, op4, N2RegVShLFrm, itin, i32imm,
                     OpcodeStr, !strconcat(Dt, "8"), v8i8, OpNode> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v4i16 : N2VDSh<op24, op23, op11_8, 0, op4, N2RegVShLFrm, itin, i32imm,
                     OpcodeStr, !strconcat(Dt, "16"), v4i16, OpNode> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v2i32 : N2VDSh<op24, op23, op11_8, 0, op4, N2RegVShLFrm, itin, i32imm,
                     OpcodeStr, !strconcat(Dt, "32"), v2i32, OpNode> {
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
```
- EN: Declares TableGen `multiclass N2VShL_QHSD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VShL_QHSD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4065-4068
```tablegen
  }
  def v1i64 : N2VDSh<op24, op23, op11_8, 1, op4, N2RegVShLFrm, itin, i32imm,
                     OpcodeStr, !strconcat(Dt, "64"), v1i64, OpNode>;
                             // imm6 = xxxxxx
```
- EN: Defines TableGen record `v1i64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v1i64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4070-4087
```tablegen
  // 128-bit vector types.
  def v16i8 : N2VQSh<op24, op23, op11_8, 0, op4, N2RegVShLFrm, itin, i32imm,
                     OpcodeStr, !strconcat(Dt, "8"), v16i8, OpNode> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v8i16 : N2VQSh<op24, op23, op11_8, 0, op4, N2RegVShLFrm, itin, i32imm,
                     OpcodeStr, !strconcat(Dt, "16"), v8i16, OpNode> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v4i32 : N2VQSh<op24, op23, op11_8, 0, op4, N2RegVShLFrm, itin, i32imm,
                     OpcodeStr, !strconcat(Dt, "32"), v4i32, OpNode> {
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
  }
  def v2i64 : N2VQSh<op24, op23, op11_8, 1, op4, N2RegVShLFrm, itin, i32imm,
                     OpcodeStr, !strconcat(Dt, "64"), v2i64, OpNode>;
                             // imm6 = xxxxxx
}
multiclass N2VShR_QHSD<bit op24, bit op23, bits<4> op11_8, bit op4,
```
- EN: Declares TableGen `multiclass N2VShR_QHSD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VShR_QHSD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4088-4105
```tablegen
                       InstrItinClass itin, string OpcodeStr, string Dt,
                       SDNode OpNode> {
  // 64-bit vector types.
  def v8i8  : N2VDSh<op24, op23, op11_8, 0, op4, N2RegVShRFrm, itin, shr_imm8,
                     OpcodeStr, !strconcat(Dt, "8"), v8i8, OpNode> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v4i16 : N2VDSh<op24, op23, op11_8, 0, op4, N2RegVShRFrm, itin, shr_imm16,
                     OpcodeStr, !strconcat(Dt, "16"), v4i16, OpNode> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v2i32 : N2VDSh<op24, op23, op11_8, 0, op4, N2RegVShRFrm, itin, shr_imm32,
                     OpcodeStr, !strconcat(Dt, "32"), v2i32, OpNode> {
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
  }
  def v1i64 : N2VDSh<op24, op23, op11_8, 1, op4, N2RegVShRFrm, itin, shr_imm64,
                     OpcodeStr, !strconcat(Dt, "64"), v1i64, OpNode>;
                             // imm6 = xxxxxx
```
- EN: Defines TableGen record `v8i8` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `v8i8`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 4107-4123
```tablegen
  // 128-bit vector types.
  def v16i8 : N2VQSh<op24, op23, op11_8, 0, op4, N2RegVShRFrm, itin, shr_imm8,
                     OpcodeStr, !strconcat(Dt, "8"), v16i8, OpNode> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v8i16 : N2VQSh<op24, op23, op11_8, 0, op4, N2RegVShRFrm, itin, shr_imm16,
                     OpcodeStr, !strconcat(Dt, "16"), v8i16, OpNode> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v4i32 : N2VQSh<op24, op23, op11_8, 0, op4, N2RegVShRFrm, itin, shr_imm32,
                     OpcodeStr, !strconcat(Dt, "32"), v4i32, OpNode> {
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
  }
  def v2i64 : N2VQSh<op24, op23, op11_8, 1, op4, N2RegVShRFrm, itin, shr_imm64,
                     OpcodeStr, !strconcat(Dt, "64"), v2i64, OpNode>;
                             // imm6 = xxxxxx
}
```
- EN: Defines TableGen record `v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4125-4142
```tablegen
// Neon Shift-Accumulate vector operations,
//   element sizes of 8, 16, 32 and 64 bits:
multiclass N2VShAdd_QHSD<bit op24, bit op23, bits<4> op11_8, bit op4,
                         string OpcodeStr, string Dt, SDNode ShOp> {
  // 64-bit vector types.
  def v8i8  : N2VDShAdd<op24, op23, op11_8, 0, op4, shr_imm8,
                        OpcodeStr, !strconcat(Dt, "8"), v8i8, ShOp> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v4i16 : N2VDShAdd<op24, op23, op11_8, 0, op4, shr_imm16,
                        OpcodeStr, !strconcat(Dt, "16"), v4i16, ShOp> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v2i32 : N2VDShAdd<op24, op23, op11_8, 0, op4, shr_imm32,
                        OpcodeStr, !strconcat(Dt, "32"), v2i32, ShOp> {
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
  }
  def v1i64 : N2VDShAdd<op24, op23, op11_8, 1, op4, shr_imm64,
```
- EN: Declares TableGen `multiclass N2VShAdd_QHSD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VShAdd_QHSD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4143-4144
```tablegen
                        OpcodeStr, !strconcat(Dt, "64"), v1i64, ShOp>;
                             // imm6 = xxxxxx
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4146-4162
```tablegen
  // 128-bit vector types.
  def v16i8 : N2VQShAdd<op24, op23, op11_8, 0, op4, shr_imm8,
                        OpcodeStr, !strconcat(Dt, "8"), v16i8, ShOp> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v8i16 : N2VQShAdd<op24, op23, op11_8, 0, op4, shr_imm16,
                        OpcodeStr, !strconcat(Dt, "16"), v8i16, ShOp> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v4i32 : N2VQShAdd<op24, op23, op11_8, 0, op4, shr_imm32,
                        OpcodeStr, !strconcat(Dt, "32"), v4i32, ShOp> {
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
  }
  def v2i64 : N2VQShAdd<op24, op23, op11_8, 1, op4, shr_imm64,
                        OpcodeStr, !strconcat(Dt, "64"), v2i64, ShOp>;
                             // imm6 = xxxxxx
}
```
- EN: Defines TableGen record `v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4164-4181
```tablegen
// Neon Shift-Insert vector operations,
//   with f of either N2RegVShLFrm or N2RegVShRFrm
//   element sizes of 8, 16, 32 and 64 bits:
multiclass N2VShInsL_QHSD<bit op24, bit op23, bits<4> op11_8, bit op4,
                          string OpcodeStr> {
  // 64-bit vector types.
  def v8i8  : N2VDShIns<op24, op23, op11_8, 0, op4, i32imm,
                        N2RegVShLFrm, OpcodeStr, "8", v8i8, NEONvsliImm> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v4i16 : N2VDShIns<op24, op23, op11_8, 0, op4, i32imm,
                        N2RegVShLFrm, OpcodeStr, "16", v4i16, NEONvsliImm> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v2i32 : N2VDShIns<op24, op23, op11_8, 0, op4, i32imm,
                        N2RegVShLFrm, OpcodeStr, "32", v2i32, NEONvsliImm> {
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
  }
```
- EN: Declares TableGen `multiclass N2VShInsL_QHSD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VShInsL_QHSD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4182-4184
```tablegen
  def v1i64 : N2VDShIns<op24, op23, op11_8, 1, op4, i32imm,
                        N2RegVShLFrm, OpcodeStr, "64", v1i64, NEONvsliImm>;
                             // imm6 = xxxxxx
```
- EN: Defines TableGen record `v1i64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v1i64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4186-4203
```tablegen
  // 128-bit vector types.
  def v16i8 : N2VQShIns<op24, op23, op11_8, 0, op4, i32imm,
                        N2RegVShLFrm, OpcodeStr, "8", v16i8, NEONvsliImm> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v8i16 : N2VQShIns<op24, op23, op11_8, 0, op4, i32imm,
                        N2RegVShLFrm, OpcodeStr, "16", v8i16, NEONvsliImm> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v4i32 : N2VQShIns<op24, op23, op11_8, 0, op4, i32imm,
                        N2RegVShLFrm, OpcodeStr, "32", v4i32, NEONvsliImm> {
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
  }
  def v2i64 : N2VQShIns<op24, op23, op11_8, 1, op4, i32imm,
                        N2RegVShLFrm, OpcodeStr, "64", v2i64, NEONvsliImm>;
                             // imm6 = xxxxxx
}
multiclass N2VShInsR_QHSD<bit op24, bit op23, bits<4> op11_8, bit op4,
```
- EN: Declares TableGen `multiclass N2VShInsR_QHSD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VShInsR_QHSD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4204-4220
```tablegen
                          string OpcodeStr> {
  // 64-bit vector types.
  def v8i8  : N2VDShIns<op24, op23, op11_8, 0, op4, shr_imm8,
                        N2RegVShRFrm, OpcodeStr, "8", v8i8, NEONvsriImm> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v4i16 : N2VDShIns<op24, op23, op11_8, 0, op4, shr_imm16,
                        N2RegVShRFrm, OpcodeStr, "16", v4i16, NEONvsriImm> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v2i32 : N2VDShIns<op24, op23, op11_8, 0, op4, shr_imm32,
                        N2RegVShRFrm, OpcodeStr, "32", v2i32, NEONvsriImm> {
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
  }
  def v1i64 : N2VDShIns<op24, op23, op11_8, 1, op4, shr_imm64,
                        N2RegVShRFrm, OpcodeStr, "64", v1i64, NEONvsriImm>;
                             // imm6 = xxxxxx
```
- EN: Defines TableGen record `v8i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v8i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4222-4238
```tablegen
  // 128-bit vector types.
  def v16i8 : N2VQShIns<op24, op23, op11_8, 0, op4, shr_imm8,
                        N2RegVShRFrm, OpcodeStr, "8", v16i8, NEONvsriImm> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v8i16 : N2VQShIns<op24, op23, op11_8, 0, op4, shr_imm16,
                        N2RegVShRFrm, OpcodeStr, "16", v8i16, NEONvsriImm> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v4i32 : N2VQShIns<op24, op23, op11_8, 0, op4, shr_imm32,
                        N2RegVShRFrm, OpcodeStr, "32", v4i32, NEONvsriImm> {
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
  }
  def v2i64 : N2VQShIns<op24, op23, op11_8, 1, op4, shr_imm64,
                        N2RegVShRFrm, OpcodeStr, "64", v2i64, NEONvsriImm>;
                             // imm6 = xxxxxx
}
```
- EN: Defines TableGen record `v16i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v16i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4240-4257
```tablegen
// Neon Shift Long operations,
//   element sizes of 8, 16, 32 bits:
multiclass N2VLSh_QHS<bit op24, bit op23, bits<4> op11_8, bit op7, bit op6,
                      bit op4, string OpcodeStr, string Dt,
                      SDPatternOperator OpNode> {
  def v8i16 : N2VLSh<op24, op23, op11_8, op7, op6, op4,
              OpcodeStr, !strconcat(Dt, "8"), v8i16, v8i8, imm1_7, OpNode> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v4i32 : N2VLSh<op24, op23, op11_8, op7, op6, op4,
               OpcodeStr, !strconcat(Dt, "16"), v4i32, v4i16, imm1_15, OpNode> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v2i64 : N2VLSh<op24, op23, op11_8, op7, op6, op4,
               OpcodeStr, !strconcat(Dt, "32"), v2i64, v2i32, imm1_31, OpNode> {
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
  }
}
```
- EN: Declares TableGen `multiclass N2VLSh_QHS`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VLSh_QHS`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4259-4276
```tablegen
// Neon Shift Narrow operations,
//   element sizes of 16, 32, 64 bits:
multiclass N2VNSh_HSD<bit op24, bit op23, bits<4> op11_8, bit op7, bit op6,
                      bit op4, InstrItinClass itin, string OpcodeStr, string Dt,
                      SDPatternOperator OpNode> {
  def v8i8 : N2VNSh<op24, op23, op11_8, op7, op6, op4, itin,
                    OpcodeStr, !strconcat(Dt, "16"),
                    v8i8, v8i16, shr_imm8, OpNode> {
    let Inst{21-19} = 0b001; // imm6 = 001xxx
  }
  def v4i16 : N2VNSh<op24, op23, op11_8, op7, op6, op4, itin,
                     OpcodeStr, !strconcat(Dt, "32"),
                     v4i16, v4i32, shr_imm16, OpNode> {
    let Inst{21-20} = 0b01;  // imm6 = 01xxxx
  }
  def v2i32 : N2VNSh<op24, op23, op11_8, op7, op6, op4, itin,
                     OpcodeStr, !strconcat(Dt, "64"),
                     v2i32, v2i64, shr_imm32, OpNode> {
```
- EN: Declares TableGen `multiclass N2VNSh_HSD`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N2VNSh_HSD`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4277-4279
```tablegen
    let Inst{21} = 0b1;      // imm6 = 1xxxxx
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4281-4283
```tablegen
//===----------------------------------------------------------------------===//
// Instruction Definitions.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4285-4285
```tablegen
// Vector Add Operations.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4287-4304
```tablegen
//   VADD     : Vector Add (integer and floating-point)
defm VADD     : N3V_QHSD<0, 0, 0b1000, 0, IIC_VBINiD, IIC_VBINiQ, "vadd", "i",
                         add, 1>;
def  VADDfd   : N3VD<0, 0, 0b00, 0b1101, 0, IIC_VBIND, "vadd", "f32",
                     v2f32, v2f32, fadd, 1>;
def  VADDfq   : N3VQ<0, 0, 0b00, 0b1101, 0, IIC_VBINQ, "vadd", "f32",
                     v4f32, v4f32, fadd, 1>;
def  VADDhd   : N3VD<0, 0, 0b01, 0b1101, 0, IIC_VBIND, "vadd", "f16",
                     v4f16, v4f16, fadd, 1>,
                Requires<[HasNEON,HasFullFP16]>;
def  VADDhq   : N3VQ<0, 0, 0b01, 0b1101, 0, IIC_VBINQ, "vadd", "f16",
                     v8f16, v8f16, fadd, 1>,
                Requires<[HasNEON,HasFullFP16]>;
//   VADDL    : Vector Add Long (Q = D + D)
defm VADDLs   : N3VLExt_QHS<0,1,0b0000,0, IIC_VSHLiD, IIC_VSHLiD,
                            "vaddl", "s", add, sext, 1>;
defm VADDLu   : N3VLExt_QHS<1,1,0b0000,0, IIC_VSHLiD, IIC_VSHLiD,
                            "vaddl", "u", add, zanyext, 1>;
```
- EN: Defines TableGen record `VADD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VADD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4305-4322
```tablegen
//   VADDW    : Vector Add Wide (Q = Q + D)
defm VADDWs   : N3VW_QHS<0,1,0b0001,0, "vaddw", "s", add, sext, 0>;
defm VADDWu   : N3VW_QHS<1,1,0b0001,0, "vaddw", "u", add, zanyext, 0>;
//   VHADD    : Vector Halving Add
defm VHADDs   : N3VInt_QHS<0, 0, 0b0000, 0, N3RegFrm,
                           IIC_VBINi4D, IIC_VBINi4D, IIC_VBINi4Q, IIC_VBINi4Q,
                           "vhadd", "s", int_arm_neon_vhadds, 1>;
defm VHADDu   : N3VInt_QHS<1, 0, 0b0000, 0, N3RegFrm,
                           IIC_VBINi4D, IIC_VBINi4D, IIC_VBINi4Q, IIC_VBINi4Q,
                           "vhadd", "u", int_arm_neon_vhaddu, 1>;
//   VRHADD   : Vector Rounding Halving Add
defm VRHADDs  : N3VInt_QHS<0, 0, 0b0001, 0, N3RegFrm,
                           IIC_VBINi4D, IIC_VBINi4D, IIC_VBINi4Q, IIC_VBINi4Q,
                           "vrhadd", "s", int_arm_neon_vrhadds, 1>;
defm VRHADDu  : N3VInt_QHS<1, 0, 0b0001, 0, N3RegFrm,
                           IIC_VBINi4D, IIC_VBINi4D, IIC_VBINi4Q, IIC_VBINi4Q,
                           "vrhadd", "u", int_arm_neon_vrhaddu, 1>;
//   VQADD    : Vector Saturating Add
```
- EN: Defines TableGen record `VADDWs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VADDWs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4323-4333
```tablegen
defm VQADDs   : N3VInt_QHSD<0, 0, 0b0000, 1, N3RegFrm,
                            IIC_VBINi4D, IIC_VBINi4D, IIC_VBINi4Q, IIC_VBINi4Q,
                            "vqadd", "s", saddsat, 1>;
defm VQADDu   : N3VInt_QHSD<1, 0, 0b0000, 1, N3RegFrm,
                            IIC_VBINi4D, IIC_VBINi4D, IIC_VBINi4Q, IIC_VBINi4Q,
                            "vqadd", "u", uaddsat, 1>;
//   VADDHN   : Vector Add and Narrow Returning High Half (D = Q + Q)
defm VADDHN   : N3VNInt_HSD<0,1,0b0100,0, "vaddhn", "i", null_frag, 1>;
//   VRADDHN  : Vector Rounding Add and Narrow Returning High Half (D = Q + Q)
defm VRADDHN  : N3VNInt_HSD<1,1,0b0100,0, "vraddhn", "i",
                            int_arm_neon_vraddhn, 1>;
```
- EN: Defines TableGen record `VQADDs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQADDs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4335-4342
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i8  (trunc (ARMvshruImm (add (v8i16 QPR:$Vn), QPR:$Vm), 8))),
          (VADDHNv8i8 QPR:$Vn, QPR:$Vm)>;
def : Pat<(v4i16 (trunc (ARMvshruImm (add (v4i32 QPR:$Vn), QPR:$Vm), 16))),
          (VADDHNv4i16 QPR:$Vn, QPR:$Vm)>;
def : Pat<(v2i32 (trunc (ARMvshruImm (add (v2i64 QPR:$Vn), QPR:$Vm), 32))),
          (VADDHNv2i32 QPR:$Vn, QPR:$Vm)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4344-4344
```tablegen
// Vector Multiply Operations.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4346-4363
```tablegen
//   VMUL     : Vector Multiply (integer, polynomial and floating-point)
defm VMUL     : N3V_QHS<0, 0, 0b1001, 1, IIC_VMULi16D, IIC_VMULi32D,
                        IIC_VMULi16Q, IIC_VMULi32Q, "vmul", "i", mul, 1>;
def  VMULpd   : N3VDInt<1, 0, 0b00, 0b1001, 1, N3RegFrm, IIC_VMULi16D, "vmul",
                        "p8", v8i8, v8i8, int_arm_neon_vmulp, 1>;
def  VMULpq   : N3VQInt<1, 0, 0b00, 0b1001, 1, N3RegFrm, IIC_VMULi16Q, "vmul",
                        "p8", v16i8, v16i8, int_arm_neon_vmulp, 1>;
def  VMULfd   : N3VD<1, 0, 0b00, 0b1101, 1, IIC_VFMULD, "vmul", "f32",
                     v2f32, v2f32, fmul, 1>;
def  VMULfq   : N3VQ<1, 0, 0b00, 0b1101, 1, IIC_VFMULQ, "vmul", "f32",
                     v4f32, v4f32, fmul, 1>;
def  VMULhd   : N3VD<1, 0, 0b01, 0b1101, 1, IIC_VFMULD, "vmul", "f16",
                     v4f16, v4f16, fmul, 1>,
                Requires<[HasNEON,HasFullFP16]>;
def  VMULhq   : N3VQ<1, 0, 0b01, 0b1101, 1, IIC_VFMULQ, "vmul", "f16",
                     v8f16, v8f16, fmul, 1>,
                Requires<[HasNEON,HasFullFP16]>;
defm VMULsl   : N3VSL_HS<0b1000, "vmul", mul>;
```
- EN: Defines TableGen record `VMUL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMUL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4364-4371
```tablegen
def  VMULslfd : N3VDSL<0b10, 0b1001, IIC_VBIND, "vmul", "f32", v2f32, fmul>;
def  VMULslfq : N3VQSL<0b10, 0b1001, IIC_VBINQ, "vmul", "f32", v4f32,
                       v2f32, fmul>;
def  VMULslhd : N3VDSL16<0b01, 0b1001, "vmul", "f16", v4f16, fmul>,
                Requires<[HasNEON,HasFullFP16]>;
def  VMULslhq : N3VQSL16<0b01, 0b1001, "vmul", "f16", v8f16,
                       v4f16, fmul>,
                Requires<[HasNEON,HasFullFP16]>;
```
- EN: Defines TableGen record `VMULslfd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMULslfd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4373-4390
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i16 (mul (v8i16 QPR:$src1),
                      (v8i16 (ARMvduplane (v8i16 QPR:$src2), imm:$lane)))),
          (v8i16 (VMULslv8i16 (v8i16 QPR:$src1),
                              (v4i16 (EXTRACT_SUBREG QPR:$src2,
                                      (DSubReg_i16_reg imm:$lane))),
                              (SubReg_i16_lane imm:$lane)))>;
def : Pat<(v4i32 (mul (v4i32 QPR:$src1),
                      (v4i32 (ARMvduplane (v4i32 QPR:$src2), imm:$lane)))),
          (v4i32 (VMULslv4i32 (v4i32 QPR:$src1),
                              (v2i32 (EXTRACT_SUBREG QPR:$src2,
                                      (DSubReg_i32_reg imm:$lane))),
                              (SubReg_i32_lane imm:$lane)))>;
def : Pat<(v4f32 (fmul (v4f32 QPR:$src1),
                       (v4f32 (ARMvduplane (v4f32 QPR:$src2), imm:$lane)))),
          (v4f32 (VMULslfq (v4f32 QPR:$src1),
                           (v2f32 (EXTRACT_SUBREG QPR:$src2,
                                   (DSubReg_i32_reg imm:$lane))),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4391-4397
```tablegen
                           (SubReg_i32_lane imm:$lane)))>;
def : Pat<(v8f16 (fmul (v8f16 QPR:$src1),
                       (v8f16 (ARMvduplane (v8f16 QPR:$src2), imm:$lane)))),
          (v8f16 (VMULslhq(v8f16 QPR:$src1),
                           (v4f16 (EXTRACT_SUBREG QPR:$src2,
                                   (DSubReg_i16_reg imm:$lane))),
                           (SubReg_i16_lane imm:$lane)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4399-4415
```tablegen
def : Pat<(v2f32 (fmul DPR:$Rn, (ARMvdup (f32 SPR:$Rm)))),
          (VMULslfd DPR:$Rn,
            (INSERT_SUBREG (v2f32 (IMPLICIT_DEF)), SPR:$Rm, ssub_0),
            (i32 0))>;
def : Pat<(v4f16 (fmul DPR:$Rn, (ARMvdup (f16 HPR:$Rm)))),
          (VMULslhd DPR:$Rn,
            (INSERT_SUBREG (v4f16 (IMPLICIT_DEF)), (f16 HPR:$Rm), ssub_0),
            (i32 0))>;
def : Pat<(v4f32 (fmul QPR:$Rn, (ARMvdup (f32 SPR:$Rm)))),
          (VMULslfq QPR:$Rn,
            (INSERT_SUBREG (v2f32 (IMPLICIT_DEF)), SPR:$Rm, ssub_0),
            (i32 0))>;
def : Pat<(v8f16 (fmul QPR:$Rn, (ARMvdup (f16 HPR:$Rm)))),
          (VMULslhq QPR:$Rn,
            (INSERT_SUBREG (v4f16 (IMPLICIT_DEF)), (f16 HPR:$Rm), ssub_0),
            (i32 0))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4417-4423
```tablegen
//   VQDMULH  : Vector Saturating Doubling Multiply Returning High Half
defm VQDMULH  : N3VInt_HS<0, 0, 0b1011, 0, N3RegFrm, IIC_VMULi16D, IIC_VMULi32D,
                          IIC_VMULi16Q, IIC_VMULi32Q,
                          "vqdmulh", "s", int_arm_neon_vqdmulh, 1>;
defm VQDMULHsl: N3VIntSL_HS<0b1100, IIC_VMULi16D, IIC_VMULi32D,
                            IIC_VMULi16Q, IIC_VMULi32Q,
                            "vqdmulh", "s",  int_arm_neon_vqdmulh>;
```
- EN: Defines TableGen record `VQDMULH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQDMULH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4425-4440
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i16 (int_arm_neon_vqdmulh (v8i16 QPR:$src1),
                                       (v8i16 (ARMvduplane (v8i16 QPR:$src2),
                                                            imm:$lane)))),
          (v8i16 (VQDMULHslv8i16 (v8i16 QPR:$src1),
                                 (v4i16 (EXTRACT_SUBREG QPR:$src2,
                                         (DSubReg_i16_reg imm:$lane))),
                                 (SubReg_i16_lane imm:$lane)))>;
def : Pat<(v4i32 (int_arm_neon_vqdmulh (v4i32 QPR:$src1),
                                       (v4i32 (ARMvduplane (v4i32 QPR:$src2),
                                                            imm:$lane)))),
          (v4i32 (VQDMULHslv4i32 (v4i32 QPR:$src1),
                                 (v2i32 (EXTRACT_SUBREG QPR:$src2,
                                         (DSubReg_i32_reg imm:$lane))),
                                 (SubReg_i32_lane imm:$lane)))>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4442-4448
```tablegen
//   VQRDMULH : Vector Rounding Saturating Doubling Multiply Returning High Half
defm VQRDMULH   : N3VInt_HS<1, 0, 0b1011, 0, N3RegFrm,
                            IIC_VMULi16D,IIC_VMULi32D,IIC_VMULi16Q,IIC_VMULi32Q,
                            "vqrdmulh", "s", int_arm_neon_vqrdmulh, 1>;
defm VQRDMULHsl : N3VIntSL_HS<0b1101, IIC_VMULi16D, IIC_VMULi32D,
                              IIC_VMULi16Q, IIC_VMULi32Q,
                              "vqrdmulh", "s",  int_arm_neon_vqrdmulh>;
```
- EN: Defines TableGen record `VQRDMULH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQRDMULH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4450-4465
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i16 (int_arm_neon_vqrdmulh (v8i16 QPR:$src1),
                                        (v8i16 (ARMvduplane (v8i16 QPR:$src2),
                                                             imm:$lane)))),
          (v8i16 (VQRDMULHslv8i16 (v8i16 QPR:$src1),
                                  (v4i16 (EXTRACT_SUBREG QPR:$src2,
                                          (DSubReg_i16_reg imm:$lane))),
                                  (SubReg_i16_lane imm:$lane)))>;
def : Pat<(v4i32 (int_arm_neon_vqrdmulh (v4i32 QPR:$src1),
                                        (v4i32 (ARMvduplane (v4i32 QPR:$src2),
                                                             imm:$lane)))),
          (v4i32 (VQRDMULHslv4i32 (v4i32 QPR:$src1),
                                  (v2i32 (EXTRACT_SUBREG QPR:$src2,
                                          (DSubReg_i32_reg imm:$lane))),
                                  (SubReg_i32_lane imm:$lane)))>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4467-4481
```tablegen
//   VMULL    : Vector Multiply Long (integer and polynomial) (Q = D * D)
let PostEncoderMethod = "NEONThumb2DataIPostEncoder",
    DecoderNamespace = "NEONData" in {
  defm VMULLs   : N3VL_QHS<0,1,0b1100,0, IIC_VMULi16D, IIC_VMULi32D,
                           "vmull", "s", ARMvmulls, 1>;
  defm VMULLu   : N3VL_QHS<1,1,0b1100,0, IIC_VMULi16D, IIC_VMULi32D,
                           "vmull", "u", ARMvmullu, 1>;
  def  VMULLp8   :  N3VLInt<0, 1, 0b00, 0b1110, 0, IIC_VMULi16D, "vmull", "p8",
                            v8i16, v8i8, int_arm_neon_vmullp, 1>;
  def  VMULLp64  : N3VLIntnp<0b00101, 0b10, 0b1110, 0, 0, NoItinerary,
                          "vmull", "p64", v2i64, v1i64, int_arm_neon_vmullp, 1>,
                    Requires<[HasV8, HasAES]>;
}
defm VMULLsls : N3VLSL_HS<0, 0b1010, IIC_VMULi16D, "vmull", "s", ARMvmulls>;
defm VMULLslu : N3VLSL_HS<1, 0b1010, IIC_VMULi16D, "vmull", "u", ARMvmullu>;
```
- EN: Defines TableGen record `VMULLs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMULLs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4483-4487
```tablegen
//   VQDMULL  : Vector Saturating Doubling Multiply Long (Q = D * D)
defm VQDMULL  : N3VLInt_HS<0,1,0b1101,0, IIC_VMULi16D, IIC_VMULi32D,
                           "vqdmull", "s", int_arm_neon_vqdmull, 1>;
defm VQDMULLsl: N3VLIntSL_HS<0, 0b1011, IIC_VMULi16D,
                             "vqdmull", "s", int_arm_neon_vqdmull>;
```
- EN: Defines TableGen record `VQDMULL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQDMULL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4489-4489
```tablegen
// Vector Multiply-Accumulate and Multiply-Subtract Operations.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4491-4508
```tablegen
//   VMLA     : Vector Multiply Accumulate (integer and floating-point)
defm VMLA     : N3VMulOp_QHS<0, 0, 0b1001, 0, IIC_VMACi16D, IIC_VMACi32D,
                             IIC_VMACi16Q, IIC_VMACi32Q, "vmla", "i", add>;
def  VMLAfd   : N3VDMulOp<0, 0, 0b00, 0b1101, 1, IIC_VMACD, "vmla", "f32",
                          v2f32, fmul_su, fadd_mlx>,
                Requires<[HasNEON, UseFPVMLx]>;
def  VMLAfq   : N3VQMulOp<0, 0, 0b00, 0b1101, 1, IIC_VMACQ, "vmla", "f32",
                          v4f32, fmul_su, fadd_mlx>,
                Requires<[HasNEON, UseFPVMLx]>;
def  VMLAhd   : N3VDMulOp<0, 0, 0b01, 0b1101, 1, IIC_VMACD, "vmla", "f16",
                          v4f16, fmul_su, fadd_mlx>,
                Requires<[HasNEON, HasFullFP16, UseFPVMLx]>;
def  VMLAhq   : N3VQMulOp<0, 0, 0b01, 0b1101, 1, IIC_VMACQ, "vmla", "f16",
                          v8f16, fmul_su, fadd_mlx>,
                Requires<[HasNEON, HasFullFP16, UseFPVMLx]>;
defm VMLAsl   : N3VMulOpSL_HS<0b0000, IIC_VMACi16D, IIC_VMACi32D,
                              IIC_VMACi16Q, IIC_VMACi32Q, "vmla", "i", add>;
def  VMLAslfd : N3VDMulOpSL<0b10, 0b0001, IIC_VMACD, "vmla", "f32",
```
- EN: Defines TableGen record `VMLA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4509-4519
```tablegen
                            v2f32, fmul_su, fadd_mlx>,
                Requires<[HasNEON, UseFPVMLx]>;
def  VMLAslfq : N3VQMulOpSL<0b10, 0b0001, IIC_VMACQ, "vmla", "f32",
                            v4f32, v2f32, fmul_su, fadd_mlx>,
                Requires<[HasNEON, UseFPVMLx]>;
def  VMLAslhd : N3VDMulOpSL16<0b01, 0b0001, IIC_VMACD, "vmla", "f16",
                            v4f16, fmul, fadd>,
                Requires<[HasNEON, HasFullFP16, UseFPVMLx]>;
def  VMLAslhq : N3VQMulOpSL16<0b01, 0b0001, IIC_VMACQ, "vmla", "f16",
                            v8f16, v4f16, fmul, fadd>,
                Requires<[HasNEON, HasFullFP16, UseFPVMLx]>;
```
- EN: Defines TableGen record `VMLAslfq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLAslfq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4521-4528
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i16 (add (v8i16 QPR:$src1),
                  (mul (v8i16 QPR:$src2),
                       (v8i16 (ARMvduplane (v8i16 QPR:$src3), imm:$lane))))),
          (v8i16 (VMLAslv8i16 (v8i16 QPR:$src1), (v8i16 QPR:$src2),
                              (v4i16 (EXTRACT_SUBREG QPR:$src3,
                                      (DSubReg_i16_reg imm:$lane))),
                              (SubReg_i16_lane imm:$lane)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4530-4537
```tablegen
def : Pat<(v4i32 (add (v4i32 QPR:$src1),
                  (mul (v4i32 QPR:$src2),
                       (v4i32 (ARMvduplane (v4i32 QPR:$src3), imm:$lane))))),
          (v4i32 (VMLAslv4i32 (v4i32 QPR:$src1), (v4i32 QPR:$src2),
                              (v2i32 (EXTRACT_SUBREG QPR:$src3,
                                      (DSubReg_i32_reg imm:$lane))),
                              (SubReg_i32_lane imm:$lane)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4539-4547
```tablegen
def : Pat<(v4f32 (fadd_mlx (v4f32 QPR:$src1),
                  (fmul_su (v4f32 QPR:$src2),
                        (v4f32 (ARMvduplane (v4f32 QPR:$src3), imm:$lane))))),
          (v4f32 (VMLAslfq (v4f32 QPR:$src1),
                           (v4f32 QPR:$src2),
                           (v2f32 (EXTRACT_SUBREG QPR:$src3,
                                   (DSubReg_i32_reg imm:$lane))),
                           (SubReg_i32_lane imm:$lane)))>,
          Requires<[HasNEON, UseFPVMLx]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4549-4553
```tablegen
//   VMLAL    : Vector Multiply Accumulate Long (Q += D * D)
defm VMLALs   : N3VLMulOp_QHS<0,1,0b1000,0, IIC_VMACi16D, IIC_VMACi32D,
                              "vmlal", "s", ARMvmulls, add>;
defm VMLALu   : N3VLMulOp_QHS<1,1,0b1000,0, IIC_VMACi16D, IIC_VMACi32D,
                              "vmlal", "u", ARMvmullu, add>;
```
- EN: Defines TableGen record `VMLALs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLALs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4555-4556
```tablegen
defm VMLALsls : N3VLMulOpSL_HS<0, 0b0010, "vmlal", "s", ARMvmulls, add>;
defm VMLALslu : N3VLMulOpSL_HS<1, 0b0010, "vmlal", "u", ARMvmullu, add>;
```
- EN: Defines TableGen record `VMLALsls` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLALsls`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4558-4575
```tablegen
let Predicates = [HasNEON, HasV8_1a] in {
  // v8.1a Neon Rounding Double Multiply-Op vector operations,
  // VQRDMLAH : Vector Saturating Rounding Doubling Multiply Accumulate Long
  //            (Q += D * D)
  defm VQRDMLAH : N3VInt3_HS<1, 0, 0b1011, 1, IIC_VMACi16D, IIC_VMACi32D,
                             IIC_VMACi16Q, IIC_VMACi32Q, "vqrdmlah", "s",
                             null_frag>;
  def : Pat<(v4i16 (int_arm_neon_vqrdmlah (v4i16 DPR:$src1), (v4i16 DPR:$Vn),
                                                   (v4i16 DPR:$Vm))),
            (v4i16 (VQRDMLAHv4i16 DPR:$src1, DPR:$Vn, DPR:$Vm))>;
  def : Pat<(v2i32 (int_arm_neon_vqrdmlah (v2i32 DPR:$src1), (v2i32 DPR:$Vn),
                                                   (v2i32 DPR:$Vm))),
            (v2i32 (VQRDMLAHv2i32 DPR:$src1, DPR:$Vn, DPR:$Vm))>;
  def : Pat<(v8i16 (int_arm_neon_vqrdmlah (v8i16 QPR:$src1), (v8i16 QPR:$Vn),
                                                   (v8i16 QPR:$Vm))),
            (v8i16 (VQRDMLAHv8i16 QPR:$src1, QPR:$Vn, QPR:$Vm))>;
  def : Pat<(v4i32 (int_arm_neon_vqrdmlah (v4i32 QPR:$src1), (v4i32 QPR:$Vn),
                                                   (v4i32 QPR:$Vm))),
```
- EN: Defines TableGen record `VQRDMLAH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQRDMLAH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4576-4576
```tablegen
            (v4i32 (VQRDMLAHv4i32 QPR:$src1, QPR:$Vn, QPR:$Vm))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4578-4595
```tablegen
  defm VQRDMLAHsl : N3VMulOpSL_HS<0b1110, IIC_VMACi16D, IIC_VMACi32D,
                                  IIC_VMACi16Q, IIC_VMACi32Q, "vqrdmlah", "s",
                                  null_frag>;
  def : Pat<(v4i16 (int_arm_neon_vqrdmlah (v4i16 DPR:$src1),
                              (v4i16 DPR:$Vn),
                              (v4i16 (ARMvduplane (v4i16 DPR_8:$Vm),
                                                   imm:$lane)))),
            (v4i16 (VQRDMLAHslv4i16 DPR:$src1, DPR:$Vn, DPR_8:$Vm,
                                    imm:$lane))>;
  def : Pat<(v2i32 (int_arm_neon_vqrdmlah (v2i32 DPR:$src1),
                              (v2i32 DPR:$Vn),
                              (v2i32 (ARMvduplane (v2i32 DPR_VFP2:$Vm),
                                                   imm:$lane)))),
            (v2i32 (VQRDMLAHslv2i32 DPR:$src1, DPR:$Vn, DPR_VFP2:$Vm,
                                    imm:$lane))>;
  def : Pat<(v8i16 (int_arm_neon_vqrdmlah (v8i16 QPR:$src1),
                              (v8i16 QPR:$src2),
                              (v8i16 (ARMvduplane (v8i16 QPR:$src3),
```
- EN: Defines TableGen record `VQRDMLAHsl` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQRDMLAHsl`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4596-4612
```tablegen
                                                   imm:$lane)))),
            (v8i16 (VQRDMLAHslv8i16 (v8i16 QPR:$src1),
                                    (v8i16 QPR:$src2),
                                    (v4i16 (EXTRACT_SUBREG
                                             QPR:$src3,
                                             (DSubReg_i16_reg imm:$lane))),
                                    (SubReg_i16_lane imm:$lane)))>;
  def : Pat<(v4i32 (int_arm_neon_vqrdmlah (v4i32 QPR:$src1),
                              (v4i32 QPR:$src2),
                              (v4i32 (ARMvduplane (v4i32 QPR:$src3),
                                                   imm:$lane)))),
            (v4i32 (VQRDMLAHslv4i32 (v4i32 QPR:$src1),
                                    (v4i32 QPR:$src2),
                                    (v2i32 (EXTRACT_SUBREG
                                             QPR:$src3,
                                             (DSubReg_i32_reg imm:$lane))),
                                    (SubReg_i32_lane imm:$lane)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4614-4630
```tablegen
  //   VQRDMLSH : Vector Saturating Rounding Doubling Multiply Subtract Long
  //              (Q -= D * D)
  defm VQRDMLSH : N3VInt3_HS<1, 0, 0b1100, 1, IIC_VMACi16D, IIC_VMACi32D,
                             IIC_VMACi16Q, IIC_VMACi32Q, "vqrdmlsh", "s",
                             null_frag>;
  def : Pat<(v4i16 (int_arm_neon_vqrdmlsh (v4i16 DPR:$src1), (v4i16 DPR:$Vn),
                                                   (v4i16 DPR:$Vm))),
            (v4i16 (VQRDMLSHv4i16 DPR:$src1, DPR:$Vn, DPR:$Vm))>;
  def : Pat<(v2i32 (int_arm_neon_vqrdmlsh (v2i32 DPR:$src1), (v2i32 DPR:$Vn),
                                                   (v2i32 DPR:$Vm))),
            (v2i32 (VQRDMLSHv2i32 DPR:$src1, DPR:$Vn, DPR:$Vm))>;
  def : Pat<(v8i16 (int_arm_neon_vqrdmlsh (v8i16 QPR:$src1), (v8i16 QPR:$Vn),
                                                   (v8i16 QPR:$Vm))),
            (v8i16 (VQRDMLSHv8i16 QPR:$src1, QPR:$Vn, QPR:$Vm))>;
  def : Pat<(v4i32 (int_arm_neon_vqrdmlsh (v4i32 QPR:$src1), (v4i32 QPR:$Vn),
                                                   (v4i32 QPR:$Vm))),
            (v4i32 (VQRDMLSHv4i32 QPR:$src1, QPR:$Vn, QPR:$Vm))>;
```
- EN: Defines TableGen record `VQRDMLSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQRDMLSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4632-4649
```tablegen
  defm VQRDMLSHsl : N3VMulOpSL_HS<0b1111, IIC_VMACi16D, IIC_VMACi32D,
                                  IIC_VMACi16Q, IIC_VMACi32Q, "vqrdmlsh", "s",
                                  null_frag>;
  def : Pat<(v4i16 (int_arm_neon_vqrdmlsh (v4i16 DPR:$src1),
                              (v4i16 DPR:$Vn),
                              (v4i16 (ARMvduplane (v4i16 DPR_8:$Vm),
                                                   imm:$lane)))),
            (v4i16 (VQRDMLSHslv4i16 DPR:$src1, DPR:$Vn, DPR_8:$Vm, imm:$lane))>;
  def : Pat<(v2i32 (int_arm_neon_vqrdmlsh (v2i32 DPR:$src1),
                              (v2i32 DPR:$Vn),
                              (v2i32 (ARMvduplane (v2i32 DPR_VFP2:$Vm),
                                                   imm:$lane)))),
            (v2i32 (VQRDMLSHslv2i32 DPR:$src1, DPR:$Vn, DPR_VFP2:$Vm,
                                    imm:$lane))>;
  def : Pat<(v8i16 (int_arm_neon_vqrdmlsh (v8i16 QPR:$src1),
                              (v8i16 QPR:$src2),
                              (v8i16 (ARMvduplane (v8i16 QPR:$src3),
                                                   imm:$lane)))),
```
- EN: Defines TableGen record `VQRDMLSHsl` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQRDMLSHsl`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4650-4667
```tablegen
            (v8i16 (VQRDMLSHslv8i16 (v8i16 QPR:$src1),
                                    (v8i16 QPR:$src2),
                                    (v4i16 (EXTRACT_SUBREG
                                             QPR:$src3,
                                             (DSubReg_i16_reg imm:$lane))),
                                    (SubReg_i16_lane imm:$lane)))>;
  def : Pat<(v4i32 (int_arm_neon_vqrdmlsh (v4i32 QPR:$src1),
                              (v4i32 QPR:$src2),
                              (v4i32 (ARMvduplane (v4i32 QPR:$src3),
                                                    imm:$lane)))),
            (v4i32 (VQRDMLSHslv4i32 (v4i32 QPR:$src1),
                                    (v4i32 QPR:$src2),
                                    (v2i32 (EXTRACT_SUBREG
                                             QPR:$src3,
                                             (DSubReg_i32_reg imm:$lane))),
                                    (SubReg_i32_lane imm:$lane)))>;
}
//   VQDMLAL  : Vector Saturating Doubling Multiply Accumulate Long (Q += D * D)
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4668-4670
```tablegen
defm VQDMLAL  : N3VLInt3_HS<0, 1, 0b1001, 0, IIC_VMACi16D, IIC_VMACi32D,
                            "vqdmlal", "s", null_frag>;
defm VQDMLALsl: N3VLInt3SL_HS<0, 0b0011, "vqdmlal", "s", null_frag>;
```
- EN: Defines TableGen record `VQDMLAL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQDMLAL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4672-4689
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v4i32 (saddsat (v4i32 QPR:$src1),
                     (v4i32 (int_arm_neon_vqdmull (v4i16 DPR:$Vn),
                                                  (v4i16 DPR:$Vm))))),
          (VQDMLALv4i32 QPR:$src1, DPR:$Vn, DPR:$Vm)>;
def : Pat<(v2i64 (saddsat (v2i64 QPR:$src1),
                     (v2i64 (int_arm_neon_vqdmull (v2i32 DPR:$Vn),
                                                  (v2i32 DPR:$Vm))))),
          (VQDMLALv2i64 QPR:$src1, DPR:$Vn, DPR:$Vm)>;
def : Pat<(v4i32 (saddsat (v4i32 QPR:$src1),
                     (v4i32 (int_arm_neon_vqdmull (v4i16 DPR:$Vn),
                                (v4i16 (ARMvduplane (v4i16 DPR_8:$Vm),
                                                     imm:$lane)))))),
          (VQDMLALslv4i16 QPR:$src1, DPR:$Vn, DPR_8:$Vm, imm:$lane)>;
def : Pat<(v2i64 (saddsat (v2i64 QPR:$src1),
                     (v2i64 (int_arm_neon_vqdmull (v2i32 DPR:$Vn),
                                (v2i32 (ARMvduplane (v2i32 DPR_VFP2:$Vm),
                                                     imm:$lane)))))),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4690-4691
```tablegen
          (VQDMLALslv2i32 QPR:$src1, DPR:$Vn, DPR_VFP2:$Vm, imm:$lane)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4693-4710
```tablegen
//   VMLS     : Vector Multiply Subtract (integer and floating-point)
defm VMLS     : N3VMulOp_QHS<1, 0, 0b1001, 0, IIC_VMACi16D, IIC_VMACi32D,
                             IIC_VMACi16Q, IIC_VMACi32Q, "vmls", "i", sub>;
def  VMLSfd   : N3VDMulOp<0, 0, 0b10, 0b1101, 1, IIC_VMACD, "vmls", "f32",
                          v2f32, fmul_su, fsub_mlx>,
                Requires<[HasNEON, UseFPVMLx]>;
def  VMLSfq   : N3VQMulOp<0, 0, 0b10, 0b1101, 1, IIC_VMACQ, "vmls", "f32",
                          v4f32, fmul_su, fsub_mlx>,
                Requires<[HasNEON, UseFPVMLx]>;
def  VMLShd   : N3VDMulOp<0, 0, 0b11, 0b1101, 1, IIC_VMACD, "vmls", "f16",
                          v4f16, fmul, fsub>,
                Requires<[HasNEON, HasFullFP16, UseFPVMLx]>;
def  VMLShq   : N3VQMulOp<0, 0, 0b11, 0b1101, 1, IIC_VMACQ, "vmls", "f16",
                          v8f16, fmul, fsub>,
                Requires<[HasNEON, HasFullFP16, UseFPVMLx]>;
defm VMLSsl   : N3VMulOpSL_HS<0b0100, IIC_VMACi16D, IIC_VMACi32D,
                              IIC_VMACi16Q, IIC_VMACi32Q, "vmls", "i", sub>;
def  VMLSslfd : N3VDMulOpSL<0b10, 0b0101, IIC_VMACD, "vmls", "f32",
```
- EN: Defines TableGen record `VMLS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4711-4721
```tablegen
                            v2f32, fmul_su, fsub_mlx>,
                Requires<[HasNEON, UseFPVMLx]>;
def  VMLSslfq : N3VQMulOpSL<0b10, 0b0101, IIC_VMACQ, "vmls", "f32",
                            v4f32, v2f32, fmul_su, fsub_mlx>,
                Requires<[HasNEON, UseFPVMLx]>;
def  VMLSslhd : N3VDMulOpSL16<0b01, 0b0101, IIC_VMACD, "vmls", "f16",
                            v4f16, fmul, fsub>,
                Requires<[HasNEON, HasFullFP16, UseFPVMLx]>;
def  VMLSslhq : N3VQMulOpSL16<0b01, 0b0101, IIC_VMACQ, "vmls", "f16",
                            v8f16, v4f16, fmul, fsub>,
                Requires<[HasNEON, HasFullFP16, UseFPVMLx]>;
```
- EN: Defines TableGen record `VMLSslfq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLSslfq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4723-4730
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i16 (sub (v8i16 QPR:$src1),
                  (mul (v8i16 QPR:$src2),
                       (v8i16 (ARMvduplane (v8i16 QPR:$src3), imm:$lane))))),
          (v8i16 (VMLSslv8i16 (v8i16 QPR:$src1), (v8i16 QPR:$src2),
                              (v4i16 (EXTRACT_SUBREG QPR:$src3,
                                      (DSubReg_i16_reg imm:$lane))),
                              (SubReg_i16_lane imm:$lane)))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4732-4739
```tablegen
def : Pat<(v4i32 (sub (v4i32 QPR:$src1),
                  (mul (v4i32 QPR:$src2),
                     (v4i32 (ARMvduplane (v4i32 QPR:$src3), imm:$lane))))),
          (v4i32 (VMLSslv4i32 (v4i32 QPR:$src1), (v4i32 QPR:$src2),
                              (v2i32 (EXTRACT_SUBREG QPR:$src3,
                                      (DSubReg_i32_reg imm:$lane))),
                              (SubReg_i32_lane imm:$lane)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4741-4748
```tablegen
def : Pat<(v4f32 (fsub_mlx (v4f32 QPR:$src1),
                  (fmul_su (v4f32 QPR:$src2),
                        (v4f32 (ARMvduplane (v4f32 QPR:$src3), imm:$lane))))),
          (v4f32 (VMLSslfq (v4f32 QPR:$src1), (v4f32 QPR:$src2),
                           (v2f32 (EXTRACT_SUBREG QPR:$src3,
                                   (DSubReg_i32_reg imm:$lane))),
                           (SubReg_i32_lane imm:$lane)))>,
          Requires<[HasNEON, UseFPVMLx]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4750-4754
```tablegen
//   VMLSL    : Vector Multiply Subtract Long (Q -= D * D)
defm VMLSLs   : N3VLMulOp_QHS<0,1,0b1010,0, IIC_VMACi16D, IIC_VMACi32D,
                              "vmlsl", "s", ARMvmulls, sub>;
defm VMLSLu   : N3VLMulOp_QHS<1,1,0b1010,0, IIC_VMACi16D, IIC_VMACi32D,
                              "vmlsl", "u", ARMvmullu, sub>;
```
- EN: Defines TableGen record `VMLSLs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLSLs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4756-4757
```tablegen
defm VMLSLsls : N3VLMulOpSL_HS<0, 0b0110, "vmlsl", "s", ARMvmulls, sub>;
defm VMLSLslu : N3VLMulOpSL_HS<1, 0b0110, "vmlsl", "u", ARMvmullu, sub>;
```
- EN: Defines TableGen record `VMLSLsls` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMLSLsls`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4759-4762
```tablegen
//   VQDMLSL  : Vector Saturating Doubling Multiply Subtract Long (Q -= D * D)
defm VQDMLSL  : N3VLInt3_HS<0, 1, 0b1011, 0, IIC_VMACi16D, IIC_VMACi32D,
                            "vqdmlsl", "s", null_frag>;
defm VQDMLSLsl: N3VLInt3SL_HS<0, 0b0111, "vqdmlsl", "s", null_frag>;
```
- EN: Defines TableGen record `VQDMLSL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQDMLSL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4764-4781
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v4i32 (ssubsat (v4i32 QPR:$src1),
                     (v4i32 (int_arm_neon_vqdmull (v4i16 DPR:$Vn),
                                                  (v4i16 DPR:$Vm))))),
          (VQDMLSLv4i32 QPR:$src1, DPR:$Vn, DPR:$Vm)>;
def : Pat<(v2i64 (ssubsat (v2i64 QPR:$src1),
                     (v2i64 (int_arm_neon_vqdmull (v2i32 DPR:$Vn),
                                                  (v2i32 DPR:$Vm))))),
          (VQDMLSLv2i64 QPR:$src1, DPR:$Vn, DPR:$Vm)>;
def : Pat<(v4i32 (ssubsat (v4i32 QPR:$src1),
                     (v4i32 (int_arm_neon_vqdmull (v4i16 DPR:$Vn),
                                (v4i16 (ARMvduplane (v4i16 DPR_8:$Vm),
                                                     imm:$lane)))))),
          (VQDMLSLslv4i16 QPR:$src1, DPR:$Vn, DPR_8:$Vm, imm:$lane)>;
def : Pat<(v2i64 (ssubsat (v2i64 QPR:$src1),
                     (v2i64 (int_arm_neon_vqdmull (v2i32 DPR:$Vn),
                                (v2i32 (ARMvduplane (v2i32 DPR_VFP2:$Vm),
                                                     imm:$lane)))))),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4782-4783
```tablegen
          (VQDMLSLslv2i32 QPR:$src1, DPR:$Vn, DPR_VFP2:$Vm, imm:$lane)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4785-4788
```tablegen
// Fused Vector Multiply-Accumulate and Fused Multiply-Subtract Operations.
def  VFMAfd   : N3VDMulOp<0, 0, 0b00, 0b1100, 1, IIC_VFMACD, "vfma", "f32",
                          v2f32, fmul_su, fadd_mlx>,
                Requires<[HasNEON,HasVFP4,UseFusedMAC]>;
```
- EN: Defines TableGen record `VFMAfd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFMAfd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4790-4795
```tablegen
def  VFMAfq   : N3VQMulOp<0, 0, 0b00, 0b1100, 1, IIC_VFMACQ, "vfma", "f32",
                          v4f32, fmul_su, fadd_mlx>,
                Requires<[HasNEON,HasVFP4,UseFusedMAC]>;
def  VFMAhd   : N3VDMulOp<0, 0, 0b01, 0b1100, 1, IIC_VFMACD, "vfma", "f16",
                          v4f16, fmul, fadd>,
                Requires<[HasNEON,HasFullFP16,UseFusedMAC]>;
```
- EN: Defines TableGen record `VFMAfq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFMAfq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4797-4799
```tablegen
def  VFMAhq   : N3VQMulOp<0, 0, 0b01, 0b1100, 1, IIC_VFMACQ, "vfma", "f16",
                          v8f16, fmul, fadd>,
                Requires<[HasNEON,HasFullFP16,UseFusedMAC]>;
```
- EN: Defines TableGen record `VFMAhq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFMAhq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4801-4813
```tablegen
//   Fused Vector Multiply Subtract (floating-point)
def  VFMSfd   : N3VDMulOp<0, 0, 0b10, 0b1100, 1, IIC_VFMACD, "vfms", "f32",
                          v2f32, fmul_su, fsub_mlx>,
                Requires<[HasNEON,HasVFP4,UseFusedMAC]>;
def  VFMSfq   : N3VQMulOp<0, 0, 0b10, 0b1100, 1, IIC_VFMACQ, "vfms", "f32",
                          v4f32, fmul_su, fsub_mlx>,
                Requires<[HasNEON,HasVFP4,UseFusedMAC]>;
def  VFMShd   : N3VDMulOp<0, 0, 0b11, 0b1100, 1, IIC_VFMACD, "vfms", "f16",
                          v4f16, fmul, fsub>,
                Requires<[HasNEON,HasFullFP16,UseFusedMAC]>;
def  VFMShq   : N3VQMulOp<0, 0, 0b11, 0b1100, 1, IIC_VFMACQ, "vfms", "f16",
                          v8f16, fmul, fsub>,
                Requires<[HasNEON,HasFullFP16,UseFusedMAC]>;
```
- EN: Defines TableGen record `VFMSfd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFMSfd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4815-4832
```tablegen
// Match @llvm.fma.* intrinsics
def : Pat<(v4f16 (fma DPR:$Vn, DPR:$Vm, DPR:$src1)),
          (VFMAhd DPR:$src1, DPR:$Vn, DPR:$Vm)>,
          Requires<[HasNEON,HasFullFP16]>;
def : Pat<(v8f16 (fma QPR:$Vn, QPR:$Vm, QPR:$src1)),
          (VFMAhq QPR:$src1, QPR:$Vn, QPR:$Vm)>,
          Requires<[HasNEON,HasFullFP16]>;
def : Pat<(v2f32 (fma DPR:$Vn, DPR:$Vm, DPR:$src1)),
          (VFMAfd DPR:$src1, DPR:$Vn, DPR:$Vm)>,
          Requires<[HasNEON,HasVFP4]>;
def : Pat<(v4f32 (fma QPR:$Vn, QPR:$Vm, QPR:$src1)),
          (VFMAfq QPR:$src1, QPR:$Vn, QPR:$Vm)>,
          Requires<[HasNEON,HasVFP4]>;
def : Pat<(v2f32 (fma (fneg DPR:$Vn), DPR:$Vm, DPR:$src1)),
          (VFMSfd DPR:$src1, DPR:$Vn, DPR:$Vm)>,
      Requires<[HasNEON,HasVFP4]>;
def : Pat<(v4f32 (fma (fneg QPR:$Vn), QPR:$Vm, QPR:$src1)),
          (VFMSfq QPR:$src1, QPR:$Vn, QPR:$Vm)>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4833-4833
```tablegen
      Requires<[HasNEON,HasVFP4]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4835-4852
```tablegen
// ARMv8.2a dot product instructions.
// We put them in the VFPV8 decoder namespace because the ARM and Thumb
// encodings are the same and thus no further bit twiddling is necessary
// in the disassembler.
class VDOT<bit op6, bit op4, bit op23, RegisterClass RegTy, string Asm,
           string AsmTy, ValueType AccumTy, ValueType InputTy,
           SDPatternOperator OpNode> :
      N3Vnp<{0b1100, op23}, 0b10, 0b1101, op6, op4, (outs RegTy:$dst),
            (ins RegTy:$Vd, RegTy:$Vn, RegTy:$Vm), N3RegFrm, IIC_VDOTPROD,
            Asm, AsmTy,
            [(set (AccumTy RegTy:$dst),
                  (OpNode (AccumTy RegTy:$Vd),
                          (InputTy RegTy:$Vn),
                          (InputTy RegTy:$Vm)))]> {
  let Predicates = [HasDotProd];
  let DecoderNamespace = "VFPV8";
  let Constraints = "$dst = $Vd";
}
```
- EN: Declares reusable TableGen class `VDOT` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VDOT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4854-4857
```tablegen
def VUDOTD : VDOT<0, 1, 0, DPR, "vudot", "u8", v2i32, v8i8,  int_arm_neon_udot>;
def VSDOTD : VDOT<0, 0, 0, DPR, "vsdot", "s8", v2i32, v8i8,  int_arm_neon_sdot>;
def VUDOTQ : VDOT<1, 1, 0, QPR, "vudot", "u8", v4i32, v16i8, int_arm_neon_udot>;
def VSDOTQ : VDOT<1, 0, 0, QPR, "vsdot", "s8", v4i32, v16i8, int_arm_neon_sdot>;
```
- EN: Defines TableGen record `VUDOTD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VUDOTD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4859-4873
```tablegen
// Indexed dot product instructions:
multiclass DOTI<string opc, string dt, bit Q, bit U, RegisterClass Ty,
           ValueType AccumType, ValueType InputType, SDPatternOperator OpNode,
           dag RHS> {
  def "" : N3Vnp<0b11100, 0b10, 0b1101, Q, U, (outs Ty:$dst),
                 (ins Ty:$Vd, Ty:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane),
                 N3RegFrm, IIC_VDOTPROD, opc, dt, []> {
    bit lane;
    let Inst{5} = lane;
    let AsmString = !strconcat(opc, ".", dt, "\t$Vd, $Vn, $Vm$lane");
    let Constraints = "$dst = $Vd";
    let Predicates = [HasDotProd];
    let DecoderNamespace = "VFPV8";
    let hasSideEffects = 0;
  }
```
- EN: Declares TableGen `multiclass DOTI`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass DOTI`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4875-4882
```tablegen
  def : Pat<
    (AccumType (OpNode (AccumType Ty:$Vd),
                       (InputType Ty:$Vn),
                       (InputType (bitconvert (AccumType
                                  (ARMvduplane (AccumType Ty:$Vm),
                                                 VectorIndex32:$lane)))))),
    (!cast<Instruction>(NAME) Ty:$Vd, Ty:$Vn, RHS, VectorIndex32:$lane)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4884-4891
```tablegen
defm VUDOTDI : DOTI<"vudot", "u8", 0b0, 0b1, DPR, v2i32, v8i8,
                    int_arm_neon_udot, (v2i32 DPR_VFP2:$Vm)>;
defm VSDOTDI : DOTI<"vsdot", "s8", 0b0, 0b0, DPR, v2i32, v8i8,
                    int_arm_neon_sdot, (v2i32 DPR_VFP2:$Vm)>;
defm VUDOTQI : DOTI<"vudot", "u8", 0b1, 0b1, QPR, v4i32, v16i8,
                    int_arm_neon_udot, (EXTRACT_SUBREG QPR:$Vm, dsub_0)>;
defm VSDOTQI : DOTI<"vsdot", "s8", 0b1, 0b0, QPR, v4i32, v16i8,
                    int_arm_neon_sdot, (EXTRACT_SUBREG QPR:$Vm, dsub_0)>;
```
- EN: Defines TableGen record `VUDOTDI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VUDOTDI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4893-4908
```tablegen
let Predicates = [HasDotProd, IsLE] in {
  def : Pat<(v4i32 (int_arm_neon_sdot (v4i32 QPR:$Vd),
                        (v16i8 QPR:$Vn),
                        (v16i8 (bitconvert (v4i32
                                    (ARMvduplane (v4i32 QPR:$Vm),
                                                  VectorIndex32_Hi:$lane)))))),
            (VSDOTQI QPR:$Vd, QPR:$Vn,
                      (EXTRACT_SUBREG QPR:$Vm, dsub_1), VectorIndex32_Hi:$lane)>;
  def : Pat<(v4i32 (int_arm_neon_udot (v4i32 QPR:$Vd),
                        (v16i8 QPR:$Vn),
                        (v16i8 (bitconvert (v4i32
                                    (ARMvduplane (v4i32 QPR:$Vm),
                                                  VectorIndex32_Hi:$lane)))))),
            (VUDOTQI QPR:$Vd, QPR:$Vn,
                      (EXTRACT_SUBREG QPR:$Vm, dsub_1), VectorIndex32_Hi:$lane)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4910-4922
```tablegen
// v8.6A matrix multiplication extension
let Predicates = [HasMatMulInt8] in {
  class N3VMatMul<bit B, bit U, string Asm, string AsmTy,
                  SDPatternOperator OpNode>
        : N3Vnp<{0b1100, B}, 0b10, 0b1100, 1, U, (outs QPR:$dst),
                (ins QPR:$Vd, QPR:$Vn, QPR:$Vm), N3RegFrm, NoItinerary,
                Asm, AsmTy,
                [(set (v4i32 QPR:$dst), (OpNode (v4i32 QPR:$Vd),
                                                (v16i8 QPR:$Vn),
                                                (v16i8 QPR:$Vm)))]> {
    let DecoderNamespace = "VFPV8";
    let Constraints = "$dst = $Vd";
  }
```
- EN: Declares reusable TableGen class `N3VMatMul` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VMatMul`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4924-4926
```tablegen
  multiclass N3VMixedDotLane<bit Q, bit U, string Asm, string AsmTy, RegisterClass RegTy,
                        ValueType AccumTy, ValueType InputTy, SDPatternOperator OpNode,
                        dag RHS> {
```
- EN: Declares TableGen `multiclass N3VMixedDotLane`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VMixedDotLane`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4928-4936
```tablegen
    def "" : N3Vnp<0b11101, 0b00, 0b1101, Q, U, (outs RegTy:$dst),
                (ins RegTy:$Vd, RegTy:$Vn, DPR_VFP2:$Vm, VectorIndex32:$lane), N3RegFrm,
                 NoItinerary, Asm, AsmTy, []> {
      bit lane;
      let Inst{5} = lane;
      let AsmString = !strconcat(Asm, ".", AsmTy, "\t$Vd, $Vn, $Vm$lane");
      let DecoderNamespace = "VFPV8";
      let Constraints = "$dst = $Vd";
    }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4938-4944
```tablegen
    def : Pat<
      (AccumTy (OpNode (AccumTy RegTy:$Vd),
                       (InputTy RegTy:$Vn),
                       (InputTy (bitconvert (AccumTy
                                (ARMvduplane (AccumTy RegTy:$Vm),
                                              VectorIndex32:$lane)))))),
      (!cast<Instruction>(NAME) RegTy:$Vd, RegTy:$Vn, RHS, VectorIndex32:$lane)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4946-4946
```tablegen
  }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4948-4957
```tablegen
  multiclass SUDOTLane<bit Q, RegisterClass RegTy, ValueType AccumTy, ValueType InputTy, dag RHS>
        : N3VMixedDotLane<Q, 1, "vsudot", "u8", RegTy, AccumTy, InputTy, null_frag, (ins)> {
    def : Pat<
      (AccumTy (int_arm_neon_usdot (AccumTy RegTy:$Vd),
                                   (InputTy (bitconvert (AccumTy
                                            (ARMvduplane (AccumTy RegTy:$Vm),
                                                          VectorIndex32:$lane)))),
                                   (InputTy RegTy:$Vn))),
      (!cast<Instruction>(NAME) RegTy:$Vd, RegTy:$Vn, RHS, VectorIndex32:$lane)>;
  }
```
- EN: Declares TableGen `multiclass SUDOTLane`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass SUDOTLane`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4959-4963
```tablegen
  def VSMMLA  : N3VMatMul<0, 0, "vsmmla",  "s8", int_arm_neon_smmla>;
  def VUMMLA  : N3VMatMul<0, 1, "vummla",  "u8", int_arm_neon_ummla>;
  def VUSMMLA : N3VMatMul<1, 0, "vusmmla", "s8", int_arm_neon_usmmla>;
  def VUSDOTD : VDOT<0, 0, 1, DPR, "vusdot", "s8", v2i32, v8i8,  int_arm_neon_usdot>;
  def VUSDOTQ : VDOT<1, 0, 1, QPR, "vusdot", "s8", v4i32, v16i8, int_arm_neon_usdot>;
```
- EN: Defines TableGen record `VSMMLA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSMMLA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4965-4971
```tablegen
  defm VUSDOTDI : N3VMixedDotLane<0, 0, "vusdot", "s8", DPR, v2i32, v8i8,
                                  int_arm_neon_usdot, (v2i32 DPR_VFP2:$Vm)>;
  defm VUSDOTQI : N3VMixedDotLane<1, 0, "vusdot", "s8", QPR, v4i32, v16i8,
                                  int_arm_neon_usdot, (EXTRACT_SUBREG QPR:$Vm, dsub_0)>;
  defm VSUDOTDI : SUDOTLane<0, DPR, v2i32, v8i8, (v2i32 DPR_VFP2:$Vm)>;
  defm VSUDOTQI : SUDOTLane<1, QPR, v4i32, v16i8, (EXTRACT_SUBREG QPR:$Vm, dsub_0)>;
}
```
- EN: Defines TableGen record `VUSDOTDI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VUSDOTDI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4973-4981
```tablegen
// ARMv8.3 complex operations
class BaseN3VCP8ComplexTied<bit op21, bit op4, bit s, bit q,
                            InstrItinClass itin, dag oops, dag iops,
                            string opc, string dt, list<dag> pattern>
  : N3VCP8<{?,?}, {op21,s}, q, op4, oops,
           iops, itin, opc, dt, "$Vd, $Vn, $Vm, $rot", "$src1 = $Vd", pattern>{
  bits<2> rot;
  let Inst{24-23} = rot;
}
```
- EN: Declares reusable TableGen class `BaseN3VCP8ComplexTied` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `BaseN3VCP8ComplexTied`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4983-4990
```tablegen
class BaseN3VCP8ComplexOdd<bit op23, bit op21, bit op4, bit s, bit q,
                           InstrItinClass itin, dag oops, dag iops, string opc,
                            string dt, list<dag> pattern>
  : N3VCP8<{?,op23}, {op21,s}, q, op4, oops,
           iops, itin, opc, dt, "$Vd, $Vn, $Vm, $rot", "", pattern> {
  bits<1> rot;
  let Inst{24} = rot;
}
```
- EN: Declares reusable TableGen class `BaseN3VCP8ComplexOdd` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `BaseN3VCP8ComplexOdd`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4992-4998
```tablegen
class BaseN3VCP8ComplexTiedLane32<bit op4, bit s, bit q, InstrItinClass itin,
                                  dag oops, dag iops, string opc, string dt,
                                  list<dag> pattern>
  : N3VLaneCP8<s, {?,?}, q, op4, oops, iops, itin, opc, dt,
               "$Vd, $Vn, $Vm$lane, $rot", "$src1 = $Vd", pattern> {
  bits<2> rot;
  bit lane;
```
- EN: Declares reusable TableGen class `BaseN3VCP8ComplexTiedLane32` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `BaseN3VCP8ComplexTiedLane32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5000-5002
```tablegen
  let Inst{21-20} = rot;
  let Inst{5} = lane;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5004-5010
```tablegen
class BaseN3VCP8ComplexTiedLane64<bit op4, bit s, bit q, InstrItinClass itin,
                            dag oops, dag iops, string opc, string dt,
                            list<dag> pattern>
  : N3VLaneCP8<s, {?,?}, q, op4, oops, iops, itin, opc, dt,
               "$Vd, $Vn, $Vm$lane, $rot", "$src1 = $Vd", pattern> {
  bits<2> rot;
  bit lane;
```
- EN: Declares reusable TableGen class `BaseN3VCP8ComplexTiedLane64` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `BaseN3VCP8ComplexTiedLane64`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5012-5018
```tablegen
  let Inst{21-20} = rot;
  let Inst{5} = Vm{4};
  // This is needed because the lane operand does not have any bits in the
  // encoding (it only has one possible value), so we need to manually set it
  // to it's default value.
  let DecoderMethod = "DecodeNEONComplexLane64Instruction";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5020-5037
```tablegen
multiclass N3VCP8ComplexTied<bit op21, bit op4,
                       string OpcodeStr> {
  let Predicates = [HasNEON,HasV8_3a,HasFullFP16] in {
  def v4f16 : BaseN3VCP8ComplexTied<op21, op4, 0, 0, IIC_VMACD, (outs DPR:$Vd),
              (ins DPR:$src1, DPR:$Vn, DPR:$Vm, complexrotateop:$rot),
              OpcodeStr, "f16", []>;
  def v8f16 : BaseN3VCP8ComplexTied<op21, op4, 0, 1, IIC_VMACQ, (outs QPR:$Vd),
              (ins QPR:$src1, QPR:$Vn, QPR:$Vm, complexrotateop:$rot),
              OpcodeStr, "f16", []>;
  }
  let Predicates = [HasNEON,HasV8_3a] in {
  def v2f32 : BaseN3VCP8ComplexTied<op21, op4, 1, 0, IIC_VMACD, (outs DPR:$Vd),
              (ins DPR:$src1, DPR:$Vn, DPR:$Vm, complexrotateop:$rot),
              OpcodeStr, "f32", []>;
  def v4f32 : BaseN3VCP8ComplexTied<op21, op4, 1, 1, IIC_VMACQ, (outs QPR:$Vd),
              (ins QPR:$src1, QPR:$Vn, QPR:$Vm, complexrotateop:$rot),
              OpcodeStr, "f32", []>;
  }
```
- EN: Declares TableGen `multiclass N3VCP8ComplexTied`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VCP8ComplexTied`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5038-5038
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5040-5057
```tablegen
multiclass N3VCP8ComplexOdd<bit op23, bit op21, bit op4,
                       string OpcodeStr> {
  let Predicates = [HasNEON,HasV8_3a,HasFullFP16] in {
  def v4f16 : BaseN3VCP8ComplexOdd<op23, op21, op4, 0, 0, IIC_VMACD,
              (outs DPR:$Vd),
              (ins DPR:$Vn, DPR:$Vm, complexrotateopodd:$rot),
              OpcodeStr, "f16", []>;
  def v8f16 : BaseN3VCP8ComplexOdd<op23, op21, op4, 0, 1, IIC_VMACQ,
              (outs QPR:$Vd),
              (ins QPR:$Vn, QPR:$Vm, complexrotateopodd:$rot),
              OpcodeStr, "f16", []>;
  }
  let Predicates = [HasNEON,HasV8_3a] in {
  def v2f32 : BaseN3VCP8ComplexOdd<op23, op21, op4, 1, 0, IIC_VMACD,
              (outs DPR:$Vd),
              (ins DPR:$Vn, DPR:$Vm, complexrotateopodd:$rot),
              OpcodeStr, "f32", []>;
  def v4f32 : BaseN3VCP8ComplexOdd<op23, op21, op4, 1, 1, IIC_VMACQ,
```
- EN: Declares TableGen `multiclass N3VCP8ComplexOdd`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VCP8ComplexOdd`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5058-5062
```tablegen
              (outs QPR:$Vd),
              (ins QPR:$Vn, QPR:$Vm, complexrotateopodd:$rot),
              OpcodeStr, "f32", []>;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5064-5081
```tablegen
// These instructions index by pairs of lanes, so the VectorIndexes are twice
// as wide as the data types.
multiclass N3VCP8ComplexTiedLane<bit op4, string OpcodeStr> {
  let Predicates = [HasNEON,HasV8_3a,HasFullFP16] in {
  def v4f16_indexed : BaseN3VCP8ComplexTiedLane32<op4, 0, 0, IIC_VMACD,
                      (outs DPR:$Vd),
                      (ins DPR:$src1, DPR:$Vn, DPR_VFP2:$Vm,
                      VectorIndex32:$lane, complexrotateop:$rot),
                      OpcodeStr, "f16", []>;
  def v8f16_indexed : BaseN3VCP8ComplexTiedLane32<op4, 0, 1, IIC_VMACQ,
                      (outs QPR:$Vd),
                      (ins QPR:$src1, QPR:$Vn, DPR_VFP2:$Vm,
                      VectorIndex32:$lane, complexrotateop:$rot),
                      OpcodeStr, "f16", []>;
  }
  let Predicates = [HasNEON,HasV8_3a] in {
  def v2f32_indexed : BaseN3VCP8ComplexTiedLane64<op4, 1, 0, IIC_VMACD,
                      (outs DPR:$Vd),
```
- EN: Declares TableGen `multiclass N3VCP8ComplexTiedLane`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass N3VCP8ComplexTiedLane`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5082-5091
```tablegen
                      (ins DPR:$src1, DPR:$Vn, DPR:$Vm, VectorIndex64:$lane,
                      complexrotateop:$rot),
                      OpcodeStr, "f32", []>;
  def v4f32_indexed : BaseN3VCP8ComplexTiedLane64<op4, 1, 1, IIC_VMACQ,
                      (outs QPR:$Vd),
                      (ins QPR:$src1, QPR:$Vn, DPR:$Vm, VectorIndex64:$lane,
                      complexrotateop:$rot),
                      OpcodeStr, "f32", []>;
  }
}
```
- EN: Defines TableGen record `v4f32_indexed` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `v4f32_indexed`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5093-5097
```tablegen
let hasSideEffects = 0 in {
defm VCMLA : N3VCP8ComplexTied<1, 0, "vcmla">;
defm VCADD : N3VCP8ComplexOdd<1, 0, 0, "vcadd">;
defm VCMLA : N3VCP8ComplexTiedLane<0, "vcmla">;
}
```
- EN: Defines TableGen record `VCMLA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCMLA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5099-5116
```tablegen
let Predicates = [HasNEON,HasV8_3a,HasFullFP16] in {
  def : Pat<(v4f16 (int_arm_neon_vcadd_rot90 (v4f16 DPR:$Rn), (v4f16 DPR:$Rm))),
            (VCADDv4f16 (v4f16 DPR:$Rn), (v4f16 DPR:$Rm), (i32 0))>;
  def : Pat<(v4f16 (int_arm_neon_vcadd_rot270 (v4f16 DPR:$Rn), (v4f16 DPR:$Rm))),
            (VCADDv4f16 (v4f16 DPR:$Rn), (v4f16 DPR:$Rm), (i32 1))>;
  def : Pat<(v8f16 (int_arm_neon_vcadd_rot90 (v8f16 QPR:$Rn), (v8f16 QPR:$Rm))),
            (VCADDv8f16 (v8f16 QPR:$Rn), (v8f16 QPR:$Rm), (i32 0))>;
  def : Pat<(v8f16 (int_arm_neon_vcadd_rot270 (v8f16 QPR:$Rn), (v8f16 QPR:$Rm))),
            (VCADDv8f16 (v8f16 QPR:$Rn), (v8f16 QPR:$Rm), (i32 1))>;
}
let Predicates = [HasNEON,HasV8_3a] in {
  def : Pat<(v2f32 (int_arm_neon_vcadd_rot90 (v2f32 DPR:$Rn), (v2f32 DPR:$Rm))),
            (VCADDv2f32 (v2f32 DPR:$Rn), (v2f32 DPR:$Rm), (i32 0))>;
  def : Pat<(v2f32 (int_arm_neon_vcadd_rot270 (v2f32 DPR:$Rn), (v2f32 DPR:$Rm))),
            (VCADDv2f32 (v2f32 DPR:$Rn), (v2f32 DPR:$Rm), (i32 1))>;
  def : Pat<(v4f32 (int_arm_neon_vcadd_rot90 (v4f32 QPR:$Rn), (v4f32 QPR:$Rm))),
            (VCADDv4f32 (v4f32 QPR:$Rn), (v4f32 QPR:$Rm), (i32 0))>;
  def : Pat<(v4f32 (int_arm_neon_vcadd_rot270 (v4f32 QPR:$Rn), (v4f32 QPR:$Rm))),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5117-5118
```tablegen
            (VCADDv4f32 (v4f32 QPR:$Rn), (v4f32 QPR:$Rm), (i32 1))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5120-5120
```tablegen
// Vector Subtract Operations.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5122-5139
```tablegen
//   VSUB     : Vector Subtract (integer and floating-point)
defm VSUB     : N3V_QHSD<1, 0, 0b1000, 0, IIC_VSUBiD, IIC_VSUBiQ,
                         "vsub", "i", sub, 0>;
def  VSUBfd   : N3VD<0, 0, 0b10, 0b1101, 0, IIC_VBIND, "vsub", "f32",
                     v2f32, v2f32, fsub, 0>;
def  VSUBfq   : N3VQ<0, 0, 0b10, 0b1101, 0, IIC_VBINQ, "vsub", "f32",
                     v4f32, v4f32, fsub, 0>;
def  VSUBhd   : N3VD<0, 0, 0b11, 0b1101, 0, IIC_VBIND, "vsub", "f16",
                     v4f16, v4f16, fsub, 0>,
                Requires<[HasNEON,HasFullFP16]>;
def  VSUBhq   : N3VQ<0, 0, 0b11, 0b1101, 0, IIC_VBINQ, "vsub", "f16",
                     v8f16, v8f16, fsub, 0>,
                Requires<[HasNEON,HasFullFP16]>;
//   VSUBL    : Vector Subtract Long (Q = D - D)
defm VSUBLs   : N3VLExt_QHS<0,1,0b0010,0, IIC_VSHLiD, IIC_VSHLiD,
                            "vsubl", "s", sub, sext, 0>;
defm VSUBLu   : N3VLExt_QHS<1,1,0b0010,0, IIC_VSHLiD, IIC_VSHLiD,
                            "vsubl", "u", sub, zanyext, 0>;
```
- EN: Defines TableGen record `VSUB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSUB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5140-5157
```tablegen
//   VSUBW    : Vector Subtract Wide (Q = Q - D)
defm VSUBWs   : N3VW_QHS<0,1,0b0011,0, "vsubw", "s", sub, sext, 0>;
defm VSUBWu   : N3VW_QHS<1,1,0b0011,0, "vsubw", "u", sub, zanyext, 0>;
//   VHSUB    : Vector Halving Subtract
defm VHSUBs   : N3VInt_QHS<0, 0, 0b0010, 0, N3RegFrm,
                           IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q, IIC_VSUBi4Q,
                           "vhsub", "s", int_arm_neon_vhsubs, 0>;
defm VHSUBu   : N3VInt_QHS<1, 0, 0b0010, 0, N3RegFrm,
                           IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q, IIC_VSUBi4Q,
                           "vhsub", "u", int_arm_neon_vhsubu, 0>;
//   VQSUB    : Vector Saturing Subtract
defm VQSUBs   : N3VInt_QHSD<0, 0, 0b0010, 1, N3RegFrm,
                            IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q, IIC_VSUBi4Q,
                            "vqsub", "s", ssubsat, 0>;
defm VQSUBu   : N3VInt_QHSD<1, 0, 0b0010, 1, N3RegFrm,
                            IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q, IIC_VSUBi4Q,
                            "vqsub", "u", usubsat, 0>;
//   VSUBHN   : Vector Subtract and Narrow Returning High Half (D = Q - Q)
```
- EN: Defines TableGen record `VSUBWs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSUBWs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5158-5161
```tablegen
defm VSUBHN   : N3VNInt_HSD<0,1,0b0110,0, "vsubhn", "i", null_frag, 0>;
//   VRSUBHN  : Vector Rounding Subtract and Narrow Returning High Half (D=Q-Q)
defm VRSUBHN  : N3VNInt_HSD<1,1,0b0110,0, "vrsubhn", "i",
                            int_arm_neon_vrsubhn, 0>;
```
- EN: Defines TableGen record `VSUBHN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSUBHN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5163-5170
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i8  (trunc (ARMvshruImm (sub (v8i16 QPR:$Vn), QPR:$Vm), 8))),
          (VSUBHNv8i8 QPR:$Vn, QPR:$Vm)>;
def : Pat<(v4i16 (trunc (ARMvshruImm (sub (v4i32 QPR:$Vn), QPR:$Vm), 16))),
          (VSUBHNv4i16 QPR:$Vn, QPR:$Vm)>;
def : Pat<(v2i32 (trunc (ARMvshruImm (sub (v2i64 QPR:$Vn), QPR:$Vm), 32))),
          (VSUBHNv2i32 QPR:$Vn, QPR:$Vm)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5172-5172
```tablegen
// Vector Comparisons.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5174-5186
```tablegen
//   VCEQ     : Vector Compare Equal
defm VCEQ     : N3V_QHS_cmp<1, 0, 0b1000, 1, IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q,
                            IIC_VSUBi4Q, "vceq", "i", ARMCCeq, 1>;
def  VCEQfd   : N3VD_cmp<0,0,0b00,0b1110,0, IIC_VBIND, "vceq", "f32", v2i32, v2f32,
                         ARMCCeq, 1>;
def  VCEQfq   : N3VQ_cmp<0,0,0b00,0b1110,0, IIC_VBINQ, "vceq", "f32", v4i32, v4f32,
                         ARMCCeq, 1>;
def  VCEQhd   : N3VD_cmp<0,0,0b01,0b1110,0, IIC_VBIND, "vceq", "f16", v4i16, v4f16,
                         ARMCCeq, 1>,
                Requires<[HasNEON, HasFullFP16]>;
def  VCEQhq   : N3VQ_cmp<0,0,0b01,0b1110,0, IIC_VBINQ, "vceq", "f16", v8i16, v8f16,
                         ARMCCeq, 1>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VCEQ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCEQ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5188-5190
```tablegen
let TwoOperandAliasConstraint = "$Vm = $Vd" in
defm VCEQz    : N2V_QHS_cmp<0b11, 0b11, 0b01, 0b00010, 0, "vceq", "i",
                            "$Vd, $Vm, #0", ARMCCeq>;
```
- EN: Defines TableGen record `VCEQz` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCEQz`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5192-5206
```tablegen
//   VCGE     : Vector Compare Greater Than or Equal
defm VCGEs    : N3V_QHS_cmp<0, 0, 0b0011, 1, IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q,
                            IIC_VSUBi4Q, "vcge", "s", ARMCCge, 0>;
defm VCGEu    : N3V_QHS_cmp<1, 0, 0b0011, 1, IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q,
                            IIC_VSUBi4Q, "vcge", "u", ARMCChs, 0>;
def  VCGEfd   : N3VD_cmp<1,0,0b00,0b1110,0, IIC_VBIND, "vcge", "f32", v2i32, v2f32,
                         ARMCCge, 0>;
def  VCGEfq   : N3VQ_cmp<1,0,0b00,0b1110,0, IIC_VBINQ, "vcge", "f32", v4i32, v4f32,
                         ARMCCge, 0>;
def  VCGEhd   : N3VD_cmp<1,0,0b01,0b1110,0, IIC_VBIND, "vcge", "f16", v4i16, v4f16,
                         ARMCCge, 0>,
                Requires<[HasNEON, HasFullFP16]>;
def  VCGEhq   : N3VQ_cmp<1,0,0b01,0b1110,0, IIC_VBINQ, "vcge", "f16", v8i16, v8f16,
                         ARMCCge, 0>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VCGEs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCGEs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5208-5213
```tablegen
let TwoOperandAliasConstraint = "$Vm = $Vd" in {
defm VCGEz    : N2V_QHS_cmp<0b11, 0b11, 0b01, 0b00001, 0, "vcge", "s",
                            "$Vd, $Vm, #0", ARMCCge>;
defm VCLEz    : N2V_QHS_cmp<0b11, 0b11, 0b01, 0b00011, 0, "vcle", "s",
                            "$Vd, $Vm, #0", ARMCCle>;
}
```
- EN: Defines TableGen record `VCGEz` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCGEz`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5215-5229
```tablegen
//   VCGT     : Vector Compare Greater Than
defm VCGTs    : N3V_QHS_cmp<0, 0, 0b0011, 0, IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q,
                            IIC_VSUBi4Q, "vcgt", "s", ARMCCgt, 0>;
defm VCGTu    : N3V_QHS_cmp<1, 0, 0b0011, 0, IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q,
                            IIC_VSUBi4Q, "vcgt", "u", ARMCChi, 0>;
def  VCGTfd   : N3VD_cmp<1,0,0b10,0b1110,0, IIC_VBIND, "vcgt", "f32", v2i32, v2f32,
                         ARMCCgt, 0>;
def  VCGTfq   : N3VQ_cmp<1,0,0b10,0b1110,0, IIC_VBINQ, "vcgt", "f32", v4i32, v4f32,
                         ARMCCgt, 0>;
def  VCGThd   : N3VD_cmp<1,0,0b11,0b1110,0, IIC_VBIND, "vcgt", "f16", v4i16, v4f16,
                         ARMCCgt, 0>,
                Requires<[HasNEON, HasFullFP16]>;
def  VCGThq   : N3VQ_cmp<1,0,0b11,0b1110,0, IIC_VBINQ, "vcgt", "f16", v8i16, v8f16,
                         ARMCCgt, 0>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VCGTs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCGTs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5231-5236
```tablegen
let TwoOperandAliasConstraint = "$Vm = $Vd" in {
defm VCGTz    : N2V_QHS_cmp<0b11, 0b11, 0b01, 0b00000, 0, "vcgt", "s",
                            "$Vd, $Vm, #0", ARMCCgt>;
defm VCLTz    : N2V_QHS_cmp<0b11, 0b11, 0b01, 0b00100, 0, "vclt", "s",
                            "$Vd, $Vm, #0", ARMCClt>;
}
```
- EN: Defines TableGen record `VCGTz` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCGTz`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5238-5255
```tablegen
//   VACGE    : Vector Absolute Compare Greater Than or Equal (aka VCAGE)
def  VACGEfd   : N3VDInt<1, 0, 0b00, 0b1110, 1, N3RegFrm, IIC_VBIND, "vacge",
                        "f32", v2i32, v2f32, int_arm_neon_vacge, 0>;
def  VACGEfq   : N3VQInt<1, 0, 0b00, 0b1110, 1, N3RegFrm, IIC_VBINQ, "vacge",
                        "f32", v4i32, v4f32, int_arm_neon_vacge, 0>;
def  VACGEhd   : N3VDInt<1, 0, 0b01, 0b1110, 1, N3RegFrm, IIC_VBIND, "vacge",
                        "f16", v4i16, v4f16, int_arm_neon_vacge, 0>,
                 Requires<[HasNEON, HasFullFP16]>;
def  VACGEhq   : N3VQInt<1, 0, 0b01, 0b1110, 1, N3RegFrm, IIC_VBINQ, "vacge",
                        "f16", v8i16, v8f16, int_arm_neon_vacge, 0>,
                 Requires<[HasNEON, HasFullFP16]>;
//   VACGT    : Vector Absolute Compare Greater Than (aka VCAGT)
def  VACGTfd   : N3VDInt<1, 0, 0b10, 0b1110, 1, N3RegFrm, IIC_VBIND, "vacgt",
                        "f32", v2i32, v2f32, int_arm_neon_vacgt, 0>;
def  VACGTfq   : N3VQInt<1, 0, 0b10, 0b1110, 1, N3RegFrm, IIC_VBINQ, "vacgt",
                        "f32", v4i32, v4f32, int_arm_neon_vacgt, 0>;
def  VACGThd   : N3VDInt<1, 0, 0b11, 0b1110, 1, N3RegFrm, IIC_VBIND, "vacgt",
                        "f16", v4i16, v4f16, int_arm_neon_vacgt, 0>,
```
- EN: Defines TableGen record `VACGEfd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VACGEfd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5256-5262
```tablegen
                 Requires<[HasNEON, HasFullFP16]>;
def  VACGThq   : N3VQInt<1, 0, 0b11, 0b1110, 1, N3RegFrm, IIC_VBINQ, "vacgt",
                        "f16", v8i16, v8f16, int_arm_neon_vacgt, 0>,
                 Requires<[HasNEON, HasFullFP16]>;
//   VTST     : Vector Test Bits
defm VTST     : N3V_QHS<0, 0, 0b1000, 1, IIC_VBINi4D, IIC_VBINi4D, IIC_VBINi4Q,
                        IIC_VBINi4Q, "vtst", "", NEONvtst, 1>;
```
- EN: Defines TableGen record `VACGThq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VACGThq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5264-5281
```tablegen
def: NEONInstAlias<"vaclt${p}.f32 $Vd, $Vn, $Vm",
                   (VACGTfd DPR:$Vd, DPR:$Vm, DPR:$Vn, pred:$p)>;
def: NEONInstAlias<"vaclt${p}.f32 $Vd, $Vn, $Vm",
                   (VACGTfq QPR:$Vd, QPR:$Vm, QPR:$Vn, pred:$p)>;
def: NEONInstAlias<"vacle${p}.f32 $Vd, $Vn, $Vm",
                   (VACGEfd DPR:$Vd, DPR:$Vm, DPR:$Vn, pred:$p)>;
def: NEONInstAlias<"vacle${p}.f32 $Vd, $Vn, $Vm",
                   (VACGEfq QPR:$Vd, QPR:$Vm, QPR:$Vn, pred:$p)>;
let Predicates = [HasNEON, HasFullFP16] in {
def: NEONInstAlias<"vaclt${p}.f16 $Vd, $Vn, $Vm",
                   (VACGThd DPR:$Vd, DPR:$Vm, DPR:$Vn, pred:$p)>;
def: NEONInstAlias<"vaclt${p}.f16 $Vd, $Vn, $Vm",
                   (VACGThq QPR:$Vd, QPR:$Vm, QPR:$Vn, pred:$p)>;
def: NEONInstAlias<"vacle${p}.f16 $Vd, $Vn, $Vm",
                   (VACGEhd DPR:$Vd, DPR:$Vm, DPR:$Vn, pred:$p)>;
def: NEONInstAlias<"vacle${p}.f16 $Vd, $Vn, $Vm",
                   (VACGEhq QPR:$Vd, QPR:$Vm, QPR:$Vn, pred:$p)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5283-5284
```tablegen
// +fp16fml Floating Point Multiplication Variants
let Predicates = [HasNEON, HasFP16FML], DecoderNamespace= "VFPV8" in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5286-5289
```tablegen
class N3VCP8F16Q1<string asm, RegisterClass Td, RegisterClass Tn,
                RegisterClass Tm, bits<2> op1, bits<2> op2, bit op3>
  : N3VCP8<op1, op2, 1, op3, (outs Td:$Vd), (ins Tn:$Vn, Tm:$Vm), NoItinerary,
           asm, "f16", "$Vd, $Vn, $Vm", "", []>;
```
- EN: Declares reusable TableGen class `N3VCP8F16Q1` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VCP8F16Q1`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5291-5294
```tablegen
class N3VCP8F16Q0<string asm, RegisterClass Td, RegisterClass Tn,
                RegisterClass Tm, bits<2> op1, bits<2> op2, bit op3>
  : N3VCP8Q0<op1, op2, 0, op3, (outs Td:$Vd), (ins Tn:$Vn, Tm:$Vm), NoItinerary,
           asm, "f16", "$Vd, $Vn, $Vm", "", []>;
```
- EN: Declares reusable TableGen class `N3VCP8F16Q0` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VCP8F16Q0`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5296-5307
```tablegen
// Vd, Vs, Vs[0-15], Idx[0-1]
class VFMD<string opc, string type, bits<2> S>
  : N3VLaneCP8<0, S, 0, 1, (outs DPR:$Vd),
               (ins SPR:$Vn, SPR_8:$Vm, VectorIndex32:$idx),
               IIC_VMACD, opc, type, "$Vd, $Vn, $Vm$idx", "", []> {
  bit idx;
  let Inst{3} = idx;
  let Inst{19-16} = Vn{4-1};
  let Inst{7}     = Vn{0};
  let Inst{5}     = Vm{0};
  let Inst{2-0}   = Vm{3-1};
}
```
- EN: Declares reusable TableGen class `VFMD` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VFMD`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5309-5317
```tablegen
// Vq, Vd, Vd[0-7], Idx[0-3]
class VFMQ<string opc, string type, bits<2> S>
  : N3VLaneCP8<0, S, 1, 1, (outs QPR:$Vd),
               (ins DPR:$Vn, DPR_8:$Vm, VectorIndex16:$idx),
               IIC_VMACD, opc, type, "$Vd, $Vn, $Vm$idx", "", []> {
  bits<2> idx;
  let Inst{5} = idx{1};
  let Inst{3} = idx{0};
}
```
- EN: Declares reusable TableGen class `VFMQ` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VFMQ`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5319-5328
```tablegen
//                                                op1   op2   op3
def VFMALD  : N3VCP8F16Q0<"vfmal", DPR, SPR, SPR, 0b00, 0b10, 1>;
def VFMSLD  : N3VCP8F16Q0<"vfmsl", DPR, SPR, SPR, 0b01, 0b10, 1>;
def VFMALQ  : N3VCP8F16Q1<"vfmal", QPR, DPR, DPR, 0b00, 0b10, 1>;
def VFMSLQ  : N3VCP8F16Q1<"vfmsl", QPR, DPR, DPR, 0b01, 0b10, 1>;
def VFMALDI : VFMD<"vfmal", "f16", 0b00>;
def VFMSLDI : VFMD<"vfmsl", "f16", 0b01>;
def VFMALQI : VFMQ<"vfmal", "f16", 0b00>;
def VFMSLQI : VFMQ<"vfmsl", "f16", 0b01>;
} // HasNEON, HasFP16FML
```
- EN: Defines TableGen record `VFMALD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VFMALD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5331-5348
```tablegen
def: NEONInstAlias<"vaclt${p}.f32 $Vd, $Vm",
                   (VACGTfd DPR:$Vd, DPR:$Vm, DPR:$Vd, pred:$p)>;
def: NEONInstAlias<"vaclt${p}.f32 $Vd, $Vm",
                   (VACGTfq QPR:$Vd, QPR:$Vm, QPR:$Vd, pred:$p)>;
def: NEONInstAlias<"vacle${p}.f32 $Vd, $Vm",
                   (VACGEfd DPR:$Vd, DPR:$Vm, DPR:$Vd, pred:$p)>;
def: NEONInstAlias<"vacle${p}.f32 $Vd, $Vm",
                   (VACGEfq QPR:$Vd, QPR:$Vm, QPR:$Vd, pred:$p)>;
let Predicates = [HasNEON, HasFullFP16] in {
def: NEONInstAlias<"vaclt${p}.f16 $Vd, $Vm",
                   (VACGThd DPR:$Vd, DPR:$Vm, DPR:$Vd, pred:$p)>;
def: NEONInstAlias<"vaclt${p}.f16 $Vd, $Vm",
                   (VACGThq QPR:$Vd, QPR:$Vm, QPR:$Vd, pred:$p)>;
def: NEONInstAlias<"vacle${p}.f16 $Vd, $Vm",
                   (VACGEhd DPR:$Vd, DPR:$Vm, DPR:$Vd, pred:$p)>;
def: NEONInstAlias<"vacle${p}.f16 $Vd, $Vm",
                   (VACGEhq QPR:$Vd, QPR:$Vm, QPR:$Vd, pred:$p)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5350-5350
```tablegen
// Vector Bitwise Operations.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5352-5355
```tablegen
def vnotd : PatFrag<(ops node:$in),
                    (xor node:$in, ARMimmAllOnesD)>;
def vnotq : PatFrag<(ops node:$in),
                    (xor node:$in, ARMimmAllOnesV)>;
```
- EN: Defines TableGen record `vnotd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `vnotd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5358-5362
```tablegen
//   VAND     : Vector Bitwise AND
def  VANDd    : N3VDX<0, 0, 0b00, 0b0001, 1, IIC_VBINiD, "vand",
                      v2i32, v2i32, and, 1>;
def  VANDq    : N3VQX<0, 0, 0b00, 0b0001, 1, IIC_VBINiQ, "vand",
                      v4i32, v4i32, and, 1>;
```
- EN: Defines TableGen record `VANDd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VANDd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5364-5368
```tablegen
//   VEOR     : Vector Bitwise Exclusive OR
def  VEORd    : N3VDX<1, 0, 0b00, 0b0001, 1, IIC_VBINiD, "veor",
                      v2i32, v2i32, xor, 1>;
def  VEORq    : N3VQX<1, 0, 0b00, 0b0001, 1, IIC_VBINiQ, "veor",
                      v4i32, v4i32, xor, 1>;
```
- EN: Defines TableGen record `VEORd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VEORd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5370-5374
```tablegen
//   VORR     : Vector Bitwise OR
def  VORRd    : N3VDX<0, 0, 0b10, 0b0001, 1, IIC_VBINiD, "vorr",
                      v2i32, v2i32, or, 1>;
def  VORRq    : N3VQX<0, 0, 0b10, 0b0001, 1, IIC_VBINiQ, "vorr",
                      v4i32, v4i32, or, 1>;
```
- EN: Defines TableGen record `VORRd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VORRd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5376-5383
```tablegen
multiclass BitwisePatterns<string Name, SDPatternOperator OpNodeD,
                           SDPatternOperator OpNodeQ> {
  def : Pat<(v8i8 (OpNodeD DPR:$LHS, DPR:$RHS)),
            (!cast<Instruction>(Name#"d") DPR:$LHS, DPR:$RHS)>;
  def : Pat<(v4i16 (OpNodeD DPR:$LHS, DPR:$RHS)),
            (!cast<Instruction>(Name#"d") DPR:$LHS, DPR:$RHS)>;
  def : Pat<(v1i64 (OpNodeD DPR:$LHS, DPR:$RHS)),
            (!cast<Instruction>(Name#"d") DPR:$LHS, DPR:$RHS)>;
```
- EN: Declares TableGen `multiclass BitwisePatterns`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass BitwisePatterns`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5385-5391
```tablegen
  def : Pat<(v16i8 (OpNodeQ QPR:$LHS, QPR:$RHS)),
            (!cast<Instruction>(Name#"q") QPR:$LHS, QPR:$RHS)>;
  def : Pat<(v8i16 (OpNodeQ QPR:$LHS, QPR:$RHS)),
            (!cast<Instruction>(Name#"q") QPR:$LHS, QPR:$RHS)>;
  def : Pat<(v2i64 (OpNodeQ QPR:$LHS, QPR:$RHS)),
            (!cast<Instruction>(Name#"q") QPR:$LHS, QPR:$RHS)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5393-5397
```tablegen
let Predicates = [HasNEON] in {
  defm : BitwisePatterns<"VAND", and, and>;
  defm : BitwisePatterns<"VORR", or, or>;
  defm : BitwisePatterns<"VEOR", xor, xor>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5399-5406
```tablegen
def VORRiv4i16 : N1ModImm<1, 0b000, {1,0,?,1}, 0, 0, 0, 1,
                          (outs DPR:$Vd), (ins nImmSplatI16:$SIMM, DPR:$src),
                          IIC_VMOVImm,
                          "vorr", "i16", "$Vd, $SIMM", "$src = $Vd",
                          [(set DPR:$Vd,
                            (v4i16 (ARMvorrImm DPR:$src, timm:$SIMM)))]> {
  let Inst{9} = SIMM{9};
}
```
- EN: Defines TableGen record `VORRiv4i16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VORRiv4i16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5408-5415
```tablegen
def VORRiv2i32 : N1ModImm<1, 0b000, {0,?,?,1}, 0, 0, 0, 1,
                          (outs DPR:$Vd), (ins nImmSplatI32:$SIMM, DPR:$src),
                          IIC_VMOVImm,
                          "vorr", "i32", "$Vd, $SIMM", "$src = $Vd",
                          [(set DPR:$Vd,
                            (v2i32 (ARMvorrImm DPR:$src, timm:$SIMM)))]> {
  let Inst{10-9} = SIMM{10-9};
}
```
- EN: Defines TableGen record `VORRiv2i32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VORRiv2i32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5417-5424
```tablegen
def VORRiv8i16 : N1ModImm<1, 0b000, {1,0,?,1}, 0, 1, 0, 1,
                          (outs QPR:$Vd), (ins nImmSplatI16:$SIMM, QPR:$src),
                          IIC_VMOVImm,
                          "vorr", "i16", "$Vd, $SIMM", "$src = $Vd",
                          [(set QPR:$Vd,
                            (v8i16 (ARMvorrImm QPR:$src, timm:$SIMM)))]> {
  let Inst{9} = SIMM{9};
}
```
- EN: Defines TableGen record `VORRiv8i16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VORRiv8i16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5426-5433
```tablegen
def VORRiv4i32 : N1ModImm<1, 0b000, {0,?,?,1}, 0, 1, 0, 1,
                          (outs QPR:$Vd), (ins nImmSplatI32:$SIMM, QPR:$src),
                          IIC_VMOVImm,
                          "vorr", "i32", "$Vd, $SIMM", "$src = $Vd",
                          [(set QPR:$Vd,
                            (v4i32 (ARMvorrImm QPR:$src, timm:$SIMM)))]> {
  let Inst{10-9} = SIMM{10-9};
}
```
- EN: Defines TableGen record `VORRiv4i32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VORRiv4i32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5436-5448
```tablegen
//   VBIC     : Vector Bitwise Bit Clear (AND NOT)
let TwoOperandAliasConstraint = "$Vn = $Vd" in {
def  VBICd    : N3VX<0, 0, 0b01, 0b0001, 0, 1, (outs DPR:$Vd),
                     (ins DPR:$Vn, DPR:$Vm), N3RegFrm, IIC_VBINiD,
                     "vbic", "$Vd, $Vn, $Vm", "",
                     [(set DPR:$Vd, (v2i32 (and DPR:$Vn,
                                                 (vnotd DPR:$Vm))))]>;
def  VBICq    : N3VX<0, 0, 0b01, 0b0001, 1, 1, (outs QPR:$Vd),
                     (ins QPR:$Vn, QPR:$Vm), N3RegFrm, IIC_VBINiQ,
                     "vbic", "$Vd, $Vn, $Vm", "",
                     [(set QPR:$Vd, (v4i32 (and QPR:$Vn,
                                                 (vnotq QPR:$Vm))))]>;
}
```
- EN: Defines TableGen record `VBICd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBICd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5450-5453
```tablegen
let Predicates = [HasNEON] in {
  defm : BitwisePatterns<"VBIC", BinOpFrag<(and node:$LHS, (vnotd node:$RHS))>,
                                 BinOpFrag<(and node:$LHS, (vnotq node:$RHS))>>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5455-5462
```tablegen
def VBICiv4i16 : N1ModImm<1, 0b000, {1,0,?,1}, 0, 0, 1, 1,
                          (outs DPR:$Vd), (ins nImmSplatI16:$SIMM, DPR:$src),
                          IIC_VMOVImm,
                          "vbic", "i16", "$Vd, $SIMM", "$src = $Vd",
                          [(set DPR:$Vd,
                            (v4i16 (ARMvbicImm DPR:$src, timm:$SIMM)))]> {
  let Inst{9} = SIMM{9};
}
```
- EN: Defines TableGen record `VBICiv4i16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBICiv4i16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5464-5471
```tablegen
def VBICiv2i32 : N1ModImm<1, 0b000, {0,?,?,1}, 0, 0, 1, 1,
                          (outs DPR:$Vd), (ins nImmSplatI32:$SIMM, DPR:$src),
                          IIC_VMOVImm,
                          "vbic", "i32", "$Vd, $SIMM", "$src = $Vd",
                          [(set DPR:$Vd,
                            (v2i32 (ARMvbicImm DPR:$src, timm:$SIMM)))]> {
  let Inst{10-9} = SIMM{10-9};
}
```
- EN: Defines TableGen record `VBICiv2i32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBICiv2i32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5473-5480
```tablegen
def VBICiv8i16 : N1ModImm<1, 0b000, {1,0,?,1}, 0, 1, 1, 1,
                          (outs QPR:$Vd), (ins nImmSplatI16:$SIMM, QPR:$src),
                          IIC_VMOVImm,
                          "vbic", "i16", "$Vd, $SIMM", "$src = $Vd",
                          [(set QPR:$Vd,
                            (v8i16 (ARMvbicImm QPR:$src, timm:$SIMM)))]> {
  let Inst{9} = SIMM{9};
}
```
- EN: Defines TableGen record `VBICiv8i16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBICiv8i16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5482-5489
```tablegen
def VBICiv4i32 : N1ModImm<1, 0b000, {0,?,?,1}, 0, 1, 1, 1,
                          (outs QPR:$Vd), (ins nImmSplatI32:$SIMM, QPR:$src),
                          IIC_VMOVImm,
                          "vbic", "i32", "$Vd, $SIMM", "$src = $Vd",
                          [(set QPR:$Vd,
                            (v4i32 (ARMvbicImm QPR:$src, timm:$SIMM)))]> {
  let Inst{10-9} = SIMM{10-9};
}
```
- EN: Defines TableGen record `VBICiv4i32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBICiv4i32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5491-5501
```tablegen
//   VORN     : Vector Bitwise OR NOT
def  VORNd    : N3VX<0, 0, 0b11, 0b0001, 0, 1, (outs DPR:$Vd),
                     (ins DPR:$Vn, DPR:$Vm), N3RegFrm, IIC_VBINiD,
                     "vorn", "$Vd, $Vn, $Vm", "",
                     [(set DPR:$Vd, (v2i32 (or DPR:$Vn,
                                                (vnotd DPR:$Vm))))]>;
def  VORNq    : N3VX<0, 0, 0b11, 0b0001, 1, 1, (outs QPR:$Vd),
                     (ins QPR:$Vn, QPR:$Vm), N3RegFrm, IIC_VBINiQ,
                     "vorn", "$Vd, $Vn, $Vm", "",
                     [(set QPR:$Vd, (v4i32 (or QPR:$Vn,
                                                (vnotq QPR:$Vm))))]>;
```
- EN: Defines TableGen record `VORNd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VORNd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5503-5506
```tablegen
let Predicates = [HasNEON] in {
  defm : BitwisePatterns<"VORN", BinOpFrag<(or node:$LHS, (vnotd node:$RHS))>,
                                 BinOpFrag<(or node:$LHS, (vnotq node:$RHS))>>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5508-5508
```tablegen
//   VMVN     : Vector Bitwise NOT (Immediate)
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5510-5510
```tablegen
let isReMaterializable = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5512-5517
```tablegen
def VMVNv4i16 : N1ModImm<1, 0b000, {1,0,?,0}, 0, 0, 1, 1, (outs DPR:$Vd),
                         (ins nImmSplatI16:$SIMM), IIC_VMOVImm,
                         "vmvn", "i16", "$Vd, $SIMM", "",
                         [(set DPR:$Vd, (v4i16 (ARMvmvnImm timm:$SIMM)))]> {
  let Inst{9} = SIMM{9};
}
```
- EN: Defines TableGen record `VMVNv4i16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMVNv4i16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5519-5524
```tablegen
def VMVNv8i16 : N1ModImm<1, 0b000, {1,0,?,0}, 0, 1, 1, 1, (outs QPR:$Vd),
                         (ins nImmSplatI16:$SIMM), IIC_VMOVImm,
                         "vmvn", "i16", "$Vd, $SIMM", "",
                         [(set QPR:$Vd, (v8i16 (ARMvmvnImm timm:$SIMM)))]> {
  let Inst{9} = SIMM{9};
}
```
- EN: Defines TableGen record `VMVNv8i16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMVNv8i16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5526-5531
```tablegen
def VMVNv2i32 : N1ModImm<1, 0b000, {?,?,?,?}, 0, 0, 1, 1, (outs DPR:$Vd),
                         (ins nImmVMOVI32:$SIMM), IIC_VMOVImm,
                         "vmvn", "i32", "$Vd, $SIMM", "",
                         [(set DPR:$Vd, (v2i32 (ARMvmvnImm timm:$SIMM)))]> {
  let Inst{11-8} = SIMM{11-8};
}
```
- EN: Defines TableGen record `VMVNv2i32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMVNv2i32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5533-5539
```tablegen
def VMVNv4i32 : N1ModImm<1, 0b000, {?,?,?,?}, 0, 1, 1, 1, (outs QPR:$Vd),
                         (ins nImmVMOVI32:$SIMM), IIC_VMOVImm,
                         "vmvn", "i32", "$Vd, $SIMM", "",
                         [(set QPR:$Vd, (v4i32 (ARMvmvnImm timm:$SIMM)))]> {
  let Inst{11-8} = SIMM{11-8};
}
}
```
- EN: Defines TableGen record `VMVNv4i32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMVNv4i32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5541-5558
```tablegen
//   VMVN     : Vector Bitwise NOT
def  VMVNd    : N2VX<0b11, 0b11, 0b00, 0b00, 0b01011, 0, 0,
                     (outs DPR:$Vd), (ins DPR:$Vm), IIC_VSUBiD,
                     "vmvn", "$Vd, $Vm", "",
                     [(set DPR:$Vd, (v2i32 (vnotd DPR:$Vm)))]>;
def  VMVNq    : N2VX<0b11, 0b11, 0b00, 0b00, 0b01011, 1, 0,
                     (outs QPR:$Vd), (ins QPR:$Vm), IIC_VSUBiD,
                     "vmvn", "$Vd, $Vm", "",
                     [(set QPR:$Vd, (v4i32 (vnotq QPR:$Vm)))]>;
let Predicates = [HasNEON] in {
def : Pat<(v1i64 (vnotd DPR:$src)),
          (VMVNd DPR:$src)>;
def : Pat<(v4i16 (vnotd DPR:$src)),
          (VMVNd DPR:$src)>;
def : Pat<(v8i8 (vnotd DPR:$src)),
          (VMVNd DPR:$src)>;
def : Pat<(v2i64 (vnotq QPR:$src)),
          (VMVNq QPR:$src)>;
```
- EN: Defines TableGen record `VMVNd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMVNd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5559-5563
```tablegen
def : Pat<(v8i16 (vnotq QPR:$src)),
          (VMVNq QPR:$src)>;
def : Pat<(v16i8 (vnotq QPR:$src)),
          (VMVNq QPR:$src)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5565-5582
```tablegen
// The TwoAddress pass will not go looking for equivalent operations
// with different register constraints; it just inserts copies.
// That is why pseudo VBSP implemented. Is is expanded later into
// VBIT/VBIF/VBSL taking into account register constraints to avoid copies.
def  VBSPd : PseudoNeonI<(outs DPR:$Vd), (ins DPR:$src1, DPR:$Vn, DPR:$Vm),
                         IIC_VBINiD, "", []>;
let Predicates = [HasNEON] in {
def : Pat<(v8i8 (NEONvbsp (v8i8 DPR:$src1),
                          (v8i8 DPR:$Vn), (v8i8 DPR:$Vm))),
          (VBSPd DPR:$src1, DPR:$Vn, DPR:$Vm)>;
def : Pat<(v4i16 (NEONvbsp (v4i16 DPR:$src1),
                           (v4i16 DPR:$Vn), (v4i16 DPR:$Vm))),
          (VBSPd DPR:$src1, DPR:$Vn, DPR:$Vm)>;
def : Pat<(v2i32 (NEONvbsp (v2i32 DPR:$src1),
                           (v2i32 DPR:$Vn), (v2i32 DPR:$Vm))),
          (VBSPd DPR:$src1, DPR:$Vn, DPR:$Vm)>;
def : Pat<(v2f32 (NEONvbsp (v2f32 DPR:$src1),
                           (v2f32 DPR:$Vn), (v2f32 DPR:$Vm))),
```
- EN: Defines TableGen record `VBSPd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBSPd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5583-5586
```tablegen
          (VBSPd DPR:$src1, DPR:$Vn, DPR:$Vm)>;
def : Pat<(v1i64 (NEONvbsp (v1i64 DPR:$src1),
                           (v1i64 DPR:$Vn), (v1i64 DPR:$Vm))),
          (VBSPd DPR:$src1, DPR:$Vn, DPR:$Vm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5588-5600
```tablegen
def : Pat<(v8i8 (or (and DPR:$Vn, DPR:$Vd),
                    (and DPR:$Vm, (vnotd DPR:$Vd)))),
          (VBSPd DPR:$Vd, DPR:$Vn, DPR:$Vm)>;
def : Pat<(v4i16 (or (and DPR:$Vn, DPR:$Vd),
                     (and DPR:$Vm, (vnotd DPR:$Vd)))),
          (VBSPd DPR:$Vd, DPR:$Vn, DPR:$Vm)>;
def : Pat<(v2i32 (or (and DPR:$Vn, DPR:$Vd),
                     (and DPR:$Vm, (vnotd DPR:$Vd)))),
          (VBSPd DPR:$Vd, DPR:$Vn, DPR:$Vm)>;
def : Pat<(v1i64 (or (and DPR:$Vn, DPR:$Vd),
                     (and DPR:$Vm, (vnotd DPR:$Vd)))),
          (VBSPd DPR:$Vd, DPR:$Vn, DPR:$Vm)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5602-5619
```tablegen
def  VBSPq : PseudoNeonI<(outs QPR:$Vd), (ins QPR:$src1, QPR:$Vn, QPR:$Vm),
                         IIC_VBINiQ, "", []>;
let Predicates = [HasNEON] in {
def : Pat<(v16i8 (NEONvbsp (v16i8 QPR:$src1),
                           (v16i8 QPR:$Vn), (v16i8 QPR:$Vm))),
          (VBSPq QPR:$src1, QPR:$Vn, QPR:$Vm)>;
def : Pat<(v8i16 (NEONvbsp (v8i16 QPR:$src1),
                           (v8i16 QPR:$Vn), (v8i16 QPR:$Vm))),
          (VBSPq QPR:$src1, QPR:$Vn, QPR:$Vm)>;
def : Pat<(v4i32 (NEONvbsp (v4i32 QPR:$src1),
                           (v4i32 QPR:$Vn), (v4i32 QPR:$Vm))),
          (VBSPq QPR:$src1, QPR:$Vn, QPR:$Vm)>;
def : Pat<(v4f32 (NEONvbsp (v4f32 QPR:$src1),
                           (v4f32 QPR:$Vn), (v4f32 QPR:$Vm))),
          (VBSPq QPR:$src1, QPR:$Vn, QPR:$Vm)>;
def : Pat<(v2i64 (NEONvbsp (v2i64 QPR:$src1),
                           (v2i64 QPR:$Vn), (v2i64 QPR:$Vm))),
          (VBSPq QPR:$src1, QPR:$Vn, QPR:$Vm)>;
```
- EN: Defines TableGen record `VBSPq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBSPq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5621-5633
```tablegen
def : Pat<(v16i8 (or (and QPR:$Vn, QPR:$Vd),
                     (and QPR:$Vm, (vnotq QPR:$Vd)))),
          (VBSPq QPR:$Vd, QPR:$Vn, QPR:$Vm)>;
def : Pat<(v8i16 (or (and QPR:$Vn, QPR:$Vd),
                     (and QPR:$Vm, (vnotq QPR:$Vd)))),
          (VBSPq QPR:$Vd, QPR:$Vn, QPR:$Vm)>;
def : Pat<(v4i32 (or (and QPR:$Vn, QPR:$Vd),
                     (and QPR:$Vm, (vnotq QPR:$Vd)))),
          (VBSPq QPR:$Vd, QPR:$Vn, QPR:$Vm)>;
def : Pat<(v2i64 (or (and QPR:$Vn, QPR:$Vd),
                     (and QPR:$Vm, (vnotq QPR:$Vd)))),
          (VBSPq QPR:$Vd, QPR:$Vn, QPR:$Vm)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5635-5635
```tablegen
let hasSideEffects = 0 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5637-5642
```tablegen
//   VBSL     : Vector Bitwise Select
def  VBSLd    : N3VX<1, 0, 0b01, 0b0001, 0, 1, (outs DPR:$Vd),
                     (ins DPR:$src1, DPR:$Vn, DPR:$Vm),
                     N3RegFrm, IIC_VBINiD,
                     "vbsl", "$Vd, $Vn, $Vm", "$src1 = $Vd",
                     []>;
```
- EN: Defines TableGen record `VBSLd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBSLd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5644-5648
```tablegen
def  VBSLq    : N3VX<1, 0, 0b01, 0b0001, 1, 1, (outs QPR:$Vd),
                     (ins QPR:$src1, QPR:$Vn, QPR:$Vm),
                     N3RegFrm, IIC_VBINiQ,
                     "vbsl", "$Vd, $Vn, $Vm", "$src1 = $Vd",
                     []>;
```
- EN: Defines TableGen record `VBSLq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBSLq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5650-5661
```tablegen
//   VBIF     : Vector Bitwise Insert if False
//              like VBSL but with: "vbif $dst, $src3, $src1", "$src2 = $dst",
def  VBIFd    : N3VX<1, 0, 0b11, 0b0001, 0, 1,
                     (outs DPR:$Vd), (ins DPR:$src1, DPR:$Vn, DPR:$Vm),
                     N3RegFrm, IIC_VBINiD,
                     "vbif", "$Vd, $Vn, $Vm", "$src1 = $Vd",
                     []>;
def  VBIFq    : N3VX<1, 0, 0b11, 0b0001, 1, 1,
                     (outs QPR:$Vd), (ins QPR:$src1, QPR:$Vn, QPR:$Vm),
                     N3RegFrm, IIC_VBINiQ,
                     "vbif", "$Vd, $Vn, $Vm", "$src1 = $Vd",
                     []>;
```
- EN: Defines TableGen record `VBIFd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBIFd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5663-5674
```tablegen
//   VBIT     : Vector Bitwise Insert if True
//              like VBSL but with: "vbit $dst, $src2, $src1", "$src3 = $dst",
def  VBITd    : N3VX<1, 0, 0b10, 0b0001, 0, 1,
                     (outs DPR:$Vd), (ins DPR:$src1, DPR:$Vn, DPR:$Vm),
                     N3RegFrm, IIC_VBINiD,
                     "vbit", "$Vd, $Vn, $Vm", "$src1 = $Vd",
                     []>;
def  VBITq    : N3VX<1, 0, 0b10, 0b0001, 1, 1,
                     (outs QPR:$Vd), (ins QPR:$src1, QPR:$Vn, QPR:$Vm),
                     N3RegFrm, IIC_VBINiQ,
                     "vbit", "$Vd, $Vn, $Vm", "$src1 = $Vd",
                     []>;
```
- EN: Defines TableGen record `VBITd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBITd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5676-5676
```tablegen
} // hasSideEffects = 0
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5678-5678
```tablegen
// Vector Absolute Differences.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5680-5696
```tablegen
//   VABD     : Vector Absolute Difference
defm VABDs    : N3VInt_QHS<0, 0, 0b0111, 0, N3RegFrm,
                           IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q, IIC_VSUBi4Q,
                           "vabd", "s", abds, 1>;
defm VABDu    : N3VInt_QHS<1, 0, 0b0111, 0, N3RegFrm,
                           IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q, IIC_VSUBi4Q,
                           "vabd", "u", abdu, 1>;
def  VABDfd   : N3VDInt<1, 0, 0b10, 0b1101, 0, N3RegFrm, IIC_VBIND,
                        "vabd", "f32", v2f32, v2f32, int_arm_neon_vabds, 1>;
def  VABDfq   : N3VQInt<1, 0, 0b10, 0b1101, 0, N3RegFrm, IIC_VBINQ,
                        "vabd", "f32", v4f32, v4f32, int_arm_neon_vabds, 1>;
def  VABDhd   : N3VDInt<1, 0, 0b11, 0b1101, 0, N3RegFrm, IIC_VBIND,
                        "vabd", "f16", v4f16, v4f16, int_arm_neon_vabds, 1>,
                Requires<[HasNEON, HasFullFP16]>;
def  VABDhq   : N3VQInt<1, 0, 0b11, 0b1101, 0, N3RegFrm, IIC_VBINQ,
                        "vabd", "f16", v8f16, v8f16, int_arm_neon_vabds, 1>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VABDs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VABDs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5698-5702
```tablegen
//   VABDL    : Vector Absolute Difference Long (Q = | D - D |)
defm VABDLs   : N3VLIntExt_QHS<0,1,0b0111,0, IIC_VSUBi4Q,
                               "vabdl", "s", abds, zext, 1>;
defm VABDLu   : N3VLIntExt_QHS<1,1,0b0111,0, IIC_VSUBi4Q,
                               "vabdl", "u", abdu, zext, 1>;
```
- EN: Defines TableGen record `VABDLs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VABDLs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5704-5711
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i16 (zext (abdu (v8i8 DPR:$opA), (v8i8 DPR:$opB)))),
          (VABDLuv8i16 DPR:$opA, DPR:$opB)>;
def : Pat<(v4i32 (zext (abdu (v4i16 DPR:$opA), (v4i16 DPR:$opB)))),
          (VABDLuv4i32 DPR:$opA, DPR:$opB)>;
def : Pat<(v2i64 (zext (abdu (v2i32 DPR:$opA), (v2i32 DPR:$opB)))),
          (VABDLuv2i64 DPR:$opA, DPR:$opB)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5713-5717
```tablegen
//   VABA     : Vector Absolute Difference and Accumulate
defm VABAs    : N3VIntOp_QHS<0,0,0b0111,1, IIC_VABAD, IIC_VABAQ,
                             "vaba", "s", abds, add>;
defm VABAu    : N3VIntOp_QHS<1,0,0b0111,1, IIC_VABAD, IIC_VABAQ,
                             "vaba", "u", abdu, add>;
```
- EN: Defines TableGen record `VABAs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VABAs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5719-5723
```tablegen
//   VABAL    : Vector Absolute Difference and Accumulate Long (Q += | D - D |)
defm VABALs   : N3VLIntExtOp_QHS<0,1,0b0101,0, IIC_VABAD,
                                 "vabal", "s", abds, zext, add>;
defm VABALu   : N3VLIntExtOp_QHS<1,1,0b0101,0, IIC_VABAD,
                                 "vabal", "u", abdu, zext, add>;
```
- EN: Defines TableGen record `VABALs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VABALs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5725-5725
```tablegen
// Vector Maximum and Minimum.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5727-5744
```tablegen
//   VMAX     : Vector Maximum
defm VMAXs    : N3VInt_QHS<0, 0, 0b0110, 0, N3RegFrm,
                           IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q, IIC_VSUBi4Q,
                           "vmax", "s", smax, 1>;
defm VMAXu    : N3VInt_QHS<1, 0, 0b0110, 0, N3RegFrm,
                           IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q, IIC_VSUBi4Q,
                           "vmax", "u", umax, 1>;
def  VMAXfd   : N3VDInt<0, 0, 0b00, 0b1111, 0, N3RegFrm, IIC_VBIND,
                        "vmax", "f32",
                        v2f32, v2f32, fmaximum, 1>;
def  VMAXfq   : N3VQInt<0, 0, 0b00, 0b1111, 0, N3RegFrm, IIC_VBINQ,
                        "vmax", "f32",
                        v4f32, v4f32, fmaximum, 1>;
def  VMAXhd   : N3VDInt<0, 0, 0b01, 0b1111, 0, N3RegFrm, IIC_VBIND,
                        "vmax", "f16",
                        v4f16, v4f16, fmaximum, 1>,
                Requires<[HasNEON, HasFullFP16]>;
def  VMAXhq   : N3VQInt<0, 0, 0b01, 0b1111, 0, N3RegFrm, IIC_VBINQ,
```
- EN: Defines TableGen record `VMAXs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMAXs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5745-5747
```tablegen
                        "vmax", "f16",
                        v8f16, v8f16, fmaximum, 1>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5749-5766
```tablegen
// VMAXNM
let PostEncoderMethod = "NEONThumb2V8PostEncoder", DecoderNamespace = "v8NEON" in {
  def NEON_VMAXNMNDf  : N3VDIntnp<0b00110, 0b00, 0b1111, 0, 1,
                                  N3RegFrm, NoItinerary, "vmaxnm", "f32",
                                  v2f32, v2f32, fmaxnum, 1>,
                                  Requires<[HasFPARMv8, HasNEON]>;
  def NEON_VMAXNMNQf  : N3VQIntnp<0b00110, 0b00, 0b1111, 1, 1,
                                  N3RegFrm, NoItinerary, "vmaxnm", "f32",
                                  v4f32, v4f32, fmaxnum, 1>,
                                  Requires<[HasFPARMv8, HasNEON]>;
  def NEON_VMAXNMNDh  : N3VDIntnp<0b00110, 0b01, 0b1111, 0, 1,
                                  N3RegFrm, NoItinerary, "vmaxnm", "f16",
                                  v4f16, v4f16, fmaxnum, 1>,
                                  Requires<[HasFPARMv8, HasNEON, HasFullFP16]>;
  def NEON_VMAXNMNQh  : N3VQIntnp<0b00110, 0b01, 0b1111, 1, 1,
                                  N3RegFrm, NoItinerary, "vmaxnm", "f16",
                                  v8f16, v8f16, fmaxnum, 1>,
                                  Requires<[HasFPARMv8, HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `NEON_VMAXNMNDf` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NEON_VMAXNMNDf`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5767-5767
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5769-5786
```tablegen
//   VMIN     : Vector Minimum
defm VMINs    : N3VInt_QHS<0, 0, 0b0110, 1, N3RegFrm,
                           IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q, IIC_VSUBi4Q,
                           "vmin", "s", smin, 1>;
defm VMINu    : N3VInt_QHS<1, 0, 0b0110, 1, N3RegFrm,
                           IIC_VSUBi4D, IIC_VSUBi4D, IIC_VSUBi4Q, IIC_VSUBi4Q,
                           "vmin", "u", umin, 1>;
def  VMINfd   : N3VDInt<0, 0, 0b10, 0b1111, 0, N3RegFrm, IIC_VBIND,
                        "vmin", "f32",
                        v2f32, v2f32, fminimum, 1>;
def  VMINfq   : N3VQInt<0, 0, 0b10, 0b1111, 0, N3RegFrm, IIC_VBINQ,
                        "vmin", "f32",
                        v4f32, v4f32, fminimum, 1>;
def  VMINhd   : N3VDInt<0, 0, 0b11, 0b1111, 0, N3RegFrm, IIC_VBIND,
                        "vmin", "f16",
                        v4f16, v4f16, fminimum, 1>,
                Requires<[HasNEON, HasFullFP16]>;
def  VMINhq   : N3VQInt<0, 0, 0b11, 0b1111, 0, N3RegFrm, IIC_VBINQ,
```
- EN: Defines TableGen record `VMINs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMINs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5787-5789
```tablegen
                        "vmin", "f16",
                        v8f16, v8f16, fminimum, 1>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5791-5808
```tablegen
// VMINNM
let PostEncoderMethod = "NEONThumb2V8PostEncoder", DecoderNamespace = "v8NEON" in {
  def NEON_VMINNMNDf  : N3VDIntnp<0b00110, 0b10, 0b1111, 0, 1,
                                  N3RegFrm, NoItinerary, "vminnm", "f32",
                                  v2f32, v2f32, fminnum, 1>,
                                  Requires<[HasFPARMv8, HasNEON]>;
  def NEON_VMINNMNQf  : N3VQIntnp<0b00110, 0b10, 0b1111, 1, 1,
                                  N3RegFrm, NoItinerary, "vminnm", "f32",
                                  v4f32, v4f32, fminnum, 1>,
                                  Requires<[HasFPARMv8, HasNEON]>;
  def NEON_VMINNMNDh  : N3VDIntnp<0b00110, 0b11, 0b1111, 0, 1,
                                  N3RegFrm, NoItinerary, "vminnm", "f16",
                                  v4f16, v4f16, fminnum, 1>,
                                  Requires<[HasFPARMv8, HasNEON, HasFullFP16]>;
  def NEON_VMINNMNQh  : N3VQIntnp<0b00110, 0b11, 0b1111, 1, 1,
                                  N3RegFrm, NoItinerary, "vminnm", "f16",
                                  v8f16, v8f16, fminnum, 1>,
                                  Requires<[HasFPARMv8, HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `NEON_VMINNMNDf` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NEON_VMINNMNDf`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5809-5809
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5811-5811
```tablegen
// Vector Pairwise Operations.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5813-5829
```tablegen
//   VPADD    : Vector Pairwise Add
def  VPADDi8  : N3VDInt<0, 0, 0b00, 0b1011, 1, N3RegFrm, IIC_VSHLiD,
                        "vpadd", "i8",
                        v8i8, v8i8, int_arm_neon_vpadd, 0>;
def  VPADDi16 : N3VDInt<0, 0, 0b01, 0b1011, 1, N3RegFrm, IIC_VSHLiD,
                        "vpadd", "i16",
                        v4i16, v4i16, int_arm_neon_vpadd, 0>;
def  VPADDi32 : N3VDInt<0, 0, 0b10, 0b1011, 1, N3RegFrm, IIC_VSHLiD,
                        "vpadd", "i32",
                        v2i32, v2i32, int_arm_neon_vpadd, 0>;
def  VPADDf   : N3VDInt<1, 0, 0b00, 0b1101, 0, N3RegFrm,
                        IIC_VPBIND, "vpadd", "f32",
                        v2f32, v2f32, int_arm_neon_vpadd, 0>;
def  VPADDh   : N3VDInt<1, 0, 0b01, 0b1101, 0, N3RegFrm,
                        IIC_VPBIND, "vpadd", "f16",
                        v4f16, v4f16, int_arm_neon_vpadd, 0>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VPADDi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VPADDi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5831-5835
```tablegen
//   VPADDL   : Vector Pairwise Add Long
defm VPADDLs  : N2VPLInt_QHS<0b11, 0b11, 0b00, 0b00100, 0, "vpaddl", "s",
                             int_arm_neon_vpaddls>;
defm VPADDLu  : N2VPLInt_QHS<0b11, 0b11, 0b00, 0b00101, 0, "vpaddl", "u",
                             int_arm_neon_vpaddlu>;
```
- EN: Defines TableGen record `VPADDLs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VPADDLs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5837-5841
```tablegen
//   VPADAL   : Vector Pairwise Add and Accumulate Long
defm VPADALs  : N2VPLInt2_QHS<0b11, 0b11, 0b00, 0b01100, 0, "vpadal", "s",
                              int_arm_neon_vpadals>;
defm VPADALu  : N2VPLInt2_QHS<0b11, 0b11, 0b00, 0b01101, 0, "vpadal", "u",
                              int_arm_neon_vpadalu>;
```
- EN: Defines TableGen record `VPADALs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VPADALs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5843-5860
```tablegen
//   VPMAX    : Vector Pairwise Maximum
def  VPMAXs8  : N3VDInt<0, 0, 0b00, 0b1010, 0, N3RegFrm, IIC_VSUBi4D, "vpmax",
                        "s8", v8i8, v8i8, int_arm_neon_vpmaxs, 0>;
def  VPMAXs16 : N3VDInt<0, 0, 0b01, 0b1010, 0, N3RegFrm, IIC_VSUBi4D, "vpmax",
                        "s16", v4i16, v4i16, int_arm_neon_vpmaxs, 0>;
def  VPMAXs32 : N3VDInt<0, 0, 0b10, 0b1010, 0, N3RegFrm, IIC_VSUBi4D, "vpmax",
                        "s32", v2i32, v2i32, int_arm_neon_vpmaxs, 0>;
def  VPMAXu8  : N3VDInt<1, 0, 0b00, 0b1010, 0, N3RegFrm, IIC_VSUBi4D, "vpmax",
                        "u8", v8i8, v8i8, int_arm_neon_vpmaxu, 0>;
def  VPMAXu16 : N3VDInt<1, 0, 0b01, 0b1010, 0, N3RegFrm, IIC_VSUBi4D, "vpmax",
                        "u16", v4i16, v4i16, int_arm_neon_vpmaxu, 0>;
def  VPMAXu32 : N3VDInt<1, 0, 0b10, 0b1010, 0, N3RegFrm, IIC_VSUBi4D, "vpmax",
                        "u32", v2i32, v2i32, int_arm_neon_vpmaxu, 0>;
def  VPMAXf   : N3VDInt<1, 0, 0b00, 0b1111, 0, N3RegFrm, IIC_VPBIND, "vpmax",
                        "f32", v2f32, v2f32, int_arm_neon_vpmaxs, 0>;
def  VPMAXh   : N3VDInt<1, 0, 0b01, 0b1111, 0, N3RegFrm, IIC_VPBIND, "vpmax",
                        "f16", v4f16, v4f16, int_arm_neon_vpmaxs, 0>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VPMAXs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VPMAXs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5862-5879
```tablegen
//   VPMIN    : Vector Pairwise Minimum
def  VPMINs8  : N3VDInt<0, 0, 0b00, 0b1010, 1, N3RegFrm, IIC_VSUBi4D, "vpmin",
                        "s8", v8i8, v8i8, int_arm_neon_vpmins, 0>;
def  VPMINs16 : N3VDInt<0, 0, 0b01, 0b1010, 1, N3RegFrm, IIC_VSUBi4D, "vpmin",
                        "s16", v4i16, v4i16, int_arm_neon_vpmins, 0>;
def  VPMINs32 : N3VDInt<0, 0, 0b10, 0b1010, 1, N3RegFrm, IIC_VSUBi4D, "vpmin",
                        "s32", v2i32, v2i32, int_arm_neon_vpmins, 0>;
def  VPMINu8  : N3VDInt<1, 0, 0b00, 0b1010, 1, N3RegFrm, IIC_VSUBi4D, "vpmin",
                        "u8", v8i8, v8i8, int_arm_neon_vpminu, 0>;
def  VPMINu16 : N3VDInt<1, 0, 0b01, 0b1010, 1, N3RegFrm, IIC_VSUBi4D, "vpmin",
                        "u16", v4i16, v4i16, int_arm_neon_vpminu, 0>;
def  VPMINu32 : N3VDInt<1, 0, 0b10, 0b1010, 1, N3RegFrm, IIC_VSUBi4D, "vpmin",
                        "u32", v2i32, v2i32, int_arm_neon_vpminu, 0>;
def  VPMINf   : N3VDInt<1, 0, 0b10, 0b1111, 0, N3RegFrm, IIC_VPBIND, "vpmin",
                        "f32", v2f32, v2f32, int_arm_neon_vpmins, 0>;
def  VPMINh   : N3VDInt<1, 0, 0b11, 0b1111, 0, N3RegFrm, IIC_VPBIND, "vpmin",
                        "f16", v4f16, v4f16, int_arm_neon_vpmins, 0>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VPMINs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VPMINs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5881-5881
```tablegen
// Vector Reciprocal and Reciprocal Square Root Estimate and Step.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5883-5900
```tablegen
//   VRECPE   : Vector Reciprocal Estimate
def  VRECPEd  : N2VDInt<0b11, 0b11, 0b10, 0b11, 0b01000, 0,
                        IIC_VUNAD, "vrecpe", "u32",
                        v2i32, v2i32, int_arm_neon_vrecpe>;
def  VRECPEq  : N2VQInt<0b11, 0b11, 0b10, 0b11, 0b01000, 0,
                        IIC_VUNAQ, "vrecpe", "u32",
                        v4i32, v4i32, int_arm_neon_vrecpe>;
def  VRECPEfd : N2VDInt<0b11, 0b11, 0b10, 0b11, 0b01010, 0,
                        IIC_VUNAD, "vrecpe", "f32",
                        v2f32, v2f32, int_arm_neon_vrecpe>;
def  VRECPEfq : N2VQInt<0b11, 0b11, 0b10, 0b11, 0b01010, 0,
                        IIC_VUNAQ, "vrecpe", "f32",
                        v4f32, v4f32, int_arm_neon_vrecpe>;
def  VRECPEhd : N2VDInt<0b11, 0b11, 0b01, 0b11, 0b01010, 0,
                        IIC_VUNAD, "vrecpe", "f16",
                        v4f16, v4f16, int_arm_neon_vrecpe>,
                Requires<[HasNEON, HasFullFP16]>;
def  VRECPEhq : N2VQInt<0b11, 0b11, 0b01, 0b11, 0b01010, 0,
```
- EN: Defines TableGen record `VRECPEd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VRECPEd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5901-5903
```tablegen
                        IIC_VUNAQ, "vrecpe", "f16",
                        v8f16, v8f16, int_arm_neon_vrecpe>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5905-5919
```tablegen
//   VRECPS   : Vector Reciprocal Step
def  VRECPSfd : N3VDInt<0, 0, 0b00, 0b1111, 1, N3RegFrm,
                        IIC_VRECSD, "vrecps", "f32",
                        v2f32, v2f32, int_arm_neon_vrecps, 1>;
def  VRECPSfq : N3VQInt<0, 0, 0b00, 0b1111, 1, N3RegFrm,
                        IIC_VRECSQ, "vrecps", "f32",
                        v4f32, v4f32, int_arm_neon_vrecps, 1>;
def  VRECPShd : N3VDInt<0, 0, 0b01, 0b1111, 1, N3RegFrm,
                        IIC_VRECSD, "vrecps", "f16",
                        v4f16, v4f16, int_arm_neon_vrecps, 1>,
                Requires<[HasNEON, HasFullFP16]>;
def  VRECPShq : N3VQInt<0, 0, 0b01, 0b1111, 1, N3RegFrm,
                        IIC_VRECSQ, "vrecps", "f16",
                        v8f16, v8f16, int_arm_neon_vrecps, 1>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VRECPSfd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VRECPSfd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5921-5938
```tablegen
//   VRSQRTE  : Vector Reciprocal Square Root Estimate
def  VRSQRTEd  : N2VDInt<0b11, 0b11, 0b10, 0b11, 0b01001, 0,
                         IIC_VUNAD, "vrsqrte", "u32",
                         v2i32, v2i32, int_arm_neon_vrsqrte>;
def  VRSQRTEq  : N2VQInt<0b11, 0b11, 0b10, 0b11, 0b01001, 0,
                         IIC_VUNAQ, "vrsqrte", "u32",
                         v4i32, v4i32, int_arm_neon_vrsqrte>;
def  VRSQRTEfd : N2VDInt<0b11, 0b11, 0b10, 0b11, 0b01011, 0,
                         IIC_VUNAD, "vrsqrte", "f32",
                         v2f32, v2f32, int_arm_neon_vrsqrte>;
def  VRSQRTEfq : N2VQInt<0b11, 0b11, 0b10, 0b11, 0b01011, 0,
                         IIC_VUNAQ, "vrsqrte", "f32",
                         v4f32, v4f32, int_arm_neon_vrsqrte>;
def  VRSQRTEhd : N2VDInt<0b11, 0b11, 0b01, 0b11, 0b01011, 0,
                         IIC_VUNAD, "vrsqrte", "f16",
                         v4f16, v4f16, int_arm_neon_vrsqrte>,
                Requires<[HasNEON, HasFullFP16]>;
def  VRSQRTEhq : N2VQInt<0b11, 0b11, 0b01, 0b11, 0b01011, 0,
```
- EN: Defines TableGen record `VRSQRTEd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VRSQRTEd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5939-5941
```tablegen
                         IIC_VUNAQ, "vrsqrte", "f16",
                         v8f16, v8f16, int_arm_neon_vrsqrte>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5943-5957
```tablegen
//   VRSQRTS  : Vector Reciprocal Square Root Step
def VRSQRTSfd : N3VDInt<0, 0, 0b10, 0b1111, 1, N3RegFrm,
                        IIC_VRECSD, "vrsqrts", "f32",
                        v2f32, v2f32, int_arm_neon_vrsqrts, 1>;
def VRSQRTSfq : N3VQInt<0, 0, 0b10, 0b1111, 1, N3RegFrm,
                        IIC_VRECSQ, "vrsqrts", "f32",
                        v4f32, v4f32, int_arm_neon_vrsqrts, 1>;
def VRSQRTShd : N3VDInt<0, 0, 0b11, 0b1111, 1, N3RegFrm,
                        IIC_VRECSD, "vrsqrts", "f16",
                        v4f16, v4f16, int_arm_neon_vrsqrts, 1>,
                Requires<[HasNEON, HasFullFP16]>;
def VRSQRTShq : N3VQInt<0, 0, 0b11, 0b1111, 1, N3RegFrm,
                        IIC_VRECSQ, "vrsqrts", "f16",
                        v8f16, v8f16, int_arm_neon_vrsqrts, 1>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VRSQRTSfd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VRSQRTSfd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5959-5959
```tablegen
// Vector Shifts.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5961-5967
```tablegen
//   VSHL     : Vector Shift
defm VSHLs    : N3VInt_QHSDSh<0, 0, 0b0100, 0, N3RegVShFrm,
                            IIC_VSHLiD, IIC_VSHLiD, IIC_VSHLiQ, IIC_VSHLiQ,
                            "vshl", "s", int_arm_neon_vshifts>;
defm VSHLu    : N3VInt_QHSDSh<1, 0, 0b0100, 0, N3RegVShFrm,
                            IIC_VSHLiD, IIC_VSHLiD, IIC_VSHLiQ, IIC_VSHLiQ,
                            "vshl", "u", int_arm_neon_vshiftu>;
```
- EN: Defines TableGen record `VSHLs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSHLs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5969-5985
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i8 (ARMvshls (v8i8 DPR:$Dn), (v8i8 DPR:$Dm))),
          (VSHLsv8i8 DPR:$Dn, DPR:$Dm)>;
def : Pat<(v4i16 (ARMvshls (v4i16 DPR:$Dn), (v4i16 DPR:$Dm))),
          (VSHLsv4i16 DPR:$Dn, DPR:$Dm)>;
def : Pat<(v2i32 (ARMvshls (v2i32 DPR:$Dn), (v2i32 DPR:$Dm))),
          (VSHLsv2i32 DPR:$Dn, DPR:$Dm)>;
def : Pat<(v1i64 (ARMvshls (v1i64 DPR:$Dn), (v1i64 DPR:$Dm))),
          (VSHLsv1i64 DPR:$Dn, DPR:$Dm)>;
def : Pat<(v16i8 (ARMvshls (v16i8 QPR:$Dn), (v16i8 QPR:$Dm))),
          (VSHLsv16i8 QPR:$Dn, QPR:$Dm)>;
def : Pat<(v8i16 (ARMvshls (v8i16 QPR:$Dn), (v8i16 QPR:$Dm))),
          (VSHLsv8i16 QPR:$Dn, QPR:$Dm)>;
def : Pat<(v4i32 (ARMvshls (v4i32 QPR:$Dn), (v4i32 QPR:$Dm))),
          (VSHLsv4i32 QPR:$Dn, QPR:$Dm)>;
def : Pat<(v2i64 (ARMvshls (v2i64 QPR:$Dn), (v2i64 QPR:$Dm))),
          (VSHLsv2i64 QPR:$Dn, QPR:$Dm)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5987-6002
```tablegen
def : Pat<(v8i8 (ARMvshlu (v8i8 DPR:$Dn), (v8i8 DPR:$Dm))),
          (VSHLuv8i8 DPR:$Dn, DPR:$Dm)>;
def : Pat<(v4i16 (ARMvshlu (v4i16 DPR:$Dn), (v4i16 DPR:$Dm))),
          (VSHLuv4i16 DPR:$Dn, DPR:$Dm)>;
def : Pat<(v2i32 (ARMvshlu (v2i32 DPR:$Dn), (v2i32 DPR:$Dm))),
          (VSHLuv2i32 DPR:$Dn, DPR:$Dm)>;
def : Pat<(v1i64 (ARMvshlu (v1i64 DPR:$Dn), (v1i64 DPR:$Dm))),
          (VSHLuv1i64 DPR:$Dn, DPR:$Dm)>;
def : Pat<(v16i8 (ARMvshlu (v16i8 QPR:$Dn), (v16i8 QPR:$Dm))),
          (VSHLuv16i8 QPR:$Dn, QPR:$Dm)>;
def : Pat<(v8i16 (ARMvshlu (v8i16 QPR:$Dn), (v8i16 QPR:$Dm))),
          (VSHLuv8i16 QPR:$Dn, QPR:$Dm)>;
def : Pat<(v4i32 (ARMvshlu (v4i32 QPR:$Dn), (v4i32 QPR:$Dm))),
          (VSHLuv4i32 QPR:$Dn, QPR:$Dm)>;
def : Pat<(v2i64 (ARMvshlu (v2i64 QPR:$Dn), (v2i64 QPR:$Dm))),
          (VSHLuv2i64 QPR:$Dn, QPR:$Dm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6004-6004
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6006-6007
```tablegen
//   VSHL     : Vector Shift Left (Immediate)
defm VSHLi    : N2VShL_QHSD<0, 1, 0b0101, 1, IIC_VSHLiD, "vshl", "i", ARMvshlImm>;
```
- EN: Defines TableGen record `VSHLi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSHLi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6009-6013
```tablegen
//   VSHR     : Vector Shift Right (Immediate)
defm VSHRs    : N2VShR_QHSD<0, 1, 0b0000, 1, IIC_VSHLiD, "vshr", "s",
                            ARMvshrsImm>;
defm VSHRu    : N2VShR_QHSD<1, 1, 0b0000, 1, IIC_VSHLiD, "vshr", "u",
                            ARMvshruImm>;
```
- EN: Defines TableGen record `VSHRs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSHRs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6015-6019
```tablegen
//   VSHLL    : Vector Shift Left Long
defm VSHLLs   : N2VLSh_QHS<0, 1, 0b1010, 0, 0, 1, "vshll", "s",
  PatFrag<(ops node:$LHS, node:$RHS), (ARMvshlImm (sext node:$LHS), node:$RHS)>>;
defm VSHLLu   : N2VLSh_QHS<1, 1, 0b1010, 0, 0, 1, "vshll", "u",
  PatFrag<(ops node:$LHS, node:$RHS), (ARMvshlImm (zext node:$LHS), node:$RHS)>>;
```
- EN: Defines TableGen record `VSHLLs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSHLLs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6021-6035
```tablegen
//   VSHLL    : Vector Shift Left Long (with maximum shift count)
class N2VLShMax<bit op24, bit op23, bits<6> op21_16, bits<4> op11_8, bit op7,
                bit op6, bit op4, string OpcodeStr, string Dt, ValueType ResTy,
                ValueType OpTy, Operand ImmTy>
  : N2VLSh<op24, op23, op11_8, op7, op6, op4, OpcodeStr, Dt,
           ResTy, OpTy, ImmTy, null_frag> {
  let Inst{21-16} = op21_16;
  let DecoderMethod = "DecodeVSHLMaxInstruction";
}
def  VSHLLi8  : N2VLShMax<1, 1, 0b110010, 0b0011, 0, 0, 0, "vshll", "i8",
                          v8i16, v8i8, imm8>;
def  VSHLLi16 : N2VLShMax<1, 1, 0b110110, 0b0011, 0, 0, 0, "vshll", "i16",
                          v4i32, v4i16, imm16>;
def  VSHLLi32 : N2VLShMax<1, 1, 0b111010, 0b0011, 0, 0, 0, "vshll", "i32",
                          v2i64, v2i32, imm32>;
```
- EN: Declares reusable TableGen class `N2VLShMax` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VLShMax`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6037-6054
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i16 (ARMvshlImm (zext (v8i8 DPR:$Rn)), (i32 8))),
          (VSHLLi8 DPR:$Rn, 8)>;
def : Pat<(v4i32 (ARMvshlImm (zext (v4i16 DPR:$Rn)), (i32 16))),
          (VSHLLi16 DPR:$Rn, 16)>;
def : Pat<(v2i64 (ARMvshlImm (zext (v2i32 DPR:$Rn)), (i32 32))),
          (VSHLLi32 DPR:$Rn, 32)>;
def : Pat<(v8i16 (ARMvshlImm (sext (v8i8 DPR:$Rn)), (i32 8))),
          (VSHLLi8 DPR:$Rn, 8)>;
def : Pat<(v4i32 (ARMvshlImm (sext (v4i16 DPR:$Rn)), (i32 16))),
          (VSHLLi16 DPR:$Rn, 16)>;
def : Pat<(v2i64 (ARMvshlImm (sext (v2i32 DPR:$Rn)), (i32 32))),
          (VSHLLi32 DPR:$Rn, 32)>;
def : Pat<(v8i16 (ARMvshlImm (anyext (v8i8 DPR:$Rn)), (i32 8))),
          (VSHLLi8 DPR:$Rn, 8)>;
def : Pat<(v4i32 (ARMvshlImm (anyext (v4i16 DPR:$Rn)), (i32 16))),
          (VSHLLi16 DPR:$Rn, 16)>;
def : Pat<(v2i64 (ARMvshlImm (anyext (v2i32 DPR:$Rn)), (i32 32))),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6055-6056
```tablegen
          (VSHLLi32 DPR:$Rn, 32)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6058-6061
```tablegen
//   VSHRN    : Vector Shift Right and Narrow
defm VSHRN    : N2VNSh_HSD<0,1,0b1000,0,0,1, IIC_VSHLiD, "vshrn", "i",
                           PatFrag<(ops node:$Rn, node:$amt),
                                   (trunc (ARMvshrsImm node:$Rn, node:$amt))>>;
```
- EN: Defines TableGen record `VSHRN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSHRN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6063-6070
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i8 (trunc (ARMvshruImm (v8i16 QPR:$Vn), shr_imm8:$amt))),
          (VSHRNv8i8 QPR:$Vn, shr_imm8:$amt)>;
def : Pat<(v4i16 (trunc (ARMvshruImm (v4i32 QPR:$Vn), shr_imm16:$amt))),
          (VSHRNv4i16 QPR:$Vn, shr_imm16:$amt)>;
def : Pat<(v2i32 (trunc (ARMvshruImm (v2i64 QPR:$Vn), shr_imm32:$amt))),
          (VSHRNv2i32 QPR:$Vn, shr_imm32:$amt)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6072-6083
```tablegen
//   VRSHL    : Vector Rounding Shift
defm VRSHLs   : N3VInt_QHSDSh<0, 0, 0b0101, 0, N3RegVShFrm,
                            IIC_VSHLi4D, IIC_VSHLi4D, IIC_VSHLi4Q, IIC_VSHLi4Q,
                            "vrshl", "s", int_arm_neon_vrshifts>;
defm VRSHLu   : N3VInt_QHSDSh<1, 0, 0b0101, 0, N3RegVShFrm,
                            IIC_VSHLi4D, IIC_VSHLi4D, IIC_VSHLi4Q, IIC_VSHLi4Q,
                            "vrshl", "u", int_arm_neon_vrshiftu>;
//   VRSHR    : Vector Rounding Shift Right
defm VRSHRs   : N2VShR_QHSD<0,1,0b0010,1, IIC_VSHLi4D, "vrshr", "s",
                            NEONvrshrsImm>;
defm VRSHRu   : N2VShR_QHSD<1,1,0b0010,1, IIC_VSHLi4D, "vrshr", "u",
                            NEONvrshruImm>;
```
- EN: Defines TableGen record `VRSHLs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VRSHLs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6085-6087
```tablegen
//   VRSHRN   : Vector Rounding Shift Right and Narrow
defm VRSHRN   : N2VNSh_HSD<0, 1, 0b1000, 0, 1, 1, IIC_VSHLi4D, "vrshrn", "i",
                           NEONvrshrnImm>;
```
- EN: Defines TableGen record `VRSHRN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VRSHRN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6089-6098
```tablegen
//   VQSHL    : Vector Saturating Shift
defm VQSHLs   : N3VInt_QHSDSh<0, 0, 0b0100, 1, N3RegVShFrm,
                            IIC_VSHLi4D, IIC_VSHLi4D, IIC_VSHLi4Q, IIC_VSHLi4Q,
                            "vqshl", "s", int_arm_neon_vqshifts>;
defm VQSHLu   : N3VInt_QHSDSh<1, 0, 0b0100, 1, N3RegVShFrm,
                            IIC_VSHLi4D, IIC_VSHLi4D, IIC_VSHLi4Q, IIC_VSHLi4Q,
                            "vqshl", "u", int_arm_neon_vqshiftu>;
//   VQSHL    : Vector Saturating Shift Left (Immediate)
defm VQSHLsi  : N2VShL_QHSD<0,1,0b0111,1, IIC_VSHLi4D, "vqshl", "s",NEONvqshlsImm>;
defm VQSHLui  : N2VShL_QHSD<1,1,0b0111,1, IIC_VSHLi4D, "vqshl", "u",NEONvqshluImm>;
```
- EN: Defines TableGen record `VQSHLs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQSHLs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6100-6101
```tablegen
//   VQSHLU   : Vector Saturating Shift Left (Immediate, Unsigned)
defm VQSHLsu  : N2VShL_QHSD<1,1,0b0110,1, IIC_VSHLi4D,"vqshlu","s",NEONvqshlsuImm>;
```
- EN: Defines TableGen record `VQSHLsu` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQSHLsu`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6103-6107
```tablegen
//   VQSHRN   : Vector Saturating Shift Right and Narrow
defm VQSHRNs  : N2VNSh_HSD<0, 1, 0b1001, 0, 0, 1, IIC_VSHLi4D, "vqshrn", "s",
                           NEONvqshrnsImm>;
defm VQSHRNu  : N2VNSh_HSD<1, 1, 0b1001, 0, 0, 1, IIC_VSHLi4D, "vqshrn", "u",
                           NEONvqshrnuImm>;
```
- EN: Defines TableGen record `VQSHRNs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQSHRNs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6109-6111
```tablegen
//   VQSHRUN  : Vector Saturating Shift Right and Narrow (Unsigned)
defm VQSHRUN  : N2VNSh_HSD<1, 1, 0b1000, 0, 0, 1, IIC_VSHLi4D, "vqshrun", "s",
                           NEONvqshrnsuImm>;
```
- EN: Defines TableGen record `VQSHRUN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQSHRUN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6113-6119
```tablegen
//   VQRSHL   : Vector Saturating Rounding Shift
defm VQRSHLs  : N3VInt_QHSDSh<0, 0, 0b0101, 1, N3RegVShFrm,
                            IIC_VSHLi4D, IIC_VSHLi4D, IIC_VSHLi4Q, IIC_VSHLi4Q,
                            "vqrshl", "s", int_arm_neon_vqrshifts>;
defm VQRSHLu  : N3VInt_QHSDSh<1, 0, 0b0101, 1, N3RegVShFrm,
                            IIC_VSHLi4D, IIC_VSHLi4D, IIC_VSHLi4Q, IIC_VSHLi4Q,
                            "vqrshl", "u", int_arm_neon_vqrshiftu>;
```
- EN: Defines TableGen record `VQRSHLs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQRSHLs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6121-6125
```tablegen
//   VQRSHRN  : Vector Saturating Rounding Shift Right and Narrow
defm VQRSHRNs : N2VNSh_HSD<0, 1, 0b1001, 0, 1, 1, IIC_VSHLi4D, "vqrshrn", "s",
                           NEONvqrshrnsImm>;
defm VQRSHRNu : N2VNSh_HSD<1, 1, 0b1001, 0, 1, 1, IIC_VSHLi4D, "vqrshrn", "u",
                           NEONvqrshrnuImm>;
```
- EN: Defines TableGen record `VQRSHRNs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQRSHRNs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6127-6129
```tablegen
//   VQRSHRUN : Vector Saturating Rounding Shift Right and Narrow (Unsigned)
defm VQRSHRUN : N2VNSh_HSD<1, 1, 0b1000, 0, 1, 1, IIC_VSHLi4D, "vqrshrun", "s",
                           NEONvqrshrnsuImm>;
```
- EN: Defines TableGen record `VQRSHRUN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQRSHRUN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6131-6136
```tablegen
//   VSRA     : Vector Shift Right and Accumulate
defm VSRAs    : N2VShAdd_QHSD<0, 1, 0b0001, 1, "vsra", "s", ARMvshrsImm>;
defm VSRAu    : N2VShAdd_QHSD<1, 1, 0b0001, 1, "vsra", "u", ARMvshruImm>;
//   VRSRA    : Vector Rounding Shift Right and Accumulate
defm VRSRAs   : N2VShAdd_QHSD<0, 1, 0b0011, 1, "vrsra", "s", NEONvrshrsImm>;
defm VRSRAu   : N2VShAdd_QHSD<1, 1, 0b0011, 1, "vrsra", "u", NEONvrshruImm>;
```
- EN: Defines TableGen record `VSRAs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSRAs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6138-6139
```tablegen
//   VSLI     : Vector Shift Left and Insert
defm VSLI     : N2VShInsL_QHSD<1, 1, 0b0101, 1, "vsli">;
```
- EN: Defines TableGen record `VSLI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSLI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6141-6142
```tablegen
//   VSRI     : Vector Shift Right and Insert
defm VSRI     : N2VShInsR_QHSD<1, 1, 0b0100, 1, "vsri">;
```
- EN: Defines TableGen record `VSRI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSRI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6144-6144
```tablegen
// Vector Absolute and Saturating Absolute.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6146-6162
```tablegen
//   VABS     : Vector Absolute Value
defm VABS     : N2VInt_QHS<0b11, 0b11, 0b01, 0b00110, 0,
                           IIC_VUNAiD, IIC_VUNAiQ, "vabs", "s", abs>;
def  VABSfd   : N2VD<0b11, 0b11, 0b10, 0b01, 0b01110, 0,
                     "vabs", "f32",
                     v2f32, v2f32, fabs>;
def  VABSfq   : N2VQ<0b11, 0b11, 0b10, 0b01, 0b01110, 0,
                     "vabs", "f32",
                      v4f32, v4f32, fabs>;
def  VABShd   : N2VD<0b11, 0b11, 0b01, 0b01, 0b01110, 0,
                     "vabs", "f16",
                     v4f16, v4f16, fabs>,
                Requires<[HasNEON, HasFullFP16]>;
def  VABShq   : N2VQ<0b11, 0b11, 0b01, 0b01, 0b01110, 0,
                     "vabs", "f16",
                      v8f16, v8f16, fabs>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VABS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VABS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6164-6167
```tablegen
//   VQABS    : Vector Saturating Absolute Value
defm VQABS    : N2VInt_QHS<0b11, 0b11, 0b00, 0b01110, 0,
                           IIC_VQUNAiD, IIC_VQUNAiQ, "vqabs", "s",
                           int_arm_neon_vqabs>;
```
- EN: Defines TableGen record `VQABS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQABS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6169-6169
```tablegen
// Vector Negate.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6171-6174
```tablegen
def vnegd  : PatFrag<(ops node:$in),
                     (sub ARMimmAllZerosD, node:$in)>;
def vnegq  : PatFrag<(ops node:$in),
                     (sub ARMimmAllZerosV, node:$in)>;
```
- EN: Defines TableGen record `vnegd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `vnegd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6176-6183
```tablegen
class VNEGD<bits<2> size, string OpcodeStr, string Dt, ValueType Ty>
  : N2V<0b11, 0b11, size, 0b01, 0b00111, 0, 0, (outs DPR:$Vd), (ins DPR:$Vm),
        IIC_VSHLiD, OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set DPR:$Vd, (Ty (vnegd DPR:$Vm)))]>;
class VNEGQ<bits<2> size, string OpcodeStr, string Dt, ValueType Ty>
  : N2V<0b11, 0b11, size, 0b01, 0b00111, 1, 0, (outs QPR:$Vd), (ins QPR:$Vm),
        IIC_VSHLiQ, OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set QPR:$Vd, (Ty (vnegq QPR:$Vm)))]>;
```
- EN: Declares reusable TableGen class `VNEGD` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VNEGD`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6185-6191
```tablegen
//   VNEG     : Vector Negate (integer)
def  VNEGs8d  : VNEGD<0b00, "vneg", "s8", v8i8>;
def  VNEGs16d : VNEGD<0b01, "vneg", "s16", v4i16>;
def  VNEGs32d : VNEGD<0b10, "vneg", "s32", v2i32>;
def  VNEGs8q  : VNEGQ<0b00, "vneg", "s8", v16i8>;
def  VNEGs16q : VNEGQ<0b01, "vneg", "s16", v8i16>;
def  VNEGs32q : VNEGQ<0b10, "vneg", "s32", v4i32>;
```
- EN: Defines TableGen record `VNEGs8d` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNEGs8d`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6193-6210
```tablegen
//   VNEG     : Vector Negate (floating-point)
def  VNEGfd   : N2V<0b11, 0b11, 0b10, 0b01, 0b01111, 0, 0,
                    (outs DPR:$Vd), (ins DPR:$Vm), IIC_VUNAD,
                    "vneg", "f32", "$Vd, $Vm", "",
                    [(set DPR:$Vd, (v2f32 (fneg DPR:$Vm)))]>;
def  VNEGf32q : N2V<0b11, 0b11, 0b10, 0b01, 0b01111, 1, 0,
                    (outs QPR:$Vd), (ins QPR:$Vm), IIC_VUNAQ,
                    "vneg", "f32", "$Vd, $Vm", "",
                    [(set QPR:$Vd, (v4f32 (fneg QPR:$Vm)))]>;
def  VNEGhd   : N2V<0b11, 0b11, 0b01, 0b01, 0b01111, 0, 0,
                    (outs DPR:$Vd), (ins DPR:$Vm), IIC_VUNAD,
                    "vneg", "f16", "$Vd, $Vm", "",
                    [(set DPR:$Vd, (v4f16 (fneg DPR:$Vm)))]>,
                Requires<[HasNEON, HasFullFP16]>;
def  VNEGhq   : N2V<0b11, 0b11, 0b01, 0b01, 0b01111, 1, 0,
                    (outs QPR:$Vd), (ins QPR:$Vm), IIC_VUNAQ,
                    "vneg", "f16", "$Vd, $Vm", "",
                    [(set QPR:$Vd, (v8f16 (fneg QPR:$Vm)))]>,
```
- EN: Defines TableGen record `VNEGfd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VNEGfd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6211-6211
```tablegen
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6213-6220
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i8  (vnegd  DPR:$src)), (VNEGs8d DPR:$src)>;
def : Pat<(v4i16 (vnegd  DPR:$src)), (VNEGs16d DPR:$src)>;
def : Pat<(v2i32 (vnegd  DPR:$src)), (VNEGs32d DPR:$src)>;
def : Pat<(v16i8 (vnegq QPR:$src)), (VNEGs8q QPR:$src)>;
def : Pat<(v8i16 (vnegq QPR:$src)), (VNEGs16q QPR:$src)>;
def : Pat<(v4i32 (vnegq QPR:$src)), (VNEGs32q QPR:$src)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6222-6225
```tablegen
//   VQNEG    : Vector Saturating Negate
defm VQNEG    : N2VInt_QHS<0b11, 0b11, 0b00, 0b01111, 0,
                           IIC_VQUNAiD, IIC_VQUNAiQ, "vqneg", "s",
                           int_arm_neon_vqneg>;
```
- EN: Defines TableGen record `VQNEG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VQNEG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6227-6227
```tablegen
// Vector Bit Counting Operations.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6229-6243
```tablegen
//   VCLS     : Vector Count Leading Sign Bits
defm VCLS     : N2VInt_QHS<0b11, 0b11, 0b00, 0b01000, 0,
                           IIC_VCNTiD, IIC_VCNTiQ, "vcls", "s",
                           ctls>;
//   VCLZ     : Vector Count Leading Zeros
defm VCLZ     : N2VInt_QHS<0b11, 0b11, 0b00, 0b01001, 0,
                           IIC_VCNTiD, IIC_VCNTiQ, "vclz", "i",
                           ctlz>;
//   VCNT     : Vector Count One Bits
def  VCNTd    : N2VDInt<0b11, 0b11, 0b00, 0b00, 0b01010, 0,
                        IIC_VCNTiD, "vcnt", "8",
                        v8i8, v8i8, ctpop>;
def  VCNTq    : N2VQInt<0b11, 0b11, 0b00, 0b00, 0b01010, 0,
                        IIC_VCNTiQ, "vcnt", "8",
                        v16i8, v16i8, ctpop>;
```
- EN: Defines TableGen record `VCLS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCLS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6245-6253
```tablegen
// Vector Swap
def  VSWPd    : N2VX<0b11, 0b11, 0b00, 0b10, 0b00000, 0, 0,
                     (outs DPR:$Vd, DPR:$Vm), (ins DPR:$in1, DPR:$in2),
                     NoItinerary, "vswp", "$Vd, $Vm", "$in1 = $Vd, $in2 = $Vm",
                     []>;
def  VSWPq    : N2VX<0b11, 0b11, 0b00, 0b10, 0b00000, 1, 0,
                     (outs QPR:$Vd, QPR:$Vm), (ins QPR:$in1, QPR:$in2),
                     NoItinerary, "vswp", "$Vd, $Vm", "$in1 = $Vd, $in2 = $Vm",
                     []>;
```
- EN: Defines TableGen record `VSWPd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSWPd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6255-6255
```tablegen
// Vector Move Operations.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6257-6261
```tablegen
//   VMOV     : Vector Move (Register)
def : NEONInstAlias<"vmov${p} $Vd, $Vm",
                    (VORRd DPR:$Vd, DPR:$Vm, DPR:$Vm, pred:$p)>;
def : NEONInstAlias<"vmov${p} $Vd, $Vm",
                    (VORRq QPR:$Vd, QPR:$Vm, QPR:$Vm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6263-6263
```tablegen
//   VMOV     : Vector Move (Immediate)
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6265-6276
```tablegen
// Although VMOVs are not strictly speaking cheap, they are as expensive
// as their copies counterpart (VORR), so we should prefer rematerialization
// over splitting when it applies.
let isReMaterializable = 1, isAsCheapAsAMove=1 in {
def VMOVv8i8  : N1ModImm<1, 0b000, 0b1110, 0, 0, 0, 1, (outs DPR:$Vd),
                         (ins nImmSplatI8:$SIMM), IIC_VMOVImm,
                         "vmov", "i8", "$Vd, $SIMM", "",
                         [(set DPR:$Vd, (v8i8 (ARMvmovImm timm:$SIMM)))]>;
def VMOVv16i8 : N1ModImm<1, 0b000, 0b1110, 0, 1, 0, 1, (outs QPR:$Vd),
                         (ins nImmSplatI8:$SIMM), IIC_VMOVImm,
                         "vmov", "i8", "$Vd, $SIMM", "",
                         [(set QPR:$Vd, (v16i8 (ARMvmovImm timm:$SIMM)))]>;
```
- EN: Defines TableGen record `VMOVv8i8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVv8i8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6278-6283
```tablegen
def VMOVv4i16 : N1ModImm<1, 0b000, {1,0,?,0}, 0, 0, 0, 1, (outs DPR:$Vd),
                         (ins nImmSplatI16:$SIMM), IIC_VMOVImm,
                         "vmov", "i16", "$Vd, $SIMM", "",
                         [(set DPR:$Vd, (v4i16 (ARMvmovImm timm:$SIMM)))]> {
  let Inst{9} = SIMM{9};
}
```
- EN: Defines TableGen record `VMOVv4i16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVv4i16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6285-6290
```tablegen
def VMOVv8i16 : N1ModImm<1, 0b000, {1,0,?,0}, 0, 1, 0, 1, (outs QPR:$Vd),
                         (ins nImmSplatI16:$SIMM), IIC_VMOVImm,
                         "vmov", "i16", "$Vd, $SIMM", "",
                         [(set QPR:$Vd, (v8i16 (ARMvmovImm timm:$SIMM)))]> {
 let Inst{9} = SIMM{9};
}
```
- EN: Defines TableGen record `VMOVv8i16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVv8i16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6292-6297
```tablegen
def VMOVv2i32 : N1ModImm<1, 0b000, {?,?,?,?}, 0, 0, 0, 1, (outs DPR:$Vd),
                         (ins nImmVMOVI32:$SIMM), IIC_VMOVImm,
                         "vmov", "i32", "$Vd, $SIMM", "",
                         [(set DPR:$Vd, (v2i32 (ARMvmovImm timm:$SIMM)))]> {
  let Inst{11-8} = SIMM{11-8};
}
```
- EN: Defines TableGen record `VMOVv2i32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVv2i32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6299-6304
```tablegen
def VMOVv4i32 : N1ModImm<1, 0b000, {?,?,?,?}, 0, 1, 0, 1, (outs QPR:$Vd),
                         (ins nImmVMOVI32:$SIMM), IIC_VMOVImm,
                         "vmov", "i32", "$Vd, $SIMM", "",
                         [(set QPR:$Vd, (v4i32 (ARMvmovImm timm:$SIMM)))]> {
  let Inst{11-8} = SIMM{11-8};
}
```
- EN: Defines TableGen record `VMOVv4i32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVv4i32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6306-6313
```tablegen
def VMOVv1i64 : N1ModImm<1, 0b000, 0b1110, 0, 0, 1, 1, (outs DPR:$Vd),
                         (ins nImmSplatI64:$SIMM), IIC_VMOVImm,
                         "vmov", "i64", "$Vd, $SIMM", "",
                         [(set DPR:$Vd, (v1i64 (ARMvmovImm timm:$SIMM)))]>;
def VMOVv2i64 : N1ModImm<1, 0b000, 0b1110, 0, 1, 1, 1, (outs QPR:$Vd),
                         (ins nImmSplatI64:$SIMM), IIC_VMOVImm,
                         "vmov", "i64", "$Vd, $SIMM", "",
                         [(set QPR:$Vd, (v2i64 (ARMvmovImm timm:$SIMM)))]>;
```
- EN: Defines TableGen record `VMOVv1i64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVv1i64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6315-6323
```tablegen
def VMOVv2f32 : N1ModImm<1, 0b000, 0b1111, 0, 0, 0, 1, (outs DPR:$Vd),
                         (ins nImmVMOVF32:$SIMM), IIC_VMOVImm,
                         "vmov", "f32", "$Vd, $SIMM", "",
                         [(set DPR:$Vd, (v2f32 (ARMvmovFPImm timm:$SIMM)))]>;
def VMOVv4f32 : N1ModImm<1, 0b000, 0b1111, 0, 1, 0, 1, (outs QPR:$Vd),
                         (ins nImmVMOVF32:$SIMM), IIC_VMOVImm,
                         "vmov", "f32", "$Vd, $SIMM", "",
                         [(set QPR:$Vd, (v4f32 (ARMvmovFPImm timm:$SIMM)))]>;
} // isReMaterializable, isAsCheapAsAMove
```
- EN: Defines TableGen record `VMOVv2f32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVv2f32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6325-6342
```tablegen
// Add support for bytes replication feature, so it could be GAS compatible.
multiclass NEONImmReplicateI8InstAlias<ValueType To> {
  // E.g. instructions below:
  // "vmov.i32 d0, #0xffffffff"
  // "vmov.i32 d0, #0xabababab"
  // "vmov.i16 d0, #0xabab"
  // are incorrect, but we could deal with such cases.
  // For last two instructions, for example, it should emit:
  // "vmov.i8 d0, #0xab"
  def : NEONInstAlias<"vmov${p}.i" # To.Size # " $Vd, $Vm",
                      (VMOVv8i8 DPR:$Vd, nImmVMOVIReplicate<i8, To>:$Vm, pred:$p)>;
  def : NEONInstAlias<"vmov${p}.i" # To.Size # " $Vd, $Vm",
                      (VMOVv16i8 QPR:$Vd, nImmVMOVIReplicate<i8, To>:$Vm, pred:$p)>;
  // Also add same support for VMVN instructions. So instruction:
  // "vmvn.i32 d0, #0xabababab"
  // actually means:
  // "vmov.i8 d0, #0x54"
  def : NEONInstAlias<"vmvn${p}.i" # To.Size # " $Vd, $Vm",
```
- EN: Declares TableGen `multiclass NEONImmReplicateI8InstAlias`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass NEONImmReplicateI8InstAlias`，它是一个可复用模板，可展开为多个相关记录。

### Lines 6343-6346
```tablegen
                      (VMOVv8i8 DPR:$Vd, nImmVINVIReplicate<i8, To>:$Vm, pred:$p)>;
  def : NEONInstAlias<"vmvn${p}.i" # To.Size # " $Vd, $Vm",
                      (VMOVv16i8 QPR:$Vd, nImmVINVIReplicate<i8, To>:$Vm, pred:$p)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6348-6350
```tablegen
defm : NEONImmReplicateI8InstAlias<i16>;
defm : NEONImmReplicateI8InstAlias<i32>;
defm : NEONImmReplicateI8InstAlias<i64>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6352-6366
```tablegen
// Similar to above for types other than i8, e.g.:
// "vmov.i32 d0, #0xab00ab00" -> "vmov.i16 d0, #0xab00"
// "vmvn.i64 q0, #0xab000000ab000000" -> "vmvn.i32 q0, #0xab000000"
// In this case we do not canonicalize VMVN to VMOV
multiclass NEONImmReplicateInstAlias<ValueType From, NeonI V8, NeonI V16,
                                     NeonI NV8, NeonI NV16, ValueType To> {
  def : NEONInstAlias<"vmov${p}.i" # To.Size # " $Vd, $Vm",
                      (V8 DPR:$Vd, nImmVMOVIReplicate<From, To>:$Vm, pred:$p)>;
  def : NEONInstAlias<"vmov${p}.i" # To.Size # " $Vd, $Vm",
                      (V16 QPR:$Vd, nImmVMOVIReplicate<From, To>:$Vm, pred:$p)>;
  def : NEONInstAlias<"vmvn${p}.i" # To.Size # " $Vd, $Vm",
                      (NV8 DPR:$Vd, nImmVMOVIReplicate<From, To>:$Vm, pred:$p)>;
  def : NEONInstAlias<"vmvn${p}.i" # To.Size # " $Vd, $Vm",
                      (NV16 QPR:$Vd, nImmVMOVIReplicate<From, To>:$Vm, pred:$p)>;
}
```
- EN: Declares TableGen `multiclass NEONImmReplicateInstAlias`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass NEONImmReplicateInstAlias`，它是一个可复用模板，可展开为多个相关记录。

### Lines 6368-6376
```tablegen
defm : NEONImmReplicateInstAlias<i16, VMOVv4i16, VMOVv8i16,
                                      VMVNv4i16, VMVNv8i16, i32>;
defm : NEONImmReplicateInstAlias<i16, VMOVv4i16, VMOVv8i16,
                                      VMVNv4i16, VMVNv8i16, i64>;
defm : NEONImmReplicateInstAlias<i32, VMOVv2i32, VMOVv4i32,
                                      VMVNv2i32, VMVNv4i32, i64>;
// TODO: add "VMOV <-> VMVN" conversion for cases like
// "vmov.i32 d0, #0xffaaffaa" -> "vmvn.i16 d0, #0x55"
// "vmvn.i32 d0, #0xaaffaaff" -> "vmov.i16 d0, #0xff00"
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6378-6380
```tablegen
// On some CPUs the two instructions "vmov.i32 dD, #0" and "vmov.i32 qD, #0"
// require zero cycles to execute so they should be used wherever possible for
// setting a register to zero.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6382-6384
```tablegen
// Even without these pseudo-insts we would probably end up with the correct
// instruction, but we could not mark the general ones with "isAsCheapAsAMove"
// since they are sometimes rather expensive (in general).
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6386-6395
```tablegen
let AddedComplexity = 50, isAsCheapAsAMove = 1, isReMaterializable = 1 in {
  def VMOVD0 : ARMPseudoExpand<(outs DPR:$Vd), (ins), 4, IIC_VMOVImm,
                               [(set DPR:$Vd, (v2i32 ARMimmAllZerosD))],
                               (VMOVv2i32 DPR:$Vd, 0, (ops 14, zero_reg))>,
               Requires<[HasZCZ]>;
  def VMOVQ0 : ARMPseudoExpand<(outs QPR:$Vd), (ins), 4, IIC_VMOVImm,
                               [(set QPR:$Vd, (v4i32 ARMimmAllZerosV))],
                               (VMOVv4i32 QPR:$Vd, 0, (ops 14, zero_reg))>,
               Requires<[HasZCZ]>;
}
```
- EN: Defines TableGen record `VMOVD0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVD0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6397-6397
```tablegen
//   VMOV     : Vector Get Lane (move scalar to ARM core register)
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6399-6416
```tablegen
def VGETLNs8  : NVGetLane<{1,1,1,0,0,1,?,1}, 0b1011, {?,?},
                          (outs GPR:$R), (ins DPR:$V, VectorIndex8:$lane),
                          IIC_VMOVSI, "vmov", "s8", "$R, $V$lane",
                          [(set GPR:$R, (ARMvgetlanes (v8i8 DPR:$V),
                                           imm:$lane))]> {
  let Inst{21}  = lane{2};
  let Inst{6-5} = lane{1-0};
}
def VGETLNs16 : NVGetLane<{1,1,1,0,0,0,?,1}, 0b1011, {?,1},
                          (outs GPR:$R), (ins DPR:$V, VectorIndex16:$lane),
                          IIC_VMOVSI, "vmov", "s16", "$R, $V$lane",
                          [(set GPR:$R, (ARMvgetlanes (v4i16 DPR:$V),
                                           imm:$lane))]> {
  let Inst{21} = lane{1};
  let Inst{6}  = lane{0};
}
def VGETLNu8  : NVGetLane<{1,1,1,0,1,1,?,1}, 0b1011, {?,?},
                          (outs GPR:$R), (ins DPR:$V, VectorIndex8:$lane),
```
- EN: Defines TableGen record `VGETLNs8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VGETLNs8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6417-6434
```tablegen
                          IIC_VMOVSI, "vmov", "u8", "$R, $V$lane",
                          [(set GPR:$R, (ARMvgetlaneu (v8i8 DPR:$V),
                                           imm:$lane))]> {
  let Inst{21}  = lane{2};
  let Inst{6-5} = lane{1-0};
}
def VGETLNu16 : NVGetLane<{1,1,1,0,1,0,?,1}, 0b1011, {?,1},
                          (outs GPR:$R), (ins DPR:$V, VectorIndex16:$lane),
                          IIC_VMOVSI, "vmov", "u16", "$R, $V$lane",
                          [(set GPR:$R, (ARMvgetlaneu (v4i16 DPR:$V),
                                           imm:$lane))]> {
  let Inst{21} = lane{1};
  let Inst{6}  = lane{0};
}
def VGETLNi32 : NVGetLane<{1,1,1,0,0,0,?,1}, 0b1011, 0b00,
                          (outs GPR:$R), (ins DPR:$V, VectorIndex32:$lane),
                          IIC_VMOVSI, "vmov", "32", "$R, $V$lane",
                          [(set GPR:$R, (extractelt (v2i32 DPR:$V),
```
- EN: Defines TableGen record `VGETLNu16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VGETLNu16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6435-6452
```tablegen
                                           imm:$lane))]>,
                Requires<[HasFPRegs, HasFastVGETLNi32]> {
  let Inst{21} = lane{0};
}
// VGETLNi32 is also legal as just vmov r0,d0[0] without the .32 suffix
def : InstAlias<"vmov${p} $R, $V$lane",
    (VGETLNi32 GPR:$R, DPR:$V, VectorIndex32:$lane, pred:$p), 0>,
    Requires<VGETLNi32.Predicates>;
let Predicates = [HasNEON] in {
// def VGETLNf32: see FMRDH and FMRDL in ARMInstrVFP.td
def : Pat<(ARMvgetlanes (v16i8 QPR:$src), imm:$lane),
          (VGETLNs8 (v8i8 (EXTRACT_SUBREG QPR:$src,
                           (DSubReg_i8_reg imm:$lane))),
                     (SubReg_i8_lane imm:$lane))>;
def : Pat<(ARMvgetlanes (v8i16 QPR:$src), imm:$lane),
          (VGETLNs16 (v4i16 (EXTRACT_SUBREG QPR:$src,
                             (DSubReg_i16_reg imm:$lane))),
                     (SubReg_i16_lane imm:$lane))>;
```
- EN: Defines TableGen record `VGETLNf32:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VGETLNf32:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6453-6470
```tablegen
def : Pat<(ARMvgetlaneu (v16i8 QPR:$src), imm:$lane),
          (VGETLNu8 (v8i8 (EXTRACT_SUBREG QPR:$src,
                           (DSubReg_i8_reg imm:$lane))),
                     (SubReg_i8_lane imm:$lane))>;
def : Pat<(ARMvgetlaneu (v8i16 QPR:$src), imm:$lane),
          (VGETLNu16 (v4i16 (EXTRACT_SUBREG QPR:$src,
                             (DSubReg_i16_reg imm:$lane))),
                     (SubReg_i16_lane imm:$lane))>;
def : Pat<(ARMvgetlaneu (v8f16 QPR:$src), imm:$lane),
          (VGETLNu16 (v4f16 (EXTRACT_SUBREG QPR:$src,
                             (DSubReg_i16_reg imm:$lane))),
                     (SubReg_i16_lane imm:$lane))>;
def : Pat<(ARMvgetlaneu (v4f16 DPR:$src), imm:$lane),
          (VGETLNu16 (v4f16 DPR:$src), imm:$lane)>;
def : Pat<(ARMvgetlaneu (v8bf16 QPR:$src), imm:$lane),
          (VGETLNu16 (v4bf16 (EXTRACT_SUBREG QPR:$src,
                             (DSubReg_i16_reg imm:$lane))),
                     (SubReg_i16_lane imm:$lane))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6471-6488
```tablegen
def : Pat<(ARMvgetlaneu (v4bf16 DPR:$src), imm:$lane),
          (VGETLNu16 (v4bf16 DPR:$src), imm:$lane)>;
}
def : Pat<(extractelt (v4i32 QPR:$src), imm:$lane),
          (VGETLNi32 (v2i32 (EXTRACT_SUBREG QPR:$src,
                             (DSubReg_i32_reg imm:$lane))),
                     (SubReg_i32_lane imm:$lane))>,
      Requires<[HasNEON, HasFastVGETLNi32]>;
def : Pat<(extractelt (v2i32 DPR:$src), imm:$lane),
          (COPY_TO_REGCLASS
            (i32 (EXTRACT_SUBREG DPR:$src, (SSubReg_f32_reg imm:$lane))), GPR)>,
      Requires<[HasNEON, HasSlowVGETLNi32]>;
def : Pat<(extractelt (v4i32 QPR:$src), imm:$lane),
          (COPY_TO_REGCLASS
            (i32 (EXTRACT_SUBREG QPR:$src, (SSubReg_f32_reg imm:$lane))), GPR)>,
      Requires<[HasNEON, HasSlowVGETLNi32]>;
let Predicates = [HasNEON] in {
def : Pat<(extractelt (v2f32 DPR:$src1), imm:$src2),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6489-6498
```tablegen
          (EXTRACT_SUBREG (v2f32 (COPY_TO_REGCLASS (v2f32 DPR:$src1),DPR_VFP2)),
                          (SSubReg_f32_reg imm:$src2))>;
def : Pat<(extractelt (v4f32 QPR:$src1), imm:$src2),
          (EXTRACT_SUBREG (v4f32 (COPY_TO_REGCLASS (v4f32 QPR:$src1),QPR_VFP2)),
                          (SSubReg_f32_reg imm:$src2))>;
//def : Pat<(extractelt (v2i64 QPR:$src1), imm:$src2),
//          (EXTRACT_SUBREG QPR:$src1, (DSubReg_f64_reg imm:$src2))>;
def : Pat<(extractelt (v2f64 QPR:$src1), imm:$src2),
          (EXTRACT_SUBREG QPR:$src1, (DSubReg_f64_reg imm:$src2))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6500-6509
```tablegen
multiclass ExtractEltEvenF16<ValueType VT4, ValueType VT8> {
  def : Pat<(extractelt (VT4 DPR:$src), imm_even:$lane),
              (EXTRACT_SUBREG
                  (v2f32 (COPY_TO_REGCLASS (VT4 DPR:$src), DPR_VFP2)),
                  (SSubReg_f16_reg imm_even:$lane))>;
  def : Pat<(extractelt (VT8 QPR:$src), imm_even:$lane),
              (EXTRACT_SUBREG
                  (v4f32 (COPY_TO_REGCLASS (VT8 QPR:$src), QPR_VFP2)),
                  (SSubReg_f16_reg imm_even:$lane))>;
}
```
- EN: Declares TableGen `multiclass ExtractEltEvenF16`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass ExtractEltEvenF16`，它是一个可复用模板，可展开为多个相关记录。

### Lines 6511-6524
```tablegen
multiclass ExtractEltOddF16VMOVH<ValueType VT4, ValueType VT8> {
  def : Pat<(extractelt (VT4 DPR:$src), imm_odd:$lane),
            (COPY_TO_REGCLASS
              (VMOVH (EXTRACT_SUBREG
                        (v2f32 (COPY_TO_REGCLASS (VT4 DPR:$src), DPR_VFP2)),
                        (SSubReg_f16_reg imm_odd:$lane))),
              HPR)>;
  def : Pat<(extractelt (VT8 QPR:$src), imm_odd:$lane),
            (COPY_TO_REGCLASS
              (VMOVH (EXTRACT_SUBREG
                        (v4f32 (COPY_TO_REGCLASS (VT8 QPR:$src), QPR_VFP2)),
                        (SSubReg_f16_reg imm_odd:$lane))),
              HPR)>;
}
```
- EN: Declares TableGen `multiclass ExtractEltOddF16VMOVH`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass ExtractEltOddF16VMOVH`，它是一个可复用模板，可展开为多个相关记录。

### Lines 6526-6529
```tablegen
let Predicates = [HasNEON] in {
  defm : ExtractEltEvenF16<v4f16, v8f16>;
  defm : ExtractEltOddF16VMOVH<v4f16, v8f16>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6531-6534
```tablegen
let AddedComplexity = 1, Predicates = [HasNEON, HasBF16, HasFullFP16] in {
  // If VMOVH (vmovx.f16) is available use it to extract BF16 from the odd lanes
  defm : ExtractEltOddF16VMOVH<v4bf16, v8bf16>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6536-6537
```tablegen
let Predicates = [HasBF16, HasNEON] in {
  defm : ExtractEltEvenF16<v4bf16, v8bf16>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6539-6544
```tablegen
  // Otherwise, if VMOVH is not available resort to extracting the odd lane
  // into a GPR and then moving to HPR
  def : Pat<(extractelt (v4bf16 DPR:$src), imm_odd:$lane),
            (COPY_TO_REGCLASS
              (VGETLNu16 (v4bf16 DPR:$src), imm:$lane),
              HPR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6546-6552
```tablegen
  def : Pat<(extractelt (v8bf16 QPR:$src), imm_odd:$lane),
            (COPY_TO_REGCLASS
              (VGETLNu16 (v4i16 (EXTRACT_SUBREG QPR:$src,
                                                (DSubReg_i16_reg imm:$lane))),
                         (SubReg_i16_lane imm:$lane)),
              HPR)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6554-6554
```tablegen
//   VMOV     : Vector Set Lane (move ARM core register to scalar)
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6556-6573
```tablegen
let Constraints = "$src1 = $V" in {
def VSETLNi8  : NVSetLane<{1,1,1,0,0,1,?,0}, 0b1011, {?,?}, (outs DPR:$V),
                          (ins DPR:$src1, GPR:$R, VectorIndex8:$lane),
                          IIC_VMOVISL, "vmov", "8", "$V$lane, $R",
                          [(set DPR:$V, (vector_insert (v8i8 DPR:$src1),
                                           GPR:$R, imm:$lane))]> {
  let Inst{21}  = lane{2};
  let Inst{6-5} = lane{1-0};
}
def VSETLNi16 : NVSetLane<{1,1,1,0,0,0,?,0}, 0b1011, {?,1}, (outs DPR:$V),
                          (ins DPR:$src1, GPR:$R, VectorIndex16:$lane),
                          IIC_VMOVISL, "vmov", "16", "$V$lane, $R",
                          [(set DPR:$V, (vector_insert (v4i16 DPR:$src1),
                                           GPR:$R, imm:$lane))]> {
  let Inst{21} = lane{1};
  let Inst{6}  = lane{0};
}
def VSETLNi32 : NVSetLane<{1,1,1,0,0,0,?,0}, 0b1011, 0b00, (outs DPR:$V),
```
- EN: Defines TableGen record `VSETLNi8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VSETLNi8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6574-6588
```tablegen
                          (ins DPR:$src1, GPR:$R, VectorIndex32:$lane),
                          IIC_VMOVISL, "vmov", "32", "$V$lane, $R",
                          [(set DPR:$V, (insertelt (v2i32 DPR:$src1),
                                           GPR:$R, imm:$lane))]>,
                Requires<[HasVFP2]> {
  let Inst{21} = lane{0};
  // This instruction is equivalent as
  // $V = INSERT_SUBREG $src1, $R, translateImmToSubIdx($imm)
  let isInsertSubreg = 1;
}
}
// VSETLNi32 is also legal as just vmov d0[0],r0 without the .32 suffix
def : InstAlias<"vmov${p} $V$lane, $R",
    (VSETLNi32 DPR:$V, GPR:$R, VectorIndex32:$lane, pred:$p), 0>,
    Requires<VSETLNi32.Predicates>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6590-6603
```tablegen
// TODO: for odd lanes we could optimize this a bit by using the VINS
// FullFP16 instruction when it is available
multiclass InsertEltF16<ValueType VTScalar, ValueType VT4, ValueType VT8> {
  def : Pat<(insertelt (VT4 DPR:$src1), (VTScalar HPR:$src2), imm:$lane),
            (VT4 (VSETLNi16 DPR:$src1,
                 (COPY_TO_REGCLASS HPR:$src2, GPR), imm:$lane))>;
  def : Pat<(insertelt (VT8 QPR:$src1), (VTScalar HPR:$src2), imm:$lane),
            (VT8 (INSERT_SUBREG QPR:$src1,
                    (v4i16 (VSETLNi16 (v4i16 (EXTRACT_SUBREG QPR:$src1,
                                        (DSubReg_i16_reg imm:$lane))),
                              (COPY_TO_REGCLASS HPR:$src2, GPR),
                              (SubReg_i16_lane imm:$lane))),
                    (DSubReg_i16_reg imm:$lane)))>;
}
```
- EN: Declares TableGen `multiclass InsertEltF16`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass InsertEltF16`，它是一个可复用模板，可展开为多个相关记录。

### Lines 6605-6622
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(vector_insert (v16i8 QPR:$src1), GPR:$src2, imm:$lane),
          (v16i8 (INSERT_SUBREG QPR:$src1,
                  (v8i8 (VSETLNi8 (v8i8 (EXTRACT_SUBREG QPR:$src1,
                                   (DSubReg_i8_reg imm:$lane))),
                            GPR:$src2, (SubReg_i8_lane imm:$lane))),
                  (DSubReg_i8_reg imm:$lane)))>;
def : Pat<(vector_insert (v8i16 QPR:$src1), GPR:$src2, imm:$lane),
          (v8i16 (INSERT_SUBREG QPR:$src1,
                  (v4i16 (VSETLNi16 (v4i16 (EXTRACT_SUBREG QPR:$src1,
                                     (DSubReg_i16_reg imm:$lane))),
                             GPR:$src2, (SubReg_i16_lane imm:$lane))),
                  (DSubReg_i16_reg imm:$lane)))>;
def : Pat<(insertelt (v4i32 QPR:$src1), GPR:$src2, imm:$lane),
          (v4i32 (INSERT_SUBREG QPR:$src1,
                  (v2i32 (VSETLNi32 (v2i32 (EXTRACT_SUBREG QPR:$src1,
                                     (DSubReg_i32_reg imm:$lane))),
                             GPR:$src2, (SubReg_i32_lane imm:$lane))),
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6623-6623
```tablegen
                  (DSubReg_i32_reg imm:$lane)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6625-6630
```tablegen
def : Pat<(v2f32 (insertelt DPR:$src1, SPR:$src2, imm:$src3)),
          (INSERT_SUBREG (v2f32 (COPY_TO_REGCLASS DPR:$src1, DPR_VFP2)),
                                SPR:$src2, (SSubReg_f32_reg imm:$src3))>;
def : Pat<(v4f32 (insertelt QPR:$src1, SPR:$src2, imm:$src3)),
          (INSERT_SUBREG (v4f32 (COPY_TO_REGCLASS QPR:$src1, QPR_VFP2)),
                                SPR:$src2, (SSubReg_f32_reg imm:$src3))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6632-6632
```tablegen
defm : InsertEltF16<f16, v4f16, v8f16>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6634-6635
```tablegen
def : Pat<(v2f64 (insertelt QPR:$src1, DPR:$src2, imm:$src3)),
          (INSERT_SUBREG QPR:$src1, DPR:$src2, (DSubReg_f64_reg imm:$src3))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6637-6642
```tablegen
def : Pat<(v2f32 (scalar_to_vector SPR:$src)),
          (INSERT_SUBREG (v2f32 (IMPLICIT_DEF)), SPR:$src, ssub_0)>;
def : Pat<(v2f64 (scalar_to_vector (f64 DPR:$src))),
          (INSERT_SUBREG (v2f64 (IMPLICIT_DEF)), DPR:$src, dsub_0)>;
def : Pat<(v4f32 (scalar_to_vector SPR:$src)),
          (INSERT_SUBREG (v4f32 (IMPLICIT_DEF)), SPR:$src, ssub_0)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6644-6647
```tablegen
def : Pat<(v4f16 (scalar_to_vector (f16 HPR:$src))),
          (INSERT_SUBREG (v4f16 (IMPLICIT_DEF)), HPR:$src, ssub_0)>;
def : Pat<(v8f16 (scalar_to_vector (f16 HPR:$src))),
          (INSERT_SUBREG (v8f16 (IMPLICIT_DEF)), HPR:$src, ssub_0)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6649-6654
```tablegen
def : Pat<(v8i8 (scalar_to_vector GPR:$src)),
          (VSETLNi8  (v8i8  (IMPLICIT_DEF)), GPR:$src, (i32 0))>;
def : Pat<(v4i16 (scalar_to_vector GPR:$src)),
          (VSETLNi16 (v4i16 (IMPLICIT_DEF)), GPR:$src, (i32 0))>;
def : Pat<(v2i32 (scalar_to_vector GPR:$src)),
          (VSETLNi32 (v2i32 (IMPLICIT_DEF)), GPR:$src, (i32 0))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6656-6668
```tablegen
def : Pat<(v16i8 (scalar_to_vector GPR:$src)),
          (INSERT_SUBREG (v16i8 (IMPLICIT_DEF)),
                         (VSETLNi8 (v8i8 (IMPLICIT_DEF)), GPR:$src, (i32 0)),
                         dsub_0)>;
def : Pat<(v8i16 (scalar_to_vector GPR:$src)),
          (INSERT_SUBREG (v8i16 (IMPLICIT_DEF)),
                         (VSETLNi16 (v4i16 (IMPLICIT_DEF)), GPR:$src, (i32 0)),
                         dsub_0)>;
def : Pat<(v4i32 (scalar_to_vector GPR:$src)),
          (INSERT_SUBREG (v4i32 (IMPLICIT_DEF)),
                         (VSETLNi32 (v2i32 (IMPLICIT_DEF)), GPR:$src, (i32 0)),
                         dsub_0)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6670-6671
```tablegen
let Predicates = [HasNEON, HasBF16] in
defm : InsertEltF16<bf16, v4bf16, v8bf16>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6673-6673
```tablegen
//   VDUP     : Vector Duplicate (from ARM core register to all elements)
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6675-6682
```tablegen
class VDUPD<bits<8> opcod1, bits<2> opcod3, string Dt, ValueType Ty>
  : NVDup<opcod1, 0b1011, opcod3, (outs DPR:$V), (ins GPR:$R),
          IIC_VMOVIS, "vdup", Dt, "$V, $R",
          [(set DPR:$V, (Ty (ARMvdup (i32 GPR:$R))))]>;
class VDUPQ<bits<8> opcod1, bits<2> opcod3, string Dt, ValueType Ty>
  : NVDup<opcod1, 0b1011, opcod3, (outs QPR:$V), (ins GPR:$R),
          IIC_VMOVIS, "vdup", Dt, "$V, $R",
          [(set QPR:$V, (Ty (ARMvdup (i32 GPR:$R))))]>;
```
- EN: Declares reusable TableGen class `VDUPD` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VDUPD`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6684-6690
```tablegen
def  VDUP8d   : VDUPD<0b11101100, 0b00, "8", v8i8>;
def  VDUP16d  : VDUPD<0b11101000, 0b01, "16", v4i16>;
def  VDUP32d  : VDUPD<0b11101000, 0b00, "32", v2i32>,
                Requires<[HasNEON, HasFastVDUP32]>;
def  VDUP8q   : VDUPQ<0b11101110, 0b00, "8", v16i8>;
def  VDUP16q  : VDUPQ<0b11101010, 0b01, "16", v8i16>;
def  VDUP32q  : VDUPQ<0b11101010, 0b00, "32", v4i32>;
```
- EN: Defines TableGen record `VDUP8d` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VDUP8d`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6692-6696
```tablegen
// ARMvdup patterns for uarchs with fast VDUP.32.
def : Pat<(v2f32 (ARMvdup (f32 (bitconvert GPR:$R)))), (VDUP32d GPR:$R)>,
      Requires<[HasNEON,HasFastVDUP32]>;
def : Pat<(v4f32 (ARMvdup (f32 (bitconvert GPR:$R)))), (VDUP32q GPR:$R)>,
      Requires<[HasNEON]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6698-6702
```tablegen
// ARMvdup patterns for uarchs with slow VDUP.32 - use VMOVDRR instead.
def : Pat<(v2i32 (ARMvdup (i32 GPR:$R))), (VMOVDRR GPR:$R, GPR:$R)>,
      Requires<[HasNEON,HasSlowVDUP32]>;
def : Pat<(v2f32 (ARMvdup (f32 (bitconvert GPR:$R)))), (VMOVDRR GPR:$R, GPR:$R)>,
      Requires<[HasNEON,HasSlowVDUP32]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6704-6704
```tablegen
//   VDUP     : Vector Duplicate Lane (from scalar to all elements)
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6706-6710
```tablegen
class VDUPLND<bits<4> op19_16, string OpcodeStr, string Dt,
              ValueType Ty, Operand IdxTy>
  : NVDupLane<op19_16, 0, (outs DPR:$Vd), (ins DPR:$Vm, IdxTy:$lane),
              IIC_VMOVD, OpcodeStr, Dt, "$Vd, $Vm$lane",
              [(set DPR:$Vd, (Ty (ARMvduplane (Ty DPR:$Vm), imm:$lane)))]>;
```
- EN: Declares reusable TableGen class `VDUPLND` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VDUPLND`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6712-6717
```tablegen
class VDUPLNQ<bits<4> op19_16, string OpcodeStr, string Dt,
              ValueType ResTy, ValueType OpTy, Operand IdxTy>
  : NVDupLane<op19_16, 1, (outs QPR:$Vd), (ins DPR:$Vm, IdxTy:$lane),
              IIC_VMOVQ, OpcodeStr, Dt, "$Vd, $Vm$lane",
              [(set QPR:$Vd, (ResTy (ARMvduplane (OpTy DPR:$Vm),
                                      VectorIndex32:$lane)))]>;
```
- EN: Declares reusable TableGen class `VDUPLNQ` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VDUPLNQ`，通常用于抽象共享字段、谓词或编码结构。

### Lines 6719-6719
```tablegen
// Inst{19-16} is partially specified depending on the element size.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6721-6738
```tablegen
def VDUPLN8d  : VDUPLND<{?,?,?,1}, "vdup", "8", v8i8, VectorIndex8> {
  bits<3> lane;
  let Inst{19-17} = lane{2-0};
}
def VDUPLN16d : VDUPLND<{?,?,1,0}, "vdup", "16", v4i16, VectorIndex16> {
  bits<2> lane;
  let Inst{19-18} = lane{1-0};
}
def VDUPLN32d : VDUPLND<{?,1,0,0}, "vdup", "32", v2i32, VectorIndex32> {
  bits<1> lane;
  let Inst{19} = lane{0};
}
def VDUPLN8q  : VDUPLNQ<{?,?,?,1}, "vdup", "8", v16i8, v8i8, VectorIndex8> {
  bits<3> lane;
  let Inst{19-17} = lane{2-0};
}
def VDUPLN16q : VDUPLNQ<{?,?,1,0}, "vdup", "16", v8i16, v4i16, VectorIndex16> {
  bits<2> lane;
```
- EN: Defines TableGen record `VDUPLN8d` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VDUPLN8d`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6739-6744
```tablegen
  let Inst{19-18} = lane{1-0};
}
def VDUPLN32q : VDUPLNQ<{?,1,0,0}, "vdup", "32", v4i32, v2i32, VectorIndex32> {
  bits<1> lane;
  let Inst{19} = lane{0};
}
```
- EN: Defines TableGen record `VDUPLN32q` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VDUPLN32q`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6746-6748
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v4f16 (ARMvduplane (v4f16 DPR:$Vm), imm:$lane)),
          (VDUPLN16d DPR:$Vm, imm:$lane)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6750-6751
```tablegen
def : Pat<(v2f32 (ARMvduplane (v2f32 DPR:$Vm), imm:$lane)),
          (VDUPLN32d DPR:$Vm, imm:$lane)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6753-6754
```tablegen
def : Pat<(v4f32 (ARMvduplane (v2f32 DPR:$Vm), imm:$lane)),
          (VDUPLN32q DPR:$Vm, imm:$lane)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6756-6773
```tablegen
def : Pat<(v16i8 (ARMvduplane (v16i8 QPR:$src), imm:$lane)),
          (v16i8 (VDUPLN8q (v8i8 (EXTRACT_SUBREG QPR:$src,
                                  (DSubReg_i8_reg imm:$lane))),
                           (SubReg_i8_lane imm:$lane)))>;
def : Pat<(v8i16 (ARMvduplane (v8i16 QPR:$src), imm:$lane)),
          (v8i16 (VDUPLN16q (v4i16 (EXTRACT_SUBREG QPR:$src,
                                    (DSubReg_i16_reg imm:$lane))),
                            (SubReg_i16_lane imm:$lane)))>;
def : Pat<(v8f16 (ARMvduplane (v8f16 QPR:$src), imm:$lane)),
          (v8f16 (VDUPLN16q (v4f16 (EXTRACT_SUBREG QPR:$src,
                                    (DSubReg_i16_reg imm:$lane))),
                            (SubReg_i16_lane imm:$lane)))>;
def : Pat<(v4i32 (ARMvduplane (v4i32 QPR:$src), imm:$lane)),
          (v4i32 (VDUPLN32q (v2i32 (EXTRACT_SUBREG QPR:$src,
                                    (DSubReg_i32_reg imm:$lane))),
                            (SubReg_i32_lane imm:$lane)))>;
def : Pat<(v4f32 (ARMvduplane (v4f32 QPR:$src), imm:$lane)),
          (v4f32 (VDUPLN32q (v2f32 (EXTRACT_SUBREG QPR:$src,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6774-6775
```tablegen
                                   (DSubReg_i32_reg imm:$lane))),
                           (SubReg_i32_lane imm:$lane)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6777-6789
```tablegen
def : Pat<(v4f16 (ARMvdup (f16 HPR:$src))),
          (v4f16 (VDUPLN16d (INSERT_SUBREG (v4f16 (IMPLICIT_DEF)),
                             (f16 HPR:$src), ssub_0), (i32 0)))>;
def : Pat<(v2f32 (ARMvdup (f32 SPR:$src))),
          (v2f32 (VDUPLN32d (INSERT_SUBREG (v2f32 (IMPLICIT_DEF)),
                             SPR:$src, ssub_0), (i32 0)))>;
def : Pat<(v4f32 (ARMvdup (f32 SPR:$src))),
          (v4f32 (VDUPLN32q (INSERT_SUBREG (v2f32 (IMPLICIT_DEF)),
                             SPR:$src, ssub_0), (i32 0)))>;
def : Pat<(v8f16 (ARMvdup (f16 HPR:$src))),
          (v8f16 (VDUPLN16q (INSERT_SUBREG (v4f16 (IMPLICIT_DEF)),
                             (f16 HPR:$src), ssub_0), (i32 0)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6791-6793
```tablegen
let Predicates = [HasNEON, HasBF16] in {
def : Pat<(v4bf16 (ARMvduplane (v4bf16 DPR:$Vm), imm:$lane)),
          (VDUPLN16d DPR:$Vm, imm:$lane)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6795-6798
```tablegen
def : Pat<(v8bf16 (ARMvduplane (v8bf16 QPR:$src), imm:$lane)),
          (v8bf16 (VDUPLN16q (v4bf16 (EXTRACT_SUBREG QPR:$src,
                                    (DSubReg_i16_reg imm:$lane))),
                            (SubReg_i16_lane imm:$lane)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6800-6806
```tablegen
def : Pat<(v4bf16 (ARMvdup (bf16 HPR:$src))),
          (v4bf16 (VDUPLN16d (INSERT_SUBREG (v4bf16 (IMPLICIT_DEF)),
                             (bf16 HPR:$src), ssub_0), (i32 0)))>;
def : Pat<(v8bf16 (ARMvdup (bf16 HPR:$src))),
          (v8bf16 (VDUPLN16q (INSERT_SUBREG (v4bf16 (IMPLICIT_DEF)),
                             (bf16 HPR:$src), ssub_0), (i32 0)))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6808-6820
```tablegen
//   VMOVN    : Vector Narrowing Move
defm VMOVN    : N2VN_HSD<0b11,0b11,0b10,0b00100,0,0, IIC_VMOVN,
                         "vmovn", "i", trunc>;
//   VQMOVN   : Vector Saturating Narrowing Move
defm VQMOVNs  : N2VNInt_HSD<0b11,0b11,0b10,0b00101,0,0, IIC_VQUNAiD,
                            "vqmovn", "s", int_arm_neon_vqmovns>;
defm VQMOVNu  : N2VNInt_HSD<0b11,0b11,0b10,0b00101,1,0, IIC_VQUNAiD,
                            "vqmovn", "u", int_arm_neon_vqmovnu>;
defm VQMOVNsu : N2VNInt_HSD<0b11,0b11,0b10,0b00100,1,0, IIC_VQUNAiD,
                            "vqmovun", "s", int_arm_neon_vqmovnsu>;
//   VMOVL    : Vector Lengthening Move
defm VMOVLs   : N2VL_QHS<0b01,0b10100,0,1, "vmovl", "s", sext>;
defm VMOVLu   : N2VL_QHS<0b11,0b10100,0,1, "vmovl", "u", zext>;
```
- EN: Defines TableGen record `VMOVN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMOVN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6822-6826
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i16 (anyext (v8i8 DPR:$Vm))), (VMOVLuv8i16 DPR:$Vm)>;
def : Pat<(v4i32 (anyext (v4i16 DPR:$Vm))), (VMOVLuv4i32 DPR:$Vm)>;
def : Pat<(v2i64 (anyext (v2i32 DPR:$Vm))), (VMOVLuv2i64 DPR:$Vm)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6828-6828
```tablegen
// Vector Conversions.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6830-6838
```tablegen
//   VCVT     : Vector Convert Between Floating-Point and Integers
def  VCVTf2sd : N2VD<0b11, 0b11, 0b10, 0b11, 0b01110, 0, "vcvt", "s32.f32",
                     v2i32, v2f32, fp_to_sint>;
def  VCVTf2ud : N2VD<0b11, 0b11, 0b10, 0b11, 0b01111, 0, "vcvt", "u32.f32",
                     v2i32, v2f32, fp_to_uint>;
def  VCVTs2fd : N2VD<0b11, 0b11, 0b10, 0b11, 0b01100, 0, "vcvt", "f32.s32",
                     v2f32, v2i32, sint_to_fp>;
def  VCVTu2fd : N2VD<0b11, 0b11, 0b10, 0b11, 0b01101, 0, "vcvt", "f32.u32",
                     v2f32, v2i32, uint_to_fp>;
```
- EN: Defines TableGen record `VCVTf2sd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTf2sd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6840-6847
```tablegen
def  VCVTf2sq : N2VQ<0b11, 0b11, 0b10, 0b11, 0b01110, 0, "vcvt", "s32.f32",
                     v4i32, v4f32, fp_to_sint>;
def  VCVTf2uq : N2VQ<0b11, 0b11, 0b10, 0b11, 0b01111, 0, "vcvt", "u32.f32",
                     v4i32, v4f32, fp_to_uint>;
def  VCVTs2fq : N2VQ<0b11, 0b11, 0b10, 0b11, 0b01100, 0, "vcvt", "f32.s32",
                     v4f32, v4i32, sint_to_fp>;
def  VCVTu2fq : N2VQ<0b11, 0b11, 0b10, 0b11, 0b01101, 0, "vcvt", "f32.u32",
                     v4f32, v4i32, uint_to_fp>;
```
- EN: Defines TableGen record `VCVTf2sq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTf2sq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6849-6860
```tablegen
def  VCVTh2sd : N2VD<0b11, 0b11, 0b01, 0b11, 0b01110, 0, "vcvt", "s16.f16",
                     v4i16, v4f16, fp_to_sint>,
                Requires<[HasNEON, HasFullFP16]>;
def  VCVTh2ud : N2VD<0b11, 0b11, 0b01, 0b11, 0b01111, 0, "vcvt", "u16.f16",
                     v4i16, v4f16, fp_to_uint>,
                Requires<[HasNEON, HasFullFP16]>;
def  VCVTs2hd : N2VD<0b11, 0b11, 0b01, 0b11, 0b01100, 0, "vcvt", "f16.s16",
                     v4f16, v4i16, sint_to_fp>,
                Requires<[HasNEON, HasFullFP16]>;
def  VCVTu2hd : N2VD<0b11, 0b11, 0b01, 0b11, 0b01101, 0, "vcvt", "f16.u16",
                     v4f16, v4i16, uint_to_fp>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VCVTh2sd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTh2sd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6862-6873
```tablegen
def  VCVTh2sq : N2VQ<0b11, 0b11, 0b01, 0b11, 0b01110, 0, "vcvt", "s16.f16",
                     v8i16, v8f16, fp_to_sint>,
                Requires<[HasNEON, HasFullFP16]>;
def  VCVTh2uq : N2VQ<0b11, 0b11, 0b01, 0b11, 0b01111, 0, "vcvt", "u16.f16",
                     v8i16, v8f16, fp_to_uint>,
                Requires<[HasNEON, HasFullFP16]>;
def  VCVTs2hq : N2VQ<0b11, 0b11, 0b01, 0b11, 0b01100, 0, "vcvt", "f16.s16",
                     v8f16, v8i16, sint_to_fp>,
                Requires<[HasNEON, HasFullFP16]>;
def  VCVTu2hq : N2VQ<0b11, 0b11, 0b01, 0b11, 0b01101, 0, "vcvt", "f16.u16",
                     v8f16, v8i16, uint_to_fp>,
                Requires<[HasNEON, HasFullFP16]>;
```
- EN: Defines TableGen record `VCVTh2sq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTh2sq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6875-6892
```tablegen
// VCVT{A, N, P, M}
multiclass VCVT_FPI<string op, bits<3> op10_8, SDPatternOperator IntS,
                    SDPatternOperator IntU> {
  let PostEncoderMethod = "NEONThumb2V8PostEncoder", DecoderNamespace = "v8NEON" in {
    def SDf : N2VDIntnp<0b10, 0b11, op10_8, 0, NoItinerary, !strconcat("vcvt", op),
                       "s32.f32", v2i32, v2f32, IntS>, Requires<[HasV8, HasNEON]>;
    def SQf : N2VQIntnp<0b10, 0b11, op10_8, 0, NoItinerary, !strconcat("vcvt", op),
                       "s32.f32", v4i32, v4f32, IntS>, Requires<[HasV8, HasNEON]>;
    def UDf : N2VDIntnp<0b10, 0b11, op10_8, 1, NoItinerary, !strconcat("vcvt", op),
                       "u32.f32", v2i32, v2f32, IntU>, Requires<[HasV8, HasNEON]>;
    def UQf : N2VQIntnp<0b10, 0b11, op10_8, 1, NoItinerary, !strconcat("vcvt", op),
                       "u32.f32", v4i32, v4f32, IntU>, Requires<[HasV8, HasNEON]>;
    def SDh : N2VDIntnp<0b01, 0b11, op10_8, 0, NoItinerary, !strconcat("vcvt", op),
                       "s16.f16", v4i16, v4f16, IntS>,
              Requires<[HasV8, HasNEON, HasFullFP16]>;
    def SQh : N2VQIntnp<0b01, 0b11, op10_8, 0, NoItinerary, !strconcat("vcvt", op),
                       "s16.f16", v8i16, v8f16, IntS>,
              Requires<[HasV8, HasNEON, HasFullFP16]>;
```
- EN: Declares TableGen `multiclass VCVT_FPI`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VCVT_FPI`，它是一个可复用模板，可展开为多个相关记录。

### Lines 6893-6900
```tablegen
    def UDh : N2VDIntnp<0b01, 0b11, op10_8, 1, NoItinerary, !strconcat("vcvt", op),
                       "u16.f16", v4i16, v4f16, IntU>,
              Requires<[HasV8, HasNEON, HasFullFP16]>;
    def UQh : N2VQIntnp<0b01, 0b11, op10_8, 1, NoItinerary, !strconcat("vcvt", op),
                       "u16.f16", v8i16, v8f16, IntU>,
              Requires<[HasV8, HasNEON, HasFullFP16]>;
  }
}
```
- EN: Defines TableGen record `UDh` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UDh`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6902-6905
```tablegen
defm VCVTAN : VCVT_FPI<"a", 0b000, int_arm_neon_vcvtas, int_arm_neon_vcvtau>;
defm VCVTNN : VCVT_FPI<"n", 0b001, int_arm_neon_vcvtns, int_arm_neon_vcvtnu>;
defm VCVTPN : VCVT_FPI<"p", 0b010, int_arm_neon_vcvtps, int_arm_neon_vcvtpu>;
defm VCVTMN : VCVT_FPI<"m", 0b011, int_arm_neon_vcvtms, int_arm_neon_vcvtmu>;
```
- EN: Defines TableGen record `VCVTAN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTAN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6907-6924
```tablegen
//   VCVT     : Vector Convert Between Floating-Point and Fixed-Point.
let DecoderMethod = "DecodeVCVTD" in {
def VCVTf2xsd : N2VCvtD<0, 1, 0b1111, 0, 1, "vcvt", "s32.f32",
                        v2i32, v2f32, int_arm_neon_vcvtfp2fxs>;
def VCVTf2xud : N2VCvtD<1, 1, 0b1111, 0, 1, "vcvt", "u32.f32",
                        v2i32, v2f32, int_arm_neon_vcvtfp2fxu>;
def VCVTxs2fd : N2VCvtD<0, 1, 0b1110, 0, 1, "vcvt", "f32.s32",
                        v2f32, v2i32, int_arm_neon_vcvtfxs2fp>;
def VCVTxu2fd : N2VCvtD<1, 1, 0b1110, 0, 1, "vcvt", "f32.u32",
                        v2f32, v2i32, int_arm_neon_vcvtfxu2fp>;
let Predicates = [HasNEON, HasFullFP16] in {
def VCVTh2xsd : N2VCvtD<0, 1, 0b1101, 0, 1, "vcvt", "s16.f16",
                        v4i16, v4f16, int_arm_neon_vcvtfp2fxs>;
def VCVTh2xud : N2VCvtD<1, 1, 0b1101, 0, 1, "vcvt", "u16.f16",
                        v4i16, v4f16, int_arm_neon_vcvtfp2fxu>;
def VCVTxs2hd : N2VCvtD<0, 1, 0b1100, 0, 1, "vcvt", "f16.s16",
                        v4f16, v4i16, int_arm_neon_vcvtfxs2fp>;
def VCVTxu2hd : N2VCvtD<1, 1, 0b1100, 0, 1, "vcvt", "f16.u16",
```
- EN: Defines TableGen record `VCVTf2xsd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTf2xsd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6925-6927
```tablegen
                        v4f16, v4i16, int_arm_neon_vcvtfxu2fp>;
} // Predicates = [HasNEON, HasFullFP16]
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6929-6946
```tablegen
let DecoderMethod = "DecodeVCVTQ" in {
def VCVTf2xsq : N2VCvtQ<0, 1, 0b1111, 0, 1, "vcvt", "s32.f32",
                        v4i32, v4f32, int_arm_neon_vcvtfp2fxs>;
def VCVTf2xuq : N2VCvtQ<1, 1, 0b1111, 0, 1, "vcvt", "u32.f32",
                        v4i32, v4f32, int_arm_neon_vcvtfp2fxu>;
def VCVTxs2fq : N2VCvtQ<0, 1, 0b1110, 0, 1, "vcvt", "f32.s32",
                        v4f32, v4i32, int_arm_neon_vcvtfxs2fp>;
def VCVTxu2fq : N2VCvtQ<1, 1, 0b1110, 0, 1, "vcvt", "f32.u32",
                        v4f32, v4i32, int_arm_neon_vcvtfxu2fp>;
let Predicates = [HasNEON, HasFullFP16] in {
def VCVTh2xsq : N2VCvtQ<0, 1, 0b1101, 0, 1, "vcvt", "s16.f16",
                        v8i16, v8f16, int_arm_neon_vcvtfp2fxs>;
def VCVTh2xuq : N2VCvtQ<1, 1, 0b1101, 0, 1, "vcvt", "u16.f16",
                        v8i16, v8f16, int_arm_neon_vcvtfp2fxu>;
def VCVTxs2hq : N2VCvtQ<0, 1, 0b1100, 0, 1, "vcvt", "f16.s16",
                        v8f16, v8i16, int_arm_neon_vcvtfxs2fp>;
def VCVTxu2hq : N2VCvtQ<1, 1, 0b1100, 0, 1, "vcvt", "f16.u16",
                        v8f16, v8i16, int_arm_neon_vcvtfxu2fp>;
```
- EN: Defines TableGen record `VCVTf2xsq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTf2xsq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6947-6948
```tablegen
} // Predicates = [HasNEON, HasFullFP16]
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6950-6957
```tablegen
def : NEONInstAlias<"vcvt${p}.s32.f32 $Dd, $Dm, #0",
                    (VCVTf2sd DPR:$Dd, DPR:$Dm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.u32.f32 $Dd, $Dm, #0",
                    (VCVTf2ud DPR:$Dd, DPR:$Dm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.f32.s32 $Dd, $Dm, #0",
                    (VCVTs2fd DPR:$Dd, DPR:$Dm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.f32.u32 $Dd, $Dm, #0",
                    (VCVTu2fd DPR:$Dd, DPR:$Dm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6959-6966
```tablegen
def : NEONInstAlias<"vcvt${p}.s32.f32 $Qd, $Qm, #0",
                    (VCVTf2sq QPR:$Qd, QPR:$Qm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.u32.f32 $Qd, $Qm, #0",
                    (VCVTf2uq QPR:$Qd, QPR:$Qm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.f32.s32 $Qd, $Qm, #0",
                    (VCVTs2fq QPR:$Qd, QPR:$Qm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.f32.u32 $Qd, $Qm, #0",
                    (VCVTu2fq QPR:$Qd, QPR:$Qm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6968-6975
```tablegen
def : NEONInstAlias<"vcvt${p}.s16.f16 $Dd, $Dm, #0",
                    (VCVTh2sd DPR:$Dd, DPR:$Dm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.u16.f16 $Dd, $Dm, #0",
                    (VCVTh2ud DPR:$Dd, DPR:$Dm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.f16.s16 $Dd, $Dm, #0",
                    (VCVTs2hd DPR:$Dd, DPR:$Dm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.f16.u16 $Dd, $Dm, #0",
                    (VCVTu2hd DPR:$Dd, DPR:$Dm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6977-6984
```tablegen
def : NEONInstAlias<"vcvt${p}.s16.f16 $Qd, $Qm, #0",
                    (VCVTh2sq QPR:$Qd, QPR:$Qm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.u16.f16 $Qd, $Qm, #0",
                    (VCVTh2uq QPR:$Qd, QPR:$Qm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.f16.s16 $Qd, $Qm, #0",
                    (VCVTs2hq QPR:$Qd, QPR:$Qm, pred:$p)>;
def : NEONInstAlias<"vcvt${p}.f16.u16 $Qd, $Qm, #0",
                    (VCVTu2hq QPR:$Qd, QPR:$Qm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6987-6995
```tablegen
//   VCVT     : Vector Convert Between Half-Precision and Single-Precision.
def  VCVTf2h  : N2VNInt<0b11, 0b11, 0b01, 0b10, 0b01100, 0, 0,
                        IIC_VUNAQ, "vcvt", "f16.f32",
                        v4i16, v4f32, int_arm_neon_vcvtfp2hf>,
                Requires<[HasNEON, HasFP16]>;
def  VCVTh2f  : N2VLInt<0b11, 0b11, 0b01, 0b10, 0b01110, 0, 0,
                        IIC_VUNAQ, "vcvt", "f32.f16",
                        v4f32, v4i16, int_arm_neon_vcvthf2fp>,
                Requires<[HasNEON, HasFP16]>;
```
- EN: Defines TableGen record `VCVTf2h` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VCVTf2h`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6997-6998
```tablegen
def : Pat<(v4f16 (fpround (v4f32 QPR:$src))), (VCVTf2h QPR:$src)>;
def : Pat<(v4f32 (fpextend (v4f16 DPR:$src))), (VCVTh2f DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7000-7000
```tablegen
// Vector Reverse.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7002-7002
```tablegen
//   VREV64   : Vector Reverse elements within 64-bit doublewords
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7004-7013
```tablegen
class VREV64D<bits<2> op19_18, string OpcodeStr, string Dt, ValueType Ty>
  : N2V<0b11, 0b11, op19_18, 0b00, 0b00000, 0, 0, (outs DPR:$Vd),
        (ins DPR:$Vm), IIC_VMOVD,
        OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set DPR:$Vd, (Ty (ARMvrev64 (Ty DPR:$Vm))))]>;
class VREV64Q<bits<2> op19_18, string OpcodeStr, string Dt, ValueType Ty>
  : N2V<0b11, 0b11, op19_18, 0b00, 0b00000, 1, 0, (outs QPR:$Vd),
        (ins QPR:$Vm), IIC_VMOVQ,
        OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set QPR:$Vd, (Ty (ARMvrev64 (Ty QPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `VREV64D` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VREV64D`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7015-7020
```tablegen
def VREV64d8  : VREV64D<0b00, "vrev64", "8", v8i8>;
def VREV64d16 : VREV64D<0b01, "vrev64", "16", v4i16>;
def VREV64d32 : VREV64D<0b10, "vrev64", "32", v2i32>;
let Predicates = [HasNEON] in {
def : Pat<(v2f32 (ARMvrev64 (v2f32 DPR:$Vm))), (VREV64d32 DPR:$Vm)>;
}
```
- EN: Defines TableGen record `VREV64d8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VREV64d8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7022-7024
```tablegen
def VREV64q8  : VREV64Q<0b00, "vrev64", "8", v16i8>;
def VREV64q16 : VREV64Q<0b01, "vrev64", "16", v8i16>;
def VREV64q32 : VREV64Q<0b10, "vrev64", "32", v4i32>;
```
- EN: Defines TableGen record `VREV64q8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VREV64q8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7026-7037
```tablegen
let Predicates = [HasNEON] in {
  def : Pat<(v4f32 (ARMvrev64 (v4f32 QPR:$Vm))),
            (VREV64q32 QPR:$Vm)>;
  def : Pat<(v8f16 (ARMvrev64 (v8f16 QPR:$Vm))),
            (VREV64q16 QPR:$Vm)>;
  def : Pat<(v4f16 (ARMvrev64 (v4f16 DPR:$Vm))),
            (VREV64d16 DPR:$Vm)>;
  def : Pat<(v8bf16 (ARMvrev64 (v8bf16 QPR:$Vm))),
            (VREV64q16 QPR:$Vm)>;
  def : Pat<(v4bf16 (ARMvrev64 (v4bf16 DPR:$Vm))),
            (VREV64d16 DPR:$Vm)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7039-7039
```tablegen
//   VREV32   : Vector Reverse elements within 32-bit words
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7041-7050
```tablegen
class VREV32D<bits<2> op19_18, string OpcodeStr, string Dt, ValueType Ty>
  : N2V<0b11, 0b11, op19_18, 0b00, 0b00001, 0, 0, (outs DPR:$Vd),
        (ins DPR:$Vm), IIC_VMOVD,
        OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set DPR:$Vd, (Ty (ARMvrev32 (Ty DPR:$Vm))))]>;
class VREV32Q<bits<2> op19_18, string OpcodeStr, string Dt, ValueType Ty>
  : N2V<0b11, 0b11, op19_18, 0b00, 0b00001, 1, 0, (outs QPR:$Vd),
        (ins QPR:$Vm), IIC_VMOVQ,
        OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set QPR:$Vd, (Ty (ARMvrev32 (Ty QPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `VREV32D` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VREV32D`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7052-7053
```tablegen
def VREV32d8  : VREV32D<0b00, "vrev32", "8", v8i8>;
def VREV32d16 : VREV32D<0b01, "vrev32", "16", v4i16>;
```
- EN: Defines TableGen record `VREV32d8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VREV32d8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7055-7056
```tablegen
def VREV32q8  : VREV32Q<0b00, "vrev32", "8", v16i8>;
def VREV32q16 : VREV32Q<0b01, "vrev32", "16", v8i16>;
```
- EN: Defines TableGen record `VREV32q8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VREV32q8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7058-7067
```tablegen
let Predicates = [HasNEON] in {
  def : Pat<(v8f16 (ARMvrev32 (v8f16 QPR:$Vm))),
            (VREV32q16 QPR:$Vm)>;
  def : Pat<(v4f16 (ARMvrev32 (v4f16 DPR:$Vm))),
            (VREV32d16 DPR:$Vm)>;
  def : Pat<(v8bf16 (ARMvrev32 (v8bf16 QPR:$Vm))),
            (VREV32q16 QPR:$Vm)>;
  def : Pat<(v4bf16 (ARMvrev32 (v4bf16 DPR:$Vm))),
            (VREV32d16 DPR:$Vm)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7069-7069
```tablegen
//   VREV16   : Vector Reverse elements within 16-bit halfwords
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7071-7080
```tablegen
class VREV16D<bits<2> op19_18, string OpcodeStr, string Dt, ValueType Ty>
  : N2V<0b11, 0b11, op19_18, 0b00, 0b00010, 0, 0, (outs DPR:$Vd),
        (ins DPR:$Vm), IIC_VMOVD,
        OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set DPR:$Vd, (Ty (ARMvrev16 (Ty DPR:$Vm))))]>;
class VREV16Q<bits<2> op19_18, string OpcodeStr, string Dt, ValueType Ty>
  : N2V<0b11, 0b11, op19_18, 0b00, 0b00010, 1, 0, (outs QPR:$Vd),
        (ins QPR:$Vm), IIC_VMOVQ,
        OpcodeStr, Dt, "$Vd, $Vm", "",
        [(set QPR:$Vd, (Ty (ARMvrev16 (Ty QPR:$Vm))))]>;
```
- EN: Declares reusable TableGen class `VREV16D` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VREV16D`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7082-7083
```tablegen
def VREV16d8  : VREV16D<0b00, "vrev16", "8", v8i8>;
def VREV16q8  : VREV16Q<0b00, "vrev16", "8", v16i8>;
```
- EN: Defines TableGen record `VREV16d8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VREV16d8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7085-7085
```tablegen
// Other Vector Shuffles.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7087-7087
```tablegen
//  Aligned extractions: really just dropping registers
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7089-7092
```tablegen
class AlignedVEXTq<ValueType DestTy, ValueType SrcTy, SDNodeXForm LaneCVT>
      : Pat<(DestTy (vector_extract_subvec (SrcTy QPR:$src), (i32 imm:$start))),
             (EXTRACT_SUBREG (SrcTy QPR:$src), (LaneCVT imm:$start))>,
        Requires<[HasNEON]>;
```
- EN: Declares reusable TableGen class `AlignedVEXTq` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `AlignedVEXTq`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7094-7100
```tablegen
def : AlignedVEXTq<v8i8, v16i8, DSubReg_i8_reg>;
def : AlignedVEXTq<v4i16, v8i16, DSubReg_i16_reg>;
def : AlignedVEXTq<v2i32, v4i32, DSubReg_i32_reg>;
def : AlignedVEXTq<v1i64, v2i64, DSubReg_f64_reg>;
def : AlignedVEXTq<v2f32, v4f32, DSubReg_i32_reg>;
def : AlignedVEXTq<v4f16, v8f16, DSubReg_i16_reg>;
def : AlignedVEXTq<v4bf16, v8bf16, DSubReg_i16_reg>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7103-7103
```tablegen
//   VEXT     : Vector Extract
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7105-7116
```tablegen
// All of these have a two-operand InstAlias.
let TwoOperandAliasConstraint = "$Vn = $Vd" in {
class VEXTd<string OpcodeStr, string Dt, ValueType Ty, Operand immTy>
  : N3V<0,1,0b11,{?,?,?,?},0,0, (outs DPR:$Vd),
        (ins DPR:$Vn, DPR:$Vm, immTy:$index), NVExtFrm,
        IIC_VEXTD, OpcodeStr, Dt, "$Vd, $Vn, $Vm, $index", "",
        [(set DPR:$Vd, (Ty (NEONvext (Ty DPR:$Vn),
                                     (Ty DPR:$Vm), imm:$index)))]> {
  bits<3> index;
  let Inst{11} = 0b0;
  let Inst{10-8} = index{2-0};
}
```
- EN: Declares reusable TableGen class `VEXTd` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VEXTd`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7118-7127
```tablegen
class VEXTq<string OpcodeStr, string Dt, ValueType Ty, Operand immTy>
  : N3V<0,1,0b11,{?,?,?,?},1,0, (outs QPR:$Vd),
        (ins QPR:$Vn, QPR:$Vm, immTy:$index), NVExtFrm,
        IIC_VEXTQ, OpcodeStr, Dt, "$Vd, $Vn, $Vm, $index", "",
        [(set QPR:$Vd, (Ty (NEONvext (Ty QPR:$Vn),
                                     (Ty QPR:$Vm), imm:$index)))]> {
  bits<4> index;
  let Inst{11-8} = index{3-0};
}
}
```
- EN: Declares reusable TableGen class `VEXTq` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VEXTq`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7129-7141
```tablegen
def VEXTd8  : VEXTd<"vext", "8",  v8i8, imm0_7> {
  let Inst{10-8} = index{2-0};
}
def VEXTd16 : VEXTd<"vext", "16", v4i16, imm0_3> {
  let Inst{10-9} = index{1-0};
  let Inst{8}    = 0b0;
}
let Predicates = [HasNEON] in {
def : Pat<(v4f16 (NEONvext (v4f16 DPR:$Vn), (v4f16 DPR:$Vm), (i32 imm:$index))),
          (VEXTd16 DPR:$Vn, DPR:$Vm, imm:$index)>;
def : Pat<(v4bf16 (NEONvext (v4bf16 DPR:$Vn), (v4bf16 DPR:$Vm), (i32 imm:$index))),
          (VEXTd16 DPR:$Vn, DPR:$Vm, imm:$index)>;
}
```
- EN: Defines TableGen record `VEXTd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VEXTd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7143-7150
```tablegen
def VEXTd32 : VEXTd<"vext", "32", v2i32, imm0_1> {
  let Inst{10}     = index{0};
  let Inst{9-8}    = 0b00;
}
let Predicates = [HasNEON] in {
def : Pat<(v2f32 (NEONvext (v2f32 DPR:$Vn), (v2f32 DPR:$Vm), (i32 imm:$index))),
          (VEXTd32 DPR:$Vn, DPR:$Vm, imm:$index)>;
}
```
- EN: Defines TableGen record `VEXTd32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VEXTd32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7152-7164
```tablegen
def VEXTq8  : VEXTq<"vext", "8",  v16i8, imm0_15> {
  let Inst{11-8} = index{3-0};
}
def VEXTq16 : VEXTq<"vext", "16", v8i16, imm0_7> {
  let Inst{11-9} = index{2-0};
  let Inst{8}    = 0b0;
}
let Predicates = [HasNEON] in {
def : Pat<(v8f16 (NEONvext (v8f16 QPR:$Vn), (v8f16 QPR:$Vm), (i32 imm:$index))),
          (VEXTq16 QPR:$Vn, QPR:$Vm, imm:$index)>;
def : Pat<(v8bf16 (NEONvext (v8bf16 QPR:$Vn), (v8bf16 QPR:$Vm), (i32 imm:$index))),
          (VEXTq16 QPR:$Vn, QPR:$Vm, imm:$index)>;
}
```
- EN: Defines TableGen record `VEXTq8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VEXTq8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7166-7177
```tablegen
def VEXTq32 : VEXTq<"vext", "32", v4i32, imm0_3> {
  let Inst{11-10} = index{1-0};
  let Inst{9-8}    = 0b00;
}
def VEXTq64 : VEXTq<"vext", "64", v2i64, imm0_1> {
  let Inst{11} = index{0};
  let Inst{10-8}    = 0b000;
}
let Predicates = [HasNEON] in {
def : Pat<(v4f32 (NEONvext (v4f32 QPR:$Vn), (v4f32 QPR:$Vm), (i32 imm:$index))),
          (VEXTq32 QPR:$Vn, QPR:$Vm, imm:$index)>;
}
```
- EN: Defines TableGen record `VEXTq32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VEXTq32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7179-7179
```tablegen
//   VTRN     : Vector Transpose
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7181-7183
```tablegen
def  VTRNd8   : N2VDShuffle<0b00, 0b00001, "vtrn", "8">;
def  VTRNd16  : N2VDShuffle<0b01, 0b00001, "vtrn", "16">;
def  VTRNd32  : N2VDShuffle<0b10, 0b00001, "vtrn", "32">;
```
- EN: Defines TableGen record `VTRNd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTRNd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7185-7187
```tablegen
def  VTRNq8   : N2VQShuffle<0b00, 0b00001, IIC_VPERMQ, "vtrn", "8">;
def  VTRNq16  : N2VQShuffle<0b01, 0b00001, IIC_VPERMQ, "vtrn", "16">;
def  VTRNq32  : N2VQShuffle<0b10, 0b00001, IIC_VPERMQ, "vtrn", "32">;
```
- EN: Defines TableGen record `VTRNq8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTRNq8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7189-7189
```tablegen
//   VUZP     : Vector Unzip (Deinterleave)
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7191-7195
```tablegen
def  VUZPd8   : N2VDShuffle<0b00, 0b00010, "vuzp", "8">;
def  VUZPd16  : N2VDShuffle<0b01, 0b00010, "vuzp", "16">;
// vuzp.32 Dd, Dm is a pseudo-instruction expanded to vtrn.32 Dd, Dm.
def : NEONInstAlias<"vuzp${p}.32 $Dd, $Dm",
                    (VTRNd32 DPR:$Dd, DPR:$Dm, pred:$p)>;
```
- EN: Defines TableGen record `VUZPd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VUZPd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7197-7199
```tablegen
def  VUZPq8   : N2VQShuffle<0b00, 0b00010, IIC_VPERMQ3, "vuzp", "8">;
def  VUZPq16  : N2VQShuffle<0b01, 0b00010, IIC_VPERMQ3, "vuzp", "16">;
def  VUZPq32  : N2VQShuffle<0b10, 0b00010, IIC_VPERMQ3, "vuzp", "32">;
```
- EN: Defines TableGen record `VUZPq8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VUZPq8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7201-7201
```tablegen
//   VZIP     : Vector Zip (Interleave)
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7203-7207
```tablegen
def  VZIPd8   : N2VDShuffle<0b00, 0b00011, "vzip", "8">;
def  VZIPd16  : N2VDShuffle<0b01, 0b00011, "vzip", "16">;
// vzip.32 Dd, Dm is a pseudo-instruction expanded to vtrn.32 Dd, Dm.
def : NEONInstAlias<"vzip${p}.32 $Dd, $Dm",
                    (VTRNd32 DPR:$Dd, DPR:$Dm, pred:$p)>;
```
- EN: Defines TableGen record `VZIPd8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VZIPd8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7209-7211
```tablegen
def  VZIPq8   : N2VQShuffle<0b00, 0b00011, IIC_VPERMQ3, "vzip", "8">;
def  VZIPq16  : N2VQShuffle<0b01, 0b00011, IIC_VPERMQ3, "vzip", "16">;
def  VZIPq32  : N2VQShuffle<0b10, 0b00011, IIC_VPERMQ3, "vzip", "32">;
```
- EN: Defines TableGen record `VZIPq8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VZIPq8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7213-7213
```tablegen
// Vector Table Lookup and Table Extension.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7215-7221
```tablegen
//   VTBL     : Vector Table Lookup
let DecoderMethod = "DecodeTBLInstruction" in {
def  VTBL1
  : N3V<1,1,0b11,0b1000,0,0, (outs DPR:$Vd),
        (ins VecListOneD:$Vn, DPR:$Vm), NVTBLFrm, IIC_VTB1,
        "vtbl", "8", "$Vd, $Vn, $Vm", "",
        [(set DPR:$Vd, (v8i8 (NEONvtbl1 VecListOneD:$Vn, DPR:$Vm)))]>;
```
- EN: Defines TableGen record `VTBL1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTBL1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7223-7237
```tablegen
let hasExtraSrcRegAllocReq = 1 in {
def  VTBL2
  : N3V<1,1,0b11,0b1001,0,0, (outs DPR:$Vd),
        (ins VecListDPair:$Vn, DPR:$Vm), NVTBLFrm, IIC_VTB2,
        "vtbl", "8", "$Vd, $Vn, $Vm", "", []>;
def  VTBL3
  : N3V<1,1,0b11,0b1010,0,0, (outs DPR:$Vd),
        (ins VecListThreeD:$Vn, DPR:$Vm), NVTBLFrm, IIC_VTB3,
        "vtbl", "8", "$Vd, $Vn, $Vm", "", []>;
def  VTBL4
  : N3V<1,1,0b11,0b1011,0,0, (outs DPR:$Vd),
        (ins VecListFourD:$Vn, DPR:$Vm),
        NVTBLFrm, IIC_VTB4,
        "vtbl", "8", "$Vd, $Vn, $Vm", "", []>;
} // hasExtraSrcRegAllocReq = 1
```
- EN: Defines TableGen record `VTBL2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTBL2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7239-7242
```tablegen
def  VTBL3Pseudo
  : PseudoNeonI<(outs DPR:$dst), (ins QQPR:$tbl, DPR:$src), IIC_VTB3, "", []>;
def  VTBL4Pseudo
  : PseudoNeonI<(outs DPR:$dst), (ins QQPR:$tbl, DPR:$src), IIC_VTB4, "", []>;
```
- EN: Defines TableGen record `VTBL3Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTBL3Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7244-7261
```tablegen
//   VTBX     : Vector Table Extension
def  VTBX1
  : N3V<1,1,0b11,0b1000,1,0, (outs DPR:$Vd),
        (ins DPR:$orig, VecListOneD:$Vn, DPR:$Vm), NVTBLFrm, IIC_VTBX1,
        "vtbx", "8", "$Vd, $Vn, $Vm", "$orig = $Vd",
        [(set DPR:$Vd, (v8i8 (int_arm_neon_vtbx1
                               DPR:$orig, VecListOneD:$Vn, DPR:$Vm)))]>;
let hasExtraSrcRegAllocReq = 1 in {
def  VTBX2
  : N3V<1,1,0b11,0b1001,1,0, (outs DPR:$Vd),
        (ins DPR:$orig, VecListDPair:$Vn, DPR:$Vm), NVTBLFrm, IIC_VTBX2,
        "vtbx", "8", "$Vd, $Vn, $Vm", "$orig = $Vd", []>;
def  VTBX3
  : N3V<1,1,0b11,0b1010,1,0, (outs DPR:$Vd),
        (ins DPR:$orig, VecListThreeD:$Vn, DPR:$Vm),
        NVTBLFrm, IIC_VTBX3,
        "vtbx", "8", "$Vd, $Vn, $Vm",
        "$orig = $Vd", []>;
```
- EN: Defines TableGen record `VTBX1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTBX1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7262-7267
```tablegen
def  VTBX4
  : N3V<1,1,0b11,0b1011,1,0, (outs DPR:$Vd),
        (ins DPR:$orig, VecListFourD:$Vn, DPR:$Vm), NVTBLFrm, IIC_VTBX4,
        "vtbx", "8", "$Vd, $Vn, $Vm",
        "$orig = $Vd", []>;
} // hasExtraSrcRegAllocReq = 1
```
- EN: Defines TableGen record `VTBX4` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTBX4`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7269-7275
```tablegen
def  VTBX3Pseudo
  : PseudoNeonI<(outs DPR:$dst), (ins DPR:$orig, QQPR:$tbl, DPR:$src),
                IIC_VTBX3, "$orig = $dst", []>;
def  VTBX4Pseudo
  : PseudoNeonI<(outs DPR:$dst), (ins DPR:$orig, QQPR:$tbl, DPR:$src),
                IIC_VTBX4, "$orig = $dst", []>;
} // DecoderMethod = "DecodeTBLInstruction"
```
- EN: Defines TableGen record `VTBX3Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VTBX3Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7277-7287
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v8i8 (NEONvtbl2 v8i8:$Vn0, v8i8:$Vn1, v8i8:$Vm)),
          (v8i8 (VTBL2 (REG_SEQUENCE DPair, v8i8:$Vn0, dsub_0,
                                            v8i8:$Vn1, dsub_1),
                       v8i8:$Vm))>;
def : Pat<(v8i8 (int_arm_neon_vtbx2 v8i8:$orig, v8i8:$Vn0, v8i8:$Vn1,
                                    v8i8:$Vm)),
          (v8i8 (VTBX2 v8i8:$orig,
                       (REG_SEQUENCE DPair, v8i8:$Vn0, dsub_0,
                                            v8i8:$Vn1, dsub_1),
                       v8i8:$Vm))>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7289-7303
```tablegen
def : Pat<(v8i8 (int_arm_neon_vtbl3 v8i8:$Vn0, v8i8:$Vn1,
                                    v8i8:$Vn2, v8i8:$Vm)),
          (v8i8 (VTBL3Pseudo (REG_SEQUENCE QQPR, v8i8:$Vn0, dsub_0,
                                                 v8i8:$Vn1, dsub_1,
                                                 v8i8:$Vn2, dsub_2,
                                                 (v8i8 (IMPLICIT_DEF)), dsub_3),
                             v8i8:$Vm))>;
def : Pat<(v8i8 (int_arm_neon_vtbx3 v8i8:$orig, v8i8:$Vn0, v8i8:$Vn1,
                                    v8i8:$Vn2, v8i8:$Vm)),
          (v8i8 (VTBX3Pseudo v8i8:$orig,
                             (REG_SEQUENCE QQPR, v8i8:$Vn0, dsub_0,
                                                 v8i8:$Vn1, dsub_1,
                                                 v8i8:$Vn2, dsub_2,
                                                 (v8i8 (IMPLICIT_DEF)), dsub_3),
                             v8i8:$Vm))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7305-7320
```tablegen
def : Pat<(v8i8 (int_arm_neon_vtbl4 v8i8:$Vn0, v8i8:$Vn1,
                                    v8i8:$Vn2, v8i8:$Vn3, v8i8:$Vm)),
          (v8i8 (VTBL4Pseudo (REG_SEQUENCE QQPR, v8i8:$Vn0, dsub_0,
                                                 v8i8:$Vn1, dsub_1,
                                                 v8i8:$Vn2, dsub_2,
                                                 v8i8:$Vn3, dsub_3),
                             v8i8:$Vm))>;
def : Pat<(v8i8 (int_arm_neon_vtbx4 v8i8:$orig, v8i8:$Vn0, v8i8:$Vn1,
                                    v8i8:$Vn2, v8i8:$Vn3, v8i8:$Vm)),
          (v8i8 (VTBX4Pseudo v8i8:$orig,
                             (REG_SEQUENCE QQPR, v8i8:$Vn0, dsub_0,
                                                 v8i8:$Vn1, dsub_1,
                                                 v8i8:$Vn2, dsub_2,
                                                 v8i8:$Vn3, dsub_3),
                             v8i8:$Vm))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7322-7339
```tablegen
// VRINT      : Vector Rounding
multiclass VRINT_FPI<string op, bits<3> op9_7, SDPatternOperator Int> {
  let PostEncoderMethod = "NEONThumb2V8PostEncoder", DecoderNamespace = "v8NEON" in {
    def Df : N2VDIntnp<0b10, 0b10, 0b100, 0, NoItinerary,
                      !strconcat("vrint", op), "f32",
                      v2f32, v2f32, Int>, Requires<[HasV8, HasNEON]> {
      let Inst{9-7} = op9_7;
    }
    def Qf : N2VQIntnp<0b10, 0b10, 0b100, 0, NoItinerary,
                      !strconcat("vrint", op), "f32",
                      v4f32, v4f32, Int>, Requires<[HasV8, HasNEON]> {
      let Inst{9-7} = op9_7;
    }
    def Dh : N2VDIntnp<0b01, 0b10, 0b100, 0, NoItinerary,
                      !strconcat("vrint", op), "f16",
                      v4f16, v4f16, Int>,
             Requires<[HasV8, HasNEON, HasFullFP16]> {
      let Inst{9-7} = op9_7;
```
- EN: Declares TableGen `multiclass VRINT_FPI`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VRINT_FPI`，它是一个可复用模板，可展开为多个相关记录。

### Lines 7340-7347
```tablegen
    }
    def Qh : N2VQIntnp<0b01, 0b10, 0b100, 0, NoItinerary,
                      !strconcat("vrint", op), "f16",
                      v8f16, v8f16, Int>,
             Requires<[HasV8, HasNEON, HasFullFP16]> {
      let Inst{9-7} = op9_7;
    }
  }
```
- EN: Defines TableGen record `Qh` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Qh`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7349-7359
```tablegen
  def : NEONInstAlias<!strconcat("vrint", op, ".f32.f32\t$Dd, $Dm"),
                  (!cast<Instruction>(NAME#"Df") DPR:$Dd, DPR:$Dm)>;
  def : NEONInstAlias<!strconcat("vrint", op, ".f32.f32\t$Qd, $Qm"),
                  (!cast<Instruction>(NAME#"Qf") QPR:$Qd, QPR:$Qm)>;
  let Predicates = [HasNEON, HasFullFP16] in {
  def : NEONInstAlias<!strconcat("vrint", op, ".f16.f16\t$Dd, $Dm"),
                  (!cast<Instruction>(NAME#"Dh") DPR:$Dd, DPR:$Dm)>;
  def : NEONInstAlias<!strconcat("vrint", op, ".f16.f16\t$Qd, $Qm"),
                  (!cast<Instruction>(NAME#"Qh") QPR:$Qd, QPR:$Qm)>;
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7361-7366
```tablegen
defm VRINTNN : VRINT_FPI<"n", 0b000, any_froundeven>;
defm VRINTXN : VRINT_FPI<"x", 0b001, any_frint>;
defm VRINTAN : VRINT_FPI<"a", 0b010, any_fround>;
defm VRINTZN : VRINT_FPI<"z", 0b011, any_ftrunc>;
defm VRINTMN : VRINT_FPI<"m", 0b101, any_ffloor>;
defm VRINTPN : VRINT_FPI<"p", 0b111, any_fceil>;
```
- EN: Defines TableGen record `VRINTNN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VRINTNN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7368-7385
```tablegen
// Cryptography instructions
let PostEncoderMethod = "NEONThumb2DataIPostEncoder",
    DecoderNamespace = "v8Crypto", hasSideEffects = 0 in {
  class AES<string op, bit op7, bit op6, SDPatternOperator Int>
    : N2VQIntXnp<0b00, 0b00, 0b011, op6, op7, NoItinerary,
                 !strconcat("aes", op), "8", v16i8, v16i8, Int>;
  class AES2Op<string op, bit op7, bit op6, SDPatternOperator Int>
    : N2VQIntX2np<0b00, 0b00, 0b011, op6, op7, NoItinerary,
                 !strconcat("aes", op), "8", v16i8, v16i8, Int>;
  class N2SHA<string op, bits<2> op17_16, bits<3> op10_8, bit op7, bit op6,
              SDPatternOperator Int>
    : N2VQIntXnp<0b10, op17_16, op10_8, op6, op7, NoItinerary,
                 !strconcat("sha", op), "32", v4i32, v4i32, Int>;
  class N2SHA2Op<string op, bits<2> op17_16, bits<3> op10_8, bit op7, bit op6,
              SDPatternOperator Int>
    : N2VQIntX2np<0b10, op17_16, op10_8, op6, op7, NoItinerary,
                 !strconcat("sha", op), "32", v4i32, v4i32, Int>;
  class N3SHA3Op<string op, bits<5> op27_23, bits<2> op21_20, SDPatternOperator Int>
```
- EN: Declares reusable TableGen class `AES` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `AES`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7386-7388
```tablegen
    : N3VQInt3np<op27_23, op21_20, 0b1100, 1, 0, N3RegFrm, NoItinerary,
                !strconcat("sha", op), "32", v4i32, v4i32, Int>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7390-7397
```tablegen
let Predicates = [HasV8, HasAES] in {
let isCommutable = 1 in {
def AESD : AES2Op<"d", 0, 1, int_arm_neon_aesd>;
def AESE : AES2Op<"e", 0, 0, int_arm_neon_aese>;
}
def AESIMC : AES<"imc", 1, 1, int_arm_neon_aesimc>;
def AESMC : AES<"mc", 1, 0, int_arm_neon_aesmc>;
}
```
- EN: Defines TableGen record `AESD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AESD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7399-7410
```tablegen
let Predicates = [HasV8, HasSHA2] in {
def SHA1H : N2SHA<"1h", 0b01, 0b010, 1, 1, null_frag>;
def SHA1SU1 : N2SHA2Op<"1su1", 0b10, 0b011, 1, 0, int_arm_neon_sha1su1>;
def SHA256SU0 : N2SHA2Op<"256su0", 0b10, 0b011, 1, 1, int_arm_neon_sha256su0>;
def SHA1C : N3SHA3Op<"1c", 0b00100, 0b00, null_frag>;
def SHA1M : N3SHA3Op<"1m", 0b00100, 0b10, null_frag>;
def SHA1P : N3SHA3Op<"1p", 0b00100, 0b01, null_frag>;
def SHA1SU0 : N3SHA3Op<"1su0", 0b00100, 0b11, int_arm_neon_sha1su0>;
def SHA256H : N3SHA3Op<"256h", 0b00110, 0b00, int_arm_neon_sha256h>;
def SHA256H2 : N3SHA3Op<"256h2", 0b00110, 0b01, int_arm_neon_sha256h2>;
def SHA256SU1 : N3SHA3Op<"256su1", 0b00110, 0b10, int_arm_neon_sha256su1>;
}
```
- EN: Defines TableGen record `SHA1H` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SHA1H`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7412-7418
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(i32 (int_arm_neon_sha1h i32:$Rn)),
          (COPY_TO_REGCLASS (f32 (EXTRACT_SUBREG
              (SHA1H (INSERT_SUBREG (IMPLICIT_DEF),
                                    (f32 (COPY_TO_REGCLASS i32:$Rn, SPR)),
                                    ssub_0)),
              ssub_0)), GPR)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7420-7425
```tablegen
def : Pat<(v4i32 (int_arm_neon_sha1c v4i32:$hash_abcd, i32:$hash_e, v4i32:$wk)),
          (SHA1C v4i32:$hash_abcd,
                 (INSERT_SUBREG (IMPLICIT_DEF),
                                (f32 (COPY_TO_REGCLASS i32:$hash_e, SPR)),
                                ssub_0),
                 v4i32:$wk)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7427-7432
```tablegen
def : Pat<(v4i32 (int_arm_neon_sha1m v4i32:$hash_abcd, i32:$hash_e, v4i32:$wk)),
          (SHA1M v4i32:$hash_abcd,
                 (INSERT_SUBREG (IMPLICIT_DEF),
                                (f32 (COPY_TO_REGCLASS i32:$hash_e, SPR)),
                                ssub_0),
                 v4i32:$wk)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7434-7440
```tablegen
def : Pat<(v4i32 (int_arm_neon_sha1p v4i32:$hash_abcd, i32:$hash_e, v4i32:$wk)),
          (SHA1P v4i32:$hash_abcd,
                 (INSERT_SUBREG (IMPLICIT_DEF),
                                (f32 (COPY_TO_REGCLASS i32:$hash_e, SPR)),
                                ssub_0),
                 v4i32:$wk)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7442-7444
```tablegen
//===----------------------------------------------------------------------===//
// NEON instructions for single-precision FP math
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 7446-7452
```tablegen
class N2VSPat<SDNode OpNode, NeonI Inst>
  : NEONFPPat<(f32 (OpNode SPR:$a)),
              (EXTRACT_SUBREG
               (v2f32 (COPY_TO_REGCLASS (Inst
                (INSERT_SUBREG
                 (v2f32 (COPY_TO_REGCLASS (v2f32 (IMPLICIT_DEF)), DPR_VFP2)),
                 SPR:$a, ssub_0)), DPR_VFP2)), ssub_0)>;
```
- EN: Declares reusable TableGen class `N2VSPat` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N2VSPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7454-7463
```tablegen
class N3VSPat<SDNode OpNode, NeonI Inst>
  : NEONFPPat<(f32 (OpNode SPR:$a, SPR:$b)),
              (EXTRACT_SUBREG
               (v2f32 (COPY_TO_REGCLASS (Inst
                (INSERT_SUBREG
                 (v2f32 (COPY_TO_REGCLASS (v2f32 (IMPLICIT_DEF)), DPR_VFP2)),
                 SPR:$a, ssub_0),
                (INSERT_SUBREG
                 (v2f32 (COPY_TO_REGCLASS (v2f32 (IMPLICIT_DEF)), DPR_VFP2)),
                 SPR:$b, ssub_0)), DPR_VFP2)), ssub_0)>;
```
- EN: Declares reusable TableGen class `N3VSPat` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VSPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7465-7474
```tablegen
class N3VSPatFP16<SDNode OpNode, NeonI Inst>
  : NEONFPPat<(f16 (OpNode HPR:$a, HPR:$b)),
              (EXTRACT_SUBREG
               (v4f16 (COPY_TO_REGCLASS (Inst
                (INSERT_SUBREG
                 (v4f16 (COPY_TO_REGCLASS (v4f16 (IMPLICIT_DEF)), DPR_VFP2)),
                 HPR:$a, ssub_0),
                (INSERT_SUBREG
                 (v4f16 (COPY_TO_REGCLASS (v4f16 (IMPLICIT_DEF)), DPR_VFP2)),
                 HPR:$b, ssub_0)), DPR_VFP2)), ssub_0)>;
```
- EN: Declares reusable TableGen class `N3VSPatFP16` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VSPatFP16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7476-7488
```tablegen
class N3VSMulOpPat<SDNode MulNode, SDNode OpNode, NeonI Inst>
  : NEONFPPat<(f32 (OpNode SPR:$acc, (f32 (MulNode SPR:$a, SPR:$b)))),
              (EXTRACT_SUBREG
               (v2f32 (COPY_TO_REGCLASS (Inst
                (INSERT_SUBREG
                 (v2f32 (COPY_TO_REGCLASS (v2f32 (IMPLICIT_DEF)), DPR_VFP2)),
                 SPR:$acc, ssub_0),
                (INSERT_SUBREG
                 (v2f32 (COPY_TO_REGCLASS (v2f32 (IMPLICIT_DEF)), DPR_VFP2)),
                 SPR:$a, ssub_0),
                (INSERT_SUBREG
                 (v2f32 (COPY_TO_REGCLASS (v2f32 (IMPLICIT_DEF)), DPR_VFP2)),
                 SPR:$b, ssub_0)), DPR_VFP2)), ssub_0)>;
```
- EN: Declares reusable TableGen class `N3VSMulOpPat` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `N3VSMulOpPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7490-7503
```tablegen
class NVCVTIFPat<SDNode OpNode, NeonI Inst>
  : NEONFPPat<(f32 (OpNode GPR:$a)),
              (f32 (EXTRACT_SUBREG
                     (v2f32 (Inst
                       (INSERT_SUBREG
                         (v2f32 (IMPLICIT_DEF)),
                         (i32 (COPY_TO_REGCLASS GPR:$a, SPR)), ssub_0))),
                     ssub_0))>;
class NVCVTFIPat<SDNode OpNode, NeonI Inst>
  : NEONFPPat<(i32 (OpNode SPR:$a)),
              (i32 (EXTRACT_SUBREG
                     (v2f32 (Inst (INSERT_SUBREG (v2f32 (IMPLICIT_DEF)),
                                                 SPR:$a, ssub_0))),
                     ssub_0))>;
```
- EN: Declares reusable TableGen class `NVCVTIFPat` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `NVCVTIFPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 7505-7522
```tablegen
def : N3VSPat<fadd, VADDfd>;
def : N3VSPat<fsub, VSUBfd>;
def : N3VSPat<fmul, VMULfd>;
def : N3VSMulOpPat<fmul, fadd, VMLAfd>,
      Requires<[HasNEON, UseNEONForFP, UseFPVMLx]>;
def : N3VSMulOpPat<fmul, fsub, VMLSfd>,
      Requires<[HasNEON, UseNEONForFP, UseFPVMLx]>;
def : N3VSMulOpPat<fmul, fadd, VFMAfd>,
      Requires<[HasVFP4, UseNEONForFP, UseFusedMAC]>;
def : N3VSMulOpPat<fmul, fsub, VFMSfd>,
      Requires<[HasVFP4, UseNEONForFP, UseFusedMAC]>;
def : N2VSPat<fabs, VABSfd>;
def : N2VSPat<fneg, VNEGfd>;
def : N3VSPatFP16<fmaximum, VMAXhd>, Requires<[HasFullFP16]>;
def : N3VSPatFP16<fminimum, VMINhd>, Requires<[HasFullFP16]>;
def : N3VSPat<fmaximum, VMAXfd>, Requires<[HasNEON]>;
def : N3VSPat<fminimum, VMINfd>, Requires<[HasNEON]>;
def : NVCVTFIPat<fp_to_sint, VCVTf2sd>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7523-7525
```tablegen
def : NVCVTFIPat<fp_to_uint, VCVTf2ud>;
def : NVCVTIFPat<sint_to_fp, VCVTs2fd>;
def : NVCVTIFPat<uint_to_fp, VCVTu2fd>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7527-7536
```tablegen
// NEON doesn't have any f64 conversions, so provide patterns to make
// sure the VFP conversions match when extracting from a vector.
def : VFPPat<(f64 (sint_to_fp (extractelt (v2i32 DPR:$src), imm:$lane))),
             (VSITOD (EXTRACT_SUBREG DPR:$src, (SSubReg_f32_reg imm:$lane)))>;
def : VFPPat<(f64 (sint_to_fp (extractelt (v4i32 QPR:$src), imm:$lane))),
             (VSITOD (EXTRACT_SUBREG QPR:$src, (SSubReg_f32_reg imm:$lane)))>;
def : VFPPat<(f64 (uint_to_fp (extractelt (v2i32 DPR:$src), imm:$lane))),
             (VUITOD (EXTRACT_SUBREG DPR:$src, (SSubReg_f32_reg imm:$lane)))>;
def : VFPPat<(f64 (uint_to_fp (extractelt (v4i32 QPR:$src), imm:$lane))),
             (VUITOD (EXTRACT_SUBREG QPR:$src, (SSubReg_f32_reg imm:$lane)))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7539-7545
```tablegen
// Prefer VMOVDRR for i32 -> f32 bitcasts, it can write all DPR registers.
def : Pat<(f32 (bitconvert GPR:$a)),
          (EXTRACT_SUBREG (VMOVDRR GPR:$a, GPR:$a), ssub_0)>,
        Requires<[HasNEON, DontUseVMOVSR]>;
def : Pat<(arm_vmovsr GPR:$a),
          (EXTRACT_SUBREG (VMOVDRR GPR:$a, GPR:$a), ssub_0)>,
        Requires<[HasNEON, DontUseVMOVSR]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7547-7549
```tablegen
//===----------------------------------------------------------------------===//
// Non-Instruction Patterns or Endianess - Revert Patterns
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 7551-7555
```tablegen
// bit_convert
// 64 bit conversions
let Predicates = [HasNEON] in {
def : Pat<(f64   (bitconvert (v1i64 DPR:$src))), (f64   DPR:$src)>;
def : Pat<(v1i64 (bitconvert (f64   DPR:$src))), (v1i64 DPR:$src)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7557-7558
```tablegen
def : Pat<(v2f32 (bitconvert (v2i32 DPR:$src))), (v2f32 DPR:$src)>;
def : Pat<(v2i32 (bitconvert (v2f32 DPR:$src))), (v2i32 DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7560-7561
```tablegen
def : Pat<(v4i16 (bitconvert (v4f16 DPR:$src))), (v4i16  DPR:$src)>;
def : Pat<(v4f16 (bitconvert (v4i16 DPR:$src))), (v4f16  DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7563-7564
```tablegen
def : Pat<(v4i16 (bitconvert (v4bf16 DPR:$src))), (v4i16  DPR:$src)>;
def : Pat<(v4bf16 (bitconvert (v4i16 DPR:$src))), (v4bf16  DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7566-7568
```tablegen
// 128 bit conversions
def : Pat<(v2f64 (bitconvert (v2i64 QPR:$src))), (v2f64 QPR:$src)>;
def : Pat<(v2i64 (bitconvert (v2f64 QPR:$src))), (v2i64 QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7570-7571
```tablegen
def : Pat<(v4i32 (bitconvert (v4f32 QPR:$src))), (v4i32 QPR:$src)>;
def : Pat<(v4f32 (bitconvert (v4i32 QPR:$src))), (v4f32 QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7573-7574
```tablegen
def : Pat<(v8i16 (bitconvert (v8f16 QPR:$src))), (v8i16  QPR:$src)>;
def : Pat<(v8f16 (bitconvert (v8i16 QPR:$src))), (v8f16  QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7576-7578
```tablegen
def : Pat<(v8i16 (bitconvert (v8bf16 QPR:$src))), (v8i16  QPR:$src)>;
def : Pat<(v8bf16 (bitconvert (v8i16 QPR:$src))), (v8bf16  QPR:$src)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7580-7587
```tablegen
let Predicates = [IsLE,HasNEON] in {
  // 64 bit conversions
  def : Pat<(f64   (bitconvert (v2f32 DPR:$src))), (f64   DPR:$src)>;
  def : Pat<(f64   (bitconvert (v2i32 DPR:$src))), (f64   DPR:$src)>;
  def : Pat<(f64   (bitconvert (v4f16 DPR:$src))), (f64   DPR:$src)>;
  def : Pat<(f64   (bitconvert (v4bf16 DPR:$src))), (f64   DPR:$src)>;
  def : Pat<(f64   (bitconvert (v4i16 DPR:$src))), (f64   DPR:$src)>;
  def : Pat<(f64   (bitconvert (v8i8  DPR:$src))), (f64   DPR:$src)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7589-7594
```tablegen
  def : Pat<(v1i64 (bitconvert (v2f32 DPR:$src))), (v1i64 DPR:$src)>;
  def : Pat<(v1i64 (bitconvert (v2i32 DPR:$src))), (v1i64 DPR:$src)>;
  def : Pat<(v1i64 (bitconvert (v4f16 DPR:$src))), (v1i64 DPR:$src)>;
  def : Pat<(v1i64 (bitconvert (v4bf16 DPR:$src))), (v1i64 DPR:$src)>;
  def : Pat<(v1i64 (bitconvert (v4i16 DPR:$src))), (v1i64 DPR:$src)>;
  def : Pat<(v1i64 (bitconvert (v8i8  DPR:$src))), (v1i64 DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7596-7601
```tablegen
  def : Pat<(v2f32 (bitconvert (f64   DPR:$src))), (v2f32 DPR:$src)>;
  def : Pat<(v2f32 (bitconvert (v1i64 DPR:$src))), (v2f32 DPR:$src)>;
  def : Pat<(v2f32 (bitconvert (v4f16 DPR:$src))), (v2f32 DPR:$src)>;
  def : Pat<(v2f32 (bitconvert (v4bf16 DPR:$src))), (v2f32 DPR:$src)>;
  def : Pat<(v2f32 (bitconvert (v4i16 DPR:$src))), (v2f32 DPR:$src)>;
  def : Pat<(v2f32 (bitconvert (v8i8  DPR:$src))), (v2f32 DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7603-7608
```tablegen
  def : Pat<(v2i32 (bitconvert (f64   DPR:$src))), (v2i32 DPR:$src)>;
  def : Pat<(v2i32 (bitconvert (v1i64 DPR:$src))), (v2i32 DPR:$src)>;
  def : Pat<(v2i32 (bitconvert (v4f16 DPR:$src))), (v2i32 DPR:$src)>;
  def : Pat<(v2i32 (bitconvert (v4bf16 DPR:$src))), (v2i32 DPR:$src)>;
  def : Pat<(v2i32 (bitconvert (v4i16 DPR:$src))), (v2i32 DPR:$src)>;
  def : Pat<(v2i32 (bitconvert (v8i8  DPR:$src))), (v2i32 DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7610-7614
```tablegen
  def : Pat<(v4f16 (bitconvert (f64   DPR:$src))), (v4f16 DPR:$src)>;
  def : Pat<(v4f16 (bitconvert (v1i64 DPR:$src))), (v4f16 DPR:$src)>;
  def : Pat<(v4f16 (bitconvert (v2f32 DPR:$src))), (v4f16 DPR:$src)>;
  def : Pat<(v4f16 (bitconvert (v2i32 DPR:$src))), (v4f16 DPR:$src)>;
  def : Pat<(v4f16 (bitconvert (v8i8  DPR:$src))), (v4f16 DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7616-7620
```tablegen
  def : Pat<(v4bf16 (bitconvert (f64   DPR:$src))), (v4bf16 DPR:$src)>;
  def : Pat<(v4bf16 (bitconvert (v1i64 DPR:$src))), (v4bf16 DPR:$src)>;
  def : Pat<(v4bf16 (bitconvert (v2f32 DPR:$src))), (v4bf16 DPR:$src)>;
  def : Pat<(v4bf16 (bitconvert (v2i32 DPR:$src))), (v4bf16 DPR:$src)>;
  def : Pat<(v4bf16 (bitconvert (v8i8  DPR:$src))), (v4bf16 DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7622-7626
```tablegen
  def : Pat<(v4i16 (bitconvert (f64   DPR:$src))), (v4i16 DPR:$src)>;
  def : Pat<(v4i16 (bitconvert (v1i64 DPR:$src))), (v4i16 DPR:$src)>;
  def : Pat<(v4i16 (bitconvert (v2f32 DPR:$src))), (v4i16 DPR:$src)>;
  def : Pat<(v4i16 (bitconvert (v2i32 DPR:$src))), (v4i16 DPR:$src)>;
  def : Pat<(v4i16 (bitconvert (v8i8  DPR:$src))), (v4i16 DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7628-7634
```tablegen
  def : Pat<(v8i8  (bitconvert (f64   DPR:$src))), (v8i8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v1i64 DPR:$src))), (v8i8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v2f32 DPR:$src))), (v8i8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v2i32 DPR:$src))), (v8i8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v4f16 DPR:$src))), (v8i8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v4bf16 DPR:$src))), (v8i8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v4i16 DPR:$src))), (v8i8  DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7636-7642
```tablegen
  // 128 bit conversions
  def : Pat<(v2f64 (bitconvert (v4f32 QPR:$src))), (v2f64 QPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v4i32 QPR:$src))), (v2f64 QPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v8f16 QPR:$src))), (v2f64 QPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v8bf16 QPR:$src))), (v2f64 QPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v8i16 QPR:$src))), (v2f64 QPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v16i8 QPR:$src))), (v2f64 QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7644-7649
```tablegen
  def : Pat<(v2i64 (bitconvert (v4f32 QPR:$src))), (v2i64 QPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v4i32 QPR:$src))), (v2i64 QPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v8f16 QPR:$src))), (v2i64 QPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v8bf16 QPR:$src))), (v2i64 QPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v8i16 QPR:$src))), (v2i64 QPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v16i8 QPR:$src))), (v2i64 QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7651-7656
```tablegen
  def : Pat<(v4f32 (bitconvert (v2f64 QPR:$src))), (v4f32 QPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v2i64 QPR:$src))), (v4f32 QPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v8f16 QPR:$src))), (v4f32 QPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v8bf16 QPR:$src))), (v4f32 QPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v8i16 QPR:$src))), (v4f32 QPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v16i8 QPR:$src))), (v4f32 QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7658-7663
```tablegen
  def : Pat<(v4i32 (bitconvert (v2f64 QPR:$src))), (v4i32 QPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v2i64 QPR:$src))), (v4i32 QPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v8f16 QPR:$src))), (v4i32 QPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v8bf16 QPR:$src))), (v4i32 QPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v8i16 QPR:$src))), (v4i32 QPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v16i8 QPR:$src))), (v4i32 QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7665-7669
```tablegen
  def : Pat<(v8f16 (bitconvert (v2f64 QPR:$src))), (v8f16 QPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v2i64 QPR:$src))), (v8f16 QPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v4f32 QPR:$src))), (v8f16 QPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v4i32 QPR:$src))), (v8f16 QPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v16i8 QPR:$src))), (v8f16 QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7671-7675
```tablegen
  def : Pat<(v8bf16 (bitconvert (v2f64 QPR:$src))), (v8bf16 QPR:$src)>;
  def : Pat<(v8bf16 (bitconvert (v2i64 QPR:$src))), (v8bf16 QPR:$src)>;
  def : Pat<(v8bf16 (bitconvert (v4f32 QPR:$src))), (v8bf16 QPR:$src)>;
  def : Pat<(v8bf16 (bitconvert (v4i32 QPR:$src))), (v8bf16 QPR:$src)>;
  def : Pat<(v8bf16 (bitconvert (v16i8 QPR:$src))), (v8bf16 QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7677-7681
```tablegen
  def : Pat<(v8i16 (bitconvert (v2f64 QPR:$src))), (v8i16 QPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v2i64 QPR:$src))), (v8i16 QPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v4f32 QPR:$src))), (v8i16 QPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v4i32 QPR:$src))), (v8i16 QPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v16i8 QPR:$src))), (v8i16 QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7683-7690
```tablegen
  def : Pat<(v16i8 (bitconvert (v2f64 QPR:$src))), (v16i8 QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v2i64 QPR:$src))), (v16i8 QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v4f32 QPR:$src))), (v16i8 QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v4i32 QPR:$src))), (v16i8 QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v8f16 QPR:$src))), (v16i8 QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v8bf16 QPR:$src))), (v16i8 QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v8i16 QPR:$src))), (v16i8 QPR:$src)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7692-7699
```tablegen
let Predicates = [IsBE,HasNEON] in {
  // 64 bit conversions
  def : Pat<(f64   (bitconvert (v2f32 DPR:$src))), (VREV64d32 DPR:$src)>;
  def : Pat<(f64   (bitconvert (v2i32 DPR:$src))), (VREV64d32 DPR:$src)>;
  def : Pat<(f64   (bitconvert (v4f16 DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(f64   (bitconvert (v4bf16 DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(f64   (bitconvert (v4i16 DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(f64   (bitconvert (v8i8  DPR:$src))), (VREV64d8  DPR:$src)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7701-7706
```tablegen
  def : Pat<(v1i64 (bitconvert (v2f32 DPR:$src))), (VREV64d32 DPR:$src)>;
  def : Pat<(v1i64 (bitconvert (v2i32 DPR:$src))), (VREV64d32 DPR:$src)>;
  def : Pat<(v1i64 (bitconvert (v4f16 DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(v1i64 (bitconvert (v4bf16 DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(v1i64 (bitconvert (v4i16 DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(v1i64 (bitconvert (v8i8  DPR:$src))), (VREV64d8  DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7708-7713
```tablegen
  def : Pat<(v2f32 (bitconvert (f64   DPR:$src))), (VREV64d32 DPR:$src)>;
  def : Pat<(v2f32 (bitconvert (v1i64 DPR:$src))), (VREV64d32 DPR:$src)>;
  def : Pat<(v2f32 (bitconvert (v4f16 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v2f32 (bitconvert (v4bf16 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v2f32 (bitconvert (v4i16 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v2f32 (bitconvert (v8i8  DPR:$src))), (VREV32d8  DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7715-7720
```tablegen
  def : Pat<(v2i32 (bitconvert (f64   DPR:$src))), (VREV64d32 DPR:$src)>;
  def : Pat<(v2i32 (bitconvert (v1i64 DPR:$src))), (VREV64d32 DPR:$src)>;
  def : Pat<(v2i32 (bitconvert (v4f16 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v2i32 (bitconvert (v4bf16 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v2i32 (bitconvert (v4i16 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v2i32 (bitconvert (v8i8  DPR:$src))), (VREV32d8  DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7722-7726
```tablegen
  def : Pat<(v4f16 (bitconvert (f64   DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(v4f16 (bitconvert (v1i64 DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(v4f16 (bitconvert (v2f32 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v4f16 (bitconvert (v2i32 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v4f16 (bitconvert (v8i8  DPR:$src))), (VREV16d8  DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7728-7732
```tablegen
  def : Pat<(v4bf16 (bitconvert (f64   DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(v4bf16 (bitconvert (v1i64 DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(v4bf16 (bitconvert (v2f32 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v4bf16 (bitconvert (v2i32 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v4bf16 (bitconvert (v8i8  DPR:$src))), (VREV16d8  DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7734-7738
```tablegen
  def : Pat<(v4i16 (bitconvert (f64   DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(v4i16 (bitconvert (v1i64 DPR:$src))), (VREV64d16 DPR:$src)>;
  def : Pat<(v4i16 (bitconvert (v2f32 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v4i16 (bitconvert (v2i32 DPR:$src))), (VREV32d16 DPR:$src)>;
  def : Pat<(v4i16 (bitconvert (v8i8  DPR:$src))), (VREV16d8  DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7740-7746
```tablegen
  def : Pat<(v8i8  (bitconvert (f64   DPR:$src))), (VREV64d8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v1i64 DPR:$src))), (VREV64d8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v2f32 DPR:$src))), (VREV32d8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v2i32 DPR:$src))), (VREV32d8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v4f16 DPR:$src))), (VREV16d8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v4bf16 DPR:$src))), (VREV16d8  DPR:$src)>;
  def : Pat<(v8i8  (bitconvert (v4i16 DPR:$src))), (VREV16d8  DPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7748-7754
```tablegen
  // 128 bit conversions
  def : Pat<(v2f64 (bitconvert (v4f32 QPR:$src))), (VREV64q32 QPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v4i32 QPR:$src))), (VREV64q32 QPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v8f16 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v8bf16 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v8i16 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v2f64 (bitconvert (v16i8 QPR:$src))), (VREV64q8  QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7756-7761
```tablegen
  def : Pat<(v2i64 (bitconvert (v4f32 QPR:$src))), (VREV64q32 QPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v4i32 QPR:$src))), (VREV64q32 QPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v8f16 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v8bf16 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v8i16 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v2i64 (bitconvert (v16i8 QPR:$src))), (VREV64q8  QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7763-7768
```tablegen
  def : Pat<(v4f32 (bitconvert (v2f64 QPR:$src))), (VREV64q32 QPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v2i64 QPR:$src))), (VREV64q32 QPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v8f16 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v8bf16 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v8i16 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v4f32 (bitconvert (v16i8 QPR:$src))), (VREV32q8  QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7770-7775
```tablegen
  def : Pat<(v4i32 (bitconvert (v2f64 QPR:$src))), (VREV64q32 QPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v2i64 QPR:$src))), (VREV64q32 QPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v8f16 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v8bf16 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v8i16 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v4i32 (bitconvert (v16i8 QPR:$src))), (VREV32q8  QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7777-7781
```tablegen
  def : Pat<(v8f16 (bitconvert (v2f64 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v2i64 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v4f32 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v4i32 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v8f16 (bitconvert (v16i8 QPR:$src))), (VREV16q8  QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7783-7787
```tablegen
  def : Pat<(v8bf16 (bitconvert (v2f64 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v8bf16 (bitconvert (v2i64 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v8bf16 (bitconvert (v4f32 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v8bf16 (bitconvert (v4i32 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v8bf16 (bitconvert (v16i8 QPR:$src))), (VREV16q8  QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7789-7793
```tablegen
  def : Pat<(v8i16 (bitconvert (v2f64 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v2i64 QPR:$src))), (VREV64q16 QPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v4f32 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v4i32 QPR:$src))), (VREV32q16 QPR:$src)>;
  def : Pat<(v8i16 (bitconvert (v16i8 QPR:$src))), (VREV16q8  QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7795-7802
```tablegen
  def : Pat<(v16i8 (bitconvert (v2f64 QPR:$src))), (VREV64q8  QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v2i64 QPR:$src))), (VREV64q8  QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v4f32 QPR:$src))), (VREV32q8  QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v4i32 QPR:$src))), (VREV32q8  QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v8f16 QPR:$src))), (VREV16q8  QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v8bf16 QPR:$src))), (VREV16q8  QPR:$src)>;
  def : Pat<(v16i8 (bitconvert (v8i16 QPR:$src))), (VREV16q8  QPR:$src)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7804-7809
```tablegen
let Predicates = [HasNEON] in {
  // Here we match the specific SDNode type 'ARMVectorRegCastImpl'
  // rather than the more general 'ARMVectorRegCast' which would also
  // match some bitconverts. If we use the latter in cases where the
  // input and output types are the same, the bitconvert gets elided
  // and we end up generating a nonsense match of nothing.
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7811-7813
```tablegen
  foreach VT = [ v16i8, v8i16, v8f16, v8bf16, v4i32, v4f32, v2i64, v2f64 ] in
    foreach VT2 = [ v16i8, v8i16, v8f16, v8bf16, v4i32, v4f32, v2i64, v2f64 ] in
      def : Pat<(VT (ARMVectorRegCastImpl (VT2 QPR:$src))), (VT QPR:$src)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7815-7818
```tablegen
  foreach VT = [ v8i8, v4i16, v4f16, v4bf16, v2i32, v2f32, v1i64, f64 ] in
    foreach VT2 = [ v8i8, v4i16, v4f16, v4bf16, v2i32, v2f32, v1i64, f64 ] in
      def : Pat<(VT (ARMVectorRegCastImpl (VT2 DPR:$src))), (VT DPR:$src)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7820-7830
```tablegen
// Use VLD1/VST1 + VREV for non-word-aligned v2f64 load/store on Big Endian
let Predicates = [IsBE,HasNEON] in {
def : Pat<(v2f64 (byte_alignedload addrmode6:$addr)),
          (VREV64q8 (VLD1q8 addrmode6:$addr))>;
def : Pat<(byte_alignedstore (v2f64 QPR:$value), addrmode6:$addr),
          (VST1q8 addrmode6:$addr, (VREV64q8 QPR:$value))>;
def : Pat<(v2f64 (hword_alignedload addrmode6:$addr)),
          (VREV64q16 (VLD1q16 addrmode6:$addr))>;
def : Pat<(hword_alignedstore (v2f64 QPR:$value), addrmode6:$addr),
          (VST1q16 addrmode6:$addr, (VREV64q16 QPR:$value))>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 7832-7835
```tablegen
// Fold extracting an element out of a v2i32 into a vfp register.
def : Pat<(f32 (bitconvert (i32 (extractelt (v2i32 DPR:$src), imm:$lane)))),
          (f32 (EXTRACT_SUBREG DPR:$src, (SSubReg_f32_reg imm:$lane)))>,
      Requires<[HasNEON]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 7837-7837
```tablegen
// Vector lengthening move with load, matching extending loads.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 7839-7850
```tablegen
// extload, zextload and sextload for a standard lengthening load. Example:
// Lengthen_Single<"8", "i16", "8"> =
//     Pat<(v8i16 (extloadvi8 addrmode6:$addr))
//         (VMOVLuv8i16 (VLD1d8 addrmode6:$addr,
//                              (f64 (IMPLICIT_DEF)), (i32 0)))>;
multiclass Lengthen_Single<string DestLanes, string DestTy, string SrcTy> {
  let AddedComplexity = 10 in {
  def _Any : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                    (!cast<PatFrag>("extloadvi" # SrcTy) addrmode6:$addr)),
                  (!cast<Instruction>("VMOVLuv" # DestLanes # DestTy)
                    (!cast<Instruction>("VLD1d" # SrcTy) addrmode6:$addr))>,
             Requires<[HasNEON]>;
```
- EN: Declares TableGen `multiclass Lengthen_Single`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass Lengthen_Single`，它是一个可复用模板，可展开为多个相关记录。

### Lines 7852-7856
```tablegen
  def _Z : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                  (!cast<PatFrag>("zextloadvi" # SrcTy) addrmode6:$addr)),
                (!cast<Instruction>("VMOVLuv" # DestLanes # DestTy)
                    (!cast<Instruction>("VLD1d" # SrcTy) addrmode6:$addr))>,
           Requires<[HasNEON]>;
```
- EN: Defines TableGen record `_Z` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_Z`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7858-7864
```tablegen
  def _S : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                  (!cast<PatFrag>("sextloadvi" # SrcTy) addrmode6:$addr)),
                (!cast<Instruction>("VMOVLsv" # DestLanes # DestTy)
                    (!cast<Instruction>("VLD1d" # SrcTy) addrmode6:$addr))>,
           Requires<[HasNEON]>;
  }
}
```
- EN: Defines TableGen record `_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7866-7883
```tablegen
// extload, zextload and sextload for a lengthening load which only uses
// half the lanes available. Example:
// Lengthen_HalfSingle<"4", "i16", "8", "i16", "i8"> =
//     Pat<(v4i16 (extloadvi8 addrmode6oneL32:$addr)),
//         (EXTRACT_SUBREG (VMOVLuv8i16 (VLD1LNd32 addrmode6oneL32:$addr,
//                                      (f64 (IMPLICIT_DEF)), (i32 0))),
//                         dsub_0)>;
multiclass Lengthen_HalfSingle<string DestLanes, string DestTy, string SrcTy,
                               string InsnLanes, string InsnTy> {
  def _Any : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("extloadv" # SrcTy) addrmode6oneL32:$addr)),
       (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # InsnLanes # InsnTy)
         (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0))),
         dsub_0)>,
             Requires<[HasNEON]>;
  def _Z   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("zextloadv" # SrcTy) addrmode6oneL32:$addr)),
       (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # InsnLanes # InsnTy)
```
- EN: Declares TableGen `multiclass Lengthen_HalfSingle`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass Lengthen_HalfSingle`，它是一个可复用模板，可展开为多个相关记录。

### Lines 7884-7893
```tablegen
         (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0))),
         dsub_0)>,
             Requires<[HasNEON]>;
  def _S   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("sextloadv" # SrcTy) addrmode6oneL32:$addr)),
       (EXTRACT_SUBREG (!cast<Instruction>("VMOVLsv" # InsnLanes # InsnTy)
         (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0))),
         dsub_0)>,
             Requires<[HasNEON]>;
}
```
- EN: Defines TableGen record `_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7895-7912
```tablegen
// The following class definition is basically a copy of the
// Lengthen_HalfSingle definition above, however with an additional parameter
// "RevLanes" to select the correct VREV32dXX instruction. This is to convert
// data loaded by VLD1LN into proper vector format in big endian mode.
multiclass Lengthen_HalfSingle_Big_Endian<string DestLanes, string DestTy, string SrcTy,
                               string InsnLanes, string InsnTy, string RevLanes> {
  def _Any : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("extloadv" # SrcTy) addrmode6oneL32:$addr)),
       (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # InsnLanes # InsnTy)
         (!cast<Instruction>("VREV32d" # RevLanes)
           (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0)))),
         dsub_0)>,
             Requires<[HasNEON]>;
  def _Z   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("zextloadv" # SrcTy) addrmode6oneL32:$addr)),
       (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # InsnLanes # InsnTy)
         (!cast<Instruction>("VREV32d" # RevLanes)
           (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0)))),
```
- EN: Declares TableGen `multiclass Lengthen_HalfSingle_Big_Endian`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass Lengthen_HalfSingle_Big_Endian`，它是一个可复用模板，可展开为多个相关记录。

### Lines 7913-7922
```tablegen
         dsub_0)>,
             Requires<[HasNEON]>;
  def _S   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("sextloadv" # SrcTy) addrmode6oneL32:$addr)),
       (EXTRACT_SUBREG (!cast<Instruction>("VMOVLsv" # InsnLanes # InsnTy)
         (!cast<Instruction>("VREV32d" # RevLanes)
           (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0)))),
         dsub_0)>,
             Requires<[HasNEON]>;
}
```
- EN: Defines TableGen record `_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7924-7941
```tablegen
// extload, zextload and sextload for a lengthening load followed by another
// lengthening load, to quadruple the initial length.
//
// Lengthen_Double<"4", "i32", "i8", "8", "i16", "4", "i32"> =
//     Pat<(v4i32 (extloadvi8 addrmode6oneL32:$addr))
//         (EXTRACT_SUBREG (VMOVLuv4i32
//           (EXTRACT_SUBREG (VMOVLuv8i16 (VLD1LNd32 addrmode6oneL32:$addr,
//                                                   (f64 (IMPLICIT_DEF)),
//                                                   (i32 0))),
//                           dsub_0)),
//           dsub_0)>;
multiclass Lengthen_Double<string DestLanes, string DestTy, string SrcTy,
                           string Insn1Lanes, string Insn1Ty, string Insn2Lanes,
                           string Insn2Ty> {
  def _Any : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("extloadv" # SrcTy) addrmode6oneL32:$addr)),
         (!cast<Instruction>("VMOVLuv" # Insn2Lanes # Insn2Ty)
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn1Lanes # Insn1Ty)
```
- EN: Declares TableGen `multiclass Lengthen_Double`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass Lengthen_Double`，它是一个可复用模板，可展开为多个相关记录。

### Lines 7942-7959
```tablegen
             (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0))),
             dsub_0))>,
             Requires<[HasNEON]>;
  def _Z   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("zextloadv" # SrcTy) addrmode6oneL32:$addr)),
         (!cast<Instruction>("VMOVLuv" # Insn2Lanes # Insn2Ty)
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn1Lanes # Insn1Ty)
             (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0))),
             dsub_0))>,
             Requires<[HasNEON]>;
  def _S   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("sextloadv" # SrcTy) addrmode6oneL32:$addr)),
         (!cast<Instruction>("VMOVLsv" # Insn2Lanes # Insn2Ty)
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLsv" # Insn1Lanes # Insn1Ty)
             (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0))),
             dsub_0))>,
             Requires<[HasNEON]>;
}
```
- EN: Defines TableGen record `_Z` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_Z`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7961-7978
```tablegen
// The following class definition is basically a copy of the
// Lengthen_Double definition above, however with an additional parameter
// "RevLanes" to select the correct VREV32dXX instruction. This is to convert
// data loaded by VLD1LN into proper vector format in big endian mode.
multiclass Lengthen_Double_Big_Endian<string DestLanes, string DestTy, string SrcTy,
                           string Insn1Lanes, string Insn1Ty, string Insn2Lanes,
                           string Insn2Ty, string RevLanes> {
  def _Any : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("extloadv" # SrcTy) addrmode6oneL32:$addr)),
         (!cast<Instruction>("VMOVLuv" # Insn2Lanes # Insn2Ty)
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn1Lanes # Insn1Ty)
            (!cast<Instruction>("VREV32d" # RevLanes)
             (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0)))),
             dsub_0))>,
             Requires<[HasNEON]>;
  def _Z   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("zextloadv" # SrcTy) addrmode6oneL32:$addr)),
         (!cast<Instruction>("VMOVLuv" # Insn2Lanes # Insn2Ty)
```
- EN: Declares TableGen `multiclass Lengthen_Double_Big_Endian`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass Lengthen_Double_Big_Endian`，它是一个可复用模板，可展开为多个相关记录。

### Lines 7979-7992
```tablegen
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn1Lanes # Insn1Ty)
            (!cast<Instruction>("VREV32d" # RevLanes)
             (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0)))),
             dsub_0))>,
             Requires<[HasNEON]>;
  def _S   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("sextloadv" # SrcTy) addrmode6oneL32:$addr)),
         (!cast<Instruction>("VMOVLsv" # Insn2Lanes # Insn2Ty)
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLsv" # Insn1Lanes # Insn1Ty)
            (!cast<Instruction>("VREV32d" # RevLanes)
             (VLD1LNd32 addrmode6oneL32:$addr, (f64 (IMPLICIT_DEF)), (i32 0)))),
             dsub_0))>,
             Requires<[HasNEON]>;
}
```
- EN: Defines TableGen record `_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 7994-8011
```tablegen
// extload, zextload and sextload for a lengthening load followed by another
// lengthening load, to quadruple the initial length, but which ends up only
// requiring half the available lanes (a 64-bit outcome instead of a 128-bit).
//
// Lengthen_HalfDouble<"2", "i32", "i8", "8", "i16", "4", "i32"> =
// Pat<(v2i32 (extloadvi8 addrmode6:$addr))
//     (EXTRACT_SUBREG (VMOVLuv4i32
//       (EXTRACT_SUBREG (VMOVLuv8i16 (VLD1LNd16 addrmode6:$addr,
//                                               (f64 (IMPLICIT_DEF)), (i32 0))),
//                       dsub_0)),
//       dsub_0)>;
multiclass Lengthen_HalfDouble<string DestLanes, string DestTy, string SrcTy,
                           string Insn1Lanes, string Insn1Ty, string Insn2Lanes,
                           string Insn2Ty> {
  def _Any : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("extloadv" # SrcTy) addrmode6:$addr)),
         (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn2Lanes # Insn2Ty)
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn1Lanes # Insn1Ty)
```
- EN: Declares TableGen `multiclass Lengthen_HalfDouble`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass Lengthen_HalfDouble`，它是一个可复用模板，可展开为多个相关记录。

### Lines 8012-8029
```tablegen
             (VLD1LNd16 addrmode6:$addr, (f64 (IMPLICIT_DEF)), (i32 0))),
             dsub_0)),
          dsub_0)>,
             Requires<[HasNEON]>;
  def _Z   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("zextloadv" # SrcTy) addrmode6:$addr)),
         (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn2Lanes # Insn2Ty)
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn1Lanes # Insn1Ty)
             (VLD1LNd16 addrmode6:$addr, (f64 (IMPLICIT_DEF)), (i32 0))),
             dsub_0)),
          dsub_0)>,
              Requires<[HasNEON]>;
  def _S   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("sextloadv" # SrcTy) addrmode6:$addr)),
         (EXTRACT_SUBREG (!cast<Instruction>("VMOVLsv" # Insn2Lanes # Insn2Ty)
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLsv" # Insn1Lanes # Insn1Ty)
             (VLD1LNd16 addrmode6:$addr, (f64 (IMPLICIT_DEF)), (i32 0))),
             dsub_0)),
```
- EN: Defines TableGen record `_Z` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_Z`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8030-8032
```tablegen
          dsub_0)>,
             Requires<[HasNEON]>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8034-8051
```tablegen
// The following class definition is basically a copy of the
// Lengthen_HalfDouble definition above, however with an additional VREV16d8
// instruction to convert data loaded by VLD1LN into proper vector format
// in big endian mode.
multiclass Lengthen_HalfDouble_Big_Endian<string DestLanes, string DestTy, string SrcTy,
                           string Insn1Lanes, string Insn1Ty, string Insn2Lanes,
                           string Insn2Ty> {
  def _Any : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("extloadv" # SrcTy) addrmode6:$addr)),
         (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn2Lanes # Insn2Ty)
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn1Lanes # Insn1Ty)
             (VREV16d8 (VLD1LNd16 addrmode6:$addr, (f64 (IMPLICIT_DEF)), (i32 0)))),
             dsub_0)),
           dsub_0)>,
             Requires<[HasNEON]>;
  def _Z   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("zextloadv" # SrcTy) addrmode6:$addr)),
         (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn2Lanes # Insn2Ty)
```
- EN: Declares TableGen `multiclass Lengthen_HalfDouble_Big_Endian`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass Lengthen_HalfDouble_Big_Endian`，它是一个可复用模板，可展开为多个相关记录。

### Lines 8052-8065
```tablegen
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLuv" # Insn1Lanes # Insn1Ty)
             (VREV16d8 (VLD1LNd16 addrmode6:$addr, (f64 (IMPLICIT_DEF)), (i32 0)))),
             dsub_0)),
           dsub_0)>,
             Requires<[HasNEON]>;
  def _S   : Pat<(!cast<ValueType>("v" # DestLanes # DestTy)
                   (!cast<PatFrag>("sextloadv" # SrcTy) addrmode6:$addr)),
         (EXTRACT_SUBREG (!cast<Instruction>("VMOVLsv" # Insn2Lanes # Insn2Ty)
           (EXTRACT_SUBREG (!cast<Instruction>("VMOVLsv" # Insn1Lanes # Insn1Ty)
             (VREV16d8 (VLD1LNd16 addrmode6:$addr, (f64 (IMPLICIT_DEF)), (i32 0)))),
             dsub_0)),
           dsub_0)>,
             Requires<[HasNEON]>;
}
```
- EN: Defines TableGen record `_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8067-8069
```tablegen
defm : Lengthen_Single<"8", "i16", "8">; // v8i8 -> v8i16
defm : Lengthen_Single<"4", "i32", "16">; // v4i16 -> v4i32
defm : Lengthen_Single<"2", "i64", "32">; // v2i32 -> v2i64
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8071-8073
```tablegen
let Predicates = [HasNEON,IsLE] in {
  defm : Lengthen_HalfSingle<"4", "i16", "i8", "8", "i16">; // v4i8 -> v4i16
  defm : Lengthen_HalfSingle<"2", "i32", "i16", "4", "i32">; // v2i16 -> v2i32
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 8075-8081
```tablegen
  // Double lengthening - v4i8 -> v4i16 -> v4i32
  defm : Lengthen_Double<"4", "i32", "i8", "8", "i16", "4", "i32">;
  // v2i8 -> v2i16 -> v2i32
  defm : Lengthen_HalfDouble<"2", "i32", "i8", "8", "i16", "4", "i32">;
  // v2i16 -> v2i32 -> v2i64
  defm : Lengthen_Double<"2", "i64", "i16", "4", "i32", "2", "i64">;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8083-8085
```tablegen
let Predicates = [HasNEON,IsBE] in {
  defm : Lengthen_HalfSingle_Big_Endian<"4", "i16", "i8", "8", "i16", "8">; // v4i8 -> v4i16
  defm : Lengthen_HalfSingle_Big_Endian<"2", "i32", "i16", "4", "i32", "16">; // v2i16 -> v2i32
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 8087-8093
```tablegen
  // Double lengthening - v4i8 -> v4i16 -> v4i32
  defm : Lengthen_Double_Big_Endian<"4", "i32", "i8", "8", "i16", "4", "i32", "8">;
  // v2i8 -> v2i16 -> v2i32
  defm : Lengthen_HalfDouble_Big_Endian<"2", "i32", "i8", "8", "i16", "4", "i32">;
  // v2i16 -> v2i32 -> v2i64
  defm : Lengthen_Double_Big_Endian<"2", "i64", "i16", "4", "i32", "2", "i64", "16">;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8095-8112
```tablegen
// Triple lengthening - v2i8 -> v2i16 -> v2i32 -> v2i64
let Predicates = [HasNEON,IsLE] in {
  def : Pat<(v2i64 (extloadvi8 addrmode6:$addr)),
        (VMOVLuv2i64 (EXTRACT_SUBREG (VMOVLuv4i32 (EXTRACT_SUBREG (VMOVLuv8i16
           (VLD1LNd16 addrmode6:$addr,
                      (f64 (IMPLICIT_DEF)), (i32 0))), dsub_0)), dsub_0))>;
  def : Pat<(v2i64 (zextloadvi8 addrmode6:$addr)),
        (VMOVLuv2i64 (EXTRACT_SUBREG (VMOVLuv4i32 (EXTRACT_SUBREG (VMOVLuv8i16
           (VLD1LNd16 addrmode6:$addr,
                      (f64 (IMPLICIT_DEF)), (i32 0))), dsub_0)), dsub_0))>;
  def : Pat<(v2i64 (sextloadvi8 addrmode6:$addr)),
        (VMOVLsv2i64 (EXTRACT_SUBREG (VMOVLsv4i32 (EXTRACT_SUBREG (VMOVLsv8i16
           (VLD1LNd16 addrmode6:$addr,
                      (f64 (IMPLICIT_DEF)), (i32 0))), dsub_0)), dsub_0))>;
}
// The following patterns are basically a copy of the patterns above,
// however with an additional VREV16d instruction to convert data
// loaded by VLD1LN into proper vector format in big endian mode.
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 8113-8129
```tablegen
let Predicates = [HasNEON,IsBE] in {
  def : Pat<(v2i64 (extloadvi8 addrmode6:$addr)),
        (VMOVLuv2i64 (EXTRACT_SUBREG (VMOVLuv4i32 (EXTRACT_SUBREG (VMOVLuv8i16
           (VREV16d8
             (VLD1LNd16 addrmode6:$addr,
                        (f64 (IMPLICIT_DEF)), (i32 0)))), dsub_0)), dsub_0))>;
  def : Pat<(v2i64 (zextloadvi8 addrmode6:$addr)),
        (VMOVLuv2i64 (EXTRACT_SUBREG (VMOVLuv4i32 (EXTRACT_SUBREG (VMOVLuv8i16
           (VREV16d8
             (VLD1LNd16 addrmode6:$addr,
                        (f64 (IMPLICIT_DEF)), (i32 0)))), dsub_0)), dsub_0))>;
  def : Pat<(v2i64 (sextloadvi8 addrmode6:$addr)),
        (VMOVLsv2i64 (EXTRACT_SUBREG (VMOVLsv4i32 (EXTRACT_SUBREG (VMOVLsv8i16
           (VREV16d8
             (VLD1LNd16 addrmode6:$addr,
                        (f64 (IMPLICIT_DEF)), (i32 0)))), dsub_0)), dsub_0))>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 8131-8146
```tablegen
let Predicates = [HasNEON] in {
def : Pat<(v2i64 (concat_vectors DPR:$Dn, DPR:$Dm)),
          (REG_SEQUENCE QPR, DPR:$Dn, dsub_0, DPR:$Dm, dsub_1)>;
def : Pat<(v4i32 (concat_vectors DPR:$Dn, DPR:$Dm)),
          (REG_SEQUENCE QPR, DPR:$Dn, dsub_0, DPR:$Dm, dsub_1)>;
def : Pat<(v8i16 (concat_vectors DPR:$Dn, DPR:$Dm)),
          (REG_SEQUENCE QPR, DPR:$Dn, dsub_0, DPR:$Dm, dsub_1)>;
def : Pat<(v16i8 (concat_vectors DPR:$Dn, DPR:$Dm)),
          (REG_SEQUENCE QPR, DPR:$Dn, dsub_0, DPR:$Dm, dsub_1)>;
def : Pat<(v4f32 (concat_vectors DPR:$Dn, DPR:$Dm)),
          (REG_SEQUENCE QPR, DPR:$Dn, dsub_0, DPR:$Dm, dsub_1)>;
def : Pat<(v8f16 (concat_vectors DPR:$Dn, DPR:$Dm)),
          (REG_SEQUENCE QPR, DPR:$Dn, dsub_0, DPR:$Dm, dsub_1)>;
def : Pat<(v8bf16 (concat_vectors DPR:$Dn, DPR:$Dm)),
          (REG_SEQUENCE QPR, DPR:$Dn, dsub_0, DPR:$Dm, dsub_1)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 8148-8150
```tablegen
//===----------------------------------------------------------------------===//
// Assembler aliases
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 8152-8155
```tablegen
def : VFP2InstAlias<"fmdhr${p} $Dd, $Rn",
                    (VSETLNi32 DPR:$Dd, GPR:$Rn, 1, pred:$p)>;
def : VFP2InstAlias<"fmdlr${p} $Dd, $Rn",
                    (VSETLNi32 DPR:$Dd, GPR:$Rn, 0, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8157-8174
```tablegen
// VAND/VBIC/VEOR/VORR accept but do not require a type suffix.
defm : NEONDTAnyInstAlias<"vand${p}", "$Vd, $Vn, $Vm",
                         (VANDd DPR:$Vd, DPR:$Vn, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vand${p}", "$Vd, $Vn, $Vm",
                         (VANDq QPR:$Vd, QPR:$Vn, QPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vbic${p}", "$Vd, $Vn, $Vm",
                         (VBICd DPR:$Vd, DPR:$Vn, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vbic${p}", "$Vd, $Vn, $Vm",
                         (VBICq QPR:$Vd, QPR:$Vn, QPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"veor${p}", "$Vd, $Vn, $Vm",
                         (VEORd DPR:$Vd, DPR:$Vn, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"veor${p}", "$Vd, $Vn, $Vm",
                         (VEORq QPR:$Vd, QPR:$Vn, QPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vorr${p}", "$Vd, $Vn, $Vm",
                         (VORRd DPR:$Vd, DPR:$Vn, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vorr${p}", "$Vd, $Vn, $Vm",
                         (VORRq QPR:$Vd, QPR:$Vn, QPR:$Vm, pred:$p)>;
// ... two-operand aliases
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8175-8192
```tablegen
defm : NEONDTAnyInstAlias<"vand${p}", "$Vdn, $Vm",
                         (VANDd DPR:$Vdn, DPR:$Vdn, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vand${p}", "$Vdn, $Vm",
                         (VANDq QPR:$Vdn, QPR:$Vdn, QPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"veor${p}", "$Vdn, $Vm",
                         (VEORd DPR:$Vdn, DPR:$Vdn, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"veor${p}", "$Vdn, $Vm",
                         (VEORq QPR:$Vdn, QPR:$Vdn, QPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vorr${p}", "$Vdn, $Vm",
                         (VORRd DPR:$Vdn, DPR:$Vdn, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vorr${p}", "$Vdn, $Vm",
                         (VORRq QPR:$Vdn, QPR:$Vdn, QPR:$Vm, pred:$p)>;
// ... immediates
def : NEONInstAlias<"vand${p}.i16 $Vd, $imm",
                    (VBICiv4i16 DPR:$Vd, nImmSplatNotI16:$imm, pred:$p)>;
def : NEONInstAlias<"vand${p}.i32 $Vd, $imm",
                    (VBICiv2i32 DPR:$Vd, nImmSplatNotI32:$imm, pred:$p)>;
def : NEONInstAlias<"vand${p}.i16 $Vd, $imm",
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8193-8195
```tablegen
                    (VBICiv8i16 QPR:$Vd, nImmSplatNotI16:$imm, pred:$p)>;
def : NEONInstAlias<"vand${p}.i32 $Vd, $imm",
                    (VBICiv4i32 QPR:$Vd, nImmSplatNotI32:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8198-8208
```tablegen
// VLD1 single-lane pseudo-instructions. These need special handling for
// the lane index that an InstAlias can't handle, so we use these instead.
def VLD1LNdAsm_8 : NEONDataTypeAsmPseudoInst<"vld1${p}", ".8", "$list, $addr",
                 (ins VecListOneDByteIndexed:$list, addrmode6alignNone:$addr,
                      pred:$p)>;
def VLD1LNdAsm_16 : NEONDataTypeAsmPseudoInst<"vld1${p}", ".16", "$list, $addr",
                 (ins VecListOneDHWordIndexed:$list, addrmode6align16:$addr,
                      pred:$p)>;
def VLD1LNdAsm_32 : NEONDataTypeAsmPseudoInst<"vld1${p}", ".32", "$list, $addr",
                 (ins VecListOneDWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
```
- EN: Defines TableGen record `VLD1LNdAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1LNdAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8210-8227
```tablegen
def VLD1LNdWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld1${p}", ".8", "$list, $addr!",
                 (ins VecListOneDByteIndexed:$list, addrmode6alignNone:$addr,
                      pred:$p)>;
def VLD1LNdWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld1${p}", ".16", "$list, $addr!",
                 (ins VecListOneDHWordIndexed:$list, addrmode6align16:$addr,
                      pred:$p)>;
def VLD1LNdWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld1${p}", ".32", "$list, $addr!",
                 (ins VecListOneDWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
def VLD1LNdWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld1${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListOneDByteIndexed:$list, addrmode6alignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD1LNdWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld1${p}", ".16", "$list, $addr, $Rm",
```
- EN: Defines TableGen record `VLD1LNdWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1LNdWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8228-8233
```tablegen
                  (ins VecListOneDHWordIndexed:$list, addrmode6align16:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD1LNdWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld1${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListOneDWordIndexed:$list, addrmode6align32:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD1LNdWB_register_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD1LNdWB_register_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8235-8235
```tablegen
let mayStore = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 8237-8247
```tablegen
// VST1 single-lane pseudo-instructions. These need special handling for
// the lane index that an InstAlias can't handle, so we use these instead.
def VST1LNdAsm_8 : NEONDataTypeAsmPseudoInst<"vst1${p}", ".8", "$list, $addr",
                 (ins VecListOneDByteIndexed:$list, addrmode6alignNone:$addr,
                      pred:$p)>;
def VST1LNdAsm_16 : NEONDataTypeAsmPseudoInst<"vst1${p}", ".16", "$list, $addr",
                 (ins VecListOneDHWordIndexed:$list, addrmode6align16:$addr,
                      pred:$p)>;
def VST1LNdAsm_32 : NEONDataTypeAsmPseudoInst<"vst1${p}", ".32", "$list, $addr",
                 (ins VecListOneDWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
```
- EN: Defines TableGen record `VST1LNdAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1LNdAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8249-8266
```tablegen
def VST1LNdWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst1${p}", ".8", "$list, $addr!",
                 (ins VecListOneDByteIndexed:$list, addrmode6alignNone:$addr,
                      pred:$p)>;
def VST1LNdWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst1${p}", ".16", "$list, $addr!",
                 (ins VecListOneDHWordIndexed:$list, addrmode6align16:$addr,
                      pred:$p)>;
def VST1LNdWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst1${p}", ".32", "$list, $addr!",
                 (ins VecListOneDWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
def VST1LNdWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst1${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListOneDByteIndexed:$list, addrmode6alignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST1LNdWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst1${p}", ".16", "$list, $addr, $Rm",
```
- EN: Defines TableGen record `VST1LNdWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1LNdWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8267-8272
```tablegen
                  (ins VecListOneDHWordIndexed:$list, addrmode6align16:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST1LNdWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst1${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListOneDWordIndexed:$list, addrmode6align32:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VST1LNdWB_register_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST1LNdWB_register_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8274-8274
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8276-8291
```tablegen
// VLD2 single-lane pseudo-instructions. These need special handling for
// the lane index that an InstAlias can't handle, so we use these instead.
def VLD2LNdAsm_8 : NEONDataTypeAsmPseudoInst<"vld2${p}", ".8", "$list, $addr",
                 (ins VecListTwoDByteIndexed:$list, addrmode6align16:$addr,
                  pred:$p)>;
def VLD2LNdAsm_16 : NEONDataTypeAsmPseudoInst<"vld2${p}", ".16", "$list, $addr",
                 (ins VecListTwoDHWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
def VLD2LNdAsm_32 : NEONDataTypeAsmPseudoInst<"vld2${p}", ".32", "$list, $addr",
                 (ins VecListTwoDWordIndexed:$list, addrmode6align64:$addr, pred:$p)>;
def VLD2LNqAsm_16 : NEONDataTypeAsmPseudoInst<"vld2${p}", ".16", "$list, $addr",
                 (ins VecListTwoQHWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
def VLD2LNqAsm_32 : NEONDataTypeAsmPseudoInst<"vld2${p}", ".32", "$list, $addr",
                 (ins VecListTwoQWordIndexed:$list, addrmode6align64:$addr,
                      pred:$p)>;
```
- EN: Defines TableGen record `VLD2LNdAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNdAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8293-8310
```tablegen
def VLD2LNdWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld2${p}", ".8", "$list, $addr!",
                 (ins VecListTwoDByteIndexed:$list, addrmode6align16:$addr,
                      pred:$p)>;
def VLD2LNdWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld2${p}", ".16", "$list, $addr!",
                 (ins VecListTwoDHWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
def VLD2LNdWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld2${p}", ".32", "$list, $addr!",
                 (ins VecListTwoDWordIndexed:$list, addrmode6align64:$addr,
                      pred:$p)>;
def VLD2LNqWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld2${p}", ".16", "$list, $addr!",
                 (ins VecListTwoQHWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
def VLD2LNqWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld2${p}", ".32", "$list, $addr!",
```
- EN: Defines TableGen record `VLD2LNdWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNdWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8311-8328
```tablegen
                 (ins VecListTwoQWordIndexed:$list, addrmode6align64:$addr,
                      pred:$p)>;
def VLD2LNdWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld2${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListTwoDByteIndexed:$list, addrmode6align16:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD2LNdWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld2${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListTwoDHWordIndexed:$list, addrmode6align32:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD2LNdWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld2${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListTwoDWordIndexed:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD2LNqWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld2${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListTwoQHWordIndexed:$list, addrmode6align32:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD2LNdWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNdWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8329-8332
```tablegen
def VLD2LNqWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld2${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListTwoQWordIndexed:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD2LNqWB_register_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD2LNqWB_register_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8334-8334
```tablegen
let mayStore = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 8336-8352
```tablegen
// VST2 single-lane pseudo-instructions. These need special handling for
// the lane index that an InstAlias can't handle, so we use these instead.
def VST2LNdAsm_8 : NEONDataTypeAsmPseudoInst<"vst2${p}", ".8", "$list, $addr",
                 (ins VecListTwoDByteIndexed:$list, addrmode6align16:$addr,
                      pred:$p)>;
def VST2LNdAsm_16 : NEONDataTypeAsmPseudoInst<"vst2${p}", ".16", "$list, $addr",
                 (ins VecListTwoDHWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
def VST2LNdAsm_32 : NEONDataTypeAsmPseudoInst<"vst2${p}", ".32", "$list, $addr",
                 (ins VecListTwoDWordIndexed:$list, addrmode6align64:$addr,
                      pred:$p)>;
def VST2LNqAsm_16 : NEONDataTypeAsmPseudoInst<"vst2${p}", ".16", "$list, $addr",
                 (ins VecListTwoQHWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
def VST2LNqAsm_32 : NEONDataTypeAsmPseudoInst<"vst2${p}", ".32", "$list, $addr",
                 (ins VecListTwoQWordIndexed:$list, addrmode6align64:$addr,
                      pred:$p)>;
```
- EN: Defines TableGen record `VST2LNdAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNdAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8354-8371
```tablegen
def VST2LNdWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst2${p}", ".8", "$list, $addr!",
                 (ins VecListTwoDByteIndexed:$list, addrmode6align16:$addr,
                      pred:$p)>;
def VST2LNdWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst2${p}", ".16", "$list, $addr!",
                 (ins VecListTwoDHWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
def VST2LNdWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst2${p}", ".32", "$list, $addr!",
                 (ins VecListTwoDWordIndexed:$list, addrmode6align64:$addr,
                      pred:$p)>;
def VST2LNqWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst2${p}", ".16", "$list, $addr!",
                 (ins VecListTwoQHWordIndexed:$list, addrmode6align32:$addr,
                      pred:$p)>;
def VST2LNqWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst2${p}", ".32", "$list, $addr!",
```
- EN: Defines TableGen record `VST2LNdWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNdWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8372-8389
```tablegen
                 (ins VecListTwoQWordIndexed:$list, addrmode6align64:$addr,
                      pred:$p)>;
def VST2LNdWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst2${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListTwoDByteIndexed:$list, addrmode6align16:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST2LNdWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst2${p}", ".16","$list, $addr, $Rm",
                  (ins VecListTwoDHWordIndexed:$list, addrmode6align32:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST2LNdWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst2${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListTwoDWordIndexed:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST2LNqWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst2${p}", ".16","$list, $addr, $Rm",
                  (ins VecListTwoQHWordIndexed:$list, addrmode6align32:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VST2LNdWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNdWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8390-8393
```tablegen
def VST2LNqWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst2${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListTwoQWordIndexed:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VST2LNqWB_register_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST2LNqWB_register_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8395-8395
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8397-8414
```tablegen
// VLD3 all-lanes pseudo-instructions. These need special handling for
// the lane index that an InstAlias can't handle, so we use these instead.
def VLD3DUPdAsm_8 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr",
               (ins VecListThreeDAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
def VLD3DUPdAsm_16: NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr",
               (ins VecListThreeDAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
def VLD3DUPdAsm_32: NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr",
               (ins VecListThreeDAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
def VLD3DUPqAsm_8 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr",
               (ins VecListThreeQAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
def VLD3DUPqAsm_16: NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr",
               (ins VecListThreeQAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
def VLD3DUPqAsm_32: NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr",
```
- EN: Defines TableGen record `VLD3DUPdAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPdAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8415-8416
```tablegen
               (ins VecListThreeQAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8418-8435
```tablegen
def VLD3DUPdWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr!",
               (ins VecListThreeDAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
def VLD3DUPdWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr!",
               (ins VecListThreeDAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
def VLD3DUPdWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr!",
               (ins VecListThreeDAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
def VLD3DUPqWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr!",
               (ins VecListThreeQAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
def VLD3DUPqWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr!",
```
- EN: Defines TableGen record `VLD3DUPdWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPdWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8436-8453
```tablegen
               (ins VecListThreeQAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
def VLD3DUPqWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr!",
               (ins VecListThreeQAllLanes:$list, addrmode6dupalignNone:$addr,
                    pred:$p)>;
def VLD3DUPdWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListThreeDAllLanes:$list, addrmode6dupalignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD3DUPdWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListThreeDAllLanes:$list, addrmode6dupalignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD3DUPdWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListThreeDAllLanes:$list, addrmode6dupalignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD3DUPqWB_fixed_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPqWB_fixed_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8454-8465
```tablegen
def VLD3DUPqWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListThreeQAllLanes:$list, addrmode6dupalignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD3DUPqWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListThreeQAllLanes:$list, addrmode6dupalignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD3DUPqWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListThreeQAllLanes:$list, addrmode6dupalignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD3DUPqWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3DUPqWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8468-8484
```tablegen
// VLD3 single-lane pseudo-instructions. These need special handling for
// the lane index that an InstAlias can't handle, so we use these instead.
def VLD3LNdAsm_8 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr",
               (ins VecListThreeDByteIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VLD3LNdAsm_16 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr",
               (ins VecListThreeDHWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VLD3LNdAsm_32 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr",
               (ins VecListThreeDWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VLD3LNqAsm_16 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr",
               (ins VecListThreeQHWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VLD3LNqAsm_32 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr",
               (ins VecListThreeQWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
```
- EN: Defines TableGen record `VLD3LNdAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNdAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8486-8503
```tablegen
def VLD3LNdWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr!",
               (ins VecListThreeDByteIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VLD3LNdWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr!",
               (ins VecListThreeDHWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VLD3LNdWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr!",
               (ins VecListThreeDWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VLD3LNqWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr!",
               (ins VecListThreeQHWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VLD3LNqWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr!",
```
- EN: Defines TableGen record `VLD3LNdWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNdWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8504-8521
```tablegen
               (ins VecListThreeQWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VLD3LNdWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListThreeDByteIndexed:$list, addrmode6alignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD3LNdWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListThreeDHWordIndexed:$list,
                       addrmode6alignNone:$addr, rGPR:$Rm, pred:$p)>;
def VLD3LNdWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListThreeDWordIndexed:$list, addrmode6alignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD3LNqWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListThreeQHWordIndexed:$list,
                       addrmode6alignNone:$addr, rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD3LNdWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNdWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8522-8525
```tablegen
def VLD3LNqWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListThreeQWordIndexed:$list, addrmode6alignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD3LNqWB_register_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3LNqWB_register_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8527-8541
```tablegen
// VLD3 multiple structure pseudo-instructions. These need special handling for
// the vector operands that the normal instructions don't yet model.
// FIXME: Remove these when the register classes and instructions are updated.
def VLD3dAsm_8 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VLD3dAsm_16 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VLD3dAsm_32 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VLD3qAsm_8 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
def VLD3qAsm_16 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
def VLD3qAsm_32 : NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
```
- EN: Defines TableGen record `VLD3dAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3dAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8543-8560
```tablegen
def VLD3dWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr!",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VLD3dWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr!",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VLD3dWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr!",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VLD3qWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr!",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
def VLD3qWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr!",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
def VLD3qWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr!",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
```
- EN: Defines TableGen record `VLD3dWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3dWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8561-8578
```tablegen
def VLD3dWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListThreeD:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD3dWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListThreeD:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD3dWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListThreeD:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD3qWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListThreeQ:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD3qWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".16", "$list, $addr, $Rm",
```
- EN: Defines TableGen record `VLD3dWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3dWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8579-8584
```tablegen
                  (ins VecListThreeQ:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD3qWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld3${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListThreeQ:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD3qWB_register_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD3qWB_register_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8586-8586
```tablegen
let mayStore = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 8588-8604
```tablegen
// VST3 single-lane pseudo-instructions. These need special handling for
// the lane index that an InstAlias can't handle, so we use these instead.
def VST3LNdAsm_8 : NEONDataTypeAsmPseudoInst<"vst3${p}", ".8", "$list, $addr",
               (ins VecListThreeDByteIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VST3LNdAsm_16 : NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr",
               (ins VecListThreeDHWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VST3LNdAsm_32 : NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr",
               (ins VecListThreeDWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VST3LNqAsm_16 : NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr",
               (ins VecListThreeQHWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VST3LNqAsm_32 : NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr",
               (ins VecListThreeQWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
```
- EN: Defines TableGen record `VST3LNdAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNdAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8606-8623
```tablegen
def VST3LNdWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".8", "$list, $addr!",
               (ins VecListThreeDByteIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VST3LNdWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr!",
               (ins VecListThreeDHWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VST3LNdWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr!",
               (ins VecListThreeDWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VST3LNqWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr!",
               (ins VecListThreeQHWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VST3LNqWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr!",
```
- EN: Defines TableGen record `VST3LNdWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNdWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8624-8641
```tablegen
               (ins VecListThreeQWordIndexed:$list, addrmode6alignNone:$addr,
                    pred:$p)>;
def VST3LNdWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListThreeDByteIndexed:$list, addrmode6alignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST3LNdWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListThreeDHWordIndexed:$list,
                       addrmode6alignNone:$addr, rGPR:$Rm, pred:$p)>;
def VST3LNdWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListThreeDWordIndexed:$list, addrmode6alignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST3LNqWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListThreeQHWordIndexed:$list,
                       addrmode6alignNone:$addr, rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VST3LNdWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNdWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8642-8645
```tablegen
def VST3LNqWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListThreeQWordIndexed:$list, addrmode6alignNone:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VST3LNqWB_register_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3LNqWB_register_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8648-8662
```tablegen
// VST3 multiple structure pseudo-instructions. These need special handling for
// the vector operands that the normal instructions don't yet model.
// FIXME: Remove these when the register classes and instructions are updated.
def VST3dAsm_8 : NEONDataTypeAsmPseudoInst<"vst3${p}", ".8", "$list, $addr",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VST3dAsm_16 : NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VST3dAsm_32 : NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VST3qAsm_8 : NEONDataTypeAsmPseudoInst<"vst3${p}", ".8", "$list, $addr",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
def VST3qAsm_16 : NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
def VST3qAsm_32 : NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
```
- EN: Defines TableGen record `VST3dAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3dAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8664-8681
```tablegen
def VST3dWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".8", "$list, $addr!",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VST3dWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr!",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VST3dWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr!",
               (ins VecListThreeD:$list, addrmode6align64:$addr, pred:$p)>;
def VST3qWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".8", "$list, $addr!",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
def VST3qWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr!",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
def VST3qWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr!",
               (ins VecListThreeQ:$list, addrmode6align64:$addr, pred:$p)>;
```
- EN: Defines TableGen record `VST3dWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3dWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8682-8699
```tablegen
def VST3dWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListThreeD:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST3dWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListThreeD:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST3dWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListThreeD:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST3qWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListThreeQ:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST3qWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".16", "$list, $addr, $Rm",
```
- EN: Defines TableGen record `VST3dWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3dWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8700-8705
```tablegen
                  (ins VecListThreeQ:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST3qWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst3${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListThreeQ:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VST3qWB_register_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST3qWB_register_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8707-8707
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8709-8726
```tablegen
// VLD4 all-lanes pseudo-instructions. These need special handling for
// the lane index that an InstAlias can't handle, so we use these instead.
def VLD4DUPdAsm_8 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr",
               (ins VecListFourDAllLanes:$list, addrmode6dupalign32:$addr,
                    pred:$p)>;
def VLD4DUPdAsm_16: NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr",
               (ins VecListFourDAllLanes:$list, addrmode6dupalign64:$addr,
                    pred:$p)>;
def VLD4DUPdAsm_32: NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr",
               (ins VecListFourDAllLanes:$list, addrmode6dupalign64or128:$addr,
                    pred:$p)>;
def VLD4DUPqAsm_8 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr",
               (ins VecListFourQAllLanes:$list, addrmode6dupalign32:$addr,
                    pred:$p)>;
def VLD4DUPqAsm_16: NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr",
               (ins VecListFourQAllLanes:$list, addrmode6dupalign64:$addr,
                    pred:$p)>;
def VLD4DUPqAsm_32: NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr",
```
- EN: Defines TableGen record `VLD4DUPdAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPdAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8727-8728
```tablegen
               (ins VecListFourQAllLanes:$list, addrmode6dupalign64or128:$addr,
                    pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 8730-8747
```tablegen
def VLD4DUPdWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr!",
               (ins VecListFourDAllLanes:$list, addrmode6dupalign32:$addr,
                    pred:$p)>;
def VLD4DUPdWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr!",
               (ins VecListFourDAllLanes:$list, addrmode6dupalign64:$addr,
                    pred:$p)>;
def VLD4DUPdWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr!",
               (ins VecListFourDAllLanes:$list, addrmode6dupalign64or128:$addr,
                    pred:$p)>;
def VLD4DUPqWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr!",
               (ins VecListFourQAllLanes:$list, addrmode6dupalign32:$addr,
                    pred:$p)>;
def VLD4DUPqWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr!",
```
- EN: Defines TableGen record `VLD4DUPdWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPdWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8748-8765
```tablegen
               (ins VecListFourQAllLanes:$list, addrmode6dupalign64:$addr,
                    pred:$p)>;
def VLD4DUPqWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr!",
               (ins VecListFourQAllLanes:$list, addrmode6dupalign64or128:$addr,
                    pred:$p)>;
def VLD4DUPdWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListFourDAllLanes:$list, addrmode6dupalign32:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD4DUPdWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListFourDAllLanes:$list, addrmode6dupalign64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD4DUPdWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListFourDAllLanes:$list,
                       addrmode6dupalign64or128:$addr, rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD4DUPqWB_fixed_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPqWB_fixed_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8766-8777
```tablegen
def VLD4DUPqWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListFourQAllLanes:$list, addrmode6dupalign32:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD4DUPqWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListFourQAllLanes:$list, addrmode6dupalign64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD4DUPqWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListFourQAllLanes:$list,
                       addrmode6dupalign64or128:$addr, rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD4DUPqWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4DUPqWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8780-8796
```tablegen
// VLD4 single-lane pseudo-instructions. These need special handling for
// the lane index that an InstAlias can't handle, so we use these instead.
def VLD4LNdAsm_8 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr",
               (ins VecListFourDByteIndexed:$list, addrmode6align32:$addr,
                    pred:$p)>;
def VLD4LNdAsm_16 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr",
               (ins VecListFourDHWordIndexed:$list, addrmode6align64:$addr,
                    pred:$p)>;
def VLD4LNdAsm_32 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr",
               (ins VecListFourDWordIndexed:$list, addrmode6align64or128:$addr,
                    pred:$p)>;
def VLD4LNqAsm_16 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr",
               (ins VecListFourQHWordIndexed:$list, addrmode6align64:$addr,
                    pred:$p)>;
def VLD4LNqAsm_32 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr",
               (ins VecListFourQWordIndexed:$list, addrmode6align64or128:$addr,
                    pred:$p)>;
```
- EN: Defines TableGen record `VLD4LNdAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNdAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8798-8815
```tablegen
def VLD4LNdWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr!",
               (ins VecListFourDByteIndexed:$list, addrmode6align32:$addr,
                    pred:$p)>;
def VLD4LNdWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr!",
               (ins VecListFourDHWordIndexed:$list, addrmode6align64:$addr,
                    pred:$p)>;
def VLD4LNdWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr!",
               (ins VecListFourDWordIndexed:$list, addrmode6align64or128:$addr,
                    pred:$p)>;
def VLD4LNqWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr!",
               (ins VecListFourQHWordIndexed:$list, addrmode6align64:$addr,
                    pred:$p)>;
def VLD4LNqWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr!",
```
- EN: Defines TableGen record `VLD4LNdWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNdWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8816-8833
```tablegen
               (ins VecListFourQWordIndexed:$list, addrmode6align64or128:$addr,
                    pred:$p)>;
def VLD4LNdWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListFourDByteIndexed:$list, addrmode6align32:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD4LNdWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListFourDHWordIndexed:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD4LNdWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListFourDWordIndexed:$list,
                       addrmode6align64or128:$addr, rGPR:$Rm, pred:$p)>;
def VLD4LNqWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListFourQHWordIndexed:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD4LNdWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNdWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8834-8837
```tablegen
def VLD4LNqWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListFourQWordIndexed:$list,
                       addrmode6align64or128:$addr, rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD4LNqWB_register_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4LNqWB_register_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8841-8858
```tablegen
// VLD4 multiple structure pseudo-instructions. These need special handling for
// the vector operands that the normal instructions don't yet model.
// FIXME: Remove these when the register classes and instructions are updated.
def VLD4dAsm_8 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
def VLD4dAsm_16 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
def VLD4dAsm_32 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
def VLD4qAsm_8 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr",
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
def VLD4qAsm_16 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr",
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
```
- EN: Defines TableGen record `VLD4dAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4dAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8859-8861
```tablegen
def VLD4qAsm_32 : NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr",
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
```
- EN: Defines TableGen record `VLD4qAsm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4qAsm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8863-8880
```tablegen
def VLD4dWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr!",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
def VLD4dWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr!",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
def VLD4dWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr!",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
def VLD4qWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr!",
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
def VLD4qWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr!",
```
- EN: Defines TableGen record `VLD4dWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4dWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8881-8898
```tablegen
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
def VLD4qWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr!",
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                pred:$p)>;
def VLD4dWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD4dWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD4dWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD4qWB_fixed_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4qWB_fixed_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8899-8910
```tablegen
def VLD4qWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD4qWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
def VLD4qWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vld4${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VLD4qWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VLD4qWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8912-8912
```tablegen
let mayStore = 1 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 8914-8930
```tablegen
// VST4 single-lane pseudo-instructions. These need special handling for
// the lane index that an InstAlias can't handle, so we use these instead.
def VST4LNdAsm_8 : NEONDataTypeAsmPseudoInst<"vst4${p}", ".8", "$list, $addr",
               (ins VecListFourDByteIndexed:$list, addrmode6align32:$addr,
                    pred:$p)>;
def VST4LNdAsm_16 : NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr",
               (ins VecListFourDHWordIndexed:$list, addrmode6align64:$addr,
                    pred:$p)>;
def VST4LNdAsm_32 : NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr",
               (ins VecListFourDWordIndexed:$list, addrmode6align64or128:$addr,
                    pred:$p)>;
def VST4LNqAsm_16 : NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr",
               (ins VecListFourQHWordIndexed:$list, addrmode6align64:$addr,
                    pred:$p)>;
def VST4LNqAsm_32 : NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr",
               (ins VecListFourQWordIndexed:$list, addrmode6align64or128:$addr,
                    pred:$p)>;
```
- EN: Defines TableGen record `VST4LNdAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNdAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8932-8949
```tablegen
def VST4LNdWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".8", "$list, $addr!",
               (ins VecListFourDByteIndexed:$list, addrmode6align32:$addr,
                    pred:$p)>;
def VST4LNdWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr!",
               (ins VecListFourDHWordIndexed:$list, addrmode6align64:$addr,
                    pred:$p)>;
def VST4LNdWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr!",
               (ins VecListFourDWordIndexed:$list, addrmode6align64or128:$addr,
                    pred:$p)>;
def VST4LNqWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr!",
               (ins VecListFourQHWordIndexed:$list, addrmode6align64:$addr,
                    pred:$p)>;
def VST4LNqWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr!",
```
- EN: Defines TableGen record `VST4LNdWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNdWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8950-8967
```tablegen
               (ins VecListFourQWordIndexed:$list, addrmode6align64or128:$addr,
                    pred:$p)>;
def VST4LNdWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListFourDByteIndexed:$list, addrmode6align32:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST4LNdWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListFourDHWordIndexed:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST4LNdWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListFourDWordIndexed:$list,
                       addrmode6align64or128:$addr, rGPR:$Rm, pred:$p)>;
def VST4LNqWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListFourQHWordIndexed:$list, addrmode6align64:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VST4LNdWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNdWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8968-8971
```tablegen
def VST4LNqWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListFourQWordIndexed:$list,
                       addrmode6align64or128:$addr, rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VST4LNqWB_register_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4LNqWB_register_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8974-8991
```tablegen
// VST4 multiple structure pseudo-instructions. These need special handling for
// the vector operands that the normal instructions don't yet model.
// FIXME: Remove these when the register classes and instructions are updated.
def VST4dAsm_8 : NEONDataTypeAsmPseudoInst<"vst4${p}", ".8", "$list, $addr",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
def VST4dAsm_16 : NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
def VST4dAsm_32 : NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
def VST4qAsm_8 : NEONDataTypeAsmPseudoInst<"vst4${p}", ".8", "$list, $addr",
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
def VST4qAsm_16 : NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr",
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
```
- EN: Defines TableGen record `VST4dAsm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4dAsm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8992-8994
```tablegen
def VST4qAsm_32 : NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr",
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
```
- EN: Defines TableGen record `VST4qAsm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4qAsm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 8996-9013
```tablegen
def VST4dWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".8", "$list, $addr!",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
def VST4dWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr!",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
def VST4dWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr!",
               (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
def VST4qWB_fixed_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".8", "$list, $addr!",
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
def VST4qWB_fixed_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr!",
```
- EN: Defines TableGen record `VST4dWB_fixed_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4dWB_fixed_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 9014-9031
```tablegen
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
def VST4qWB_fixed_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr!",
               (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                    pred:$p)>;
def VST4dWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST4dWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST4dWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListFourD:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
```
- EN: Defines TableGen record `VST4qWB_fixed_Asm_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4qWB_fixed_Asm_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 9032-9044
```tablegen
def VST4qWB_register_Asm_8 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".8", "$list, $addr, $Rm",
                  (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST4qWB_register_Asm_16 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".16", "$list, $addr, $Rm",
                  (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
def VST4qWB_register_Asm_32 :
        NEONDataTypeAsmPseudoInst<"vst4${p}", ".32", "$list, $addr, $Rm",
                  (ins VecListFourQ:$list, addrmode6align64or128or256:$addr,
                       rGPR:$Rm, pred:$p)>;
}
```
- EN: Defines TableGen record `VST4qWB_register_Asm_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VST4qWB_register_Asm_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 9046-9050
```tablegen
// VMOV/VMVN takes an optional datatype suffix
defm : NEONDTAnyInstAlias<"vmov${p}", "$Vd, $Vm",
                         (VORRd DPR:$Vd, DPR:$Vm, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vmov${p}", "$Vd, $Vm",
                         (VORRq QPR:$Vd, QPR:$Vm, QPR:$Vm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9052-9055
```tablegen
defm : NEONDTAnyInstAlias<"vmvn${p}", "$Vd, $Vm",
                         (VMVNd DPR:$Vd, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vmvn${p}", "$Vd, $Vm",
                         (VMVNq QPR:$Vd, QPR:$Vm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9057-9074
```tablegen
// VCLT (register) is an assembler alias for VCGT w/ the operands reversed.
// D-register versions.
def : NEONInstAlias<"vcle${p}.s8 $Dd, $Dn, $Dm",
                    (VCGEsv8i8 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.s16 $Dd, $Dn, $Dm",
                    (VCGEsv4i16 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.s32 $Dd, $Dn, $Dm",
                    (VCGEsv2i32 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.u8 $Dd, $Dn, $Dm",
                    (VCGEuv8i8 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.u16 $Dd, $Dn, $Dm",
                    (VCGEuv4i16 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.u32 $Dd, $Dn, $Dm",
                    (VCGEuv2i32 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.f32 $Dd, $Dn, $Dm",
                    (VCGEfd DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
let Predicates = [HasNEON, HasFullFP16] in
def : NEONInstAlias<"vcle${p}.f16 $Dd, $Dn, $Dm",
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 9075-9092
```tablegen
                    (VCGEhd DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
// Q-register versions.
def : NEONInstAlias<"vcle${p}.s8 $Qd, $Qn, $Qm",
                    (VCGEsv16i8 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.s16 $Qd, $Qn, $Qm",
                    (VCGEsv8i16 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.s32 $Qd, $Qn, $Qm",
                    (VCGEsv4i32 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.u8 $Qd, $Qn, $Qm",
                    (VCGEuv16i8 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.u16 $Qd, $Qn, $Qm",
                    (VCGEuv8i16 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.u32 $Qd, $Qn, $Qm",
                    (VCGEuv4i32 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vcle${p}.f32 $Qd, $Qn, $Qm",
                    (VCGEfq QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
let Predicates = [HasNEON, HasFullFP16] in
def : NEONInstAlias<"vcle${p}.f16 $Qd, $Qn, $Qm",
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 9093-9093
```tablegen
                    (VCGEhq QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9095-9112
```tablegen
// VCLT (register) is an assembler alias for VCGT w/ the operands reversed.
// D-register versions.
def : NEONInstAlias<"vclt${p}.s8 $Dd, $Dn, $Dm",
                    (VCGTsv8i8 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.s16 $Dd, $Dn, $Dm",
                    (VCGTsv4i16 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.s32 $Dd, $Dn, $Dm",
                    (VCGTsv2i32 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.u8 $Dd, $Dn, $Dm",
                    (VCGTuv8i8 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.u16 $Dd, $Dn, $Dm",
                    (VCGTuv4i16 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.u32 $Dd, $Dn, $Dm",
                    (VCGTuv2i32 DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.f32 $Dd, $Dn, $Dm",
                    (VCGTfd DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
let Predicates = [HasNEON, HasFullFP16] in
def : NEONInstAlias<"vclt${p}.f16 $Dd, $Dn, $Dm",
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 9113-9130
```tablegen
                    (VCGThd DPR:$Dd, DPR:$Dm, DPR:$Dn, pred:$p)>;
// Q-register versions.
def : NEONInstAlias<"vclt${p}.s8 $Qd, $Qn, $Qm",
                    (VCGTsv16i8 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.s16 $Qd, $Qn, $Qm",
                    (VCGTsv8i16 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.s32 $Qd, $Qn, $Qm",
                    (VCGTsv4i32 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.u8 $Qd, $Qn, $Qm",
                    (VCGTuv16i8 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.u16 $Qd, $Qn, $Qm",
                    (VCGTuv8i16 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.u32 $Qd, $Qn, $Qm",
                    (VCGTuv4i32 QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
def : NEONInstAlias<"vclt${p}.f32 $Qd, $Qn, $Qm",
                    (VCGTfq QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
let Predicates = [HasNEON, HasFullFP16] in
def : NEONInstAlias<"vclt${p}.f16 $Qd, $Qn, $Qm",
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 9131-9131
```tablegen
                    (VCGThq QPR:$Qd, QPR:$Qm, QPR:$Qn, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9133-9137
```tablegen
// VSWP allows, but does not require, a type suffix.
defm : NEONDTAnyInstAlias<"vswp${p}", "$Vd, $Vm",
                         (VSWPd DPR:$Vd, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vswp${p}", "$Vd, $Vm",
                         (VSWPq QPR:$Vd, QPR:$Vm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9139-9151
```tablegen
// VBIF, VBIT, and VBSL allow, but do not require, a type suffix.
defm : NEONDTAnyInstAlias<"vbif${p}", "$Vd, $Vn, $Vm",
                         (VBIFd DPR:$Vd, DPR:$Vn, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vbit${p}", "$Vd, $Vn, $Vm",
                         (VBITd DPR:$Vd, DPR:$Vn, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vbsl${p}", "$Vd, $Vn, $Vm",
                         (VBSLd DPR:$Vd, DPR:$Vn, DPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vbif${p}", "$Vd, $Vn, $Vm",
                         (VBIFq QPR:$Vd, QPR:$Vn, QPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vbit${p}", "$Vd, $Vn, $Vm",
                         (VBITq QPR:$Vd, QPR:$Vn, QPR:$Vm, pred:$p)>;
defm : NEONDTAnyInstAlias<"vbsl${p}", "$Vd, $Vn, $Vm",
                         (VBSLq QPR:$Vd, QPR:$Vn, QPR:$Vm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9153-9161
```tablegen
// "vmov Rd, #-imm" can be handled via "vmvn".
def : NEONInstAlias<"vmov${p}.i32 $Vd, $imm",
                    (VMVNv2i32 DPR:$Vd, nImmVMOVI32Neg:$imm, pred:$p)>;
def : NEONInstAlias<"vmov${p}.i32 $Vd, $imm",
                    (VMVNv4i32 QPR:$Vd, nImmVMOVI32Neg:$imm, pred:$p)>;
def : NEONInstAlias<"vmvn${p}.i32 $Vd, $imm",
                    (VMOVv2i32 DPR:$Vd, nImmVMOVI32Neg:$imm, pred:$p)>;
def : NEONInstAlias<"vmvn${p}.i32 $Vd, $imm",
                    (VMOVv4i32 QPR:$Vd, nImmVMOVI32Neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9163-9170
```tablegen
// 'gas' compatibility aliases for quad-word instructions. Strictly speaking,
// these should restrict to just the Q register variants, but the register
// classes are enough to match correctly regardless, so we keep it simple
// and just use MnemonicAlias.
def : NEONMnemonicAlias<"vbicq", "vbic">;
def : NEONMnemonicAlias<"vandq", "vand">;
def : NEONMnemonicAlias<"veorq", "veor">;
def : NEONMnemonicAlias<"vorrq", "vorr">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9172-9177
```tablegen
def : NEONMnemonicAlias<"vmovq", "vmov">;
def : NEONMnemonicAlias<"vmvnq", "vmvn">;
// Explicit versions for floating point so that the FPImm variants get
// handled early. The parser gets confused otherwise.
def : NEONMnemonicAlias<"vmovq.f32", "vmov.f32">;
def : NEONMnemonicAlias<"vmovq.f64", "vmov.f64">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9179-9180
```tablegen
def : NEONMnemonicAlias<"vaddq", "vadd">;
def : NEONMnemonicAlias<"vsubq", "vsub">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9182-9183
```tablegen
def : NEONMnemonicAlias<"vminq", "vmin">;
def : NEONMnemonicAlias<"vmaxq", "vmax">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9185-9185
```tablegen
def : NEONMnemonicAlias<"vmulq", "vmul">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9187-9187
```tablegen
def : NEONMnemonicAlias<"vabsq", "vabs">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9189-9190
```tablegen
def : NEONMnemonicAlias<"vshlq", "vshl">;
def : NEONMnemonicAlias<"vshrq", "vshr">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9192-9192
```tablegen
def : NEONMnemonicAlias<"vcvtq", "vcvt">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9194-9195
```tablegen
def : NEONMnemonicAlias<"vcleq", "vcle">;
def : NEONMnemonicAlias<"vceqq", "vceq">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9197-9198
```tablegen
def : NEONMnemonicAlias<"vzipq", "vzip">;
def : NEONMnemonicAlias<"vswpq", "vswp">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9200-9201
```tablegen
def : NEONMnemonicAlias<"vrecpeq.f32", "vrecpe.f32">;
def : NEONMnemonicAlias<"vrecpeq.u32", "vrecpe.u32">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9204-9210
```tablegen
// Alias for loading floating point immediates that aren't representable
// using the vmov.f32 encoding but the bitpattern is representable using
// the .i32 encoding.
def : NEONInstAlias<"vmov${p}.f32 $Vd, $imm",
                     (VMOVv4i32 QPR:$Vd, nImmVMOVI32:$imm, pred:$p)>;
def : NEONInstAlias<"vmov${p}.f32 $Vd, $imm",
                     (VMOVv2i32 DPR:$Vd, nImmVMOVI32:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9212-9220
```tablegen
// ARMv8.6a BFloat16 instructions.
let Predicates = [HasBF16, HasNEON] in {
class BF16VDOT<bits<5> op27_23, bits<2> op21_20, bit op6,
               dag oops, dag iops, list<dag> pattern>
   : N3Vnp<op27_23, op21_20, 0b1101, op6, 0, oops, iops,
           N3RegFrm, IIC_VDOTPROD, "", "", pattern>
{
    let DecoderNamespace = "VFPV8";
}
```
- EN: Declares reusable TableGen class `BF16VDOT` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `BF16VDOT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 9222-9232
```tablegen
class BF16VDOTS<bit Q, RegisterClass RegTy, string opc, ValueType AccumTy, ValueType InputTy>
   : BF16VDOT<0b11000, 0b00,  Q, (outs RegTy:$dst),
              (ins RegTy:$Vd, RegTy:$Vn, RegTy:$Vm),
            [(set (AccumTy RegTy:$dst),
                  (int_arm_neon_bfdot (AccumTy RegTy:$Vd),
                                      (InputTy RegTy:$Vn),
                                      (InputTy RegTy:$Vm)))]> {
  let Constraints = "$dst = $Vd";
  let AsmString = !strconcat(opc, ".bf16", "\t$Vd, $Vn, $Vm");
    let DecoderNamespace = "VFPV8";
}
```
- EN: Declares reusable TableGen class `BF16VDOTS` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `BF16VDOTS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 9234-9235
```tablegen
multiclass BF16VDOTI<bit Q, RegisterClass RegTy, string opc, ValueType AccumTy,
                     ValueType InputTy, dag RHS> {
```
- EN: Declares TableGen `multiclass BF16VDOTI`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass BF16VDOTI`，它是一个可复用模板，可展开为多个相关记录。

### Lines 9237-9246
```tablegen
  def "" : BF16VDOT<0b11100, 0b00, Q, (outs RegTy:$dst),
                    (ins RegTy:$Vd, RegTy:$Vn,
                    DPR_VFP2:$Vm, VectorIndex32:$lane), []> {
    bit lane;
    let Inst{5} = lane;
    let Constraints = "$dst = $Vd";
    let AsmString = !strconcat(opc, ".bf16", "\t$Vd, $Vn, $Vm$lane");
    let DecoderNamespace = "VFPV8";
    let hasSideEffects = 0;
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 9248-9255
```tablegen
  def : Pat<
    (AccumTy (int_arm_neon_bfdot (AccumTy RegTy:$Vd),
                                 (InputTy RegTy:$Vn),
                                 (InputTy (bitconvert (AccumTy
                                          (ARMvduplane (AccumTy RegTy:$Vm),
                                                        VectorIndex32:$lane)))))),
    (!cast<Instruction>(NAME) RegTy:$Vd, RegTy:$Vn, RHS, VectorIndex32:$lane)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9257-9258
```tablegen
def BF16VDOTS_VDOTD : BF16VDOTS<0, DPR, "vdot", v2f32, v4bf16>;
def BF16VDOTS_VDOTQ : BF16VDOTS<1, QPR, "vdot", v4f32, v8bf16>;
```
- EN: Defines TableGen record `BF16VDOTS_VDOTD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BF16VDOTS_VDOTD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 9260-9261
```tablegen
defm BF16VDOTI_VDOTD : BF16VDOTI<0, DPR, "vdot", v2f32, v4bf16, (v2f32 DPR_VFP2:$Vm)>;
defm BF16VDOTI_VDOTQ : BF16VDOTI<1, QPR, "vdot", v4f32, v8bf16, (EXTRACT_SUBREG QPR:$Vm, dsub_0)>;
```
- EN: Defines TableGen record `BF16VDOTI_VDOTD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BF16VDOTI_VDOTD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 9263-9274
```tablegen
class BF16MM<bit Q, RegisterClass RegTy,
             string opc>
   : N3Vnp<0b11000, 0b00, 0b1100, Q, 0,
           (outs RegTy:$dst), (ins RegTy:$Vd, RegTy:$Vn, RegTy:$Vm),
           N3RegFrm, IIC_VDOTPROD, "", "",
                [(set (v4f32 QPR:$dst), (int_arm_neon_bfmmla (v4f32 QPR:$Vd),
                                                (v8bf16 QPR:$Vn),
                                                (v8bf16 QPR:$Vm)))]> {
   let Constraints = "$dst = $Vd";
   let AsmString = !strconcat(opc, ".bf16", "\t$Vd, $Vn, $Vm");
   let DecoderNamespace = "VFPV8";
}
```
- EN: Declares reusable TableGen class `BF16MM` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `BF16MM`，通常用于抽象共享字段、谓词或编码结构。

### Lines 9276-9276
```tablegen
def VMMLA : BF16MM<1, QPR, "vmmla">;
```
- EN: Defines TableGen record `VMMLA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMMLA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 9278-9288
```tablegen
class VBF16MALQ<bit T, string suffix, SDPatternOperator OpNode>
  : N3VCP8<0b00, 0b11, T, 1,
           (outs QPR:$dst), (ins QPR:$Vd, QPR:$Vn, QPR:$Vm),
           NoItinerary, "vfma" # suffix, "bf16", "$Vd, $Vn, $Vm", "",
                [(set (v4f32 QPR:$dst),
                      (OpNode (v4f32 QPR:$Vd),
                              (v8bf16 QPR:$Vn),
                              (v8bf16 QPR:$Vm)))]> {
  let Constraints = "$dst = $Vd";
  let DecoderNamespace = "VFPV8";
}
```
- EN: Declares reusable TableGen class `VBF16MALQ` for `ARMInstrNEON`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrNEON` 声明可复用的 TableGen 类 `VBF16MALQ`，通常用于抽象共享字段、谓词或编码结构。

### Lines 9290-9291
```tablegen
def VBF16MALTQ: VBF16MALQ<1, "t", int_arm_neon_bfmlalt>;
def VBF16MALBQ: VBF16MALQ<0, "b", int_arm_neon_bfmlalb>;
```
- EN: Defines TableGen record `VBF16MALTQ:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBF16MALTQ:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 9293-9303
```tablegen
multiclass VBF16MALQI<bit T, string suffix, SDPatternOperator OpNode> {
  def "" : N3VLaneCP8<0, 0b11, T, 1, (outs QPR:$dst),
              (ins QPR:$Vd, QPR:$Vn, DPR_8:$Vm, VectorIndex16:$idx),
               IIC_VMACD, "vfma" # suffix, "bf16", "$Vd, $Vn, $Vm$idx", "", []> {
  bits<2> idx;
  let Inst{5} = idx{1};
  let Inst{3} = idx{0};
  let Constraints = "$dst = $Vd";
  let DecoderNamespace = "VFPV8";
  let hasSideEffects = 0;
  }
```
- EN: Declares TableGen `multiclass VBF16MALQI`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass VBF16MALQI`，它是一个可复用模板，可展开为多个相关记录。

### Lines 9305-9315
```tablegen
  def : Pat<
    (v4f32 (OpNode (v4f32 QPR:$Vd),
                   (v8bf16 QPR:$Vn),
                   (v8bf16 (ARMvduplane (v8bf16 QPR:$Vm),
                            VectorIndex16:$lane)))),
    (!cast<Instruction>(NAME) QPR:$Vd,
                              QPR:$Vn,
                              (EXTRACT_SUBREG QPR:$Vm,
                                (DSubReg_i16_reg VectorIndex16:$lane)),
                              (SubReg_i16_lane VectorIndex16:$lane))>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 9317-9318
```tablegen
defm VBF16MALTQI: VBF16MALQI<1, "t", int_arm_neon_bfmlalt>;
defm VBF16MALBQI: VBF16MALQI<0, "b", int_arm_neon_bfmlalb>;
```
- EN: Defines TableGen record `VBF16MALTQI:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VBF16MALTQI:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 9320-9325
```tablegen
let hasSideEffects = 0 in
def BF16_VCVT :  N2V<0b11, 0b11, 0b01, 0b10, 0b01100, 1, 0,
                    (outs DPR:$Vd), (ins QPR:$Vm),
                    NoItinerary, "vcvt", "bf16.f32", "$Vd, $Vm", "", []>;
} // Predicates = [HasBF16, HasNEON]
// End of BFloat16 instructions
```
- EN: Defines TableGen record `BF16_VCVT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BF16_VCVT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: instruction definitions and target opcode metadata.
  - CN: 核心职责：指令定义与目标操作码元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: SelectionDAG patterns or node profiles connect IR-level intent to target instructions.
  - CN: SelectionDAG 模式或节点轮廓把 IR 层意图连接到目标指令。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
