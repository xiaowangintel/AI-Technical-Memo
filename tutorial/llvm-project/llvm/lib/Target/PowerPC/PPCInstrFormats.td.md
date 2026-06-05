# PPCInstrFormats.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCInstrFormats.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PowerPCInstrFormats.td - PowerPC Instruction Formats. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCInstrFormats.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===- PowerPCInstrFormats.td - PowerPC Instruction Formats --*- tablegen -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-8

```tablegen
//===----------------------------------------------------------------------===//
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 9-15

```tablegen
//===----------------------------------------------------------------------===//
//
// PowerPC instruction formats

class I<bits<6> opcode, dag OOL, dag IOL, string asmstr, InstrItinClass itin>
        : Instruction {
  field bits<32> Inst;
```
- **EN**: Declares a backend-facing type `I` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `I`，并勾勒出周边代码会依赖的接口或状态。

### Lines 16-22

```tablegen
  let Size = 4;

  bit PPC64 = 0;  // Default value, override with isPPC64

  let Namespace = "PPC";
  let Inst{0...5} = opcode;
  let OutOperandList = OOL;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 23-33

```tablegen
  let InOperandList = IOL;
  let AsmString = asmstr;
  let Itinerary = itin;

  bits<1> PPC970_First = 0;
  bits<1> PPC970_Single = 0;
  bits<1> PPC970_Cracked = 0;
  bits<3> PPC970_Unit = 0;

  /// These fields correspond to the fields in PPCInstrInfo.h.  Any changes to
  /// these must be reflected there!  See comments there for what these are.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。 这一段包含调度或处理器模型元数据。

### Lines 34-40

```tablegen
  let TSFlags{0}   = PPC970_First;
  let TSFlags{1}   = PPC970_Single;
  let TSFlags{2}   = PPC970_Cracked;
  let TSFlags{5...3} = PPC970_Unit;

  // Indicate that this instruction is of type X-Form Load or Store
  bits<1> XFormMemOp = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 41-49

```tablegen
  let TSFlags{6}  = XFormMemOp;

  // Indicate that this instruction is prefixed.
  bits<1> Prefixed = 0;
  let TSFlags{7}  = Prefixed;
  
  // Indicate that this instruction produces a result that is sign extended from
  // 32 bits to 64 bits.
  bits<1> SExt32To64 = 0;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Indicate that this instruction is prefixed.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Indicate that this instruction is prefixed.”。

### Lines 50-58

```tablegen
  let TSFlags{8} = SExt32To64;

  // Indicate that this instruction produces a result that is zero extended from
  // 32 bits to 64 bits.
  bits<1> ZExt32To64 = 0;
  let TSFlags{9} = ZExt32To64;

  // Indicate that this instruction takes a register+immediate memory operand.
  bits<1> MemriOp = 0;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Indicate that this instruction produces a result that is zero extended from".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Indicate that this instruction produces a result that is zero extended from”。

### Lines 59-70

```tablegen
  let TSFlags{10} = MemriOp;

  // Fields used for relation models.
  string BaseName = "";

  // For cases where multiple instruction definitions really represent the
  // same underlying instruction but with one definition for 64-bit arguments
  // and one for 32-bit arguments, this bit breaks the degeneracy between
  // the two forms and allows TableGen to generate mapping tables.
  bit Interpretation64Bit = 0;
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Fields used for relation models.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Fields used for relation models.”。

### Lines 71-77

```tablegen
class PPC970_DGroup_First   { bits<1> PPC970_First = 1;  }
class PPC970_DGroup_Single  { bits<1> PPC970_Single = 1; }
class PPC970_DGroup_Cracked { bits<1> PPC970_Cracked = 1; }
class PPC970_MicroCode;

class PPC970_Unit_Pseudo   { bits<3> PPC970_Unit = 0;   }
class PPC970_Unit_FXU      { bits<3> PPC970_Unit = 1;   }
```
- **EN**: Declares a backend-facing type `PPC970_DGroup_First`, `PPC970_DGroup_Single`, `PPC970_DGroup_Cracked` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `PPC970_DGroup_First`, `PPC970_DGroup_Single`, `PPC970_DGroup_Cracked`，并勾勒出周边代码会依赖的接口或状态。

### Lines 78-84

```tablegen
class PPC970_Unit_LSU      { bits<3> PPC970_Unit = 2;   }
class PPC970_Unit_FPU      { bits<3> PPC970_Unit = 3;   }
class PPC970_Unit_CRU      { bits<3> PPC970_Unit = 4;   }
class PPC970_Unit_VALU     { bits<3> PPC970_Unit = 5;   }
class PPC970_Unit_VPERM    { bits<3> PPC970_Unit = 6;   }
class PPC970_Unit_BRU      { bits<3> PPC970_Unit = 7;   }
```
- **EN**: Declares a backend-facing type `PPC970_Unit_LSU`, `PPC970_Unit_FPU`, `PPC970_Unit_CRU` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `PPC970_Unit_LSU`, `PPC970_Unit_FPU`, `PPC970_Unit_CRU`，并勾勒出周边代码会依赖的接口或状态。

### Lines 85-92

```tablegen
class XFormMemOp { bits<1> XFormMemOp = 1; }
class SExt32To64 { bits<1> SExt32To64 = 1; }
class ZExt32To64 { bits<1> ZExt32To64 = 1; }
class MemriOp      { bits<1> MemriOp = 1;      }

// Two joined instructions; used to emit two adjacent instructions as one.
// The itinerary from the first instruction is used for scheduling and
// classification.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Two joined instructions; used to emit two adjacent instructions as one.". Notable symbols in this range include `XFormMemOp`, `SExt32To64`, `ZExt32To64`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Two joined instructions; used to emit two adjacent instructions as one.”。 该区间中较显眼的符号包括 `XFormMemOp`, `SExt32To64`, `ZExt32To64`。

### Lines 93-100

```tablegen
class I2<bits<6> opcode1, bits<6> opcode2, dag OOL, dag IOL, string asmstr,
         InstrItinClass itin>
        : Instruction {
  field bits<64> Inst;
  let Size = 8;

  bit PPC64 = 0;  // Default value, override with isPPC64
```
- **EN**: Declares a backend-facing type `I2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `I2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 101-115

```tablegen
  let Namespace = "PPC";
  let Inst{0...5} = opcode1;
  let Inst{32...37} = opcode2;
  let OutOperandList = OOL;
  let InOperandList = IOL;
  let AsmString = asmstr;
  let Itinerary = itin;

  bits<1> PPC970_First = 0;
  bits<1> PPC970_Single = 0;
  bits<1> PPC970_Cracked = 0;
  bits<3> PPC970_Unit = 0;

  /// These fields correspond to the fields in PPCInstrInfo.h.  Any changes to
  /// these must be reflected there!  See comments there for what these are.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。 这一段包含调度或处理器模型元数据。

### Lines 116-126

```tablegen
  let TSFlags{0}   = PPC970_First;
  let TSFlags{1}   = PPC970_Single;
  let TSFlags{2}   = PPC970_Cracked;
  let TSFlags{5...3} = PPC970_Unit;

  // Fields used for relation models.
  string BaseName = "";
  bit Interpretation64Bit = 0;
}

// Base class for all X-Form memory instructions
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。

### Lines 127-134

```tablegen
class IXFormMemOp<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                  InstrItinClass itin>
        :I<opcode, OOL, IOL, asmstr, itin>, XFormMemOp;

// 1.7.1 I-Form
class IForm<bits<6> opcode, bit aa, bit lk, dag OOL, dag IOL, string asmstr,
            InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
```
- **EN**: Declares a backend-facing type `IXFormMemOp`, `IForm` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `IXFormMemOp`, `IForm`，并勾勒出周边代码会依赖的接口或状态。

### Lines 135-143

```tablegen
  let Pattern = pattern;
  bits<24> LI;

  let Inst{6...29}  = LI;
  let Inst{30}    = aa;
  let Inst{31}    = lk;
}

// 1.7.2 B-Form
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 144-150

```tablegen
class BForm<bits<6> opcode, bit aa, bit lk, dag OOL, dag IOL, string asmstr>
  : I<opcode, OOL, IOL, asmstr, IIC_BrB> {
  bits<7> BIBO;  // 2 bits of BI and 5 bits of BO.
  bits<3>  CR;
  bits<14> BD;

  bits<5> BI;
```
- **EN**: Declares a backend-facing type `BForm` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `BForm`，并勾勒出周边代码会依赖的接口或状态。

### Lines 151-157

```tablegen
  let BI{0...1} = BIBO{5...6};
  let BI{2...4} = CR{0...2};

  let Inst{6...10}  = BIBO{4...0};
  let Inst{11...15} = BI;
  let Inst{16...29} = BD;
  let Inst{30}    = aa;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 158-164

```tablegen
  let Inst{31}    = lk;
}

class BForm_1<bits<6> opcode, bits<5> bo, bit aa, bit lk, dag OOL, dag IOL,
             string asmstr>
  : BForm<opcode, aa, lk, OOL, IOL, asmstr> {
  let BIBO{4...0} = bo;
```
- **EN**: Declares a backend-facing type `BForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `BForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 165-173

```tablegen
  let BIBO{6...5} = 0;
  let CR = 0;
}

class BForm_2<bits<6> opcode, bits<5> bo, bits<5> bi, bit aa, bit lk,
              dag OOL, dag IOL, string asmstr>
  : I<opcode, OOL, IOL, asmstr, IIC_BrB> {
  bits<14> BD;
```
- **EN**: Declares a backend-facing type `BForm_2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `BForm_2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 174-180

```tablegen
  let Inst{6...10}  = bo;
  let Inst{11...15} = bi;
  let Inst{16...29} = BD;
  let Inst{30}    = aa;
  let Inst{31}    = lk;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 181-187

```tablegen
class BForm_3<bits<6> opcode, bit aa, bit lk,
              dag OOL, dag IOL, string asmstr>
  : I<opcode, OOL, IOL, asmstr, IIC_BrB> {
  bits<5> BO;
  bits<5> BI;
  bits<14> BD;
```
- **EN**: Declares a backend-facing type `BForm_3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `BForm_3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 188-194

```tablegen
  let Inst{6...10}  = BO;
  let Inst{11...15} = BI;
  let Inst{16...29} = BD;
  let Inst{30}    = aa;
  let Inst{31}    = lk;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 195-202

```tablegen
class BForm_3_at<bits<6> opcode, bit aa, bit lk,
                 dag OOL, dag IOL, string asmstr>
  : I<opcode, OOL, IOL, asmstr, IIC_BrB> {
  bits<5> BO;
  bits<2> at;
  bits<5> BI;
  bits<14> BD;
```
- **EN**: Declares a backend-facing type `BForm_3_at` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `BForm_3_at`，并勾勒出周边代码会依赖的接口或状态。

### Lines 203-210

```tablegen
  let Inst{6...8}   = BO{4...2};
  let Inst{9...10}  = at;
  let Inst{11...15} = BI;
  let Inst{16...29} = BD;
  let Inst{30}    = aa;
  let Inst{31}    = lk;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 211-217

```tablegen
class
BForm_4<bits<6> opcode, bits<5> bo, bit aa, bit lk,
              dag OOL, dag IOL, string asmstr>
  : I<opcode, OOL, IOL, asmstr, IIC_BrB> {
  bits<5> BI;
  bits<14> BD;
```
- **EN**: Declares a backend-facing type `BForm_4` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `BForm_4`，并勾勒出周边代码会依赖的接口或状态。

### Lines 218-225

```tablegen
  let Inst{6...10}  = bo;
  let Inst{11...15} = BI;
  let Inst{16...29} = BD;
  let Inst{30}    = aa;
  let Inst{31}    = lk;
}

// 1.7.3 SC-Form
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 226-233

```tablegen
class SCForm<bits<6> opcode, bits<1> xo1, bits<1> xo2,
                     dag OOL, dag IOL, string asmstr, InstrItinClass itin,
                     list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<7>  LEV;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `SCForm` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `SCForm`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 234-246

```tablegen
  let Inst{20...26} = LEV;
  let Inst{30}    = xo1;
  let Inst{31}    = xo2;
}

// 1.7.4 D-Form
class DForm_base<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                 InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5>  RST;
  bits<5>  RA;
  bits<16> D;
```
- **EN**: Declares a backend-facing type `DForm_base` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DForm_base`，并勾勒出周边代码会依赖的接口或状态。

### Lines 247-253

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RST;
  let Inst{11...15} = RA;
  let Inst{16...31} = D;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 254-267

```tablegen
class DForm_1<bits<6> opcode, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : DForm_base<opcode, OOL, IOL, asmstr, itin, pattern>, MemriOp {
}

class DForm_2<bits<6> opcode, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : DForm_base<opcode, OOL, IOL, asmstr, itin, pattern> {

  // Even though ADDIC_rec does not really have an RC bit, provide
  // the declaration of one here so that isRecordForm has something to set.
  bit RC = 0;
}
```
- **EN**: Declares a backend-facing type `DForm_1`, `DForm_2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DForm_1`, `DForm_2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 268-275

```tablegen
class DForm_2_r0<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                 InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5>  RST;
  bits<16> D;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `DForm_2_r0` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `DForm_2_r0`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 276-287

```tablegen
  let Inst{6...10}  = RST;
  let Inst{11...15} = 0;
  let Inst{16...31} = D;
}

class DForm_4<bits<6> opcode, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5>  RA;
  bits<5>  RST;
  bits<16> D;
```
- **EN**: Declares a backend-facing type `DForm_4` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DForm_4`，并勾勒出周边代码会依赖的接口或状态。

### Lines 288-294

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RST;
  let Inst{11...15} = RA;
  let Inst{16...31} = D;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 295-303

```tablegen
class DForm_4_zero<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                   InstrItinClass itin, list<dag> pattern>
  : DForm_1<opcode, OOL, IOL, asmstr, itin, pattern> {
  let RST = 0;
  let RA = 0;
  let D = 0;
  let MemriOp = 0;
}
```
- **EN**: Declares a backend-facing type `DForm_4_zero` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DForm_4_zero`，并勾勒出周边代码会依赖的接口或状态。

### Lines 304-312

```tablegen
class DForm_4_fixedreg_zero<bits<6> opcode, bits<5> R, dag OOL, dag IOL,
                            string asmstr, InstrItinClass itin,
                            list<dag> pattern>
  : DForm_4<opcode, OOL, IOL, asmstr, itin, pattern> {
  let RST = R;
  let RA = R;
  let D = 0;
}
```
- **EN**: Declares a backend-facing type `DForm_4_fixedreg_zero` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DForm_4_fixedreg_zero`，并勾勒出周边代码会依赖的接口或状态。

### Lines 313-320

```tablegen
class IForm_and_DForm_1<bits<6> opcode1, bit aa, bit lk, bits<6> opcode2,
            dag OOL, dag IOL, string asmstr,
            InstrItinClass itin, list<dag> pattern>
         : I2<opcode1, opcode2, OOL, IOL, asmstr, itin> {
  bits<5>  RST;
  bits<5>  RA;
  bits<16> D;
```
- **EN**: Declares a backend-facing type `IForm_and_DForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `IForm_and_DForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 321-327

```tablegen
  let Pattern = pattern;
  bits<24> LI;

  let Inst{6...29}  = LI;
  let Inst{30}    = aa;
  let Inst{31}    = lk;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 328-338

```tablegen
  let Inst{38...42}  = RST;
  let Inst{43...47} = RA;
  let Inst{48...63} = D;
}

// This is used to emit BL8+NOP.
class IForm_and_DForm_4_zero<bits<6> opcode1, bit aa, bit lk, bits<6> opcode2,
            dag OOL, dag IOL, string asmstr,
            InstrItinClass itin, list<dag> pattern>
         :  IForm_and_DForm_1<opcode1, aa, lk, opcode2,
                              OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `IForm_and_DForm_4_zero` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `IForm_and_DForm_4_zero`，并勾勒出周边代码会依赖的接口或状态。

### Lines 339-351

```tablegen
  let RST = 0;
  let RA = 0;
  let D = 0;
}

class DForm_5<bits<6> opcode, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3>  BF;
  bits<1>  L;
  bits<5>  RA;
  bits<16> D;
```
- **EN**: Declares a backend-facing type `DForm_5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DForm_5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 352-358

```tablegen
  let Inst{6...8}   = BF;
  let Inst{9}     = 0;
  let Inst{10}    = L;
  let Inst{11...15} = RA;
  let Inst{16...31} = D;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 359-368

```tablegen
class DForm_5_ext<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                  InstrItinClass itin>
  : DForm_5<opcode, OOL, IOL, asmstr, itin> {
  let L = PPC64;
}

class DForm_6<bits<6> opcode, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin> 
  : DForm_5<opcode, OOL, IOL, asmstr, itin>;
```
- **EN**: Declares a backend-facing type `DForm_5_ext`, `DForm_6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DForm_5_ext`, `DForm_6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 369-376

```tablegen
class DForm_6_ext<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                  InstrItinClass itin>
  : DForm_6<opcode, OOL, IOL, asmstr, itin> {
  let L = PPC64;
}


// 1.7.5 DS-Form
```
- **EN**: Declares a backend-facing type `DForm_6_ext` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DForm_6_ext`，并勾勒出周边代码会依赖的接口或状态。

### Lines 377-383

```tablegen
class DSForm_1<bits<6> opcode, bits<2> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin>, MemriOp {
  bits<5>  RST;
  bits<5>  RA;
  bits<14> D;
```
- **EN**: Declares a backend-facing type `DSForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DSForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 384-392

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RST;
  let Inst{11...15} = RA;
  let Inst{16...29} = D;
  let Inst{30...31} = xo;
}

// ISA V3.0B 1.6.6 DX-Form
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 393-400

```tablegen
class DXForm<bits<6> opcode, bits<5> xo, dag OOL, dag IOL, string asmstr,
             InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5>  RT;
  bits<16> D;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `DXForm` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `DXForm`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 401-408

```tablegen
  let Inst{6...10}  = RT;
  let Inst{11...15} = D{5...1};  // d1
  let Inst{16...25} = D{15...6}; // d0
  let Inst{26...30} = xo;
  let Inst{31}    = D{0};    // d2
}

// DQ-Form: [PO T RA DQ TX XO] or [PO S RA DQ SX XO]
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 409-415

```tablegen
class DQ_RD6_RS5_DQ12<bits<6> opcode, bits<3> xo, dag OOL, dag IOL,
                      string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin>, MemriOp {
  bits<6>  XT;
  bits<5> RA;
  bits<12> DQ;
```
- **EN**: Declares a backend-facing type `DQ_RD6_RS5_DQ12` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DQ_RD6_RS5_DQ12`，并勾勒出周边代码会依赖的接口或状态。

### Lines 416-424

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = XT{4...0};
  let Inst{11...15} = RA;
  let Inst{16...27} = DQ;
  let Inst{28}    = XT{5};
  let Inst{29...31} = xo;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 425-431

```tablegen
class DQForm_RTp5_RA17_MEM<bits<6> opcode, bits<4> xo, dag OOL, dag IOL,
                           string asmstr, InstrItinClass itin,
                           list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin>, MemriOp {
  bits<5> RTp;
  bits<5> RA;
  bits<12> DQ;
```
- **EN**: Declares a backend-facing type `DQForm_RTp5_RA17_MEM` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DQForm_RTp5_RA17_MEM`，并勾勒出周边代码会依赖的接口或状态。

### Lines 432-440

```tablegen
  let Pattern = pattern;

  let Inst{6...10} =  RTp{4...0};
  let Inst{11...15} = RA;
  let Inst{16...27} = DQ;
  let Inst{28...31} = xo;
}

// 1.7.6 X-Form
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 441-447

```tablegen
class XForm_base_r3xo<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr, 
                      InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RST;
  bits<5> RA;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `XForm_base_r3xo` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_base_r3xo`，并勾勒出周边代码会依赖的接口或状态。

### Lines 448-454

```tablegen
  let Pattern = pattern;

  bit RC = 0;    // set by isRecordForm

  let Inst{6...10}  = RST;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 455-463

```tablegen
  let Inst{21...30} = xo;
  let Inst{31}    = RC;
}

class XForm_base_r3xo_memOp<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                            string asmstr, InstrItinClass itin,
                            list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern>, XFormMemOp;
```
- **EN**: Declares a backend-facing type `XForm_base_r3xo_memOp` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_base_r3xo_memOp`，并勾勒出周边代码会依赖的接口或状态。

### Lines 464-471

```tablegen
class XForm_tlb<bits<10> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin> : XForm_base_r3xo<31, xo, OOL, IOL, asmstr, itin, []> {
  let RST = 0;
}

class XForm_tlbilx<bits<10> xo, dag OOL, dag IOL, string asmstr,
      InstrItinClass itin> : XForm_base_r3xo<31, xo, OOL, IOL, asmstr, itin, []> {
  bits<5> T;
```
- **EN**: Declares a backend-facing type `XForm_tlb`, `XForm_tlbilx` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_tlb`, `XForm_tlbilx`，并勾勒出周边代码会依赖的接口或状态。

### Lines 472-482

```tablegen
  let RST = T;
}

class XForm_attn<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                 InstrItinClass itin>
  : I<opcode, OOL, IOL, asmstr, itin> {
  let Inst{21...30} = xo;
}

// This is the same as XForm_base_r3xo, but the first two operands are swapped
// when code is emitted.
```
- **EN**: Declares a backend-facing type `XForm_attn` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_attn`，并勾勒出周边代码会依赖的接口或状态。

### Lines 483-492

```tablegen
class XForm_base_r3xo_swapped
        <bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
        InstrItinClass itin> 
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RA;
  bits<5> RST;
  bits<5> RB;

  bit RC = 0;    // set by isRecordForm
```
- **EN**: Declares a backend-facing type `XForm_base_r3xo_swapped` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_base_r3xo_swapped`，并勾勒出周边代码会依赖的接口或状态。

### Lines 493-500

```tablegen
  let Inst{6...10}  = RST;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
  let Inst{31}    = RC;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 501-508

```tablegen
class XForm_1<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern>;

class XForm_1_memOp<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo_memOp<opcode, xo, OOL, IOL, asmstr, itin, pattern>;
```
- **EN**: Declares a backend-facing type `XForm_1`, `XForm_1_memOp` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_1`, `XForm_1_memOp`，并勾勒出周边代码会依赖的接口或状态。

### Lines 509-517

```tablegen
class XForm_1a<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
  let RST = 0;
}

class XForm_rs<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XForm_1a`, `XForm_rs` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_1a`, `XForm_rs`，并勾勒出周边代码会依赖的接口或状态。

### Lines 518-528

```tablegen
  let RA = 0;
  let RB = 0;
}

class XForm_tlbws<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RST;
  bits<5> RA;
  bits<1> WS;
```
- **EN**: Declares a backend-facing type `XForm_tlbws` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_tlbws`，并勾勒出周边代码会依赖的接口或状态。

### Lines 529-537

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RST;
  let Inst{11...15} = RA;
  let Inst{20}    = WS;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 538-547

```tablegen
class XForm_6<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern> 
  : XForm_base_r3xo_swapped<opcode, xo, OOL, IOL, asmstr, itin> {
  let Pattern = pattern;
}

class XForm_8<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern> 
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern>;
```
- **EN**: Declares a backend-facing type `XForm_6`, `XForm_8` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_6`, `XForm_8`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 548-554

```tablegen
class XForm_8_memOp<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern> 
  : XForm_base_r3xo_memOp<opcode, xo, OOL, IOL, asmstr, itin, pattern>;

class XForm_10<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern> 
  : XForm_base_r3xo_swapped<opcode, xo, OOL, IOL, asmstr, itin> {
```
- **EN**: Declares a backend-facing type `XForm_8_memOp`, `XForm_10` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_8_memOp`, `XForm_10`，并勾勒出周边代码会依赖的接口或状态。

### Lines 555-561

```tablegen
    let Pattern = pattern;
}

class XForm_11<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern> 
  : XForm_base_r3xo_swapped<opcode, xo, OOL, IOL, asmstr, itin> {
  let RB = 0;
```
- **EN**: Declares a backend-facing type `XForm_11` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_11`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 562-572

```tablegen
  let Pattern = pattern;
}

class XForm_16<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bits<1> L;
  bits<5> RA;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `XForm_16` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_16`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 573-581

```tablegen
  let Inst{6...8}   = BF;
  let Inst{9}     = 0;
  let Inst{10}    = L;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 582-588

```tablegen
class XForm_icbt<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                 InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<4> CT;
  bits<5> RA;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `XForm_icbt` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_icbt`，并勾勒出周边代码会依赖的接口或状态。

### Lines 589-596

```tablegen
  let Inst{6} = 0;
  let Inst{7...10} = CT;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
  let Inst{31} = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 597-603

```tablegen
class XForm_sr<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RS;
  bits<4> SR;

  let Inst{6...10} = RS;
```
- **EN**: Declares a backend-facing type `XForm_sr` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_sr`，并勾勒出周边代码会依赖的接口或状态。

### Lines 604-612

```tablegen
  let Inst{12...15} = SR;
  let Inst{21...30} = xo;
}

class XForm_mbar<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> MO;
```
- **EN**: Declares a backend-facing type `XForm_mbar` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_mbar`，并勾勒出周边代码会依赖的接口或状态。

### Lines 613-622

```tablegen
  let Inst{6...10} = MO;
  let Inst{21...30} = xo;
}

class XForm_srin<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RS;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `XForm_srin` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_srin`，并勾勒出周边代码会依赖的接口或状态。

### Lines 623-633

```tablegen
  let Inst{6...10} = RS;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
}

class XForm_mtmsr<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RS;
  bits<1> L;
```
- **EN**: Declares a backend-facing type `XForm_mtmsr` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_mtmsr`，并勾勒出周边代码会依赖的接口或状态。

### Lines 634-641

```tablegen
  let Inst{6...10} = RS;
  let Inst{15} = L;
  let Inst{21...30} = xo;
}

class XForm_16_ext<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                   InstrItinClass itin>
  : XForm_16<opcode, xo, OOL, IOL, asmstr, itin> {
```
- **EN**: Declares a backend-facing type `XForm_16_ext` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_16_ext`，并勾勒出周边代码会依赖的接口或状态。

### Lines 642-651

```tablegen
  let L = PPC64;
}

class XForm_17<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bits<5> RA;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `XForm_17` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_17`，并勾勒出周边代码会依赖的接口或状态。

### Lines 652-659

```tablegen
  let Inst{6...8}   = BF;
  let Inst{9...10}  = 0;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 660-666

```tablegen
class XForm_17a<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
  : XForm_17<opcode, xo, OOL, IOL, asmstr, itin > {
  let RA = 0;
  let Pattern = pattern;
}
```
- **EN**: Declares a backend-facing type `XForm_17a` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_17a`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 667-673

```tablegen
class XForm_18<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> FRT;
  bits<5> FRA;
  bits<5> FRB;
```
- **EN**: Declares a backend-facing type `XForm_18` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_18`，并勾勒出周边代码会依赖的接口或状态。

### Lines 674-682

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = FRT;
  let Inst{11...15} = FRA;
  let Inst{16...20} = FRB;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 683-696

```tablegen
class XForm_19<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern> 
  : XForm_18<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
  let FRA = 0;
}

class XForm_20<bits<6> opcode, bits<6> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> FRT;
  bits<5> FRA;
  bits<5> FRB;
  bits<4> tttt;
```
- **EN**: Declares a backend-facing type `XForm_19`, `XForm_20` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_19`, `XForm_20`，并勾勒出周边代码会依赖的接口或状态。

### Lines 697-703

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = FRT;
  let Inst{11...15} = FRA;
  let Inst{16...20} = FRB;
  let Inst{21...24} = tttt;
  let Inst{25...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 704-710

```tablegen
  let Inst{31}    = 0;
}

class XForm_24<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern> 
  : I<opcode, OOL, IOL, asmstr, itin> {
  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `XForm_24` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_24`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 711-717

```tablegen
  let Inst{6...10}  = 31;
  let Inst{11...15} = 0;
  let Inst{16...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 718-724

```tablegen
class XForm_24_sync<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
               string asmstr, InstrItinClass itin, list<dag> pattern> 
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<2> L;

  let Pattern = pattern;
  let Inst{6...8}   = 0;
```
- **EN**: Declares a backend-facing type `XForm_24_sync` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_24_sync`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 725-731

```tablegen
  let Inst{9...10}  = L;
  let Inst{11...15} = 0;
  let Inst{16...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 732-738

```tablegen
class XForm_IMM2_IMM2<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
               string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<2> L;
  bits<2> PL;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `XForm_IMM2_IMM2` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_IMM2_IMM2`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 739-747

```tablegen
  let Inst{6...8}   = 0;
  let Inst{9...10}  = L;
  let Inst{11...13} = 0;
  let Inst{14...15} = PL;
  let Inst{16...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 748-754

```tablegen
class XForm_IMM3_IMM2<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
               string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> L;
  bits<2> SC;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `XForm_IMM3_IMM2` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_IMM3_IMM2`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 755-763

```tablegen
  let Inst{6...7}   = 0;
  let Inst{8...10}  = L;
  let Inst{11...13} = 0;
  let Inst{14...15} = SC;
  let Inst{16...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 764-774

```tablegen
class XForm_24_eieio<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
               string asmstr, InstrItinClass itin, list<dag> pattern>
  : XForm_24_sync<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
  let L = 0;
}

class XForm_25<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
}
```
- **EN**: Declares a backend-facing type `XForm_24_eieio`, `XForm_25` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_24_eieio`, `XForm_25`，并勾勒出周边代码会依赖的接口或状态。

### Lines 775-783

```tablegen
class XForm_25_memOp<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                    string asmstr, InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo_memOp<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
}

// [PO RT /// RB XO RC]
class XForm_26<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XForm_25_memOp`, `XForm_26` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_25_memOp`, `XForm_26`，并勾勒出周边代码会依赖的接口或状态。

### Lines 784-791

```tablegen
  let RA = 0;
}

class XForm_28_memOp<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                    string asmstr, InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo_memOp<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
}
```
- **EN**: Declares a backend-facing type `XForm_28_memOp` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_28_memOp`，并勾勒出周边代码会依赖的接口或状态。

### Lines 792-798

```tablegen
class XForm_28<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
}

// This is used for MFFS, MTFSB0, MTFSB1.  42 is arbitrary; this series of
// numbers presumably relates to some document, but I haven't found it.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is used for MFFS, MTFSB0, MTFSB1.  42 is arbitrary; this series of". Notable symbols in this range include `XForm_28`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is used for MFFS, MTFSB0, MTFSB1.  42 is arbitrary; this series of”。 该区间中较显眼的符号包括 `XForm_28`。

### Lines 799-805

```tablegen
class XForm_42<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
  let Pattern = pattern;

  bit RC = 0;    // set by isRecordForm
```
- **EN**: Declares a backend-facing type `XForm_42` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_42`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 806-813

```tablegen
  let Inst{6...10}  = RST;
  let Inst{11...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = RC;
}
class XForm_43<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XForm_43` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_43`，并勾勒出周边代码会依赖的接口或状态。

### Lines 814-820

```tablegen
  let Pattern = pattern;
  bits<5> FM;

  bit RC = 0;    // set by isRecordForm

  let Inst{6...10}  = FM;
  let Inst{11...20} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 821-830

```tablegen
  let Inst{21...30} = xo;
  let Inst{31}    = RC;
}

class XForm_44<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<3> BFA;
```
- **EN**: Declares a backend-facing type `XForm_44` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_44`，并勾勒出周边代码会依赖的接口或状态。

### Lines 831-838

```tablegen
  let Inst{6...10}  = RT;
  let Inst{11...13} = BFA;
  let Inst{14...15} = 0;
  let Inst{16...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 839-845

```tablegen
class XForm_45<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<2> L;

  let Inst{6...10}  = RT;
```
- **EN**: Declares a backend-facing type `XForm_45` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_45`，并勾勒出周边代码会依赖的接口或状态。

### Lines 846-852

```tablegen
  let Inst{11...13} = 0;
  let Inst{14...15} = L;
  let Inst{16...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 853-860

```tablegen
class XForm_RSB5_UIMM2<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                              string asmstr, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, NoItinerary> {

  bits<5> RS;
  bits<5> RB;
  bits<2> RIC;
```
- **EN**: Declares a backend-facing type `XForm_RSB5_UIMM2` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `XForm_RSB5_UIMM2`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 861-867

```tablegen
  let Pattern = pattern;

  let Inst{6...10} = RS;
  let Inst{11} = 0;
  let Inst{12...13} = RIC;
  let Inst{14...15} = 0;
  let Inst{16...20} = RB;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 868-878

```tablegen
  let Inst{21...30} = xo;
  let Inst{31} = 0;
}

class XForm_RSB5_UIMM2_2UIMM1<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                              string asmstr, list<dag> pattern>
    : XForm_RSB5_UIMM2<opcode, xo, OOL, IOL, asmstr, pattern> {

  bits<1> PRS;
  bits<1> R;
```
- **EN**: Declares a backend-facing type `XForm_RSB5_UIMM2_2UIMM1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_RSB5_UIMM2_2UIMM1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 879-886

```tablegen
  let Inst{14} = PRS;
  let Inst{15} = R;
}

class X_FRT5_XO2_XO3_XO10<bits<6> opcode, bits<2> xo1, bits<3> xo2, bits<10> xo,
                         dag OOL, dag IOL, string asmstr, InstrItinClass itin,
                         list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `X_FRT5_XO2_XO3_XO10` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_FRT5_XO2_XO3_XO10`，并勾勒出周边代码会依赖的接口或状态。

### Lines 887-893

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RST;
  let Inst{11...12} = xo1;
  let Inst{13...15} = xo2;
  let Inst{16...20} = 0;
  let Inst{21...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 894-900

```tablegen
  let Inst{31}    = 0;
}

class X_FRT5_XO2_XO3_FRB5_XO10<bits<6> opcode, bits<2> xo1, bits<3> xo2,
                              bits<10> xo, dag OOL, dag IOL, string asmstr,
                              InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `X_FRT5_XO2_XO3_FRB5_XO10` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_FRT5_XO2_XO3_FRB5_XO10`，并勾勒出周边代码会依赖的接口或状态。

### Lines 901-907

```tablegen
  let Pattern = pattern;
  bits<5> FRB;

  let Inst{6...10}  = RST;
  let Inst{11...12} = xo1;
  let Inst{13...15} = xo2;
  let Inst{16...20} = FRB;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 908-915

```tablegen
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}

class X_FRT5_XO2_XO3_DRM3_XO10<bits<6> opcode, bits<2> xo1, bits<3> xo2,
                              bits<10> xo, dag OOL, dag IOL, string asmstr,
                              InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `X_FRT5_XO2_XO3_DRM3_XO10` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_FRT5_XO2_XO3_DRM3_XO10`，并勾勒出周边代码会依赖的接口或状态。

### Lines 916-922

```tablegen
  let Pattern = pattern;
  bits<3> DRM;

  let Inst{6...10}  = RST;
  let Inst{11...12} = xo1;
  let Inst{13...15} = xo2;
  let Inst{16...17} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 923-931

```tablegen
  let Inst{18...20} = DRM;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}

class X_FRT5_XO2_XO3_RM2_X10<bits<6> opcode, bits<2> xo1, bits<3> xo2,
                            bits<10> xo, dag OOL, dag IOL, string asmstr,
                            InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `X_FRT5_XO2_XO3_RM2_X10` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_FRT5_XO2_XO3_RM2_X10`，并勾勒出周边代码会依赖的接口或状态。

### Lines 932-938

```tablegen
  let Pattern = pattern;
  bits<2> RM;

  let Inst{6...10}  = RST;
  let Inst{11...12} = xo1;
  let Inst{13...15} = xo2;
  let Inst{16...18} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 939-947

```tablegen
  let Inst{19...20} = RM;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}


class XForm_0<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XForm_0` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_0`，并勾勒出周边代码会依赖的接口或状态。

### Lines 948-955

```tablegen
  let RST = 0;
  let RA = 0;
  let RB = 0;
}

class XForm_16b<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XForm_16b` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_16b`，并勾勒出周边代码会依赖的接口或状态。

### Lines 956-966

```tablegen
  let RST = 0;
  let RA = 0;
}

class XForm_htm0<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                 string asmstr, InstrItinClass itin>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bit R;

  bit RC = 1;
```
- **EN**: Declares a backend-facing type `XForm_htm0` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_htm0`，并勾勒出周边代码会依赖的接口或状态。

### Lines 967-973

```tablegen
  let Inst{6...9}   = 0;
  let Inst{10}    = R;
  let Inst{11...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = RC;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 974-980

```tablegen
class XForm_htm1<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                 string asmstr, InstrItinClass itin>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bit A;

  bit RC = 1;
```
- **EN**: Declares a backend-facing type `XForm_htm1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_htm1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 981-993

```tablegen
  let Inst{6}     = A;
  let Inst{7...20}  = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = RC;
}

class XForm_htm2<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bit L;

  bit RC = 0;    // set by isRecordForm
```
- **EN**: Declares a backend-facing type `XForm_htm2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_htm2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 994-1000

```tablegen
  let Inst{7...9}   = 0;
  let Inst{10}    = L;
  let Inst{11...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = RC;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1001-1007

```tablegen
class XForm_htm3<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;

  bit RC = 0;
```
- **EN**: Declares a backend-facing type `XForm_htm3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_htm3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1008-1014

```tablegen
  let Inst{6...8}   = BF;
  let Inst{9...20}  = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = RC;
}

// [PO RT RA RB XO /]
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1015-1022

```tablegen
class X_BF3_L1_RS5_RS5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                       string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bits<1> L;
  bits<5> RA;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `X_BF3_L1_RS5_RS5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_BF3_L1_RS5_RS5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1023-1029

```tablegen
  let Pattern = pattern;

  let Inst{6...8}   = BF;
  let Inst{9}     = 0;
  let Inst{10}    = L;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1030-1041

```tablegen
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}

// Same as XForm_17 but with GPR's and new naming convention
class X_BF3_RS5_RS5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                    string asmstr, InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bits<5> RA;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `X_BF3_RS5_RS5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_BF3_RS5_RS5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1042-1048

```tablegen
  let Pattern = pattern;

  let Inst{6...8}   = BF;
  let Inst{9...10}  = 0;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1049-1055

```tablegen
  let Inst{31}    = 0;
}

// e.g. [PO VRT XO VRB XO /] or [PO VRT XO VRB XO RO]
class X_RD5_XO5_RS5<bits<6> opcode, bits<5> xo2, bits<10> xo, dag OOL, dag IOL,
                    string asmstr, InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `X_RD5_XO5_RS5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_RD5_XO5_RS5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1056-1065

```tablegen
  let RA = xo2;
}

class X_BF3_DCMX7_RS5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                      string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bits<7> DCMX;
  bits<5> VB;
```
- **EN**: Declares a backend-facing type `X_BF3_DCMX7_RS5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_BF3_DCMX7_RS5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1066-1074

```tablegen
  let Pattern = pattern;

  let Inst{6...8}  = BF;
  let Inst{9...15} = DCMX;
  let Inst{16...20} = VB;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1075-1082

```tablegen
class X_RD6_IMM8<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                 string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<8> IMM8;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `X_RD6_IMM8` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `X_RD6_IMM8`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 1083-1091

```tablegen
  let Inst{6...10}  = XT{4...0};
  let Inst{11...12} = 0;
  let Inst{13...20} = IMM8;
  let Inst{21...30} = xo;
  let Inst{31}    = XT{5};
}

// XForm_base_r3xo for instructions such as P9 atomics where we don't want
// to specify an SDAG pattern for matching.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。

### Lines 1092-1099

```tablegen
class X_RD5_RS5_IM5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                    string asmstr, InstrItinClass itin>
  : XForm_base_r3xo_memOp<opcode, xo, OOL, IOL, asmstr, itin, []> {
}

class X_BF3<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
            InstrItinClass itin>
  : XForm_17<opcode, xo, OOL, IOL, asmstr, itin> {
```
- **EN**: Declares a backend-facing type `X_RD5_RS5_IM5`, `X_BF3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_RD5_RS5_IM5`, `X_BF3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1100-1107

```tablegen
  let RA = 0;
  let RB = 0;
}

// [PO /// L RA RB XO /]
class X_L1_RS5_RS5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                   string asmstr, InstrItinClass itin, list<dag> pattern>
  : XForm_16<opcode, xo, OOL, IOL, asmstr, itin> {
```
- **EN**: Declares a backend-facing type `X_L1_RS5_RS5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_L1_RS5_RS5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1108-1115

```tablegen
  let BF = 0;
  let Pattern = pattern;

  bit RC = 0;
  let Inst{31} = RC;
}

// XX*-Form (VSX)
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 1116-1122

```tablegen
class XX1Form<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr, 
              InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<5> RA;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `XX1Form` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX1Form`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1123-1131

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = XT{4...0};
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
  let Inst{31}    = XT{5};
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1132-1138

```tablegen
class XX1Form_memOp<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                    string asmstr, InstrItinClass itin, list<dag> pattern>
  : XX1Form<opcode, xo, OOL, IOL, asmstr, itin, pattern>, XFormMemOp;

class XX1_RS6_RD5_XO<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                     string asmstr, InstrItinClass itin, list<dag> pattern>
  : XX1Form<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XX1Form_memOp`, `XX1_RS6_RD5_XO` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX1Form_memOp`, `XX1_RS6_RD5_XO`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1139-1147

```tablegen
  let RB = 0;
}

class XX2Form<bits<6> opcode, bits<9> xo, dag OOL, dag IOL, string asmstr, 
              InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX2Form` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX2Form`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1148-1154

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = XT{4...0};
  let Inst{11...15} = 0;
  let Inst{16...20} = XB{4...0};
  let Inst{21...29} = xo;
  let Inst{30}    = XB{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1155-1163

```tablegen
  let Inst{31}    = XT{5};
}

class XX2Form_1<bits<6> opcode, bits<9> xo, dag OOL, dag IOL, string asmstr, 
                InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> CR;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX2Form_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX2Form_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1164-1170

```tablegen
  let Pattern = pattern;

  let Inst{6...8}   = CR;
  let Inst{9...15}  = 0;
  let Inst{16...20} = XB{4...0};
  let Inst{21...29} = xo;
  let Inst{30}    = XB{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1171-1180

```tablegen
  let Inst{31}    = 0;
}

class XX2Form_2<bits<6> opcode, bits<9> xo, dag OOL, dag IOL, string asmstr, 
                InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<6> XB;
  bits<2> D;
```
- **EN**: Declares a backend-facing type `XX2Form_2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX2Form_2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1181-1187

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = XT{4...0};
  let Inst{11...13} = 0;
  let Inst{14...15} = D;
  let Inst{16...20} = XB{4...0};
  let Inst{21...29} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1188-1198

```tablegen
  let Inst{30}    = XB{5};
  let Inst{31}    = XT{5};
}

class XX2_RD6_UIM5_RS6<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
                       string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<6> XB;
  bits<5> UIM5;
```
- **EN**: Declares a backend-facing type `XX2_RD6_UIM5_RS6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX2_RD6_UIM5_RS6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1199-1205

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = XT{4...0};
  let Inst{11...15} = UIM5;
  let Inst{16...20} = XB{4...0};
  let Inst{21...29} = xo;
  let Inst{30}    = XB{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1206-1215

```tablegen
  let Inst{31}    = XT{5};
}

// [PO T XO B XO BX /]
class XX2_RD5_XO5_RS6<bits<6> opcode, bits<5> xo2, bits<9> xo, dag OOL, dag IOL,
                       string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX2_RD5_XO5_RS6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX2_RD5_XO5_RS6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1216-1222

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RT;
  let Inst{11...15} = xo2;
  let Inst{16...20} = XB{4...0};
  let Inst{21...29} = xo;
  let Inst{30}    = XB{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1223-1232

```tablegen
  let Inst{31}    = 0;
}

// [PO T XO B XO BX TX]
class XX2_RD6_XO5_RS6<bits<6> opcode, bits<5> xo2, bits<9> xo, dag OOL, dag IOL,
                      string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX2_RD6_XO5_RS6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX2_RD6_XO5_RS6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1233-1239

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = XT{4...0};
  let Inst{11...15} = xo2;
  let Inst{16...20} = XB{4...0};
  let Inst{21...29} = xo;
  let Inst{30}    = XB{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1240-1249

```tablegen
  let Inst{31}    = XT{5};
}

class XX2_BF3_DCMX7_RS6<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
                      string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bits<7> DCMX;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX2_BF3_DCMX7_RS6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX2_BF3_DCMX7_RS6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1250-1256

```tablegen
  let Pattern = pattern;

  let Inst{6...8}  = BF;
  let Inst{9...15} = DCMX;
  let Inst{16...20} = XB{4...0};
  let Inst{21...29} = xo;
  let Inst{30}    = XB{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1257-1267

```tablegen
  let Inst{31}    = 0;
}

class XX2_RD6_DCMX7_RS6<bits<6> opcode, bits<4> xo1, bits<3> xo2,
                        dag OOL, dag IOL, string asmstr, InstrItinClass itin,
                        list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<7> DCMX;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX2_RD6_DCMX7_RS6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX2_RD6_DCMX7_RS6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1268-1274

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = XT{4...0};
  let Inst{11...15} = DCMX{4...0};
  let Inst{16...20} = XB{4...0};
  let Inst{21...24} = xo1;
  let Inst{25}    = DCMX{6};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1275-1287

```tablegen
  let Inst{26...28} = xo2;
  let Inst{29}    = DCMX{5};
  let Inst{30}    = XB{5};
  let Inst{31}    = XT{5};
}

class XForm_XD6_RA5_RB5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                        string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin>, MemriOp {
  bits<5> RA;
  bits<6> D;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `XForm_XD6_RA5_RB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_XD6_RA5_RB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1288-1296

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = D{4...0};  // D
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
  let Inst{31}    = D{5};    // DX
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1297-1303

```tablegen
class XForm_BF3_UIM6_FRB5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                          string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bits<6> UIM;
  bits<5> FRB;
```
- **EN**: Declares a backend-facing type `XForm_BF3_UIM6_FRB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_BF3_UIM6_FRB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1304-1310

```tablegen
  let Pattern = pattern;

  let Inst{6...8}   = BF;
  let Inst{9}     = 0;
  let Inst{10...15} = UIM;
  let Inst{16...20} = FRB;
  let Inst{21...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1311-1320

```tablegen
  let Inst{31}    = 0;
}

class XForm_SP2_FRTB5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                  list<dag> pattern, InstrItinClass itin>
    : I<opcode, OOL, IOL, asmstr, itin> {
  bits<2> SP;
  bits<5> FRT;
  bits<5> FRB;
```
- **EN**: Declares a backend-facing type `XForm_SP2_FRTB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_SP2_FRTB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1321-1327

```tablegen
  let Pattern = pattern;

  bit RC = 0; // set by isRecordForm

  let Inst{6...10} = FRT;
  let Inst{11...12} = SP;
  let Inst{13...15} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1328-1339

```tablegen
  let Inst{16...20} = FRB;
  let Inst{21...30} = xo;
  let Inst{31} = RC;
}

class XForm_S1_FRTB5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                 string asmstr, list<dag> pattern, InstrItinClass itin>
    : I<opcode, OOL, IOL, asmstr, itin> {
  bit S;
  bits<5> FRT;
  bits<5> FRB;
```
- **EN**: Declares a backend-facing type `XForm_S1_FRTB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_S1_FRTB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1340-1346

```tablegen
  let Pattern = pattern;

  bit RC = 0; // set by isRecordForm

  let Inst{6...10} = FRT;
  let Inst{11} = S;
  let Inst{12...15} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1347-1358

```tablegen
  let Inst{16...20} = FRB;
  let Inst{21...30} = xo;
  let Inst{31} = RC;
}

class XX3Form<bits<6> opcode, bits<8> xo, dag OOL, dag IOL, string asmstr, 
              InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<6> XA;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX3Form` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX3Form`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1359-1365

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = XT{4...0};
  let Inst{11...15} = XA{4...0};
  let Inst{16...20} = XB{4...0};
  let Inst{21...28} = xo;
  let Inst{29}    = XA{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1366-1372

```tablegen
  let Inst{30}    = XB{5};
  let Inst{31}    = XT{5};
}

class XX3Form_SameOp<bits<6> opcode, bits<8> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : XX3Form<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XX3Form_SameOp` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX3Form_SameOp`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1373-1383

```tablegen
  let XA = XT;
  let XB = XT;
}

class XX3Form_1<bits<6> opcode, bits<8> xo, dag OOL, dag IOL, string asmstr, 
                InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> CR;
  bits<6> XA;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX3Form_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX3Form_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1384-1390

```tablegen
  let Pattern = pattern;

  let Inst{6...8}   = CR;
  let Inst{9...10}  = 0;
  let Inst{11...15} = XA{4...0};
  let Inst{16...20} = XB{4...0};
  let Inst{21...28} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1391-1403

```tablegen
  let Inst{29}    = XA{5};
  let Inst{30}    = XB{5};
  let Inst{31}    = 0;
}

class XX3Form_2<bits<6> opcode, bits<5> xo, dag OOL, dag IOL, string asmstr, 
                InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<6> XA;
  bits<6> XB;
  bits<2> D;
```
- **EN**: Declares a backend-facing type `XX3Form_2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX3Form_2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1404-1410

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = XT{4...0};
  let Inst{11...15} = XA{4...0};
  let Inst{16...20} = XB{4...0};
  let Inst{21}    = 0;
  let Inst{22...23} = D;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1411-1423

```tablegen
  let Inst{24...28} = xo;
  let Inst{29}    = XA{5};
  let Inst{30}    = XB{5};
  let Inst{31}    = XT{5};
}

class XX3Form_Rc<bits<6> opcode, bits<7> xo, dag OOL, dag IOL, string asmstr, 
              InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<6> XA;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX3Form_Rc` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX3Form_Rc`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1424-1430

```tablegen
  let Pattern = pattern;

  bit RC = 0;    // set by isRecordForm

  let Inst{6...10}  = XT{4...0};
  let Inst{11...15} = XA{4...0};
  let Inst{16...20} = XB{4...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1431-1437

```tablegen
  let Inst{21}    = RC;
  let Inst{22...28} = xo;
  let Inst{29}    = XA{5};
  let Inst{30}    = XB{5};
  let Inst{31}    = XT{5};
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1438-1445

```tablegen
class XX4Form<bits<6> opcode, bits<2> xo, dag OOL, dag IOL, string asmstr, 
              InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<6> XA;
  bits<6> XB;
  bits<6> XC;
```
- **EN**: Declares a backend-facing type `XX4Form` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX4Form`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1446-1452

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = XT{4...0};
  let Inst{11...15} = XA{4...0};
  let Inst{16...20} = XB{4...0};
  let Inst{21...25} = XC{4...0};
  let Inst{26...27} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1453-1459

```tablegen
  let Inst{28}    = XC{5};
  let Inst{29}    = XA{5};
  let Inst{30}    = XB{5};
  let Inst{31}    = XT{5};
}

// DCB_Form - Form X instruction, used for dcb* instructions.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。

### Lines 1460-1467

```tablegen
class DCB_Form<bits<10> xo, bits<5> immfield, dag OOL, dag IOL, string asmstr, 
                      InstrItinClass itin, list<dag> pattern>
  : I<31, OOL, IOL, asmstr, itin> {
  bits<5> RA;
  bits<5> RB;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `DCB_Form` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `DCB_Form`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 1468-1474

```tablegen
  let Inst{6...10}  = immfield;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1475-1481

```tablegen
class DCB_Form_hint<bits<10> xo, dag OOL, dag IOL, string asmstr,
                    InstrItinClass itin, list<dag> pattern>
  : I<31, OOL, IOL, asmstr, itin> {
  bits<5> TH;
  bits<5> RA;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `DCB_Form_hint` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DCB_Form_hint`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1482-1491

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = TH;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}

// DSS_Form - Form X instruction, used for altivec dss* instructions.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step. These declarations feed generated pattern-matching logic.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。 这些声明会进入生成式模式匹配逻辑。

### Lines 1492-1498

```tablegen
class DSS_Form<bits<1> T, bits<10> xo, dag OOL, dag IOL, string asmstr,
                      InstrItinClass itin, list<dag> pattern>
  : I<31, OOL, IOL, asmstr, itin> {
  bits<2> STRM;
  bits<5> RA;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `DSS_Form` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DSS_Form`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1499-1505

```tablegen
  let Pattern = pattern;

  let Inst{6}     = T;
  let Inst{7...8}   = 0;
  let Inst{9...10}  = STRM;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1506-1517

```tablegen
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}

// 1.7.7 XL-Form
class XLForm_1<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> CRD;
  bits<5> CRA;
  bits<5> CRB;
  
```
- **EN**: Declares a backend-facing type `XLForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1518-1527

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = CRD;
  let Inst{11...15} = CRA;
  let Inst{16...20} = CRB;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}

// XL-Form for unary alias for CRNOR (CRNOT)
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 1528-1536

```tablegen
class XLForm_1s<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin, list<dag> pattern>
    : XLForm_1<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
  let CRB = CRA;
}

class XLForm_1_np<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                  InstrItinClass itin, list<dag> pattern>
  : XLForm_1<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XLForm_1s`, `XLForm_1_np` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_1s`, `XLForm_1_np`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1537-1547

```tablegen
  let CRD = 0;
  let CRA = 0;
  let CRB = 0;
}

class XLForm_1_gen<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                   InstrItinClass itin, list<dag> pattern>
  : XLForm_1<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
  bits<5> RT;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `XLForm_1_gen` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_1_gen`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1548-1557

```tablegen
  let CRD = RT;
  let CRA = 0;
  let CRB = RB;
}

class XLForm_1_ext<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> CRD;
  
```
- **EN**: Declares a backend-facing type `XLForm_1_ext` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_1_ext`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1558-1566

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = CRD;
  let Inst{11...15} = CRD;
  let Inst{16...20} = CRD;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1567-1573

```tablegen
class XLForm_2<bits<6> opcode, bits<10> xo, bit lk, dag OOL, dag IOL, string asmstr, 
               InstrItinClass itin, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> BO;
  bits<5> BI;
  bits<2> BH;
  
```
- **EN**: Declares a backend-facing type `XLForm_2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1574-1580

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = BO;
  let Inst{11...15} = BI;
  let Inst{16...18} = 0;
  let Inst{19...20} = BH;
  let Inst{21...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1581-1589

```tablegen
  let Inst{31}    = lk;
}

class XLForm_2_br<bits<6> opcode, bits<10> xo, bit lk,
                  dag OOL, dag IOL, string asmstr, InstrItinClass itin, list<dag> pattern>
  : XLForm_2<opcode, xo, lk, OOL, IOL, asmstr, itin, pattern> {
  bits<7> BIBO;  // 2 bits of BI and 5 bits of BO.
  bits<3>  CR;
  
```
- **EN**: Declares a backend-facing type `XLForm_2_br` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_2_br`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1590-1598

```tablegen
  let BO = BIBO{4...0};
  let BI{0...1} = BIBO{5...6};
  let BI{2...4} = CR{0...2};
  let BH = 0;
}

class XLForm_2_br2<bits<6> opcode, bits<10> xo, bits<5> bo, bit lk,
                   dag OOL, dag IOL, string asmstr, InstrItinClass itin, list<dag> pattern>
  : XLForm_2<opcode, xo, lk, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XLForm_2_br2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_2_br2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1599-1605

```tablegen
  let BO = bo;
  let BH = 0;
}

class XLForm_2_ext<bits<6> opcode, bits<10> xo, bits<5> bo,  bits<5> bi, bit lk,
                  dag OOL, dag IOL, string asmstr, InstrItinClass itin, list<dag> pattern>
  : XLForm_2<opcode, xo, lk, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XLForm_2_ext` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_2_ext`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1606-1616

```tablegen
  let BO = bo;
  let BI = bi;
  let BH = 0;
}

class XLForm_3<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bits<3> BFA;
  
```
- **EN**: Declares a backend-facing type `XLForm_3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1617-1625

```tablegen
  let Inst{6...8}   = BF;
  let Inst{9...10}  = 0;
  let Inst{11...13} = BFA;
  let Inst{14...15} = 0;
  let Inst{16...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1626-1634

```tablegen
class XLForm_4<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bit W;
  bits<4> U;
  
  bit RC = 0;
  
```
- **EN**: Declares a backend-facing type `XLForm_4` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_4`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1635-1641

```tablegen
  let Inst{6...8}   = BF;
  let Inst{9...10}  = 0;
  let Inst{11...14} = 0;
  let Inst{15}    = W;
  let Inst{16...19} = U;
  let Inst{20}    = 0;
  let Inst{21...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1642-1649

```tablegen
  let Inst{31}    = RC;
}

class XLForm_S<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, itin> {
  bits<1> S;
  
```
- **EN**: Declares a backend-facing type `XLForm_S` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_S`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1650-1657

```tablegen
  let Pattern = pattern;
  
  let Inst{6...19}  = 0;
  let Inst{20}    = S;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1658-1670

```tablegen
class XLForm_2_and_DSForm_1<bits<6> opcode1, bits<10> xo1, bit lk,
                            bits<6> opcode2, bits<2> xo2,
                            dag OOL, dag IOL, string asmstr,
                            InstrItinClass itin, list<dag> pattern>
        : I2<opcode1, opcode2, OOL, IOL, asmstr, itin> {
  bits<5> BO;
  bits<5> BI;
  bits<2> BH;

  bits<5>  RST;
  bits<5>  RA;
  bits<14> D;
```
- **EN**: Declares a backend-facing type `XLForm_2_and_DSForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_2_and_DSForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1671-1677

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = BO;
  let Inst{11...15} = BI;
  let Inst{16...18} = 0;
  let Inst{19...20} = BH;
  let Inst{21...30} = xo1;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1678-1685

```tablegen
  let Inst{31}    = lk;

  let Inst{38...42} = RST;
  let Inst{43...47} = RA;
  let Inst{48...61} = D;
  let Inst{62...63} = xo2;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1686-1692

```tablegen
class XLForm_2_ext_and_DSForm_1<bits<6> opcode1, bits<10> xo1,
                                bits<5> bo, bits<5> bi, bit lk,
                                bits<6> opcode2, bits<2> xo2,
                                dag OOL, dag IOL, string asmstr,
                                InstrItinClass itin, list<dag> pattern>
  : XLForm_2_and_DSForm_1<opcode1, xo1, lk, opcode2, xo2,
                          OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XLForm_2_ext_and_DSForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_2_ext_and_DSForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1693-1707

```tablegen
  let BO = bo;
  let BI = bi;
  let BH = 0;
}

class XLForm_2_ext_and_DForm_1<bits<6> opcode1, bits<10> xo1, bits<5> bo,
                               bits<5> bi, bit lk, bits<6> opcode2, dag OOL,
                               dag IOL, string asmstr, InstrItinClass itin,
                               list<dag> pattern>
  : I2<opcode1, opcode2, OOL, IOL, asmstr, itin> {

  bits<5>  RST;
  bits<5>  RA;
  bits<16> D;
```
- **EN**: Declares a backend-facing type `XLForm_2_ext_and_DForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XLForm_2_ext_and_DForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1708-1714

```tablegen
  let Pattern = pattern;

  let Inst{6...10} = bo;
  let Inst{11...15} = bi;
  let Inst{16...18} = 0;
  let Inst{19...20} = 0;  // Unused (BH)
  let Inst{21...30} = xo1;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 1715-1722

```tablegen
  let Inst{31} = lk;

  let Inst{38...42} = RST;
  let Inst{43...47} = RA;
  let Inst{48...63} = D;
}

// 1.7.8 XFX-Form
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1723-1729

```tablegen
class XFXForm_1<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5>  RST;
  bits<10> SPR;

  let Inst{6...10}  = RST;
```
- **EN**: Declares a backend-facing type `XFXForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XFXForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1730-1736

```tablegen
  let Inst{11}    = SPR{4};
  let Inst{12}    = SPR{3};
  let Inst{13}    = SPR{2};
  let Inst{14}    = SPR{1};
  let Inst{15}    = SPR{0};
  let Inst{16}    = SPR{9};
  let Inst{17}    = SPR{8};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1737-1743

```tablegen
  let Inst{18}    = SPR{7};
  let Inst{19}    = SPR{6};
  let Inst{20}    = SPR{5};
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1744-1754

```tablegen
class XFXForm_1_ext<bits<6> opcode, bits<10> xo, bits<10> spr, 
                   dag OOL, dag IOL, string asmstr, InstrItinClass itin> 
  : XFXForm_1<opcode, xo, OOL, IOL, asmstr, itin> {
  let SPR = spr;
}

class XFXForm_3<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5>  RT;
   
```
- **EN**: Declares a backend-facing type `XFXForm_1_ext`, `XFXForm_3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XFXForm_1_ext`, `XFXForm_3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1755-1765

```tablegen
  let Inst{6...10}  = RT;
  let Inst{11...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}

class XFXForm_3p<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                 InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5>  RT;
  bits<10> imm;
```
- **EN**: Declares a backend-facing type `XFXForm_3p` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XFXForm_3p`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1766-1773

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RT;
  let Inst{11...20} = imm;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1774-1780

```tablegen
class XFXForm_5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<8>  FXM;
  bits<5>  RST;

  let Inst{6...10}  = RST;
```
- **EN**: Declares a backend-facing type `XFXForm_5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XFXForm_5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1781-1787

```tablegen
  let Inst{11}    = 0;
  let Inst{12...19} = FXM;
  let Inst{20}    = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1788-1794

```tablegen
class XFXForm_5a<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                 InstrItinClass itin>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5>  RST;
  bits<8>  FXM;

  let Inst{6...10}  = RST;
```
- **EN**: Declares a backend-facing type `XFXForm_5a` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XFXForm_5a`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1795-1804

```tablegen
  let Inst{11}    = 1;
  let Inst{12...19} = FXM;
  let Inst{20}    = 0;
  let Inst{21...30} = xo;
  let Inst{31}    = 0;
}

// XFL-Form - MTFSF
// This is probably 1.7.9, but I don't have the reference that uses this
// numbering scheme...
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XFL-Form - MTFSF".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XFL-Form - MTFSF”。

### Lines 1805-1811

```tablegen
class XFLForm<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr, 
              InstrItinClass itin, list<dag>pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<8> FM;
  bits<5> RT;

  bit RC = 0;    // set by isRecordForm
```
- **EN**: Declares a backend-facing type `XFLForm` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XFLForm`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1812-1818

```tablegen
  let Pattern = pattern;

  let Inst{6} = 0;
  let Inst{7...14}  = FM;
  let Inst{15} = 0;
  let Inst{16...20} = RT;
  let Inst{21...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1819-1830

```tablegen
  let Inst{31}    = RC;
}

class XFLForm_1<bits<6> opcode, bits<10> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin, list<dag>pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bit L;
  bits<8> FLM;
  bit W;
  bits<5> FRB;

  bit RC = 0;    // set by isRecordForm
```
- **EN**: Declares a backend-facing type `XFLForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XFLForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1831-1837

```tablegen
  let Pattern = pattern;

  let Inst{6}     = L;
  let Inst{7...14}  = FLM;
  let Inst{15}    = W;
  let Inst{16...20} = FRB;
  let Inst{21...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1838-1849

```tablegen
  let Inst{31}    = RC;
}

// 1.7.10 XS-Form - SRADI.
class XSForm_1<bits<6> opcode, bits<9> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RA;
  bits<5> RS;
  bits<6> SH;

  bit RC = 0;    // set by isRecordForm
```
- **EN**: Declares a backend-facing type `XSForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XSForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1850-1856

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RS;
  let Inst{11...15} = RA;
  let Inst{16...20} = SH{4,3,2,1,0};
  let Inst{21...29} = xo;
  let Inst{30}    = SH{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1857-1867

```tablegen
  let Inst{31}    = RC;
}

// 1.7.11 XO-Form
class XOForm_1<bits<6> opcode, bits<9> xo, bit oe, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<5> RA;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `XOForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XOForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1868-1874

```tablegen
  let Pattern = pattern;

  bit RC = 0;    // set by isRecordForm

  let Inst{6...10}  = RT;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1875-1882

```tablegen
  let Inst{21}    = oe;
  let Inst{22...30} = xo;
  let Inst{31}    = RC;  
}

class XOForm_3<bits<6> opcode, bits<9> xo, bit oe, 
               dag OOL, dag IOL, string asmstr, InstrItinClass itin, list<dag> pattern>
  : XOForm_1<opcode, xo, oe, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `XOForm_3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XOForm_3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1883-1894

```tablegen
  let RB = 0;
}

// 1.7.12 A-Form
class AForm_1<bits<6> opcode, bits<5> xo, dag OOL, dag IOL, string asmstr, 
              InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> FRT;
  bits<5> FRA;
  bits<5> FRC;
  bits<5> FRB;
```
- **EN**: Declares a backend-facing type `AForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `AForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1895-1901

```tablegen
  let Pattern = pattern;

  bit RC = 0;    // set by isRecordForm

  let Inst{6...10}  = FRT;
  let Inst{11...15} = FRA;
  let Inst{16...20} = FRB;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1902-1909

```tablegen
  let Inst{21...25} = FRC;
  let Inst{26...30} = xo;
  let Inst{31}    = RC;
}

class AForm_2<bits<6> opcode, bits<5> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
  : AForm_1<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
```
- **EN**: Declares a backend-facing type `AForm_2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `AForm_2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1910-1918

```tablegen
  let FRC = 0;
}

class AForm_3<bits<6> opcode, bits<5> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern> 
  : AForm_1<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
  let FRB = 0;
}
```
- **EN**: Declares a backend-facing type `AForm_3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `AForm_3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1919-1926

```tablegen
class AForm_4<bits<6> opcode, bits<5> xo, dag OOL, dag IOL, string asmstr, 
              InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<5> RA;
  bits<5> RB;
  bits<5> COND;
```
- **EN**: Declares a backend-facing type `AForm_4` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `AForm_4`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1927-1933

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RT;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...25} = COND;
  let Inst{26...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1934-1946

```tablegen
  let Inst{31}    = 0;
}

// 1.7.13 M-Form
class MForm_1<bits<6> opcode, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RA;
  bits<5> RS;
  bits<5> RB;
  bits<5> MB;
  bits<5> ME;
```
- **EN**: Declares a backend-facing type `MForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1947-1953

```tablegen
  let Pattern = pattern;

  bit RC = 0;    // set by isRecordForm

  let Inst{6...10}  = RS;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1954-1967

```tablegen
  let Inst{21...25} = MB;
  let Inst{26...30} = ME;
  let Inst{31}    = RC;
}

class MForm_2<bits<6> opcode, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RA;
  bits<5> RS;
  bits<5> SH;
  bits<5> MB;
  bits<5> ME;
```
- **EN**: Declares a backend-facing type `MForm_2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MForm_2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1968-1974

```tablegen
  let Pattern = pattern;

  bit RC = 0;    // set by isRecordForm

  let Inst{6...10}  = RS;
  let Inst{11...15} = RA;
  let Inst{16...20} = SH;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1975-1988

```tablegen
  let Inst{21...25} = MB;
  let Inst{26...30} = ME;
  let Inst{31}    = RC;
}

// 1.7.14 MD-Form
class MDForm_1<bits<6> opcode, bits<3> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RA;
  bits<5> RS;
  bits<6> SH;
  bits<6> MBE;
```
- **EN**: Declares a backend-facing type `MDForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MDForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1989-1995

```tablegen
  let Pattern = pattern;

  bit RC = 0;    // set by isRecordForm

  let Inst{6...10}  = RS;
  let Inst{11...15} = RA;
  let Inst{16...20} = SH{4,3,2,1,0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1996-2009

```tablegen
  let Inst{21...26} = MBE{4,3,2,1,0,5};
  let Inst{27...29} = xo;
  let Inst{30}    = SH{5};
  let Inst{31}    = RC;
}

class MDSForm_1<bits<6> opcode, bits<4> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RA;
  bits<5> RS;
  bits<5> RB;
  bits<6> MBE;
```
- **EN**: Declares a backend-facing type `MDSForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MDSForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2010-2016

```tablegen
  let Pattern = pattern;

  bit RC = 0;    // set by isRecordForm

  let Inst{6...10}  = RS;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2017-2025

```tablegen
  let Inst{21...26} = MBE{4,3,2,1,0,5};
  let Inst{27...30} = xo;
  let Inst{31}    = RC;
}


// E-1 VA-Form

// VAForm_1 - DACB ordering.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2026-2033

```tablegen
class VAForm_1<bits<6> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<5> RA;
  bits<5> RC;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `VAForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VAForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2034-2043

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = RT;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...25} = RC;
  let Inst{26...31} = xo;
}

// VAForm_1a - DABC ordering.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2044-2051

```tablegen
class VAForm_1a<bits<6> xo, dag OOL, dag IOL, string asmstr,
                InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<5> RA;
  bits<5> RB;
  bits<5> RC;
```
- **EN**: Declares a backend-facing type `VAForm_1a` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VAForm_1a`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2052-2060

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = RT;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...25} = RC;
  let Inst{26...31} = xo;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2061-2068

```tablegen
class VAForm_2<bits<6> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<5> RA;
  bits<5> RB;
  bits<4> SH;
```
- **EN**: Declares a backend-facing type `VAForm_2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VAForm_2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2069-2075

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = RT;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21}    = 0;
  let Inst{22...25} = SH;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2076-2086

```tablegen
  let Inst{26...31} = xo;
}

// E-2 VX-Form
class VXForm_1<bits<11> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VD;
  bits<5> VA;
  bits<5> VB;
  
```
- **EN**: Declares a backend-facing type `VXForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2087-2094

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = VD;
  let Inst{11...15} = VA;
  let Inst{16...20} = VB;
  let Inst{21...31} = xo;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2095-2102

```tablegen
class VXForm_setzero<bits<11> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : VXForm_1<xo, OOL, IOL, asmstr, itin, pattern> {
  let VA = VD;
  let VB = VD;
}
```
- **EN**: Declares a backend-facing type `VXForm_setzero` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_setzero`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2103-2110

```tablegen
class VXForm_2<bits<11> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VD;
  bits<5> VB;
  
  let Pattern = pattern;
  
```
- **EN**: Declares a backend-facing type `VXForm_2` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `VXForm_2`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 2111-2122

```tablegen
  let Inst{6...10}  = VD;
  let Inst{11...15} = 0;
  let Inst{16...20} = VB;
  let Inst{21...31} = xo;
}

class VXForm_3<bits<11> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VD;
  bits<5> IMM;
  
```
- **EN**: Declares a backend-facing type `VXForm_3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2123-2131

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = VD;
  let Inst{11...15} = IMM;
  let Inst{16...20} = 0;
  let Inst{21...31} = xo;
}

/// VXForm_4 - VX instructions with "VD,0,0" register fields, like mfvscr.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2132-2138

```tablegen
class VXForm_4<bits<11> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VD;
  
  let Pattern = pattern;
  
```
- **EN**: Declares a backend-facing type `VXForm_4` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `VXForm_4`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 2139-2145

```tablegen
  let Inst{6...10}  = VD;
  let Inst{11...15} = 0;
  let Inst{16...20} = 0;
  let Inst{21...31} = xo;
}

/// VXForm_5 - VX instructions with "0,0,VB" register fields, like mtvscr.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2146-2152

```tablegen
class VXForm_5<bits<11> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VB;
  
  let Pattern = pattern;
  
```
- **EN**: Declares a backend-facing type `VXForm_5` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `VXForm_5`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 2153-2159

```tablegen
  let Inst{6...10}  = 0;
  let Inst{11...15} = 0;
  let Inst{16...20} = VB;
  let Inst{21...31} = xo;
}

// e.g. [PO VRT EO VRB XO]
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2160-2167

```tablegen
class VXForm_RD5_XO5_RS5<bits<11> xo, bits<5> eo, dag OOL, dag IOL,
                         string asmstr, InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VD;
  bits<5> VB;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `VXForm_RD5_XO5_RS5` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `VXForm_RD5_XO5_RS5`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 2168-2174

```tablegen
  let Inst{6...10}  = VD;
  let Inst{11...15} = eo;
  let Inst{16...20} = VB;
  let Inst{21...31} = xo;
}

/// VXForm_CR - VX crypto instructions with "VRT, VRA, ST, SIX"
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2175-2182

```tablegen
class VXForm_CR<bits<11> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VD;
  bits<5> VA;
  bits<1> ST;
  bits<4> SIX;
  
```
- **EN**: Declares a backend-facing type `VXForm_CR` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_CR`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2183-2192

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = VD;
  let Inst{11...15} = VA;
  let Inst{16} =  ST;
  let Inst{17...20} = SIX;
  let Inst{21...31} = xo;
}

/// VXForm_BX - VX crypto instructions with "VRT, VRA, 0 - like vsbox"
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2193-2200

```tablegen
class VXForm_BX<bits<11> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VD;
  bits<5> VA;
  
  let Pattern = pattern;
  
```
- **EN**: Declares a backend-facing type `VXForm_BX` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `VXForm_BX`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 2201-2207

```tablegen
  let Inst{6...10}  = VD;
  let Inst{11...15} = VA;
  let Inst{16...20} = 0;
  let Inst{21...31} = xo;
}

// E-4 VXR-Form
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2208-2215

```tablegen
class VXRForm_1<bits<10> xo, dag OOL, dag IOL, string asmstr,
               InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VD;
  bits<5> VA;
  bits<5> VB;
  bit RC = 0;
  
```
- **EN**: Declares a backend-facing type `VXRForm_1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXRForm_1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2216-2225

```tablegen
  let Pattern = pattern;
  
  let Inst{6...10}  = VD;
  let Inst{11...15} = VA;
  let Inst{16...20} = VB;
  let Inst{21}    = RC;
  let Inst{22...31} = xo;
}

// VX-Form: [PO VRT EO VRB 1 PS XO]
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2226-2233

```tablegen
class VX_RD5_EO5_RS5_PS1_XO9<bits<5> eo, bits<9> xo,
                             dag OOL, dag IOL, string asmstr,
                             InstrItinClass itin, list<dag> pattern>
  : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VD;
  bits<5> VB;
  bit PS;
```
- **EN**: Declares a backend-facing type `VX_RD5_EO5_RS5_PS1_XO9` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VX_RD5_EO5_RS5_PS1_XO9`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2234-2240

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = VD;
  let Inst{11...15} = eo;
  let Inst{16...20} = VB;
  let Inst{21}    = 1;
  let Inst{22}    = PS;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2241-2252

```tablegen
  let Inst{23...31} = xo;
}

// VX-Form: [PO VRT VRA VRB 1 PS XO] or [PO VRT VRA VRB 1 / XO]
class VX_RD5_RSp5_PS1_XO9<bits<9> xo, dag OOL, dag IOL, string asmstr,
                          InstrItinClass itin, list<dag> pattern>
  : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VD;
  bits<5> VA;
  bits<5> VB;
  bit PS;
```
- **EN**: Declares a backend-facing type `VX_RD5_RSp5_PS1_XO9` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VX_RD5_RSp5_PS1_XO9`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2253-2259

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = VD;
  let Inst{11...15} = VA;
  let Inst{16...20} = VB;
  let Inst{21}    = 1;
  let Inst{22}    = PS;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2260-2270

```tablegen
  let Inst{23...31} = xo;
}

class Z22Form_BF3_FRA5_DCM6<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
                            string asmstr, InstrItinClass itin,
                            list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bits<5> FRA;
  bits<6> DCM;
```
- **EN**: Declares a backend-facing type `Z22Form_BF3_FRA5_DCM6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `Z22Form_BF3_FRA5_DCM6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2271-2277

```tablegen
  let Pattern = pattern;

  let Inst{6...8}   = BF;
  let Inst{9...10}  = 0;
  let Inst{11...15} = FRA;
  let Inst{16...21} = DCM;
  let Inst{22...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2278-2288

```tablegen
  let Inst{31}    = 0;
}

class Z22Form_FRTA5_SH6<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
              string asmstr, list<dag> pattern, InstrItinClass itin>
    : I<opcode, OOL, IOL, asmstr, itin> {

  bits<5> FRT;
  bits<5> FRA;
  bits<6> SH;
```
- **EN**: Declares a backend-facing type `Z22Form_FRTA5_SH6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `Z22Form_FRTA5_SH6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2289-2295

```tablegen
  let Pattern = pattern;

  bit RC = 0; // set by isRecordForm

  let Inst{6...10} = FRT;
  let Inst{11...15} = FRA;
  let Inst{16...21} = SH;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2296-2307

```tablegen
  let Inst{22...30} = xo;
  let Inst{31} = RC;
}

class Z23Form_8<bits<6> opcode, bits<8> xo, dag OOL, dag IOL, string asmstr,
              InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> VRT;
  bit R;
  bits<5> VRB;
  bits<2> idx;
```
- **EN**: Declares a backend-facing type `Z23Form_8` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `Z23Form_8`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2308-2314

```tablegen
  let Pattern = pattern;

  bit RC = 0;    // set by isRecordForm

  let Inst{6...10}  = VRT;
  let Inst{11...14} = 0;
  let Inst{15} = R;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2315-2328

```tablegen
  let Inst{16...20} = VRB;
  let Inst{21...22} = idx;
  let Inst{23...30} = xo;
  let Inst{31}    = RC;
}

class Z23Form_RTAB5_CY2<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                        string asmstr, InstrItinClass itin, list<dag> pattern>
         : I<opcode, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<5> RA;
  bits<5> RB;
  bits<2> CY;
```
- **EN**: Declares a backend-facing type `Z23Form_RTAB5_CY2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `Z23Form_RTAB5_CY2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2329-2335

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RT;
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...22} = CY;
  let Inst{23...30} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2336-2346

```tablegen
  let Inst{31} = 0;
}

class Z23Form_FRTAB5_RMC2<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                          string asmstr, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, NoItinerary> {
  bits<5> FRT;
  bits<5> FRA;
  bits<5> FRB;
  bits<2> RMC;
```
- **EN**: Declares a backend-facing type `Z23Form_FRTAB5_RMC2` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `Z23Form_FRTAB5_RMC2`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 2347-2353

```tablegen
  let Pattern = pattern;

  bit RC = 0; // set by isRecordForm

  let Inst{6...10} = FRT;
  let Inst{11...15} = FRA;
  let Inst{16...20} = FRB;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2354-2362

```tablegen
  let Inst{21...22} = RMC;
  let Inst{23...30} = xo;
  let Inst{31} = RC;
}

class Z23Form_TE5_FRTB5_RMC2<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                             string asmstr, list<dag> pattern>
    : Z23Form_FRTAB5_RMC2<opcode, xo, OOL, IOL, asmstr, pattern> {
  bits<5> TE;
```
- **EN**: Declares a backend-facing type `Z23Form_TE5_FRTB5_RMC2` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `Z23Form_TE5_FRTB5_RMC2`，并勾勒出周边代码会依赖的接口或状态。

### Lines 2363-2373

```tablegen
  let FRA = TE;
}

class Z23Form_FRTB5_R1_RMC2<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                            string asmstr, list<dag> pattern>
    : I<opcode, OOL, IOL, asmstr, NoItinerary> {
  bits<5> FRT;
  bits<1> R;
  bits<5> FRB;
  bits<2> RMC;
```
- **EN**: Declares a backend-facing type `Z23Form_FRTB5_R1_RMC2` and outlines the API or state that nearby code will rely on. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明面向后端的类型 `Z23Form_FRTB5_R1_RMC2`，并勾勒出周边代码会依赖的接口或状态。 这一段包含调度或处理器模型元数据。

### Lines 2374-2380

```tablegen
  let Pattern = pattern;

  bit RC = 0; // set by isRecordForm

  let Inst{6...10} = FRT;
  let Inst{11...14} = 0;
  let Inst{15} = R;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 2381-2386

```tablegen
  let Inst{16...20} = FRB;
  let Inst{21...22} = RMC;
  let Inst{23...30} = xo;
  let Inst{31} = RC;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2387-2393

```tablegen
//===----------------------------------------------------------------------===//
// EmitTimePseudo won't have encoding information for the [MC]CodeEmitter
// stuff
class PPCEmitTimePseudo<dag OOL, dag IOL, string asmstr, list<dag> pattern>
    : I<0, OOL, IOL, asmstr, NoItinerary> {
  let isCodeGenOnly = 1;
  let PPC64 = 0;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "EmitTimePseudo won't have encoding information for the [MC]CodeEmitter". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“EmitTimePseudo won't have encoding information for the [MC]CodeEmitter”。 这一段包含调度或处理器模型元数据。

### Lines 2394-2400

```tablegen
  let Pattern = pattern;
  let Inst{31...0} = 0;
  let hasNoSchedulingInfo = 1;
}

// Instruction that require custom insertion support
// a.k.a. ISelPseudos, however, these won't have isPseudo set
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instruction that require custom insertion support". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instruction that require custom insertion support”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2401-2408

```tablegen
class PPCCustomInserterPseudo<dag OOL, dag IOL, string asmstr,
                              list<dag> pattern>
    : PPCEmitTimePseudo<OOL, IOL, asmstr, pattern> {
  let usesCustomInserter = 1;
}

// PostRAPseudo will be expanded in expandPostRAPseudo, isPseudo flag in td
// files is set only for PostRAPseudo
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PostRAPseudo will be expanded in expandPostRAPseudo, isPseudo flag in td". Notable symbols in this range include `PPCCustomInserterPseudo`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PostRAPseudo will be expanded in expandPostRAPseudo, isPseudo flag in td”。 该区间中较显眼的符号包括 `PPCCustomInserterPseudo`。

### Lines 2409-2416

```tablegen
class PPCPostRAExpPseudo<dag OOL, dag IOL, string asmstr, list<dag> pattern>
    : PPCEmitTimePseudo<OOL, IOL, asmstr, pattern> {
  let isPseudo = 1;
}

class PseudoXFormMemOp<dag OOL, dag IOL, string asmstr, list<dag> pattern>
    : PPCPostRAExpPseudo<OOL, IOL, asmstr, pattern>, XFormMemOp;
```
- **EN**: Declares a backend-facing type `PPCPostRAExpPseudo`, `PseudoXFormMemOp` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `PPCPostRAExpPseudo`, `PseudoXFormMemOp`，并勾勒出周边代码会依赖的接口或状态。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Declarative TableGen records / 声明式 TableGen 记录
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
