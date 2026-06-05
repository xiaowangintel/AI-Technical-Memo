# RISCVInstrInfoZb.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZb.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZb.td - RISC-V Bitmanip instructions -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard Bitmanip
// extensions, versions:
//   Zba - 1.0
//   Zbb - 1.0
//   Zbc - 1.0
//   Zbs - 1.0
//
// This file also describes RISC-V instructions from the Zbk* extensions in
// Cryptography Extensions Volume I: Scalar & Entropy Source Instructions,
// versions:
//   Zbkb - 1.0
//   Zbkc - 1.0
//   Zbkx - 1.0
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 24-40: TableGen record SDTIntShiftAddOp / TableGen 记录 SDTIntShiftAddOp
```tablegen

//===----------------------------------------------------------------------===//
// Operand and SDNode transformation definitions.
//===----------------------------------------------------------------------===//

def SDTIntShiftAddOp : SDTypeProfile<1, 3, [   // shl_add
  SDTCisSameAs<0, 1>, SDTCisSameAs<0, 3>, SDTCisInt<0>, SDTCisInt<2>,
  SDTCisInt<3>
]>;

def riscv_shl_add : RVSDNode<"SHL_ADD", SDTIntShiftAddOp>;

// RV64IB rotates, directly matching the semantics of the named RISC-V
// instructions.
def riscv_rolw    : RVSDNode<"ROLW",    SDT_RISCVIntBinOpW>;
def riscv_rorw    : RVSDNode<"RORW",    SDT_RISCVIntBinOpW>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 41-55: TableGen record riscv_clzw / TableGen 记录 riscv_clzw
```tablegen
// RV64IZbb bit counting instructions directly matching the semantics of the
// named RISC-V instructions.
def riscv_clzw    : RVSDNode<"CLZW",    SDT_RISCVIntUnaryOpW>;
def riscv_ctzw    : RVSDNode<"CTZW",    SDT_RISCVIntUnaryOpW>;

// brev8, orc.b, zip, and unzip from Zbb and Zbkb. All operands are i32 or
// XLenVT.
def riscv_brev8   : RVSDNode<"BREV8",   SDTIntUnaryOp>;
def riscv_orc_b   : RVSDNode<"ORC_B",   SDTIntUnaryOp>;
def riscv_zip     : RVSDNode<"ZIP",     SDTIntUnaryOp>;
def riscv_unzip   : RVSDNode<"UNZIP",   SDTIntUnaryOp>;

// RV64IZbb absolute value for i32. Expanded to (max (negw X), X) during isel.
def riscv_negw_max : RVSDNode<"NEGW_MAX",    SDTIntUnaryOp>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 56-74: TableGen record BCLRXForm / TableGen 记录 BCLRXForm
```tablegen
def BCLRXForm : SDNodeXForm<imm, [{
  // Find the lowest 0.
  return CurDAG->getTargetConstant(llvm::countr_one(N->getZExtValue()),
                                   SDLoc(N), N->getValueType(0));
}]>;

def SingleBitSetMaskToIndex : SDNodeXForm<imm, [{
  // Find the lowest 1.
  return CurDAG->getTargetConstant(llvm::countr_zero(N->getZExtValue()),
                                   SDLoc(N), N->getValueType(0));
}]>;

// Checks if this mask has a single 0 bit and cannot be used with ANDI.
def BCLRMask : ImmLeaf<XLenVT, [{
  if (Subtarget->is64Bit())
    return !isInt<12>(Imm) && isPowerOf2_64(~Imm);
  return !isInt<12>(Imm) && isPowerOf2_32(~Imm);
}], BCLRXForm>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 75-93: TableGen record SingleBitSetMask / TableGen 记录 SingleBitSetMask
```tablegen
// Checks if this mask has a single 1 bit and cannot be used with ORI/XORI.
def SingleBitSetMask : ImmLeaf<XLenVT, [{
  if (Subtarget->is64Bit())
    return !isInt<12>(Imm) && isPowerOf2_64(Imm);
  return !isInt<12>(Imm) && isPowerOf2_32(Imm);
}], SingleBitSetMaskToIndex>;

// Check if (or r, i) can be optimized to (BSETI (BSETI r, i0), i1),
// in which i = (1 << i0) | (1 << i1).
def BSETINVTwoBitsMask : PatLeaf<(imm), [{
  if (!N->hasOneUse())
    return false;
  // The immediate should not be a simm12.
  if (isInt<12>(N->getSExtValue()))
    return false;
  // The immediate must have exactly two bits set.
  return llvm::popcount(N->getZExtValue()) == 2;
}]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 94-111: TableGen record BSETINVTwoBitsMaskHigh / TableGen 记录 BSETINVTwoBitsMaskHigh
```tablegen
def BSETINVTwoBitsMaskHigh : SDNodeXForm<imm, [{
  uint64_t I = N->getZExtValue();
  return CurDAG->getTargetConstant(llvm::Log2_64(I), SDLoc(N),
                                   N->getValueType(0));
}]>;

// Check if (or r, imm) can be optimized to (BSETI (ORI r, i0), i1),
// in which imm = i0 | (1 << i1).
def BSETINVORIMask : PatLeaf<(imm), [{
  if (!N->hasOneUse())
    return false;
  // The immediate should not be a simm12.
  if (isInt<12>(N->getSExtValue()))
    return false;
  // There should be only one set bit from bit 11 to the top.
  return isPowerOf2_64(N->getZExtValue() & ~0x7ff);
}]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 112-128: TableGen record BSETINVORIMaskLow / TableGen 记录 BSETINVORIMaskLow
```tablegen
def BSETINVORIMaskLow : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(N->getZExtValue() & 0x7ff,
                                   SDLoc(N), N->getValueType(0));
}]>;

// Check if (and r, i) can be optimized to (BCLRI (BCLRI r, i0), i1),
// in which i = ~((1<<i0) | (1<<i1)).
def BCLRITwoBitsMask : PatLeaf<(imm), [{
  if (!N->hasOneUse())
    return false;
  // The immediate should not be a simm12.
  if (isInt<12>(N->getSExtValue()))
    return false;
  // The immediate must have exactly two bits clear.
  return (unsigned)llvm::popcount(N->getZExtValue()) == Subtarget->getXLen() - 2;
}]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 129-154: TableGen record BCLRITwoBitsMaskLow / TableGen 记录 BCLRITwoBitsMaskLow
```tablegen
def BCLRITwoBitsMaskLow : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(llvm::countr_zero(~N->getZExtValue()),
                                   SDLoc(N), N->getValueType(0));
}]>;

def BCLRITwoBitsMaskHigh : SDNodeXForm<imm, [{
  uint64_t I = N->getZExtValue();
  if (!Subtarget->is64Bit())
    I |= maskLeadingOnes<uint64_t>(32);
  return CurDAG->getTargetConstant(llvm::Log2_64(~I), SDLoc(N),
                                   N->getValueType(0));
}]>;

// Check if (and r, i) can be optimized to (BCLRI (ANDI r, i0), i1),
// in which i = i0 & ~(1<<i1).
def BCLRIANDIMask : PatLeaf<(imm), [{
  if (!N->hasOneUse())
    return false;
  // The immediate should not be a simm12.
  if (isInt<12>(N->getSExtValue()))
    return false;
  // There should be only one clear bit from bit 11 to the top.
  uint64_t I = N->getZExtValue() | 0x7ff;
  return Subtarget->is64Bit() ? isPowerOf2_64(~I) : isPowerOf2_32(~I);
}]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 155-169: TableGen record BCLRIANDIMaskLow / TableGen 记录 BCLRIANDIMaskLow
```tablegen
def BCLRIANDIMaskLow : SDNodeXForm<imm, [{
  return CurDAG->getSignedTargetConstant((N->getZExtValue() & 0x7ff) | ~0x7ffull,
                                         SDLoc(N), N->getValueType(0));
}]>;

def SimmShiftRightBy2XForm : SDNodeXForm<imm, [{
  return CurDAG->getSignedTargetConstant(N->getSExtValue() >> 2, SDLoc(N),
                                         N->getValueType(0));
}]>;

def SimmShiftRightBy3XForm : SDNodeXForm<imm, [{
  return CurDAG->getSignedTargetConstant(N->getSExtValue() >> 3, SDLoc(N),
                                         N->getValueType(0));
}]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 170-186: TableGen record CSImm12MulBy4 / TableGen 记录 CSImm12MulBy4
```tablegen
def CSImm12MulBy4 : PatLeaf<(imm), [{
  if (!N->hasOneUse())
    return false;
  int64_t C = N->getSExtValue();
  // Skip if C is simm12, an lui, or can be optimized by the PatLeaf AddiPair.
  return !isInt<13>(C) && !isShiftedInt<20, 12>(C) && isShiftedInt<12, 2>(C);
}], SimmShiftRightBy2XForm>;

def CSImm12MulBy8 : PatLeaf<(imm), [{
  if (!N->hasOneUse())
    return false;
  int64_t C = N->getSExtValue();
  // Skip if C is simm12, an lui or can be optimized by the PatLeaf AddiPair or
  // CSImm12MulBy4.
  return !isInt<14>(C) && !isShiftedInt<20, 12>(C) && isShiftedInt<12, 3>(C);
}], SimmShiftRightBy3XForm>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 187-204: TableGen class binop_with_non_imm12<SDPatternOperator / TableGen 类 binop_with_non_imm12<SDPatternOperator
```tablegen
// Pattern to exclude simm12 immediates from matching, namely `non_imm12`.
// GISel currently doesn't support PatFrag for leaf nodes, so `non_imm12`
// cannot be implemented in that way. To reuse patterns between the two
// ISels, we instead create PatFrag on operators that use `non_imm12`.
class binop_with_non_imm12<SDPatternOperator binop>
  : PatFrag<(ops node:$x, node:$y), (binop node:$x, node:$y), [{
  auto *C = dyn_cast<ConstantSDNode>(Operands[1]);
  return !C || !isInt<12>(C->getSExtValue());
}]> {
  let PredicateCodeUsesOperands = 1;
  let GISelPredicateCode = [{
    const MachineOperand &ImmOp = *Operands[1];

    if (ImmOp.isReg() && ImmOp.getReg())
      if (auto Val = getIConstantVRegValWithLookThrough(ImmOp.getReg(), MRI)) {
        // We do NOT want immediates that fit in 12 bits.
        return !isInt<12>(Val->Value.getSExtValue());
      }
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 205-220: TableGen record add_non_imm12 / TableGen 记录 add_non_imm12
```tablegen

    return true;
  }];
}
def add_non_imm12       : binop_with_non_imm12<add>;
def riscv_add_like_non_imm12 : binop_with_non_imm12<riscv_add_like>;

def Shifted32OnesMask : IntImmLeaf<XLenVT, [{
  if (!Imm.isShiftedMask())
    return false;

  unsigned TrailingZeros = Imm.countr_zero();
  return TrailingZeros > 0 && TrailingZeros < 32 &&
         Imm == UINT64_C(0xFFFFFFFF) << TrailingZeros;
}], TrailingZeros>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 221-241: TableGen record sh1add_op / TableGen 记录 sh1add_op
```tablegen
def sh1add_op : ComplexPattern<XLenVT, 1, "selectSHXADDOp<1>", [], [], 6>;
def sh2add_op : ComplexPattern<XLenVT, 1, "selectSHXADDOp<2>", [], [], 6>;
def sh3add_op : ComplexPattern<XLenVT, 1, "selectSHXADDOp<3>", [], [], 6>;
def gi_sh1add_op : GIComplexOperandMatcher<s32, "selectSHXADDOp<1>">,
                   GIComplexPatternEquiv<sh1add_op>;
def gi_sh2add_op : GIComplexOperandMatcher<s32, "selectSHXADDOp<2>">,
                   GIComplexPatternEquiv<sh2add_op>;
def gi_sh3add_op : GIComplexOperandMatcher<s32, "selectSHXADDOp<3>">,
                   GIComplexPatternEquiv<sh3add_op>;


def sh1add_uw_op : ComplexPattern<XLenVT, 1, "selectSHXADD_UWOp<1>", [], [], 6>;
def sh2add_uw_op : ComplexPattern<XLenVT, 1, "selectSHXADD_UWOp<2>", [], [], 6>;
def sh3add_uw_op : ComplexPattern<XLenVT, 1, "selectSHXADD_UWOp<3>", [], [], 6>;
def gi_sh1add_uw_op : GIComplexOperandMatcher<s32, "selectSHXADD_UWOp<1>">,
                      GIComplexPatternEquiv<sh1add_uw_op>;
def gi_sh2add_uw_op : GIComplexOperandMatcher<s32, "selectSHXADD_UWOp<2>">,
                      GIComplexPatternEquiv<sh2add_uw_op>;
def gi_sh3add_uw_op : GIComplexOperandMatcher<s32, "selectSHXADD_UWOp<3>">,
                      GIComplexPatternEquiv<sh3add_uw_op>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 242-260: TableGen class templates / TableGen 类 templates
```tablegen
//===----------------------------------------------------------------------===//
// Instruction class templates
//===----------------------------------------------------------------------===//

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class RVBUnaryR<bits<7> funct7, bits<3> funct3,
                RISCVOpcode opcode, string opcodestr>
    : RVInstR<funct7, funct3, opcode, (outs GPR:$rd), (ins GPR:$rs1),
              opcodestr, "$rd, $rs1"> {
  let rs2 = 0;
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 0 in
class RVBShift_ri<bits<5> imm11_7, bits<3> funct3, RISCVOpcode opcode,
                  string opcodestr>
    : RVInstIShift<imm11_7, funct3, opcode, (outs GPR:$rd),
                   (ins GPR:$rs1, uimmlog2xlen:$shamt), opcodestr,
                   "$rd, $rs1, $shamt">;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 261-282: TableGen record ANDN / TableGen 记录 ANDN
```tablegen
//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZbbOrZbkb] in {
def ANDN  : ALU_rr<0b0100000, 0b111, "andn">,
            Sched<[WriteIALU, ReadIALU, ReadIALU]>;
def ORN   : ALU_rr<0b0100000, 0b110, "orn">,
            Sched<[WriteIALU, ReadIALU, ReadIALU]>;
def XNOR  : ALU_rr<0b0100000, 0b100, "xnor", Commutable=1>,
            Sched<[WriteIALU, ReadIALU, ReadIALU]>;
} // Predicates = [HasStdExtZbbOrZbkb]

let Predicates = [HasStdExtZba] in {
def SH1ADD : ALU_rr<0b0010000, 0b010, "sh1add">,
             Sched<[WriteSHXADD, ReadSHXADD, ReadSHXADD]>;
def SH2ADD : ALU_rr<0b0010000, 0b100, "sh2add">,
             Sched<[WriteSHXADD, ReadSHXADD, ReadSHXADD]>;
def SH3ADD : ALU_rr<0b0010000, 0b110, "sh3add">,
             Sched<[WriteSHXADD, ReadSHXADD, ReadSHXADD]>;
} // Predicates = [HasStdExtZba]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 283-301: TableGen record SLLI_UW / TableGen 记录 SLLI_UW
```tablegen
let Predicates = [HasStdExtZba, IsRV64] in {
def SLLI_UW : RVBShift_ri<0b00001, 0b001, OPC_OP_IMM_32, "slli.uw">,
              Sched<[WriteShiftImm32, ReadShiftImm32]>;
def ADD_UW : ALUW_rr<0b0000100, 0b000, "add.uw">,
             Sched<[WriteIALU32, ReadIALU32, ReadIALU32]>;
def SH1ADD_UW : ALUW_rr<0b0010000, 0b010, "sh1add.uw">,
                Sched<[WriteSHXADD32, ReadSHXADD32, ReadSHXADD32]>;
def SH2ADD_UW : ALUW_rr<0b0010000, 0b100, "sh2add.uw">,
                Sched<[WriteSHXADD32, ReadSHXADD32, ReadSHXADD32]>;
def SH3ADD_UW : ALUW_rr<0b0010000, 0b110, "sh3add.uw">,
                Sched<[WriteSHXADD32, ReadSHXADD32, ReadSHXADD32]>;
} // Predicates = [HasStdExtZba, IsRV64]

let Predicates = [HasStdExtZbbOrZbkb] in {
def ROL   : ALU_rr<0b0110000, 0b001, "rol">,
            Sched<[WriteRotateReg, ReadRotateReg, ReadRotateReg]>;
def ROR   : ALU_rr<0b0110000, 0b101, "ror">,
            Sched<[WriteRotateReg, ReadRotateReg, ReadRotateReg]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 302-315: TableGen record RORI / TableGen 记录 RORI
```tablegen
def RORI  : Shift_ri<0b01100, 0b101, "rori">,
            Sched<[WriteRotateImm, ReadRotateImm]>;
} // Predicates = [HasStdExtZbbOrZbkb]

let Predicates = [HasStdExtZbbOrZbkb, IsRV64], IsSignExtendingOpW = 1 in {
def ROLW  : ALUW_rr<0b0110000, 0b001, "rolw">,
            Sched<[WriteRotateReg32, ReadRotateReg32, ReadRotateReg32]>;
def RORW  : ALUW_rr<0b0110000, 0b101, "rorw">,
            Sched<[WriteRotateReg32, ReadRotateReg32, ReadRotateReg32]>;

def RORIW : ShiftW_ri<0b0110000, 0b101, "roriw">,
            Sched<[WriteRotateImm32, ReadRotateImm32]>;
} // Predicates = [HasStdExtZbbOrZbkb, IsRV64]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 316-337: TableGen record BCLR / TableGen 记录 BCLR
```tablegen
let Predicates = [HasStdExtZbs] in {
def BCLR : ALU_rr<0b0100100, 0b001, "bclr">,
           Sched<[WriteSingleBit, ReadSingleBit, ReadSingleBit]>;
def BSET : ALU_rr<0b0010100, 0b001, "bset">,
           Sched<[WriteSingleBit, ReadSingleBit, ReadSingleBit]>;
def BINV : ALU_rr<0b0110100, 0b001, "binv">,
           Sched<[WriteSingleBit, ReadSingleBit, ReadSingleBit]>;
let IsSignExtendingOpW = 1 in
def BEXT : ALU_rr<0b0100100, 0b101, "bext">,
           Sched<[WriteBEXT, ReadSingleBit, ReadSingleBit]>;

def BCLRI : Shift_ri<0b01001, 0b001, "bclri">,
            Sched<[WriteSingleBitImm, ReadSingleBitImm]>;
def BSETI : Shift_ri<0b00101, 0b001, "bseti">,
            Sched<[WriteSingleBitImm, ReadSingleBitImm]>;
def BINVI : Shift_ri<0b01101, 0b001, "binvi">,
            Sched<[WriteSingleBitImm, ReadSingleBitImm]>;
let IsSignExtendingOpW = 1 in
def BEXTI : Shift_ri<0b01001, 0b101, "bexti">,
            Sched<[WriteBEXTI, ReadSingleBitImm]>;
} // Predicates = [HasStdExtZbs]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 338-356: TableGen record XPERM4 / TableGen 记录 XPERM4
```tablegen
// These instructions were named xperm.n and xperm.b in the last version of
// the draft bit manipulation specification they were included in. However, we
// use the mnemonics given to them in the ratified Zbkx extension.
let Predicates = [HasStdExtZbkx] in {
def XPERM4 : ALU_rr<0b0010100, 0b010, "xperm4">,
             Sched<[WriteXPERM, ReadXPERM, ReadXPERM]>;
def XPERM8 : ALU_rr<0b0010100, 0b100, "xperm8">,
             Sched<[WriteXPERM, ReadXPERM, ReadXPERM]>;
} // Predicates = [HasStdExtZbkx]

let Predicates = [HasStdExtZbb], IsSignExtendingOpW = 1 in {
def CLZ  : Unary_r<0b011000000000, 0b001, "clz">,
           Sched<[WriteCLZ, ReadCLZ]>;
def CTZ  : Unary_r<0b011000000001, 0b001, "ctz">,
           Sched<[WriteCTZ, ReadCTZ]>;
def CPOP : Unary_r<0b011000000010, 0b001, "cpop">,
           Sched<[WriteCPOP, ReadCPOP]>;
} // Predicates = [HasStdExtZbb]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 357-372: TableGen record CLZW / TableGen 记录 CLZW
```tablegen
let Predicates = [HasStdExtZbb, IsRV64], IsSignExtendingOpW = 1 in {
def CLZW  : UnaryW_r<0b011000000000, 0b001, "clzw">,
            Sched<[WriteCLZ32, ReadCLZ32]>;
def CTZW  : UnaryW_r<0b011000000001, 0b001, "ctzw">,
            Sched<[WriteCTZ32, ReadCTZ32]>;
def CPOPW : UnaryW_r<0b011000000010, 0b001, "cpopw">,
            Sched<[WriteCPOP32, ReadCPOP32]>;
} // Predicates = [HasStdExtZbb, IsRV64]

let Predicates = [HasStdExtZbb], IsSignExtendingOpW = 1 in {
def SEXT_B : Unary_r<0b011000000100, 0b001, "sext.b">,
             Sched<[WriteIALU, ReadIALU]>;
def SEXT_H : Unary_r<0b011000000101, 0b001, "sext.h">,
             Sched<[WriteIALU, ReadIALU]>;
} // Predicates = [HasStdExtZbb]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 373-395: TableGen record CLMULR / TableGen 记录 CLMULR
```tablegen
let Predicates = [HasStdExtZbc] in {
def CLMULR : ALU_rr<0b0000101, 0b010, "clmulr", Commutable=1>,
             Sched<[WriteCLMUL, ReadCLMUL, ReadCLMUL]>;
} // Predicates = [HasStdExtZbc]

let Predicates = [HasStdExtZbkc] in {
def CLMUL  : ALU_rr<0b0000101, 0b001, "clmul", Commutable=1>,
             Sched<[WriteCLMUL, ReadCLMUL, ReadCLMUL]>;
def CLMULH : ALU_rr<0b0000101, 0b011, "clmulh", Commutable=1>,
             Sched<[WriteCLMUL, ReadCLMUL, ReadCLMUL]>;
} // Predicates = [HasStdExtZbkc]

let Predicates = [HasStdExtZbb] in {
def MIN  : ALU_rr<0b0000101, 0b100, "min", Commutable=1>,
           Sched<[WriteIMinMax, ReadIMinMax, ReadIMinMax]>;
def MINU : ALU_rr<0b0000101, 0b101, "minu", Commutable=1>,
           Sched<[WriteIMinMax, ReadIMinMax, ReadIMinMax]>;
def MAX  : ALU_rr<0b0000101, 0b110, "max", Commutable=1>,
           Sched<[WriteIMinMax, ReadIMinMax, ReadIMinMax]>;
def MAXU : ALU_rr<0b0000101, 0b111, "maxu", Commutable=1>,
           Sched<[WriteIMinMax, ReadIMinMax, ReadIMinMax]>;
} // Predicates = [HasStdExtZbb]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 396-413: TableGen record PACK / TableGen 记录 PACK
```tablegen
let Predicates = [HasStdExtZbkbOrP] in
def PACK  : ALU_rr<0b0000100, 0b100, "pack">,
            Sched<[WritePACK, ReadPACK, ReadPACK]>;
let Predicates = [HasStdExtZbkb] in {
let IsSignExtendingOpW = 1 in
def PACKH : ALU_rr<0b0000100, 0b111, "packh">,
            Sched<[WritePACK, ReadPACK, ReadPACK]>;
} // Predicates = [HasStdExtZbkb]

let Predicates = [HasStdExtZbkb, IsRV64], IsSignExtendingOpW = 1 in
def PACKW  : ALUW_rr<0b0000100, 0b100, "packw">,
             Sched<[WritePACK32, ReadPACK32, ReadPACK32]>;

let Predicates = [HasStdExtZbbOrZbkb, IsRV32] in {
def ZEXT_H_RV32 : RVBUnaryR<0b0000100, 0b100, OPC_OP, "zext.h">,
                  Sched<[WriteIALU, ReadIALU]>;
} // Predicates = [HasStdExtZbbOrZbkb, IsRV32]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 414-428: TableGen record ZEXT_H_RV64 / TableGen 记录 ZEXT_H_RV64
```tablegen
let Predicates = [HasStdExtZbbOrZbkb, IsRV64], IsSignExtendingOpW = 1 in {
def ZEXT_H_RV64 : RVBUnaryR<0b0000100, 0b100, OPC_OP_32, "zext.h">,
                  Sched<[WriteIALU, ReadIALU]>;
} // Predicates = [HasStdExtZbbOrZbkb, IsRV64]

let Predicates = [HasStdExtZbbOrZbkb, IsRV32] in {
def REV8_RV32 : Unary_r<0b011010011000, 0b101, "rev8">,
                Sched<[WriteREV8, ReadREV8]>;
} // Predicates = [HasStdExtZbbOrZbkb, IsRV32]

let Predicates = [HasStdExtZbbOrZbkb, IsRV64] in {
def REV8_RV64 : Unary_r<0b011010111000, 0b101, "rev8">,
                Sched<[WriteREV8, ReadREV8]>;
} // Predicates = [HasStdExtZbbOrZbkb, IsRV64]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 429-444: TableGen record ORC_B / TableGen 记录 ORC_B
```tablegen
let Predicates = [HasStdExtZbb] in {
def ORC_B : Unary_r<0b001010000111, 0b101, "orc.b">,
            Sched<[WriteORCB, ReadORCB]>;
} // Predicates = [HasStdExtZbb]

let Predicates = [HasStdExtZbkb] in
def BREV8 : Unary_r<0b011010000111, 0b101, "brev8">,
            Sched<[WriteBREV8, ReadBREV8]>;

let Predicates = [HasStdExtZbkb, IsRV32] in {
def ZIP_RV32   : Unary_r<0b000010001111, 0b001, "zip">,
                 Sched<[WriteZIP, ReadZIP]>;
def UNZIP_RV32 : Unary_r<0b000010001111, 0b101, "unzip">,
                 Sched<[WriteZIP, ReadZIP]>;
} // Predicates = [HasStdExtZbkb, IsRV32]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 445-458: TableGen record definition / TableGen 记录定义
```tablegen

//===----------------------------------------------------------------------===//
// Pseudo Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZba, IsRV64] in {
def : InstAlias<"zext.w $rd, $rs", (ADD_UW GPR:$rd, GPR:$rs, X0)>;
} // Predicates = [HasStdExtZba, IsRV64]

let Predicates = [HasStdExtZbbOrZbkb] in {
def : InstAlias<"ror $rd, $rs1, $shamt",
                (RORI GPR:$rd, GPR:$rs1, uimmlog2xlen:$shamt), 0>;
} // Predicates = [HasStdExtZbbOrZbkb]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 459-474: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZbbOrZbkb, IsRV64] in {
def : InstAlias<"rorw $rd, $rs1, $shamt",
                (RORIW GPR:$rd, GPR:$rs1, uimm5:$shamt), 0>;
} // Predicates = [HasStdExtZbbOrZbkb, IsRV64]

let Predicates = [HasStdExtZbs] in {
def : InstAlias<"bset $rd, $rs1, $shamt",
                (BSETI  GPR:$rd, GPR:$rs1, uimmlog2xlen:$shamt), 0>;
def : InstAlias<"bclr $rd, $rs1, $shamt",
                (BCLRI GPR:$rd, GPR:$rs1, uimmlog2xlen:$shamt), 0>;
def : InstAlias<"binv $rd, $rs1, $shamt",
                (BINVI GPR:$rd, GPR:$rs1, uimmlog2xlen:$shamt), 0>;
def : InstAlias<"bext $rd, $rs1, $shamt",
                (BEXTI GPR:$rd, GPR:$rs1, uimmlog2xlen:$shamt), 0>;
} // Predicates = [HasStdExtZbs]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 475-491: TableGen record invLogicImm / TableGen 记录 invLogicImm
```tablegen
//===----------------------------------------------------------------------===//
// Codegen patterns
//===----------------------------------------------------------------------===//

def invLogicImm : ComplexPattern<XLenVT, 1, "selectInvLogicImm", [], [], 0>;

let Predicates = [HasStdExtZbbOrZbkb] in {
def : Pat<(XLenVT (and GPR:$rs1, (not GPR:$rs2))), (ANDN GPR:$rs1, GPR:$rs2)>;
def : Pat<(XLenVT (or  GPR:$rs1, (not GPR:$rs2))), (ORN  GPR:$rs1, GPR:$rs2)>;
def : Pat<(XLenVT (not (riscv_xor_like GPR:$rs1, GPR:$rs2))),
          (XNOR GPR:$rs1, GPR:$rs2)>;

def : Pat<(XLenVT (and GPR:$rs1, invLogicImm:$rs2)), (ANDN GPR:$rs1, invLogicImm:$rs2)>;
def : Pat<(XLenVT (or  GPR:$rs1, invLogicImm:$rs2)), (ORN  GPR:$rs1, invLogicImm:$rs2)>;
def : Pat<(XLenVT (xor GPR:$rs1, invLogicImm:$rs2)), (XNOR GPR:$rs1, invLogicImm:$rs2)>;
} // Predicates = [HasStdExtZbbOrZbkb]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 492-510: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZbbOrZbkb] in {
def : PatGprShiftMaskXLen<rotl, ROL>;
def : PatGprShiftMaskXLen<rotr, ROR>;

def : PatGprImm<rotr, RORI, uimmlog2xlen>;
// There's no encoding for roli in the the 'B' extension as it can be
// implemented with rori by negating the immediate.
def : Pat<(XLenVT (rotl GPR:$rs1, uimmlog2xlen:$shamt)),
          (RORI GPR:$rs1, (ImmSubFromXLen uimmlog2xlen:$shamt))>;
} // Predicates = [HasStdExtZbbOrZbkb]

let Predicates = [HasStdExtZbbOrZbkb, IsRV64] in {
def : PatGprShiftMask32<riscv_rolw, ROLW>;
def : PatGprShiftMask32<riscv_rorw, RORW>;
def : PatGprImm<riscv_rorw, RORIW, uimm5>;
def : Pat<(riscv_rolw GPR:$rs1, uimm5:$rs2),
          (RORIW GPR:$rs1, (ImmSubFrom32 uimm5:$rs2))>;
} // Predicates = [HasStdExtZbbOrZbkb, IsRV64]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 511-529: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZbs] in {
def : Pat<(XLenVT (and (not (shl 1, shiftMaskXLen:$rs2)), GPR:$rs1)),
          (BCLR GPR:$rs1, shiftMaskXLen:$rs2)>;
def : Pat<(XLenVT (and (rotl -2, shiftMaskXLen:$rs2), GPR:$rs1)),
          (BCLR GPR:$rs1, shiftMaskXLen:$rs2)>;
def : Pat<(XLenVT (or (shl 1, shiftMaskXLen:$rs2), GPR:$rs1)),
          (BSET GPR:$rs1, shiftMaskXLen:$rs2)>;
def : Pat<(XLenVT (xor (shl 1, shiftMaskXLen:$rs2), GPR:$rs1)),
          (BINV GPR:$rs1, shiftMaskXLen:$rs2)>;
def : Pat<(XLenVT (and (srl GPR:$rs1, shiftMaskXLen:$rs2), 1)),
          (BEXT GPR:$rs1, shiftMaskXLen:$rs2)>;

def : Pat<(XLenVT (shl 1, shiftMaskXLen:$rs2)),
          (BSET (XLenVT X0), shiftMaskXLen:$rs2)>;
def : Pat<(XLenVT (add (shl -1, shiftMaskXLen:$rs2), GPR:$rs3)),
          (SUB GPR:$rs3, (XLenVT (BSET (XLenVT X0), shiftMaskXLen:$rs2)))>;
def : Pat<(XLenVT (not (shl -1, shiftMaskXLen:$rs2))),
          (ADDI (XLenVT (BSET (XLenVT X0), shiftMaskXLen:$rs2)), -1)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 530-557: TableGen record definition / TableGen 记录定义
```tablegen
def : Pat<(XLenVT (and GPR:$rs1, BCLRMask:$mask)),
          (BCLRI GPR:$rs1, BCLRMask:$mask)>;
def : Pat<(XLenVT (or GPR:$rs1, SingleBitSetMask:$mask)),
          (BSETI GPR:$rs1, SingleBitSetMask:$mask)>;
def : Pat<(XLenVT (xor GPR:$rs1, SingleBitSetMask:$mask)),
          (BINVI GPR:$rs1, SingleBitSetMask:$mask)>;

def : Pat<(XLenVT (and (srl GPR:$rs1, uimmlog2xlen:$shamt), (XLenVT 1))),
          (BEXTI GPR:$rs1, uimmlog2xlen:$shamt)>;

def : Pat<(XLenVT (seteq (XLenVT (and GPR:$rs1, SingleBitSetMask:$mask)), 0)),
          (BEXTI (XLenVT (XORI GPR:$rs1, -1)), SingleBitSetMask:$mask)>;

def : Pat<(XLenVT (or GPR:$r, BSETINVTwoBitsMask:$i)),
          (BSETI (XLenVT (BSETI GPR:$r, (TrailingZeros BSETINVTwoBitsMask:$i))),
                 (BSETINVTwoBitsMaskHigh BSETINVTwoBitsMask:$i))>;
def : Pat<(XLenVT (xor GPR:$r, BSETINVTwoBitsMask:$i)),
          (BINVI (XLenVT (BINVI GPR:$r, (TrailingZeros BSETINVTwoBitsMask:$i))),
                 (BSETINVTwoBitsMaskHigh BSETINVTwoBitsMask:$i))>;
def : Pat<(XLenVT (or GPR:$r, BSETINVORIMask:$i)),
          (BSETI (XLenVT (ORI GPR:$r, (BSETINVORIMaskLow BSETINVORIMask:$i))),
                 (BSETINVTwoBitsMaskHigh BSETINVORIMask:$i))>;
def : Pat<(XLenVT (xor GPR:$r, BSETINVORIMask:$i)),
          (BINVI (XLenVT (XORI GPR:$r, (BSETINVORIMaskLow BSETINVORIMask:$i))),
                 (BSETINVTwoBitsMaskHigh BSETINVORIMask:$i))>;
def : Pat<(XLenVT (and GPR:$r, BCLRITwoBitsMask:$i)),
          (BCLRI (XLenVT (BCLRI GPR:$r, (BCLRITwoBitsMaskLow BCLRITwoBitsMask:$i))),
                 (BCLRITwoBitsMaskHigh BCLRITwoBitsMask:$i))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 558-574: TableGen record definition / TableGen 记录定义
```tablegen
def : Pat<(XLenVT (and GPR:$r, BCLRIANDIMask:$i)),
          (BCLRI (XLenVT (ANDI GPR:$r, (BCLRIANDIMaskLow BCLRIANDIMask:$i))),
                 (BCLRITwoBitsMaskHigh BCLRIANDIMask:$i))>;
} // Predicates = [HasStdExtZbs]

let Predicates = [HasStdExtZbs, IsRV64] in {
// If the original code was setting, clearing, or inverting bit 31 of an i32,
// type legalization might have sign extended the constant so it doesn't have a
// single 0 or 1 bit. If the upper 32 bits aren't used by later instructions we
// can still use bseti/bclri.
def : Pat<(binop_allwusers<and> GPR:$rs1, (XLenVT 2147483647)),
          (BCLRI GPR:$rs1, 31)>;
def : Pat<(binop_allwusers<or> GPR:$rs1, (XLenVT -2147483648)),
          (BSETI GPR:$rs1, 31)>;
def : Pat<(binop_allwusers<xor> GPR:$rs1, (XLenVT -2147483648)),
          (BINVI GPR:$rs1, 31)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 575-593: TableGen record definition / TableGen 记录定义
```tablegen

let Predicates = [HasStdExtZbb] in
def : PatGpr<riscv_orc_b, ORC_B>;

let Predicates = [HasStdExtZbkb] in
def : PatGpr<riscv_brev8, BREV8>;

let Predicates = [HasStdExtZbkb, IsRV32] in {
// We treat zip and unzip as separate instructions, so match it directly.
def : PatGpr<riscv_zip, ZIP_RV32, i32>;
def : PatGpr<riscv_unzip, UNZIP_RV32, i32>;
} // Predicates = [HasStdExtZbkb, IsRV32]

let Predicates = [HasStdExtZbb] in {
def : PatGpr<ctlz, CLZ>;
def : PatGpr<cttz, CTZ>;
def : PatGpr<ctpop, CPOP>;
} // Predicates = [HasStdExtZbb]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 594-607: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZbb, IsRV64] in {
def : PatGpr<riscv_clzw, CLZW>;
def : PatGpr<riscv_ctzw, CTZW>;
def : Pat<(i64 (ctpop (i64 (zexti32 (i64 GPR:$rs1))))), (CPOPW GPR:$rs1)>;

def : Pat<(i64 (riscv_negw_max GPR:$rs1)),
          (MAX GPR:$rs1, (XLenVT (SUBW (XLenVT X0), GPR:$rs1)))>;
} // Predicates = [HasStdExtZbb, IsRV64]

let Predicates = [HasStdExtZbb] in {
def : Pat<(XLenVT (sext_inreg GPR:$rs1, i8)), (SEXT_B GPR:$rs1)>;
def : Pat<(XLenVT (sext_inreg GPR:$rs1, i16)), (SEXT_H GPR:$rs1)>;
} // Predicates = [HasStdExtZbb]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 608-631: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZbb] in {
def : PatGprGpr<smin, MIN>;
def : PatGprGpr<smax, MAX>;
def : PatGprGpr<umin, MINU>;
def : PatGprGpr<umax, MAXU>;
} // Predicates = [HasStdExtZbb]

let Predicates = [HasStdExtZbbOrZbkb, IsRV32] in
def : PatGpr<bswap, REV8_RV32, i32>;

let Predicates = [HasStdExtZbbOrZbkb, IsRV64] in
def : PatGpr<bswap, REV8_RV64, i64>;

let Predicates = [HasStdExtZbkb] in {
def : Pat<(or (and (shl GPR:$rs2, (XLenVT 8)), 0xFFFF),
              zexti8:$rs1),
          (PACKH zexti8:$rs1, GPR:$rs2)>;
def : Pat<(or (shl zexti8:$rs2, (XLenVT 8)),
              zexti8:$rs1),
          (PACKH zexti8:$rs1, zexti8:$rs2)>;
def : Pat<(and (or (shl GPR:$rs2, (XLenVT 8)),
                   zexti8:$rs1), 0xFFFF),
          (PACKH zexti8:$rs1, GPR:$rs2)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 632-648: TableGen record definition / TableGen 记录定义
```tablegen
def : Pat<(binop_allhusers<or> (shl GPR:$rs2, (XLenVT 8)),
                               zexti8:$rs1),
          (PACKH zexti8:$rs1, GPR:$rs2)>;

def : Pat<(shl (and GPR:$rs2, 0xFF), (XLenVT 8)),
          (PACKH (XLenVT X0), GPR:$rs2)>;
} // Predicates = [HasStdExtZbkb]

let Predicates = [HasStdExtZbkbOrP, IsRV32] in
def : Pat<(i32 (or zexti16:$rs1, (shl GPR:$rs2, (i32 16)))),
          (PACK zexti16:$rs1, GPR:$rs2)>;

let Predicates = [HasStdExtZbkb, IsRV32] in {
def : Pat<(i32 (or (shl GPR:$rs2, (XLenVT 24)),
                   (shl zexti8:$rs1, (XLenVT 16)))),
          (SLLI (XLenVT (PACKH zexti8:$rs1, GPR:$rs2)), (XLenVT 16))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 649-663: TableGen record definition / TableGen 记录定义
```tablegen
// Match a pattern of 2 bytes being inserted into bits [31:16], with bits
// bits [15:0] coming from a zero extended value. We can use pack with packh for
// bits [31:16]. If bits [15:0] can also be a packh, it can be matched
// separately.
def : Pat<(i32 (or (or (shl GPR:$op1rs2, (XLenVT 24)),
                       (shl zexti8:$op1rs1, (XLenVT 16))),
                   zexti16:$rs1)),
          (PACK zexti16:$rs1,
                (XLenVT (PACKH zexti8:$op1rs1, GPR:$op1rs2)))>;
}

let Predicates = [HasStdExtZbkbOrP, IsRV64] in
def : Pat<(i64 (or zexti32:$rs1, (shl GPR:$rs2, (i64 32)))),
          (PACK zexti32:$rs1, GPR:$rs2)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 664-678: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZbkb, IsRV64] in {
def : Pat<(i64 (or (shl zexti8:$rs2, (XLenVT 24)),
                   (shl zexti8:$rs1, (XLenVT 16)))),
          (SLLI (XLenVT (PACKH zexti8:$rs1, zexti8:$rs2)), (XLenVT 16))>;
def : Pat<(binop_allwusers<or> (shl GPR:$rs2, (XLenVT 24)),
                               (shl zexti8:$rs1, (XLenVT 16))),
          (SLLI (XLenVT (PACKH zexti8:$rs1, GPR:$rs2)), (XLenVT 16))>;

def : Pat<(binop_allwusers<or> (shl GPR:$rs2, (i64 16)),
                               zexti16:$rs1),
          (PACKW zexti16:$rs1, GPR:$rs2)>;
def : Pat<(i64 (or (sext_inreg (shl GPR:$rs2, (i64 16)), i32),
                   zexti16:$rs1)),
          (PACKW zexti16:$rs1, GPR:$rs2)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 679-699: TableGen record definition / TableGen 记录定义
```tablegen
// Match a pattern of 2 bytes being inserted into bits [31:16], with bits
// bits [15:0] coming from a zero extended value, and bits [63:32] being
// ignored. We can use packw with packh for bits [31:16]. If bits [15:0] can
// also be a packh, it can be matched separately.
def : Pat<(binop_allwusers<or>
               (or (shl GPR:$op1rs2, (XLenVT 24)),
                   (shl zexti8:$op1rs1, (XLenVT 16))),
               zexti16:$rs1),
          (PACKW zexti16:$rs1, (XLenVT (PACKH zexti8:$op1rs1, GPR:$op1rs2)))>;
// We need to manually reassociate the patterns because of the binop_allwusers.
def : Pat<(binop_allwusers<or>
               (or zexti16:$rs1,
                   (shl zexti8:$op1rs1, (XLenVT 16))),
               (shl GPR:$op1rs2, (XLenVT 24))),
          (PACKW zexti16:$rs1, (XLenVT (PACKH zexti8:$op1rs1, GPR:$op1rs2)))>;
def : Pat<(binop_allwusers<or>
               (or zexti16:$rs1,
                   (shl GPR:$op1rs2, (XLenVT 24))),
               (shl zexti8:$op1rs1, (XLenVT 16))),
          (PACKW zexti16:$rs1, (XLenVT (PACKH zexti8:$op1rs1, GPR:$op1rs2)))>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 700-715: TableGen record definition / TableGen 记录定义
```tablegen
def : Pat<(i64 (or (or (zexti16 (XLenVT GPR:$rs1)),
                       (shl zexti8:$op1rs1, (XLenVT 16))),
                   (sext_inreg (shl GPR:$op1rs2, (XLenVT 24)), i32))),
          (PACKW GPR:$rs1, (XLenVT (PACKH zexti8:$op1rs1, GPR:$op1rs2)))>;

// Match a pattern of 2 halfwords being inserted into bits [63:32], with bits
// bits [31:0] coming from a zero extended value. We can use pack with packw for
// bits [63:32]. If bits [63:31] can also be a packw, it can be matched
// separately.
def : Pat<(or (or (shl GPR:$op1rs2, (i64 48)),
                  (shl zexti16:$op1rs1, (i64 32))),
              zexti32:$rs1),
          (PACK zexti32:$rs1,
                (XLenVT (PACKW zexti16:$op1rs1, GPR:$op1rs2)))>;
} // Predicates = [HasStdExtZbkb, IsRV64]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 716-731: Reusable TableGen multiclass ShxAddPat / 可复用的 TableGen 多类 ShxAddPat
```tablegen
let Predicates = [HasStdExtZbbOrZbkb, IsRV32] in
def : Pat<(i32 (and GPR:$rs, 0xFFFF)), (ZEXT_H_RV32 GPR:$rs)>;
let Predicates = [HasStdExtZbbOrZbkb, IsRV64] in
def : Pat<(i64 (and GPR:$rs, 0xFFFF)), (ZEXT_H_RV64 GPR:$rs)>;

multiclass ShxAddPat<int i, Instruction shxadd> {
  def : Pat<(XLenVT (riscv_add_like_non_imm12 (shl GPR:$rs1, (XLenVT i)), GPR:$rs2)),
            (shxadd GPR:$rs1, GPR:$rs2)>;
  def : Pat<(XLenVT (riscv_shl_add GPR:$rs1, (XLenVT i), GPR:$rs2)),
            (shxadd GPR:$rs1, GPR:$rs2)>;

  defvar pat = !cast<ComplexPattern>("sh"#i#"add_op");
  // More complex cases use a ComplexPattern.
  def : Pat<(XLenVT (riscv_add_like_non_imm12 pat:$rs1, GPR:$rs2)),
            (shxadd pat:$rs1, GPR:$rs2)>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 732-747: TableGen class CSImm12MulBy4Pat<Instruction / TableGen 类 CSImm12MulBy4Pat<Instruction
```tablegen

class CSImm12MulBy4Pat<Instruction sh2add>
    : Pat<(riscv_add_like (XLenVT GPR:$r), CSImm12MulBy4:$i),
          (sh2add (XLenVT (ADDI (XLenVT X0), CSImm12MulBy4:$i)),
                  GPR:$r)>;

class CSImm12MulBy8Pat<Instruction sh3add>
    : Pat<(riscv_add_like (XLenVT GPR:$r), CSImm12MulBy8:$i),
          (sh3add (XLenVT (ADDI (XLenVT X0), CSImm12MulBy8:$i)),
                  GPR:$r)>;

let Predicates = [HasStdExtZba] in {
foreach i = {1,2,3} in {
  defvar shxadd = !cast<Instruction>("SH"#i#"ADD");
  defm : ShxAddPat<i, shxadd>;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 748-761: Reusable TableGen multiclass zExtImm32 / 可复用的 TableGen 多类 zExtImm32
```tablegen

def : CSImm12MulBy4Pat<SH2ADD>;
def : CSImm12MulBy8Pat<SH3ADD>;
} // Predicates = [HasStdExtZba]

def zExtImm32 : ComplexPattern<i64, 1, "selectZExtImm32", [], [], 0>;

multiclass ADD_UWPat<Instruction add_uw> {
  def : Pat<(i64 (riscv_add_like_non_imm12 (and GPR:$rs1, 0xFFFFFFFF), GPR:$rs2)),
            (add_uw GPR:$rs1, GPR:$rs2)>;
  def : Pat<(i64 (riscv_add_like zExtImm32:$rs1, GPR:$rs2)),
            (add_uw zExtImm32:$rs1, GPR:$rs2)>;
  def : Pat<(i64 (and GPR:$rs, 0xFFFFFFFF)), (add_uw GPR:$rs, (XLenVT X0))>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 762-775: Reusable TableGen multiclass ShxAdd_UWPat / 可复用的 TableGen 多类 ShxAdd_UWPat
```tablegen

multiclass ShxAdd_UWPat<int i, Instruction shxadd_uw> {
  def : Pat<(i64 (riscv_add_like_non_imm12 (shl (and GPR:$rs1, 0xFFFFFFFF), (i64 i)),
                                     (XLenVT GPR:$rs2))),
            (shxadd_uw GPR:$rs1, GPR:$rs2)>;
  def : Pat<(i64 (riscv_shl_add (and GPR:$rs1, 0xFFFFFFFF), (i64 i), GPR:$rs2)),
            (shxadd_uw GPR:$rs1, GPR:$rs2)>;

  defvar pat = !cast<ComplexPattern>("sh"#i#"add_uw_op");
  // More complex cases use a ComplexPattern.
  def : Pat<(i64 (riscv_add_like_non_imm12 pat:$rs1, (XLenVT GPR:$rs2))),
            (shxadd_uw pat:$rs1, GPR:$rs2)>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 776-794: Reusable TableGen multiclass Sh1Add_UWPat / 可复用的 TableGen 多类 Sh1Add_UWPat
```tablegen
multiclass Sh1Add_UWPat<Instruction sh1add_uw> {
  def : Pat<(riscv_add_like_non_imm12 (and (shl GPR:$rs1, (i64 1)), (i64 0x1FFFFFFFF)),
                                     (XLenVT GPR:$rs2)),
            (sh1add_uw GPR:$rs1, GPR:$rs2)>;
  // Use SRLI to clear the LSBs and SHXADD_UW to mask and shift.
  def : Pat<(riscv_add_like_non_imm12 (and GPR:$rs1, (i64 0x1FFFFFFFE)),
                                (XLenVT GPR:$rs2)),
            (sh1add_uw (XLenVT (SRLI GPR:$rs1, 1)), GPR:$rs2)>;
}

multiclass Sh2Add_UWPat<Instruction sh2add_uw> {
  def : Pat<(riscv_add_like_non_imm12 (and (shl GPR:$rs1, (i64 2)), (i64 0x3FFFFFFFF)),
                                (XLenVT GPR:$rs2)),
            (sh2add_uw GPR:$rs1, GPR:$rs2)>;
  // Use SRLI to clear the LSBs and SHXADD_UW to mask and shift.
  def : Pat<(riscv_add_like_non_imm12 (and GPR:$rs1, (i64 0x3FFFFFFFC)),
                                (XLenVT GPR:$rs2)),
            (sh2add_uw (XLenVT (SRLI GPR:$rs1, 2)), GPR:$rs2)>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 795-810: Reusable TableGen multiclass Sh1AddPat<Instruction / 可复用的 TableGen 多类 Sh1AddPat<Instruction
```tablegen

multiclass Sh3Add_UWPat<Instruction sh3add_uw> {
  def : Pat<(riscv_add_like_non_imm12 (and (shl GPR:$rs1, (i64 3)), (i64 0x7FFFFFFFF)),
                                (XLenVT GPR:$rs2)),
            (sh3add_uw GPR:$rs1, GPR:$rs2)>;
  // Use SRLI to clear the LSBs and SHXADD_UW to mask and shift.
  def : Pat<(riscv_add_like_non_imm12 (and GPR:$rs1, (i64 0x7FFFFFFF8)),
                                (XLenVT GPR:$rs2)),
            (sh3add_uw (XLenVT (SRLI GPR:$rs1, 3)), GPR:$rs2)>;
}

class Sh1AddPat<Instruction sh1add>
    : Pat<(i64 (riscv_add_like_non_imm12 (and GPR:$rs1, 0xFFFFFFFE),
                                   (XLenVT GPR:$rs2))),
          (sh1add (XLenVT (SRLIW GPR:$rs1, 1)), GPR:$rs2)>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 811-829: TableGen class Sh2AddPat<Instruction / TableGen 类 Sh2AddPat<Instruction
```tablegen
class Sh2AddPat<Instruction sh2add>
    : Pat<(i64 (riscv_add_like_non_imm12 (and GPR:$rs1, 0xFFFFFFFC),
                                   (XLenVT GPR:$rs2))),
          (sh2add (XLenVT (SRLIW GPR:$rs1, 2)), GPR:$rs2)>;

class Sh3AddPat<Instruction sh3add>
    : Pat<(i64 (riscv_add_like_non_imm12 (and GPR:$rs1, 0xFFFFFFF8),
                                   (XLenVT GPR:$rs2))),
          (sh3add (XLenVT (SRLIW GPR:$rs1, 3)), GPR:$rs2)>;

let Predicates = [HasStdExtZba, IsRV64] in {
def : Pat<(i64 (shl (and GPR:$rs1, 0xFFFFFFFF), uimm5:$shamt)),
          (SLLI_UW GPR:$rs1, uimm5:$shamt)>;
// Match a shifted 0xffffffff mask. Use SRLI to clear the LSBs and SLLI_UW to
// mask and shift.
def : Pat<(i64 (and GPR:$rs1, Shifted32OnesMask:$mask)),
          (SLLI_UW (XLenVT (SRLI GPR:$rs1, Shifted32OnesMask:$mask)),
                   Shifted32OnesMask:$mask)>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 830-845: Bulk record instantiation i / 批量记录实例化 i
```tablegen
defm : ADD_UWPat<ADD_UW>;

foreach i = {1,2,3} in {
  defvar shxadd_uw = !cast<Instruction>("SH"#i#"ADD_UW");
  defm : ShxAdd_UWPat<i, shxadd_uw>;
}

defm : Sh1Add_UWPat<SH1ADD_UW>;
defm : Sh2Add_UWPat<SH2ADD_UW>;
defm : Sh3Add_UWPat<SH3ADD_UW>;

def : Sh1AddPat<SH1ADD>;
def : Sh2AddPat<SH2ADD>;
def : Sh3AddPat<SH3ADD>;
} // Predicates = [HasStdExtZba, IsRV64]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 846-859: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZbkc] in {
def : PatGprGpr<clmul, CLMUL>;
def : PatGprGpr<clmulh, CLMULH>;
} // Predicates = [HasStdExtZbkc]

let Predicates = [HasStdExtZbkc, IsRV64, NoStdExtZba] in {
// Special case for calculating the full 64-bit product of a 32x32 unsigned
// multiply where the inputs aren't known to be zero extended. We can shift the
// inputs left by 32 and use a CLMULH. This saves two SRLIs needed to finish
// zeroing the upper 32 bits.
def : Pat<(i64 (clmul (and GPR:$rs1, 0xffffffff), (and GPR:$rs2, 0xffffffff))),
          (CLMULH (i64 (SLLI GPR:$rs1, 32)), (i64 (SLLI GPR:$rs2, 32)))>;
} // Predicates = [HasStdExtZbkc, IsRV64, NoStdExtZba]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 860-866: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = [HasStdExtZbc] in
def : PatGprGpr<clmulr, CLMULR>;

let Predicates = [HasStdExtZbkx] in {
def : PatGprGpr<int_riscv_xperm4, XPERM4>;
def : PatGprGpr<int_riscv_xperm8, XPERM8>;
} // Predicates = [HasStdExtZbkx]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
