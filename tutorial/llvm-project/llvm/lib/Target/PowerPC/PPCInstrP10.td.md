# PPCInstrP10.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCInstrP10.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCInstrP10.td - Power10 Instruction Set. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCInstrP10.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```tablegen
//===-- PPCInstrP10.td - Power10 Instruction Set -----------*- tablegen -*-===//
//
//                     The LLVM Compiler Infrastructure
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 9-12

```tablegen
//===----------------------------------------------------------------------===//
//
// This file describes the instructions introduced for the Power10 CPU.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file describes the instructions introduced for the Power10 CPU.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file describes the instructions introduced for the Power10 CPU.”。

### Lines 13-14

```tablegen
//===----------------------------------------------------------------------===//
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 15-52

```tablegen
//===----------------------------------------------------------------------===//
// Naming convention for future instruction formats
//
// <INSTR_FORM>{_<OP_TYPE><OP_LENGTH>}+
//
// Where:
// <INSTR_FORM> - name of instruction format as per the ISA
//                (X-Form, VX-Form, etc.)
// <OP_TYPE> - operand type
//             * FRT/RT/VT/XT/BT - target register
//                                 (FPR, GPR, VR, VSR, CR-bit respectively)
//                                 In some situations, the 'T' is replaced by
//                                 'D' when describing the target register.
//             * [FR|R|V|X|B][A-Z] - register source (i.e. FRA, RA, XB, etc.)
//             * IMM - immediate (where signedness matters,
//                     this is SI/UI for signed/unsigned)
//             * [R|X|FR]Tp - register pair target (i.e. FRTp, RTp)
//             * R - PC-Relative bit
//                   (denotes that the address is computed pc-relative)
//             * VRM - Masked Registers
//             * AT - target accumulator
//             * N - the Nth bit in a VSR
//             * Additional 1-bit operands may be required for certain
//               instruction formats such as: MC, P, MP
//             * X / Y / P - mask values. In the instruction encoding, this is
//                           represented as XMSK, YMSK and PMSK.
//             * MEM - indicates if the instruction format requires any memory
//                     accesses. This does not have <OP_LENGTH> attached to it.
// <OP_LENGTH> - the length of each operand in bits.
//               For operands that are 1 bit, the '1' is omitted from the name.
//
// Example: 8RR_XX4Form_IMM8_XTAB6
//          8RR_XX4Form is the instruction format.
//          The operand is an 8-bit immediate (IMM), the destination (XT)
//          and sources (XA, XB) that are all 6-bits. The destination and
//          source registers are combined if they are of the same length.
//          Moreover, the order of operands reflects the order of operands
//          in the encoding.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Naming convention for future instruction formats".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Naming convention for future instruction formats”。

### Lines 53-53

```tablegen

```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 54-60

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC ISA 3.1 specific type constraints.
//

def SDT_PPCSplat32 : SDTypeProfile<1, 3, [ SDTCisVT<0, v2i64>,
  SDTCisVec<1>, SDTCisInt<2>, SDTCisInt<3>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCSplat32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCSplat32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 61-67

```tablegen
def SDT_PPCAccBuild : SDTypeProfile<1, 4, [
  SDTCisVT<0, v512i1>, SDTCisVT<1, v4i32>, SDTCisVT<2, v4i32>,
                       SDTCisVT<3, v4i32>, SDTCisVT<4, v4i32>
]>;
def SDT_PPCPairBuild : SDTypeProfile<1, 2, [
  SDTCisVT<0, v256i1>, SDTCisVT<1, v4i32>, SDTCisVT<2, v4i32>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCAccBuild`, `SDT_PPCPairBuild` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCAccBuild`, `SDT_PPCPairBuild`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 68-77

```tablegen
def SDT_PPCAccExtractVsx : SDTypeProfile<1, 2, [
  SDTCisVT<0, v4i32>, SDTCisVT<1, v512i1>, SDTCisPtrTy<2>
]>;
def SDT_PPCPairExtractVsx : SDTypeProfile<1, 2, [
  SDTCisVT<0, v4i32>, SDTCisVT<1, v256i1>, SDTCisPtrTy<2>
]>;
def SDT_PPCxxmfacc : SDTypeProfile<1, 1, [
  SDTCisVT<0, v512i1>, SDTCisVT<1, v512i1>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCAccExtractVsx`, `SDT_PPCPairExtractVsx`, `SDT_PPCxxmfacc` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCAccExtractVsx`, `SDT_PPCPairExtractVsx`, `SDT_PPCxxmfacc`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 78-81

```tablegen
def SDT_PPCsetbc : SDTypeProfile<1, 1, [
  SDTCisInt<0>, SDTCisInt<1>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCsetbc` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCsetbc`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 82-89

```tablegen
//===----------------------------------------------------------------------===//
// ISA 3.1 specific PPCISD nodes.
//

// XXSPLTI32DX - The PPC XXSPLTI32DX instruction.
def PPCxxsplti32dx : SDNode<"PPCISD::XXSPLTI32DX", SDT_PPCSplat32, []>;

// ACC_BUILD = Build an accumulator register from 4 VSX registers.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "ISA 3.1 specific PPCISD nodes.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“ISA 3.1 specific PPCISD nodes.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 90-98

```tablegen
def PPCAccBuild : SDNode<"PPCISD::ACC_BUILD", SDT_PPCAccBuild, []>;

// PAIR_BUILD = Build a vector pair register from 2 VSX registers.
def PPCPairBuild : SDNode<"PPCISD::PAIR_BUILD", SDT_PPCPairBuild, []>;

// EXTRACT_VSX_REG = Extract one of the underlying vsx registers of
// an accumulator or pair register. This node is needed because
// EXTRACT_SUBVECTOR expects the input and output vectors to have the same
// element type.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PAIR_BUILD = Build a vector pair register from 2 VSX registers.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PAIR_BUILD = Build a vector pair register from 2 VSX registers.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 99-107

```tablegen
def PPCAccExtractVsx : SDNode<"PPCISD::EXTRACT_VSX_REG", SDT_PPCAccExtractVsx,
                       []>;
def PPCPairExtractVsx : SDNode<"PPCISD::EXTRACT_VSX_REG", SDT_PPCPairExtractVsx,
                        []>;

// XXMFACC = This corresponds to the xxmfacc instruction.
def PPCxxmfacc : SDNode<"PPCISD::XXMFACC", SDT_PPCxxmfacc, []>;

// SETBC - The ISA 3.1 (P10) SETBC instruction.
```
- **EN**: Adds declarative TableGen records such as `PPCAccExtractVsx`, `PPCPairExtractVsx`, `PPCxxmfacc` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCAccExtractVsx`, `PPCPairExtractVsx`, `PPCxxmfacc`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 108-112

```tablegen
def PPCsetbc  : SDNode<"PPCISD::SETBC",   SDT_PPCsetbc, []>;

// SETBCR - The ISA 3.1 (P10) SETBCR instruction.
def PPCsetbcr : SDNode<"PPCISD::SETBCR",  SDT_PPCsetbc, []>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "SETBCR - The ISA 3.1 (P10) SETBCR instruction.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“SETBCR - The ISA 3.1 (P10) SETBCR instruction.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 113-119

```tablegen
//===----------------------------------------------------------------------===//

// PC Relative flag (for instructions that use the address of the prefix for
// address computations).
class isPCRel { bit PCRel = 1; }

// PowerPC specific type constraints.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PC Relative flag (for instructions that use the address of the prefix for". Notable symbols in this range include `isPCRel`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PC Relative flag (for instructions that use the address of the prefix for”。 该区间中较显眼的符号包括 `isPCRel`。

### Lines 120-127

```tablegen
def SDT_PPCLXVRZX : SDTypeProfile<1, 2, [
  SDTCisVT<0, v1i128>, SDTCisPtrTy<1>, SDTCisPtrTy<2>
]>;

// LXVRZX - Load VSX Vector Rightmost and Zero Extend
// This node represents v1i128 BUILD_VECTOR of a zero extending load
// instruction from <byte, halfword, word, or doubleword> to i128.
// Allows utilization of the Load VSX Vector Rightmost Instructions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "LXVRZX - Load VSX Vector Rightmost and Zero Extend". Notable symbols in this range include `SDT_PPCLXVRZX`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“LXVRZX - Load VSX Vector Rightmost and Zero Extend”。 该区间中较显眼的符号包括 `SDT_PPCLXVRZX`。

### Lines 128-135

```tablegen
def PPClxvrzx : SDNode<"PPCISD::LXVRZX", SDT_PPCLXVRZX,
                       [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

// Top-level class for prefixed instructions.
class PI<bits<6> pref, bits<6> opcode, dag OOL, dag IOL, string asmstr,
         InstrItinClass itin> : Instruction {
  field bits<64> Inst;
  bit PCRel = 0; // Default value, set by isPCRel.
```
- **EN**: Declares a backend-facing type `PPClxvrzx`, `PI` and outlines the API or state that nearby code will rely on. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里声明面向后端的类型 `PPClxvrzx`, `PI`，并勾勒出周边代码会依赖的接口或状态。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 136-142

```tablegen
  let Size = 8;

  let Namespace = "PPC";
  let OutOperandList = OOL;
  let InOperandList = IOL;
  let AsmString = asmstr;
  let Itinerary = itin;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. Scheduling or processor-model metadata is part of this section.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这一段包含调度或处理器模型元数据。

### Lines 143-152

```tablegen
  let Inst{0...5} = pref;
  let Inst{32...37} = opcode;

  bits<1> PPC970_First = 0;
  bits<1> PPC970_Single = 0;
  bits<1> PPC970_Cracked = 0;
  bits<3> PPC970_Unit = 0;

  /// These fields correspond to the fields in PPCInstrInfo.h.  Any changes to
  /// these must be reflected there!  See comments there for what these are.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。

### Lines 153-162

```tablegen
  let TSFlags{0}   = PPC970_First;
  let TSFlags{1}   = PPC970_Single;
  let TSFlags{2}   = PPC970_Cracked;
  let TSFlags{5...3} = PPC970_Unit;

  bits<1> Prefixed = 1;  // This is a prefixed instruction.
  let TSFlags{7}  = Prefixed;

  // Indicate that this instruction takes a register+immediate memory operand.
  bits<1> MemriOp = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 163-175

```tablegen
  let TSFlags{10} = MemriOp;

  // For cases where multiple instruction definitions really represent the
  // same underlying instruction but with one definition for 64-bit arguments
  // and one for 32-bit arguments, this bit breaks the degeneracy between
  // the two forms and allows TableGen to generate mapping tables.
  bit Interpretation64Bit = 0;

  // Fields used for relation models.
  string BaseName = "";
}

// VX-Form: [ PO VT R VB RC XO ]
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "For cases where multiple instruction definitions really represent the".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“For cases where multiple instruction definitions really represent the”。

### Lines 176-182

```tablegen
class VXForm_VTB5_RC<bits<10> xo, bits<5> R, dag OOL, dag IOL, string asmstr,
                      InstrItinClass itin, list<dag> pattern>
  : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VT;
  bits<5> VB;
  bit RC = 0;
```
- **EN**: Declares a backend-facing type `VXForm_VTB5_RC` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_VTB5_RC`，并勾勒出周边代码会依赖的接口或状态。

### Lines 183-193

```tablegen
  let Pattern = pattern;

  let Inst{6...10} = VT;
  let Inst{11...15} = R;
  let Inst{16...20} = VB;
  let Inst{21} = RC;
  let Inst{22...31} = xo;
}

// Multiclass definition to account for record and non-record form
// instructions of VXRForm.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step. These declarations feed generated pattern-matching logic.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。 这些声明会进入生成式模式匹配逻辑。

### Lines 194-200

```tablegen
multiclass VXForm_VTB5_RCr<bits<10> xo, bits<5> R, dag OOL, dag IOL,
                            string asmbase, string asmstr,
                            InstrItinClass itin, list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : VXForm_VTB5_RC<xo, R, OOL, IOL,
                               !strconcat(asmbase, !strconcat(" ", asmstr)),
                               itin, pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `VXForm_VTB5_RCr` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VXForm_VTB5_RCr`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 201-207

```tablegen
    let Defs = [CR6] in
    def _rec : VXForm_VTB5_RC<xo, R, OOL, IOL,
                               !strconcat(asmbase, !strconcat(". ", asmstr)),
                               itin, []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 208-214

```tablegen
class MLS_DForm_R_SI34_RTA5_MEM<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                                InstrItinClass itin, list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin>, MemriOp {
  bits<5> RST;
  bits<5> RA;
  bits<34> D;
```
- **EN**: Declares a backend-facing type `MLS_DForm_R_SI34_RTA5_MEM` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MLS_DForm_R_SI34_RTA5_MEM`，并勾勒出周边代码会依赖的接口或状态。

### Lines 215-221

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 2;
  let Inst{8...10} = 0;
  let Inst{11} = PCRel;
  let Inst{12...13} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 222-229

```tablegen
  let Inst{14...31} = D{33...16}; // d0

  // The instruction.
  let Inst{38...42} = RST{4...0};
  let Inst{43...47} = RA;
  let Inst{48...63} = D{15...0}; // d1
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 230-236

```tablegen
class MLS_DForm_R_SI34_RTA5<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                            InstrItinClass itin, list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<5> RA;
  bits<34> SI;
```
- **EN**: Declares a backend-facing type `MLS_DForm_R_SI34_RTA5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MLS_DForm_R_SI34_RTA5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 237-243

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 2;
  let Inst{8...10} = 0;
  let Inst{11} = PCRel;
  let Inst{12...13} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 244-251

```tablegen
  let Inst{14...31} = SI{33...16};

  // The instruction.
  let Inst{38...42} = RT;
  let Inst{43...47} = RA;
  let Inst{48...63} = SI{15...0};
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 252-260

```tablegen
class MLS_DForm_SI34_RT5<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                         InstrItinClass itin, list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<5> RT;
  bits<34> SI;

  let Pattern = pattern;

  // The prefix.
```
- **EN**: Declares a backend-facing type `MLS_DForm_SI34_RT5` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `MLS_DForm_SI34_RT5`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 261-267

```tablegen
  let Inst{6...7} = 2;
  let Inst{8...10} = 0;
  let Inst{11} = 0;
  let Inst{12...13} = 0;
  let Inst{14...31} = SI{33...16};

  // The instruction.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 268-275

```tablegen
  let Inst{38...42} = RT;
  let Inst{43...47} = 0;
  let Inst{48...63} = SI{15...0};
}

multiclass MLS_DForm_R_SI34_RTA5_p<bits<6> opcode, dag OOL, dag IOL,
                                   dag PCRel_IOL, string asmstr,
                                   InstrItinClass itin> {
```
- **EN**: Adds declarative TableGen records such as `MLS_DForm_R_SI34_RTA5_p` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MLS_DForm_R_SI34_RTA5_p`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 276-288

```tablegen
  def NAME : MLS_DForm_R_SI34_RTA5<opcode, OOL, IOL,
                                   !strconcat(asmstr, ", 0"), itin, []>;
  def pc : MLS_DForm_R_SI34_RTA5<opcode, OOL, PCRel_IOL,
                                 !strconcat(asmstr, ", 1"), itin, []>, isPCRel;
}

class 8LS_DForm_R_SI34_RTA5_MEM<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                                InstrItinClass itin, list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin>, MemriOp {
  bits<5> RST;
  bits<5> RA;
  bits<34> D;
```
- **EN**: Declares a backend-facing type and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型，并勾勒出周边代码会依赖的接口或状态。

### Lines 289-297

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...10} = 0;
  let Inst{11} = PCRel;
  let Inst{12...13} = 0;
  let Inst{14...31} = D{33...16}; // d0

  // The instruction.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 298-304

```tablegen
  let Inst{38...42} = RST{4...0};
  let Inst{43...47} = RA;
  let Inst{48...63} = D{15...0}; // d1
}

// 8LS:D-Form: [ 1 0 0 // R // d0
//               PO TX T RA d1 ]
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "8LS:D-Form: [ 1 0 0 // R // d0".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“8LS:D-Form: [ 1 0 0 // R // d0”。

### Lines 305-312

```tablegen
class 8LS_DForm_R_SI34_XT6_RA5_MEM<bits<5> opcode, dag OOL, dag IOL,
                                   string asmstr, InstrItinClass itin,
                                   list<dag> pattern>
  : PI<1, { opcode, ? }, OOL, IOL, asmstr, itin>, MemriOp {
  bits<6> XST;
  bits<5> RA;
  bits<34> D;
```
- **EN**: Declares a backend-facing type and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型，并勾勒出周边代码会依赖的接口或状态。

### Lines 313-319

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 0;
  let Inst{8} = 0;
  let Inst{9...10} = 0; // reserved
  let Inst{11} = PCRel;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 320-326

```tablegen
  let Inst{12...13} = 0; // reserved
  let Inst{14...31} = D{33...16}; // d0

  // The instruction.
  let Inst{37} = XST{5};
  let Inst{38...42} = XST{4...0};
  let Inst{43...47} = RA;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 327-337

```tablegen
  let Inst{48...63} = D{15...0}; // d1
}

// X-Form: [PO T IMM VRB XO TX]
class XForm_XT6_IMM5_VB5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                         string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<5> VRB;
  bits<5> IMM;
```
- **EN**: Declares a backend-facing type `XForm_XT6_IMM5_VB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_XT6_IMM5_VB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 338-345

```tablegen
  let Pattern = pattern;
  let Inst{6...10} = XT{4...0};
  let Inst{11...15} = IMM;
  let Inst{16...20} = VRB;
  let Inst{21...30} = xo;
  let Inst{31} = XT{5};
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 346-355

```tablegen
class 8RR_XX4Form_IMM8_XTAB6<bits<6> opcode, bits<2> xo,
                             dag OOL, dag IOL, string asmstr,
                             InstrItinClass itin, list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
    bits<6> XT;
    bits<6> XA;
    bits<6> XB;
    bits<6> XC;
    bits<8> IMM;
```
- **EN**: Declares a backend-facing type and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型，并勾勒出周边代码会依赖的接口或状态。

### Lines 356-362

```tablegen
    let Pattern = pattern;

    // The prefix.
    let Inst{6...7} = 1;
    let Inst{8} = 0;
    let Inst{9...11} = 0;
    let Inst{12...13} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 363-369

```tablegen
    let Inst{14...23} = 0;
    let Inst{24...31} = IMM;

    // The instruction.
    let Inst{38...42} = XT{4...0};
    let Inst{43...47} = XA{4...0};
    let Inst{48...52} = XB{4...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 370-377

```tablegen
    let Inst{53...57} = XC{4...0};
    let Inst{58...59} = xo;
    let Inst{60} = XC{5};
    let Inst{61} = XA{5};
    let Inst{62} = XB{5};
    let Inst{63} = XT{5};
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 378-384

```tablegen
class VXForm_RD5_N3_VB5<bits<11> xo, dag OOL, dag IOL, string asmstr,
                        InstrItinClass itin, list<dag> pattern>
  : I<4, OOL, IOL, asmstr, itin> {
  bits<5> RD;
  bits<5> VB;
  bits<3> N;
```
- **EN**: Declares a backend-facing type `VXForm_RD5_N3_VB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_RD5_N3_VB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 385-396

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RD;
  let Inst{11...12} = 0;
  let Inst{13...15} = N;
  let Inst{16...20} = VB;
  let Inst{21...31} = xo;
}


// VX-Form: [PO VRT RA VRB XO].
// Destructive (insert) forms are suffixed with _ins.
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 397-403

```tablegen
class VXForm_VTB5_RA5_ins<bits<11> xo, string opc, list<dag> pattern>
  : VXForm_1<xo, (outs vrrc:$VD), (ins vrrc:$VDi, gprc:$VA, vrrc:$VB),
             !strconcat(opc, " $VD, $VA, $VB"), IIC_VecGeneral, pattern>,
             RegConstraint<"$VDi = $VD">;

// VX-Form: [PO VRT RA RB XO].
// Destructive (insert) forms are suffixed with _ins.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VX-Form: [PO VRT RA RB XO].". Notable symbols in this range include `VXForm_VTB5_RA5_ins`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VX-Form: [PO VRT RA RB XO].”。 该区间中较显眼的符号包括 `VXForm_VTB5_RA5_ins`。

### Lines 404-416

```tablegen
class VXForm_VRT5_RAB5_ins<bits<11> xo, string opc, list<dag> pattern>
  : VXForm_1<xo, (outs vrrc:$VD), (ins vrrc:$VDi, gprc:$VA, gprc:$VB),
             !strconcat(opc, " $VD, $VA, $VB"), IIC_VecGeneral, pattern>,
             RegConstraint<"$VDi = $VD">;

// VX-Form: [ PO BF // VRA VRB XO ]
class VXForm_BF3_VAB5<bits<11> xo, dag OOL, dag IOL, string asmstr,
                      InstrItinClass itin, list<dag> pattern>
  : I<4, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bits<5> VA;
  bits<5> VB;
```
- **EN**: Declares a backend-facing type `VXForm_VRT5_RAB5_ins`, `VXForm_BF3_VAB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_VRT5_RAB5_ins`, `VXForm_BF3_VAB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 417-427

```tablegen
  let Pattern = pattern;

  let Inst{6...8} = BF;
  let Inst{9...10} = 0;
  let Inst{11...15} = VA;
  let Inst{16...20} = VB;
  let Inst{21...31} = xo;
}

// VN-Form: [PO VRT VRA VRB PS SD XO]
// SD is "Shift Direction"
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 428-435

```tablegen
class VNForm_VTAB5_SD3<bits<6> xo, bits<2> ps, dag OOL, dag IOL, string asmstr,
                       InstrItinClass itin, list<dag> pattern>
    : I<4, OOL, IOL, asmstr, itin> {
  bits<5> VRT;
  bits<5> VRA;
  bits<5> VRB;
  bits<3> SD;
```
- **EN**: Declares a backend-facing type `VNForm_VTAB5_SD3` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VNForm_VTAB5_SD3`，并勾勒出周边代码会依赖的接口或状态。

### Lines 436-442

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = VRT;
  let Inst{11...15} = VRA;
  let Inst{16...20} = VRB;
  let Inst{21...22} = ps;
  let Inst{23...25} = SD;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 443-452

```tablegen
  let Inst{26...31} = xo;
}

class VXForm_RD5_MP_VB5<bits<11> xo, bits<4> eo, dag OOL, dag IOL,
                        string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<4, OOL, IOL, asmstr, itin> {
  bits<5> RD;
  bits<5> VB;
  bit MP;
```
- **EN**: Declares a backend-facing type `VXForm_RD5_MP_VB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `VXForm_RD5_MP_VB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 453-463

```tablegen
  let Pattern = pattern;

  let Inst{6...10}  = RD;
  let Inst{11...14} = eo;
  let Inst{15} = MP;
  let Inst{16...20} = VB;
  let Inst{21...31} = xo;
}

// 8RR:D-Form: [ 1 1 0 // // imm0
//               PO T XO TX imm1 ].
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 464-470

```tablegen
class 8RR_DForm_IMM32_XT6<bits<6> opcode, bits<4> xo, dag OOL, dag IOL,
                          string asmstr, InstrItinClass itin,
                          list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<32> IMM32;
```
- **EN**: Declares a backend-facing type and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型，并勾勒出周边代码会依赖的接口或状态。

### Lines 471-477

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 1;
  let Inst{8...11} = 0;
  let Inst{12...13} = 0; // reserved
  let Inst{14...15} = 0; // reserved
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 478-488

```tablegen
  let Inst{16...31} = IMM32{31...16};

  // The instruction.
  let Inst{38...42} = XT{4...0};
  let Inst{43...46} = xo;
  let Inst{47} = XT{5};
  let Inst{48...63} = IMM32{15...0};
}

// 8RR:D-Form: [ 1 1 0 // // imm0
//               PO T XO IX TX imm1 ].
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The instruction.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The instruction.”。

### Lines 489-496

```tablegen
class 8RR_DForm_IMM32_XT6_IX<bits<6> opcode, bits<3> xo, dag OOL, dag IOL,
                             string asmstr, InstrItinClass itin,
                             list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bit IX;
  bits<32> IMM32;
```
- **EN**: Declares a backend-facing type and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型，并勾勒出周边代码会依赖的接口或状态。

### Lines 497-503

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 1;
  let Inst{8...11} = 0;
  let Inst{12...13} = 0; // reserved
  let Inst{14...15} = 0; // reserved
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 504-510

```tablegen
  let Inst{16...31} = IMM32{31...16};

  // The instruction.
  let Inst{38...42} = XT{4...0};
  let Inst{43...45} = xo;
  let Inst{46} = IX;
  let Inst{47} = XT{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 511-521

```tablegen
  let Inst{48...63} = IMM32{15...0};
}

class 8RR_XX4Form_XTABC6<bits<6> opcode, bits<2> xo, dag OOL, dag IOL,
                         string asmstr, InstrItinClass itin, list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<6> XA;
  bits<6> XB;
  bits<6> XC;
```
- **EN**: Declares a backend-facing type and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型，并勾勒出周边代码会依赖的接口或状态。

### Lines 522-530

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 1;
  let Inst{8...11} = 0;
  let Inst{12...13} = 0;
  let Inst{14...31} = 0;

  // The instruction.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 531-537

```tablegen
  let Inst{38...42} = XT{4...0};
  let Inst{43...47} = XA{4...0};
  let Inst{48...52} = XB{4...0};
  let Inst{53...57} = XC{4...0};
  let Inst{58...59} = xo;
  let Inst{60} = XC{5};
  let Inst{61} = XA{5};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 538-551

```tablegen
  let Inst{62} = XB{5};
  let Inst{63} = XT{5};
}

class 8RR_XX4Form_IMM3_XTABC6<bits<6> opcode, bits<2> xo, dag OOL, dag IOL,
                              string asmstr, InstrItinClass itin,
                              list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<6> XA;
  bits<6> XB;
  bits<6> XC;
  bits<3> IMM;
```
- **EN**: Declares a backend-facing type and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型，并勾勒出周边代码会依赖的接口或状态。

### Lines 552-558

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 1;
  let Inst{8...11} = 0;
  let Inst{12...13} = 0;
  let Inst{14...28} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 559-565

```tablegen
  let Inst{29...31} = IMM;

  // The instruction.
  let Inst{38...42} = XT{4...0};
  let Inst{43...47} = XA{4...0};
  let Inst{48...52} = XB{4...0};
  let Inst{53...57} = XC{4...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 566-573

```tablegen
  let Inst{58...59} = xo;
  let Inst{60} = XC{5};
  let Inst{61} = XA{5};
  let Inst{62} = XB{5};
  let Inst{63} = XT{5};
}

// [PO BF / XO2 B XO BX /]
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 574-580

```tablegen
class XX2_BF3_XO5_XB6_XO9<bits<6> opcode, bits<5> xo2, bits<9> xo, dag OOL,
                          dag IOL, string asmstr, InstrItinClass itin,
                          list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> BF;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX2_BF3_XO5_XB6_XO9` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX2_BF3_XO5_XB6_XO9`，并勾勒出周边代码会依赖的接口或状态。

### Lines 581-587

```tablegen
  let Pattern = pattern;

  let Inst{6...8}   = BF;
  let Inst{9...10}  = 0;
  let Inst{11...15} = xo2;
  let Inst{16...20} = XB{4...0};
  let Inst{21...29} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 588-596

```tablegen
  let Inst{30}    = XB{5};
  let Inst{31}    = 0;
}

// X-Form: [ PO RT BI /// XO / ]
class XForm_XT5_BI5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                    string asmstr, InstrItinClass itin, list<dag> pattern>
  : XForm_base_r3xo<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
  bits<5> BI;
```
- **EN**: Declares a backend-facing type `XForm_XT5_BI5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_XT5_BI5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 597-604

```tablegen
  let RA = BI;
  let RB = 0;
}

multiclass MLS_DForm_R_SI34_RTA5_MEM_p<bits<6> opcode, dag OOL, dag IOL,
                                       dag PCRel_IOL, dag PCRelOnly_IOL,
                                       string asmstr, string asmstr_pcext,
                                       InstrItinClass itin> {
```
- **EN**: Adds declarative TableGen records such as `MLS_DForm_R_SI34_RTA5_MEM_p` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MLS_DForm_R_SI34_RTA5_MEM_p`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 605-611

```tablegen
  def NAME : MLS_DForm_R_SI34_RTA5_MEM<opcode, OOL, IOL,
                                       !strconcat(asmstr, ", 0"), itin, []>;
  def pc : MLS_DForm_R_SI34_RTA5_MEM<opcode, OOL, PCRel_IOL,
                                     !strconcat(asmstr, ", 1"), itin, []>,
                                     isPCRel;
  let isAsmParserOnly = 1, hasNoSchedulingInfo = 1 in {
    def nopc : MLS_DForm_R_SI34_RTA5_MEM<opcode, OOL, IOL, asmstr, itin, []>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这一段包含调度或处理器模型元数据。

### Lines 612-621

```tablegen
    let RA = 0, MemriOp = 0 in
      def onlypc : MLS_DForm_R_SI34_RTA5_MEM<opcode, OOL, PCRelOnly_IOL,
                                             asmstr_pcext, itin, []>, isPCRel;
  }
}

multiclass 8LS_DForm_R_SI34_RTA5_MEM_p<bits<6> opcode, dag OOL, dag IOL,
                                       dag PCRel_IOL, dag PCRelOnly_IOL,
                                       string asmstr, string asmstr_pcext,
                                       InstrItinClass itin> {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 622-628

```tablegen
  def NAME : 8LS_DForm_R_SI34_RTA5_MEM<opcode, OOL, IOL,
                                       !strconcat(asmstr, ", 0"), itin, []>;
  def pc : 8LS_DForm_R_SI34_RTA5_MEM<opcode, OOL, PCRel_IOL,
                                     !strconcat(asmstr, ", 1"), itin, []>,
                                     isPCRel;
  let isAsmParserOnly = 1, hasNoSchedulingInfo = 1 in {
    def nopc : 8LS_DForm_R_SI34_RTA5_MEM<opcode, OOL, IOL, asmstr, itin, []>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这一段包含调度或处理器模型元数据。

### Lines 629-638

```tablegen
    let RA = 0, MemriOp = 0 in
      def onlypc : 8LS_DForm_R_SI34_RTA5_MEM<opcode, OOL, PCRelOnly_IOL,
                                             asmstr_pcext, itin, []>, isPCRel;
  }
}

multiclass 8LS_DForm_R_SI34_XT6_RA5_MEM_p<bits<5> opcode, dag OOL, dag IOL,
                                          dag PCRel_IOL, dag PCRelOnly_IOL,
                                          string asmstr, string asmstr_pcext,
                                          InstrItinClass itin> {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 639-645

```tablegen
  def NAME : 8LS_DForm_R_SI34_XT6_RA5_MEM<opcode, OOL, IOL,
                                          !strconcat(asmstr, ", 0"), itin, []>;
  def pc : 8LS_DForm_R_SI34_XT6_RA5_MEM<opcode, OOL, PCRel_IOL,
                                        !strconcat(asmstr, ", 1"), itin, []>,
                                        isPCRel;
  let isAsmParserOnly = 1, hasNoSchedulingInfo = 1 in {
    def nopc : 8LS_DForm_R_SI34_XT6_RA5_MEM<opcode, OOL, IOL, asmstr, itin, []>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这一段包含调度或处理器模型元数据。

### Lines 646-657

```tablegen
    let RA = 0, MemriOp = 0 in
      def onlypc : 8LS_DForm_R_SI34_XT6_RA5_MEM<opcode, OOL, PCRelOnly_IOL,
                                                asmstr_pcext, itin, []>, isPCRel;
  }
}

def RCCp {
  dag AToVSRC = (COPY_TO_REGCLASS $XA, VSRC);
  dag BToVSRC = (COPY_TO_REGCLASS $XB, VSRC);
  dag CToVSRC = (COPY_TO_REGCLASS $XC, VSRC);
}
```
- **EN**: Adds declarative TableGen records such as `RCCp` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RCCp`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 658-664

```tablegen
let Predicates = [PrefixInstrs] in {
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
    defm PADDI8 :
      MLS_DForm_R_SI34_RTA5_p<14, (outs g8rc:$RT), (ins g8rc_nox0:$RA, s34imm64:$SI),
                              (ins immZero:$RA, s34imm64_pcrel:$SI),
                              "paddi $RT, $RA, $SI", IIC_LdStLFD>;
    let isReMaterializable = 1, isAsCheapAsAMove = 1, isMoveImm = 1 in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 665-673

```tablegen
      def PLI8 : MLS_DForm_SI34_RT5<14, (outs g8rc:$RT),
                                    (ins s34imm64:$SI),
                                    "pli $RT, $SI", IIC_IntSimple, []>;
    }
  }
  defm PADDI :
    MLS_DForm_R_SI34_RTA5_p<14, (outs gprc:$RT), (ins gprc_nor0:$RA, s34imm:$SI),
                            (ins immZero:$RA, s34imm64_pcrel:$SI),
                            "paddi $RT, $RA, $SI", IIC_LdStLFD>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 674-680

```tablegen
  let isReMaterializable = 1, isAsCheapAsAMove = 1, isMoveImm = 1 in {
    def PLI : MLS_DForm_SI34_RT5<14, (outs gprc:$RT),
                                 (ins s34imm:$SI),
                                 "pli $RT, $SI", IIC_IntSimple, []>;
  }

  let mayLoad = 1, mayStore = 0 in {
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 681-691

```tablegen
    let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
      defm PLBZ8 :
        MLS_DForm_R_SI34_RTA5_MEM_p<34, (outs g8rc:$RST), (ins (memri34 $D, $RA):$addr),
                                    (ins (memri34_pcrel $D, $RA):$addr),
                                    (ins s34imm64_pcrel:$D), "plbz $RST, $addr",
                                    "plbz $RST, $D", IIC_LdStLFD>;
      defm PLHZ8 :
        MLS_DForm_R_SI34_RTA5_MEM_p<40, (outs g8rc:$RST), (ins (memri34 $D, $RA):$addr),
                                    (ins (memri34_pcrel $D, $RA):$addr),
                                    (ins s34imm64_pcrel:$D), "plhz $RST, $addr",
                                    "plhz $RST, $D", IIC_LdStLFD>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 692-701

```tablegen
      defm PLHA8 :
        MLS_DForm_R_SI34_RTA5_MEM_p<42, (outs g8rc:$RST), (ins (memri34 $D, $RA):$addr),
                                    (ins (memri34_pcrel $D, $RA):$addr),
                                    (ins s34imm64_pcrel:$D), "plha $RST, $addr",
                                    "plha $RST, $D", IIC_LdStLFD>;
      defm PLWA8 :
        8LS_DForm_R_SI34_RTA5_MEM_p<41, (outs g8rc:$RST), (ins (memri34 $D, $RA):$addr),
                                    (ins (memri34_pcrel $D, $RA):$addr),
                                    (ins s34imm64_pcrel:$D),
                                    "plwa $RST, $addr", "plwa $RST, $D",  IIC_LdStLFD>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 702-712

```tablegen
      defm PLWZ8 :
        MLS_DForm_R_SI34_RTA5_MEM_p<32, (outs g8rc:$RST), (ins (memri34 $D, $RA):$addr),
                                    (ins (memri34_pcrel $D, $RA):$addr),
                                    (ins s34imm64_pcrel:$D), "plwz $RST, $addr",
                                    "plwz $RST, $D", IIC_LdStLFD>;
    }
    defm PLBZ :
      MLS_DForm_R_SI34_RTA5_MEM_p<34, (outs gprc:$RST), (ins (memri34 $D, $RA):$addr),
                                  (ins (memri34_pcrel $D, $RA):$addr),
                                  (ins s34imm64_pcrel:$D), "plbz $RST, $addr",
                                  "plbz $RST, $D", IIC_LdStLFD>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 713-722

```tablegen
    defm PLHZ :
      MLS_DForm_R_SI34_RTA5_MEM_p<40, (outs gprc:$RST), (ins (memri34 $D, $RA):$addr),
                                  (ins (memri34_pcrel $D, $RA):$addr),
                                  (ins s34imm64_pcrel:$D), "plhz $RST, $addr",
                                  "plhz $RST, $D", IIC_LdStLFD>;
    defm PLHA :
      MLS_DForm_R_SI34_RTA5_MEM_p<42, (outs gprc:$RST), (ins (memri34 $D, $RA):$addr),
                                  (ins (memri34_pcrel $D, $RA):$addr),
                                  (ins s34imm64_pcrel:$D), "plha $RST, $addr",
                                  "plha $RST, $D", IIC_LdStLFD>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 723-733

```tablegen
    defm PLWZ :
      MLS_DForm_R_SI34_RTA5_MEM_p<32, (outs gprc:$RST), (ins (memri34 $D, $RA):$addr),
                                  (ins (memri34_pcrel $D, $RA):$addr),
                                  (ins s34imm64_pcrel:$D), "plwz $RST, $addr",
                                  "plwz $RST, $D", IIC_LdStLFD>;
    defm PLWA :
      8LS_DForm_R_SI34_RTA5_MEM_p<41, (outs gprc:$RST), (ins (memri34 $D, $RA):$addr),
                                  (ins (memri34_pcrel $D, $RA):$addr),
                                  (ins s34imm64_pcrel:$D),
                                  "plwa $RST, $addr", "plwa $RST, $D",
                                  IIC_LdStLFD>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 734-741

```tablegen
    defm PLD :
      8LS_DForm_R_SI34_RTA5_MEM_p<57, (outs g8rc:$RST), (ins (memri34 $D, $RA):$addr),
                                  (ins (memri34_pcrel $D, $RA):$addr),
                                  (ins s34imm64_pcrel:$D),
                                  "pld $RST, $addr", "pld $RST, $D",
                                  IIC_LdStLFD>;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 742-748

```tablegen
  let mayStore = 1, mayLoad = 0 in {
    let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
      defm PSTB8 :
        MLS_DForm_R_SI34_RTA5_MEM_p<38, (outs), (ins g8rc:$RST, (memri34 $D, $RA):$addr),
                                    (ins g8rc:$RST, (memri34_pcrel $D, $RA):$addr),
                                    (ins g8rc:$RST, s34imm64_pcrel:$D),
                                    "pstb $RST, $addr", "pstb $RST, $D", IIC_LdStLFD>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 749-759

```tablegen
      defm PSTH8 :
        MLS_DForm_R_SI34_RTA5_MEM_p<44, (outs), (ins g8rc:$RST, (memri34 $D, $RA):$addr),
                                    (ins g8rc:$RST, (memri34_pcrel $D, $RA):$addr),
                                    (ins g8rc:$RST, s34imm64_pcrel:$D),
                                    "psth $RST, $addr", "psth $RST, $D", IIC_LdStLFD>;
      defm PSTW8 :
        MLS_DForm_R_SI34_RTA5_MEM_p<36, (outs), (ins g8rc:$RST, (memri34 $D, $RA):$addr),
                                    (ins g8rc:$RST, (memri34_pcrel $D, $RA):$addr),
                                    (ins g8rc:$RST, s34imm64_pcrel:$D),
                                    "pstw $RST, $addr", "pstw $RST, $D", IIC_LdStLFD>;
    }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 760-769

```tablegen
    defm PSTB :
      MLS_DForm_R_SI34_RTA5_MEM_p<38, (outs), (ins gprc:$RST, (memri34 $D, $RA):$addr),
                                  (ins gprc:$RST, (memri34_pcrel $D, $RA):$addr),
                                  (ins gprc:$RST, s34imm64_pcrel:$D),
                                  "pstb $RST, $addr", "pstb $RST, $D", IIC_LdStLFD>;
    defm PSTH :
      MLS_DForm_R_SI34_RTA5_MEM_p<44, (outs), (ins gprc:$RST, (memri34 $D, $RA):$addr),
                                  (ins gprc:$RST, (memri34_pcrel $D, $RA):$addr),
                                  (ins gprc:$RST, s34imm64_pcrel:$D),
                                  "psth $RST, $addr", "psth $RST, $D", IIC_LdStLFD>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 770-782

```tablegen
    defm PSTW :
      MLS_DForm_R_SI34_RTA5_MEM_p<36, (outs), (ins gprc:$RST, (memri34 $D, $RA):$addr),
                                  (ins gprc:$RST, (memri34_pcrel $D, $RA):$addr),
                                  (ins gprc:$RST, s34imm64_pcrel:$D),
                                  "pstw $RST, $addr", "pstw $RST, $D", IIC_LdStLFD>;
    defm PSTD :
      8LS_DForm_R_SI34_RTA5_MEM_p<61, (outs), (ins g8rc:$RST, (memri34 $D, $RA):$addr),
                                  (ins g8rc:$RST, (memri34_pcrel $D, $RA):$addr),
                                  (ins g8rc:$RST, s34imm64_pcrel:$D),
                                  "pstd $RST, $addr", "pstd $RST, $D", IIC_LdStLFD>;
  }
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 783-789

```tablegen
let Predicates = [PrefixInstrs, HasFPU] in {
  let mayLoad = 1, mayStore = 0 in {
    defm PLFS :
      MLS_DForm_R_SI34_RTA5_MEM_p<48, (outs f4rc:$RST), (ins (memri34 $D, $RA):$addr),
                                  (ins (memri34_pcrel $D, $RA):$addr),
                                  (ins s34imm64_pcrel:$D), "plfs $RST, $addr",
                                  "plfs $RST, $D", IIC_LdStLFD>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 790-796

```tablegen
    defm PLFD :
      MLS_DForm_R_SI34_RTA5_MEM_p<50, (outs f8rc:$RST), (ins (memri34 $D, $RA):$addr),
                                  (ins  (memri34_pcrel $D, $RA):$addr),
                                  (ins s34imm64_pcrel:$D), "plfd $RST, $addr",
                                  "plfd $RST, $D", IIC_LdStLFD>;
  }
  let mayStore = 1, mayLoad = 0 in {
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 797-809

```tablegen
    defm PSTFS :
      MLS_DForm_R_SI34_RTA5_MEM_p<52, (outs), (ins f4rc:$RST, (memri34 $D, $RA):$addr),
                                  (ins f4rc:$RST, (memri34_pcrel $D, $RA):$addr),
                                  (ins f4rc:$RST, s34imm64_pcrel:$D),
                                  "pstfs $RST, $addr", "pstfs $RST, $D", IIC_LdStLFD>;
    defm PSTFD :
      MLS_DForm_R_SI34_RTA5_MEM_p<54, (outs), (ins f8rc:$RST, (memri34 $D, $RA):$addr),
                                  (ins f8rc:$RST, (memri34_pcrel $D, $RA):$addr),
                                  (ins f8rc:$RST, s34imm64_pcrel:$D),
                                  "pstfd $RST, $addr", "pstfd $RST, $D", IIC_LdStLFD>;
  }
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 810-816

```tablegen
let Predicates = [PrefixInstrs, HasP10Vector] in {
  let mayLoad = 1, mayStore = 0 in {
    defm PLXV :
      8LS_DForm_R_SI34_XT6_RA5_MEM_p<25, (outs vsrc:$XST), (ins (memri34 $D, $RA):$addr),
                                     (ins (memri34_pcrel $D, $RA):$addr),
                                     (ins s34imm64_pcrel:$D),
                                     "plxv $XST, $addr", "plxv $XST, $D", IIC_LdStLFD>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 817-829

```tablegen
    defm PLXSSP :
      8LS_DForm_R_SI34_RTA5_MEM_p<43, (outs vfrc:$RST), (ins (memri34 $D, $RA):$addr),
                                  (ins (memri34_pcrel $D, $RA):$addr),
                                  (ins s34imm64_pcrel:$D),
                                  "plxssp $RST, $addr",  "plxssp $RST, $D",
                                  IIC_LdStLFD>;
    defm PLXSD :
      8LS_DForm_R_SI34_RTA5_MEM_p<42, (outs vfrc:$RST), (ins (memri34 $D, $RA):$addr),
                                  (ins (memri34_pcrel $D, $RA):$addr),
                                  (ins s34imm64_pcrel:$D),
                                  "plxsd $RST, $addr", "plxsd $RST, $D",
                                  IIC_LdStLFD>;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 830-840

```tablegen
 let mayStore = 1, mayLoad = 0 in {
    defm PSTXV :
      8LS_DForm_R_SI34_XT6_RA5_MEM_p<27, (outs), (ins vsrc:$XST, (memri34 $D, $RA):$addr),
                                     (ins vsrc:$XST, (memri34_pcrel $D, $RA):$addr),
                                     (ins vsrc:$XST, s34imm64_pcrel:$D),
                                     "pstxv $XST, $addr", "pstxv $XST, $D", IIC_LdStLFD>;
    defm PSTXSSP :
      8LS_DForm_R_SI34_RTA5_MEM_p<47, (outs), (ins vfrc:$RST, (memri34 $D, $RA):$addr),
                                  (ins vfrc:$RST, (memri34_pcrel $D, $RA):$addr),
                                  (ins vfrc:$RST, s34imm64_pcrel:$D),
                                  "pstxssp $RST, $addr", "pstxssp $RST, $D", IIC_LdStLFD>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 841-851

```tablegen
    defm PSTXSD :
      8LS_DForm_R_SI34_RTA5_MEM_p<46, (outs), (ins vfrc:$RST, (memri34 $D, $RA):$addr),
                                  (ins vfrc:$RST, (memri34_pcrel $D, $RA):$addr),
                                  (ins vfrc:$RST, s34imm64_pcrel:$D),
                                  "pstxsd $RST, $addr", "pstxsd $RST, $D", IIC_LdStLFD>;
  }
  def XXPERMX :
    8RR_XX4Form_IMM3_XTABC6<34, 0, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB,
                            vsrc:$XC, u3imm:$IMM),
                            "xxpermx $XT, $XA, $XB, $XC, $IMM",
                            IIC_VecPerm, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 852-859

```tablegen
  def XXBLENDVB :
    8RR_XX4Form_XTABC6<33, 0, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB,
                       vsrc:$XC), "xxblendvb $XT, $XA, $XB, $XC",
                       IIC_VecGeneral, []>;
  def XXBLENDVH :
    8RR_XX4Form_XTABC6<33, 1, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB,
                       vsrc:$XC), "xxblendvh $XT, $XA, $XB, $XC",
                       IIC_VecGeneral, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 860-869

```tablegen
  def XXBLENDVW :
    8RR_XX4Form_XTABC6<33, 2, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB,
                       vsrc:$XC), "xxblendvw $XT, $XA, $XB, $XC",
                       IIC_VecGeneral, []>;
  def XXBLENDVD :
    8RR_XX4Form_XTABC6<33, 3, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB,
                       vsrc:$XC), "xxblendvd $XT, $XA, $XB, $XC",
                       IIC_VecGeneral, []>;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 870-876

```tablegen
class DQForm_XTp5_RA17_MEM<bits<6> opcode, bits<4> xo, dag OOL, dag IOL,
                           string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin>, MemriOp {
  bits<5> XTp;
  bits<5> RA;
  bits<12> DQ;
```
- **EN**: Declares a backend-facing type `DQForm_XTp5_RA17_MEM` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `DQForm_XTp5_RA17_MEM`，并勾勒出周边代码会依赖的接口或状态。

### Lines 877-885

```tablegen
  let Pattern = pattern;

  let Inst{6...9} = XTp{3...0};
  let Inst{10} = XTp{4};
  let Inst{11...15} = RA;
  let Inst{16...27} = DQ;
  let Inst{28...31} = xo;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 886-892

```tablegen
class XForm_XTp5_RAB5<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                      string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin>, XFormMemOp {
  bits<5> XTp;
  bits<5> RA;
  bits<5> RB;
```
- **EN**: Declares a backend-facing type `XForm_XTp5_RAB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XForm_XTp5_RAB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 893-901

```tablegen
  let Pattern = pattern;
  let Inst{6...9} = XTp{3...0};
  let Inst{10} = XTp{4};
  let Inst{11...15} = RA;
  let Inst{16...20} = RB;
  let Inst{21...30} = xo;
  let Inst{31} = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 902-908

```tablegen
class 8LS_DForm_R_XTp5_SI34_MEM<bits<6> opcode, dag OOL, dag IOL, string asmstr,
                                InstrItinClass itin, list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin>, MemriOp {
  bits<5> XTp;
  bits<5> RA;
  bits<34> D;
```
- **EN**: Declares a backend-facing type and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型，并勾勒出周边代码会依赖的接口或状态。

### Lines 909-917

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...10} = 0;
  let Inst{11} = PCRel;
  let Inst{12...13} = 0;
  let Inst{14...31} = D{33...16}; // Imm18

  // The instruction.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 918-927

```tablegen
  let Inst{38...41} = XTp{3...0};
  let Inst{42}    = XTp{4};
  let Inst{43...47} = RA;
  let Inst{48...63} = D{15...0};
}

multiclass 8LS_DForm_R_XTp5_SI34_MEM_p<bits<6> opcode, dag OOL,
                                       dag IOL, dag PCRel_IOL, dag PCRelOnly_IOL,
                                       string asmstr, string asmstr_pcext,
                                       InstrItinClass itin> {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 928-934

```tablegen
  def NAME : 8LS_DForm_R_XTp5_SI34_MEM<opcode, OOL, IOL,
                                       !strconcat(asmstr, ", 0"), itin, []>;
  def pc : 8LS_DForm_R_XTp5_SI34_MEM<opcode, OOL, PCRel_IOL,
                                     !strconcat(asmstr, ", 1"), itin, []>,
                                     isPCRel;
  let isAsmParserOnly = 1, hasNoSchedulingInfo = 1 in {
    def nopc : 8LS_DForm_R_XTp5_SI34_MEM<opcode, OOL, IOL, asmstr, itin, []>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。 这一段包含调度或处理器模型元数据。

### Lines 935-943

```tablegen
    let RA = 0, MemriOp = 0 in
      def onlypc : 8LS_DForm_R_XTp5_SI34_MEM<opcode, OOL, PCRelOnly_IOL,
                                             asmstr_pcext, itin, []>, isPCRel;
  }
}



// [PO AS XO2 XO]
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 944-950

```tablegen
class XForm_AT3<bits<6> opcode, bits<5> xo2, bits<10> xo, dag OOL, dag IOL,
                    string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> AT;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `XForm_AT3` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_AT3`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 951-959

```tablegen
  let Inst{6...8}  = AT;
  let Inst{9...10}  = 0;
  let Inst{11...15} = xo2;
  let Inst{16...20} = 0;
  let Inst{21...30} = xo;
  let Inst{31} = 0;
}

// X-Form: [ PO T EO UIM XO TX ]
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 960-967

```tablegen
class XForm_XT6_IMM5<bits<6> opcode, bits<5> eo, bits<10> xo, dag OOL, dag IOL,
                     string asmstr, InstrItinClass itin, list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<6> XT;
  bits<5> UIM;

  let Pattern = pattern;
```
- **EN**: Declares a backend-facing type `XForm_XT6_IMM5` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XForm_XT6_IMM5`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 968-974

```tablegen
  let Inst{6...10} = XT{4...0};
  let Inst{11...15} = eo;
  let Inst{16...20} = UIM;
  let Inst{21...30} = xo;
  let Inst{31} = XT{5};
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 975-982

```tablegen
class XX3Form_AT3_XAB6<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                           string asmstr, InstrItinClass itin,
                           list<dag> pattern>
  : I<opcode, OOL, IOL, asmstr, itin> {
  bits<3> AT;
  bits<6> XA;
  bits<6> XB;
```
- **EN**: Declares a backend-facing type `XX3Form_AT3_XAB6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX3Form_AT3_XAB6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 983-989

```tablegen
  let Pattern = pattern;

  let Inst{6...8} = AT;
  let Inst{9...10} = 0;
  let Inst{11...15} = XA{4...0};
  let Inst{16...20} = XB{4...0};
  let Inst{21...28} = xo;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 990-1005

```tablegen
  let Inst{29}    = XA{5};
  let Inst{30}    = XB{5};
  let Inst{31} = 0;
}

class MMIRR_XX3Form_XY4P2_XAB6<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                               string asmstr, InstrItinClass itin,
                               list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<3> AT;
  bits<6> XA;
  bits<6> XB;
  bits<4> XMSK;
  bits<4> YMSK;
  bits<2> PMSK;
```
- **EN**: Declares a backend-facing type `MMIRR_XX3Form_XY4P2_XAB6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MMIRR_XX3Form_XY4P2_XAB6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1006-1012

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 3;
  let Inst{8...11} = 9;
  let Inst{12...15} = 0;
  let Inst{16...17} = PMSK;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1013-1019

```tablegen
  let Inst{18...23} = 0;
  let Inst{24...27} = XMSK;
  let Inst{28...31} = YMSK;

  // The instruction.
  let Inst{38...40} = AT;
  let Inst{41...42} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1020-1027

```tablegen
  let Inst{43...47} = XA{4...0};
  let Inst{48...52} = XB{4...0};
  let Inst{53...60} = xo;
  let Inst{61} = XA{5};
  let Inst{62} = XB{5};
  let Inst{63} = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1028-1037

```tablegen
class MMIRR_XX3Form_XY4_XAB6<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                             string asmstr, InstrItinClass itin,
                             list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<3> AT;
  bits<6> XA;
  bits<6> XB;
  bits<4> XMSK;
  bits<4> YMSK;
```
- **EN**: Declares a backend-facing type `MMIRR_XX3Form_XY4_XAB6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MMIRR_XX3Form_XY4_XAB6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1038-1044

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 3;
  let Inst{8...11} = 9;
  let Inst{12...23} = 0;
  let Inst{24...27} = XMSK;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1045-1051

```tablegen
  let Inst{28...31} = YMSK;

  // The instruction.
  let Inst{38...40} = AT;
  let Inst{41...42} = 0;
  let Inst{43...47} = XA{4...0};
  let Inst{48...52} = XB{4...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1052-1067

```tablegen
  let Inst{53...60} = xo;
  let Inst{61} = XA{5};
  let Inst{62} = XB{5};
  let Inst{63} = 0;
}

class MMIRR_XX3Form_X4Y2_XAB6<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                              string asmstr, InstrItinClass itin,
                              list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<3> AT;
  bits<6> XA;
  bits<6> XB;
  bits<4> XMSK;
  bits<2> YMSK;
```
- **EN**: Declares a backend-facing type `MMIRR_XX3Form_X4Y2_XAB6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MMIRR_XX3Form_X4Y2_XAB6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1068-1074

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 3;
  let Inst{8...11} = 9;
  let Inst{12...23} = 0;
  let Inst{24...27} = XMSK;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1075-1081

```tablegen
  let Inst{28...29} = YMSK;
  let Inst{30...31} = 0;

  // The instruction.
  let Inst{38...40} = AT;
  let Inst{41...42} = 0;
  let Inst{43...47} = XA{4...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1082-1088

```tablegen
  let Inst{48...52} = XB{4...0};
  let Inst{53...60} = xo;
  let Inst{61} = XA{5};
  let Inst{62} = XB{5};
  let Inst{63} = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1089-1099

```tablegen
class MMIRR_XX3Form_XY4P8_XAB6<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                               string asmstr, InstrItinClass itin,
                               list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<3> AT;
  bits<6> XA;
  bits<6> XB;
  bits<4> XMSK;
  bits<4> YMSK;
  bits<8> PMSK;
```
- **EN**: Declares a backend-facing type `MMIRR_XX3Form_XY4P8_XAB6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MMIRR_XX3Form_XY4P8_XAB6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1100-1106

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 3;
  let Inst{8...11} = 9;
  let Inst{12...15} = 0;
  let Inst{16...23} = PMSK;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1107-1113

```tablegen
  let Inst{24...27} = XMSK;
  let Inst{28...31} = YMSK;

  // The instruction.
  let Inst{38...40} = AT;
  let Inst{41...42} = 0;
  let Inst{43...47} = XA{4...0};
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1114-1120

```tablegen
  let Inst{48...52} = XB{4...0};
  let Inst{53...60} = xo;
  let Inst{61} = XA{5};
  let Inst{62} = XB{5};
  let Inst{63} = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1121-1131

```tablegen
class MMIRR_XX3Form_XYP4_XAB6<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                              string asmstr, InstrItinClass itin,
                              list<dag> pattern>
  : PI<1, opcode, OOL, IOL, asmstr, itin> {
  bits<3> AT;
  bits<6> XA;
  bits<6> XB;
  bits<4> XMSK;
  bits<4> YMSK;
  bits<4> PMSK;
```
- **EN**: Declares a backend-facing type `MMIRR_XX3Form_XYP4_XAB6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `MMIRR_XX3Form_XYP4_XAB6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 1132-1138

```tablegen
  let Pattern = pattern;

  // The prefix.
  let Inst{6...7} = 3;
  let Inst{8...11} = 9;
  let Inst{12...15} = 0;
  let Inst{16...19} = PMSK;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1139-1145

```tablegen
  let Inst{20...23} = 0;
  let Inst{24...27} = XMSK;
  let Inst{28...31} = YMSK;

  // The instruction.
  let Inst{38...40} = AT;
  let Inst{41...42} = 0;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1146-1153

```tablegen
  let Inst{43...47} = XA{4...0};
  let Inst{48...52} = XB{4...0};
  let Inst{53...60} = xo;
  let Inst{61} = XA{5};
  let Inst{62} = XB{5};
  let Inst{63} = 0;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1154-1160

```tablegen
def Concats {
  dag VecsToVecPair0 =
    (v256i1 (REG_SEQUENCE VSRpRC, $vs0, sub_vsx1, $vs1, sub_vsx0));
  dag VecsToVecPair1 =
    (v256i1 (REG_SEQUENCE VSRpRC, $vs2, sub_vsx1, $vs3, sub_vsx0));
}
```
- **EN**: Adds declarative TableGen records such as `Concats` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Concats`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1161-1167

```tablegen
let Predicates = [PairedVectorMemops] in {
  def : Pat<(v256i1 (PPCPairBuild v4i32:$vs1, v4i32:$vs0)),
            Concats.VecsToVecPair0>;
  def : Pat<(v256i1 (int_ppc_vsx_assemble_pair v16i8:$vs1, v16i8:$vs0)),
            Concats.VecsToVecPair0>;
  def : Pat<(v4i32 (PPCPairExtractVsx vsrpevenrc:$v, 0)),
            (v4i32 (EXTRACT_SUBREG $v, sub_vsx0))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1168-1174

```tablegen
  def : Pat<(v4i32 (PPCPairExtractVsx vsrpevenrc:$v, 1)),
            (v4i32 (EXTRACT_SUBREG $v, sub_vsx1))>;

  let mayLoad = 1, mayStore = 0 in {
    def LXVP : DQForm_XTp5_RA17_MEM<6, 0, (outs vsrprc:$XTp),
                                    (ins (memrix16 $DQ, $RA):$addr), "lxvp $XTp, $addr",
                                    IIC_LdStLFD, []>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 1175-1183

```tablegen
    def LXVPX : XForm_XTp5_RAB5<31, 333, (outs vsrprc:$XTp), (ins (memrr $RA, $RB):$addr),
                                "lxvpx $XTp, $addr", IIC_LdStLFD,
                                []>;
  }

  let mayLoad = 0, mayStore = 1 in {
    def STXVP : DQForm_XTp5_RA17_MEM<6, 1, (outs), (ins vsrprc:$XTp,
                                     (memrix16 $DQ, $RA):$addr), "stxvp $XTp, $addr",
                                     IIC_LdStLFD, []>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 1184-1197

```tablegen
    def STXVPX : XForm_XTp5_RAB5<31, 461, (outs), (ins vsrprc:$XTp, (memrr $RA, $RB):$addr),
                                 "stxvpx $XTp, $addr", IIC_LdStLFD,
                                 []>;
  }
}
let mayLoad = 1, mayStore = 0, Predicates = [PairedVectorMemops, PrefixInstrs, HasP10Vector] in {
  defm PLXVP :
    8LS_DForm_R_XTp5_SI34_MEM_p<58, (outs vsrprc:$XTp), (ins (memri34 $D, $RA):$addr),
                                (ins (memri34_pcrel $D, $RA):$addr),
                                (ins s34imm64_pcrel:$D),
                                "plxvp $XTp, $addr", "plxvp $XTp, $D",
                                IIC_LdStLFD>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1198-1205

```tablegen
let mayLoad = 0, mayStore = 1, Predicates = [PairedVectorMemops, PrefixInstrs, HasP10Vector] in {
  defm PSTXVP :
    8LS_DForm_R_XTp5_SI34_MEM_p<62, (outs), (ins vsrprc:$XTp, (memri34 $D, $RA):$addr),
                                (ins vsrprc:$XTp, (memri34_pcrel $D, $RA):$addr),
                                (ins vsrprc:$XTp, s34imm64_pcrel:$D),
                                "pstxvp $XTp, $addr", "pstxvp $XTp, $D", IIC_LdStLFD>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1206-1213

```tablegen
let Predicates = [PairedVectorMemops] in {
  // Intrinsics for Paired Vector Loads.
  def : Pat<(v256i1 (int_ppc_vsx_lxvp DQForm:$src)), (LXVP memrix16:$src)>;
  def : Pat<(v256i1 (int_ppc_vsx_lxvp XForm:$src)), (LXVPX XForm:$src)>;
  let Predicates = [PairedVectorMemops, PrefixInstrs, HasP10Vector] in {
    def : Pat<(v256i1 (int_ppc_vsx_lxvp PDForm:$src)), (PLXVP memri34:$src)>;
  }
  // Intrinsics for Paired Vector Stores.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Intrinsics for Paired Vector Loads.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Intrinsics for Paired Vector Loads.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 1214-1223

```tablegen
  def : Pat<(int_ppc_vsx_stxvp v256i1:$XSp, DQForm:$dst),
            (STXVP $XSp, memrix16:$dst)>;
  def : Pat<(int_ppc_vsx_stxvp v256i1:$XSp, XForm:$dst),
            (STXVPX $XSp, XForm:$dst)>;
  let Predicates = [PairedVectorMemops, PrefixInstrs, HasP10Vector] in {
    def : Pat<(int_ppc_vsx_stxvp v256i1:$XSp, PDForm:$dst),
              (PSTXVP $XSp, memri34:$dst)>;
  }
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1224-1231

```tablegen
let Predicates = [IsISA3_1] in {
  def XSCMPEQQP : X_VT5_VA5_VB5<63, 68, "xscmpeqqp", []>;
  def XSCMPGEQP : X_VT5_VA5_VB5<63, 196, "xscmpgeqp", []>;
  def XSCMPGTQP : X_VT5_VA5_VB5<63, 228, "xscmpgtqp", []>;
}

let Predicates = [PCRelativeMemops] in {
  // Load i32
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1232-1239

```tablegen
  def : Pat<(i32 (zextloadi1  (PPCmatpcreladdr PCRelForm:$ga))),
            (PLBZpc $ga, 0)>;
  def : Pat<(i32 (extloadi1  (PPCmatpcreladdr PCRelForm:$ga))),
            (PLBZpc $ga, 0)>;
  def : Pat<(i32 (zextloadi8  (PPCmatpcreladdr PCRelForm:$ga))),
            (PLBZpc $ga, 0)>;
  def : Pat<(i32 (extloadi8   (PPCmatpcreladdr PCRelForm:$ga))),
            (PLBZpc $ga, 0)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1240-1248

```tablegen
  def : Pat<(i32 (sextloadi16 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLHApc $ga, 0)>;
  def : Pat<(i32 (zextloadi16 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLHZpc $ga, 0)>;
  def : Pat<(i32 (extloadi16 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLHZpc $ga, 0)>;
  def : Pat<(i32 (load (PPCmatpcreladdr PCRelForm:$ga))), (PLWZpc $ga, 0)>;

  // Store i32
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1249-1256

```tablegen
  def : Pat<(truncstorei8 i32:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTBpc $RS, $ga, 0)>;
  def : Pat<(truncstorei16 i32:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTHpc $RS, $ga, 0)>;
  def : Pat<(store i32:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTWpc $RS, $ga, 0)>;

  // Load i64
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1257-1264

```tablegen
  def : Pat<(i64 (zextloadi1  (PPCmatpcreladdr PCRelForm:$ga))),
            (PLBZ8pc $ga, 0)>;
  def : Pat<(i64 (extloadi1  (PPCmatpcreladdr PCRelForm:$ga))),
            (PLBZ8pc $ga, 0)>;
  def : Pat<(i64 (zextloadi8  (PPCmatpcreladdr PCRelForm:$ga))),
            (PLBZ8pc $ga, 0)>;
  def : Pat<(i64 (extloadi8   (PPCmatpcreladdr PCRelForm:$ga))),
            (PLBZ8pc $ga, 0)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1265-1272

```tablegen
  def : Pat<(i64 (sextloadi16 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLHA8pc $ga, 0)>;
  def : Pat<(i64 (zextloadi16 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLHZ8pc $ga, 0)>;
  def : Pat<(i64 (extloadi16 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLHZ8pc $ga, 0)>;
  def : Pat<(i64 (zextloadi32 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLWZ8pc $ga, 0)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1273-1279

```tablegen
  def : Pat<(i64 (sextloadi32 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLWA8pc $ga, 0)>;
  def : Pat<(i64 (extloadi32 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLWZ8pc $ga, 0)>;
  def : Pat<(i64 (load (PPCmatpcreladdr PCRelForm:$ga))), (PLDpc $ga, 0)>;

  // Store i64
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1280-1289

```tablegen
  def : Pat<(truncstorei8 i64:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTB8pc $RS, $ga, 0)>;
  def : Pat<(truncstorei16 i64:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTH8pc $RS, $ga, 0)>;
  def : Pat<(truncstorei32 i64:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTW8pc $RS, $ga, 0)>;
  def : Pat<(store i64:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTDpc $RS, $ga, 0)>;

  // Atomic Load
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1290-1299

```tablegen
  def : Pat<(i32 (atomic_load_azext_8 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLBZpc $ga, 0)>;
  def : Pat<(i32 (atomic_load_azext_16 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLHZpc $ga, 0)>;
  def : Pat<(i32 (atomic_load_nonext_32 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLWZpc $ga, 0)>;
  def : Pat<(i64 (atomic_load_nonext_64 (PPCmatpcreladdr PCRelForm:$ga))),
            (PLDpc $ga, 0)>;

  // Atomic Store
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1300-1307

```tablegen
  def : Pat<(atomic_store_8 i32:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTBpc $RS, $ga, 0)>;
  def : Pat<(atomic_store_16 i32:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTHpc $RS, $ga, 0)>;
  def : Pat<(atomic_store_32 i32:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTWpc $RS, $ga, 0)>;
  def : Pat<(atomic_store_8 i64:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTB8pc $RS, $ga, 0)>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1308-1316

```tablegen
  def : Pat<(atomic_store_16 i64:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTH8pc $RS, $ga, 0)>;
  def : Pat<(atomic_store_32 i64:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTW8pc $RS, $ga, 0)>;
  def : Pat<(atomic_store_64 i64:$RS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTDpc $RS, $ga, 0)>;

  // If the PPCmatpcreladdr node is not caught by any other pattern it should be
  // caught here and turned into a paddi instruction to materialize the address.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1317-1326

```tablegen
  def : Pat<(PPCmatpcreladdr PCRelForm:$addr), (PADDI8pc 0, $addr)>;
  // PPCtlsdynamatpcreladdr node is used for TLS dynamic models to materialize
  // tls global address with paddi instruction.
  def : Pat<(PPCtlsdynamatpcreladdr PCRelForm:$addr), (PADDI8pc 0, $addr)>;
  // PPCtlslocalexecmataddr node is used for TLS local exec models to
  // materialize tls global address with paddi instruction.
  def : Pat<(PPCaddTls i64:$in, (PPCtlslocalexecmataddr tglobaltlsaddr:$addr)),
            (PADDI8 $in, $addr)>;
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PPCtlsdynamatpcreladdr node is used for TLS dynamic models to materialize".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PPCtlsdynamatpcreladdr node is used for TLS dynamic models to materialize”。

### Lines 1327-1335

```tablegen
let Predicates = [PCRelativeMemops, HasFPU] in {
  // Load f32
  def : Pat<(f32 (load (PPCmatpcreladdr PCRelForm:$addr))), (PLFSpc $addr, 0)>;

  // Store f32
  def : Pat<(store f32:$FRS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTFSpc $FRS, $ga, 0)>;

  // Load f64
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load f32". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load f32”。 这些声明会进入生成式模式匹配逻辑。

### Lines 1336-1343

```tablegen
  def : Pat<(f64 (extloadf32 (PPCmatpcreladdr PCRelForm:$addr))),
            (COPY_TO_REGCLASS (PLFSpc $addr, 0), VSFRC)>;
  def : Pat<(f64 (load (PPCmatpcreladdr PCRelForm:$addr))), (PLFDpc $addr, 0)>;

  // Store f64
  def : Pat<(store f64:$FRS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTFDpc $FRS, $ga, 0)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1344-1353

```tablegen
  def : Pat<(v4f32 (PPCldvsxlh (PPCmatpcreladdr PCRelForm:$addr))),
            (SUBREG_TO_REG (PLFDpc $addr, 0), sub_64)>;
}

let Predicates = [PCRelativeMemops, HasP10Vector] in {
  // Load f128
  def : Pat<(f128 (load (PPCmatpcreladdr PCRelForm:$addr))),
            (COPY_TO_REGCLASS (PLXVpc $addr, 0), VRRC)>;

  // Store f128
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1354-1360

```tablegen
  def : Pat<(store f128:$XS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTXVpc (COPY_TO_REGCLASS $XS, VSRC), $ga, 0)>;

  // Load v4i32
  def : Pat<(v4i32 (load (PPCmatpcreladdr PCRelForm:$addr))), (PLXVpc $addr, 0)>;

  // Store v4i32
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load v4i32".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load v4i32”。

### Lines 1361-1367

```tablegen
  def : Pat<(store v4i32:$XS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTXVpc $XS, $ga, 0)>;

  // Load v2i64
  def : Pat<(v2i64 (load (PPCmatpcreladdr PCRelForm:$addr))), (PLXVpc $addr, 0)>;

  // Store v2i64
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load v2i64".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load v2i64”。

### Lines 1368-1374

```tablegen
  def : Pat<(store v2i64:$XS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTXVpc $XS, $ga, 0)>;

  // Load v4f32
  def : Pat<(v4f32 (load (PPCmatpcreladdr PCRelForm:$addr))), (PLXVpc $addr, 0)>;

  // Store v4f32
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load v4f32".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load v4f32”。

### Lines 1375-1381

```tablegen
  def : Pat<(store v4f32:$XS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTXVpc $XS, $ga, 0)>;

  // Load v2f64
  def : Pat<(v2f64 (load (PPCmatpcreladdr PCRelForm:$addr))), (PLXVpc $addr, 0)>;

  // Store v2f64
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load v2f64".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load v2f64”。

### Lines 1382-1393

```tablegen
  def : Pat<(store v2f64:$XS, (PPCmatpcreladdr PCRelForm:$ga)),
            (PSTXVpc $XS, $ga, 0)>;

  // Special Cases For PPCstore_scal_int_from_vsr
  def : Pat<(PPCstore_scal_int_from_vsr f64:$src, (PPCmatpcreladdr PCRelForm:$dst), 8),
            (PSTXSDpc $src, $dst, 0)>;
  def : Pat<(PPCstore_scal_int_from_vsr f128:$src, (PPCmatpcreladdr PCRelForm:$dst), 8),
            (PSTXSDpc (COPY_TO_REGCLASS $src, VFRC), $dst, 0)>;
}

// XXSPLTIW/DP/32DX need extra flags to make sure the compiler does not attempt
// to spill part of the instruction when the values are similar.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1394-1403

```tablegen
let isReMaterializable = 1, isAsCheapAsAMove = 1, isMoveImm = 1, Predicates = [PrefixInstrs] in {
  def XXSPLTIW : 8RR_DForm_IMM32_XT6<32, 3, (outs vsrc:$XT),
                                     (ins i32imm:$IMM32),
                                     "xxspltiw $XT, $IMM32", IIC_VecGeneral,
                                     []>;
  def XXSPLTIDP : 8RR_DForm_IMM32_XT6<32, 2, (outs vsrc:$XT),
                                      (ins i32imm:$IMM32),
                                      "xxspltidp $XT, $IMM32", IIC_VecGeneral,
                                      [(set v2f64:$XT,
                                            (PPCxxspltidp i32:$IMM32))]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1404-1413

```tablegen
  def XXSPLTI32DX :
      8RR_DForm_IMM32_XT6_IX<32, 0, (outs vsrc:$XT),
                             (ins vsrc:$XTi, u1imm:$IX, i32imm:$IMM32),
                             "xxsplti32dx $XT, $IX, $IMM32", IIC_VecGeneral,
                             [(set v2i64:$XT,
                                   (PPCxxsplti32dx v2i64:$XTi, i32:$IX,
                                                   i32:$IMM32))]>,
                             RegConstraint<"$XTi = $XT">;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1414-1422

```tablegen
let Predicates = [IsISA3_1] in {
  def SETBC : XForm_XT5_BI5<31, 384, (outs gprc:$RST), (ins crbitrc:$BI),
                            "setbc $RST, $BI", IIC_IntCompare,
                            [(set i32:$RST, (PPCsetbc i1:$BI))]>,
                            SExt32To64, ZExt32To64;
  def SETBCR : XForm_XT5_BI5<31, 416, (outs gprc:$RST), (ins crbitrc:$BI),
                             "setbcr $RST, $BI", IIC_IntCompare,
                             [(set i32:$RST, (PPCsetbcr i1:$BI))]>,
                             SExt32To64, ZExt32To64;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1423-1429

```tablegen
  def SETNBC : XForm_XT5_BI5<31, 448, (outs gprc:$RST), (ins crbitrc:$BI),
                             "setnbc $RST, $BI", IIC_IntCompare, []>,
                             SExt32To64;
  def SETNBCR : XForm_XT5_BI5<31, 480, (outs gprc:$RST), (ins crbitrc:$BI),
                              "setnbcr $RST, $BI", IIC_IntCompare, []>,
                              SExt32To64;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1430-1436

```tablegen
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
    def SETBC8 : XForm_XT5_BI5<31, 384, (outs g8rc:$RST), (ins crbitrc:$BI),
                               "setbc $RST, $BI", IIC_IntCompare, []>,
                               SExt32To64, ZExt32To64;
    def SETBCR8 : XForm_XT5_BI5<31, 416, (outs g8rc:$RST), (ins crbitrc:$BI),
                                "setbcr $RST, $BI", IIC_IntCompare, []>,
                                SExt32To64, ZExt32To64;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1437-1444

```tablegen
    def SETNBC8 : XForm_XT5_BI5<31, 448, (outs g8rc:$RST), (ins crbitrc:$BI),
                                "setnbc $RST, $BI", IIC_IntCompare, []>,
                                SExt32To64;
    def SETNBCR8 : XForm_XT5_BI5<31, 480, (outs g8rc:$RST), (ins crbitrc:$BI),
                                 "setnbcr $RST, $BI", IIC_IntCompare, []>,
                                 SExt32To64;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1445-1452

```tablegen
  def VSLDBI : VNForm_VTAB5_SD3<22, 0, (outs vrrc:$VRT),
                                (ins vrrc:$VRA, vrrc:$VRB, u3imm:$SD),
                                "vsldbi $VRT, $VRA, $VRB, $SD",
                                IIC_VecGeneral,
                                [(set v16i8:$VRT,
                                      (int_ppc_altivec_vsldbi v16i8:$VRA,
                                                              v16i8:$VRB,
                                                              u3imm_timm:$SD))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1453-1460

```tablegen
  def VSRDBI : VNForm_VTAB5_SD3<22, 1, (outs vrrc:$VRT),
                                (ins vrrc:$VRA, vrrc:$VRB, u3imm:$SD),
                                "vsrdbi $VRT, $VRA, $VRB, $SD",
                                IIC_VecGeneral,
                                [(set v16i8:$VRT,
                                      (int_ppc_altivec_vsrdbi v16i8:$VRA,
                                                              v16i8:$VRB,
                                                              u3imm_timm:$SD))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1461-1468

```tablegen
  defm VSTRIBR : VXForm_VTB5_RCr<13, 1, (outs vrrc:$VT), (ins vrrc:$VB),
                                 "vstribr", "$VT, $VB", IIC_VecGeneral,
				 [(set v16i8:$VT,
                                       (int_ppc_altivec_vstribr v16i8:$VB))]>;
  defm VSTRIBL : VXForm_VTB5_RCr<13, 0, (outs vrrc:$VT), (ins vrrc:$VB),
                                 "vstribl", "$VT, $VB", IIC_VecGeneral,
                                 [(set v16i8:$VT,
                                       (int_ppc_altivec_vstribl v16i8:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1469-1476

```tablegen
  defm VSTRIHR : VXForm_VTB5_RCr<13, 3, (outs vrrc:$VT), (ins vrrc:$VB),
                                 "vstrihr", "$VT, $VB", IIC_VecGeneral,
                                 [(set v8i16:$VT,
                                       (int_ppc_altivec_vstrihr v8i16:$VB))]>;
  defm VSTRIHL : VXForm_VTB5_RCr<13, 2, (outs vrrc:$VT), (ins vrrc:$VB),
                                 "vstrihl", "$VT, $VB", IIC_VecGeneral,
                                 [(set v8i16:$VT,
                                       (int_ppc_altivec_vstrihl v8i16:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1477-1488

```tablegen
  def VINSW :
    VXForm_1<207, (outs vrrc:$VD), (ins vrrc:$VDi, u4imm:$VA, gprc:$VB),
             "vinsw $VD, $VB, $VA", IIC_VecGeneral,
             [(set v4i32:$VD,
                   (int_ppc_altivec_vinsw v4i32:$VDi, i32:$VB, u4imm_timm:$VA))]>,
             RegConstraint<"$VDi = $VD">;
  def VINSD :
    VXForm_1<463, (outs vrrc:$VD), (ins vrrc:$VDi, u4imm:$VA, g8rc:$VB),
             "vinsd $VD, $VB, $VA", IIC_VecGeneral,
             [(set v2i64:$VD,
                   (int_ppc_altivec_vinsd v2i64:$VDi, i64:$VB, u4imm_timm:$VA))]>,
             RegConstraint<"$VDi = $VD">;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1489-1498

```tablegen
  def VINSBVLX :
    VXForm_VTB5_RA5_ins<15, "vinsbvlx",
                        [(set v16i8:$VD,
                              (int_ppc_altivec_vinsbvlx v16i8:$VDi, i32:$VA,
                                                        v16i8:$VB))]>;
  def VINSBVRX :
    VXForm_VTB5_RA5_ins<271, "vinsbvrx",
                        [(set v16i8:$VD,
                              (int_ppc_altivec_vinsbvrx v16i8:$VDi, i32:$VA,
                                                        v16i8:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1499-1508

```tablegen
  def VINSHVLX :
    VXForm_VTB5_RA5_ins<79, "vinshvlx",
                        [(set v8i16:$VD,
                              (int_ppc_altivec_vinshvlx v8i16:$VDi, i32:$VA,
                                                        v8i16:$VB))]>;
  def VINSHVRX :
    VXForm_VTB5_RA5_ins<335, "vinshvrx",
                        [(set v8i16:$VD,
                              (int_ppc_altivec_vinshvrx v8i16:$VDi, i32:$VA,
                                                        v8i16:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1509-1518

```tablegen
  def VINSWVLX :
    VXForm_VTB5_RA5_ins<143, "vinswvlx",
                        [(set v4i32:$VD,
                              (int_ppc_altivec_vinswvlx v4i32:$VDi, i32:$VA,
                                                        v4i32:$VB))]>;
  def VINSWVRX :
    VXForm_VTB5_RA5_ins<399, "vinswvrx",
                        [(set v4i32:$VD,
                              (int_ppc_altivec_vinswvrx v4i32:$VDi, i32:$VA,
                                                        v4i32:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1519-1528

```tablegen
  def VINSBLX :
    VXForm_VRT5_RAB5_ins<527, "vinsblx",
                         [(set v16i8:$VD,
                               (int_ppc_altivec_vinsblx v16i8:$VDi, i32:$VA,
                                                        i32:$VB))]>;
  def VINSBRX :
    VXForm_VRT5_RAB5_ins<783, "vinsbrx",
                         [(set v16i8:$VD,
                               (int_ppc_altivec_vinsbrx v16i8:$VDi, i32:$VA,
                                                        i32:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1529-1538

```tablegen
  def VINSHLX :
    VXForm_VRT5_RAB5_ins<591, "vinshlx",
                         [(set v8i16:$VD,
                               (int_ppc_altivec_vinshlx v8i16:$VDi, i32:$VA,
                                                        i32:$VB))]>;
  def VINSHRX :
    VXForm_VRT5_RAB5_ins<847, "vinshrx",
                         [(set v8i16:$VD,
                               (int_ppc_altivec_vinshrx v8i16:$VDi, i32:$VA,
                                                        i32:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1539-1548

```tablegen
  def VINSWLX :
    VXForm_VRT5_RAB5_ins<655, "vinswlx",
                         [(set v4i32:$VD,
                               (int_ppc_altivec_vinswlx v4i32:$VDi, i32:$VA,
                                                        i32:$VB))]>;
  def VINSWRX :
    VXForm_VRT5_RAB5_ins<911, "vinswrx",
                         [(set v4i32:$VD,
                               (int_ppc_altivec_vinswrx v4i32:$VDi, i32:$VA,
                                                        i32:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1549-1560

```tablegen
  def VINSDLX :
    VXForm_1<719, (outs vrrc:$VD), (ins vrrc:$VDi, g8rc:$VA, g8rc:$VB),
             "vinsdlx $VD, $VA, $VB", IIC_VecGeneral,
              [(set v2i64:$VD,
                    (int_ppc_altivec_vinsdlx v2i64:$VDi, i64:$VA, i64:$VB))]>,
              RegConstraint<"$VDi = $VD">;
  def VINSDRX :
    VXForm_1<975, (outs vrrc:$VD), (ins vrrc:$VDi, g8rc:$VA, g8rc:$VB),
             "vinsdrx $VD, $VA, $VB", IIC_VecGeneral,
              [(set v2i64:$VD,
                    (int_ppc_altivec_vinsdrx v2i64:$VDi, i64:$VA, i64:$VB))]>,
              RegConstraint<"$VDi = $VD">;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1561-1570

```tablegen
  def VEXTRACTBM : VXForm_RD5_XO5_RS5<1602, 8, (outs gprc:$VD), (ins vrrc:$VB),
                                      "vextractbm $VD, $VB", IIC_VecGeneral,
                                      [(set i32:$VD,
                                      (int_ppc_altivec_vextractbm v16i8:$VB))]>,
                                      ZExt32To64;
  def VEXTRACTHM : VXForm_RD5_XO5_RS5<1602, 9, (outs gprc:$VD), (ins vrrc:$VB),
                                      "vextracthm $VD, $VB", IIC_VecGeneral,
                                      [(set i32:$VD,
                                      (int_ppc_altivec_vextracthm v8i16:$VB))]>,
                                      ZExt32To64;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1571-1580

```tablegen
  def VEXTRACTWM : VXForm_RD5_XO5_RS5<1602, 10, (outs gprc:$VD), (ins vrrc:$VB),
                                      "vextractwm $VD, $VB", IIC_VecGeneral,
                                      [(set i32:$VD,
                                      (int_ppc_altivec_vextractwm v4i32:$VB))]>,
                                      ZExt32To64;
  def VEXTRACTDM : VXForm_RD5_XO5_RS5<1602, 11, (outs gprc:$VD), (ins vrrc:$VB),
                                      "vextractdm $VD, $VB", IIC_VecGeneral,
                                      [(set i32:$VD,
                                      (int_ppc_altivec_vextractdm v2i64:$VB))]>,
                                      ZExt32To64;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1581-1588

```tablegen
  def VEXTRACTQM : VXForm_RD5_XO5_RS5<1602, 12, (outs gprc:$VD), (ins vrrc:$VB),
                                      "vextractqm $VD, $VB", IIC_VecGeneral,
                                      [(set i32:$VD,
                                      (int_ppc_altivec_vextractqm v1i128:$VB))]>;
  def VEXPANDBM : VXForm_RD5_XO5_RS5<1602, 0, (outs vrrc:$VD), (ins vrrc:$VB),
                                     "vexpandbm $VD, $VB", IIC_VecGeneral,
                                     [(set v16i8:$VD, (int_ppc_altivec_vexpandbm
                                           v16i8:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1589-1596

```tablegen
  def VEXPANDHM : VXForm_RD5_XO5_RS5<1602, 1, (outs vrrc:$VD), (ins vrrc:$VB),
                                     "vexpandhm $VD, $VB", IIC_VecGeneral,
                                     [(set v8i16:$VD, (int_ppc_altivec_vexpandhm
                                           v8i16:$VB))]>;
  def VEXPANDWM : VXForm_RD5_XO5_RS5<1602, 2, (outs vrrc:$VD), (ins vrrc:$VB),
                                     "vexpandwm $VD, $VB", IIC_VecGeneral,
                                     [(set v4i32:$VD, (int_ppc_altivec_vexpandwm
                                           v4i32:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1597-1604

```tablegen
  def VEXPANDDM : VXForm_RD5_XO5_RS5<1602, 3, (outs vrrc:$VD), (ins vrrc:$VB),
                                     "vexpanddm $VD, $VB", IIC_VecGeneral,
                                     [(set v2i64:$VD, (int_ppc_altivec_vexpanddm
                                           v2i64:$VB))]>;
  def VEXPANDQM : VXForm_RD5_XO5_RS5<1602, 4, (outs vrrc:$VD), (ins vrrc:$VB),
                                     "vexpandqm $VD, $VB", IIC_VecGeneral,
                                     [(set v1i128:$VD, (int_ppc_altivec_vexpandqm
                                           v1i128:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1605-1612

```tablegen
  def MTVSRBM : VXForm_RD5_XO5_RS5<1602, 16, (outs vrrc:$VD), (ins g8rc:$VB),
                                   "mtvsrbm $VD, $VB", IIC_VecGeneral,
                                   [(set v16i8:$VD,
                                         (int_ppc_altivec_mtvsrbm i64:$VB))]>;
  def MTVSRHM : VXForm_RD5_XO5_RS5<1602, 17, (outs vrrc:$VD), (ins g8rc:$VB),
                                   "mtvsrhm $VD, $VB", IIC_VecGeneral,
                                   [(set v8i16:$VD,
                                         (int_ppc_altivec_mtvsrhm i64:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1613-1620

```tablegen
  def MTVSRWM : VXForm_RD5_XO5_RS5<1602, 18, (outs vrrc:$VD), (ins g8rc:$VB),
                                   "mtvsrwm $VD, $VB", IIC_VecGeneral,
                                   [(set v4i32:$VD,
                                         (int_ppc_altivec_mtvsrwm i64:$VB))]>;
  def MTVSRDM : VXForm_RD5_XO5_RS5<1602, 19, (outs vrrc:$VD), (ins g8rc:$VB),
                                   "mtvsrdm $VD, $VB", IIC_VecGeneral,
                                   [(set v2i64:$VD,
                                         (int_ppc_altivec_mtvsrdm i64:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1621-1628

```tablegen
  def MTVSRQM : VXForm_RD5_XO5_RS5<1602, 20, (outs vrrc:$VD), (ins g8rc:$VB),
                                   "mtvsrqm $VD, $VB", IIC_VecGeneral,
                                   [(set v1i128:$VD,
                                         (int_ppc_altivec_mtvsrqm i64:$VB))]>;
  def MTVSRBMI : DXForm<4, 10, (outs vrrc:$RT), (ins u16imm64:$D),
                        "mtvsrbmi $RT, $D", IIC_VecGeneral,
                        [(set v16i8:$RT,
                              (int_ppc_altivec_mtvsrbm imm:$D))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1629-1638

```tablegen
  def VCNTMBB : VXForm_RD5_MP_VB5<1602, 12, (outs g8rc:$RD),
                                  (ins vrrc:$VB, u1imm:$MP),
                                  "vcntmbb $RD, $VB, $MP", IIC_VecGeneral,
                                  [(set i64:$RD, (int_ppc_altivec_vcntmbb
                                        v16i8:$VB, u1imm_timm:$MP))]>;
  def VCNTMBH : VXForm_RD5_MP_VB5<1602, 13, (outs g8rc:$RD),
                                  (ins vrrc:$VB, u1imm:$MP),
                                  "vcntmbh $RD, $VB, $MP", IIC_VecGeneral,
                                  [(set i64:$RD, (int_ppc_altivec_vcntmbh
                                        v8i16:$VB, u1imm_timm:$MP))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1639-1648

```tablegen
  def VCNTMBW : VXForm_RD5_MP_VB5<1602, 14, (outs g8rc:$RD),
                                  (ins vrrc:$VB, u1imm:$MP),
                                  "vcntmbw $RD, $VB, $MP", IIC_VecGeneral,
                                  [(set i64:$RD, (int_ppc_altivec_vcntmbw
                                        v4i32:$VB, u1imm_timm:$MP))]>;
  def VCNTMBD : VXForm_RD5_MP_VB5<1602, 15, (outs g8rc:$RD),
                                  (ins vrrc:$VB, u1imm:$MP),
                                  "vcntmbd $RD, $VB, $MP", IIC_VecGeneral,
                                  [(set i64:$RD, (int_ppc_altivec_vcntmbd
                                        v2i64:$VB, u1imm_timm:$MP))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1649-1656

```tablegen
  def VEXTDUBVLX : VAForm_1a<24, (outs vrrc:$RT),
                             (ins vrrc:$RA, vrrc:$RB, gprc:$RC),
                             "vextdubvlx $RT, $RA, $RB, $RC",
                             IIC_VecGeneral,
                             [(set v2i64:$RT,
                                   (int_ppc_altivec_vextdubvlx v16i8:$RA,
                                                               v16i8:$RB,
                                                               i32:$RC))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1657-1664

```tablegen
  def VEXTDUBVRX : VAForm_1a<25, (outs vrrc:$RT),
                             (ins vrrc:$RA, vrrc:$RB, gprc:$RC),
                             "vextdubvrx $RT, $RA, $RB, $RC",
                             IIC_VecGeneral,
                             [(set v2i64:$RT,
                                   (int_ppc_altivec_vextdubvrx v16i8:$RA,
                                                               v16i8:$RB,
                                                               i32:$RC))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1665-1672

```tablegen
  def VEXTDUHVLX : VAForm_1a<26, (outs vrrc:$RT),
                             (ins vrrc:$RA, vrrc:$RB, gprc:$RC),
                             "vextduhvlx $RT, $RA, $RB, $RC",
                             IIC_VecGeneral,
                             [(set v2i64:$RT,
                                   (int_ppc_altivec_vextduhvlx v8i16:$RA,
                                                               v8i16:$RB,
                                                               i32:$RC))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1673-1680

```tablegen
  def VEXTDUHVRX : VAForm_1a<27, (outs vrrc:$RT),
                             (ins vrrc:$RA, vrrc:$RB, gprc:$RC),
                             "vextduhvrx $RT, $RA, $RB, $RC",
                             IIC_VecGeneral,
                             [(set v2i64:$RT,
                                   (int_ppc_altivec_vextduhvrx v8i16:$RA,
                                                               v8i16:$RB,
                                                               i32:$RC))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1681-1688

```tablegen
  def VEXTDUWVLX : VAForm_1a<28, (outs vrrc:$RT),
                             (ins vrrc:$RA, vrrc:$RB, gprc:$RC),
                             "vextduwvlx $RT, $RA, $RB, $RC",
                             IIC_VecGeneral,
                             [(set v2i64:$RT,
                                   (int_ppc_altivec_vextduwvlx v4i32:$RA,
                                                               v4i32:$RB,
                                                               i32:$RC))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1689-1696

```tablegen
  def VEXTDUWVRX : VAForm_1a<29, (outs vrrc:$RT),
                             (ins vrrc:$RA, vrrc:$RB, gprc:$RC),
                             "vextduwvrx $RT, $RA, $RB, $RC",
                             IIC_VecGeneral,
                             [(set v2i64:$RT,
                                   (int_ppc_altivec_vextduwvrx v4i32:$RA,
                                                               v4i32:$RB,
                                                               i32:$RC))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1697-1704

```tablegen
  def VEXTDDVLX : VAForm_1a<30, (outs vrrc:$RT),
                            (ins vrrc:$RA, vrrc:$RB, gprc:$RC),
                            "vextddvlx $RT, $RA, $RB, $RC",
                            IIC_VecGeneral,
                            [(set v2i64:$RT,
                                  (int_ppc_altivec_vextddvlx v2i64:$RA,
                                                             v2i64:$RB,
                                                             i32:$RC))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1705-1712

```tablegen
  def VEXTDDVRX : VAForm_1a<31, (outs vrrc:$RT),
                            (ins vrrc:$RA, vrrc:$RB, gprc:$RC),
                            "vextddvrx $RT, $RA, $RB, $RC",
                            IIC_VecGeneral,
                            [(set v2i64:$RT,
                                  (int_ppc_altivec_vextddvrx v2i64:$RA,
                                                             v2i64:$RB,
                                                             i32:$RC))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1713-1720

```tablegen
   def VPDEPD : VXForm_1<1485, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vpdepd $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v2i64:$VD,
                         (int_ppc_altivec_vpdepd v2i64:$VA, v2i64:$VB))]>;
   def VPEXTD : VXForm_1<1421, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vpextd $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v2i64:$VD,
                         (int_ppc_altivec_vpextd v2i64:$VA, v2i64:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1721-1730

```tablegen
   def PDEPD : XForm_6<31, 156, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                       "pdepd $RA, $RST, $RB", IIC_IntGeneral,
                       [(set i64:$RA, (int_ppc_pdepd i64:$RST, i64:$RB))]>;
   def PEXTD : XForm_6<31, 188, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                       "pextd $RA, $RST, $RB", IIC_IntGeneral,
                       [(set i64:$RA, (int_ppc_pextd i64:$RST, i64:$RB))]>;
   def VCFUGED : VXForm_1<1357, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                          "vcfuged $VD, $VA, $VB", IIC_VecGeneral,
                          [(set v2i64:$VD,
                          (int_ppc_altivec_vcfuged v2i64:$VA, v2i64:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1731-1737

```tablegen
   def VGNB : VXForm_RD5_N3_VB5<1228, (outs g8rc:$RD), (ins vrrc:$VB, u3imm:$N),
                                "vgnb $RD, $VB, $N", IIC_VecGeneral,
                                [(set i64:$RD,
                                (int_ppc_altivec_vgnb v1i128:$VB, u3imm_timm:$N))]>;
   def CFUGED : XForm_6<31, 220, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                        "cfuged $RA, $RST, $RB", IIC_IntGeneral,
                        [(set i64:$RA, (int_ppc_cfuged i64:$RST, i64:$RB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1738-1747

```tablegen
   def XXEVAL :
     8RR_XX4Form_IMM8_XTAB6<34, 1, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB,
                            vsrc:$XC, u8imm:$IMM),
                            "xxeval $XT, $XA, $XB, $XC, $IMM", IIC_VecGeneral,
                            [(set v2i64:$XT, (int_ppc_vsx_xxeval v2i64:$XA,
                                  v2i64:$XB, v2i64:$XC, u8imm_timm:$IMM))]>;
   def VCLZDM : VXForm_1<1924, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vclzdm $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v2i64:$VD,
                         (int_ppc_altivec_vclzdm v2i64:$VA, v2i64:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1748-1755

```tablegen
   def VCTZDM : VXForm_1<1988, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vctzdm $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v2i64:$VD,
                         (int_ppc_altivec_vctzdm v2i64:$VA, v2i64:$VB))]>;
   def CNTLZDM : XForm_6<31, 59, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                         "cntlzdm $RA, $RST, $RB", IIC_IntGeneral,
                         [(set i64:$RA,
                         (int_ppc_cntlzdm i64:$RST, i64:$RB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1756-1762

```tablegen
   def CNTTZDM : XForm_6<31, 571, (outs g8rc:$RA), (ins g8rc:$RST, g8rc:$RB),
                         "cnttzdm $RA, $RST, $RB", IIC_IntGeneral,
                         [(set i64:$RA,
                         (int_ppc_cnttzdm i64:$RST, i64:$RB))]>;
   def XXGENPCVBM :
     XForm_XT6_IMM5_VB5<60, 916, (outs vsrc:$XT), (ins vrrc:$VRB, s5imm:$IMM),
                        "xxgenpcvbm $XT, $VRB, $IMM", IIC_VecGeneral, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1763-1771

```tablegen
   def XXGENPCVHM :
     XForm_XT6_IMM5_VB5<60, 917, (outs vsrc:$XT), (ins vrrc:$VRB, s5imm:$IMM),
                        "xxgenpcvhm $XT, $VRB, $IMM", IIC_VecGeneral, []>;
   def XXGENPCVWM :
     XForm_XT6_IMM5_VB5<60, 948, (outs vsrc:$XT), (ins vrrc:$VRB, s5imm:$IMM),
                        "xxgenpcvwm $XT, $VRB, $IMM", IIC_VecGeneral, []>;
   def XXGENPCVDM :
     XForm_XT6_IMM5_VB5<60, 949, (outs vsrc:$XT), (ins vrrc:$VRB, s5imm:$IMM),
                        "xxgenpcvdm $XT, $VRB, $IMM", IIC_VecGeneral, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1772-1779

```tablegen
   def VCLRLB : VXForm_1<397, (outs vrrc:$VD), (ins vrrc:$VA, gprc:$VB),
                         "vclrlb $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v16i8:$VD,
                               (int_ppc_altivec_vclrlb v16i8:$VA, i32:$VB))]>;
   def VCLRRB : VXForm_1<461, (outs vrrc:$VD), (ins vrrc:$VA, gprc:$VB),
                         "vclrrb $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v16i8:$VD,
                               (int_ppc_altivec_vclrrb v16i8:$VA, i32:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1780-1788

```tablegen
  def VMULLD : VXForm_1<457, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vmulld $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v2i64:$VD, (mul v2i64:$VA, v2i64:$VB))]>;
  def VMULHSW : VXForm_1<905, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vmulhsw $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v4i32:$VD, (mulhs v4i32:$VA, v4i32:$VB))]>;
  def VMULHUW : VXForm_1<649, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vmulhuw $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v4i32:$VD, (mulhu v4i32:$VA, v4i32:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1789-1797

```tablegen
  def VMULHSD : VXForm_1<969, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vmulhsd $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v2i64:$VD, (mulhs v2i64:$VA, v2i64:$VB))]>;
  def VMULHUD : VXForm_1<713, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vmulhud $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v2i64:$VD, (mulhu v2i64:$VA, v2i64:$VB))]>;
  def VMODSW : VXForm_1<1931, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vmodsw $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v4i32:$VD, (srem v4i32:$VA, v4i32:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1798-1806

```tablegen
  def VMODUW : VXForm_1<1675, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vmoduw $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v4i32:$VD, (urem v4i32:$VA, v4i32:$VB))]>;
  def VMODSD : VXForm_1<1995, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vmodsd $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v2i64:$VD, (srem v2i64:$VA, v2i64:$VB))]>;
  def VMODUD : VXForm_1<1739, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vmodud $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v2i64:$VD, (urem v2i64:$VA, v2i64:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1807-1815

```tablegen
  def VDIVSW : VXForm_1<395, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vdivsw $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v4i32:$VD, (sdiv v4i32:$VA, v4i32:$VB))]>;
  def VDIVUW : VXForm_1<139, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vdivuw $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v4i32:$VD, (udiv v4i32:$VA, v4i32:$VB))]>;
  def VDIVSD : VXForm_1<459, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vdivsd $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v2i64:$VD, (sdiv v2i64:$VA, v2i64:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1816-1822

```tablegen
  def VDIVUD : VXForm_1<203, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vdivud $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v2i64:$VD, (udiv v2i64:$VA, v2i64:$VB))]>;
  def VDIVESW : VXForm_1<907, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vdivesw $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v4i32:$VD, (int_ppc_altivec_vdivesw v4i32:$VA,
                               v4i32:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1823-1830

```tablegen
  def VDIVEUW : VXForm_1<651, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vdiveuw $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v4i32:$VD, (int_ppc_altivec_vdiveuw v4i32:$VA,
                               v4i32:$VB))]>;
  def VDIVESD : VXForm_1<971, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vdivesd $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v2i64:$VD, (int_ppc_altivec_vdivesd v2i64:$VA,
                               v2i64:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1831-1838

```tablegen
  def VDIVEUD : VXForm_1<715, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vdiveud $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v2i64:$VD, (int_ppc_altivec_vdiveud v2i64:$VA,
                               v2i64:$VB))]>;
  def XVTLSBB : XX2_BF3_XO5_XB6_XO9<60, 2, 475, (outs crrc:$BF), (ins vsrc:$XB),
                                    "xvtlsbb $BF, $XB", IIC_VecGeneral, []>;
  def BRH : XForm_11<31, 219, (outs gprc:$RA), (ins gprc:$RST),
                     "brh $RA, $RST", IIC_IntRotate, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1839-1847

```tablegen
  def BRW : XForm_11<31, 155, (outs gprc:$RA), (ins gprc:$RST),
                     "brw $RA, $RST", IIC_IntRotate,
                     [(set i32:$RA, (bswap i32:$RST))]>;
  let isCodeGenOnly = 1 in {
    def BRH8 : XForm_11<31, 219, (outs g8rc:$RA), (ins g8rc:$RST),
                        "brh $RA, $RST", IIC_IntRotate, []>;
    def BRW8 : XForm_11<31, 155, (outs g8rc:$RA), (ins g8rc:$RST),
                        "brw $RA, $RST", IIC_IntRotate, []>;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1848-1854

```tablegen
  def BRD : XForm_11<31, 187, (outs g8rc:$RA), (ins g8rc:$RST),
                     "brd $RA, $RST", IIC_IntRotate,
                     [(set i64:$RA, (bswap i64:$RST))]>;

  // The XFormMemOp flag for the following 8 instructions is set on
  // the instruction format.
  let mayLoad = 1, mayStore = 0 in {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The XFormMemOp flag for the following 8 instructions is set on".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The XFormMemOp flag for the following 8 instructions is set on”。

### Lines 1855-1861

```tablegen
    def LXVRBX : X_XT6_RA5_RB5<31, 13, "lxvrbx", vsrc, []>;
    def LXVRHX : X_XT6_RA5_RB5<31, 45, "lxvrhx", vsrc, []>;
    def LXVRWX : X_XT6_RA5_RB5<31, 77, "lxvrwx", vsrc, []>;
    def LXVRDX : X_XT6_RA5_RB5<31, 109, "lxvrdx", vsrc, []>;
  }

  let mayLoad = 0, mayStore = 1 in {
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1862-1871

```tablegen
    def STXVRBX : X_XS6_RA5_RB5<31, 141, "stxvrbx", vsrc, []>;
    def STXVRHX : X_XS6_RA5_RB5<31, 173, "stxvrhx", vsrc, []>;
    def STXVRWX : X_XS6_RA5_RB5<31, 205, "stxvrwx", vsrc, []>;
    def STXVRDX : X_XS6_RA5_RB5<31, 237, "stxvrdx", vsrc, []>;
  }

  def VMULESD : VXForm_1<968, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vmulesd $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v1i128:$VD, (int_ppc_altivec_vmulesd v2i64:$VA,
                               v2i64:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1872-1879

```tablegen
  def VMULEUD : VXForm_1<712, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vmuleud $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v1i128:$VD, (int_ppc_altivec_vmuleud v2i64:$VA,
                               v2i64:$VB))]>;
  def VMULOSD : VXForm_1<456, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vmulosd $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v1i128:$VD, (int_ppc_altivec_vmulosd v2i64:$VA,
                               v2i64:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1880-1887

```tablegen
  def VMULOUD : VXForm_1<200, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vmuloud $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v1i128:$VD, (int_ppc_altivec_vmuloud v2i64:$VA,
                               v2i64:$VB))]>;
  def VMSUMCUD : VAForm_1a<23, (outs vrrc:$RT), (ins vrrc:$RA, vrrc:$RB, vrrc:$RC),
                           "vmsumcud $RT, $RA, $RB, $RC", IIC_VecGeneral,
                           [(set v1i128:$RT, (int_ppc_altivec_vmsumcud
                                 v2i64:$RA, v2i64:$RB, v1i128:$RC))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1888-1897

```tablegen
  def VDIVSQ : VXForm_1<267, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vdivsq $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v1i128:$VD, (sdiv v1i128:$VA, v1i128:$VB))]>;
  def VDIVUQ : VXForm_1<11, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vdivuq $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v1i128:$VD, (udiv v1i128:$VA, v1i128:$VB))]>;
  def VDIVESQ : VXForm_1<779, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vdivesq $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v1i128:$VD, (int_ppc_altivec_vdivesq v1i128:$VA,
			       v1i128:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1898-1904

```tablegen
  def VDIVEUQ : VXForm_1<523, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                         "vdiveuq $VD, $VA, $VB", IIC_VecGeneral,
                         [(set v1i128:$VD, (int_ppc_altivec_vdiveuq v1i128:$VA,
			       v1i128:$VB))]>;
  def VCMPEQUQ : VCMP <455, "vcmpequq $VD, $VA, $VB" , v1i128>;
  def VCMPGTSQ : VCMP <903, "vcmpgtsq $VD, $VA, $VB" , v1i128>;
  def VCMPGTUQ : VCMP <647, "vcmpgtuq $VD, $VA, $VB" , v1i128>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1905-1913

```tablegen
  def VCMPEQUQ_rec : VCMP_rec <455, "vcmpequq. $VD, $VA, $VB" , v1i128>;
  def VCMPGTSQ_rec : VCMP_rec <903, "vcmpgtsq. $VD, $VA, $VB" , v1i128>;
  def VCMPGTUQ_rec : VCMP_rec <647, "vcmpgtuq. $VD, $VA, $VB" , v1i128>;
  def VMODSQ : VXForm_1<1803, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vmodsq $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v1i128:$VD, (srem v1i128:$VA, v1i128:$VB))]>;
  def VMODUQ : VXForm_1<1547, (outs vrrc:$VD), (ins vrrc:$VA, vrrc:$VB),
                        "vmoduq $VD, $VA, $VB", IIC_VecGeneral,
                        [(set v1i128:$VD, (urem v1i128:$VA, v1i128:$VB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1914-1920

```tablegen
  def VEXTSD2Q : VXForm_RD5_XO5_RS5<1538, 27, (outs vrrc:$VD), (ins vrrc:$VB),
                               "vextsd2q $VD, $VB", IIC_VecGeneral,
                               [(set v1i128:$VD, (int_ppc_altivec_vextsd2q v2i64:$VB))]>;
  def VCMPUQ : VXForm_BF3_VAB5<257, (outs crrc:$BF), (ins vrrc:$VA, vrrc:$VB),
                               "vcmpuq $BF, $VA, $VB", IIC_VecGeneral, []>;
  def VCMPSQ : VXForm_BF3_VAB5<321, (outs crrc:$BF), (ins vrrc:$VA, vrrc:$VB),
                               "vcmpsq $BF, $VA, $VB", IIC_VecGeneral, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1921-1931

```tablegen
  def VRLQNM : VX1_VT5_VA5_VB5<325, "vrlqnm",
                               [(set v1i128:$VD,
                                   (int_ppc_altivec_vrlqnm v1i128:$VA,
                                                           v1i128:$VB))]>;
  def VRLQMI : VXForm_1<69, (outs vrrc:$VD),
                        (ins vrrc:$VA, vrrc:$VB, vrrc:$VDi),
                        "vrlqmi $VD, $VA, $VB", IIC_VecFP,
                        [(set v1i128:$VD,
                          (int_ppc_altivec_vrlqmi v1i128:$VA, v1i128:$VB,
                                                  v1i128:$VDi))]>,
                        RegConstraint<"$VDi = $VD">;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1932-1938

```tablegen
  def VSLQ : VX1_VT5_VA5_VB5<261, "vslq", []>;
  def VSRAQ : VX1_VT5_VA5_VB5<773, "vsraq", []>;
  def VSRQ : VX1_VT5_VA5_VB5<517, "vsrq", 
                            [(set v4i32:$VD, (PPCvsrq v4i32:$VA, v4i32:$VB))]>;
  def VRLQ : VX1_VT5_VA5_VB5<5, "vrlq", []>;
  def XSCVQPUQZ : X_VT5_XO5_VB5<63, 0, 836, "xscvqpuqz", []>;
  def XSCVQPSQZ : X_VT5_XO5_VB5<63, 8, 836, "xscvqpsqz", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1939-1945

```tablegen
  def XSCVUQQP : X_VT5_XO5_VB5<63, 3, 836, "xscvuqqp", []>;
  def XSCVSQQP : X_VT5_XO5_VB5<63, 11, 836, "xscvsqqp", []>;
  def LXVKQ : XForm_XT6_IMM5<60, 31, 360, (outs vsrc:$XT), (ins u5imm:$UIM),
                             "lxvkq $XT, $UIM", IIC_VecGeneral, []>;
}

let Predicates = [IsISA3_1, HasVSX] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1946-1953

```tablegen
  def XVCVSPBF16 : XX2_XT6_XO5_XB6<60, 17, 475, "xvcvspbf16", vsrc, []>;
  def XVCVBF16SPN : XX2_XT6_XO5_XB6<60, 16, 475, "xvcvbf16spn", vsrc, []>;
  def XSMAXCQP : X_VT5_VA5_VB5<63, 676, "xsmaxcqp",
                               [(set f128:$RST, (PPCxsmaxc f128:$RA, f128:$RB))]>;
  def XSMINCQP : X_VT5_VA5_VB5<63, 740, "xsmincqp",
                               [(set f128:$RST, (PPCxsminc f128:$RA, f128:$RB))]>;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1954-1966

```tablegen
let Predicates = [IsISA3_1] in {
  def WAITP10 : XForm_IMM2_IMM2<31, 30, (outs), (ins u2imm:$L, u2imm:$PL),
                                "wait $L $PL", IIC_LdStLoad, []>;
  def SYNCP10 : XForm_IMM3_IMM2<31, 598, (outs), (ins u3imm:$L, u2imm:$SC),
                                "sync $L, $SC", IIC_LdStSync, []>;
}

// Multiclass defining patterns for Set Boolean Extension Reverse Instructions.
// This is analogous to the CRNotPat multiclass but specifically for Power10
// and newer subtargets since the extended forms use Set Boolean instructions.
// The first two anonymous patterns defined are actually a duplicate of those
// in CRNotPat, but it is preferable to define both multiclasses as complete
// ones rather than pulling that small common section out.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Multiclass defining patterns for Set Boolean Extension Reverse Instructions.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Multiclass defining patterns for Set Boolean Extension Reverse Instructions.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 1967-1975

```tablegen
multiclass P10ReverseSetBool<dag pattern, dag result> {
  def : Pat<pattern, (crnot result)>;
  def : Pat<(not pattern), result>;

  def : Pat<(i32 (zext pattern)),
            (SETBCR result)>;
  def : Pat<(i64 (zext pattern)),
            (SETBCR8 result)>;
```
- **EN**: Adds declarative TableGen records such as `P10ReverseSetBool` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `P10ReverseSetBool`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1976-1982

```tablegen
  def : Pat<(i32 (sext pattern)),
            (SETNBCR result)>;
  def : Pat<(i64 (sext pattern)),
            (SETNBCR8 result)>;

  def : Pat<(i32 (anyext pattern)),
            (SETBCR result)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1983-1989

```tablegen
  def : Pat<(i64 (anyext pattern)),
            (SETBCR8 result)>;
}

multiclass IntSetP10RevSetBool<SDNode SetCC, ValueType Ty, PatLeaf ZExtTy,
                               ImmLeaf SExtTy, I Cmpi, I Cmpli,
                               I Cmp, I Cmpl> {
```
- **EN**: Adds declarative TableGen records such as `IntSetP10RevSetBool` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `IntSetP10RevSetBool`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1990-1997

```tablegen
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETUGE)),
                           (EXTRACT_SUBREG (Cmpl $s1, $s2), sub_lt)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETGE)),
                           (EXTRACT_SUBREG (Cmp $s1, $s2), sub_lt)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETULE)),
                           (EXTRACT_SUBREG (Cmpl $s1, $s2), sub_gt)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETLE)),
                           (EXTRACT_SUBREG (Cmp $s1, $s2), sub_gt)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1998-2004

```tablegen
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETNE)),
                           (EXTRACT_SUBREG (Cmp $s1, $s2), sub_eq)>;

  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, ZExtTy:$imm, SETUGE)),
                           (EXTRACT_SUBREG (Cmpli $s1, imm:$imm), sub_lt)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, SExtTy:$imm, SETGE)),
                           (EXTRACT_SUBREG (Cmpi $s1, imm:$imm), sub_lt)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2005-2014

```tablegen
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, ZExtTy:$imm, SETULE)),
                           (EXTRACT_SUBREG (Cmpli $s1, imm:$imm), sub_gt)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, SExtTy:$imm, SETLE)),
                           (EXTRACT_SUBREG (Cmpi $s1, imm:$imm), sub_gt)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, SExtTy:$imm, SETNE)),
                           (EXTRACT_SUBREG (Cmpi $s1, imm:$imm), sub_eq)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, ZExtTy:$imm, SETNE)),
                           (EXTRACT_SUBREG (Cmpli $s1, imm:$imm), sub_eq)>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2015-2021

```tablegen
multiclass FSetP10RevSetBool<SDNode SetCC, ValueType Ty, I FCmp> {
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETUGE)),
                           (EXTRACT_SUBREG (FCmp $s1, $s2), sub_lt)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETGE)),
                           (EXTRACT_SUBREG (FCmp $s1, $s2), sub_lt)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETULE)),
                           (EXTRACT_SUBREG (FCmp $s1, $s2), sub_gt)>;
```
- **EN**: Adds declarative TableGen records such as `FSetP10RevSetBool` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FSetP10RevSetBool`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2022-2031

```tablegen
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETLE)),
                           (EXTRACT_SUBREG (FCmp $s1, $s2), sub_gt)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETUNE)),
                           (EXTRACT_SUBREG (FCmp $s1, $s2), sub_eq)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETNE)),
                           (EXTRACT_SUBREG (FCmp $s1, $s2), sub_eq)>;
  defm : P10ReverseSetBool<(i1 (SetCC Ty:$s1, Ty:$s2, SETO)),
                           (EXTRACT_SUBREG (FCmp $s1, $s2), sub_un)>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2032-2038

```tablegen
let Predicates = [IsISA3_1] in {
  def : Pat<(i32 (zext i1:$in)),
            (SETBC $in)>;
  def : Pat<(i64 (zext i1:$in)),
            (SETBC8 $in)>;
  def : Pat<(i32 (sext i1:$in)),
            (SETNBC $in)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2039-2046

```tablegen
  def : Pat<(i64 (sext i1:$in)),
            (SETNBC8 $in)>;
  def : Pat<(i32 (anyext i1:$in)),
            (SETBC $in)>;
  def : Pat<(i64 (anyext i1:$in)),
            (SETBC8 $in)>;

  // Instantiation of the set boolean reverse patterns for 32-bit integers.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2047-2053

```tablegen
  defm : IntSetP10RevSetBool<setcc, i32, immZExt16, imm32SExt16,
                             CMPWI, CMPLWI, CMPW, CMPLW>;
  defm : P10ReverseSetBool<(i1 (setcc i32:$s1, imm:$imm, SETNE)),
                           (EXTRACT_SUBREG (CMPLWI (XORIS $s1, (HI16 imm:$imm)),
                                           (LO16 imm:$imm)), sub_eq)>;

  // Instantiation of the set boolean reverse patterns for 64-bit integers.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2054-2061

```tablegen
  defm : IntSetP10RevSetBool<setcc, i64, immZExt16, imm64SExt16,
                             CMPDI, CMPLDI, CMPD, CMPLD>;
  defm : P10ReverseSetBool<(i1 (setcc i64:$s1, imm64ZExt32:$imm, SETNE)),
                           (EXTRACT_SUBREG (CMPLDI (XORIS8 $s1, (HI16 imm:$imm)),
                                           (LO16 imm:$imm)), sub_eq)>;
}

// Instantiation of the set boolean reverse patterns for f32, f64, f128.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2062-2068

```tablegen
let Predicates = [IsISA3_1, HasFPU] in {
  defm : FSetP10RevSetBool<setcc, f32, FCMPUS>;
  defm : FSetP10RevSetBool<setcc, f64, FCMPUD>;
  defm : FSetP10RevSetBool<setcc, f128, XSCMPUQP>;
}

//---------------------------- Anonymous Patterns ----------------------------//
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2069-2075

```tablegen
let Predicates = [IsISA3_1] in {
  // Exploit vsrq instruction to optimize VSR(VSRO (input, vsro_byte_shift), vsr_bit_shift)
  // to VSRQ(input, vsrq_bit_shift)
  def : Pat<(VSRVSRO v4i32:$vA, v4i32:$vB), (VSRQ $vA, $vB)>;
  // Exploit the vector multiply high instructions using intrinsics.
  def : Pat<(v4i32 (int_ppc_altivec_vmulhsw v4i32:$vA, v4i32:$vB)),
            (v4i32 (VMULHSW $vA, $vB))>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Exploit vsrq instruction to optimize VSR(VSRO (input, vsro_byte_shift), vsr_bit_shift)". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Exploit vsrq instruction to optimize VSR(VSRO (input, vsro_byte_shift), vsr_bit_shift)”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2076-2083

```tablegen
  def : Pat<(v4i32 (int_ppc_altivec_vmulhuw v4i32:$vA, v4i32:$vB)),
            (v4i32 (VMULHUW $vA, $vB))>;
  def : Pat<(v2i64 (int_ppc_altivec_vmulhsd v2i64:$vA, v2i64:$vB)),
            (v2i64 (VMULHSD $vA, $vB))>;
  def : Pat<(v2i64 (int_ppc_altivec_vmulhud v2i64:$vA, v2i64:$vB)),
            (v2i64 (VMULHUD $vA, $vB))>;
  def : Pat<(v16i8 (int_ppc_vsx_xxgenpcvbm v16i8:$VRB, imm:$IMM)),
            (v16i8 (COPY_TO_REGCLASS (XXGENPCVBM $VRB, imm:$IMM), VRRC))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2084-2091

```tablegen
  def : Pat<(v8i16 (int_ppc_vsx_xxgenpcvhm v8i16:$VRB, imm:$IMM)),
            (v8i16 (COPY_TO_REGCLASS (XXGENPCVHM $VRB, imm:$IMM), VRRC))>;
  def : Pat<(v4i32 (int_ppc_vsx_xxgenpcvwm v4i32:$VRB, imm:$IMM)),
            (v4i32 (COPY_TO_REGCLASS (XXGENPCVWM $VRB, imm:$IMM), VRRC))>;
  def : Pat<(v2i64 (int_ppc_vsx_xxgenpcvdm v2i64:$VRB, imm:$IMM)),
            (v2i64 (COPY_TO_REGCLASS (XXGENPCVDM $VRB, imm:$IMM), VRRC))>;
  def : Pat<(i32 (int_ppc_vsx_xvtlsbb v16i8:$XB, 1)),
            (EXTRACT_SUBREG (XVTLSBB (COPY_TO_REGCLASS $XB, VSRC)), sub_lt)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2092-2099

```tablegen
  def : Pat<(i32 (int_ppc_vsx_xvtlsbb v16i8:$XB, 0)),
            (EXTRACT_SUBREG (XVTLSBB (COPY_TO_REGCLASS $XB, VSRC)), sub_eq)>;
  def : Pat<(srl (bswap i32:$RS), (i32 16)),
            (RLDICL_32 (BRH $RS), 0, 48)>;
  def : Pat<(i64 (zext (i32 (srl (bswap i32:$RS), (i32 16))))),
            (RLDICL_32_64 (BRH $RS), 0, 48)>;
  def : Pat <(v1i128 (PPClxvrzx ForceXForm:$src, 8)),
             (v1i128 (COPY_TO_REGCLASS (LXVRBX ForceXForm:$src), VRRC))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2100-2106

```tablegen
  def : Pat <(v1i128 (PPClxvrzx ForceXForm:$src, 16)),
             (v1i128 (COPY_TO_REGCLASS (LXVRHX ForceXForm:$src), VRRC))>;
  def : Pat <(v1i128 (PPClxvrzx ForceXForm:$src, 32)),
             (v1i128 (COPY_TO_REGCLASS (LXVRWX ForceXForm:$src), VRRC))>;
  def : Pat <(v1i128 (PPClxvrzx ForceXForm:$src, 64)),
             (v1i128 (COPY_TO_REGCLASS (LXVRDX ForceXForm:$src), VRRC))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2107-2113

```tablegen
  def : Pat<(v1i128 (rotl v1i128:$vA, v1i128:$vB)),
            (v1i128 (VRLQ v1i128:$vA, v1i128:$vB))>;

  def : Pat <(v2i64 (PPCxxsplti32dx v2i64:$XT, i32:$XI, i32:$IMM32)),
             (v2i64 (XXSPLTI32DX v2i64:$XT, i32:$XI, i32:$IMM32))>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2114-2120

```tablegen
let Predicates = [IsISA3_1, HasVSX] in {
  def : Pat<(v16i8 (int_ppc_vsx_xvcvspbf16 v16i8:$XA)),
            (COPY_TO_REGCLASS (XVCVSPBF16 RCCp.AToVSRC), VRRC)>;
  def : Pat<(v16i8 (int_ppc_vsx_xvcvbf16spn v16i8:$XA)),
            (COPY_TO_REGCLASS (XVCVBF16SPN RCCp.AToVSRC), VRRC)>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2121-2128

```tablegen
let AddedComplexity = 400, Predicates = [IsISA3_1, IsLittleEndian] in {
  // Store element 0 of a VSX register to memory
  def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$src, 0)), ForceXForm:$dst),
            (STXVRBX (COPY_TO_REGCLASS v16i8:$src, VSRC), ForceXForm:$dst)>;
  def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$src, 0)), ForceXForm:$dst),
            (STXVRHX (COPY_TO_REGCLASS v8i16:$src, VSRC), ForceXForm:$dst)>;
  def : Pat<(store (i32 (extractelt v4i32:$src, 0)), ForceXForm:$dst),
            (STXVRWX $src, ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2129-2135

```tablegen
  def : Pat<(store (f32 (extractelt v4f32:$src, 0)), ForceXForm:$dst),
            (STXVRWX $src, ForceXForm:$dst)>;
  def : Pat<(store (i64 (extractelt v2i64:$src, 0)), ForceXForm:$dst),
            (STXVRDX $src, ForceXForm:$dst)>;
  def : Pat<(store (f64 (extractelt v2f64:$src, 0)), ForceXForm:$dst),
            (STXVRDX $src, ForceXForm:$dst)>;
  // Load element 0 of a VSX register to memory
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2136-2143

```tablegen
  def : Pat<(v8i16 (scalar_to_vector (i32 (extloadi16 ForceXForm:$src)))),
            (v8i16 (COPY_TO_REGCLASS (LXVRHX ForceXForm:$src), VSRC))>;
  def : Pat<(v16i8 (scalar_to_vector (i32 (extloadi8 ForceXForm:$src)))),
            (v16i8 (COPY_TO_REGCLASS (LXVRBX ForceXForm:$src), VSRC))>;
  def : Pat<(store (i64 (extractelt v2i64:$A, 1)), ForceXForm:$src),
            (XFSTOREf64 (EXTRACT_SUBREG $A, sub_64), ForceXForm:$src)>;
 }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2144-2150

```tablegen
let Predicates = [IsISA3_1, IsBigEndian] in {
  def : Pat<(store (i64 (extractelt v2i64:$A, 0)), ForceXForm:$src),
            (XFSTOREf64 (EXTRACT_SUBREG $A, sub_64), ForceXForm:$src)>;
}

// FIXME: The swap is overkill when the shift amount is a constant.
// We should just fix the constant in the DAG.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FIXME: The swap is overkill when the shift amount is a constant.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FIXME: The swap is overkill when the shift amount is a constant.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2151-2159

```tablegen
let AddedComplexity = 400, Predicates = [IsISA3_1, HasVSX] in {
  def : Pat<(v1i128 (shl v1i128:$VRA, v1i128:$VRB)),
            (v1i128 (VSLQ v1i128:$VRA,
                     (XXPERMDI (COPY_TO_REGCLASS $VRB, VSRC),
                               (COPY_TO_REGCLASS $VRB, VSRC), 2)))>;
  def : Pat<(v1i128 (PPCshl v1i128:$VRA, v1i128:$VRB)),
            (v1i128 (VSLQ v1i128:$VRA,
                     (XXPERMDI (COPY_TO_REGCLASS $VRB, VSRC),
                               (COPY_TO_REGCLASS $VRB, VSRC), 2)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2160-2167

```tablegen
  def : Pat<(v1i128 (srl v1i128:$VRA, v1i128:$VRB)),
            (v1i128 (VSRQ v1i128:$VRA,
                     (XXPERMDI (COPY_TO_REGCLASS $VRB, VSRC),
                               (COPY_TO_REGCLASS $VRB, VSRC), 2)))>;
  def : Pat<(v1i128 (PPCsrl v1i128:$VRA, v1i128:$VRB)),
            (v1i128 (VSRQ v1i128:$VRA,
                     (XXPERMDI (COPY_TO_REGCLASS $VRB, VSRC),
                               (COPY_TO_REGCLASS $VRB, VSRC), 2)))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2168-2193

```tablegen
  def : Pat<(v1i128 (sra v1i128:$VRA, v1i128:$VRB)),
            (v1i128 (VSRAQ v1i128:$VRA,
                     (XXPERMDI (COPY_TO_REGCLASS $VRB, VSRC),
                               (COPY_TO_REGCLASS $VRB, VSRC), 2)))>;
  def : Pat<(v1i128 (PPCsra v1i128:$VRA, v1i128:$VRB)),
            (v1i128 (VSRAQ v1i128:$VRA,
                     (XXPERMDI (COPY_TO_REGCLASS $VRB, VSRC),
                               (COPY_TO_REGCLASS $VRB, VSRC), 2)))>;
}

// =============================================================================
// XXEVAL Instruction Pattern Definitions
// =============================================================================
//
// XXEVAL instruction performs 256 different logical operations on three vector
// operands using an 8-bit immediate value to select the operation.
// Format: xxeval XT, XA, XB, XC, IMM
// For example:
// Equivalent function A?xor(B,C):and(B,C) is performed by
// xxeval XT, XA, XB, XC, 22
//
// REGISTER CLASS CONSTRAINTS:
// - XXEVAL natively supports: VSRC register class [v4i32, v4f32, v2f64, v2i64]
// - Other vector types [v16i8, v8i16] require COPY_TO_REGCLASS to/from VRRC
// =============================================================================
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XXEVAL Instruction Pattern Definitions". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XXEVAL Instruction Pattern Definitions”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2194-2213

```tablegen
class XXEvalPattern<ValueType Vt, dag InputPattern, bits<8> Imm>
    : Pat<(Vt InputPattern),
          !if(!or(!eq(Vt, v4i32), !eq(Vt, v2i64)),
              // VSRC path: direct XXEVAL for v4i32 and v2i64
              (XXEVAL $vA, $vB, $vC, Imm),
              // VRRC path: wrap with COPY_TO_REGCLASS for other types
              (COPY_TO_REGCLASS(XXEVAL(COPY_TO_REGCLASS Vt:$vA, VSRC),
                   (COPY_TO_REGCLASS Vt:$vB, VSRC),
                   (COPY_TO_REGCLASS Vt:$vC, VSRC), Imm),
                  VRRC))> {}

// =============================================================================
// PatFrags for Bitcast-Aware Vector bitwise Operations
//
// Each PatFrags defines TWO alternatives for pattern matcher to choose:
// - Direct operation (for v4i32)
// - Bitcast operation (for other types: v2i64, v16i8, v8i16)
// =============================================================================

// Basic Binary Operations
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VSRC path: direct XXEVAL for v4i32 and v2i64". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VSRC path: direct XXEVAL for v4i32 and v2i64”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2214-2225

```tablegen
def VAnd
    : PatFrags<(ops node:$a, node:$b), [(and node:$a, node:$b),
                                        (bitconvert(and
                                            (v4i32(bitconvert node:$a)),
                                            (v4i32(bitconvert node:$b))))]>;

def VXor
    : PatFrags<(ops node:$a, node:$b), [(xor node:$a, node:$b),
                                        (bitconvert(xor
                                            (v4i32(bitconvert node:$a)),
                                            (v4i32(bitconvert node:$b))))]>;
```
- **EN**: Adds declarative TableGen records such as `VAnd`, `VXor` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VAnd`, `VXor`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2226-2236

```tablegen
def VOr : PatFrags<(ops node:$a, node:$b), [(or node:$a, node:$b),
                                            (bitconvert(or
                                                (v4i32(bitconvert node:$a)),
                                                (v4i32(bitconvert node:$b))))]>;

def VNot
    : PatFrags<(ops node:$a), [(vnot node:$a),
                               (bitconvert(vnot(v4i32(bitconvert node:$a))))]>;

// Derived bitwise operations
// Vector NOR operation (not(or))
```
- **EN**: Adds declarative TableGen records such as `VOr`, `VNot` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VOr`, `VNot`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2237-2243

```tablegen
def VNor
    : PatFrags<(ops node:$a, node:$b), [(vnot(or node:$a, node:$b)),
                                        (bitconvert(vnot(or
                                            (v4i32(bitconvert node:$a)),
                                            (v4i32(bitconvert node:$b)))))]>;

// Vector EQV operation (not(xor))
```
- **EN**: Adds declarative TableGen records such as `VNor` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VNor`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2244-2250

```tablegen
def VEqv
    : PatFrags<(ops node:$a, node:$b), [(vnot(xor node:$a, node:$b)),
                                        (bitconvert(vnot(xor
                                            (v4i32(bitconvert node:$a)),
                                            (v4i32(bitconvert node:$b)))))]>;

// Vector NAND operation (not(and))
```
- **EN**: Adds declarative TableGen records such as `VEqv` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VEqv`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2251-2266

```tablegen
def VNand
    : PatFrags<(ops node:$a, node:$b), [(vnot(and node:$a, node:$b)),
                                        (bitconvert(vnot(and
                                            (v4i32(bitconvert node:$a)),
                                            (v4i32(bitconvert node:$b)))))]>;

// =============================================================================
// XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectAnd
// This class matches the equivalent Ternary Operation: A ? f(B,C) : AND(B,C)
// and emit the corresponding xxeval instruction with the imm value.
//
// The patterns implement xxeval vector select operations where:
// - A is the selector vector
// - f(B,C) is the "true" case op on vectors B and C (XOR, NOR, EQV, or NOT)
// - AND(B,C) is the "false" case op on vectors B and C
// =============================================================================
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectAnd". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectAnd”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2267-2273

```tablegen
multiclass XXEvalTernarySelectAnd<ValueType Vt> {
  // Pattern: A ? XOR(B,C) : AND(B,C) XXEVAL immediate value: 22
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VXor Vt:$vB, Vt:$vC), (VAnd Vt:$vB, Vt:$vC)),
            22>;

  // Pattern: A ? NOR(B,C) : AND(B,C) XXEVAL immediate value: 24
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: A ? XOR(B,C) : AND(B,C) XXEVAL immediate value: 22". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: A ? XOR(B,C) : AND(B,C) XXEVAL immediate value: 22”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2274-2283

```tablegen
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VNor Vt:$vB, Vt:$vC), (VAnd Vt:$vB, Vt:$vC)),
            24>;

  // Pattern: A ? EQV(B,C) : AND(B,C) XXEVAL immediate value: 25
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VEqv Vt:$vB, Vt:$vC), (VAnd Vt:$vB, Vt:$vC)),
            25>;

  // Pattern: A ? NOT(C) : AND(B,C) XXEVAL immediate value: 26
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 2284-2305

```tablegen
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VNot Vt:$vC), (VAnd Vt:$vB, Vt:$vC)), 26>;

  // Pattern: A ? NOT(B) : AND(B,C) XXEVAL immediate value: 28
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VNot Vt:$vB), (VAnd Vt:$vB, Vt:$vC)), 28>;
}

// =============================================================================
// XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectB
// This class matches the equivalent Ternary Operation: A ? f(B,C) : B
// and emit the corresponding xxeval instruction with the imm value.
//
// The patterns implement xxeval vector select operations where:
// - A is the selector vector
// - f(B,C) is the "true" case op on vectors B and C (AND, NOR, EQV, NAND)
// - B is the "false" case operand (vector B)
//
// Note: Patterns (A? C : B) and (A? not(C) : B) are not considered
// for XXEVAL instruction (4 Cycle) as XXSEL (3 cycle) instruction performs 
// better.
// =============================================================================
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: A ? NOT(B) : AND(B,C) XXEVAL immediate value: 28". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: A ? NOT(B) : AND(B,C) XXEVAL immediate value: 28”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2306-2313

```tablegen
multiclass XXEvalTernarySelectB<ValueType Vt>{
  // Pattern: (A ? AND(B,C) : B) XXEVAL immediate value: 49
  def : XXEvalPattern<Vt, (vselect Vt:$vA, (VAnd Vt:$vB, Vt:$vC), Vt:$vB), 49>;
  // Pattern: (A ? NOR(B,C) : B) XXEVAL immediate value: 56
  def : XXEvalPattern<Vt, (vselect Vt:$vA, (VNor Vt:$vB, Vt:$vC), Vt:$vB), 56>;
  // Pattern: (A ? EQV(B,C) : B) XXEVAL immediate value: 57
  def : XXEvalPattern<Vt, (vselect Vt:$vA, (VEqv Vt:$vB, Vt:$vC), Vt:$vB), 57>;
  // Pattern: (A ? NAND(B,C) : B) XXEVAL immediate value: 62
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? AND(B,C) : B) XXEVAL immediate value: 49". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? AND(B,C) : B) XXEVAL immediate value: 49”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2314-2330

```tablegen
  def : XXEvalPattern<Vt, (vselect Vt:$vA, (VNand Vt:$vB, Vt:$vC), Vt:$vB), 62>;
}

// =============================================================================
// XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectC
// This class matches the equivalent Ternary Operation: A ? f(B,C) : C
// and emit the corresponding xxeval instruction with the imm value.
//
// The patterns implement xxeval vector select operations where:
// - A is the selector vector
// - f(B,C) is the "true" case op on vectors B and C (AND, NOR, EQV, NAND)
// - C is the "false" case operand (vector C)
//
// Note: Patterns (A? B : C) and (A? not(B) : C) are not considered
// for XXEVAL instruction (4 Cycle) as XXSEL (3 cycle) instruction performs 
// better.
// =============================================================================
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectC". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectC”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2331-2338

```tablegen
multiclass XXEvalTernarySelectC<ValueType Vt>{
  // Pattern: (A ? AND(B,C) : C) XXEVAL immediate value: 81
  def : XXEvalPattern<Vt, (vselect Vt:$vA, (VAnd Vt:$vB, Vt:$vC), Vt:$vC), 81>;
  // Pattern: (A ? NOR(B,C) : C) XXEVAL immediate value: 88
  def : XXEvalPattern<Vt, (vselect Vt:$vA, (VNor Vt:$vB, Vt:$vC), Vt:$vC), 88>;
  // Pattern: (A ? EQV(B,C) : C) XXEVAL immediate value: 89
  def : XXEvalPattern<Vt, (vselect Vt:$vA, (VEqv Vt:$vB, Vt:$vC), Vt:$vC), 89>;
  // Pattern: (A ? NAND(B,C) : C) XXEVAL immediate value: 94
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? AND(B,C) : C) XXEVAL immediate value: 81". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? AND(B,C) : C) XXEVAL immediate value: 81”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2339-2351

```tablegen
  def : XXEvalPattern<Vt, (vselect Vt:$vA, (VNand Vt:$vB, Vt:$vC), Vt:$vC), 94>;
}

// =============================================================================
// XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectXor
// This class matches the equivalent Ternary Operation: A ? f(B,C) : XOR(B,C)
// and emit the corresponding xxeval instruction with the imm value.
//
// The patterns implement xxeval vector select operations where:
// - A is the selector vector
// - f(B,C) is the "true" case op in set {B, C, AND(B,C), OR(B,C), NOR(B,C)} 
// - XOR(B,C) is the "false" case op on vectors B and C
// =============================================================================
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectXor". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectXor”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2352-2358

```tablegen
multiclass XXEvalTernarySelectXor<ValueType Vt> {
  // Pattern: A ? AND(B,C) : XOR(B,C) XXEVAL immediate value: 97
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VAnd Vt:$vB, Vt:$vC), (VXor Vt:$vB, Vt:$vC)),
            97>;
  
  // Pattern: A ? B : XOR(B,C) XXEVAL immediate value: 99
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: A ? AND(B,C) : XOR(B,C) XXEVAL immediate value: 97". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: A ? AND(B,C) : XOR(B,C) XXEVAL immediate value: 97”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2359-2368

```tablegen
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, Vt:$vB, (VXor Vt:$vB, Vt:$vC)),
            99>;
  
  // Pattern: A ? C : XOR(B,C) XXEVAL immediate value: 101
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, Vt:$vC, (VXor Vt:$vB, Vt:$vC)),
            101>;
  
  // Pattern: A ? OR(B,C) : XOR(B,C) XXEVAL immediate value: 103
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 2369-2389

```tablegen
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VOr Vt:$vB, Vt:$vC), (VXor Vt:$vB, Vt:$vC)),
            103>;
  
  // Pattern: A ? NOR(B,C) : XOR(B,C) XXEVAL immediate value: 104
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VNor Vt:$vB, Vt:$vC), (VXor Vt:$vB, Vt:$vC)),
            104>;
}

// =============================================================================
// XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectOr
// This class matches the equivalent Ternary Operation: A ? f(B,C) : OR(B,C)
// and emit the corresponding xxeval instruction with the imm value.
//
// The patterns implement xxeval vector select operations where:
// - A is the selector vector
// - f(B,C) is the "true" case op in set {B, C, AND(B,C), EQV(B,C), NOT(B), 
//   NOT(C), NAND(B,C)} 
// - OR(B,C) is the "false" case op on vectors B and C
// =============================================================================
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: A ? NOR(B,C) : XOR(B,C) XXEVAL immediate value: 104". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: A ? NOR(B,C) : XOR(B,C) XXEVAL immediate value: 104”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2390-2396

```tablegen
multiclass XXEvalTernarySelectOr<ValueType Vt> {
  // Pattern: A ? AND(B,C) : OR(B,C) XXEVAL immediate value: 113
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VAnd Vt:$vB, Vt:$vC), (VOr Vt:$vB, Vt:$vC)),
            113>;
  
  // Pattern: A ? B : OR(B,C) XXEVAL immediate value: 115
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: A ? AND(B,C) : OR(B,C) XXEVAL immediate value: 113". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: A ? AND(B,C) : OR(B,C) XXEVAL immediate value: 113”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2397-2406

```tablegen
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, Vt:$vB, (VOr Vt:$vB, Vt:$vC)),
            115>;
  
  // Pattern: A ? C : OR(B,C) XXEVAL immediate value: 117
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, Vt:$vC, (VOr Vt:$vB, Vt:$vC)),
            117>;
  
  // Pattern: A ? EQV(B,C) : OR(B,C) XXEVAL immediate value: 121
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 2407-2416

```tablegen
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VEqv Vt:$vB, Vt:$vC), (VOr Vt:$vB, Vt:$vC)),
            121>;
  
  // Pattern: A ? NOT(C) : OR(B,C) XXEVAL immediate value: 122
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VNot Vt:$vC), (VOr Vt:$vB, Vt:$vC)),
            122>;
  
  // Pattern: A ? NOT(B) : OR(B,C) XXEVAL immediate value: 124
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 2417-2437

```tablegen
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VNot Vt:$vB), (VOr Vt:$vB, Vt:$vC)),
            124>;
  
  // Pattern: A ? NAND(B,C) : OR(B,C) XXEVAL immediate value: 126
  def : XXEvalPattern<
            Vt, (vselect Vt:$vA, (VNand Vt:$vB, Vt:$vC), (VOr Vt:$vB, Vt:$vC)),
            126>;
}

// =============================================================================
// XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectNor
// This class matches the equivalent Ternary Operation: A ? f(B,C) : NOR(B,C)
// and emit the corresponding xxeval instruction with the imm value.
//
// The patterns implement xxeval vector select operations where:
// - A is the selector vector
// - f(B,C) is the "true" case op in set {B, C, AND(B,C), XOR(B,C), NOT(C),
//   NOT(B), NAND(B,C)}
// - C is the "false" case op NOR(B,C)
// =============================================================================
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: A ? NAND(B,C) : OR(B,C) XXEVAL immediate value: 126". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: A ? NAND(B,C) : OR(B,C) XXEVAL immediate value: 126”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2438-2444

```tablegen
multiclass XXEvalTernarySelectNor<ValueType Vt>{
  // Pattern: (A ? AND(B,C) : NOR(B,C)) XXEVAL immediate value: 129
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VAnd Vt:$vB, Vt:$vC), (VNor Vt:$vB, Vt:$vC)),
          129>;

  // Pattern: (A ? B : NOR(B,C)) XXEVAL immediate value: 131
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? AND(B,C) : NOR(B,C)) XXEVAL immediate value: 129". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? AND(B,C) : NOR(B,C)) XXEVAL immediate value: 129”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2445-2452

```tablegen
  def : XXEvalPattern<Vt, (vselect Vt:$vA, Vt:$vB, (VNor Vt:$vB, Vt:$vC)),131>;

  // Pattern: (A ? C : NOR(B,C)) XXEVAL immediate value: 133
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, Vt:$vC, (VNor Vt:$vB, Vt:$vC)),
          133>;

  // Pattern: (A ? XOR(B,C) : NOR(B,C)) XXEVAL immediate value: 134
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? C : NOR(B,C)) XXEVAL immediate value: 133". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? C : NOR(B,C)) XXEVAL immediate value: 133”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2453-2462

```tablegen
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VXor Vt:$vB, Vt:$vC), (VNor Vt:$vB, Vt:$vC)),
          134>;

  // Pattern: (A ? NOT(C) : NOR(B,C)) XXEVAL immediate value: 138
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VNot Vt:$vC), (VNor Vt:$vB, Vt:$vC)),
          138>;

  // Pattern: (A ? NOT(B) : NOR(B,C)) XXEVAL immediate value: 140
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 2463-2483

```tablegen
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VNot Vt:$vB), (VNor Vt:$vB, Vt:$vC)),
          140>;

  // Pattern: (A ? NAND(B,C) : NOR(B,C)) XXEVAL immediate value: 142
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VNand Vt:$vB, Vt:$vC), (VNor Vt:$vB, Vt:$vC)),
          142>;
}

// =============================================================================
// XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectEqv
// This class matches the equivalent Ternary Operation: A ? f(B,C) : EQV(B,C)
// and emit the corresponding xxeval instruction with the imm value.
//
// The patterns implement xxeval vector select operations where:
// - A is the selector vector
// - f(B,C) is the "true" case op in set {OR(B,C), NOR(B,C), NAND(B,C), NOT(B),
//   NOT(C)}
// - C is the "false" case op EQV(B,C)
// =============================================================================
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? NAND(B,C) : NOR(B,C)) XXEVAL immediate value: 142". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? NAND(B,C) : NOR(B,C)) XXEVAL immediate value: 142”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2484-2490

```tablegen
multiclass XXEvalTernarySelectEqv<ValueType Vt>{
  // Pattern: (A ? OR(B,C) : EQV(B,C)) XXEVAL immediate value: 151
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VOr Vt:$vB, Vt:$vC), (VEqv Vt:$vB, Vt:$vC)),
          151>;

  // Pattern: (A ? NOR(B,C) : EQV(B,C)) XXEVAL immediate value: 152
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? OR(B,C) : EQV(B,C)) XXEVAL immediate value: 151". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? OR(B,C) : EQV(B,C)) XXEVAL immediate value: 151”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2491-2500

```tablegen
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VNor Vt:$vB, Vt:$vC), (VEqv Vt:$vB, Vt:$vC)),
          152>;

  // Pattern: (A ? NOT(C) : EQV(B,C)) XXEVAL immediate value: 154
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VNot Vt:$vC), (VEqv Vt:$vB, Vt:$vC)),
          154>;

  // Pattern: (A ? NAND(B,C) : EQV(B,C)) XXEVAL immediate value: 158
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 2501-2516

```tablegen
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VNand Vt:$vB, Vt:$vC), (VEqv Vt:$vB, Vt:$vC)),
          158>;
}

// =============================================================================
// XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectNotC
// This class matches the equivalent Ternary Operation: A ? f(B,C) : NOT(C)
// and emit the corresponding xxeval instruction with the imm value.
//
// The patterns implement xxeval vector select operations where:
// - A is the selector vector
// - f(B,C) is the "true" case op in set {AND(B,C), OR(B,C), XOR(B,C), NAND(B,C),
//   B, NOT(B)}
// - C is the "false" case op NOT(C)
// =============================================================================
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectNotC". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectNotC”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2517-2525

```tablegen
multiclass XXEvalTernarySelectNotC<ValueType Vt>{
  // Pattern: (A ? AND(B,C) : NOT(C)) XXEVAL immediate value: 161
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VAnd Vt:$vB, Vt:$vC), (VNot Vt:$vC)), 161>;

  // Pattern: (A ? B : NOT(C)) XXEVAL immediate value: 163
  def : XXEvalPattern<Vt, (vselect Vt:$vA, Vt:$vB, (VNot Vt:$vC)), 163>;

  // Pattern: (A ? XOR(B,C) : NOT(C)) XXEVAL immediate value: 166
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? AND(B,C) : NOT(C)) XXEVAL immediate value: 161". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? AND(B,C) : NOT(C)) XXEVAL immediate value: 161”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2526-2533

```tablegen
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VXor Vt:$vB, Vt:$vC), (VNot Vt:$vC)), 166>;

  // Pattern: (A ? OR(B,C) : NOT(C)) XXEVAL immediate value: 167
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VOr Vt:$vB, Vt:$vC), (VNot Vt:$vC)), 167>;
  
  // Pattern: (A ? NOT(B) : NOT(C)) XXEVAL immediate value: 172
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? OR(B,C) : NOT(C)) XXEVAL immediate value: 167". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? OR(B,C) : NOT(C)) XXEVAL immediate value: 167”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2534-2551

```tablegen
  def : XXEvalPattern<Vt, (vselect Vt:$vA, (VNot Vt:$vB), (VNot Vt:$vC)), 172>;

  // Pattern: (A ? NAND(B,C) : NOT(C)) XXEVAL immediate value: 174
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VNand Vt:$vB, Vt:$vC), (VNot Vt:$vC)), 174>;
}

// =============================================================================
// XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectNotB
// This class matches the equivalent Ternary Operation: A ? f(B,C) : NOT(B)
// and emit the corresponding xxeval instruction with the imm value.
//
// The patterns implement xxeval vector select operations where:
// - A is the selector vector
// - f(B,C) is the "true" case op in set {AND(B,C), OR(B,C), XOR(B,C), NAND(B,C),
//   C, NOT(B)}
// - C is the "false" case op NOT(B)
// =============================================================================
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? NAND(B,C) : NOT(C)) XXEVAL immediate value: 174". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? NAND(B,C) : NOT(C)) XXEVAL immediate value: 174”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2552-2560

```tablegen
multiclass XXEvalTernarySelectNotB<ValueType Vt>{
  // Pattern: (A ? AND(B,C) : NOT(B)) XXEVAL immediate value: 193
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VAnd Vt:$vB, Vt:$vC), (VNot Vt:$vB)), 193>;

  // Pattern: (A ? C : NOT(B)) XXEVAL immediate value: 197
  def : XXEvalPattern<Vt, (vselect Vt:$vA, Vt:$vC, (VNot Vt:$vB)), 197>;

  // Pattern: (A ? XOR(B,C) : NOT(B)) XXEVAL immediate value: 198
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? AND(B,C) : NOT(B)) XXEVAL immediate value: 193". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? AND(B,C) : NOT(B)) XXEVAL immediate value: 193”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2561-2568

```tablegen
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VXor Vt:$vB, Vt:$vC), (VNot Vt:$vB)), 198>;

  // Pattern: (A ? OR(B,C) : NOT(B)) XXEVAL immediate value: 199
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VOr Vt:$vB, Vt:$vC), (VNot Vt:$vB)), 199>;
  
  // Pattern: (A ? NOT(C) : NOT(B)) XXEVAL immediate value: 202
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? OR(B,C) : NOT(B)) XXEVAL immediate value: 199". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? OR(B,C) : NOT(B)) XXEVAL immediate value: 199”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2569-2585

```tablegen
  def : XXEvalPattern<Vt, (vselect Vt:$vA, (VNot Vt:$vC), (VNot Vt:$vB)), 202>;

  // Pattern: (A ? NAND(B,C) : NOT(B)) XXEVAL immediate value: 206
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VNand Vt:$vB, Vt:$vC), (VNot Vt:$vB)), 206>;
}

// =============================================================================
// XXEVAL Ternary Pattern Multiclass: XXEvalTernarySelectNand
// This class matches the equivalent Ternary Operation: A ? f(B,C) : NAND(B,C)
// and emit the corresponding xxeval instruction with the imm value.
//
// The patterns implement xxeval vector select operations where:
// - A is the selector vector
// - f(B,C) is the "true" case op in set {B, C, XOR(B,C), OR(B,C), EQV(B,C)}
// - C is the "false" case op NAND(B,C)
// =============================================================================
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? NAND(B,C) : NOT(B)) XXEVAL immediate value: 206". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? NAND(B,C) : NOT(B)) XXEVAL immediate value: 206”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2586-2595

```tablegen
multiclass XXEvalTernarySelectNand<ValueType Vt>{
  // Pattern: (A ? B : NAND(B,C)) XXEVAL immediate value: 227
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, Vt:$vB, (VNand Vt:$vB, Vt:$vC)), 227>;

  // Pattern: (A ? C : NAND(B,C)) XXEVAL immediate value: 229
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, Vt:$vC, (VNand Vt:$vB, Vt:$vC)), 229>;

  // Pattern: (A ? XOR(B,C) : NAND(B,C)) XXEVAL immediate value: 230
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern: (A ? B : NAND(B,C)) XXEVAL immediate value: 227". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern: (A ? B : NAND(B,C)) XXEVAL immediate value: 227”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2596-2605

```tablegen
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VXor Vt:$vB, Vt:$vC), (VNand Vt:$vB, Vt:$vC)),
          230>;

  // Pattern: (A ? OR(B,C) : NAND(B,C)) XXEVAL immediate value: 231
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VOr Vt:$vB, Vt:$vC), (VNand Vt:$vB, Vt:$vC)),
          231>;

  // Pattern: (A ? EQV(B,C) : NAND(B,C)) XXEVAL immediate value: 233
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 2606-2612

```tablegen
  def : XXEvalPattern<
          Vt, (vselect Vt:$vA, (VEqv Vt:$vB, Vt:$vC), (VNand Vt:$vB, Vt:$vC)),
          233>;
}

let Predicates = [PrefixInstrs, HasP10Vector] in {
  let AddedComplexity = 400 in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2613-2620

```tablegen
    def : Pat<(v4i32 (build_vector i32immNonAllOneNonZero:$A,
				   i32immNonAllOneNonZero:$A,
                                   i32immNonAllOneNonZero:$A,
                                   i32immNonAllOneNonZero:$A)),
              (v4i32 (XXSPLTIW imm:$A))>;
    def : Pat<(f32 nzFPImmAsi32:$A),
              (COPY_TO_REGCLASS (XXSPLTIDP (getFPAs32BitInt fpimm:$A)),
                              VSFRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2621-2631

```tablegen
    def : Pat<(f64 nzFPImmAsi32:$A),
              (COPY_TO_REGCLASS (XXSPLTIDP (getFPAs32BitInt fpimm:$A)),
                              VSFRC)>;

    // To replace constant pool with XXSPLTI32DX for scalars.
    def : Pat<(f32 nzFPImmAsi64:$A),
              (COPY_TO_REGCLASS (XXSPLTI32DX (XXSPLTI32DX(IMPLICIT_DEF), 0,
                                             (getFPAs64BitIntHi $A)),
                                             1, (getFPAs64BitIntLo $A)),
                                VSSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2632-2640

```tablegen
    def : Pat<(f64 nzFPImmAsi64:$A),
              (COPY_TO_REGCLASS (XXSPLTI32DX (XXSPLTI32DX (IMPLICIT_DEF), 0,
                                             (getFPAs64BitIntHi $A)),
                                             1, (getFPAs64BitIntLo $A)),
                                 VSFRC)>;

    // Anonymous patterns for XXEVAL
    // AND
    // and(A, B, C)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Anonymous patterns for XXEVAL".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Anonymous patterns for XXEVAL”。

### Lines 2641-2648

```tablegen
    def : XXEvalPattern<v4i32, (and v4i32:$vA, (and v4i32:$vB, v4i32:$vC)), 1>;
    // and(A, xor(B, C))
    def : XXEvalPattern<v4i32, (and v4i32:$vA, (xor v4i32:$vB, v4i32:$vC)), 6>;
    // and(A, or(B, C))
    def : XXEvalPattern<v4i32, (and v4i32:$vA, (or v4i32:$vB, v4i32:$vC)), 7>;
    // and(A, nor(B, C))
    def : XXEvalPattern<v4i32, (and v4i32:$vA, (vnot (or v4i32:$vB, v4i32:$vC))), 8>;
    // and(A, eqv(B, C))
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "and(A, xor(B, C))". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“and(A, xor(B, C))”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2649-2657

```tablegen
    def : XXEvalPattern<v4i32, (and v4i32:$vA, (vnot (xor v4i32:$vB, v4i32:$vC))), 9>;
    // and(A, nand(B, C))
    def : XXEvalPattern<v4i32, (and v4i32:$vA, (vnot (and v4i32:$vB, v4i32:$vC))), 14>;

    // NAND
    // nand(A, B, C)
    def : XXEvalPattern<v4i32, (vnot (and v4i32:$vA, (and v4i32:$vB, v4i32:$vC))),
                         !sub(255, 1)>;
    // nand(A, xor(B, C))
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "and(A, nand(B, C))". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“and(A, nand(B, C))”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2658-2666

```tablegen
    def : XXEvalPattern<v4i32, (vnot (and v4i32:$vA, (xor v4i32:$vB, v4i32:$vC))),
                         !sub(255, 6)>;
    // nand(A, or(B, C))
    def : XXEvalPattern<v4i32, (vnot (and v4i32:$vA, (or v4i32:$vB, v4i32:$vC))),
                         !sub(255, 7)>;
    // nand(A, nor(B, C))
    def : XXEvalPattern<v4i32, (or (vnot v4i32:$vA), (or v4i32:$vB, v4i32:$vC)),
                         !sub(255, 8)>;
    // nand(A, eqv(B, C))
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "nand(A, or(B, C))". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“nand(A, or(B, C))”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2667-2674

```tablegen
    def : XXEvalPattern<v4i32, (or (vnot v4i32:$vA), (xor v4i32:$vB, v4i32:$vC)),
                         !sub(255, 9)>;
    // nand(A, nand(B, C))
    def : XXEvalPattern<v4i32, (or (vnot v4i32:$vA), (and v4i32:$vB, v4i32:$vC)),
                         !sub(255, 14)>;

    // EQV
    // (eqv A, B, C)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "nand(A, nand(B, C))". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“nand(A, nand(B, C))”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2675-2684

```tablegen
    def : XXEvalPattern<v4i32, (or (and v4i32:$vA, (and v4i32:$vB, v4i32:$vC)),
                            (vnot (or v4i32:$vA, (or v4i32:$vB, v4i32:$vC)))),
                         150>;
    // (eqv A, (and B, C))
    def : XXEvalPattern<v4i32, (vnot (xor v4i32:$vA, (and v4i32:$vB, v4i32:$vC))), 225>;
    // (eqv A, (or B, C))
    def : XXEvalPattern<v4i32, (vnot (xor v4i32:$vA, (or v4i32:$vB, v4i32:$vC))), 135>;

    // NOR
    // (nor A, B, C)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "(eqv A, (and B, C))". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“(eqv A, (and B, C))”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2685-2692

```tablegen
    def : XXEvalPattern<v4i32, (vnot (or v4i32:$vA, (or v4i32:$vB, v4i32:$vC))), 128>;
    // (nor A, (and B, C))
    def : XXEvalPattern<v4i32, (vnot (or v4i32:$vA, (and v4i32:$vB, v4i32:$vC))), 224>;
    // (nor A, (eqv B, C))
    def : XXEvalPattern<v4i32, (and (vnot v4i32:$vA), (xor v4i32:$vB, v4i32:$vC)), 96>;
    // (nor A, (nand B, C))
    def : XXEvalPattern<v4i32, (and (vnot v4i32:$vA), (and v4i32:$vB, v4i32:$vC)), 16>;
    // (nor A, (nor B, C))
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "(nor A, (and B, C))". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“(nor A, (and B, C))”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2693-2700

```tablegen
    def : XXEvalPattern<v4i32, (and (vnot v4i32:$vA), (or v4i32:$vB, v4i32:$vC)), 112>;
    // (nor A, (xor B, C))
    def : XXEvalPattern<v4i32, (vnot (or v4i32:$vA, (xor v4i32:$vB, v4i32:$vC))), 144>;

    // OR
    // (or A, B, C)
    def : XXEvalPattern<v4i32, (or v4i32:$vA, (or v4i32:$vB, v4i32:$vC)), 127>;
    // (or A, (and B, C))
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "(nor A, (xor B, C))". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“(nor A, (xor B, C))”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2701-2708

```tablegen
    def : XXEvalPattern<v4i32, (or v4i32:$vA, (and v4i32:$vB, v4i32:$vC)), 31>;
    // (or A, (eqv B, C))
    def : XXEvalPattern<v4i32, (or v4i32:$vA, (vnot (xor v4i32:$vB, v4i32:$vC))), 159>;
    // (or A, (nand B, C))
    def : XXEvalPattern<v4i32, (or v4i32:$vA, (vnot (and v4i32:$vB, v4i32:$vC))), 239>;
    // (or A, (nor B, C))
    def : XXEvalPattern<v4i32, (or v4i32:$vA, (vnot (or v4i32:$vB, v4i32:$vC))), 143>;
    // (or A, (xor B, C))
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "(or A, (eqv B, C))". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“(or A, (eqv B, C))”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2709-2716

```tablegen
    def : XXEvalPattern<v4i32, (or v4i32:$vA, (xor v4i32:$vB, v4i32:$vC)), 111>;

    // XOR
    // (xor A, B, C)
    def : XXEvalPattern<v4i32, (xor v4i32:$vA, (xor v4i32:$vB, v4i32:$vC)), 105>;
    // (xor A, (and B, C))
    def : XXEvalPattern<v4i32, (xor v4i32:$vA, (and v4i32:$vB, v4i32:$vC)), 30>;
    // (xor A, (or B, C))
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XOR". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XOR”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2717-2723

```tablegen
    def : XXEvalPattern<v4i32, (xor v4i32:$vA, (or v4i32:$vB, v4i32:$vC)), 120>;

    // XXEval Patterns for ternary Operations.
    foreach Ty = [v4i32, v2i64, v8i16, v16i8] in {
        defm : XXEvalTernarySelectAnd<Ty>;
        defm : XXEvalTernarySelectB<Ty>;
        defm : XXEvalTernarySelectC<Ty>;
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step. These declarations feed generated pattern-matching logic.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。 这些声明会进入生成式模式匹配逻辑。

### Lines 2724-2734

```tablegen
        defm : XXEvalTernarySelectXor<Ty>;
        defm : XXEvalTernarySelectOr<Ty>;
        defm : XXEvalTernarySelectNor<Ty>;
        defm : XXEvalTernarySelectEqv<Ty>;
        defm : XXEvalTernarySelectNotC<Ty>;
        defm : XXEvalTernarySelectNotB<Ty>;
        defm : XXEvalTernarySelectNand<Ty>;
    }

    // Anonymous patterns to select prefixed VSX loads and stores.
    // Load / Store f128
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2735-2741

```tablegen
    def : Pat<(f128 (load PDForm:$src)),
              (COPY_TO_REGCLASS (PLXV memri34:$src), VRRC)>;
    def : Pat<(store f128:$XS, PDForm:$dst),
              (PSTXV (COPY_TO_REGCLASS $XS, VSRC), memri34:$dst)>;

    // Load / Store v4i32
    def : Pat<(v4i32 (load PDForm:$src)), (PLXV memri34:$src)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2742-2748

```tablegen
    def : Pat<(store v4i32:$XS, PDForm:$dst), (PSTXV $XS, memri34:$dst)>;

    // Load / Store v2i64
    def : Pat<(v2i64 (load PDForm:$src)), (PLXV memri34:$src)>;
    def : Pat<(store v2i64:$XS, PDForm:$dst), (PSTXV $XS, memri34:$dst)>;

    // Load / Store v4f32
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load / Store v2i64".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load / Store v2i64”。

### Lines 2749-2756

```tablegen
    def : Pat<(v4f32 (load PDForm:$src)), (PLXV memri34:$src)>;
    def : Pat<(store v4f32:$XS, PDForm:$dst), (PSTXV $XS, memri34:$dst)>;

    // Load / Store v2f64
    def : Pat<(v2f64 (load PDForm:$src)), (PLXV memri34:$src)>;
    def : Pat<(store v2f64:$XS, PDForm:$dst), (PSTXV $XS, memri34:$dst)>;

    // Cases For PPCstore_scal_int_from_vsr
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load / Store v2f64".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load / Store v2f64”。

### Lines 2757-2763

```tablegen
    def : Pat<(PPCstore_scal_int_from_vsr f64:$src, PDForm:$dst, 8),
              (PSTXSD $src, PDForm:$dst)>;
    def : Pat<(PPCstore_scal_int_from_vsr f128:$src, PDForm:$dst, 8),
              (PSTXSD (COPY_TO_REGCLASS $src, VFRC), PDForm:$dst)>;
    }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2764-2774

```tablegen
  def : Pat<(i32 s34imm_pat:$imm), (PLI (getImmAs64BitInt imm:$imm))>;
  def : Pat<(i64 s34imm64_pat:$imm), (PLI8 (getImmAs64BitInt imm:$imm))>;
  def : Pat<(v16i8 (int_ppc_vsx_xxpermx v16i8:$A, v16i8:$B, v16i8:$C, u3imm_timm:$D)),
            (COPY_TO_REGCLASS (XXPERMX (COPY_TO_REGCLASS $A, VSRC),
                                       (COPY_TO_REGCLASS $B, VSRC),
                                       (COPY_TO_REGCLASS $C, VSRC), $D), VSRC)>;
  def : Pat<(v16i8 (int_ppc_vsx_xxblendvb v16i8:$A, v16i8:$B, v16i8:$C)),
            (COPY_TO_REGCLASS
                   (XXBLENDVB (COPY_TO_REGCLASS $A, VSRC),
                              (COPY_TO_REGCLASS $B, VSRC),
                              (COPY_TO_REGCLASS $C, VSRC)), VSRC)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2775-2781

```tablegen
  def : Pat<(v8i16 (int_ppc_vsx_xxblendvh v8i16:$A, v8i16:$B, v8i16:$C)),
            (COPY_TO_REGCLASS
                   (XXBLENDVH (COPY_TO_REGCLASS $A, VSRC),
                              (COPY_TO_REGCLASS $B, VSRC),
                              (COPY_TO_REGCLASS $C, VSRC)), VSRC)>;
  def : Pat<(int_ppc_vsx_xxblendvw v4i32:$A, v4i32:$B, v4i32:$C),
            (XXBLENDVW $A, $B, $C)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2782-2788

```tablegen
  def : Pat<(int_ppc_vsx_xxblendvd v2i64:$A, v2i64:$B, v2i64:$C),
            (XXBLENDVD $A, $B, $C)>;
}

let Predicates = [PrefixInstrs] in {
  // Anonymous patterns to select prefixed loads and stores.
  // Load i32
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Anonymous patterns to select prefixed loads and stores.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Anonymous patterns to select prefixed loads and stores.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2789-2795

```tablegen
  def : Pat<(i32 (extloadi1 PDForm:$src)), (PLBZ memri34:$src)>;
  def : Pat<(i32 (zextloadi1 PDForm:$src)), (PLBZ memri34:$src)>;
  def : Pat<(i32 (extloadi8 PDForm:$src)), (PLBZ memri34:$src)>;
  def : Pat<(i32 (zextloadi8 PDForm:$src)), (PLBZ memri34:$src)>;
  def : Pat<(i32 (extloadi16 PDForm:$src)), (PLHZ memri34:$src)>;
  def : Pat<(i32 (zextloadi16 PDForm:$src)), (PLHZ memri34:$src)>;
  def : Pat<(i32 (sextloadi16 PDForm:$src)), (PLHA memri34:$src)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2796-2803

```tablegen
  def : Pat<(i32 (load PDForm:$src)), (PLWZ memri34:$src)>;

  // Store i32
  def : Pat<(truncstorei8 i32:$rS, PDForm:$dst), (PSTB gprc:$rS, memri34:$dst)>;
  def : Pat<(truncstorei16 i32:$rS, PDForm:$dst), (PSTH gprc:$rS, memri34:$dst)>;
  def : Pat<(store i32:$rS, PDForm:$dst), (PSTW gprc:$rS, memri34:$dst)>;

  // Load i64
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Store i32".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Store i32”。

### Lines 2804-2810

```tablegen
  def : Pat<(i64 (extloadi1 PDForm:$src)), (PLBZ8 memri34:$src)>;
  def : Pat<(i64 (zextloadi1 PDForm:$src)), (PLBZ8 memri34:$src)>;
  def : Pat<(i64 (extloadi8 PDForm:$src)), (PLBZ8 memri34:$src)>;
  def : Pat<(i64 (zextloadi8 PDForm:$src)), (PLBZ8 memri34:$src)>;
  def : Pat<(i64 (extloadi16 PDForm:$src)), (PLHZ8 memri34:$src)>;
  def : Pat<(i64 (zextloadi16 PDForm:$src)), (PLHZ8 memri34:$src)>;
  def : Pat<(i64 (sextloadi16 PDForm:$src)), (PLHA8 memri34:$src)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2811-2817

```tablegen
  def : Pat<(i64 (extloadi32 PDForm:$src)), (PLWZ8 memri34:$src)>;
  def : Pat<(i64 (zextloadi32 PDForm:$src)), (PLWZ8 memri34:$src)>;
  def : Pat<(i64 (sextloadi32 PDForm:$src)), (PLWA8 memri34:$src)>;
  def : Pat<(i64 (load PDForm:$src)), (PLD memri34:$src)>;

  // Store i64
  def : Pat<(truncstorei8 i64:$rS, PDForm:$dst), (PSTB8 g8rc:$rS, memri34:$dst)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2818-2824

```tablegen
  def : Pat<(truncstorei16 i64:$rS, PDForm:$dst), (PSTH8 g8rc:$rS, memri34:$dst)>;
  def : Pat<(truncstorei32 i64:$rS, PDForm:$dst), (PSTW8 g8rc:$rS, memri34:$dst)>;
  def : Pat<(store i64:$rS, PDForm:$dst), (PSTD g8rc:$rS, memri34:$dst)>;

  // Atomic Load
  def : Pat<(i32 (atomic_load_azext_8 PDForm:$src)), (PLBZ memri34:$src)>;
  def : Pat<(i32 (atomic_load_azext_16 PDForm:$src)), (PLHZ memri34:$src)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2825-2831

```tablegen
  def : Pat<(i32 (atomic_load_nonext_32 PDForm:$src)), (PLWZ memri34:$src)>;
  def : Pat<(i64 (atomic_load_nonext_64 PDForm:$src)), (PLD memri34:$src)>;

  // Atomic Store
  def : Pat<(atomic_store_8 i32:$RS, PDForm:$dst), (PSTB $RS, memri34:$dst)>;
  def : Pat<(atomic_store_16 i32:$RS, PDForm:$dst), (PSTH $RS, memri34:$dst)>;
  def : Pat<(atomic_store_32 i32:$RS, PDForm:$dst), (PSTW $RS, memri34:$dst)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2832-2840

```tablegen
  def : Pat<(atomic_store_64 i64:$RS, PDForm:$dst), (PSTD $RS, memri34:$dst)>;
}

let Predicates = [PrefixInstrs, HasFPU] in {
  // Load / Store f32
  def : Pat<(f32 (load PDForm:$src)), (PLFS memri34:$src)>;
  def : Pat<(store f32:$FRS, PDForm:$dst), (PSTFS $FRS, memri34:$dst)>;

  // Load / Store f64
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2841-2850

```tablegen
  def : Pat<(f64 (extloadf32 PDForm:$src)),
            (COPY_TO_REGCLASS (PLFS memri34:$src), VSFRC)>;
  def : Pat<(f64 (load PDForm:$src)), (PLFD memri34:$src)>;
  def : Pat<(store f64:$FRS, PDForm:$dst), (PSTFD $FRS, memri34:$dst)>;
  // Prefixed fpext to v2f64
  def : Pat<(v4f32 (PPCldvsxlh PDForm:$src)),
            (SUBREG_TO_REG (PLFD PDForm:$src), sub_64)>;

}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2851-2859

```tablegen
def InsertEltShift {
  dag Sub32 = (i32 (EXTRACT_SUBREG $rB, sub_32));
  dag Sub32Left1 = (RLWINM (EXTRACT_SUBREG $rB, sub_32), 1, 0, 30);
  dag Sub32Left2 = (RLWINM (EXTRACT_SUBREG $rB, sub_32), 2, 0, 29);
  dag Left1 = (RLWINM $rB, 1, 0, 30);
  dag Left2 = (RLWINM $rB, 2, 0, 29);
  dag Left3 = (RLWINM8 $rB, 3, 0, 28);
}
```
- **EN**: Adds declarative TableGen records such as `InsertEltShift` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `InsertEltShift`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2860-2867

```tablegen
let Predicates = [IsISA3_1, HasVSX, IsLittleEndian] in {
  // Indexed vector insert element
  def : Pat<(v16i8 (vector_insert v16i8:$vDi, i32:$rA, i64:$rB)),
            (VINSBRX $vDi, InsertEltShift.Sub32, $rA)>;
  def : Pat<(v8i16 (vector_insert v8i16:$vDi, i32:$rA, i64:$rB)),
            (VINSHRX $vDi, InsertEltShift.Sub32Left1, $rA)>;
  def : Pat<(v4i32 (insertelt v4i32:$vDi, i32:$rA, i64:$rB)),
            (VINSWRX $vDi, InsertEltShift.Sub32Left2, $rA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2868-2875

```tablegen
  def : Pat<(v2i64 (insertelt v2i64:$vDi, i64:$rA, i64:$rB)),
            (VINSDRX $vDi, InsertEltShift.Left3, $rA)>;

  def : Pat<(v4f32 (insertelt v4f32:$vDi, f32:$rA, i64:$rB)),
            (VINSWVRX $vDi, InsertEltShift.Sub32Left2, (XSCVDPSPN $rA))>;

  def : Pat<(v2f64 (insertelt v2f64:$vDi,  f64:$A, i64:$rB)),
            (VINSDRX $vDi, InsertEltShift.Left3, Bitcast.DblToLong)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2876-2883

```tablegen
  def : Pat<(v2f64 (insertelt v2f64:$vDi, (f64 (load DSForm:$rA)), i64:$rB)),
            (VINSDRX $vDi, InsertEltShift.Left3, (LD memrix:$rA))>;
  def : Pat<(v2f64 (insertelt v2f64:$vDi, (f64 (load PDForm:$rA)), i64:$rB)),
            (VINSDRX $vDi, InsertEltShift.Left3, (PLD memri34:$rA))>;
  def : Pat<(v2f64 (insertelt v2f64:$vDi, (f64 (load XForm:$rA)), i64:$rB)),
            (VINSDRX $vDi, InsertEltShift.Left3, (LDX memrr:$rA))>;
  let AddedComplexity = 400 in {
    // Immediate vector insert element
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 2884-2893

```tablegen
    foreach Idx = [0, 1, 2, 3] in {
      def : Pat<(v4i32 (insertelt v4i32:$vDi, i32:$rA, Idx)),
                (VINSW $vDi, !mul(!sub(3, Idx), 4), $rA)>;
    }
    foreach i = [0, 1] in
     def : Pat<(v2i64 (insertelt v2i64:$vDi, i64:$rA, (i64 i))),
               (VINSD $vDi, !mul(!sub(1, i), 8), $rA)>;
  }
}
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 2894-2902

```tablegen
let Predicates = [IsISA3_1, HasVSX, IsBigEndian, IsPPC32] in {
  // Indexed vector insert element
  def : Pat<(v16i8 (vector_insert v16i8:$vDi, i32:$rA, i32:$rB)),
            (VINSBLX $vDi, $rB, $rA)>;
  def : Pat<(v8i16 (vector_insert v8i16:$vDi, i32:$rA, i32:$rB)),
            (VINSHLX $vDi, InsertEltShift.Left1, $rA)>;
  def : Pat<(v4i32 (insertelt v4i32:$vDi, i32:$rA, i32:$rB)),
            (VINSWLX $vDi, InsertEltShift.Left2, $rA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2903-2910

```tablegen
  def : Pat<(v4f32 (insertelt v4f32:$vDi,  f32:$rA, i32:$rB)),
            (VINSWVLX $vDi, InsertEltShift.Left2, (XSCVDPSPN $rA))>;
}

let Predicates = [IsISA3_1, HasVSX, IsBigEndian, IsPPC64] in {
  // Indexed vector insert element
  def : Pat<(v16i8 (vector_insert v16i8:$vDi, i32:$rA, i64:$rB)),
            (VINSBLX $vDi, InsertEltShift.Sub32, $rA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2911-2917

```tablegen
  def : Pat<(v8i16 (vector_insert v8i16:$vDi, i32:$rA, i64:$rB)),
            (VINSHLX $vDi, InsertEltShift.Sub32Left1, $rA)>;
  def : Pat<(v4i32 (insertelt v4i32:$vDi, i32:$rA, i64:$rB)),
            (VINSWLX $vDi, InsertEltShift.Sub32Left2, $rA)>;
  def : Pat<(v2i64 (insertelt v2i64:$vDi, i64:$rA, i64:$rB)),
            (VINSDLX $vDi, InsertEltShift.Left3, $rA)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2918-2924

```tablegen
  def : Pat<(v4f32 (insertelt v4f32:$vDi,  f32:$rA, i64:$rB)),
            (VINSWVLX $vDi, InsertEltShift.Sub32Left2, (XSCVDPSPN $rA))>;

  def : Pat<(v2f64 (insertelt v2f64:$vDi,  f64:$A, i64:$rB)),
            (VINSDLX $vDi, InsertEltShift.Left3, Bitcast.DblToLong)>;
  def : Pat<(v2f64 (insertelt v2f64:$vDi, (f64 (load DSForm:$rA)), i64:$rB)),
            (VINSDLX $vDi, InsertEltShift.Left3, (LD memrix:$rA))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2925-2932

```tablegen
  def : Pat<(v2f64 (insertelt v2f64:$vDi, (f64 (load PDForm:$rA)), i64:$rB)),
            (VINSDLX $vDi, InsertEltShift.Left3, (PLD memri34:$rA))>;
  def : Pat<(v2f64 (insertelt v2f64:$vDi, (f64 (load XForm:$rA)), i64:$rB)),
            (VINSDLX $vDi, InsertEltShift.Left3, (LDX memrr:$rA))>;
}

let AddedComplexity = 400, Predicates = [IsISA3_1, HasVSX, IsBigEndian] in {
  // Immediate vector insert element
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2933-2939

```tablegen
  foreach Ty = [i32, i64] in {
    foreach Idx = [0, 1, 2, 3] in {
      def : Pat<(v4i32 (insertelt v4i32:$vDi, i32:$rA, (Ty Idx))),
               (VINSW $vDi, !mul(Idx, 4), $rA)>;
    }
  }
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 2940-2945

```tablegen
  foreach Idx = [0, 1] in
    def : Pat<(v2i64 (insertelt v2i64:$vDi, i64:$rA, Idx)),
              (VINSD $vDi, !mul(Idx, 8), $rA)>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2946-2952

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC ISA 3.1 Extended Mnemonics.
//

let Predicates = [IsISA3_1] in {
  def : InstAlias<"wait", (WAITP10 0, 0)>;
  def : InstAlias<"wait 0", (WAITP10 0, 0), 0>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2953-2959

```tablegen
  def : InstAlias<"wait 1", (WAITP10 1, 0), 0>;
  def : InstAlias<"waitrsv", (WAITP10 1, 0)>;
  def : InstAlias<"pause_short", (WAITP10 2, 0), 0>;

  def : InstAlias<"sync", (SYNCP10 0, 0)>;
  def : InstAlias<"hwsync", (SYNCP10 0, 0), 0>;
  def : InstAlias<"wsync", (SYNCP10 1, 0), 0>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2960-2967

```tablegen
  def : InstAlias<"ptesync", (SYNCP10 2, 0)>;
  def : InstAlias<"phwsync", (SYNCP10 4, 0)>;
  def : InstAlias<"plwsync", (SYNCP10 5, 0)>;
  def : InstAlias<"sync $L", (SYNCP10 u3imm:$L, 0)>;
  def : InstAlias<"stncisync", (SYNCP10 1, 1)>;
  def : InstAlias<"stcisync", (SYNCP10 0, 2)>;
  def : InstAlias<"stsync", (SYNCP10 0, 3)>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2968-2975

```tablegen
  def : InstAlias<"paddi $RT, $RA, $SI", (PADDI8 g8rc:$RT, g8rc_nox0:$RA, s34imm64:$SI)>;
}

let Predicates = [IsISA3_1, PrefixInstrs], isAsmParserOnly = 1, hasNoSchedulingInfo = 1 in {
  let Interpretation64Bit = 1 in {
    def PLA8 : MLS_DForm_SI34_RT5<14, (outs g8rc:$RT),
                                  (ins g8rc_nox0:$RA, s34imm64:$SI),
                                  "pla $RT, ${SI} ${RA}", IIC_IntSimple, []>, MemriOp;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2976-2983

```tablegen
    def PLA8pc : MLS_DForm_SI34_RT5<14, (outs g8rc:$RT),
                                    (ins s34imm64_pcrel:$SI),
                                    "pla $RT, $SI", IIC_IntSimple, []>, isPCRel;
  }

  def PSUBI : PPCAsmPseudo<"psubi $RT, $RA, $SI",
                           (ins g8rc:$RT, g8rc_nox0:$RA, s34imm64:$SI)>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2984-2990

```tablegen
  def PLA : MLS_DForm_SI34_RT5<14, (outs gprc:$RT),
                               (ins gprc_nor0:$RA, s34imm64:$SI),
                               "pla $RT, ${SI} ${RA}", IIC_IntSimple, []>, MemriOp;
  def PLApc : MLS_DForm_SI34_RT5<14, (outs gprc:$RT),
                                 (ins s34imm64_pcrel:$SI),
                                 "pla $RT, $SI", IIC_IntSimple, []>, isPCRel;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Assembly parsing / 汇编解析
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Atomic operation support / 原子操作支持
- Vector or SIMD behavior / 向量或 SIMD 行为

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
