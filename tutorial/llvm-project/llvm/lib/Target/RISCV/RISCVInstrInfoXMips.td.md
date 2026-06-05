# RISCVInstrInfoXMips.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrInfoXMips.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for declarative instruction records, selection patterns, and extension-specific metadata for the RISC-V backend. / 使用 TableGen 定义RISC-V 后端的声明式指令记录、选择模式与扩展专用元数据。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrInfoXMips.td -----------------------------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the vendor extensions defined by MIPS.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-22: TableGen record uimm7_lsb000 / TableGen 记录 uimm7_lsb000
```tablegen

//===----------------------------------------------------------------------===//
// Operand definitions.
//===----------------------------------------------------------------------===//

// A 7-bit unsigned immediate where the least significant three bits are zero.
def uimm7_lsb000 : RISCVUImmLsbZeroOp<7, 3>;

// A 9-bit unsigned offset
def uimm9 : RISCVUImmOp<9>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 23-37: TableGen class LDPFormat<dag / TableGen 类 LDPFormat<dag
```tablegen
// Custom prefetch ADDR selector
def AddrRegImm9 : ComplexPattern<iPTR, 2, "SelectAddrRegImm9">;

//===----------------------------------------------------------------------===//
// MIPS custom instruction formats
//===----------------------------------------------------------------------===//

// Load double pair format.
class LDPFormat<dag outs, dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatI> {
  bits<7> imm7;
  bits<5> rs1;
  bits<5> rd1;
  bits<5> rd2;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 38-54: TableGen class LWPFormat<dag / TableGen 类 LWPFormat<dag
```tablegen
  let Inst{31-27} = rd2;
  let Inst{26-23} = imm7{6-3};
  let Inst{22-20} = 0b000;
  let Inst{19-15} = rs1;
  let Inst{14-12} = 0b100;
  let Inst{11-7} = rd1;
  let Inst{6-0} = OPC_CUSTOM_0.Value;
}

// Load word pair format.
class LWPFormat<dag outs, dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatI> {
  bits<7> imm7;
  bits<5> rs1;
  bits<5> rd1;
  bits<5> rd2;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 55-71: TableGen class SDPFormat<dag / TableGen 类 SDPFormat<dag
```tablegen
  let Inst{31-27} = rd2;
  let Inst{26-22} = imm7{6-2};
  let Inst{21-20} = 0b01;
  let Inst{19-15} = rs1;
  let Inst{14-12} = 0b100;
  let Inst{11-7} = rd1;
  let Inst{6-0} = OPC_CUSTOM_0.Value;
}

// Store double pair format.
class SDPFormat<dag outs, dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatI> {
  bits<7> imm7;
  bits<5> rs3;
  bits<5> rs2;
  bits<5> rs1;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 72-81: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Inst{31-27} = rs3;
  let Inst{26-25} = imm7{6-5};
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = 0b101;
  let Inst{11-10} = imm7{4-3};
  let Inst{9-7} = 0b000;
  let Inst{6-0} = OPC_CUSTOM_0.Value;
}
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 82-98: TableGen class SWPFormat<dag / TableGen 类 SWPFormat<dag
```tablegen
// Store word pair format.
class SWPFormat<dag outs, dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatI> {
  bits<7> imm7;
  bits<5> rs3;
  bits<5> rs2;
  bits<5> rs1;

  let Inst{31-27} = rs3;
  let Inst{26-25} = imm7{6-5};
  let Inst{24-20} = rs2;
  let Inst{19-15} = rs1;
  let Inst{14-12} = 0b101;
  let Inst{11-9} = imm7{4-2};
  let Inst{8-7} = 0b01;
  let Inst{6-0} = OPC_CUSTOM_0.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 99-114: TableGen class Mips_prefetch_ri<dag / TableGen 类 Mips_prefetch_ri<dag
```tablegen

// Prefetch format.
let hasSideEffects = 0, mayLoad = 1, mayStore = 1 in
class Mips_prefetch_ri<dag outs, dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatI> {
  bits<9> imm9;
  bits<5> rs1;
  bits<5> hint;

  let Inst{31-29} = 0b000;
  let Inst{28-20} = imm9;
  let Inst{19-15} = rs1;
  let Inst{14-12} = 0b000;
  let Inst{11-7} = hint;
  let Inst{6-0} = OPC_CUSTOM_0.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 115-124: TableGen class MIPSExtInst_ri<bits<6> / TableGen 类 MIPSExtInst_ri<bits<6>
```tablegen

// MIPS Custom Barrier Insns Format.
let hasSideEffects = 1, mayLoad = 0, mayStore = 0 in
class MIPSExtInst_ri<bits<6> shimm5, string opcodestr>
    : RVInstIShift<0b00000, 0b001, OPC_OP_IMM, (outs), (ins), opcodestr, ""> {
  let shamt = shimm5;
  let rd = 0;
  let rs1 = 0;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 125-139: TableGen record MIPS_EHB / TableGen 记录 MIPS_EHB
```tablegen
//===----------------------------------------------------------------------===//
// MIPS extensions
//===----------------------------------------------------------------------===//
let Predicates = [HasVendorXMIPSEXECTL], DecoderNamespace = "XMIPS" in {
  def MIPS_EHB : MIPSExtInst_ri<0b000011, "mips.ehb">;
  def MIPS_IHB : MIPSExtInst_ri<0b000001, "mips.ihb">;
  def MIPS_PAUSE : MIPSExtInst_ri<0b000101, "mips.pause">;
}

let Predicates = [HasVendorXMIPSEXECTL] in {
  // Intrinsics
  def : Pat<(int_riscv_mips_pause), (MIPS_PAUSE)>;
  def : Pat<(int_riscv_mips_ihb), (MIPS_IHB)>;
  def : Pat<(int_riscv_mips_ehb), (MIPS_EHB)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 140-156: TableGen record MIPS_PREF / TableGen 记录 MIPS_PREF
```tablegen

let Predicates = [HasVendorXMIPSCBOP], DecoderNamespace = "XMIPS" in {
  def MIPS_PREF : Mips_prefetch_ri<(outs), (ins GPR:$rs1, uimm9:$imm9, uimm5:$hint),
                                    "mips.pref", "$hint, ${imm9}(${rs1})">,
                   Sched<[]>;
}

let Predicates = [HasVendorXMIPSCBOP] in {
  // Prefetch Data Write.
  def : Pat<(prefetch (AddrRegImm9  (XLenVT GPR:$rs1), uimm9:$imm9),
             (i32 1), timm, (i32 1)),
            (MIPS_PREF GPR:$rs1, uimm9:$imm9, 9)>;
  // Prefetch Data Read.
  def : Pat<(prefetch (AddrRegImm9 (XLenVT GPR:$rs1), uimm9:$imm9),
             (i32 0), timm, (i32 1)),
            (MIPS_PREF GPR:$rs1, uimm9:$imm9, 8)>;
}
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 157-173: TableGen record MIPS_CCMOV / TableGen 记录 MIPS_CCMOV
```tablegen

let Predicates = [HasVendorXMIPSCMov], hasSideEffects = 0, mayLoad = 0, mayStore = 0,
    DecoderNamespace = "XMIPS" in {
def MIPS_CCMOV : RVInstR4<0b11, 0b011, OPC_CUSTOM_0, (outs GPR:$rd),
                          (ins GPR:$rs1, GPR:$rs2, GPR:$rs3),
                          "mips.ccmov", "$rd, $rs2, $rs1, $rs3">,
                 Sched<[]>;
}

let Predicates = [UseMIPSCCMovInsn] in {
def : Pat<(select (riscv_setne (XLenVT GPR:$rs2)),
                  (XLenVT GPR:$rs1), (XLenVT GPR:$rs3)),
          (MIPS_CCMOV GPR:$rs1, GPR:$rs2, GPR:$rs3)>;
def : Pat<(select (riscv_seteq (XLenVT GPR:$rs2)),
                  (XLenVT GPR:$rs3), (XLenVT GPR:$rs1)),
          (MIPS_CCMOV GPR:$rs1, GPR:$rs2, GPR:$rs3)>;
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 174-188: TableGen record MIPS_LWP / TableGen 记录 MIPS_LWP
```tablegen
def : Pat<(select (XLenVT GPR:$rs2), (XLenVT GPR:$rs1), (XLenVT GPR:$rs3)),
          (MIPS_CCMOV GPR:$rs1, GPR:$rs2, GPR:$rs3)>;
}

let Predicates = [HasVendorXMIPSLSP], hasSideEffects = 0,
    DecoderNamespace = "XMIPS" in {
let mayLoad = 1, mayStore = 0 in {
def MIPS_LWP : LWPFormat<(outs GPR:$rd1, GPR:$rd2), (ins GPR:$rs1, uimm7_lsb00:$imm7),
                         "mips.lwp", "$rd1, $rd2, ${imm7}(${rs1})">,
               Sched<[WriteLDW, WriteLDW, ReadMemBase]>;
def MIPS_LDP : LDPFormat<(outs GPR:$rd1, GPR:$rd2), (ins GPR:$rs1, uimm7_lsb000:$imm7),
                         "mips.ldp", "$rd1, $rd2, ${imm7}(${rs1})">,
               Sched<[WriteLDD, WriteLDD, ReadMemBase]>;
} // mayLoad = 1, mayStore = 0
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

### Lines 189-197: TableGen record MIPS_SWP / TableGen 记录 MIPS_SWP
```tablegen
let mayLoad = 0, mayStore = 1 in {
def MIPS_SWP : SWPFormat<(outs), (ins GPR:$rs2, GPR:$rs3, GPR:$rs1, uimm7_lsb00:$imm7),
                         "mips.swp", "$rs2, $rs3, ${imm7}(${rs1})">,
               Sched<[WriteSTW, ReadStoreData, ReadStoreData, ReadMemBase]>;
def MIPS_SDP : SDPFormat<(outs), (ins GPR:$rs2, GPR:$rs3, GPR:$rs1, uimm7_lsb000:$imm7),
                         "mips.sdp", "$rs2, $rs3, ${imm7}(${rs1})">,
               Sched<[WriteSTD, ReadStoreData, ReadStoreData, ReadMemBase]>;
} // mayLoad = 0, mayStore = 1
} // Predicates = [HasVendorXMIPSLSP], hasSideEffects = 0, DecoderNamespace = "XMIPS"
```
**EN:** This block introduces concrete declarative metadata that TableGen backends later turn into generated matchers, encoders, or schedulers.

**CN:** 该区段引入具体的声明式元数据，随后会被 TableGen 后端转换为生成的匹配器、编码器或调度信息。

## Key Concepts / 关键概念
- **Instruction definitions** / **指令定义**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
