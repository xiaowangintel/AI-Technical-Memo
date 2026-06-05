# RISCVInstrInfoA.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoA.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoA.td - RISC-V 'A' instructions -------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard 'A', Atomic
// Instructions extension.
//
//===----------------------------------------------------------------------===//

//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 15-32: TableGen class templates / TableGen 类 templates
```tablegen
// Operand and SDNode transformation definitions.
//===----------------------------------------------------------------------===//

def ordering : RISCVOp {
  let OperandType = "OPERAND_ATOMIC_ORDERING";
}

//===----------------------------------------------------------------------===//
// Instruction class templates
//===----------------------------------------------------------------------===//

let hasSideEffects = 0, mayLoad = 1, mayStore = 0 in
class LR_r<bit aq, bit rl, bits<3> funct3, string opcodestr>
    : RVInstRAtomic<0b00010, aq, rl, funct3, OPC_AMO,
                    (outs GPR:$rd), (ins GPRMemZeroOffset:$rs1),
                    opcodestr, "$rd, $rs1"> {
  let rs2 = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 33-46: Reusable TableGen multiclass SC_r<bit / 可复用的 TableGen 多类 SC_r<bit
```tablegen

multiclass LR_r_aq_rl<bits<3> funct3, string opcodestr> {
  def ""    : LR_r<0, 0, funct3, opcodestr>;
  def _AQ   : LR_r<1, 0, funct3, opcodestr # ".aq">;
  def _RL   : LR_r<0, 1, funct3, opcodestr # ".rl">;
  def _AQRL : LR_r<1, 1, funct3, opcodestr # ".aqrl">;
}

let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
class SC_r<bit aq, bit rl, bits<3> funct3, string opcodestr>
    : RVInstRAtomic<0b00011, aq, rl, funct3, OPC_AMO,
                    (outs GPR:$rd), (ins GPR:$rs2, GPRMemZeroOffset:$rs1),
                    opcodestr, "$rd, $rs2, $rs1">;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 47-65: Reusable TableGen multiclass AMO_rr<bits<5> / 可复用的 TableGen 多类 AMO_rr<bits<5>
```tablegen
multiclass SC_r_aq_rl<bits<3> funct3, string opcodestr> {
  def ""    : SC_r<0, 0, funct3, opcodestr>;
  def _AQ   : SC_r<1, 0, funct3, opcodestr # ".aq">;
  def _RL   : SC_r<0, 1, funct3, opcodestr # ".rl">;
  def _AQRL : SC_r<1, 1, funct3, opcodestr # ".aqrl">;
}

let hasSideEffects = 0, mayLoad = 1, mayStore = 1 in
class AMO_rr<bits<5> funct5, bit aq, bit rl, bits<3> funct3, string opcodestr>
    : RVInstRAtomic<funct5, aq, rl, funct3, OPC_AMO,
                    (outs GPR:$rd), (ins GPR:$rs2, GPRMemZeroOffset:$rs1),
                    opcodestr, "$rd, $rs2, $rs1">;

multiclass AMO_rr_aq_rl<bits<5> funct5, bits<3> funct3, string opcodestr> {
  def ""    : AMO_rr<funct5, 0, 0, funct3, opcodestr>;
  def _AQ   : AMO_rr<funct5, 1, 0, funct3, opcodestr # ".aq">;
  def _RL   : AMO_rr<funct5, 0, 1, funct3, opcodestr # ".rl">;
  def _AQRL : AMO_rr<funct5, 1, 1, funct3, opcodestr # ".aqrl">;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 66-93: Bulk record instantiation LR_W / 批量记录实例化 LR_W
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZalrsc], IsSignExtendingOpW = 1 in {
defm LR_W       : LR_r_aq_rl<0b010, "lr.w">, Sched<[WriteAtomicLDW, ReadAtomicLDW]>;
defm SC_W       : SC_r_aq_rl<0b010, "sc.w">,
                  Sched<[WriteAtomicSTW, ReadAtomicSTW, ReadAtomicSTW]>;
} // Predicates = [HasStdExtZalrsc], IsSignExtendingOpW = 1

let Predicates = [HasStdExtZaamo], IsSignExtendingOpW = 1 in {
defm AMOSWAP_W  : AMO_rr_aq_rl<0b00001, 0b010, "amoswap.w">,
                  Sched<[WriteAtomicW, ReadAtomicWA, ReadAtomicWD]>;
defm AMOADD_W   : AMO_rr_aq_rl<0b00000, 0b010, "amoadd.w">,
                  Sched<[WriteAtomicW, ReadAtomicWA, ReadAtomicWD]>;
defm AMOXOR_W   : AMO_rr_aq_rl<0b00100, 0b010, "amoxor.w">,
                  Sched<[WriteAtomicW, ReadAtomicWA, ReadAtomicWD]>;
defm AMOAND_W   : AMO_rr_aq_rl<0b01100, 0b010, "amoand.w">,
                  Sched<[WriteAtomicW, ReadAtomicWA, ReadAtomicWD]>;
defm AMOOR_W    : AMO_rr_aq_rl<0b01000, 0b010, "amoor.w">,
                  Sched<[WriteAtomicW, ReadAtomicWA, ReadAtomicWD]>;
defm AMOMIN_W   : AMO_rr_aq_rl<0b10000, 0b010, "amomin.w">,
                  Sched<[WriteAtomicW, ReadAtomicWA, ReadAtomicWD]>;
defm AMOMAX_W   : AMO_rr_aq_rl<0b10100, 0b010, "amomax.w">,
                  Sched<[WriteAtomicW, ReadAtomicWA, ReadAtomicWD]>;
defm AMOMINU_W  : AMO_rr_aq_rl<0b11000, 0b010, "amominu.w">,
                  Sched<[WriteAtomicW, ReadAtomicWA, ReadAtomicWD]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 94-121: Bulk record instantiation AMOMAXU_W / 批量记录实例化 AMOMAXU_W
```tablegen
defm AMOMAXU_W  : AMO_rr_aq_rl<0b11100, 0b010, "amomaxu.w">,
                  Sched<[WriteAtomicW, ReadAtomicWA, ReadAtomicWD]>;
} // Predicates = [HasStdExtZaamo], IsSignExtendingOpW = 1

let Predicates = [HasStdExtZalrsc, IsRV64] in {
defm LR_D       : LR_r_aq_rl<0b011, "lr.d">, Sched<[WriteAtomicLDD, ReadAtomicLDD]>;
defm SC_D       : SC_r_aq_rl<0b011, "sc.d">,
                  Sched<[WriteAtomicSTD, ReadAtomicSTD, ReadAtomicSTD]>;
} // Predicates = [HasStdExtZalrsc, IsRV64]

let Predicates = [HasStdExtZaamo, IsRV64] in {
defm AMOSWAP_D  : AMO_rr_aq_rl<0b00001, 0b011, "amoswap.d">,
                  Sched<[WriteAtomicD, ReadAtomicDA, ReadAtomicDD]>;
defm AMOADD_D   : AMO_rr_aq_rl<0b00000, 0b011, "amoadd.d">,
                  Sched<[WriteAtomicD, ReadAtomicDA, ReadAtomicDD]>;
defm AMOXOR_D   : AMO_rr_aq_rl<0b00100, 0b011, "amoxor.d">,
                  Sched<[WriteAtomicD, ReadAtomicDA, ReadAtomicDD]>;
defm AMOAND_D   : AMO_rr_aq_rl<0b01100, 0b011, "amoand.d">,
                  Sched<[WriteAtomicD, ReadAtomicDA, ReadAtomicDD]>;
defm AMOOR_D    : AMO_rr_aq_rl<0b01000, 0b011, "amoor.d">,
                  Sched<[WriteAtomicD, ReadAtomicDA, ReadAtomicDD]>;
defm AMOMIN_D   : AMO_rr_aq_rl<0b10000, 0b011, "amomin.d">,
                  Sched<[WriteAtomicD, ReadAtomicDA, ReadAtomicDD]>;
defm AMOMAX_D   : AMO_rr_aq_rl<0b10100, 0b011, "amomax.d">,
                  Sched<[WriteAtomicD, ReadAtomicDA, ReadAtomicDD]>;
defm AMOMINU_D  : AMO_rr_aq_rl<0b11000, 0b011, "amominu.d">,
                  Sched<[WriteAtomicD, ReadAtomicDA, ReadAtomicDD]>;
defm AMOMAXU_D  : AMO_rr_aq_rl<0b11100, 0b011, "amomaxu.d">,
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 122-135: TableGen class relaxed_load<PatFrags / TableGen 类 relaxed_load<PatFrags
```tablegen
                  Sched<[WriteAtomicD, ReadAtomicDA, ReadAtomicDD]>;
} // Predicates = [HasStdExtZaamo, IsRV64]

//===----------------------------------------------------------------------===//
// Pseudo-instructions and codegen patterns
//===----------------------------------------------------------------------===//

let IsAtomic = 1 in {
// An atomic load operation that does not need either acquire or release
// semantics.
class relaxed_load<PatFrags base>
    : PatFrag<(ops node:$ptr), (base node:$ptr)> {
  let IsAtomicOrderingAcquireOrStronger = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 136-154: TableGen class acquiring_load<PatFrags / TableGen 类 acquiring_load<PatFrags
```tablegen

// A atomic load operation that actually needs acquire semantics.
class acquiring_load<PatFrags base>
    : PatFrag<(ops node:$ptr), (base node:$ptr)> {
  let IsAtomicOrderingAcquire = 1;
}

// An atomic load operation that needs sequential consistency.
class seq_cst_load<PatFrags base>
    : PatFrag<(ops node:$ptr), (base node:$ptr)> {
  let IsAtomicOrderingSequentiallyConsistent = 1;
}

// An atomic store operation that does not need either acquire or release
// semantics.
class relaxed_store<PatFrag base>
    : PatFrag<(ops node:$val, node:$ptr), (base node:$val, node:$ptr)> {
  let IsAtomicOrderingReleaseOrStronger = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 155-168: TableGen class releasing_store<PatFrag / TableGen 类 releasing_store<PatFrag
```tablegen

// A store operation that actually needs release semantics.
class releasing_store<PatFrag base>
    : PatFrag<(ops node:$val, node:$ptr), (base node:$val, node:$ptr)> {
  let IsAtomicOrderingRelease = 1;
}

// A store operation that actually needs sequential consistency.
class seq_cst_store<PatFrag base>
    : PatFrag<(ops node:$val, node:$ptr), (base node:$val, node:$ptr)> {
  let IsAtomicOrderingSequentiallyConsistent = 1;
}
} // IsAtomic = 1
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 169-185: Commentary and design intent / 注释与设计意图
```tablegen
// Atomic load/store are available under +zalrsc (thus also +a) and
// +force-atomics.  Fences will be inserted for atomic load/stores according to
// the logic in RISCVTargetLowering::{emitLeadingFence,emitTrailingFence}.
// The normal loads/stores are relaxed (unordered) loads/stores that don't have
// any ordering. This is necessary because AtomicExpandPass has added fences to
// atomic load/stores and changed them to unordered ones.
let Predicates = [HasAtomicLdSt] in {
  // Use unsigned for aext due to no c.lb in Zcb.
  def : LdPat<relaxed_load<atomic_load_sext_8>,   LB>;
  def : LdPat<relaxed_load<atomic_load_azext_8>,  LBU>;
  def : LdPat<relaxed_load<atomic_load_asext_16>, LH>;
  def : LdPat<relaxed_load<atomic_load_zext_16>,  LHU>;

  def : StPat<relaxed_store<atomic_store_8>,  SB, GPR, XLenVT>;
  def : StPat<relaxed_store<atomic_store_16>, SH, GPR, XLenVT>;
  def : StPat<relaxed_store<atomic_store_32>, SW, GPR, XLenVT>;
}
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 186-199: TableGen record definition / TableGen 记录定义
```tablegen

let Predicates = [HasAtomicLdSt, IsRV32] in {
  def : LdPat<relaxed_load<atomic_load_nonext_32>, LW, i32>;
}

let Predicates = [HasAtomicLdSt, IsRV64] in {
  def : LdPat<relaxed_load<atomic_load_asext_32>, LW, i64>;
  def : LdPat<relaxed_load<atomic_load_zext_32>, LWU, i64>;
  def : LdPat<relaxed_load<atomic_load_nonext_64>, LD, i64>;
  def : StPat<relaxed_store<atomic_store_64>, SD, GPR, i64>;
}

/// AMOs
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 200-216: Reusable TableGen multiclass PatAMO<SDPatternOperator / 可复用的 TableGen 多类 PatAMO<SDPatternOperator
```tablegen
class PatAMO<SDPatternOperator OpNode, RVInst Inst, ValueType vt = XLenVT>
    : Pat<(vt (OpNode (XLenVT GPR:$rs1), (vt GPR:$rs2))), (Inst GPR:$rs2, GPR:$rs1)>;

multiclass AMOPat<string AtomicOp, string BaseInst, ValueType vt = XLenVT,
                  list<Predicate> ExtraPreds = []> {
let Predicates = !listconcat([HasStdExtA, NoStdExtZtso], ExtraPreds) in {
  def : PatAMO<!cast<PatFrag>(AtomicOp#"_monotonic"),
               !cast<RVInst>(BaseInst), vt>;
  def : PatAMO<!cast<PatFrag>(AtomicOp#"_acquire"),
               !cast<RVInst>(BaseInst#"_AQ"), vt>;
  def : PatAMO<!cast<PatFrag>(AtomicOp#"_release"),
               !cast<RVInst>(BaseInst#"_RL"), vt>;
  def : PatAMO<!cast<PatFrag>(AtomicOp#"_acq_rel"),
               !cast<RVInst>(BaseInst#"_AQRL"), vt>;
  def : PatAMO<!cast<PatFrag>(AtomicOp#"_seq_cst"),
               !cast<RVInst>(BaseInst#"_AQRL"), vt>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 217-230: TableGen record definition / TableGen 记录定义
```tablegen
let Predicates = !listconcat([HasStdExtA, HasStdExtZtso], ExtraPreds) in {
  def : PatAMO<!cast<PatFrag>(AtomicOp#"_monotonic"),
               !cast<RVInst>(BaseInst), vt>;
  def : PatAMO<!cast<PatFrag>(AtomicOp#"_acquire"),
               !cast<RVInst>(BaseInst), vt>;
  def : PatAMO<!cast<PatFrag>(AtomicOp#"_release"),
               !cast<RVInst>(BaseInst), vt>;
  def : PatAMO<!cast<PatFrag>(AtomicOp#"_acq_rel"),
               !cast<RVInst>(BaseInst), vt>;
  def : PatAMO<!cast<PatFrag>(AtomicOp#"_seq_cst"),
               !cast<RVInst>(BaseInst), vt>;
}
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 231-250: Bulk record instantiation / 批量记录实例化
```tablegen
defm : AMOPat<"atomic_swap_i32", "AMOSWAP_W">;
defm : AMOPat<"atomic_load_add_i32", "AMOADD_W">;
defm : AMOPat<"atomic_load_and_i32", "AMOAND_W">;
defm : AMOPat<"atomic_load_or_i32", "AMOOR_W">;
defm : AMOPat<"atomic_load_xor_i32", "AMOXOR_W">;
defm : AMOPat<"atomic_load_max_i32", "AMOMAX_W">;
defm : AMOPat<"atomic_load_min_i32", "AMOMIN_W">;
defm : AMOPat<"atomic_load_umax_i32", "AMOMAXU_W">;
defm : AMOPat<"atomic_load_umin_i32", "AMOMINU_W">;

defm : AMOPat<"atomic_swap_i64", "AMOSWAP_D", i64, [IsRV64]>;
defm : AMOPat<"atomic_load_add_i64", "AMOADD_D", i64, [IsRV64]>;
defm : AMOPat<"atomic_load_and_i64", "AMOAND_D", i64, [IsRV64]>;
defm : AMOPat<"atomic_load_or_i64", "AMOOR_D", i64, [IsRV64]>;
defm : AMOPat<"atomic_load_xor_i64", "AMOXOR_D", i64, [IsRV64]>;
defm : AMOPat<"atomic_load_max_i64", "AMOMAX_D", i64, [IsRV64]>;
defm : AMOPat<"atomic_load_min_i64", "AMOMIN_D", i64, [IsRV64]>;
defm : AMOPat<"atomic_load_umax_i64", "AMOMAXU_D", i64, [IsRV64]>;
defm : AMOPat<"atomic_load_umin_i64", "AMOMINU_D", i64, [IsRV64]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 251-269: TableGen class PseudoAMO / TableGen 类 PseudoAMO
```tablegen

/// Pseudo AMOs

class PseudoAMO : Pseudo<(outs GPR:$res, GPR:$scratch),
                         (ins GPR:$addr, GPR:$incr, ordering:$ordering), []> {
  let Constraints = "@earlyclobber $res,@earlyclobber $scratch";
  let mayLoad = 1;
  let mayStore = 1;
  let hasSideEffects = 0;
}

class PseudoMaskedAMO
    : Pseudo<(outs GPR:$res, GPR:$scratch),
             (ins GPR:$addr, GPR:$incr, GPR:$mask, ordering:$ordering), []> {
  let Constraints = "@earlyclobber $res,@earlyclobber $scratch";
  let mayLoad = 1;
  let mayStore = 1;
  let hasSideEffects = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 270-290: TableGen class PseudoMaskedAMOMinMax / TableGen 类 PseudoMaskedAMOMinMax
```tablegen

class PseudoMaskedAMOMinMax
    : Pseudo<(outs GPR:$res, GPR:$scratch1, GPR:$scratch2),
             (ins GPR:$addr, GPR:$incr, GPR:$mask, GPR:$sextshamt,
                  ordering:$ordering), []> {
  let Constraints = "@earlyclobber $res,@earlyclobber $scratch1,"
                    "@earlyclobber $scratch2";
  let mayLoad = 1;
  let mayStore = 1;
  let hasSideEffects = 0;
}

class PseudoMaskedAMOUMinUMax
    : Pseudo<(outs GPR:$res, GPR:$scratch1, GPR:$scratch2),
             (ins GPR:$addr, GPR:$incr, GPR:$mask, ordering:$ordering), []> {
  let Constraints = "@earlyclobber $res,@earlyclobber $scratch1,"
                    "@earlyclobber $scratch2";
  let mayLoad = 1;
  let mayStore = 1;
  let hasSideEffects = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 291-305: Reusable TableGen multiclass in / 可复用的 TableGen 多类 in
```tablegen

// Ordering constants must be kept in sync with the AtomicOrdering enum in
// AtomicOrdering.h.
multiclass PseudoAMOPat<string AtomicOp, Pseudo AMOInst, ValueType vt = XLenVT> {
  def : Pat<(vt (!cast<PatFrag>(AtomicOp#"_monotonic") GPR:$addr, GPR:$incr)),
            (AMOInst GPR:$addr, GPR:$incr, 2)>;
  def : Pat<(vt (!cast<PatFrag>(AtomicOp#"_acquire") GPR:$addr, GPR:$incr)),
            (AMOInst GPR:$addr, GPR:$incr, 4)>;
  def : Pat<(vt (!cast<PatFrag>(AtomicOp#"_release") GPR:$addr, GPR:$incr)),
            (AMOInst GPR:$addr, GPR:$incr, 5)>;
  def : Pat<(vt (!cast<PatFrag>(AtomicOp#"_acq_rel") GPR:$addr, GPR:$incr)),
            (AMOInst GPR:$addr, GPR:$incr, 6)>;
  def : Pat<(vt (!cast<PatFrag>(AtomicOp#"_seq_cst") GPR:$addr, GPR:$incr)),
            (AMOInst GPR:$addr, GPR:$incr, 7)>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 306-320: TableGen class PseudoMaskedAMOPat<Intrinsic / TableGen 类 PseudoMaskedAMOPat<Intrinsic
```tablegen

class PseudoMaskedAMOPat<Intrinsic intrin, Pseudo AMOInst>
    : Pat<(XLenVT (intrin (XLenVT GPR:$addr), (XLenVT GPR:$incr),
                          (XLenVT GPR:$mask), (XLenVT timm:$ordering))),
          (AMOInst GPR:$addr, GPR:$incr, GPR:$mask, timm:$ordering)>;

class PseudoMaskedAMOMinMaxPat<Intrinsic intrin, Pseudo AMOInst>
    : Pat<(XLenVT (intrin (XLenVT GPR:$addr), (XLenVT GPR:$incr),
                          (XLenVT GPR:$mask), (XLenVT GPR:$shiftamt),
           (XLenVT timm:$ordering))),
          (AMOInst GPR:$addr, GPR:$incr, GPR:$mask, GPR:$shiftamt,
           timm:$ordering)>;

let Predicates = [HasStdExtZalrsc, NoStdExtZaamo] in {
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 321-335: TableGen record PseudoAtomicSwap32 / TableGen 记录 PseudoAtomicSwap32
```tablegen
let Size = 16 in {
def PseudoAtomicSwap32 : PseudoAMO;
def PseudoAtomicLoadAdd32 : PseudoAMO;
def PseudoAtomicLoadSub32 : PseudoAMO;
def PseudoAtomicLoadAnd32 : PseudoAMO;
def PseudoAtomicLoadOr32 : PseudoAMO;
def PseudoAtomicLoadXor32 : PseudoAMO;
} // Size = 16
let Size = 24 in {
def PseudoAtomicLoadMax32 : PseudoAMO;
def PseudoAtomicLoadMin32 : PseudoAMO;
def PseudoAtomicLoadUMax32 : PseudoAMO;
def PseudoAtomicLoadUMin32 : PseudoAMO;
} // Size = 24
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 336-349: Bulk record instantiation / 批量记录实例化
```tablegen
defm : PseudoAMOPat<"atomic_swap_i32", PseudoAtomicSwap32>;
defm : PseudoAMOPat<"atomic_load_add_i32", PseudoAtomicLoadAdd32>;
defm : PseudoAMOPat<"atomic_load_sub_i32", PseudoAtomicLoadSub32>;
defm : PseudoAMOPat<"atomic_load_and_i32", PseudoAtomicLoadAnd32>;
defm : PseudoAMOPat<"atomic_load_or_i32", PseudoAtomicLoadOr32>;
defm : PseudoAMOPat<"atomic_load_xor_i32", PseudoAtomicLoadXor32>;
defm : PseudoAMOPat<"atomic_load_max_i32", PseudoAtomicLoadMax32>;
defm : PseudoAMOPat<"atomic_load_min_i32", PseudoAtomicLoadMin32>;
defm : PseudoAMOPat<"atomic_load_umax_i32", PseudoAtomicLoadUMax32>;
defm : PseudoAMOPat<"atomic_load_umin_i32", PseudoAtomicLoadUMin32>;
} // Predicates = [HasStdExtZalrsc, NoStdExtZaamo]

let Predicates = [HasStdExtZalrsc, NoStdExtZaamo, IsRV64] in {
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 350-364: TableGen record PseudoAtomicSwap64 / TableGen 记录 PseudoAtomicSwap64
```tablegen
let Size = 16 in {
def PseudoAtomicSwap64 : PseudoAMO;
def PseudoAtomicLoadAdd64 : PseudoAMO;
def PseudoAtomicLoadSub64 : PseudoAMO;
def PseudoAtomicLoadAnd64 : PseudoAMO;
def PseudoAtomicLoadOr64 : PseudoAMO;
def PseudoAtomicLoadXor64 : PseudoAMO;
} // Size = 16
let Size = 24 in {
def PseudoAtomicLoadMax64 : PseudoAMO;
def PseudoAtomicLoadMin64 : PseudoAMO;
def PseudoAtomicLoadUMax64 : PseudoAMO;
def PseudoAtomicLoadUMin64 : PseudoAMO;
} // Size = 24
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 365-378: Bulk record instantiation / 批量记录实例化
```tablegen
defm : PseudoAMOPat<"atomic_swap_i64", PseudoAtomicSwap64, i64>;
defm : PseudoAMOPat<"atomic_load_add_i64", PseudoAtomicLoadAdd64, i64>;
defm : PseudoAMOPat<"atomic_load_sub_i64", PseudoAtomicLoadSub64, i64>;
defm : PseudoAMOPat<"atomic_load_and_i64", PseudoAtomicLoadAnd64, i64>;
defm : PseudoAMOPat<"atomic_load_or_i64", PseudoAtomicLoadOr64, i64>;
defm : PseudoAMOPat<"atomic_load_xor_i64", PseudoAtomicLoadXor64, i64>;
defm : PseudoAMOPat<"atomic_load_max_i64", PseudoAtomicLoadMax64, i64>;
defm : PseudoAMOPat<"atomic_load_min_i64", PseudoAtomicLoadMin64, i64>;
defm : PseudoAMOPat<"atomic_load_umax_i64", PseudoAtomicLoadUMax64, i64>;
defm : PseudoAMOPat<"atomic_load_umin_i64", PseudoAtomicLoadUMin64, i64>;
} // Predicates = [HasStdExtZalrsc, NoStdExtZaamo, IsRV64]

let Predicates = [HasStdExtZalrsc] in {
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 379-394: Bulk record instantiation PseudoAtomicLoadNand32 / 批量记录实例化 PseudoAtomicLoadNand32
```tablegen
let Size = 20 in
def PseudoAtomicLoadNand32 : PseudoAMO;
defm : PseudoAMOPat<"atomic_load_nand_i32", PseudoAtomicLoadNand32>;

let Size = 28 in {
  def PseudoMaskedAtomicSwap32 : PseudoMaskedAMO;
  def PseudoMaskedAtomicLoadAdd32 : PseudoMaskedAMO;
  def PseudoMaskedAtomicLoadSub32 : PseudoMaskedAMO;
}
let Size = 32 in {
  def PseudoMaskedAtomicLoadNand32 : PseudoMaskedAMO;
}
let Size = 44 in {
  def PseudoMaskedAtomicLoadMax32 : PseudoMaskedAMOMinMax;
  def PseudoMaskedAtomicLoadMin32 : PseudoMaskedAMOMinMax;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 395-417: TableGen record PseudoMaskedAtomicLoadUMax32 / TableGen 记录 PseudoMaskedAtomicLoadUMax32
```tablegen
let Size = 36 in {
 def PseudoMaskedAtomicLoadUMax32 : PseudoMaskedAMOUMinUMax;
 def PseudoMaskedAtomicLoadUMin32 : PseudoMaskedAMOUMinUMax;
}

def : PseudoMaskedAMOPat<int_riscv_masked_atomicrmw_xchg,
                         PseudoMaskedAtomicSwap32>;
def : PseudoMaskedAMOPat<int_riscv_masked_atomicrmw_add,
                         PseudoMaskedAtomicLoadAdd32>;
def : PseudoMaskedAMOPat<int_riscv_masked_atomicrmw_sub,
                         PseudoMaskedAtomicLoadSub32>;
def : PseudoMaskedAMOPat<int_riscv_masked_atomicrmw_nand,
                         PseudoMaskedAtomicLoadNand32>;
def : PseudoMaskedAMOMinMaxPat<int_riscv_masked_atomicrmw_max,
                               PseudoMaskedAtomicLoadMax32>;
def : PseudoMaskedAMOMinMaxPat<int_riscv_masked_atomicrmw_min,
                               PseudoMaskedAtomicLoadMin32>;
def : PseudoMaskedAMOPat<int_riscv_masked_atomicrmw_umax,
                         PseudoMaskedAtomicLoadUMax32>;
def : PseudoMaskedAMOPat<int_riscv_masked_atomicrmw_umin,
                         PseudoMaskedAtomicLoadUMin32>;
} // Predicates = [HasStdExtZalrsc]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 418-436: TableGen class PseudoCmpXchg / TableGen 类 PseudoCmpXchg
```tablegen
let Predicates = [HasStdExtZalrsc, IsRV64] in {

let Size = 20 in
def PseudoAtomicLoadNand64 : PseudoAMO;
defm : PseudoAMOPat<"atomic_load_nand_i64", PseudoAtomicLoadNand64, i64>;
} // Predicates = [HasStdExtZalrsc, IsRV64]


/// Compare and exchange

class PseudoCmpXchg
    : Pseudo<(outs GPR:$res, GPR:$scratch),
             (ins GPR:$addr, GPR:$cmpval, GPR:$newval, ordering:$ordering), []> {
  let Constraints = "@earlyclobber $res,@earlyclobber $scratch";
  let mayLoad = 1;
  let mayStore = 1;
  let hasSideEffects = 0;
  let Size = 16;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 437-452: Reusable TableGen multiclass in / 可复用的 TableGen 多类 in
```tablegen

// Ordering constants must be kept in sync with the AtomicOrdering enum in
// AtomicOrdering.h.
multiclass PseudoCmpXchgPat<string Op, Pseudo CmpXchgInst,
                            ValueType vt = XLenVT> {
  def : Pat<(vt (!cast<PatFrag>(Op#"_monotonic") GPR:$addr, GPR:$cmp, GPR:$new)),
            (CmpXchgInst GPR:$addr, GPR:$cmp, GPR:$new, 2)>;
  def : Pat<(vt (!cast<PatFrag>(Op#"_acquire") GPR:$addr, GPR:$cmp, GPR:$new)),
            (CmpXchgInst GPR:$addr, GPR:$cmp, GPR:$new, 4)>;
  def : Pat<(vt (!cast<PatFrag>(Op#"_release") GPR:$addr, GPR:$cmp, GPR:$new)),
            (CmpXchgInst GPR:$addr, GPR:$cmp, GPR:$new, 5)>;
  def : Pat<(vt (!cast<PatFrag>(Op#"_acq_rel") GPR:$addr, GPR:$cmp, GPR:$new)),
            (CmpXchgInst GPR:$addr, GPR:$cmp, GPR:$new, 6)>;
  def : Pat<(vt (!cast<PatFrag>(Op#"_seq_cst") GPR:$addr, GPR:$cmp, GPR:$new)),
            (CmpXchgInst GPR:$addr, GPR:$cmp, GPR:$new, 7)>;
}
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 453-474: Bulk record instantiation PseudoCmpXchg32 / 批量记录实例化 PseudoCmpXchg32
```tablegen

let Predicates = [HasStdExtZalrsc, NoStdExtZacas] in {
def PseudoCmpXchg32 : PseudoCmpXchg;
defm : PseudoCmpXchgPat<"atomic_cmp_swap_i32", PseudoCmpXchg32>;
}

let Predicates = [HasStdExtZalrsc, NoStdExtZacas, IsRV64] in {
def PseudoCmpXchg64 : PseudoCmpXchg;
defm : PseudoCmpXchgPat<"atomic_cmp_swap_i64", PseudoCmpXchg64, i64>;
}

let Predicates = [HasStdExtZalrsc] in {
def PseudoMaskedCmpXchg32
    : Pseudo<(outs GPR:$res, GPR:$scratch),
             (ins GPR:$addr, GPR:$cmpval, GPR:$newval, GPR:$mask,
              ordering:$ordering), []> {
  let Constraints = "@earlyclobber $res,@earlyclobber $scratch";
  let mayLoad = 1;
  let mayStore = 1;
  let hasSideEffects = 0;
  let Size = 32;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 475-481: TableGen record definition / TableGen 记录定义
```tablegen

def : Pat<(XLenVT (int_riscv_masked_cmpxchg
            (XLenVT GPR:$addr), (XLenVT GPR:$cmpval), (XLenVT GPR:$newval),
            (XLenVT GPR:$mask), (XLenVT timm:$ordering))),
          (PseudoMaskedCmpXchg32
            GPR:$addr, GPR:$cmpval, GPR:$newval, GPR:$mask, timm:$ordering)>;
} // Predicates = [HasStdExtZalrsc]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
