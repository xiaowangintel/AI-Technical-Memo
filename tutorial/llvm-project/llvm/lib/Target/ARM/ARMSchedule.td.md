# ARMSchedule.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMSchedule.td`
- Repository: `llvm-project`
- Purpose (EN): Defines `ARMSchedule` in LLVM TableGen DSL for the ARM backend, covering target descriptions, instruction records, and code-generation metadata.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMSchedule`，涵盖目标描述、指令记录以及代码生成元数据。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```tablegen
//===-- ARMSchedule.td - ARM Scheduling Definitions --------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//===----------------------------------------------------------------------===//
// Instruction scheduling annotations for in-order and out-of-order CPUs.
// These annotations are independent of the itinerary class defined below.
// Here we define the subtarget independent read/write per-operand resources.
// The subtarget schedule definitions will then map these to the subtarget's
// resource usages.
// For example:
// The instruction cycle timings table might contain an entry for an operation
// like the following:
// Rd <- ADD Rn, Rm, <shift> Rs
//  Uops | Latency from register | Uops - resource requirements - latency
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 19-36
```tablegen
//  2    | Rn: 1 Rm: 4 Rs: 4     | uop T0, Rm, Rs - P01 - 3
//       |                       | uopc Rd, Rn, T0 -  P01 - 1
// This is telling us that the result will be available in destination register
// Rd after a minimum of three cycles after the result in Rm and Rs is available
// and one cycle after the result in Rn is available. The micro-ops can execute
// on resource P01.
// To model this, we need to express that we need to dispatch two micro-ops,
// that the resource P01 is needed and that the latency to Rn is different than
// the latency to Rm and Rs. The scheduler can decrease Rn's producer latency by
// two.
// We will do this by assigning (abstract) resources to register defs/uses.
// ARMSchedule.td:
//   def WriteALUsr : SchedWrite;
//   def ReadAdvanceALUsr : ScheRead;
//
// ARMInstrInfo.td:
//   def ADDrs : I<>, Sched<[WriteALUsr, ReadAdvanceALUsr, ReadDefault,
//                           ReadDefault]> { ...}
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 37-54
```tablegen
// ReadAdvance read resources allow us to define "pipeline by-passes" or
// shorter latencies to certain registers as needed in the example above.
// The "ReadDefault" can be omitted.
// Next, the subtarget td file assigns resources to the abstract resources
// defined here.
// ARMScheduleSubtarget.td:
//  // Resources.
//  def P01 : ProcResource<3>; // ALU unit (3 of it).
//  ...
//  // Resource usages.
//  def : WriteRes<WriteALUsr, [P01, P01]> {
//    Latency = 4; // Latency of 4.
//    NumMicroOps = 2; // Dispatch 2 micro-ops.
//    // The two instances of resource P01 are occupied for one cycle. It is one
//    // cycle because these resources happen to be pipelined.
//    ReleaseAtCycles = [1, 1];
//  }
//  def : ReadAdvance<ReadAdvanceALUsr, 3>;
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 56-61
```tablegen
//===----------------------------------------------------------------------===//
// Sched definitions for integer pipeline instructions
//
// Basic ALU operation.
def WriteALU : SchedWrite;
def ReadALU : SchedRead;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 63-67
```tablegen
// Basic ALU with shifts.
def WriteALUsi : SchedWrite; // Shift by immediate.
def WriteALUsr : SchedWrite; // Shift by register.
def WriteALUSsr : SchedWrite; // Shift by register (flag setting).
def ReadALUsr : SchedRead; // Some operands are read later.
```
- EN: Defines TableGen record `WriteALUsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteALUsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 69-72
```tablegen
// Compares.
def WriteCMP : SchedWrite;
def WriteCMPsi : SchedWrite;
def WriteCMPsr : SchedWrite;
```
- EN: Defines TableGen record `WriteCMP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteCMP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 74-79
```tablegen
// Multiplys.
def WriteMUL16   : SchedWrite; // 16-bit multiply.
def WriteMUL32   : SchedWrite; // 32-bit multiply.
def WriteMUL64Lo : SchedWrite; // 64-bit result. Low reg.
def WriteMUL64Hi : SchedWrite; // 64-bit result. High reg.
def ReadMUL  : SchedRead;
```
- EN: Defines TableGen record `WriteMUL16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteMUL16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 81-86
```tablegen
// Multiply-accumulates.
def WriteMAC16   : SchedWrite; // 16-bit mac.
def WriteMAC32   : SchedWrite; // 32-bit mac.
def WriteMAC64Lo : SchedWrite; // 64-bit mac. Low reg.
def WriteMAC64Hi : SchedWrite; // 64-bit mac. High reg.
def ReadMAC : SchedRead;
```
- EN: Defines TableGen record `WriteMAC16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteMAC16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 88-89
```tablegen
// Divisions.
def WriteDIV : SchedWrite;
```
- EN: Defines TableGen record `WriteDIV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteDIV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 91-94
```tablegen
// Loads/Stores.
def WriteLd : SchedWrite;
def WritePreLd : SchedWrite;
def WriteST : SchedWrite;
```
- EN: Defines TableGen record `WriteLd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteLd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 96-99
```tablegen
// Branches.
def WriteBr : SchedWrite;
def WriteBrL : SchedWrite;
def WriteBrTbl : SchedWrite;
```
- EN: Defines TableGen record `WriteBr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteBr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 101-102
```tablegen
// Noop.
def WriteNoop : SchedWrite;
```
- EN: Defines TableGen record `WriteNoop` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteNoop`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 104-109
```tablegen
//===----------------------------------------------------------------------===//
// Sched definitions for floating-point and neon instructions
//
// Floating point conversions
def WriteFPCVT : SchedWrite;
def WriteFPMOV : SchedWrite; // FP -> GPR and vice-versa
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 111-113
```tablegen
// ALU operations (32/64-bit)
def WriteFPALU32 : SchedWrite;
def WriteFPALU64 : SchedWrite;
```
- EN: Defines TableGen record `WriteFPALU32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteFPALU32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 115-119
```tablegen
// Multiplication
def WriteFPMUL32 : SchedWrite;
def WriteFPMUL64 : SchedWrite;
def ReadFPMUL    : SchedRead; // multiplier read
def ReadFPMAC    : SchedRead; // accumulator read
```
- EN: Defines TableGen record `WriteFPMUL32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteFPMUL32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 121-123
```tablegen
// Multiply-accumulate
def WriteFPMAC32 : SchedWrite;
def WriteFPMAC64 : SchedWrite;
```
- EN: Defines TableGen record `WriteFPMAC32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteFPMAC32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 125-127
```tablegen
// Division
def WriteFPDIV32 : SchedWrite;
def WriteFPDIV64 : SchedWrite;
```
- EN: Defines TableGen record `WriteFPDIV32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteFPDIV32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 129-131
```tablegen
// Square-root
def WriteFPSQRT32 : SchedWrite;
def WriteFPSQRT64 : SchedWrite;
```
- EN: Defines TableGen record `WriteFPSQRT32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteFPSQRT32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 133-141
```tablegen
// Vector load and stores
def WriteVLD1 : SchedWrite;
def WriteVLD2 : SchedWrite;
def WriteVLD3 : SchedWrite;
def WriteVLD4 : SchedWrite;
def WriteVST1 : SchedWrite;
def WriteVST2 : SchedWrite;
def WriteVST3 : SchedWrite;
def WriteVST4 : SchedWrite;
```
- EN: Defines TableGen record `WriteVLD1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WriteVLD1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 144-152
```tablegen
// Define TII for use in SchedVariant Predicates.
def : PredicateProlog<[{
  const ARMBaseInstrInfo *TII =
    static_cast<const ARMBaseInstrInfo*>(SchedModel->getInstrInfo());
  (void)TII;
  const ARMSubtarget *STI =
    static_cast<const ARMSubtarget*>(SchedModel->getSubtargetInfo());
  (void)STI;
}]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 154-158
```tablegen
def IsPredicated : CheckFunctionPredicateWithTII<
  "ARM_MC::isPredicated",
  "isPredicated"
>;
def IsPredicatedPred : MCSchedPredicate<IsPredicated>;
```
- EN: Defines TableGen record `IsPredicated` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsPredicated`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 160-163
```tablegen
def IsCPSRDefined : CheckFunctionPredicateWithTII<
  "ARM_MC::isCPSRDefined",
  "ARMBaseInstrInfo::isCPSRDefined"
>;
```
- EN: Defines TableGen record `IsCPSRDefined` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsCPSRDefined`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 165-165
```tablegen
def IsCPSRDefinedPred : MCSchedPredicate<IsCPSRDefined>;
```
- EN: Defines TableGen record `IsCPSRDefinedPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsCPSRDefinedPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 167-170
```tablegen
let FunctionMapper = "ARM_AM::getAM2ShiftOpc" in {
  class CheckAM2NoShift<int n> : CheckImmOperand_s<n, "ARM_AM::no_shift">;
  class CheckAM2ShiftLSL<int n> : CheckImmOperand_s<n, "ARM_AM::lsl">;
}
```
- EN: Declares reusable TableGen class `CheckAM2NoShift` for `ARMSchedule`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMSchedule` 声明可复用的 TableGen 类 `CheckAM2NoShift`，通常用于抽象共享字段、谓词或编码结构。

### Lines 172-175
```tablegen
let FunctionMapper = "ARM_AM::getAM2Op" in {
  class CheckAM2OpAdd<int n> : CheckImmOperand_s<n, "ARM_AM::add"> {}
  class CheckAM2OpSub<int n> : CheckImmOperand_s<n, "ARM_AM::sub"> {}
}
```
- EN: Declares reusable TableGen class `CheckAM2OpAdd` for `ARMSchedule`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMSchedule` 声明可复用的 TableGen 类 `CheckAM2OpAdd`，通常用于抽象共享字段、谓词或编码结构。

### Lines 177-179
```tablegen
let FunctionMapper = "ARM_AM::getAM2Offset" in {
  class CheckAM2Offset<int n, int of> : CheckImmOperand<n, of> {}
}
```
- EN: Declares reusable TableGen class `CheckAM2Offset` for `ARMSchedule`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMSchedule` 声明可复用的 TableGen 类 `CheckAM2Offset`，通常用于抽象共享字段、谓词或编码结构。

### Lines 181-183
```tablegen
def IsLDMBaseRegInList : CheckFunctionPredicate<
  "ARM_MC::isLDMBaseRegInList", "ARM_MC::isLDMBaseRegInList"
>;
```
- EN: Defines TableGen record `IsLDMBaseRegInList` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsLDMBaseRegInList`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 185-187
```tablegen
let FunctionMapper = "ARM_AM::getAM3Op" in {
  class CheckAM3OpSub<int n> : CheckImmOperand_s<n, "ARM_AM::sub"> {}
}
```
- EN: Declares reusable TableGen class `CheckAM3OpSub` for `ARMSchedule`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMSchedule` 声明可复用的 TableGen 类 `CheckAM3OpSub`，通常用于抽象共享字段、谓词或编码结构。

### Lines 189-190
```tablegen
// LDM, base reg in list
def IsLDMBaseRegInListPred : MCSchedPredicate<IsLDMBaseRegInList>;
```
- EN: Defines TableGen record `IsLDMBaseRegInListPred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IsLDMBaseRegInListPred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 192-192
```tablegen
class IsRegPCPred<int n> : MCSchedPredicate<CheckRegOperand<n, PC>>;
```
- EN: Declares reusable TableGen class `IsRegPCPred` for `ARMSchedule`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMSchedule` 声明可复用的 TableGen 类 `IsRegPCPred`，通常用于抽象共享字段、谓词或编码结构。

### Lines 194-201
```tablegen
class BranchWriteRes<int lat, int uops, list<ProcResourceKind> resl,
                     list<int> rcl, SchedWriteRes wr> :
  SchedWriteRes<!listconcat(wr.ProcResources, resl)> {
  let Latency = !add(wr.Latency, lat);
  let ReleaseAtCycles = !listconcat(wr.ReleaseAtCycles, rcl);
  let NumMicroOps = !add(wr.NumMicroOps, uops);
  SchedWriteRes BaseWr = wr;
}
```
- EN: Declares reusable TableGen class `BranchWriteRes` for `ARMSchedule`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMSchedule` 声明可复用的 TableGen 类 `BranchWriteRes`，通常用于抽象共享字段、谓词或编码结构。

### Lines 203-207
```tablegen
class CheckBranchForm<int n, BranchWriteRes br> :
  SchedWriteVariant<[
    SchedVar<IsRegPCPred<n>, [br]>,
    SchedVar<NoSchedPred,    [br.BaseWr]>
  ]>;
```
- EN: Declares reusable TableGen class `CheckBranchForm` for `ARMSchedule`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMSchedule` 声明可复用的 TableGen 类 `CheckBranchForm`，通常用于抽象共享字段、谓词或编码结构。

### Lines 209-226
```tablegen
//===----------------------------------------------------------------------===//
// Instruction Itinerary classes used for ARM
//
def IIC_iALUx      : InstrItinClass;
def IIC_iALUi      : InstrItinClass;
def IIC_iALUr      : InstrItinClass;
def IIC_iALUsi     : InstrItinClass;
def IIC_iALUsir    : InstrItinClass;
def IIC_iALUsr     : InstrItinClass;
def IIC_iBITi      : InstrItinClass;
def IIC_iBITr      : InstrItinClass;
def IIC_iBITsi     : InstrItinClass;
def IIC_iBITsr     : InstrItinClass;
def IIC_iUNAr      : InstrItinClass;
def IIC_iUNAsi     : InstrItinClass;
def IIC_iEXTr      : InstrItinClass;
def IIC_iEXTAr     : InstrItinClass;
def IIC_iEXTAsr    : InstrItinClass;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 227-244
```tablegen
def IIC_iCMPi      : InstrItinClass;
def IIC_iCMPr      : InstrItinClass;
def IIC_iCMPsi     : InstrItinClass;
def IIC_iCMPsr     : InstrItinClass;
def IIC_iTSTi      : InstrItinClass;
def IIC_iTSTr      : InstrItinClass;
def IIC_iTSTsi     : InstrItinClass;
def IIC_iTSTsr     : InstrItinClass;
def IIC_iMOVi      : InstrItinClass;
def IIC_iMOVr      : InstrItinClass;
def IIC_iMOVsi     : InstrItinClass;
def IIC_iMOVsr     : InstrItinClass;
def IIC_iMOVix2    : InstrItinClass;
def IIC_iMOVix2addpc : InstrItinClass;
def IIC_iMOVix2ld  : InstrItinClass;
def IIC_iMVNi      : InstrItinClass;
def IIC_iMVNr      : InstrItinClass;
def IIC_iMVNsi     : InstrItinClass;
```
- EN: Defines TableGen record `IIC_iCMPi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_iCMPi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 245-262
```tablegen
def IIC_iMVNsr     : InstrItinClass;
def IIC_iCMOVi     : InstrItinClass;
def IIC_iCMOVr     : InstrItinClass;
def IIC_iCMOVsi    : InstrItinClass;
def IIC_iCMOVsr    : InstrItinClass;
def IIC_iCMOVix2   : InstrItinClass;
def IIC_iMUL16     : InstrItinClass;
def IIC_iMAC16     : InstrItinClass;
def IIC_iMUL32     : InstrItinClass;
def IIC_iMAC32     : InstrItinClass;
def IIC_iMUL64     : InstrItinClass;
def IIC_iMAC64     : InstrItinClass;
def IIC_iDIV     : InstrItinClass;
def IIC_iLoad_i    : InstrItinClass;
def IIC_iLoad_r    : InstrItinClass;
def IIC_iLoad_si   : InstrItinClass;
def IIC_iLoad_iu   : InstrItinClass;
def IIC_iLoad_ru   : InstrItinClass;
```
- EN: Defines TableGen record `IIC_iMVNsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_iMVNsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 263-280
```tablegen
def IIC_iLoad_siu  : InstrItinClass;
def IIC_iLoad_bh_i   : InstrItinClass;
def IIC_iLoad_bh_r   : InstrItinClass;
def IIC_iLoad_bh_si  : InstrItinClass;
def IIC_iLoad_bh_iu  : InstrItinClass;
def IIC_iLoad_bh_ru  : InstrItinClass;
def IIC_iLoad_bh_siu : InstrItinClass;
def IIC_iLoad_d_i  : InstrItinClass;
def IIC_iLoad_d_r  : InstrItinClass;
def IIC_iLoad_d_ru : InstrItinClass;
def IIC_iLoad_m    : InstrItinClass;
def IIC_iLoad_mu   : InstrItinClass;
def IIC_iLoad_mBr  : InstrItinClass;
def IIC_iPop       : InstrItinClass;
def IIC_iPop_Br    : InstrItinClass;
def IIC_iLoadiALU  : InstrItinClass;
def IIC_iStore_i   : InstrItinClass;
def IIC_iStore_r   : InstrItinClass;
```
- EN: Defines TableGen record `IIC_iLoad_siu` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_iLoad_siu`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 281-298
```tablegen
def IIC_iStore_si  : InstrItinClass;
def IIC_iStore_iu  : InstrItinClass;
def IIC_iStore_ru  : InstrItinClass;
def IIC_iStore_siu : InstrItinClass;
def IIC_iStore_bh_i   : InstrItinClass;
def IIC_iStore_bh_r   : InstrItinClass;
def IIC_iStore_bh_si  : InstrItinClass;
def IIC_iStore_bh_iu  : InstrItinClass;
def IIC_iStore_bh_ru  : InstrItinClass;
def IIC_iStore_bh_siu : InstrItinClass;
def IIC_iStore_d_i   : InstrItinClass;
def IIC_iStore_d_r   : InstrItinClass;
def IIC_iStore_d_ru  : InstrItinClass;
def IIC_iStore_m   : InstrItinClass;
def IIC_iStore_mu  : InstrItinClass;
def IIC_Preload    : InstrItinClass;
def IIC_Br         : InstrItinClass;
def IIC_fpSTAT     : InstrItinClass;
```
- EN: Defines TableGen record `IIC_iStore_si` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_iStore_si`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 299-316
```tablegen
def IIC_fpUNA16    : InstrItinClass;
def IIC_fpUNA32    : InstrItinClass;
def IIC_fpUNA64    : InstrItinClass;
def IIC_fpCMP16    : InstrItinClass;
def IIC_fpCMP32    : InstrItinClass;
def IIC_fpCMP64    : InstrItinClass;
def IIC_fpCVTSD    : InstrItinClass;
def IIC_fpCVTDS    : InstrItinClass;
def IIC_fpCVTSH    : InstrItinClass;
def IIC_fpCVTHS    : InstrItinClass;
def IIC_fpCVTIH    : InstrItinClass;
def IIC_fpCVTIS    : InstrItinClass;
def IIC_fpCVTID    : InstrItinClass;
def IIC_fpCVTHI    : InstrItinClass;
def IIC_fpCVTSI    : InstrItinClass;
def IIC_fpCVTDI    : InstrItinClass;
def IIC_fpMOVIS    : InstrItinClass;
def IIC_fpMOVID    : InstrItinClass;
```
- EN: Defines TableGen record `IIC_fpUNA16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_fpUNA16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 317-334
```tablegen
def IIC_fpMOVSI    : InstrItinClass;
def IIC_fpMOVDI    : InstrItinClass;
def IIC_fpALU16    : InstrItinClass;
def IIC_fpALU32    : InstrItinClass;
def IIC_fpALU64    : InstrItinClass;
def IIC_fpMUL16    : InstrItinClass;
def IIC_fpMUL32    : InstrItinClass;
def IIC_fpMUL64    : InstrItinClass;
def IIC_fpMAC16    : InstrItinClass;
def IIC_fpMAC32    : InstrItinClass;
def IIC_fpMAC64    : InstrItinClass;
def IIC_fpFMAC16   : InstrItinClass;
def IIC_fpFMAC32   : InstrItinClass;
def IIC_fpFMAC64   : InstrItinClass;
def IIC_fpDIV16    : InstrItinClass;
def IIC_fpDIV32    : InstrItinClass;
def IIC_fpDIV64    : InstrItinClass;
def IIC_fpSQRT16   : InstrItinClass;
```
- EN: Defines TableGen record `IIC_fpMOVSI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_fpMOVSI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 335-352
```tablegen
def IIC_fpSQRT32   : InstrItinClass;
def IIC_fpSQRT64   : InstrItinClass;
def IIC_fpLoad16   : InstrItinClass;
def IIC_fpLoad32   : InstrItinClass;
def IIC_fpLoad64   : InstrItinClass;
def IIC_fpLoad_m   : InstrItinClass;
def IIC_fpLoad_mu  : InstrItinClass;
def IIC_fpStore16  : InstrItinClass;
def IIC_fpStore32  : InstrItinClass;
def IIC_fpStore64  : InstrItinClass;
def IIC_fpStore_m  : InstrItinClass;
def IIC_fpStore_mu : InstrItinClass;
def IIC_VLD1       : InstrItinClass;
def IIC_VLD1x2     : InstrItinClass;
def IIC_VLD1x3     : InstrItinClass;
def IIC_VLD1x4     : InstrItinClass;
def IIC_VLD1u      : InstrItinClass;
def IIC_VLD1x2u    : InstrItinClass;
```
- EN: Defines TableGen record `IIC_fpSQRT32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_fpSQRT32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 353-370
```tablegen
def IIC_VLD1x3u    : InstrItinClass;
def IIC_VLD1x4u    : InstrItinClass;
def IIC_VLD1ln     : InstrItinClass;
def IIC_VLD1lnu    : InstrItinClass;
def IIC_VLD1dup    : InstrItinClass;
def IIC_VLD1dupu   : InstrItinClass;
def IIC_VLD2       : InstrItinClass;
def IIC_VLD2x2     : InstrItinClass;
def IIC_VLD2u      : InstrItinClass;
def IIC_VLD2x2u    : InstrItinClass;
def IIC_VLD2ln     : InstrItinClass;
def IIC_VLD2lnu    : InstrItinClass;
def IIC_VLD2dup    : InstrItinClass;
def IIC_VLD2dupu   : InstrItinClass;
def IIC_VLD3       : InstrItinClass;
def IIC_VLD3ln     : InstrItinClass;
def IIC_VLD3u      : InstrItinClass;
def IIC_VLD3lnu    : InstrItinClass;
```
- EN: Defines TableGen record `IIC_VLD1x3u` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_VLD1x3u`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 371-388
```tablegen
def IIC_VLD3dup    : InstrItinClass;
def IIC_VLD3dupu   : InstrItinClass;
def IIC_VLD4       : InstrItinClass;
def IIC_VLD4ln     : InstrItinClass;
def IIC_VLD4u      : InstrItinClass;
def IIC_VLD4lnu    : InstrItinClass;
def IIC_VLD4dup    : InstrItinClass;
def IIC_VLD4dupu   : InstrItinClass;
def IIC_VST1       : InstrItinClass;
def IIC_VST1x2     : InstrItinClass;
def IIC_VST1x3     : InstrItinClass;
def IIC_VST1x4     : InstrItinClass;
def IIC_VST1u      : InstrItinClass;
def IIC_VST1x2u    : InstrItinClass;
def IIC_VST1x3u    : InstrItinClass;
def IIC_VST1x4u    : InstrItinClass;
def IIC_VST1ln     : InstrItinClass;
def IIC_VST1lnu    : InstrItinClass;
```
- EN: Defines TableGen record `IIC_VLD3dup` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_VLD3dup`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 389-406
```tablegen
def IIC_VST2       : InstrItinClass;
def IIC_VST2x2     : InstrItinClass;
def IIC_VST2u      : InstrItinClass;
def IIC_VST2x2u    : InstrItinClass;
def IIC_VST2ln     : InstrItinClass;
def IIC_VST2lnu    : InstrItinClass;
def IIC_VST3       : InstrItinClass;
def IIC_VST3u      : InstrItinClass;
def IIC_VST3ln     : InstrItinClass;
def IIC_VST3lnu    : InstrItinClass;
def IIC_VST4       : InstrItinClass;
def IIC_VST4u      : InstrItinClass;
def IIC_VST4ln     : InstrItinClass;
def IIC_VST4lnu    : InstrItinClass;
def IIC_VUNAD      : InstrItinClass;
def IIC_VUNAQ      : InstrItinClass;
def IIC_VBIND      : InstrItinClass;
def IIC_VBINQ      : InstrItinClass;
```
- EN: Defines TableGen record `IIC_VST2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_VST2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 407-424
```tablegen
def IIC_VPBIND     : InstrItinClass;
def IIC_VFMULD     : InstrItinClass;
def IIC_VFMULQ     : InstrItinClass;
def IIC_VMOV       : InstrItinClass;
def IIC_VMOVImm    : InstrItinClass;
def IIC_VMOVD      : InstrItinClass;
def IIC_VMOVQ      : InstrItinClass;
def IIC_VMOVIS     : InstrItinClass;
def IIC_VMOVID     : InstrItinClass;
def IIC_VMOVISL    : InstrItinClass;
def IIC_VMOVSI     : InstrItinClass;
def IIC_VMOVDI     : InstrItinClass;
def IIC_VMOVN      : InstrItinClass;
def IIC_VPERMD     : InstrItinClass;
def IIC_VPERMQ     : InstrItinClass;
def IIC_VPERMQ3    : InstrItinClass;
def IIC_VMACD      : InstrItinClass;
def IIC_VMACQ      : InstrItinClass;
```
- EN: Defines TableGen record `IIC_VPBIND` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_VPBIND`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 425-442
```tablegen
def IIC_VFMACD     : InstrItinClass;
def IIC_VFMACQ     : InstrItinClass;
def IIC_VRECSD     : InstrItinClass;
def IIC_VRECSQ     : InstrItinClass;
def IIC_VCNTiD     : InstrItinClass;
def IIC_VCNTiQ     : InstrItinClass;
def IIC_VUNAiD     : InstrItinClass;
def IIC_VUNAiQ     : InstrItinClass;
def IIC_VQUNAiD    : InstrItinClass;
def IIC_VQUNAiQ    : InstrItinClass;
def IIC_VBINiD     : InstrItinClass;
def IIC_VBINiQ     : InstrItinClass;
def IIC_VSUBiD     : InstrItinClass;
def IIC_VSUBiQ     : InstrItinClass;
def IIC_VBINi4D    : InstrItinClass;
def IIC_VBINi4Q    : InstrItinClass;
def IIC_VSUBi4D    : InstrItinClass;
def IIC_VSUBi4Q    : InstrItinClass;
```
- EN: Defines TableGen record `IIC_VFMACD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_VFMACD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 443-460
```tablegen
def IIC_VABAD      : InstrItinClass;
def IIC_VABAQ      : InstrItinClass;
def IIC_VSHLiD     : InstrItinClass;
def IIC_VSHLiQ     : InstrItinClass;
def IIC_VSHLi4D    : InstrItinClass;
def IIC_VSHLi4Q    : InstrItinClass;
def IIC_VPALiD     : InstrItinClass;
def IIC_VPALiQ     : InstrItinClass;
def IIC_VMULi16D   : InstrItinClass;
def IIC_VMULi32D   : InstrItinClass;
def IIC_VMULi16Q   : InstrItinClass;
def IIC_VMULi32Q   : InstrItinClass;
def IIC_VMACi16D   : InstrItinClass;
def IIC_VMACi32D   : InstrItinClass;
def IIC_VMACi16Q   : InstrItinClass;
def IIC_VMACi32Q   : InstrItinClass;
def IIC_VEXTD      : InstrItinClass;
def IIC_VEXTQ      : InstrItinClass;
```
- EN: Defines TableGen record `IIC_VABAD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_VABAD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 461-469
```tablegen
def IIC_VTB1       : InstrItinClass;
def IIC_VTB2       : InstrItinClass;
def IIC_VTB3       : InstrItinClass;
def IIC_VTB4       : InstrItinClass;
def IIC_VTBX1      : InstrItinClass;
def IIC_VTBX2      : InstrItinClass;
def IIC_VTBX3      : InstrItinClass;
def IIC_VTBX4      : InstrItinClass;
def IIC_VDOTPROD   : InstrItinClass;
```
- EN: Defines TableGen record `IIC_VTB1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IIC_VTB1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: target descriptions, instruction records, and code-generation metadata.
  - CN: 核心职责：目标描述、指令记录以及代码生成元数据。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `(none)`.
  - CN: TableGen 包含项：`(none)`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
