# RISCVInstrInfoXAndes.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoXAndes.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoXAndes.td ----------------------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the vendor extensions defined by Andes Technology.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// RISC-V specific DAG Nodes.
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 16-29: TableGen record SDT_NDS_FMV_BF16_X / TableGen 记录 SDT_NDS_FMV_BF16_X
```tablegen

def SDT_NDS_FMV_BF16_X
    : SDTypeProfile<1, 1, [SDTCisVT<0, bf16>, SDTCisVT<1, XLenVT>]>;
def SDT_NDS_FMV_X_ANYEXTBF16
    : SDTypeProfile<1, 1, [SDTCisVT<0, XLenVT>, SDTCisVT<1, bf16>]>;

def riscv_nds_fmv_bf16_x
    : SDNode<"RISCVISD::NDS_FMV_BF16_X", SDT_NDS_FMV_BF16_X>;
def riscv_nds_fmv_x_anyextbf16
    : SDNode<"RISCVISD::NDS_FMV_X_ANYEXTBF16", SDT_NDS_FMV_X_ANYEXTBF16>;

//===----------------------------------------------------------------------===//
// Operand and SDNode transformation definitions.
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 30-44: TableGen record bare_simm11_lsb0 / TableGen 记录 bare_simm11_lsb0
```tablegen

// A 11-bit signed immediate where the least significant bit is zero.
def bare_simm11_lsb0 : Operand<OtherVT> {
  let ParserMatchClass = BareSImmNLsb0AsmOperand<11>;
  let PrintMethod = "printBranchOperand";
  let EncoderMethod = "getImmOpValueAsrN<1>";
  let DecoderMethod = "decodeSImmOperandAndLslN<11, 1>";
  let MCOperandPredicate = [{
    int64_t Imm;
    if (MCOp.evaluateAsConstantImm(Imm))
      return isShiftedInt<10, 1>(Imm);
    return MCOp.isBareSymbolRef();
  }];
  let OperandType = "OPERAND_PCREL";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 45-62: TableGen record simm18 / TableGen 记录 simm18
```tablegen

def simm18 : Operand<XLenVT> {
  let ParserMatchClass = SImmAsmOperand<18>;
  let EncoderMethod = "getImmOpValue";
  let DecoderMethod = "decodeSImmOperand<18>";
}

def simm18_lsb0 : Operand<XLenVT> {
  let ParserMatchClass = SImmAsmOperand<18, "Lsb0">;
  let EncoderMethod = "getImmOpValueAsrN<1>";
  let DecoderMethod = "decodeSImmOperandAndLslN<18, 1>";
}

def simm19_lsb00 : Operand<XLenVT> {
  let ParserMatchClass = SImmAsmOperand<19, "Lsb00">;
  let EncoderMethod = "getImmOpValueAsrN<2>";
  let DecoderMethod = "decodeSImmOperandAndLslN<19, 2>";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 63-80: TableGen record simm20_lsb000 / TableGen 记录 simm20_lsb000
```tablegen

def simm20_lsb000 : Operand<XLenVT> {
  let ParserMatchClass = SImmAsmOperand<20, "Lsb000">;
  let EncoderMethod = "getImmOpValueAsrN<3>";
  let DecoderMethod = "decodeSImmOperandAndLslN<20, 3>";
}

// Predicate: True if immediate is a power of 2.
def PowerOf2 : IntImmLeaf<XLenVT, [{
  return isPowerOf2_64(Imm.getZExtValue());
}]>;

// Transformation function: Get log2 of immediate.
def Log2 : SDNodeXForm<imm, [{
  uint64_t Imm = Log2_64(N->getZExtValue());
  return CurDAG->getTargetConstant(Imm, SDLoc(N), N->getValueType(0));
}]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 81-99: TableGen class RISCVNDSVLN<bit / TableGen 类 RISCVNDSVLN<bit
```tablegen
//===----------------------------------------------------------------------===//
// Pseudo table
//===----------------------------------------------------------------------===//

class RISCVNDSVLN<bit M, bit U, bits<3> S, bits<3> L> {
  bits<1> Masked = M;
  bits<1> Unsigned = U;
  bits<3> Log2SEW = S;
  bits<3> LMUL = L;
  Pseudo Pseudo = !cast<Pseudo>(NAME);
}

def RISCVNDSVLNTable : GenericTable {
  let FilterClass = "RISCVNDSVLN";
  let CppTypeName = "NDSVLNPseudo";
  let Fields = ["Masked", "Unsigned", "Log2SEW", "LMUL", "Pseudo"];
  let PrimaryKey = ["Masked", "Unsigned", "Log2SEW", "LMUL"];
  let PrimaryKeyName = "getNDSVLNPseudo";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 100-113: TableGen class NDSRVInstBB<bit / TableGen 类 NDSRVInstBB<bit
```tablegen

//===----------------------------------------------------------------------===//
// Instruction Class Templates
//===----------------------------------------------------------------------===//

class NDSRVInstBB<bit cs, string opcodestr>
    : RVInst<(outs),
             (ins GPR:$rs1, uimmlog2xlen:$cimm, bare_simm11_lsb0:$imm10),
             opcodestr, "$rs1, $cimm, $imm10", [], InstFormatNDS_BRANCH_10>,
      Sched<[WriteJmp, ReadIALU]> {
  bits<10> imm10;
  bits<5> rs1;
  bits<6> cimm;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 114-128: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Inst{31} = imm10{9};
  let Inst{30} = cs;
  let Inst{29-25} = imm10{8-4};
  let Inst{24-20} = cimm{4-0};
  let Inst{19-15} = rs1;
  let Inst{14-12} = 0b111;
  let Inst{11-8} = imm10{3-0};
  let Inst{7} = cimm{5};
  let Inst{6-0} = OPC_CUSTOM_2.Value;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
  let isBranch = 1;
  let isTerminator = 1;
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 129-152: TableGen class NDSRVInstBC<bits<3> / TableGen 类 NDSRVInstBC<bits<3>
```tablegen

class NDSRVInstBC<bits<3> funct3, string opcodestr>
    : RVInst<(outs), (ins GPR:$rs1, uimm7:$cimm, bare_simm11_lsb0:$imm10),
             opcodestr, "$rs1, $cimm, $imm10", [], InstFormatNDS_BRANCH_10>,
      Sched<[WriteJmp, ReadIALU]> {
  bits<10> imm10;
  bits<5> rs1;
  bits<7> cimm;

  let Inst{31} = imm10{9};
  let Inst{30} = cimm{6};
  let Inst{29-25} = imm10{8-4};
  let Inst{24-20} = cimm{4-0};
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-8} = imm10{3-0};
  let Inst{7} = cimm{5};
  let Inst{6-0} = OPC_CUSTOM_2.Value;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
  let isBranch = 1;
  let isTerminator = 1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 153-173: TableGen class NDSRVInstBFO<bits<3> / TableGen 类 NDSRVInstBFO<bits<3>
```tablegen

class NDSRVInstBFO<bits<3> funct3, string opcodestr>
    : RVInst<(outs GPR:$rd),
             (ins GPR:$rs1, uimmlog2xlen:$msb, uimmlog2xlen:$lsb),
             opcodestr, "$rd, $rs1, $msb, $lsb", [], InstFormatOther>,
      Sched<[WriteIALU, ReadIALU]> {
  bits<5> rd;
  bits<5> rs1;
  bits<6> msb;
  bits<6> lsb;

  let Inst{31-26} = msb;
  let Inst{25-20} = lsb;
  let Inst{19-15} = rs1;
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = OPC_CUSTOM_2.Value;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 174-193: TableGen class NDSRVInstRR<bits<7> / TableGen 类 NDSRVInstRR<bits<7>
```tablegen

class NDSRVInstRR<bits<7> funct7, string opcodestr>
    : RVInstR<funct7, 0b000, OPC_CUSTOM_2,
              (outs GPR:$rd), (ins GPR:$rs1, GPR:$rs2),
              opcodestr, "$rd, $rs1, $rs2">,
      Sched<[WriteIALU, ReadIALU, ReadIALU]> {
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
}

class NDSRVInstLEA<bits<7> funct7, string opcodestr>
    : RVInstR<funct7, 0b000, OPC_CUSTOM_2,
              (outs GPR:$rd), (ins GPR:$rs2, GPR:$rs1),
              opcodestr, "$rd, $rs1, $rs2">,
      Sched<[WriteIALU, ReadIALU, ReadIALU]> {
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 194-214: TableGen class NDSRVInstLBGP<bits<2> / TableGen 类 NDSRVInstLBGP<bits<2>
```tablegen

// GP: ADDI, LB, LBU
class NDSRVInstLBGP<bits<2> funct2, string opcodestr>
    : RVInst<(outs GPR:$rd), (ins simm18:$imm18),
             opcodestr, "$rd, ${imm18}", [], InstFormatOther> {
  bits<18> imm18;
  bits<5> rd;

  let Inst{31} = imm18{17};
  let Inst{30-21} = imm18{10-1};
  let Inst{20} = imm18{11};
  let Inst{19-17} = imm18{14-12};
  let Inst{16-15} = imm18{16-15};
  let Inst{14} = imm18{0};
  let Inst{13-12} = funct2;
  let Inst{11-7} = rd;
  let Inst{6-0} = OPC_CUSTOM_0.Value;
  let hasSideEffects = 0;
  let mayLoad = 1;
  let mayStore = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 215-234: TableGen class NDSRVInstLHGP<bits<3> / TableGen 类 NDSRVInstLHGP<bits<3>
```tablegen

// GP: LH, LHU
class NDSRVInstLHGP<bits<3> funct3, string opcodestr>
    : RVInst<(outs GPR:$rd), (ins simm18_lsb0:$imm17),
             opcodestr, "$rd, ${imm17}", [], InstFormatOther> {
  bits<17> imm17;
  bits<5> rd;

  let Inst{31} = imm17{16};
  let Inst{30-21} = imm17{9-0};
  let Inst{20} = imm17{10};
  let Inst{19-17} = imm17{13-11};
  let Inst{16-15} = imm17{15-14};
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = OPC_CUSTOM_1.Value;
  let hasSideEffects = 0;
  let mayLoad = 1;
  let mayStore = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 235-255: TableGen class NDSRVInstLWGP<bits<3> / TableGen 类 NDSRVInstLWGP<bits<3>
```tablegen

// GP: LW, LWU
class NDSRVInstLWGP<bits<3> funct3, string opcodestr>
    : RVInst<(outs GPR:$rd), (ins simm19_lsb00:$imm17),
             opcodestr, "$rd, ${imm17}", [], InstFormatOther> {
  bits<17> imm17;
  bits<5> rd;

  let Inst{31} = imm17{16};
  let Inst{30-22} = imm17{8-0};
  let Inst{21} = imm17{15};
  let Inst{20} = imm17{9};
  let Inst{19-17} = imm17{12-10};
  let Inst{16-15} = imm17{14-13};
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = OPC_CUSTOM_1.Value;
  let hasSideEffects = 0;
  let mayLoad = 1;
  let mayStore = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 256-276: TableGen class NDSRVInstLDGP<bits<3> / TableGen 类 NDSRVInstLDGP<bits<3>
```tablegen

// GP: LD
class NDSRVInstLDGP<bits<3> funct3, string opcodestr>
    : RVInst<(outs GPR:$rd), (ins simm20_lsb000:$imm17),
             opcodestr, "$rd, ${imm17}", [], InstFormatOther> {
  bits<17> imm17;
  bits<5> rd;

  let Inst{31} = imm17{16};
  let Inst{30-23} = imm17{7-0};
  let Inst{22-21} = imm17{15-14};
  let Inst{20} = imm17{8};
  let Inst{19-17} = imm17{11-9};
  let Inst{16-15} = imm17{13-12};
  let Inst{14-12} = funct3;
  let Inst{11-7} = rd;
  let Inst{6-0} = OPC_CUSTOM_1.Value;
  let hasSideEffects = 0;
  let mayLoad = 1;
  let mayStore = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 277-298: TableGen class NDSRVInstSBGP<bits<2> / TableGen 类 NDSRVInstSBGP<bits<2>
```tablegen

// GP: SB
class NDSRVInstSBGP<bits<2> funct2, string opcodestr>
    : RVInst<(outs), (ins GPR:$rs2, simm18:$imm18),
             opcodestr, "$rs2, ${imm18}", [], InstFormatOther> {
  bits<18> imm18;
  bits<5> rs2;

  let Inst{31} = imm18{17};
  let Inst{30-25} = imm18{10-5};
  let Inst{24-20} = rs2;
  let Inst{19-17} = imm18{14-12};
  let Inst{16-15} = imm18{16-15};
  let Inst{14} = imm18{0};
  let Inst{13-12} = funct2;
  let Inst{11-8} = imm18{4-1};
  let Inst{7} = imm18{11};
  let Inst{6-0} = OPC_CUSTOM_0.Value;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 299-319: TableGen class NDSRVInstSHGP<bits<3> / TableGen 类 NDSRVInstSHGP<bits<3>
```tablegen

// GP: SH
class NDSRVInstSHGP<bits<3> funct3, string opcodestr>
    : RVInst<(outs), (ins GPR:$rs2, simm18_lsb0:$imm17),
             opcodestr, "$rs2, ${imm17}", [], InstFormatOther> {
  bits<17> imm17;
  bits<5> rs2;

  let Inst{31} = imm17{16};
  let Inst{30-25} = imm17{9-4};
  let Inst{24-20} = rs2;
  let Inst{19-17} = imm17{13-11};
  let Inst{16-15} = imm17{15-14};
  let Inst{14-12} = funct3;
  let Inst{11-8} = imm17{3-0};
  let Inst{7} = imm17{10};
  let Inst{6-0} = OPC_CUSTOM_1.Value;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 320-341: TableGen class NDSRVInstSWGP<bits<3> / TableGen 类 NDSRVInstSWGP<bits<3>
```tablegen

// GP: SW
class NDSRVInstSWGP<bits<3> funct3, string opcodestr>
    : RVInst<(outs), (ins GPR:$rs2, simm19_lsb00:$imm17),
             opcodestr, "$rs2, ${imm17}", [], InstFormatOther> {
  bits<17> imm17;
  bits<5> rs2;

  let Inst{31} = imm17{16};
  let Inst{30-25} = imm17{8-3};
  let Inst{24-20} = rs2;
  let Inst{19-17} = imm17{12-10};
  let Inst{16-15} = imm17{14-13};
  let Inst{14-12} = funct3;
  let Inst{11-9} = imm17{2-0};
  let Inst{8} = imm17{15};
  let Inst{7} = imm17{9};
  let Inst{6-0} = OPC_CUSTOM_1.Value;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 342-363: TableGen class NDSRVInstSDGP<bits<3> / TableGen 类 NDSRVInstSDGP<bits<3>
```tablegen

// GP: SD
class NDSRVInstSDGP<bits<3> funct3, string opcodestr>
    : RVInst<(outs), (ins GPR:$rs2, simm20_lsb000:$imm17),
             opcodestr, "$rs2, ${imm17}", [], InstFormatOther> {
  bits<17> imm17;
  bits<5> rs2;

  let Inst{31} = imm17{16};
  let Inst{30-25} = imm17{7-2};
  let Inst{24-20} = rs2;
  let Inst{19-17} = imm17{11-9};
  let Inst{16-15} = imm17{13-12};
  let Inst{14-12} = funct3;
  let Inst{11-10} = imm17{1-0};
  let Inst{9-8} = imm17{15-14};
  let Inst{7} = imm17{8};
  let Inst{6-0} = OPC_CUSTOM_1.Value;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 364-383: TableGen class NDSRVInstVSINTLN<bits<5> / TableGen 类 NDSRVInstVSINTLN<bits<5>
```tablegen

class NDSRVInstVSINTLN<bits<5> funct5, string opcodestr>
    : RVInst<(outs VR:$vd), (ins GPRMemZeroOffset:$rs1),
             opcodestr, "$vd, ${rs1}", [], InstFormatR>,
      VLESchedMC {
  bits<5> rs1;
  bits<5> vd;

  let Inst{31-26} = 0b000001;
  let Inst{25} = 1;
  let Inst{24-20} = funct5;
  let Inst{19-15} = rs1;
  let Inst{14-12} = 0b100;
  let Inst{11-7} = vd;
  let Inst{6-0} = OPC_CUSTOM_2.Value;
  let hasSideEffects = 0;
  let mayLoad = 1;
  let mayStore = 0;
  let Uses = [VL, VTYPE];
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 384-406: TableGen class NDSRVInstVSINTCvt<bits<5> / TableGen 类 NDSRVInstVSINTCvt<bits<5>
```tablegen
  let UseNamedOperandTable = true;
}

class NDSRVInstVSINTCvt<bits<5> funct5, string opcodestr>
    : RVInst<(outs VR:$vd), (ins VR:$vs, VMaskOp:$vm),
             opcodestr, "$vd, $vs$vm", [], InstFormatR> {
  bits<5> vs;
  bits<5> vd;
  bit vm;

  let Inst{31-26} = 0b000000;
  let Inst{25} = vm;
  let Inst{24-20} = vs;
  let Inst{19-15} = funct5;
  let Inst{14-12} = 0b100;
  let Inst{11-7} = vd;
  let Inst{6-0} = OPC_CUSTOM_2.Value;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
  let Uses = [FRM, VL, VTYPE];
  let VMConstraint = true;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 407-420: TableGen class NDSRVInstBFHCvt<bits<7> / TableGen 类 NDSRVInstBFHCvt<bits<7>
```tablegen
  let UseNamedOperandTable = true;
}

class NDSRVInstBFHCvt<bits<7> funct7, bits<5> rs1val, DAGOperand rdty,
                      DAGOperand rs2ty, string opcodestr>
    : RVInstR<funct7, 0b100, OPC_CUSTOM_2, (outs rdty:$rd),
              (ins rs2ty:$rs2), opcodestr, "$rd, $rs2"> {
  let rs1 = rs1val;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
  let mayRaiseFPException = 1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 421-440: TableGen class NDSRVInstVFPMAD<bits<6> / TableGen 类 NDSRVInstVFPMAD<bits<6>
```tablegen
class NDSRVInstVFPMAD<bits<6> funct6, string opcodestr>
    : RVInst<(outs VR:$vd), (ins VR:$vs2, FPR32:$rs1, VMaskOp:$vm),
             opcodestr # "." # "vf", "$vd, $rs1, $vs2$vm", [], InstFormatR>,
      SchedBinaryMC<"WriteVFMulAddF", "ReadVFMulAddV", "ReadVFMulAddF"> {
  bits<5> vs2;
  bits<5> rs1;
  bits<5> vd;
  bit vm;

  let Inst{31-26} = funct6;
  let Inst{25} = vm;
  let Inst{24-20} = vs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = 0b100;
  let Inst{11-7} = vd;
  let Inst{6-0} = OPC_CUSTOM_2.Value;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 441-454: TableGen class NDSRVInstVD4DOT<bits<6> / TableGen 类 NDSRVInstVD4DOT<bits<6>
```tablegen
  let VMConstraint = true;

  let UseNamedOperandTable = true;
}

class NDSRVInstVD4DOT<bits<6> funct6, string opcodestr>
    : RVInst<(outs VR:$vd), (ins VR:$vs1, VR:$vs2, VMaskOp:$vm),
             opcodestr # "." # "vv", "$vd, $vs1, $vs2$vm", [], InstFormatR>,
      SchedBinaryMC<"WriteVIMulAddV", "ReadVIMulAddV", "ReadVIMulAddV"> {
  bits<5> vs2;
  bits<5> vs1;
  bits<5> vd;
  bit vm;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 455-469: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Inst{31-26} = funct6;
  let Inst{25} = vm;
  let Inst{24-20} = vs2;
  let Inst{19-15} = vs1;
  let Inst{14-12} = 0b100;
  let Inst{11-7} = vd;
  let Inst{6-0} = OPC_CUSTOM_2.Value;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;

  let VMConstraint = true;

  let UseNamedOperandTable = true;
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 470-486: TableGen class NDSRVInstVBFHCvt<bits<5> / TableGen 类 NDSRVInstVBFHCvt<bits<5>
```tablegen

class NDSRVInstVBFHCvt<bits<5> vs1, string opcodestr>
    : RVInst<(outs VR:$vd), (ins VR:$vs2),
             opcodestr, "$vd, $vs2", [], InstFormatR> {
  bits<5> vs2;
  bits<5> vd;

  let Inst{31-25} = 0b0000000;
  let Inst{24-20} = vs2;
  let Inst{19-15} = vs1;
  let Inst{14-12} = 0b100;
  let Inst{11-7} = vd;
  let Inst{6-0} = OPC_CUSTOM_2.Value;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 487-510: TableGen class NDSRVInstVLN<bits<5> / TableGen 类 NDSRVInstVLN<bits<5>
```tablegen
  let Uses = [VL, VTYPE];

  let UseNamedOperandTable = true;
}

class NDSRVInstVLN<bits<5> funct5, string opcodestr>
    : RVInst<(outs VR:$vd), (ins GPRMemZeroOffset:$rs1, VMaskOp:$vm),
             opcodestr, "$vd, ${rs1}$vm", [], InstFormatR>,
      VLESchedMC {
  bits<5> rs1;
  bits<5> vd;
  bit vm;

  let Inst{31-26} = 0b000001;
  let Inst{25} = vm;
  let Inst{24-20} = funct5;
  let Inst{19-15} = rs1;
  let Inst{14-12} = 0b100;
  let Inst{11-7} = vd;
  let Inst{6-0} = OPC_CUSTOM_2.Value;
  let hasSideEffects = 0;
  let mayLoad = 1;
  let mayStore = 0;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 511-530: TableGen class VPseudoVLN8NoMask<VReg / TableGen 类 VPseudoVLN8NoMask<VReg
```tablegen
  let Uses = [VL, VTYPE];
  let VMConstraint = true;

  let UseNamedOperandTable = true;
}

class VPseudoVLN8NoMask<VReg RetClass, bit U> :
      RISCVVPseudo<(outs RetClass:$rd),
                   (ins RetClass:$passthru,
                        GPRMemZeroOffset:$rs1,
                        AVL:$vl, sew:$sew, vec_policy:$policy), []>,
      RISCVNDSVLN</*Masked*/0, /*Unsigned*/U, !logtwo(8), VLMul> {
  let mayLoad = 1;
  let mayStore = 0;
  let hasSideEffects = 0;
  let HasVLOp = 1;
  let HasSEWOp = 1;
  let HasVecPolicyOp = 1;
  let Constraints = "$rd = $passthru";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 531-547: TableGen class VPseudoVLN8Mask<VReg / TableGen 类 VPseudoVLN8Mask<VReg
```tablegen

class VPseudoVLN8Mask<VReg RetClass, bit U> :
      RISCVVPseudo<(outs GetVRegNoV0<RetClass>.R:$rd),
                   (ins GetVRegNoV0<RetClass>.R:$passthru,
                        GPRMemZeroOffset:$rs1,
                        VMV0:$vm, AVL:$vl, sew:$sew, vec_policy:$policy),
                   []>,
      RISCVNDSVLN</*Masked*/1, /*Unsigned*/U, !logtwo(8), VLMul> {
  let mayLoad = 1;
  let mayStore = 0;
  let hasSideEffects = 0;
  let HasVLOp = 1;
  let HasSEWOp = 1;
  let HasVecPolicyOp = 1;
  let UsesMaskPolicy = 1;
  let Constraints = "$rd = $passthru";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 548-561: Reusable TableGen multiclass VPseudoVWCVT_S_BF16 / 可复用的 TableGen 多类 VPseudoVWCVT_S_BF16
```tablegen

//===----------------------------------------------------------------------===//
// Multiclass
//===----------------------------------------------------------------------===//

multiclass VPseudoVWCVT_S_BF16 {
  defvar constraint = "@earlyclobber $rd";
  foreach m = MxListFW in {
    let VLMul = m.value, SEW=16 in
    def "_" # m.MX : VPseudoUnaryNoMask<m.wvrclass, m.vrclass, constraint>,
                     SchedUnary<"WriteVFWCvtIToFV", "ReadVFWCvtIToFV", m.MX, 16,
                                forcePassthruRead=true>;
  }
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 562-582: Reusable TableGen multiclass VPseudoVNCVT_BF16_S / 可复用的 TableGen 多类 VPseudoVNCVT_BF16_S
```tablegen

multiclass VPseudoVNCVT_BF16_S {
  defvar constraint = "@earlyclobber $rd";
  foreach m = MxListFW in {
    let VLMul = m.value, SEW=16 in
    def "_" # m.MX : VPseudoUnaryNoMaskRoundingMode<m.vrclass, m.wvrclass,
                                                    constraint>,
                     SchedUnary<"WriteVFNCvtFToFV", "ReadVFNCvtFToFV", m.MX, 16,
                                forcePassthruRead=true>;
  }
}

multiclass VPatConversionS_BF16<string intrinsic, string instruction> {
  foreach fvtiToFWti = AllWidenableBF16ToFloatVectors in {
    defvar fvti = fvtiToFWti.Vti;
    defvar fwti = fvtiToFWti.Wti;
    def : VPatUnaryNoMask<intrinsic, instruction, "BF16",
                          fwti.Vector, fvti.Vector,
                          fvti.Log2SEW, fvti.LMul,
                          fwti.RegClass, fvti.RegClass>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 583-608: Reusable TableGen multiclass VPatConversionBF16_S / 可复用的 TableGen 多类 VPatConversionBF16_S
```tablegen
}

multiclass VPatConversionBF16_S<string intrinsic, string instruction> {
  foreach fvtiToFWti = AllWidenableBF16ToFloatVectors in {
    defvar fvti = fvtiToFWti.Vti;
    defvar fwti = fvtiToFWti.Wti;
    def : VPatUnaryNoMaskRoundingMode<intrinsic, instruction, "S",
                                      fvti.Vector, fwti.Vector,
                                      fvti.Log2SEW, fvti.LMul,
                                      fvti.RegClass, fwti.RegClass>;
  }
}

multiclass VPseudoVLN8<bit U> {
  foreach lmul = MxSet<8>.m in {
    defvar LInfo = lmul.MX;
    defvar vreg = lmul.vrclass;
    let VLMul = lmul.value in {
      def "_V_" # LInfo :
        VPseudoVLN8NoMask<vreg, U>,
        VLESched<LInfo>;
      def "_V_" # LInfo # "_MASK" :
        VPseudoVLN8Mask<vreg, U>,
        RISCVMaskedPseudo<MaskIdx=2>,
        VLESched<LInfo>;
    }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 609-622: Reusable TableGen multiclass SCALAR_F16_FPR32 / 可复用的 TableGen 多类 SCALAR_F16_FPR32
```tablegen
  }
}

let fprclass = !cast<RegisterClass>("FPR32") in
def SCALAR_F16_FPR32 : FPR_Info<16>;

let hasSideEffects = 0 in
multiclass VPseudoVFPMAD_VF_RM {
  foreach m = SCALAR_F16_FPR32.MxList in {
    defm "" : VPseudoBinaryV_VF_RM<m, SCALAR_F16_FPR32, 0>,
              SchedBinary<"WriteVFMulAddF", "ReadVFMulAddV", "ReadVFMulAddF",
                          m.MX,  SCALAR_F16_FPR32.SEW, forcePassthruRead=true>;
  }
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 623-641: Reusable TableGen multiclass VPatVFPMADBinaryV_VX_RM / 可复用的 TableGen 多类 VPatVFPMADBinaryV_VX_RM
```tablegen

multiclass VPatVFPMADBinaryV_VX_RM<string intrinsic, string instruction,
                                   list<VTypeInfo> vtilist> {
  foreach vti = vtilist in {
    defvar kind = "V"#vti.ScalarSuffix;
    defm : VPatBinaryRoundingMode<intrinsic,
                                  instruction#"_"#kind#"_"#vti.LMul.MX,
                                  vti.Vector, vti.Vector, f32, vti.Mask,
                                  vti.Log2SEW, vti.RegClass,
                                  vti.RegClass, FPR32>;
  }
}

multiclass VPseudoVD4DOT_VV {
  foreach m = [V_MF2, V_M1, V_M2, V_M4, V_M8] in {
    defm "" : VPseudoBinaryV_VV<m>,
              SchedBinary<"WriteVIALUV", "ReadVIALUV", "ReadVIALUV", m.MX,
                          forcePassthruRead=true>;
  }
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 642-655: Reusable TableGen multiclass VPatTernaryVD4DOT_VV / 可复用的 TableGen 多类 VPatTernaryVD4DOT_VV
```tablegen
}

multiclass VPatTernaryVD4DOT_VV<string intrinsic, string instruction,
                                list<VTypeInfoToWide> vtilist> {
  foreach vtiToWti = vtilist in {
    defvar vti = vtiToWti.Vti;
    defvar wti = vtiToWti.Wti;
    let Predicates = GetVTypePredicates<wti>.Predicates in
    defm : VPatTernaryWithPolicy<intrinsic, instruction, "VV",
                                 wti.Vector, vti.Vector, vti.Vector,
                                 wti.Mask, wti.Log2SEW, vti.LMul,
                                 wti.RegClass, vti.RegClass, vti.RegClass>;
  }
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 656-669: TableGen record NDS_BBC / TableGen 记录 NDS_BBC
```tablegen

//===----------------------------------------------------------------------===//
// XAndesPerf
//===----------------------------------------------------------------------===//

let DecoderNamespace = "XAndes" in {

let Predicates = [HasVendorXAndesPerf] in {
  def NDS_BBC : NDSRVInstBB<0, "nds.bbc">;
  def NDS_BBS : NDSRVInstBB<1, "nds.bbs">;

  def NDS_BEQC : NDSRVInstBC<0b101, "nds.beqc">;
  def NDS_BNEC : NDSRVInstBC<0b110, "nds.bnec">;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 670-685: TableGen record NDS_BFOS / TableGen 记录 NDS_BFOS
```tablegen
  def NDS_BFOS : NDSRVInstBFO<0b011, "nds.bfos">;
  def NDS_BFOZ : NDSRVInstBFO<0b010, "nds.bfoz">;

  def NDS_LEA_H : NDSRVInstLEA<0b0000101, "nds.lea.h">;
  def NDS_LEA_W : NDSRVInstLEA<0b0000110, "nds.lea.w">;
  def NDS_LEA_D : NDSRVInstLEA<0b0000111, "nds.lea.d">;

  let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
    def NDS_ADDIGP : NDSRVInstLBGP<0b01, "nds.addigp">;

  def NDS_LBGP  : NDSRVInstLBGP<0b00, "nds.lbgp">;
  def NDS_LBUGP : NDSRVInstLBGP<0b10, "nds.lbugp">;
  def NDS_LHGP  : NDSRVInstLHGP<0b001, "nds.lhgp">;
  def NDS_LHUGP : NDSRVInstLHGP<0b101, "nds.lhugp">;
  def NDS_LWGP  : NDSRVInstLWGP<0b010, "nds.lwgp">;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 686-701: TableGen record NDS_SBGP / TableGen 记录 NDS_SBGP
```tablegen
  def NDS_SBGP  : NDSRVInstSBGP<0b11, "nds.sbgp">;
  def NDS_SHGP  : NDSRVInstSHGP<0b000, "nds.shgp">;
  def NDS_SWGP  : NDSRVInstSWGP<0b100, "nds.swgp">;

  def NDS_FFB     : NDSRVInstRR<0b0010000, "nds.ffb">;
  def NDS_FFZMISM : NDSRVInstRR<0b0010001, "nds.ffzmism">;
  def NDS_FFMISM  : NDSRVInstRR<0b0010010, "nds.ffmism">;
  def NDS_FLMISM  : NDSRVInstRR<0b0010011, "nds.flmism">;
} // Predicates = [HasVendorXAndesPerf]

let Predicates = [HasVendorXAndesPerf, IsRV64] in {
  def NDS_LEA_B_ZE : NDSRVInstLEA<0b0001000, "nds.lea.b.ze">;
  def NDS_LEA_H_ZE : NDSRVInstLEA<0b0001001, "nds.lea.h.ze">;
  def NDS_LEA_W_ZE : NDSRVInstLEA<0b0001010, "nds.lea.w.ze">;
  def NDS_LEA_D_ZE : NDSRVInstLEA<0b0001011, "nds.lea.d.ze">;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 702-720: TableGen record NDS_LWUGP / TableGen 记录 NDS_LWUGP
```tablegen
  def NDS_LWUGP : NDSRVInstLWGP<0b110, "nds.lwugp">;
  def NDS_LDGP  : NDSRVInstLDGP<0b011, "nds.ldgp">;

  def NDS_SDGP  : NDSRVInstSDGP<0b111, "nds.sdgp">;
} // Predicates = [HasVendorXAndesPerf, IsRV64]

//===----------------------------------------------------------------------===//
// XAndesBFHCvt
//===----------------------------------------------------------------------===//

let Predicates = [HasVendorXAndesBFHCvt] in {
  def NDS_FCVT_S_BF16 : NDSRVInstBFHCvt<0b0000000, 0b00010,
                                        FPR32, FPR16, "nds.fcvt.s.bf16">,
                        Sched<[WriteFCvtF16ToF32, ReadFCvtF16ToF32]>;
  def NDS_FCVT_BF16_S : NDSRVInstBFHCvt<0b0000000, 0b00011,
                                        FPR16, FPR32, "nds.fcvt.bf16.s">,
                        Sched<[WriteFCvtF32ToF16, ReadFCvtF32ToF16]>;
} // Predicates = [HasVendorXAndesBFHCvt]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 721-734: TableGen record NDS_VFWCVT_S_BF16 / TableGen 记录 NDS_VFWCVT_S_BF16
```tablegen
//===----------------------------------------------------------------------===//
// XAndesVBFHCvt
//===----------------------------------------------------------------------===//

let Predicates = [HasVendorXAndesVBFHCvt],
    Constraints = "@earlyclobber $vd" in {
  let VS1VS2Constraint = VS2Constraint, DestEEW = EEWSEWx2 in
  def NDS_VFWCVT_S_BF16 : NDSRVInstVBFHCvt<0b00000, "nds.vfwcvt.s.bf16">;
  let Uses = [FRM, VL, VTYPE], mayRaiseFPException = true in
  def NDS_VFNCVT_BF16_S : NDSRVInstVBFHCvt<0b00001, "nds.vfncvt.bf16.s">;
} // Predicates = [HasVendorXAndesVBFHCvt],
  // Constraints = "@earlyclobber $vd"

//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 735-748: TableGen record NDS_VFWCVT_F_N / TableGen 记录 NDS_VFWCVT_F_N
```tablegen
// XAndesVSIntH
//===----------------------------------------------------------------------===//

let Predicates = [HasVendorXAndesVSIntH] in {
  def NDS_VFWCVT_F_N  : NDSRVInstVSINTCvt<0b00100, "nds.vfwcvt.f.n.v">;
  def NDS_VFWCVT_F_NU : NDSRVInstVSINTCvt<0b00101, "nds.vfwcvt.f.nu.v">;
  def NDS_VFWCVT_F_B  : NDSRVInstVSINTCvt<0b00110, "nds.vfwcvt.f.b.v">;
  def NDS_VFWCVT_F_BU : NDSRVInstVSINTCvt<0b00111, "nds.vfwcvt.f.bu.v">;
  def NDS_VLE4_V : NDSRVInstVSINTLN<0b00000, "nds.vle4.v">;
} // Predicates = [HasVendorXAndesVSIntH]

//===----------------------------------------------------------------------===//
// XAndesVSIntLoad
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 749-765: TableGen record NDS_VLN8_V / TableGen 记录 NDS_VLN8_V
```tablegen

let Predicates = [HasVendorXAndesVSIntLoad] in {
  def NDS_VLN8_V  : NDSRVInstVLN<0b00010, "nds.vln8.v">;
  def NDS_VLNU8_V : NDSRVInstVLN<0b00011, "nds.vlnu8.v">;
} // Predicates = [HasVendorXAndesVSIntLoad]

//===----------------------------------------------------------------------===//
// XAndesVPackFPH
//===----------------------------------------------------------------------===//

let Predicates = [HasVendorXAndesVPackFPH],
    Uses = [FRM, VL, VTYPE], mayRaiseFPException = true in {
  def NDS_VFPMADT_VF : NDSRVInstVFPMAD<0b000010, "nds.vfpmadt">;
  def NDS_VFPMADB_VF : NDSRVInstVFPMAD<0b000011, "nds.vfpmadb">;
} // Predicates = [HasVendorXAndesVPackFPH],
  // Uses = [FRM, VL, VTYPE], mayRaiseFPException = true
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 766-779: TableGen record NDS_VD4DOTS_VV / TableGen 记录 NDS_VD4DOTS_VV
```tablegen
//===----------------------------------------------------------------------===//
// XAndesVDot
//===----------------------------------------------------------------------===//

let Predicates = [HasVendorXAndesVDot], Uses = [VL, VTYPE] in {
  def NDS_VD4DOTS_VV  : NDSRVInstVD4DOT<0b000100, "nds.vd4dots">;
  def NDS_VD4DOTU_VV  : NDSRVInstVD4DOT<0b000111, "nds.vd4dotu">;
  def NDS_VD4DOTSU_VV : NDSRVInstVD4DOT<0b000101, "nds.vd4dotsu">;
} // Predicates = [HasVendorXAndesVDot], Uses = [VL, VTYPE]
} // DecoderNamespace = "XAndes"

//===----------------------------------------------------------------------===//
// Pseudo-instructions and codegen patterns
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 780-798: TableGen class NDS_BBPat<CondCode / TableGen 类 NDS_BBPat<CondCode
```tablegen

class NDS_BBPat<CondCode Cond, NDSRVInstBB Inst>
    : Pat<(riscv_brcc (and(XLenVT GPR:$rs1), PowerOf2:$mask), 0, Cond,
              bb:$imm10),
          (Inst GPR:$rs1, (Log2 PowerOf2:$mask), bare_simm11_lsb0:$imm10)>;

class NDS_BCPat<CondCode Cond, NDSRVInstBC Inst>
    : Pat<(riscv_brcc (XLenVT GPR:$rs1), uimm7:$cimm, Cond, bb:$imm10),
          (Inst GPR:$rs1, uimm7:$cimm, bare_simm11_lsb0:$imm10)>;

defm CC_UImmLog2XLen_NDS : SelectCC_GPR_riirr<GPR, uimmlog2xlen>;
defm CC_UImm7_NDS        : SelectCC_GPR_riirr<GPR, uimm7>;

class SelectNDS_BB<CondCode Cond>
    : Pat<(riscv_selectcc (and (XLenVT GPR:$lhs), PowerOf2:$mask), 0,
                          Cond:$cc, (XLenVT GPR:$truev), GPR:$falsev),
          (Select_GPR_Using_CC_UImmLog2XLen_NDS GPR:$lhs, (Log2 PowerOf2:$mask),
              (CCtoRISCVCC $cc), GPR:$truev, GPR:$falsev)>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 799-814: TableGen class SelectNDS_BC<CondCode / TableGen 类 SelectNDS_BC<CondCode
```tablegen
class SelectNDS_BC<CondCode Cond>
    : Pat<(riscv_selectcc (XLenVT GPR:$lhs), uimm7:$cimm, Cond:$cc,
              (XLenVT GPR:$truev), GPR:$falsev),
          (Select_GPR_Using_CC_UImm7_NDS GPR:$lhs, uimm7:$cimm,
              (CCtoRISCVCC $cc), GPR:$truev, GPR:$falsev)>;

let Predicates = [HasVendorXAndesPerf] in {
  def : NDS_BBPat<SETEQ, NDS_BBC>;
  def : NDS_BBPat<SETNE, NDS_BBS>;

  def : SelectNDS_BB<SETEQ>;
  def : SelectNDS_BB<SETNE>;

  def : NDS_BCPat<SETEQ, NDS_BEQC>;
  def : NDS_BCPat<SETNE, NDS_BNEC>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 815-829: Bulk record instantiation / 批量记录实例化
```tablegen
  def : SelectNDS_BC<SETEQ>;
  def : SelectNDS_BC<SETNE>;

  def : Pat<(sext_inreg (XLenVT GPR:$rs1), i16), (NDS_BFOS GPR:$rs1, 15, 0)>;
  def : Pat<(sext_inreg (XLenVT GPR:$rs1), i8), (NDS_BFOS GPR:$rs1, 7, 0)>;
  def : Pat<(sext_inreg (XLenVT GPR:$rs1), i1), (NDS_BFOS GPR:$rs1, 0, 0)>;

  defm : ShxAddPat<1, NDS_LEA_H>;
  defm : ShxAddPat<2, NDS_LEA_W>;
  defm : ShxAddPat<3, NDS_LEA_D>;

  def : CSImm12MulBy4Pat<NDS_LEA_W>;
  def : CSImm12MulBy8Pat<NDS_LEA_D>;
} // Predicates = [HasVendorXAndesPerf]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 830-845: Bulk record instantiation / 批量记录实例化
```tablegen
let Predicates = [HasVendorXAndesPerf, IsRV64] in {
  defm : ADD_UWPat<NDS_LEA_B_ZE>;

  defm : ShxAdd_UWPat<1, NDS_LEA_H_ZE>;
  defm : ShxAdd_UWPat<2, NDS_LEA_W_ZE>;
  defm : ShxAdd_UWPat<3, NDS_LEA_D_ZE>;

  defm : Sh1Add_UWPat<NDS_LEA_H_ZE>;
  defm : Sh2Add_UWPat<NDS_LEA_W_ZE>;
  defm : Sh3Add_UWPat<NDS_LEA_D_ZE>;

  def : Sh1AddPat<NDS_LEA_H>;
  def : Sh2AddPat<NDS_LEA_W>;
  def : Sh3AddPat<NDS_LEA_D>;
} // Predicates = [HasVendorXAndesPerf, IsRV64]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 846-862: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasVendorXAndesPerf] in {
  def : Pat<(XLenVT (int_riscv_nds_ffb (XLenVT GPR:$rs1), (XLenVT GPR:$rs2))),
            (NDS_FFB GPR:$rs1, GPR:$rs2)>;
  def : Pat<(XLenVT (int_riscv_nds_ffzmism (XLenVT GPR:$rs1), (XLenVT GPR:$rs2))),
            (NDS_FFZMISM GPR:$rs1, GPR:$rs2)>;
  def : Pat<(XLenVT (int_riscv_nds_ffmism (XLenVT GPR:$rs1), (XLenVT GPR:$rs2))),
            (NDS_FFMISM GPR:$rs1, GPR:$rs2)>;
  def : Pat<(XLenVT (int_riscv_nds_flmism (XLenVT GPR:$rs1), (XLenVT GPR:$rs2))),
            (NDS_FLMISM GPR:$rs1, GPR:$rs2)>;
} // Predicates = [HasVendorXAndesPerf]

let Predicates = [HasVendorXAndesBFHCvt] in {
  def : Pat<(fpextend (bf16 FPR16:$rs)),
            (NDS_FCVT_S_BF16 (bf16 FPR16:$rs))>;
  def : Pat<(bf16 (fpround FPR32:$rs)),
            (NDS_FCVT_BF16_S FPR32:$rs)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 863-880: TableGen record NDS_FMV_BF16_X / TableGen 记录 NDS_FMV_BF16_X
```tablegen
  let isCodeGenOnly = 1 in {
    def NDS_FMV_BF16_X : FPUnaryOp_r<0b1111000, 0b00000, 0b000, FPR16, GPR, "fmv.w.x">,
                         Sched<[WriteFMovI32ToF32, ReadFMovI32ToF32]>;
    def NDS_FMV_X_BF16 : FPUnaryOp_r<0b1110000, 0b00000, 0b000, GPR, FPR16, "fmv.x.w">,
                         Sched<[WriteFMovF32ToI32, ReadFMovF32ToI32]>;
  }

  def : Pat<(riscv_nds_fmv_bf16_x GPR:$src), (NDS_FMV_BF16_X GPR:$src)>;
  def : Pat<(riscv_nds_fmv_x_anyextbf16 (bf16 FPR16:$src)),
            (NDS_FMV_X_BF16 (bf16 FPR16:$src))>;
} // Predicates = [HasVendorXAndesBFHCvt]

// Use flh/fsh to load/store bf16 if zfh is enabled.
let Predicates = [HasStdExtZfh, HasVendorXAndesBFHCvt] in {
  def : LdPat<load, FLH, bf16>;
  def : StPat<store, FSH, FPR16, bf16>;
} // Predicates = [HasStdExtZfh, HasVendorXAndesBFHCvt]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 881-897: Bulk record instantiation PseudoNDS_VFWCVT_S_BF16 / 批量记录实例化 PseudoNDS_VFWCVT_S_BF16
```tablegen
let Predicates = [HasVendorXAndesVBFHCvt] in {
  defm PseudoNDS_VFWCVT_S_BF16 : VPseudoVWCVT_S_BF16;
  defm PseudoNDS_VFNCVT_BF16_S : VPseudoVNCVT_BF16_S;
} // Predicates = [HasVendorXAndesVBFHCvt]

let Predicates = [HasVendorXAndesVBFHCvt] in {
  defm : VPatConversionS_BF16<"int_riscv_nds_vfwcvt_s_bf16",
                              "PseudoNDS_VFWCVT_S">;
  defm : VPatConversionBF16_S<"int_riscv_nds_vfncvt_bf16_s",
                              "PseudoNDS_VFNCVT_BF16">;
} // Predicates = [HasVendorXAndesVBFHCvt]

let Predicates = [HasVendorXAndesVSIntLoad] in {
  defm PseudoNDS_VLN8  : VPseudoVLN8<0>;
  defm PseudoNDS_VLNU8 : VPseudoVLN8<1>;
} // Predicates = [HasVendorXAndesVSIntLoad]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 898-916: Bulk record instantiation PseudoNDS_VFPMADT / 批量记录实例化 PseudoNDS_VFPMADT
```tablegen
let Predicates = [HasVendorXAndesVPackFPH],
    mayRaiseFPException = true in {
  defm PseudoNDS_VFPMADT : VPseudoVFPMAD_VF_RM;
  defm PseudoNDS_VFPMADB : VPseudoVFPMAD_VF_RM;
} // Predicates = [HasVendorXAndesVPackFPH]

let Predicates = [HasVendorXAndesVPackFPH] in {
  defm : VPatVFPMADBinaryV_VX_RM<"int_riscv_nds_vfpmadt", "PseudoNDS_VFPMADT",
                                 AllFP16Vectors>;
  defm : VPatVFPMADBinaryV_VX_RM<"int_riscv_nds_vfpmadb", "PseudoNDS_VFPMADB",
                                 AllFP16Vectors>;
} // Predicates = [HasVendorXAndesVPackFPH]

let Predicates = [HasVendorXAndesVDot] in {
  defm PseudoNDS_VD4DOTS  : VPseudoVD4DOT_VV;
  defm PseudoNDS_VD4DOTU  : VPseudoVD4DOT_VV;
  defm PseudoNDS_VD4DOTSU : VPseudoVD4DOT_VV;
} // Predicates = [HasVendorXAndesVDot]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 917-937: Bulk record instantiation / 批量记录实例化
```tablegen
defset list<VTypeInfoToWide> AllQuadWidenableVD4DOTVectors = {
  def : VTypeInfoToWide<VI8MF2,  VI32MF2>;
  def : VTypeInfoToWide<VI8M1,   VI32M1>;
  def : VTypeInfoToWide<VI8M2,   VI32M2>;
  def : VTypeInfoToWide<VI8M4,   VI32M4>;
  def : VTypeInfoToWide<VI8M8,   VI32M8>;
  def : VTypeInfoToWide<VI16M1,  VI64M1>;
  def : VTypeInfoToWide<VI16M2,  VI64M2>;
  def : VTypeInfoToWide<VI16M4,  VI64M4>;
  def : VTypeInfoToWide<VI16M8,  VI64M8>;
}

let Predicates = [HasVendorXAndesVDot] in {
  defm : VPatTernaryVD4DOT_VV<"int_riscv_nds_vd4dots", "PseudoNDS_VD4DOTS",
                              AllQuadWidenableVD4DOTVectors>;
  defm : VPatTernaryVD4DOT_VV<"int_riscv_nds_vd4dotu", "PseudoNDS_VD4DOTU",
                              AllQuadWidenableVD4DOTVectors>;
  defm : VPatTernaryVD4DOT_VV<"int_riscv_nds_vd4dotsu", "PseudoNDS_VD4DOTSU",
                              AllQuadWidenableVD4DOTVectors>;
} // Predicates = [HasVendorXAndesVDot]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 938-953: TableGen class SFBNDS_BFO / TableGen 类 SFBNDS_BFO
```tablegen
//===----------------------------------------------------------------------===//
// Pseudo-instructions for SFB (Short Forward Branch)
//===----------------------------------------------------------------------===//

class SFBNDS_BFO
    : Pseudo<(outs GPR:$dst),
             (ins GPR:$falsev, GPR:$rs1, uimmlog2xlen:$msb, uimmlog2xlen:$lsb,
                  bcc_opcode:$bcc, GPR:$lhs, sfb_rhs:$rhs), []>,
      Sched<[WriteSFB, ReadSFBALU, ReadSFBALU,
             ReadSFBALU, ReadSFBJmp, ReadSFBJmp]> {
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
  let Size = 8;
  let Constraints = "$dst = $falsev";
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 954-958: TableGen record PseudoCCNDS_BFOS / TableGen 记录 PseudoCCNDS_BFOS
```tablegen

let Predicates = [HasShortForwardBranchIALU] in {
  def PseudoCCNDS_BFOS : SFBNDS_BFO;
  def PseudoCCNDS_BFOZ : SFBNDS_BFO;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
