# RISCVInstrInfoC.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoC.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: Commentary and design intent / 注释与设计意图
```tablegen
//===- RISCVInstrInfoC.td - Compressed RISC-V instructions -*- tblgen-*----===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
// Operand definitions.
//===----------------------------------------------------------------------===//

def UImmLog2XLenNonZeroAsmOperand : AsmOperandClass {
  let Name = "UImmLog2XLenNonZero";
  let RenderMethod = "addImmOperands";
  let DiagnosticType = "InvalidUImmLog2XLenNonZero";
}
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 18-35: TableGen record uimmlog2xlennonzero / TableGen 记录 uimmlog2xlennonzero
```tablegen

def uimmlog2xlennonzero : RISCVOp, ImmLeaf<XLenVT, [{
  if (Subtarget->is64Bit())
    return isUInt<6>(Imm) && (Imm != 0);
  return isUInt<5>(Imm) && (Imm != 0);
}]> {
  let ParserMatchClass = UImmLog2XLenNonZeroAsmOperand;
  let DecoderMethod = "decodeUImmLog2XLenNonZeroOperand";
  let OperandType = "OPERAND_UIMMLOG2XLEN_NONZERO";
  let MCOperandPredicate = [{
    int64_t Imm;
    if (!MCOp.evaluateAsConstantImm(Imm))
      return false;
    if (STI.getTargetTriple().isArch64Bit())
      return  isUInt<6>(Imm) && (Imm != 0);
    return isUInt<5>(Imm) && (Imm != 0);
  }];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 36-58: TableGen record simm6 / TableGen 记录 simm6
```tablegen

def simm6 : RISCVSImmLeafOp<6> {
  let MCOperandPredicate = [{
    int64_t Imm;
    if (MCOp.evaluateAsConstantImm(Imm))
      return isInt<6>(Imm);
    return MCOp.isBareSymbolRef();
  }];
}

def simm6nonzero : RISCVOp,
                   ImmLeaf<XLenVT, [{return (Imm != 0) && isInt<6>(Imm);}]> {
  let ParserMatchClass = SImmAsmOperand<6, "NonZero">;
  let EncoderMethod = "getImmOpValue";
  let DecoderMethod = "decodeSImmNonZeroOperand<6>";
  let OperandType = "OPERAND_SIMM6_NONZERO";
  let MCOperandPredicate = [{
    int64_t Imm;
    if (MCOp.evaluateAsConstantImm(Imm))
      return (Imm != 0) && isInt<6>(Imm);
    return MCOp.isBareSymbolRef();
  }];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 59-86: TableGen record CLUIImmAsmOperand / TableGen 记录 CLUIImmAsmOperand
```tablegen

def CLUIImmAsmOperand : AsmOperandClass {
  let Name = "CLUIImm";
  let RenderMethod = "addImmOperands";
  let DiagnosticType = !strconcat("Invalid", Name);
}


// c_lui_imm checks the immediate range is in [1, 31] or [0xfffe0, 0xfffff].
// The RISC-V ISA describes the constraint as [1, 63], with that value being
// loaded in to bits 17-12 of the destination register and sign extended from
// bit 17. Therefore, this 6-bit immediate can represent values in the ranges
// [1, 31] and [0xfffe0, 0xfffff].
def c_lui_imm : RISCVOp,
                ImmLeaf<XLenVT, [{return (Imm != 0) &&
                                 (isUInt<5>(Imm) ||
                                  (Imm >= 0xfffe0 && Imm <= 0xfffff));}]> {
  let ParserMatchClass = CLUIImmAsmOperand;
  let EncoderMethod = "getImmOpValue";
  let DecoderMethod = "decodeCLUIImmOperand";
  let OperandType = "OPERAND_CLUI_IMM";
  let MCOperandPredicate = [{
    int64_t Imm;
    if (!MCOp.evaluateAsConstantImm(Imm))
      return false;
    return (Imm != 0) && (isUInt<5>(Imm) ||
           (Imm >= 0xfffe0 && Imm <= 0xfffff));
  }];
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 87-101: TableGen class RISCVUImmLsbZeroOp<int / TableGen 类 RISCVUImmLsbZeroOp<int
```tablegen
}

// A N-bit unsigned immediate where the least significant bits are zero.
class RISCVUImmLsbZeroOp<int width, int zeros>
    : RISCVOp, ImmLeaf<XLenVT, [{return isShiftedUInt<}] # !sub(width, zeros) #
                       [{, }] # zeros # [{>(Imm);}]> {

  // Create suffix with the appropriate number of 0s to append to all names
  defvar Suffix = !foldl("", !range(zeros), acc, ignore, !strconcat(acc, "0"));

  let ParserMatchClass = UImmAsmOperand<width, "Lsb" # Suffix>;
  let EncoderMethod = "getImmOpValue";
  let DecoderMethod = "decodeUImmOperand<" # width # ">";
  let OperandType = "OPERAND_UIMM" # width # "_LSB" # Suffix;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 102-115: TableGen record uimm7_lsb00 / TableGen 记录 uimm7_lsb00
```tablegen
  let MCOperandPredicate = [{
    int64_t Imm;
    if (!MCOp.evaluateAsConstantImm(Imm))
      return false;
    return isShiftedUInt<}] # !sub(width, zeros) # [{, }] # zeros # [{>(Imm);
  }];
}

// A 7-bit unsigned immediate where the least significant two bits are zero.
def uimm7_lsb00 : RISCVUImmLsbZeroOp<7, 2>;

// A 8-bit unsigned immediate where the least significant two bits are zero.
def uimm8_lsb00 : RISCVUImmLsbZeroOp<8, 2>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 116-133: TableGen record uimm8_lsb000 / TableGen 记录 uimm8_lsb000
```tablegen
// A 8-bit unsigned immediate where the least significant three bits are zero.
def uimm8_lsb000 : RISCVUImmLsbZeroOp<8, 3>;

// A 9-bit signed immediate where the least significant bit is zero.
def bare_simm9_lsb0 : Operand<OtherVT>,
                      ImmLeaf<XLenVT, [{return isShiftedInt<8, 1>(Imm);}]> {
  let ParserMatchClass = BareSImmNLsb0AsmOperand<9>;
  let PrintMethod = "printBranchOperand";
  let EncoderMethod = "getImmOpValueAsrN<1>";
  let DecoderMethod = "decodeSImmOperandAndLslN<9, 1>";
  let MCOperandPredicate = [{
    int64_t Imm;
    if (MCOp.evaluateAsConstantImm(Imm))
      return isShiftedInt<8, 1>(Imm);
    return MCOp.isBareSymbolRef();
  }];
  let OperandType = "OPERAND_PCREL";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 134-153: TableGen record uimm9_lsb000 / TableGen 记录 uimm9_lsb000
```tablegen

// A 9-bit unsigned immediate where the least significant three bits are zero.
def uimm9_lsb000 : RISCVUImmLsbZeroOp<9, 3>;

// A 10-bit unsigned immediate where the least significant two bits are zero
// and the immediate can't be zero.
def uimm10_lsb00nonzero : RISCVOp,
                          ImmLeaf<XLenVT,
                          [{return isShiftedUInt<8, 2>(Imm) && (Imm != 0);}]> {
  let ParserMatchClass = UImmAsmOperand<10, "Lsb00NonZero">;
  let EncoderMethod = "getImmOpValue";
  let DecoderMethod = "decodeUImmNonZeroOperand<10>";
  let OperandType = "OPERAND_UIMM10_LSB00_NONZERO";
  let MCOperandPredicate = [{
    int64_t Imm;
    if (!MCOp.evaluateAsConstantImm(Imm))
      return false;
    return isShiftedUInt<8, 2>(Imm) && (Imm != 0);
  }];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 154-169: TableGen record simm10_lsb0000nonzero / TableGen 记录 simm10_lsb0000nonzero
```tablegen

// A 10-bit signed immediate where the least significant four bits are zero.
def simm10_lsb0000nonzero : RISCVOp,
                            ImmLeaf<XLenVT,
                            [{return (Imm != 0) && isShiftedInt<6, 4>(Imm);}]> {
  let ParserMatchClass = SImmAsmOperand<10, "Lsb0000NonZero">;
  let EncoderMethod = "getImmOpValue";
  let DecoderMethod = "decodeSImmNonZeroOperand<10>";
  let OperandType = "OPERAND_SIMM10_LSB0000_NONZERO";
  let MCOperandPredicate = [{
    int64_t Imm;
    if (!MCOp.evaluateAsConstantImm(Imm))
      return false;
    return isShiftedInt<6, 4>(Imm) && (Imm != 0);
  }];
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 170-185: TableGen record bare_simm12_lsb0 / TableGen 记录 bare_simm12_lsb0
```tablegen

// A 12-bit signed immediate where the least significant bit is zero.
def bare_simm12_lsb0 : Operand<OtherVT>,
                       ImmLeaf<XLenVT, [{return isShiftedInt<11, 1>(Imm);}]> {
  let ParserMatchClass = BareSImmNLsb0AsmOperand<12>;
  let PrintMethod = "printBranchOperand";
  let EncoderMethod = "getImmOpValueAsrN<1>";
  let DecoderMethod = "decodeSImmOperandAndLslN<12, 1>";
  let MCOperandPredicate = [{
    int64_t Imm;
    if (MCOp.evaluateAsConstantImm(Imm))
      return isShiftedInt<11, 1>(Imm);
    return MCOp.isBareSymbolRef();
  }];
  let OperandType = "OPERAND_PCREL";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 186-200: TableGen record InsnCDirectiveOpcode / TableGen 记录 InsnCDirectiveOpcode
```tablegen

def InsnCDirectiveOpcode : AsmOperandClass {
  let Name = "InsnCDirectiveOpcode";
  let ParserMethod = "parseInsnCDirectiveOpcode";
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isImm";
}

def uimm2_opcode : RISCVUImmOp<2> {
  let ParserMatchClass = InsnCDirectiveOpcode;
}

//===----------------------------------------------------------------------===//
// Instruction Class Templates
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 201-216: TableGen class CStackLoad<bits<3> / TableGen 类 CStackLoad<bits<3>
```tablegen

let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
class CStackLoad<bits<3> funct3, string OpcodeStr,
                 DAGOperand cls, DAGOperand opnd>
    : RVInst16CI<funct3, OPC_C2, (outs cls:$rd), (ins SPMem:$rs1, opnd:$imm),
                 OpcodeStr, "$rd, ${imm}(${rs1})"> {
  bits<0> rs1;
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
class CStackStore<bits<3> funct3, string OpcodeStr,
                  DAGOperand cls, DAGOperand opnd>
    : RVInst16CSS<funct3, OPC_C2, (outs), (ins cls:$rs2, SPMem:$rs1, opnd:$imm),
                  OpcodeStr, "$rs2, ${imm}(${rs1})"> {
  bits<0> rs1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 217-241: TableGen class CLoad_ri<bits<3> / TableGen 类 CLoad_ri<bits<3>
```tablegen

let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
class CLoad_ri<bits<3> funct3, string OpcodeStr,
               DAGOperand cls, DAGOperand opnd>
    : RVInst16CL<funct3, OPC_C0, (outs cls:$rd), (ins GPRCMem:$rs1, opnd:$imm),
                 OpcodeStr, "$rd, ${imm}(${rs1})">;

let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
class CStore_rri<bits<3> funct3, string OpcodeStr,
                 DAGOperand cls, DAGOperand opnd>
    : RVInst16CS<funct3, OPC_C0, (outs), (ins cls:$rs2,GPRCMem:$rs1, opnd:$imm),
                 OpcodeStr, "$rs2, ${imm}(${rs1})">;

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class Bcz<bits<3> funct3, string OpcodeStr>
    : RVInst16CB<funct3, OPC_C1, (outs), (ins GPRC:$rs1, bare_simm9_lsb0:$imm),
                 OpcodeStr, "$rs1, $imm"> {
  let isBranch = 1;
  let isTerminator = 1;
  let Inst{12} = imm{7};
  let Inst{11-10} = imm{3-2};
  let Inst{6-5} = imm{6-5};
  let Inst{4-3} = imm{1-0};
  let Inst{2} = imm{4};
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 242-261: TableGen class Shift_right<bits<2> / TableGen 类 Shift_right<bits<2>
```tablegen

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class Shift_right<bits<2> funct2, string OpcodeStr>
    : RVInst16CB<0b100, OPC_C1, (outs GPRC:$rd),
                 (ins GPRC:$rs1, uimmlog2xlen:$imm),
                 OpcodeStr, "$rs1, $imm"> {
  let Constraints = "$rs1 = $rd";
  let Inst{12} = imm{5};
  let Inst{11-10} = funct2;
  let Inst{6-2} = imm{4-0};
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class CA_ALU<bits<6> funct6, bits<2> funct2, string OpcodeStr>
    : RVInst16CA<funct6, funct2, OPC_C1, (outs GPRC:$rd_wb),
                 (ins GPRC:$rd, GPRC:$rs2), OpcodeStr, "$rd, $rs2"> {
  bits<3> rd;
  let Constraints = "$rd = $rd_wb";
  let Inst{9-7} = rd;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 262-279: TableGen record C_ADDI4SPN / TableGen 记录 C_ADDI4SPN
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZca] in {

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
def C_ADDI4SPN : RVInst16CIW<0b000, OPC_C0, (outs GPRC:$rd),
                             (ins SP:$rs1, uimm10_lsb00nonzero:$imm),
                             "c.addi4spn", "$rd, $rs1, $imm">,
                             Sched<[WriteIALU, ReadIALU]> {
  bits<0> rs1;
  let Inst{12-11} = imm{5-4};
  let Inst{10-7} = imm{9-6};
  let Inst{6} = imm{2};
  let Inst{5} = imm{3};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 280-296: TableGen record C_LW / TableGen 记录 C_LW
```tablegen

def C_LW : CLoad_ri<0b010, "c.lw", GPRC, uimm7_lsb00>,
           Sched<[WriteLDW, ReadMemBase]> {
  bits<7> imm;
  let Inst{12-10} = imm{5-3};
  let Inst{6} = imm{2};
  let Inst{5} = imm{6};
}

let isCodeGenOnly = 1 in
def C_LW_INX : CLoad_ri<0b010, "c.lw", GPRF32C, uimm7_lsb00>,
               Sched<[WriteLDW, ReadMemBase]> {
  bits<7> imm;
  let Inst{12-10} = imm{5-3};
  let Inst{6} = imm{2};
  let Inst{5} = imm{6};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 297-312: TableGen record C_LD / TableGen 记录 C_LD
```tablegen

let Predicates = [HasStdExtZca, IsRV64] in
def C_LD : CLoad_ri<0b011, "c.ld", GPRC, uimm8_lsb000>,
           Sched<[WriteLDD, ReadMemBase]> {
  bits<8> imm;
  let Inst{12-10} = imm{5-3};
  let Inst{6-5} = imm{7-6};
}

def C_SW : CStore_rri<0b110, "c.sw", GPRC, uimm7_lsb00>,
           Sched<[WriteSTW, ReadStoreData, ReadMemBase]> {
  bits<7> imm;
  let Inst{12-10} = imm{5-3};
  let Inst{6} = imm{2};
  let Inst{5} = imm{6};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 313-329: TableGen record C_SW_INX / TableGen 记录 C_SW_INX
```tablegen

let isCodeGenOnly = 1 in
def C_SW_INX : CStore_rri<0b110, "c.sw", GPRF32C, uimm7_lsb00>,
               Sched<[WriteSTW, ReadStoreData, ReadMemBase]> {
  bits<7> imm;
  let Inst{12-10} = imm{5-3};
  let Inst{6} = imm{2};
  let Inst{5} = imm{6};
}

let Predicates = [HasStdExtZca, IsRV64] in
def C_SD : CStore_rri<0b111, "c.sd", GPRC, uimm8_lsb000>,
           Sched<[WriteSTD, ReadStoreData, ReadMemBase]> {
  bits<8> imm;
  let Inst{12-10} = imm{5-3};
  let Inst{6-5} = imm{7-6};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 330-344: TableGen record C_NOP / TableGen 记录 C_NOP
```tablegen

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
def C_NOP : RVInst16CI<0b000, OPC_C1, (outs), (ins), "c.nop", "">,
            Sched<[WriteNop]> {
  let rd = 0;
  let imm = 0;
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
def C_ADDI : RVInst16CI<0b000, OPC_C1, (outs GPRNoX0:$rd_wb),
                        (ins GPRNoX0:$rd, simm6:$imm),
                        "c.addi", "$rd, $imm">,
             Sched<[WriteIALU, ReadIALU]> {
  let Constraints = "$rd = $rd_wb";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 345-358: TableGen record PseudoC_ADDI_NOP / TableGen 记录 PseudoC_ADDI_NOP
```tablegen

// Alternate syntax for c.nop. Converted to C_NOP/C_NOP_HINT by the assembler.
let hasSideEffects = 0, mayLoad = 0, mayStore = 0, isCodeGenOnly = 0 in
def PseudoC_ADDI_NOP : Pseudo<(outs GPRX0:$rd), (ins GPRX0:$rs1, simm6:$imm),
                              [], "c.addi", "$rd, $imm"> {
  let Constraints = "$rs1 = $rd";
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 0, isCall = 1,
    DecoderNamespace = "RV32Only", Defs = [X1],
    Predicates = [HasStdExtZca, IsRV32]  in
def C_JAL : RVInst16CJ<0b001, OPC_C1, (outs), (ins bare_simm12_lsb0:$offset),
                       "c.jal", "$offset">, Sched<[WriteJal]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 359-372: TableGen record C_ADDIW / TableGen 记录 C_ADDIW
```tablegen
let hasSideEffects = 0, mayLoad = 0, mayStore = 0,
    Predicates = [HasStdExtZca, IsRV64] in
def C_ADDIW : RVInst16CI<0b001, OPC_C1, (outs GPRNoX0:$rd_wb),
                         (ins GPRNoX0:$rd, simm6:$imm),
                         "c.addiw", "$rd, $imm">,
              Sched<[WriteIALU32, ReadIALU32]> {
  let Constraints = "$rd = $rd_wb";
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
def C_LI : RVInst16CI<0b010, OPC_C1, (outs GPR:$rd), (ins simm6:$imm),
                      "c.li", "$rd, $imm">,
           Sched<[WriteIALU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 373-386: TableGen record C_ADDI16SP / TableGen 记录 C_ADDI16SP
```tablegen
let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
def C_ADDI16SP : RVInst16CI<0b011, OPC_C1, (outs SP:$rd_wb),
                            (ins SP:$rd, simm10_lsb0000nonzero:$imm),
                            "c.addi16sp", "$rd, $imm">,
                 Sched<[WriteIALU, ReadIALU]> {
  let Constraints = "$rd = $rd_wb";
  let rd = 2;
  let Inst{12} = imm{9};
  let Inst{6} = imm{4};
  let Inst{5} = imm{6};
  let Inst{4-3} = imm{8-7};
  let Inst{2} = imm{5};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 387-406: TableGen record C_LUI / TableGen 记录 C_LUI
```tablegen
let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
def C_LUI : RVInst16CI<0b011, OPC_C1, (outs GPRNoX2:$rd),
                       (ins c_lui_imm:$imm),
                       "c.lui", "$rd, $imm">,
            Sched<[WriteIALU]>;

def C_SRLI : Shift_right<0b00, "c.srli">,
             Sched<[WriteShiftImm, ReadShiftImm]>;
def C_SRAI : Shift_right<0b01, "c.srai">,
             Sched<[WriteShiftImm, ReadShiftImm]>;

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
def C_ANDI : RVInst16CB<0b100, OPC_C1, (outs GPRC:$rd), (ins GPRC:$rs1, simm6:$imm),
                        "c.andi", "$rs1, $imm">,
             Sched<[WriteIALU, ReadIALU]> {
  let Constraints = "$rs1 = $rd";
  let Inst{12} = imm{5};
  let Inst{11-10} = 0b10;
  let Inst{6-2} = imm{4-0};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 407-422: TableGen record C_SUB / TableGen 记录 C_SUB
```tablegen

def C_SUB  : CA_ALU<0b100011, 0b00, "c.sub">,
             Sched<[WriteIALU, ReadIALU, ReadIALU]>;
def C_XOR  : CA_ALU<0b100011, 0b01, "c.xor">,
             Sched<[WriteIALU, ReadIALU, ReadIALU]>;
def C_OR   : CA_ALU<0b100011, 0b10, "c.or">,
             Sched<[WriteIALU, ReadIALU, ReadIALU]>;
def C_AND  : CA_ALU<0b100011, 0b11, "c.and">,
             Sched<[WriteIALU, ReadIALU, ReadIALU]>;

let Predicates = [HasStdExtZca, IsRV64] in {
def C_SUBW : CA_ALU<0b100111, 0b00, "c.subw">,
             Sched<[WriteIALU32, ReadIALU32, ReadIALU32]>;
def C_ADDW : CA_ALU<0b100111, 0b01, "c.addw">,
             Sched<[WriteIALU32, ReadIALU32, ReadIALU32]>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 423-441: TableGen record C_J / TableGen 记录 C_J
```tablegen

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
def C_J : RVInst16CJ<0b101, OPC_C1, (outs), (ins bare_simm12_lsb0:$offset),
                     "c.j", "$offset">, Sched<[WriteJmp]> {
  let isBranch = 1;
  let isTerminator=1;
  let isBarrier=1;
}

def C_BEQZ : Bcz<0b110, "c.beqz">, Sched<[WriteJmp, ReadJmp]>;
def C_BNEZ : Bcz<0b111, "c.bnez">, Sched<[WriteJmp, ReadJmp]>;

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
def C_SLLI : RVInst16CI<0b000, OPC_C2, (outs GPR:$rd_wb),
                        (ins GPR:$rd, uimmlog2xlen:$imm),
                        "c.slli", "$rd, $imm">,
             Sched<[WriteShiftImm, ReadShiftImm]> {
  let Constraints = "$rd = $rd_wb";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 442-458: TableGen record C_LWSP / TableGen 记录 C_LWSP
```tablegen

def C_LWSP : CStackLoad<0b010, "c.lwsp", GPRNoX0, uimm8_lsb00>,
             Sched<[WriteLDW, ReadMemBase]> {
  let Inst{3-2} = imm{7-6};
}

let isCodeGenOnly = 1 in
def C_LWSP_INX : CStackLoad<0b010, "c.lwsp", GPRF32NoX0, uimm8_lsb00>,
                 Sched<[WriteLDW, ReadMemBase]> {
  let Inst{3-2} = imm{7-6};
}

let Predicates = [HasStdExtZca, IsRV64] in
def C_LDSP : CStackLoad<0b011, "c.ldsp", GPRNoX0, uimm9_lsb000>,
             Sched<[WriteLDD, ReadMemBase]> {
  let Inst{4-2} = imm{8-6};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 459-473: TableGen record C_JR / TableGen 记录 C_JR
```tablegen

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
def C_JR : RVInst16CR<0b1000, OPC_C2, (outs), (ins GPRNoX0:$rs1),
                      "c.jr", "$rs1">, Sched<[WriteJalr, ReadJalr]> {
  let isBarrier = 1;
  let isTerminator = 1;
  let rs2 = 0;
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 0, isMoveReg = 1,
    isAsCheapAsAMove = 1 in
def C_MV : RVInst16CR<0b1000, OPC_C2, (outs GPR:$rs1), (ins GPRNoX0:$rs2),
                      "c.mv", "$rs1, $rs2">,
           Sched<[WriteIALU, ReadIALU]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 474-488: TableGen record C_EBREAK / TableGen 记录 C_EBREAK
```tablegen
let rs1 = 0, rs2 = 0, hasSideEffects = 1, mayLoad = 0, mayStore = 0 in
def C_EBREAK : RVInst16CR<0b1001, OPC_C2, (outs), (ins), "c.ebreak", "">, Sched<[]>;

let hasSideEffects = 0, mayLoad = 0, mayStore = 0,
    isCall=1, Defs=[X1], rs2 = 0 in
def C_JALR : RVInst16CR<0b1001, OPC_C2, (outs), (ins GPRNoX0:$rs1),
                        "c.jalr", "$rs1">, Sched<[WriteJalr, ReadJalr]>;

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
def C_ADD : RVInst16CR<0b1001, OPC_C2, (outs GPR:$rd),
                       (ins GPR:$rs1, GPRNoX0:$rs2),
                       "c.add", "$rs1, $rs2">,
            Sched<[WriteIALU, ReadIALU, ReadIALU]> {
  let Constraints = "$rs1 = $rd";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 489-505: TableGen record C_SWSP / TableGen 记录 C_SWSP
```tablegen

def C_SWSP : CStackStore<0b110, "c.swsp", GPR, uimm8_lsb00>,
             Sched<[WriteSTW, ReadStoreData, ReadMemBase]> {
  let Inst{8-7}  = imm{7-6};
}

let isCodeGenOnly = 1 in
def C_SWSP_INX : CStackStore<0b110, "c.swsp", GPRF32, uimm8_lsb00>,
                 Sched<[WriteSTW, ReadStoreData, ReadMemBase]> {
  let Inst{8-7}  = imm{7-6};
}

let Predicates = [HasStdExtZca, IsRV64] in
def C_SDSP : CStackStore<0b111, "c.sdsp", GPR, uimm9_lsb000>,
             Sched<[WriteSTD, ReadStoreData, ReadMemBase]> {
  let Inst{9-7}   = imm{8-6};
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 506-525: TableGen record C_UNIMP / TableGen 记录 C_UNIMP
```tablegen

// The all zeros pattern isn't a valid RISC-V instruction. It's used by GNU
// binutils as 16-bit instruction known to be unimplemented (i.e., trapping).
let hasSideEffects = 1, mayLoad = 0, mayStore = 0, isTrap = 1 in
def C_UNIMP : RVInst16<(outs), (ins), "c.unimp", "", [], InstFormatOther>,
              Sched<[]> {
  let Inst{15-0} = 0;
}

} // Predicates = [HasStdExtZca]

let DecoderNamespace = "RV32Only",
    Predicates = [HasStdExtZcf, IsRV32] in {
  def C_FLW  : CLoad_ri<0b011, "c.flw", FPR32C, uimm7_lsb00>,
               Sched<[WriteFLD32, ReadFMemBase]> {
    bits<7> imm;
    let Inst{12-10} = imm{5-3};
    let Inst{6} = imm{2};
    let Inst{5} = imm{6};
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 526-539: TableGen record C_FSW / TableGen 记录 C_FSW
```tablegen

  def C_FSW  : CStore_rri<0b111, "c.fsw", FPR32C, uimm7_lsb00>,
               Sched<[WriteFST32, ReadFStoreData, ReadFMemBase]> {
    bits<7> imm;
    let Inst{12-10} = imm{5-3};
    let Inst{6} = imm{2};
    let Inst{5} = imm{6};
  }

  def C_FLWSP  : CStackLoad<0b011, "c.flwsp", FPR32, uimm8_lsb00>,
                 Sched<[WriteFLD32, ReadFMemBase]> {
    let Inst{3-2} = imm{7-6};
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 540-553: TableGen record C_FSWSP / TableGen 记录 C_FSWSP
```tablegen
  def C_FSWSP  : CStackStore<0b111, "c.fswsp", FPR32, uimm8_lsb00>,
                 Sched<[WriteFST32, ReadFStoreData, ReadFMemBase]> {
    let Inst{8-7}  = imm{7-6};
  }
} // DecoderNamespace = "RV32Only", Predicates = [HasStdExtZcf, IsRV32]

let Predicates = [HasStdExtZcd] in {
  def C_FLD  : CLoad_ri<0b001, "c.fld", FPR64C, uimm8_lsb000>,
               Sched<[WriteFLD64, ReadFMemBase]> {
    bits<8> imm;
    let Inst{12-10} = imm{5-3};
    let Inst{6-5} = imm{7-6};
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 554-569: TableGen record C_FSD / TableGen 记录 C_FSD
```tablegen
  def C_FSD  : CStore_rri<0b101, "c.fsd", FPR64C, uimm8_lsb000>,
               Sched<[WriteFST64, ReadFStoreData, ReadFMemBase]> {
    bits<8> imm;
    let Inst{12-10} = imm{5-3};
    let Inst{6-5} = imm{7-6};
  }

  def C_FLDSP  : CStackLoad<0b001, "c.fldsp", FPR64, uimm9_lsb000>,
                 Sched<[WriteFLD64, ReadFMemBase]> {
    let Inst{4-2} = imm{8-6};
  }

  def C_FSDSP  : CStackStore<0b101, "c.fsdsp", FPR64, uimm9_lsb000>,
                 Sched<[WriteFST64, ReadFStoreData, ReadFMemBase]> {
    let Inst{9-7}   = imm{8-6};
  }
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 570-583: TableGen record C_NOP_HINT / TableGen 记录 C_NOP_HINT
```tablegen
} // Predicates = [HasStdExtZcd] in {

//===----------------------------------------------------------------------===//
// HINT Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZca], hasSideEffects = 0, mayLoad = 0,
    mayStore = 0 in {

def C_NOP_HINT : RVInst16CI<0b000, OPC_C1, (outs), (ins simm6nonzero:$imm),
                            "c.nop", "$imm">, Sched<[WriteNop]> {
  let rd = 0;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 584-597: TableGen record definition / TableGen 记录定义
```tablegen
} // Predicates = [HasStdExtZca], hasSideEffects = 0, mayLoad = 0,
  // mayStore = 0

//===----------------------------------------------------------------------===//
// Assembler Pseudo Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZca] in {
// Legacy aliases.
def : InstAlias<"c.slli64 $rd", (C_SLLI GPR:$rd, 0), 0>;
def : InstAlias<"c.srli64 $rs1", (C_SRLI GPRC:$rs1, 0), 0>;
def : InstAlias<"c.srai64 $rs1", (C_SRAI GPRC:$rs1, 0), 0>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 598-613: TableGen record definition / TableGen 记录定义
```tablegen
// c.ntl.* hints are always available when Zca is present, even without
// enabling Zihintntl, per psABI decision (riscv-non-isa/riscv-elf-psabi-doc#474).
let Predicates = [HasStdExtZca] in {
def : InstAlias<"c.ntl.p1", (C_ADD X0, X2)>;
def : InstAlias<"c.ntl.pall", (C_ADD X0, X3)>;
def : InstAlias<"c.ntl.s1", (C_ADD X0, X4)>;
def : InstAlias<"c.ntl.all", (C_ADD X0, X5)>;
} // Predicates = [HasStdExtZca]

let EmitPriority = 0 in {
let Predicates = [HasStdExtZca] in {
def : InstAlias<"c.lw $rd, (${rs1})", (C_LW GPRC:$rd, GPRCMem:$rs1, 0)>;
def : InstAlias<"c.sw $rs2, (${rs1})", (C_SW GPRC:$rs2, GPRCMem:$rs1, 0)>;
def : InstAlias<"c.lwsp $rd, (${rs1})", (C_LWSP GPRNoX0:$rd, SPMem:$rs1, 0)>;
def : InstAlias<"c.swsp $rs2, (${rs1})", (C_SWSP GPR:$rs2, SPMem:$rs1, 0)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 614-627: TableGen record definition / TableGen 记录定义
```tablegen

let Predicates = [HasStdExtZca, IsRV64] in {
def : InstAlias<"c.ld $rd, (${rs1})", (C_LD GPRC:$rd, GPRCMem:$rs1, 0)>;
def : InstAlias<"c.sd $rs2, (${rs1})", (C_SD GPRC:$rs2, GPRCMem:$rs1, 0)>;
def : InstAlias<"c.ldsp $rd, (${rs1})", (C_LDSP GPRNoX0:$rd, SPMem:$rs1, 0)>;
def : InstAlias<"c.sdsp $rs2, (${rs1})", (C_SDSP GPR:$rs2, SPMem:$rs1, 0)>;
}

let Predicates = [HasStdExtZcf, IsRV32] in {
def : InstAlias<"c.flw $rd, (${rs1})", (C_FLW FPR32C:$rd, GPRCMem:$rs1, 0)>;
def : InstAlias<"c.fsw $rs2, (${rs1})", (C_FSW FPR32C:$rs2, GPRCMem:$rs1, 0)>;
def : InstAlias<"c.flwsp $rd, (${rs1})", (C_FLWSP FPR32:$rd, SPMem:$rs1, 0)>;
def : InstAlias<"c.fswsp $rs2, (${rs1})", (C_FSWSP FPR32:$rs2, SPMem:$rs1, 0)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 628-645: TableGen record AnyRegCOperand / TableGen 记录 AnyRegCOperand
```tablegen

let Predicates = [HasStdExtZcd] in {
def : InstAlias<"c.fld $rd, (${rs1})", (C_FLD FPR64C:$rd, GPRCMem:$rs1, 0)>;
def : InstAlias<"c.fsd $rs2, (${rs1})", (C_FSD FPR64C:$rs2, GPRCMem:$rs1, 0)>;
def : InstAlias<"c.fldsp $rd, (${rs1})", (C_FLDSP FPR64:$rd, SPMem:$rs1, 0)>;
def : InstAlias<"c.fsdsp $rs2, (${rs1})", (C_FSDSP FPR64:$rs2, SPMem:$rs1, 0)>;
}
} // EmitPriority = 0

//===----------------------------------------------------------------------===//
// .insn directive instructions
//===----------------------------------------------------------------------===//

def AnyRegCOperand : AsmOperandClass {
  let Name = "AnyRegCOperand";
  let RenderMethod = "addRegOperands";
  let PredicateMethod = "isAnyRegC";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 646-673: TableGen record AnyRegC / TableGen 记录 AnyRegC
```tablegen

def AnyRegC : Operand<XLenVT> {
  let OperandType = "OPERAND_REGISTER";
  let ParserMatchClass = AnyRegCOperand;
}

// isCodeGenOnly = 1 to hide them from the tablegened assembly parser.
let isCodeGenOnly = 1, hasSideEffects = 1, mayLoad = 1, mayStore = 1,
    hasNoSchedulingInfo = 1, Predicates = [HasStdExtZca] in {
def InsnCR : DirectiveInsnCR<(outs AnyReg:$rd), (ins uimm2_opcode:$opcode,
                                                     uimm4:$funct4,
                                                     AnyReg:$rs2),
                             "$opcode, $funct4, $rd, $rs2">;
def InsnCI : DirectiveInsnCI<(outs AnyReg:$rd), (ins uimm2_opcode:$opcode,
                                                     uimm3:$funct3,
                                                     simm6:$imm6),
                             "$opcode, $funct3, $rd, $imm6">;
def InsnCIW : DirectiveInsnCIW<(outs AnyRegC:$rd), (ins uimm2_opcode:$opcode,
                                                        uimm3:$funct3,
                                                        uimm8:$imm8),
                               "$opcode, $funct3, $rd, $imm8">;
def InsnCSS : DirectiveInsnCSS<(outs), (ins uimm2_opcode:$opcode,
                                            uimm3:$funct3,
                                            AnyReg:$rs2,
                                            uimm6:$imm6),
                               "$opcode, $funct3, $rs2, $imm6">;
def InsnCL : DirectiveInsnCL<(outs AnyRegC:$rd), (ins uimm2_opcode:$opcode,
                                                      uimm3:$funct3,
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 674-696: TableGen record InsnCS / TableGen 记录 InsnCS
```tablegen
                                                      AnyRegC:$rs1,
                                                      uimm5:$imm5),
                             "$opcode, $funct3, $rd, ${imm5}(${rs1})">;
def InsnCS : DirectiveInsnCS<(outs), (ins uimm2_opcode:$opcode,
                                          uimm3:$funct3,
                                          AnyRegC:$rs2,
                                          AnyRegC:$rs1,
                                          uimm5:$imm5),
                             "$opcode, $funct3, $rs2, ${imm5}(${rs1})">;
def InsnCA : DirectiveInsnCA<(outs AnyRegC:$rd), (ins uimm2_opcode:$opcode,
                                                      uimm6:$funct6,
                                                      uimm2:$funct2,
                                                      AnyRegC:$rs2),
                             "$opcode, $funct6, $funct2, $rd, $rs2">;
def InsnCB : DirectiveInsnCB<(outs), (ins uimm2_opcode:$opcode, uimm3:$funct3,
                                          AnyRegC:$rs1,
                                          bare_simm9_lsb0:$imm8),
                             "$opcode, $funct3, $rs1, $imm8">;
def InsnCJ : DirectiveInsnCJ<(outs), (ins uimm2_opcode:$opcode,
                                          uimm3:$funct3,
                                          bare_simm12_lsb0:$imm11),
                             "$opcode, $funct3, $imm11">;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 697-724: TableGen record definition / TableGen 记录定义
```tablegen

// Use InstAliases to match these so that we can combine the insn and format
// into a mnemonic to use as the key for the tablegened asm matcher table. The
// parser will take care of creating these fake mnemonics and will only do it
// for known formats.
let EmitPriority = 0, Predicates = [HasStdExtZca] in {
def : InstAlias<".insn_cr $opcode, $funct4, $rd, $rs2",
                (InsnCR AnyReg:$rd, uimm2_opcode:$opcode, uimm4:$funct4,
                        AnyReg:$rs2)>;
def : InstAlias<".insn_ci $opcode, $funct3, $rd, $imm6",
                (InsnCI AnyReg:$rd, uimm2_opcode:$opcode, uimm3:$funct3,
                        simm6:$imm6)>;
def : InstAlias<".insn_ciw $opcode, $funct3, $rd, $imm8",
                (InsnCIW AnyRegC:$rd, uimm2_opcode:$opcode, uimm3:$funct3,
                         uimm8:$imm8)>;
def : InstAlias<".insn_css $opcode, $funct3, $rs2, $imm6",
                (InsnCSS uimm2_opcode:$opcode, uimm3:$funct3, AnyReg:$rs2,
                         uimm6:$imm6)>;
def : InstAlias<".insn_cl $opcode, $funct3, $rd, ${imm5}(${rs1})",
                (InsnCL AnyRegC:$rd, uimm2_opcode:$opcode, uimm3:$funct3,
                        AnyRegC:$rs1, uimm5:$imm5)>;
def : InstAlias<".insn_cl $opcode, $funct3, $rd, (${rs1})",
                (InsnCL AnyRegC:$rd, uimm2_opcode:$opcode, uimm3:$funct3,
                        AnyRegC:$rs1, 0)>;
def : InstAlias<".insn_cs $opcode, $funct3, $rs2, ${imm5}(${rs1})",
                (InsnCS uimm2_opcode:$opcode, uimm3:$funct3, AnyRegC:$rs2,
                        AnyRegC:$rs1, uimm5:$imm5)>;
def : InstAlias<".insn_cs $opcode, $funct3, $rs2, (${rs1})",
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 725-739: TableGen record definition / TableGen 记录定义
```tablegen
                (InsnCS uimm2_opcode:$opcode, uimm3:$funct3, AnyRegC:$rs2,
                        AnyRegC:$rs1, 0)>;
def : InstAlias<".insn_ca $opcode, $funct6, $funct2, $rd, $rs2",
                (InsnCA AnyRegC:$rd, uimm2_opcode:$opcode, uimm6:$funct6,
                        uimm2:$funct2, AnyRegC:$rs2)>;
def : InstAlias<".insn_cb $opcode, $funct3, $rs1, $imm8",
                (InsnCB uimm2_opcode:$opcode, uimm3:$funct3, AnyRegC:$rs1,
                        bare_simm9_lsb0:$imm8)>;
def : InstAlias<".insn_cj $opcode, $funct3, $imm11",
                (InsnCJ uimm2_opcode:$opcode, uimm3:$funct3, bare_simm12_lsb0:$imm11)>;
}

//===----------------------------------------------------------------------===/i
// Compress Instruction tablegen backend.
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 740-755: TableGen record definition / TableGen 记录定义
```tablegen

// Zca patterns are defined in the same order the compressed instructions appear
// under the "RVC Instruction Set Listings" section of the ISA manual.

// Zca Instructions

// Quadrant 0
let Predicates = [HasStdExtZca] in {
def : CompressPat<(ADDI GPRC:$rd, SP:$rs1, uimm10_lsb00nonzero:$imm),
                  (C_ADDI4SPN GPRC:$rd, SP:$rs1, uimm10_lsb00nonzero:$imm)>;
} // Predicates = [HasStdExtZca]

let Predicates = [HasStdExtZca] in {
def : CompressPat<(LW GPRC:$rd, GPRCMem:$rs1, uimm7_lsb00:$imm),
                  (C_LW GPRC:$rd, GPRCMem:$rs1, uimm7_lsb00:$imm)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 756-769: TableGen record definition / TableGen 记录定义
```tablegen
let isCompressOnly = true in
def : CompressPat<(LW_INX GPRF32C:$rd, GPRCMem:$rs1, uimm7_lsb00:$imm),
                  (C_LW_INX GPRF32C:$rd, GPRCMem:$rs1, uimm7_lsb00:$imm)>;
} // Predicates = [HasStdExtZca]

let Predicates = [HasStdExtZca, IsRV64] in {
def : CompressPat<(LD GPRC:$rd, GPRCMem:$rs1, uimm8_lsb000:$imm),
                  (C_LD GPRC:$rd, GPRCMem:$rs1, uimm8_lsb000:$imm)>;
} // Predicates = [HasStdExtZca, IsRV64]

let Predicates = [HasStdExtZca] in {
def : CompressPat<(SW GPRC:$rs2, GPRCMem:$rs1, uimm7_lsb00:$imm),
                  (C_SW GPRC:$rs2, GPRCMem:$rs1, uimm7_lsb00:$imm)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 770-786: TableGen record definition / TableGen 记录定义
```tablegen
let isCompressOnly = true in
def : CompressPat<(SW_INX GPRF32C:$rs2, GPRCMem:$rs1, uimm7_lsb00:$imm),
                  (C_SW_INX GPRF32C:$rs2, GPRCMem:$rs1, uimm7_lsb00:$imm)>;
} // Predicates = [HasStdExtZca]

let Predicates = [HasStdExtZca, IsRV64] in {
def : CompressPat<(SD GPRC:$rs2, GPRCMem:$rs1, uimm8_lsb000:$imm),
                  (C_SD GPRC:$rs2, GPRCMem:$rs1, uimm8_lsb000:$imm)>;
} // Predicates = [HasStdExtZca, IsRV64]

// Quadrant 1
let Predicates = [HasStdExtZca] in {
def : CompressPat<(ADDI X0, X0, 0), (C_NOP)>;
def : CompressPat<(ADDI GPRNoX0:$rs1, GPRNoX0:$rs1, simm6nonzero:$imm),
                  (C_ADDI GPRNoX0:$rs1, simm6nonzero:$imm)>;
} // Predicates = [HasStdExtZca]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 787-814: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZca, IsRV32] in {
def : CompressPat<(JAL X1, bare_simm12_lsb0:$offset),
                  (C_JAL bare_simm12_lsb0:$offset)>;
} // Predicates = [HasStdExtZca, IsRV32]

let Predicates = [HasStdExtZca, IsRV64] in {
def : CompressPat<(ADDIW GPRNoX0:$rs1, GPRNoX0:$rs1, simm6:$imm),
                  (C_ADDIW GPRNoX0:$rs1, simm6:$imm)>;
} // Predicates = [HasStdExtZca, IsRV64]

let Predicates = [HasStdExtZca] in {
def : CompressPat<(ADDI GPRNoX0:$rd, X0, simm6:$imm),
                  (C_LI GPRNoX0:$rd, simm6:$imm)>;
def : CompressPat<(ADDI X2, X2, simm10_lsb0000nonzero:$imm),
                  (C_ADDI16SP X2, simm10_lsb0000nonzero:$imm)>;
def : CompressPat<(LUI GPRNoX0X2:$rd, c_lui_imm:$imm),
                  (C_LUI GPRNoX0X2:$rd, c_lui_imm:$imm)>;
def : CompressPat<(SRLI GPRC:$rs1, GPRC:$rs1, uimmlog2xlennonzero:$imm),
                  (C_SRLI GPRC:$rs1, uimmlog2xlennonzero:$imm)>;
def : CompressPat<(SRAI GPRC:$rs1, GPRC:$rs1, uimmlog2xlennonzero:$imm),
                  (C_SRAI GPRC:$rs1, uimmlog2xlennonzero:$imm)>;
def : CompressPat<(ANDI GPRC:$rs1, GPRC:$rs1, simm6:$imm),
                  (C_ANDI GPRC:$rs1, simm6:$imm)>;
def : CompressPat<(SUB GPRC:$rs1, GPRC:$rs1, GPRC:$rs2),
                  (C_SUB GPRC:$rs1, GPRC:$rs2)>;
def : CompressPat<(XOR GPRC:$rs1, GPRC:$rs1, GPRC:$rs2),
                  (C_XOR GPRC:$rs1, GPRC:$rs2)>;
let isCompressOnly = true in
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 815-828: TableGen record definition / TableGen 记录定义
```tablegen
def : CompressPat<(XOR GPRC:$rs1, GPRC:$rs2, GPRC:$rs1),
                  (C_XOR GPRC:$rs1, GPRC:$rs2)>;
def : CompressPat<(OR GPRC:$rs1, GPRC:$rs1, GPRC:$rs2),
                  (C_OR GPRC:$rs1, GPRC:$rs2)>;
let isCompressOnly = true in
def : CompressPat<(OR GPRC:$rs1, GPRC:$rs2, GPRC:$rs1),
                  (C_OR GPRC:$rs1, GPRC:$rs2)>;
def : CompressPat<(AND GPRC:$rs1, GPRC:$rs1, GPRC:$rs2),
                  (C_AND GPRC:$rs1, GPRC:$rs2)>;
let isCompressOnly = true in
def : CompressPat<(AND GPRC:$rs1, GPRC:$rs2, GPRC:$rs1),
                  (C_AND GPRC:$rs1, GPRC:$rs2)>;
} // Predicates = [HasStdExtZca]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 829-856: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZca, IsRV64] in {
let isCompressOnly = true in
def : CompressPat<(ADDIW GPRNoX0:$rd, X0, simm6:$imm),
                  (C_LI GPRNoX0:$rd, simm6:$imm)>;
def : CompressPat<(SUBW GPRC:$rs1, GPRC:$rs1, GPRC:$rs2),
                  (C_SUBW GPRC:$rs1, GPRC:$rs2)>;
def : CompressPat<(ADDW GPRC:$rs1, GPRC:$rs1, GPRC:$rs2),
                   (C_ADDW GPRC:$rs1, GPRC:$rs2)>;
let isCompressOnly = true in
def : CompressPat<(ADDW GPRC:$rs1, GPRC:$rs2, GPRC:$rs1),
                   (C_ADDW GPRC:$rs1, GPRC:$rs2)>;
} // Predicates = [HasStdExtZca, IsRV64]

let Predicates = [HasStdExtZca] in {
def : CompressPat<(JAL X0, bare_simm12_lsb0:$offset),
                  (C_J bare_simm12_lsb0:$offset)>;
def : CompressPat<(BEQ GPRC:$rs1, X0, bare_simm9_lsb0:$imm),
                  (C_BEQZ GPRC:$rs1, bare_simm9_lsb0:$imm)>;
let isCompressOnly = true in
def : CompressPat<(BEQ X0, GPRC:$rs1, bare_simm9_lsb0:$imm),
                  (C_BEQZ GPRC:$rs1, bare_simm9_lsb0:$imm)>;
def : CompressPat<(BNE GPRC:$rs1, X0, bare_simm9_lsb0:$imm),
                  (C_BNEZ GPRC:$rs1, bare_simm9_lsb0:$imm)>;
let isCompressOnly = true in
def : CompressPat<(BNE X0, GPRC:$rs1, bare_simm9_lsb0:$imm),
                  (C_BNEZ GPRC:$rs1, bare_simm9_lsb0:$imm)>;
} // Predicates = [HasStdExtZca]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 857-871: TableGen record definition / TableGen 记录定义
```tablegen
// Quadrant 2
let Predicates = [HasStdExtZca] in {
def : CompressPat<(SLLI GPRNoX0:$rs1, GPRNoX0:$rs1, uimmlog2xlennonzero:$imm),
                  (C_SLLI GPRNoX0:$rs1, uimmlog2xlennonzero:$imm)>;
} // Predicates = [HasStdExtZca]

let Predicates = [HasStdExtZca] in {
def : CompressPat<(LW GPRNoX0:$rd, SPMem:$rs1,  uimm8_lsb00:$imm),
                  (C_LWSP GPRNoX0:$rd, SPMem:$rs1, uimm8_lsb00:$imm)>;

let isCompressOnly = true in
def : CompressPat<(LW_INX GPRF32NoX0:$rd, SPMem:$rs1,  uimm8_lsb00:$imm),
                  (C_LWSP_INX GPRF32NoX0:$rd, SPMem:$rs1, uimm8_lsb00:$imm)>;
} // Predicates = [HasStdExtZca]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 872-885: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZca, IsRV64] in {
def : CompressPat<(LD GPRNoX0:$rd, SPMem:$rs1, uimm9_lsb000:$imm),
                  (C_LDSP GPRNoX0:$rd, SPMem:$rs1, uimm9_lsb000:$imm)>;
} // Predicates = [HasStdExtZca, IsRV64]

let Predicates = [HasStdExtZca] in {
def : CompressPat<(JALR X0, GPRNoX0:$rs1, 0),
                  (C_JR GPRNoX0:$rs1)>;
let isCompressOnly = true in {
def : CompressPat<(ADD GPRNoX0:$rs1, X0, GPRNoX0:$rs2),
                  (C_MV GPRNoX0:$rs1, GPRNoX0:$rs2)>;
def : CompressPat<(ADD GPRNoX0:$rs1, GPRNoX0:$rs2, X0),
                  (C_MV GPRNoX0:$rs1, GPRNoX0:$rs2)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 886-902: TableGen record definition / TableGen 记录定义
```tablegen
def : CompressPat<(ADDI GPRNoX0:$rs1, GPRNoX0:$rs2, 0),
                  (C_MV GPRNoX0:$rs1, GPRNoX0:$rs2)>;
def : CompressPat<(EBREAK), (C_EBREAK)>;
def : CompressPat<(UNIMP), (C_UNIMP)>;
def : CompressPat<(JALR X1, GPRNoX0:$rs1, 0),
                  (C_JALR GPRNoX0:$rs1)>;
def : CompressPat<(ADD GPRNoX0:$rs1, GPRNoX0:$rs1, GPRNoX0:$rs2),
                  (C_ADD GPRNoX0:$rs1, GPRNoX0:$rs2)>;
let isCompressOnly = true in
def : CompressPat<(ADD GPRNoX0:$rs1, GPRNoX0:$rs2, GPRNoX0:$rs1),
                  (C_ADD GPRNoX0:$rs1, GPRNoX0:$rs2)>;
} // Predicates = [HasStdExtZca]

let Predicates = [HasStdExtZca] in {
def : CompressPat<(SW GPR:$rs2, SPMem:$rs1, uimm8_lsb00:$imm),
                  (C_SWSP GPR:$rs2, SPMem:$rs1, uimm8_lsb00:$imm)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 903-920: TableGen record definition / TableGen 记录定义
```tablegen
let isCompressOnly = true in
def : CompressPat<(SW_INX GPRF32:$rs2, SPMem:$rs1, uimm8_lsb00:$imm),
                  (C_SWSP_INX GPRF32:$rs2, SPMem:$rs1, uimm8_lsb00:$imm)>;
} // Predicates = [HasStdExtZca]

let Predicates = [HasStdExtZca, IsRV64] in {
def : CompressPat<(SD GPR:$rs2, SPMem:$rs1, uimm9_lsb000:$imm),
                  (C_SDSP GPR:$rs2, SPMem:$rs1, uimm9_lsb000:$imm)>;
} // Predicates = [HasStdExtZca, IsRV64]

// Zcf Instructions
let Predicates = [HasStdExtZcf, IsRV32] in {
  // Quadrant 0
  def : CompressPat<(FLW FPR32C:$rd, GPRCMem:$rs1, uimm7_lsb00:$imm),
                    (C_FLW FPR32C:$rd, GPRCMem:$rs1, uimm7_lsb00:$imm)>;
  def : CompressPat<(FSW FPR32C:$rs2, GPRCMem:$rs1, uimm7_lsb00:$imm),
                    (C_FSW FPR32C:$rs2, GPRCMem:$rs1, uimm7_lsb00:$imm)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 921-935: TableGen record definition / TableGen 记录定义
```tablegen
  // Quadrant 2
  def : CompressPat<(FLW FPR32:$rd, SPMem:$rs1, uimm8_lsb00:$imm),
                    (C_FLWSP FPR32:$rd, SPMem:$rs1, uimm8_lsb00:$imm)>;
  def : CompressPat<(FSW FPR32:$rs2, SPMem:$rs1, uimm8_lsb00:$imm),
                    (C_FSWSP FPR32:$rs2, SPMem:$rs1, uimm8_lsb00:$imm)>;
} // Predicates = [HasStdExtZcf, IsRV32]

// Zcd Instructions
let Predicates = [HasStdExtZcd] in {
  // Quadrant 0
  def : CompressPat<(FLD FPR64C:$rd, GPRCMem:$rs1, uimm8_lsb000:$imm),
                    (C_FLD FPR64C:$rd, GPRCMem:$rs1, uimm8_lsb000:$imm)>;
  def : CompressPat<(FSD FPR64C:$rs2, GPRCMem:$rs1, uimm8_lsb000:$imm),
                    (C_FSD FPR64C:$rs2, GPRCMem:$rs1, uimm8_lsb000:$imm)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 936-941: TableGen record definition / TableGen 记录定义
```tablegen
  // Quadrant 2
  def : CompressPat<(FLD FPR64:$rd, SPMem:$rs1, uimm9_lsb000:$imm),
                    (C_FLDSP FPR64:$rd, SPMem:$rs1, uimm9_lsb000:$imm)>;
  def : CompressPat<(FSD FPR64:$rs2, SPMem:$rs1, uimm9_lsb000:$imm),
                    (C_FSDSP FPR64:$rs2, SPMem:$rs1, uimm9_lsb000:$imm)>;
} // Predicates = [HasStdExtZcd]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
