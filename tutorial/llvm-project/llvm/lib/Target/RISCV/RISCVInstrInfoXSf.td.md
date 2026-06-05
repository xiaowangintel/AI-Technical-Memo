# RISCVInstrInfoXSf.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoXSf.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoXsf.td - SiFive custom instructions --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the vendor extensions defined by SiFive.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// XSFVCP extension instructions.
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 16-32: TableGen class VCIXType<bits<4> / TableGen 类 VCIXType<bits<4>
```tablegen

def VCIXVS2    : RISCVVConstraint<VS2 = 1>;
def VCIXVS2VS1 : RISCVVConstraint<VS2 = 1, VS1 = 1>;

class VCIXType<bits<4> val, string suffix> {
  bits<4> Val = val;
  string Suffix = suffix;
  bit Vs2IsSource = 1;      // Whether vs2 is a register source
  bit VdIsSource = 0;       // Whether vd appears in ins as a source operand
  bit IsWidening = 0;       // Whether this is a widening instruction (XVW)
}

def VCIX_Rs1   : VCIXType<0b0000, "">   { let Vs2IsSource = 0; }
def VCIX_Rs1V  : VCIXType<0b0010, "v">;
def VCIX_Rs1VV : VCIXType<0b1010, "vv"> { let VdIsSource = 1; }
def VCIX_Rs1VW : VCIXType<0b1111, "vw"> { let VdIsSource = 1; let IsWidening = 1; }
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 33-46: TableGen class to / TableGen 类 to
```tablegen
// Helper class to encapsulate Rs1-related properties
class VCIXRs1Info<DAGOperand ty, bits<3> funct3, string prefix, string name,
                  dag op> {
  DAGOperand Ty = ty;
  bits<3> Funct3 = funct3;
  string Prefix = prefix;
  string Name = name;
  dag Op = op;
  bit IsVR = !eq(ty, VR);
  string Funct6Name = !if(!eq(ty, FPR32), "$funct6_lo1", "$funct6_lo2");
  dag Funct6Op = !if(!eq(ty, FPR32), (ins uimm1:$funct6_lo1),
                                     (ins uimm2:$funct6_lo2));
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 47-61: TableGen class PayloadOp<int / TableGen 类 PayloadOp<int
```tablegen
def VCIX_GPR   : VCIXRs1Info<GPR,   0b100, "x", "$rs1", (ins GPR:$rs1)>;
def VCIX_FPR32 : VCIXRs1Info<FPR32, 0b101, "f", "$rs1", (ins FPR32:$rs1)>;
def VCIX_VR    : VCIXRs1Info<VR,    0b000, "v", "$vs1", (ins VR:$vs1)>;
def VCIX_simm5 : VCIXRs1Info<simm5, 0b011, "i", "$imm", (ins simm5:$imm)>;

// The payload and tsimm5 operands are all marked as ImmArg in the IR
// intrinsic and will be target constant, so use TImmLeaf rather than ImmLeaf.
class PayloadOp<int bitsNum> : RISCVOp, TImmLeaf<XLenVT, "return isUInt<" # bitsNum # ">(Imm);"> {
  let OperandType = "OPERAND_UIMM" # bitsNum;
}

def payload1 : PayloadOp<1>;
def payload2 : PayloadOp<2>;
def payload5 : PayloadOp<5>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 62-79: TableGen class VCIXInfo<VCIXType / TableGen 类 VCIXInfo<VCIXType
```tablegen
def tsimm5 : RISCVOp, TImmLeaf<XLenVT, [{return isInt<5>(Imm);}]> {
  let ParserMatchClass = SImmAsmOperand<5>;
  let EncoderMethod = "getImmOpValue";
  let DecoderMethod = "decodeSImmOperand<5>";
  let MCOperandPredicate = [{
    int64_t Imm;
    if (MCOp.evaluateAsConstantImm(Imm))
      return isInt<5>(Imm);
    return MCOp.isBareSymbolRef();
  }];
  let OperandType = "OPERAND_SIMM5";
}

class VCIXInfo<VCIXType type, VCIXRs1Info rs1info, bit hasOutput> {
  bit HasOutput = hasOutput;
  VCIXRs1Info Rs1 = rs1info;
  defvar Suffix = rs1info.Prefix # type.Suffix;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 80-95: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
  string OpcodeStr = !if(hasOutput, "sf.vc.v." # Suffix,
                                    "sf.vc." # Suffix);
  bits<4> Funct6_hi4 = type.Val;
  bits<3> Funct3 = rs1info.Funct3;

  // TyRd is VR when there's an output or vd is a source operand, otherwise uimm5
  defvar TyRd = !if(!or(hasOutput, type.VdIsSource), VR, uimm5);

  dag Outs = !cond(!not(hasOutput): (outs),
                   type.VdIsSource: (outs TyRd:$vd_wb),
                   true:            (outs TyRd:$vd));

  // Vd operand: omitted from ins when hasOutput and not a source operand
  defvar VdOp = !if(!and(hasOutput, !not(type.VdIsSource)),
                    (ins), (ins TyRd:$vd));
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 96-111: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
  // TyRs2 is uimm5 when vs2 is an immediate, otherwise VR
  defvar TyRs2 = !if(type.Vs2IsSource, VR, uimm5);
  defvar Vs2Op = (ins TyRs2:$vs2);

  // Build Ins DAG: swap vd and vs2 order when vs2 is not a register source
  dag Ins = !if(type.Vs2IsSource,
                !con(rs1info.Funct6Op, VdOp, Vs2Op, rs1info.Op),
                !con(rs1info.Funct6Op, Vs2Op, VdOp, rs1info.Op));

  string ArgStr = !if(type.Vs2IsSource,
                      rs1info.Funct6Name # ", $vd, $vs2, " # rs1info.Name,
                      rs1info.Funct6Name # ", $vs2, $vd, " # rs1info.Name);

  string Constraints = !if(!and(hasOutput, type.VdIsSource),
                           "$vd = $vd_wb", "");
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 112-129: TableGen class for / TableGen 类 for
```tablegen
  RISCVVConstraint VS1VS2Constraint = !if(!and(hasOutput, type.IsWidening),
                                       !if(rs1info.IsVR, VCIXVS2VS1, VCIXVS2),
                                       NoConstraint);
}

// Common base class for VCIX instructions
class RVInstVCCustom2Base<VCIXInfo info>
    : RVInst<info.Outs, info.Ins, info.OpcodeStr, info.ArgStr, [], InstFormatR> {
  bits<5> vs2;
  bits<5> vd;

  let Inst{31-28} = info.Funct6_hi4;
  let Inst{25} = !not(info.HasOutput);
  let Inst{24-20} = vs2;
  let Inst{14-12} = info.Funct3;
  let Inst{11-7} = vd;
  let Inst{6-0} = OPC_CUSTOM_2.Value;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 130-145: TableGen class RVInstVCCustom2_X<VCIXInfo / TableGen 类 RVInstVCCustom2_X<VCIXInfo
```tablegen
  let Uses = [VL, VTYPE];
  let Constraints = info.Constraints;
  let VS1VS2Constraint = info.VS1VS2Constraint;
  let ReadsPastVL = 1;

  let UseNamedOperandTable = true;
}

// VCIX instructions with GPR rs1 operand
class RVInstVCCustom2_X<VCIXInfo info> : RVInstVCCustom2Base<info> {
  bits<5> rs1;
  bits<2> funct6_lo2;

  let Inst{27-26} = funct6_lo2;
  let Inst{19-15} = rs1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 146-160: TableGen class RVInstVCCustom2_V<VCIXInfo / TableGen 类 RVInstVCCustom2_V<VCIXInfo
```tablegen

// VCIX instructions with VR vs1 operand
class RVInstVCCustom2_V<VCIXInfo info> : RVInstVCCustom2Base<info> {
  bits<5> vs1;
  bits<2> funct6_lo2;

  let Inst{27-26} = funct6_lo2;
  let Inst{19-15} = vs1;
}

// VCIX instructions with immediate operand
class RVInstVCCustom2_I<VCIXInfo info> : RVInstVCCustom2Base<info> {
  bits<5> imm;
  bits<2> funct6_lo2;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 161-174: TableGen class RVInstVCFCustom2_F<VCIXInfo / TableGen 类 RVInstVCFCustom2_F<VCIXInfo
```tablegen
  let Inst{27-26} = funct6_lo2;
  let Inst{19-15} = imm;
}

// VCIX instructions with FPR rs1 operand
class RVInstVCFCustom2_F<VCIXInfo info> : RVInstVCCustom2Base<info> {
  bits<5> rs1;
  bit funct6_lo1;

  let Inst{27} = 1;
  let Inst{26} = funct6_lo1;
  let Inst{19-15} = rs1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 175-191: Reusable TableGen multiclass CustomSiFiveVCIXorVCIF / 可复用的 TableGen 多类 CustomSiFiveVCIXorVCIF
```tablegen
multiclass CustomSiFiveVCIXorVCIF<VCIXInfo info> {
  defvar TyRs1 = info.Rs1.Ty;
  if !eq(TyRs1, FPR32) then {
    def NAME : RVInstVCFCustom2_F<info>;
  } else if !eq(TyRs1, GPR) then {
    def NAME : RVInstVCCustom2_X<info>;
  } else if !eq(TyRs1, VR) then {
    def NAME : RVInstVCCustom2_V<info>;
  } else {
    def NAME : RVInstVCCustom2_I<info>;
  }
}

multiclass CustomSiFiveVCIX<VCIXType type, VCIXRs1Info rs1> {
  defm SF_VC_ # NAME   : CustomSiFiveVCIXorVCIF<VCIXInfo<type, rs1, hasOutput=0>>;
  defm SF_VC_V_ # NAME : CustomSiFiveVCIXorVCIF<VCIXInfo<type, rs1, hasOutput=1>>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 192-209: Bulk record instantiation X / 批量记录实例化 X
```tablegen

let Predicates = [HasVendorXSfvcp], mayLoad = 0, mayStore = 0,
    hasSideEffects = 1, hasNoSchedulingInfo = 1, DecoderNamespace = "XSfvector" in {
  defm X   : CustomSiFiveVCIX<VCIX_Rs1,   VCIX_GPR>,   Sched<[]>;
  defm I   : CustomSiFiveVCIX<VCIX_Rs1,   VCIX_simm5>, Sched<[]>;
  defm XV  : CustomSiFiveVCIX<VCIX_Rs1V,  VCIX_GPR>,   Sched<[]>;
  defm IV  : CustomSiFiveVCIX<VCIX_Rs1V,  VCIX_simm5>, Sched<[]>;
  defm VV  : CustomSiFiveVCIX<VCIX_Rs1V,  VCIX_VR>,    Sched<[]>;
  defm FV  : CustomSiFiveVCIX<VCIX_Rs1V,  VCIX_FPR32>, Sched<[]>;
  defm XVV : CustomSiFiveVCIX<VCIX_Rs1VV, VCIX_GPR>,   Sched<[]>;
  defm IVV : CustomSiFiveVCIX<VCIX_Rs1VV, VCIX_simm5>, Sched<[]>;
  defm VVV : CustomSiFiveVCIX<VCIX_Rs1VV, VCIX_VR>,    Sched<[]>;
  defm FVV : CustomSiFiveVCIX<VCIX_Rs1VV, VCIX_FPR32>, Sched<[]>;
  defm XVW : CustomSiFiveVCIX<VCIX_Rs1VW, VCIX_GPR>,   Sched<[]>;
  defm IVW : CustomSiFiveVCIX<VCIX_Rs1VW, VCIX_simm5>, Sched<[]>;
  defm VVW : CustomSiFiveVCIX<VCIX_Rs1VW, VCIX_VR>,    Sched<[]>;
  defm FVW : CustomSiFiveVCIX<VCIX_Rs1VW, VCIX_FPR32>, Sched<[]>;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 210-231: TableGen class SiFiveVMACCScheds<string / TableGen 类 SiFiveVMACCScheds<string
```tablegen

// For XSfvqmaccdod/qoq and XSfvfwmaccqqq
class SiFiveVMACCScheds<string name> {
  defvar n = !tolower(name);
  defvar prefix = !if(!ne(!find(n, "fw"), -1), "FW", "Q");
  defvar suffix = !if(!ne(!find(n, "2x8x2"), -1), "DOD",
                      !if(!eq(prefix, "Q"), "QOQ", "QQQ"));

  string read = "ReadSF_V" # prefix # "MACC_" # suffix;
  string write = "WriteSF_V" # prefix # "MACC_" # suffix;
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in {
class CustomSiFiveVMACC<bits<6> funct6, RISCVVFormat opv, string opcodestr>
    : RVInstVV<funct6, opv, (outs VR:$vd_wb), (ins VR:$vd, VR:$vs1, VR:$vs2), opcodestr,
               "$vd, $vs1, $vs2">,
      SchedTernaryMC<SiFiveVMACCScheds<NAME>.write,
                     SiFiveVMACCScheds<NAME>.read,
                     SiFiveVMACCScheds<NAME>.read,
                     SiFiveVMACCScheds<NAME>.read> {
  let vm = 1;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 232-245: TableGen class CustomSiFiveVFNRCLIP<bits<6> / TableGen 类 CustomSiFiveVFNRCLIP<bits<6>
```tablegen
  let Inst{6-0} = OPC_CUSTOM_2.Value;

  let Constraints = "$vd = $vd_wb";
  let VMConstraint = false;
  let ReadsPastVL = true;
}
}

class CustomSiFiveVFNRCLIP<bits<6> funct6, RISCVVFormat opv, string opcodestr>
    : VALUVF<funct6, opv, opcodestr>,
      SchedBinaryMC<"WriteSF_VFNRClipV", "ReadSF_VFNRClipV", "ReadSF_VFNRClipF"> {
  let Inst{6-0} = OPC_CUSTOM_2.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 246-262: TableGen record SF_VFEXP_V / TableGen 记录 SF_VFEXP_V
```tablegen
let Predicates = [HasVendorXSfvfexpAny], DecoderNamespace = "XSfvector" in {
  def SF_VFEXP_V : VALUVs2<0b010011, 0b00111, OPFVV, "sf.vfexp.v">,
                   SchedUnaryMC<"WriteSF_VFExp", "ReadSF_VFExp">;
}

let Predicates = [HasVendorXSfvfexpa], DecoderNamespace = "XSfvector" in {
  def SF_VFEXPA_V : VALUVs2<0b010011, 0b00110, OPFVV, "sf.vfexpa.v">,
                    SchedUnaryMC<"WriteSF_VFExpa", "ReadSF_VFExpa">;
}

let Predicates = [HasVendorXSfvqmaccdod], DecoderNamespace = "XSfvector",
    DestEEW = EEWSEWx4, VS1VS2Constraint=VS1Constraint in {
  def SF_VQMACCU_2x8x2  : CustomSiFiveVMACC<0b101100, OPMVV, "sf.vqmaccu.2x8x2">;
  def SF_VQMACC_2x8x2   : CustomSiFiveVMACC<0b101101, OPMVV, "sf.vqmacc.2x8x2">;
  def SF_VQMACCUS_2x8x2 : CustomSiFiveVMACC<0b101110, OPMVV, "sf.vqmaccus.2x8x2">;
  def SF_VQMACCSU_2x8x2 : CustomSiFiveVMACC<0b101111, OPMVV, "sf.vqmaccsu.2x8x2">;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 263-276: TableGen record SF_VQMACCU_4x8x4 / TableGen 记录 SF_VQMACCU_4x8x4
```tablegen

let Predicates = [HasVendorXSfvqmaccqoq], DecoderNamespace = "XSfvector",
    DestEEW = EEWSEWx4, VS1VS2Constraint=WidenV in {
  def SF_VQMACCU_4x8x4  : CustomSiFiveVMACC<0b111100, OPMVV, "sf.vqmaccu.4x8x4">;
  def SF_VQMACC_4x8x4   : CustomSiFiveVMACC<0b111101, OPMVV, "sf.vqmacc.4x8x4">;
  def SF_VQMACCUS_4x8x4 : CustomSiFiveVMACC<0b111110, OPMVV, "sf.vqmaccus.4x8x4">;
  def SF_VQMACCSU_4x8x4 : CustomSiFiveVMACC<0b111111, OPMVV, "sf.vqmaccsu.4x8x4">;
}

let Predicates = [HasVendorXSfvfwmaccqqq], DecoderNamespace = "XSfvector",
    DestEEW = EEWSEWx2, VS1VS2Constraint=WidenV in {
  def SF_VFWMACC_4x4x4 : CustomSiFiveVMACC<0b111100, OPFVV, "sf.vfwmacc.4x4x4">;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 277-292: TableGen class VPseudoVC_X<Operand / TableGen 类 VPseudoVC_X<Operand
```tablegen
let Predicates = [HasVendorXSfvfnrclipxfqf], DecoderNamespace = "XSfvector",
    Uses = [FRM, VL, VTYPE] in {
  def SF_VFNRCLIP_XU_F_QF : CustomSiFiveVFNRCLIP<0b100010, OPFVF, "sf.vfnrclip.xu.f.qf">;
  def SF_VFNRCLIP_X_F_QF : CustomSiFiveVFNRCLIP<0b100011, OPFVF, "sf.vfnrclip.x.f.qf">;
}

class VPseudoVC_X<Operand OpClass, DAGOperand RS1Class> :
      RISCVVPseudo<(outs),
                   (ins OpClass:$op1, payload5:$rs2, payload5:$rd,
                        RS1Class:$rs1, AVL:$vl, sew:$sew), []> {
  let mayLoad = 0;
  let mayStore = 0;
  let HasVLOp = 1;
  let HasSEWOp = 1;
  let hasSideEffects = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 293-315: TableGen class VPseudoVC_XV<Operand / TableGen 类 VPseudoVC_XV<Operand
```tablegen

class VPseudoVC_XV<Operand OpClass, VReg RS2Class, DAGOperand RS1Class> :
      RISCVVPseudo<(outs),
                   (ins OpClass:$op1, payload5:$rd, RS2Class:$rs2,
                        RS1Class:$rs1, AVL:$vl, sew:$sew), []> {
  let mayLoad = 0;
  let mayStore = 0;
  let HasVLOp = 1;
  let HasSEWOp = 1;
  let hasSideEffects = 0;
}

class VPseudoVC_XVV<Operand OpClass, VReg RDClass, VReg RS2Class,
                    DAGOperand RS1Class> :
      RISCVVPseudo<(outs),
                   (ins OpClass:$op1, RDClass:$rd, RS2Class:$rs2, RS1Class:$rs1,
                        AVL:$vl, sew:$sew), []> {
  let mayLoad = 0;
  let mayStore = 0;
  let HasVLOp = 1;
  let HasSEWOp = 1;
  let hasSideEffects = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 316-338: TableGen class VPseudoVC_V_X<Operand / TableGen 类 VPseudoVC_V_X<Operand
```tablegen

class VPseudoVC_V_X<Operand OpClass, VReg RDClass, DAGOperand RS1Class> :
      RISCVVPseudo<(outs RDClass:$rd),
                   (ins OpClass:$op1, payload5:$rs2, RS1Class:$rs1,
                        AVL:$vl, sew:$sew), []> {
  let mayLoad = 0;
  let mayStore = 0;
  let HasVLOp = 1;
  let HasSEWOp = 1;
  let hasSideEffects = 0;
}

class VPseudoVC_V_XV<Operand OpClass, VReg RDClass, VReg RS2Class,
                     DAGOperand RS1Class> :
      RISCVVPseudo<(outs RDClass:$rd),
                   (ins OpClass:$op1, RS2Class:$rs2, RS1Class:$rs1,
                        AVL:$vl, sew:$sew), []> {
  let mayLoad = 0;
  let mayStore = 0;
  let HasVLOp = 1;
  let HasSEWOp = 1;
  let hasSideEffects = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 339-352: TableGen class VPseudoVC_V_XVV<Operand / TableGen 类 VPseudoVC_V_XVV<Operand
```tablegen

class VPseudoVC_V_XVV<Operand OpClass, VReg RDClass, VReg RS2Class,
                      DAGOperand RS1Class> :
      RISCVVPseudo<(outs RDClass:$rd),
                   (ins OpClass:$op1, RDClass:$rs3, RS2Class:$rs2, RS1Class:$rs1,
                        AVL:$vl, sew:$sew), []> {
  let mayLoad = 0;
  let mayStore = 0;
  let HasVLOp = 1;
  let HasSEWOp = 1;
  let hasSideEffects = 0;
  let Constraints = "$rd = $rs3";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 353-367: Reusable TableGen multiclass VPseudoVC_X / 可复用的 TableGen 多类 VPseudoVC_X
```tablegen
multiclass VPseudoVC_X<LMULInfo m, DAGOperand RS1Class,
                       Operand OpClass = payload2> {
  let VLMul = m.value in {
    let Defs = [SF_VCIX_STATE], Uses = [SF_VCIX_STATE] in {
      def "PseudoSF_VC_" # NAME # "_SE_" # m.MX
        : VPseudoVC_X<OpClass, RS1Class>,
          Sched<[!cast<SchedWrite>("WriteVC_" # NAME # "_" # m.MX)]>;
      def "PseudoSF_VC_V_" # NAME # "_SE_" # m.MX
        : VPseudoVC_V_X<OpClass, m.vrclass, RS1Class>,
          Sched<[!cast<SchedWrite>("WriteVC_V_" # NAME # "_" # m.MX)]>;
    }
    def "PseudoSF_VC_V_" # NAME # "_" # m.MX
      : VPseudoVC_V_X<OpClass, m.vrclass, RS1Class>,
        Sched<[!cast<SchedWrite>("WriteVC_V_" # NAME # "_" # m.MX)]>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 368-384: Reusable TableGen multiclass VPseudoVC_XV / 可复用的 TableGen 多类 VPseudoVC_XV
```tablegen
}

multiclass VPseudoVC_XV<LMULInfo m, DAGOperand RS1Class,
                        Operand OpClass = payload2> {
  let VLMul = m.value in {
    let Defs = [SF_VCIX_STATE], Uses = [SF_VCIX_STATE] in {
      def "PseudoSF_VC_" # NAME # "_SE_" # m.MX
        : VPseudoVC_XV<OpClass, m.vrclass, RS1Class>,
          Sched<[!cast<SchedWrite>("WriteVC_" # NAME # "_" # m.MX)]>;
      def "PseudoSF_VC_V_" # NAME # "_SE_" # m.MX
        : VPseudoVC_V_XV<OpClass, m.vrclass, m.vrclass, RS1Class>,
          Sched<[!cast<SchedWrite>("WriteVC_V_" # NAME # "_" # m.MX)]>;
    }
    def "PseudoSF_VC_V_" # NAME # "_" # m.MX
      : VPseudoVC_V_XV<OpClass, m.vrclass, m.vrclass, RS1Class>,
        Sched<[!cast<SchedWrite>("WriteVC_V_" # NAME # "_" # m.MX)]>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 385-401: Reusable TableGen multiclass VPseudoVC_XVV / 可复用的 TableGen 多类 VPseudoVC_XVV
```tablegen
}

multiclass VPseudoVC_XVV<LMULInfo m, DAGOperand RS1Class,
                         Operand OpClass = payload2> {
  let VLMul = m.value in {
    let Defs = [SF_VCIX_STATE], Uses = [SF_VCIX_STATE] in {
      def "PseudoSF_VC_" # NAME # "_SE_" # m.MX
        : VPseudoVC_XVV<OpClass, m.vrclass, m.vrclass, RS1Class>,
          Sched<[!cast<SchedWrite>("WriteVC_" # NAME # "_" # m.MX)]>;
      def "PseudoSF_VC_V_" # NAME # "_SE_" # m.MX
        : VPseudoVC_V_XVV<OpClass, m.vrclass, m.vrclass, RS1Class>,
          Sched<[!cast<SchedWrite>("WriteVC_V_" # NAME # "_" # m.MX)]>;
    }
    def "PseudoSF_VC_V_" # NAME # "_" # m.MX
      : VPseudoVC_V_XVV<OpClass, m.vrclass, m.vrclass, RS1Class>,
        Sched<[!cast<SchedWrite>("WriteVC_V_" # NAME # "_" # m.MX)]>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 402-419: Reusable TableGen multiclass VPseudoVC_XVW / 可复用的 TableGen 多类 VPseudoVC_XVW
```tablegen
}

multiclass VPseudoVC_XVW<LMULInfo m, DAGOperand RS1Class,
                         Operand OpClass = payload2> {
  let VLMul = m.value in {
    let Defs = [SF_VCIX_STATE], Uses = [SF_VCIX_STATE] in
    def "PseudoSF_VC_" # NAME # "_SE_" # m.MX
      : VPseudoVC_XVV<OpClass, m.wvrclass, m.vrclass, RS1Class>,
        Sched<[!cast<SchedWrite>("WriteVC_" # NAME # "_" # m.MX)]>;
    let Constraints = "@earlyclobber $rd, $rd = $rs3" in {
      let Defs = [SF_VCIX_STATE], Uses = [SF_VCIX_STATE] in
      def "PseudoSF_VC_V_" # NAME # "_SE_" # m.MX
        : VPseudoVC_V_XVV<OpClass, m.wvrclass, m.vrclass, RS1Class>,
          Sched<[!cast<SchedWrite>("WriteVC_V_" # NAME # "_" # m.MX)]>;
      def "PseudoSF_VC_V_" # NAME # "_" # m.MX
        : VPseudoVC_V_XVV<OpClass, m.wvrclass, m.vrclass, RS1Class>,
          Sched<[!cast<SchedWrite>("WriteVC_V_" # NAME # "_" # m.MX)]>;
    }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 420-437: Reusable TableGen multiclass VPseudoSiFiveVMACC / 可复用的 TableGen 多类 VPseudoSiFiveVMACC
```tablegen
  }
}

multiclass VPseudoSiFiveVMACC<string mx, VReg vd_type, VReg vs2_type> {
  defvar SchedWriteName = SiFiveVMACCScheds<NAME>.write;
  defvar SchedReadName = SiFiveVMACCScheds<NAME>.read;
  def "Pseudo" # NAME # "_" # mx
      : VPseudoTernaryNoMaskWithPolicy<vd_type, V_M1.vrclass, vs2_type,
                                       "@earlyclobber $rd">,
        SchedTernary<SchedWriteName, SchedReadName, SchedReadName,
                     SchedReadName, mx>;
}

multiclass VPseudoSiFiveVQMACCDOD {
  foreach m = MxListVF8 in
    let VLMul = m.value in
    defm NAME : VPseudoSiFiveVMACC<m.MX, m.vrclass, m.vrclass>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 438-462: Reusable TableGen multiclass VPseudoSiFiveVQMACCQOQ / 可复用的 TableGen 多类 VPseudoSiFiveVQMACCQOQ
```tablegen

multiclass VPseudoSiFiveVQMACCQOQ {
  foreach m = [V_MF2, V_M1, V_M2, V_M4] in
    let VLMul = m.value in
    defm NAME : VPseudoSiFiveVMACC<m.MX, m.wvrclass, m.vrclass>;
}

multiclass VPseudoSiFiveVFWMACC {
  foreach m = MxListFW in
    let VLMul = m.value in
    defm NAME : VPseudoSiFiveVMACC<m.MX, m.wvrclass, m.vrclass>;
}

multiclass VPseudoSiFiveVFNRCLIP<string Constraint = "@earlyclobber $rd"> {
  foreach i = 0-4 in
    let hasSideEffects = 0, hasPostISelHook = 1 in
      defm "Pseudo" # NAME : VPseudoBinaryRoundingMode<MxListW[i].vrclass,
                                                       MxListVF4[i].vrclass,
                                                       FPR32, MxListW[i],
                                                       Constraint, /*sew*/0,
                                                       UsesVXRM=0>,
                             SchedBinary<"WriteSF_VFNRClipV", "ReadSF_VFNRClipV",
                                         "ReadSF_VFNRClipF",
                                         MxListW[i].MX>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 463-480: Bulk record instantiation m / 批量记录实例化 m
```tablegen

let Predicates = [HasVendorXSfvcp] in {
  foreach m = MxList in {
    defm X : VPseudoVC_X<m, GPR>;
    defm I : VPseudoVC_X<m, tsimm5>;
    defm XV : VPseudoVC_XV<m, GPR>;
    defm IV : VPseudoVC_XV<m, tsimm5>;
    defm VV : VPseudoVC_XV<m, m.vrclass>;
    defm XVV : VPseudoVC_XVV<m, GPR>;
    defm IVV : VPseudoVC_XVV<m, tsimm5>;
    defm VVV : VPseudoVC_XVV<m, m.vrclass>;
  }
  foreach f = FPList in {
    foreach m = f.MxList in {
      let AltFmtType = IS_NOT_ALTFMT in {
        defm f.FX # "V" : VPseudoVC_XV<m, f.fprclass, payload1>;
        defm f.FX # "VV" : VPseudoVC_XVV<m, f.fprclass, payload1>;
      }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 481-494: Bulk record instantiation m / 批量记录实例化 m
```tablegen
    }
  }
  foreach m = MxListW in {
    defm XVW : VPseudoVC_XVW<m, GPR>;
    defm IVW : VPseudoVC_XVW<m, tsimm5>;
    defm VVW : VPseudoVC_XVW<m, m.vrclass>;
  }
  foreach f = FPListW in {
    foreach m = f.MxList in
      let AltFmtType = IS_NOT_ALTFMT in
        defm f.FX # "VW" : VPseudoVC_XVW<m, f.fprclass, payload1>;
  }
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 495-508: Bulk record instantiation SF_VQMACCU_2x8x2 / 批量记录实例化 SF_VQMACCU_2x8x2
```tablegen
let Predicates = [HasVendorXSfvqmaccdod] in {
  defm SF_VQMACCU_2x8x2  : VPseudoSiFiveVQMACCDOD;
  defm SF_VQMACC_2x8x2   : VPseudoSiFiveVQMACCDOD;
  defm SF_VQMACCUS_2x8x2 : VPseudoSiFiveVQMACCDOD;
  defm SF_VQMACCSU_2x8x2 : VPseudoSiFiveVQMACCDOD;
}

let Predicates = [HasVendorXSfvqmaccqoq] in {
  defm SF_VQMACCU_4x8x4  : VPseudoSiFiveVQMACCQOQ;
  defm SF_VQMACC_4x8x4   : VPseudoSiFiveVQMACCQOQ;
  defm SF_VQMACCUS_4x8x4 : VPseudoSiFiveVQMACCQOQ;
  defm SF_VQMACCSU_4x8x4 : VPseudoSiFiveVQMACCQOQ;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 509-522: TableGen class VFExpSchedSEWSet<string / TableGen 类 VFExpSchedSEWSet<string
```tablegen
let Predicates = [HasVendorXSfvfwmaccqqq] in {
  defm SF_VFWMACC_4x4x4 : VPseudoSiFiveVFWMACC;
}

let Predicates = [HasVendorXSfvfnrclipxfqf], AltFmtType = IS_NOT_ALTFMT in {
  defm SF_VFNRCLIP_XU_F_QF : VPseudoSiFiveVFNRCLIP;
  defm SF_VFNRCLIP_X_F_QF : VPseudoSiFiveVFNRCLIP;
}

class VFExpSchedSEWSet<string mx, bit IsBF16, bit IsApprox> {
  defvar BaseSet = SchedSEWSet<mx, isF=1>.val;
  list<int> val = !if(IsBF16, !listremove(BaseSet, [32, 64]),
                      !if(IsApprox, BaseSet, !listremove(BaseSet, [64])));
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 523-539: Reusable TableGen multiclass VPseudoVFExp_V / 可复用的 TableGen 多类 VPseudoVFExp_V
```tablegen
multiclass VPseudoVFExp_V<bit IsBF16 = false, bit IsApprox = false> {
  defvar SchedSuffix = !if(IsApprox, "VFExpa", "VFExp");

  foreach m = MxListF in {
    defvar mx = m.MX;
    foreach e = VFExpSchedSEWSet<mx, IsBF16, IsApprox>.val in {
      let VLMul = m.value in {
        def "_V_" # mx # "_E" # e
            : VPseudoUnaryNoMask<m.vrclass, m.vrclass>,
              SchedUnary<"WriteSF_" # SchedSuffix, "ReadSF_" # SchedSuffix,
                         mx, e, forcePassthruRead=true>;
        def "_V_" # mx # "_E" # e # "_MASK"
            : VPseudoUnaryMask<m.vrclass, m.vrclass>,
              RISCVMaskedPseudo<MaskIdx = 2>,
              SchedUnary<"WriteSF_" # SchedSuffix, "ReadSF_" # SchedSuffix,
                         mx, e, forcePassthruRead=true>;
      }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 540-553: Bulk record instantiation PseudoSF_VFEXP_ALT / 批量记录实例化 PseudoSF_VFEXP_ALT
```tablegen
    }
  }
}

let Predicates = [HasVendorXSfvfbfexp16e], hasSideEffects = 0 in {
  let AltFmtType = IS_ALTFMT in {
    defm PseudoSF_VFEXP_ALT : VPseudoVFExp_V<IsBF16=true>;
  }
}

let Predicates = [HasVendorXSfvfexpAnyFloat], hasSideEffects = 0 in {
  let AltFmtType = IS_NOT_ALTFMT in {
    defm PseudoSF_VFEXP : VPseudoVFExp_V;
  }
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 554-571: Bulk record instantiation PseudoSF_VFEXPA / 批量记录实例化 PseudoSF_VFEXPA
```tablegen
}

let Predicates = [HasVendorXSfvfexpa], AltFmtType = IS_NOT_ALTFMT in {
  defm PseudoSF_VFEXPA : VPseudoVFExp_V<IsApprox=true>;
}

// SDNode
def SDT_SF_VC_V_X : SDTypeProfile<1, 4, [SDTCisVec<0>,
                                         SDTCisVT<1, XLenVT>,
                                         SDTCisSameAs<1, 2>,
                                         SDTCisSameAs<1, 3>,
                                         SDTCisSameAs<1, 4>]>;

def SDT_SF_VC_XV : SDTypeProfile<0, 5, [SDTCisSameAs<0, 1>,
                                        SDTCisVec<2>,
                                        SDTCisSameAs<0, 4>,
                                        SDTCisVT<0, XLenVT>]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 572-587: TableGen record SDT_SF_VC_V_XV / TableGen 记录 SDT_SF_VC_V_XV
```tablegen
def SDT_SF_VC_V_XV : SDTypeProfile<1, 4, [SDTCisVec<0>,
                                          SDTCisVT<1, XLenVT>,
                                          SDTCisSameAs<0, 2>,
                                          SDTCisSameAs<1, 4>]>;

def SDT_SF_VC_XVV : SDTypeProfile<0, 5, [SDTCisVT<0, XLenVT>,
                                         SDTCisVec<1>,
                                         SDTCisSameAs<1, 2>,
                                         SDTCisSameAs<0, 4>]>;

def SDT_SF_VC_V_XVV : SDTypeProfile<1, 5, [SDTCisVec<0>,
                                           SDTCisVT<1, XLenVT>,
                                           SDTCisSameAs<0, 2>,
                                           SDTCisSameAs<0, 3>,
                                           SDTCisSameAs<1, 5>]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 588-615: TableGen record SDT_SF_VC_XVW / TableGen 记录 SDT_SF_VC_XVW
```tablegen
def SDT_SF_VC_XVW : SDTypeProfile<0, 5, [SDTCisVT<0, XLenVT>,
                                         SDTCisVec<1>, SDTCisVec<2>,
                                         SDTCisSameAs<0, 4>]>;

def SDT_SF_VC_V_XVW : SDTypeProfile<1, 5, [SDTCisVec<0>,
                                           SDTCisVT<1, XLenVT>,
                                           SDTCisSameAs<0, 2>,
                                           SDTCisVec<3>,
                                           SDTCisSameAs<1, 5>]>;

def sf_vc_v_x_se : RVSDNode<"SF_VC_V_X_SE", SDT_SF_VC_V_X, [SDNPHasChain]>;
def sf_vc_v_i_se : RVSDNode<"SF_VC_V_I_SE", SDT_SF_VC_V_X, [SDNPHasChain]>;
def sf_vc_vv_se : RVSDNode<"SF_VC_VV_SE", SDT_SF_VC_XV, [SDNPHasChain]>;
def sf_vc_xv_se : RVSDNode<"SF_VC_XV_SE", SDT_SF_VC_XV, [SDNPHasChain]>;
def sf_vc_iv_se : RVSDNode<"SF_VC_IV_SE", SDT_SF_VC_XV, [SDNPHasChain]>;
def sf_vc_fv_se : RVSDNode<"SF_VC_FV_SE", SDT_SF_VC_XV, [SDNPHasChain]>;
def sf_vc_v_vv_se : RVSDNode<"SF_VC_V_VV_SE", SDT_SF_VC_V_XV, [SDNPHasChain]>;
def sf_vc_v_xv_se : RVSDNode<"SF_VC_V_XV_SE", SDT_SF_VC_V_XV, [SDNPHasChain]>;
def sf_vc_v_iv_se : RVSDNode<"SF_VC_V_IV_SE", SDT_SF_VC_V_XV, [SDNPHasChain]>;
def sf_vc_v_fv_se : RVSDNode<"SF_VC_V_FV_SE", SDT_SF_VC_V_XV, [SDNPHasChain]>;
def sf_vc_vvv_se : RVSDNode<"SF_VC_VVV_SE", SDT_SF_VC_XVV, [SDNPHasChain]>;
def sf_vc_xvv_se : RVSDNode<"SF_VC_XVV_SE", SDT_SF_VC_XVV, [SDNPHasChain]>;
def sf_vc_ivv_se : RVSDNode<"SF_VC_IVV_SE", SDT_SF_VC_XVV, [SDNPHasChain]>;
def sf_vc_fvv_se : RVSDNode<"SF_VC_FVV_SE", SDT_SF_VC_XVV, [SDNPHasChain]>;
def sf_vc_v_vvv_se : RVSDNode<"SF_VC_V_VVV_SE", SDT_SF_VC_V_XVV, [SDNPHasChain]>;
def sf_vc_v_xvv_se : RVSDNode<"SF_VC_V_XVV_SE", SDT_SF_VC_V_XVV, [SDNPHasChain]>;
def sf_vc_v_ivv_se : RVSDNode<"SF_VC_V_IVV_SE", SDT_SF_VC_V_XVV, [SDNPHasChain]>;
def sf_vc_v_fvv_se : RVSDNode<"SF_VC_V_FVV_SE", SDT_SF_VC_V_XVV, [SDNPHasChain]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 616-643: TableGen class VPatVC_OP4_ISD<SDPatternOperator / TableGen 类 VPatVC_OP4_ISD<SDPatternOperator
```tablegen
def sf_vc_vvw_se : RVSDNode<"SF_VC_VVW_SE", SDT_SF_VC_XVW, [SDNPHasChain]>;
def sf_vc_xvw_se : RVSDNode<"SF_VC_XVW_SE", SDT_SF_VC_XVW, [SDNPHasChain]>;
def sf_vc_ivw_se : RVSDNode<"SF_VC_IVW_SE", SDT_SF_VC_XVW, [SDNPHasChain]>;
def sf_vc_fvw_se : RVSDNode<"SF_VC_FVW_SE", SDT_SF_VC_XVW, [SDNPHasChain]>;
def sf_vc_v_vvw_se : RVSDNode<"SF_VC_V_VVW_SE", SDT_SF_VC_V_XVW, [SDNPHasChain]>;
def sf_vc_v_xvw_se : RVSDNode<"SF_VC_V_XVW_SE", SDT_SF_VC_V_XVW, [SDNPHasChain]>;
def sf_vc_v_ivw_se : RVSDNode<"SF_VC_V_IVW_SE", SDT_SF_VC_V_XVW, [SDNPHasChain]>;
def sf_vc_v_fvw_se : RVSDNode<"SF_VC_V_FVW_SE", SDT_SF_VC_V_XVW, [SDNPHasChain]>;

class VPatVC_OP4_ISD<SDPatternOperator op,
                     string inst,
                     ValueType op2_type,
                     ValueType op3_type,
                     ValueType op4_type,
                     int sew,
                     DAGOperand op2_kind,
                     DAGOperand op3_kind,
                     DAGOperand op4_kind,
                     Operand op1_kind = payload2> :
  Pat<(op
       (XLenVT   op1_kind:$op1),
       (op2_type op2_kind:$op2),
       (op3_type op3_kind:$op3),
       (op4_type op4_kind:$op4),
       VLOpFrag),
      (!cast<Instruction>(inst)
       (XLenVT   op1_kind:$op1),
       (op2_type op2_kind:$op2),
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 644-671: TableGen class VPatVC_V_OP4_ISD<SDPatternOperator / TableGen 类 VPatVC_V_OP4_ISD<SDPatternOperator
```tablegen
       (op3_type op3_kind:$op3),
       (op4_type op4_kind:$op4),
       GPR:$vl, sew)>;

class VPatVC_V_OP4_ISD<SDPatternOperator op,
                       string inst,
                       ValueType result_type,
                       ValueType op2_type,
                       ValueType op3_type,
                       ValueType op4_type,
                       int sew,
                       DAGOperand op2_kind,
                       DAGOperand op3_kind,
                       DAGOperand op4_kind,
                       Operand op1_kind = payload2> :
  Pat<(result_type (op
                    (XLenVT   op1_kind:$op1),
                    (op2_type op2_kind:$op2),
                    (op3_type op3_kind:$op3),
                    (op4_type op4_kind:$op4),
                    VLOpFrag)),
                   (!cast<Instruction>(inst)
                    (XLenVT   op1_kind:$op1),
                    (op2_type op2_kind:$op2),
                    (op3_type op3_kind:$op3),
                    (op4_type op4_kind:$op4),
                    GPR:$vl, sew)>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 672-692: TableGen class VPatVC_V_OP3_ISD<SDPatternOperator / TableGen 类 VPatVC_V_OP3_ISD<SDPatternOperator
```tablegen

class VPatVC_V_OP3_ISD<SDPatternOperator op,
                       string inst,
                       ValueType result_type,
                       ValueType op2_type,
                       ValueType op3_type,
                       int sew,
                       DAGOperand op2_kind,
                       DAGOperand op3_kind,
                       Operand op1_kind = payload2> :
  Pat<(result_type (op
                    (XLenVT   op1_kind:$op1),
                    (op2_type op2_kind:$op2),
                    (op3_type op3_kind:$op3),
                    VLOpFrag)),
                   (!cast<Instruction>(inst)
                    (XLenVT   op1_kind:$op1),
                    (op2_type op2_kind:$op2),
                    (op3_type op3_kind:$op3),
                    GPR:$vl, sew)>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 693-715: TableGen class VPatVC_OP4<string / TableGen 类 VPatVC_OP4<string
```tablegen
class VPatVC_OP4<string intrinsic_name,
                 string inst,
                 ValueType op2_type,
                 ValueType op3_type,
                 ValueType op4_type,
                 int sew,
                 DAGOperand op2_kind,
                 DAGOperand op3_kind,
                 DAGOperand op4_kind,
                 Operand op1_kind = payload2> :
  Pat<(!cast<Intrinsic>(intrinsic_name)
       (XLenVT   op1_kind:$op1),
       (op2_type op2_kind:$op2),
       (op3_type op3_kind:$op3),
       (op4_type op4_kind:$op4),
       VLOpFrag),
      (!cast<Instruction>(inst)
       (XLenVT   op1_kind:$op1),
       (op2_type op2_kind:$op2),
       (op3_type op3_kind:$op3),
       (op4_type op4_kind:$op4),
       GPR:$vl, sew)>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 716-739: TableGen class VPatVC_V_OP4<string / TableGen 类 VPatVC_V_OP4<string
```tablegen
class VPatVC_V_OP4<string intrinsic_name,
                   string inst,
                   ValueType result_type,
                   ValueType op2_type,
                   ValueType op3_type,
                   ValueType op4_type,
                   int sew,
                   DAGOperand op2_kind,
                   DAGOperand op3_kind,
                   DAGOperand op4_kind,
                   Operand op1_kind = payload2> :
  Pat<(result_type (!cast<Intrinsic>(intrinsic_name)
                    (XLenVT   op1_kind:$op1),
                    (op2_type op2_kind:$op2),
                    (op3_type op3_kind:$op3),
                    (op4_type op4_kind:$op4),
                    VLOpFrag)),
                   (!cast<Instruction>(inst)
                    (XLenVT   op1_kind:$op1),
                    (op2_type op2_kind:$op2),
                    (op3_type op3_kind:$op3),
                    (op4_type op4_kind:$op4),
                    GPR:$vl, sew)>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 740-759: TableGen class VPatVC_V_OP3<string / TableGen 类 VPatVC_V_OP3<string
```tablegen
class VPatVC_V_OP3<string intrinsic_name,
                   string inst,
                   ValueType result_type,
                   ValueType op2_type,
                   ValueType op3_type,
                   int sew,
                   DAGOperand op2_kind,
                   DAGOperand op3_kind,
                   Operand op1_kind = payload2> :
  Pat<(result_type (!cast<Intrinsic>(intrinsic_name)
                    (XLenVT   op1_kind:$op1),
                    (op2_type op2_kind:$op2),
                    (op3_type op3_kind:$op3),
                    VLOpFrag)),
                   (!cast<Instruction>(inst)
                    (XLenVT   op1_kind:$op1),
                    (op2_type op2_kind:$op2),
                    (op3_type op3_kind:$op3),
                    GPR:$vl, sew)>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 760-787: Reusable TableGen multiclass VPatVC_X / 可复用的 TableGen 多类 VPatVC_X
```tablegen
multiclass VPatVC_X<string intrinsic_suffix, string instruction_suffix,
                    VTypeInfo vti, ValueType type, DAGOperand kind> {
  def : VPatVC_V_OP3_ISD<!cast<SDPatternOperator>("sf_vc_v_" # intrinsic_suffix # "_se"),
                         "PseudoSF_VC_V_" # instruction_suffix # "_SE_" # vti.LMul.MX,
                         vti.Vector, XLenVT, type, vti.Log2SEW,
                         payload5, kind>;
  def : VPatVC_V_OP3<"int_riscv_sf_vc_v_" # intrinsic_suffix,
                     "PseudoSF_VC_V_" # instruction_suffix # "_" # vti.LMul.MX,
                     vti.Vector, XLenVT, type, vti.Log2SEW,
                     payload5, kind>;
}

multiclass VPatVC_XV<string intrinsic_suffix, string instruction_suffix,
                     VTypeInfo vti, ValueType type, DAGOperand kind,
                     Operand op1_kind = payload2> {
  def : VPatVC_OP4_ISD<!cast<SDPatternOperator>("sf_vc_" # intrinsic_suffix # "_se"),
                   "PseudoSF_VC_" # instruction_suffix # "_SE_" # vti.LMul.MX,
                   XLenVT, vti.Vector, type, vti.Log2SEW,
                   payload5, vti.RegClass, kind, op1_kind>;
  def : VPatVC_V_OP3_ISD<!cast<SDPatternOperator>("sf_vc_v_" # intrinsic_suffix # "_se"),
                         "PseudoSF_VC_V_" # instruction_suffix # "_SE_" # vti.LMul.MX,
                         vti.Vector, vti.Vector, type, vti.Log2SEW,
                         vti.RegClass, kind, op1_kind>;
  def : VPatVC_V_OP3<"int_riscv_sf_vc_v_" # intrinsic_suffix,
                     "PseudoSF_VC_V_" # instruction_suffix # "_" # vti.LMul.MX,
                     vti.Vector, vti.Vector, type, vti.Log2SEW,
                     vti.RegClass, kind, op1_kind>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 788-804: Reusable TableGen multiclass VPatVC_XVV / 可复用的 TableGen 多类 VPatVC_XVV
```tablegen

multiclass VPatVC_XVV<string intrinsic_suffix, string instruction_suffix,
                      VTypeInfo wti, VTypeInfo vti, ValueType type, DAGOperand kind,
                      Operand op1_kind = payload2> {
  def : VPatVC_OP4_ISD<!cast<SDPatternOperator>("sf_vc_" # intrinsic_suffix # "_se"),
                   "PseudoSF_VC_" # instruction_suffix # "_SE_" # vti.LMul.MX,
                   wti.Vector, vti.Vector, type, vti.Log2SEW,
                   wti.RegClass, vti.RegClass, kind, op1_kind>;
  def : VPatVC_V_OP4_ISD<!cast<SDPatternOperator>("sf_vc_v_" # intrinsic_suffix # "_se"),
                     "PseudoSF_VC_V_" # instruction_suffix # "_SE_" # vti.LMul.MX,
                     wti.Vector, wti.Vector, vti.Vector, type, vti.Log2SEW,
                     wti.RegClass, vti.RegClass, kind, op1_kind>;
  def : VPatVC_V_OP4<"int_riscv_sf_vc_v_" # intrinsic_suffix,
                     "PseudoSF_VC_V_" # instruction_suffix # "_" # vti.LMul.MX,
                     wti.Vector, wti.Vector, vti.Vector, type, vti.Log2SEW,
                     wti.RegClass, vti.RegClass, kind, op1_kind>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 805-818: TableGen class GetFTypeInfo<int / TableGen 类 GetFTypeInfo<int
```tablegen

class GetFTypeInfo<int Sew> {
  ValueType Scalar = !cond(!eq(Sew, 16) : f16,
                           !eq(Sew, 32) : f32,
                           !eq(Sew, 64) : f64);
  RegisterClass ScalarRegClass = !cond(!eq(Sew, 16) : FPR16,
                                       !eq(Sew, 32) : FPR32,
                                       !eq(Sew, 64) : FPR64);

  string ScalarSuffix = !cond(!eq(Scalar, f16) : "FPR16",
                              !eq(Scalar, f32) : "FPR32",
                              !eq(Scalar, f64) : "FPR64");
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 819-832: Reusable TableGen multiclass VPatVMACC / 可复用的 TableGen 多类 VPatVMACC
```tablegen
multiclass VPatVMACC<string intrinsic, string instruction, string kind,
                     list<VTypeInfoToWide> info_pairs, ValueType vec_m1> {
  foreach pair = info_pairs in {
    defvar VdInfo = pair.Wti;
    defvar Vs2Info = pair.Vti;
    let Predicates = [HasVInstructions] in
    def : VPatTernaryNoMaskWithPolicy<"int_riscv_sf_" # intrinsic,
                                      "Pseudo" # instruction, kind, VdInfo.Vector,
                                      vec_m1, Vs2Info.Vector,
                                      Vs2Info.Log2SEW, Vs2Info.LMul,
                                      VdInfo.RegClass, VR, Vs2Info.RegClass>;
  }
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 833-846: TableGen record definition / TableGen 记录定义
```tablegen
defset list<VTypeInfoToWide> VQMACCDODInfoPairs = {
  def : VTypeInfoToWide<VI8M1, VI32M1>;
  def : VTypeInfoToWide<VI8M2, VI32M2>;
  def : VTypeInfoToWide<VI8M4, VI32M4>;
  def : VTypeInfoToWide<VI8M8, VI32M8>;
}

defset list<VTypeInfoToWide> VQMACCQOQInfoPairs = {
  def : VTypeInfoToWide<VI8MF2, VI32M1>;
  def : VTypeInfoToWide<VI8M1, VI32M2>;
  def : VTypeInfoToWide<VI8M2, VI32M4>;
  def : VTypeInfoToWide<VI8M4, VI32M8>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 847-863: Reusable TableGen multiclass VPatVQMACCDOD / 可复用的 TableGen 多类 VPatVQMACCDOD
```tablegen
multiclass VPatVQMACCDOD<string intrinsic, string instruction, string kind>
    : VPatVMACC<intrinsic, instruction, kind, VQMACCDODInfoPairs, vint8m1_t>;

multiclass VPatVQMACCQOQ<string intrinsic, string instruction, string kind>
    : VPatVMACC<intrinsic, instruction, kind, VQMACCQOQInfoPairs, vint8m1_t>;

multiclass VPatVFWMACC<string intrinsic, string instruction, string kind>
    : VPatVMACC<intrinsic, instruction, kind, AllWidenableBF16ToFloatVectors,
                vbfloat16m1_t>;

defset list<VTypeInfoToWide> VFNRCLIPInfoPairs = {
  def : VTypeInfoToWide<VI8MF8, VF32MF2>;
  def : VTypeInfoToWide<VI8MF4, VF32M1>;
  def : VTypeInfoToWide<VI8MF2, VF32M2>;
  def : VTypeInfoToWide<VI8M1,  VF32M4>;
  def : VTypeInfoToWide<VI8M2,  VF32M8>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 864-887: Reusable TableGen multiclass VPatVFNRCLIP / 可复用的 TableGen 多类 VPatVFNRCLIP
```tablegen

multiclass VPatVFNRCLIP<string intrinsic, string instruction> {
  foreach pair = VFNRCLIPInfoPairs in {
    defvar Vti = pair.Vti;
    defvar Wti = pair.Wti;
    defm : VPatBinaryRoundingMode<"int_riscv_sf_" # intrinsic,
                                  "Pseudo" # instruction # "_" # Vti.LMul.MX,
                                  Vti.Vector, Wti.Vector, Wti.Scalar, Vti.Mask,
                                  Vti.Log2SEW, Vti.RegClass,
                                  Wti.RegClass, Wti.ScalarRegClass>;
  }
}

let Predicates = [HasVendorXSfvcp] in {
  foreach vti = AllIntegerVectors in {
    defm : VPatVC_X<"x", "X", vti, XLenVT, GPR>;
    defm : VPatVC_X<"i", "I", vti, XLenVT, tsimm5>;
    defm : VPatVC_XV<"xv", "XV", vti, XLenVT, GPR>;
    defm : VPatVC_XV<"iv", "IV", vti, XLenVT, tsimm5>;
    defm : VPatVC_XV<"vv", "VV", vti, vti.Vector, vti.RegClass>;
    defm : VPatVC_XVV<"xvv", "XVV", vti, vti, XLenVT, GPR>;
    defm : VPatVC_XVV<"ivv", "IVV", vti, vti, XLenVT, tsimm5>;
    defm : VPatVC_XVV<"vvv", "VVV", vti, vti, vti.Vector, vti.RegClass>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 888-903: Bulk record instantiation VtiToWti / 批量记录实例化 VtiToWti
```tablegen
    if !ne(vti.SEW, 8) then {
      defvar finfo = GetFTypeInfo<vti.SEW>;
      defm : VPatVC_XV<"fv", finfo.ScalarSuffix # "V", vti, finfo.Scalar,
                       finfo.ScalarRegClass, payload1>;
      defm : VPatVC_XVV<"fvv", finfo.ScalarSuffix # "VV", vti, vti, finfo.Scalar,
                        finfo.ScalarRegClass, payload1>;
    }
  }
  foreach VtiToWti = AllWidenableIntVectors in {
    defvar vti = VtiToWti.Vti;
    defvar wti = VtiToWti.Wti;
    defvar iinfo = GetIntVTypeInfo<vti>.Vti;
    defm : VPatVC_XVV<"xvw", "XVW", wti, vti, iinfo.Scalar, iinfo.ScalarRegClass>;
    defm : VPatVC_XVV<"ivw", "IVW", wti, vti, XLenVT, tsimm5>;
    defm : VPatVC_XVV<"vvw", "VVW", wti, vti, vti.Vector, vti.RegClass>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 904-917: Bulk record instantiation / 批量记录实例化
```tablegen
    if !ne(vti.SEW, 8) then {
      defvar finfo = GetFTypeInfo<vti.SEW>;
      defm : VPatVC_XVV<"fvw", finfo.ScalarSuffix # "VW", wti, vti, finfo.Scalar,
                        finfo.ScalarRegClass, payload1>;
    }
  }
}

let Predicates = [HasVendorXSfvqmaccdod] in {
  defm : VPatVQMACCDOD<"vqmaccu_2x8x2", "SF_VQMACCU", "2x8x2">;
  defm : VPatVQMACCDOD<"vqmacc_2x8x2", "SF_VQMACC", "2x8x2">;
  defm : VPatVQMACCDOD<"vqmaccus_2x8x2", "SF_VQMACCUS", "2x8x2">;
  defm : VPatVQMACCDOD<"vqmaccsu_2x8x2", "SF_VQMACCSU", "2x8x2">;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 918-933: Bulk record instantiation / 批量记录实例化
```tablegen

let Predicates = [HasVendorXSfvqmaccqoq] in {
  defm : VPatVQMACCQOQ<"vqmaccu_4x8x4", "SF_VQMACCU", "4x8x4">;
  defm : VPatVQMACCQOQ<"vqmacc_4x8x4", "SF_VQMACC", "4x8x4">;
  defm : VPatVQMACCQOQ<"vqmaccus_4x8x4", "SF_VQMACCUS", "4x8x4">;
  defm : VPatVQMACCQOQ<"vqmaccsu_4x8x4", "SF_VQMACCSU", "4x8x4">;
}

let Predicates = [HasVendorXSfvfwmaccqqq] in {
  defm : VPatVFWMACC<"vfwmacc_4x4x4", "SF_VFWMACC", "4x4x4">;
}

let Predicates = [HasVendorXSfvfnrclipxfqf] in {
  defm : VPatVFNRCLIP<"vfnrclip_xu_f_qf", "SF_VFNRCLIP_XU_F_QF">;
  defm : VPatVFNRCLIP<"vfnrclip_x_f_qf", "SF_VFNRCLIP_X_F_QF">;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 934-953: TableGen record SF_CDISCARD_D_L1 / TableGen 记录 SF_CDISCARD_D_L1
```tablegen

let Predicates = [HasVendorXSiFivecdiscarddlone] in {
  let hasNoSchedulingInfo = 1, hasSideEffects = 1, mayLoad = 0, mayStore = 0,
      DecoderNamespace = "XSfsystem" in
  def SF_CDISCARD_D_L1
      : RVInstIUnary<0b111111000010, 0b000, OPC_SYSTEM, (outs), (ins GPR:$rs1),
                     "sf.cdiscard.d.l1", "$rs1">, Sched<[]> {
    let rd = 0;
  }
  def : InstAlias<"sf.cdiscard.d.l1", (SF_CDISCARD_D_L1 X0)>;
} // Predicates = [HasVendorXSifivecdiscarddlone]

let Predicates = [HasVendorXSiFivecflushdlone] in {
  let hasNoSchedulingInfo = 1, hasSideEffects = 1, mayLoad = 0, mayStore = 0,
      DecoderNamespace = "XSfsystem" in
  def SF_CFLUSH_D_L1
      : RVInstIUnary<0b111111000000, 0b000, OPC_SYSTEM, (outs), (ins GPR:$rs1),
                     "sf.cflush.d.l1", "$rs1">, Sched<[]> {
    let rd = 0;
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 954-971: Bulk record instantiation SF_CEASE / 批量记录实例化 SF_CEASE
```tablegen
  def : InstAlias<"sf.cflush.d.l1", (SF_CFLUSH_D_L1 X0)>;
} // Predicates = [HasVendorXSifivecflushdlone]

let Predicates = [HasVendorXSfcease] in {
  let hasNoSchedulingInfo = 1, hasSideEffects = 1, mayLoad = 0, mayStore = 0,
      DecoderNamespace = "XSfcease" in
  def SF_CEASE : Priv<"sf.cease", 0b0011000>, Sched<[]> {
    let rd = 0b00000;
    let rs1 = 0b00000;
    let rs2 = 0b00101;
  }
}

let Predicates = [HasVendorXSfvfbfexp16e] in {
  defm : VPatUnaryV_V<"int_riscv_sf_vfexp", "PseudoSF_VFEXP_ALT",
                      AllBF16Vectors,
                      isSEWAware=1>;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 972-987: Bulk record instantiation / 批量记录实例化
```tablegen

let Predicates = [HasVendorXSfvfexp16e] in {
  defm : VPatUnaryV_V<"int_riscv_sf_vfexp", "PseudoSF_VFEXP",
                      [VF16MF4, VF16MF2, VF16M1, VF16M2, VF16M4, VF16M8],
                      isSEWAware=1>;
}

let Predicates = [HasVendorXSfvfexp32e] in {
  defm : VPatUnaryV_V<"int_riscv_sf_vfexp", "PseudoSF_VFEXP",
                      [VF32MF2, VF32M1, VF32M2, VF32M4, VF32M8], isSEWAware=1>;
}

let Predicates = [HasVendorXSfvfexpa] in {
  defm : VPatUnaryV_V<"int_riscv_sf_vfexpa", "PseudoSF_VFEXPA",
                      [VF32MF2, VF32M1, VF32M2, VF32M4, VF32M8], isSEWAware=1>;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 988-998: Bulk record instantiation / 批量记录实例化
```tablegen

let Predicates = [HasVendorXSfvfexpa, HasVInstructionsF16] in {
  defm : VPatUnaryV_V<"int_riscv_sf_vfexpa", "PseudoSF_VFEXPA",
                      [VF16MF4, VF16MF2, VF16M1, VF16M2, VF16M4, VF16M8],
                      isSEWAware=1>;
}

let Predicates = [HasVendorXSfvfexpa64e] in {
  defm : VPatUnaryV_V<"int_riscv_sf_vfexpa", "PseudoSF_VFEXPA",
                      [VF64M1, VF64M2, VF64M4, VF64M8], isSEWAware=1>;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
