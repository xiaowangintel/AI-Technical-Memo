# RISCVInstrFormats.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrFormats.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for instruction format classes, bit layouts, and operand encodings for RISC-V. / 使用 TableGen 定义RISC-V 的指令格式类、位布局与操作数编码。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-25: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrFormats.td - RISC-V Instruction Formats --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
//
//  These instruction format definitions are structured to match the
//  description in the RISC-V User-Level ISA specification as closely as
//  possible. For instance, the specification describes instructions with the
//  MSB (31st bit) on the left and the LSB (0th bit) on the right. This is
//  reflected in the order of parameters to each instruction class.
//
//  One area of divergence is in the description of immediates. The
//  specification describes immediate encoding in terms of bit-slicing
//  operations on the logical value represented. The immediate argument to
//  these instruction formats instead represents the bit sequence that will be
//  inserted into the instruction. e.g. although JAL's immediate is logically
//  a 21-bit value (where the LSB is always zero), we describe it as an imm20
//  to match how it is encoded.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 26-53: TableGen class InstFormat<bits<5> / TableGen 类 InstFormat<bits<5>
```tablegen

// Format specifies the encoding used by the instruction. This is used by
// RISCVMCCodeEmitter to determine which form of fixup to use. These
// definitions must be kept in-sync with RISCVBaseInfo.h.
class InstFormat<bits<5> val> {
  bits<5> Value = val;
}
def InstFormatPseudo        : InstFormat<0>;
def InstFormatR             : InstFormat<1>;
def InstFormatR4            : InstFormat<2>;
def InstFormatI             : InstFormat<3>;
def InstFormatS             : InstFormat<4>;
def InstFormatB             : InstFormat<5>;
def InstFormatU             : InstFormat<6>;
def InstFormatJ             : InstFormat<7>;
def InstFormatCR            : InstFormat<8>;
def InstFormatCI            : InstFormat<9>;
def InstFormatCSS           : InstFormat<10>;
def InstFormatCIW           : InstFormat<11>;
def InstFormatCL            : InstFormat<12>;
def InstFormatCS            : InstFormat<13>;
def InstFormatCA            : InstFormat<14>;
def InstFormatCB            : InstFormat<15>;
def InstFormatCJ            : InstFormat<16>;
def InstFormatCU            : InstFormat<17>;
def InstFormatCLB           : InstFormat<18>;
def InstFormatCLH           : InstFormat<19>;
def InstFormatCSB           : InstFormat<20>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 54-72: TableGen class RISCVVConstraint<bit / TableGen 类 RISCVVConstraint<bit
```tablegen
def InstFormatCSH           : InstFormat<21>;
def InstFormatQC_EAI        : InstFormat<22>;
def InstFormatQC_EI         : InstFormat<23>;
def InstFormatQC_EB         : InstFormat<24>;
def InstFormatQC_EJ         : InstFormat<25>;
def InstFormatQC_ES         : InstFormat<26>;
def InstFormatNDS_BRANCH_10 : InstFormat<27>;
def InstFormatOther         : InstFormat<31>;


// Indicates whether Vs1/Vs2 can be the same as the Vd operand. This is used
// by the assembler to provide some checking of the RVV overlap rules. We can't
// check all overlap rules. Narrowing instructions allow overlap in the first
// part of the register group, but not the later parts. We are not able to check
// this in the assembler as we do not know how big the register group is since
// that is controlled dynamically with vtype.
class RISCVVConstraint<bit VS2 = 0, bit VS1 = 0> {
  bits<2> Value = {VS1, VS2};
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 73-100: TableGen record NoConstraint / TableGen 记录 NoConstraint
```tablegen
def NoConstraint  : RISCVVConstraint<>;
def VS2Constraint : RISCVVConstraint<VS2 = 1>;
def VS1Constraint : RISCVVConstraint<VS1 = 1>;

// Illegal instructions:
//
// * Widening: The destination EEW is greater than the source EEW, the source
// EMUL is at least 1. The destination vector register group cannot overlap
// with the source vector register groups besides the highest-numbered part of
// the destination register group.
//
// * Narrowing: The destination EEW is smaller than the source EEW. The
// destination vector register group cannot overlap with the source vector
// register groups besides the lowest-numbered part of the source register
// group.
//
// * vmsbf.m/vmsif.m/vmsof.m: The destination register cannot overlap the
// source register.
//
// * viota: The destination register cannot overlap the source register.
//
// * v[f]slide[1]up: The destination vector register group for vslideup cannot
// overlap the source vector register group.
//
// * vrgather: The destination vector register group cannot overlap with the
// source vector register groups.
//
// * vcompress: The destination vector register group cannot overlap the
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 101-116: TableGen class RISCVOpcode<string / TableGen 类 RISCVOpcode<string
```tablegen
// source vector register group.
def WidenV         : RISCVVConstraint<VS2 = 1, VS1 = 1>;
def WidenW         : RISCVVConstraint<VS1 = 1>;
def WidenCvt       : RISCVVConstraint<VS2 = 1>;
def Iota           : RISCVVConstraint<VS2 = 1>;
def SlideUp        : RISCVVConstraint<VS2 = 1>;
def Vrgather       : RISCVVConstraint<VS2 = 1, VS1 = 1>;
def Vcompress      : RISCVVConstraint<VS2 = 1, VS1 = 1>;
def Sha2Constraint : RISCVVConstraint<VS2 = 1, VS1 = 1>;

// The following opcode names match those given in Table 19.1 in the
// RISC-V User-level ISA specification ("RISC-V base opcode map").
class RISCVOpcode<string name, bits<7> val> {
  string Name = name;
  bits<7> Value = val;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 117-144: TableGen record RISCVOpcodesList / TableGen 记录 RISCVOpcodesList
```tablegen
def RISCVOpcodesList : GenericTable {
  let FilterClass = "RISCVOpcode";
  let Fields = [
    "Name", "Value"
  ];
  let PrimaryKey = [ "Value" ];
  let PrimaryKeyName = "lookupRISCVOpcodeByValue";
}
def lookupRISCVOpcodeByName : SearchIndex {
  let Table = RISCVOpcodesList;
  let Key = [ "Name" ];
}
def OPC_LOAD      : RISCVOpcode<"LOAD",      0b0000011>;
def OPC_LOAD_FP   : RISCVOpcode<"LOAD_FP",   0b0000111>;
def OPC_CUSTOM_0  : RISCVOpcode<"CUSTOM_0",  0b0001011>;
def OPC_MISC_MEM  : RISCVOpcode<"MISC_MEM",  0b0001111>;
def OPC_OP_IMM    : RISCVOpcode<"OP_IMM",    0b0010011>;
def OPC_AUIPC     : RISCVOpcode<"AUIPC",     0b0010111>;
def OPC_OP_IMM_32 : RISCVOpcode<"OP_IMM_32", 0b0011011>;
def OPC_0011111   : RISCVOpcode<"",          0b0011111>;
def OPC_STORE     : RISCVOpcode<"STORE",     0b0100011>;
def OPC_STORE_FP  : RISCVOpcode<"STORE_FP",  0b0100111>;
def OPC_CUSTOM_1  : RISCVOpcode<"CUSTOM_1",  0b0101011>;
def OPC_AMO       : RISCVOpcode<"AMO",       0b0101111>;
def OPC_OP        : RISCVOpcode<"OP",        0b0110011>;
def OPC_LUI       : RISCVOpcode<"LUI",       0b0110111>;
def OPC_OP_32     : RISCVOpcode<"OP_32",     0b0111011>;
def OPC_0111111   : RISCVOpcode<"",          0b0111111>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 145-159: TableGen record OPC_MADD / TableGen 记录 OPC_MADD
```tablegen
def OPC_MADD      : RISCVOpcode<"MADD",      0b1000011>;
def OPC_MSUB      : RISCVOpcode<"MSUB",      0b1000111>;
def OPC_NMSUB     : RISCVOpcode<"NMSUB",     0b1001011>;
def OPC_NMADD     : RISCVOpcode<"NMADD",     0b1001111>;
def OPC_OP_FP     : RISCVOpcode<"OP_FP",     0b1010011>;
def OPC_OP_V      : RISCVOpcode<"OP_V",      0b1010111>;
def OPC_CUSTOM_2  : RISCVOpcode<"CUSTOM_2",  0b1011011>;
def OPC_1011111   : RISCVOpcode<"",          0b1011111>;
def OPC_BRANCH    : RISCVOpcode<"BRANCH",    0b1100011>;
def OPC_JALR      : RISCVOpcode<"JALR",      0b1100111>;
def OPC_JAL       : RISCVOpcode<"JAL",       0b1101111>;
def OPC_SYSTEM    : RISCVOpcode<"SYSTEM",    0b1110011>;
def OPC_OP_VE     : RISCVOpcode<"OP_VE",     0b1110111>;
def OPC_CUSTOM_3  : RISCVOpcode<"CUSTOM_3",  0b1111011>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 160-177: TableGen class EltDeps<bit / TableGen 类 EltDeps<bit
```tablegen
class EltDeps<bit vl, bit mask> {
  bit VL = vl;
  bit Mask = mask;
}

def EltDepsNone      : EltDeps<vl=0, mask=0>;
def EltDepsVL        : EltDeps<vl=1, mask=0>;
def EltDepsMask      : EltDeps<vl=0, mask=1>;
def EltDepsVLMask    : EltDeps<vl=1, mask=1>;

class EEW<bits<2> val> {
  bits<2> Value = val;
}
def EEW1     : EEW<0>;
def EEWSEWx1 : EEW<1>;
def EEWSEWx2 : EEW<2>;
def EEWSEWx4 : EEW<3>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 178-193: TableGen class AltFmtType<bits<2> / TableGen 类 AltFmtType<bits<2>
```tablegen
class AltFmtType<bits<2> val> {
  bits<2> Value = val;
}
def DONT_CARE_ALTFMT : AltFmtType<0>;
def IS_NOT_ALTFMT    : AltFmtType<1>;
def IS_ALTFMT        : AltFmtType<2>;

class RVInstCommon<dag outs, dag ins, string opcodestr, string argstr,
                   list<dag> pattern, InstFormat format> : Instruction {
  let Namespace = "RISCV";

  dag OutOperandList = outs;
  dag InOperandList = ins;
  let AsmString = opcodestr # !if(!empty(argstr), "", "\t" # argstr);
  let Pattern = pattern;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 194-209: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  InstFormat Format = format;

  let TSFlags{4-0} = Format.Value;

  // Indicates when Vd and Vs1/Vs2 cannot be the same register.
  // We require that the Vd operand be named $vd, the Vs1 operand be named
  // $vs1, and the Vs2 operand be named $vs2.
  RISCVVConstraint VS1VS2Constraint = NoConstraint;
  let TSFlags{6-5} = VS1VS2Constraint.Value;

  // Indicates that the destination and the VM operand cannot both be V0.
  // We require the the VM operand to be named $vm and the destination
  // operand to be named $vd.
  bit VMConstraint = 0;
  let TSFlags{7} = VMConstraint;
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 210-224: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  bits<3> VLMul = 0;
  let TSFlags{10-8} = VLMul;

  bit IsTiedPseudo = 0;
  let TSFlags{11} = IsTiedPseudo;

  bit HasSEWOp = 0;
  let TSFlags{12} = HasSEWOp;

  bit HasVLOp = 0;
  let TSFlags{13} = HasVLOp;

  bit HasVecPolicyOp = 0;
  let TSFlags{14} = HasVecPolicyOp;
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 225-240: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  bit IsRVVWideningReduction = 0;
  let TSFlags{15} = IsRVVWideningReduction;

  bit UsesMaskPolicy = 0;
  let TSFlags{16} = UsesMaskPolicy;

  // Indicates that the result can be considered sign extended from bit 31. Some
  // instructions with this flag aren't W instructions, but are either sign
  // extended from a smaller size, always outputs a small integer, or put zeros
  // in bits 63:31. Used by the SExtWRemoval pass.
  bit IsSignExtendingOpW = 0;
  let TSFlags{17} = IsSignExtendingOpW;

  bit HasRoundModeOp = 0;
  let TSFlags{18} =  HasRoundModeOp;
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 241-257: Commentary and design intent / 注释与设计意图
```tablegen
  // This is only valid when HasRoundModeOp is set to 1. HasRoundModeOp is set
  // to 1 for vector fixed-point or floating-point intrinsics. This bit is
  // processed under pass 'RISCVInsertReadWriteCSR' pass to distinguish between
  // fixed-point / floating-point instructions and emit appropriate read/write
  // to the correct CSR.
  bit UsesVXRM = 0;
  let TSFlags{19} =  UsesVXRM;

  // Indicates whether these instructions can partially overlap between source
  // registers and destination registers according to the vector spec.
  // 0 -> not a vector pseudo
  // 1 -> default value for vector pseudos. not widening or narrowing.
  // 2 -> narrowing case
  // 3 -> widening case
  bits<2> TargetOverlapConstraintType = 0;
  let TSFlags{21-20} = TargetOverlapConstraintType;
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 258-276: Commentary and design intent / 注释与设计意图
```tablegen
  // Most vector instructions are elementwise, but some may depend on the value
  // of vl (vslide1down.vx), and others may depend on the mask (viota.m) or both
  // (vredsum.vs). By default we assume elements depend on both vl and the mask,
  // but if marked as EltDepsNone or EltDepsMask then RISCVVLOptimizer will
  // reduce its vl.
  EltDeps ElementsDependOn = EltDepsVLMask;
  let TSFlags{22} = ElementsDependOn.VL;
  let TSFlags{23} = ElementsDependOn.Mask;

  // Indicates the EEW of a vector instruction's destination operand.
  EEW DestEEW = EEWSEWx1;
  let TSFlags{25-24} = DestEEW.Value;

  // Some vector instructions like vslidedown/vrgather will read elements past
  // VL, and should be marked to make sure RISCVVLOptimizer doesn't reduce its
  // operands' VLs.
  bit ReadsPastVL = 0;
  let TSFlags{26} = ReadsPastVL;
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 277-292: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  // 0 -> Don't care about altfmt bit in VTYPE.
  // 1 -> Is not altfmt.
  // 2 -> Is altfmt(BF16).
  AltFmtType AltFmtType = DONT_CARE_ALTFMT;
  let TSFlags{28-27} = AltFmtType.Value;

  // XSfmmbase
  bit HasTWidenOp = 0;
  let TSFlags{29} = HasTWidenOp;

  bit HasTmOp = 0;
  let TSFlags{30} = HasTmOp;

  bit HasTkOp = 0;
  let TSFlags{31} = HasTkOp;
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 293-306: TableGen class RVInst<dag / TableGen 类 RVInst<dag
```tablegen

class RVInst<dag outs, dag ins, string opcodestr, string argstr,
             list<dag> pattern, InstFormat format>
    : RVInstCommon<outs, ins, opcodestr, argstr, pattern, format> {
  field bits<32> Inst;
  let Size = 4;
}

class RVInst48<dag outs, dag ins, string opcodestr, string argstr,
               list<dag> pattern, InstFormat format>
    : RVInstCommon<outs, ins, opcodestr, argstr, pattern, format> {
  field bits<48> Inst;
  let Size = 6;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 307-321: TableGen class RVInst64<dag / TableGen 类 RVInst64<dag
```tablegen

class RVInst64<dag outs, dag ins, string opcodestr, string argstr,
               list<dag> pattern, InstFormat format>
    : RVInstCommon<outs, ins, opcodestr, argstr, pattern, format> {
  field bits<64> Inst;
  let Size = 8;
}

// Pseudo instructions
class Pseudo<dag outs, dag ins, list<dag> pattern, string opcodestr = "", string argstr = "">
    : RVInstCommon<outs, ins, opcodestr, argstr, pattern, InstFormatPseudo> {
  let isPseudo = 1;
  let isCodeGenOnly = 1;
  let Size = 4;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 322-337: TableGen class PseudoQuietFCMP<DAGOperand / TableGen 类 PseudoQuietFCMP<DAGOperand
```tablegen

class PseudoQuietFCMP<DAGOperand Ty>
    : Pseudo<(outs GPR:$rd), (ins Ty:$rs1, Ty:$rs2), []> {
  let hasSideEffects = 1;
  let mayLoad = 0;
  let mayStore = 0;
}

// Pseudo load instructions.
class PseudoLoad<string opcodestr, DAGOperand rdty = GPR>
    : Pseudo<(outs rdty:$rd), (ins bare_symbol:$addr), [], opcodestr, "$rd, $addr"> {
  let hasSideEffects = 0;
  let mayLoad = 1;
  let mayStore = 0;
  let isCodeGenOnly = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 338-354: TableGen class PseudoFloatLoad<string / TableGen 类 PseudoFloatLoad<string
```tablegen

class PseudoFloatLoad<string opcodestr, RegisterClass rdty>
    : Pseudo<(outs GPR:$tmp, rdty:$rd), (ins bare_symbol:$addr), [], opcodestr, "$rd, $addr, $tmp"> {
  let hasSideEffects = 0;
  let mayLoad = 1;
  let mayStore = 0;
  let isCodeGenOnly = 0;
}

// Pseudo store instructions.
class PseudoStore<string opcodestr, DAGOperand rsty = GPR>
    : Pseudo<(outs GPR:$tmp), (ins rsty:$rs, bare_symbol:$addr), [], opcodestr, "$rs, $addr, $tmp"> {
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 1;
  let isCodeGenOnly = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 355-373: TableGen class for / TableGen 类 for
```tablegen

// Instruction formats are listed in the order they appear in the RISC-V
// instruction set manual (R, R4, I, S, B, U, J).

// Common base class for R format instructions. Bits {31-25} should be set by
// the subclasses.
class RVInstRBase<bits<3> funct3, RISCVOpcode opcode, dag outs,
                  dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> rs2;
  bits<5> rs1;
  bits<5> rd;

  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 374-388: TableGen class RVInstR<bits<7> / TableGen 类 RVInstR<bits<7>
```tablegen

class RVInstR<bits<7> funct7, bits<3> funct3, RISCVOpcode opcode, dag outs,
              dag ins, string opcodestr, string argstr>
    : RVInstRBase<funct3, opcode, outs, ins, opcodestr, argstr> {
  let Inst{31-25} = funct7;
}

class RVInstRAtomic<bits<5> funct5, bit aq, bit rl, bits<3> funct3,
                    RISCVOpcode opcode, dag outs, dag ins, string opcodestr,
                    string argstr>
    : RVInstRBase<funct3, opcode, outs, ins, opcodestr, argstr> {
  let Inst{31-27} = funct5;
  let Inst{26} = aq;
  let Inst{25} = rl;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 389-404: TableGen class RVInstRFrm<bits<7> / TableGen 类 RVInstRFrm<bits<7>
```tablegen

class RVInstRFrm<bits<7> funct7, RISCVOpcode opcode, dag outs, dag ins,
                 string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> rs2;
  bits<5> rs1;
  bits<3> frm;
  bits<5> rd;

  let Inst{31-25} = funct7;
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = frm;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 405-421: TableGen class RVInstR4<bits<2> / TableGen 类 RVInstR4<bits<2>
```tablegen

class RVInstR4<bits<2> funct2, bits<3> funct3, RISCVOpcode opcode, dag outs,
               dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR4> {
  bits<5> rs3;
  bits<5> rs2;
  bits<5> rs1;
  bits<5> rd;

  let Inst{31-27} = rs3;
  let Inst{26-25} = funct2;
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 422-439: TableGen class RVInstR4Frm<bits<2> / TableGen 类 RVInstR4Frm<bits<2>
```tablegen

class RVInstR4Frm<bits<2> funct2, RISCVOpcode opcode, dag outs, dag ins,
                  string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR4> {
  bits<5> rs3;
  bits<5> rs2;
  bits<5> rs1;
  bits<3> frm;
  bits<5> rd;

  let Inst{31-27} = rs3;
  let Inst{26-25} = funct2;
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = frm;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 440-453: TableGen class for / TableGen 类 for
```tablegen

// Common base class for I format instructions. Bits {31-20} should be set by
// the subclasses.
class RVInstIBase<bits<3> funct3, RISCVOpcode opcode, dag outs, dag ins,
                  string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatI> {
  bits<5> rs1;
  bits<5> rd;

  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 454-467: TableGen class RVInstI<bits<3> / TableGen 类 RVInstI<bits<3>
```tablegen

class RVInstI<bits<3> funct3, RISCVOpcode opcode, dag outs, dag ins,
              string opcodestr, string argstr>
    : RVInstIBase<funct3, opcode, outs, ins, opcodestr, argstr> {
  bits<12> imm12;

  let Inst{31-20} = imm12;
}

class RVInstIShift<bits<5> imm11_7, bits<3> funct3, RISCVOpcode opcode,
                   dag outs, dag ins, string opcodestr, string argstr>
    : RVInstIBase<funct3, opcode, outs, ins, opcodestr, argstr> {
  bits<6> shamt;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 468-481: TableGen class RVInstIShiftW<bits<7> / TableGen 类 RVInstIShiftW<bits<7>
```tablegen
  let Inst{31-27} = imm11_7;
  let Inst{26} = 0;
  let Inst{25-20} = shamt;
}

class RVInstIShiftW<bits<7> imm11_5, bits<3> funct3, RISCVOpcode opcode,
                    dag outs, dag ins, string opcodestr, string argstr>
    : RVInstIBase<funct3, opcode, outs, ins, opcodestr, argstr> {
  bits<5> shamt;

  let Inst{31-25} = imm11_5;
  let Inst{24-20} = shamt;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 482-501: TableGen class RVInstIUnary<bits<12> / TableGen 类 RVInstIUnary<bits<12>
```tablegen
class RVInstIUnary<bits<12> imm12, bits<3> funct3, RISCVOpcode opcode,
                   dag outs, dag ins, string opcodestr, string argstr>
    : RVInstIBase<funct3, opcode, outs, ins, opcodestr, argstr> {
  let Inst{31-20} = imm12;
}

class RVInstS<bits<3> funct3, RISCVOpcode opcode, dag outs, dag ins,
              string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatS> {
  bits<12> imm12;
  bits<5> rs2;
  bits<5> rs1;

  let Inst{31-25} = imm12{11-5};
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = imm12{4-0};
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 502-518: TableGen class RVInstB<bits<3> / TableGen 类 RVInstB<bits<3>
```tablegen

class RVInstB<bits<3> funct3, RISCVOpcode opcode, dag outs, dag ins,
              string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatB> {
  bits<12> imm12;
  bits<5> rs2;
  bits<5> rs1;

  let Inst{31} = imm12{11};
  let Inst{30-25} = imm12{9-4};
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-8} = imm12{3-0};
  let Inst{7} = imm12{10};
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 519-534: TableGen class RVInstBIMM<bits<3> / TableGen 类 RVInstBIMM<bits<3>
```tablegen

class RVInstBIMM<bits<3> funct3, RISCVOpcode opcode, dag outs, dag ins,
                 string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatB> {
  bits<12> imm12;
  bits<5> cimm;
  bits<5> rs1;
  let Inst{31} = imm12{11};
  let Inst{30-25} = imm12{9-4};
  let Inst{24-20} = cimm;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-8} = imm12{3-0};
  let Inst{7} = imm12{10};
  let Inst{6-0} = opcode.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 535-552: TableGen class RVInstU<RISCVOpcode / TableGen 类 RVInstU<RISCVOpcode
```tablegen

class RVInstU<RISCVOpcode opcode, dag outs, dag ins, string opcodestr,
              string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatU> {
  bits<20> imm20;
  bits<5> rd;

  let Inst{31-12} = imm20;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}

class RVInstJ<RISCVOpcode opcode, dag outs, dag ins, string opcodestr,
              string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatJ> {
  bits<20> imm20;
  bits<5> rd;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 553-570: TableGen class DirectiveInsnR<dag / TableGen 类 DirectiveInsnR<dag
```tablegen
  let Inst{31} = imm20{19};
  let Inst{30-21} = imm20{9-0};
  let Inst{20} = imm20{10};
  let Inst{19-12} = imm20{18-11};
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode.Value;
}

//===----------------------------------------------------------------------===//
// Instruction classes for .insn directives
//===----------------------------------------------------------------------===//

class DirectiveInsnR<dag outs, dag ins, string argstr>
  : RVInst<outs, ins, "", "", [], InstFormatR> {
  bits<7> opcode;
  bits<7> funct7;
  bits<3> funct3;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 571-584: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  bits<5> rs2;
  bits<5> rs1;
  bits<5> rd;

  let Inst{31-25} = funct7;
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode;

  let AsmString = ".insn r " # argstr;
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 585-603: TableGen class DirectiveInsnR4<dag / TableGen 类 DirectiveInsnR4<dag
```tablegen
class DirectiveInsnR4<dag outs, dag ins, string argstr>
  : RVInst<outs, ins, "", "", [], InstFormatR4> {
  bits<7> opcode;
  bits<2> funct2;
  bits<3> funct3;

  bits<5> rs3;
  bits<5> rs2;
  bits<5> rs1;
  bits<5> rd;

  let Inst{31-27} = rs3;
  let Inst{26-25} = funct2;
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 604-621: TableGen class DirectiveInsnI<dag / TableGen 类 DirectiveInsnI<dag
```tablegen
  let AsmString = ".insn r4 " # argstr;
}

class DirectiveInsnI<dag outs, dag ins, string argstr>
  : RVInst<outs, ins, "", "", [], InstFormatI> {
  bits<7> opcode;
  bits<3> funct3;

  bits<12> imm12;
  bits<5> rs1;
  bits<5> rd;

  let Inst{31-20} = imm12;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 622-640: TableGen class DirectiveInsnS<dag / TableGen 类 DirectiveInsnS<dag
```tablegen
  let AsmString = ".insn i " # argstr;
}

class DirectiveInsnS<dag outs, dag ins, string argstr>
  : RVInst<outs, ins, "", "", [], InstFormatS> {
  bits<7> opcode;
  bits<3> funct3;

  bits<12> imm12;
  bits<5> rs2;
  bits<5> rs1;

  let Inst{31-25} = imm12{11-5};
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = imm12{4-0};
  let Inst{6-0} = opcode;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 641-661: TableGen class DirectiveInsnB<dag / TableGen 类 DirectiveInsnB<dag
```tablegen
  let AsmString = ".insn s " # argstr;
}

class DirectiveInsnB<dag outs, dag ins, string argstr>
  : RVInst<outs, ins, "", "", [], InstFormatB> {
  bits<7> opcode;
  bits<3> funct3;

  bits<12> imm12;
  bits<5> rs2;
  bits<5> rs1;

  let Inst{31} = imm12{11};
  let Inst{30-25} = imm12{9-4};
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-8} = imm12{3-0};
  let Inst{7} = imm12{10};
  let Inst{6-0} = opcode;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 662-675: TableGen class DirectiveInsnU<dag / TableGen 类 DirectiveInsnU<dag
```tablegen
  let AsmString = ".insn b " # argstr;
}

class DirectiveInsnU<dag outs, dag ins, string argstr>
  : RVInst<outs, ins, "", "", [], InstFormatU> {
  bits<7> opcode;

  bits<20> imm20;
  bits<5> rd;

  let Inst{31-12} = imm20;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 676-689: TableGen class DirectiveInsnJ<dag / TableGen 类 DirectiveInsnJ<dag
```tablegen
  let AsmString = ".insn u " # argstr;
}

class DirectiveInsnJ<dag outs, dag ins, string argstr>
  : RVInst<outs, ins, "", "", [], InstFormatJ> {
  bits<7> opcode;

  bits<20> imm20;
  bits<5> rd;

  let Inst{31-12} = imm20;
  let Inst{11-7} = rd;
  let Inst{6-0} = opcode;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 690-691: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let AsmString = ".insn j " # argstr;
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

## Key Concepts / 关键概念
- **Instruction encodings** / **指令编码**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
