# Mips64InstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/Mips64InstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes Mips64 instructions.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 Mips 后端中的 `Mips64InstrInfo`，涵盖指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===- Mips64InstrInfo.td - Mips64 Instruction Information -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes Mips64 instructions.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-15
```tablegen
//===----------------------------------------------------------------------===//
// Mips Operand, Complex Patterns and Transformations Definitions.
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 17-19
```tablegen
// shamt must fit in 6 bits.
def immZExt6 : ImmLeaf<i32, [{return Imm == (Imm & 0x3f);}]>;
def timmZExt6 : TImmLeaf<i32, [{return Imm == (Imm & 0x3f);}]>;
```
- EN: Defines TableGen record `immZExt6` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExt6`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 21-24
```tablegen
// Node immediate fits as 10-bit sign extended on target immediate.
// e.g. seqi, snei
def immSExt10_64 : PatLeaf<(i64 imm),
                           [{ return isInt<10>(N->getSExtValue()); }]>;
```
- EN: Defines TableGen record `immSExt10_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immSExt10_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 26-27
```tablegen
def immZExt16_64 : PatLeaf<(i64 imm),
                           [{ return isUInt<16>(N->getZExtValue()); }]>;
```
- EN: Defines TableGen record `immZExt16_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExt16_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 29-29
```tablegen
def immZExt5_64 : ImmLeaf<i64, [{ return Imm == (Imm & 0x1f); }]>;
```
- EN: Defines TableGen record `immZExt5_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `immZExt5_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 31-34
```tablegen
// Transformation function: get log2 of low 32 bits of immediate
def Log2LO : SDNodeXForm<imm, [{
  return getImm(N, Log2_64((unsigned) N->getZExtValue()));
}]>;
```
- EN: Defines TableGen record `Log2LO` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `Log2LO`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 36-39
```tablegen
// Transformation function: get log2 of high 32 bits of immediate
def Log2HI : SDNodeXForm<imm, [{
  return getImm(N, Log2_64((unsigned) (N->getZExtValue() >> 32)));
}]>;
```
- EN: Defines TableGen record `Log2HI` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `Log2HI`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 41-49
```tablegen
// Predicate: True if immediate is a power of 2 and fits 32 bits
def PowerOf2LO : PatLeaf<(imm), [{
  if (N->getValueType(0) == MVT::i64) {
    uint64_t Imm = N->getZExtValue();
    return isPowerOf2_64(Imm) && (Imm & 0xffffffff) == Imm;
  }
  else
    return false;
}]>;
```
- EN: Defines TableGen record `PowerOf2LO` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PowerOf2LO`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 51-59
```tablegen
// Predicate: True if immediate is a power of 2 and exceeds 32 bits
def PowerOf2HI : PatLeaf<(imm), [{
  if (N->getValueType(0) == MVT::i64) {
    uint64_t Imm = N->getZExtValue();
    return isPowerOf2_64(Imm) && (Imm & 0xffffffff00000000) == Imm;
  }
  else
    return false;
}]>;
```
- EN: Defines TableGen record `PowerOf2HI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PowerOf2HI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 61-68
```tablegen
def PowerOf2LO_i32 : PatLeaf<(imm), [{
  if (N->getValueType(0) == MVT::i32) {
    uint64_t Imm = N->getZExtValue();
    return isPowerOf2_32(Imm) && isUInt<32>(Imm);
  }
  else
    return false;
}]>;
```
- EN: Defines TableGen record `PowerOf2LO_i32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PowerOf2LO_i32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 70-72
```tablegen
def assertzext_lt_i32 : PatFrag<(ops node:$src), (assertzext node:$src), [{
  return cast<VTSDNode>(N->getOperand(1))->getVT().bitsLT(MVT::i32);
}]>;
```
- EN: Defines TableGen record `assertzext_lt_i32` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `assertzext_lt_i32`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 74-90
```tablegen
//===----------------------------------------------------------------------===//
// Instructions specific format
//===----------------------------------------------------------------------===//
let usesCustomInserter = 1 in {
  def ATOMIC_LOAD_ADD_I64  : Atomic2Ops<atomic_load_add_i64, GPR64>;
  def ATOMIC_LOAD_SUB_I64  : Atomic2Ops<atomic_load_sub_i64, GPR64>;
  def ATOMIC_LOAD_AND_I64  : Atomic2Ops<atomic_load_and_i64, GPR64>;
  def ATOMIC_LOAD_OR_I64   : Atomic2Ops<atomic_load_or_i64, GPR64>;
  def ATOMIC_LOAD_XOR_I64  : Atomic2Ops<atomic_load_xor_i64, GPR64>;
  def ATOMIC_LOAD_NAND_I64 : Atomic2Ops<atomic_load_nand_i64, GPR64>;
  def ATOMIC_SWAP_I64      : Atomic2Ops<atomic_swap_i64, GPR64>;
  def ATOMIC_CMP_SWAP_I64  : AtomicCmpSwap<atomic_cmp_swap_i64, GPR64>;
  def ATOMIC_LOAD_MIN_I64  : Atomic2Ops<atomic_load_min_i64, GPR64>;
  def ATOMIC_LOAD_MAX_I64  : Atomic2Ops<atomic_load_max_i64, GPR64>;
  def ATOMIC_LOAD_UMIN_I64 : Atomic2Ops<atomic_load_umin_i64, GPR64>;
  def ATOMIC_LOAD_UMAX_I64 : Atomic2Ops<atomic_load_umax_i64, GPR64>;
}
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 92-97
```tablegen
def ATOMIC_LOAD_ADD_I64_POSTRA  : Atomic2OpsPostRA<GPR64>;
def ATOMIC_LOAD_SUB_I64_POSTRA  : Atomic2OpsPostRA<GPR64>;
def ATOMIC_LOAD_AND_I64_POSTRA  : Atomic2OpsPostRA<GPR64>;
def ATOMIC_LOAD_OR_I64_POSTRA   : Atomic2OpsPostRA<GPR64>;
def ATOMIC_LOAD_XOR_I64_POSTRA  : Atomic2OpsPostRA<GPR64>;
def ATOMIC_LOAD_NAND_I64_POSTRA : Atomic2OpsPostRA<GPR64>;
```
- EN: Defines TableGen record `ATOMIC_LOAD_ADD_I64_POSTRA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_LOAD_ADD_I64_POSTRA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 99-99
```tablegen
def ATOMIC_SWAP_I64_POSTRA      : Atomic2OpsPostRA<GPR64>;
```
- EN: Defines TableGen record `ATOMIC_SWAP_I64_POSTRA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_SWAP_I64_POSTRA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 101-101
```tablegen
def ATOMIC_CMP_SWAP_I64_POSTRA  : AtomicCmpSwapPostRA<GPR64>;
```
- EN: Defines TableGen record `ATOMIC_CMP_SWAP_I64_POSTRA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_CMP_SWAP_I64_POSTRA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 103-106
```tablegen
def ATOMIC_LOAD_MIN_I64_POSTRA  : Atomic2OpsPostRA<GPR64>;
def ATOMIC_LOAD_MAX_I64_POSTRA  : Atomic2OpsPostRA<GPR64>;
def ATOMIC_LOAD_UMIN_I64_POSTRA : Atomic2OpsPostRA<GPR64>;
def ATOMIC_LOAD_UMAX_I64_POSTRA : Atomic2OpsPostRA<GPR64>;
```
- EN: Defines TableGen record `ATOMIC_LOAD_MIN_I64_POSTRA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ATOMIC_LOAD_MIN_I64_POSTRA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 108-112
```tablegen
/// Pseudo instructions for loading and storing accumulator registers.
let isPseudo = 1, isCodeGenOnly = 1, hasNoSchedulingInfo = 1 in {
  def LOAD_ACC128  : Load<"", ACC128>;
  def STORE_ACC128 : Store<"", ACC128>;
}
```
- EN: Defines TableGen record `LOAD_ACC128` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LOAD_ACC128`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 114-125
```tablegen
//===----------------------------------------------------------------------===//
// Instruction definition
//===----------------------------------------------------------------------===//
let DecoderNamespace = "Mips64" in {
/// Arithmetic Instructions (ALU Immediate)
def DADDi   : ArithLogicI<"daddi", simm16_64, GPR64Opnd>,
              ADDI_FM<0x18>, ISA_MIPS3_NOT_32R6_64R6;
let AdditionalPredicates = [NotInMicroMips] in {
  def DADDiu : ArithLogicI<"daddiu", simm16_64, GPR64Opnd,
                           immSExt16, add>,
               ADDI_FM<0x19>, IsAsCheapAsAMove, ISA_MIPS3;
}
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 127-139
```tablegen
let isCodeGenOnly = 1 in {
def SLTi64  : SetCC_I<"slti", setlt, simm16_64, immSExt16, GPR64Opnd>,
              SLTI_FM<0xa>, GPR_64;
def SLTiu64 : SetCC_I<"sltiu", setult, simm16_64, immSExt16, GPR64Opnd>,
              SLTI_FM<0xb>, GPR_64;
def ANDi64 : ArithLogicI<"andi", uimm16_64, GPR64Opnd, immZExt16, and>,
             ADDI_FM<0xc>, GPR_64;
def ORi64   : ArithLogicI<"ori", uimm16_64, GPR64Opnd, immZExt16, or>,
              ADDI_FM<0xd>, GPR_64;
def XORi64  : ArithLogicI<"xori", uimm16_64, GPR64Opnd, immZExt16, xor>,
              ADDI_FM<0xe>, GPR_64;
def LUi64   : LoadUpper<"lui", GPR64Opnd, uimm16_64_relaxed>, LUI_FM, GPR_64;
}
```
- EN: Defines TableGen record `SLTi64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SLTi64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 141-151
```tablegen
/// Arithmetic Instructions (3-Operand, R-Type)
let AdditionalPredicates = [NotInMicroMips] in {
  def DADD   : ArithLogicR<"dadd", GPR64Opnd, 1>, ADD_FM<0, 0x2c>,
               ISA_MIPS3;
  def DADDu  : ArithLogicR<"daddu", GPR64Opnd, 1, add>,
               ADD_FM<0, 0x2d>, ISA_MIPS3;
  def DSUBu  : ArithLogicR<"dsubu", GPR64Opnd, 0, sub>,
               ADD_FM<0, 0x2f>, ISA_MIPS3;
  def DSUB   : ArithLogicR<"dsub", GPR64Opnd, 0>, ADD_FM<0, 0x2e>,
               ISA_MIPS3;
}
```
- EN: Defines TableGen record `DADD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DADD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 153-163
```tablegen
let isCodeGenOnly = 1 in {
def SLT64  : SetCC_R<"slt", setlt, GPR64Opnd>, ADD_FM<0, 0x2a>, GPR_64;
def SLTu64 : SetCC_R<"sltu", setult, GPR64Opnd>, ADD_FM<0, 0x2b>, GPR_64;
def AND64  : ArithLogicR<"and", GPR64Opnd, 1, and>, ADD_FM<0, 0x24>,
             GPR_64;
def OR64   : ArithLogicR<"or", GPR64Opnd, 1, or>, ADD_FM<0, 0x25>,
             GPR_64;
def XOR64  : ArithLogicR<"xor", GPR64Opnd, 1, xor>, ADD_FM<0, 0x26>,
             GPR_64;
def NOR64  : LogicNOR<"nor", GPR64Opnd>, ADD_FM<0, 0x27>, GPR_64;
}
```
- EN: Defines TableGen record `SLT64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SLT64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 165-182
```tablegen
/// Shift Instructions
let AdditionalPredicates = [NotInMicroMips] in {
  def DSLL : shift_rotate_imm<"dsll", uimm6, GPR64Opnd, mshl_64,
                              immZExt6>,
             SRA_FM<0x38, 0>, ISA_MIPS3;
  def DSRL : shift_rotate_imm<"dsrl", uimm6, GPR64Opnd, msrl_64,
                              immZExt6>,
             SRA_FM<0x3a, 0>, ISA_MIPS3;
  def DSRA : shift_rotate_imm<"dsra", uimm6, GPR64Opnd, msra_64,
                              immZExt6>,
             SRA_FM<0x3b, 0>, ISA_MIPS3;
  def DSLLV  : shift_rotate_reg<"dsllv", GPR64Opnd, mshl_64>,
               SRLV_FM<0x14, 0>, ISA_MIPS3;
  def DSRAV  : shift_rotate_reg<"dsrav", GPR64Opnd, msra_64>,
               SRLV_FM<0x17, 0>, ISA_MIPS3;
  def DSRLV  : shift_rotate_reg<"dsrlv", GPR64Opnd, msrl_64>,
               SRLV_FM<0x16, 0>, ISA_MIPS3;
  def DSLL32 : shift_rotate_imm<"dsll32", uimm5, GPR64Opnd>,
```
- EN: Defines TableGen record `DSLL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSLL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 183-187
```tablegen
               SRA_FM<0x3c, 0>, ISA_MIPS3;
  def DSRL32 : shift_rotate_imm<"dsrl32", uimm5, GPR64Opnd>,
               SRA_FM<0x3e, 0>, ISA_MIPS3;
  def DSRA32 : shift_rotate_imm<"dsra32", uimm5, GPR64Opnd>,
               SRA_FM<0x3f, 0>, ISA_MIPS3;
```
- EN: Defines TableGen record `DSRL32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSRL32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 189-197
```tablegen
// Rotate Instructions
  def DROTR  : shift_rotate_imm<"drotr", uimm6, GPR64Opnd, rotr,
                                immZExt6>,
               SRA_FM<0x3a, 1>, ISA_MIPS64R2;
  def DROTRV : shift_rotate_reg<"drotrv", GPR64Opnd, rotr>,
               SRLV_FM<0x16, 1>, ISA_MIPS64R2;
  def DROTR32 : shift_rotate_imm<"drotr32", uimm5, GPR64Opnd>,
                SRA_FM<0x3e, 1>, ISA_MIPS64R2;
}
```
- EN: Defines TableGen record `DROTR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DROTR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 199-212
```tablegen
/// Load and Store Instructions
///  aligned
let isCodeGenOnly = 1 in {
def LB64  : Load<"lb", GPR64Opnd, sextloadi8>, LW_FM<0x20>, GPR_64;
def LBu64 : Load<"lbu", GPR64Opnd, zextloadi8>, LW_FM<0x24>, GPR_64;
def LH64  : Load<"lh", GPR64Opnd, sextloadi16>, LW_FM<0x21>, GPR_64;
def LHu64 : Load<"lhu", GPR64Opnd, zextloadi16>, LW_FM<0x25>, GPR_64;
def LW64  : Load<"lw", GPR64Opnd, sextloadi32>, LW_FM<0x23>, GPR_64;
def SB64  : Store<"sb", GPR64Opnd, truncstorei8>, LW_FM<0x28>, GPR_64;
def SH64  : Store<"sh", GPR64Opnd, truncstorei16>, LW_FM<0x29>,
            GPR_64;
def SW64  : Store<"sw", GPR64Opnd, truncstorei32>, LW_FM<0x2b>,
            GPR_64;
}
```
- EN: Defines TableGen record `LB64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LB64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 214-221
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def LWu : MMRel, Load<"lwu", GPR64Opnd, zextloadi32>,
            LW_FM<0x27>, ISA_MIPS3;
  def LD  : LoadMemory<"ld", GPR64Opnd, mem_simmptr, load>,
            LW_FM<0x37>, ISA_MIPS3;
  def SD  : StoreMemory<"sd", GPR64Opnd, mem_simmptr, store>,
            LW_FM<0x3f>, ISA_MIPS3;
}
```
- EN: Defines TableGen record `LWu` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWu`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 225-235
```tablegen
/// load/store left/right
let isCodeGenOnly = 1 in {
def LWL64 : LoadLeftRight<"lwl", MipsLWL, GPR64Opnd>, LW_FM<0x22>,
            GPR_64;
def LWR64 : LoadLeftRight<"lwr", MipsLWR, GPR64Opnd>, LW_FM<0x26>,
            GPR_64;
def SWL64 : StoreLeftRight<"swl", MipsSWL, GPR64Opnd>, LW_FM<0x2a>,
            GPR_64;
def SWR64 : StoreLeftRight<"swr", MipsSWR, GPR64Opnd>, LW_FM<0x2e>,
            GPR_64;
}
```
- EN: Defines TableGen record `LWL64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LWL64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 237-244
```tablegen
def LDL   : LoadLeftRight<"ldl", MipsLDL, GPR64Opnd>, LW_FM<0x1a>,
            ISA_MIPS3_NOT_32R6_64R6;
def LDR   : LoadLeftRight<"ldr", MipsLDR, GPR64Opnd>, LW_FM<0x1b>,
            ISA_MIPS3_NOT_32R6_64R6;
def SDL   : StoreLeftRight<"sdl", MipsSDL, GPR64Opnd>, LW_FM<0x2c>,
            ISA_MIPS3_NOT_32R6_64R6;
def SDR   : StoreLeftRight<"sdr", MipsSDR, GPR64Opnd>, LW_FM<0x2d>,
            ISA_MIPS3_NOT_32R6_64R6;
```
- EN: Defines TableGen record `LDL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 246-252
```tablegen
/// Load-linked, Store-conditional
let AdditionalPredicates = [NotInMicroMips] in {
  def LLD : LLBase<"lld", GPR64Opnd, mem_simmptr>,
            LW_FM<0x34>,
            ISA_MIPS3_NOT_32R6_64R6_R5900;
}
def SCD : SCBase<"scd", GPR64Opnd>, LW_FM<0x3c>, ISA_MIPS3_NOT_32R6_64R6_R5900;
```
- EN: Defines TableGen record `LLD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LLD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 254-265
```tablegen
let AdditionalPredicates = [NotInMicroMips],
    DecoderNamespace = "Mips32_64_PTR64" in {
  def LL64 : LLBase<"ll", GPR32Opnd>,
             LW_FM<0x30>,
             PTR_64,
             ISA_MIPS2_NOT_32R6_64R6_R5900;
  def SC64 : SCBase<"sc", GPR32Opnd>,
             LW_FM<0x38>,
             PTR_64,
             ISA_MIPS2_NOT_32R6_64R6_R5900;
  def JR64 : IndirectBranch<"jr", GPR64Opnd>, MTLO_FM<8>, PTR_64;
}
```
- EN: Defines TableGen record `LL64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LL64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 267-267
```tablegen
def JALR64 : JumpLinkReg<"jalr", GPR64Opnd>, JALR_FM, PTR_64;
```
- EN: Defines TableGen record `JALR64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JALR64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 269-286
```tablegen
/// Jump and Branch Instructions
let isCodeGenOnly = 1 in {
  def BEQ64  : CBranch<"beq", brtarget, seteq, GPR64Opnd>, BEQ_FM<4>,
               GPR_64;
  def BNE64  : CBranch<"bne", brtarget, setne, GPR64Opnd>, BEQ_FM<5>,
               GPR_64;
  def BGEZ64 : CBranchZero<"bgez", brtarget, setge, GPR64Opnd>, BGEZ_FM<1, 1>,
               GPR_64;
  def BGTZ64 : CBranchZero<"bgtz", brtarget, setgt, GPR64Opnd>, BGEZ_FM<7, 0>,
               GPR_64;
  def BLEZ64 : CBranchZero<"blez", brtarget, setle, GPR64Opnd>, BGEZ_FM<6, 0>,
               GPR_64;
  def BLTZ64 : CBranchZero<"bltz", brtarget, setlt, GPR64Opnd>, BGEZ_FM<1, 0>,
               GPR_64;
  let AdditionalPredicates = [NoIndirectJumpGuards] in
    def JALR64Pseudo : JumpLinkRegPseudo<GPR64Opnd, JALR, RA, GPR32Opnd>,
                       PTR_64;
}
```
- EN: Defines TableGen record `BEQ64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BEQ64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 287-292
```tablegen
let AdditionalPredicates = [NotInMicroMips],
    DecoderNamespace = "Mips64" in {
  def JR_HB64 : JR_HB_DESC<GPR64Opnd>, JR_HB_ENC, ISA_MIPS64_NOT_64R6;
  def JALR_HB64 : JALR_HB_DESC<GPR64Opnd>, JALR_HB_ENC, ISA_MIPS64R2;
}
def PseudoReturn64 : PseudoReturnBase<GPR64Opnd>, GPR_64;
```
- EN: Defines TableGen record `JR_HB64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JR_HB64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 294-300
```tablegen
let AdditionalPredicates = [NotInMips16Mode, NotInMicroMips,
                            NoIndirectJumpGuards] in {
  def TAILCALLREG64 : TailCallReg<JR64, GPR64Opnd>, ISA_MIPS3_NOT_32R6_64R6,
                      PTR_64;
  def PseudoIndirectBranch64 : PseudoIndirectBranchBase<JR64, GPR64Opnd>,
                               ISA_MIPS3_NOT_32R6_64R6;
}
```
- EN: Defines TableGen record `TAILCALLREG64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILCALLREG64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 302-309
```tablegen
let AdditionalPredicates = [NotInMips16Mode, NotInMicroMips,
                            UseIndirectJumpsHazard] in {
  def TAILCALLREGHB64 : TailCallReg<JR_HB64, GPR64Opnd>,
                        ISA_MIPS32R2_NOT_32R6_64R6, PTR_64;
  def PseudoIndirectHazardBranch64 : PseudoIndirectBranchBase<JR_HB64,
                                                              GPR64Opnd>,
                                     ISA_MIPS32R2_NOT_32R6_64R6, PTR_64;
}
```
- EN: Defines TableGen record `TAILCALLREGHB64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILCALLREGHB64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 311-328
```tablegen
/// Multiply and Divide Instructions.
let AdditionalPredicates = [NotInMicroMips] in {
  def DMULT : Mult<"dmult", GPR64Opnd, [HI0_64, LO0_64]>,
              MULT_FM<0, 0x1c>,
              ISA_MIPS3_NOT_32R6_64R6_R5900;
  def DMULTu : Mult<"dmultu", GPR64Opnd, [HI0_64, LO0_64]>,
               MULT_FM<0, 0x1d>,
               ISA_MIPS3_NOT_32R6_64R6_R5900;
}
def PseudoDMULT : MultDivPseudo<DMULT, ACC128, GPR64Opnd, MipsMult>,
                  ISA_MIPS3_NOT_32R6_64R6_R5900;
def PseudoDMULTu
    : MultDivPseudo<DMULTu, ACC128, GPR64Opnd, MipsMultu>,
      ISA_MIPS3_NOT_32R6_64R6_R5900;
let AdditionalPredicates = [NotInMicroMips] in {
  def DSDIV : Div<"ddiv", GPR64Opnd, [HI0_64, LO0_64]>,
              MULT_FM<0, 0x1e>,
              ISA_MIPS3_NOT_32R6_64R6_R5900;
```
- EN: Defines TableGen record `DMULT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DMULT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 329-338
```tablegen
  def DUDIV : Div<"ddivu", GPR64Opnd, [HI0_64, LO0_64]>,
              MULT_FM<0, 0x1f>,
              ISA_MIPS3_NOT_32R6_64R6_R5900;
}
def PseudoDSDIV
    : MultDivPseudo<DSDIV, ACC128, GPR64Opnd, MipsDivRem, 0, 1, 1>,
      ISA_MIPS3_NOT_32R6_64R6_R5900;
def PseudoDUDIV
    : MultDivPseudo<DUDIV, ACC128, GPR64Opnd, MipsDivRemU, 0, 1, 1>,
      ISA_MIPS3_NOT_32R6_64R6_R5900;
```
- EN: Defines TableGen record `DUDIV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DUDIV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 340-353
```tablegen
let isCodeGenOnly = 1 in {
def MTHI64 : MoveToLOHI<"mthi", GPR64Opnd, [HI0_64]>, MTLO_FM<0x11>,
             ISA_MIPS3_NOT_32R6_64R6;
def MTLO64 : MoveToLOHI<"mtlo", GPR64Opnd, [LO0_64]>, MTLO_FM<0x13>,
             ISA_MIPS3_NOT_32R6_64R6;
def MFHI64 : MoveFromLOHI<"mfhi", GPR64Opnd, AC0_64>, MFLO_FM<0x10>,
             ISA_MIPS3_NOT_32R6_64R6;
def MFLO64 : MoveFromLOHI<"mflo", GPR64Opnd, AC0_64>, MFLO_FM<0x12>,
             ISA_MIPS3_NOT_32R6_64R6;
def PseudoMFHI64 : PseudoMFLOHI<GPR64, ACC128, MipsMFHI>,
                   ISA_MIPS3_NOT_32R6_64R6;
def PseudoMFLO64 : PseudoMFLOHI<GPR64, ACC128, MipsMFLO>,
                   ISA_MIPS3_NOT_32R6_64R6;
def PseudoMTLOHI64 : PseudoMTLOHI<ACC128, GPR64>, ISA_MIPS3_NOT_32R6_64R6;
```
- EN: Defines TableGen record `MTHI64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTHI64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 355-360
```tablegen
/// Sign Ext In Register Instructions.
def SEB64 : SignExtInReg<"seb", i8, GPR64Opnd>, SEB_FM<0x10, 0x20>,
            ISA_MIPS32R2, GPR_64;
def SEH64 : SignExtInReg<"seh", i16, GPR64Opnd>, SEB_FM<0x18, 0x20>,
            ISA_MIPS32R2, GPR_64;
}
```
- EN: Defines TableGen record `SEB64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SEB64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 362-367
```tablegen
/// Count Leading
let AdditionalPredicates = [NotInMicroMips] in {
  def DCLZ : CountLeading0<"dclz", GPR64Opnd>, CLO_FM<0x24>,
             ISA_MIPS64_NOT_64R6, GPR_64;
  def DCLO : CountLeading1<"dclo", GPR64Opnd>, CLO_FM<0x25>,
             ISA_MIPS64_NOT_64R6, GPR_64;
```
- EN: Defines TableGen record `DCLZ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DCLZ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 369-373
```tablegen
/// Double Word Swap Bytes/HalfWords
  def DSBH : SubwordSwap<"dsbh", GPR64Opnd>, SEB_FM<2, 0x24>,
             ISA_MIPS64R2;
  def DSHD : SubwordSwap<"dshd", GPR64Opnd>, SEB_FM<5, 0x24>,
             ISA_MIPS64R2;
```
- EN: Defines TableGen record `DSBH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSBH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 375-377
```tablegen
  def LEA_ADDiu64 : EffectiveAddress<"daddiu", GPR64Opnd>, LW_FM<0x19>,
                    GPR_64;
}
```
- EN: Defines TableGen record `LEA_ADDiu64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LEA_ADDiu64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 379-380
```tablegen
let isCodeGenOnly = 1 in
def RDHWR64 : ReadHardware<GPR64Opnd, HWRegsOpnd>, RDHWR_FM, GPR_64;
```
- EN: Defines TableGen record `RDHWR64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RDHWR64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 382-399
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  // The 'pos + size' constraints for code generation are enforced by the
  // code that lowers into MipsISD::Ext.
  // For assembly parsing, we alias dextu and dextm to dext, and match by
  // operand were possible then check the 'pos + size' in MipsAsmParser.
  // We override the generated decoder to enforce that dext always comes out
  // for dextm and dextu like binutils.
  let DecoderMethod = "DecodeDEXT" in {
    def DEXT : ExtBase<"dext", GPR64Opnd, uimm5_report_uimm6,
                       uimm5_plus1_report_uimm6, immZExt5, immZExt5Plus1,
                       MipsExt>, EXT_FM<3>, ISA_MIPS64R2;
    def DEXTM : ExtBase<"dextm", GPR64Opnd, uimm5, uimm5_plus33, immZExt5,
                        immZExt5Plus33, MipsExt>, EXT_FM<1>, ISA_MIPS64R2;
    def DEXTU : ExtBase<"dextu", GPR64Opnd, uimm5_plus32, uimm5_plus1,
                        immZExt5Plus32, immZExt5Plus1, MipsExt>, EXT_FM<2>,
                        ISA_MIPS64R2;
  }
  // The 'pos + size' constraints for code generation are enforced by the
```
- EN: Defines TableGen record `DEXT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DEXT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 400-416
```tablegen
  // code that lowers into MipsISD::Ins.
  // For assembly parsing, we alias dinsu and dinsm to dins, and match by
  // operand were possible then check the 'pos + size' in MipsAsmParser.
  // We override the generated decoder to enforce that dins always comes out
  // for dinsm and dinsu like binutils.
  let DecoderMethod = "DecodeDINS" in {
    def DINS  : InsBase<"dins", GPR64Opnd, uimm6, uimm5_inssize_plus1,
                        immZExt5, immZExt5Plus1>, EXT_FM<7>,
                ISA_MIPS64R2;
    def DINSU : InsBase<"dinsu", GPR64Opnd, uimm5_plus32, uimm5_inssize_plus1,
                        immZExt5Plus32, immZExt5Plus1>,
                EXT_FM<6>, ISA_MIPS64R2;
    def DINSM : InsBase<"dinsm", GPR64Opnd, uimm5, uimm_range_2_64,
                        immZExt5, immZExtRange2To64>,
                EXT_FM<5>, ISA_MIPS64R2;
  }
}
```
- EN: Defines TableGen record `DINS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DINS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 418-424
```tablegen
let isCodeGenOnly = 1, AdditionalPredicates = [NotInMicroMips] in {
  def DEXT64_32
      : InstSE<(outs GPR64Opnd:$rt),
               (ins GPR32Opnd:$rs, uimm5_report_uimm6:$pos, uimm5_plus1:$size),
               "dext $rt, $rs, $pos, $size", [], FrmR, "dext">,
        EXT_FM<3>, ISA_MIPS64R2;
}
```
- EN: Defines TableGen record `DEXT64_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DEXT64_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 426-435
```tablegen
let isCodeGenOnly = 1, rs = 0, shamt = 0 in {
  def DSLL64_32 : FR<0x00, 0x3c, (outs GPR64:$rd), (ins GPR32:$rt),
                     "dsll\t$rd, $rt, 32", []>, GPR_64;
  let isMoveReg = 1 in {
    def SLL64_32 : FR<0x0, 0x00, (outs GPR64:$rd), (ins GPR32:$rt),
                      "sll\t$rd, $rt, 0", []>, GPR_64;
    def SLL64_64 : FR<0x0, 0x00, (outs GPR64:$rd), (ins GPR64:$rt),
                      "sll\t$rd, $rt, 0", []>, GPR_64;
  }
}
```
- EN: Defines TableGen record `DSLL64_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSLL64_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 437-439
```tablegen
// We need the following pseudo instruction to avoid offset calculation for
// long branches.  See the comment in file MipsLongBranch.cpp for detailed
// explanation.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 441-458
```tablegen
// Expands to: lui $dst, %highest/%higher/%hi/%lo($tgt)
def LONG_BRANCH_LUi2Op_64 :
    PseudoSE<(outs GPR64Opnd:$dst), (ins brtarget:$tgt), []>, GPR_64 {
  bit hasNoSchedulingInfo = 1;
}
// Expands to: addiu $dst, %highest/%higher/%hi/%lo($tgt)
def LONG_BRANCH_DADDiu2Op :
    PseudoSE<(outs GPR64Opnd:$dst), (ins GPR64Opnd:$src, brtarget:$tgt), []>,
    GPR_64 {
  bit hasNoSchedulingInfo = 1;
}
// Expands to: daddiu $dst, $src, %PART($tgt - $baltgt)
// where %PART may be %hi or %lo, depending on the relocation kind
// that $tgt is annotated with.
def LONG_BRANCH_DADDiu :
    PseudoSE<(outs GPR64Opnd:$dst),
             (ins GPR64Opnd:$src, brtarget:$tgt, brtarget:$baltgt), []>,
    GPR_64 {
```
- EN: Defines TableGen record `LONG_BRANCH_LUi2Op_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LONG_BRANCH_LUi2Op_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 459-460
```tablegen
  bit hasNoSchedulingInfo = 1;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 462-465
```tablegen
// Cavium Octeon cnMIPS instructions
let DecoderNamespace = "CnMips",
    // FIXME: The lack of HasStdEnc is probably a bug
    EncodingPredicates = []<Predicate> in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 467-471
```tablegen
class Count1s<string opstr, RegisterOperand RO>:
  InstSE<(outs RO:$rd), (ins RO:$rs), !strconcat(opstr, "\t$rd, $rs"),
         [(set RO:$rd, (ctpop RO:$rs))], FrmR, opstr> {
  let TwoOperandAliasConstraint = "$rd = $rs";
}
```
- EN: Declares reusable TableGen class `Count1s` for `Mips64InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64InstrInfo` 声明可复用的 TableGen 类 `Count1s`，通常用于抽象共享字段、谓词或编码结构。

### Lines 473-480
```tablegen
class ExtsCins<string opstr, RegisterOperand RO,
               PatFrag PosImm, SDPatternOperator Op = null_frag>:
  InstSE<(outs RO:$rt), (ins RO:$rs, uimm5:$pos, uimm5:$lenm1),
         !strconcat(opstr, "\t$rt, $rs, $pos, $lenm1"),
         [(set RO:$rt, (Op RO:$rs, PosImm:$pos, imm:$lenm1))],
         FrmR, opstr> {
  let TwoOperandAliasConstraint = "$rt = $rs";
}
```
- EN: Declares reusable TableGen class `ExtsCins` for `Mips64InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64InstrInfo` 声明可复用的 TableGen 类 `ExtsCins`，通常用于抽象共享字段、谓词或编码结构。

### Lines 482-489
```tablegen
class SetCC64_R<string opstr, PatFrag cond_op> :
  InstSE<(outs GPR64Opnd:$rd), (ins GPR64Opnd:$rs, GPR64Opnd:$rt),
         !strconcat(opstr, "\t$rd, $rs, $rt"),
         [(set GPR64Opnd:$rd, (zext (cond_op GPR64Opnd:$rs,
                                             GPR64Opnd:$rt)))],
         FrmR, opstr> {
  let TwoOperandAliasConstraint = "$rd = $rs";
}
```
- EN: Declares reusable TableGen class `SetCC64_R` for `Mips64InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64InstrInfo` 声明可复用的 TableGen 类 `SetCC64_R`，通常用于抽象共享字段、谓词或编码结构。

### Lines 491-498
```tablegen
class SetCC64_I<string opstr, PatFrag cond_op>:
  InstSE<(outs GPR64Opnd:$rt), (ins GPR64Opnd:$rs, simm10_64:$imm10),
         !strconcat(opstr, "\t$rt, $rs, $imm10"),
         [(set GPR64Opnd:$rt, (zext (cond_op GPR64Opnd:$rs,
                                             immSExt10_64:$imm10)))],
         FrmI, opstr> {
  let TwoOperandAliasConstraint = "$rt = $rs";
}
```
- EN: Declares reusable TableGen class `SetCC64_I` for `Mips64InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64InstrInfo` 声明可复用的 TableGen 类 `SetCC64_I`，通常用于抽象共享字段、谓词或编码结构。

### Lines 500-510
```tablegen
class CBranchBitNum<string opstr, DAGOperand opnd, PatFrag cond_op,
                    RegisterOperand RO, Operand ImmOp, bits<64> shift = 1> :
  InstSE<(outs), (ins RO:$rs, ImmOp:$p, opnd:$offset),
         !strconcat(opstr, "\t$rs, $p, $offset"),
         [(brcond (i32 (cond_op (and RO:$rs, (shl shift, immZExt5_64:$p)), 0)),
                  bb:$offset)], FrmI, opstr> {
  let isBranch = 1;
  let isTerminator = 1;
  let hasDelaySlot = 1;
  let Defs = [AT];
}
```
- EN: Declares reusable TableGen class `CBranchBitNum` for `Mips64InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64InstrInfo` 声明可复用的 TableGen 类 `CBranchBitNum`，通常用于抽象共享字段、谓词或编码结构。

### Lines 512-514
```tablegen
class MFC2OP<string asmstr, RegisterOperand RO> :
  InstSE<(outs RO:$rt, uimm16:$imm16), (ins),
         !strconcat(asmstr, "\t$rt, $imm16"), [], FrmFR>;
```
- EN: Declares reusable TableGen class `MFC2OP` for `Mips64InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64InstrInfo` 声明可复用的 TableGen 类 `MFC2OP`，通常用于抽象共享字段、谓词或编码结构。

### Lines 516-521
```tablegen
// Unsigned Byte Add
def BADDu  : ArithLogicR<"baddu", GPR64Opnd, 1>,
             ADD_FM<0x1c, 0x28>, ASE_CNMIPS {
  let Pattern = [(set GPR64Opnd:$rd,
                      (and (add GPR64Opnd:$rs, GPR64Opnd:$rt), 255))];
}
```
- EN: Defines TableGen record `BADDu` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BADDu`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 523-527
```tablegen
// Branch on Bit Clear /+32
def BBIT0  : CBranchBitNum<"bbit0", brtarget, seteq, GPR64Opnd,
                           uimm5_64_report_uimm6>, BBIT_FM<0x32>, ASE_CNMIPS;
def BBIT032: CBranchBitNum<"bbit032", brtarget, seteq, GPR64Opnd, uimm5_64,
                           0x100000000>, BBIT_FM<0x36>, ASE_CNMIPS;
```
- EN: Defines TableGen record `BBIT0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BBIT0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 529-533
```tablegen
// Branch on Bit Set /+32
def BBIT1  : CBranchBitNum<"bbit1", brtarget, setne, GPR64Opnd,
                           uimm5_64_report_uimm6>, BBIT_FM<0x3a>, ASE_CNMIPS;
def BBIT132: CBranchBitNum<"bbit132", brtarget, setne, GPR64Opnd, uimm5_64,
                           0x100000000>, BBIT_FM<0x3e>, ASE_CNMIPS;
```
- EN: Defines TableGen record `BBIT1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BBIT1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 535-539
```tablegen
// Multiply Doubleword to GPR
def DMUL  : ArithLogicR<"dmul", GPR64Opnd, 1, mul>,
            ADD_FM<0x1c, 0x03>, ASE_CNMIPS {
  let Defs = [HI0, LO0, P0, P1, P2];
}
```
- EN: Defines TableGen record `DMUL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DMUL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 541-546
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  // Extract a signed bit field /+32
  def EXTS  : ExtsCins<"exts", GPR64Opnd, immZExt5>, EXTS_FM<0x3a>,
              ASE_MIPS64_CNMIPS;
  def EXTS32: ExtsCins<"exts32", GPR64Opnd, immZExt5Plus32>,
              EXTS_FM<0x3b>, ASE_MIPS64_CNMIPS;
```
- EN: Defines TableGen record `EXTS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `EXTS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 548-562
```tablegen
  // Clear and insert a bit field /+32
  def CINS  : ExtsCins<"cins", GPR64Opnd, immZExt5, MipsCIns>,
              EXTS_FM<0x32>, ASE_MIPS64_CNMIPS;
  def CINS32: ExtsCins<"cins32", GPR64Opnd, immZExt5Plus32, MipsCIns>,
              EXTS_FM<0x33>, ASE_MIPS64_CNMIPS;
  let isCodeGenOnly = 1 in {
    def CINS_i32 : ExtsCins<"cins", GPR32Opnd, immZExt5, MipsCIns>,
                   EXTS_FM<0x32>, ASE_MIPS64_CNMIPS;
    def CINS64_32 :InstSE<(outs GPR64Opnd:$rt),
                          (ins GPR32Opnd:$rs, uimm5:$pos, uimm5:$lenm1),
                          "cins\t$rt, $rs, $pos, $lenm1", [], FrmR,
                          "cins">,
                   EXTS_FM<0x32>, ASE_MIPS64_CNMIPS;
  }
}
```
- EN: Defines TableGen record `CINS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CINS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 564-573
```tablegen
// Move to multiplier/product register
def MTM0   : MoveToLOHI<"mtm0", GPR64Opnd, [MPL0, P0, P1, P2]>, MTMR_FM<0x08>,
             ASE_CNMIPS;
def MTM1   : MoveToLOHI<"mtm1", GPR64Opnd, [MPL1, P0, P1, P2]>, MTMR_FM<0x0c>,
             ASE_CNMIPS;
def MTM2   : MoveToLOHI<"mtm2", GPR64Opnd, [MPL2, P0, P1, P2]>, MTMR_FM<0x0d>,
             ASE_CNMIPS;
def MTP0   : MoveToLOHI<"mtp0", GPR64Opnd, [P0]>, MTMR_FM<0x09>, ASE_CNMIPS;
def MTP1   : MoveToLOHI<"mtp1", GPR64Opnd, [P1]>, MTMR_FM<0x0a>, ASE_CNMIPS;
def MTP2   : MoveToLOHI<"mtp2", GPR64Opnd, [P2]>, MTMR_FM<0x0b>, ASE_CNMIPS;
```
- EN: Defines TableGen record `MTM0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MTM0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 575-577
```tablegen
// Count Ones in a Word/Doubleword
def POP   : Count1s<"pop", GPR32Opnd>, POP_FM<0x2c>, ASE_CNMIPS;
def DPOP  : Count1s<"dpop", GPR64Opnd>, POP_FM<0x2d>, ASE_CNMIPS;
```
- EN: Defines TableGen record `POP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `POP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 579-583
```tablegen
// Set on equal/not equal
def SEQ   : SetCC64_R<"seq", seteq>, SEQ_FM<0x2a>, ASE_CNMIPS;
def SEQi  : SetCC64_I<"seqi", seteq>, SEQI_FM<0x2e>, ASE_CNMIPS;
def SNE   : SetCC64_R<"sne", setne>, SEQ_FM<0x2b>, ASE_CNMIPS;
def SNEi  : SetCC64_I<"snei", setne>, SEQI_FM<0x2f>, ASE_CNMIPS;
```
- EN: Defines TableGen record `SEQ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SEQ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 585-589
```tablegen
// 192-bit x 64-bit Unsigned Multiply and Add
def V3MULU: ArithLogicR<"v3mulu", GPR64Opnd, 0>, ADD_FM<0x1c, 0x11>,
            ASE_CNMIPS {
  let Defs = [P0, P1, P2];
}
```
- EN: Defines TableGen record `V3MULU:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `V3MULU:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 591-595
```tablegen
// 64-bit Unsigned Multiply and Add Move
def VMM0  : ArithLogicR<"vmm0", GPR64Opnd, 0>, ADD_FM<0x1c, 0x10>,
            ASE_CNMIPS {
  let Defs = [MPL0, P0, P1, P2];
}
```
- EN: Defines TableGen record `VMM0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMM0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 597-601
```tablegen
// 64-bit Unsigned Multiply and Add
def VMULU : ArithLogicR<"vmulu", GPR64Opnd, 0>, ADD_FM<0x1c, 0x0f>,
            ASE_CNMIPS {
  let Defs = [MPL1, MPL2, P0, P1, P2];
}
```
- EN: Defines TableGen record `VMULU` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `VMULU`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 603-608
```tablegen
// Move between CPU and coprocessor registers
def DMFC2_OCTEON : MFC2OP<"dmfc2", GPR64Opnd>, MFC2OP_FM<0x12, 1>,
                   ASE_CNMIPS;
def DMTC2_OCTEON : MFC2OP<"dmtc2", GPR64Opnd>, MFC2OP_FM<0x12, 5>,
                   ASE_CNMIPS;
}
```
- EN: Defines TableGen record `DMFC2_OCTEON` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DMFC2_OCTEON`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 610-613
```tablegen
// Cavium Octeon+ cnMIPS instructions
let DecoderNamespace = "CnMipsP",
    // FIXME: The lack of HasStdEnc is probably a bug
    EncodingPredicates = []<Predicate> in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 615-617
```tablegen
class Saa<string opstr>:
  InstSE<(outs), (ins GPR64Opnd:$rt, GPR64Opnd:$rs),
         !strconcat(opstr, "\t$rt, (${rs})"), [], FrmR, opstr>;
```
- EN: Declares reusable TableGen class `Saa` for `Mips64InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64InstrInfo` 声明可复用的 TableGen 类 `Saa`，通常用于抽象共享字段、谓词或编码结构。

### Lines 619-620
```tablegen
def SAA  : Saa<"saa">,  SAA_FM<0x18>, ASE_CNMIPSP;
def SAAD : Saa<"saad">, SAA_FM<0x19>, ASE_CNMIPSP;
```
- EN: Defines TableGen record `SAA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SAA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 622-626
```tablegen
def SaaAddr  : MipsAsmPseudoInst<(outs), (ins GPR64Opnd:$rt, mem:$addr),
                                 "saa\t$rt, $addr">, ASE_CNMIPSP;
def SaadAddr : MipsAsmPseudoInst<(outs), (ins GPR64Opnd:$rt, mem:$addr),
                                 "saad\t$rt, $addr">, ASE_CNMIPSP;
}
```
- EN: Defines TableGen record `SaaAddr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SaaAddr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 628-628
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 630-640
```tablegen
/// Move between CPU and coprocessor registers
let DecoderNamespace = "Mips64" in {
def DMFC0 : MFC3OP<"dmfc0", GPR64Opnd, COP0Opnd>,
            MFC3OP_FM<0x10, 1, 0>, ISA_MIPS3, GPR_64;
def DMTC0 : MTC3OP<"dmtc0", COP0Opnd, GPR64Opnd>,
            MFC3OP_FM<0x10, 5, 0>, ISA_MIPS3, GPR_64;
def DMFC2 : MFC3OP<"dmfc2", GPR64Opnd, COP2Opnd>,
            MFC3OP_FM<0x12, 1, 0>, ISA_MIPS3, GPR_64;
def DMTC2 : MTC3OP<"dmtc2", COP2Opnd, GPR64Opnd>,
            MFC3OP_FM<0x12, 5, 0>, ISA_MIPS3, GPR_64;
}
```
- EN: Defines TableGen record `DMFC0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DMFC0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 642-648
```tablegen
/// Move between CPU and guest coprocessor registers (Virtualization ASE)
let DecoderNamespace = "Mips64" in {
  def DMFGC0 : MFC3OP<"dmfgc0", GPR64Opnd, COP0Opnd>,
               MFC3OP_FM<0x10, 3, 1>, ISA_MIPS64R5, ASE_VIRT;
  def DMTGC0 : MTC3OP<"dmtgc0", COP0Opnd, GPR64Opnd>,
               MFC3OP_FM<0x10, 3, 3>, ISA_MIPS64R5, ASE_VIRT;
}
```
- EN: Defines TableGen record `DMFGC0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DMFGC0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 650-651
```tablegen
let AdditionalPredicates = [UseIndirectJumpsHazard] in
  def JALRHB64Pseudo : JumpLinkRegPseudo<GPR64Opnd, JALR_HB64, RA_64>, PTR_64;
```
- EN: Defines TableGen record `JALRHB64Pseudo` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JALRHB64Pseudo`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 653-655
```tablegen
//===----------------------------------------------------------------------===//
//  Arbitrary patterns that map to one or more instructions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 657-658
```tablegen
// Materialize i64 constants.
defm : MaterializeImms<i64, ZERO_64, DADDiu, LUi64, ORi64>, ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 660-661
```tablegen
def : MipsPat<(i64 immZExt32Low16Zero:$imm),
              (DSLL (ORi64 ZERO_64, (HI16 imm:$imm)), 16)>, ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 663-665
```tablegen
def : MipsPat<(i64 immZExt32:$imm),
              (ORi64 (DSLL (ORi64 ZERO_64, (HI16 imm:$imm)), 16),
                     (LO16 imm:$imm))>, ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 667-675
```tablegen
// extended loads
def : MipsPat<(i64 (extloadi1  addr:$src)), (LB64 addr:$src)>, ISA_MIPS3,
      GPR_64;
def : MipsPat<(i64 (extloadi8  addr:$src)), (LB64 addr:$src)>, ISA_MIPS3,
      GPR_64;
def : MipsPat<(i64 (extloadi16 addr:$src)), (LH64 addr:$src)>, ISA_MIPS3,
      GPR_64;
def : MipsPat<(i64 (extloadi32 addr:$src)), (LW64 addr:$src)>, ISA_MIPS3,
      GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 677-680
```tablegen
// hi/lo relocs
let AdditionalPredicates = [NotInMicroMips] in
defm : MipsHiLoRelocs<LUi64, DADDiu, ZERO_64, GPR64Opnd>, ISA_MIPS3, GPR_64,
       SYM_32;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 682-685
```tablegen
def : MipsPat<(MipsGotHi tglobaladdr:$in), (LUi64 tglobaladdr:$in)>, ISA_MIPS3,
      GPR_64;
def : MipsPat<(MipsGotHi texternalsym:$in), (LUi64 texternalsym:$in)>,
      ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 687-688
```tablegen
def : MipsPat<(MipsTlsHi tglobaltlsaddr:$in), (LUi64 tglobaltlsaddr:$in)>,
      ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 690-693
```tablegen
// highest/higher/hi/lo relocs
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsPat<(MipsJmpLink (i64 texternalsym:$dst)),
                (JAL texternalsym:$dst)>, ISA_MIPS3, GPR_64, SYM_64;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 695-704
```tablegen
  def : MipsPat<(MipsHighest (i64 tglobaladdr:$in)),
                (LUi64 tglobaladdr:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHighest (i64 tblockaddress:$in)),
                (LUi64 tblockaddress:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHighest (i64 tjumptable:$in)),
                (LUi64 tjumptable:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHighest (i64 tconstpool:$in)),
                (LUi64 tconstpool:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHighest (i64 texternalsym:$in)),
                (LUi64 texternalsym:$in)>, ISA_MIPS3, GPR_64, SYM_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 706-715
```tablegen
  def : MipsPat<(MipsHigher (i64 tglobaladdr:$in)),
                (DADDiu ZERO_64, tglobaladdr:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHigher (i64 tblockaddress:$in)),
                (DADDiu ZERO_64, tblockaddress:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHigher (i64 tjumptable:$in)),
                (DADDiu ZERO_64, tjumptable:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHigher (i64 tconstpool:$in)),
                (DADDiu ZERO_64, tconstpool:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHigher (i64 texternalsym:$in)),
                (DADDiu ZERO_64, texternalsym:$in)>, ISA_MIPS3, GPR_64, SYM_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 717-728
```tablegen
  def : MipsPat<(add GPR64:$hi, (MipsHigher (i64 tglobaladdr:$lo))),
                (DADDiu GPR64:$hi, tglobaladdr:$lo)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsHigher (i64 tblockaddress:$lo))),
                (DADDiu GPR64:$hi, tblockaddress:$lo)>, ISA_MIPS3, GPR_64,
                SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsHigher (i64 tjumptable:$lo))),
                (DADDiu GPR64:$hi, tjumptable:$lo)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsHigher (i64 tconstpool:$lo))),
                (DADDiu GPR64:$hi, tconstpool:$lo)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsHigher (i64 texternalsym:$lo))),
                (DADDiu GPR64:$hi, texternalsym:$lo)>,
                ISA_MIPS3, GPR_64, SYM_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 730-739
```tablegen
  def : MipsPat<(MipsHi (i64 tglobaladdr:$in)),
                (DADDiu ZERO_64, tglobaladdr:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHi (i64 tblockaddress:$in)),
                (DADDiu ZERO_64, tblockaddress:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHi (i64 tjumptable:$in)),
                (DADDiu ZERO_64, tjumptable:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHi (i64 tconstpool:$in)),
                (DADDiu ZERO_64, tconstpool:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsHi (i64 texternalsym:$in)),
                (DADDiu ZERO_64, texternalsym:$in)>, ISA_MIPS3, GPR_64, SYM_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 741-752
```tablegen
  def : MipsPat<(add GPR64:$hi, (MipsHi (i64 tglobaladdr:$lo))),
                (DADDiu GPR64:$hi, tglobaladdr:$lo)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsHi (i64 tblockaddress:$lo))),
                (DADDiu GPR64:$hi, tblockaddress:$lo)>, ISA_MIPS3, GPR_64,
                SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsHi (i64 tjumptable:$lo))),
                (DADDiu GPR64:$hi, tjumptable:$lo)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsHi (i64 tconstpool:$lo))),
                (DADDiu GPR64:$hi, tconstpool:$lo)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsHi (i64 texternalsym:$lo))),
                (DADDiu GPR64:$hi, texternalsym:$lo)>,
                ISA_MIPS3, GPR_64, SYM_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 754-766
```tablegen
  def : MipsPat<(MipsLo (i64 tglobaladdr:$in)),
                (DADDiu ZERO_64, tglobaladdr:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsLo (i64 tblockaddress:$in)),
                (DADDiu ZERO_64, tblockaddress:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsLo (i64 tjumptable:$in)),
                (DADDiu ZERO_64, tjumptable:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsLo (i64 tconstpool:$in)),
                (DADDiu ZERO_64, tconstpool:$in)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsLo (i64 tglobaltlsaddr:$in)),
                (DADDiu ZERO_64, tglobaltlsaddr:$in)>,
                ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(MipsLo (i64 texternalsym:$in)),
                (DADDiu ZERO_64, texternalsym:$in)>, ISA_MIPS3, GPR_64, SYM_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 768-783
```tablegen
  def : MipsPat<(add GPR64:$hi, (MipsLo (i64 tglobaladdr:$lo))),
                (DADDiu GPR64:$hi, tglobaladdr:$lo)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsLo (i64 tblockaddress:$lo))),
                (DADDiu GPR64:$hi, tblockaddress:$lo)>, ISA_MIPS3, GPR_64,
                SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsLo (i64 tjumptable:$lo))),
                (DADDiu GPR64:$hi, tjumptable:$lo)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsLo (i64 tconstpool:$lo))),
                (DADDiu GPR64:$hi, tconstpool:$lo)>, ISA_MIPS3, GPR_64, SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsLo (i64 tglobaltlsaddr:$lo))),
                (DADDiu GPR64:$hi, tglobaltlsaddr:$lo)>, ISA_MIPS3, GPR_64,
                SYM_64;
  def : MipsPat<(add GPR64:$hi, (MipsLo (i64 texternalsym:$lo))),
                (DADDiu GPR64:$hi, texternalsym:$lo)>,
                ISA_MIPS3, GPR_64, SYM_64;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 785-786
```tablegen
def : MipsPat<(MipsGPRel tglobaladdr:$in),
              (DADDiu ZERO_64, tglobaladdr:$in)>, ISA_MIPS3, ABI_N64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 788-792
```tablegen
// gp_rel relocs
def : MipsPat<(add GPR64:$gp, (MipsGPRel tglobaladdr:$in)),
              (DADDiu GPR64:$gp, tglobaladdr:$in)>, ISA_MIPS3, ABI_N64;
def : MipsPat<(add GPR64:$gp, (MipsGPRel tconstpool:$in)),
              (DADDiu GPR64:$gp, tconstpool:$in)>, ISA_MIPS3, ABI_N64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 794-799
```tablegen
def : WrapperPat<tglobaladdr, DADDiu, GPR64>, ISA_MIPS3, GPR_64;
def : WrapperPat<tconstpool, DADDiu, GPR64>, ISA_MIPS3, GPR_64;
def : WrapperPat<texternalsym, DADDiu, GPR64>, ISA_MIPS3, GPR_64;
def : WrapperPat<tblockaddress, DADDiu, GPR64>, ISA_MIPS3, GPR_64;
def : WrapperPat<tjumptable, DADDiu, GPR64>, ISA_MIPS3, GPR_64;
def : WrapperPat<tglobaltlsaddr, DADDiu, GPR64>, ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 802-807
```tablegen
defm : BrcondPats<GPR64, BEQ64, BEQ, BNE64, SLT64, SLTu64, SLTi64, SLTiu64,
                  ZERO_64>, ISA_MIPS3, GPR_64;
def : MipsPat<(brcond (i32 (setlt i64:$lhs, 1)), bb:$dst),
              (BLEZ64 i64:$lhs, bb:$dst)>, ISA_MIPS3, GPR_64;
def : MipsPat<(brcond (i32 (setgt i64:$lhs, -1)), bb:$dst),
              (BGEZ64 i64:$lhs, bb:$dst)>, ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 809-826
```tablegen
// setcc patterns
let AdditionalPredicates = [NotInMicroMips] in {
  defm : SeteqPats<GPR64, SLTiu64, XOR64, SLTu64, ZERO_64>, ISA_MIPS3, GPR_64;
  defm : SetlePats<GPR64, XORi, SLT64, SLTu64>, ISA_MIPS3, GPR_64;
  defm : SetgtPats<GPR64, SLT64, SLTu64>, ISA_MIPS3, GPR_64;
  defm : SetgePats<GPR64, XORi, SLT64, SLTu64>, ISA_MIPS3, GPR_64;
  defm : SetgeImmPats<GPR64, XORi, SLTi64, SLTiu64>, ISA_MIPS3, GPR_64;
}
// truncate
def : MipsPat<(trunc (assertsext GPR64:$src)),
              (EXTRACT_SUBREG GPR64:$src, sub_32)>, ISA_MIPS3, GPR_64;
// The forward compatibility strategy employed by MIPS requires us to treat
// values as being sign extended to an infinite number of bits. This allows
// existing software to run without modification on any future MIPS
// implementation (e.g. 128-bit, or 1024-bit). Being compatible with this
// strategy requires that truncation acts as a sign-extension for values being
// fed into instructions operating on 32-bit values. Such instructions have
// undefined results if this is not true.
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 827-833
```tablegen
// For our case, this means that we can't issue an extract_subreg for nodes
// such as (trunc:i32 (assertzext:i64 X, i32)), because the sign-bit of the
// lower subreg would not be replicated into the upper half.
def : MipsPat<(trunc (assertzext_lt_i32 GPR64:$src)),
              (EXTRACT_SUBREG GPR64:$src, sub_32)>, ISA_MIPS3, GPR_64;
def : MipsPat<(i32 (trunc GPR64:$src)),
              (SLL (EXTRACT_SUBREG GPR64:$src, sub_32), 0)>, ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 835-852
```tablegen
// variable shift instructions patterns
def : MipsPat<(shl GPR64:$rt, (i32 (trunc GPR64:$rs))),
              (DSLLV GPR64:$rt, (EXTRACT_SUBREG GPR64:$rs, sub_32))>,
              ISA_MIPS3, GPR_64;
def : MipsPat<(srl GPR64:$rt, (i32 (trunc GPR64:$rs))),
              (DSRLV GPR64:$rt, (EXTRACT_SUBREG GPR64:$rs, sub_32))>,
              ISA_MIPS3, GPR_64;
def : MipsPat<(sra GPR64:$rt, (i32 (trunc GPR64:$rs))),
              (DSRAV GPR64:$rt, (EXTRACT_SUBREG GPR64:$rs, sub_32))>,
              ISA_MIPS3, GPR_64;
def : MipsPat<(rotr GPR64:$rt, (i32 (trunc GPR64:$rs))),
              (DROTRV GPR64:$rt, (EXTRACT_SUBREG GPR64:$rs, sub_32))>,
              ISA_MIPS3, GPR_64;
def : MipsPat<(and (srl (i32 (trunc GPR64:$src)), immZExt5:$imm5), immZExt16:$value),
              (ANDi (EXTRACT_SUBREG (DSRL GPR64:$src, immZExt5:$imm5), sub_32), immZExt16:$value)>,
              ISA_MIPS3, GPR_64;
def : MipsPat<(MipsExt (i32 (trunc GPR64:$src)), immZExt5:$pos, immZExt5:$size),
              (EXTRACT_SUBREG (DEXT GPR64:$src, immZExt5:$pos, immZExt5:$size), sub_32)>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 853-853
```tablegen
              ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 855-862
```tablegen
// 32-to-64-bit extension
def : MipsPat<(i64 (anyext GPR32:$src)),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)), GPR32:$src, sub_32)>,
      ISA_MIPS3, GPR_64;
def : MipsPat<(i64 (zext GPR32:$src)), (DSRL (DSLL64_32 GPR32:$src), 32)>,
      ISA_MIPS3, GPR_64;
def : MipsPat<(i64 (sext_like GPR32:$src)), (SLL64_32 GPR32:$src)>, ISA_MIPS3,
      GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 864-870
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsPat<(i64 (zext GPR32:$src)), (DEXT64_32 GPR32:$src, 0, 32)>,
        ISA_MIPS64R2, GPR_64;
  def : MipsPat<(i64 (zext (i32 (shl GPR32:$rt, immZExt5:$imm)))),
                (CINS64_32 GPR32:$rt, imm:$imm, (immZExt5To31 imm:$imm))>,
        ISA_MIPS64R2, GPR_64, ASE_MIPS64_CNMIPS;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 872-874
```tablegen
// Sign extend in register
def : MipsPat<(i64 (sext_inreg GPR64:$src, i32)),
              (SLL64_64 GPR64:$src)>, ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 876-877
```tablegen
// bswap MipsPattern
def : MipsPat<(bswap GPR64:$rt), (DSHD (DSBH GPR64:$rt))>, ISA_MIPS64R2;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 879-887
```tablegen
// Carry pattern
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsPat<(subc GPR64:$lhs, GPR64:$rhs),
                (DSUBu GPR64:$lhs, GPR64:$rhs)>, ISA_MIPS3, GPR_64;
  def : MipsPat<(addc GPR64:$lhs, GPR64:$rhs),
                (DADDu GPR64:$lhs, GPR64:$rhs)>, ISA_MIPS3, ASE_NOT_DSP, GPR_64;
  def : MipsPat<(addc GPR64:$lhs, immSExt16:$imm),
                (DADDiu GPR64:$lhs, imm:$imm)>, ISA_MIPS3, ASE_NOT_DSP, GPR_64;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 889-906
```tablegen
// Octeon bbit0/bbit1 MipsPattern
def : MipsPat<(brcond (i32 (seteq (and i64:$lhs, PowerOf2LO:$mask), 0)), bb:$dst),
              (BBIT0 i64:$lhs, (Log2LO PowerOf2LO:$mask), bb:$dst)>,
              ISA_MIPS64R2, ASE_MIPS64_CNMIPS;
def : MipsPat<(brcond (i32 (seteq (and i64:$lhs, PowerOf2HI:$mask), 0)), bb:$dst),
              (BBIT032 i64:$lhs, (Log2HI PowerOf2HI:$mask), bb:$dst)>,
              ISA_MIPS64R2, ASE_MIPS64_CNMIPS;
def : MipsPat<(brcond (i32 (setne (and i64:$lhs, PowerOf2LO:$mask), 0)), bb:$dst),
              (BBIT1 i64:$lhs, (Log2LO PowerOf2LO:$mask), bb:$dst)>,
              ISA_MIPS64R2, ASE_MIPS64_CNMIPS;
def : MipsPat<(brcond (i32 (setne (and i64:$lhs, PowerOf2HI:$mask), 0)), bb:$dst),
              (BBIT132 i64:$lhs, (Log2HI PowerOf2HI:$mask), bb:$dst)>,
              ISA_MIPS64R2, ASE_MIPS64_CNMIPS;
def : MipsPat<(brcond (i32 (seteq (and i32:$lhs, PowerOf2LO_i32:$mask), 0)), bb:$dst),
              (BBIT0 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), i32:$lhs, sub_32),
                     (Log2LO PowerOf2LO_i32:$mask), bb:$dst)>, ISA_MIPS64R2,
      ASE_MIPS64_CNMIPS;
def : MipsPat<(brcond (i32 (setne (and i32:$lhs, PowerOf2LO_i32:$mask), 0)), bb:$dst),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 907-909
```tablegen
              (BBIT1 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), i32:$lhs, sub_32),
                     (Log2LO PowerOf2LO_i32:$mask), bb:$dst)>, ISA_MIPS64R2,
      ASE_MIPS64_CNMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 911-915
```tablegen
// Atomic load patterns.
def : MipsPat<(atomic_load_asext_8 addr:$a), (LB64 addr:$a)>, ISA_MIPS3, GPR_64;
def : MipsPat<(atomic_load_asext_16 addr:$a), (LH64 addr:$a)>, ISA_MIPS3, GPR_64;
def : MipsPat<(atomic_load_asext_32 addr:$a), (LW64 addr:$a)>, ISA_MIPS3, GPR_64;
def : MipsPat<(atomic_load_nonext_64 addr:$a), (LD addr:$a)>, ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 917-925
```tablegen
// Atomic store patterns.
def : MipsPat<(atomic_store_8 GPR64:$v, addr:$a), (SB64 GPR64:$v, addr:$a)>,
      ISA_MIPS3, GPR_64;
def : MipsPat<(atomic_store_16 GPR64:$v, addr:$a), (SH64 GPR64:$v, addr:$a)>,
      ISA_MIPS3, GPR_64;
def : MipsPat<(atomic_store_32 GPR64:$v, addr:$a), (SW64 GPR64:$v, addr:$a)>,
      ISA_MIPS3, GPR_64;
def : MipsPat<(atomic_store_64 GPR64:$v, addr:$a), (SD GPR64:$v, addr:$a)>,
      ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 927-944
```tablegen
// Patterns used for matching away redundant sign extensions.
// MIPS32 arithmetic instructions sign extend their result implicitly.
def : MipsPat<(i64 (sext_like (i32 (add GPR32:$src, immSExt16:$imm16)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (ADDiu GPR32:$src, immSExt16:$imm16), sub_32)>;
def : MipsPat<(i64 (sext_like (i32 (add GPR32:$src, GPR32:$src2)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (ADDu GPR32:$src, GPR32:$src2), sub_32)>;
def : MipsPat<(i64 (sext_like (i32 (sub GPR32:$src, GPR32:$src2)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (SUBu GPR32:$src, GPR32:$src2), sub_32)>;
def : MipsPat<(i64 (sext_like (i32 (mul GPR32:$src, GPR32:$src2)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (MUL GPR32:$src, GPR32:$src2), sub_32)>, ISA_MIPS32_NOT_32R6_64R6;
def : MipsPat<(i64 (sext_like (i32 (MipsMFHI ACC64:$src)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (PseudoMFHI ACC64:$src), sub_32)>;
def : MipsPat<(i64 (sext_like (i32 (MipsMFLO ACC64:$src)))),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 945-962
```tablegen
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (PseudoMFLO ACC64:$src), sub_32)>;
def : MipsPat<(i64 (sext_like (i32 (shl GPR32:$src, immZExt5:$imm5)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (SLL GPR32:$src, immZExt5:$imm5), sub_32)>;
def : MipsPat<(i64 (sext_like (i32 (shl GPR32:$src, GPR32:$src2)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (SLLV GPR32:$src, GPR32:$src2), sub_32)>;
def : MipsPat<(i64 (sext_like (i32 (srl GPR32:$src, immZExt5:$imm5)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (SRL GPR32:$src, immZExt5:$imm5), sub_32)>;
def : MipsPat<(i64 (sext_like (i32 (srl GPR32:$src, GPR32:$src2)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (SRLV GPR32:$src, GPR32:$src2), sub_32)>;
def : MipsPat<(i64 (sext_like (i32 (sra GPR32:$src, immZExt5:$imm5)))),
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (SRA GPR32:$src, immZExt5:$imm5), sub_32)>;
def : MipsPat<(i64 (sext_like (i32 (sra GPR32:$src, GPR32:$src2)))),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 963-964
```tablegen
              (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
              (SRAV GPR32:$src, GPR32:$src2), sub_32)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 966-983
```tablegen
//===----------------------------------------------------------------------===//
// Instruction aliases
//===----------------------------------------------------------------------===//
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsInstAlias<"move $dst, $src",
                      (OR64 GPR64Opnd:$dst,  GPR64Opnd:$src, ZERO_64), 1>,
        GPR_64;
  def : MipsInstAlias<"move $dst, $src",
                      (DADDu GPR64Opnd:$dst,  GPR64Opnd:$src, ZERO_64), 1>,
        GPR_64;
  def : MipsInstAlias<"dadd $rs, $rt, $imm",
                      (DADDi GPR64Opnd:$rs, GPR64Opnd:$rt, simm16_64:$imm),
                      0>, ISA_MIPS3_NOT_32R6_64R6;
  def : MipsInstAlias<"dadd $rs, $imm",
                      (DADDi GPR64Opnd:$rs, GPR64Opnd:$rs, simm16_64:$imm),
                      0>, ISA_MIPS3_NOT_32R6_64R6;
  def : MipsInstAlias<"daddu $rs, $rt, $imm",
                      (DADDiu GPR64Opnd:$rs, GPR64Opnd:$rt, simm16_64:$imm),
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 984-987
```tablegen
                      0>, ISA_MIPS3;
  def : MipsInstAlias<"daddu $rs, $imm",
                      (DADDiu GPR64Opnd:$rs, GPR64Opnd:$rs, simm16_64:$imm),
                      0>, ISA_MIPS3;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 989-990
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"and", ANDi64, GPR64Opnd, imm64>,
         ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 992-993
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"or", ORi64, GPR64Opnd, imm64>,
         ISA_MIPS3, GPR_64;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 995-1012
```tablegen
  defm : OneOrTwoOperandMacroImmediateAlias<"xor", XORi64, GPR64Opnd, imm64>,
         ISA_MIPS3, GPR_64;
}
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsInstAlias<"dneg $rt, $rs",
                      (DSUB GPR64Opnd:$rt, ZERO_64, GPR64Opnd:$rs), 1>,
                      ISA_MIPS3;
  def : MipsInstAlias<"dneg $rt",
                      (DSUB GPR64Opnd:$rt, ZERO_64, GPR64Opnd:$rt), 1>,
                      ISA_MIPS3;
  def : MipsInstAlias<"dnegu $rt, $rs",
                      (DSUBu GPR64Opnd:$rt, ZERO_64, GPR64Opnd:$rs), 1>,
                      ISA_MIPS3;
  def : MipsInstAlias<"dnegu $rt",
                      (DSUBu GPR64Opnd:$rt, ZERO_64, GPR64Opnd:$rt), 1>,
                      ISA_MIPS3;
}
def : MipsInstAlias<"dsubi $rs, $rt, $imm",
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1013-1030
```tablegen
                    (DADDi GPR64Opnd:$rs, GPR64Opnd:$rt,
                           InvertedImOperand64:$imm),
                    0>, ISA_MIPS3_NOT_32R6_64R6;
def : MipsInstAlias<"dsubi $rs, $imm",
                    (DADDi GPR64Opnd:$rs, GPR64Opnd:$rs,
                           InvertedImOperand64:$imm),
                    0>, ISA_MIPS3_NOT_32R6_64R6;
def : MipsInstAlias<"dsub $rs, $rt, $imm",
                    (DADDi GPR64Opnd:$rs, GPR64Opnd:$rt,
                           InvertedImOperand64:$imm),
                    0>, ISA_MIPS3_NOT_32R6_64R6;
def : MipsInstAlias<"dsub $rs, $imm",
                    (DADDi GPR64Opnd:$rs, GPR64Opnd:$rs,
                           InvertedImOperand64:$imm),
                    0>, ISA_MIPS3_NOT_32R6_64R6;
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsInstAlias<"dsubu $rt, $rs, $imm",
                      (DADDiu GPR64Opnd:$rt, GPR64Opnd:$rs,
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1031-1048
```tablegen
                              InvertedImOperand64:$imm), 0>, ISA_MIPS3;
  def : MipsInstAlias<"dsubu $rs, $imm",
                      (DADDiu GPR64Opnd:$rs, GPR64Opnd:$rs,
                              InvertedImOperand64:$imm), 0>, ISA_MIPS3;
}
def : MipsInstAlias<"dsra $rd, $rt, $rs",
                    (DSRAV GPR64Opnd:$rd, GPR64Opnd:$rt, GPR32Opnd:$rs), 0>,
                    ISA_MIPS3;
let AdditionalPredicates = [NotInMicroMips] in {
  def : MipsInstAlias<"dsll $rd, $rt, $rs",
                      (DSLLV GPR64Opnd:$rd, GPR64Opnd:$rt, GPR32Opnd:$rs), 0>,
                      ISA_MIPS3;
  def : MipsInstAlias<"dsrl $rd, $rt, $rs",
                      (DSRLV GPR64Opnd:$rd, GPR64Opnd:$rt, GPR32Opnd:$rs), 0>,
                      ISA_MIPS3;
  def : MipsInstAlias<"dsrl $rd, $rt",
                      (DSRLV GPR64Opnd:$rd, GPR64Opnd:$rd, GPR32Opnd:$rt), 0>,
                      ISA_MIPS3;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1049-1066
```tablegen
  def : MipsInstAlias<"dsll $rd, $rt",
                      (DSLLV GPR64Opnd:$rd, GPR64Opnd:$rd, GPR32Opnd:$rt), 0>,
                      ISA_MIPS3;
  def : MipsInstAlias<"dins $rt, $rs, $pos, $size",
                      (DINSM GPR64Opnd:$rt, GPR64Opnd:$rs, uimm5:$pos,
                             uimm_range_2_64:$size), 0>, ISA_MIPS64R2;
  def : MipsInstAlias<"dins $rt, $rs, $pos, $size",
                      (DINSU GPR64Opnd:$rt, GPR64Opnd:$rs, uimm5_plus32:$pos,
                             uimm5_plus1:$size), 0>, ISA_MIPS64R2;
  def : MipsInstAlias<"dext $rt, $rs, $pos, $size",
                      (DEXTM GPR64Opnd:$rt, GPR64Opnd:$rs, uimm5:$pos,
                             uimm5_plus33:$size), 0>, ISA_MIPS64R2;
  def : MipsInstAlias<"dext $rt, $rs, $pos, $size",
                      (DEXTU GPR64Opnd:$rt, GPR64Opnd:$rs, uimm5_plus32:$pos,
                             uimm5_plus1:$size), 0>, ISA_MIPS64R2;
  def : MipsInstAlias<"jalr.hb $rs", (JALR_HB64 RA_64, GPR64Opnd:$rs), 1>,
        ISA_MIPS64;
// Two operand (implicit 0 selector) versions:
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1067-1081
```tablegen
  def : MipsInstAlias<"dmtc0 $rt, $rd",
                      (DMTC0 COP0Opnd:$rd, GPR64Opnd:$rt, 0), 0>;
  def : MipsInstAlias<"dmfc0 $rt, $rd",
                      (DMFC0 GPR64Opnd:$rt, COP0Opnd:$rd, 0), 0>;
  def : MipsInstAlias<"dmfgc0 $rt, $rd",
                      (DMFGC0 GPR64Opnd:$rt, COP0Opnd:$rd, 0), 0>,
                      ISA_MIPS64R5, ASE_VIRT;
  def : MipsInstAlias<"dmtgc0 $rt, $rd",
                      (DMTGC0 COP0Opnd:$rd, GPR64Opnd:$rt, 0), 0>,
                      ISA_MIPS64R5, ASE_VIRT;
}
def : MipsInstAlias<"dmfc2 $rt, $rd",
                    (DMFC2 GPR64Opnd:$rt, COP2Opnd:$rd, 0), 0>;
def : MipsInstAlias<"dmtc2 $rt, $rd",
                    (DMTC2 COP2Opnd:$rd, GPR64Opnd:$rt, 0), 0>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1083-1086
```tablegen
def : MipsInstAlias<"synciobdma", (SYNC 0x2), 0>, ASE_MIPS64_CNMIPS;
def : MipsInstAlias<"syncs", (SYNC 0x6), 0>, ASE_MIPS64_CNMIPS;
def : MipsInstAlias<"syncw", (SYNC 0x4), 0>, ASE_MIPS64_CNMIPS;
def : MipsInstAlias<"syncws", (SYNC 0x5), 0>, ASE_MIPS64_CNMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1088-1088
```tablegen
// cnMIPS Aliases.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1090-1098
```tablegen
// bbit* with $p 32-63 converted to bbit*32 with $p 0-31
def : MipsInstAlias<"bbit0 $rs, $p, $offset",
                    (BBIT032 GPR64Opnd:$rs, uimm5_plus32_normalize_64:$p,
                             brtarget:$offset), 0>,
      ASE_CNMIPS;
def : MipsInstAlias<"bbit1 $rs, $p, $offset",
                    (BBIT132 GPR64Opnd:$rs, uimm5_plus32_normalize_64:$p,
                             brtarget:$offset), 0>,
      ASE_CNMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1100-1108
```tablegen
// exts with $pos 32-63 in converted to exts32 with $pos 0-31
def : MipsInstAlias<"exts $rt, $rs, $pos, $lenm1",
                    (EXTS32 GPR64Opnd:$rt, GPR64Opnd:$rs,
                            uimm5_plus32_normalize:$pos, uimm5:$lenm1), 0>,
      ASE_MIPS64_CNMIPS;
def : MipsInstAlias<"exts $rt, $pos, $lenm1",
                    (EXTS32 GPR64Opnd:$rt, GPR64Opnd:$rt,
                            uimm5_plus32_normalize:$pos, uimm5:$lenm1), 0>,
      ASE_MIPS64_CNMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1110-1118
```tablegen
// cins with $pos 32-63 in converted to cins32 with $pos 0-31
def : MipsInstAlias<"cins $rt, $rs, $pos, $lenm1",
                    (CINS32 GPR64Opnd:$rt, GPR64Opnd:$rs,
                            uimm5_plus32_normalize:$pos, uimm5:$lenm1), 0>,
      ASE_MIPS64_CNMIPS;
def : MipsInstAlias<"cins $rt, $pos, $lenm1",
                    (CINS32 GPR64Opnd:$rt, GPR64Opnd:$rt,
                            uimm5_plus32_normalize:$pos, uimm5:$lenm1), 0>,
      ASE_MIPS64_CNMIPS;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1120-1122
```tablegen
//===----------------------------------------------------------------------===//
// Assembler Pseudo Instructions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1124-1127
```tablegen
class LoadImmediate64<string instr_asm, Operand Od, RegisterOperand RO> :
  MipsAsmPseudoInst<(outs RO:$rt), (ins Od:$imm64),
                     !strconcat(instr_asm, "\t$rt, $imm64")> ;
def LoadImm64 : LoadImmediate64<"dli", imm64, GPR64Opnd>;
```
- EN: Declares reusable TableGen class `LoadImmediate64` for `Mips64InstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `Mips64InstrInfo` 声明可复用的 TableGen 类 `LoadImmediate64`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1129-1132
```tablegen
def LoadAddrReg64 : MipsAsmPseudoInst<(outs GPR64Opnd:$rt), (ins mem:$addr),
                                       "dla\t$rt, $addr">;
def LoadAddrImm64 : MipsAsmPseudoInst<(outs GPR64Opnd:$rt), (ins imm64:$imm64),
                                       "dla\t$rt, $imm64">;
```
- EN: Defines TableGen record `LoadAddrReg64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LoadAddrReg64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1134-1148
```tablegen
def DMULImmMacro
    : MipsAsmPseudoInst<(outs),
                        (ins GPR64Opnd:$rs, GPR64Opnd:$rt, simm32_relaxed:$imm),
                        "dmul\t$rs, $rt, $imm">,
      ISA_MIPS3_NOT_32R6_64R6_R5900;
def DMULOMacro
    : MipsAsmPseudoInst<(outs),
                        (ins GPR64Opnd:$rs, GPR64Opnd:$rt, GPR64Opnd:$rd),
                        "dmulo\t$rs, $rt, $rd">,
      ISA_MIPS3_NOT_32R6_64R6_R5900;
def DMULOUMacro
    : MipsAsmPseudoInst<(outs),
                        (ins GPR64Opnd:$rs, GPR64Opnd:$rt, GPR64Opnd:$rd),
                        "dmulou\t$rs, $rt, $rd">,
      ISA_MIPS3_NOT_32R6_64R6_R5900;
```
- EN: Defines TableGen record `DMULImmMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DMULImmMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1150-1154
```tablegen
def DMULMacro : MipsAsmPseudoInst<(outs), (ins GPR64Opnd:$rs, GPR64Opnd:$rt,
                                               GPR64Opnd:$rd),
                                  "dmul\t$rs, $rt, $rd"> {
  let InsnPredicates = [HasMips3, NotMips64r6, NotCnMips, NotR5900];
}
```
- EN: Defines TableGen record `DMULMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DMULMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1156-1172
```tablegen
let AdditionalPredicates = [NotInMicroMips] in {
  def DSDivMacro : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                     (ins GPR64Opnd:$rs, GPR64Opnd:$rt),
                                     "ddiv\t$rd, $rs, $rt">,
                   ISA_MIPS3_NOT_32R6_64R6_R5900;
  def DSDivIMacro
      : MipsAsmPseudoInst<(outs GPR64Opnd:$rd), (ins GPR64Opnd:$rs, imm64:$imm),
                          "ddiv\t$rd, $rs, $imm">,
        ISA_MIPS3_NOT_32R6_64R6_R5900;
  def DUDivMacro : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                     (ins GPR64Opnd:$rs, GPR64Opnd:$rt),
                                     "ddivu\t$rd, $rs, $rt">,
                   ISA_MIPS3_NOT_32R6_64R6_R5900;
  def DUDivIMacro
      : MipsAsmPseudoInst<(outs GPR64Opnd:$rd), (ins GPR64Opnd:$rs, imm64:$imm),
                          "ddivu\t$rd, $rs, $imm">,
        ISA_MIPS3_NOT_32R6_64R6_R5900;
```
- EN: Defines TableGen record `DSDivMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSDivMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1174-1176
```tablegen
  // GAS expands 'div' and 'ddiv' differently when the destination
  // register is $zero and the instruction is in the two operand
  // form. 'ddiv' gets expanded, while 'div' is not expanded.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1178-1185
```tablegen
  def : MipsInstAlias<"ddiv $rs, $rt",
                      (DSDivMacro GPR64Opnd:$rs, GPR64Opnd:$rs, GPR64Opnd:$rt),
                      0>,
        ISA_MIPS3_NOT_32R6_64R6_R5900;
  def : MipsInstAlias<"ddiv $rd, $imm",
                      (DSDivIMacro GPR64Opnd:$rd, GPR64Opnd:$rd, imm64:$imm),
                      0>,
        ISA_MIPS3_NOT_32R6_64R6_R5900;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1187-1189
```tablegen
  // GAS expands 'divu' and 'ddivu' differently when the destination
  // register is $zero and the instruction is in the two operand
  // form. 'ddivu' gets expanded, while 'divu' is not expanded.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 1191-1208
```tablegen
  def : MipsInstAlias<"ddivu $rt, $rs",
                      (DUDivMacro GPR64Opnd:$rt, GPR64Opnd:$rt, GPR64Opnd:$rs),
                      0>,
        ISA_MIPS3_NOT_32R6_64R6_R5900;
  def : MipsInstAlias<"ddivu $rd, $imm",
                      (DUDivIMacro GPR64Opnd:$rd, GPR64Opnd:$rd, imm64:$imm),
                      0>,
        ISA_MIPS3_NOT_32R6_64R6_R5900;
  def DSRemMacro : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                     (ins GPR64Opnd:$rs, GPR64Opnd:$rt),
                                     "drem\t$rd, $rs, $rt">,
                   ISA_MIPS3_NOT_32R6_64R6_R5900;
  def DSRemIMacro : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                      (ins GPR64Opnd:$rs, simm32_relaxed:$imm),
                                      "drem\t$rd, $rs, $imm">,
                    ISA_MIPS3_NOT_32R6_64R6_R5900;
  def DURemMacro : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                     (ins GPR64Opnd:$rs, GPR64Opnd:$rt),
```
- EN: Defines TableGen record `DSRemMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSRemMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1209-1226
```tablegen
                                     "dremu\t$rd, $rs, $rt">,
                   ISA_MIPS3_NOT_32R6_64R6_R5900;
  def DURemIMacro : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                      (ins GPR64Opnd:$rs, simm32_relaxed:$imm),
                                      "dremu\t$rd, $rs, $imm">,
                    ISA_MIPS3_NOT_32R6_64R6_R5900;
  def : MipsInstAlias<"drem $rt, $rs",
                      (DSRemMacro GPR64Opnd:$rt, GPR64Opnd:$rt, GPR64Opnd:$rs),
                      0>,
        ISA_MIPS3_NOT_32R6_64R6_R5900;
  def : MipsInstAlias<
            "drem $rd, $imm",
            (DSRemIMacro GPR64Opnd:$rd, GPR64Opnd:$rd, simm32_relaxed:$imm), 0>,
        ISA_MIPS3_NOT_32R6_64R6_R5900;
  def : MipsInstAlias<"dremu $rt, $rs",
                      (DURemMacro GPR64Opnd:$rt, GPR64Opnd:$rt, GPR64Opnd:$rs),
                      0>,
        ISA_MIPS3_NOT_32R6_64R6_R5900;
```
- EN: Defines TableGen record `DURemIMacro` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DURemIMacro`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1227-1231
```tablegen
  def : MipsInstAlias<
            "dremu $rd, $imm",
            (DURemIMacro GPR64Opnd:$rd, GPR64Opnd:$rd, simm32_relaxed:$imm), 0>,
        ISA_MIPS3_NOT_32R6_64R6_R5900;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1233-1245
```tablegen
def NORImm64 : NORIMM_DESC_BASE<GPR64Opnd, imm64>, GPR_64;
def : MipsInstAlias<"nor\t$rs, $imm", (NORImm64 GPR64Opnd:$rs, GPR64Opnd:$rs,
                                                imm64:$imm)>, GPR_64;
def SLTImm64 : MipsAsmPseudoInst<(outs GPR64Opnd:$rs),
                                 (ins GPR64Opnd:$rt, imm64:$imm),
                                 "slt\t$rs, $rt, $imm">, GPR_64;
def : MipsInstAlias<"slt\t$rs, $imm", (SLTImm64 GPR64Opnd:$rs, GPR64Opnd:$rs,
                                                imm64:$imm)>, GPR_64;
def SLTUImm64 : MipsAsmPseudoInst<(outs GPR64Opnd:$rs),
                                  (ins GPR64Opnd:$rt, imm64:$imm),
                                  "sltu\t$rs, $rt, $imm">, GPR_64;
def : MipsInstAlias<"sltu\t$rs, $imm", (SLTUImm64 GPR64Opnd:$rs, GPR64Opnd:$rs,
                                                  imm64:$imm)>, GPR_64;
```
- EN: Defines TableGen record `NORImm64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `NORImm64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1247-1252
```tablegen
def SGEImm64 : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                 (ins GPR64Opnd:$rs, imm64:$imm),
                                 "sge\t$rd, $rs, $imm">, GPR_64;
def : MipsInstAlias<"sge $rs, $imm", (SGEImm64 GPR64Opnd:$rs,
                                               GPR64Opnd:$rs,
                                               imm64:$imm), 0>, GPR_64;
```
- EN: Defines TableGen record `SGEImm64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SGEImm64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1254-1259
```tablegen
def SGEUImm64 : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                  (ins GPR64Opnd:$rs, imm64:$imm),
                                  "sgeu\t$rd, $rs, $imm">, GPR_64;
def : MipsInstAlias<"sgeu $rs, $imm", (SGEUImm64 GPR64Opnd:$rs,
                                                 GPR64Opnd:$rs,
                                                 imm64:$imm), 0>, GPR_64;
```
- EN: Defines TableGen record `SGEUImm64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SGEUImm64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1261-1266
```tablegen
def SGTImm64 : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                 (ins GPR64Opnd:$rs, imm64:$imm),
                                 "sgt\t$rd, $rs, $imm">, GPR_64;
def : MipsInstAlias<"sgt $rs, $imm", (SGTImm64 GPR64Opnd:$rs,
                                               GPR64Opnd:$rs,
                                               imm64:$imm), 0>, GPR_64;
```
- EN: Defines TableGen record `SGTImm64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SGTImm64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1268-1273
```tablegen
def SGTUImm64 : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                  (ins GPR64Opnd:$rs, imm64:$imm),
                                  "sgtu\t$rd, $rs, $imm">, GPR_64;
def : MipsInstAlias<"sgtu $rs, $imm", (SGTUImm64 GPR64Opnd:$rs,
                                                 GPR64Opnd:$rs,
                                                 imm64:$imm), 0>, GPR_64;
```
- EN: Defines TableGen record `SGTUImm64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SGTUImm64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1275-1280
```tablegen
def SLEImm64 : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                 (ins GPR64Opnd:$rs, imm64:$imm),
                                 "sle\t$rd, $rs, $imm">, GPR_64;
def : MipsInstAlias<"sle $rs, $imm", (SLEImm64 GPR64Opnd:$rs,
                                               GPR64Opnd:$rs,
                                               imm64:$imm), 0>, GPR_64;
```
- EN: Defines TableGen record `SLEImm64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SLEImm64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1282-1287
```tablegen
def SLEUImm64 : MipsAsmPseudoInst<(outs GPR64Opnd:$rd),
                                  (ins GPR64Opnd:$rs, imm64:$imm),
                                  "sleu\t$rd, $rs, $imm">, GPR_64;
def : MipsInstAlias<"sleu $rs, $imm", (SLEUImm64 GPR64Opnd:$rs,
                                                 GPR64Opnd:$rs,
                                                 imm64:$imm), 0>, GPR_64;
```
- EN: Defines TableGen record `SLEUImm64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SLEUImm64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1289-1290
```tablegen
def : MipsInstAlias<"rdhwr $rt, $rs",
                    (RDHWR64 GPR64Opnd:$rt, HWRegsOpnd:$rs, 0), 1>, GPR_64;
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
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
