# RISCVInstrInfoZicbo.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoZicbo.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoZicbo.td - RISC-V CMO instructions ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the RISC-V instructions from the standard Base Cache
// Management Operation ISA Extensions document (Zicbom, Zicboz, and Zicbop).
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 13-25: TableGen record simm12_lsb00000 / TableGen 记录 simm12_lsb00000
```tablegen

//===----------------------------------------------------------------------===//
// Operand definitions.
//===----------------------------------------------------------------------===//

// A 12-bit signed immediate where the least significant five bits are zero.
def simm12_lsb00000 : RISCVOp,
                      ImmLeaf<XLenVT, [{return isShiftedInt<7, 5>(Imm);}]> {
  let ParserMatchClass = SImmAsmOperand<12, "Lsb00000">;
  let EncoderMethod = "getImmOpValue";
  let DecoderMethod = "decodeSImmOperand<12>";
  let OperandType = "OPERAND_SIMM12_LSB00000";
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 26-36: TableGen class CBO_r<bits<12> / TableGen 类 CBO_r<bits<12>
```tablegen

//===----------------------------------------------------------------------===//
// Instruction Class Templates
//===----------------------------------------------------------------------===//
let hasSideEffects = 0, mayLoad = 0, mayStore = 1 in
class CBO_r<bits<12> optype, string opcodestr>
    : RVInstI<0b010, OPC_MISC_MEM, (outs), (ins GPRMemZeroOffset:$rs1),
              opcodestr, "$rs1"> {
  let imm12 = optype;
  let rd = 0b00000;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 37-45: TableGen class Prefetch_ri<bits<5> / TableGen 类 Prefetch_ri<bits<5>
```tablegen

let hasSideEffects = 0, mayLoad = 1, mayStore = 1 in
class Prefetch_ri<bits<5> optype, string opcodestr>
    : RVInstS<0b110, OPC_OP_IMM, (outs), (ins GPRMem:$rs1, simm12_lsb00000:$imm12),
              opcodestr, "${imm12}(${rs1})"> {
  let Inst{11-7} = 0b00000;
  let rs2 = optype;
  let Format = InstFormatOther; // this does not follow the normal S format.
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 46-56: TableGen record CBO_CLEAN / TableGen 记录 CBO_CLEAN
```tablegen

//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

let Predicates = [HasStdExtZicbom] in {
def CBO_CLEAN : CBO_r<0b000000000001, "cbo.clean">, Sched<[]>;
def CBO_FLUSH : CBO_r<0b000000000010, "cbo.flush">, Sched<[]>;
def CBO_INVAL : CBO_r<0b000000000000, "cbo.inval">, Sched<[]>;
} // Predicates = [HasStdExtZicbom]
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 57-67: TableGen record CBO_ZERO / TableGen 记录 CBO_ZERO
```tablegen
let Predicates = [HasStdExtZicboz] in {
def CBO_ZERO : CBO_r<0b000000000100, "cbo.zero">, Sched<[]>;
} // Predicates = [HasStdExtZicboz]

// prefetch hints are always available in the assembler and disassembler, even
// without enabling Zicbop, per psABI decision
// (riscv-non-isa/riscv-elf-psabi-doc#474).
def PREFETCH_I : Prefetch_ri<0b00000, "prefetch.i">, Sched<[]>;
def PREFETCH_R : Prefetch_ri<0b00001, "prefetch.r">, Sched<[]>;
def PREFETCH_W : Prefetch_ri<0b00011, "prefetch.w">, Sched<[]>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 68-83: TableGen record AddrRegImmLsb00000 / TableGen 记录 AddrRegImmLsb00000
```tablegen
//===----------------------------------------------------------------------===//
// Patterns
//===----------------------------------------------------------------------===//

def AddrRegImmLsb00000 : ComplexPattern<iPTR, 2, "SelectAddrRegImmLsb00000">;

let Predicates = [NoVendorXMIPSCBOP] in {
  def : Pat<(prefetch (AddrRegImmLsb00000 (XLenVT GPR:$rs1), simm12_lsb00000:$imm12),
                      timm, timm, (i32 0)),
            (PREFETCH_I GPR:$rs1, simm12_lsb00000:$imm12)>;
  def : Pat<(prefetch (AddrRegImmLsb00000 (XLenVT GPR:$rs1), simm12_lsb00000:$imm12),
                      (i32 0), timm, (i32 1)),
            (PREFETCH_R GPR:$rs1, simm12_lsb00000:$imm12)>;
  def : Pat<(prefetch (AddrRegImmLsb00000 (XLenVT GPR:$rs1), simm12_lsb00000:$imm12),
                      (i32 1), timm, (i32 1)),
            (PREFETCH_W GPR:$rs1, simm12_lsb00000:$imm12)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 84-84: Definitions and supporting logic / 定义与支撑逻辑
```tablegen
} // Predicates = [NoVendorXMIPSCBOP]
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
