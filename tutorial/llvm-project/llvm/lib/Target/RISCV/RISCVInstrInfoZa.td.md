# RISCVInstrInfoZa.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZa.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZa.td - RISC-V Atomic instructions ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard atomic 'Za*'
// extensions:
//   - Zawrs (v1.0) : Wait-on-Reservation-Set.
//   - Zacas (v1.0) : Atomic Compare-and-Swap.
//   - Zabha (v1.0) : Byte and Halfword Atomic Memory Operations.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 16-26: TableGen record GPRPairRV32Operand / TableGen 记录 GPRPairRV32Operand
```tablegen

//===----------------------------------------------------------------------===//
// Zacas (Atomic Compare-and-Swap)
//===----------------------------------------------------------------------===//

def GPRPairRV32Operand : AsmOperandClass {
  let Name = "GPRPairRV32";
  let ParserMethod = "parseGPRPair<false>";
  let PredicateMethod = "isGPRPair";
  let RenderMethod = "addRegOperands";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 27-37: TableGen record GPRPairRV64Operand / TableGen 记录 GPRPairRV64Operand
```tablegen

def GPRPairRV64Operand : AsmOperandClass {
  let Name = "GPRPairRV64";
  let ParserMethod = "parseGPRPair<true>";
  let PredicateMethod = "isGPRPair";
  let RenderMethod = "addRegOperands";
}

def GPRPairRV32 : RegisterOperand<GPRPair> {
  let ParserMatchClass = GPRPairRV32Operand;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 38-49: TableGen class AMO_cas<bits<5> / TableGen 类 AMO_cas<bits<5>
```tablegen

def GPRPairRV64 : RegisterOperand<GPRPair> {
  let ParserMatchClass = GPRPairRV64Operand;
}

let hasSideEffects = 0, mayLoad = 1, mayStore = 1, Constraints = "$rd = $rd_wb" in
class AMO_cas<bits<5> funct5, bit aq, bit rl, bits<3> funct3, string opcodestr,
              DAGOperand RC>
    : RVInstRAtomic<funct5, aq, rl, funct3, OPC_AMO,
                    (outs RC:$rd_wb), (ins RC:$rd, RC:$rs2, GPRMemZeroOffset:$rs1),
                    opcodestr, "$rd, $rs2, $rs1">;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 50-61: Reusable TableGen multiclass AMO_cas_aq_rl / 可复用的 TableGen 多类 AMO_cas_aq_rl
```tablegen
multiclass AMO_cas_aq_rl<bits<5> funct5, bits<3> funct3, string opcodestr,
                         DAGOperand RC> {
  def ""    : AMO_cas<funct5, 0, 0, funct3, opcodestr, RC>;
  def _AQ   : AMO_cas<funct5, 1, 0, funct3, opcodestr # ".aq", RC>;
  def _RL   : AMO_cas<funct5, 0, 1, funct3, opcodestr # ".rl", RC>;
  def _AQRL : AMO_cas<funct5, 1, 1, funct3, opcodestr # ".aqrl", RC>;
}

let Predicates = [HasStdExtZacas], IsSignExtendingOpW = 1 in {
defm AMOCAS_W : AMO_cas_aq_rl<0b00101, 0b010, "amocas.w", GPR>;
} // Predicates = [HasStdExtZacas]
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 62-81: Reusable TableGen multiclass AMOCAS_D_RV32 / 可复用的 TableGen 多类 AMOCAS_D_RV32
```tablegen
let Predicates = [HasStdExtZacas, IsRV32], DecoderNamespace = "RV32Only"  in {
defm AMOCAS_D_RV32 : AMO_cas_aq_rl<0b00101, 0b011, "amocas.d", GPRPairRV32>;
} // Predicates = [HasStdExtZacas, IsRV32]

let Predicates = [HasStdExtZacas, IsRV64] in {
defm AMOCAS_D_RV64 : AMO_cas_aq_rl<0b00101, 0b011, "amocas.d", GPR>;
defm AMOCAS_Q : AMO_cas_aq_rl<0b00101, 0b100, "amocas.q", GPRPairRV64>;
} // Predicates = [HasStdExtZacas, IsRV64]

multiclass AMOCASPat<string AtomicOp, string BaseInst, ValueType vt = XLenVT,
                     list<Predicate> ExtraPreds = []> {
  let Predicates = !listconcat([HasStdExtZacas, NoStdExtZtso], ExtraPreds) in {
    def : Pat<(!cast<PatFrag>(AtomicOp#"_monotonic") (XLenVT GPR:$addr),
                                                     (vt GPR:$cmp),
                                                     (vt GPR:$new)),
              (!cast<RVInst>(BaseInst) GPR:$cmp, GPR:$new, GPR:$addr)>;
    def : Pat<(!cast<PatFrag>(AtomicOp#"_acquire") (XLenVT GPR:$addr),
                                                   (vt GPR:$cmp),
                                                   (vt GPR:$new)),
              (!cast<RVInst>(BaseInst#"_AQ") GPR:$cmp, GPR:$new, GPR:$addr)>;
```
**EN:** This block defines a multiclass template that emits families of related records, reducing duplication in declarative backend descriptions.

**CN:** 该区段定义一个 multiclass 模板，用于批量生成相关记录，减少声明式后端描述中的重复。

### Lines 82-101: TableGen record definition / TableGen 记录定义
```tablegen
    def : Pat<(!cast<PatFrag>(AtomicOp#"_release") (XLenVT GPR:$addr),
                                                   (vt GPR:$cmp),
                                                   (vt GPR:$new)),
              (!cast<RVInst>(BaseInst#"_RL") GPR:$cmp, GPR:$new, GPR:$addr)>;
    def : Pat<(!cast<PatFrag>(AtomicOp#"_acq_rel") (XLenVT GPR:$addr),
                                                   (vt GPR:$cmp),
                                                   (vt GPR:$new)),
              (!cast<RVInst>(BaseInst#"_AQRL") GPR:$cmp, GPR:$new, GPR:$addr)>;
    def : Pat<(!cast<PatFrag>(AtomicOp#"_seq_cst") (XLenVT GPR:$addr),
                                                   (vt GPR:$cmp),
                                                   (vt GPR:$new)),
              (!cast<RVInst>(BaseInst#"_AQRL") GPR:$cmp, GPR:$new, GPR:$addr)>;
  } // Predicates = !listconcat([HasStdExtZacas, NoStdExtZtso], ExtraPreds)
  let Predicates = !listconcat([HasStdExtZacas, HasStdExtZtso], ExtraPreds) in {
    def : Pat<(!cast<PatFrag>(AtomicOp#"_monotonic") (XLenVT GPR:$addr),
                                                     (vt GPR:$cmp),
                                                     (vt GPR:$new)),
              (!cast<RVInst>(BaseInst) GPR:$cmp, GPR:$new, GPR:$addr)>;
    def : Pat<(!cast<PatFrag>(AtomicOp#"_acquire") (XLenVT GPR:$addr),
                                                   (vt GPR:$cmp),
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 102-117: TableGen record definition / TableGen 记录定义
```tablegen
                                                   (vt GPR:$new)),
              (!cast<RVInst>(BaseInst) GPR:$cmp, GPR:$new, GPR:$addr)>;
    def : Pat<(!cast<PatFrag>(AtomicOp#"_release") (XLenVT GPR:$addr),
                                                   (vt GPR:$cmp),
                                                   (vt GPR:$new)),
              (!cast<RVInst>(BaseInst) GPR:$cmp, GPR:$new, GPR:$addr)>;
    def : Pat<(!cast<PatFrag>(AtomicOp#"_acq_rel") (XLenVT GPR:$addr),
                                                   (vt GPR:$cmp),
                                                   (vt GPR:$new)),
              (!cast<RVInst>(BaseInst) GPR:$cmp, GPR:$new, GPR:$addr)>;
    def : Pat<(!cast<PatFrag>(AtomicOp#"_seq_cst") (XLenVT GPR:$addr),
                                                   (vt GPR:$cmp),
                                                   (vt GPR:$new)),
              (!cast<RVInst>(BaseInst) GPR:$cmp, GPR:$new, GPR:$addr)>;
  } // Predicates = !listconcat([HasStdExtZacas, HasStdExtZtso], ExtraPreds)
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 118-132: TableGen class WRSInst<bits<12> / TableGen 类 WRSInst<bits<12>
```tablegen

defm : AMOCASPat<"atomic_cmp_swap_i32", "AMOCAS_W">;
defm : AMOCASPat<"atomic_cmp_swap_i64", "AMOCAS_D_RV64", i64, [IsRV64]>;

//===----------------------------------------------------------------------===//
// Zawrs (Wait-on-Reservation-Set)
//===----------------------------------------------------------------------===//

let hasSideEffects = 1, mayLoad = 0, mayStore = 0 in
class WRSInst<bits<12> funct12, string opcodestr>
    : RVInstI<0b000, OPC_SYSTEM, (outs), (ins), opcodestr, ""> {
  let rs1 = 0;
  let rd = 0;
  let imm12 = funct12;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 133-142: TableGen record WRS_NTO / TableGen 记录 WRS_NTO
```tablegen

let Predicates = [HasStdExtZawrs] in {
def WRS_NTO : WRSInst<0b000000001101, "wrs.nto">, Sched<[]>;
def WRS_STO : WRSInst<0b000000011101, "wrs.sto">, Sched<[]>;
} // Predicates = [HasStdExtZawrs]

//===----------------------------------------------------------------------===//
// Zabha (Byte and Halfword Atomic Memory Operations)
//===----------------------------------------------------------------------===//
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 143-162: Bulk record instantiation AMOSWAP_B / 批量记录实例化 AMOSWAP_B
```tablegen
let Predicates = [HasStdExtZabha], IsSignExtendingOpW = 1 in {
defm AMOSWAP_B  : AMO_rr_aq_rl<0b00001, 0b000, "amoswap.b">,
                  Sched<[WriteAtomicB, ReadAtomicBA, ReadAtomicBD]>;
defm AMOADD_B   : AMO_rr_aq_rl<0b00000, 0b000, "amoadd.b">,
                  Sched<[WriteAtomicB, ReadAtomicBA, ReadAtomicBD]>;
defm AMOXOR_B   : AMO_rr_aq_rl<0b00100, 0b000, "amoxor.b">,
                  Sched<[WriteAtomicB, ReadAtomicBA, ReadAtomicBD]>;
defm AMOAND_B   : AMO_rr_aq_rl<0b01100, 0b000, "amoand.b">,
                  Sched<[WriteAtomicB, ReadAtomicBA, ReadAtomicBD]>;
defm AMOOR_B    : AMO_rr_aq_rl<0b01000, 0b000, "amoor.b">,
                  Sched<[WriteAtomicB, ReadAtomicBA, ReadAtomicBD]>;
defm AMOMIN_B   : AMO_rr_aq_rl<0b10000, 0b000, "amomin.b">,
                  Sched<[WriteAtomicB, ReadAtomicBA, ReadAtomicBD]>;
defm AMOMAX_B   : AMO_rr_aq_rl<0b10100, 0b000, "amomax.b">,
                  Sched<[WriteAtomicB, ReadAtomicBA, ReadAtomicBD]>;
defm AMOMINU_B  : AMO_rr_aq_rl<0b11000, 0b000, "amominu.b">,
                  Sched<[WriteAtomicB, ReadAtomicBA, ReadAtomicBD]>;
defm AMOMAXU_B  : AMO_rr_aq_rl<0b11100, 0b000, "amomaxu.b">,
                  Sched<[WriteAtomicB, ReadAtomicBA, ReadAtomicBD]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 163-181: Bulk record instantiation AMOSWAP_H / 批量记录实例化 AMOSWAP_H
```tablegen
defm AMOSWAP_H  : AMO_rr_aq_rl<0b00001, 0b001, "amoswap.h">,
                  Sched<[WriteAtomicH, ReadAtomicHA, ReadAtomicHD]>;
defm AMOADD_H   : AMO_rr_aq_rl<0b00000, 0b001, "amoadd.h">,
                  Sched<[WriteAtomicH, ReadAtomicHA, ReadAtomicHD]>;
defm AMOXOR_H   : AMO_rr_aq_rl<0b00100, 0b001, "amoxor.h">,
                  Sched<[WriteAtomicH, ReadAtomicHA, ReadAtomicHD]>;
defm AMOAND_H   : AMO_rr_aq_rl<0b01100, 0b001, "amoand.h">,
                  Sched<[WriteAtomicH, ReadAtomicHA, ReadAtomicHD]>;
defm AMOOR_H    : AMO_rr_aq_rl<0b01000, 0b001, "amoor.h">,
                  Sched<[WriteAtomicH, ReadAtomicHA, ReadAtomicHD]>;
defm AMOMIN_H   : AMO_rr_aq_rl<0b10000, 0b001, "amomin.h">,
                  Sched<[WriteAtomicH, ReadAtomicHA, ReadAtomicHD]>;
defm AMOMAX_H   : AMO_rr_aq_rl<0b10100, 0b001, "amomax.h">,
                  Sched<[WriteAtomicH, ReadAtomicHA, ReadAtomicHD]>;
defm AMOMINU_H  : AMO_rr_aq_rl<0b11000, 0b001, "amominu.h">,
                  Sched<[WriteAtomicH, ReadAtomicHA, ReadAtomicHD]>;
defm AMOMAXU_H  : AMO_rr_aq_rl<0b11100, 0b001, "amomaxu.h">,
                  Sched<[WriteAtomicH, ReadAtomicHA, ReadAtomicHD]>;
}
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 182-200: Bulk record instantiation AMOCAS_B / 批量记录实例化 AMOCAS_B
```tablegen

// If Zacas extension is also implemented, Zabha further provides AMOCAS.[B|H].
let Predicates = [HasStdExtZabha, HasStdExtZacas], IsSignExtendingOpW = 1 in {
defm AMOCAS_B : AMO_cas_aq_rl<0b00101, 0b000, "amocas.b", GPR>;
defm AMOCAS_H : AMO_cas_aq_rl<0b00101, 0b001, "amocas.h", GPR>;
}

/// AMOs

defm : AMOPat<"atomic_swap_i8", "AMOSWAP_B", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_add_i8", "AMOADD_B", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_and_i8", "AMOAND_B", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_or_i8", "AMOOR_B", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_xor_i8", "AMOXOR_B", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_max_i8", "AMOMAX_B", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_min_i8", "AMOMIN_B", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_umax_i8", "AMOMAXU_B", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_umin_i8", "AMOMINU_B", XLenVT, [HasStdExtZabha]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 201-210: Bulk record instantiation / 批量记录实例化
```tablegen
defm : AMOPat<"atomic_swap_i16", "AMOSWAP_H", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_add_i16", "AMOADD_H", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_and_i16", "AMOAND_H", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_or_i16", "AMOOR_H", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_xor_i16", "AMOXOR_H", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_max_i16", "AMOMAX_H", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_min_i16", "AMOMIN_H", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_umax_i16", "AMOMAXU_H", XLenVT, [HasStdExtZabha]>;
defm : AMOPat<"atomic_load_umin_i16", "AMOMINU_H", XLenVT, [HasStdExtZabha]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

### Lines 211-214: Bulk record instantiation / 批量记录实例化
```tablegen
/// AMOCAS

defm : AMOCASPat<"atomic_cmp_swap_i8", "AMOCAS_B", XLenVT, [HasStdExtZabha]>;
defm : AMOCASPat<"atomic_cmp_swap_i16", "AMOCAS_H", XLenVT, [HasStdExtZabha]>;
```
**EN:** This block expands a template into multiple concrete records, usually to cover a full instruction family or pattern set.

**CN:** 该区段将模板展开为多个具体记录，通常用于覆盖整组指令族或模式集合。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
