# AMDGPUInstructions.td — Code Analysis / 代码分析

## Source / 来源
- **File**: `llvm/lib/Target/AMDGPU/AMDGPUInstructions.td`
- **Repository**: llvm/llvm-project
- **Purpose**: This TableGen DSL file defines AMDGPUInstructions records for the LLVM AMDGPU backend. It describes target features, instructions, predicates, or generated metadata consumed by LLVM table generators. / 该 TableGen DSL 文件为 LLVM AMDGPU 后端定义 AMDGPUInstructions 相关记录。它描述目标特性、指令、谓词或供 LLVM 表生成器使用的元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-42: File banner, comments, and TableGen overview
```tablegen
//===-- AMDGPUInstructions.td - Common instruction defs ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file contains instruction defs that are common to all hw codegen
// targets.
//
//===----------------------------------------------------------------------===//

def AddrSpaces {
  int Flat = 0;
  int Global = 1;
  int Region = 2;
  int Local = 3;
  int Constant = 4;
  int Private = 5;
  int Constant32Bit = 6;
}


class AMDGPUInst <dag outs, dag ins, string asm = "",
  list<dag> pattern = []> : Instruction {
  field bit isRegisterLoad = 0;
  field bit isRegisterStore = 0;

  let Namespace = "AMDGPU";
  let OutOperandList = outs;
  let InOperandList = ins;
  let AsmString = asm;
  let Pattern = pattern;
  let Itinerary = NullALU;

  let DecoderNamespace = Namespace;

  let TSFlags{63} = isRegisterLoad;
  let TSFlags{62} = isRegisterStore;
}

```
**EN:** This opening section establishes the file identity, licensing terms, and high-level intent before the backend-specific code begins. Main symbols: `AddrSpaces`, `AMDGPUInst`.
**CN:** 开头部分先给出文件标识、许可证信息以及总体意图，为后续后端专用代码建立上下文。 主要符号：`AddrSpaces`, `AMDGPUInst`。

### Lines 43-81: Defines TableGen class AMDGPUShaderInst
```tablegen
class AMDGPUShaderInst <dag outs, dag ins, string asm = "",
  list<dag> pattern = []> : AMDGPUInst<outs, ins, asm, pattern> {

  field bits<32> Inst = 0xffffffff;
}

//===---------------------------------------------------------------------===//
// Return instruction
//===---------------------------------------------------------------------===//

class ILFormat<dag outs, dag ins, string asmstr, list<dag> pattern>
: Instruction {

     let Namespace = "AMDGPU";
     dag OutOperandList = outs;
     dag InOperandList = ins;
     let Pattern = pattern;
     let AsmString = !strconcat(asmstr, "\n");
     let isPseudo = 1;
     let Itinerary = NullALU;
     bit hasIEEEFlag = 0;
     bit hasZeroOpFlag = 0;
     let mayLoad = 0;
     let mayStore = 0;
     let hasSideEffects = 0;
     let isCodeGenOnly = 1;
}

// Get the union of two Register lists
class RegListUnion<list<Register> lstA, list<Register> lstB> {
  list<Register> ret = !listconcat(lstA, !listremove(lstB, lstA));
}

class AMDGPUPat<dag pattern, dag result> : Pat<pattern, result>,
      PredicateControl, GISelFlags;

let GIIgnoreCopies = 1 in
class AMDGPUPatIgnoreCopies<dag pattern, dag result> : AMDGPUPat<pattern, result>;

```
**EN:** This section contains concrete logic for TableGen class AMDGPUShaderInst. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `AMDGPUShaderInst`, `ILFormat`, `RegListUnion`.
**CN:** 本节包含与 TableGen class AMDGPUShaderInst 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`AMDGPUShaderInst`, `ILFormat`, `RegListUnion`。

### Lines 82-124: Defines TableGen record FP16Denormals
```tablegen
let RecomputePerFunction = 1 in {
def FP16Denormals : Predicate<"MF->getInfo<SIMachineFunctionInfo>()->getMode().FP64FP16Denormals != DenormalMode::getPreserveSign()">;
def FP32Denormals : Predicate<"MF->getInfo<SIMachineFunctionInfo>()->getMode().FP32Denormals != DenormalMode::getPreserveSign()">;
def FP64Denormals : Predicate<"MF->getInfo<SIMachineFunctionInfo>()->getMode().FP64FP16Denormals != DenormalMode::getPreserveSign()">;
def NoFP16Denormals : Predicate<"MF->getInfo<SIMachineFunctionInfo>()->getMode().FP64FP16Denormals == DenormalMode::getPreserveSign()">;
def NoFP32Denormals : Predicate<"MF->getInfo<SIMachineFunctionInfo>()->getMode().FP32Denormals == DenormalMode::getPreserveSign()">;
def NoFP64Denormals : Predicate<"MF->getInfo<SIMachineFunctionInfo>()->getMode().FP64FP16Denormals == DenormalMode::getPreserveSign()">;
def IEEEModeEnabled : Predicate<"MF->getInfo<SIMachineFunctionInfo>()->getMode().IEEE">;
def IEEEModeDisabled : Predicate<"!MF->getInfo<SIMachineFunctionInfo>()->getMode().IEEE">;
}

def FMA : Predicate<"Subtarget->hasFMA()">;

def InstFlag : OperandWithDefaultOps <i32, (ops (i32 0))>;

def i1imm_0 : OperandWithDefaultOps<i1, (ops (i1 0))>;

class CustomOperandClass<string name, bit optional, string predicateMethod,
                         string parserMethod, string defaultMethod>
    : AsmOperandClass {
  let Name = name;
  let PredicateMethod = predicateMethod;
  let ParserMethod = parserMethod;
  let RenderMethod = "addImmOperands";
  let IsOptional = optional;
  let DefaultMethod = defaultMethod;
}

class CustomOperandProps<bit optional = 0, string name = NAME> {
  string ImmTy = "ImmTy"#name;
  string PredicateMethod = "is"#name;
  string ParserMethod = "parse"#name;
  string DefaultValue = "0";
  string DefaultMethod = "[this]() { return "#
    "AMDGPUOperand::CreateImm(this, "#DefaultValue#", SMLoc(), "#
    "AMDGPUOperand::"#ImmTy#"); }";
  string PrintMethod = "print"#name;
  AsmOperandClass ParserMatchClass =
    CustomOperandClass<name, optional, PredicateMethod, ParserMethod,
                       DefaultMethod>;
  string OperandType = "OPERAND_IMMEDIATE";
}

```
**EN:** This section contains concrete logic for TableGen record FP16Denormals. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `FP16Denormals`, `FP32Denormals`, `FP64Denormals`.
**CN:** 本节包含与 TableGen record FP16Denormals 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`FP16Denormals`, `FP32Denormals`, `FP64Denormals`。

### Lines 125-163: Defines TableGen class CustomOperand
```tablegen
class CustomOperand<bit optional = 0, string name = NAME, ValueType type = i32>
  : Operand<type>, CustomOperandProps<optional, name>;

class ImmOperand<ValueType type, string name = NAME, bit optional = 0,
                 string printer = "print"#name>
    : CustomOperand<optional, name, type=type> {
  let ImmTy = "ImmTyNone";
  let ParserMethod = "";
  let PrintMethod = printer;
}

class S16ImmOperand : ImmOperand<i16, "S16Imm", 0, "printU16ImmOperand">;

def s16imm : S16ImmOperand;
def u16imm : ImmOperand<i16, "U16Imm", 0, "printU16ImmOperand">;

class ValuePredicatedOperand<CustomOperand op, string valuePredicate,
                             bit optional = 0>
    : CustomOperand<optional, type=op.Type> {
  let ImmTy = op.ImmTy;
  defvar OpPredicate = op.ParserMatchClass.PredicateMethod;
  let PredicateMethod =
    "getPredicate([](const AMDGPUOperand &Op) -> bool { "#
    "return Op."#OpPredicate#"() && "#valuePredicate#"; })";
  let ParserMethod = op.ParserMatchClass.ParserMethod;
  let DefaultValue = op.DefaultValue;
  let DefaultMethod = op.DefaultMethod;
  let PrintMethod = op.PrintMethod;
}

//===--------------------------------------------------------------------===//
// Custom Operands
//===--------------------------------------------------------------------===//
def brtarget   : Operand<OtherVT>;

//===----------------------------------------------------------------------===//
// Misc. PatFrags
//===----------------------------------------------------------------------===//

```
**EN:** This section contains concrete logic for TableGen class CustomOperand. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `CustomOperand`, `ImmOperand`, `S16ImmOperand`.
**CN:** 本节包含与 TableGen class CustomOperand 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`CustomOperand`, `ImmOperand`, `S16ImmOperand`。

### Lines 164-206: Defines TableGen class HasOneUseUnaryOp
```tablegen
class HasOneUseUnaryOp<SDPatternOperator op> : PatFrag<
  (ops node:$src0),
  (op $src0)> {
  let HasOneUse = 1;
}

class HasOneUseBinOp<SDPatternOperator op> : PatFrag<
  (ops node:$src0, node:$src1),
  (op $src0, $src1)> {
  let HasOneUse = 1;
}

class HasOneUseTernaryOp<SDPatternOperator op> : PatFrag<
  (ops node:$src0, node:$src1, node:$src2),
  (op $src0, $src1, $src2)> {
  let HasOneUse = 1;
}

class is_canonicalized_1<SDPatternOperator op> : PatFrag<
  (ops node:$src0),
  (op $src0),
  [{
    const SITargetLowering &Lowering =
              *static_cast<const SITargetLowering *>(getTargetLowering());

    return Lowering.isCanonicalized(*CurDAG, N->getOperand(0));
   }]> {

  let GISelPredicateCode = [{
    const SITargetLowering *TLI = static_cast<const SITargetLowering *>(
      MF.getSubtarget().getTargetLowering());

    return TLI->isCanonicalized(MI.getOperand(1).getReg(), MF);
  }];
}

class is_canonicalized_2<SDPatternOperator op> : PatFrag<
  (ops node:$src0, node:$src1),
  (op $src0, $src1),
  [{
    const SITargetLowering &Lowering =
              *static_cast<const SITargetLowering *>(getTargetLowering());

```
**EN:** This section contains concrete logic for TableGen class HasOneUseUnaryOp. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `HasOneUseUnaryOp`, `HasOneUseBinOp`, `HasOneUseTernaryOp`.
**CN:** 本节包含与 TableGen class HasOneUseUnaryOp 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`HasOneUseUnaryOp`, `HasOneUseBinOp`, `HasOneUseTernaryOp`。

### Lines 207-249: Defines TableGen class FoldTernaryOpPat
```tablegen
    return Lowering.isCanonicalized(*CurDAG, N->getOperand(0)) &&
      Lowering.isCanonicalized(*CurDAG, N->getOperand(1));
   }]> {

  // TODO: Improve the Legalizer for g_build_vector in Global Isel to match this class
  let GISelPredicateCode = [{
    const SITargetLowering *TLI = static_cast<const SITargetLowering *>(
      MF.getSubtarget().getTargetLowering());

    return TLI->isCanonicalized(MI.getOperand(1).getReg(), MF) &&
      TLI->isCanonicalized(MI.getOperand(2).getReg(), MF);
  }];
}

class FoldTernaryOpPat<SDPatternOperator op1, SDPatternOperator op2> : PatFrag<
  (ops node:$src0, node:$src1, node:$src2),
  (op2 (op1 node:$src0, node:$src1), node:$src2)
>;

def imad : FoldTernaryOpPat<mul, add>;

// Match binary op with source operands that won't fold into sdwa patterns.
class BinOp_no_sdwa<SDPatternOperator binop> : PatFrag<
  (ops node:$lhs, node:$rhs),
  (binop node:$lhs, node:$rhs),
  [{
    return !isSDWAOperand(Op.getOperand(0).getNode()) &&
           !isSDWAOperand(Op.getOperand(1).getNode());
  }]> {
  let GISelPredicateCode = [{
    return true; // TODO
  }];
}

def AMDGPUmul_i24_no_sdwa : BinOp_no_sdwa<AMDGPUmul_i24>;
def AMDGPUmul_u24_no_sdwa : BinOp_no_sdwa<AMDGPUmul_u24>;

let Properties = [SDNPCommutative, SDNPAssociative] in {
def smax_oneuse : HasOneUseBinOp<smax>;
def smin_oneuse : HasOneUseBinOp<smin>;
def umax_oneuse : HasOneUseBinOp<umax>;
def umin_oneuse : HasOneUseBinOp<umin>;

```
**EN:** This section contains concrete logic for TableGen class FoldTernaryOpPat. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `FoldTernaryOpPat`, `imad`, `BinOp_no_sdwa`.
**CN:** 本节包含与 TableGen class FoldTernaryOpPat 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`FoldTernaryOpPat`, `imad`, `BinOp_no_sdwa`。

### Lines 250-293: Defines TableGen record fminnum_oneuse
```tablegen
def fminnum_oneuse : HasOneUseBinOp<fminnum>;
def fmaxnum_oneuse : HasOneUseBinOp<fmaxnum>;
def fminimum_oneuse : HasOneUseBinOp<fminimum>;
def fmaximum_oneuse : HasOneUseBinOp<fmaximum>;

def fminnum_ieee_oneuse : HasOneUseBinOp<fminnum_ieee>;
def fmaxnum_ieee_oneuse : HasOneUseBinOp<fmaxnum_ieee>;


def and_oneuse : HasOneUseBinOp<and>;
def or_oneuse : HasOneUseBinOp<or>;
def xor_oneuse : HasOneUseBinOp<xor>;
} // Properties = [SDNPCommutative, SDNPAssociative]

def not_oneuse  : HasOneUseUnaryOp<not>;
def vnot_oneuse : HasOneUseUnaryOp<vnot>;

def add_oneuse : HasOneUseBinOp<add>;
def sub_oneuse : HasOneUseBinOp<sub>;

def srl_oneuse : HasOneUseBinOp<srl>;
def shl_oneuse : HasOneUseBinOp<shl>;

def select_oneuse : HasOneUseTernaryOp<select>;

def AMDGPUmul_u24_oneuse : HasOneUseBinOp<AMDGPUmul_u24>;
def AMDGPUmul_i24_oneuse : HasOneUseBinOp<AMDGPUmul_i24>;

//===----------------------------------------------------------------------===//
// PatFrags for shifts
//===----------------------------------------------------------------------===//

// Constrained shift PatFrags.

def csh_mask_16 : PatFrag<(ops node:$src0), (and node:$src0, imm),
  [{ return isUnneededShiftMask(N, 4); }]> {
    let GISelPredicateCode = [{ return isUnneededShiftMask(MI, 4); }];
  }

def csh_mask_32 : PatFrag<(ops node:$src0), (and node:$src0, imm),
  [{ return isUnneededShiftMask(N, 5); }]> {
    let GISelPredicateCode = [{ return isUnneededShiftMask(MI, 5); }];
  }

```
**EN:** This section contains concrete logic for TableGen record fminnum_oneuse. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `fminnum_oneuse`, `fmaxnum_oneuse`, `fminimum_oneuse`.
**CN:** 本节包含与 TableGen record fminnum_oneuse 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`fminnum_oneuse`, `fmaxnum_oneuse`, `fminimum_oneuse`。

### Lines 294-333: Defines TableGen record csh_mask_64
```tablegen
def csh_mask_64 : PatFrag<(ops node:$src0), (and node:$src0, imm),
  [{ return isUnneededShiftMask(N, 6); }]> {
    let GISelPredicateCode = [{ return isUnneededShiftMask(MI, 6); }];
  }

foreach width = [16, 32, 64] in {
defvar csh_mask = !cast<SDPatternOperator>("csh_mask_"#width);

def cshl_#width : PatFrags<(ops node:$src0, node:$src1),
  [(shl node:$src0, node:$src1), (shl node:$src0, (csh_mask node:$src1))]>;
defvar cshl = !cast<SDPatternOperator>("cshl_"#width);
def cshl_#width#_oneuse : HasOneUseBinOp<cshl>;
def clshl_rev_#width : PatFrag <(ops node:$src0, node:$src1),
  (cshl $src1, $src0)>;

def csrl_#width : PatFrags<(ops node:$src0, node:$src1),
  [(srl node:$src0, node:$src1), (srl node:$src0, (csh_mask node:$src1))]>;
defvar csrl = !cast<SDPatternOperator>("csrl_"#width);
def csrl_#width#_oneuse : HasOneUseBinOp<csrl>;
def clshr_rev_#width : PatFrag <(ops node:$src0, node:$src1),
  (csrl $src1, $src0)>;

def csra_#width : PatFrags<(ops node:$src0, node:$src1),
  [(sra node:$src0, node:$src1), (sra node:$src0, (csh_mask node:$src1))]>;
defvar csra = !cast<SDPatternOperator>("csra_"#width);
def csra_#width#_oneuse : HasOneUseBinOp<csra>;
def cashr_rev_#width : PatFrag <(ops node:$src0, node:$src1),
  (csra $src1, $src0)>;
} // end foreach width

def srl_16 : PatFrag<
  (ops node:$src0), (srl_oneuse node:$src0, (i32 16))
>;


def hi_i16_elt : PatFrag<
  (ops node:$src0), (i16 (trunc (i32 (srl_16 node:$src0))))
>;


```
**EN:** This section contains concrete logic for TableGen record csh_mask_64. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `csh_mask_64`, `cshl_#width`, `cshl_#width#_oneuse`.
**CN:** 本节包含与 TableGen record csh_mask_64 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`csh_mask_64`, `cshl_#width`, `cshl_#width#_oneuse`。

### Lines 334-377: Defines TableGen record hi_f16_elt
```tablegen
def hi_f16_elt : PatLeaf<
  (vt), [{
  if (N->getOpcode() != ISD::BITCAST)
    return false;
  SDValue Tmp = N->getOperand(0);

  if (Tmp.getOpcode() != ISD::SRL)
    return false;
    if (const auto *RHS = dyn_cast<ConstantSDNode>(Tmp.getOperand(1))
      return RHS->getZExtValue() == 16;
    return false;
}]>;

//===----------------------------------------------------------------------===//
// PatLeafs for zero immediate
//===----------------------------------------------------------------------===//

def immzero : PatLeaf<(imm), [{ return N->isZero(); }]>;
def fpimmzero : PatLeaf<(fpimm), [{ return N->isZero(); }]>;

//===----------------------------------------------------------------------===//
// PatLeafs for floating-point comparisons
//===----------------------------------------------------------------------===//

def COND_OEQ : PatFrags<(ops), [(OtherVT SETOEQ), (OtherVT SETEQ)]>;
def COND_ONE : PatFrags<(ops), [(OtherVT SETONE), (OtherVT SETNE)]>;
def COND_OGT : PatFrags<(ops), [(OtherVT SETOGT), (OtherVT SETGT)]>;
def COND_OGE : PatFrags<(ops), [(OtherVT SETOGE), (OtherVT SETGE)]>;
def COND_OLT : PatFrags<(ops), [(OtherVT SETOLT), (OtherVT SETLT)]>;
def COND_OLE : PatFrags<(ops), [(OtherVT SETOLE), (OtherVT SETLE)]>;
def COND_O   : PatFrags<(ops), [(OtherVT SETO)]>;
def COND_UO  : PatFrags<(ops), [(OtherVT SETUO)]>;

//===----------------------------------------------------------------------===//
// PatLeafs for unsigned / unordered comparisons
//===----------------------------------------------------------------------===//

def COND_UEQ : PatFrag<(ops), (OtherVT SETUEQ)>;
def COND_UNE : PatFrag<(ops), (OtherVT SETUNE)>;
def COND_UGT : PatFrag<(ops), (OtherVT SETUGT)>;
def COND_UGE : PatFrag<(ops), (OtherVT SETUGE)>;
def COND_ULT : PatFrag<(ops), (OtherVT SETULT)>;
def COND_ULE : PatFrag<(ops), (OtherVT SETULE)>;

```
**EN:** This section contains concrete logic for TableGen record hi_f16_elt. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `hi_f16_elt`, `immzero`, `fpimmzero`.
**CN:** 本节包含与 TableGen record hi_f16_elt 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`hi_f16_elt`, `immzero`, `fpimmzero`。

### Lines 378-415: Defines TableGen record COND_UNE_NE
```tablegen
// XXX - For some reason R600 version is preferring to use unordered
// for setne?
def COND_UNE_NE  : PatFrags<(ops), [(OtherVT SETUNE), (OtherVT SETNE)]>;

//===----------------------------------------------------------------------===//
// PatLeafs for signed comparisons
//===----------------------------------------------------------------------===//

def COND_SGT : PatFrag<(ops), (OtherVT SETGT)>;
def COND_SGE : PatFrag<(ops), (OtherVT SETGE)>;
def COND_SLT : PatFrag<(ops), (OtherVT SETLT)>;
def COND_SLE : PatFrag<(ops), (OtherVT SETLE)>;

//===----------------------------------------------------------------------===//
// PatLeafs for integer equality
//===----------------------------------------------------------------------===//

def COND_EQ : PatFrags<(ops), [(OtherVT SETEQ), (OtherVT SETUEQ)]>;
def COND_NE : PatFrags<(ops), [(OtherVT SETNE), (OtherVT SETUNE)]>;

// FIXME: Should not need code predicate
//def COND_NULL : PatLeaf<(OtherVT null_frag)>;
def COND_NULL : PatLeaf <
  (cond),
  [{(void)N; return false;}]
>;

//===----------------------------------------------------------------------===//
// PatLeafs for Texture Constants
//===----------------------------------------------------------------------===//

def TEX_ARRAY : PatLeaf<
  (imm),
  [{uint32_t TType = (uint32_t)N->getZExtValue();
    return TType == 9 || TType == 10 || TType == 16;
  }]
>;

```
**EN:** This section contains concrete logic for TableGen record COND_UNE_NE. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `COND_UNE_NE`, `COND_SGT`, `COND_SGE`.
**CN:** 本节包含与 TableGen record COND_UNE_NE 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`COND_UNE_NE`, `COND_SGT`, `COND_SGE`。

### Lines 416-458: Defines TableGen record TEX_RECT
```tablegen
def TEX_RECT : PatLeaf<
  (imm),
  [{uint32_t TType = (uint32_t)N->getZExtValue();
    return TType == 5;
  }]
>;

def TEX_SHADOW : PatLeaf<
  (imm),
  [{uint32_t TType = (uint32_t)N->getZExtValue();
    return (TType >= 6 && TType <= 8) || TType == 13;
  }]
>;

def TEX_SHADOW_ARRAY : PatLeaf<
  (imm),
  [{uint32_t TType = (uint32_t)N->getZExtValue();
    return TType == 11 || TType == 12 || TType == 17;
  }]
>;

//===----------------------------------------------------------------------===//
// Load/Store Pattern Fragments
//===----------------------------------------------------------------------===//

def atomic_cmp_swap_glue : SDNode <"ISD::ATOMIC_CMP_SWAP", SDTAtomic3,
  [SDNPHasChain, SDNPMayStore, SDNPMayLoad, SDNPMemOperand, SDNPInGlue]
>;

class AddressSpaceList<list<int> AS> {
  list<int> AddrSpaces = AS;
}

class Aligned<int Bytes> {
  int MinAlignment = Bytes;
}

class StoreHi16<SDPatternOperator op, ValueType vt> : PatFrag <
  (ops node:$value, node:$ptr), (op (srl node:$value, (i32 16)), node:$ptr)> {
  let IsStore = 1;
  let MemoryVT = vt;
}

```
**EN:** This section contains concrete logic for TableGen record TEX_RECT. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `TEX_RECT`, `TEX_SHADOW`, `TEX_SHADOW_ARRAY`.
**CN:** 本节包含与 TableGen record TEX_RECT 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`TEX_RECT`, `TEX_SHADOW`, `TEX_SHADOW_ARRAY`。

### Lines 459-500: Defines TableGen record LoadAddress_constant
```tablegen
def LoadAddress_constant : AddressSpaceList<[ AddrSpaces.Constant,
                                              AddrSpaces.Constant32Bit ]>;
def LoadAddress_global : AddressSpaceList<[ AddrSpaces.Global,
                                            AddrSpaces.Constant,
                                            AddrSpaces.Constant32Bit ]>;
def StoreAddress_global : AddressSpaceList<[ AddrSpaces.Global,
                                             AddrSpaces.Constant,
                                             AddrSpaces.Constant32Bit ]>;

def LoadAddress_flat : AddressSpaceList<[ AddrSpaces.Flat,
                                          AddrSpaces.Global,
                                          AddrSpaces.Constant,
                                          AddrSpaces.Constant32Bit ]>;
def StoreAddress_flat : AddressSpaceList<[ AddrSpaces.Flat, AddrSpaces.Global ]>;

def LoadAddress_private : AddressSpaceList<[ AddrSpaces.Private ]>;
def StoreAddress_private : AddressSpaceList<[ AddrSpaces.Private ]>;

def LoadAddress_local : AddressSpaceList<[ AddrSpaces.Local ]>;
def StoreAddress_local : AddressSpaceList<[ AddrSpaces.Local ]>;

def LoadAddress_region : AddressSpaceList<[ AddrSpaces.Region ]>;
def StoreAddress_region : AddressSpaceList<[ AddrSpaces.Region ]>;



foreach as = [ "global", "flat", "constant", "local", "private", "region" ] in {
let AddressSpaces = !cast<AddressSpaceList>("LoadAddress_"#as).AddrSpaces in {

def load_#as : PatFrag<(ops node:$ptr), (unindexedload node:$ptr)> {
  let IsLoad = 1;
  let IsNonExtLoad = 1;
}

def extloadi8_#as  : PatFrag<(ops node:$ptr), (extloadi8 node:$ptr)> {
  let IsLoad = 1;
}

def extloadi16_#as : PatFrag<(ops node:$ptr), (extloadi16 node:$ptr)> {
  let IsLoad = 1;
}

```
**EN:** This section contains concrete logic for TableGen record LoadAddress_constant. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `LoadAddress_constant`, `LoadAddress_global`, `StoreAddress_global`.
**CN:** 本节包含与 TableGen record LoadAddress_constant 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`LoadAddress_constant`, `LoadAddress_global`, `StoreAddress_global`。

### Lines 501-544: Defines TableGen record sextloadi8_#as
```tablegen
def sextloadi8_#as  : PatFrag<(ops node:$ptr), (sextloadi8 node:$ptr)> {
  let IsLoad = 1;
}

def sextloadi16_#as : PatFrag<(ops node:$ptr), (sextloadi16 node:$ptr)> {
  let IsLoad = 1;
}

def zextloadi8_#as  : PatFrag<(ops node:$ptr), (zextloadi8 node:$ptr)> {
  let IsLoad = 1;
}

def zextloadi16_#as : PatFrag<(ops node:$ptr), (zextloadi16 node:$ptr)> {
  let IsLoad = 1;
}

def atomic_load_nonext_16_#as : PatFrag<(ops node:$ptr), (atomic_load_nonext_16 node:$ptr)> {
  let IsAtomic = 1;
}

def atomic_load_nonext_32_#as : PatFrag<(ops node:$ptr), (atomic_load_nonext_32 node:$ptr)> {
  let IsAtomic = 1;
}

def atomic_load_nonext_64_#as : PatFrag<(ops node:$ptr), (atomic_load_nonext_64 node:$ptr)> {
  let IsAtomic = 1;
}

def atomic_load_nonext_128_#as : PatFrag<(ops node:$ptr), (atomic_load_nonext_128 node:$ptr)> {
  let IsAtomic = 1;
}

def atomic_load_zext_8_#as : PatFrag<(ops node:$ptr), (atomic_load_zext_8 node:$ptr)> {
  let IsAtomic = 1;
}

def atomic_load_sext_8_#as : PatFrag<(ops node:$ptr), (atomic_load_sext_8 node:$ptr)> {
  let IsAtomic = 1;
}

def atomic_load_aext_8_#as : PatFrag<(ops node:$ptr), (atomic_load_aext_8 node:$ptr)> {
  let IsAtomic = 1;
}

```
**EN:** This section contains concrete logic for TableGen record sextloadi8_#as. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `sextloadi8_#as`, `sextloadi16_#as`, `zextloadi8_#as`.
**CN:** 本节包含与 TableGen record sextloadi8_#as 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`sextloadi8_#as`, `sextloadi16_#as`, `zextloadi8_#as`。

### Lines 545-587: Defines TableGen record atomic_load_zext_16_#as
```tablegen
def atomic_load_zext_16_#as : PatFrag<(ops node:$ptr), (atomic_load_zext_16 node:$ptr)> {
  let IsAtomic = 1;
}

def atomic_load_sext_16_#as : PatFrag<(ops node:$ptr), (atomic_load_sext_16 node:$ptr)> {
  let IsAtomic = 1;
}

def atomic_load_aext_16_#as : PatFrag<(ops node:$ptr), (atomic_load_aext_16 node:$ptr)> {
  let IsAtomic = 1;
}

} // End let AddressSpaces
} // End foreach as


foreach as = [ "global", "flat", "local", "private", "region" ] in {
let IsStore = 1, AddressSpaces = !cast<AddressSpaceList>("StoreAddress_"#as).AddrSpaces in {
def store_#as : PatFrag<(ops node:$val, node:$ptr),
                    (unindexedstore node:$val, node:$ptr)> {
  let IsTruncStore = 0;
}

// truncstore fragments.
def truncstore_#as : PatFrag<(ops node:$val, node:$ptr),
                             (unindexedstore node:$val, node:$ptr)> {
  let IsTruncStore = 1;
}

// TODO: We don't really need the truncstore here. We can use
// unindexedstore with MemoryVT directly, which will save an
// unnecessary check that the memory size is less than the value type
// in the generated matcher table.
def truncstorei8_#as : PatFrag<(ops node:$val, node:$ptr),
                               (truncstorei8 node:$val, node:$ptr)>;
def truncstorei16_#as : PatFrag<(ops node:$val, node:$ptr),
                                (truncstorei16 node:$val, node:$ptr)>;

def store_hi16_#as : StoreHi16 <truncstorei16, i16>;
def truncstorei8_hi16_#as : StoreHi16<truncstorei8, i8>;
def truncstorei16_hi16_#as : StoreHi16<truncstorei16, i16>;
} // End let IsStore = 1, AddressSpaces = ...

```
**EN:** This section contains concrete logic for TableGen record atomic_load_zext_16_#as. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `atomic_load_zext_16_#as`, `atomic_load_sext_16_#as`, `atomic_load_aext_16_#as`.
**CN:** 本节包含与 TableGen record atomic_load_zext_16_#as 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`atomic_load_zext_16_#as`, `atomic_load_sext_16_#as`, `atomic_load_aext_16_#as`。

### Lines 588-623: Defines TableGen record atomic_store_8_#as
```tablegen
let IsAtomic = 1, AddressSpaces = !cast<AddressSpaceList>("StoreAddress_"#as).AddrSpaces in {
def atomic_store_8_#as : PatFrag<(ops node:$val, node:$ptr),
                                 (atomic_store_8 node:$val, node:$ptr)>;
def atomic_store_16_#as : PatFrag<(ops node:$val, node:$ptr),
                                  (atomic_store_16 node:$val, node:$ptr)>;
def atomic_store_32_#as : PatFrag<(ops node:$val, node:$ptr),
                                  (atomic_store_32 node:$val, node:$ptr)>;
def atomic_store_64_#as : PatFrag<(ops node:$val, node:$ptr),
                                  (atomic_store_64 node:$val, node:$ptr)>;
def atomic_store_128_#as : PatFrag<(ops node:$val, node:$ptr),
                                   (atomic_store_128 node:$val, node:$ptr)>;
} // End let IsAtomic = 1, AddressSpaces = ...
} // End foreach as

multiclass noret_op {
  let HasNoUse = true in
  def "_noret" : PatFrag<(ops node:$ptr, node:$data),
    (!cast<SDPatternOperator>(NAME) node:$ptr, node:$data)>;
}

multiclass global_addr_space_atomic_op {
  def "_noret_global_addrspace" :
    PatFrag<(ops node:$ptr, node:$data),
            (!cast<SDPatternOperator>(NAME) node:$ptr, node:$data)>{
      let HasNoUse = true;
      let AddressSpaces = LoadAddress_global.AddrSpaces;
      let IsAtomic = 1;
    }
    def "_global_addrspace" :
    PatFrag<(ops node:$ptr, node:$data),
            (!cast<SDPatternOperator>(NAME) node:$ptr, node:$data)>{
      let AddressSpaces = LoadAddress_global.AddrSpaces;
      let IsAtomic = 1;
    }
}

```
**EN:** This section contains concrete logic for TableGen record atomic_store_8_#as. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `atomic_store_8_#as`, `atomic_store_16_#as`, `atomic_store_32_#as`.
**CN:** 本节包含与 TableGen record atomic_store_8_#as 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`atomic_store_8_#as`, `atomic_store_16_#as`, `atomic_store_32_#as`。

### Lines 624-666: Defines TableGen multiclass flat_addr_space_atomic_op
```tablegen
multiclass flat_addr_space_atomic_op {
  def "_noret_flat_addrspace" :
    PatFrag<(ops node:$ptr, node:$data),
            (!cast<SDPatternOperator>(NAME) node:$ptr, node:$data)>{
      let HasNoUse = true;
      let AddressSpaces = LoadAddress_flat.AddrSpaces;
      let IsAtomic = 1;
    }
    def "_flat_addrspace" :
    PatFrag<(ops node:$ptr, node:$data),
            (!cast<SDPatternOperator>(NAME) node:$ptr, node:$data)>{
      let AddressSpaces = LoadAddress_flat.AddrSpaces;
      let IsAtomic = 1;
    }
}

multiclass local_addr_space_atomic_op {
  def "_noret_local_addrspace" :
    PatFrag<(ops node:$ptr, node:$data),
            (!cast<SDPatternOperator>(NAME) node:$ptr, node:$data)>{
      let HasNoUse = true;
      let AddressSpaces = LoadAddress_local.AddrSpaces;
      let IsAtomic = 1;
    }
    def "_local_addrspace" :
    PatFrag<(ops node:$ptr, node:$data),
            (!cast<SDPatternOperator>(NAME) node:$ptr, node:$data)>{
      let AddressSpaces = LoadAddress_local.AddrSpaces;
      let IsAtomic = 1;
    }
}

defm int_amdgcn_global_atomic_ordered_add_b64 : noret_op;
defm int_amdgcn_flat_atomic_fmin_num : noret_op;
defm int_amdgcn_flat_atomic_fmax_num : noret_op;
defm int_amdgcn_global_atomic_fmin_num : noret_op;
defm int_amdgcn_global_atomic_fmax_num : noret_op;

multiclass noret_binary_atomic_op<SDNode atomic_op> {
  let HasNoUse = true in
  defm "_noret" : binary_atomic_op<atomic_op>;
}

```
**EN:** This section contains concrete logic for TableGen multiclass flat_addr_space_atomic_op. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `flat_addr_space_atomic_op`, `local_addr_space_atomic_op`, `int_amdgcn_global_atomic_ordered_add_b64`.
**CN:** 本节包含与 TableGen multiclass flat_addr_space_atomic_op 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`flat_addr_space_atomic_op`, `local_addr_space_atomic_op`, `int_amdgcn_global_atomic_ordered_add_b64`。

### Lines 667-695: Defines TableGen multiclass noret_binary_atomic_op_fp
```tablegen
multiclass noret_binary_atomic_op_fp<SDNode atomic_op> {
  let HasNoUse = true in
  defm "_noret" : binary_atomic_op_fp<atomic_op>;
}

multiclass noret_ternary_atomic_op<SDNode atomic_op> {
  let HasNoUse = true in
  defm "_noret" : ternary_atomic_op<atomic_op>;
}

defvar atomic_addrspace_names = [ "global", "flat", "constant", "local", "private", "region" ];

multiclass binary_atomic_op_all_as<SDNode atomic_op> {
  foreach as = atomic_addrspace_names in {
    let AddressSpaces = !cast<AddressSpaceList>("LoadAddress_"#as).AddrSpaces in {
      defm "_"#as : binary_atomic_op<atomic_op>;
      defm "_"#as : noret_binary_atomic_op<atomic_op>;
    }
  }
}
multiclass binary_atomic_op_fp_all_as<SDNode atomic_op> {
  foreach as = atomic_addrspace_names in {
    let AddressSpaces = !cast<AddressSpaceList>("LoadAddress_"#as).AddrSpaces in {
      defm "_"#as : binary_atomic_op_fp<atomic_op>;
      defm "_"#as : noret_binary_atomic_op_fp<atomic_op>;
    }
  }
}

```
**EN:** This section contains concrete logic for TableGen multiclass noret_binary_atomic_op_fp. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `noret_binary_atomic_op_fp`, `noret_ternary_atomic_op`, `binary_atomic_op_all_as`.
**CN:** 本节包含与 TableGen multiclass noret_binary_atomic_op_fp 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`noret_binary_atomic_op_fp`, `noret_ternary_atomic_op`, `binary_atomic_op_all_as`。

### Lines 696-734: Defines TableGen record group atomic_swap
```tablegen
defm atomic_swap : binary_atomic_op_all_as<atomic_swap>;
defm atomic_load_add : binary_atomic_op_all_as<atomic_load_add>;
defm atomic_load_and : binary_atomic_op_all_as<atomic_load_and>;
defm atomic_load_max : binary_atomic_op_all_as<atomic_load_max>;
defm atomic_load_min : binary_atomic_op_all_as<atomic_load_min>;
defm atomic_load_or : binary_atomic_op_all_as<atomic_load_or>;
defm atomic_load_sub : binary_atomic_op_all_as<atomic_load_sub>;
defm atomic_load_umax : binary_atomic_op_all_as<atomic_load_umax>;
defm atomic_load_umin : binary_atomic_op_all_as<atomic_load_umin>;
defm atomic_load_xor : binary_atomic_op_all_as<atomic_load_xor>;
defm atomic_load_fadd : binary_atomic_op_fp_all_as<atomic_load_fadd>;
defm atomic_load_fmin : binary_atomic_op_fp_all_as<atomic_load_fmin>;
defm atomic_load_fmax : binary_atomic_op_fp_all_as<atomic_load_fmax>;
defm atomic_load_uinc_wrap : binary_atomic_op_all_as<atomic_load_uinc_wrap>;
defm atomic_load_udec_wrap : binary_atomic_op_all_as<atomic_load_udec_wrap>;
defm atomic_load_usub_cond : binary_atomic_op_all_as<atomic_load_usub_cond>;
defm atomic_load_usub_sat : binary_atomic_op_all_as<atomic_load_usub_sat>;
defm AMDGPUatomic_cmp_swap : binary_atomic_op_all_as<AMDGPUatomic_cmp_swap>;

def load_align8_local : PatFrag<(ops node:$ptr), (load_local node:$ptr)>,
                       Aligned<8> {
  let IsLoad = 1;
}

def load_align16_local : PatFrag<(ops node:$ptr), (load_local node:$ptr)>,
                        Aligned<16> {
  let IsLoad = 1;
}

def store_align8_local: PatFrag<(ops node:$val, node:$ptr),
                                (store_local node:$val, node:$ptr)>, Aligned<8> {
  let IsStore = 1;
}

def store_align16_local: PatFrag<(ops node:$val, node:$ptr),
                                (store_local node:$val, node:$ptr)>, Aligned<16> {
  let IsStore = 1;
}

```
**EN:** This section contains concrete logic for TableGen record group atomic_swap. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `atomic_swap`, `atomic_load_add`, `atomic_load_and`.
**CN:** 本节包含与 TableGen record group atomic_swap 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`atomic_swap`, `atomic_load_add`, `atomic_load_and`。

### Lines 735-774: Defines TableGen record group atomic_cmp_swap_local
```tablegen
let AddressSpaces = StoreAddress_local.AddrSpaces in {
defm atomic_cmp_swap_local : ternary_atomic_op<atomic_cmp_swap>;
defm atomic_cmp_swap_local : noret_ternary_atomic_op<atomic_cmp_swap>;
defm atomic_cmp_swap_local_m0 : noret_ternary_atomic_op<atomic_cmp_swap_glue>;
defm atomic_cmp_swap_local_m0 : ternary_atomic_op<atomic_cmp_swap_glue>;
}

let AddressSpaces = StoreAddress_region.AddrSpaces in {
defm atomic_cmp_swap_region : noret_ternary_atomic_op<atomic_cmp_swap>;
defm atomic_cmp_swap_region_m0 : noret_ternary_atomic_op<atomic_cmp_swap_glue>;
defm atomic_cmp_swap_region_m0 : ternary_atomic_op<atomic_cmp_swap_glue>;
}

//===----------------------------------------------------------------------===//
// Misc Pattern Fragments
//===----------------------------------------------------------------------===//

class Constants {
int TWO_PI = 0x40c90fdb;
int PI = 0x40490fdb;
int TWO_PI_INV = 0x3e22f983;
int FP_4294966784 = 0x4f7ffffe; // 4294966784 = 4294967296 - 512 = 2^32 - 2^9
int FP16_ONE = 0x3C00;
int FP16_NEG_ONE = 0xBC00;
int FP32_ONE = 0x3f800000;
int FP32_NEG_ONE = 0xbf800000;
int FP64_ONE = 0x3ff0000000000000;
int FP64_NEG_ONE = 0xbff0000000000000;
int BF16_ONE = 0x3F80;
int BF16_NEG_ONE = 0xBF80;
}
def CONST : Constants;

def fpimm_zero : FPImmLeaf<fAny, [{ return Imm.isZero(); }]> ;
def fpimm_one : FPImmLeaf<fAny, [{ return Imm.isExactlyValue(+1.0); }]> ;
def fpimm_half : FPImmLeaf<fAny, [{ return Imm.isExactlyValue(+0.5); }]> ;

/* Generic helper patterns for intrinsics */
/* -------------------------------------- */

```
**EN:** This section contains concrete logic for TableGen record group atomic_cmp_swap_local. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `atomic_cmp_swap_local`, `atomic_cmp_swap_local_m0`, `atomic_cmp_swap_region`.
**CN:** 本节包含与 TableGen record group atomic_cmp_swap_local 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`atomic_cmp_swap_local`, `atomic_cmp_swap_local_m0`, `atomic_cmp_swap_region`。

### Lines 775-816: Defines TableGen class POW_Common
```tablegen
class POW_Common <AMDGPUInst log_ieee, AMDGPUInst exp_ieee, AMDGPUInst mul>
  : AMDGPUPat <
  (fpow f32:$src0, f32:$src1),
  (exp_ieee (mul f32:$src1, (log_ieee f32:$src0)))
>;

/* Other helper patterns */
/* --------------------- */

/* Extract element pattern */
class Extract_Element <ValueType sub_type, ValueType vec_type, int sub_idx,
                       SubRegIndex sub_reg>
  : AMDGPUPat<
  (sub_type (extractelt vec_type:$src, sub_idx)),
  (EXTRACT_SUBREG $src, sub_reg)
>;

/* Insert element pattern */
class Insert_Element <ValueType elem_type, ValueType vec_type,
                      int sub_idx, SubRegIndex sub_reg>
  : AMDGPUPat <
  (insertelt vec_type:$vec, elem_type:$elem, sub_idx),
  (INSERT_SUBREG $vec, $elem, sub_reg)
>;

// XXX: Convert to new syntax and use COPY_TO_REG, once the DFAPacketizer
// can handle COPY instructions.
// bitconvert pattern
class BitConvert <ValueType dt, ValueType st, RegisterClass rc> : AMDGPUPat <
  (dt (bitconvert (st rc:$src0))),
  (dt rc:$src0)
>;

// XXX: Convert to new syntax and use COPY_TO_REG, once the DFAPacketizer
// can handle COPY instructions.
class DwordAddrPat<ValueType vt, RegisterClass rc> : AMDGPUPat <
  (vt (AMDGPUdwordaddr (vt rc:$addr))),
  (vt rc:$addr)
>;

// Special conversion patterns

```
**EN:** This section contains concrete logic for TableGen class POW_Common. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `POW_Common`, `Extract_Element`, `Insert_Element`.
**CN:** 本节包含与 TableGen class POW_Common 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`POW_Common`, `Extract_Element`, `Insert_Element`。

### Lines 817-858: Defines TableGen record cvt_rpi_i32_f32
```tablegen
let GIIgnoreCopies = 1 in
def cvt_rpi_i32_f32 : PatFrag<
  (ops node:$src),
  (fp_to_sint (ffloor_nnan (fadd $src, fpimm_half)))
>, GISelFlags;

def cvt_flr_i32_f32 : PatFrag<
  (ops node:$src),
  (fp_to_sint (ffloor_nnan $src))
>;

let AddedComplexity = 2 in {
class IMad24Pat<Instruction Inst, bit HasClamp = 0> : AMDGPUPat <
  (add (AMDGPUmul_i24_no_sdwa i32:$src0, i32:$src1), i32:$src2),
  !if(HasClamp, (Inst $src0, $src1, $src2, (i1 0)),
                (Inst $src0, $src1, $src2))
>;

class UMad24Pat<Instruction Inst, bit HasClamp = 0> : AMDGPUPat <
  (add (AMDGPUmul_u24_no_sdwa i32:$src0, i32:$src1), i32:$src2),
  !if(HasClamp, (Inst $src0, $src1, $src2, (i1 0)),
                (Inst $src0, $src1, $src2))
>;
} // AddedComplexity.

class RcpPat<Instruction RcpInst, ValueType vt> : AMDGPUPat <
  (fdiv fpimm_one, vt:$src),
  (RcpInst $src)
>;

// Instructions which select to the same v_min_f*
def fminnum_like : PatFrags<(ops node:$src0, node:$src1),
  [(fminnum_ieee node:$src0, node:$src1),
   (fminnum node:$src0, node:$src1)]
>;

// Instructions which select to the same v_max_f*
def fmaxnum_like : PatFrags<(ops node:$src0, node:$src1),
  [(fmaxnum_ieee node:$src0, node:$src1),
   (fmaxnum node:$src0, node:$src1)]
>;

```
**EN:** This section contains concrete logic for TableGen record cvt_rpi_i32_f32. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `cvt_rpi_i32_f32`, `cvt_flr_i32_f32`, `IMad24Pat`.
**CN:** 本节包含与 TableGen record cvt_rpi_i32_f32 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`cvt_rpi_i32_f32`, `cvt_flr_i32_f32`, `IMad24Pat`。

### Lines 859-896: Defines TableGen class NeverNaNPats
```tablegen
class NeverNaNPats<dag ops, list<dag> frags> : PatFrags<ops, frags> {
  let PredicateCode = [{
    return CurDAG->isKnownNeverNaN(SDValue(N,0));
  }];
  let GISelPredicateCode = [{
    return VT->isKnownNeverNaN(MI.getOperand(0).getReg());
  }];
}

def fminnum_like_nnan : NeverNaNPats<(ops node:$src0, node:$src1),
  [(fminnum_ieee node:$src0, node:$src1),
   (fminnum node:$src0, node:$src1)]
>;

def fmaxnum_like_nnan : NeverNaNPats<(ops node:$src0, node:$src1),
  [(fmaxnum_ieee node:$src0, node:$src1),
   (fmaxnum node:$src0, node:$src1)]
>;

def fminnum_like_oneuse : PatFrags<(ops node:$src0, node:$src1),
  [(fminnum_ieee_oneuse node:$src0, node:$src1),
   (fminnum_oneuse node:$src0, node:$src1)]
>;

def fmaxnum_like_oneuse : PatFrags<(ops node:$src0, node:$src1),
  [(fmaxnum_ieee_oneuse node:$src0, node:$src1),
   (fmaxnum_oneuse node:$src0, node:$src1)]
>;

def any_fmad : PatFrags<(ops node:$src0, node:$src1, node:$src2),
  [(fmad node:$src0, node:$src1, node:$src2),
   (AMDGPUfmad_ftz node:$src0, node:$src1, node:$src2)]
>;

// FIXME: fsqrt should not select directly
def any_amdgcn_sqrt : PatFrags<(ops node:$src0),
  [(fsqrt node:$src0), (int_amdgcn_sqrt node:$src0)]
>;
```
**EN:** This section contains concrete logic for TableGen class NeverNaNPats. It performs local backend work and encodes target-specific behavior needed by the surrounding pipeline. Main symbols: `NeverNaNPats`, `fminnum_like_nnan`, `fmaxnum_like_nnan`.
**CN:** 本节包含与 TableGen class NeverNaNPats 相关的具体逻辑，负责实现局部后端工作并编码周边流程所需的目标专用行为。 主要符号：`NeverNaNPats`, `fminnum_like_nnan`, `fmaxnum_like_nnan`。

## Key Concepts / 关键概念
- **Language / 语言**: TableGen DSL
- **Primary symbols / 主要符号**: `AddrSpaces`, `AMDGPUInst`, `AMDGPUShaderInst`, `ILFormat`, `RegListUnion`, `AMDGPUPat`
- **Main themes / 核心主题**: register management / 寄存器管理; instruction semantics / 指令语义; subtarget modeling / 子目标建模; assembly handling / 汇编处理
- **Build role / 构建角色**: Generates target records consumed by LLVM TableGen backends / 生成供 LLVM TableGen 后端消费的目标记录

## Dependencies / 依赖关系
- No direct include statements in this file / 此文件中没有直接的 include 语句
