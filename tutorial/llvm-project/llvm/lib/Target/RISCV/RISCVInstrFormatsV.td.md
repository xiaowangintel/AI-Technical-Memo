# RISCVInstrFormatsV.td — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/RISCVInstrFormatsV.td`
- **Repository**: llvm/llvm-project
- **Purpose**: Defines TableGen records for instruction format classes, bit layouts, and operand encodings for RISC-V. / 使用 TableGen 定义RISC-V 的指令格式类、位布局与操作数编码。
- **Note**: This file uses the TableGen DSL to describe target metadata declaratively. / 该文件使用 TableGen DSL 以声明式方式描述目标元数据。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Commentary and design intent / 注释与设计意图
```tablegen
//===-- RISCVInstrFormatsV.td - RISC-V V Instruction Formats -*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
//  This file describes the RISC-V V extension instruction formats.
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 12-24: TableGen class RISCVVFormat<bits<3> / TableGen 类 RISCVVFormat<bits<3>
```tablegen

class RISCVVFormat<bits<3> val> {
  bits<3> Value = val;
}
def OPIVV : RISCVVFormat<0b000>;
def OPFVV : RISCVVFormat<0b001>;
def OPMVV : RISCVVFormat<0b010>;
def OPIVI : RISCVVFormat<0b011>;
def OPIVX : RISCVVFormat<0b100>;
def OPFVF : RISCVVFormat<0b101>;
def OPMVX : RISCVVFormat<0b110>;
def OPCFG : RISCVVFormat<0b111>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 25-37: TableGen class RISCVMOP<bits<2> / TableGen 类 RISCVMOP<bits<2>
```tablegen
class RISCVMOP<bits<2> val> {
  bits<2> Value = val;
}
def MOPLDUnitStride   : RISCVMOP<0b00>;
def MOPLDIndexedUnord : RISCVMOP<0b01>;
def MOPLDStrided      : RISCVMOP<0b10>;
def MOPLDIndexedOrder : RISCVMOP<0b11>;

def MOPSTUnitStride   : RISCVMOP<0b00>;
def MOPSTIndexedUnord : RISCVMOP<0b01>;
def MOPSTStrided      : RISCVMOP<0b10>;
def MOPSTIndexedOrder : RISCVMOP<0b11>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 38-48: TableGen class RISCVLUMOP<bits<5> / TableGen 类 RISCVLUMOP<bits<5>
```tablegen
class RISCVLUMOP<bits<5> val> {
  bits<5> Value = val;
}
def LUMOPUnitStride  : RISCVLUMOP<0b00000>;
def LUMOPUnitStrideMask : RISCVLUMOP<0b01011>;
def LUMOPUnitStrideWholeReg : RISCVLUMOP<0b01000>;
def LUMOPUnitStrideFF: RISCVLUMOP<0b10000>;

class RISCVSUMOP<bits<5> val> {
  bits<5> Value = val;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 49-60: TableGen class RISCVWidth<bits<4> / TableGen 类 RISCVWidth<bits<4>
```tablegen
def SUMOPUnitStride  : RISCVSUMOP<0b00000>;
def SUMOPUnitStrideMask : RISCVSUMOP<0b01011>;
def SUMOPUnitStrideWholeReg : RISCVSUMOP<0b01000>;

class RISCVWidth<bits<4> val> {
  bits<4> Value = val;
}
def LSWidth8     : RISCVWidth<0b0000>;
def LSWidth16    : RISCVWidth<0b0101>;
def LSWidth32    : RISCVWidth<0b0110>;
def LSWidth64    : RISCVWidth<0b0111>;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 61-71: TableGen class RVInstVSetiVLi<dag / TableGen 类 RVInstVSetiVLi<dag
```tablegen
class RVInstVSetiVLi<dag outs, dag ins, string opcodestr, string argstr>
    : RVInstIBase<OPCFG.Value, OPC_OP_V, outs, ins, opcodestr, argstr> {
  bits<5> uimm;
  bits<10> vtypei;

  let rs1 = uimm;

  let Inst{31} = 1;
  let Inst{30} = 1;
  let Inst{29-20} = vtypei;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 72-81: TableGen class RVInstVSetVLi<dag / TableGen 类 RVInstVSetVLi<dag
```tablegen
  let Defs = [VL, VTYPE];
}

class RVInstVSetVLi<dag outs, dag ins, string opcodestr, string argstr>
    : RVInstIBase<OPCFG.Value, OPC_OP_V, outs, ins, opcodestr, argstr> {
  bits<11> vtypei;

  let Inst{31} = 0;
  let Inst{30-20} = vtypei;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 82-95: TableGen class RVInstVSetVL<dag / TableGen 类 RVInstVSetVL<dag
```tablegen
  let Defs = [VL, VTYPE];
}

class RVInstVSetVL<dag outs, dag ins, string opcodestr, string argstr>
    : RVInstR<0b1000000, OPCFG.Value, OPC_OP_V, outs, ins, opcodestr, argstr> {
  let Defs = [VL, VTYPE];
}

class RVInstVBase<bits<6> funct6, RISCVVFormat opv, dag outs, dag ins,
                  string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> vd;
  bit vm;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 96-105: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Inst{31-26} = funct6;
  let Inst{25} = vm;
  // Inst{24-15} provide by subclasses
  let Inst{14-12} = opv.Value;
  let Inst{11-7} = vd;
  let Inst{6-0} = OPC_OP_V.Value;

  let Uses = [VL, VTYPE];
  let VMConstraint = true;
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 106-117: TableGen class RVInstVV<bits<6> / TableGen 类 RVInstVV<bits<6>
```tablegen
  let UseNamedOperandTable = true;
}

class RVInstVV<bits<6> funct6, RISCVVFormat opv, dag outs, dag ins,
               string opcodestr, string argstr>
    : RVInstVBase<funct6, opv, outs, ins, opcodestr, argstr> {
  bits<5> vs2;
  bits<5> vs1;

  let Inst{24-20} = vs2;
  let Inst{19-15} = vs1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 118-127: TableGen class RVInstVX<bits<6> / TableGen 类 RVInstVX<bits<6>
```tablegen

class RVInstVX<bits<6> funct6, RISCVVFormat opv, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInstVBase<funct6, opv, outs, ins, opcodestr, argstr> {
  bits<5> vs2;
  bits<5> rs1;

  let Inst{24-20} = vs2;
  let Inst{19-15} = rs1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 128-137: TableGen class RVInstIVI<bits<6> / TableGen 类 RVInstIVI<bits<6>
```tablegen

class RVInstIVI<bits<6> funct6, dag outs, dag ins, string opcodestr,
                string argstr>
    : RVInstVBase<funct6, OPIVI, outs, ins, opcodestr, argstr> {
  bits<5> vs2;
  bits<5> imm;

  let Inst{24-20} = vs2;
  let Inst{19-15} = imm;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 138-147: TableGen class RVInstVUnary<bits<6> / TableGen 类 RVInstVUnary<bits<6>
```tablegen

class RVInstVUnary<bits<6> funct6, bits<5> vs1, RISCVVFormat opv, dag outs,
                   dag ins, string opcodestr, string argstr>
    : RVInstVBase<funct6, opv, outs, ins, opcodestr, argstr> {
  bits<5> vs2;

  let Inst{24-20} = vs2;
  let Inst{19-15} = vs1;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 148-162: TableGen class RVInstVUnaryRd<bits<6> / TableGen 类 RVInstVUnaryRd<bits<6>
```tablegen
class RVInstVUnaryRd<bits<6> funct6, bits<5> vs1, RISCVVFormat opv, dag outs,
                     dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> rd;
  bit vm;
  bits<5> vs2;

  let Inst{31-26} = funct6;
  let Inst{25} = vm;
  let Inst{24-20} = vs2;
  let Inst{19-15} = vs1;
  let Inst{14-12} = opv.Value;
  let Inst{11-7} = rd;
  let Inst{6-0} = OPC_OP_V.Value;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 163-174: TableGen class RVInstVLoadBase<bits<3> / TableGen 类 RVInstVLoadBase<bits<3>
```tablegen
  let Uses = [VL, VTYPE];

  let UseNamedOperandTable = true;
}

class RVInstVLoadBase<bits<3> nf, RISCVWidth width, RISCVMOP mop,
                      dag outs, dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> rs1;
  bits<5> vd;
  bit vm;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 175-184: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Inst{31-29} = nf;
  let Inst{28} = width.Value{3};
  let Inst{27-26} = mop.Value;
  let Inst{25} = vm;
  // Inst{24-20} provided by subclasses
  let Inst{19-15} = rs1;
  let Inst{14-12} = width.Value{2-0};
  let Inst{11-7} = vd;
  let Inst{6-0} = OPC_LOAD_FP.Value;
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 185-196: TableGen class RVInstVLU<bits<3> / TableGen 类 RVInstVLU<bits<3>
```tablegen
  let Uses = [VL, VTYPE];
  let VMConstraint = true;

  let UseNamedOperandTable = true;
}

class RVInstVLU<bits<3> nf, RISCVWidth width, RISCVLUMOP lumop, dag outs,
                dag ins, string opcodestr, string argstr>
    : RVInstVLoadBase<nf, width, MOPLDUnitStride, outs, ins, opcodestr,
                      argstr> {
  let Inst{24-20} = lumop.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 197-210: TableGen class RVInstVLS<bits<3> / TableGen 类 RVInstVLS<bits<3>
```tablegen

class RVInstVLS<bits<3> nf, RISCVWidth width, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInstVLoadBase<nf, width, MOPLDStrided, outs, ins, opcodestr, argstr> {
  bits<5> rs2;

  let Inst{24-20} = rs2;
}

class RVInstVLX<bits<3> nf, RISCVWidth width, RISCVMOP mop, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInstVLoadBase<nf, width, mop, outs, ins, opcodestr, argstr> {
  bits<5> vs2;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 211-220: TableGen class RVInstVStoreBase<bits<3> / TableGen 类 RVInstVStoreBase<bits<3>
```tablegen
  let Inst{24-20} = vs2;
}

class RVInstVStoreBase<bits<3> nf, RISCVWidth width, RISCVMOP mop, dag outs,
                       dag ins, string opcodestr, string argstr>
    : RVInst<outs, ins, opcodestr, argstr, [], InstFormatR> {
  bits<5> rs1;
  bits<5> vs3;
  bit vm;
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 221-230: TableGen field customization and iteration / TableGen 字段定制与迭代
```tablegen
  let Inst{31-29} = nf;
  let Inst{28} = width.Value{3};
  let Inst{27-26} = mop.Value;
  let Inst{25} = vm;
  // Inst{24-20} provided by subclasses
  let Inst{19-15} = rs1;
  let Inst{14-12} = width.Value{2-0};
  let Inst{11-7} = vs3;
  let Inst{6-0} = OPC_STORE_FP.Value;
```
**EN:** This block adjusts inherited fields or iterates over parameter sets to shape many related records consistently.

**CN:** 该区段调整继承字段或遍历参数集合，以一致地塑造多条相关记录。

### Lines 231-241: TableGen class RVInstVSU<bits<3> / TableGen 类 RVInstVSU<bits<3>
```tablegen
  let Uses = [VL, VTYPE];

  let UseNamedOperandTable = true;
}

class RVInstVSU<bits<3> nf, RISCVWidth width, RISCVSUMOP sumop, dag outs,
                dag ins, string opcodestr, string argstr>
    : RVInstVStoreBase<nf, width, MOPSTUnitStride, outs, ins, opcodestr,
                       argstr> {
  let Inst{24-20} = sumop.Value;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 242-251: TableGen class RVInstVSS<bits<3> / TableGen 类 RVInstVSS<bits<3>
```tablegen

class RVInstVSS<bits<3> nf, RISCVWidth width, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInstVStoreBase<nf, width, MOPSTStrided, outs, ins, opcodestr,
                       argstr> {
  bits<5> rs2;

  let Inst{24-20} = rs2;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

### Lines 252-258: TableGen class RVInstVSX<bits<3> / TableGen 类 RVInstVSX<bits<3>
```tablegen
class RVInstVSX<bits<3> nf, RISCVWidth width, RISCVMOP mop, dag outs, dag ins,
                string opcodestr, string argstr>
    : RVInstVStoreBase<nf, width, mop, outs, ins, opcodestr, argstr> {
  bits<5> vs2;

  let Inst{24-20} = vs2;
}
```
**EN:** This block defines reusable TableGen abstractions whose fields are inherited by concrete records later in the file.

**CN:** 该区段定义可复用的 TableGen 抽象，其字段会被后续具体记录继承。

## Key Concepts / 关键概念
- **Instruction encodings** / **指令编码**
- **TableGen DSL** / **TableGen DSL**

## Dependencies / 依赖关系
- No explicit source includes in this file; dependencies are mostly implicit through surrounding backend integration. / 该文件没有显式源码级 include，依赖主要通过周边后端集成隐式体现。
