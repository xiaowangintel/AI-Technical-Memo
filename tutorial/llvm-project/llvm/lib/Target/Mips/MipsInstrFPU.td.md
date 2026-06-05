# MipsInstrFPU.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MipsInstrFPU.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the Mips FPU instruction set.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `MipsInstrFPU`，涵盖指令定义与目标操作码元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===-- MipsInstrFPU.td - Mips FPU Instruction Information -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the Mips FPU instruction set.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-23
```tablegen
//===----------------------------------------------------------------------===//
// Floating Point Instructions
// ------------------------
// * 64bit fp:
//    - 32 64-bit registers (default mode)
//    - 16 even 32-bit registers (32-bit compatible mode) for
//      single and double access.
// * 32bit fp:
//    - 16 even 32-bit registers - single and double (aliased)
//    - 32 32-bit registers (within single-only mode)
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 25-39
```tablegen
// Floating Point Compare and Branch
def SDT_MipsFPBrcond : SDTypeProfile<0, 3, [SDTCisInt<0>,
                                            SDTCisVT<1, i32>,
                                            SDTCisVT<2, OtherVT>]>;
def SDT_MipsFPCmp : SDTypeProfile<0, 3, [SDTCisSameAs<0, 1>, SDTCisFP<1>,
                                         SDTCisVT<2, i32>]>;
def SDT_MipsCMovFP : SDTypeProfile<1, 3, [SDTCisSameAs<0, 1>, SDTCisVT<2, i32>,
                                          SDTCisSameAs<1, 3>]>;
def SDT_MipsTruncIntFP : SDTypeProfile<1, 1, [SDTCisFP<0>, SDTCisFP<1>]>;
def SDT_MipsBuildPairF64 : SDTypeProfile<1, 2, [SDTCisVT<0, f64>,
                                                SDTCisVT<1, i32>,
                                                SDTCisSameAs<1, 2>]>;
def SDT_MipsExtractElementF64 : SDTypeProfile<1, 2, [SDTCisVT<0, i32>,
                                                     SDTCisVT<1, f64>,
                                                     SDTCisVT<2, i32>]>;
```
- EN: Defines TableGen record `SDT_MipsFPBrcond` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_MipsFPBrcond`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 41-42
```tablegen
def SDT_MipsMTC1_D64 : SDTypeProfile<1, 1, [SDTCisVT<0, f64>,
                                            SDTCisVT<1, i32>]>;
```
- EN: Defines TableGen record `SDT_MipsMTC1_D64` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_MipsMTC1_D64`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 44-45
```tablegen
// Floating Point Compare
def MipsFPCmp : SDNode<"MipsISD::FPCmp", SDT_MipsFPCmp, [SDNPOutGlue]>;
```
- EN: Defines TableGen record `MipsFPCmp` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsFPCmp`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 47-49
```tablegen
// Floating Point Conditional Moves
def MipsCMovFP_T : SDNode<"MipsISD::CMovFP_T", SDT_MipsCMovFP, [SDNPInGlue]>;
def MipsCMovFP_F : SDNode<"MipsISD::CMovFP_F", SDT_MipsCMovFP, [SDNPInGlue]>;
```
- EN: Defines TableGen record `MipsCMovFP_T` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsCMovFP_T`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 51-53
```tablegen
// Floating Point Branch Conditional
def MipsFPBrcond : SDNode<"MipsISD::FPBrcond", SDT_MipsFPBrcond,
                          [SDNPHasChain, SDNPOptInGlue]>;
```
- EN: Defines TableGen record `MipsFPBrcond` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsFPBrcond`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 55-56
```tablegen
// FP-to-int truncation node.
def MipsTruncIntFP : SDNode<"MipsISD::TruncIntFP", SDT_MipsTruncIntFP>;
```
- EN: Defines TableGen record `MipsTruncIntFP` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsTruncIntFP`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 58-61
```tablegen
def MipsBuildPairF64 : SDNode<"MipsISD::BuildPairF64", SDT_MipsBuildPairF64>;
def : GINodeEquiv<G_MERGE_VALUES, MipsBuildPairF64>;
def MipsExtractElementF64 : SDNode<"MipsISD::ExtractElementF64",
                                   SDT_MipsExtractElementF64>;
```
- EN: Defines TableGen record `MipsBuildPairF64` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsBuildPairF64`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 63-64
```tablegen
// Node used to generate an MTC1 i32 to f64 instruction
def MipsMTC1_D64 : SDNode<"MipsISD::MTC1_D64", SDT_MipsMTC1_D64>;
```
- EN: Defines TableGen record `MipsMTC1_D64` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `MipsMTC1_D64`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 67-69
```tablegen
// Operand for printing out a condition code.
let PrintMethod = "printFCCOperand", DecoderMethod = "DecodeCondCode" in
  def condcode : Operand<i32>;
```
- EN: Defines TableGen record `condcode` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `condcode`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 71-73
```tablegen
//===----------------------------------------------------------------------===//
// Feature predicates.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 75-86
```tablegen
def IsFP64bit        : Predicate<"Subtarget->isFP64bit()">,
                       AssemblerPredicate<(all_of FeatureFP64Bit)>;
def NotFP64bit       : Predicate<"!Subtarget->isFP64bit()">,
                       AssemblerPredicate<(all_of (not FeatureFP64Bit))>;
def IsSingleFloat    : Predicate<"Subtarget->isSingleFloat()">,
                       AssemblerPredicate<(all_of FeatureSingleFloat)>;
def IsNotSingleFloat : Predicate<"!Subtarget->isSingleFloat()">,
                       AssemblerPredicate<(all_of (not FeatureSingleFloat))>;
def IsNotSoftFloat   : Predicate<"!Subtarget->useSoftFloat()">,
                       AssemblerPredicate<(all_of (not FeatureSoftFloat))>;
def HasMips3D        : Predicate<"Subtarget->has3D()">,
                       AssemblerPredicate<(all_of FeatureMips3D)>;
```
- EN: Defines TableGen record `IsFP64bit` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsFP64bit`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 88-91
```tablegen
//===----------------------------------------------------------------------===//
// Mips FGR size adjectives.
// They are mutually exclusive.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 93-95
```tablegen
class FGR_32 { list<Predicate> FGRPredicates = [NotFP64bit, IsNotSingleFloat]; }
class FGR_64 { list<Predicate> FGRPredicates = [IsFP64bit, IsNotSingleFloat]; }
class HARDFLOAT { list<Predicate> HardFloatPredicate = [IsNotSoftFloat]; }
```
- EN: Declares reusable TableGen class `FGR_32` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `FGR_32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 97-97
```tablegen
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 99-102
```tablegen
// FP immediate patterns.
def fpimm0 : PatLeaf<(fpimm), [{
  return N->isExactlyValue(+0.0);
}]>;
```
- EN: Defines TableGen record `fpimm0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `fpimm0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 104-106
```tablegen
def fpimm0neg : PatLeaf<(fpimm), [{
  return N->isExactlyValue(-0.0);
}]>;
```
- EN: Defines TableGen record `fpimm0neg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `fpimm0neg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 108-125
```tablegen
//===----------------------------------------------------------------------===//
// Instruction Class Templates
//
// A set of multiclasses is used to address the register usage.
//
// S32 - single precision in 16 32bit even fp registers
//       single precision in 32 32bit fp registers in SingleOnly mode
// S64 - single precision in 32 64bit fp registers (In64BitMode)
// D32 - double precision in 16 32bit even fp registers
// D64 - double precision in 32 64bit fp registers (In64BitMode)
//
// Only S32 and D32 are supported right now.
//===----------------------------------------------------------------------===//
class ADDS_FT<string opstr, RegisterOperand RC, bit IsComm,
              SDPatternOperator OpNode= null_frag> :
  InstSE<(outs RC:$fd), (ins RC:$fs, RC:$ft),
         !strconcat(opstr, "\t$fd, $fs, $ft"),
         [(set RC:$fd, (OpNode RC:$fs, RC:$ft))], FrmFR, opstr>,
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 126-128
```tablegen
  HARDFLOAT {
  let isCommutable = IsComm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 130-136
```tablegen
multiclass ADDS_M<string opstr, bit IsComm,
                  SDPatternOperator OpNode = null_frag> {
  def _D32 : MMRel, ADDS_FT<opstr, AFGR64Opnd, IsComm, OpNode>, FGR_32;
  def _D64 : ADDS_FT<opstr, FGR64Opnd, IsComm, OpNode>, FGR_64 {
    string DecoderNamespace = "MipsFP64";
  }
}
```
- EN: Declares TableGen `multiclass ADDS_M`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass ADDS_M`，它是一个可复用模板，可展开为多个相关记录。

### Lines 138-146
```tablegen
class ABSS_FT_PatFrag<SDPatternOperator OpNode>:
  PatFrag<(ops node:$fs), (OpNode node:$fs),
          [{ return Subtarget->inAbs2008Mode() || N->getFlags().hasNoNaNs(); }]>;
class ABSS_FT<string opstr, RegisterOperand DstRC, RegisterOperand SrcRC,
              SDPatternOperator OpNode= null_frag> :
  InstSE<(outs DstRC:$fd), (ins SrcRC:$fs), !strconcat(opstr, "\t$fd, $fs"),
         [(set DstRC:$fd, (OpNode SrcRC:$fs))], FrmFR, opstr>,
  HARDFLOAT,
  NeverHasSideEffects;
```
- EN: Declares reusable TableGen class `ABSS_FT_PatFrag` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `ABSS_FT_PatFrag`，通常用于抽象共享字段、谓词或编码结构。

### Lines 148-156
```tablegen
class CVT_PS_S_FT<string opstr, RegisterOperand DstRC, RegisterOperand SrcRC,
                  bit IsComm,
                  SDPatternOperator OpNode = null_frag> :
  InstSE<(outs DstRC:$fd), (ins SrcRC:$fs, SrcRC:$ft),
         !strconcat(opstr, "\t$fd, $fs, $ft"),
         [(set DstRC:$fd, (OpNode SrcRC:$fs, SrcRC:$ft))], FrmFR, opstr>,
  HARDFLOAT {
  let isCommutable = IsComm;
}
```
- EN: Declares reusable TableGen class `CVT_PS_S_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `CVT_PS_S_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 158-166
```tablegen
multiclass ABSS_M<string opstr,
                  SDPatternOperator OpNode= null_frag> {
  def _D32 : MMRel, ABSS_FT<opstr, AFGR64Opnd, AFGR64Opnd, OpNode>,
             FGR_32;
  def _D64 : StdMMR6Rel, ABSS_FT<opstr, FGR64Opnd, FGR64Opnd, OpNode>,
             FGR_64 {
    string DecoderNamespace = "MipsFP64";
  }
}
```
- EN: Declares TableGen `multiclass ABSS_M`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass ABSS_M`，它是一个可复用模板，可展开为多个相关记录。

### Lines 168-173
```tablegen
multiclass ROUND_M<string opstr> {
  def _D32 : MMRel, ABSS_FT<opstr, FGR32Opnd, AFGR64Opnd>, FGR_32;
  def _D64 : StdMMR6Rel, ABSS_FT<opstr, FGR32Opnd, FGR64Opnd>, FGR_64 {
    let DecoderNamespace = "MipsFP64";
  }
}
```
- EN: Declares TableGen `multiclass ROUND_M`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass ROUND_M`，它是一个可复用模板，可展开为多个相关记录。

### Lines 175-180
```tablegen
class MFC1_FT<string opstr, RegisterOperand DstRC, RegisterOperand SrcRC,
              SDPatternOperator OpNode= null_frag> :
  InstSE<(outs DstRC:$rt), (ins SrcRC:$fs), !strconcat(opstr, "\t$rt, $fs"),
         [(set DstRC:$rt, (OpNode SrcRC:$fs))], FrmFR, opstr>, HARDFLOAT {
  let isMoveReg = 1;
}
```
- EN: Declares reusable TableGen class `MFC1_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `MFC1_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 182-187
```tablegen
class MTC1_FT<string opstr, RegisterOperand DstRC, RegisterOperand SrcRC,
              SDPatternOperator OpNode= null_frag> :
  InstSE<(outs DstRC:$fs), (ins SrcRC:$rt), !strconcat(opstr, "\t$rt, $fs"),
         [(set DstRC:$fs, (OpNode SrcRC:$rt))], FrmFR, opstr>, HARDFLOAT {
  let isMoveReg = 1;
}
```
- EN: Declares reusable TableGen class `MTC1_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `MTC1_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 189-195
```tablegen
class MTC1_64_FT<string opstr, RegisterOperand DstRC, RegisterOperand SrcRC> :
  InstSE<(outs DstRC:$fs), (ins DstRC:$fs_in, SrcRC:$rt),
         !strconcat(opstr, "\t$rt, $fs"), [], FrmFR, opstr>, HARDFLOAT {
  // $fs_in is part of a white lie to work around a widespread bug in the FPU
  // implementation. See expandBuildPairF64 for details.
  let Constraints = "$fs = $fs_in";
}
```
- EN: Declares reusable TableGen class `MTC1_64_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `MTC1_64_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 197-204
```tablegen
class LW_FT<string opstr, RegisterOperand RC, DAGOperand MO,
            SDPatternOperator OpNode = null_frag> :
  InstSE<(outs RC:$rt), (ins MO:$addr), !strconcat(opstr, "\t$rt, $addr"),
         [(set RC:$rt, (OpNode addrDefault:$addr))], FrmFI, opstr>,
  HARDFLOAT {
  let DecoderMethod = "DecodeFMem";
  let mayLoad = 1;
}
```
- EN: Declares reusable TableGen class `LW_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `LW_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 206-212
```tablegen
class SW_FT<string opstr, RegisterOperand RC, DAGOperand MO,
            SDPatternOperator OpNode = null_frag> :
  InstSE<(outs), (ins RC:$rt, MO:$addr), !strconcat(opstr, "\t$rt, $addr"),
         [(OpNode RC:$rt, addrDefault:$addr)], FrmFI, opstr>, HARDFLOAT {
  let DecoderMethod = "DecodeFMem";
  let mayStore = 1;
}
```
- EN: Declares reusable TableGen class `SW_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `SW_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 214-219
```tablegen
class MADDS_FT<string opstr, RegisterOperand RC,
               SDPatternOperator OpNode = null_frag> :
  InstSE<(outs RC:$fd), (ins RC:$fr, RC:$fs, RC:$ft),
         !strconcat(opstr, "\t$fd, $fr, $fs, $ft"),
         [(set RC:$fd, (OpNode (any_fmul RC:$fs, RC:$ft), RC:$fr))],
         FrmFR, opstr>, HARDFLOAT;
```
- EN: Declares reusable TableGen class `MADDS_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `MADDS_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 221-230
```tablegen
class NMADDS_FT_PatFrag<SDPatternOperator OpNode> :
  PatFrag<(ops node:$fr, node:$fs, node:$ft),
          (any_fsub fpimm0, (OpNode (any_fmul node:$fs, node:$ft), node:$fr)),
          [{ return N->getFlags().hasNoNaNs(); }]>;
class NMADDS_FT<string opstr, RegisterOperand RC,
                SDPatternOperator OpNode = null_frag> :
  InstSE<(outs RC:$fd), (ins RC:$fr, RC:$fs, RC:$ft),
         !strconcat(opstr, "\t$fd, $fr, $fs, $ft"),
         [(set RC:$fd, (any_fsub fpimm0, (OpNode (any_fmul RC:$fs, RC:$ft), RC:$fr)))],
         FrmFR, opstr>, HARDFLOAT;
```
- EN: Declares reusable TableGen class `NMADDS_FT_PatFrag` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `NMADDS_FT_PatFrag`，通常用于抽象共享字段、谓词或编码结构。

### Lines 232-239
```tablegen
class LWXC1_FT<string opstr, RegisterOperand DRC,
               SDPatternOperator OpNode = null_frag> :
  InstSE<(outs DRC:$fd), (ins PtrRC:$base, PtrRC:$index),
         !strconcat(opstr, "\t$fd, ${index}(${base})"),
         [(set DRC:$fd, (OpNode (add iPTR:$base, iPTR:$index)))],
         FrmFI, opstr>, HARDFLOAT {
  let AddedComplexity = 20;
}
```
- EN: Declares reusable TableGen class `LWXC1_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `LWXC1_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 241-248
```tablegen
class SWXC1_FT<string opstr, RegisterOperand DRC,
               SDPatternOperator OpNode = null_frag> :
  InstSE<(outs), (ins DRC:$fs, PtrRC:$base, PtrRC:$index),
         !strconcat(opstr, "\t$fs, ${index}(${base})"),
         [(OpNode DRC:$fs, (add iPTR:$base, iPTR:$index))],
         FrmFI, opstr>, HARDFLOAT {
  let AddedComplexity = 20;
}
```
- EN: Declares reusable TableGen class `SWXC1_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `SWXC1_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 250-261
```tablegen
class BC1F_FT<string opstr, DAGOperand opnd,
              SDPatternOperator Op = null_frag> :
  InstSE<(outs), (ins FCCRegsOpnd:$fcc, opnd:$offset),
         !strconcat(opstr, "\t$fcc, $offset"),
         [(MipsFPBrcond Op, FCCRegsOpnd:$fcc, bb:$offset)],
         FrmFI, opstr>, HARDFLOAT {
  let isBranch = 1;
  let isTerminator = 1;
  let hasDelaySlot = 1;
  let Defs = [AT];
  let hasFCCRegOperand = 1;
}
```
- EN: Declares reusable TableGen class `BC1F_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `BC1F_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 263-272
```tablegen
class BC1XL_FT<string opstr, DAGOperand opnd> :
  InstSE<(outs), (ins FCCRegsOpnd:$fcc, opnd:$offset),
         !strconcat(opstr, "\t$fcc, $offset"), [],
         FrmFI, opstr>, HARDFLOAT {
  let isBranch = 1;
  let isTerminator = 1;
  let hasDelaySlot = 1;
  let Defs = [AT];
  let hasFCCRegOperand = 1;
}
```
- EN: Declares reusable TableGen class `BC1XL_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `BC1XL_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 274-283
```tablegen
class CEQS_FT<string typestr, RegisterClass RC,
              SDPatternOperator OpNode = null_frag>  :
  InstSE<(outs), (ins RC:$fs, RC:$ft, condcode:$cond),
         !strconcat("c.$cond.", typestr, "\t$fs, $ft"),
         [(OpNode RC:$fs, RC:$ft, imm:$cond)], FrmFR,
         !strconcat("c.$cond.", typestr)>, HARDFLOAT {
  let Defs = [FCC0];
  let isCodeGenOnly = 1;
  let hasFCCRegOperand = 1;
}
```
- EN: Declares reusable TableGen class `CEQS_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `CEQS_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 286-296
```tablegen
// Note: MIPS-IV introduced $fcc1-$fcc7 and renamed FCSR31[23] $fcc0. Rather
//       duplicating the instruction definition for MIPS1 - MIPS3, we expand
//       c.cond.ft if necessary, and reject it after constructing the
//       instruction if the ISA doesn't support it.
class C_COND_FT<string CondStr, string Typestr, RegisterOperand RC>  :
   InstSE<(outs FCCRegsOpnd:$fcc), (ins RC:$fs, RC:$ft),
          !strconcat("c.", CondStr, ".", Typestr, "\t$fcc, $fs, $ft"), [],
          FrmFR>, HARDFLOAT {
  let isCompare = 1;
  let hasFCCRegOperand = 1;
}
```
- EN: Declares reusable TableGen class `C_COND_FT` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `C_COND_FT`，通常用于抽象共享字段、谓词或编码结构。

### Lines 298-315
```tablegen
multiclass C_COND_M<string TypeStr, RegisterOperand RC, bits<5> fmt> {
  def C_F_#NAME : MMRel, C_COND_FT<"f", TypeStr, RC>,
                  C_COND_FM<fmt, 0> {
    let BaseOpcode = "c.f."#NAME;
    let isCommutable = 1;
  }
  def C_EQ_#NAME : MMRel, C_COND_FT<"eq", TypeStr, RC>,
                   C_COND_FM<fmt, 2> {
    let BaseOpcode = "c.eq."#NAME;
    let isCommutable = 1;
  }
  def C_OLT_#NAME : MMRel, C_COND_FT<"olt", TypeStr, RC>,
                    C_COND_FM<fmt, 4> {
    let BaseOpcode = "c.olt."#NAME;
  }
  def C_OLE_#NAME : MMRel, C_COND_FT<"ole", TypeStr, RC>,
                    C_COND_FM<fmt, 6> {
    let BaseOpcode = "c.ole."#NAME;
```
- EN: Declares TableGen `multiclass C_COND_M`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass C_COND_M`，它是一个可复用模板，可展开为多个相关记录。

### Lines 316-317
```tablegen
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 319-336
```tablegen
multiclass C_COND_NOTR5900_M<string TypeStr, RegisterOperand RC, bits<5> fmt> {
  def C_UN_#NAME : MMRel, C_COND_FT<"un", TypeStr, RC>,
                   C_COND_FM<fmt, 1> {
    let BaseOpcode = "c.un."#NAME;
    let isCommutable = 1;
  }
  def C_UEQ_#NAME : MMRel, C_COND_FT<"ueq", TypeStr, RC>,
                    C_COND_FM<fmt, 3> {
    let BaseOpcode = "c.ueq."#NAME;
    let isCommutable = 1;
  }
  def C_ULT_#NAME : MMRel, C_COND_FT<"ult", TypeStr, RC>,
                    C_COND_FM<fmt, 5> {
    let BaseOpcode = "c.ult."#NAME;
  }
  def C_ULE_#NAME : MMRel, C_COND_FT<"ule", TypeStr, RC>,
                     C_COND_FM<fmt, 7> {
    let BaseOpcode = "c.ule."#NAME;
```
- EN: Declares TableGen `multiclass C_COND_NOTR5900_M`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass C_COND_NOTR5900_M`，它是一个可复用模板，可展开为多个相关记录。

### Lines 337-354
```tablegen
  }
  let mayRaiseFPException = 1 in {
    def C_SF_#NAME : MMRel, C_COND_FT<"sf", TypeStr, RC>,
                     C_COND_FM<fmt, 8> {
      let BaseOpcode = "c.sf."#NAME;
      let isCommutable = 1;
      let mayRaiseFPException = 1;
    }
    def C_NGLE_#NAME : MMRel, C_COND_FT<"ngle", TypeStr, RC>,
                       C_COND_FM<fmt, 9> {
      let BaseOpcode = "c.ngle."#NAME;
      let mayRaiseFPException = 1;
    }
    def C_SEQ_#NAME : MMRel, C_COND_FT<"seq", TypeStr, RC>,
                      C_COND_FM<fmt, 10> {
      let BaseOpcode = "c.seq."#NAME;
      let isCommutable = 1;
      let mayRaiseFPException = 1;
```
- EN: Defines TableGen record `C_SF_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `C_SF_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 355-372
```tablegen
    }
    def C_NGL_#NAME : MMRel, C_COND_FT<"ngl", TypeStr, RC>,
                      C_COND_FM<fmt, 11> {
      let BaseOpcode = "c.ngl."#NAME;
      let mayRaiseFPException = 1;
    }
    def C_LT_#NAME : MMRel, C_COND_FT<"lt", TypeStr, RC>,
                     C_COND_FM<fmt, 12> {
      let BaseOpcode = "c.lt."#NAME;
      let mayRaiseFPException = 1;
    }
    def C_NGE_#NAME : MMRel, C_COND_FT<"nge", TypeStr, RC>,
                      C_COND_FM<fmt, 13> {
      let BaseOpcode = "c.nge."#NAME;
      let mayRaiseFPException = 1;
    }
    def C_LE_#NAME : MMRel, C_COND_FT<"le", TypeStr, RC>,
                     C_COND_FM<fmt, 14> {
```
- EN: Defines TableGen record `C_NGL_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `C_NGL_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 373-382
```tablegen
      let BaseOpcode = "c.le."#NAME;
      let mayRaiseFPException = 1;
    }
    def C_NGT_#NAME : MMRel, C_COND_FT<"ngt", TypeStr, RC>,
                      C_COND_FM<fmt, 15> {
      let BaseOpcode = "c.ngt."#NAME;
      let mayRaiseFPException = 1;
    }
  }
}
```
- EN: Defines TableGen record `C_NGT_` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `C_NGT_`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 384-401
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
defm S : C_COND_M<"s", FGR32Opnd, 16>, ISA_MIPS1_NOT_32R6_64R6;
defm S : C_COND_NOTR5900_M<"s", FGR32Opnd, 16>,
         ISA_MIPS1_NOT_32R6_64R6_R5900;
defm D32 : C_COND_M<"d", AFGR64Opnd, 17>, ISA_MIPS1_NOT_32R6_64R6,
           FGR_32;
defm D32 : C_COND_NOTR5900_M<"d", AFGR64Opnd, 17>,
           ISA_MIPS1_NOT_32R6_64R6, FGR_32;
let DecoderNamespace = "MipsFP64" in {
defm D64 : C_COND_M<"d", FGR64Opnd, 17>, ISA_MIPS1_NOT_32R6_64R6,
           FGR_64;
defm D64 : C_COND_NOTR5900_M<"d", FGR64Opnd, 17>,
           ISA_MIPS1_NOT_32R6_64R6, FGR_64;
}
}
//===----------------------------------------------------------------------===//
// Floating Point Instructions
//===----------------------------------------------------------------------===//
```
- EN: Defines TableGen record `S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 402-415
```tablegen
let AdditionalPredicates = [NotInMicroMips], mayRaiseFPException = 1 in {
  def ROUND_W_S : MMRel, StdMMR6Rel,
                  ABSS_FT<"round.w.s", FGR32Opnd, FGR32Opnd>,
                  ABSS_FM<0xc, 16>, ISA_MIPS2_NOT_R5900;
  defm ROUND_W  : ROUND_M<"round.w.d">, ABSS_FM<0xc, 17>, ISA_MIPS2;
  def TRUNC_W_S : MMRel, StdMMR6Rel,
                  ABSS_FT<"trunc.w.s", FGR32Opnd, FGR32Opnd>,
                  ABSS_FM<0xd, 16>, ISA_MIPS2_NOT_R5900;
  def CEIL_W_S  : MMRel, StdMMR6Rel,
                  ABSS_FT<"ceil.w.s", FGR32Opnd, FGR32Opnd>,
                  ABSS_FM<0xe, 16>, ISA_MIPS2_NOT_R5900;
  def FLOOR_W_S : MMRel, StdMMR6Rel,
                  ABSS_FT<"floor.w.s", FGR32Opnd, FGR32Opnd>,
                  ABSS_FM<0xf, 16>, ISA_MIPS2_NOT_R5900;
```
- EN: Defines TableGen record `ROUND_W_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ROUND_W_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 417-419
```tablegen
  defm TRUNC_W : ROUND_M<"trunc.w.d">, ABSS_FM<0xd, 17>, ISA_MIPS2;
  defm CEIL_W  : ROUND_M<"ceil.w.d">, ABSS_FM<0xe, 17>, ISA_MIPS2;
  defm FLOOR_W : ROUND_M<"floor.w.d">, ABSS_FM<0xf, 17>, ISA_MIPS2;
```
- EN: Defines TableGen record `TRUNC_W` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TRUNC_W`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 421-426
```tablegen
  let Uses = [FCR31] in {
    def CVT_W_S   : MMRel, ABSS_FT<"cvt.w.s", FGR32Opnd, FGR32Opnd>, // fp
                    ABSS_FM<0x24, 16>, ISA_MIPS1;
    defm CVT_W   : ROUND_M<"cvt.w.d">, ABSS_FM<0x24, 17>, ISA_MIPS1;
  }
}
```
- EN: Defines TableGen record `CVT_W_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CVT_W_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 428-445
```tablegen
let AdditionalPredicates = [NotInMicroMips], mayRaiseFPException = 1, Uses = [FCR31] in {
  def RECIP_S : MMRel, ABSS_FT<"recip.s", FGR32Opnd, FGR32Opnd>,
                ABSS_FM<0b010101, 0x10>, INSN_MIPS4_32R2;
  def RECIP_D32 : MMRel, ABSS_FT<"recip.d", AFGR64Opnd, AFGR64Opnd>,
                  ABSS_FM<0b010101, 0x11>, INSN_MIPS4_32R2, FGR_32 {
    let BaseOpcode = "RECIP_D32";
  }
  let DecoderNamespace = "MipsFP64" in
    def RECIP_D64 : MMRel, ABSS_FT<"recip.d", FGR64Opnd, FGR64Opnd>,
                    ABSS_FM<0b010101, 0x11>,
                    INSN_MIPS4_32R2, FGR_64;
  def RSQRT_S : MMRel, ABSS_FT<"rsqrt.s", FGR32Opnd, FGR32Opnd>,
                ABSS_FM<0b010110, 0x10>, INSN_MIPS4_32R2;
  def RSQRT_D32 : MMRel, ABSS_FT<"rsqrt.d", AFGR64Opnd, AFGR64Opnd>,
                  ABSS_FM<0b010110, 0x11>, INSN_MIPS4_32R2, FGR_32 {
    let BaseOpcode = "RSQRT_D32";
  }
  let DecoderNamespace = "MipsFP64" in
```
- EN: Defines TableGen record `RECIP_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RECIP_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 446-449
```tablegen
    def RSQRT_D64 : MMRel, ABSS_FT<"rsqrt.d", FGR64Opnd, FGR64Opnd>,
                    ABSS_FM<0b010110, 0x11>,
                    INSN_MIPS4_32R2, FGR_64;
}
```
- EN: Defines TableGen record `RSQRT_D64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RSQRT_D64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 451-468
```tablegen
let DecoderNamespace = "MipsFP64", mayRaiseFPException = 1 in {
  let AdditionalPredicates = [NotInMicroMips] in {
  def ROUND_L_S : ABSS_FT<"round.l.s", FGR64Opnd, FGR32Opnd>,
                  ABSS_FM<0x8, 16>, ISA_MIPS2, FGR_64;
  def ROUND_L_D64 : ABSS_FT<"round.l.d", FGR64Opnd, FGR64Opnd>,
                    ABSS_FM<0x8, 17>, INSN_MIPS3_32, FGR_64;
  def TRUNC_L_S : ABSS_FT<"trunc.l.s", FGR64Opnd, FGR32Opnd>,
                  ABSS_FM<0x9, 16>, ISA_MIPS2, FGR_64;
  def TRUNC_L_D64 : ABSS_FT<"trunc.l.d", FGR64Opnd, FGR64Opnd>,
                    ABSS_FM<0x9, 17>, INSN_MIPS3_32, FGR_64;
  def CEIL_L_S  : ABSS_FT<"ceil.l.s", FGR64Opnd, FGR32Opnd>,
                  ABSS_FM<0xa, 16>, ISA_MIPS2, FGR_64;
  def CEIL_L_D64 : ABSS_FT<"ceil.l.d", FGR64Opnd, FGR64Opnd>,
                   ABSS_FM<0xa, 17>, INSN_MIPS3_32, FGR_64;
  def FLOOR_L_S : ABSS_FT<"floor.l.s", FGR64Opnd, FGR32Opnd>,
                  ABSS_FM<0xb, 16>, ISA_MIPS2, FGR_64;
  def FLOOR_L_D64 : ABSS_FT<"floor.l.d", FGR64Opnd, FGR64Opnd>,
                    ABSS_FM<0xb, 17>, INSN_MIPS3_32, FGR_64;
```
- EN: Defines TableGen record `ROUND_L_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ROUND_L_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 469-470
```tablegen
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 472-479
```tablegen
let AdditionalPredicates = [NotInMicroMips], mayRaiseFPException = 1, Uses = [FCR31] in{
  def CVT_S_W : MMRel, ABSS_FT<"cvt.s.w", FGR32Opnd, FGR32Opnd>,
                ABSS_FM<0x20, 20>, ISA_MIPS1;
  def CVT_L_S : MMRel, ABSS_FT<"cvt.l.s", FGR64Opnd, FGR32Opnd>,
                ABSS_FM<0x25, 16>, INSN_MIPS3_32R2;
  def CVT_L_D64: MMRel, ABSS_FT<"cvt.l.d", FGR64Opnd, FGR64Opnd>,
                 ABSS_FM<0x25, 17>, INSN_MIPS3_32R2;
}
```
- EN: Defines TableGen record `CVT_S_W` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CVT_S_W`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 481-488
```tablegen
let AdditionalPredicates = [NotInMicroMips], mayRaiseFPException = 1, Uses = [FCR31] in {
  def CVT_S_D32 : MMRel, ABSS_FT<"cvt.s.d", FGR32Opnd, AFGR64Opnd>,
                  ABSS_FM<0x20, 17>, ISA_MIPS1, FGR_32;
  def CVT_D32_S : MMRel, ABSS_FT<"cvt.d.s", AFGR64Opnd, FGR32Opnd>,
                  ABSS_FM<0x21, 16>, ISA_MIPS1, FGR_32;
  def CVT_D32_W : MMRel, ABSS_FT<"cvt.d.w", AFGR64Opnd, FGR32Opnd>,
                  ABSS_FM<0x21, 20>, ISA_MIPS1, FGR_32;
}
```
- EN: Defines TableGen record `CVT_S_D32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CVT_S_D32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 490-504
```tablegen
let DecoderNamespace = "MipsFP64", AdditionalPredicates = [NotInMicroMips] in {
  let mayRaiseFPException = 1, Uses = [FCR31] in {
    def FADD_PS64   : ADDS_FT<"add.ps", FGR64Opnd, 0>,
                      ADDS_FM<0x0, 22>,
                      ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
    def FMUL_PS64   : ADDS_FT<"mul.ps", FGR64Opnd, 0>,
                      ADDS_FM<0x2, 22>,
                      ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
    def FSUB_PS64   : ADDS_FT<"sub.ps", FGR64Opnd, 0>,
                      ADDS_FM<0x1, 22>,
                      ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
    def CVT_PS_S64  : CVT_PS_S_FT<"cvt.ps.s", FGR64Opnd, FGR32Opnd, 0>,
                      ADDS_FM<0x26, 16>,
                      ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
  }
```
- EN: Defines TableGen record `FADD_PS64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FADD_PS64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 506-523
```tablegen
  def PLL_PS64    : ADDS_FT<"pll.ps", FGR64Opnd, 0>,
                    ADDS_FM<0x2C, 22>,
                    ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
  def PLU_PS64    : ADDS_FT<"plu.ps", FGR64Opnd, 0>,
                    ADDS_FM<0x2D, 22>,
                    ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
  def PUL_PS64    : ADDS_FT<"pul.ps", FGR64Opnd, 0>,
                    ADDS_FM<0x2E, 22>,
                    ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
  def PUU_PS64    : ADDS_FT<"puu.ps", FGR64Opnd, 0>,
                    ADDS_FM<0x2F, 22>,
                    ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
  def CVT_S_PU64  : ABSS_FT<"cvt.s.pu", FGR32Opnd, FGR64Opnd>,
                    ABSS_FM<0x20, 22>,
                    ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
  def CVT_S_PL64  : ABSS_FT<"cvt.s.pl", FGR32Opnd, FGR64Opnd>,
                    ABSS_FM<0x28, 22>,
                    ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
```
- EN: Defines TableGen record `PLL_PS64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PLL_PS64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 524-524
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 526-539
```tablegen
let DecoderNamespace = "MipsFP64", mayRaiseFPException = 1, Uses = [FCR31] in {
  let AdditionalPredicates = [HasMips3D] in {
    def ADDR_PS64   : ADDS_FT<"addr.ps", FGR64Opnd, 0>,
                      ADDS_FM<0x18, 22>, ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
    def MULR_PS64   : ADDS_FT<"mulr.ps", FGR64Opnd, 0>,
                      ADDS_FM<0x1a, 22>, ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
    def CVT_PS_PW64 : ABSS_FT<"cvt.ps.pw", FGR64Opnd, FGR64Opnd>,
                      ABSS_FM<0x26, 20>,
                      ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
    def CVT_PW_PS64 : ABSS_FT<"cvt.pw.ps", FGR64Opnd, FGR64Opnd>,
                      ABSS_FM<0x24, 22>,
                      ISA_MIPS32R2_NOT_32R6_64R6, FGR_64;
  }
}
```
- EN: Defines TableGen record `ADDR_PS64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADDR_PS64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 541-554
```tablegen
let DecoderNamespace = "MipsFP64", mayRaiseFPException = 1, Uses = [FCR31] in {
  let AdditionalPredicates = [NotInMicroMips] in {
    def CVT_S_L   : ABSS_FT<"cvt.s.l", FGR32Opnd, FGR64Opnd>,
                    ABSS_FM<0x20, 21>, INSN_MIPS3_32R2, FGR_64;
    def CVT_S_D64 : ABSS_FT<"cvt.s.d", FGR32Opnd, FGR64Opnd>,
                    ABSS_FM<0x20, 17>, ISA_MIPS1, FGR_64;
    def CVT_D64_W : ABSS_FT<"cvt.d.w", FGR64Opnd, FGR32Opnd>,
                    ABSS_FM<0x21, 20>, ISA_MIPS1, FGR_64;
    def CVT_D64_S : ABSS_FT<"cvt.d.s", FGR64Opnd, FGR32Opnd>,
                    ABSS_FM<0x21, 16>, ISA_MIPS1, FGR_64;
    def CVT_D64_L : ABSS_FT<"cvt.d.l", FGR64Opnd, FGR64Opnd>,
                    ABSS_FM<0x21, 21>, INSN_MIPS3_32R2, FGR_64;
  }
}
```
- EN: Defines TableGen record `CVT_S_L` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CVT_S_L`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 556-564
```tablegen
let isPseudo = 1, isCodeGenOnly = 1 in {
  def PseudoCVT_S_W : ABSS_FT<"", FGR32Opnd, GPR32Opnd>;
  let Predicates = [IsNotSingleFloat] in {
    def PseudoCVT_D32_W : ABSS_FT<"", AFGR64Opnd, GPR32Opnd>;
    def PseudoCVT_S_L : ABSS_FT<"", FGR64Opnd, GPR64Opnd>;
    def PseudoCVT_D64_W : ABSS_FT<"", FGR64Opnd, GPR32Opnd>;
    def PseudoCVT_D64_L : ABSS_FT<"", FGR64Opnd, GPR64Opnd>;
  }
}
```
- EN: Defines TableGen record `PseudoCVT_S_W` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoCVT_S_W`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 566-576
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def FABS_S : MMRel, ABSS_FT<"abs.s", FGR32Opnd, FGR32Opnd, fabs>,
               ABSS_FM<0x5, 16>, ISA_MIPS1;
  defm FABS : ABSS_M<"abs.d", fabs>, ABSS_FM<0x5, 17>, ISA_MIPS1;
}
let Predicates = FABS_S.Predicates in
def : Pat<(ABSS_FT_PatFrag<fabs> FGR32Opnd:$fs), (FABS_S FGR32Opnd:$fs)>;
let Predicates = FABS_D32.Predicates in
def : Pat<(ABSS_FT_PatFrag<fabs> AFGR64Opnd:$fs), (FABS_D32 AFGR64Opnd:$fs)>;
let Predicates = FABS_D64.Predicates in
def : Pat<(ABSS_FT_PatFrag<fabs> FGR64Opnd:$fs), (FABS_D64 FGR64Opnd:$fs)>;
```
- EN: Defines TableGen record `FABS_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FABS_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 578-582
```tablegen
def FNEG_S : MMRel, ABSS_FT<"neg.s", FGR32Opnd, FGR32Opnd, fneg>,
             ABSS_FM<0x7, 16>, ISA_MIPS1;
let AdditionalPredicates = [NotInMicroMips] in {
  defm FNEG : ABSS_M<"neg.d", fneg>, ABSS_FM<0x7, 17>, ISA_MIPS1;
}
```
- EN: Defines TableGen record `FNEG_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FNEG_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 584-588
```tablegen
let AdditionalPredicates = [NotInMicroMips], mayRaiseFPException = 1, Uses = [FCR31] in {
  def FSQRT_S : MMRel, StdMMR6Rel, ABSS_FT<"sqrt.s", FGR32Opnd, FGR32Opnd,
                any_fsqrt>, ABSS_FM<0x4, 16>, ISA_MIPS2;
  defm FSQRT : ABSS_M<"sqrt.d", any_fsqrt>, ABSS_FM<0x4, 17>, ISA_MIPS2;
}
```
- EN: Defines TableGen record `FSQRT_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FSQRT_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 590-593
```tablegen
// The odd-numbered registers are only referenced when doing loads,
// stores, and moves between floating-point and integer registers.
// When defining instructions, we reference all 32-bit registers,
// regardless of register aliasing.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 595-600
```tablegen
/// Move Control Registers From/To CPU Registers
let AdditionalPredicates = [NotInMicroMips] in {
  def CFC1 : MMRel, MFC1_FT<"cfc1", GPR32Opnd, CCROpnd>, MFC1_FM<2>,
             ISA_MIPS1;
  def CTC1 : MMRel, MTC1_FT<"ctc1", CCROpnd, GPR32Opnd>, MFC1_FM<6>,
             ISA_MIPS1;
```
- EN: Defines TableGen record `CFC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CFC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 602-613
```tablegen
  def MFC1 : MMRel, StdMMR6Rel, MFC1_FT<"mfc1", GPR32Opnd, FGR32Opnd,
                                        bitconvert>, MFC1_FM<0>, ISA_MIPS1;
  def MFC1_D64 : MFC1_FT<"mfc1", GPR32Opnd, FGR64Opnd>, MFC1_FM<0>,
                 ISA_MIPS1, FGR_64 {
    let DecoderNamespace = "MipsFP64";
  }
  def MTC1 : MMRel, StdMMR6Rel, MTC1_FT<"mtc1", FGR32Opnd, GPR32Opnd,
                                        bitconvert>, MFC1_FM<4>, ISA_MIPS1;
  def MTC1_D64 : MTC1_FT<"mtc1", FGR64Opnd, GPR32Opnd>, MFC1_FM<4>,
                 ISA_MIPS1, FGR_64 {
    let DecoderNamespace = "MipsFP64";
  }
```
- EN: Defines TableGen record `MFC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 615-620
```tablegen
  def MFHC1_D32 : MMRel, MFC1_FT<"mfhc1", GPR32Opnd, AFGR64Opnd>,
                  MFC1_FM<3>, ISA_MIPS32R2, FGR_32;
  def MFHC1_D64 : MFC1_FT<"mfhc1", GPR32Opnd, FGR64Opnd>,
                  MFC1_FM<3>, ISA_MIPS32R2, FGR_64 {
    let DecoderNamespace = "MipsFP64";
  }
```
- EN: Defines TableGen record `MFHC1_D32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MFHC1_D32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 622-628
```tablegen
  def MTHC1_D32 : MMRel, StdMMR6Rel,
                  MTC1_64_FT<"mthc1", AFGR64Opnd, GPR32Opnd>,
                  MFC1_FM<7>, ISA_MIPS32R2, FGR_32;
  def MTHC1_D64 : MTC1_64_FT<"mthc1", FGR64Opnd, GPR32Opnd>,
                  MFC1_FM<7>, ISA_MIPS32R2, FGR_64 {
    let DecoderNamespace = "MipsFP64";
  }
```
- EN: Defines TableGen record `MTHC1_D32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTHC1_D32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 630-643
```tablegen
  let AdditionalPredicates = [IsNotSingleFloat] in {
    def DMTC1 : MTC1_FT<"dmtc1", FGR64Opnd, GPR64Opnd, bitconvert>,
                MFC1_FM<5>,
                ISA_MIPS3;
    def DMFC1 : MFC1_FT<"dmfc1", GPR64Opnd, FGR64Opnd, bitconvert>,
                MFC1_FM<1>,
                ISA_MIPS3;
  }
  let isMoveReg = 1 in {
    def FMOV_S   : MMRel, ABSS_FT<"mov.s", FGR32Opnd, FGR32Opnd>,
                   ABSS_FM<0x6, 16>, ISA_MIPS1;
    defm FMOV : ABSS_M<"mov.d">, ABSS_FM<0x6, 17>, ISA_MIPS1;
  } // isMoveReg
}
```
- EN: Defines TableGen record `DMTC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DMTC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 645-651
```tablegen
/// Floating Point Memory Instructions
let AdditionalPredicates = [NotInMicroMips] in {
  def LWC1 : MMRel, LW_FT<"lwc1", FGR32Opnd, mem_simm16, load>,
             LW_FM<0x31>, ISA_MIPS1;
  def SWC1 : MMRel, SW_FT<"swc1", FGR32Opnd, mem_simm16, store>,
             LW_FM<0x39>, ISA_MIPS1;
}
```
- EN: Defines TableGen record `LWC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 653-660
```tablegen
let DecoderNamespace = "MipsFP64", AdditionalPredicates = [NotInMicroMips] in {
  def LDC164 : StdMMR6Rel, LW_FT<"ldc1", FGR64Opnd, mem_simm16, load>,
               LW_FM<0x35>, ISA_MIPS2, FGR_64 {
    let BaseOpcode = "LDC164";
  }
  def SDC164 : StdMMR6Rel, SW_FT<"sdc1", FGR64Opnd, mem_simm16, store>,
               LW_FM<0x3d>, ISA_MIPS2, FGR_64;
}
```
- EN: Defines TableGen record `LDC164` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDC164`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 662-669
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def LDC1 : MMRel, StdMMR6Rel, LW_FT<"ldc1", AFGR64Opnd, mem_simm16,
                                      load>, LW_FM<0x35>, ISA_MIPS2, FGR_32 {
    let BaseOpcode = "LDC132";
  }
  def SDC1 : MMRel, SW_FT<"sdc1", AFGR64Opnd, mem_simm16, store>,
             LW_FM<0x3d>, ISA_MIPS2, FGR_32;
}
```
- EN: Defines TableGen record `LDC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 671-677
```tablegen
// Indexed loads and stores.
// Base register + offset register addressing mode (indicated by "x" in the
// instruction mnemonic).
def LWXC1 : MMRel, LWXC1_FT<"lwxc1", FGR32Opnd, load>, LWXC1_FM<0>,
            INSN_MIPS4_32R2_NOT_32R6_64R6;
def SWXC1 : MMRel, SWXC1_FT<"swxc1", FGR32Opnd, store>, SWXC1_FM<8>,
            INSN_MIPS4_32R2_NOT_32R6_64R6;
```
- EN: Defines TableGen record `LWXC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWXC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 679-684
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def LDXC1 : LWXC1_FT<"ldxc1", AFGR64Opnd, load>, LWXC1_FM<1>,
              INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_32;
  def SDXC1 : SWXC1_FT<"sdxc1", AFGR64Opnd, store>, SWXC1_FM<9>,
              INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_32;
}
```
- EN: Defines TableGen record `LDXC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDXC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 686-691
```tablegen
let DecoderNamespace="MipsFP64" in {
  def LDXC164 : LWXC1_FT<"ldxc1", FGR64Opnd, load>, LWXC1_FM<1>,
                INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_64;
  def SDXC164 : SWXC1_FT<"sdxc1", FGR64Opnd, store>, SWXC1_FM<9>,
                INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_64;
}
```
- EN: Defines TableGen record `LDXC164` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDXC164`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 693-700
```tablegen
// Load/store doubleword indexed unaligned.
// FIXME: This instruction should not be defined for FGR_32.
let AdditionalPredicates = [NotInMicroMips] in {
  def LUXC1 : MMRel, LWXC1_FT<"luxc1", AFGR64Opnd>, LWXC1_FM<0x5>,
              INSN_MIPS5_32R2_NOT_32R6_64R6, FGR_32;
  def SUXC1 : MMRel, SWXC1_FT<"suxc1", AFGR64Opnd>, SWXC1_FM<0xd>,
              INSN_MIPS5_32R2_NOT_32R6_64R6, FGR_32;
}
```
- EN: Defines TableGen record `LUXC1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LUXC1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 702-708
```tablegen
let AdditionalPredicates = [NotInMicroMips],
    DecoderNamespace="MipsFP64" in {
  def LUXC164 : LWXC1_FT<"luxc1", FGR64Opnd>, LWXC1_FM<0x5>,
                INSN_MIPS5_32R2_NOT_32R6_64R6, FGR_64;
  def SUXC164 : SWXC1_FT<"suxc1", FGR64Opnd>, SWXC1_FM<0xd>,
                INSN_MIPS5_32R2_NOT_32R6_64R6, FGR_64;
}
```
- EN: Defines TableGen record `LUXC164` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LUXC164`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 710-727
```tablegen
/// Floating-point Arithmetic
let AdditionalPredicates = [NotInMicroMips], mayRaiseFPException = 1, Uses = [FCR31] in {
  def FADD_S : MMRel, ADDS_FT<"add.s", FGR32Opnd, 1, any_fadd>,
               ADDS_FM<0x00, 16>, ISA_MIPS1;
  defm FADD :  ADDS_M<"add.d", 1, any_fadd>, ADDS_FM<0x00, 17>,
               ISA_MIPS1;
  def FDIV_S : MMRel, ADDS_FT<"div.s", FGR32Opnd, 0, any_fdiv>,
               ADDS_FM<0x03, 16>, ISA_MIPS1;
  defm FDIV :  ADDS_M<"div.d", 0, any_fdiv>, ADDS_FM<0x03, 17>,
               ISA_MIPS1;
  def FMUL_S : MMRel, ADDS_FT<"mul.s", FGR32Opnd, 1, any_fmul>,
               ADDS_FM<0x02, 16>, ISA_MIPS1;
  defm FMUL :  ADDS_M<"mul.d", 1, any_fmul>, ADDS_FM<0x02, 17>,
               ISA_MIPS1;
  def FSUB_S : MMRel, ADDS_FT<"sub.s", FGR32Opnd, 0, any_fsub>,
               ADDS_FM<0x01, 16>, ISA_MIPS1;
  defm FSUB :  ADDS_M<"sub.d", 0, any_fsub>, ADDS_FM<0x01, 17>,
               ISA_MIPS1;
```
- EN: Defines TableGen record `FADD_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FADD_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 728-728
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 730-734
```tablegen
let AdditionalPredicates = [NotInMicroMips, HasMadd4], mayRaiseFPException = 1, Uses = [FCR31] in {
  def MADD_S : MMRel, MADDS_FT<"madd.s", FGR32Opnd, any_fadd>,
               MADDS_FM<4, 0>, INSN_MIPS4_32R2_NOT_32R6_64R6;
  def MSUB_S : MMRel, MADDS_FT<"msub.s", FGR32Opnd, any_fsub>,
               MADDS_FM<5, 0>, INSN_MIPS4_32R2_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MADD_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MADD_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 736-739
```tablegen
  def MADD_D32 : MMRel, MADDS_FT<"madd.d", AFGR64Opnd, any_fadd>,
                 MADDS_FM<4, 1>, INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_32;
  def MSUB_D32 : MMRel, MADDS_FT<"msub.d", AFGR64Opnd, any_fsub>,
                 MADDS_FM<5, 1>, INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_32;
```
- EN: Defines TableGen record `MADD_D32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MADD_D32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 741-747
```tablegen
  let DecoderNamespace = "MipsFP64" in {
    def MADD_D64 : MADDS_FT<"madd.d", FGR64Opnd, any_fadd>,
                   MADDS_FM<4, 1>, INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_64;
    def MSUB_D64 : MADDS_FT<"msub.d", FGR64Opnd, any_fsub>,
                   MADDS_FM<5, 1>, INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_64;
  }
}
```
- EN: Defines TableGen record `MADD_D64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MADD_D64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 749-753
```tablegen
let AdditionalPredicates = [HasMadd4, NotInMicroMips], mayRaiseFPException = 1, Uses = [FCR31] in {
  def NMADD_S : MMRel, NMADDS_FT<"nmadd.s", FGR32Opnd>,
                MADDS_FM<6, 0>, INSN_MIPS4_32R2_NOT_32R6_64R6;
  def NMSUB_S : MMRel, NMADDS_FT<"nmsub.s", FGR32Opnd>,
                MADDS_FM<7, 0>, INSN_MIPS4_32R2_NOT_32R6_64R6;
```
- EN: Defines TableGen record `NMADD_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NMADD_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 755-758
```tablegen
  def NMADD_D32 : MMRel, NMADDS_FT<"nmadd.d", AFGR64Opnd>,
                  MADDS_FM<6, 1>, INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_32;
  def NMSUB_D32 : MMRel, NMADDS_FT<"nmsub.d", AFGR64Opnd>,
                  MADDS_FM<7, 1>, INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_32;
```
- EN: Defines TableGen record `NMADD_D32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NMADD_D32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 760-777
```tablegen
  let DecoderNamespace = "MipsFP64" in {
    def NMADD_D64 : NMADDS_FT<"nmadd.d", FGR64Opnd>,
                    MADDS_FM<6, 1>, INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_64;
    def NMSUB_D64 : NMADDS_FT<"nmsub.d", FGR64Opnd>,
                    MADDS_FM<7, 1>, INSN_MIPS4_32R2_NOT_32R6_64R6, FGR_64;
  }
}
let Predicates = NMADD_S.Predicates in
def : Pat<(NMADDS_FT_PatFrag<any_fadd> FGR32Opnd:$fr, FGR32Opnd:$fs, FGR32Opnd:$ft),
          (NMADD_S FGR32Opnd:$fr, FGR32Opnd:$fs, FGR32Opnd:$ft)>;
let Predicates = NMSUB_S.Predicates in
def : Pat<(NMADDS_FT_PatFrag<any_fsub> FGR32Opnd:$fr, FGR32Opnd:$fs, FGR32Opnd:$ft),
          (NMSUB_S FGR32Opnd:$fr, FGR32Opnd:$fs, FGR32Opnd:$ft)>;
let Predicates = NMADD_D32.Predicates in
def : Pat<(NMADDS_FT_PatFrag<any_fadd> AFGR64Opnd:$fr, AFGR64Opnd:$fs, AFGR64Opnd:$ft),
          (NMADD_D32 AFGR64Opnd:$fr, AFGR64Opnd:$fs, AFGR64Opnd:$ft)>;
let Predicates = NMSUB_D32.Predicates in
def : Pat<(NMADDS_FT_PatFrag<any_fsub> AFGR64Opnd:$fr, AFGR64Opnd:$fs, AFGR64Opnd:$ft),
```
- EN: Defines TableGen record `NMADD_D64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NMADD_D64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 778-791
```tablegen
          (NMSUB_D32 AFGR64Opnd:$fr, AFGR64Opnd:$fs, AFGR64Opnd:$ft)>;
let Predicates = NMADD_D64.Predicates in
def : Pat<(NMADDS_FT_PatFrag<any_fadd> FGR64Opnd:$fr, FGR64Opnd:$fs, FGR64Opnd:$ft),
          (NMADD_D64 FGR64Opnd:$fr, FGR64Opnd:$fs, FGR64Opnd:$ft)>;
let Predicates = NMSUB_D64.Predicates in
def : Pat<(NMADDS_FT_PatFrag<any_fsub> FGR64Opnd:$fr, FGR64Opnd:$fs, FGR64Opnd:$ft),
          (NMSUB_D64 FGR64Opnd:$fr, FGR64Opnd:$fs, FGR64Opnd:$ft)>;
//===----------------------------------------------------------------------===//
// Floating Point Branch Codes
//===----------------------------------------------------------------------===//
// Mips branch codes. These correspond to condcode in MipsInstrInfo.h.
// They must be kept in synch.
def MIPS_BRANCH_F  : PatLeaf<(i32 0)>;
def MIPS_BRANCH_T  : PatLeaf<(i32 1)>;
```
- EN: Defines TableGen record `MIPS_BRANCH_F` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MIPS_BRANCH_F`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 793-801
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def BC1F : MMRel, BC1F_FT<"bc1f", brtarget, MIPS_BRANCH_F>,
             BC1F_FM<0, 0>, ISA_MIPS1_NOT_32R6_64R6;
  def BC1FL : MMRel, BC1XL_FT<"bc1fl", brtarget>,
              BC1F_FM<1, 0>, ISA_MIPS2_NOT_32R6_64R6;
  def BC1T : MMRel, BC1F_FT<"bc1t", brtarget, MIPS_BRANCH_T>,
             BC1F_FM<0, 1>, ISA_MIPS1_NOT_32R6_64R6;
  def BC1TL : MMRel, BC1XL_FT<"bc1tl", brtarget>,
              BC1F_FM<1, 1>, ISA_MIPS2_NOT_32R6_64R6;
```
- EN: Defines TableGen record `BC1F` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BC1F`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 803-805
```tablegen
/// Floating Point Compare
  def FCMP_S32 : MMRel, CEQS_FT<"s", FGR32, MipsFPCmp>, CEQS_FM<16>,
                 ISA_MIPS1_NOT_32R6_64R6 {
```
- EN: Defines TableGen record `FCMP_S32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FCMP_S32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 807-824
```tablegen
  // FIXME: This is a required to work around the fact that these instructions
  //        only use $fcc0. Ideally, MipsFPCmp nodes could be removed and the
  //        fcc register set is used directly.
  bits<3> fcc = 0;
  }
  def FCMP_D32 : MMRel, CEQS_FT<"d", AFGR64, MipsFPCmp>, CEQS_FM<17>,
                 ISA_MIPS1_NOT_32R6_64R6, FGR_32 {
  // FIXME: This is a required to work around the fact that these instructions
  //        only use $fcc0. Ideally, MipsFPCmp nodes could be removed and the
  //        fcc register set is used directly.
  bits<3> fcc = 0;
  }
}
let DecoderNamespace = "MipsFP64" in
def FCMP_D64 : CEQS_FT<"d", FGR64, MipsFPCmp>, CEQS_FM<17>,
               ISA_MIPS1_NOT_32R6_64R6, FGR_64 {
  // FIXME: This is a required to work around the fact that thiese instructions
  //        only use $fcc0. Ideally, MipsFPCmp nodes could be removed and the
```
- EN: Defines TableGen record `FCMP_D32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FCMP_D32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 825-827
```tablegen
  //        fcc register set is used directly.
  bits<3> fcc = 0;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 829-831
```tablegen
//===----------------------------------------------------------------------===//
// Floating Point Pseudo-Instructions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 833-837
```tablegen
// This pseudo instr gets expanded into 2 mtc1 instrs after register
// allocation.
class BuildPairF64Base<RegisterOperand RO> :
  PseudoSE<(outs RO:$dst), (ins GPR32Opnd:$lo, GPR32Opnd:$hi),
           [(set RO:$dst, (MipsBuildPairF64 GPR32Opnd:$lo, GPR32Opnd:$hi))]>;
```
- EN: Declares reusable TableGen class `BuildPairF64Base` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `BuildPairF64Base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 839-840
```tablegen
def BuildPairF64 : BuildPairF64Base<AFGR64Opnd>, FGR_32, HARDFLOAT;
def BuildPairF64_64 : BuildPairF64Base<FGR64Opnd>, FGR_64, HARDFLOAT;
```
- EN: Defines TableGen record `BuildPairF64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BuildPairF64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 842-850
```tablegen
// This pseudo instr gets expanded into 2 mfc1 instrs after register
// allocation.
// if n is 0, lower part of src is extracted.
// if n is 1, higher part of src is extracted.
// This node has associated scheduling information as the pre RA scheduler
// asserts otherwise.
class ExtractElementF64Base<RegisterOperand RO> :
  PseudoSE<(outs GPR32Opnd:$dst), (ins RO:$src, i32imm:$n),
           [(set GPR32Opnd:$dst, (MipsExtractElementF64 RO:$src, imm:$n))]>;
```
- EN: Declares reusable TableGen class `ExtractElementF64Base` for `MipsInstrFPU`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `MipsInstrFPU` 声明可复用的 TableGen 类 `ExtractElementF64Base`，通常用于抽象共享字段、谓词或编码结构。

### Lines 852-853
```tablegen
def ExtractElementF64 : ExtractElementF64Base<AFGR64Opnd>, FGR_32, HARDFLOAT;
def ExtractElementF64_64 : ExtractElementF64Base<FGR64Opnd>, FGR_64, HARDFLOAT;
```
- EN: Defines TableGen record `ExtractElementF64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ExtractElementF64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 855-857
```tablegen
def PseudoTRUNC_W_S : MipsAsmPseudoInst<(outs FGR32Opnd:$fd),
                                        (ins FGR32Opnd:$fs, GPR32Opnd:$rs),
                                        "trunc.w.s\t$fd, $fs, $rs">;
```
- EN: Defines TableGen record `PseudoTRUNC_W_S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoTRUNC_W_S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 859-862
```tablegen
def PseudoTRUNC_W_D32 : MipsAsmPseudoInst<(outs FGR32Opnd:$fd),
                                          (ins AFGR64Opnd:$fs, GPR32Opnd:$rs),
                                          "trunc.w.d\t$fd, $fs, $rs">,
                        FGR_32, HARDFLOAT;
```
- EN: Defines TableGen record `PseudoTRUNC_W_D32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoTRUNC_W_D32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 864-867
```tablegen
def PseudoTRUNC_W_D : MipsAsmPseudoInst<(outs FGR32Opnd:$fd),
                                        (ins FGR64Opnd:$fs, GPR32Opnd:$rs),
                                        "trunc.w.d\t$fd, $fs, $rs">,
                      FGR_64, HARDFLOAT;
```
- EN: Defines TableGen record `PseudoTRUNC_W_D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PseudoTRUNC_W_D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 869-871
```tablegen
def LoadImmSingleGPR : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                         (ins imm64:$fpimm),
                                         "li.s\t$rd, $fpimm">;
```
- EN: Defines TableGen record `LoadImmSingleGPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LoadImmSingleGPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 873-876
```tablegen
def LoadImmSingleFGR : MipsAsmPseudoInst<(outs StrictlyFGR32Opnd:$rd),
                                         (ins imm64:$fpimm),
                                         "li.s\t$rd, $fpimm">,
                       HARDFLOAT;
```
- EN: Defines TableGen record `LoadImmSingleFGR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LoadImmSingleFGR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 878-880
```tablegen
def LoadImmDoubleGPR : MipsAsmPseudoInst<(outs GPR32Opnd:$rd),
                                         (ins imm64:$fpimm),
                                         "li.d\t$rd, $fpimm">;
```
- EN: Defines TableGen record `LoadImmDoubleGPR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LoadImmDoubleGPR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 882-885
```tablegen
def LoadImmDoubleFGR_32 : MipsAsmPseudoInst<(outs StrictlyAFGR64Opnd:$rd),
                                            (ins imm64:$fpimm),
                                            "li.d\t$rd, $fpimm">,
                          FGR_32, HARDFLOAT;
```
- EN: Defines TableGen record `LoadImmDoubleFGR_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LoadImmDoubleFGR_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 887-890
```tablegen
def LoadImmDoubleFGR : MipsAsmPseudoInst<(outs StrictlyFGR64Opnd:$rd),
                                         (ins imm64:$fpimm),
                                         "li.d\t$rd, $fpimm">,
                       FGR_64, HARDFLOAT;
```
- EN: Defines TableGen record `LoadImmDoubleFGR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LoadImmDoubleFGR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 892-895
```tablegen
def SDC1_M1 : MipsAsmPseudoInst<(outs AFGR64Opnd:$fd),
                                (ins mem_simm16:$addr),
                                "s.d\t$fd, $addr">,
              FGR_32, ISA_MIPS1, HARDFLOAT;
```
- EN: Defines TableGen record `SDC1_M1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SDC1_M1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 897-911
```tablegen
//===----------------------------------------------------------------------===//
// InstAliases.
//===----------------------------------------------------------------------===//
def : MipsInstAlias
        <"s.s $fd, $addr", (SWC1 FGR32Opnd:$fd, mem_simm16:$addr), 0>,
      ISA_MIPS2, HARDFLOAT;
def : MipsInstAlias
        <"s.d $fd, $addr", (SDC1 AFGR64Opnd:$fd, mem_simm16:$addr), 0>,
      FGR_32, ISA_MIPS2, HARDFLOAT;
def : MipsInstAlias
        <"s.d $fd, $addr", (SDC164 FGR64Opnd:$fd, mem_simm16:$addr), 0>,
      FGR_64, ISA_MIPS2, HARDFLOAT;
def : MipsInstAlias
        <"s.d $fd, $addr", (SDC1_M1 AFGR64Opnd:$fd, mem_simm16:$addr), 0>,
      FGR_32, ISA_MIPS1, HARDFLOAT;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 913-921
```tablegen
def : MipsInstAlias
        <"l.s $fd, $addr", (LWC1 FGR32Opnd:$fd, mem_simm16:$addr), 0>,
      ISA_MIPS2, HARDFLOAT;
def : MipsInstAlias
        <"l.d $fd, $addr", (LDC1 AFGR64Opnd:$fd, mem_simm16:$addr), 0>,
      FGR_32, ISA_MIPS2, HARDFLOAT;
def : MipsInstAlias
        <"l.d $fd, $addr", (LDC164 FGR64Opnd:$fd, mem_simm16:$addr), 0>,
      FGR_64, ISA_MIPS2, HARDFLOAT;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 923-936
```tablegen
multiclass C_COND_ALIASES<string TypeStr, RegisterOperand RC> {
  def : MipsInstAlias<!strconcat("c.f.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_F_"#NAME) FCC0,
                                                       RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.eq.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_EQ_"#NAME) FCC0,
                                                        RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.olt.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_OLT_"#NAME) FCC0,
                                                         RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.ole.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_OLE_"#NAME) FCC0,
                                                         RC:$fs, RC:$ft), 1>;
}
```
- EN: Declares TableGen `multiclass C_COND_ALIASES`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass C_COND_ALIASES`，它是一个可复用模板，可展开为多个相关记录。

### Lines 938-955
```tablegen
multiclass C_COND_NOTR5900_ALIASES<string TypeStr, RegisterOperand RC> {
  def : MipsInstAlias<!strconcat("c.un.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_UN_"#NAME) FCC0,
                                                        RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.ueq.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_UEQ_"#NAME) FCC0,
                                                         RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.ult.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_ULT_"#NAME) FCC0,
                                                         RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.ule.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_ULE_"#NAME) FCC0,
                                                         RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.sf.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_SF_"#NAME) FCC0,
                                                        RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.ngle.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_NGLE_"#NAME) FCC0,
```
- EN: Declares TableGen `multiclass C_COND_NOTR5900_ALIASES`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass C_COND_NOTR5900_ALIASES`，它是一个可复用模板，可展开为多个相关记录。

### Lines 956-973
```tablegen
                                                          RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.seq.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_SEQ_"#NAME) FCC0,
                                                         RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.ngl.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_NGL_"#NAME) FCC0,
                                                         RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.lt.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_LT_"#NAME) FCC0,
                                                        RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.nge.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_NGE_"#NAME) FCC0,
                                                         RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.le.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_LE_"#NAME) FCC0,
                                                        RC:$fs, RC:$ft), 1>;
  def : MipsInstAlias<!strconcat("c.ngt.", TypeStr, " $fs, $ft"),
                      (!cast<Instruction>("C_NGT_"#NAME) FCC0,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 974-975
```tablegen
                                                         RC:$fs, RC:$ft), 1>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 977-980
```tablegen
multiclass BC1_ALIASES<Instruction BCTrue, string BCTrueString,
                       Instruction BCFalse, string BCFalseString> {
  def : MipsInstAlias<!strconcat(BCTrueString, " $offset"),
                                (BCTrue FCC0, brtarget:$offset), 1>;
```
- EN: Declares TableGen `multiclass BC1_ALIASES`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass BC1_ALIASES`，它是一个可复用模板，可展开为多个相关记录。

### Lines 982-984
```tablegen
  def : MipsInstAlias<!strconcat(BCFalseString, " $offset"),
                                (BCFalse FCC0, brtarget:$offset), 1>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 986-998
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  defm S   : C_COND_ALIASES<"s", FGR32Opnd>, HARDFLOAT,
             ISA_MIPS1_NOT_32R6_64R6;
  defm S   : C_COND_NOTR5900_ALIASES<"s", FGR32Opnd>, HARDFLOAT,
             ISA_MIPS1_NOT_32R6_64R6_R5900;
  defm D32 : C_COND_ALIASES<"d", AFGR64Opnd>, HARDFLOAT,
             ISA_MIPS1_NOT_32R6_64R6, FGR_32;
  defm D32 : C_COND_NOTR5900_ALIASES<"d", AFGR64Opnd>, HARDFLOAT,
             ISA_MIPS1_NOT_32R6_64R6, FGR_32;
  defm D64 : C_COND_ALIASES<"d", FGR64Opnd>, HARDFLOAT,
             ISA_MIPS1_NOT_32R6_64R6, FGR_64;
  defm D64 : C_COND_NOTR5900_ALIASES<"d", FGR64Opnd>, HARDFLOAT,
             ISA_MIPS1_NOT_32R6_64R6, FGR_64;
```
- EN: Defines TableGen record `S` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `S`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1000-1009
```tablegen
  defm : BC1_ALIASES<BC1T, "bc1t", BC1F, "bc1f">, ISA_MIPS1_NOT_32R6_64R6,
         HARDFLOAT;
  defm : BC1_ALIASES<BC1TL, "bc1tl", BC1FL, "bc1fl">, ISA_MIPS2_NOT_32R6_64R6,
         HARDFLOAT;
}
//===----------------------------------------------------------------------===//
// Floating Point Patterns
//===----------------------------------------------------------------------===//
def : MipsPat<(f32 fpimm0), (MTC1 ZERO)>, ISA_MIPS1;
def : MipsPat<(f32 fpimm0neg), (FNEG_S (MTC1 ZERO))>, ISA_MIPS1;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1011-1018
```tablegen
def : MipsPat<(f32 (any_sint_to_fp GPR32Opnd:$src)),
              (PseudoCVT_S_W GPR32Opnd:$src)>;
let Predicates = [IsR5900] in {
  def : MipsPat<(MipsTruncIntFP FGR32Opnd:$src),
                (CVT_W_S FGR32Opnd:$src)>, ISA_MIPS1;
}
def : MipsPat<(MipsTruncIntFP FGR32Opnd:$src),
              (TRUNC_W_S FGR32Opnd:$src)>, ISA_MIPS1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1020-1021
```tablegen
def : MipsPat<(MipsMTC1_D64 GPR32Opnd:$src),
              (MTC1_D64 GPR32Opnd:$src)>, ISA_MIPS1, FGR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1023-1025
```tablegen
def : MipsPat<(setlt (i32 (bitconvert (f32 (fpround AFGR64Opnd:$src)))), 0),
           (SLTi (MFHC1_D32 AFGR64Opnd:$src), 0)>,
           ISA_MIPS32R2, FGR_32;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1027-1029
```tablegen
def : MipsPat<(setlt (i32 (bitconvert (f32 (fpround FGR64Opnd:$src)))), 0),
          (SLTi (MFHC1_D64 FGR64Opnd:$src), 0)>,
          ISA_MIPS32R2, FGR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1031-1040
```tablegen
def : MipsPat<(f64 (any_sint_to_fp GPR32Opnd:$src)),
              (PseudoCVT_D32_W GPR32Opnd:$src)>, FGR_32;
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsPat<(MipsTruncIntFP AFGR64Opnd:$src),
                (TRUNC_W_D32 AFGR64Opnd:$src)>, ISA_MIPS2, FGR_32;
  def : MipsPat<(f32 (any_fpround AFGR64Opnd:$src)),
                (CVT_S_D32 AFGR64Opnd:$src)>, ISA_MIPS1, FGR_32;
  def : MipsPat<(f64 (any_fpextend FGR32Opnd:$src)),
                (CVT_D32_S FGR32Opnd:$src)>, ISA_MIPS1, FGR_32;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1042-1044
```tablegen
def : MipsPat<(f64 fpimm0), (DMTC1 ZERO_64)>, ISA_MIPS3, GPR_64, FGR_64;
def : MipsPat<(f64 fpimm0neg), (FNEG_D64 (DMTC1 ZERO_64))>, ISA_MIPS3, GPR_64,
      FGR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1046-1051
```tablegen
def : MipsPat<(f64 (any_sint_to_fp GPR32Opnd:$src)), 
              (PseudoCVT_D64_W GPR32Opnd:$src)>, FGR_64;
def : MipsPat<(f32 (any_sint_to_fp GPR64Opnd:$src)),
              (EXTRACT_SUBREG (PseudoCVT_S_L GPR64Opnd:$src), sub_lo)>, FGR_64;
def : MipsPat<(f64 (any_sint_to_fp GPR64Opnd:$src)),
              (PseudoCVT_D64_L GPR64Opnd:$src)>, FGR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1053-1058
```tablegen
def : MipsPat<(MipsTruncIntFP FGR64Opnd:$src),
              (TRUNC_W_D64 FGR64Opnd:$src)>, ISA_MIPS2, FGR_64;
def : MipsPat<(MipsTruncIntFP FGR32Opnd:$src),
              (TRUNC_L_S FGR32Opnd:$src)>, ISA_MIPS2, FGR_64;
def : MipsPat<(MipsTruncIntFP FGR64Opnd:$src),
              (TRUNC_L_D64 FGR64Opnd:$src)>, ISA_MIPS2, FGR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1060-1065
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsPat<(f32 (any_fpround FGR64Opnd:$src)),
                (CVT_S_D64 FGR64Opnd:$src)>, ISA_MIPS1, FGR_64;
  def : MipsPat<(f64 (any_fpextend FGR32Opnd:$src)),
                (CVT_D64_S FGR32Opnd:$src)>, ISA_MIPS1, FGR_64;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1067-1076
```tablegen
// To generate NMADD and NMSUB instructions when fneg node is present
class NMADD_NMSUB_PatFrag<SDPatternOperator OpNode>: PatFrag<(ops node:$fr, node:$fs, node:$ft),
          (fneg (OpNode (any_fmul node:$fs, node:$ft), node:$fr)),
          [{ return N->getFlags().hasNoNaNs(); }]>;
multiclass NMADD_NMSUB<Instruction Nmadd, Instruction Nmsub, RegisterOperand RC> {
  def : MipsPat<(NMADD_NMSUB_PatFrag<any_fadd> RC:$fr, RC:$fs, RC:$ft),
                (Nmadd RC:$fr, RC:$fs, RC:$ft)>;
  def : MipsPat<(NMADD_NMSUB_PatFrag<any_fsub> RC:$fr, RC:$fs, RC:$ft),
                (Nmsub RC:$fr, RC:$fs, RC:$ft)>;
}
```
- EN: Declares TableGen `multiclass NMADD_NMSUB`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass NMADD_NMSUB`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1078-1085
```tablegen
let AdditionalPredicates = [HasMadd4, NotInMicroMips] in {
  defm : NMADD_NMSUB<NMADD_S, NMSUB_S, FGR32Opnd>,
         INSN_MIPS4_32R2_NOT_32R6_64R6;
  defm : NMADD_NMSUB<NMADD_D32, NMSUB_D32, AFGR64Opnd>,
         FGR_32, INSN_MIPS4_32R2_NOT_32R6_64R6;
  defm : NMADD_NMSUB<NMADD_D64, NMSUB_D64, FGR64Opnd>,
         FGR_64, INSN_MIPS4_32R2_NOT_32R6_64R6;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1087-1091
```tablegen
// Patterns for loads/stores with a reg+imm operand.
let AdditionalPredicates = [NotInMicroMips] in {
  let AddedComplexity = 40 in {
    def : LoadRegImmPat<LWC1, f32, load>, ISA_MIPS1;
    def : StoreRegImmPat<SWC1, f32>, ISA_MIPS1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1093-1094
```tablegen
    def : LoadRegImmPat<LDC164, f64, load>, ISA_MIPS1, FGR_64;
    def : StoreRegImmPat<SDC164, f64>, ISA_MIPS1, FGR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1096-1099
```tablegen
    def : LoadRegImmPat<LDC1, f64, load>, ISA_MIPS1, FGR_32;
    def : StoreRegImmPat<SDC1, f64>, ISA_MIPS1, FGR_32;
  }
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

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
