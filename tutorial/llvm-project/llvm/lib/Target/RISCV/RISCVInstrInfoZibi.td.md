# RISCVInstrInfoZibi.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZibi.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZibi.td - 'Zibi' instructions --------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
///
/// This file describes the RISC-V instructions for 'Zibi' (branch with imm).
///
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-27: TableGen record imm5_zibi / TableGen 记录 imm5_zibi
```tablegen

// A 5-bit unsigned immediate representing 1-31 and -1. 00000 represents -1.
def imm5_zibi : RISCVOp<XLenVT>, ImmLeaf<XLenVT, [{
    return (Imm != 0 && isUInt<5>(Imm)) || Imm == -1;
}]> {
  let ParserMatchClass = ImmAsmOperand<"", 5, "Zibi">;
  let EncoderMethod = "getImmOpValueZibi";
  let DecoderMethod = "decodeImmZibiOperand";
  let MCOperandPredicate = [{
    int64_t Imm;
    if (!MCOp.evaluateAsConstantImm(Imm))
      return false;
    return (Imm >= 1 && Imm <= 31) || Imm == -1;
  }];
  let OperandType = "OPERAND_IMM5_ZIBI";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 28-39: TableGen class Branch_imm<bits<3> / TableGen 类 Branch_imm<bits<3>
```tablegen

class Branch_imm<bits<3> funct3, string opcodestr>
    : RVInstBIMM<funct3, OPC_BRANCH, (outs),
                 (ins GPR:$rs1, imm5_zibi:$cimm, bare_simm13_lsb0:$imm12),
                 opcodestr, "$rs1, $cimm, $imm12">,
      Sched<[WriteJmp, ReadJmp]> {
  let isBranch = 1;
  let isTerminator = 1;
  let hasSideEffects = 0;
  let mayLoad = 0;
  let mayStore = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 40-50: TableGen record BEQI / TableGen 记录 BEQI
```tablegen

let Predicates = [HasStdExtZibi] in {
  def BEQI : Branch_imm<0b010, "beqi">;
  def BNEI : Branch_imm<0b011, "bnei">;

  // For long branch relaxation.
  // This will be expanded into BEQI/BNEI(4 bytes) + JAL(4 bytes).
  def PseudoLongBEQI : LongBcciPseudo<imm5_zibi, 8>;
  def PseudoLongBNEI : LongBcciPseudo<imm5_zibi, 8>;
} // Predicates = [HasStdExtZibi]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 51-63: Reusable TableGen multiclass SelectZibi<CondCode / 可复用的 TableGen 多类 SelectZibi<CondCode
```tablegen
multiclass BccImmPat<CondCode Cond, Branch_imm Inst> {
  def : Pat<(riscv_brcc (XLenVT GPR:$rs1), imm5_zibi:$cimm, Cond, bb:$imm12),
            (Inst GPR:$rs1, imm5_zibi:$cimm, bare_simm13_lsb0_bb:$imm12)>;
}

defm CC_Imm5_Zibi : SelectCC_GPR_riirr<GPR, imm5_zibi>;

class SelectZibi<CondCode Cond>
    : Pat<(riscv_selectcc (XLenVT GPR:$lhs), imm5_zibi:$cimm, Cond:$cc,
                          (XLenVT GPR:$truev), GPR:$falsev),
          (Select_GPR_Using_CC_Imm5_Zibi GPR:$lhs, imm5_zibi:$cimm,
              (CCtoRISCVCC $cc), GPR:$truev, GPR:$falsev)>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 64-70: Bulk record instantiation / 批量记录实例化
```tablegen
let Predicates = [HasStdExtZibi] in {
  def : SelectZibi<SETEQ>;
  def : SelectZibi<SETNE>;

  defm : BccImmPat<SETEQ, BEQI>;
  defm : BccImmPat<SETNE, BNEI>;
} // Predicates = [HasStdExtZibi]
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
