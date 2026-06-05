# MipsInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsInstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This file contains the Mips implementation of the TargetInstrInfo class.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsInstrInfo`，涵盖指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===- MipsInstrInfo.td - Target Description for Mips Target -*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains the Mips implementation of the TargetInstrInfo class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 14-17
```tablegen
//===----------------------------------------------------------------------===//
// Mips profiles and nodes
//===----------------------------------------------------------------------===//
include "MipsInstrCompiler.td"
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 19-34
```tablegen
def SDT_MipsJmpLink      : SDTypeProfile<0, 1, [SDTCisVT<0, iPTR>]>;
def SDT_MipsCMov         : SDTypeProfile<1, 4, [SDTCisSameAs<0, 1>,
                                                SDTCisSameAs<1, 2>,
                                                SDTCisSameAs<3, 4>,
                                                SDTCisInt<4>]>;
def SDT_MipsCallSeqStart : SDCallSeqStart<[SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
def SDT_MipsCallSeqEnd   : SDCallSeqEnd<[SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;
def SDT_MFLOHI : SDTypeProfile<1, 1, [SDTCisInt<0>, SDTCisVT<1, untyped>]>;
def SDT_MTLOHI : SDTypeProfile<1, 2, [SDTCisVT<0, untyped>,
                                      SDTCisInt<1>, SDTCisSameAs<1, 2>]>;
def SDT_MipsMultDiv : SDTypeProfile<1, 2, [SDTCisVT<0, untyped>, SDTCisInt<1>,
                                    SDTCisSameAs<1, 2>]>;
def SDT_MipsMAddMSub : SDTypeProfile<1, 3,
                                     [SDTCisVT<0, untyped>, SDTCisSameAs<0, 3>,
                                      SDTCisVT<1, i32>, SDTCisSameAs<1, 2>]>;
def SDT_MipsDivRem16 : SDTypeProfile<0, 2, [SDTCisInt<0>, SDTCisSameAs<0, 1>]>;
```
- EN: Defines TableGen record `SDT_MipsJmpLink` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_MipsJmpLink`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 36-36
```tablegen
def SDT_MipsThreadPointer : SDTypeProfile<1, 0, [SDTCisPtrTy<0>]>;
```
- EN: Defines TableGen record `SDT_MipsThreadPointer` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_MipsThreadPointer`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 38-38
```tablegen
def SDT_Sync             : SDTypeProfile<0, 1, [SDTCisVT<0, i32>]>;
```
- EN: Defines TableGen record `SDT_Sync` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_Sync`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 40-44
```tablegen
def SDT_Ext : SDTypeProfile<1, 3, [SDTCisInt<0>, SDTCisSameAs<0, 1>,
                                   SDTCisVT<2, i32>, SDTCisSameAs<2, 3>]>;
def SDT_Ins : SDTypeProfile<1, 4, [SDTCisInt<0>, SDTCisSameAs<0, 1>,
                                   SDTCisVT<2, i32>, SDTCisSameAs<2, 3>,
                                   SDTCisSameAs<0, 4>]>;
```
- EN: Defines TableGen record `SDT_Ext` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_Ext`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 46-48
```tablegen
def SDTMipsLoadLR  : SDTypeProfile<1, 2,
                                   [SDTCisInt<0>, SDTCisPtrTy<1>,
                                    SDTCisSameAs<0, 2>]>;
```
- EN: Defines TableGen record `SDTMipsLoadLR` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTMipsLoadLR`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 50-53
```tablegen
// Jump and link (call)
def MipsJmpLink : SDNode<"MipsISD::JmpLink",SDT_MipsJmpLink,
                         [SDNPHasChain, SDNPOutGlue, SDNPOptInGlue,
                          SDNPVariadic]>;
```
- EN: Defines TableGen record `MipsJmpLink` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsJmpLink`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 55-57
```tablegen
// Tail call
def MipsTailCall : SDNode<"MipsISD::TailCall", SDT_MipsJmpLink,
                          [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
```
- EN: Defines TableGen record `MipsTailCall` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsTailCall`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 59-61
```tablegen
// Hi and Lo nodes are used to handle global addresses. Used on
// MipsISelLowering to lower stuff like GlobalAddress, ExternalSymbol
// static model. (nothing to do with Mips Registers Hi and Lo)
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 63-67
```tablegen
// Hi is the odd node out, on MIPS64 it can expand to either daddiu when
// using static relocations with 64 bit symbols, or lui when using 32 bit
// symbols.
// Get the Higher (47-32) 16 bits from a 64-bit immediate
def MipsHigher : SDNode<"MipsISD::Higher", SDTIntUnaryOp>;
```
- EN: Defines TableGen record `MipsHigher` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsHigher`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 69-70
```tablegen
// Get the Highest (63-48) 16 bits from a 64-bit immediate
def MipsHighest : SDNode<"MipsISD::Highest", SDTIntUnaryOp>;
```
- EN: Defines TableGen record `MipsHighest` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsHighest`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 72-74
```tablegen
// Get the High 16 bits from a 32/64-bit immediate
// No relation with Mips Hi register
def MipsHi    : SDNode<"MipsISD::Hi", SDTIntUnaryOp>;
```
- EN: Defines TableGen record `MipsHi` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsHi`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 76-78
```tablegen
// Get the Lower 16 bits from a 32/64-bit immediate
// No relation with Mips Lo register
def MipsLo    : SDNode<"MipsISD::Lo", SDTIntUnaryOp>;
```
- EN: Defines TableGen record `MipsLo` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsLo`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 80-81
```tablegen
// Handle gp_rel (small data/bss sections) relocation.
def MipsGPRel : SDNode<"MipsISD::GPRel", SDTIntUnaryOp>;
```
- EN: Defines TableGen record `MipsGPRel` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsGPRel`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 83-84
```tablegen
// Get the High 16 bits from a 32 bit immediate for accessing the GOT.
def MipsGotHi : SDNode<"MipsISD::GotHi", SDTIntUnaryOp>;
```
- EN: Defines TableGen record `MipsGotHi` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsGotHi`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 86-87
```tablegen
// Get the High 16 bits from a 32-bit immediate for accessing TLS.
def MipsTlsHi   : SDNode<"MipsISD::TlsHi", SDTIntUnaryOp>;
```
- EN: Defines TableGen record `MipsTlsHi` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsTlsHi`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 89-90
```tablegen
// Thread Pointer
def MipsThreadPointer: SDNode<"MipsISD::ThreadPointer", SDT_MipsThreadPointer>;
```
- EN: Defines TableGen record `MipsThreadPointer:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsThreadPointer:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 92-94
```tablegen
// Return
def MipsRet : SDNode<"MipsISD::Ret", SDTNone,
                     [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
```
- EN: Defines TableGen record `MipsRet` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsRet`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 96-98
```tablegen
// Interrupt, exception, error trap Return
def MipsERet : SDNode<"MipsISD::ERet", SDTNone,
                      [SDNPHasChain, SDNPOptInGlue, SDNPSideEffect]>;
```
- EN: Defines TableGen record `MipsERet` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsERet`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 100-105
```tablegen
// These are target-independent nodes, but have target-specific formats.
def callseq_start : SDNode<"ISD::CALLSEQ_START", SDT_MipsCallSeqStart,
                           [SDNPHasChain, SDNPSideEffect, SDNPOutGlue]>;
def callseq_end   : SDNode<"ISD::CALLSEQ_END", SDT_MipsCallSeqEnd,
                           [SDNPHasChain, SDNPSideEffect,
                            SDNPOptInGlue, SDNPOutGlue]>;
```
- EN: Defines TableGen record `callseq_start` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `callseq_start`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 107-109
```tablegen
// Nodes used to extract LO/HI registers.
def MipsMFHI : SDNode<"MipsISD::MFHI", SDT_MFLOHI>;
def MipsMFLO : SDNode<"MipsISD::MFLO", SDT_MFLOHI>;
```
- EN: Defines TableGen record `MipsMFHI` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsMFHI`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 111-112
```tablegen
// Node used to insert 32-bit integers to LOHI register pair.
def MipsMTLOHI : SDNode<"MipsISD::MTLOHI", SDT_MTLOHI>;
```
- EN: Defines TableGen record `MipsMTLOHI` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsMTLOHI`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 114-116
```tablegen
// Mult nodes.
def MipsMult  : SDNode<"MipsISD::Mult", SDT_MipsMultDiv>;
def MipsMultu : SDNode<"MipsISD::Multu", SDT_MipsMultDiv>;
```
- EN: Defines TableGen record `MipsMult` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsMult`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 118-122
```tablegen
// MAdd*/MSub* nodes
def MipsMAdd  : SDNode<"MipsISD::MAdd", SDT_MipsMAddMSub>;
def MipsMAddu : SDNode<"MipsISD::MAddu", SDT_MipsMAddMSub>;
def MipsMSub  : SDNode<"MipsISD::MSub", SDT_MipsMAddMSub>;
def MipsMSubu : SDNode<"MipsISD::MSubu", SDT_MipsMAddMSub>;
```
- EN: Defines TableGen record `MipsMAdd` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsMAdd`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 124-130
```tablegen
// DivRem(u) nodes
def MipsDivRem    : SDNode<"MipsISD::DivRem", SDT_MipsMultDiv>;
def MipsDivRemU   : SDNode<"MipsISD::DivRemU", SDT_MipsMultDiv>;
def MipsDivRem16  : SDNode<"MipsISD::DivRem16", SDT_MipsDivRem16,
                           [SDNPOutGlue]>;
def MipsDivRemU16 : SDNode<"MipsISD::DivRemU16", SDT_MipsDivRem16,
                           [SDNPOutGlue]>;
```
- EN: Defines TableGen record `MipsDivRem` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsDivRem`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 132-140
```tablegen
// Target constant nodes that are not part of any isel patterns and remain
// unchanged can cause instructions with illegal operands to be emitted.
// Wrapper node patterns give the instruction selector a chance to replace
// target constant nodes that would otherwise remain unchanged with ADDiu
// nodes. Without these wrapper node patterns, the following conditional move
// instruction is emitted when function cmov2 in test/CodeGen/Mips/cmov.ll is
// compiled:
//  movn  %got(d)($gp), %got(c)($gp), $4
// This instruction is illegal since movn can take only register operands.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 142-142
```tablegen
def MipsWrapper    : SDNode<"MipsISD::Wrapper", SDTIntBinOp>;
```
- EN: Defines TableGen record `MipsWrapper` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsWrapper`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 144-144
```tablegen
def MipsSync : SDNode<"MipsISD::Sync", SDT_Sync, [SDNPHasChain,SDNPSideEffect]>;
```
- EN: Defines TableGen record `MipsSync` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsSync`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 146-148
```tablegen
def MipsExt :  SDNode<"MipsISD::Ext", SDT_Ext>;
def MipsIns :  SDNode<"MipsISD::Ins", SDT_Ins>;
def MipsCIns : SDNode<"MipsISD::CIns", SDT_Ext>;
```
- EN: Defines TableGen record `MipsExt` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsExt`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 150-166
```tablegen
// Load/Store Left/Right nodes.
def MipsLWL : SDNode<"MipsISD::LWL", SDTMipsLoadLR,
                     [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;
def MipsLWR : SDNode<"MipsISD::LWR", SDTMipsLoadLR,
                     [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;
def MipsSWL : SDNode<"MipsISD::SWL", SDTStore,
                     [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
def MipsSWR : SDNode<"MipsISD::SWR", SDTStore,
                     [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
def MipsLDL : SDNode<"MipsISD::LDL", SDTMipsLoadLR,
                     [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;
def MipsLDR : SDNode<"MipsISD::LDR", SDTMipsLoadLR,
                     [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;
def MipsSDL : SDNode<"MipsISD::SDL", SDTStore,
                     [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
def MipsSDR : SDNode<"MipsISD::SDR", SDTStore,
                     [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
```
- EN: Defines TableGen record `MipsLWL` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsLWL`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 168-171
```tablegen
//===----------------------------------------------------------------------===//
// Mips Instruction Predicate Definitions.
//===----------------------------------------------------------------------===//
// FIXME: Move to Mips.td
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 173-190
```tablegen
def HasMips2     :    Predicate<"Subtarget->hasMips2()">,
                      AssemblerPredicate<(all_of FeatureMips2)>;
def HasMips3_32  :    Predicate<"Subtarget->hasMips3_32()">,
                      AssemblerPredicate<(all_of FeatureMips3_32)>;
def HasMips3_32r2 :   Predicate<"Subtarget->hasMips3_32r2()">,
                      AssemblerPredicate<(all_of FeatureMips3_32r2)>;
def HasMips3     :    Predicate<"Subtarget->hasMips3()">,
                      AssemblerPredicate<(all_of FeatureMips3)>;
def NotMips3     :    Predicate<"!Subtarget->hasMips3()">,
                      AssemblerPredicate<(all_of (not FeatureMips3))>;
def HasMips4_32  :    Predicate<"Subtarget->hasMips4_32()">,
                      AssemblerPredicate<(all_of FeatureMips4_32)>;
def NotMips4_32  :    Predicate<"!Subtarget->hasMips4_32()">,
                      AssemblerPredicate<(all_of (not FeatureMips4_32))>;
def HasMips4_32r2 :   Predicate<"Subtarget->hasMips4_32r2()">,
                      AssemblerPredicate<(all_of FeatureMips4_32r2)>;
def HasMips5_32r2 :   Predicate<"Subtarget->hasMips5_32r2()">,
                      AssemblerPredicate<(all_of FeatureMips5_32r2)>;
```
- EN: Defines TableGen record `HasMips2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasMips2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 191-208
```tablegen
def HasMips32    :    Predicate<"Subtarget->hasMips32()">,
                      AssemblerPredicate<(all_of FeatureMips32)>;
def HasMips32r2  :    Predicate<"Subtarget->hasMips32r2()">,
                      AssemblerPredicate<(all_of FeatureMips32r2)>;
def HasMips32r5  :    Predicate<"Subtarget->hasMips32r5()">,
                      AssemblerPredicate<(all_of FeatureMips32r5)>;
def HasMips32r6  :    Predicate<"Subtarget->hasMips32r6()">,
                      AssemblerPredicate<(all_of FeatureMips32r6)>;
def NotMips32r6  :    Predicate<"!Subtarget->hasMips32r6()">,
                      AssemblerPredicate<(all_of (not FeatureMips32r6))>;
def IsGP64bit    :    Predicate<"Subtarget->isGP64bit()">,
                      AssemblerPredicate<(all_of FeatureGP64Bit)>;
def IsGP32bit    :    Predicate<"!Subtarget->isGP64bit()">,
                      AssemblerPredicate<(all_of (not FeatureGP64Bit))>;
def HasMips64    :    Predicate<"Subtarget->hasMips64()">,
                      AssemblerPredicate<(all_of FeatureMips64)>;
def NotMips64    :    Predicate<"!Subtarget->hasMips64()">,
                      AssemblerPredicate<(all_of (not FeatureMips64))>;
```
- EN: Defines TableGen record `HasMips32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasMips32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 209-226
```tablegen
def HasMips64r2  :    Predicate<"Subtarget->hasMips64r2()">,
                      AssemblerPredicate<(all_of FeatureMips64r2)>;
def HasMips64r5  :    Predicate<"Subtarget->hasMips64r5()">,
                      AssemblerPredicate<(all_of FeatureMips64r5)>;
def HasMips64r6  :    Predicate<"Subtarget->hasMips64r6()">,
                      AssemblerPredicate<(all_of FeatureMips64r6)>;
def NotMips64r6  :    Predicate<"!Subtarget->hasMips64r6()">,
                      AssemblerPredicate<(all_of (not FeatureMips64r6))>;
def InMips16Mode :    Predicate<"Subtarget->inMips16Mode()">,
                      AssemblerPredicate<(all_of FeatureMips16)>;
def NotInMips16Mode : Predicate<"!Subtarget->inMips16Mode()">,
                      AssemblerPredicate<(all_of (not FeatureMips16))>;
def HasCnMips    :    Predicate<"Subtarget->hasCnMips()">,
                      AssemblerPredicate<(all_of FeatureCnMips)>;
def NotCnMips    :    Predicate<"!Subtarget->hasCnMips()">,
                      AssemblerPredicate<(all_of (not FeatureCnMips))>;
def HasCnMipsP   :    Predicate<"Subtarget->hasCnMipsP()">,
                      AssemblerPredicate<(all_of FeatureCnMipsP)>;
```
- EN: Defines TableGen record `HasMips64r2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasMips64r2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 227-244
```tablegen
def NotCnMipsP   :    Predicate<"!Subtarget->hasCnMipsP()">,
                      AssemblerPredicate<(all_of (not FeatureCnMipsP))>;
def IsSym32     :     Predicate<"Subtarget->hasSym32()">,
                      AssemblerPredicate<(all_of FeatureSym32)>;
def IsSym64     :     Predicate<"!Subtarget->hasSym32()">,
                      AssemblerPredicate<(all_of (not FeatureSym32))>;
def IsN64       :     Predicate<"Subtarget->isABI_N64()">;
def IsNotN64    :     Predicate<"!Subtarget->isABI_N64()">;
def RelocNotPIC :     Predicate<"!TM.isPositionIndependent()">;
def RelocPIC    :     Predicate<"TM.isPositionIndependent()">;
def HasStdEnc :       Predicate<"Subtarget->hasStandardEncoding()">,
                      AssemblerPredicate<(all_of (not FeatureMips16))>;
def NotDSP :          Predicate<"!Subtarget->hasDSP()">;
def InMicroMips    :  Predicate<"Subtarget->inMicroMipsMode()">,
                      AssemblerPredicate<(all_of FeatureMicroMips)>;
def NotInMicroMips :  Predicate<"!Subtarget->inMicroMipsMode()">,
                      AssemblerPredicate<(all_of (not FeatureMicroMips))>;
def IsLE           :  Predicate<"Subtarget->isLittle()">;
```
- EN: Defines TableGen record `NotCnMipsP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NotCnMipsP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 245-262
```tablegen
def IsBE           :  Predicate<"!Subtarget->isLittle()">;
def UseTCCInDIV    :  AssemblerPredicate<(all_of FeatureUseTCCInDIV)>;
def HasEVA       :    Predicate<"Subtarget->hasEVA()">,
                      AssemblerPredicate<(all_of FeatureEVA)>;
def HasMSA : Predicate<"Subtarget->hasMSA()">,
             AssemblerPredicate<(all_of FeatureMSA)>;
def HasMadd4 : Predicate<"!Subtarget->disableMadd4()">,
               AssemblerPredicate<(all_of (not FeatureNoMadd4))>;
def HasMT  : Predicate<"Subtarget->hasMT()">,
             AssemblerPredicate<(all_of FeatureMT)>;
def UseIndirectJumpsHazard : Predicate<"Subtarget->useIndirectJumpsHazard()">,
                            AssemblerPredicate<(all_of FeatureUseIndirectJumpsHazard)>;
def NoIndirectJumpGuards : Predicate<"!Subtarget->useIndirectJumpsHazard()">,
                           AssemblerPredicate<(all_of (not FeatureUseIndirectJumpsHazard))>;
def IsR5900 : Predicate<"Subtarget->isR5900()">,
              AssemblerPredicate<(all_of FeatureR5900)>;
def NotR5900 : Predicate<"!Subtarget->isR5900()">,
               AssemblerPredicate<(all_of (not FeatureR5900))>;
```
- EN: Defines TableGen record `IsBE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsBE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 263-275
```tablegen
def HasCRC   : Predicate<"Subtarget->hasCRC()">,
               AssemblerPredicate<(all_of FeatureCRC)>;
def HasVirt  : Predicate<"Subtarget->hasVirt()">,
               AssemblerPredicate<(all_of FeatureVirt)>;
def HasGINV  : Predicate<"Subtarget->hasGINV()">,
               AssemblerPredicate<(all_of FeatureGINV)>;
// TODO: Add support for FPOpFusion::Standard
def AllowFPOpFusion : Predicate<"TM.Options.AllowFPOpFusion =="
                                " FPOpFusion::Fast">;
//===----------------------------------------------------------------------===//
// Mips GPR size adjectives.
// They are mutually exclusive.
//===----------------------------------------------------------------------===//
```
- EN: Defines TableGen record `HasCRC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HasCRC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 277-278
```tablegen
class GPR_32 { list<Predicate> GPRPredicates = [IsGP32bit]; }
class GPR_64 { list<Predicate> GPRPredicates = [IsGP64bit]; }
```
- EN: Declares reusable TableGen class `GPR_32` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `GPR_32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 280-281
```tablegen
class PTR_32 { list<Predicate> PTRPredicates = [IsPTR32bit]; }
class PTR_64 { list<Predicate> PTRPredicates = [IsPTR64bit]; }
```
- EN: Declares reusable TableGen class `PTR_32` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `PTR_32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 283-286
```tablegen
//===----------------------------------------------------------------------===//
// Mips Symbol size adjectives.
// They are mutally exculsive.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 288-289
```tablegen
class SYM_32 { list<Predicate> SYMPredicates = [IsSym32]; }
class SYM_64 { list<Predicate> SYMPredicates = [IsSym64]; }
```
- EN: Declares reusable TableGen class `SYM_32` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SYM_32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 291-294
```tablegen
//===----------------------------------------------------------------------===//
// Mips ISA/ASE membership and instruction group membership adjectives.
// They are mutually exclusive.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 296-313
```tablegen
// FIXME: I'd prefer to use additive predicates to build the instruction sets
//        but we are short on assembler feature bits at the moment. Using a
//        subtractive predicate will hopefully keep us under the 32 predicate
//        limit long enough to develop an alternative way to handle P1||P2
//        predicates.
class ISA_MIPS1 {
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS1_NOT_MIPS3 {
  list<Predicate> InsnPredicates = [NotMips3];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS1_NOT_4_32 {
  list<Predicate> InsnPredicates = [NotMips4_32];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS1_NOT_32R6_64R6 {
  list<Predicate> InsnPredicates = [NotMips32r6, NotMips64r6];
```
- EN: Declares reusable TableGen class `ISA_MIPS1` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ISA_MIPS1`，通常用于抽象共享字段、谓词或编码结构。

### Lines 314-331
```tablegen
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS1_NOT_32R6_64R6_R5900 {
  list<Predicate> InsnPredicates = [NotMips32r6, NotMips64r6, NotR5900];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS2 {
  list<Predicate> InsnPredicates = [HasMips2];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS2_NOT_R5900 {
  list<Predicate> InsnPredicates = [HasMips2, NotR5900];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS2_NOT_32R6_64R6 {
  list<Predicate> InsnPredicates = [HasMips2, NotMips32r6, NotMips64r6];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `ISA_MIPS1_NOT_32R6_64R6_R5900` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ISA_MIPS1_NOT_32R6_64R6_R5900`，通常用于抽象共享字段、谓词或编码结构。

### Lines 332-349
```tablegen
class ISA_MIPS2_NOT_32R6_64R6_R5900 {
  list<Predicate> InsnPredicates = [HasMips2, NotMips32r6, NotMips64r6,
                                    NotR5900];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS3 {
  list<Predicate> InsnPredicates = [HasMips3];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS3_NOT_32R6_64R6 {
  list<Predicate> InsnPredicates = [HasMips3, NotMips32r6, NotMips64r6];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS3_NOT_32R6_64R6_R5900 {
  list<Predicate> InsnPredicates = [HasMips3, NotMips32r6, NotMips64r6,
                                    NotR5900];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `ISA_MIPS2_NOT_32R6_64R6_R5900` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ISA_MIPS2_NOT_32R6_64R6_R5900`，通常用于抽象共享字段、谓词或编码结构。

### Lines 350-367
```tablegen
class ISA_MIPS32 {
  list<Predicate> InsnPredicates = [HasMips32];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS32_NOT_32R6_64R6 {
  list<Predicate> InsnPredicates = [HasMips32, NotMips32r6, NotMips64r6];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS32R2 {
  list<Predicate> InsnPredicates = [HasMips32r2];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS32R2_NOT_32R6_64R6 {
  list<Predicate> InsnPredicates = [HasMips32r2, NotMips32r6, NotMips64r6];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS32R5 {
  list<Predicate> InsnPredicates = [HasMips32r5];
```
- EN: Declares reusable TableGen class `ISA_MIPS32` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ISA_MIPS32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 368-385
```tablegen
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS64 {
  list<Predicate> InsnPredicates = [HasMips64];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS64_NOT_64R6 {
  list<Predicate> InsnPredicates = [HasMips64, NotMips64r6];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS64R2 {
  list<Predicate> InsnPredicates = [HasMips64r2];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS64R5 {
  list<Predicate> InsnPredicates = [HasMips64r5];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `ISA_MIPS64` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ISA_MIPS64`，通常用于抽象共享字段、谓词或编码结构。

### Lines 386-403
```tablegen
class ISA_MIPS32R6 {
  list<Predicate> InsnPredicates = [HasMips32r6];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MIPS64R6 {
  list<Predicate> InsnPredicates = [HasMips64r6];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
class ISA_MICROMIPS {
  list<Predicate> EncodingPredicates = [InMicroMips];
}
class ISA_MICROMIPS32R5 {
  list<Predicate> InsnPredicates = [HasMips32r5];
  list<Predicate> EncodingPredicates = [InMicroMips];
}
class ISA_MICROMIPS32R6 {
  list<Predicate> InsnPredicates = [HasMips32r6];
  list<Predicate> EncodingPredicates = [InMicroMips];
```
- EN: Declares reusable TableGen class `ISA_MIPS32R6` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ISA_MIPS32R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 404-413
```tablegen
}
class ISA_MICROMIPS64R6 {
  list<Predicate> InsnPredicates = [HasMips64r6];
  list<Predicate> EncodingPredicates = [InMicroMips];
}
class ISA_MICROMIPS32_NOT_MIPS32R6 {
  list<Predicate> InsnPredicates = [NotMips32r6];
  list<Predicate> EncodingPredicates = [InMicroMips];
}
class ASE_EVA { list<Predicate> ASEPredicate = [HasEVA]; }
```
- EN: Declares reusable TableGen class `ISA_MICROMIPS64R6` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ISA_MICROMIPS64R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 415-419
```tablegen
// The portions of MIPS-III that were also added to MIPS32
class INSN_MIPS3_32 {
  list<Predicate> InsnPredicates = [HasMips3_32];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `INSN_MIPS3_32` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `INSN_MIPS3_32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 421-426
```tablegen
// The portions of MIPS-III that were also added to MIPS32 but were removed in
// MIPS32r6 and MIPS64r6.
class INSN_MIPS3_32_NOT_32R6_64R6 {
  list<Predicate> InsnPredicates = [HasMips3_32, NotMips32r6, NotMips64r6];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `INSN_MIPS3_32_NOT_32R6_64R6` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `INSN_MIPS3_32_NOT_32R6_64R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 428-432
```tablegen
// The portions of MIPS-III that were also added to MIPS32
class INSN_MIPS3_32R2 {
  list<Predicate> InsnPredicates = [HasMips3_32r2];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `INSN_MIPS3_32R2` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `INSN_MIPS3_32R2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 434-438
```tablegen
// The portions of MIPS-IV that were also added to MIPS32.
class INSN_MIPS4_32 {
  list <Predicate> InsnPredicates = [HasMips4_32];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `INSN_MIPS4_32` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `INSN_MIPS4_32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 440-445
```tablegen
// The portions of MIPS-IV that were also added to MIPS32 but were removed in
// MIPS32r6 and MIPS64r6.
class INSN_MIPS4_32_NOT_32R6_64R6 {
  list<Predicate> InsnPredicates = [HasMips4_32, NotMips32r6, NotMips64r6];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `INSN_MIPS4_32_NOT_32R6_64R6` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `INSN_MIPS4_32_NOT_32R6_64R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 447-452
```tablegen
// The portions of MIPS-IV that were also added to MIPS32r2 but were removed in
// MIPS32r6 and MIPS64r6.
class INSN_MIPS4_32R2_NOT_32R6_64R6 {
  list<Predicate> InsnPredicates = [HasMips4_32r2, NotMips32r6, NotMips64r6];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `INSN_MIPS4_32R2_NOT_32R6_64R6` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `INSN_MIPS4_32R2_NOT_32R6_64R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 454-458
```tablegen
// The portions of MIPS-IV that were also added to MIPS32r2.
class INSN_MIPS4_32R2 {
  list<Predicate> InsnPredicates = [HasMips4_32r2];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `INSN_MIPS4_32R2` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `INSN_MIPS4_32R2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 460-465
```tablegen
// The portions of MIPS-V that were also added to MIPS32r2 but were removed in
// MIPS32r6 and MIPS64r6.
class INSN_MIPS5_32R2_NOT_32R6_64R6 {
  list<Predicate> InsnPredicates = [HasMips5_32r2, NotMips32r6, NotMips64r6];
  list<Predicate> EncodingPredicates = [HasStdEnc];
}
```
- EN: Declares reusable TableGen class `INSN_MIPS5_32R2_NOT_32R6_64R6` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `INSN_MIPS5_32R2_NOT_32R6_64R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 467-469
```tablegen
class ASE_CNMIPS {
  list<Predicate> ASEPredicate = [HasCnMips];
}
```
- EN: Declares reusable TableGen class `ASE_CNMIPS` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ASE_CNMIPS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 471-473
```tablegen
class NOT_ASE_CNMIPS {
  list<Predicate> ASEPredicate = [NotCnMips];
}
```
- EN: Declares reusable TableGen class `NOT_ASE_CNMIPS` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `NOT_ASE_CNMIPS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 475-477
```tablegen
class ASE_CNMIPSP {
  list<Predicate> ASEPredicate = [HasCnMipsP];
}
```
- EN: Declares reusable TableGen class `ASE_CNMIPSP` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ASE_CNMIPSP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 479-481
```tablegen
class NOT_ASE_CNMIPSP {
  list<Predicate> ASEPredicate = [NotCnMipsP];
}
```
- EN: Declares reusable TableGen class `NOT_ASE_CNMIPSP` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `NOT_ASE_CNMIPSP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 483-485
```tablegen
class ASE_MIPS64_CNMIPS {
  list<Predicate> ASEPredicate = [HasMips64, HasCnMips];
}
```
- EN: Declares reusable TableGen class `ASE_MIPS64_CNMIPS` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ASE_MIPS64_CNMIPS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 487-489
```tablegen
class ASE_MSA {
  list<Predicate> ASEPredicate = [HasMSA];
}
```
- EN: Declares reusable TableGen class `ASE_MSA` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ASE_MSA`，通常用于抽象共享字段、谓词或编码结构。

### Lines 491-493
```tablegen
class ASE_MSA_NOT_MSA64 {
  list<Predicate> ASEPredicate = [HasMSA, NotMips64];
}
```
- EN: Declares reusable TableGen class `ASE_MSA_NOT_MSA64` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ASE_MSA_NOT_MSA64`，通常用于抽象共享字段、谓词或编码结构。

### Lines 495-497
```tablegen
class ASE_MSA64 {
  list<Predicate> ASEPredicate = [HasMSA, HasMips64];
}
```
- EN: Declares reusable TableGen class `ASE_MSA64` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ASE_MSA64`，通常用于抽象共享字段、谓词或编码结构。

### Lines 499-501
```tablegen
class ASE_MT {
  list <Predicate> ASEPredicate = [HasMT];
}
```
- EN: Declares reusable TableGen class `ASE_MT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ASE_MT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 503-505
```tablegen
class ASE_CRC {
  list <Predicate> ASEPredicate = [HasCRC];
}
```
- EN: Declares reusable TableGen class `ASE_CRC` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ASE_CRC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 507-509
```tablegen
class ASE_VIRT {
  list <Predicate> ASEPredicate = [HasVirt];
}
```
- EN: Declares reusable TableGen class `ASE_VIRT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ASE_VIRT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 511-513
```tablegen
class ASE_GINV {
  list <Predicate> ASEPredicate = [HasGINV];
}
```
- EN: Declares reusable TableGen class `ASE_GINV` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ASE_GINV`，通常用于抽象共享字段、谓词或编码结构。

### Lines 515-520
```tablegen
// Class used for separating microMIPSr6 and microMIPS (r3) instruction.
// It can be used only on instructions that doesn't inherit PredicateControl.
class ISA_MICROMIPS_NOT_32R6 : PredicateControl {
  let InsnPredicates = [NotMips32r6];
  let EncodingPredicates = [InMicroMips];
}
```
- EN: Declares reusable TableGen class `ISA_MICROMIPS_NOT_32R6` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ISA_MICROMIPS_NOT_32R6`，通常用于抽象共享字段、谓词或编码结构。

### Lines 522-524
```tablegen
class ASE_NOT_DSP {
  list<Predicate> ASEPredicate = [NotDSP];
}
```
- EN: Declares reusable TableGen class `ASE_NOT_DSP` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ASE_NOT_DSP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 526-528
```tablegen
class MADD4 {
  list<Predicate> AdditionalPredicates = [HasMadd4];
}
```
- EN: Declares reusable TableGen class `MADD4` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `MADD4`，通常用于抽象共享字段、谓词或编码结构。

### Lines 530-534
```tablegen
// Classes used for separating expansions that differ based on the ABI in
// use.
class ABI_N64 {
  list<Predicate> AdditionalPredicates = [IsN64];
}
```
- EN: Declares reusable TableGen class `ABI_N64` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ABI_N64`，通常用于抽象共享字段、谓词或编码结构。

### Lines 536-538
```tablegen
class ABI_NOT_N64 {
  list<Predicate> AdditionalPredicates = [IsNotN64];
}
```
- EN: Declares reusable TableGen class `ABI_NOT_N64` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ABI_NOT_N64`，通常用于抽象共享字段、谓词或编码结构。

### Lines 540-542
```tablegen
class FPOP_FUSION_FAST {
  list <Predicate> AdditionalPredicates = [AllowFPOpFusion];
}
```
- EN: Declares reusable TableGen class `FPOP_FUSION_FAST` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `FPOP_FUSION_FAST`，通常用于抽象共享字段、谓词或编码结构。

### Lines 544-544
```tablegen
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 546-546
```tablegen
class MipsPat<dag pattern, dag result> : Pat<pattern, result>, PredicateControl;
```
- EN: Declares reusable TableGen class `MipsPat` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `MipsPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 548-549
```tablegen
class MipsInstAlias<string Asm, dag Result, bit Emit = 0b1> :
  InstAlias<Asm, Result, Emit>, PredicateControl;
```
- EN: Declares reusable TableGen class `MipsInstAlias` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `MipsInstAlias`，通常用于抽象共享字段、谓词或编码结构。

### Lines 551-553
```tablegen
class IsCommutable {
  bit isCommutable = 1;
}
```
- EN: Declares reusable TableGen class `IsCommutable` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `IsCommutable`，通常用于抽象共享字段、谓词或编码结构。

### Lines 555-558
```tablegen
class IsBranch {
  bit isBranch = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `IsBranch` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `IsBranch`，通常用于抽象共享字段、谓词或编码结构。

### Lines 560-563
```tablegen
class IsReturn {
  bit isReturn = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `IsReturn` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `IsReturn`，通常用于抽象共享字段、谓词或编码结构。

### Lines 565-568
```tablegen
class IsCall {
  bit isCall = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `IsCall` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `IsCall`，通常用于抽象共享字段、谓词或编码结构。

### Lines 570-578
```tablegen
class IsTailCall {
  bit isCall = 1;
  bit isTerminator = 1;
  bit isReturn = 1;
  bit isBarrier = 1;
  bit hasExtraSrcRegAllocReq = 1;
  bit isCodeGenOnly = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `IsTailCall` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `IsTailCall`，通常用于抽象共享字段、谓词或编码结构。

### Lines 580-582
```tablegen
class IsAsCheapAsAMove {
  bit isAsCheapAsAMove = 1;
}
```
- EN: Declares reusable TableGen class `IsAsCheapAsAMove` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `IsAsCheapAsAMove`，通常用于抽象共享字段、谓词或编码结构。

### Lines 584-586
```tablegen
class NeverHasSideEffects {
  bit hasSideEffects = 0;
}
```
- EN: Declares reusable TableGen class `NeverHasSideEffects` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `NeverHasSideEffects`，通常用于抽象共享字段、谓词或编码结构。

### Lines 588-590
```tablegen
//===----------------------------------------------------------------------===//
// Instruction format superclass
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 592-592
```tablegen
include "MipsInstrFormats.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 594-596
```tablegen
//===----------------------------------------------------------------------===//
// Mips Operand, Complex Patterns and Transformations Definitions.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 598-605
```tablegen
class ConstantSImmAsmOperandClass<int Bits, list<AsmOperandClass> Supers = [],
                                  int Offset = 0> : AsmOperandClass {
  let Name = "ConstantSImm" # Bits # "_" # Offset;
  let RenderMethod = "addConstantSImmOperands<" # Bits # ", " # Offset # ">";
  let PredicateMethod = "isConstantSImm<" # Bits # ", " # Offset # ">";
  let SuperClasses = Supers;
  let DiagnosticType = "SImm" # Bits # "_" # Offset;
}
```
- EN: Declares reusable TableGen class `ConstantSImmAsmOperandClass` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ConstantSImmAsmOperandClass`，通常用于抽象共享字段、谓词或编码结构。

### Lines 607-614
```tablegen
class SimmLslAsmOperandClass<int Bits, list<AsmOperandClass> Supers = [],
                                  int Shift = 0> : AsmOperandClass {
  let Name = "Simm" # Bits # "_Lsl" # Shift;
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isScaledSImm<" # Bits # ", " # Shift # ">";
  let SuperClasses = Supers;
  let DiagnosticType = "SImm" # Bits # "_Lsl" # Shift;
}
```
- EN: Declares reusable TableGen class `SimmLslAsmOperandClass` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SimmLslAsmOperandClass`，通常用于抽象共享字段、谓词或编码结构。

### Lines 616-623
```tablegen
class ConstantUImmAsmOperandClass<int Bits, list<AsmOperandClass> Supers = [],
                                  int Offset = 0> : AsmOperandClass {
  let Name = "ConstantUImm" # Bits # "_" # Offset;
  let RenderMethod = "addConstantUImmOperands<" # Bits # ", " # Offset # ">";
  let PredicateMethod = "isConstantUImm<" # Bits # ", " # Offset # ">";
  let SuperClasses = Supers;
  let DiagnosticType = "UImm" # Bits # "_" # Offset;
}
```
- EN: Declares reusable TableGen class `ConstantUImmAsmOperandClass` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ConstantUImmAsmOperandClass`，通常用于抽象共享字段、谓词或编码结构。

### Lines 625-633
```tablegen
class ConstantUImmRangeAsmOperandClass<int Bottom, int Top,
                                       list<AsmOperandClass> Supers = []>
    : AsmOperandClass {
  let Name = "ConstantUImmRange" # Bottom # "_" # Top;
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isConstantUImmRange<" # Bottom # ", " # Top # ">";
  let SuperClasses = Supers;
  let DiagnosticType = "UImmRange" # Bottom # "_" # Top;
}
```
- EN: Declares reusable TableGen class `ConstantUImmRangeAsmOperandClass` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ConstantUImmRangeAsmOperandClass`，通常用于抽象共享字段、谓词或编码结构。

### Lines 635-642
```tablegen
class SImmAsmOperandClass<int Bits, list<AsmOperandClass> Supers = []>
    : AsmOperandClass {
  let Name = "SImm" # Bits;
  let RenderMethod = "addSImmOperands<" # Bits # ">";
  let PredicateMethod = "isSImm<" # Bits # ">";
  let SuperClasses = Supers;
  let DiagnosticType = "SImm" # Bits;
}
```
- EN: Declares reusable TableGen class `SImmAsmOperandClass` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SImmAsmOperandClass`，通常用于抽象共享字段、谓词或编码结构。

### Lines 644-651
```tablegen
class UImmAsmOperandClass<int Bits, list<AsmOperandClass> Supers = []>
    : AsmOperandClass {
  let Name = "UImm" # Bits;
  let RenderMethod = "addUImmOperands<" # Bits # ">";
  let PredicateMethod = "isUImm<" # Bits # ">";
  let SuperClasses = Supers;
  let DiagnosticType = "UImm" # Bits;
}
```
- EN: Declares reusable TableGen class `UImmAsmOperandClass` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `UImmAsmOperandClass`，通常用于抽象共享字段、谓词或编码结构。

### Lines 653-661
```tablegen
// Generic case - only to support certain assembly pseudo instructions.
class UImmAnyAsmOperandClass<int Bits, list<AsmOperandClass> Supers = []>
    : AsmOperandClass {
  let Name = "ImmAny";
  let RenderMethod = "addConstantUImmOperands<32>";
  let PredicateMethod = "isSImm<" # Bits # ">";
  let SuperClasses = Supers;
  let DiagnosticType = "ImmAny";
}
```
- EN: Declares reusable TableGen class `UImmAnyAsmOperandClass` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `UImmAnyAsmOperandClass`，通常用于抽象共享字段、谓词或编码结构。

### Lines 663-680
```tablegen
// AsmOperandClasses require a strict ordering which is difficult to manage
// as a hierarchy. Instead, we use a linear ordering and impose an order that
// is in some places arbitrary.
//
// Here the rules that are in use:
// * Wider immediates are a superset of narrower immediates:
//     uimm4 < uimm5 < uimm6
// * For the same bit-width, unsigned immediates are a superset of signed
//   immediates::
//     simm4 < uimm4 < simm5 < uimm5
// * For the same upper-bound, signed immediates are a superset of unsigned
//   immediates:
//     uimm3 < simm4 < uimm4 < simm4
// * Modified immediates are a superset of ordinary immediates:
//     uimm5 < uimm5_plus1 (1..32) < uimm5_plus32 (32..63) < uimm6
//   The term 'superset' starts to break down here since the uimm5_plus* classes
//   are not true supersets of uimm5 (but they are still subsets of uimm6).
// * 'Relaxed' immediates are supersets of the corresponding unsigned immediate.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 681-685
```tablegen
//     uimm16 < uimm16_relaxed
// * The codeGen pattern type is arbitrarily ordered.
//     uimm5 < uimm5_64, and uimm5 < vsplat_uimm5
//   This is entirely arbitrary. We need an ordering and what we pick is
//   unimportant since only one is possible for a given mnemonic.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 687-704
```tablegen
def UImm32CoercedAsmOperandClass : UImmAnyAsmOperandClass<33, []> {
  let Name = "UImm32_Coerced";
  let DiagnosticType = "UImm32_Coerced";
}
def SImm32RelaxedAsmOperandClass
    : SImmAsmOperandClass<32, [UImm32CoercedAsmOperandClass]> {
  let Name = "SImm32_Relaxed";
  let PredicateMethod = "isAnyImm<33>";
  let DiagnosticType = "SImm32_Relaxed";
}
def SImm32AsmOperandClass
    : SImmAsmOperandClass<32, [SImm32RelaxedAsmOperandClass]>;
def ConstantUImm26AsmOperandClass
    : ConstantUImmAsmOperandClass<26, [SImm32AsmOperandClass]>;
def ConstantUImm20AsmOperandClass
    : ConstantUImmAsmOperandClass<20, [ConstantUImm26AsmOperandClass]>;
def ConstantSImm19Lsl2AsmOperandClass : AsmOperandClass {
  let Name = "SImm19Lsl2";
```
- EN: Defines TableGen record `UImm32CoercedAsmOperandClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UImm32CoercedAsmOperandClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 705-722
```tablegen
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isScaledSImm<19, 2>";
  let SuperClasses = [ConstantUImm20AsmOperandClass];
  let DiagnosticType = "SImm19_Lsl2";
}
def UImm16RelaxedAsmOperandClass
    : UImmAsmOperandClass<16, [ConstantUImm20AsmOperandClass]> {
  let Name = "UImm16_Relaxed";
  let PredicateMethod = "isAnyImm<16>";
  let DiagnosticType = "UImm16_Relaxed";
}
// Similar to the relaxed classes which take an SImm and render it as
// an UImm, this takes a UImm and renders it as an SImm.
def UImm16AltRelaxedAsmOperandClass
    : SImmAsmOperandClass<16, [UImm16RelaxedAsmOperandClass]> {
  let Name = "UImm16_AltRelaxed";
  let PredicateMethod = "isUImm<16>";
  let DiagnosticType = "UImm16_AltRelaxed";
```
- EN: Defines TableGen record `UImm16RelaxedAsmOperandClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UImm16RelaxedAsmOperandClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 723-740
```tablegen
}
// FIXME: One of these should probably have UImm16AsmOperandClass as the
//        superclass instead of UImm16RelaxedasmOPerandClass.
def UImm16AsmOperandClass
    : UImmAsmOperandClass<16, [UImm16RelaxedAsmOperandClass]>;
def SImm16RelaxedAsmOperandClass
    : SImmAsmOperandClass<16, [UImm16RelaxedAsmOperandClass]> {
  let Name = "SImm16_Relaxed";
  let PredicateMethod = "isAnyImm<16>";
  let DiagnosticType = "SImm16_Relaxed";
}
def SImm16AsmOperandClass
    : SImmAsmOperandClass<16, [SImm16RelaxedAsmOperandClass]>;
def ConstantSImm10Lsl3AsmOperandClass : AsmOperandClass {
  let Name = "SImm10Lsl3";
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isScaledSImm<10, 3>";
  let SuperClasses = [SImm16AsmOperandClass];
```
- EN: Defines TableGen record `UImm16AsmOperandClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UImm16AsmOperandClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 741-758
```tablegen
  let DiagnosticType = "SImm10_Lsl3";
}
def ConstantSImm10Lsl2AsmOperandClass : AsmOperandClass {
  let Name = "SImm10Lsl2";
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isScaledSImm<10, 2>";
  let SuperClasses = [ConstantSImm10Lsl3AsmOperandClass];
  let DiagnosticType = "SImm10_Lsl2";
}
def ConstantSImm11AsmOperandClass
    : ConstantSImmAsmOperandClass<11, [ConstantSImm10Lsl2AsmOperandClass]>;
def ConstantSImm10Lsl1AsmOperandClass : AsmOperandClass {
  let Name = "SImm10Lsl1";
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isScaledSImm<10, 1>";
  let SuperClasses = [ConstantSImm11AsmOperandClass];
  let DiagnosticType = "SImm10_Lsl1";
}
```
- EN: Defines TableGen record `ConstantSImm10Lsl2AsmOperandClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ConstantSImm10Lsl2AsmOperandClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 759-776
```tablegen
def ConstantUImm10AsmOperandClass
    : ConstantUImmAsmOperandClass<10, [ConstantSImm10Lsl1AsmOperandClass]>;
def ConstantSImm10AsmOperandClass
    : ConstantSImmAsmOperandClass<10, [ConstantUImm10AsmOperandClass]>;
def ConstantSImm9AsmOperandClass
    : ConstantSImmAsmOperandClass<9, [ConstantSImm10AsmOperandClass]>;
def ConstantSImm7Lsl2AsmOperandClass : AsmOperandClass {
  let Name = "SImm7Lsl2";
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isScaledSImm<7, 2>";
  let SuperClasses = [ConstantSImm9AsmOperandClass];
  let DiagnosticType = "SImm7_Lsl2";
}
def ConstantUImm8AsmOperandClass
    : ConstantUImmAsmOperandClass<8, [ConstantSImm7Lsl2AsmOperandClass]>;
def ConstantUImm7Sub1AsmOperandClass
    : ConstantUImmAsmOperandClass<7, [ConstantUImm8AsmOperandClass], -1> {
  // Specify the names since the -1 offset causes invalid identifiers otherwise.
```
- EN: Defines TableGen record `ConstantUImm10AsmOperandClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ConstantUImm10AsmOperandClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 777-794
```tablegen
  let Name = "UImm7_N1";
  let DiagnosticType = "UImm7_N1";
}
def ConstantUImm7AsmOperandClass
    : ConstantUImmAsmOperandClass<7, [ConstantUImm7Sub1AsmOperandClass]>;
def ConstantUImm6Lsl2AsmOperandClass : AsmOperandClass {
  let Name = "UImm6Lsl2";
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isScaledUImm<6, 2>";
  let SuperClasses = [ConstantUImm7AsmOperandClass];
  let DiagnosticType = "UImm6_Lsl2";
}
def ConstantUImm6AsmOperandClass
    : ConstantUImmAsmOperandClass<6, [ConstantUImm6Lsl2AsmOperandClass]>;
def ConstantSImm6AsmOperandClass
    : ConstantSImmAsmOperandClass<6, [ConstantUImm6AsmOperandClass]>;
def ConstantUImm5Lsl2AsmOperandClass : AsmOperandClass {
  let Name = "UImm5Lsl2";
```
- EN: Defines TableGen record `ConstantUImm7AsmOperandClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ConstantUImm7AsmOperandClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 795-812
```tablegen
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isScaledUImm<5, 2>";
  let SuperClasses = [ConstantSImm6AsmOperandClass];
  let DiagnosticType = "UImm5_Lsl2";
}
def ConstantUImm5_Range2_64AsmOperandClass
    : ConstantUImmRangeAsmOperandClass<2, 64, [ConstantUImm5Lsl2AsmOperandClass]>;
def ConstantUImm5Plus33AsmOperandClass
    : ConstantUImmAsmOperandClass<5, [ConstantUImm5_Range2_64AsmOperandClass],
                                  33>;
def ConstantUImm5ReportUImm6AsmOperandClass
    : ConstantUImmAsmOperandClass<5, [ConstantUImm5Plus33AsmOperandClass]> {
  let Name = "ConstantUImm5_0_Report_UImm6";
  let DiagnosticType = "UImm5_0_Report_UImm6";
}
def ConstantUImm5Plus32AsmOperandClass
    : ConstantUImmAsmOperandClass<
          5, [ConstantUImm5ReportUImm6AsmOperandClass], 32>;
```
- EN: Defines TableGen record `ConstantUImm5_Range2_64AsmOperandClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ConstantUImm5_Range2_64AsmOperandClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 813-830
```tablegen
def ConstantUImm5Plus32NormalizeAsmOperandClass
    : ConstantUImmAsmOperandClass<5, [ConstantUImm5Plus32AsmOperandClass], 32> {
  let Name = "ConstantUImm5_32_Norm";
  // We must also subtract 32 when we render the operand.
  let RenderMethod = "addConstantUImmOperands<5, 32, -32>";
}
def ConstantUImm5Plus1ReportUImm6AsmOperandClass
    : ConstantUImmAsmOperandClass<
          5, [ConstantUImm5Plus32NormalizeAsmOperandClass], 1>{
  let Name = "ConstantUImm5_Plus1_Report_UImm6";
}
def ConstantUImm5Plus1AsmOperandClass
    : ConstantUImmAsmOperandClass<
          5, [ConstantUImm5Plus1ReportUImm6AsmOperandClass], 1>;
def ConstantUImm5AsmOperandClass
    : ConstantUImmAsmOperandClass<5, [ConstantUImm5Plus1AsmOperandClass]>;
def ConstantSImm5AsmOperandClass
    : ConstantSImmAsmOperandClass<5, [ConstantUImm5AsmOperandClass]>;
```
- EN: Defines TableGen record `ConstantUImm5Plus32NormalizeAsmOperandClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ConstantUImm5Plus32NormalizeAsmOperandClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 831-848
```tablegen
def ConstantUImm4AsmOperandClass
    : ConstantUImmAsmOperandClass<4, [ConstantSImm5AsmOperandClass]>;
def ConstantSImm4AsmOperandClass
    : ConstantSImmAsmOperandClass<4, [ConstantUImm4AsmOperandClass]>;
def ConstantUImm3AsmOperandClass
    : ConstantUImmAsmOperandClass<3, [ConstantSImm4AsmOperandClass]>;
def ConstantUImm2Plus1AsmOperandClass
    : ConstantUImmAsmOperandClass<2, [ConstantUImm3AsmOperandClass], 1>;
def ConstantUImm2AsmOperandClass
    : ConstantUImmAsmOperandClass<2, [ConstantUImm3AsmOperandClass]>;
def ConstantUImm1AsmOperandClass
    : ConstantUImmAsmOperandClass<1, [ConstantUImm2AsmOperandClass]>;
def ConstantImmzAsmOperandClass : AsmOperandClass {
  let Name = "ConstantImmz";
  let RenderMethod = "addConstantUImmOperands<1>";
  let PredicateMethod = "isConstantImmz";
  let SuperClasses = [ConstantUImm1AsmOperandClass];
  let DiagnosticType = "Immz";
```
- EN: Defines TableGen record `ConstantUImm4AsmOperandClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ConstantUImm4AsmOperandClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 849-849
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 851-852
```tablegen
def Simm19Lsl2AsmOperand
    : SimmLslAsmOperandClass<19, [], 2>;
```
- EN: Defines TableGen record `Simm19Lsl2AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Simm19Lsl2AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 854-859
```tablegen
def MipsJumpTargetAsmOperand : AsmOperandClass {
  let Name = "JumpTarget";
  let ParserMethod = "parseJumpTarget";
  let PredicateMethod = "isImm";
  let RenderMethod = "addImmOperands";
}
```
- EN: Defines TableGen record `MipsJumpTargetAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsJumpTargetAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 861-878
```tablegen
// Instruction operand types
def jmptarget   : Operand<OtherVT> {
  let EncoderMethod = "getJumpTargetOpValue";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
  let PrintMethod = "printJumpOperand";
}
def brtarget    : Operand<OtherVT> {
  let EncoderMethod = "getBranchTargetOpValue";
  let OperandType = "OPERAND_PCREL";
  let DecoderMethod = "DecodeBranchTarget";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
  let PrintMethod = "printBranchOperand";
}
def brtarget1SImm16 : Operand<OtherVT> {
  let EncoderMethod = "getBranchTargetOpValue1SImm16";
  let OperandType = "OPERAND_PCREL";
  let DecoderMethod = "DecodeBranchTarget1SImm16";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
```
- EN: Defines TableGen record `jmptarget` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `jmptarget`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 879-885
```tablegen
  let PrintMethod = "printBranchOperand";
}
def calltarget  : Operand<iPTR> {
  let EncoderMethod = "getJumpTargetOpValue";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
  let PrintMethod = "printJumpOperand";
}
```
- EN: Defines TableGen record `calltarget` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `calltarget`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 887-887
```tablegen
def imm64: Operand<i64>;
```
- EN: Defines TableGen record `imm64:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm64:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 889-893
```tablegen
def ConstantImmAsmOperandClass : AsmOperandClass {
  let Name = "ConstantImm";
  let PredicateMethod = "isConstantImm";
  let RenderMethod = "addImmOperands";
}
```
- EN: Defines TableGen record `ConstantImmAsmOperandClass` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ConstantImmAsmOperandClass`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 895-897
```tablegen
def ConstantImm64: Operand<i64> {
  let ParserMatchClass = ConstantImmAsmOperandClass;
}
```
- EN: Defines TableGen record `ConstantImm64:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ConstantImm64:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 899-903
```tablegen
def simm19_lsl2 : Operand<i32> {
  let EncoderMethod = "getSimm19Lsl2Encoding";
  let DecoderMethod = "DecodeSimm19Lsl2";
  let ParserMatchClass = Simm19Lsl2AsmOperand;
}
```
- EN: Defines TableGen record `simm19_lsl2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm19_lsl2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 905-909
```tablegen
def simm18_lsl3 : Operand<i32> {
  let EncoderMethod = "getSimm18Lsl3Encoding";
  let DecoderMethod = "DecodeSimm18Lsl3";
  let ParserMatchClass = MipsJumpTargetAsmOperand;
}
```
- EN: Defines TableGen record `simm18_lsl3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm18_lsl3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 911-915
```tablegen
// Zero
def uimmz       : Operand<i32> {
  let PrintMethod = "printUImm<0>";
  let ParserMatchClass = ConstantImmzAsmOperandClass;
}
```
- EN: Defines TableGen record `uimmz` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimmz`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 917-923
```tablegen
// size operand of ins instruction
def uimm_range_2_64 : Operand<i32> {
  let PrintMethod = "printUImm<6, 2>";
  let EncoderMethod = "getSizeInsEncoding";
  let DecoderMethod = "DecodeInsSize";
  let ParserMatchClass = ConstantUImm5_Range2_64AsmOperandClass;
}
```
- EN: Defines TableGen record `uimm_range_2_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm_range_2_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 925-931
```tablegen
// Unsigned Operands
foreach I = {1, 2, 3, 4, 5, 6, 7, 8, 10, 20, 26} in
  def uimm # I : Operand<i32> {
    let PrintMethod = "printUImm<" # I # ">";
    let ParserMatchClass =
        !cast<AsmOperandClass>("ConstantUImm" # I # "AsmOperandClass");
  }
```
- EN: Defines TableGen record `uimm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 933-938
```tablegen
def uimm2_plus1 : Operand<i32> {
  let PrintMethod = "printUImm<2, 1>";
  let EncoderMethod = "getUImmWithOffsetEncoding<2, 1>";
  let DecoderMethod = "DecodeUImmWithOffset<2, 1>";
  let ParserMatchClass = ConstantUImm2Plus1AsmOperandClass;
}
```
- EN: Defines TableGen record `uimm2_plus1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm2_plus1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 940-945
```tablegen
def uimm5_plus1 : Operand<i32> {
  let PrintMethod = "printUImm<5, 1>";
  let EncoderMethod = "getUImmWithOffsetEncoding<5, 1>";
  let DecoderMethod = "DecodeUImmWithOffset<5, 1>";
  let ParserMatchClass = ConstantUImm5Plus1AsmOperandClass;
}
```
- EN: Defines TableGen record `uimm5_plus1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm5_plus1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 947-952
```tablegen
def uimm5_plus1_report_uimm6 : Operand<i32> {
  let PrintMethod = "printUImm<6, 1>";
  let EncoderMethod = "getUImmWithOffsetEncoding<5, 1>";
  let DecoderMethod = "DecodeUImmWithOffset<5, 1>";
  let ParserMatchClass = ConstantUImm5Plus1ReportUImm6AsmOperandClass;
}
```
- EN: Defines TableGen record `uimm5_plus1_report_uimm6` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm5_plus1_report_uimm6`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 954-957
```tablegen
def uimm5_plus32 : Operand<i32> {
  let PrintMethod = "printUImm<5, 32>";
  let ParserMatchClass = ConstantUImm5Plus32AsmOperandClass;
}
```
- EN: Defines TableGen record `uimm5_plus32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm5_plus32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 959-964
```tablegen
def uimm5_plus33 : Operand<i32> {
  let PrintMethod = "printUImm<5, 33>";
  let EncoderMethod = "getUImmWithOffsetEncoding<5, 1>";
  let DecoderMethod = "DecodeUImmWithOffset<5, 1>";
  let ParserMatchClass = ConstantUImm5Plus33AsmOperandClass;
}
```
- EN: Defines TableGen record `uimm5_plus33` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm5_plus33`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 966-971
```tablegen
def uimm5_inssize_plus1 : Operand<i32> {
  let PrintMethod = "printUImm<6>";
  let ParserMatchClass = ConstantUImm5Plus1AsmOperandClass;
  let EncoderMethod = "getSizeInsEncoding";
  let DecoderMethod = "DecodeInsSize";
}
```
- EN: Defines TableGen record `uimm5_inssize_plus1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm5_inssize_plus1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 973-976
```tablegen
def uimm5_plus32_normalize : Operand<i32> {
  let PrintMethod = "printUImm<5>";
  let ParserMatchClass = ConstantUImm5Plus32NormalizeAsmOperandClass;
}
```
- EN: Defines TableGen record `uimm5_plus32_normalize` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm5_plus32_normalize`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 978-982
```tablegen
def uimm5_lsl2 : Operand<OtherVT> {
  let EncoderMethod = "getUImm5Lsl2Encoding";
  let DecoderMethod = "DecodeUImmWithOffsetAndScale<5, 0, 4>";
  let ParserMatchClass = ConstantUImm5Lsl2AsmOperandClass;
}
```
- EN: Defines TableGen record `uimm5_lsl2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm5_lsl2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 984-987
```tablegen
def uimm5_plus32_normalize_64 : Operand<i64> {
  let PrintMethod = "printUImm<5>";
  let ParserMatchClass = ConstantUImm5Plus32NormalizeAsmOperandClass;
}
```
- EN: Defines TableGen record `uimm5_plus32_normalize_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm5_plus32_normalize_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 989-993
```tablegen
def uimm6_lsl2 : Operand<OtherVT> {
  let EncoderMethod = "getUImm6Lsl2Encoding";
  let DecoderMethod = "DecodeUImmWithOffsetAndScale<6, 0, 4>";
  let ParserMatchClass = ConstantUImm6Lsl2AsmOperandClass;
}
```
- EN: Defines TableGen record `uimm6_lsl2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm6_lsl2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 995-1000
```tablegen
foreach I = {16} in
  def uimm # I : Operand<i32> {
    let PrintMethod = "printUImm<" # I # ">";
    let ParserMatchClass =
        !cast<AsmOperandClass>("UImm" # I # "AsmOperandClass");
  }
```
- EN: Defines TableGen record `uimm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1002-1006
```tablegen
// Like uimm16_64 but coerces simm16 to uimm16.
def uimm16_relaxed : Operand<i32> {
  let PrintMethod = "printUImm<16>";
  let ParserMatchClass = UImm16RelaxedAsmOperandClass;
}
```
- EN: Defines TableGen record `uimm16_relaxed` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm16_relaxed`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1008-1013
```tablegen
foreach I = {5} in
  def uimm # I # _64 : Operand<i64> {
    let PrintMethod = "printUImm<" # I # ">";
    let ParserMatchClass =
        !cast<AsmOperandClass>("ConstantUImm" # I # "AsmOperandClass");
  }
```
- EN: Defines TableGen record `uimm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1015-1020
```tablegen
foreach I = {16} in
  def uimm # I # _64 : Operand<i64> {
    let PrintMethod = "printUImm<" # I # ">";
    let ParserMatchClass =
        !cast<AsmOperandClass>("UImm" # I # "AsmOperandClass");
  }
```
- EN: Defines TableGen record `uimm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1022-1026
```tablegen
// Like uimm16_64 but coerces simm16 to uimm16.
def uimm16_64_relaxed : Operand<i64> {
  let PrintMethod = "printUImm<16>";
  let ParserMatchClass = UImm16RelaxedAsmOperandClass;
}
```
- EN: Defines TableGen record `uimm16_64_relaxed` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm16_64_relaxed`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1028-1037
```tablegen
def uimm16_altrelaxed : Operand<i32> {
  let PrintMethod = "printUImm<16>";
  let ParserMatchClass = UImm16AltRelaxedAsmOperandClass;
}
// Like uimm5 but reports a less confusing error for 32-63 when
// an instruction alias permits that.
def uimm5_report_uimm6 : Operand<i32> {
  let PrintMethod = "printUImm<6>";
  let ParserMatchClass = ConstantUImm5ReportUImm6AsmOperandClass;
}
```
- EN: Defines TableGen record `uimm16_altrelaxed` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm16_altrelaxed`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1039-1044
```tablegen
// Like uimm5_64 but reports a less confusing error for 32-63 when
// an instruction alias permits that.
def uimm5_64_report_uimm6 : Operand<i64> {
  let PrintMethod = "printUImm<5>";
  let ParserMatchClass = ConstantUImm5ReportUImm6AsmOperandClass;
}
```
- EN: Defines TableGen record `uimm5_64_report_uimm6` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm5_64_report_uimm6`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1046-1051
```tablegen
foreach I = {1, 2, 3, 4} in
  def uimm # I # _ptr : Operand<iPTR> {
    let PrintMethod = "printUImm<" # I # ">";
    let ParserMatchClass =
        !cast<AsmOperandClass>("ConstantUImm" # I # "AsmOperandClass");
  }
```
- EN: Defines TableGen record `uimm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1053-1058
```tablegen
foreach I = {1, 2, 3, 4, 5, 6, 8} in
  def vsplat_uimm # I : Operand<vAny> {
    let PrintMethod = "printUImm<" # I # ">";
    let ParserMatchClass =
        !cast<AsmOperandClass>("ConstantUImm" # I # "AsmOperandClass");
  }
```
- EN: Defines TableGen record `vsplat_uimm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `vsplat_uimm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1060-1066
```tablegen
// Signed operands
foreach I = {4, 5, 6, 9, 10, 11} in
  def simm # I : Operand<i32> {
    let DecoderMethod = "DecodeSImmWithOffsetAndScale<" # I # ">";
    let ParserMatchClass =
        !cast<AsmOperandClass>("ConstantSImm" # I # "AsmOperandClass");
  }
```
- EN: Defines TableGen record `simm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1068-1073
```tablegen
foreach I = {1, 2, 3} in
  def simm10_lsl # I : Operand<i32> {
    let DecoderMethod = "DecodeSImmWithOffsetAndScale<10, " # I # ">";
    let ParserMatchClass =
        !cast<AsmOperandClass>("ConstantSImm10Lsl" # I # "AsmOperandClass");
  }
```
- EN: Defines TableGen record `simm10_lsl` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm10_lsl`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1075-1080
```tablegen
foreach I = {10} in
  def simm # I # _64 : Operand<i64> {
    let DecoderMethod = "DecodeSImmWithOffsetAndScale<" # I # ">";
    let ParserMatchClass =
        !cast<AsmOperandClass>("ConstantSImm" # I # "AsmOperandClass");
  }
```
- EN: Defines TableGen record `simm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1082-1086
```tablegen
foreach I = {5, 10} in
  def vsplat_simm # I : Operand<vAny> {
    let ParserMatchClass =
        !cast<AsmOperandClass>("ConstantSImm" # I # "AsmOperandClass");
  }
```
- EN: Defines TableGen record `vsplat_simm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `vsplat_simm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1088-1092
```tablegen
def simm7_lsl2 : Operand<OtherVT> {
  let EncoderMethod = "getSImm7Lsl2Encoding";
  let DecoderMethod = "DecodeSImmWithOffsetAndScale<" # I # ", 0, 4>";
  let ParserMatchClass = ConstantSImm7Lsl2AsmOperandClass;
}
```
- EN: Defines TableGen record `simm7_lsl2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm7_lsl2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1094-1098
```tablegen
foreach I = {16, 32} in
  def simm # I : Operand<i32> {
    let DecoderMethod = "DecodeSImmWithOffsetAndScale<" # I # ">";
    let ParserMatchClass = !cast<AsmOperandClass>("SImm" # I # "AsmOperandClass");
  }
```
- EN: Defines TableGen record `simm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1100-1104
```tablegen
// Like simm16 but coerces uimm16 to simm16.
def simm16_relaxed : Operand<i32> {
  let DecoderMethod = "DecodeSImmWithOffsetAndScale<16>";
  let ParserMatchClass = SImm16RelaxedAsmOperandClass;
}
```
- EN: Defines TableGen record `simm16_relaxed` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm16_relaxed`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1106-1109
```tablegen
def simm16_64 : Operand<i64> {
  let DecoderMethod = "DecodeSImmWithOffsetAndScale<16>";
  let ParserMatchClass = SImm16AsmOperandClass;
}
```
- EN: Defines TableGen record `simm16_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm16_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1111-1119
```tablegen
// like simm32 but coerces simm32 to uimm32.
def uimm32_coerced : Operand<i32> {
  let ParserMatchClass = UImm32CoercedAsmOperandClass;
}
// Like simm32 but coerces uimm32 to simm32.
def simm32_relaxed : Operand<i32> {
  let DecoderMethod = "DecodeSImmWithOffsetAndScale<32>";
  let ParserMatchClass = SImm32RelaxedAsmOperandClass;
}
```
- EN: Defines TableGen record `uimm32_coerced` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `uimm32_coerced`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1121-1125
```tablegen
// This is almost the same as a uimm7 but 0x7f is interpreted as -1.
def li16_imm : Operand<i32> {
  let DecoderMethod = "DecodeLi16Imm";
  let ParserMatchClass = ConstantUImm7Sub1AsmOperandClass;
}
```
- EN: Defines TableGen record `li16_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `li16_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1127-1130
```tablegen
def MipsMemAsmOperand : AsmOperandClass {
  let Name = "Mem";
  let ParserMethod = "parseMemOperand";
}
```
- EN: Defines TableGen record `MipsMemAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsMemAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1132-1140
```tablegen
class MipsMemSimmAsmOperand<int Width, int Shift = 0> : AsmOperandClass {
  let Name = "MemOffsetSimm" # Width # "_" # Shift;
  let SuperClasses = [MipsMemAsmOperand];
  let RenderMethod = "addMemOperands";
  let ParserMethod = "parseMemOperand";
  let PredicateMethod = "isMemWithSimmOffset<" # Width # ", " # Shift # ">";
  let DiagnosticType = !if(!eq(Shift, 0), "MemSImm" # Width,
                                          "MemSImm" # Width # "Lsl" # Shift);
}
```
- EN: Declares reusable TableGen class `MipsMemSimmAsmOperand` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `MipsMemSimmAsmOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1142-1149
```tablegen
def MipsMemSimmPtrAsmOperand : AsmOperandClass {
  let Name = "MemOffsetSimmPtr";
  let SuperClasses = [MipsMemAsmOperand];
  let RenderMethod = "addMemOperands";
  let ParserMethod = "parseMemOperand";
  let PredicateMethod = "isMemWithPtrSizeOffset";
  let DiagnosticType = "MemSImmPtr";
}
```
- EN: Defines TableGen record `MipsMemSimmPtrAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsMemSimmPtrAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1151-1155
```tablegen
def MipsInvertedImmoperand : AsmOperandClass {
  let Name = "InvNum";
  let RenderMethod = "addImmOperands";
  let ParserMethod = "parseInvNum";
}
```
- EN: Defines TableGen record `MipsInvertedImmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MipsInvertedImmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1157-1159
```tablegen
def InvertedImOperand : Operand<i32> {
  let ParserMatchClass = MipsInvertedImmoperand;
}
```
- EN: Defines TableGen record `InvertedImOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `InvertedImOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1161-1163
```tablegen
def InvertedImOperand64 : Operand<i64> {
  let ParserMatchClass = MipsInvertedImmoperand;
}
```
- EN: Defines TableGen record `InvertedImOperand64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `InvertedImOperand64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1165-1171
```tablegen
class mem_generic : Operand<iPTR> {
  let PrintMethod = "printMemOperand";
  let MIOperandInfo = (ops mips_ptr_rc, simm16);
  let EncoderMethod = "getMemEncoding";
  let ParserMatchClass = MipsMemAsmOperand;
  let OperandType = "OPERAND_MEMORY";
}
```
- EN: Declares reusable TableGen class `mem_generic` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `mem_generic`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1173-1174
```tablegen
// Address operand
def mem : mem_generic;
```
- EN: Defines TableGen record `mem` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1176-1180
```tablegen
// MSA specific address operand
def mem_msa : mem_generic {
  let MIOperandInfo = (ops mips_ptr_rc, simm10);
  let EncoderMethod = "getMSAMemEncoding";
}
```
- EN: Defines TableGen record `mem_msa` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_msa`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1182-1184
```tablegen
def simm12 : Operand<i32> {
  let DecoderMethod = "DecodeSimm12";
}
```
- EN: Defines TableGen record `simm12` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `simm12`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1186-1191
```tablegen
def mem_simm9_exp : mem_generic {
  let MIOperandInfo = (ops mips_ptr_rc, simm9);
  let ParserMatchClass = MipsMemSimmPtrAsmOperand;
  let OperandNamespace = "MipsII";
  let OperandType = "OPERAND_MEM_SIMM9";
}
```
- EN: Defines TableGen record `mem_simm9_exp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_simm9_exp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1193-1197
```tablegen
foreach I = {9, 10, 11, 12, 16} in
  def mem_simm # I : mem_generic {
    let MIOperandInfo = (ops mips_ptr_rc, !cast<Operand>("simm" # I));
    let ParserMatchClass = MipsMemSimmAsmOperand<I>;
  }
```
- EN: Defines TableGen record `mem_simm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_simm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1199-1204
```tablegen
foreach I = {1, 2, 3} in
  def mem_simm10_lsl # I : mem_generic {
    let MIOperandInfo = (ops mips_ptr_rc, !cast<Operand>("simm10_lsl" # I));
    let EncoderMethod = "getMemEncoding<" # I  # ">";
    let ParserMatchClass = MipsMemSimmAsmOperand<10, I>;
  }
```
- EN: Defines TableGen record `mem_simm10_lsl` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_simm10_lsl`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1206-1208
```tablegen
def mem_simmptr : mem_generic {
  let ParserMatchClass = MipsMemSimmPtrAsmOperand;
}
```
- EN: Defines TableGen record `mem_simmptr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_simmptr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1210-1215
```tablegen
def mem_ea : Operand<iPTR> {
  let PrintMethod = "printMemOperandEA";
  let MIOperandInfo = (ops mips_ptr_rc, simm16);
  let EncoderMethod = "getMemEncoding";
  let OperandType = "OPERAND_MEMORY";
}
```
- EN: Defines TableGen record `mem_ea` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `mem_ea`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1217-1221
```tablegen
def PtrRC : Operand<iPTR> {
  let MIOperandInfo = (ops mips_ptr_rc);
  let DecoderMethod = "DecodePtrRegisterClass";
  let ParserMatchClass = GPR32AsmOperand;
}
```
- EN: Defines TableGen record `PtrRC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PtrRC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1223-1227
```tablegen
// size operand of ins instruction
def size_ins : Operand<i32> {
  let EncoderMethod = "getSizeInsEncoding";
  let DecoderMethod = "DecodeInsSize";
}
```
- EN: Defines TableGen record `size_ins` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `size_ins`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1229-1232
```tablegen
// Transformation Function - get the lower 16 bits.
def LO16 : SDNodeXForm<imm, [{
  return getImm(N, N->getZExtValue() & 0xFFFF);
}]>;
```
- EN: Defines TableGen record `LO16` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `LO16`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1234-1237
```tablegen
// Transformation Function - get the higher 16 bits.
def HI16 : SDNodeXForm<imm, [{
  return getImm(N, (N->getZExtValue() >> 16) & 0xFFFF);
}]>;
```
- EN: Defines TableGen record `HI16` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `HI16`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1239-1242
```tablegen
// Plus 1.
def Plus1 : SDNodeXForm<imm, [{
  return getSignedImm(N, N->getSExtValue() + 1);
}]>;
```
- EN: Defines TableGen record `Plus1` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `Plus1`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1244-1245
```tablegen
// Node immediate is zero (e.g. insve.d)
def immz : PatLeaf<(imm), [{ return N->getSExtValue() == 0; }]>;
```
- EN: Defines TableGen record `immz` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immz`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1247-1247
```tablegen
def timmz : PatLeaf<(timm), [{ return N->getSExtValue() == 0; }]>;
```
- EN: Defines TableGen record `timmz` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `timmz`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1249-1251
```tablegen
// Node immediate fits as 16-bit sign extended on target immediate.
// e.g. addi, andi
def immSExt8  : PatLeaf<(imm), [{ return isInt<8>(N->getSExtValue()); }]>;
```
- EN: Defines TableGen record `immSExt8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immSExt8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1253-1256
```tablegen
// Node immediate fits as 16-bit sign extended on target immediate.
// e.g. addi, andi
def immSExt16  : PatLeaf<(imm), [{ return isInt<16>(N->getSExtValue()); }]>;
def imm32SExt16  : IntImmLeaf<i32, [{ return isInt<16>(Imm.getSExtValue()); }]>;
```
- EN: Defines TableGen record `immSExt16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immSExt16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1258-1260
```tablegen
// Node immediate fits as 7-bit zero extended on target immediate.
def immZExt7 : PatLeaf<(imm), [{ return isUInt<7>(N->getZExtValue()); }]>;
def timmZExt7 : PatLeaf<(timm), [{ return isUInt<7>(N->getZExtValue()); }]>;
```
- EN: Defines TableGen record `immZExt7` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExt7`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1262-1274
```tablegen
// Node immediate fits as 16-bit zero extended on target immediate.
// The LO16 param means that only the lower 16 bits of the node
// immediate are caught.
// e.g. addiu, sltiu
def immZExt16  : PatLeaf<(imm), [{
  if (N->getValueType(0) == MVT::i32)
    return (uint32_t)N->getZExtValue() == (unsigned short)N->getZExtValue();
  else
    return (uint64_t)N->getZExtValue() == (unsigned short)N->getZExtValue();
}], LO16>;
def imm32ZExt16  : IntImmLeaf<i32, [{
  return (uint32_t)Imm.getZExtValue() == (unsigned short)Imm.getZExtValue();
}]>;
```
- EN: Defines TableGen record `immZExt16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExt16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1276-1280
```tablegen
// Immediate can be loaded with LUi (32-bit int with lower 16-bit cleared).
def immSExt32Low16Zero : PatLeaf<(imm), [{
  int64_t Val = N->getSExtValue();
  return isInt<32>(Val) && !(Val & 0xffff);
}]>;
```
- EN: Defines TableGen record `immSExt32Low16Zero` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immSExt32Low16Zero`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1282-1286
```tablegen
// Zero-extended 32-bit unsigned int with lower 16-bit cleared.
def immZExt32Low16Zero : PatLeaf<(imm), [{
  uint64_t Val = N->getZExtValue();
  return isUInt<32>(Val) && !(Val & 0xffff);
}]>;
```
- EN: Defines TableGen record `immZExt32Low16Zero` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExt32Low16Zero`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1288-1289
```tablegen
// Note immediate fits as a 32 bit signed extended on target immediate.
def immSExt32  : PatLeaf<(imm), [{ return isInt<32>(N->getSExtValue()); }]>;
```
- EN: Defines TableGen record `immSExt32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immSExt32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1291-1292
```tablegen
// Note immediate fits as a 32 bit zero extended on target immediate.
def immZExt32  : PatLeaf<(imm), [{ return isUInt<32>(N->getZExtValue()); }]>;
```
- EN: Defines TableGen record `immZExt32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExt32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1294-1296
```tablegen
// shamt field must fit in 5 bits.
def immZExt5 : ImmLeaf<i32, [{return Imm == (Imm & 0x1f);}]>;
def timmZExt5 : TImmLeaf<i32, [{return Imm == (Imm & 0x1f);}]>;
```
- EN: Defines TableGen record `immZExt5` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExt5`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1298-1306
```tablegen
def immZExt5Plus1 : PatLeaf<(imm), [{
  return isUInt<5>(N->getZExtValue() - 1);
}]>;
def immZExt5Plus32 : PatLeaf<(imm), [{
  return isUInt<5>(N->getZExtValue() - 32);
}]>;
def immZExt5Plus33 : PatLeaf<(imm), [{
  return isUInt<5>(N->getZExtValue() - 33);
}]>;
```
- EN: Defines TableGen record `immZExt5Plus1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExt5Plus1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1308-1310
```tablegen
def immZExt5To31 : SDNodeXForm<imm, [{
  return getImm(N, 31 - N->getZExtValue());
}]>;
```
- EN: Defines TableGen record `immZExt5To31` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `immZExt5To31`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1312-1315
```tablegen
// True if (N + 1) fits in 16-bit field.
def immSExt16Plus1 : PatLeaf<(imm), [{
  return isInt<17>(N->getSExtValue()) && isInt<16>(N->getSExtValue() + 1);
}]>;
```
- EN: Defines TableGen record `immSExt16Plus1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immSExt16Plus1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1317-1320
```tablegen
def immZExtRange2To64 : PatLeaf<(imm), [{
  return isUInt<7>(N->getZExtValue()) && (N->getZExtValue() >= 2) &&
         (N->getZExtValue() <= 64);
}]>;
```
- EN: Defines TableGen record `immZExtRange2To64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExtRange2To64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1322-1324
```tablegen
def ORiPred  : PatLeaf<(imm), [{
  return isUInt<16>(N->getZExtValue()) && !isInt<16>(N->getSExtValue());
}], LO16>;
```
- EN: Defines TableGen record `ORiPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ORiPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1326-1329
```tablegen
def LUiPred : PatLeaf<(imm), [{
  int64_t Val = N->getSExtValue();
  return !isInt<16>(Val) && isInt<32>(Val) && !(Val & 0xffff);
}]>;
```
- EN: Defines TableGen record `LUiPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LUiPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1331-1334
```tablegen
def LUiORiPred  : PatLeaf<(imm), [{
  int64_t SVal = N->getSExtValue();
  return isInt<32>(SVal) && (SVal & 0xffff);
}]>;
```
- EN: Defines TableGen record `LUiORiPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LUiORiPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1336-1339
```tablegen
// Mips Address Mode! SDNode frameindex could possibly be a match
// since load and store instructions from stack used it.
def addr :
  ComplexPattern<iPTR, 2, "selectIntAddr", [frameindex]>;
```
- EN: Defines TableGen record `addr` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `addr`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1341-1342
```tablegen
def addrRegImm :
  ComplexPattern<iPTR, 2, "selectAddrRegImm", [frameindex]>;
```
- EN: Defines TableGen record `addrRegImm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrRegImm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1344-1345
```tablegen
def addrDefault :
  ComplexPattern<iPTR, 2, "selectAddrDefault", [frameindex]>;
```
- EN: Defines TableGen record `addrDefault` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrDefault`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1347-1353
```tablegen
def addrimm10 : ComplexPattern<iPTR, 2, "selectIntAddrSImm10", [frameindex]>;
def addrimm10lsl1 : ComplexPattern<iPTR, 2, "selectIntAddrSImm10Lsl1",
                                   [frameindex]>;
def addrimm10lsl2 : ComplexPattern<iPTR, 2, "selectIntAddrSImm10Lsl2",
                                   [frameindex]>;
def addrimm10lsl3 : ComplexPattern<iPTR, 2, "selectIntAddrSImm10Lsl3",
                                   [frameindex]>;
```
- EN: Defines TableGen record `addrimm10` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrimm10`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1355-1357
```tablegen
//===----------------------------------------------------------------------===//
// Instructions specific format
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1359-1368
```tablegen
// Arithmetic and logical instructions with 3 register operands.
class ArithLogicR<string opstr, RegisterOperand RO, bit isComm = 0,
                  SDPatternOperator OpNode = null_frag>:
  InstSE<(outs RO:$rd), (ins RO:$rs, RO:$rt),
         !strconcat(opstr, "\t$rd, $rs, $rt"),
         [(set RO:$rd, (OpNode RO:$rs, RO:$rt))], FrmR, opstr> {
  let isCommutable = isComm;
  let isReMaterializable = 1;
  let TwoOperandAliasConstraint = "$rd = $rs";
}
```
- EN: Declares reusable TableGen class `ArithLogicR` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ArithLogicR`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1370-1380
```tablegen
// Arithmetic and logical instructions with 2 register operands.
class ArithLogicI<string opstr, Operand Od, RegisterOperand RO,
                  SDPatternOperator imm_type = null_frag,
                  SDPatternOperator OpNode = null_frag> :
  InstSE<(outs RO:$rt), (ins RO:$rs, Od:$imm16),
         !strconcat(opstr, "\t$rt, $rs, $imm16"),
         [(set RO:$rt, (OpNode RO:$rs, imm_type:$imm16))],
         FrmI, opstr> {
  let isReMaterializable = 1;
  let TwoOperandAliasConstraint = "$rs = $rt";
}
```
- EN: Declares reusable TableGen class `ArithLogicI` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ArithLogicI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1382-1389
```tablegen
// Arithmetic Multiply ADD/SUB
class MArithR<string opstr, bit isComm = 0> :
  InstSE<(outs), (ins GPR32Opnd:$rs, GPR32Opnd:$rt),
         !strconcat(opstr, "\t$rs, $rt"), [], FrmR, opstr> {
  let Defs = [HI0, LO0];
  let Uses = [HI0, LO0];
  let isCommutable = isComm;
}
```
- EN: Declares reusable TableGen class `MArithR` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `MArithR`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1391-1397
```tablegen
//  Logical
class LogicNOR<string opstr, RegisterOperand RO>:
  InstSE<(outs RO:$rd), (ins RO:$rs, RO:$rt),
         !strconcat(opstr, "\t$rd, $rs, $rt"),
         [(set RO:$rd, (not (or RO:$rs, RO:$rt)))], FrmR, opstr> {
  let isCommutable = 1;
}
```
- EN: Declares reusable TableGen class `LogicNOR` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `LogicNOR`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1399-1408
```tablegen
// Shifts
class shift_rotate_imm<string opstr, Operand ImmOpnd,
                       RegisterOperand RO,
                       SDPatternOperator OpNode = null_frag,
                       SDPatternOperator PF = null_frag> :
  InstSE<(outs RO:$rd), (ins RO:$rt, ImmOpnd:$shamt),
         !strconcat(opstr, "\t$rd, $rt, $shamt"),
         [(set RO:$rd, (OpNode RO:$rt, PF:$shamt))], FrmR, opstr> {
  let TwoOperandAliasConstraint = "$rt = $rd";
}
```
- EN: Declares reusable TableGen class `shift_rotate_imm` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `shift_rotate_imm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1410-1415
```tablegen
class shift_rotate_reg<string opstr, RegisterOperand RO,
                       SDPatternOperator OpNode = null_frag>:
  InstSE<(outs RO:$rd), (ins RO:$rt, GPR32Opnd:$rs),
         !strconcat(opstr, "\t$rd, $rt, $rs"),
         [(set RO:$rd, (OpNode RO:$rt, GPR32Opnd:$rs))], FrmR,
         opstr>;
```
- EN: Declares reusable TableGen class `shift_rotate_reg` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `shift_rotate_reg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1417-1423
```tablegen
// Load Upper Immediate
class LoadUpper<string opstr, RegisterOperand RO, Operand Imm>:
  InstSE<(outs RO:$rt), (ins Imm:$imm16), !strconcat(opstr, "\t$rt, $imm16"),
         [], FrmI, opstr>, IsAsCheapAsAMove {
  let hasSideEffects = 0;
  let isReMaterializable = 1;
}
```
- EN: Declares reusable TableGen class `LoadUpper` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `LoadUpper`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1425-1435
```tablegen
// Memory Load/Store
class LoadMemory<string opstr, DAGOperand RO, DAGOperand MO,
                 SDPatternOperator OpNode = null_frag,
                 ComplexPattern Addr = addr> :
  InstSE<(outs RO:$rt), (ins MO:$addr), !strconcat(opstr, "\t$rt, $addr"),
         [(set RO:$rt, (OpNode Addr:$addr))], FrmI, opstr> {
  let DecoderMethod = "DecodeMem";
  let canFoldAsLoad = 1;
  string BaseOpcode = opstr;
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LoadMemory` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `LoadMemory`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1437-1439
```tablegen
class Load<string opstr, DAGOperand RO, SDPatternOperator OpNode = null_frag,
           ComplexPattern Addr = addr> :
  LoadMemory<opstr, RO, mem, OpNode, Addr>;
```
- EN: Declares reusable TableGen class `Load` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `Load`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1441-1449
```tablegen
class StoreMemory<string opstr, DAGOperand RO, DAGOperand MO,
            SDPatternOperator OpNode = null_frag,
            ComplexPattern Addr = addr> :
  InstSE<(outs), (ins RO:$rt, MO:$addr), !strconcat(opstr, "\t$rt, $addr"),
         [(OpNode RO:$rt, Addr:$addr)], FrmI, opstr> {
  let DecoderMethod = "DecodeMem";
  string BaseOpcode = opstr;
  let mayStore = 1;
}
```
- EN: Declares reusable TableGen class `StoreMemory` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `StoreMemory`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1451-1453
```tablegen
class Store<string opstr, DAGOperand RO, SDPatternOperator OpNode = null_frag,
            ComplexPattern Addr = addr, DAGOperand MO = mem> :
  StoreMemory<opstr, RO, MO, OpNode, Addr>;
```
- EN: Declares reusable TableGen class `Store` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `Store`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1455-1464
```tablegen
// Load/Store Left/Right
let canFoldAsLoad = 1 in
class LoadLeftRight<string opstr, SDNode OpNode, RegisterOperand RO> :
  InstSE<(outs RO:$rt), (ins mem:$addr, RO:$src),
         !strconcat(opstr, "\t$rt, $addr"),
         [(set RO:$rt, (OpNode addr:$addr, RO:$src))], FrmI> {
  let DecoderMethod = "DecodeMem";
  string Constraints = "$src = $rt";
  let BaseOpcode = opstr;
}
```
- EN: Declares reusable TableGen class `LoadLeftRight` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `LoadLeftRight`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1466-1471
```tablegen
class StoreLeftRight<string opstr, SDNode OpNode, RegisterOperand RO> :
  InstSE<(outs), (ins RO:$rt, mem:$addr), !strconcat(opstr, "\t$rt, $addr"),
         [(OpNode RO:$rt, addr:$addr)], FrmI> {
  let DecoderMethod = "DecodeMem";
  let BaseOpcode = opstr;
}
```
- EN: Declares reusable TableGen class `StoreLeftRight` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `StoreLeftRight`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1473-1481
```tablegen
// COP2 Load/Store
class LW_FT2<string opstr, RegisterOperand RC,
             SDPatternOperator OpNode= null_frag> :
  InstSE<(outs RC:$rt), (ins mem_simm16:$addr),
         !strconcat(opstr, "\t$rt, $addr"),
         [(set RC:$rt, (OpNode addrDefault:$addr))], FrmFI, opstr> {
  let DecoderMethod = "DecodeFMem2";
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LW_FT2` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `LW_FT2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1483-1490
```tablegen
class SW_FT2<string opstr, RegisterOperand RC,
             SDPatternOperator OpNode= null_frag> :
  InstSE<(outs), (ins RC:$rt, mem_simm16:$addr),
         !strconcat(opstr, "\t$rt, $addr"),
         [(OpNode RC:$rt, addrDefault:$addr)], FrmFI, opstr> {
  let DecoderMethod = "DecodeFMem2";
  let mayStore = 1;
}
```
- EN: Declares reusable TableGen class `SW_FT2` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SW_FT2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1492-1499
```tablegen
// COP3 Load/Store
class LW_FT3<string opstr, RegisterOperand RC,
             SDPatternOperator OpNode= null_frag> :
  InstSE<(outs RC:$rt), (ins mem:$addr), !strconcat(opstr, "\t$rt, $addr"),
         [(set RC:$rt, (OpNode addrDefault:$addr))], FrmFI, opstr> {
  let DecoderMethod = "DecodeFMem3";
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LW_FT3` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `LW_FT3`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1501-1507
```tablegen
class SW_FT3<string opstr, RegisterOperand RC,
             SDPatternOperator OpNode= null_frag> :
  InstSE<(outs), (ins RC:$rt, mem:$addr), !strconcat(opstr, "\t$rt, $addr"),
         [(OpNode RC:$rt, addrDefault:$addr)], FrmFI, opstr> {
  let DecoderMethod = "DecodeFMem3";
  let mayStore = 1;
}
```
- EN: Declares reusable TableGen class `SW_FT3` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SW_FT3`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1509-1521
```tablegen
// Conditional Branch
class CBranch<string opstr, DAGOperand opnd, PatFrag cond_op,
              RegisterOperand RO> :
  InstSE<(outs), (ins RO:$rs, RO:$rt, opnd:$offset),
         !strconcat(opstr, "\t$rs, $rt, $offset"),
         [(brcond (i32 (cond_op RO:$rs, RO:$rt)), bb:$offset)],
         FrmI, opstr> {
  let isBranch = 1;
  let isTerminator = 1;
  let hasDelaySlot = 1;
  let Defs = [AT];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `CBranch` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `CBranch`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1523-1531
```tablegen
class CBranchLikely<string opstr, DAGOperand opnd, RegisterOperand RO> :
  InstSE<(outs), (ins RO:$rs, RO:$rt, opnd:$offset),
         !strconcat(opstr, "\t$rs, $rt, $offset"), [], FrmI, opstr> {
  let isBranch = 1;
  let isTerminator = 1;
  let hasDelaySlot = 1;
  let Defs = [AT];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `CBranchLikely` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `CBranchLikely`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1533-1544
```tablegen
class CBranchZero<string opstr, DAGOperand opnd, PatFrag cond_op,
                  RegisterOperand RO> :
  InstSE<(outs), (ins RO:$rs, opnd:$offset),
         !strconcat(opstr, "\t$rs, $offset"),
         [(brcond (i32 (cond_op RO:$rs, 0)), bb:$offset)],
         FrmI, opstr> {
  let isBranch = 1;
  let isTerminator = 1;
  let hasDelaySlot = 1;
  let Defs = [AT];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `CBranchZero` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `CBranchZero`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1546-1554
```tablegen
class CBranchZeroLikely<string opstr, DAGOperand opnd, RegisterOperand RO> :
  InstSE<(outs), (ins RO:$rs, opnd:$offset),
         !strconcat(opstr, "\t$rs, $offset"), [], FrmI, opstr> {
  let isBranch = 1;
  let isTerminator = 1;
  let hasDelaySlot = 1;
  let Defs = [AT];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `CBranchZeroLikely` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `CBranchZeroLikely`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1556-1561
```tablegen
// SetCC
class SetCC_R<string opstr, PatFrag cond_op, RegisterOperand RO> :
  InstSE<(outs GPR32Opnd:$rd), (ins RO:$rs, RO:$rt),
         !strconcat(opstr, "\t$rd, $rs, $rt"),
         [(set GPR32Opnd:$rd, (cond_op RO:$rs, RO:$rt))],
         FrmR, opstr>;
```
- EN: Declares reusable TableGen class `SetCC_R` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SetCC_R`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1563-1568
```tablegen
class SetCC_I<string opstr, PatFrag cond_op, Operand Od, PatLeaf imm_type,
              RegisterOperand RO>:
  InstSE<(outs GPR32Opnd:$rt), (ins RO:$rs, Od:$imm16),
         !strconcat(opstr, "\t$rt, $rs, $imm16"),
         [(set GPR32Opnd:$rt, (cond_op RO:$rs, imm_type:$imm16))],
         FrmI, opstr>;
```
- EN: Declares reusable TableGen class `SetCC_I` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SetCC_I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1570-1581
```tablegen
// Jump
class JumpFJ<DAGOperand opnd, string opstr, SDPatternOperator operator,
             SDPatternOperator targetoperator, string bopstr> :
  InstSE<(outs), (ins opnd:$target), !strconcat(opstr, "\t$target"),
         [(operator targetoperator:$target)], FrmJ, bopstr> {
  let isTerminator=1;
  let isBarrier=1;
  let hasDelaySlot = 1;
  let DecoderMethod = "DecodeJumpTarget";
  let Defs = [AT];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `JumpFJ` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `JumpFJ`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1583-1594
```tablegen
// Unconditional branch
class UncondBranch<Instruction BEQInst, DAGOperand opnd> :
  PseudoSE<(outs), (ins brtarget:$offset), [(br bb:$offset)]>,
  PseudoInstExpansion<(BEQInst ZERO, ZERO, opnd:$offset)> {
  let isBranch = 1;
  let isTerminator = 1;
  let isBarrier = 1;
  let hasDelaySlot = 1;
  let AdditionalPredicates = [RelocPIC];
  let Defs = [AT];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `UncondBranch` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `UncondBranch`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1596-1601
```tablegen
// Base class for indirect branch and return instruction classes.
let isTerminator=1, isBarrier=1, hasDelaySlot = 1, isCTI = 1 in
class JumpFR<string opstr, RegisterOperand RO,
             SDPatternOperator operator = null_frag>:
  InstSE<(outs), (ins RO:$rs), "jr\t$rs", [(operator RO:$rs)],
         FrmR, opstr>;
```
- EN: Declares reusable TableGen class `for` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1603-1607
```tablegen
// Indirect branch
class IndirectBranch<string opstr, RegisterOperand RO> : JumpFR<opstr, RO> {
  let isBranch = 1;
  let isIndirectBranch = 1;
}
```
- EN: Declares reusable TableGen class `IndirectBranch` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `IndirectBranch`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1609-1615
```tablegen
// Jump and Link (Call)
let isCall=1, hasDelaySlot=1, isCTI=1, Defs = [RA] in {
  class JumpLink<string opstr, DAGOperand opnd> :
    InstSE<(outs), (ins opnd:$target), !strconcat(opstr, "\t$target"),
           [(MipsJmpLink tglobaladdr:$target)], FrmJ, opstr> {
    let DecoderMethod = "DecodeJumpTarget";
  }
```
- EN: Declares reusable TableGen class `JumpLink` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `JumpLink`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1617-1622
```tablegen
  class JumpLinkRegPseudo<RegisterOperand RO, Instruction JALRInst,
                          Register RetReg, RegisterOperand ResRO = RO>:
    PseudoSE<(outs), (ins RO:$rs), [(MipsJmpLink RO:$rs)]>,
    PseudoInstExpansion<(JALRInst RetReg, ResRO:$rs)> {
    let hasPostISelHook = 1;
  }
```
- EN: Declares reusable TableGen class `JumpLinkRegPseudo` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `JumpLinkRegPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1624-1628
```tablegen
  class JumpLinkReg<string opstr, RegisterOperand RO>:
    InstSE<(outs RO:$rd), (ins RO:$rs), !strconcat(opstr, "\t$rd, $rs"),
           [], FrmR, opstr> {
    let hasPostISelHook = 1;
  }
```
- EN: Declares reusable TableGen class `JumpLinkReg` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `JumpLinkReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1630-1635
```tablegen
  class BGEZAL_FT<string opstr, DAGOperand opnd,
                  RegisterOperand RO> :
    InstSE<(outs), (ins RO:$rs, opnd:$offset),
           !strconcat(opstr, "\t$rs, $offset"), [], FrmI, opstr> {
    let hasDelaySlot = 1;
  }
```
- EN: Declares reusable TableGen class `BGEZAL_FT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `BGEZAL_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1637-1637
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1639-1643
```tablegen
let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, hasDelaySlot = 1,
    hasExtraSrcRegAllocReq = 1, isCTI = 1, Defs = [AT] in {
  class TailCall<Instruction JumpInst, DAGOperand Opnd> :
    PseudoSE<(outs), (ins calltarget:$target), []>,
    PseudoInstExpansion<(JumpInst Opnd:$target)>;
```
- EN: Declares reusable TableGen class `TailCall` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `TailCall`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1645-1650
```tablegen
  class TailCallReg<Instruction JumpInst, RegisterOperand RO> :
    PseudoSE<(outs), (ins RO:$rs), [(MipsTailCall RO:$rs)]>,
    PseudoInstExpansion<(JumpInst RO:$rs)> {
    let hasPostISelHook = 1;
  }
}
```
- EN: Declares reusable TableGen class `TailCallReg` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `TailCallReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1652-1661
```tablegen
class BAL_BR_Pseudo<Instruction RealInst, DAGOperand opnd> :
  PseudoSE<(outs), (ins opnd:$offset), []>,
  PseudoInstExpansion<(RealInst ZERO, opnd:$offset)> {
  let isBranch = 1;
  let isTerminator = 1;
  let isBarrier = 1;
  let hasDelaySlot = 1;
  let Defs = [RA];
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `BAL_BR_Pseudo` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `BAL_BR_Pseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1663-1672
```tablegen
let isCTI = 1 in {
// Syscall
class SYS_FT<string opstr, Operand ImmOp> :
  InstSE<(outs), (ins ImmOp:$code_),
         !strconcat(opstr, "\t$code_"), [], FrmI, opstr>;
// Break
class BRK_FT<string opstr> :
  InstSE<(outs), (ins uimm10:$code_1, uimm10:$code_2),
         !strconcat(opstr, "\t$code_1, $code_2"), [],
         FrmOther, opstr>;
```
- EN: Declares reusable TableGen class `SYS_FT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SYS_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1674-1677
```tablegen
// (D)Eret
class ER_FT<string opstr> :
  InstSE<(outs), (ins),
         opstr, [], FrmOther, opstr>;
```
- EN: Declares reusable TableGen class `ER_FT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ER_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1679-1682
```tablegen
// Wait
class WAIT_FT<string opstr> :
  InstSE<(outs), (ins), opstr, [], FrmOther, opstr>;
}
```
- EN: Declares reusable TableGen class `WAIT_FT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `WAIT_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1684-1687
```tablegen
// Interrupts
class DEI_FT<string opstr, RegisterOperand RO> :
  InstSE<(outs RO:$rt), (ins),
         !strconcat(opstr, "\t$rt"), [], FrmOther, opstr>;
```
- EN: Declares reusable TableGen class `DEI_FT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `DEI_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1689-1693
```tablegen
// Sync
let hasSideEffects = 1 in
class SYNC_FT<string opstr> :
  InstSE<(outs), (ins uimm5:$stype), "sync $stype",
         [(MipsSync timmZExt5:$stype)], FrmOther, opstr>;
```
- EN: Declares reusable TableGen class `SYNC_FT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SYNC_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1695-1700
```tablegen
class SYNCI_FT<string opstr, DAGOperand MO> :
  InstSE<(outs), (ins MO:$addr), !strconcat(opstr, "\t$addr"), [],
         FrmOther, opstr> {
  let hasSideEffects = 1;
  let DecoderMethod = "DecodeSyncI";
}
```
- EN: Declares reusable TableGen class `SYNCI_FT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SYNCI_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1702-1705
```tablegen
let hasSideEffects = 1, isCTI = 1 in {
class TEQ_FT<string opstr, RegisterOperand RO, Operand ImmOp> :
  InstSE<(outs), (ins RO:$rs, RO:$rt, ImmOp:$code_),
         !strconcat(opstr, "\t$rs, $rt, $code_"), [], FrmI, opstr>;
```
- EN: Declares reusable TableGen class `TEQ_FT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `TEQ_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1707-1710
```tablegen
class TEQI_FT<string opstr, RegisterOperand RO> :
  InstSE<(outs), (ins RO:$rs, simm16:$imm16),
         !strconcat(opstr, "\t$rs, $imm16"), [], FrmOther, opstr>;
}
```
- EN: Declares reusable TableGen class `TEQI_FT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `TEQI_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1712-1720
```tablegen
// Mul, Div
class Mult<string opstr, RegisterOperand RO,
           list<Register> DefRegs> :
  InstSE<(outs), (ins RO:$rs, RO:$rt), !strconcat(opstr, "\t$rs, $rt"), [],
         FrmR, opstr> {
  let isCommutable = 1;
  let Defs = DefRegs;
  let hasSideEffects = 0;
}
```
- EN: Declares reusable TableGen class `Mult` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `Mult`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1722-1734
```tablegen
// Pseudo multiply/divide instruction with explicit accumulator register
// operands.
class MultDivPseudo<Instruction RealInst, RegisterClass R0, RegisterOperand R1,
                    SDPatternOperator OpNode,
                    bit IsComm = 1, bit HasSideEffects = 0,
                    bit UsesCustomInserter = 0> :
  PseudoSE<(outs R0:$ac), (ins R1:$rs, R1:$rt),
           [(set R0:$ac, (OpNode R1:$rs, R1:$rt))]>,
  PseudoInstExpansion<(RealInst R1:$rs, R1:$rt)> {
  let isCommutable = IsComm;
  let hasSideEffects = HasSideEffects;
  let usesCustomInserter = UsesCustomInserter;
}
```
- EN: Declares reusable TableGen class `MultDivPseudo` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `MultDivPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1736-1745
```tablegen
// Pseudo multiply add/sub instruction with explicit accumulator register
// operands.
class MAddSubPseudo<Instruction RealInst, SDPatternOperator OpNode>
  : PseudoSE<(outs ACC64:$ac),
             (ins GPR32Opnd:$rs, GPR32Opnd:$rt, ACC64:$acin),
             [(set ACC64:$ac,
              (OpNode GPR32Opnd:$rs, GPR32Opnd:$rt, ACC64:$acin))]>,
    PseudoInstExpansion<(RealInst GPR32Opnd:$rs, GPR32Opnd:$rt)> {
  string Constraints = "$acin = $ac";
}
```
- EN: Declares reusable TableGen class `MAddSubPseudo` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `MAddSubPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1747-1752
```tablegen
class Div<string opstr, RegisterOperand RO,
          list<Register> DefRegs> :
  InstSE<(outs), (ins RO:$rs, RO:$rt), !strconcat(opstr, "\t$$zero, $rs, $rt"),
         [], FrmR, opstr> {
  let Defs = DefRegs;
}
```
- EN: Declares reusable TableGen class `Div` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `Div`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1754-1757
```tablegen
// Move from Hi/Lo
class PseudoMFLOHI<RegisterClass DstRC, RegisterClass SrcRC, SDNode OpNode>
  : PseudoSE<(outs DstRC:$rd), (ins SrcRC:$hilo),
             [(set DstRC:$rd, (OpNode SrcRC:$hilo))]>;
```
- EN: Declares reusable TableGen class `PseudoMFLOHI` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `PseudoMFLOHI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1759-1765
```tablegen
class MoveFromLOHI<string opstr, RegisterOperand RO, Register UseReg>:
  InstSE<(outs RO:$rd), (ins), !strconcat(opstr, "\t$rd"), [],
         FrmR, opstr> {
  let Uses = [UseReg];
  let hasSideEffects = 0;
  let isMoveReg = 1;
}
```
- EN: Declares reusable TableGen class `MoveFromLOHI` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `MoveFromLOHI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1767-1769
```tablegen
class PseudoMTLOHI<RegisterClass DstRC, RegisterClass SrcRC>
  : PseudoSE<(outs DstRC:$lohi), (ins SrcRC:$lo, SrcRC:$hi),
             [(set DstRC:$lohi, (MipsMTLOHI SrcRC:$lo, SrcRC:$hi))]>;
```
- EN: Declares reusable TableGen class `PseudoMTLOHI` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `PseudoMTLOHI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1771-1777
```tablegen
class MoveToLOHI<string opstr, RegisterOperand RO, list<Register> DefRegs>:
  InstSE<(outs), (ins RO:$rs), !strconcat(opstr, "\t$rs"), [],
  FrmR, opstr> {
  let Defs = DefRegs;
  let hasSideEffects = 0;
  let isMoveReg = 1;
}
```
- EN: Declares reusable TableGen class `MoveToLOHI` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `MoveToLOHI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1779-1786
```tablegen
class EffectiveAddress<string opstr, RegisterOperand RO> :
  InstSE<(outs RO:$rt), (ins mem_ea:$addr), !strconcat(opstr, "\t$rt, $addr"),
         [(set RO:$rt, addr:$addr)], FrmI,
         !strconcat(opstr, "_lea")> {
  let isCodeGenOnly = 1;
  let hasNoSchedulingInfo = 1;
  let DecoderMethod = "DecodeMem";
}
```
- EN: Declares reusable TableGen class `EffectiveAddress` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `EffectiveAddress`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1788-1791
```tablegen
// Count Leading Ones/Zeros in Word
class CountLeading0<string opstr, RegisterOperand RO>:
  InstSE<(outs RO:$rd), (ins RO:$rs), !strconcat(opstr, "\t$rd, $rs"),
         [(set RO:$rd, (ctlz RO:$rs))], FrmR, opstr>;
```
- EN: Declares reusable TableGen class `CountLeading0` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `CountLeading0`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1793-1795
```tablegen
class CountLeading1<string opstr, RegisterOperand RO>:
  InstSE<(outs RO:$rd), (ins RO:$rs), !strconcat(opstr, "\t$rd, $rs"),
         [(set RO:$rd, (ctlz (not RO:$rs)))], FrmR, opstr>;
```
- EN: Declares reusable TableGen class `CountLeading1` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `CountLeading1`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1797-1800
```tablegen
// Sign Extend in Register.
class SignExtInReg<string opstr, ValueType vt, RegisterOperand RO> :
  InstSE<(outs RO:$rd), (ins RO:$rt), !strconcat(opstr, "\t$rd, $rt"),
         [(set RO:$rd, (sext_inreg RO:$rt, vt))], FrmR, opstr>;
```
- EN: Declares reusable TableGen class `SignExtInReg` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SignExtInReg`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1802-1807
```tablegen
// Subword Swap
class SubwordSwap<string opstr, RegisterOperand RO>:
  InstSE<(outs RO:$rd), (ins RO:$rt), !strconcat(opstr, "\t$rd, $rt"), [],
         FrmR, opstr> {
  let hasSideEffects = 0;
}
```
- EN: Declares reusable TableGen class `SubwordSwap` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SubwordSwap`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1809-1812
```tablegen
// Read Hardware
class ReadHardware<RegisterOperand CPURegOperand, RegisterOperand RO> :
  InstSE<(outs CPURegOperand:$rt), (ins RO:$rd, uimm8:$sel),
         "rdhwr\t$rt, $rd, $sel", [], FrmR, "rdhwr">;
```
- EN: Declares reusable TableGen class `ReadHardware` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ReadHardware`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1814-1821
```tablegen
// Ext and Ins
class ExtBase<string opstr, RegisterOperand RO, Operand PosOpnd,
              Operand SizeOpnd, PatFrag PosImm, PatFrag SizeImm,
              SDPatternOperator Op = null_frag> :
  InstSE<(outs RO:$rt), (ins RO:$rs, PosOpnd:$pos, SizeOpnd:$size),
         !strconcat(opstr, "\t$rt, $rs, $pos, $size"),
         [(set RO:$rt, (Op RO:$rs, PosImm:$pos, SizeImm:$size))],
         FrmR, opstr>;
```
- EN: Declares reusable TableGen class `ExtBase` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `ExtBase`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1823-1832
```tablegen
// 'ins' and its 64 bit variants are matched by C++ code.
class InsBase<string opstr, RegisterOperand RO, Operand PosOpnd,
              Operand SizeOpnd, PatFrag PosImm, PatFrag SizeImm>:
  InstSE<(outs RO:$rt), (ins RO:$rs, PosOpnd:$pos, SizeOpnd:$size, RO:$src),
         !strconcat(opstr, "\t$rt, $rs, $pos, $size"),
         [(set RO:$rt, (null_frag RO:$rs, PosImm:$pos, SizeImm:$size,
                                  RO:$src))],
         FrmR, opstr> {
  let Constraints = "$src = $rt";
}
```
- EN: Declares reusable TableGen class `InsBase` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `InsBase`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1834-1839
```tablegen
// Atomic instructions with 2 source operands (ATOMIC_SWAP & ATOMIC_LOAD_*).
class Atomic2Ops<PatFrag Op, RegisterClass DRC> :
  PseudoSE<(outs DRC:$dst), (ins PtrRC:$ptr, DRC:$incr),
           [(set DRC:$dst, (Op iPTR:$ptr, DRC:$incr))]> {
  let hasNoSchedulingInfo = 1;
}
```
- EN: Declares reusable TableGen class `Atomic2Ops` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `Atomic2Ops`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1841-1846
```tablegen
class Atomic2OpsPostRA<RegisterClass RC> :
  PseudoSE<(outs RC:$dst), (ins PtrRC:$ptr, RC:$incr), []> {
  let mayLoad = 1;
  let mayStore = 1;
  let Predicates = [NotR5900];
}
```
- EN: Declares reusable TableGen class `Atomic2OpsPostRA` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `Atomic2OpsPostRA`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1848-1853
```tablegen
class Atomic2OpsSubwordPostRA<RegisterClass RC>
    : PseudoSE<
          (outs RC:$dst),
          (ins PtrRC:$ptr, RC:$incr, RC:$mask, RC:$mask2, RC:$shiftamnt), []> {
  let Predicates = [NotR5900];
}
```
- EN: Declares reusable TableGen class `Atomic2OpsSubwordPostRA` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `Atomic2OpsSubwordPostRA`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1855-1862
```tablegen
// Atomic Compare & Swap.
// Atomic compare and swap is lowered into two stages. The first stage happens
// during ISelLowering, which produces the PostRA version of this instruction.
class AtomicCmpSwap<PatFrag Op, RegisterClass DRC> :
  PseudoSE<(outs DRC:$dst), (ins PtrRC:$ptr, DRC:$cmp, DRC:$swap),
           [(set DRC:$dst, (Op iPTR:$ptr, DRC:$cmp, DRC:$swap))]> {
  let hasNoSchedulingInfo = 1;
}
```
- EN: Declares reusable TableGen class `AtomicCmpSwap` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `AtomicCmpSwap`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1864-1869
```tablegen
class AtomicCmpSwapPostRA<RegisterClass RC> :
  PseudoSE<(outs RC:$dst), (ins PtrRC:$ptr, RC:$cmp, RC:$swap), []> {
  let mayLoad = 1;
  let mayStore = 1;
  let Predicates = [NotR5900];
}
```
- EN: Declares reusable TableGen class `AtomicCmpSwapPostRA` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `AtomicCmpSwapPostRA`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1871-1877
```tablegen
class AtomicCmpSwapSubwordPostRA<RegisterClass RC> :
  PseudoSE<(outs RC:$dst), (ins PtrRC:$ptr, RC:$mask, RC:$ShiftCmpVal,
                                RC:$mask2, RC:$ShiftNewVal, RC:$ShiftAmt), []> {
  let mayLoad = 1;
  let mayStore = 1;
  let Predicates = [NotR5900];
}
```
- EN: Declares reusable TableGen class `AtomicCmpSwapSubwordPostRA` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `AtomicCmpSwapSubwordPostRA`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1879-1884
```tablegen
class LLBase<string opstr, RegisterOperand RO, DAGOperand MO = mem> :
  InstSE<(outs RO:$rt), (ins MO:$addr), !strconcat(opstr, "\t$rt, $addr"),
         [], FrmI, opstr> {
  let DecoderMethod = "DecodeMem";
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LLBase` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `LLBase`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1886-1892
```tablegen
class SCBase<string opstr, RegisterOperand RO> :
  InstSE<(outs RO:$dst), (ins RO:$rt, mem:$addr),
         !strconcat(opstr, "\t$rt, $addr"), [], FrmI> {
  let DecoderMethod = "DecodeMem";
  let mayStore = 1;
  let Constraints = "$rt = $dst";
}
```
- EN: Declares reusable TableGen class `SCBase` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `SCBase`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1894-1898
```tablegen
class MFC3OP<string asmstr, RegisterOperand RO, RegisterOperand RD> :
  InstSE<(outs RO:$rt), (ins RD:$rd, uimm3:$sel),
         !strconcat(asmstr, "\t$rt, $rd, $sel"), [], FrmFR> {
  let BaseOpcode = asmstr;
}
```
- EN: Declares reusable TableGen class `MFC3OP` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `MFC3OP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1900-1904
```tablegen
class MTC3OP<string asmstr, RegisterOperand RO, RegisterOperand RD> :
  InstSE<(outs RO:$rd), (ins RD:$rt, uimm3:$sel),
         !strconcat(asmstr, "\t$rt, $rd, $sel"), [], FrmFR> {
  let BaseOpcode = asmstr;
}
```
- EN: Declares reusable TableGen class `MTC3OP` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `MTC3OP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1906-1914
```tablegen
class TrapBase<Instruction RealInst>
  : PseudoSE<(outs), (ins), [(trap)]>,
    PseudoInstExpansion<(RealInst 0, 0)> {
  let mayStore = 0;
  let mayLoad = 0;
  let hasSideEffects = 1;
  let isTrap = 1;
  let isCodeGenOnly = 1;
}
```
- EN: Declares reusable TableGen class `TrapBase` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `TrapBase`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1916-1918
```tablegen
//===----------------------------------------------------------------------===//
// Pseudo instructions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1920-1923
```tablegen
// Return RA.
let isReturn=1, isTerminator=1, isBarrier=1, hasCtrlDep=1, isCTI=1 in {
  let hasDelaySlot=1 in
  def RetRA : PseudoSE<(outs), (ins), [(MipsRet)]>;
```
- EN: Defines TableGen record `RetRA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RetRA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1925-1927
```tablegen
  let hasSideEffects=1 in
  def ERet : PseudoSE<(outs), (ins), [(MipsERet)]>;
}
```
- EN: Defines TableGen record `ERet` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ERet`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1929-1934
```tablegen
let Defs = [SP], Uses = [SP], hasSideEffects = 1, hasNoSchedulingInfo = 1 in {
def ADJCALLSTACKDOWN : MipsPseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
                                  [(callseq_start timm:$amt1, timm:$amt2)]>;
def ADJCALLSTACKUP   : MipsPseudo<(outs), (ins i32imm:$amt1, i32imm:$amt2),
                                  [(callseq_end timm:$amt1, timm:$amt2)]>;
}
```
- EN: Defines TableGen record `ADJCALLSTACKDOWN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADJCALLSTACKDOWN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1936-1953
```tablegen
let usesCustomInserter = 1 in {
  def ATOMIC_LOAD_ADD_I8   : Atomic2Ops<atomic_load_add_i8, GPR32>;
  def ATOMIC_LOAD_ADD_I16  : Atomic2Ops<atomic_load_add_i16, GPR32>;
  def ATOMIC_LOAD_ADD_I32  : Atomic2Ops<atomic_load_add_i32, GPR32>;
  def ATOMIC_LOAD_SUB_I8   : Atomic2Ops<atomic_load_sub_i8, GPR32>;
  def ATOMIC_LOAD_SUB_I16  : Atomic2Ops<atomic_load_sub_i16, GPR32>;
  def ATOMIC_LOAD_SUB_I32  : Atomic2Ops<atomic_load_sub_i32, GPR32>;
  def ATOMIC_LOAD_AND_I8   : Atomic2Ops<atomic_load_and_i8, GPR32>;
  def ATOMIC_LOAD_AND_I16  : Atomic2Ops<atomic_load_and_i16, GPR32>;
  def ATOMIC_LOAD_AND_I32  : Atomic2Ops<atomic_load_and_i32, GPR32>;
  def ATOMIC_LOAD_OR_I8    : Atomic2Ops<atomic_load_or_i8, GPR32>;
  def ATOMIC_LOAD_OR_I16   : Atomic2Ops<atomic_load_or_i16, GPR32>;
  def ATOMIC_LOAD_OR_I32   : Atomic2Ops<atomic_load_or_i32, GPR32>;
  def ATOMIC_LOAD_XOR_I8   : Atomic2Ops<atomic_load_xor_i8, GPR32>;
  def ATOMIC_LOAD_XOR_I16  : Atomic2Ops<atomic_load_xor_i16, GPR32>;
  def ATOMIC_LOAD_XOR_I32  : Atomic2Ops<atomic_load_xor_i32, GPR32>;
  def ATOMIC_LOAD_NAND_I8  : Atomic2Ops<atomic_load_nand_i8, GPR32>;
  def ATOMIC_LOAD_NAND_I16 : Atomic2Ops<atomic_load_nand_i16, GPR32>;
```
- EN: Defines TableGen record `ATOMIC_LOAD_ADD_I8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_LOAD_ADD_I8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1954-1954
```tablegen
  def ATOMIC_LOAD_NAND_I32 : Atomic2Ops<atomic_load_nand_i32, GPR32>;
```
- EN: Defines TableGen record `ATOMIC_LOAD_NAND_I32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_LOAD_NAND_I32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1956-1958
```tablegen
  def ATOMIC_SWAP_I8       : Atomic2Ops<atomic_swap_i8, GPR32>;
  def ATOMIC_SWAP_I16      : Atomic2Ops<atomic_swap_i16, GPR32>;
  def ATOMIC_SWAP_I32      : Atomic2Ops<atomic_swap_i32, GPR32>;
```
- EN: Defines TableGen record `ATOMIC_SWAP_I8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_SWAP_I8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1960-1962
```tablegen
  def ATOMIC_CMP_SWAP_I8   : AtomicCmpSwap<atomic_cmp_swap_i8, GPR32>;
  def ATOMIC_CMP_SWAP_I16  : AtomicCmpSwap<atomic_cmp_swap_i16, GPR32>;
  def ATOMIC_CMP_SWAP_I32  : AtomicCmpSwap<atomic_cmp_swap_i32, GPR32>;
```
- EN: Defines TableGen record `ATOMIC_CMP_SWAP_I8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_CMP_SWAP_I8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1964-1976
```tablegen
  def ATOMIC_LOAD_MIN_I8   : Atomic2Ops<atomic_load_min_i8, GPR32>;
  def ATOMIC_LOAD_MIN_I16  : Atomic2Ops<atomic_load_min_i16, GPR32>;
  def ATOMIC_LOAD_MIN_I32  : Atomic2Ops<atomic_load_min_i32, GPR32>;
  def ATOMIC_LOAD_MAX_I8   : Atomic2Ops<atomic_load_max_i8, GPR32>;
  def ATOMIC_LOAD_MAX_I16  : Atomic2Ops<atomic_load_max_i16, GPR32>;
  def ATOMIC_LOAD_MAX_I32  : Atomic2Ops<atomic_load_max_i32, GPR32>;
  def ATOMIC_LOAD_UMIN_I8  : Atomic2Ops<atomic_load_umin_i8, GPR32>;
  def ATOMIC_LOAD_UMIN_I16 : Atomic2Ops<atomic_load_umin_i16, GPR32>;
  def ATOMIC_LOAD_UMIN_I32 : Atomic2Ops<atomic_load_umin_i32, GPR32>;
  def ATOMIC_LOAD_UMAX_I8  : Atomic2Ops<atomic_load_umax_i8, GPR32>;
  def ATOMIC_LOAD_UMAX_I16 : Atomic2Ops<atomic_load_umax_i16, GPR32>;
  def ATOMIC_LOAD_UMAX_I32 : Atomic2Ops<atomic_load_umax_i32, GPR32>;
}
```
- EN: Defines TableGen record `ATOMIC_LOAD_MIN_I8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_LOAD_MIN_I8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1978-1995
```tablegen
def ATOMIC_LOAD_ADD_I8_POSTRA   : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_ADD_I16_POSTRA  : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_ADD_I32_POSTRA  : Atomic2OpsPostRA<GPR32>;
def ATOMIC_LOAD_SUB_I8_POSTRA   : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_SUB_I16_POSTRA  : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_SUB_I32_POSTRA  : Atomic2OpsPostRA<GPR32>;
def ATOMIC_LOAD_AND_I8_POSTRA   : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_AND_I16_POSTRA  : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_AND_I32_POSTRA  : Atomic2OpsPostRA<GPR32>;
def ATOMIC_LOAD_OR_I8_POSTRA    : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_OR_I16_POSTRA   : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_OR_I32_POSTRA   : Atomic2OpsPostRA<GPR32>;
def ATOMIC_LOAD_XOR_I8_POSTRA   : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_XOR_I16_POSTRA  : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_XOR_I32_POSTRA  : Atomic2OpsPostRA<GPR32>;
def ATOMIC_LOAD_NAND_I8_POSTRA  : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_NAND_I16_POSTRA : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_NAND_I32_POSTRA : Atomic2OpsPostRA<GPR32>;
```
- EN: Defines TableGen record `ATOMIC_LOAD_ADD_I8_POSTRA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_LOAD_ADD_I8_POSTRA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1997-1999
```tablegen
def ATOMIC_SWAP_I8_POSTRA  : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_SWAP_I16_POSTRA : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_SWAP_I32_POSTRA : Atomic2OpsPostRA<GPR32>;
```
- EN: Defines TableGen record `ATOMIC_SWAP_I8_POSTRA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_SWAP_I8_POSTRA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2001-2003
```tablegen
def ATOMIC_CMP_SWAP_I8_POSTRA : AtomicCmpSwapSubwordPostRA<GPR32>;
def ATOMIC_CMP_SWAP_I16_POSTRA : AtomicCmpSwapSubwordPostRA<GPR32>;
def ATOMIC_CMP_SWAP_I32_POSTRA : AtomicCmpSwapPostRA<GPR32>;
```
- EN: Defines TableGen record `ATOMIC_CMP_SWAP_I8_POSTRA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_CMP_SWAP_I8_POSTRA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2005-2016
```tablegen
def ATOMIC_LOAD_MIN_I8_POSTRA   : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_MIN_I16_POSTRA  : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_MIN_I32_POSTRA  : Atomic2OpsPostRA<GPR32>;
def ATOMIC_LOAD_MAX_I8_POSTRA   : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_MAX_I16_POSTRA  : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_MAX_I32_POSTRA  : Atomic2OpsPostRA<GPR32>;
def ATOMIC_LOAD_UMIN_I8_POSTRA  : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_UMIN_I16_POSTRA : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_UMIN_I32_POSTRA : Atomic2OpsPostRA<GPR32>;
def ATOMIC_LOAD_UMAX_I8_POSTRA  : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_UMAX_I16_POSTRA : Atomic2OpsSubwordPostRA<GPR32>;
def ATOMIC_LOAD_UMAX_I32_POSTRA : Atomic2OpsPostRA<GPR32>;
```
- EN: Defines TableGen record `ATOMIC_LOAD_MIN_I8_POSTRA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_LOAD_MIN_I8_POSTRA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2018-2022
```tablegen
/// Pseudo instructions for loading and storing accumulator registers.
let isPseudo = 1, isCodeGenOnly = 1, hasNoSchedulingInfo = 1 in {
  def LOAD_ACC64  : Load<"", ACC64>;
  def STORE_ACC64 : Store<"", ACC64>;
}
```
- EN: Defines TableGen record `LOAD_ACC64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LOAD_ACC64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2024-2026
```tablegen
// We need these two pseudo instructions to avoid offset calculation for long
// branches.  See the comment in file MipsLongBranch.cpp for detailed
// explanation.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2028-2037
```tablegen
// Expands to: lui $dst, %highest/%higher/%hi/%lo($tgt - $baltgt)
def LONG_BRANCH_LUi : PseudoSE<(outs GPR32Opnd:$dst),
  (ins brtarget:$tgt, brtarget:$baltgt), []> {
  bit hasNoSchedulingInfo = 1;
}
// Expands to: lui $dst, highest/%higher/%hi/%lo($tgt)
def LONG_BRANCH_LUi2Op : PseudoSE<(outs GPR32Opnd:$dst),
  (ins brtarget:$tgt), []> {
  bit hasNoSchedulingInfo = 1;
}
```
- EN: Defines TableGen record `LONG_BRANCH_LUi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LONG_BRANCH_LUi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2039-2048
```tablegen
// Expands to: addiu $dst, $src, %highest/%higher/%hi/%lo($tgt - $baltgt)
def LONG_BRANCH_ADDiu : PseudoSE<(outs GPR32Opnd:$dst),
  (ins GPR32Opnd:$src, brtarget:$tgt, brtarget:$baltgt), []> {
  bit hasNoSchedulingInfo = 1;
}
// Expands to: addiu $dst, $src, %highest/%higher/%hi/%lo($tgt)
def LONG_BRANCH_ADDiu2Op : PseudoSE<(outs GPR32Opnd:$dst),
  (ins GPR32Opnd:$src, brtarget:$tgt), []> {
  bit hasNoSchedulingInfo = 1;
}
```
- EN: Defines TableGen record `LONG_BRANCH_ADDiu` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LONG_BRANCH_ADDiu`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2050-2055
```tablegen
//===----------------------------------------------------------------------===//
// Instruction definition
//===----------------------------------------------------------------------===//
//===----------------------------------------------------------------------===//
// MipsI Instructions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2057-2061
```tablegen
/// Arithmetic Instructions (ALU Immediate)
let AdditionalPredicates = [NotInMicroMips] in {
  def ADDiu : MMRel, StdMMR6Rel, ArithLogicI<"addiu", simm16_relaxed, GPR32Opnd,
                                             imm32SExt16, add>,
              ADDI_FM<0x9>, IsAsCheapAsAMove, ISA_MIPS1;
```
- EN: Defines TableGen record `ADDiu` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADDiu`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2063-2077
```tablegen
  def ANDi : MMRel, StdMMR6Rel,
             ArithLogicI<"andi", uimm16, GPR32Opnd, imm32ZExt16, and>,
             ADDI_FM<0xc>, ISA_MIPS1;
  def ORi  : MMRel, StdMMR6Rel,
             ArithLogicI<"ori", uimm16, GPR32Opnd, imm32ZExt16, or>,
             ADDI_FM<0xd>, ISA_MIPS1;
  def XORi : MMRel, StdMMR6Rel,
             ArithLogicI<"xori", uimm16, GPR32Opnd, imm32ZExt16, xor>,
             ADDI_FM<0xe>, ISA_MIPS1;
  def ADDi  : MMRel, ArithLogicI<"addi", simm16_relaxed, GPR32Opnd>,
              ADDI_FM<0x8>, ISA_MIPS1_NOT_32R6_64R6;
  def SLTi  : MMRel, SetCC_I<"slti", setlt, simm16, immSExt16, GPR32Opnd>,
              SLTI_FM<0xa>, ISA_MIPS1;
  def SLTiu : MMRel, SetCC_I<"sltiu", setult, simm16, immSExt16, GPR32Opnd>,
              SLTI_FM<0xb>, ISA_MIPS1;
```
- EN: Defines TableGen record `ANDi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ANDi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2079-2080
```tablegen
  def LUi   : MMRel, LoadUpper<"lui", GPR32Opnd, uimm16_relaxed>, LUI_FM,
              ISA_MIPS1;
```
- EN: Defines TableGen record `LUi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LUi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2082-2086
```tablegen
  /// Arithmetic Instructions (3-Operand, R-Type)
  def ADDu : MMRel, StdMMR6Rel, ArithLogicR<"addu", GPR32Opnd, 1, add>,
             ADD_FM<0, 0x21>, ISA_MIPS1;
  def SUBu : MMRel, StdMMR6Rel, ArithLogicR<"subu", GPR32Opnd, 0, sub>,
             ADD_FM<0, 0x23>, ISA_MIPS1;
```
- EN: Defines TableGen record `ADDu` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADDu`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2088-2090
```tablegen
  let Defs = [HI0, LO0] in
    def MUL   : MMRel, ArithLogicR<"mul", GPR32Opnd, 1, mul>,
                ADD_FM<0x1c, 2>, ISA_MIPS32_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MUL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MUL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2092-2095
```tablegen
  def ADD   : MMRel, StdMMR6Rel, ArithLogicR<"add", GPR32Opnd, 1>,
              ADD_FM<0, 0x20>, ISA_MIPS1;
  def SUB   : MMRel, StdMMR6Rel, ArithLogicR<"sub", GPR32Opnd, 0>,
              ADD_FM<0, 0x22>, ISA_MIPS1;
```
- EN: Defines TableGen record `ADD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2097-2109
```tablegen
  def SLT   : MMRel, SetCC_R<"slt", setlt, GPR32Opnd>, ADD_FM<0, 0x2a>,
              ISA_MIPS1;
  def SLTu  : MMRel, SetCC_R<"sltu", setult, GPR32Opnd>, ADD_FM<0, 0x2b>,
              ISA_MIPS1;
  def AND   : MMRel, StdMMR6Rel, ArithLogicR<"and", GPR32Opnd, 1, and>,
              ADD_FM<0, 0x24>, ISA_MIPS1;
  def OR    : MMRel, StdMMR6Rel, ArithLogicR<"or", GPR32Opnd, 1, or>,
              ADD_FM<0, 0x25>, ISA_MIPS1;
  def XOR   : MMRel, StdMMR6Rel, ArithLogicR<"xor", GPR32Opnd, 1, xor>,
              ADD_FM<0, 0x26>, ISA_MIPS1;
  def NOR   : MMRel, StdMMR6Rel, LogicNOR<"nor", GPR32Opnd>, ADD_FM<0, 0x27>,
              ISA_MIPS1;
}
```
- EN: Defines TableGen record `SLT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SLT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2111-2124
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  /// Shift Instructions
  def SLL  : MMRel, shift_rotate_imm<"sll", uimm5, GPR32Opnd, mshl_32,
                                     immZExt5>, SRA_FM<0, 0>, ISA_MIPS1;
  def SRL  : MMRel, shift_rotate_imm<"srl", uimm5, GPR32Opnd, msrl_32,
                                     immZExt5>, SRA_FM<2, 0>, ISA_MIPS1;
  def SRA  : MMRel, shift_rotate_imm<"sra", uimm5, GPR32Opnd, msra_32,
                                     immZExt5>, SRA_FM<3, 0>, ISA_MIPS1;
  def SLLV : MMRel, shift_rotate_reg<"sllv", GPR32Opnd, mshl_32>,
             SRLV_FM<4, 0>, ISA_MIPS1;
  def SRLV : MMRel, shift_rotate_reg<"srlv", GPR32Opnd, msrl_32>,
             SRLV_FM<6, 0>, ISA_MIPS1;
  def SRAV : MMRel, shift_rotate_reg<"srav", GPR32Opnd, msra_32>,
             SRLV_FM<7, 0>, ISA_MIPS1;
```
- EN: Defines TableGen record `SLL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SLL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2126-2132
```tablegen
  // Rotate Instructions
  def ROTR  : MMRel, shift_rotate_imm<"rotr", uimm5, GPR32Opnd, rotr,
                                      immZExt5>,
              SRA_FM<2, 1>, ISA_MIPS32R2;
  def ROTRV : MMRel, shift_rotate_reg<"rotrv", GPR32Opnd, rotr>,
              SRLV_FM<6, 1>, ISA_MIPS32R2;
}
```
- EN: Defines TableGen record `ROTR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ROTR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2134-2151
```tablegen
/// Load and Store Instructions
///  aligned
let AdditionalPredicates = [NotInMicroMips] in {
  def LB  : LoadMemory<"lb", GPR32Opnd, mem_simmptr, sextloadi8>, MMRel,
            LW_FM<0x20>, ISA_MIPS1;
  def LBu : LoadMemory<"lbu", GPR32Opnd, mem_simmptr, zextloadi8,
                       addrDefault>, MMRel, LW_FM<0x24>, ISA_MIPS1;
  def LH  : LoadMemory<"lh", GPR32Opnd, mem_simmptr, sextloadi16,
                       addrDefault>, MMRel, LW_FM<0x21>, ISA_MIPS1;
  def LHu : LoadMemory<"lhu", GPR32Opnd, mem_simmptr, zextloadi16>,
            MMRel, LW_FM<0x25>, ISA_MIPS1;
  def LW  : StdMMR6Rel, Load<"lw", GPR32Opnd, load, addrDefault>, MMRel,
            LW_FM<0x23>, ISA_MIPS1;
  def SB  : StdMMR6Rel, Store<"sb", GPR32Opnd, truncstorei8>, MMRel,
            LW_FM<0x28>, ISA_MIPS1;
  def SH  : Store<"sh", GPR32Opnd, truncstorei16>, MMRel, LW_FM<0x29>,
            ISA_MIPS1;
  def SW  : StdMMR6Rel, Store<"sw", GPR32Opnd, store>,
```
- EN: Defines TableGen record `LB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2152-2153
```tablegen
            MMRel, LW_FM<0x2b>, ISA_MIPS1;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2155-2164
```tablegen
/// load/store left/right
let AdditionalPredicates = [NotInMicroMips] in {
def LWL : MMRel, LoadLeftRight<"lwl", MipsLWL, GPR32Opnd>, LW_FM<0x22>,
          ISA_MIPS1_NOT_32R6_64R6;
def LWR : MMRel, LoadLeftRight<"lwr", MipsLWR, GPR32Opnd>, LW_FM<0x26>,
          ISA_MIPS1_NOT_32R6_64R6;
def SWL : MMRel, StoreLeftRight<"swl", MipsSWL, GPR32Opnd>, LW_FM<0x2a>,
          ISA_MIPS1_NOT_32R6_64R6;
def SWR : MMRel, StoreLeftRight<"swr", MipsSWR, GPR32Opnd>, LW_FM<0x2e>,
          ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `LWL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2166-2174
```tablegen
// COP2 Memory Instructions
def LWC2 : StdMMR6Rel, LW_FT2<"lwc2", COP2Opnd, load>, LW_FM<0x32>,
           ISA_MIPS1_NOT_32R6_64R6;
def SWC2 : StdMMR6Rel, SW_FT2<"swc2", COP2Opnd, store>,
           LW_FM<0x3a>, ISA_MIPS1_NOT_32R6_64R6;
def LDC2 : StdMMR6Rel, LW_FT2<"ldc2", COP2Opnd, load>, LW_FM<0x36>,
           ISA_MIPS2_NOT_32R6_64R6;
def SDC2 : StdMMR6Rel, SW_FT2<"sdc2", COP2Opnd, store>,
           LW_FM<0x3e>, ISA_MIPS2_NOT_32R6_64R6;
```
- EN: Defines TableGen record `LWC2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWC2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2176-2193
```tablegen
// COP3 Memory Instructions
let DecoderNamespace = "COP3_" in {
  def LWC3 : LW_FT3<"lwc3", COP3Opnd, load>,
             LW_FM<0x33>,
             ISA_MIPS1_NOT_32R6_64R6_R5900,
             NOT_ASE_CNMIPS;
  def SWC3 : SW_FT3<"swc3", COP3Opnd, store>,
             LW_FM<0x3b>,
             ISA_MIPS1_NOT_32R6_64R6_R5900,
             NOT_ASE_CNMIPS;
  def LDC3 : LW_FT3<"ldc3", COP3Opnd, load>,
             LW_FM<0x37>,
             ISA_MIPS2_NOT_R5900,
             NOT_ASE_CNMIPS;
  def SDC3 : SW_FT3<"sdc3", COP3Opnd, store>,
             LW_FM<0x3f>,
             ISA_MIPS2_NOT_R5900,
             NOT_ASE_CNMIPS;
```
- EN: Defines TableGen record `LWC3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWC3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2194-2194
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2196-2199
```tablegen
  def SYNC : MMRel, StdMMR6Rel, SYNC_FT<"sync">, SYNC_FM, ISA_MIPS2;
  def SYNCI : MMRel, StdMMR6Rel, SYNCI_FT<"synci", mem_simm16>, SYNCI_FM,
              ISA_MIPS32R2;
}
```
- EN: Defines TableGen record `SYNC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SYNC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2201-2213
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def TEQ : MMRel, TEQ_FT<"teq", GPR32Opnd, uimm10>, TEQ_FM<0x34>,
            ISA_MIPS2;
  def TGE : MMRel, TEQ_FT<"tge", GPR32Opnd, uimm10>, TEQ_FM<0x30>,
            ISA_MIPS2;
  def TGEU : MMRel, TEQ_FT<"tgeu", GPR32Opnd, uimm10>, TEQ_FM<0x31>,
             ISA_MIPS2;
  def TLT : MMRel, TEQ_FT<"tlt", GPR32Opnd, uimm10>, TEQ_FM<0x32>,
            ISA_MIPS2;
  def TLTU : MMRel, TEQ_FT<"tltu", GPR32Opnd, uimm10>, TEQ_FM<0x33>,
            ISA_MIPS2;
  def TNE : MMRel, TEQ_FT<"tne", GPR32Opnd, uimm10>, TEQ_FM<0x36>,
            ISA_MIPS2;
```
- EN: Defines TableGen record `TEQ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TEQ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2215-2227
```tablegen
  def TEQI : MMRel, TEQI_FT<"teqi", GPR32Opnd>, TEQI_FM<0xc>,
             ISA_MIPS2_NOT_32R6_64R6;
  def TGEI : MMRel, TEQI_FT<"tgei", GPR32Opnd>, TEQI_FM<0x8>,
             ISA_MIPS2_NOT_32R6_64R6;
  def TGEIU : MMRel, TEQI_FT<"tgeiu", GPR32Opnd>, TEQI_FM<0x9>,
              ISA_MIPS2_NOT_32R6_64R6;
  def TLTI : MMRel, TEQI_FT<"tlti", GPR32Opnd>, TEQI_FM<0xa>,
             ISA_MIPS2_NOT_32R6_64R6;
  def TTLTIU : MMRel, TEQI_FT<"tltiu", GPR32Opnd>, TEQI_FM<0xb>,
               ISA_MIPS2_NOT_32R6_64R6;
  def TNEI : MMRel, TEQI_FT<"tnei", GPR32Opnd>, TEQI_FM<0xe>,
             ISA_MIPS2_NOT_32R6_64R6;
}
```
- EN: Defines TableGen record `TEQI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TEQI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2229-2235
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def BREAK : MMRel, StdMMR6Rel, BRK_FT<"break">, BRK_FM<0xd>, ISA_MIPS1;
  def SYSCALL : MMRel, SYS_FT<"syscall", uimm20>, SYS_FM<0xc>,
                ISA_MIPS1;
  def TRAP : TrapBase<BREAK>, ISA_MIPS1;
  def SDBBP : MMRel, SYS_FT<"sdbbp", uimm20>, SDBBP_FM,
              ISA_MIPS32_NOT_32R6_64R6;
```
- EN: Defines TableGen record `BREAK` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BREAK`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2237-2240
```tablegen
  def ERET : MMRel, ER_FT<"eret">, ER_FM<0x18, 0x0>, INSN_MIPS3_32;
  def ERETNC : MMRel, ER_FT<"eretnc">, ER_FM<0x18, 0x1>,
               ISA_MIPS32R5;
  def DERET : MMRel, ER_FT<"deret">, ER_FM<0x1f, 0x0>, ISA_MIPS32;
```
- EN: Defines TableGen record `ERET` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ERET`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2242-2245
```tablegen
  def EI : MMRel, StdMMR6Rel, DEI_FT<"ei", GPR32Opnd>, EI_FM<1>,
           ISA_MIPS32R2;
  def DI : MMRel, StdMMR6Rel, DEI_FT<"di", GPR32Opnd>, EI_FM<0>,
           ISA_MIPS32R2;
```
- EN: Defines TableGen record `EI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `EI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2247-2248
```tablegen
  def WAIT : MMRel, StdMMR6Rel, WAIT_FT<"wait">, WAIT_FM, INSN_MIPS3_32;
}
```
- EN: Defines TableGen record `WAIT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WAIT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2250-2264
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
/// Load-linked, Store-conditional
def LL : LLBase<"ll", GPR32Opnd>,
         LW_FM<0x30>,
         PTR_32,
         ISA_MIPS2_NOT_32R6_64R6_R5900;
def SC : SCBase<"sc", GPR32Opnd>,
         LW_FM<0x38>,
         PTR_32,
         ISA_MIPS2_NOT_32R6_64R6_R5900;
}
/// Jump and Branch Instructions
let AdditionalPredicates = [NotInMicroMips, RelocNotPIC] in
def J       : MMRel, JumpFJ<jmptarget, "j", br, bb, "j">, FJ<2>,
              IsBranch, ISA_MIPS1;
```
- EN: Defines TableGen record `LL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2266-2283
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
def JR      : MMRel, IndirectBranch<"jr", GPR32Opnd>, MTLO_FM<8>,
              ISA_MIPS1_NOT_32R6_64R6;
def BEQ     : MMRel, CBranch<"beq", brtarget, seteq, GPR32Opnd>, BEQ_FM<4>,
              ISA_MIPS1;
def BEQL    : MMRel, CBranchLikely<"beql", brtarget, GPR32Opnd>,
              BEQ_FM<20>, ISA_MIPS2_NOT_32R6_64R6;
def BNE     : MMRel, CBranch<"bne", brtarget, setne, GPR32Opnd>, BEQ_FM<5>,
              ISA_MIPS1;
def BNEL    : MMRel, CBranchLikely<"bnel", brtarget, GPR32Opnd>,
              BEQ_FM<21>, ISA_MIPS2_NOT_32R6_64R6;
def BGEZ    : MMRel, CBranchZero<"bgez", brtarget, setge, GPR32Opnd>,
              BGEZ_FM<1, 1>, ISA_MIPS1;
def BGEZL   : MMRel, CBranchZeroLikely<"bgezl", brtarget, GPR32Opnd>,
              BGEZ_FM<1, 3>, ISA_MIPS2_NOT_32R6_64R6;
def BGTZ    : MMRel, CBranchZero<"bgtz", brtarget, setgt, GPR32Opnd>,
              BGEZ_FM<7, 0>, ISA_MIPS1;
def BGTZL   : MMRel, CBranchZeroLikely<"bgtzl", brtarget, GPR32Opnd>,
```
- EN: Defines TableGen record `JR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2284-2293
```tablegen
              BGEZ_FM<23, 0>, ISA_MIPS2_NOT_32R6_64R6;
def BLEZ    : MMRel, CBranchZero<"blez", brtarget, setle, GPR32Opnd>,
              BGEZ_FM<6, 0>, ISA_MIPS1;
def BLEZL   : MMRel, CBranchZeroLikely<"blezl", brtarget, GPR32Opnd>,
              BGEZ_FM<22, 0>, ISA_MIPS2_NOT_32R6_64R6;
def BLTZ    : MMRel, CBranchZero<"bltz", brtarget, setlt, GPR32Opnd>,
              BGEZ_FM<1, 0>, ISA_MIPS1;
def BLTZL   : MMRel, CBranchZeroLikely<"bltzl", brtarget, GPR32Opnd>,
              BGEZ_FM<1, 2>, ISA_MIPS2_NOT_32R6_64R6;
def B       : UncondBranch<BEQ, brtarget>, ISA_MIPS1;
```
- EN: Defines TableGen record `BLEZ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BLEZ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2295-2295
```tablegen
def JAL  : MMRel, JumpLink<"jal", calltarget>, FJ<3>, ISA_MIPS1;
```
- EN: Defines TableGen record `JAL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JAL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2297-2297
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2299-2302
```tablegen
let AdditionalPredicates = [NotInMicroMips, NoIndirectJumpGuards] in {
  def JALR : JumpLinkReg<"jalr", GPR32Opnd>, JALR_FM, ISA_MIPS1;
  def JALRPseudo : JumpLinkRegPseudo<GPR32Opnd, JALR, RA>, ISA_MIPS1;
}
```
- EN: Defines TableGen record `JALR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JALR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2304-2321
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def JALX : MMRel, JumpLink<"jalx", calltarget>, FJ<0x1D>,
             ISA_MIPS32_NOT_32R6_64R6;
  def BGEZAL : MMRel, BGEZAL_FT<"bgezal", brtarget, GPR32Opnd>, BGEZAL_FM<0x11>,
               ISA_MIPS1_NOT_32R6_64R6;
  def BGEZALL : MMRel, BGEZAL_FT<"bgezall", brtarget, GPR32Opnd>,
                BGEZAL_FM<0x13>, ISA_MIPS2_NOT_32R6_64R6;
  def BLTZAL : MMRel, BGEZAL_FT<"bltzal", brtarget, GPR32Opnd>, BGEZAL_FM<0x10>,
               ISA_MIPS1_NOT_32R6_64R6;
  def BLTZALL : MMRel, BGEZAL_FT<"bltzall", brtarget, GPR32Opnd>,
                BGEZAL_FM<0x12>, ISA_MIPS2_NOT_32R6_64R6;
  def BAL_BR : BAL_BR_Pseudo<BGEZAL, brtarget>, ISA_MIPS1;
}
let AdditionalPredicates = [NotInMips16Mode, NotInMicroMips] in {
  def TAILCALL : TailCall<J, jmptarget>, ISA_MIPS1;
}
let AdditionalPredicates = [NotInMips16Mode, NotInMicroMips,
                            NoIndirectJumpGuards] in
```
- EN: Defines TableGen record `JALX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JALX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2322-2322
```tablegen
  def TAILCALLREG : TailCallReg<JR, GPR32Opnd>, ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `TAILCALLREG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILCALLREG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2324-2335
```tablegen
// Indirect branches are matched as PseudoIndirectBranch/PseudoIndirectBranch64
// then are expanded to JR, JR64, JALR, or JALR64 depending on the ISA.
class PseudoIndirectBranchBase<Instruction JumpInst, RegisterOperand RO> :
    MipsPseudo<(outs), (ins RO:$rs), [(brind RO:$rs)]>,
    PseudoInstExpansion<(JumpInst RO:$rs)> {
  let isTerminator=1;
  let isBarrier=1;
  let hasDelaySlot = 1;
  let isBranch = 1;
  let isIndirectBranch = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `PseudoIndirectBranchBase` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `PseudoIndirectBranchBase`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2337-2340
```tablegen
let AdditionalPredicates = [NotInMips16Mode, NotInMicroMips,
                            NoIndirectJumpGuards] in
  def PseudoIndirectBranch : PseudoIndirectBranchBase<JR, GPR32Opnd>,
                             ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `PseudoIndirectBranch` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoIndirectBranch`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2342-2356
```tablegen
// Return instructions are matched as a RetRA instruction, then are expanded
// into PseudoReturn/PseudoReturn64 after register allocation. Finally,
// MipsAsmPrinter expands this into JR, JR64, JALR, or JALR64 depending on the
// ISA.
class PseudoReturnBase<RegisterOperand RO> : MipsPseudo<(outs), (ins RO:$rs),
                                                        []> {
  let isTerminator = 1;
  let isBarrier = 1;
  let hasDelaySlot = 1;
  let isReturn = 1;
  let isCodeGenOnly = 1;
  let hasCtrlDep = 1;
  let hasExtraSrcRegAllocReq = 1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `PseudoReturnBase` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `PseudoReturnBase`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2358-2358
```tablegen
def PseudoReturn : PseudoReturnBase<GPR32Opnd>;
```
- EN: Defines TableGen record `PseudoReturn` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoReturn`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2360-2368
```tablegen
// Exception handling related node and instructions.
// The conversion sequence is:
// ISD::EH_RETURN -> MipsISD::EH_RETURN ->
// MIPSeh_return -> (stack change + indirect branch)
//
// MIPSeh_return takes the place of regular return instruction
// but takes two arguments (V1, V0) which are used for storing
// the offset and return address respectively.
def SDT_MipsEHRET : SDTypeProfile<0, 2, [SDTCisInt<0>, SDTCisPtrTy<1>]>;
```
- EN: Defines TableGen record `SDT_MipsEHRET` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_MipsEHRET`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 2370-2372
```tablegen
// Software Exception Return.
def MIPSehret : SDNode<"MipsISD::EH_RETURN", SDT_MipsEHRET,
                      [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
```
- EN: Defines TableGen record `MIPSehret` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MIPSehret`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 2374-2380
```tablegen
let Uses = [V0, V1], isTerminator = 1, isReturn = 1,
           isBarrier = 1, isCTI = 1, hasNoSchedulingInfo = 1 in {
  def MIPSeh_return32 : MipsPseudo<(outs), (ins GPR32:$spoff, GPR32:$dst),
                                   [(MIPSehret GPR32:$spoff, GPR32:$dst)]>;
  def MIPSeh_return64 : MipsPseudo<(outs), (ins GPR64:$spoff, GPR64:$dst),
                                   [(MIPSehret GPR64:$spoff, GPR64:$dst)]>;
}
```
- EN: Defines TableGen record `MIPSeh_return32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MIPSeh_return32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2382-2399
```tablegen
/// Multiply and Divide Instructions.
let AdditionalPredicates = [NotInMicroMips] in {
  def MULT  : MMRel, Mult<"mult", GPR32Opnd, [HI0, LO0]>,
              MULT_FM<0, 0x18>, ISA_MIPS1_NOT_32R6_64R6;
  def MULTu : MMRel, Mult<"multu", GPR32Opnd, [HI0, LO0]>,
              MULT_FM<0, 0x19>, ISA_MIPS1_NOT_32R6_64R6;
  def SDIV  : MMRel, Div<"div", GPR32Opnd, [HI0, LO0]>,
              MULT_FM<0, 0x1a>, ISA_MIPS1_NOT_32R6_64R6;
  def UDIV  : MMRel, Div<"divu", GPR32Opnd, [HI0, LO0]>,
              MULT_FM<0, 0x1b>, ISA_MIPS1_NOT_32R6_64R6;
  def MTHI : MMRel, MoveToLOHI<"mthi", GPR32Opnd, [HI0]>, MTLO_FM<0x11>,
             ISA_MIPS1_NOT_32R6_64R6;
  def MTLO : MMRel, MoveToLOHI<"mtlo", GPR32Opnd, [LO0]>, MTLO_FM<0x13>,
             ISA_MIPS1_NOT_32R6_64R6;
  def MFHI : MMRel, MoveFromLOHI<"mfhi", GPR32Opnd, AC0>, MFLO_FM<0x10>,
             ISA_MIPS1_NOT_32R6_64R6;
  def MFLO : MMRel, MoveFromLOHI<"mflo", GPR32Opnd, AC0>, MFLO_FM<0x12>,
             ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MULT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MULT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2401-2405
```tablegen
  /// Sign Ext In Register Instructions.
  def SEB : MMRel, StdMMR6Rel, SignExtInReg<"seb", i8, GPR32Opnd>,
            SEB_FM<0x10, 0x20>, ISA_MIPS32R2;
  def SEH : MMRel, StdMMR6Rel, SignExtInReg<"seh", i16, GPR32Opnd>,
            SEB_FM<0x18, 0x20>, ISA_MIPS32R2;
```
- EN: Defines TableGen record `SEB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SEB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2407-2411
```tablegen
  /// Count Leading
  def CLZ : MMRel, CountLeading0<"clz", GPR32Opnd>, CLO_FM<0x20>,
            ISA_MIPS32_NOT_32R6_64R6;
  def CLO : MMRel, CountLeading1<"clo", GPR32Opnd>, CLO_FM<0x21>,
            ISA_MIPS32_NOT_32R6_64R6;
```
- EN: Defines TableGen record `CLZ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CLZ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2413-2415
```tablegen
  /// Word Swap Bytes Within Halfwords
  def WSBH : MMRel, SubwordSwap<"wsbh", GPR32Opnd>, SEB_FM<2, 0x20>,
             ISA_MIPS32R2;
```
- EN: Defines TableGen record `WSBH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WSBH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2417-2419
```tablegen
  /// No operation.
  def NOP : PseudoSE<(outs), (ins), []>,
                     PseudoInstExpansion<(SLL ZERO, ZERO, 0)>, ISA_MIPS1;
```
- EN: Defines TableGen record `NOP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NOP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2421-2427
```tablegen
  // FrameIndexes are legalized when they are operands from load/store
  // instructions. The same not happens for stack address copies, so an
  // add op with mem ComplexPattern is used and the stack address copy
  // can be matched. It's similar to Sparc LEA_ADDRi
  let AdditionalPredicates = [NotInMicroMips] in
    def LEA_ADDiu : MMRel, EffectiveAddress<"addiu", GPR32Opnd>, LW_FM<9>,
                    ISA_MIPS1;
```
- EN: Defines TableGen record `LEA_ADDiu` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LEA_ADDiu`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2429-2438
```tablegen
  // MADD*/MSUB*
  def MADD  : MMRel, MArithR<"madd", 1>, MULT_FM<0x1c, 0>,
              ISA_MIPS32_NOT_32R6_64R6;
  def MADDU : MMRel, MArithR<"maddu", 1>, MULT_FM<0x1c, 1>,
              ISA_MIPS32_NOT_32R6_64R6;
  def MSUB  : MMRel, MArithR<"msub">, MULT_FM<0x1c, 4>,
              ISA_MIPS32_NOT_32R6_64R6;
  def MSUBU : MMRel, MArithR<"msubu">, MULT_FM<0x1c, 5>,
              ISA_MIPS32_NOT_32R6_64R6;
}
```
- EN: Defines TableGen record `MADD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MADD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2440-2456
```tablegen
let AdditionalPredicates = [NotDSP] in {
def PseudoMULT  : MultDivPseudo<MULT, ACC64, GPR32Opnd, MipsMult>,
                  ISA_MIPS1_NOT_32R6_64R6;
def PseudoMULTu : MultDivPseudo<MULTu, ACC64, GPR32Opnd, MipsMultu>,
                  ISA_MIPS1_NOT_32R6_64R6;
def PseudoMFHI : PseudoMFLOHI<GPR32, ACC64, MipsMFHI>, ISA_MIPS1_NOT_32R6_64R6;
def PseudoMFLO : PseudoMFLOHI<GPR32, ACC64, MipsMFLO>, ISA_MIPS1_NOT_32R6_64R6;
def PseudoMTLOHI : PseudoMTLOHI<ACC64, GPR32>, ISA_MIPS1_NOT_32R6_64R6;
def PseudoMADD  : MAddSubPseudo<MADD, MipsMAdd>,
                  ISA_MIPS32_NOT_32R6_64R6;
def PseudoMADDU : MAddSubPseudo<MADDU, MipsMAddu>,
                  ISA_MIPS32_NOT_32R6_64R6;
def PseudoMSUB  : MAddSubPseudo<MSUB, MipsMSub>,
                  ISA_MIPS32_NOT_32R6_64R6;
def PseudoMSUBU : MAddSubPseudo<MSUBU, MipsMSubu>,
                  ISA_MIPS32_NOT_32R6_64R6;
}
```
- EN: Defines TableGen record `PseudoMULT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoMULT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2458-2475
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def PseudoSDIV : MultDivPseudo<SDIV, ACC64, GPR32Opnd, MipsDivRem,
                                 0, 1, 1>, ISA_MIPS1_NOT_32R6_64R6;
  def PseudoUDIV : MultDivPseudo<UDIV, ACC64, GPR32Opnd, MipsDivRemU,
                                 0, 1, 1>, ISA_MIPS1_NOT_32R6_64R6;
  def RDHWR : MMRel, ReadHardware<GPR32Opnd, HWRegsOpnd>, RDHWR_FM, ISA_MIPS1;
  // TODO: Add '0 < pos+size <= 32' constraint check to ext instruction
  def EXT : MMRel, StdMMR6Rel, ExtBase<"ext", GPR32Opnd, uimm5, uimm5_plus1,
                                       immZExt5, immZExt5Plus1, MipsExt>,
            EXT_FM<0>, ISA_MIPS32R2;
  def INS : MMRel, StdMMR6Rel, InsBase<"ins", GPR32Opnd, uimm5,
                                       uimm5_inssize_plus1, immZExt5,
                                       immZExt5Plus1>,
            EXT_FM<4>, ISA_MIPS32R2;
}
/// Move Control Registers From/To CPU Registers
let AdditionalPredicates = [NotInMicroMips] in {
  def MTC0 : MTC3OP<"mtc0", COP0Opnd, GPR32Opnd>,
```
- EN: Defines TableGen record `PseudoSDIV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoSDIV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2476-2483
```tablegen
             MFC3OP_FM<0x10, 4, 0>, ISA_MIPS1;
  def MFC0 : MFC3OP<"mfc0", GPR32Opnd, COP0Opnd>,
             MFC3OP_FM<0x10, 0, 0>, ISA_MIPS1;
  def MFC2 : MFC3OP<"mfc2", GPR32Opnd, COP2Opnd>,
             MFC3OP_FM<0x12, 0, 0>, ISA_MIPS1;
  def MTC2 : MTC3OP<"mtc2", COP2Opnd, GPR32Opnd>,
             MFC3OP_FM<0x12, 4, 0>, ISA_MIPS1;
}
```
- EN: Defines TableGen record `MFC0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFC0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2485-2490
```tablegen
class Barrier<string asmstr> :
  InstSE<(outs), (ins), asmstr, [], FrmOther, asmstr>;
let AdditionalPredicates = [NotInMicroMips] in {
  def SSNOP : MMRel, StdMMR6Rel, Barrier<"ssnop">, BARRIER_FM<1>,
              ISA_MIPS1;
  def EHB : MMRel, Barrier<"ehb">, BARRIER_FM<3>, ISA_MIPS1;
```
- EN: Declares reusable TableGen class `Barrier` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `Barrier`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2492-2495
```tablegen
  let isCTI = 1 in
  def PAUSE : MMRel, StdMMR6Rel, Barrier<"pause">, BARRIER_FM<5>,
              ISA_MIPS32R2;
}
```
- EN: Defines TableGen record `PAUSE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PAUSE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2497-2507
```tablegen
// JR_HB and JALR_HB are defined here using the new style naming
// scheme because some of this code is shared with Mips32r6InstrInfo.td
// and because of that it doesn't follow the naming convention of the
// rest of the file. To avoid a mixture of old vs new style, the new
// style was chosen.
class JR_HB_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> {
  dag OutOperandList = (outs);
  dag InOperandList = (ins GPROpnd:$rs);
  string AsmString = !strconcat(instr_asm, "\t$rs");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `JR_HB_DESC_BASE` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `JR_HB_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2509-2514
```tablegen
class JALR_HB_DESC_BASE<string instr_asm, RegisterOperand GPROpnd> {
  dag OutOperandList = (outs GPROpnd:$rd);
  dag InOperandList = (ins GPROpnd:$rs);
  string AsmString = !strconcat(instr_asm, "\t$rd, $rs");
  list<dag> Pattern = [];
}
```
- EN: Declares reusable TableGen class `JALR_HB_DESC_BASE` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `JALR_HB_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2516-2524
```tablegen
class JR_HB_DESC<RegisterOperand RO> :
  InstSE<(outs), (ins), "", [], FrmJ>, JR_HB_DESC_BASE<"jr.hb", RO> {
  let isBranch=1;
  let isIndirectBranch=1;
  let hasDelaySlot=1;
  let isTerminator=1;
  let isBarrier=1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `JR_HB_DESC` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `JR_HB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2526-2532
```tablegen
class JALR_HB_DESC<RegisterOperand RO> :
  InstSE<(outs), (ins), "", [], FrmJ>, JALR_HB_DESC_BASE<"jalr.hb",
                                                                     RO> {
  let isIndirectBranch=1;
  let hasDelaySlot=1;
  bit isCTI = 1;
}
```
- EN: Declares reusable TableGen class `JALR_HB_DESC` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `JALR_HB_DESC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2534-2535
```tablegen
class JR_HB_ENC : JR_HB_FM<8>;
class JALR_HB_ENC : JALR_HB_FM<9>;
```
- EN: Declares reusable TableGen class `JR_HB_ENC` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `JR_HB_ENC`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2537-2538
```tablegen
def JR_HB : JR_HB_DESC<GPR32Opnd>, JR_HB_ENC, ISA_MIPS32R2_NOT_32R6_64R6;
def JALR_HB : JALR_HB_DESC<GPR32Opnd>, JALR_HB_ENC, ISA_MIPS32;
```
- EN: Defines TableGen record `JR_HB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JR_HB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2540-2541
```tablegen
let AdditionalPredicates = [NotInMicroMips, UseIndirectJumpsHazard] in
  def JALRHBPseudo : JumpLinkRegPseudo<GPR32Opnd, JALR_HB, RA>;
```
- EN: Defines TableGen record `JALRHBPseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JALRHBPseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2544-2549
```tablegen
let AdditionalPredicates = [NotInMips16Mode, NotInMicroMips,
                            UseIndirectJumpsHazard] in {
  def TAILCALLREGHB : TailCallReg<JR_HB, GPR32Opnd>, ISA_MIPS32_NOT_32R6_64R6;
  def PseudoIndirectHazardBranch : PseudoIndirectBranchBase<JR_HB, GPR32Opnd>,
                                   ISA_MIPS32R2_NOT_32R6_64R6;
}
```
- EN: Defines TableGen record `TAILCALLREGHB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILCALLREGHB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2551-2564
```tablegen
class TLB<string asmstr> :
  InstSE<(outs), (ins), asmstr, [], FrmOther, asmstr>;
let AdditionalPredicates = [NotInMicroMips] in {
  def TLBP : MMRel, TLB<"tlbp">, COP0_TLB_FM<0x08>, ISA_MIPS1;
  def TLBR : MMRel, TLB<"tlbr">, COP0_TLB_FM<0x01>, ISA_MIPS1;
  def TLBWI : MMRel, TLB<"tlbwi">, COP0_TLB_FM<0x02>, ISA_MIPS1;
  def TLBWR : MMRel, TLB<"tlbwr">, COP0_TLB_FM<0x06>, ISA_MIPS1;
}
class CacheOp<string instr_asm, Operand MemOpnd> :
    InstSE<(outs), (ins  MemOpnd:$addr, uimm5:$hint),
           !strconcat(instr_asm, "\t$hint, $addr"), [], FrmOther,
           instr_asm> {
  let DecoderMethod = "DecodeCacheOp";
}
```
- EN: Declares reusable TableGen class `TLB` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `TLB`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2566-2582
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def CACHE : MMRel, CacheOp<"cache", mem>, CACHEOP_FM<0b101111>,
              INSN_MIPS3_32_NOT_32R6_64R6;
  def PREF :  MMRel, CacheOp<"pref", mem>, CACHEOP_FM<0b110011>,
              INSN_MIPS3_32_NOT_32R6_64R6;
}
// FIXME: We are missing the prefx instruction.
def ROL : MipsAsmPseudoInst<(outs),
                            (ins GPR32Opnd:$rs, GPR32Opnd:$rt, GPR32Opnd:$rd),
                            "rol\t$rs, $rt, $rd">;
def ROLImm : MipsAsmPseudoInst<(outs),
                               (ins GPR32Opnd:$rs, GPR32Opnd:$rt, simm16:$imm),
                               "rol\t$rs, $rt, $imm">;
def : MipsInstAlias<"rol $rd, $rs",
                    (ROL GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rs), 0>;
def : MipsInstAlias<"rol $rd, $imm",
                    (ROLImm GPR32Opnd:$rd, GPR32Opnd:$rd, simm16:$imm), 0>;
```
- EN: Defines TableGen record `CACHE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CACHE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2584-2593
```tablegen
def ROR : MipsAsmPseudoInst<(outs),
                            (ins GPR32Opnd:$rs, GPR32Opnd:$rt, GPR32Opnd:$rd),
                            "ror\t$rs, $rt, $rd">;
def RORImm : MipsAsmPseudoInst<(outs),
                               (ins GPR32Opnd:$rs, GPR32Opnd:$rt, simm16:$imm),
                               "ror\t$rs, $rt, $imm">;
def : MipsInstAlias<"ror $rd, $rs",
                    (ROR GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rs), 0>;
def : MipsInstAlias<"ror $rd, $imm",
                    (RORImm GPR32Opnd:$rd, GPR32Opnd:$rd, simm16:$imm), 0>;
```
- EN: Defines TableGen record `ROR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ROR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2595-2606
```tablegen
def DROL : MipsAsmPseudoInst<(outs),
                             (ins GPR32Opnd:$rs, GPR32Opnd:$rt, GPR32Opnd:$rd),
                             "drol\t$rs, $rt, $rd">, ISA_MIPS64;
def DROLImm : MipsAsmPseudoInst<(outs),
                                (ins GPR32Opnd:$rs, GPR32Opnd:$rt, simm16:$imm),
                                "drol\t$rs, $rt, $imm">, ISA_MIPS64;
def : MipsInstAlias<"drol $rd, $rs",
                    (DROL GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rs), 0>,
      ISA_MIPS64;
def : MipsInstAlias<"drol $rd, $imm",
                    (DROLImm GPR32Opnd:$rd, GPR32Opnd:$rd, simm16:$imm), 0>,
      ISA_MIPS64;
```
- EN: Defines TableGen record `DROL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DROL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2608-2619
```tablegen
def DROR : MipsAsmPseudoInst<(outs),
                             (ins GPR32Opnd:$rs, GPR32Opnd:$rt, GPR32Opnd:$rd),
                             "dror\t$rs, $rt, $rd">, ISA_MIPS64;
def DRORImm : MipsAsmPseudoInst<(outs),
                                (ins GPR32Opnd:$rs, GPR32Opnd:$rt, simm16:$imm),
                                "dror\t$rs, $rt, $imm">, ISA_MIPS64;
def : MipsInstAlias<"dror $rd, $rs",
                    (DROR GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rs), 0>,
      ISA_MIPS64;
def : MipsInstAlias<"dror $rd, $imm",
                    (DRORImm GPR32Opnd:$rd, GPR32Opnd:$rd, simm16:$imm), 0>,
      ISA_MIPS64;
```
- EN: Defines TableGen record `DROR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DROR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2621-2622
```tablegen
def ABSMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd), (ins GPR32Opnd:$rs),
                                 "abs\t$rd, $rs">;
```
- EN: Defines TableGen record `ABSMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ABSMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2624-2626
```tablegen
def SEQMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                 (ins GPR32Opnd:$rs, GPR32Opnd:$rt),
                                 "seq $rd, $rs, $rt">, NOT_ASE_CNMIPS;
```
- EN: Defines TableGen record `SEQMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SEQMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2628-2630
```tablegen
def : MipsInstAlias<"seq $rd, $rs",
                    (SEQMacro GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rs), 0>,
                    NOT_ASE_CNMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2632-2634
```tablegen
def SEQIMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                  (ins GPR32Opnd:$rs, simm32_relaxed:$imm),
                                  "seq $rd, $rs, $imm">, NOT_ASE_CNMIPS;
```
- EN: Defines TableGen record `SEQIMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SEQIMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2636-2638
```tablegen
def : MipsInstAlias<"seq $rd, $imm",
                    (SEQIMacro GPR32Opnd:$rd, GPR32Opnd:$rd, simm32:$imm), 0>,
                    NOT_ASE_CNMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2640-2642
```tablegen
def SNEMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                 (ins GPR32Opnd:$rs, GPR32Opnd:$rt),
                                 "sne $rd, $rs, $rt">, NOT_ASE_CNMIPS;
```
- EN: Defines TableGen record `SNEMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SNEMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2644-2646
```tablegen
def : MipsInstAlias<"sne $rd, $rs",
                    (SNEMacro GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rs), 0>,
                    NOT_ASE_CNMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2648-2650
```tablegen
def SNEIMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                  (ins GPR32Opnd:$rs, simm32_relaxed:$imm),
                                  "sne $rd, $rs, $imm">, NOT_ASE_CNMIPS;
```
- EN: Defines TableGen record `SNEIMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SNEIMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2652-2654
```tablegen
def : MipsInstAlias<"sne $rd, $imm",
                    (SNEIMacro GPR32Opnd:$rd, GPR32Opnd:$rd, simm32:$imm), 0>,
                    NOT_ASE_CNMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2656-2667
```tablegen
def MULImmMacro : MipsAsmPseudoInst<(outs), (ins GPR32Opnd:$rd, GPR32Opnd:$rs,
                                                 simm32_relaxed:$imm),
                                    "mul\t$rd, $rs, $imm">,
                  ISA_MIPS1_NOT_32R6_64R6;
def MULOMacro : MipsAsmPseudoInst<(outs), (ins GPR32Opnd:$rd, GPR32Opnd:$rs,
                                               GPR32Opnd:$rt),
                                  "mulo\t$rd, $rs, $rt">,
                ISA_MIPS1_NOT_32R6_64R6;
def MULOUMacro : MipsAsmPseudoInst<(outs), (ins GPR32Opnd:$rd, GPR32Opnd:$rs,
                                                GPR32Opnd:$rt),
                                   "mulou\t$rd, $rs, $rt">,
                 ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MULImmMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MULImmMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2669-2674
```tablegen
// Virtualization ASE
class HYPCALL_FT<string opstr> :
  InstSE<(outs), (ins uimm10:$code_),
         !strconcat(opstr, "\t$code_"), [], FrmOther, opstr> {
  let BaseOpcode = opstr;
}
```
- EN: Declares reusable TableGen class `HYPCALL_FT` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `HYPCALL_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2676-2693
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def MFGC0    : MMRel, MFC3OP<"mfgc0", GPR32Opnd, COP0Opnd>,
                 MFC3OP_FM<0x10, 3, 0>, ISA_MIPS32R5, ASE_VIRT;
  def MTGC0    : MMRel, MTC3OP<"mtgc0", COP0Opnd, GPR32Opnd>,
                 MFC3OP_FM<0x10, 3, 2>, ISA_MIPS32R5, ASE_VIRT;
  def MFHGC0   : MMRel, MFC3OP<"mfhgc0", GPR32Opnd, COP0Opnd>,
                 MFC3OP_FM<0x10, 3, 4>, ISA_MIPS32R5, ASE_VIRT;
  def MTHGC0   : MMRel, MTC3OP<"mthgc0", COP0Opnd, GPR32Opnd>,
                 MFC3OP_FM<0x10, 3, 6>, ISA_MIPS32R5, ASE_VIRT;
  def TLBGINV  : MMRel, TLB<"tlbginv">, COP0_TLB_FM<0b001011>,
                 ISA_MIPS32R5, ASE_VIRT;
  def TLBGINVF : MMRel, TLB<"tlbginvf">, COP0_TLB_FM<0b001100>,
                 ISA_MIPS32R5, ASE_VIRT;
  def TLBGP    : MMRel, TLB<"tlbgp">, COP0_TLB_FM<0b010000>,
                 ISA_MIPS32R5, ASE_VIRT;
  def TLBGR    : MMRel, TLB<"tlbgr">, COP0_TLB_FM<0b001001>,
                 ISA_MIPS32R5, ASE_VIRT;
  def TLBGWI   : MMRel, TLB<"tlbgwi">, COP0_TLB_FM<0b001010>,
```
- EN: Defines TableGen record `MFGC0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFGC0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2694-2699
```tablegen
                 ISA_MIPS32R5, ASE_VIRT;
  def TLBGWR   : MMRel, TLB<"tlbgwr">, COP0_TLB_FM<0b001110>,
                 ISA_MIPS32R5, ASE_VIRT;
  def HYPCALL  : MMRel, HYPCALL_FT<"hypcall">,
                 HYPCALL_FM<0b101000>, ISA_MIPS32R5, ASE_VIRT;
}
```
- EN: Defines TableGen record `TLBGWR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TLBGWR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2701-2703
```tablegen
//===----------------------------------------------------------------------===//
// Instruction aliases
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2705-2717
```tablegen
multiclass OneOrTwoOperandMacroImmediateAlias<string Memnomic,
                                              Instruction Opcode,
                                              RegisterOperand RO = GPR32Opnd,
                                              Operand Imm = simm32_relaxed> {
  def : MipsInstAlias<!strconcat(Memnomic, " $rs, $rt, $imm"),
                                (Opcode RO:$rs,
                                        RO:$rt,
                                        Imm:$imm), 0>;
  def : MipsInstAlias<!strconcat(Memnomic, " $rs, $imm"),
                                (Opcode RO:$rs,
                                        RO:$rs,
                                        Imm:$imm), 0>;
}
```
- EN: Declares TableGen `multiclass OneOrTwoOperandMacroImmediateAlias`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass OneOrTwoOperandMacroImmediateAlias`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2719-2725
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsInstAlias<"move $dst, $src",
                      (OR GPR32Opnd:$dst, GPR32Opnd:$src, ZERO), 1>,
        GPR_32, ISA_MIPS1;
  def : MipsInstAlias<"move $dst, $src",
                      (ADDu GPR32Opnd:$dst, GPR32Opnd:$src, ZERO), 1>,
        GPR_32, ISA_MIPS1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2727-2728
```tablegen
  def : MipsInstAlias<"bal $offset", (BGEZAL ZERO, brtarget:$offset), 1>,
        ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2730-2730
```tablegen
  def : MipsInstAlias<"j $rs", (JR GPR32Opnd:$rs), 0>, ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2732-2732
```tablegen
  def : MipsInstAlias<"jalr $rs", (JALR RA, GPR32Opnd:$rs), 0>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2734-2735
```tablegen
  def : MipsInstAlias<"jalr.hb $rs", (JALR_HB RA, GPR32Opnd:$rs), 1>,
        ISA_MIPS32;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2737-2744
```tablegen
  def : MipsInstAlias<"neg $rt, $rs",
                      (SUB GPR32Opnd:$rt, ZERO, GPR32Opnd:$rs), 1>, ISA_MIPS1;
  def : MipsInstAlias<"neg $rt",
                      (SUB GPR32Opnd:$rt, ZERO, GPR32Opnd:$rt), 1>, ISA_MIPS1;
  def : MipsInstAlias<"negu $rt, $rs",
                      (SUBu GPR32Opnd:$rt, ZERO, GPR32Opnd:$rs), 1>, ISA_MIPS1;
  def : MipsInstAlias<"negu $rt",
                      (SUBu GPR32Opnd:$rt, ZERO, GPR32Opnd:$rt), 1>, ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2746-2758
```tablegen
  def SGE : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                              (ins GPR32Opnd:$rs, GPR32Opnd:$rt),
                              "sge\t$rd, $rs, $rt">, ISA_MIPS1;
  def : MipsInstAlias<"sge $rs, $rt",
                      (SGE GPR32Opnd:$rs, GPR32Opnd:$rs, GPR32Opnd:$rt), 0>,
        ISA_MIPS1;
  def SGEImm : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                 (ins GPR32Opnd:$rs, simm32:$imm),
                                 "sge\t$rd, $rs, $imm">, GPR_32;
  def : MipsInstAlias<"sge $rs, $imm", (SGEImm GPR32Opnd:$rs,
                                               GPR32Opnd:$rs,
                                               simm32:$imm), 0>,
        GPR_32;
```
- EN: Defines TableGen record `SGE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SGE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2760-2772
```tablegen
  def SGEU : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                               (ins GPR32Opnd:$rs, GPR32Opnd:$rt),
                               "sgeu\t$rd, $rs, $rt">, ISA_MIPS1;
  def : MipsInstAlias<"sgeu $rs, $rt",
                      (SGEU GPR32Opnd:$rs, GPR32Opnd:$rs, GPR32Opnd:$rt), 0>,
        ISA_MIPS1;
  def SGEUImm : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                  (ins GPR32Opnd:$rs, uimm32_coerced:$imm),
                                  "sgeu\t$rd, $rs, $imm">, GPR_32;
  def : MipsInstAlias<"sgeu $rs, $imm", (SGEUImm GPR32Opnd:$rs,
                                                 GPR32Opnd:$rs,
                                                 uimm32_coerced:$imm), 0>,
        GPR_32;
```
- EN: Defines TableGen record `SGEU` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SGEU`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2774-2779
```tablegen
  def : MipsInstAlias<
          "sgt $rd, $rs, $rt",
          (SLT GPR32Opnd:$rd, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>, ISA_MIPS1;
  def : MipsInstAlias<
          "sgt $rs, $rt",
          (SLT GPR32Opnd:$rs, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>, ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2781-2793
```tablegen
  def SGTImm : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                 (ins GPR32Opnd:$rs, simm32:$imm),
                                 "sgt\t$rd, $rs, $imm">, GPR_32;
  def : MipsInstAlias<"sgt $rs, $imm", (SGTImm GPR32Opnd:$rs,
                                               GPR32Opnd:$rs,
                                               simm32:$imm), 0>,
        GPR_32;
  def : MipsInstAlias<
          "sgtu $rd, $rs, $rt",
          (SLTu GPR32Opnd:$rd, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>, ISA_MIPS1;
  def : MipsInstAlias<
          "sgtu $$rs, $rt",
          (SLTu GPR32Opnd:$rs, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>, ISA_MIPS1;
```
- EN: Defines TableGen record `SGTImm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SGTImm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2795-2801
```tablegen
  def SGTUImm : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                  (ins GPR32Opnd:$rs, uimm32_coerced:$imm),
                                  "sgtu\t$rd, $rs, $imm">, GPR_32;
  def : MipsInstAlias<"sgtu $rs, $imm", (SGTUImm GPR32Opnd:$rs,
                                                 GPR32Opnd:$rs,
                                                 uimm32_coerced:$imm), 0>,
        GPR_32;
```
- EN: Defines TableGen record `SGTUImm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SGTUImm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2803-2815
```tablegen
  def SLE : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                              (ins GPR32Opnd:$rs, GPR32Opnd:$rt),
                              "sle\t$rd, $rs, $rt">, ISA_MIPS1;
  def : MipsInstAlias<"sle $rs, $rt",
                      (SLE GPR32Opnd:$rs, GPR32Opnd:$rs, GPR32Opnd:$rt), 0>,
        ISA_MIPS1;
  def SLEImm : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                 (ins GPR32Opnd:$rs, simm32:$imm),
                                 "sle\t$rd, $rs, $imm">, GPR_32;
  def : MipsInstAlias<"sle $rs, $imm", (SLEImm GPR32Opnd:$rs,
                                               GPR32Opnd:$rs,
                                               simm32:$imm), 0>,
        GPR_32;
```
- EN: Defines TableGen record `SLE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SLE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2817-2829
```tablegen
  def SLEU : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                               (ins GPR32Opnd:$rs, GPR32Opnd:$rt),
                               "sleu\t$rd, $rs, $rt">, ISA_MIPS1;
  def : MipsInstAlias<"sleu $rs, $rt",
                      (SLEU GPR32Opnd:$rs, GPR32Opnd:$rs, GPR32Opnd:$rt), 0>,
        ISA_MIPS1;
  def SLEUImm : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                  (ins GPR32Opnd:$rs, uimm32_coerced:$imm),
                                  "sleu\t$rd, $rs, $imm">, GPR_32;
  def : MipsInstAlias<"sleu $rs, $imm", (SLEUImm GPR32Opnd:$rs,
                                                 GPR32Opnd:$rs,
                                                 uimm32_coerced:$imm), 0>,
        GPR_32;
```
- EN: Defines TableGen record `SLEU` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SLEU`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2831-2836
```tablegen
  def : MipsInstAlias<
          "not $rt, $rs",
          (NOR GPR32Opnd:$rt, GPR32Opnd:$rs, ZERO), 0>, ISA_MIPS1;
  def : MipsInstAlias<
          "not $rt",
          (NOR GPR32Opnd:$rt, GPR32Opnd:$rt, ZERO), 0>, ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2838-2838
```tablegen
  def : MipsInstAlias<"nop", (SLL ZERO, ZERO, 0), 1>, ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2840-2841
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"add", ADDi>,
         ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2843-2843
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"addu", ADDiu>, ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2845-2845
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"and", ANDi>, ISA_MIPS1, GPR_32;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2847-2847
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"or", ORi>, ISA_MIPS1, GPR_32;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2849-2849
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"xor", XORi>, ISA_MIPS1, GPR_32;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2851-2851
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"slt", SLTi>, ISA_MIPS1, GPR_32;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2853-2853
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"sltu", SLTiu>, ISA_MIPS1, GPR_32;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2855-2872
```tablegen
  def : MipsInstAlias<"mfgc0 $rt, $rd",
                      (MFGC0 GPR32Opnd:$rt, COP0Opnd:$rd, 0), 0>,
                      ISA_MIPS32R5, ASE_VIRT;
  def : MipsInstAlias<"mtgc0 $rt, $rd",
                      (MTGC0 COP0Opnd:$rd, GPR32Opnd:$rt, 0), 0>,
                      ISA_MIPS32R5, ASE_VIRT;
  def : MipsInstAlias<"mfhgc0 $rt, $rd",
                      (MFHGC0 GPR32Opnd:$rt, COP0Opnd:$rd, 0), 0>,
                      ISA_MIPS32R5, ASE_VIRT;
  def : MipsInstAlias<"mthgc0 $rt, $rd",
                      (MTHGC0 COP0Opnd:$rd, GPR32Opnd:$rt, 0), 0>,
                      ISA_MIPS32R5, ASE_VIRT;
  def : MipsInstAlias<"mfc0 $rt, $rd", (MFC0 GPR32Opnd:$rt, COP0Opnd:$rd, 0), 0>,
        ISA_MIPS1;
  def : MipsInstAlias<"mtc0 $rt, $rd", (MTC0 COP0Opnd:$rd, GPR32Opnd:$rt, 0), 0>,
        ISA_MIPS1;
  def : MipsInstAlias<"mfc2 $rt, $rd", (MFC2 GPR32Opnd:$rt, COP2Opnd:$rd, 0), 0>,
        ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2873-2874
```tablegen
  def : MipsInstAlias<"mtc2 $rt, $rd", (MTC2 COP2Opnd:$rd, GPR32Opnd:$rt, 0), 0>,
        ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2876-2877
```tablegen
  def : MipsInstAlias<"b $offset", (BEQ ZERO, ZERO, brtarget:$offset), 0>,
        ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2879-2890
```tablegen
  def : MipsInstAlias<"bnez $rs,$offset",
                      (BNE GPR32Opnd:$rs, ZERO, brtarget:$offset), 0>,
        ISA_MIPS1;
  def : MipsInstAlias<"bnezl $rs, $offset",
                      (BNEL GPR32Opnd:$rs, ZERO, brtarget:$offset), 1>,
        ISA_MIPS2;
  def : MipsInstAlias<"beqz $rs,$offset",
                      (BEQ GPR32Opnd:$rs, ZERO, brtarget:$offset), 0>,
        ISA_MIPS1;
  def : MipsInstAlias<"beqzl $rs, $offset",
                      (BEQL GPR32Opnd:$rs, ZERO, brtarget:$offset), 1>,
        ISA_MIPS2;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2892-2892
```tablegen
  def : MipsInstAlias<"syscall", (SYSCALL 0), 1>, ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2894-2897
```tablegen
  def : MipsInstAlias<"break", (BREAK 0, 0), 1>, ISA_MIPS1;
  def : MipsInstAlias<"break $imm", (BREAK uimm10:$imm, 0), 1>, ISA_MIPS1;
  def : MipsInstAlias<"ei", (EI ZERO), 1>, ISA_MIPS32R2;
  def : MipsInstAlias<"di", (DI ZERO), 1>, ISA_MIPS32R2;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2899-2912
```tablegen
  def : MipsInstAlias<"teq $rs, $rt",
                      (TEQ GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>, ISA_MIPS2;
  def : MipsInstAlias<"tge $rs, $rt",
                      (TGE GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>, ISA_MIPS2;
  def : MipsInstAlias<"tgeu $rs, $rt",
                      (TGEU GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>, ISA_MIPS2;
  def : MipsInstAlias<"tlt $rs, $rt",
                      (TLT GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>, ISA_MIPS2;
  def : MipsInstAlias<"tltu $rs, $rt",
                      (TLTU GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>, ISA_MIPS2;
  def : MipsInstAlias<"tne $rs, $rt",
                      (TNE GPR32Opnd:$rs, GPR32Opnd:$rt, 0), 1>, ISA_MIPS2;
  def : MipsInstAlias<"rdhwr $rt, $rs",
                      (RDHWR GPR32Opnd:$rt, HWRegsOpnd:$rs, 0), 1>, ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2914-2931
```tablegen
}
def : MipsInstAlias<"sub, $rd, $rs, $imm",
                    (ADDi GPR32Opnd:$rd, GPR32Opnd:$rs,
                          InvertedImOperand:$imm), 0>, ISA_MIPS1_NOT_32R6_64R6;
def : MipsInstAlias<"sub $rs, $imm",
                    (ADDi GPR32Opnd:$rs, GPR32Opnd:$rs, InvertedImOperand:$imm),
                    0>, ISA_MIPS1_NOT_32R6_64R6;
def : MipsInstAlias<"subu, $rd, $rs, $imm",
                    (ADDiu GPR32Opnd:$rd, GPR32Opnd:$rs,
                           InvertedImOperand:$imm), 0>;
def : MipsInstAlias<"subu $rs, $imm", (ADDiu GPR32Opnd:$rs, GPR32Opnd:$rs,
                                             InvertedImOperand:$imm), 0>;
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsInstAlias<"sll $rd, $rt, $rs",
                      (SLLV GPR32Opnd:$rd, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>;
  def : MipsInstAlias<"sra $rd, $rt, $rs",
                      (SRAV GPR32Opnd:$rd, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>;
  def : MipsInstAlias<"srl $rd, $rt, $rs",
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2932-2946
```tablegen
                      (SRLV GPR32Opnd:$rd, GPR32Opnd:$rt, GPR32Opnd:$rs), 0>;
  def : MipsInstAlias<"sll $rd, $rt",
                      (SLLV GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rt), 0>;
  def : MipsInstAlias<"sra $rd, $rt",
                      (SRAV GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rt), 0>;
  def : MipsInstAlias<"srl $rd, $rt",
                      (SRLV GPR32Opnd:$rd, GPR32Opnd:$rd, GPR32Opnd:$rt), 0>;
  def : MipsInstAlias<"seh $rd", (SEH GPR32Opnd:$rd, GPR32Opnd:$rd), 0>,
                     ISA_MIPS32R2;
  def : MipsInstAlias<"seb $rd", (SEB GPR32Opnd:$rd, GPR32Opnd:$rd), 0>,
                     ISA_MIPS32R2;
}
def : MipsInstAlias<"sdbbp", (SDBBP 0)>, ISA_MIPS32_NOT_32R6_64R6;
let AdditionalPredicates = [NotInMicroMips] in
  def : MipsInstAlias<"sync", (SYNC 0), 1>, ISA_MIPS2;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2948-2953
```tablegen
def : MipsInstAlias<"mulo $rs, $rt",
                    (MULOMacro GPR32Opnd:$rs, GPR32Opnd:$rs, GPR32Opnd:$rt), 0>,
                    ISA_MIPS1_NOT_32R6_64R6;
def : MipsInstAlias<"mulou $rs, $rt",
                    (MULOUMacro GPR32Opnd:$rs, GPR32Opnd:$rs, GPR32Opnd:$rt), 0>,
                    ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2955-2956
```tablegen
let AdditionalPredicates = [NotInMicroMips] in
  def : MipsInstAlias<"hypcall", (HYPCALL 0), 1>, ISA_MIPS32R5, ASE_VIRT;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2958-2960
```tablegen
//===----------------------------------------------------------------------===//
// Assembler Pseudo Instructions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2962-2967
```tablegen
// We use uimm32_coerced to accept a 33 bit signed number that is rendered into
// a 32 bit number.
class LoadImmediate32<string instr_asm, Operand Od, RegisterOperand RO> :
  MipsAsmPseudoInst<(outs RO:$rt), (ins Od:$imm32),
                     !strconcat(instr_asm, "\t$rt, $imm32")> ;
def LoadImm32 : LoadImmediate32<"li", uimm32_coerced, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `LoadImmediate32` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `LoadImmediate32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2969-2973
```tablegen
class LoadAddressFromReg32<string instr_asm, Operand MemOpnd,
                           RegisterOperand RO> :
  MipsAsmPseudoInst<(outs RO:$rt), (ins MemOpnd:$addr),
                     !strconcat(instr_asm, "\t$rt, $addr")> ;
def LoadAddrReg32 : LoadAddressFromReg32<"la", mem, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `LoadAddressFromReg32` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `LoadAddressFromReg32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2975-2978
```tablegen
class LoadAddressFromImm32<string instr_asm, Operand Od, RegisterOperand RO> :
  MipsAsmPseudoInst<(outs RO:$rt), (ins Od:$imm32),
                     !strconcat(instr_asm, "\t$rt, $imm32")> ;
def LoadAddrImm32 : LoadAddressFromImm32<"la", i32imm, GPR32Opnd>;
```
- EN: Declares reusable TableGen class `LoadAddressFromImm32` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `LoadAddressFromImm32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2980-2983
```tablegen
def JalTwoReg : MipsAsmPseudoInst<(outs GPR32Opnd:$rd), (ins GPR32Opnd:$rs),
                      "jal\t$rd, $rs"> ;
def JalOneReg : MipsAsmPseudoInst<(outs), (ins GPR32Opnd:$rs),
                      "jal\t$rs"> ;
```
- EN: Defines TableGen record `JalTwoReg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JalTwoReg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2985-2990
```tablegen
class NORIMM_DESC_BASE<RegisterOperand RO, DAGOperand Imm> :
   MipsAsmPseudoInst<(outs RO:$rs), (ins RO:$rt, Imm:$imm),
                      "nor\t$rs, $rt, $imm">;
def NORImm : NORIMM_DESC_BASE<GPR32Opnd, simm32_relaxed>, GPR_32;
def : MipsInstAlias<"nor\t$rs, $imm", (NORImm GPR32Opnd:$rs, GPR32Opnd:$rs,
                                              simm32_relaxed:$imm)>, GPR_32;
```
- EN: Declares reusable TableGen class `NORIMM_DESC_BASE` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `NORIMM_DESC_BASE`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2992-2998
```tablegen
let hasDelaySlot = 1, isCTI = 1 in {
def BneImm : MipsAsmPseudoInst<(outs GPR32Opnd:$rt),
                               (ins ConstantImm64:$imm64, brtarget:$offset),
                               "bne\t$rt, $imm64, $offset">;
def BeqImm : MipsAsmPseudoInst<(outs GPR32Opnd:$rt),
                               (ins ConstantImm64:$imm64, brtarget:$offset),
                               "beq\t$rt, $imm64, $offset">;
```
- EN: Defines TableGen record `BneImm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BneImm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3000-3004
```tablegen
class CondBranchPseudo<string instr_asm> :
  MipsAsmPseudoInst<(outs), (ins GPR32Opnd:$rs, GPR32Opnd:$rt,
                                 brtarget:$offset),
                    !strconcat(instr_asm, "\t$rs, $rt, $offset")>;
}
```
- EN: Declares reusable TableGen class `CondBranchPseudo` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `CondBranchPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3006-3021
```tablegen
def BLT : CondBranchPseudo<"blt">;
def BLE : CondBranchPseudo<"ble">;
def BGE : CondBranchPseudo<"bge">;
def BGT : CondBranchPseudo<"bgt">;
def BLTU : CondBranchPseudo<"bltu">;
def BLEU : CondBranchPseudo<"bleu">;
def BGEU : CondBranchPseudo<"bgeu">;
def BGTU : CondBranchPseudo<"bgtu">;
def BLTL : CondBranchPseudo<"bltl">, ISA_MIPS2_NOT_32R6_64R6;
def BLEL : CondBranchPseudo<"blel">, ISA_MIPS2_NOT_32R6_64R6;
def BGEL : CondBranchPseudo<"bgel">, ISA_MIPS2_NOT_32R6_64R6;
def BGTL : CondBranchPseudo<"bgtl">, ISA_MIPS2_NOT_32R6_64R6;
def BLTUL: CondBranchPseudo<"bltul">, ISA_MIPS2_NOT_32R6_64R6;
def BLEUL: CondBranchPseudo<"bleul">, ISA_MIPS2_NOT_32R6_64R6;
def BGEUL: CondBranchPseudo<"bgeul">, ISA_MIPS2_NOT_32R6_64R6;
def BGTUL: CondBranchPseudo<"bgtul">, ISA_MIPS2_NOT_32R6_64R6;
```
- EN: Defines TableGen record `BLT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BLT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3023-3026
```tablegen
let isCTI = 1 in
class CondBranchImmPseudo<string instr_asm> :
  MipsAsmPseudoInst<(outs), (ins GPR32Opnd:$rs, ConstantImm64:$imm, brtarget:$offset),
                    !strconcat(instr_asm, "\t$rs, $imm, $offset")>;
```
- EN: Declares reusable TableGen class `CondBranchImmPseudo` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `CondBranchImmPseudo`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3028-3029
```tablegen
def BEQLImmMacro : CondBranchImmPseudo<"beql">, ISA_MIPS2_NOT_32R6_64R6;
def BNELImmMacro : CondBranchImmPseudo<"bnel">, ISA_MIPS2_NOT_32R6_64R6;
```
- EN: Defines TableGen record `BEQLImmMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BEQLImmMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3031-3046
```tablegen
def BLTImmMacro  : CondBranchImmPseudo<"blt">;
def BLEImmMacro  : CondBranchImmPseudo<"ble">;
def BGEImmMacro  : CondBranchImmPseudo<"bge">;
def BGTImmMacro  : CondBranchImmPseudo<"bgt">;
def BLTUImmMacro : CondBranchImmPseudo<"bltu">;
def BLEUImmMacro : CondBranchImmPseudo<"bleu">;
def BGEUImmMacro : CondBranchImmPseudo<"bgeu">;
def BGTUImmMacro : CondBranchImmPseudo<"bgtu">;
def BLTLImmMacro : CondBranchImmPseudo<"bltl">, ISA_MIPS2_NOT_32R6_64R6;
def BLELImmMacro : CondBranchImmPseudo<"blel">, ISA_MIPS2_NOT_32R6_64R6;
def BGELImmMacro : CondBranchImmPseudo<"bgel">, ISA_MIPS2_NOT_32R6_64R6;
def BGTLImmMacro : CondBranchImmPseudo<"bgtl">, ISA_MIPS2_NOT_32R6_64R6;
def BLTULImmMacro : CondBranchImmPseudo<"bltul">, ISA_MIPS2_NOT_32R6_64R6;
def BLEULImmMacro : CondBranchImmPseudo<"bleul">, ISA_MIPS2_NOT_32R6_64R6;
def BGEULImmMacro : CondBranchImmPseudo<"bgeul">, ISA_MIPS2_NOT_32R6_64R6;
def BGTULImmMacro : CondBranchImmPseudo<"bgtul">, ISA_MIPS2_NOT_32R6_64R6;
```
- EN: Defines TableGen record `BLTImmMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BLTImmMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3048-3052
```tablegen
// FIXME: Predicates are removed because instructions are matched regardless of
// predicates, because PredicateControl was not in the hierarchy. This was
// done to emit more precise error message from expansion function.
// Once the tablegen-erated errors are made better, this needs to be fixed and
// predicates needs to be restored.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3054-3069
```tablegen
def SDivMacro : MipsAsmPseudoInst<(outs GPR32NonZeroOpnd:$rd),
                                  (ins GPR32Opnd:$rs, GPR32Opnd:$rt),
                                  "div\t$rd, $rs, $rt">,
                ISA_MIPS1_NOT_32R6_64R6;
def SDivIMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                   (ins GPR32Opnd:$rs, simm32:$imm),
                                   "div\t$rd, $rs, $imm">,
                 ISA_MIPS1_NOT_32R6_64R6;
def UDivMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                  (ins GPR32Opnd:$rs, GPR32Opnd:$rt),
                                  "divu\t$rd, $rs, $rt">,
                ISA_MIPS1_NOT_32R6_64R6;
def UDivIMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                   (ins GPR32Opnd:$rs, simm32:$imm),
                                   "divu\t$rd, $rs, $imm">,
                 ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `SDivMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SDivMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3072-3081
```tablegen
def : MipsInstAlias<"div $rs, $rt", (SDIV GPR32ZeroOpnd:$rs,
                                          GPR32Opnd:$rt), 0>,
     ISA_MIPS1_NOT_32R6_64R6;
def : MipsInstAlias<"div $rs, $rt", (SDivMacro GPR32NonZeroOpnd:$rs,
                                               GPR32NonZeroOpnd:$rs,
                                               GPR32Opnd:$rt), 0>,
     ISA_MIPS1_NOT_32R6_64R6;
def : MipsInstAlias<"div $rd, $imm", (SDivIMacro GPR32Opnd:$rd, GPR32Opnd:$rd,
                                                 simm32:$imm), 0>,
      ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3083-3089
```tablegen
def : MipsInstAlias<"divu $rt, $rs", (UDIV GPR32ZeroOpnd:$rt,
                                           GPR32Opnd:$rs), 0>,
      ISA_MIPS1_NOT_32R6_64R6;
def : MipsInstAlias<"divu $rt, $rs", (UDivMacro GPR32NonZeroOpnd:$rt,
                                                GPR32NonZeroOpnd:$rt,
                                                GPR32Opnd:$rs), 0>,
      ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3091-3093
```tablegen
def : MipsInstAlias<"divu $rd, $imm", (UDivIMacro GPR32Opnd:$rd, GPR32Opnd:$rd,
                                                  simm32:$imm), 0>,
      ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3096-3096
```tablegen
def : MipsInstAlias<"nal", (BLTZAL ZERO, 0), 1>, ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3098-3113
```tablegen
def SRemMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                  (ins GPR32Opnd:$rs, GPR32Opnd:$rt),
                                  "rem\t$rd, $rs, $rt">,
                ISA_MIPS1_NOT_32R6_64R6;
def SRemIMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                   (ins GPR32Opnd:$rs, simm32_relaxed:$imm),
                                   "rem\t$rd, $rs, $imm">,
                 ISA_MIPS1_NOT_32R6_64R6;
def URemMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                  (ins GPR32Opnd:$rs, GPR32Opnd:$rt),
                                  "remu\t$rd, $rs, $rt">,
                ISA_MIPS1_NOT_32R6_64R6;
def URemIMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                   (ins GPR32Opnd:$rs, simm32_relaxed:$imm),
                                   "remu\t$rd, $rs, $imm">,
                 ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `SRemMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SRemMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3115-3126
```tablegen
def : MipsInstAlias<"rem $rt, $rs", (SRemMacro GPR32Opnd:$rt, GPR32Opnd:$rt,
                                               GPR32Opnd:$rs), 0>,
      ISA_MIPS1_NOT_32R6_64R6;
def : MipsInstAlias<"rem $rd, $imm", (SRemIMacro GPR32Opnd:$rd, GPR32Opnd:$rd,
                                      simm32_relaxed:$imm), 0>,
      ISA_MIPS1_NOT_32R6_64R6;
def : MipsInstAlias<"remu $rt, $rs", (URemMacro GPR32Opnd:$rt, GPR32Opnd:$rt,
                                                GPR32Opnd:$rs), 0>,
      ISA_MIPS1_NOT_32R6_64R6;
def : MipsInstAlias<"remu $rd, $imm", (URemIMacro GPR32Opnd:$rd, GPR32Opnd:$rd,
                                       simm32_relaxed:$imm), 0>,
      ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3128-3129
```tablegen
def Ulh : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins mem:$addr),
                            "ulh\t$rt, $addr">; //, ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `Ulh` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Ulh`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3131-3132
```tablegen
def Ulhu : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins mem:$addr),
                             "ulhu\t$rt, $addr">; //, ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `Ulhu` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Ulhu`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3134-3135
```tablegen
def Ulw : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins mem:$addr),
                            "ulw\t$rt, $addr">; //, ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `Ulw` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Ulw`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3137-3138
```tablegen
def Ush : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins mem:$addr),
                            "ush\t$rt, $addr">; //, ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `Ush` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Ush`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3140-3141
```tablegen
def Usw : MipsAsmPseudoInst<(outs GPR32Opnd:$rt), (ins mem:$addr),
                            "usw\t$rt, $addr">; //, ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Defines TableGen record `Usw` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Usw`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3143-3151
```tablegen
def LDMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rt),
                                (ins mem_simm16:$addr), "ld $rt, $addr">,
                                ISA_MIPS1_NOT_MIPS3;
def SDMacro : MipsAsmPseudoInst<(outs GPR32Opnd:$rt),
                                (ins mem_simm16:$addr), "sd $rt, $addr">,
                                ISA_MIPS1_NOT_MIPS3;
//===----------------------------------------------------------------------===//
//  Arbitrary patterns that map to one or more instructions
//===----------------------------------------------------------------------===//
```
- EN: Defines TableGen record `LDMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3153-3155
```tablegen
// Load/store pattern templates.
class LoadRegImmPat<Instruction LoadInst, ValueType ValTy, PatFrag Node> :
  MipsPat<(ValTy (Node addrRegImm:$a)), (LoadInst addrRegImm:$a)>;
```
- EN: Declares reusable TableGen class `LoadRegImmPat` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `LoadRegImmPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3157-3158
```tablegen
class StoreRegImmPat<Instruction StoreInst, ValueType ValTy> :
  MipsPat<(store ValTy:$v, addrRegImm:$a), (StoreInst ValTy:$v, addrRegImm:$a)>;
```
- EN: Declares reusable TableGen class `StoreRegImmPat` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `StoreRegImmPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3160-3163
```tablegen
// Materialize constants.
multiclass MaterializeImms<ValueType VT, Register ZEROReg,
                           Instruction ADDiuOp, Instruction LUiOp,
                           Instruction ORiOp> {
```
- EN: Declares TableGen `multiclass MaterializeImms`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MaterializeImms`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3165-3168
```tablegen
// Constant synthesis previously relied on the ordering of the patterns below.
// By making the predicates they use non-overlapping, the patterns were
// reordered so that the effect of the newly introduced predicates can be
// observed.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3170-3172
```tablegen
// Arbitrary immediates
def : MipsPat<(VT LUiORiPred:$imm),
              (ORiOp (LUiOp (HI16 imm:$imm)), (LO16 imm:$imm))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3174-3175
```tablegen
// Bits 32-16 set, sign/zero extended.
def : MipsPat<(VT LUiPred:$imm), (LUiOp (HI16 imm:$imm))>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3177-3180
```tablegen
// Small immediates
def : MipsPat<(VT ORiPred:$imm), (ORiOp ZEROReg, imm:$imm)>;
def : MipsPat<(VT immSExt16:$imm), (ADDiuOp ZEROReg, imm:$imm)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3182-3183
```tablegen
let AdditionalPredicates = [NotInMicroMips] in
  defm : MaterializeImms<i32, ZERO, ADDiu, LUi, ORi>, ISA_MIPS1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3185-3193
```tablegen
// Carry MipsPatterns
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsPat<(subc GPR32:$lhs, GPR32:$rhs),
                (SUBu GPR32:$lhs, GPR32:$rhs)>, ISA_MIPS1;
}
def : MipsPat<(addc GPR32:$lhs, GPR32:$rhs),
              (ADDu GPR32:$lhs, GPR32:$rhs)>, ISA_MIPS1, ASE_NOT_DSP;
def : MipsPat<(addc  GPR32:$src, immSExt16:$imm),
              (ADDiu GPR32:$src, imm:$imm)>, ISA_MIPS1, ASE_NOT_DSP;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3195-3199
```tablegen
// Support multiplication for pre-Mips32 targets that don't have
// the MUL instruction.
def : MipsPat<(mul GPR32:$lhs, GPR32:$rhs),
              (PseudoMFLO (PseudoMULT GPR32:$lhs, GPR32:$rhs))>,
      ISA_MIPS1_NOT_32R6_64R6;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3201-3203
```tablegen
// SYNC
def : MipsPat<(MipsSync (i32 timmz)),
              (SYNC 0)>, ISA_MIPS2;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3205-3209
```tablegen
// Call
def : MipsPat<(MipsJmpLink (i32 texternalsym:$dst)),
              (JAL texternalsym:$dst)>, ISA_MIPS1;
//def : MipsPat<(MipsJmpLink GPR32:$dst),
//              (JALR GPR32:$dst)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3211-3225
```tablegen
// Tail call
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsPat<(MipsTailCall (iPTR tglobaladdr:$dst)),
                (TAILCALL tglobaladdr:$dst)>, ISA_MIPS1;
  def : MipsPat<(MipsTailCall (iPTR texternalsym:$dst)),
                (TAILCALL texternalsym:$dst)>, ISA_MIPS1;
}
// hi/lo relocs
multiclass MipsHiLoRelocs<Instruction Lui, Instruction Addiu,
                          Register ZeroReg, RegisterOperand GPROpnd> {
  def : MipsPat<(MipsHi tglobaladdr:$in), (Lui tglobaladdr:$in)>;
  def : MipsPat<(MipsHi tblockaddress:$in), (Lui tblockaddress:$in)>;
  def : MipsPat<(MipsHi tjumptable:$in), (Lui tjumptable:$in)>;
  def : MipsPat<(MipsHi tconstpool:$in), (Lui tconstpool:$in)>;
  def : MipsPat<(MipsHi texternalsym:$in), (Lui texternalsym:$in)>;
```
- EN: Declares TableGen `multiclass MipsHiLoRelocs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass MipsHiLoRelocs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3227-3238
```tablegen
  def : MipsPat<(MipsLo tglobaladdr:$in),
                (Addiu ZeroReg, tglobaladdr:$in)>;
  def : MipsPat<(MipsLo tblockaddress:$in),
                (Addiu ZeroReg, tblockaddress:$in)>;
  def : MipsPat<(MipsLo tjumptable:$in),
                (Addiu ZeroReg, tjumptable:$in)>;
  def : MipsPat<(MipsLo tconstpool:$in),
                (Addiu ZeroReg, tconstpool:$in)>;
  def : MipsPat<(MipsLo tglobaltlsaddr:$in),
                (Addiu ZeroReg, tglobaltlsaddr:$in)>;
  def : MipsPat<(MipsLo texternalsym:$in),
                (Addiu ZeroReg, texternalsym:$in)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3240-3252
```tablegen
  def : MipsPat<(add GPROpnd:$hi, (MipsLo tglobaladdr:$lo)),
                (Addiu GPROpnd:$hi, tglobaladdr:$lo)>;
  def : MipsPat<(add GPROpnd:$hi, (MipsLo tblockaddress:$lo)),
                (Addiu GPROpnd:$hi, tblockaddress:$lo)>;
  def : MipsPat<(add GPROpnd:$hi, (MipsLo tjumptable:$lo)),
                (Addiu GPROpnd:$hi, tjumptable:$lo)>;
  def : MipsPat<(add GPROpnd:$hi, (MipsLo tconstpool:$lo)),
                (Addiu GPROpnd:$hi, tconstpool:$lo)>;
  def : MipsPat<(add GPROpnd:$hi, (MipsLo tglobaltlsaddr:$lo)),
                (Addiu GPROpnd:$hi, tglobaltlsaddr:$lo)>;
  def : MipsPat<(add GPROpnd:$hi, (MipsLo texternalsym:$lo)),
                (Addiu GPROpnd:$hi, texternalsym:$lo)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3254-3256
```tablegen
// wrapper_pic
class WrapperPat<SDNode node, Instruction ADDiuOp, RegisterClass RC>:
      MipsPat<(MipsWrapper RC:$gp, node:$in), (ADDiuOp RC:$gp, node:$in)>;
```
- EN: Declares reusable TableGen class `WrapperPat` for `MipsInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrInfo` 声明可复用的 TableGen 类 `WrapperPat`，通常用于抽象共享字段、谓词或编码结构。

### Lines 3258-3259
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  defm : MipsHiLoRelocs<LUi, ADDiu, ZERO, GPR32Opnd>, ISA_MIPS1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3261-3263
```tablegen
  def : MipsPat<(MipsGotHi tglobaladdr:$in), (LUi tglobaladdr:$in)>, ISA_MIPS1;
  def : MipsPat<(MipsGotHi texternalsym:$in), (LUi texternalsym:$in)>,
        ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3265-3266
```tablegen
  def : MipsPat<(MipsTlsHi tglobaltlsaddr:$in), (LUi tglobaltlsaddr:$in)>,
        ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3268-3272
```tablegen
  // gp_rel relocs
  def : MipsPat<(add GPR32:$gp, (MipsGPRel tglobaladdr:$in)),
                (ADDiu GPR32:$gp, tglobaladdr:$in)>, ISA_MIPS1, ABI_NOT_N64;
  def : MipsPat<(add GPR32:$gp, (MipsGPRel tconstpool:$in)),
                (ADDiu GPR32:$gp, tconstpool:$in)>, ISA_MIPS1, ABI_NOT_N64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3274-3279
```tablegen
  def : WrapperPat<tglobaladdr, ADDiu, GPR32>, ISA_MIPS1;
  def : WrapperPat<tconstpool, ADDiu, GPR32>, ISA_MIPS1;
  def : WrapperPat<texternalsym, ADDiu, GPR32>, ISA_MIPS1;
  def : WrapperPat<tblockaddress, ADDiu, GPR32>, ISA_MIPS1;
  def : WrapperPat<tjumptable, ADDiu, GPR32>, ISA_MIPS1;
  def : WrapperPat<tglobaltlsaddr, ADDiu, GPR32>, ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3281-3284
```tablegen
  // Mips does not have "not", so we expand our way
  def : MipsPat<(not GPR32:$in),
                (NOR GPR32Opnd:$in, ZERO)>, ISA_MIPS1;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3286-3290
```tablegen
// extended loads
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsPat<(i32 (extloadi1  addr:$src)), (LBu addr:$src)>, ISA_MIPS1;
  def : MipsPat<(i32 (extloadi8  addr:$src)), (LBu addr:$src)>, ISA_MIPS1;
  def : MipsPat<(i32 (extloadi16 addr:$src)), (LHu addr:$src)>, ISA_MIPS1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3292-3294
```tablegen
  // peepholes
  def : MipsPat<(store (i32 0), addr:$dst), (SW ZERO, addr:$dst)>, ISA_MIPS1;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3296-3304
```tablegen
// brcond patterns
multiclass BrcondPats<RegisterClass RC, Instruction BEQOp, Instruction BEQOp1,
                      Instruction BNEOp, Instruction SLTOp, Instruction SLTuOp,
                      Instruction SLTiOp, Instruction SLTiuOp,
                      Register ZEROReg> {
def : MipsPat<(brcond (i32 (setne RC:$lhs, 0)), bb:$dst),
              (BNEOp RC:$lhs, ZEROReg, bb:$dst)>;
def : MipsPat<(brcond (i32 (seteq RC:$lhs, 0)), bb:$dst),
              (BEQOp RC:$lhs, ZEROReg, bb:$dst)>;
```
- EN: Declares TableGen `multiclass BrcondPats`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass BrcondPats`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3306-3317
```tablegen
def : MipsPat<(brcond (i32 (setge RC:$lhs, RC:$rhs)), bb:$dst),
              (BEQOp1 (SLTOp RC:$lhs, RC:$rhs), ZERO, bb:$dst)>;
def : MipsPat<(brcond (i32 (setuge RC:$lhs, RC:$rhs)), bb:$dst),
              (BEQOp1 (SLTuOp RC:$lhs, RC:$rhs), ZERO, bb:$dst)>;
def : MipsPat<(brcond (i32 (setge RC:$lhs, immSExt16:$rhs)), bb:$dst),
              (BEQOp1 (SLTiOp RC:$lhs, immSExt16:$rhs), ZERO, bb:$dst)>;
def : MipsPat<(brcond (i32 (setuge RC:$lhs, immSExt16:$rhs)), bb:$dst),
              (BEQOp1 (SLTiuOp RC:$lhs, immSExt16:$rhs), ZERO, bb:$dst)>;
def : MipsPat<(brcond (i32 (setgt RC:$lhs, immSExt16Plus1:$rhs)), bb:$dst),
              (BEQOp1 (SLTiOp RC:$lhs, (Plus1 imm:$rhs)), ZERO, bb:$dst)>;
def : MipsPat<(brcond (i32 (setugt RC:$lhs, immSExt16Plus1:$rhs)), bb:$dst),
              (BEQOp1 (SLTiuOp RC:$lhs, (Plus1 imm:$rhs)), ZERO, bb:$dst)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3319-3322
```tablegen
def : MipsPat<(brcond (i32 (setle RC:$lhs, RC:$rhs)), bb:$dst),
              (BEQOp1 (SLTOp RC:$rhs, RC:$lhs), ZERO, bb:$dst)>;
def : MipsPat<(brcond (i32 (setule RC:$lhs, RC:$rhs)), bb:$dst),
              (BEQOp1 (SLTuOp RC:$rhs, RC:$lhs), ZERO, bb:$dst)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3324-3334
```tablegen
def : MipsPat<(brcond RC:$cond, bb:$dst),
              (BNEOp RC:$cond, ZEROReg, bb:$dst)>;
}
let AdditionalPredicates = [NotInMicroMips] in {
  defm : BrcondPats<GPR32, BEQ, BEQ, BNE, SLT, SLTu, SLTi, SLTiu, ZERO>,
         ISA_MIPS1;
  def : MipsPat<(brcond (i32 (setlt i32:$lhs, 1)), bb:$dst),
                (BLEZ i32:$lhs, bb:$dst)>, ISA_MIPS1;
  def : MipsPat<(brcond (i32 (setgt i32:$lhs, -1)), bb:$dst),
                (BGEZ i32:$lhs, bb:$dst)>, ISA_MIPS1;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3336-3347
```tablegen
// setcc patterns
multiclass SeteqPats<RegisterClass RC, Instruction SLTiuOp, Instruction XOROp,
                     Instruction SLTuOp, Register ZEROReg> {
  def : MipsPat<(seteq RC:$lhs, 0),
                (SLTiuOp RC:$lhs, 1)>;
  def : MipsPat<(setne RC:$lhs, 0),
                (SLTuOp ZEROReg, RC:$lhs)>;
  def : MipsPat<(seteq RC:$lhs, RC:$rhs),
                (SLTiuOp (XOROp RC:$lhs, RC:$rhs), 1)>;
  def : MipsPat<(setne RC:$lhs, RC:$rhs),
                (SLTuOp ZEROReg, (XOROp RC:$lhs, RC:$rhs))>;
}
```
- EN: Declares TableGen `multiclass SeteqPats`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass SeteqPats`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3349-3355
```tablegen
multiclass SetlePats<RegisterClass RC, Instruction XORiOp, Instruction SLTOp,
                     Instruction SLTuOp> {
  def : MipsPat<(setle RC:$lhs, RC:$rhs),
                (XORiOp (SLTOp RC:$rhs, RC:$lhs), 1)>;
  def : MipsPat<(setule RC:$lhs, RC:$rhs),
                (XORiOp (SLTuOp RC:$rhs, RC:$lhs), 1)>;
}
```
- EN: Declares TableGen `multiclass SetlePats`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass SetlePats`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3357-3362
```tablegen
multiclass SetgtPats<RegisterClass RC, Instruction SLTOp, Instruction SLTuOp> {
  def : MipsPat<(setgt RC:$lhs, RC:$rhs),
                (SLTOp RC:$rhs, RC:$lhs)>;
  def : MipsPat<(setugt RC:$lhs, RC:$rhs),
                (SLTuOp RC:$rhs, RC:$lhs)>;
}
```
- EN: Declares TableGen `multiclass SetgtPats`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass SetgtPats`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3364-3370
```tablegen
multiclass SetgePats<RegisterClass RC, Instruction XORiOp, Instruction SLTOp,
                     Instruction SLTuOp> {
  def : MipsPat<(setge RC:$lhs, RC:$rhs),
                (XORiOp (SLTOp RC:$lhs, RC:$rhs), 1)>;
  def : MipsPat<(setuge RC:$lhs, RC:$rhs),
                (XORiOp (SLTuOp RC:$lhs, RC:$rhs), 1)>;
}
```
- EN: Declares TableGen `multiclass SetgePats`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass SetgePats`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3372-3378
```tablegen
multiclass SetgeImmPats<RegisterClass RC, Instruction XORiOp,
                        Instruction SLTiOp, Instruction SLTiuOp> {
  def : MipsPat<(setge RC:$lhs, immSExt16:$rhs),
                (XORiOp (SLTiOp RC:$lhs, immSExt16:$rhs), 1)>;
  def : MipsPat<(setuge RC:$lhs, immSExt16:$rhs),
                (XORiOp (SLTiuOp RC:$lhs, immSExt16:$rhs), 1)>;
}
```
- EN: Declares TableGen `multiclass SetgeImmPats`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass SetgeImmPats`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3380-3385
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  defm : SeteqPats<GPR32, SLTiu, XOR, SLTu, ZERO>, ISA_MIPS1;
  defm : SetlePats<GPR32, XORi, SLT, SLTu>, ISA_MIPS1;
  defm : SetgtPats<GPR32, SLT, SLTu>, ISA_MIPS1;
  defm : SetgePats<GPR32, XORi, SLT, SLTu>, ISA_MIPS1;
  defm : SetgeImmPats<GPR32, XORi, SLTi, SLTiu>, ISA_MIPS1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3387-3389
```tablegen
  // bswap pattern
  def : MipsPat<(bswap GPR32:$rt), (ROTR (WSBH GPR32:$rt), 16)>, ISA_MIPS32R2;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3391-3399
```tablegen
// Load halfword/word patterns.
let AdditionalPredicates = [NotInMicroMips] in {
  let AddedComplexity = 40 in {
    def : LoadRegImmPat<LBu, i32, zextloadi8>, ISA_MIPS1;
    def : LoadRegImmPat<LHu, i32, zextloadi16>, ISA_MIPS1;
    def : LoadRegImmPat<LB, i32, sextloadi8>, ISA_MIPS1;
    def : LoadRegImmPat<LH, i32, sextloadi16>, ISA_MIPS1;
    def : LoadRegImmPat<LW, i32, load>, ISA_MIPS1;
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3401-3404
```tablegen
  // Atomic load patterns.
  def : MipsPat<(atomic_load_asext_8 addr:$a), (LB addr:$a)>, ISA_MIPS1;
  def : MipsPat<(atomic_load_asext_16 addr:$a), (LH addr:$a)>, ISA_MIPS1;
  def : MipsPat<(atomic_load_nonext_32 addr:$a), (LW addr:$a)>, ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3406-3413
```tablegen
  // Atomic store patterns.
  def : MipsPat<(atomic_store_8 GPR32:$v, addr:$a), (SB GPR32:$v, addr:$a)>,
        ISA_MIPS1;
  def : MipsPat<(atomic_store_16 GPR32:$v, addr:$a), (SH GPR32:$v, addr:$a)>,
        ISA_MIPS1;
  def : MipsPat<(atomic_store_32 GPR32:$v, addr:$a), (SW GPR32:$v, addr:$a)>,
        ISA_MIPS1;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3415-3417
```tablegen
//===----------------------------------------------------------------------===//
// Floating Point Support
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3419-3421
```tablegen
include "MipsInstrFPU.td"
include "Mips64InstrInfo.td"
include "MipsCondMov.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 3423-3424
```tablegen
include "Mips32r6InstrInfo.td"
include "Mips64r6InstrInfo.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 3426-3427
```tablegen
//
// Mips16
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3429-3430
```tablegen
include "Mips16InstrFormats.td"
include "Mips16InstrInfo.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 3432-3434
```tablegen
// DSP
include "MipsDSPInstrFormats.td"
include "MipsDSPInstrInfo.td"
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3436-3438
```tablegen
// MSA
include "MipsMSAInstrFormats.td"
include "MipsMSAInstrInfo.td"
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3440-3442
```tablegen
// EVA
include "MipsEVAInstrFormats.td"
include "MipsEVAInstrInfo.td"
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3444-3446
```tablegen
// MT
include "MipsMTInstrFormats.td"
include "MipsMTInstrInfo.td"
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3448-3451
```tablegen
// Micromips
include "MicroMipsInstrFormats.td"
include "MicroMipsInstrInfo.td"
include "MicroMipsInstrFPU.td"
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3453-3455
```tablegen
// Micromips r6
include "MicroMips32r6InstrFormats.td"
include "MicroMips32r6InstrInfo.td"
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3457-3459
```tablegen
// Micromips DSP
include "MicroMipsDSPInstrFormats.td"
include "MicroMipsDSPInstrInfo.td"
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

## Key Concepts / 关键概念

- EN: Primary role: instruction semantics, scheduling hints, and machine-level helpers.
  - CN: 核心职责：指令语义、调度提示以及机器级辅助逻辑。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: SelectionDAG patterns or node profiles connect IR-level intent to target instructions.
  - CN: SelectionDAG 模式或节点轮廓把 IR 层意图连接到目标指令。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `MipsInstrCompiler.td`, `MipsInstrFormats.td`, `MipsInstrFPU.td`, `Mips64InstrInfo.td`, `MipsCondMov.td`, `Mips32r6InstrInfo.td`, `Mips64r6InstrInfo.td`, `Mips16InstrFormats.td` ... (+16 more).
  - CN: TableGen 包含项：`MipsInstrCompiler.td`, `MipsInstrFormats.td`, `MipsInstrFPU.td`, `Mips64InstrInfo.td`, `MipsCondMov.td`, `Mips32r6InstrInfo.td`, `Mips64r6InstrInfo.td`, `Mips16InstrFormats.td` ... (+16 more)。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
