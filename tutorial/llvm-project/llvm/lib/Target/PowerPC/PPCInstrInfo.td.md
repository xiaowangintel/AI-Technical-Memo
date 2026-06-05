# PPCInstrInfo.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCInstrInfo.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCInstrInfo.td - The PowerPC Instruction Set. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCInstrInfo.td`，主要负责 PowerPC 后端的指令信息与 lowering 辅助逻辑。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===-- PPCInstrInfo.td - The PowerPC Instruction Set ------*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-11

```tablegen
//===----------------------------------------------------------------------===//
//
// This file describes the subset of the 32-bit PowerPC instruction set, as used
// by the PowerPC instruction selector.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file describes the subset of the 32-bit PowerPC instruction set, as used".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file describes the subset of the 32-bit PowerPC instruction set, as used”。

### Lines 12-15

```tablegen
//===----------------------------------------------------------------------===//

include "PPCInstrFormats.td"
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 16-24

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC specific type constraints.
//
def SDT_PPCstfiwx : SDTypeProfile<0, 2, [ // stfiwx
  SDTCisVT<0, f64>, SDTCisPtrTy<1>
]>;
def SDT_PPClfiwx : SDTypeProfile<1, 1, [ // lfiw[az]x
  SDTCisVT<0, f64>, SDTCisPtrTy<1>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCstfiwx`, `SDT_PPClfiwx` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCstfiwx`, `SDT_PPClfiwx`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 25-33

```tablegen
def SDT_PPCLxsizx : SDTypeProfile<1, 2, [
  SDTCisVT<0, f64>, SDTCisPtrTy<1>, SDTCisPtrTy<2>
]>;
def SDT_PPCstxsix : SDTypeProfile<0, 3, [
  SDTCisVT<0, f64>, SDTCisPtrTy<1>, SDTCisPtrTy<2>
]>;
def SDT_PPCcv_fp_to_int  : SDTypeProfile<1, 1, [
  SDTCisFP<0>, SDTCisFP<1>
  ]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCLxsizx`, `SDT_PPCstxsix`, `SDT_PPCcv_fp_to_int` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCLxsizx`, `SDT_PPCstxsix`, `SDT_PPCcv_fp_to_int`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 34-40

```tablegen
def SDT_PPCstore_scal_int_from_vsr : SDTypeProfile<0, 3, [
  SDTCisFP<0>, SDTCisPtrTy<1>, SDTCisPtrTy<2>
]>;
def SDT_PPCVexts  : SDTypeProfile<1, 2, [
  SDTCisVT<0, f64>, SDTCisVT<1, f64>, SDTCisPtrTy<2>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCstore_scal_int_from_vsr`, `SDT_PPCVexts` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCstore_scal_int_from_vsr`, `SDT_PPCVexts`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 41-48

```tablegen
def SDT_PPCCallSeqStart : SDCallSeqStart<[ SDTCisVT<0, i32>,
                                           SDTCisVT<1, i32> ]>;
def SDT_PPCCallSeqEnd   : SDCallSeqEnd<[ SDTCisVT<0, i32>,
                                         SDTCisVT<1, i32> ]>;
def SDT_PPCvperm   : SDTypeProfile<1, 3, [
  SDTCisVT<3, v16i8>, SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCCallSeqStart`, `SDT_PPCCallSeqEnd`, `SDT_PPCvperm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCCallSeqStart`, `SDT_PPCCallSeqEnd`, `SDT_PPCvperm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 49-56

```tablegen
def SDT_PPCVecSplat : SDTypeProfile<1, 2, [ SDTCisVec<0>,
  SDTCisVec<1>, SDTCisInt<2>
]>;

def SDT_PPCSpToDp : SDTypeProfile<1, 1, [ SDTCisVT<0, v2f64>,
  SDTCisInt<1>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCVecSplat`, `SDT_PPCSpToDp` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCVecSplat`, `SDT_PPCSpToDp`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 57-64

```tablegen
def SDT_PPCVecShift : SDTypeProfile<1, 3, [ SDTCisVec<0>,
  SDTCisVec<1>, SDTCisVec<2>, SDTCisPtrTy<3>
]>;

def SDT_PPCVecShiftQuad : SDTypeProfile<1, 2, [
  SDTCisVec<0>, SDTCisSameAs<0,1>, SDTCisSameAs<0,2>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCVecShift`, `SDT_PPCVecShiftQuad` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCVecShift`, `SDT_PPCVecShiftQuad`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 65-72

```tablegen
def SDT_PPCVecInsert : SDTypeProfile<1, 3, [ SDTCisVec<0>,
  SDTCisVec<1>, SDTCisVec<2>, SDTCisInt<3>
]>;

def SDT_PPCxxpermdi: SDTypeProfile<1, 3, [ SDTCisVec<0>,
  SDTCisVec<1>, SDTCisVec<2>, SDTCisInt<3>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCVecInsert`, `SDT_PPCxxpermdi` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCVecInsert`, `SDT_PPCxxpermdi`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 73-80

```tablegen
def SDT_PPCvcmp : SDTypeProfile<1, 3, [
  SDTCisSameAs<0, 1>, SDTCisSameAs<1, 2>, SDTCisVT<3, i32>
]>;

def SDT_PPCcondbr : SDTypeProfile<0, 3, [
  SDTCisVT<0, i32>, SDTCisVT<2, OtherVT>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCvcmp`, `SDT_PPCcondbr` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCvcmp`, `SDT_PPCcondbr`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 81-89

```tablegen
def SDT_PPCFtsqrt : SDTypeProfile<1, 1, [
  SDTCisVT<0, i32>]>;

def SDT_PPClbrx : SDTypeProfile<1, 2, [
  SDTCisInt<0>, SDTCisPtrTy<1>, SDTCisVT<2, OtherVT>
]>;
def SDT_PPCstbrx : SDTypeProfile<0, 3, [
  SDTCisInt<0>, SDTCisPtrTy<1>, SDTCisVT<2, OtherVT>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCFtsqrt`, `SDT_PPClbrx`, `SDT_PPCstbrx` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCFtsqrt`, `SDT_PPClbrx`, `SDT_PPCstbrx`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 90-97

```tablegen
def SDT_StoreCond : SDTypeProfile<0, 3, [
  SDTCisPtrTy<0>, SDTCisInt<1>, SDTCisPtrTy<2>
]>;

def SDT_PPCTC_ret : SDTypeProfile<0, 2, [
  SDTCisPtrTy<0>, SDTCisVT<1, i32>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_StoreCond`, `SDT_PPCTC_ret` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_StoreCond`, `SDT_PPCTC_ret`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 98-104

```tablegen
def tocentry32 : Operand<iPTR> {
  let MIOperandInfo = (ops i32imm:$imm);
}

def SDT_PPCqvfperm   : SDTypeProfile<1, 3, [
  SDTCisVec<0>, SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>, SDTCisVec<3>
]>;
```
- **EN**: Adds declarative TableGen records such as `tocentry32`, `SDT_PPCqvfperm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `tocentry32`, `SDT_PPCqvfperm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 105-114

```tablegen
def SDT_PPCqvgpci   : SDTypeProfile<1, 1, [
  SDTCisVec<0>, SDTCisInt<1>
]>;
def SDT_PPCqvaligni   : SDTypeProfile<1, 3, [
  SDTCisVec<0>, SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>, SDTCisInt<3>
]>;
def SDT_PPCqvesplati   : SDTypeProfile<1, 2, [
  SDTCisVec<0>, SDTCisSameAs<0, 1>, SDTCisInt<2>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCqvgpci`, `SDT_PPCqvaligni`, `SDT_PPCqvesplati` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCqvgpci`, `SDT_PPCqvaligni`, `SDT_PPCqvesplati`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 115-122

```tablegen
def SDT_PPCqbflt : SDTypeProfile<1, 1, [
  SDTCisVec<0>, SDTCisVec<1>
]>;

def SDT_PPCqvlfsb : SDTypeProfile<1, 1, [
  SDTCisVec<0>, SDTCisPtrTy<1>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCqbflt`, `SDT_PPCqvlfsb` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCqbflt`, `SDT_PPCqvlfsb`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 123-131

```tablegen
def SDT_PPCextswsli : SDTypeProfile<1, 2, [  // extswsli
  SDTCisInt<0>, SDTCisInt<1>, SDTCisOpSmallerThanOp<1, 0>, SDTCisInt<2>
]>;

def SDT_PPCFPMinMax : SDTypeProfile<1, 2, [
  SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>, SDTCisFP<0>
]>;

// RES, CARRY = op LHS, RHS
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCextswsli`, `SDT_PPCFPMinMax` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCextswsli`, `SDT_PPCFPMinMax`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 132-138

```tablegen
def SDT_PPCBinaryArithWithFlagsOut : SDTypeProfile<2, 2, [
  SDTCisSameAs<0, 2>, SDTCisSameAs<0, 3>,
  SDTCisInt<0>,
  SDTCisVT<1, i32>,
]>;

// RES, CARRY = op LHS, RHS, CARRY
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCBinaryArithWithFlagsOut` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCBinaryArithWithFlagsOut`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 139-145

```tablegen
def SDT_PPCBinaryArithWithFlagsInOut : SDTypeProfile<2, 3, [
  SDTCisSameAs<0, 2>, SDTCisSameAs<0, 3>,
  SDTCisInt<0>,
  SDTCisSameAs<1, 4>,
  SDTCisVT<1, i32>,
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCBinaryArithWithFlagsInOut` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCBinaryArithWithFlagsInOut`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 146-153

```tablegen
def SDT_PPCBcdShift : SDTypeProfile<1, 3, [ SDTCisVT<0, v16i8>,
  SDTCisVT<1, v4i32>, SDTCisVT<2, v16i8>, SDTCisInt<3>
]>;

def SDT_PPCBcdShiftRound : SDTypeProfile<1, 3, [ SDTCisVT<0, v16i8>,
  SDTCisVT<1, v4i32>, SDTCisVT<2, v16i8>, SDTCisInt<3>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCBcdShift`, `SDT_PPCBcdShiftRound` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCBcdShift`, `SDT_PPCBcdShiftRound`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 154-161

```tablegen
def SDT_PPCBcdTrunc : SDTypeProfile<1, 3, [ SDTCisVT<0, v16i8>,
  SDTCisVT<1, v4i32>, SDTCisVT<2, v16i8>, SDTCisInt<3>
]>;

def SDT_PPCBcdUTrunc : SDTypeProfile<1, 2, [ SDTCisVT<0, v16i8>,
  SDTCisVT<1, v4i32>, SDTCisVT<2, v16i8>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCBcdTrunc`, `SDT_PPCBcdUTrunc` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCBcdTrunc`, `SDT_PPCBcdUTrunc`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 162-165

```tablegen
def SDT_PPCBcdUShift : SDTypeProfile<1, 2, [ SDTCisVT<0, v16i8>,
  SDTCisVT<1, v4i32>, SDTCisVT<2, v16i8>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCBcdUShift` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCBcdUShift`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 166-173

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC specific DAG Nodes.
//

def PPCfre    : SDNode<"PPCISD::FRE",     SDTFPUnaryOp, []>;
def PPCfrsqrte: SDNode<"PPCISD::FRSQRTE", SDTFPUnaryOp, []>;

// Square root instruction.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PowerPC specific DAG Nodes.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PowerPC specific DAG Nodes.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 174-181

```tablegen
def PPCfsqrt  : SDNode<"PPCISD::FSQRT",   SDTFPUnaryOp, []>;

// Test instruction for software square root.
def PPCftsqrt : SDNode<"PPCISD::FTSQRT",  SDT_PPCFtsqrt,[]>;

// FCFID - The FCFID instruction, taking an f64 operand and producing
// and f64 value containing the FP representation of the integer that
// was temporarily in the f64 operand.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Test instruction for software square root.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Test instruction for software square root.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 182-192

```tablegen
def PPCfcfid  : SDNode<"PPCISD::FCFID",   SDTFPUnaryOp, []>;

// Newer FCFID[US] integer-to-floating-point conversion instructions for
// unsigned integers and single-precision outputs.
def PPCfcfidu : SDNode<"PPCISD::FCFIDU",  SDTFPUnaryOp, []>;
def PPCfcfids : SDNode<"PPCISD::FCFIDS",  SDTFPRoundOp, []>;
def PPCfcfidus: SDNode<"PPCISD::FCFIDUS", SDTFPRoundOp, []>;

// FCTI[D,W]Z - The FCTIDZ and FCTIWZ instructions, taking an f32 or f64
// operand, producing an f64 value containing the integer representation
// of that FP value.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Newer FCFID[US] integer-to-floating-point conversion instructions for". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Newer FCFID[US] integer-to-floating-point conversion instructions for”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 193-201

```tablegen
def PPCfctidz : SDNode<"PPCISD::FCTIDZ", SDTFPUnaryOp, []>;
def PPCfctiwz : SDNode<"PPCISD::FCTIWZ", SDTFPUnaryOp, []>;

// Newer FCTI[D,W]UZ floating-point-to-integer conversion instructions for
// unsigned integers with round toward zero.
def PPCfctiduz: SDNode<"PPCISD::FCTIDUZ",SDTFPUnaryOp, []>;
def PPCfctiwuz: SDNode<"PPCISD::FCTIWUZ",SDTFPUnaryOp, []>;

// VSRQ - The ISA 3.1 (P10) Vector Shift right quadword instruction
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Newer FCTI[D,W]UZ floating-point-to-integer conversion instructions for". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Newer FCTI[D,W]UZ floating-point-to-integer conversion instructions for”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 202-209

```tablegen
def PPCvsrq: SDNode<"PPCISD::VSRQ", SDT_PPCVecShiftQuad, []>;

// Constrained integer-to-floating-point conversion instructions.
let IsStrictFP = true in {
  def PPCstrict_fcfid  : SDNode<"PPCISD::STRICT_FCFID",
                                SDTFPUnaryOp, [SDNPHasChain]>;
  def PPCstrict_fcfidu : SDNode<"PPCISD::STRICT_FCFIDU",
                                SDTFPUnaryOp, [SDNPHasChain]>;
```
- **EN**: Adds declarative TableGen records such as `PPCvsrq` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCvsrq`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 210-218

```tablegen
  def PPCstrict_fcfids : SDNode<"PPCISD::STRICT_FCFIDS",
                                SDTFPRoundOp, [SDNPHasChain]>;
  def PPCstrict_fcfidus : SDNode<"PPCISD::STRICT_FCFIDUS",
                                 SDTFPRoundOp, [SDNPHasChain]>;
}

def PPCany_fcfid : PatFrags<(ops node:$op),
                             [(PPCfcfid node:$op),
                              (PPCstrict_fcfid node:$op)]>;
```
- **EN**: Adds declarative TableGen records such as `PPCany_fcfid` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCany_fcfid`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 219-229

```tablegen
def PPCany_fcfidu : PatFrags<(ops node:$op),
                             [(PPCfcfidu node:$op),
                              (PPCstrict_fcfidu node:$op)]>;
def PPCany_fcfids : PatFrags<(ops node:$op),
                              [(PPCfcfids node:$op),
                               (PPCstrict_fcfids node:$op)]>;
def PPCany_fcfidus : PatFrags<(ops node:$op),
                              [(PPCfcfidus node:$op),
                               (PPCstrict_fcfidus node:$op)]>;

// Store scalar integers from VSR.
```
- **EN**: Adds declarative TableGen records such as `PPCany_fcfidu`, `PPCany_fcfids`, `PPCany_fcfidus` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCany_fcfidu`, `PPCany_fcfids`, `PPCany_fcfidus`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 230-241

```tablegen
def PPCstore_scal_int_from_vsr:
   SDNode<"PPCISD::ST_VSR_SCAL_INT", SDT_PPCstore_scal_int_from_vsr,
           [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;

// STFIWX - The STFIWX instruction.  The first operand is an input token
// chain, then an f64 value to store, then an address to store it to.
def PPCstfiwx : SDNode<"PPCISD::STFIWX", SDT_PPCstfiwx,
                       [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;

// GPRC, CHAIN = LFIWAX CHAIN, Ptr - This is a floating-point
// load which sign-extends from a 32-bit integer value into the
// destination 64-bit register.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "STFIWX - The STFIWX instruction.  The first operand is an input token". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“STFIWX - The STFIWX instruction.  The first operand is an input token”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 242-253

```tablegen
def PPClfiwax : SDNode<"PPCISD::LFIWAX", SDT_PPClfiwx,
                       [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

// GPRC, CHAIN = LFIWZX CHAIN, Ptr - This is a floating-point
// load which zero-extends from a 32-bit integer value into the
// destination 64-bit register.
def PPClfiwzx : SDNode<"PPCISD::LFIWZX", SDT_PPClfiwx,
                       [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

// GPRC, CHAIN = LXSIZX, CHAIN, Ptr, ByteWidth - This is a load of an
// integer smaller than 64 bits into a VSR. The integer is zero-extended.
// This can be used for converting loaded integers to floating point.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "GPRC, CHAIN = LFIWZX CHAIN, Ptr - This is a floating-point". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“GPRC, CHAIN = LFIWZX CHAIN, Ptr - This is a floating-point”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 254-264

```tablegen
def PPClxsizx : SDNode<"PPCISD::LXSIZX", SDT_PPCLxsizx,
                       [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

// STXSIX - The STXSI[bh]X instruction. The first operand is an input
// chain, then an f64 value to store, then an address to store it to,
// followed by a byte-width for the store.
def PPCstxsix : SDNode<"PPCISD::STXSIX", SDT_PPCstxsix,
                       [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;

// VEXTS, ByteWidth - takes an input in VSFRC and produces an output in
// VSFRC that is sign-extended from ByteWidth to a 64-byte integer.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "STXSIX - The STXSI[bh]X instruction. The first operand is an input". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“STXSIX - The STXSI[bh]X instruction. The first operand is an input”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 265-274

```tablegen
def PPCVexts  : SDNode<"PPCISD::VEXTS", SDT_PPCVexts, []>;

// F8RC = MFFS - This moves the FPSCR (not modeled) into the register.
def PPCmffs   : SDNode<"PPCISD::MFFS",
                       SDTypeProfile<1, 0, [SDTCisVT<0, f64>]>,
                       [SDNPHasChain]>;

// F8RC = FADDRTZ F8RC, F8RC - This is an FADD done with rounding
// towards zero.  Used only as part of the long double-to-int
// conversion sequence.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "F8RC = MFFS - This moves the FPSCR (not modeled) into the register.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“F8RC = MFFS - This moves the FPSCR (not modeled) into the register.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 275-281

```tablegen
def PPCfaddrtz: SDNode<"PPCISD::FADDRTZ", SDTFPBinOp, []>;

// Constrained floating point add in round-to-zero mode.
let IsStrictFP = true in
def PPCstrict_faddrtz: SDNode<"PPCISD::STRICT_FADDRTZ", SDTFPBinOp,
                              [SDNPHasChain]>;
```
- **EN**: Adds declarative TableGen records such as `PPCfaddrtz`, `PPCstrict_faddrtz` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCfaddrtz`, `PPCstrict_faddrtz`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 282-292

```tablegen
def PPCany_faddrtz: PatFrags<(ops node:$lhs, node:$rhs),
                             [(PPCfaddrtz node:$lhs, node:$rhs),
                              (PPCstrict_faddrtz node:$lhs, node:$rhs)]>;

// FSEL - Traditional three-operand fsel node.
def PPCfsel   : SDNode<"PPCISD::FSEL",
   // Type constraint for fsel.
   SDTypeProfile<1, 3, [SDTCisSameAs<0, 2>, SDTCisSameAs<0, 3>,
                        SDTCisFP<0>, SDTCisVT<1, f64>]>, []>;

// XSMAXC[DQ]P, XSMINC[DQ]P - C-type min/max instructions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FSEL - Traditional three-operand fsel node.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FSEL - Traditional three-operand fsel node.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 293-300

```tablegen
def PPCxsmaxc : SDNode<"PPCISD::XSMAXC", SDT_PPCFPMinMax, []>;
def PPCxsminc : SDNode<"PPCISD::XSMINC", SDT_PPCFPMinMax, []>;

// Hi/Lo - These represent the high and low 16-bit parts of a global
// address respectively.  These nodes have two operands, the first of
// which must be a TargetGlobalAddress, and the second of which must be a
// Constant.  Selected naively, these turn into 'lis G+C' and 'li G+C',
// though these are usually folded into other nodes.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Hi/Lo - These represent the high and low 16-bit parts of a global". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Hi/Lo - These represent the high and low 16-bit parts of a global”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 301-311

```tablegen
def PPChi       : SDNode<"PPCISD::Hi", SDTIntBinOp, []>;
def PPClo       : SDNode<"PPCISD::Lo", SDTIntBinOp, []>;

// GPRC = TOC_ENTRY GA, TOC
// Loads the entry for GA from the TOC, where the TOC base is given by
// the last operand.
def PPCtoc_entry: SDNode<"PPCISD::TOC_ENTRY", SDTIntBinOp,
                         [SDNPMayLoad, SDNPMemOperand]>;

// GPRC = address of _GLOBAL_OFFSET_TABLE_. Used by initial-exec TLS
// for non-position independent code on PPC32.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "GPRC = TOC_ENTRY GA, TOC". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“GPRC = TOC_ENTRY GA, TOC”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 312-322

```tablegen
def PPCppc32GOT : SDNode<"PPCISD::PPC32_GOT", SDTIntLeaf, []>;

// G8RC = ADDIS_GOT_TPREL_HA %x2, Symbol - Used by the initial-exec
// TLS model, produces an ADDIS8 instruction that adds the GOT
// base to sym\@got\@tprel\@ha.
def PPCaddisGotTprelHA : SDNode<"PPCISD::ADDIS_GOT_TPREL_HA", SDTIntBinOp>;

// G8RC = LD_GOT_TPREL_L Symbol, G8RReg - Used by the initial-exec
// TLS model, produces a LD instruction with base register G8RReg
// and offset sym\@got\@tprel\@l.  This completes the addition that
// finds the offset of "sym" relative to the thread pointer.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "G8RC = ADDIS_GOT_TPREL_HA %x2, Symbol - Used by the initial-exec". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“G8RC = ADDIS_GOT_TPREL_HA %x2, Symbol - Used by the initial-exec”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 323-331

```tablegen
def PPCldGotTprelL : SDNode<"PPCISD::LD_GOT_TPREL_L", SDTIntBinOp,
                            [SDNPMayLoad]>;

// G8RC = ADD_TLS G8RReg, Symbol - Can be used by the initial-exec
// and local-exec TLS models, produces an ADD instruction that adds
// the contents of G8RReg to the thread pointer.  Symbol contains a
// relocation sym\@tls which is to be replaced by the thread pointer
// and identifies to the linker that the instruction is part of a
// TLS sequence.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "G8RC = ADD_TLS G8RReg, Symbol - Can be used by the initial-exec". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“G8RC = ADD_TLS G8RReg, Symbol - Can be used by the initial-exec”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 332-342

```tablegen
def PPCaddTls     : SDNode<"PPCISD::ADD_TLS", SDTIntBinOp, []>;

// G8RC = ADDIS_TLSGD_HA %x2, Symbol - For the general-dynamic TLS
// model, produces an ADDIS8 instruction that adds the GOT base
// register to sym\@got\@tlsgd\@ha.
def PPCaddisTlsgdHA : SDNode<"PPCISD::ADDIS_TLSGD_HA", SDTIntBinOp>;

// %x3 = ADDI_TLSGD_L G8RReg, Symbol - For the general-dynamic TLS
// model, produces an ADDI8 instruction that adds G8RReg to
// sym\@got\@tlsgd\@l and stores the result in X3.  Hidden by
// ADDIS_TLSGD_L_ADDR until after register assignment.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "G8RC = ADDIS_TLSGD_HA %x2, Symbol - For the general-dynamic TLS". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“G8RC = ADDIS_TLSGD_HA %x2, Symbol - For the general-dynamic TLS”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 343-351

```tablegen
def PPCaddiTlsgdL   : SDNode<"PPCISD::ADDI_TLSGD_L", SDTIntBinOp>;

// %x3 = GET_TLS_ADDR %x3, Symbol - For the general-dynamic TLS
// model, produces a call to __tls_get_addr(sym\@tlsgd).  Hidden by
// ADDIS_TLSGD_L_ADDR until after register assignment.
def PPCgetTlsAddr   : SDNode<"PPCISD::GET_TLS_ADDR", SDTIntBinOp>;

// %x3 = GET_TLS_MOD_AIX _$TLSML - For the AIX local-dynamic TLS model,
// produces a call to .__tls_get_mod(_$TLSML\@ml).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "%x3 = GET_TLS_ADDR %x3, Symbol - For the general-dynamic TLS". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“%x3 = GET_TLS_ADDR %x3, Symbol - For the general-dynamic TLS”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 352-361

```tablegen
def PPCgetTlsMod   : SDNode<"PPCISD::GET_TLS_MOD_AIX", SDTIntUnaryOp>;

// %x3 = GET_TPOINTER - Used for the local- and initial-exec TLS model on
// 32-bit AIX, produces a call to .__get_tpointer to retrieve the thread
// pointer. At the end of the call, the thread pointer is found in R3.
def PPCgetTpointer : SDNode<"PPCISD::GET_TPOINTER", SDTIntLeaf, []>;

// G8RC = ADDI_TLSGD_L_ADDR G8RReg, Symbol, Symbol - Op that
// combines ADDI_TLSGD_L and GET_TLS_ADDR until expansion following
// register assignment.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "%x3 = GET_TPOINTER - Used for the local- and initial-exec TLS model on". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“%x3 = GET_TPOINTER - Used for the local- and initial-exec TLS model on”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 362-373

```tablegen
def PPCaddiTlsgdLAddr : SDNode<"PPCISD::ADDI_TLSGD_L_ADDR",
                               SDTypeProfile<1, 3, [
                                 SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>,
                                 SDTCisSameAs<0, 3>, SDTCisInt<0> ]>>;

// GPRC = TLSGD_AIX, TOC_ENTRY, TOC_ENTRY
// G8RC = TLSGD_AIX, TOC_ENTRY, TOC_ENTRY
// Op that combines two register copies of TOC entries
// (region handle into R3 and variable offset into R4) followed by a
// GET_TLS_ADDR node which will be expanded to a call to .__tls_get_addr.
// This node is used in 64-bit mode as well (in which case the result is
// G8RC and inputs are X3/X4).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "GPRC = TLSGD_AIX, TOC_ENTRY, TOC_ENTRY". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“GPRC = TLSGD_AIX, TOC_ENTRY, TOC_ENTRY”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 374-380

```tablegen
def PPCTlsgdAIX     : SDNode<"PPCISD::TLSGD_AIX", SDTIntBinOp>;

// [GP|G8]RC = TLSLD_AIX, TOC_ENTRY(module handle)
// Op that requires a single input of the module handle TOC entry in R3,
// and generates a GET_TLS_MOD_AIX node which will be expanded into a call
// to .__tls_get_mod. This node is used in both 32-bit and 64-bit modes.
// The only difference is the register class.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "[GP|G8]RC = TLSLD_AIX, TOC_ENTRY(module handle)". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“[GP|G8]RC = TLSLD_AIX, TOC_ENTRY(module handle)”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 381-391

```tablegen
def PPCTlsldAIX     : SDNode<"PPCISD::TLSLD_AIX", SDTIntUnaryOp>;

// G8RC = ADDIS_TLSLD_HA %x2, Symbol - For the local-dynamic TLS
// model, produces an ADDIS8 instruction that adds the GOT base
// register to sym\@got\@tlsld\@ha.
def PPCaddisTlsldHA : SDNode<"PPCISD::ADDIS_TLSLD_HA", SDTIntBinOp>;

// %x3 = ADDI_TLSLD_L G8RReg, Symbol - For the local-dynamic TLS
// model, produces an ADDI8 instruction that adds G8RReg to
// sym\@got\@tlsld\@l and stores the result in X3.  Hidden by
// ADDIS_TLSLD_L_ADDR until after register assignment.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "G8RC = ADDIS_TLSLD_HA %x2, Symbol - For the local-dynamic TLS". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“G8RC = ADDIS_TLSLD_HA %x2, Symbol - For the local-dynamic TLS”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 392-401

```tablegen
def PPCaddiTlsldL   : SDNode<"PPCISD::ADDI_TLSLD_L", SDTIntBinOp>;

// %x3 = GET_TLSLD_ADDR %x3, Symbol - For the local-dynamic TLS
// model, produces a call to __tls_get_addr(sym\@tlsld).  Hidden by
// ADDIS_TLSLD_L_ADDR until after register assignment.
def PPCgetTlsldAddr : SDNode<"PPCISD::GET_TLSLD_ADDR", SDTIntBinOp>;

// G8RC = ADDI_TLSLD_L_ADDR G8RReg, Symbol, Symbol - Op that
// combines ADDI_TLSLD_L and GET_TLSLD_ADDR until expansion
// following register assignment.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "%x3 = GET_TLSLD_ADDR %x3, Symbol - For the local-dynamic TLS". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“%x3 = GET_TLSLD_ADDR %x3, Symbol - For the local-dynamic TLS”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 402-409

```tablegen
def PPCaddiTlsldLAddr : SDNode<"PPCISD::ADDI_TLSLD_L_ADDR",
                               SDTypeProfile<1, 3, [
                                 SDTCisSameAs<0, 1>, SDTCisSameAs<0, 2>,
                                 SDTCisSameAs<0, 3>, SDTCisInt<0> ]>>;

// G8RC = ADDIS_DTPREL_HA %x3, Symbol - For the local-dynamic TLS
// model, produces an ADDIS8 instruction that adds X3 to
// sym\@dtprel\@ha.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "G8RC = ADDIS_DTPREL_HA %x3, Symbol - For the local-dynamic TLS". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“G8RC = ADDIS_DTPREL_HA %x3, Symbol - For the local-dynamic TLS”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 410-418

```tablegen
def PPCaddisDtprelHA : SDNode<"PPCISD::ADDIS_DTPREL_HA", SDTIntBinOp>;

// G8RC = ADDI_DTPREL_L G8RReg, Symbol - For the local-dynamic TLS
// model, produces an ADDI8 instruction that adds G8RReg to
// sym\@got\@dtprel\@l.
def PPCaddiDtprelL   : SDNode<"PPCISD::ADDI_DTPREL_L", SDTIntBinOp>;

// G8RC = PADDI_DTPREL %x3, Symbol - For the pc-rel based local-dynamic TLS
// model, produces a PADDI8 instruction that adds X3 to sym\@dtprel.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "G8RC = ADDI_DTPREL_L G8RReg, Symbol - For the local-dynamic TLS". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“G8RC = ADDI_DTPREL_L G8RReg, Symbol - For the local-dynamic TLS”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 419-429

```tablegen
def PPCpaddiDtprel   : SDNode<"PPCISD::PADDI_DTPREL", SDTIntBinOp>;

// VPERM - The PPC VPERM Instruction.
def PPCvperm     : SDNode<"PPCISD::VPERM", SDT_PPCvperm, []>;

// XXSPLT - The PPC VSX splat instructions
def PPCxxsplt    : SDNode<"PPCISD::XXSPLT", SDT_PPCVecSplat, []>;

// XXSPLTI_SP_TO_DP - The PPC VSX splat instructions for immediates for
// converting immediate single precision numbers to double precision
// vector or scalar.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VPERM - The PPC VPERM Instruction.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VPERM - The PPC VPERM Instruction.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 430-438

```tablegen
def PPCxxspltidp : SDNode<"PPCISD::XXSPLTI_SP_TO_DP", SDT_PPCSpToDp, []>;

// VECINSERT - The PPC vector insert instruction
def PPCvecinsert : SDNode<"PPCISD::VECINSERT", SDT_PPCVecInsert, []>;

// XXPERMDI - The PPC XXPERMDI instruction
def PPCxxpermdi  : SDNode<"PPCISD::XXPERMDI", SDT_PPCxxpermdi, []>;

// VECSHL - The PPC vector shift left instruction
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VECINSERT - The PPC vector insert instruction". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VECINSERT - The PPC vector insert instruction”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 439-451

```tablegen
def PPCvecshl    : SDNode<"PPCISD::VECSHL", SDT_PPCVecShift, []>;

// The CMPB instruction (takes two operands of i32 or i64).
def PPCcmpb     : SDNode<"PPCISD::CMPB", SDTIntBinOp, []>;

// These nodes represent the 32-bit PPC shifts that operate on 6-bit shift
// amounts.  These nodes are generated by the multi-precision shift code.
//
// For scalar types, only the last `n + 1` bits of the shift amounts
// are used, where n is log2(sizeof(element) * 8). See sld/slw, etc.
// for exact behaviors.
//
// For vector types, only the last n bits are used. See vsld.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The CMPB instruction (takes two operands of i32 or i64).". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The CMPB instruction (takes two operands of i32 or i64).”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 452-460

```tablegen
def PPCsrl        : SDNode<"PPCISD::SRL"       , SDTIntShiftOp>;
def PPCsra        : SDNode<"PPCISD::SRA"       , SDTIntShiftOp>;
def PPCshl        : SDNode<"PPCISD::SHL"       , SDTIntShiftOp>;

// FNMSUB - Negated multiply-subtract instruction.
def PPCfnmsub     : SDNode<"PPCISD::FNMSUB"    , SDTFPTernaryOp>;

// EXTSWSLI = The PPC extswsli instruction, which does an extend-sign
// word and shift left immediate.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FNMSUB - Negated multiply-subtract instruction.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FNMSUB - Negated multiply-subtract instruction.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 461-468

```tablegen
def PPCextswsli : SDNode<"PPCISD::EXTSWSLI" , SDT_PPCextswsli>;

// Constrained conversion from floating point to int
let IsStrictFP = true in {
  def PPCstrict_fctidz : SDNode<"PPCISD::STRICT_FCTIDZ",
                                SDTFPUnaryOp, [SDNPHasChain]>;
  def PPCstrict_fctiwz : SDNode<"PPCISD::STRICT_FCTIWZ",
                                SDTFPUnaryOp, [SDNPHasChain]>;
```
- **EN**: Adds declarative TableGen records such as `PPCextswsli` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCextswsli`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 469-477

```tablegen
  def PPCstrict_fctiduz : SDNode<"PPCISD::STRICT_FCTIDUZ",
                                 SDTFPUnaryOp, [SDNPHasChain]>;
  def PPCstrict_fctiwuz : SDNode<"PPCISD::STRICT_FCTIWUZ",
                                 SDTFPUnaryOp, [SDNPHasChain]>;
}

def PPCany_fctidz : PatFrags<(ops node:$op),
                             [(PPCstrict_fctidz node:$op),
                              (PPCfctidz node:$op)]>;
```
- **EN**: Adds declarative TableGen records such as `PPCany_fctidz` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCany_fctidz`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 478-488

```tablegen
def PPCany_fctiwz : PatFrags<(ops node:$op),
                             [(PPCstrict_fctiwz node:$op),
                              (PPCfctiwz node:$op)]>;
def PPCany_fctiduz : PatFrags<(ops node:$op),
                              [(PPCstrict_fctiduz node:$op),
                               (PPCfctiduz node:$op)]>;
def PPCany_fctiwuz : PatFrags<(ops node:$op),
                              [(PPCstrict_fctiwuz node:$op),
                               (PPCfctiwuz node:$op)]>;

// Direct move of 2 consecutive GPR to a VSX register.
```
- **EN**: Adds declarative TableGen records such as `PPCany_fctiwz`, `PPCany_fctiduz`, `PPCany_fctiwuz` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCany_fctiwz`, `PPCany_fctiduz`, `PPCany_fctiwuz`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 489-498

```tablegen
def PPCbuild_fp128: SDNode<"PPCISD::BUILD_FP128",
                           SDTypeProfile<1, 2,
                             [SDTCisFP<0>, SDTCisSameSizeAs<1,2>,
                              SDTCisSameAs<1,2>]>,
                           []>;

// BUILD_SPE64 and EXTRACT_SPE are analogous to BUILD_PAIR and
// EXTRACT_ELEMENT but take f64 arguments instead of i64, as i64 is
// unsupported for this target.
// Merge 2 GPRs to a single SPE register.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "BUILD_SPE64 and EXTRACT_SPE are analogous to BUILD_PAIR and". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“BUILD_SPE64 and EXTRACT_SPE are analogous to BUILD_PAIR and”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 499-505

```tablegen
def PPCbuild_spe64: SDNode<"PPCISD::BUILD_SPE64",
                           SDTypeProfile<1, 2,
                             [SDTCisVT<0, f64>, SDTCisVT<1,i32>,
                             SDTCisVT<1,i32>]>,
                           []>;

// Extract SPE register component, second argument is high or low.
```
- **EN**: Adds declarative TableGen records such as `PPCbuild_spe64` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCbuild_spe64`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 506-512

```tablegen
def PPCextract_spe : SDNode<"PPCISD::EXTRACT_SPE",
                            SDTypeProfile<1, 2,
                              [SDTCisVT<0, i32>, SDTCisVT<1, f64>,
                              SDTCisPtrTy<2>]>,
                              []>;

// These are target-independent nodes, but have target-specific formats.
```
- **EN**: Adds declarative TableGen records such as `PPCextract_spe` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCextract_spe`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 513-523

```tablegen
def callseq_start : SDNode<"ISD::CALLSEQ_START", SDT_PPCCallSeqStart,
                           [SDNPHasChain, SDNPOutGlue]>;
def callseq_end   : SDNode<"ISD::CALLSEQ_END",   SDT_PPCCallSeqEnd,
                           [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;

def SDT_PPCCall   : SDTypeProfile<0, -1, [SDTCisInt<0>]>;

// CALL - A direct function call.
// CALL_NOP is a call with the special NOP which follows 64-bit
// CALL_NOTOC the caller does not use the TOC.
// SVR4 calls and 32-bit/64-bit AIX calls.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "CALL - A direct function call.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“CALL - A direct function call.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 524-535

```tablegen
def PPCcall  : SDNode<"PPCISD::CALL", SDT_PPCCall,
                      [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                       SDNPVariadic]>;
def PPCcall_nop  : SDNode<"PPCISD::CALL_NOP", SDT_PPCCall,
                          [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                           SDNPVariadic]>;
def PPCcall_notoc : SDNode<"PPCISD::CALL_NOTOC", SDT_PPCCall,
                           [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                            SDNPVariadic]>;

// CHAIN,FLAG = MTCTR(VAL, CHAIN[, INFLAG]) - Directly corresponds to a
// MTCTR instruction.
```
- **EN**: Adds declarative TableGen records such as `PPCcall`, `PPCcall_nop`, `PPCcall_notoc` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCcall`, `PPCcall_nop`, `PPCcall_notoc`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 536-547

```tablegen
def PPCmtctr      : SDNode<"PPCISD::MTCTR", SDT_PPCCall,
                           [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;

// CHAIN,FLAG = BCTRL(CHAIN, INFLAG) - Directly corresponds to a
// BCTRL instruction.
def PPCbctrl : SDNode<"PPCISD::BCTRL", SDTNone,
                      [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                       SDNPVariadic]>;

// CHAIN,FLAG = BCTRL(CHAIN, ADDR, INFLAG) - The combination of a bctrl
// instruction and the TOC reload required on 64-bit ELF, 32-bit AIX
// and 64-bit AIX.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "CHAIN,FLAG = BCTRL(CHAIN, INFLAG) - Directly corresponds to a". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“CHAIN,FLAG = BCTRL(CHAIN, INFLAG) - Directly corresponds to a”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 548-559

```tablegen
def PPCbctrl_load_toc : SDNode<"PPCISD::BCTRL_LOAD_TOC",
                               SDTypeProfile<0, 1, []>,
                               [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                                SDNPVariadic]>;

def PPCbl_load_toc : SDNode<"PPCISD::BL_LOAD_TOC",
                            SDTypeProfile<0, 1, []>,
                            [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                            SDNPVariadic]>;

// The variants that implicitly define rounding mode for calls with
// strictfp semantics.
```
- **EN**: Adds declarative TableGen records such as `PPCbctrl_load_toc`, `PPCbl_load_toc` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCbctrl_load_toc`, `PPCbl_load_toc`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 560-568

```tablegen
def PPCcall_rm  : SDNode<"PPCISD::CALL_RM", SDT_PPCCall,
                         [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                          SDNPVariadic]>;
def PPCcall_nop_rm  : SDNode<"PPCISD::CALL_NOP_RM", SDT_PPCCall,
                             [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                              SDNPVariadic]>;
def PPCcall_notoc_rm : SDNode<"PPCISD::CALL_NOTOC_RM", SDT_PPCCall,
                              [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                               SDNPVariadic]>;
```
- **EN**: Adds declarative TableGen records such as `PPCcall_rm`, `PPCcall_nop_rm`, `PPCcall_notoc_rm` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCcall_rm`, `PPCcall_nop_rm`, `PPCcall_notoc_rm`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 569-575

```tablegen
def PPCbctrl_rm : SDNode<"PPCISD::BCTRL_RM", SDTNone,
                         [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                          SDNPVariadic]>;
def PPCbctrl_load_toc_rm : SDNode<"PPCISD::BCTRL_LOAD_TOC_RM",
                                  SDTypeProfile<0, 1, []>,
                                  [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                                   SDNPVariadic]>;
```
- **EN**: Adds declarative TableGen records such as `PPCbctrl_rm`, `PPCbctrl_load_toc_rm` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCbctrl_rm`, `PPCbctrl_load_toc_rm`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 576-589

```tablegen
def PPCbl_load_toc_rm : SDNode<"PPCISD::BL_LOAD_TOC_RM",
                            SDTypeProfile<0, 1, []>,
                            [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                            SDNPVariadic]>;

// Return with a glue operand, matched by 'blr'
def PPCretglue   : SDNode<"PPCISD::RET_GLUE", SDTNone,
                           [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;

// TC_RETURN - A tail call return.
//   operand #0 chain
//   operand #1 callee (register or absolute)
//   operand #2 stack adjustment
//   operand #3 optional in flag
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Return with a glue operand, matched by 'blr'". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Return with a glue operand, matched by 'blr'”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 590-599

```tablegen
def PPCtc_return : SDNode<"PPCISD::TC_RETURN", SDT_PPCTC_ret,
                        [SDNPHasChain,  SDNPOptInGlue, SDNPVariadic]>;

// EH_SJLJ_SETJMP - SjLj exception handling setjmp.
def PPCeh_sjlj_setjmp  : SDNode<"PPCISD::EH_SJLJ_SETJMP",
                                SDTypeProfile<1, 1, [SDTCisInt<0>,
                                                     SDTCisPtrTy<1>]>,
                                [SDNPHasChain, SDNPSideEffect]>;

// EH_SJLJ_LONGJMP - SjLj exception handling longjmp.
```
- **EN**: Adds declarative TableGen records such as `PPCtc_return`, `PPCeh_sjlj_setjmp` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCtc_return`, `PPCeh_sjlj_setjmp`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 600-607

```tablegen
def PPCeh_sjlj_longjmp : SDNode<"PPCISD::EH_SJLJ_LONGJMP",
                                SDTypeProfile<0, 1, [SDTCisPtrTy<0>]>,
                                [SDNPHasChain, SDNPSideEffect]>;

// RESVEC = VCMP(LHS, RHS, OPC) - Represents one of the altivec VCMP*
// instructions.  For lack of better number, we use the opcode number
// encoding for the OPC field to identify the compare.  For example, 838
// is VCMPGTSH.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "RESVEC = VCMP(LHS, RHS, OPC) - Represents one of the altivec VCMP*". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“RESVEC = VCMP(LHS, RHS, OPC) - Represents one of the altivec VCMP*”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 608-620

```tablegen
def PPCvcmp       : SDNode<"PPCISD::VCMP" , SDT_PPCvcmp, []>;

// RESVEC, OUTFLAG = VCMP_rec(LHS, RHS, OPC) - Represents one of the
// altivec VCMP*_rec instructions.  For lack of better number, we use the
// opcode number encoding for the OPC field to identify the compare.  For
// example, 838 is VCMPGTSH.
def PPCvcmp_rec   : SDNode<"PPCISD::VCMP_rec", SDT_PPCvcmp, [SDNPOutGlue]>;

// CHAIN = COND_BRANCH CHAIN, CRRC, OPC, DESTBB [, INFLAG] - This
// corresponds to the COND_BRANCH pseudo instruction.  CRRC is the
// condition register to branch on, OPC is the branch opcode to use (e.g.
// PPC::BLE), DESTBB is the destination block to branch to, and INFLAG is
// an optional input flag argument.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "RESVEC, OUTFLAG = VCMP_rec(LHS, RHS, OPC) - Represents one of the". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“RESVEC, OUTFLAG = VCMP_rec(LHS, RHS, OPC) - Represents one of the”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 621-629

```tablegen
def PPCcondbranch : SDNode<"PPCISD::COND_BRANCH", SDT_PPCcondbr,
                           [SDNPHasChain, SDNPOptInGlue]>;

// ATOMIC_CMP_SWAP - the exact same as the target-independent nodes
// except they ensure that the compare input is zero-extended for
// sub-word versions because the atomic loads zero-extend.
def PPCatomicCmpSwap_8 :
  SDNode<"PPCISD::ATOMIC_CMP_SWAP_8", SDTAtomic3,
         [SDNPHasChain, SDNPMayStore, SDNPMayLoad, SDNPMemOperand]>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "ATOMIC_CMP_SWAP - the exact same as the target-independent nodes". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“ATOMIC_CMP_SWAP - the exact same as the target-independent nodes”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 630-637

```tablegen
def PPCatomicCmpSwap_16 :
  SDNode<"PPCISD::ATOMIC_CMP_SWAP_16", SDTAtomic3,
         [SDNPHasChain, SDNPMayStore, SDNPMayLoad, SDNPMemOperand]>;

// GPRC, CHAIN = LBRX CHAIN, Ptr, Type - This is a
// byte-swapping load instruction.  It loads "Type" bits, byte swaps it,
// then puts it in the bottom bits of the GPRC.  TYPE can be either i16
// or i32.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "GPRC, CHAIN = LBRX CHAIN, Ptr, Type - This is a". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“GPRC, CHAIN = LBRX CHAIN, Ptr, Type - This is a”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 638-644

```tablegen
def PPClbrx       : SDNode<"PPCISD::LBRX", SDT_PPClbrx,
                           [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

// CHAIN = STBRX CHAIN, GPRC, Ptr, Type - This is a
// byte-swapping store instruction.  It byte-swaps the low "Type" bits of
// the GPRC input, then stores it through Ptr.  Type can be either i16 or
// i32.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "CHAIN = STBRX CHAIN, GPRC, Ptr, Type - This is a". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“CHAIN = STBRX CHAIN, GPRC, Ptr, Type - This is a”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 645-655

```tablegen
def PPCstbrx      : SDNode<"PPCISD::STBRX", SDT_PPCstbrx,
                           [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;

// CHAIN,Glue = STORE_COND CHAIN, GPR, Ptr
// The store conditional instruction ST[BHWD]ARX that produces a glue
// result to attach it to a conditional branch.
def PPCStoreCond  : SDNode<"PPCISD::STORE_COND", SDT_StoreCond,
                           [SDNPHasChain, SDNPMayStore,
                            SDNPMemOperand, SDNPOutGlue]>;

// ch, gl = CR6[UN]SET ch, inglue - Toggle CR bit 6 for SVR4 vararg calls
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "CHAIN,Glue = STORE_COND CHAIN, GPR, Ptr". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“CHAIN,Glue = STORE_COND CHAIN, GPR, Ptr”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 656-662

```tablegen
def PPCcr6set   : SDNode<"PPCISD::CR6SET", SDTNone,
                         [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;
def PPCcr6unset : SDNode<"PPCISD::CR6UNSET", SDTNone,
                         [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;

// Instructions to support dynamic alloca.
def SDTDynOp  : SDTypeProfile<1, 2, []>;
```
- **EN**: Adds declarative TableGen records such as `PPCcr6set`, `PPCcr6unset`, `SDTDynOp` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCcr6set`, `PPCcr6unset`, `SDTDynOp`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 663-670

```tablegen
def SDTDynAreaOp  : SDTypeProfile<1, 1, []>;

// The following two target-specific nodes are used for calls through
// function pointers in the 64-bit SVR4 ABI.

// OPRC, CHAIN = DYNALLOC(CHAIN, NEGSIZE, FRAME_INDEX)
// This instruction is lowered in PPCRegisterInfo::eliminateFrameIndex to
// compute an allocation on the stack.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The following two target-specific nodes are used for calls through". Register assignment and register-class constraints matter here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The following two target-specific nodes are used for calls through”。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 671-679

```tablegen
def PPCdynalloc   : SDNode<"PPCISD::DYNALLOC", SDTDynOp, [SDNPHasChain]>;

// This instruction is lowered in PPCRegisterInfo::eliminateFrameIndex to
// compute an offset from native SP to the address  of the most recent
// dynamic alloca.
def PPCdynareaoffset   : SDNode<"PPCISD::DYNAREAOFFSET", SDTDynAreaOp, [SDNPHasChain]>;

// To avoid stack clash, allocation is performed by block and each block is
// probed.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This instruction is lowered in PPCRegisterInfo::eliminateFrameIndex to". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This instruction is lowered in PPCRegisterInfo::eliminateFrameIndex to”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 680-686

```tablegen
def PPCprobedalloca : SDNode<"PPCISD::PROBED_ALLOCA", SDTDynOp, [SDNPHasChain]>;

// PC Relative Specific Nodes

// MAT_PCREL_ADDR = Materialize a PC Relative address. This can be done
// either through an add like PADDI or through a PC Relative load like
// PLD.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PC Relative Specific Nodes". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PC Relative Specific Nodes”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 687-697

```tablegen
def PPCmatpcreladdr : SDNode<"PPCISD::MAT_PCREL_ADDR", SDTIntUnaryOp, []>;

// TLS_DYNAMIC_MAT_PCREL_ADDR = Materialize a PC Relative address for
// TLS global address when using dynamic access models. This can be done
// through an add like PADDI.
def PPCtlsdynamatpcreladdr : SDNode<"PPCISD::TLS_DYNAMIC_MAT_PCREL_ADDR",
                                    SDTIntUnaryOp, []>;

// TLS_LOCAL_EXEC_MAT_ADDR = Materialize an address for TLS global address
// when using local exec access models, and when prefixed instructions are
// available. This is used with ADD_TLS to produce an add like PADDI.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "TLS_DYNAMIC_MAT_PCREL_ADDR = Materialize a PC Relative address for". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“TLS_DYNAMIC_MAT_PCREL_ADDR = Materialize a PC Relative address for”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 698-705

```tablegen
def PPCtlslocalexecmataddr : SDNode<"PPCISD::TLS_LOCAL_EXEC_MAT_ADDR",
                                    SDTIntUnaryOp, []>;

// These nodes represent PPC arithmetic operations with carry.
def PPCaddc : SDNode<"PPCISD::ADDC", SDT_PPCBinaryArithWithFlagsOut,
                     [SDNPCommutative]>;
def PPCadde : SDNode<"PPCISD::ADDE", SDT_PPCBinaryArithWithFlagsInOut,
                     []>;
```
- **EN**: Adds declarative TableGen records such as `PPCtlslocalexecmataddr`, `PPCaddc`, `PPCadde` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCtlslocalexecmataddr`, `PPCaddc`, `PPCadde`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 706-712

```tablegen
def PPCsubc : SDNode<"PPCISD::SUBC", SDT_PPCBinaryArithWithFlagsOut,
                     []>;
def PPCsube : SDNode<"PPCISD::SUBE", SDT_PPCBinaryArithWithFlagsInOut,
                     []>;
def PPCbcds      : SDNode<"PPCISD::BCDSHIFT", SDT_PPCBcdShift, []>;
def PPCbcdsr     : SDNode<"PPCISD::BCDSHIFTROUND", SDT_PPCBcdShiftRound, []>;
def PPCbcdtrunc  : SDNode<"PPCISD::BCDTRUNC", SDT_PPCBcdTrunc, []>;
```
- **EN**: Adds declarative TableGen records such as `PPCsubc`, `PPCsube`, `PPCbcds` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCsubc`, `PPCsube`, `PPCbcds`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 713-715

```tablegen
def PPCbcdutrunc : SDNode<"PPCISD::BCDUTRUNC", SDT_PPCBcdUTrunc, []>;
def PPCbcdus     : SDNode<"PPCISD::BCDUSHIFT", SDT_PPCBcdUShift, []>;
```
- **EN**: Adds declarative TableGen records such as `PPCbcdutrunc`, `PPCbcdus` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCbcdutrunc`, `PPCbcdus`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 716-722

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC specific transformation functions and pattern fragments.
//

// A floating point immediate that is not a positive zero and can be converted
// to a single precision floating point non-denormal immediate without loss of
// information.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PowerPC specific transformation functions and pattern fragments.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PowerPC specific transformation functions and pattern fragments.”。

### Lines 723-729

```tablegen
def nzFPImmAsi32 : PatLeaf<(fpimm), [{
  APFloat APFloatOfN = N->getValueAPF();
  return convertToNonDenormSingle(APFloatOfN) && !N->isExactlyValue(+0.0);
}]>;

// A floating point immediate that is exactly an integer (for example 3.0, -5.0)
// and can be represented in 5 bits (range of [-16, 15]).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "A floating point immediate that is exactly an integer (for example 3.0, -5.0)". Notable symbols in this range include `nzFPImmAsi32`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“A floating point immediate that is exactly an integer (for example 3.0, -5.0)”。 该区间中较显眼的符号包括 `nzFPImmAsi32`。

### Lines 730-738

```tablegen
def nzFPImmExactInti5 : PatLeaf<(fpimm), [{
  APFloat FloatValue = N->getValueAPF();
  bool IsExact;
  APSInt IntResult(16, false);
  FloatValue.convertToInteger(IntResult, APFloat::rmTowardZero, &IsExact);
  return IsExact && IntResult <= 15 && IntResult >= -16 && !FloatValue.isZero();
}]>;

// Floating point zero immediates (positive and negative)
```
- **EN**: Adds declarative TableGen records such as `nzFPImmExactInti5` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `nzFPImmExactInti5`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 739-751

```tablegen
def fpimm0 : PatLeaf<(fpimm), [{ return N->isExactlyValue(+0.0); }]>;
def fpimm0neg : PatLeaf<(fpimm), [{return N->isExactlyValue(-0.0);}]>;

def getFPAs5BitExactInt : SDNodeXForm<fpimm, [{
  APFloat FloatValue = N->getValueAPF();
  bool IsExact;
  APSInt IntResult(32, false);
  FloatValue.convertToInteger(IntResult, APFloat::rmTowardZero, &IsExact);
  return CurDAG->getTargetConstant(IntResult, SDLoc(N), MVT::i32);
}]>;

// Convert the floating point immediate into a 32 bit floating point immediate
// and get a i32 with the resulting bits.
```
- **EN**: Adds declarative TableGen records such as `fpimm0`, `fpimm0neg`, `getFPAs5BitExactInt` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `fpimm0`, `fpimm0neg`, `getFPAs5BitExactInt`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 752-761

```tablegen
def getFPAs32BitInt : SDNodeXForm<fpimm, [{
  APFloat APFloatOfN = N->getValueAPF();
  convertToNonDenormSingle(APFloatOfN);
  return CurDAG->getTargetConstant(APFloatOfN.bitcastToAPInt().getZExtValue(),
                                   SDLoc(N), MVT::i32);
}]>;

// Check if the value can be converted to be single precision immediate, which
// can be exploited by XXSPLTIDP. Ensure that it cannot be converted to single
// precision before exploiting with XXSPLTI32DX.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Check if the value can be converted to be single precision immediate, which". Notable symbols in this range include `getFPAs32BitInt`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Check if the value can be converted to be single precision immediate, which”。 该区间中较显眼的符号包括 `getFPAs32BitInt`。

### Lines 762-778

```tablegen
def nzFPImmAsi64 : PatLeaf<(fpimm), [{
  APFloat APFloatOfN = N->getValueAPF();
  return !N->isExactlyValue(+0.0) && !checkConvertToNonDenormSingle(APFloatOfN);
}]>;

// Get the Hi bits of a 64 bit immediate.
def getFPAs64BitIntHi : SDNodeXForm<fpimm, [{
  APFloat APFloatOfN = N->getValueAPF();
  bool Unused;
  APFloatOfN.convert(APFloat::IEEEdouble(), APFloat::rmNearestTiesToEven,
                     &Unused);
  uint32_t Hi = (uint32_t)((APFloatOfN.bitcastToAPInt().getZExtValue() &
                            0xFFFFFFFF00000000LL) >> 32);
  return CurDAG->getTargetConstant(Hi, SDLoc(N), MVT::i32);
}]>;

// Get the Lo bits of a 64 bit immediate.
```
- **EN**: Adds declarative TableGen records such as `nzFPImmAsi64`, `getFPAs64BitIntHi` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `nzFPImmAsi64`, `getFPAs64BitIntHi`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 779-789

```tablegen
def getFPAs64BitIntLo : SDNodeXForm<fpimm, [{
  APFloat APFloatOfN = N->getValueAPF();
  bool Unused;
  APFloatOfN.convert(APFloat::IEEEdouble(), APFloat::rmNearestTiesToEven,
                     &Unused);
  uint32_t Lo = (uint32_t)(APFloatOfN.bitcastToAPInt().getZExtValue() &
                           0xFFFFFFFF);
  return CurDAG->getTargetConstant(Lo, SDLoc(N), MVT::i32);
}]>;
```
- **EN**: Adds declarative TableGen records such as `getFPAs64BitIntLo` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `getFPAs64BitIntLo`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 790-798

```tablegen
def getImmAs64BitInt : SDNodeXForm<imm, [{
  return getI64Imm(N->getSExtValue(), SDLoc(N));
}]>;

def SHL32 : SDNodeXForm<imm, [{
  // Transformation function: 31 - imm
  return getI32Imm(31 - N->getZExtValue(), SDLoc(N));
}]>;
```
- **EN**: Adds declarative TableGen records such as `getImmAs64BitInt`, `SHL32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `getImmAs64BitInt`, `SHL32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 799-809

```tablegen
def SRL32 : SDNodeXForm<imm, [{
  // Transformation function: 32 - imm
  return N->getZExtValue() ? getI32Imm(32 - N->getZExtValue(), SDLoc(N))
                           : getI32Imm(0, SDLoc(N));
}]>;

def LO16 : SDNodeXForm<imm, [{
  // Transformation function: get the low 16 bits.
  return getI32Imm((unsigned short)N->getZExtValue(), SDLoc(N));
}]>;
```
- **EN**: Adds declarative TableGen records such as `SRL32`, `LO16` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SRL32`, `LO16`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 810-819

```tablegen
def HI16 : SDNodeXForm<imm, [{
  // Transformation function: shift the immediate value down into the low bits.
  return getI32Imm((unsigned)N->getZExtValue() >> 16, SDLoc(N));
}]>;

def HA16 : SDNodeXForm<imm, [{
  // Transformation function: shift the immediate value down into the low bits.
  int64_t Val = N->getZExtValue();
  return getI32Imm((Val - (signed short)Val) >> 16, SDLoc(N));
}]>;
```
- **EN**: Adds declarative TableGen records such as `HI16`, `HA16` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HI16`, `HA16`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 820-826

```tablegen
def MB : SDNodeXForm<imm, [{
  // Transformation function: get the start bit of a mask
  unsigned mb = 0, me;
  (void)isRunOfOnes((unsigned)N->getZExtValue(), mb, me);
  return getI32Imm(mb, SDLoc(N));
}]>;
```
- **EN**: Adds declarative TableGen records such as `MB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 827-841

```tablegen
def ME : SDNodeXForm<imm, [{
  // Transformation function: get the end bit of a mask
  unsigned mb, me = 0;
  (void)isRunOfOnes((unsigned)N->getZExtValue(), mb, me);
  return getI32Imm(me, SDLoc(N));
}]>;
def maskimm32 : PatLeaf<(imm), [{
  // maskImm predicate - True if immediate is a run of ones.
  unsigned mb, me;
  if (N->getValueType(0) == MVT::i32)
    return isRunOfOnes((unsigned)N->getZExtValue(), mb, me);
  else
    return false;
}]>;
```
- **EN**: Adds declarative TableGen records such as `ME`, `maskimm32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ME`, `maskimm32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 842-851

```tablegen
def imm32SExt16  : Operand<i32>, ImmLeaf<i32, [{
  // imm32SExt16 predicate - True if the i32 immediate fits in a 16-bit
  // sign extended field.  Used by instructions like 'addi'.
  return (int32_t)Imm == (short)Imm;
}]>;
def imm64SExt16  : Operand<i64>, ImmLeaf<i64, [{
  // imm64SExt16 predicate - True if the i64 immediate fits in a 16-bit
  // sign extended field.  Used by instructions like 'addi'.
  return (int64_t)Imm == (short)Imm;
}]>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "imm32SExt16 predicate - True if the i32 immediate fits in a 16-bit". Notable symbols in this range include `imm32SExt16`, `imm64SExt16`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“imm32SExt16 predicate - True if the i32 immediate fits in a 16-bit”。 该区间中较显眼的符号包括 `imm32SExt16`, `imm64SExt16`。

### Lines 852-859

```tablegen
def immZExt16  : PatLeaf<(imm), [{
  // immZExt16 predicate - True if the immediate fits in a 16-bit zero extended
  // field.  Used by instructions like 'ori'.
  return (uint64_t)N->getZExtValue() == (unsigned short)N->getZExtValue();
}], LO16>;
def immNonAllOneAnyExt8 : ImmLeaf<i32, [{
  return (isInt<8>(Imm) && (Imm != -1)) || (isUInt<8>(Imm) && (Imm != 0xFF));
}]>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "immZExt16 predicate - True if the immediate fits in a 16-bit zero extended". Notable symbols in this range include `immZExt16`, `immNonAllOneAnyExt8`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“immZExt16 predicate - True if the immediate fits in a 16-bit zero extended”。 该区间中较显眼的符号包括 `immZExt16`, `immNonAllOneAnyExt8`。

### Lines 860-867

```tablegen
def i32immNonAllOneNonZero : ImmLeaf<i32, [{ return Imm && (Imm != -1); }]>;
def immSExt5NonZero : ImmLeaf<i32, [{ return Imm && isInt<5>(Imm); }]>;

// imm16Shifted* - These match immediates where the low 16-bits are zero.  There
// are two forms: imm16ShiftedSExt and imm16ShiftedZExt.  These two forms are
// identical in 32-bit mode, but in 64-bit mode, they return true if the
// immediate fits into a sign/zero extended 32-bit immediate (with the low bits
// clear).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "imm16Shifted* - These match immediates where the low 16-bits are zero.  There". Notable symbols in this range include `i32immNonAllOneNonZero`, `immSExt5NonZero`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“imm16Shifted* - These match immediates where the low 16-bits are zero.  There”。 该区间中较显眼的符号包括 `i32immNonAllOneNonZero`, `immSExt5NonZero`。

### Lines 868-884

```tablegen
def imm16ShiftedZExt : PatLeaf<(imm), [{
  // imm16ShiftedZExt predicate - True if only bits in the top 16-bits of the
  // immediate are set.  Used by instructions like 'xoris'.
  return (N->getZExtValue() & ~uint64_t(0xFFFF0000)) == 0;
}], HI16>;

def imm16ShiftedSExt : PatLeaf<(imm), [{
  // imm16ShiftedSExt predicate - True if only bits in the top 16-bits of the
  // immediate are set.  Used by instructions like 'addis'.  Identical to
  // imm16ShiftedZExt in 32-bit mode.
  if (N->getZExtValue() & 0xFFFF) return false;
  if (N->getValueType(0) == MVT::i32)
    return true;
  // For 64-bit, make sure it is sext right.
  return N->getZExtValue() == (uint64_t)(int)N->getZExtValue();
}], HI16>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "imm16ShiftedZExt predicate - True if only bits in the top 16-bits of the". Notable symbols in this range include `imm16ShiftedZExt`, `imm16ShiftedSExt`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“imm16ShiftedZExt predicate - True if only bits in the top 16-bits of the”。 该区间中较显眼的符号包括 `imm16ShiftedZExt`, `imm16ShiftedSExt`。

### Lines 885-900

```tablegen
def imm64ZExt32  : Operand<i64>, ImmLeaf<i64, [{
  // imm64ZExt32 predicate - True if the i64 immediate fits in a 32-bit
  // zero extended field.
  return isUInt<32>(Imm);
}]>;

// This is a somewhat weaker condition than actually checking for 4-byte
// alignment. It is simply checking that the displacement can be represented
// as an immediate that is a multiple of 4 (i.e. the requirements for DS-Form
// instructions).
// But some r+i load/store instructions (such as LD, STD, LDU, etc.) that require
// restricted memrix (4-aligned) constants are alignment sensitive. If these
// offsets are hidden behind TOC entries than the values of the lower-order
// bits cannot be checked directly. As a result, we need to also incorporate
// an alignment check into the relevant patterns.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "imm64ZExt32 predicate - True if the i64 immediate fits in a 32-bit". Notable symbols in this range include `imm64ZExt32`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“imm64ZExt32 predicate - True if the i64 immediate fits in a 32-bit”。 该区间中较显眼的符号包括 `imm64ZExt32`。

### Lines 901-907

```tablegen
def DSFormLoad : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return isOffsetMultipleOf(N, 4) || cast<LoadSDNode>(N)->getAlign() >= 4;
}]>;
def DSFormStore : PatFrag<(ops node:$val, node:$ptr),
                            (store node:$val, node:$ptr), [{
  return isOffsetMultipleOf(N, 4) || cast<StoreSDNode>(N)->getAlign() >= 4;
}]>;
```
- **EN**: Adds declarative TableGen records such as `DSFormLoad`, `DSFormStore` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DSFormLoad`, `DSFormStore`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 908-916

```tablegen
def DSFormSextLoadi32 : PatFrag<(ops node:$ptr), (sextloadi32 node:$ptr), [{
  return isOffsetMultipleOf(N, 4) || cast<LoadSDNode>(N)->getAlign() >= 4;
}]>;
def DSFormPreStore : PatFrag<
                          (ops node:$val, node:$base, node:$offset),
                          (pre_store node:$val, node:$base, node:$offset), [{
  return isOffsetMultipleOf(N, 4) || cast<StoreSDNode>(N)->getAlign() >= 4;
}]>;
```
- **EN**: Adds declarative TableGen records such as `DSFormSextLoadi32`, `DSFormPreStore` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DSFormSextLoadi32`, `DSFormPreStore`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 917-923

```tablegen
def NonDSFormLoad : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return cast<LoadSDNode>(N)->getAlign() < 4 && !isOffsetMultipleOf(N, 4);
}]>;
def NonDSFormStore : PatFrag<(ops node:$val, node:$ptr),
                              (store node:$val, node:$ptr), [{
  return cast<StoreSDNode>(N)->getAlign() < 4 && !isOffsetMultipleOf(N, 4);
}]>;
```
- **EN**: Adds declarative TableGen records such as `NonDSFormLoad`, `NonDSFormStore` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NonDSFormLoad`, `NonDSFormStore`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 924-931

```tablegen
def NonDSFormSextLoadi32 : PatFrag<(ops node:$ptr), (sextloadi32 node:$ptr), [{
  return cast<LoadSDNode>(N)->getAlign() < 4 && !isOffsetMultipleOf(N, 4);
}]>;

// This is a somewhat weaker condition than actually checking for 16-byte
// alignment. It is simply checking that the displacement can be represented
// as an immediate that is a multiple of 16 (i.e. the requirements for DQ-Form
// instructions).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is a somewhat weaker condition than actually checking for 16-byte". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is a somewhat weaker condition than actually checking for 16-byte”。 这些声明会进入生成式模式匹配逻辑。

### Lines 932-938

```tablegen
def quadwOffsetLoad : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return isOffsetMultipleOf(N, 16);
}]>;
def quadwOffsetStore : PatFrag<(ops node:$val, node:$ptr),
                               (store node:$val, node:$ptr), [{
  return isOffsetMultipleOf(N, 16);
}]>;
```
- **EN**: Adds declarative TableGen records such as `quadwOffsetLoad`, `quadwOffsetStore` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `quadwOffsetLoad`, `quadwOffsetStore`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 939-947

```tablegen
def nonQuadwOffsetLoad : PatFrag<(ops node:$ptr), (load node:$ptr), [{
  return !isOffsetMultipleOf(N, 16);
}]>;
def nonQuadwOffsetStore : PatFrag<(ops node:$val, node:$ptr),
                                  (store node:$val, node:$ptr), [{
  return !isOffsetMultipleOf(N, 16);
}]>;

// PatFrag for binary operation whose operands are both non-constant
```
- **EN**: Adds declarative TableGen records such as `nonQuadwOffsetLoad`, `nonQuadwOffsetStore` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `nonQuadwOffsetLoad`, `nonQuadwOffsetStore`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 948-954

```tablegen
class BinOpWithoutSImm16Operand<SDNode opcode> :
  PatFrag<(ops node:$left, node:$right), (opcode node:$left, node:$right), [{
    int16_t Imm;
    return !isIntS16Immediate(N->getOperand(0), Imm)
             && !isIntS16Immediate(N->getOperand(1), Imm);
}]>;
```
- **EN**: Declares a backend-facing type `BinOpWithoutSImm16Operand` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `BinOpWithoutSImm16Operand`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 955-957

```tablegen
def add_without_simm16 : BinOpWithoutSImm16Operand<add>;
def mul_without_simm16 : BinOpWithoutSImm16Operand<mul>;
```
- **EN**: Adds declarative TableGen records such as `add_without_simm16`, `mul_without_simm16` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `add_without_simm16`, `mul_without_simm16`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 958-966

```tablegen
//===----------------------------------------------------------------------===//
// Output pattern fragments.
//

// Create an even/odd register pair.
def PAIR8 : OutPatFrag<(ops node:$even, node:$odd),
                       (REG_SEQUENCE G8pRC, $even, sub_gp8_x0, $odd, sub_gp8_x1)>;

// Return the even part of an even/odd register pair.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Output pattern fragments.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Output pattern fragments.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 967-973

```tablegen
def EVEN8 : OutPatFrag<(ops node:$pair), (EXTRACT_SUBREG $pair, sub_gp8_x0)>;

// Any-extend a 32-bit value in GPRC to a 64-bit value in G8RC.
def AEXT8 : OutPatFrag<(ops node:$r),
                       (INSERT_SUBREG (i64 (IMPLICIT_DEF)), $r, sub_32)>;

// Truncate a 64-bit value in a G8RC value to 32-bit value in GPRC.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Any-extend a 32-bit value in GPRC to a 64-bit value in G8RC.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Any-extend a 32-bit value in GPRC to a 64-bit value in G8RC.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 974-975

```tablegen
def TRUNC4 : OutPatFrag<(ops node:$r), (EXTRACT_SUBREG $r, sub_32)>;
```
- **EN**: Adds declarative TableGen records such as `TRUNC4` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TRUNC4`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 976-989

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC Flag Definitions.

class isPPC64 { bit PPC64 = 1; }
class isRecordForm   { bit RC = 1; }

class RegConstraint<string C> {
  string Constraints = C;
}


// Define PowerPC specific addressing mode.

// d-form
```
- **EN**: Declares a backend-facing type `isPPC64`, `isRecordForm`, `RegConstraint` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `isPPC64`, `isRecordForm`, `RegConstraint`，并勾勒出周边代码会依赖的接口或状态。

### Lines 990-1000

```tablegen
def iaddr    : ComplexPattern<iPTR, 2, "SelectAddrImm",     [], []>; // "stb"
// ds-form
def iaddrX4  : ComplexPattern<iPTR, 2, "SelectAddrImmX4",   [], []>; // "std"
// dq-form
def iaddrX16 : ComplexPattern<iPTR, 2, "SelectAddrImmX16",  [], []>; // "stxv"
// 8LS:d-form
def iaddrX34 : ComplexPattern<iPTR, 2, "SelectAddrImmX34",  [], []>; // "pstxvp"

// Below forms are all x-form addressing mode, use three different ones so we
// can make a accurate check for x-form instructions in ISEL.
// x-form addressing mode whose associated displacement form is D.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "ds-form". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“ds-form”。 这些声明会进入生成式模式匹配逻辑。

### Lines 1001-1010

```tablegen
def xaddr  : ComplexPattern<iPTR, 2, "SelectAddrIdx",     [], []>;    // "stbx"
// x-form addressing mode whose associated displacement form is DS.
def xaddrX4 : ComplexPattern<iPTR, 2, "SelectAddrIdxX4",    [], []>;  // "stdx"
// x-form addressing mode whose associated displacement form is DQ.
def xaddrX16 : ComplexPattern<iPTR, 2, "SelectAddrIdxX16",   [], []>; // "stxvx"

def xoaddr : ComplexPattern<iPTR, 2, "SelectAddrIdxOnly",[], []>;

// The address in a single register. This is used with the SjLj
// pseudo-instructions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "x-form addressing mode whose associated displacement form is DS.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“x-form addressing mode whose associated displacement form is DS.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 1011-1017

```tablegen
def addr   : ComplexPattern<iPTR, 1, "SelectAddr",[], []>;

/// This is just the offset part of iaddr, used for preinc.
def iaddroff : ComplexPattern<iPTR, 1, "SelectAddrImmOffs", [], []>;

// Load and Store Instruction Selection addressing modes.
let WantsParent = true in {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is just the offset part of iaddr, used for preinc.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is just the offset part of iaddr, used for preinc.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 1018-1026

```tablegen
  def DForm  : ComplexPattern<iPTR, 2, "SelectDForm">;
  def DSForm : ComplexPattern<iPTR, 2, "SelectDSForm">;
  def DQForm : ComplexPattern<iPTR, 2, "SelectDQForm">;
  def XForm  : ComplexPattern<iPTR, 2, "SelectXForm">;
  def ForceXForm : ComplexPattern<iPTR, 2, "SelectForceXForm">;
  def PCRelForm : ComplexPattern<iPTR, 2, "SelectPCRelForm">;
  def PDForm : ComplexPattern<iPTR, 2, "SelectPDForm">;
}
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1027-1034

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC Multiclass Definitions.
multiclass XForm_base_r3xo_r<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                      string asmbase, string asmstr, list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : XForm_base_r3xo<opcode, xo, OOL, IOL,
                                !strconcat(asmbase, !strconcat(" ", asmstr)),
                                NoItinerary, pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XForm_base_r3xo_r` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XForm_base_r3xo_r`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 1035-1041

```tablegen
    let Defs = [CR1] in
    def _rec : XForm_base_r3xo<opcode, xo, OOL, IOL,
                               !strconcat(asmbase, !strconcat(". ", asmstr)),
                               NoItinerary, []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这一段包含调度或处理器模型元数据。

### Lines 1042-1048

```tablegen
multiclass XForm_6r<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : XForm_6<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XForm_6r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XForm_6r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1049-1055

```tablegen
    let Defs = [CR0] in
    def _rec    : XForm_6<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1056-1063

```tablegen
multiclass XForm_6rc<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                     string asmbase, string asmstr, InstrItinClass itin,
                     list<dag> pattern> {
  let BaseName = asmbase in {
    let Defs = [CARRY] in
    def NAME : XForm_6<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XForm_6rc` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XForm_6rc`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1064-1070

```tablegen
    let Defs = [CARRY, CR0] in
    def _rec    : XForm_6<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1071-1078

```tablegen
multiclass XForm_10rc<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                      string asmbase, string asmstr, InstrItinClass itin,
                      list<dag> pattern> {
  let BaseName = asmbase in {
    let Defs = [CARRY] in
    def NAME : XForm_10<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XForm_10rc` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XForm_10rc`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1079-1085

```tablegen
    let Defs = [CARRY, CR0] in
    def _rec    : XForm_10<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1086-1092

```tablegen
multiclass XForm_11r<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : XForm_11<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XForm_11r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XForm_11r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1093-1099

```tablegen
    let Defs = [CR0] in
    def _rec    : XForm_11<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1100-1106

```tablegen
multiclass XOForm_1r<bits<6> opcode, bits<9> xo, bit oe, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : XOForm_1<opcode, xo, oe, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XOForm_1r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XOForm_1r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1107-1115

```tablegen
    let Defs = [CR0] in
    def _rec    : XOForm_1<opcode, xo, oe, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}

// Multiclass for instructions which have a record overflow form as well
// as a record form but no carry (i.e. mulld, mulldo, subf, subfo, etc.)
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1116-1122

```tablegen
multiclass XOForm_1rx<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
                      string asmbase, string asmstr, InstrItinClass itin,
                      list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : XOForm_1<opcode, xo, 0, OOL, IOL,
                        !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                        pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XOForm_1rx` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XOForm_1rx`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1123-1129

```tablegen
    let Defs = [CR0] in
    def _rec    : XOForm_1<opcode, xo, 0, OOL, IOL,
                        !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                        []>, isRecordForm, RecFormRel;
  }
  let BaseName = !strconcat(asmbase, "O") in {
    let Defs = [XER] in
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 1130-1141

```tablegen
    def O    : XOForm_1<opcode, xo, 1, OOL, IOL,
                        !strconcat(asmbase, !strconcat("o ", asmstr)), itin,
                        []>, RecFormRel;
    let Defs = [XER, CR0] in
    def O_rec    : XOForm_1<opcode, xo, 1, OOL, IOL,
                         !strconcat(asmbase, !strconcat("o. ", asmstr)), itin,
                         []>, isRecordForm, RecFormRel;
  }
}

// Multiclass for instructions for which the non record form is not cracked
// and the record form is cracked (i.e. divw, mullw, etc.)
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1142-1148

```tablegen
multiclass XOForm_1rcr<bits<6> opcode, bits<9> xo, bit oe, dag OOL, dag IOL,
                      string asmbase, string asmstr, InstrItinClass itin,
                      list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : XOForm_1<opcode, xo, oe, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XOForm_1rcr` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XOForm_1rcr`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1149-1155

```tablegen
    let Defs = [CR0] in
    def _rec    : XOForm_1<opcode, xo, oe, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel, PPC970_DGroup_First,
                       PPC970_DGroup_Cracked;
  }
  let BaseName = !strconcat(asmbase, "O") in {
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 1156-1166

```tablegen
    let Defs = [XER] in
    def O    : XOForm_1<opcode, xo, 1, OOL, IOL,
                        !strconcat(asmbase, !strconcat("o ", asmstr)), itin,
                        []>, RecFormRel;
    let Defs = [XER, CR0] in
    def O_rec   : XOForm_1<opcode, xo, 1, OOL, IOL,
                        !strconcat(asmbase, !strconcat("o. ", asmstr)), itin,
                        []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1167-1174

```tablegen
multiclass XOForm_1rc<bits<6> opcode, bits<9> xo, bit oe, dag OOL, dag IOL,
                      string asmbase, string asmstr, InstrItinClass itin,
                      list<dag> pattern> {
  let BaseName = asmbase in {
    let Defs = [CARRY] in
    def NAME : XOForm_1<opcode, xo, oe, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XOForm_1rc` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XOForm_1rc`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1175-1181

```tablegen
    let Defs = [CARRY, CR0] in
    def _rec    : XOForm_1<opcode, xo, oe, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
  let BaseName = !strconcat(asmbase, "O") in {
    let Defs = [CARRY, XER] in
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 1182-1191

```tablegen
    def O    : XOForm_1<opcode, xo, 1, OOL, IOL,
                        !strconcat(asmbase, !strconcat("o ", asmstr)), itin,
                        []>, RecFormRel;
    let Defs = [CARRY, XER, CR0] in
    def O_rec   : XOForm_1<opcode, xo, 1, OOL, IOL,
                        !strconcat(asmbase, !strconcat("o. ", asmstr)), itin,
                        []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1192-1198

```tablegen
multiclass XOForm_3r<bits<6> opcode, bits<9> xo, bit oe, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : XOForm_3<opcode, xo, oe, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XOForm_3r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XOForm_3r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1199-1205

```tablegen
    let Defs = [CR0] in
    def _rec    : XOForm_3<opcode, xo, oe, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
  let BaseName = !strconcat(asmbase, "O") in {
    let Defs = [XER] in
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 1206-1215

```tablegen
    def O    : XOForm_3<opcode, xo, 1, OOL, IOL,
                        !strconcat(asmbase, !strconcat("o ", asmstr)), itin,
                        []>, RecFormRel;
    let Defs = [XER, CR0] in
    def O_rec   : XOForm_3<opcode, xo, 1, OOL, IOL,
                        !strconcat(asmbase, !strconcat("o. ", asmstr)), itin,
                        []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1216-1223

```tablegen
multiclass XOForm_3rc<bits<6> opcode, bits<9> xo, bit oe, dag OOL, dag IOL,
                      string asmbase, string asmstr, InstrItinClass itin,
                      list<dag> pattern> {
  let BaseName = asmbase in {
    let Defs = [CARRY] in
    def NAME : XOForm_3<opcode, xo, oe, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XOForm_3rc` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XOForm_3rc`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1224-1230

```tablegen
    let Defs = [CARRY, CR0] in
    def _rec    : XOForm_3<opcode, xo, oe, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
  let BaseName = !strconcat(asmbase, "O") in {
    let Defs = [CARRY, XER] in
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 1231-1240

```tablegen
    def O    : XOForm_3<opcode, xo, 1, OOL, IOL,
                        !strconcat(asmbase, !strconcat("o ", asmstr)), itin,
                        []>, RecFormRel;
    let Defs = [CARRY, XER, CR0] in
    def O_rec   : XOForm_3<opcode, xo, 1, OOL, IOL,
                        !strconcat(asmbase, !strconcat("o. ", asmstr)), itin,
                        []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1241-1247

```tablegen
multiclass MForm_1r<bits<6> opcode, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : MForm_1<opcode, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `MForm_1r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MForm_1r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1248-1254

```tablegen
    let Defs = [CR0] in
    def _rec    : MForm_1<opcode, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1255-1261

```tablegen
multiclass MForm_2r<bits<6> opcode, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : MForm_2<opcode, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `MForm_2r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MForm_2r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1262-1268

```tablegen
    let Defs = [CR0] in
    def _rec    : MForm_2<opcode, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1269-1275

```tablegen
multiclass MDForm_1r<bits<6> opcode, bits<3> xo, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : MDForm_1<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `MDForm_1r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MDForm_1r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1276-1282

```tablegen
    let Defs = [CR0] in
    def _rec    : MDForm_1<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1283-1289

```tablegen
multiclass MDSForm_1r<bits<6> opcode, bits<4> xo, dag OOL, dag IOL,
                     string asmbase, string asmstr, InstrItinClass itin,
                     list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : MDSForm_1<opcode, xo, OOL, IOL,
                        !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                        pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `MDSForm_1r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MDSForm_1r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1290-1296

```tablegen
    let Defs = [CR0] in
    def _rec    : MDSForm_1<opcode, xo, OOL, IOL,
                        !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                        []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1297-1304

```tablegen
multiclass XSForm_1rc<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
                      string asmbase, string asmstr, InstrItinClass itin,
                      list<dag> pattern> {
  let BaseName = asmbase in {
    let Defs = [CARRY] in
    def NAME : XSForm_1<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XSForm_1rc` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XSForm_1rc`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1305-1311

```tablegen
    let Defs = [CARRY, CR0] in
    def _rec    : XSForm_1<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1312-1318

```tablegen
multiclass XSForm_1r<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : XSForm_1<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XSForm_1r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XSForm_1r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1319-1325

```tablegen
    let Defs = [CR0] in
    def _rec    : XSForm_1<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1326-1332

```tablegen
multiclass XForm_26r<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : XForm_26<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XForm_26r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XForm_26r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1333-1339

```tablegen
    let Defs = [CR1] in
    def _rec    : XForm_26<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1340-1346

```tablegen
multiclass XForm_28r<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : XForm_28<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `XForm_28r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XForm_28r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1347-1353

```tablegen
    let Defs = [CR1] in
    def _rec    : XForm_28<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1354-1360

```tablegen
multiclass XForm_SP2_FRTB5r<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                        string asmbase, string asmstr, list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : XForm_SP2_FRTB5<opcode, xo, OOL, IOL,
                               !strconcat(asmbase, !strconcat(" ", asmstr)),
                               pattern, NoItinerary>, RecFormRel;
    let Defs = [CR1] in
```
- **EN**: Adds declarative TableGen records such as `XForm_SP2_FRTB5r` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XForm_SP2_FRTB5r`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 1361-1368

```tablegen
    def _rec : XForm_SP2_FRTB5<opcode, xo, OOL, IOL,
                               !strconcat(asmbase, !strconcat(". ", asmstr)),
                               [], NoItinerary>, isRecordForm, RecFormRel;
  }
}

multiclass XForm_S1_FRTB5r<bits<6> opcode, bits<10> xo, dag OOL, dag IOL,
                      string asmbase, string asmstr, list<dag> pattern> {
```
- **EN**: Adds declarative TableGen records such as `XForm_S1_FRTB5r` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XForm_S1_FRTB5r`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 1369-1379

```tablegen
  let BaseName = asmbase in {
    def NAME : XForm_S1_FRTB5<opcode, xo, OOL, IOL,
                              !strconcat(asmbase, !strconcat(" ", asmstr)),
                              pattern, NoItinerary>, RecFormRel;
    let Defs = [CR1] in
    def _rec : XForm_S1_FRTB5<opcode, xo, OOL, IOL,
                              !strconcat(asmbase, !strconcat(". ", asmstr)), [],
                              NoItinerary>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。 这一段包含调度或处理器模型元数据。

### Lines 1380-1386

```tablegen
multiclass AForm_1r<bits<6> opcode, bits<5> xo, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : AForm_1<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `AForm_1r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `AForm_1r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1387-1393

```tablegen
    let Defs = [CR1] in
    def _rec    : AForm_1<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1394-1400

```tablegen
multiclass AForm_2r<bits<6> opcode, bits<5> xo, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : AForm_2<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `AForm_2r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `AForm_2r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1401-1407

```tablegen
    let Defs = [CR1] in
    def _rec    : AForm_2<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1408-1414

```tablegen
multiclass AForm_3r<bits<6> opcode, bits<5> xo, dag OOL, dag IOL,
                    string asmbase, string asmstr, InstrItinClass itin,
                    list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : AForm_3<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `AForm_3r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `AForm_3r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1415-1421

```tablegen
    let Defs = [CR1] in
    def _rec    : AForm_3<opcode, xo, OOL, IOL,
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1422-1429

```tablegen
multiclass
    Z23Form_TE5_FRTB5_RMC2r<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                            string asmbase, string asmstr, list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME
        : Z23Form_TE5_FRTB5_RMC2<opcode, xo, OOL, IOL,
                                 !strconcat(asmbase, !strconcat(" ", asmstr)),
                                 pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `Z23Form_TE5_FRTB5_RMC2r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Z23Form_TE5_FRTB5_RMC2r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1430-1436

```tablegen
    let Defs = [CR0] in
    def _rec : Z23Form_TE5_FRTB5_RMC2<opcode, xo, OOL, IOL,
                                 !strconcat(asmbase, !strconcat(". ", asmstr)),
                                 []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1437-1443

```tablegen
multiclass
    Z23Form_FRTAB5_RMC2r<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                         string asmbase, string asmstr, list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : Z23Form_FRTAB5_RMC2<opcode, xo, OOL, IOL,
                                   !strconcat(asmbase, !strconcat(" ", asmstr)),
                                   pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `Z23Form_FRTAB5_RMC2r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Z23Form_FRTAB5_RMC2r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1444-1450

```tablegen
    let Defs = [CR1] in
    def _rec : Z23Form_FRTAB5_RMC2<opcode, xo, OOL, IOL,
                              !strconcat(asmbase, !strconcat(". ", asmstr)),
                              []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1451-1457

```tablegen
multiclass
    Z23Form_FRTB5_R1_RMC2r<bits<6> opcode, bits<8> xo, dag OOL, dag IOL,
                           string asmbase, string asmstr, list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : Z23Form_FRTB5_R1_RMC2<opcode, xo, OOL, IOL,
                               !strconcat(asmbase, !strconcat(" ", asmstr)),
                               pattern>, RecFormRel;
```
- **EN**: Adds declarative TableGen records such as `Z23Form_FRTB5_R1_RMC2r` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Z23Form_FRTB5_R1_RMC2r`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1458-1464

```tablegen
    let Defs = [CR1] in
    def _rec : Z23Form_FRTB5_R1_RMC2<opcode, xo, OOL, IOL,
                               !strconcat(asmbase, !strconcat(". ", asmstr)),
                               []>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1465-1471

```tablegen
multiclass Z22Form_FRTA5_SH6r<bits<6> opcode, bits<9> xo, dag OOL, dag IOL,
                      string asmbase, string asmstr, list<dag> pattern> {
  let BaseName = asmbase in {
    def NAME : Z22Form_FRTA5_SH6<opcode, xo, OOL, IOL,
                                 !strconcat(asmbase, !strconcat(" ", asmstr)),
                                 pattern, NoItinerary>, RecFormRel;
    let Defs = [CR1] in
```
- **EN**: Adds declarative TableGen records such as `Z22Form_FRTA5_SH6r` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Z22Form_FRTA5_SH6r`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 1472-1477

```tablegen
    def _rec : Z22Form_FRTA5_SH6<opcode, xo, OOL, IOL,
                                 !strconcat(asmbase, !strconcat(". ", asmstr)),
                                 [], NoItinerary>, isRecordForm, RecFormRel;
  }
}
```
- **EN**: Declares function entry points that other backend components call later. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这一段包含调度或处理器模型元数据。

### Lines 1478-1479

```tablegen
//===----------------------------------------------------------------------===//
// END OF MULTICLASS DEFINITIONS
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "END OF MULTICLASS DEFINITIONS".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“END OF MULTICLASS DEFINITIONS”。

### Lines 1480-1481

```tablegen
//===----------------------------------------------------------------------===//
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1482-1488

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC Instruction Definitions.

// Pseudo instructions:

let hasCtrlDep = 1 in {
let Defs = [R1], Uses = [R1] in {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PowerPC Instruction Definitions.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PowerPC Instruction Definitions.”。

### Lines 1489-1497

```tablegen
def ADJCALLSTACKDOWN : PPCEmitTimePseudo<(outs), (ins u32imm:$amt1, u32imm:$amt2),
                              "#ADJCALLSTACKDOWN $amt1 $amt2",
                              [(callseq_start u32imm_timm:$amt1, u32imm_timm:$amt2)]>;
def ADJCALLSTACKUP   : PPCEmitTimePseudo<(outs), (ins u32imm:$amt1, u32imm:$amt2),
                              "#ADJCALLSTACKUP $amt1 $amt2",
                              [(callseq_end u32imm_timm:$amt1, u32imm_timm:$amt2)]>;
}
} // hasCtrlDep
```
- **EN**: Adds declarative TableGen records such as `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1498-1504

```tablegen
let Defs = [R1], Uses = [R1] in
def DYNALLOC : PPCEmitTimePseudo<(outs gprc:$result), (ins gprc:$negsize, memri:$fpsi), "#DYNALLOC",
                       [(set i32:$result,
                             (PPCdynalloc i32:$negsize, iaddr:$fpsi))]>;
def DYNAREAOFFSET : PPCEmitTimePseudo<(outs i32imm:$result), (ins memri:$fpsi), "#DYNAREAOFFSET",
                       [(set i32:$result, (PPCdynareaoffset iaddr:$fpsi))]>;
// Probed alloca to support stack clash protection.
```
- **EN**: Adds declarative TableGen records such as `DYNALLOC`, `DYNAREAOFFSET` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DYNALLOC`, `DYNAREAOFFSET`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1505-1512

```tablegen
let Defs = [R1], Uses = [R1], hasNoSchedulingInfo = 1 in {
def PROBED_ALLOCA_32 : PPCCustomInserterPseudo<(outs gprc:$result),
                         (ins gprc:$negsize, memri:$fpsi), "#PROBED_ALLOCA_32",
                           [(set i32:$result,
                             (PPCprobedalloca i32:$negsize, iaddr:$fpsi))]>;
def PREPARE_PROBED_ALLOCA_32 : PPCEmitTimePseudo<(outs
    gprc:$fp, gprc:$actual_negsize),
    (ins gprc:$negsize, memri:$fpsi), "#PREPARE_PROBED_ALLOCA_32", []>;
```
- **EN**: Adds declarative TableGen records such as `PROBED_ALLOCA_32`, `PREPARE_PROBED_ALLOCA_32` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PROBED_ALLOCA_32`, `PREPARE_PROBED_ALLOCA_32`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 1513-1524

```tablegen
def PREPARE_PROBED_ALLOCA_NEGSIZE_SAME_REG_32 : PPCEmitTimePseudo<(outs
    gprc:$fp, gprc:$actual_negsize),
    (ins gprc:$negsize, memri:$fpsi),
    "#PREPARE_PROBED_ALLOCA_NEGSIZE_SAME_REG_32", []>,
    RegConstraint<"$actual_negsize = $negsize">;
def PROBED_STACKALLOC_32 : PPCEmitTimePseudo<(outs gprc:$scratch, gprc:$temp),
    (ins i64imm:$stacksize),
    "#PROBED_STACKALLOC_32", []>;
}

// SELECT_CC_* - Used to implement the SELECT_CC DAG operation.  Expanded after
// instruction selection into a branch sequence.
```
- **EN**: Adds declarative TableGen records such as `PREPARE_PROBED_ALLOCA_NEGSIZE_SAME_REG_32`, `PROBED_STACKALLOC_32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PREPARE_PROBED_ALLOCA_NEGSIZE_SAME_REG_32`, `PROBED_STACKALLOC_32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1525-1532

```tablegen
let PPC970_Single = 1 in {
  // Note that SELECT_CC_I4 and SELECT_CC_I8 use the no-r0 register classes
  // because either operand might become the first operand in an isel, and
  // that operand cannot be r0.
  def SELECT_CC_I4 : PPCCustomInserterPseudo<(outs gprc:$dst), (ins crrc:$cond,
                              gprc_nor0:$T, gprc_nor0:$F,
                              i32imm:$BROPC), "#SELECT_CC_I4",
                              []>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Note that SELECT_CC_I4 and SELECT_CC_I8 use the no-r0 register classes".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Note that SELECT_CC_I4 and SELECT_CC_I8 use the no-r0 register classes”。

### Lines 1533-1539

```tablegen
  def SELECT_CC_I8 : PPCCustomInserterPseudo<(outs g8rc:$dst), (ins crrc:$cond,
                              g8rc_nox0:$T, g8rc_nox0:$F,
                              i32imm:$BROPC), "#SELECT_CC_I8",
                              []>;
  def SELECT_CC_F4  : PPCCustomInserterPseudo<(outs f4rc:$dst), (ins crrc:$cond, f4rc:$T, f4rc:$F,
                              i32imm:$BROPC), "#SELECT_CC_F4",
                              []>;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1540-1551

```tablegen
  def SELECT_CC_F8  : PPCCustomInserterPseudo<(outs f8rc:$dst), (ins crrc:$cond, f8rc:$T, f8rc:$F,
                              i32imm:$BROPC), "#SELECT_CC_F8",
                              []>;
  def SELECT_CC_F16  : PPCCustomInserterPseudo<(outs vrrc:$dst), (ins crrc:$cond, vrrc:$T, vrrc:$F,
                              i32imm:$BROPC), "#SELECT_CC_F16",
                              []>;
  def SELECT_CC_VRRC: PPCCustomInserterPseudo<(outs vrrc:$dst), (ins crrc:$cond, vrrc:$T, vrrc:$F,
                              i32imm:$BROPC), "#SELECT_CC_VRRC",
                              []>;

  // SELECT_* pseudo instructions, like SELECT_CC_* but taking condition
  // register bit directly.
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1552-1558

```tablegen
  def SELECT_I4 : PPCCustomInserterPseudo<(outs gprc:$dst), (ins crbitrc:$cond,
                          gprc_nor0:$T, gprc_nor0:$F), "#SELECT_I4",
                          [(set i32:$dst, (select i1:$cond, i32:$T, i32:$F))]>;
  def SELECT_I8 : PPCCustomInserterPseudo<(outs g8rc:$dst), (ins crbitrc:$cond,
                          g8rc_nox0:$T, g8rc_nox0:$F), "#SELECT_I8",
                          [(set i64:$dst, (select i1:$cond, i64:$T, i64:$F))]>;
let Predicates = [HasFPU] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1559-1568

```tablegen
  def SELECT_F4  : PPCCustomInserterPseudo<(outs f4rc:$dst), (ins crbitrc:$cond,
                          f4rc:$T, f4rc:$F), "#SELECT_F4",
                          [(set f32:$dst, (select i1:$cond, f32:$T, f32:$F))]>;
  def SELECT_F8  : PPCCustomInserterPseudo<(outs f8rc:$dst), (ins crbitrc:$cond,
                          f8rc:$T, f8rc:$F), "#SELECT_F8",
                          [(set f64:$dst, (select i1:$cond, f64:$T, f64:$F))]>;
  def SELECT_F16  : PPCCustomInserterPseudo<(outs vrrc:$dst), (ins crbitrc:$cond,
                          vrrc:$T, vrrc:$F), "#SELECT_F16",
                          [(set f128:$dst, (select i1:$cond, f128:$T, f128:$F))]>;
}
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1569-1576

```tablegen
  def SELECT_VRRC: PPCCustomInserterPseudo<(outs vrrc:$dst), (ins crbitrc:$cond,
                          vrrc:$T, vrrc:$F), "#SELECT_VRRC",
                          [(set v4i32:$dst,
                                (select i1:$cond, v4i32:$T, v4i32:$F))]>;
}

// SPILL_CR - Indicate that we're dumping the CR register, so we'll need to
// scavenge a register for it.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "SPILL_CR - Indicate that we're dumping the CR register, so we'll need to".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“SPILL_CR - Indicate that we're dumping the CR register, so we'll need to”。

### Lines 1577-1585

```tablegen
let mayStore = 1 in {
def SPILL_CR : PPCEmitTimePseudo<(outs), (ins crrc:$cond, memri:$F),
                     "#SPILL_CR", []>;
def SPILL_CRBIT : PPCEmitTimePseudo<(outs), (ins crbitrc:$cond, memri:$F),
                         "#SPILL_CRBIT", []>;
}

// RESTORE_CR - Indicate that we're restoring the CR register (previously
// spilled), so we'll need to scavenge a register for it.
```
- **EN**: Adds declarative TableGen records such as `SPILL_CR`, `SPILL_CRBIT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SPILL_CR`, `SPILL_CRBIT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1586-1592

```tablegen
let mayLoad = 1 in {
def RESTORE_CR : PPCEmitTimePseudo<(outs crrc:$cond), (ins memri:$F),
                     "#RESTORE_CR", []>;
def RESTORE_CRBIT : PPCEmitTimePseudo<(outs crbitrc:$cond), (ins memri:$F),
                           "#RESTORE_CRBIT", []>;
}
```
- **EN**: Adds declarative TableGen records such as `RESTORE_CR`, `RESTORE_CRBIT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RESTORE_CR`, `RESTORE_CRBIT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1593-1601

```tablegen
let isTerminator = 1, isBarrier = 1, PPC970_Unit = 7, hasSideEffects = 0 in {
  let isPredicable = 1, isReturn = 1, Uses = [LR, RM] in
    def BLR : XLForm_2_ext<19, 16, 20, 0, 0, (outs), (ins), "blr", IIC_BrB,
                           [(PPCretglue)]>, Requires<[IsPPC32]>;
  let isBranch = 1, isIndirectBranch = 1, Uses = [CTR] in {
    let isPredicable = 1 in
      def BCTR : XLForm_2_ext<19, 528, 20, 0, 0, (outs), (ins), "bctr", IIC_BrB,
                              []>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1602-1608

```tablegen
    let isCodeGenOnly = 1 in {
      def BCCCTR : XLForm_2_br<19, 528, 0, (outs), (ins (pred $BIBO, $CR):$cond),
                               "b${cond:cc}ctr${cond:pm} ${cond:reg}", IIC_BrB,
                               []>;

      def BCCTR :  XLForm_2_br2<19, 528, 12, 0, (outs), (ins crbitrc:$BI),
                                "bcctr 12, $BI, 0", IIC_BrB, []>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 1609-1615

```tablegen
      def BCCTRn : XLForm_2_br2<19, 528, 4, 0, (outs), (ins crbitrc:$BI),
                                "bcctr 4, $BI, 0", IIC_BrB, []>;
    }
  }
}

// Set the float rounding mode.
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1616-1622

```tablegen
let Uses = [RM], Defs = [RM] in {
def SETRNDi : PPCCustomInserterPseudo<(outs f8rc:$FRT), (ins u2imm:$RND),
                    "#SETRNDi", [(set f64:$FRT, (int_ppc_setrnd (i32 imm:$RND)))]>;

def SETRND : PPCCustomInserterPseudo<(outs f8rc:$FRT), (ins gprc:$in),
                    "#SETRND", [(set f64:$FRT, (int_ppc_setrnd gprc :$in))]>;
```
- **EN**: Adds declarative TableGen records such as `SETRNDi`, `SETRND` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SETRNDi`, `SETRND`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1623-1629

```tablegen
def SETFLM : PPCCustomInserterPseudo<(outs f8rc:$FRT), (ins f8rc:$FLM),
                    "#SETFLM", [(set f64:$FRT, (int_ppc_setflm f8rc:$FLM))]>;
}

let isBarrier = 1, hasSideEffects = 1, Defs = [RM] in
def FENCE : PPCEmitTimePseudo<(outs), (ins), "#FENCE", []>;
```
- **EN**: Adds declarative TableGen records such as `SETFLM`, `FENCE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SETFLM`, `FENCE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1630-1636

```tablegen
let Defs = [LR] in
  def MovePCtoLR : PPCEmitTimePseudo<(outs), (ins), "#MovePCtoLR", []>,
                   PPC970_Unit_BRU;
let Defs = [LR] in
  def MoveGOTtoLR : PPCEmitTimePseudo<(outs), (ins), "#MoveGOTtoLR", []>,
                    PPC970_Unit_BRU;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1637-1643

```tablegen
let isBranch = 1, isTerminator = 1, hasCtrlDep = 1, PPC970_Unit = 7,
    hasSideEffects = 0 in {
  let isBarrier = 1 in {
    let isPredicable = 1 in
      def B : IForm<18, 0, 0, (outs), (ins directbrtarget:$LI),
                    "b $LI", IIC_BrB,
                    [(br bb:$LI)]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1644-1650

```tablegen
  def BA  : IForm<18, 1, 0, (outs), (ins absdirectbrtarget:$LI),
                  "ba $LI", IIC_BrB, []>;
  }

  // BCC represents an arbitrary conditional branch on a predicate.
  // FIXME: should be able to write a pattern for PPCcondbranch, but can't use
  // a two-value operand where a dag node expects two operands. :(
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "BCC represents an arbitrary conditional branch on a predicate.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“BCC represents an arbitrary conditional branch on a predicate.”。

### Lines 1651-1658

```tablegen
  let isCodeGenOnly = 1 in {
    class BCC_class : BForm<16, 0, 0, (outs), (ins (pred $BIBO, $CR):$cond, condbrtarget:$BD),
                            "b${cond:cc}${cond:pm} ${cond:reg}, $BD"
                            /*[(PPCcondbranch crrc:$crS, imm:$opc, bb:$BD)]*/>;
    def BCC : BCC_class;

    // The same as BCC, except that it's not a terminator. Used for introducing
    // control flow dependency without creating new blocks.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "[(PPCcondbranch crrc:$crS, imm:$opc, bb:$BD)]*/>;".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“[(PPCcondbranch crrc:$crS, imm:$opc, bb:$BD)]*/>;”。

### Lines 1659-1668

```tablegen
    let isTerminator = 0 in def CTRL_DEP : BCC_class;

    def BCCA : BForm<16, 1, 0, (outs), (ins (pred $BIBO, $CR):$cond, abscondbrtarget:$BD),
                     "b${cond:cc}a${cond:pm} ${cond:reg}, $BD">;

    let isReturn = 1, Uses = [LR, RM] in
    def BCCLR : XLForm_2_br<19, 16, 0, (outs), (ins (pred $BIBO, $CR):$cond),
                           "b${cond:cc}lr${cond:pm} ${cond:reg}", IIC_BrB, []>;
  }
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 1669-1677

```tablegen
  let isCodeGenOnly = 1 in {
    let Pattern = [(brcond i1:$BI, bb:$BD)] in
    def BC  : BForm_4<16, 12, 0, 0, (outs), (ins crbitrc:$BI, condbrtarget:$BD),
             "bc 12, $BI, $BD">;

    let Pattern = [(brcond (not i1:$BI), bb:$BD)] in
    def BCn : BForm_4<16, 4, 0, 0, (outs), (ins crbitrc:$BI, condbrtarget:$BD),
             "bc 4, $BI, $BD">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 1678-1685

```tablegen
    let isReturn = 1, Uses = [LR, RM] in {
    def BCLR  : XLForm_2_br2<19, 16, 12, 0, (outs), (ins crbitrc:$BI),
                             "bclr 12, $BI, 0", IIC_BrB, []>;
    def BCLRn : XLForm_2_br2<19, 16, 4, 0, (outs), (ins crbitrc:$BI),
                             "bclr 4, $BI, 0", IIC_BrB, []>;
    }
  }
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1686-1692

```tablegen
  let isReturn = 1, Defs = [CTR], Uses = [CTR, LR, RM] in {
   def BDZLR  : XLForm_2_ext<19, 16, 18, 0, 0, (outs), (ins),
                             "bdzlr", IIC_BrB, []>;
   def BDNZLR : XLForm_2_ext<19, 16, 16, 0, 0, (outs), (ins),
                             "bdnzlr", IIC_BrB, []>;
   def BDZLRp : XLForm_2_ext<19, 16, 27, 0, 0, (outs), (ins),
                             "bdzlr+", IIC_BrB, []>;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1693-1700

```tablegen
   def BDNZLRp: XLForm_2_ext<19, 16, 25, 0, 0, (outs), (ins),
                             "bdnzlr+", IIC_BrB, []>;
   def BDZLRm : XLForm_2_ext<19, 16, 26, 0, 0, (outs), (ins),
                             "bdzlr-", IIC_BrB, []>;
   def BDNZLRm: XLForm_2_ext<19, 16, 24, 0, 0, (outs), (ins),
                             "bdnzlr-", IIC_BrB, []>;
  }
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1701-1707

```tablegen
  let Defs = [CTR], Uses = [CTR] in {
    def BDZ  : BForm_1<16, 18, 0, 0, (outs), (ins condbrtarget:$BD),
                       "bdz $BD">;
    def BDNZ : BForm_1<16, 16, 0, 0, (outs), (ins condbrtarget:$BD),
                       "bdnz $BD">;
    def BDZA  : BForm_1<16, 18, 1, 0, (outs), (ins abscondbrtarget:$BD),
                        "bdza $BD">;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1708-1715

```tablegen
    def BDNZA : BForm_1<16, 16, 1, 0, (outs), (ins abscondbrtarget:$BD),
                        "bdnza $BD">;
    def BDZp : BForm_1<16, 27, 0, 0, (outs), (ins condbrtarget:$BD),
                       "bdz+ $BD">;
    def BDNZp: BForm_1<16, 25, 0, 0, (outs), (ins condbrtarget:$BD),
                       "bdnz+ $BD">;
    def BDZAp : BForm_1<16, 27, 1, 0, (outs), (ins abscondbrtarget:$BD),
                        "bdza+ $BD">;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1716-1723

```tablegen
    def BDNZAp: BForm_1<16, 25, 1, 0, (outs), (ins abscondbrtarget:$BD),
                        "bdnza+ $BD">;
    def BDZm : BForm_1<16, 26, 0, 0, (outs), (ins condbrtarget:$BD),
                       "bdz- $BD">;
    def BDNZm: BForm_1<16, 24, 0, 0, (outs), (ins condbrtarget:$BD),
                       "bdnz- $BD">;
    def BDZAm : BForm_1<16, 26, 1, 0, (outs), (ins abscondbrtarget:$BD),
                        "bdza- $BD">;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1724-1731

```tablegen
    def BDNZAm: BForm_1<16, 24, 1, 0, (outs), (ins abscondbrtarget:$BD),
                        "bdnza- $BD">;
  }
}

// The unconditional BCL used by the SjLj setjmp code.
let isCall = 1, hasCtrlDep = 1, isCodeGenOnly = 1, PPC970_Unit = 7,
    hasSideEffects = 0 in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1732-1739

```tablegen
  let Defs = [LR], Uses = [RM] in {
    def BCLalways  : BForm_2<16, 20, 31, 0, 1, (outs), (ins condbrtarget:$BD),
                            "bcl 20, 31, $BD">;
  }
}

let isCall = 1, PPC970_Unit = 7, Defs = [LR] in {
  // Convenient aliases for call instructions
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1740-1746

```tablegen
  let Uses = [RM] in {
    def BL  : IForm<18, 0, 1, (outs), (ins calltarget:$LI),
                    "bl $LI", IIC_BrB, []>;  // See Pat patterns below.
    def BLA : IForm<18, 1, 1, (outs), (ins abscalltarget:$LI),
                    "bla $LI", IIC_BrB, [(PPCcall (i32 imm:$LI))]>;

    let isCodeGenOnly = 1 in {
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 1747-1753

```tablegen
      def BL_TLS  : IForm<18, 0, 1, (outs), (ins tlscall32:$LI),
                          "bl $LI", IIC_BrB, []>;
      def BCCL : BForm<16, 0, 1, (outs), (ins (pred $BIBO, $CR):$cond, condbrtarget:$BD),
                       "b${cond:cc}l${cond:pm} ${cond:reg}, $BD">;
      def BCCLA : BForm<16, 1, 1, (outs), (ins (pred $BIBO, $CR):$cond, abscondbrtarget:$BD),
                        "b${cond:cc}la${cond:pm} ${cond:reg}, $BD">;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 1754-1764

```tablegen
      def BCL  : BForm_4<16, 12, 0, 1, (outs),
                         (ins crbitrc:$BI, condbrtarget:$BD),
                         "bcl 12, $BI, $BD">;
      def BCLn : BForm_4<16, 4, 0, 1, (outs),
                         (ins crbitrc:$BI, condbrtarget:$BD),
                         "bcl 4, $BI, $BD">;
      def BL_NOP  : IForm_and_DForm_4_zero<18, 0, 1, 24,
                                           (outs), (ins calltarget:$LI),
                                           "bl $LI\n\tnop", IIC_BrB, []>;
    }
  }
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1765-1771

```tablegen
  let Uses = [CTR, RM] in {
    let isPredicable = 1 in
      def BCTRL : XLForm_2_ext<19, 528, 20, 0, 1, (outs), (ins),
                              "bctrl", IIC_BrB, [(PPCbctrl)]>,
                  Requires<[IsPPC32]>;

    let isCodeGenOnly = 1 in {
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1772-1781

```tablegen
      def BCCCTRL : XLForm_2_br<19, 528, 1, (outs), (ins (pred $BIBO, $CR):$cond),
                                "b${cond:cc}ctrl${cond:pm} ${cond:reg}", IIC_BrB,
                                []>;

      def BCCTRL  : XLForm_2_br2<19, 528, 12, 1, (outs), (ins crbitrc:$BI),
                                 "bcctrl 12, $BI, 0", IIC_BrB, []>;
      def BCCTRLn : XLForm_2_br2<19, 528, 4, 1, (outs), (ins crbitrc:$BI),
                                 "bcctrl 4, $BI, 0", IIC_BrB, []>;
    }
  }
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 1782-1790

```tablegen
  let Uses = [LR, RM] in {
    def BLRL : XLForm_2_ext<19, 16, 20, 0, 1, (outs), (ins),
                            "blrl", IIC_BrB, []>;

    let isCodeGenOnly = 1 in {
      def BCCLRL : XLForm_2_br<19, 16, 1, (outs), (ins (pred $BIBO, $CR):$cond),
                              "b${cond:cc}lrl${cond:pm} ${cond:reg}", IIC_BrB,
                              []>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 1791-1797

```tablegen
      def BCLRL  : XLForm_2_br2<19, 16, 12, 1, (outs), (ins crbitrc:$BI),
                                "bclrl 12, $BI, 0", IIC_BrB, []>;
      def BCLRLn : XLForm_2_br2<19, 16, 4, 1, (outs), (ins crbitrc:$BI),
                                "bclrl 4, $BI, 0", IIC_BrB, []>;
    }
  }
  let Defs = [CTR], Uses = [CTR, RM] in {
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1798-1805

```tablegen
    def BDZL  : BForm_1<16, 18, 0, 1, (outs), (ins condbrtarget:$BD),
                        "bdzl $BD">;
    def BDNZL : BForm_1<16, 16, 0, 1, (outs), (ins condbrtarget:$BD),
                        "bdnzl $BD">;
    def BDZLA  : BForm_1<16, 18, 1, 1, (outs), (ins abscondbrtarget:$BD),
                         "bdzla $BD">;
    def BDNZLA : BForm_1<16, 16, 1, 1, (outs), (ins abscondbrtarget:$BD),
                         "bdnzla $BD">;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1806-1813

```tablegen
    def BDZLp : BForm_1<16, 27, 0, 1, (outs), (ins condbrtarget:$BD),
                        "bdzl+ $BD">;
    def BDNZLp: BForm_1<16, 25, 0, 1, (outs), (ins condbrtarget:$BD),
                        "bdnzl+ $BD">;
    def BDZLAp : BForm_1<16, 27, 1, 1, (outs), (ins abscondbrtarget:$BD),
                         "bdzla+ $BD">;
    def BDNZLAp: BForm_1<16, 25, 1, 1, (outs), (ins abscondbrtarget:$BD),
                         "bdnzla+ $BD">;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1814-1822

```tablegen
    def BDZLm : BForm_1<16, 26, 0, 1, (outs), (ins condbrtarget:$BD),
                        "bdzl- $BD">;
    def BDNZLm: BForm_1<16, 24, 0, 1, (outs), (ins condbrtarget:$BD),
                        "bdnzl- $BD">;
    def BDZLAm : BForm_1<16, 26, 1, 1, (outs), (ins abscondbrtarget:$BD),
                         "bdzla- $BD">;
    def BDNZLAm: BForm_1<16, 24, 1, 1, (outs), (ins abscondbrtarget:$BD),
                         "bdnzla- $BD">;
  }
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1823-1829

```tablegen
  let Defs = [CTR], Uses = [CTR, LR, RM] in {
    def BDZLRL  : XLForm_2_ext<19, 16, 18, 0, 1, (outs), (ins),
                               "bdzlrl", IIC_BrB, []>;
    def BDNZLRL : XLForm_2_ext<19, 16, 16, 0, 1, (outs), (ins),
                               "bdnzlrl", IIC_BrB, []>;
    def BDZLRLp : XLForm_2_ext<19, 16, 27, 0, 1, (outs), (ins),
                               "bdzlrl+", IIC_BrB, []>;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1830-1838

```tablegen
    def BDNZLRLp: XLForm_2_ext<19, 16, 25, 0, 1, (outs), (ins),
                               "bdnzlrl+", IIC_BrB, []>;
    def BDZLRLm : XLForm_2_ext<19, 16, 26, 0, 1, (outs), (ins),
                               "bdzlrl-", IIC_BrB, []>;
    def BDNZLRLm: XLForm_2_ext<19, 16, 24, 0, 1, (outs), (ins),
                               "bdnzlrl-", IIC_BrB, []>;
  }
}
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1839-1846

```tablegen
let isCall = 1, PPC970_Unit = 7, Defs = [LR, RM], isCodeGenOnly = 1 in {
  // Convenient aliases for call instructions
  let Uses = [RM] in {
    def BL_RM  : IForm<18, 0, 1, (outs), (ins calltarget:$LI),
                       "bl $LI", IIC_BrB, []>;  // See Pat patterns below.
    def BLA_RM : IForm<18, 1, 1, (outs), (ins abscalltarget:$LI),
                       "bla $LI", IIC_BrB, [(PPCcall_rm (i32 imm:$LI))]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1847-1858

```tablegen
    def BL_NOP_RM  : IForm_and_DForm_4_zero<18, 0, 1, 24,
                                            (outs), (ins calltarget:$LI),
                                            "bl $LI\n\tnop", IIC_BrB, []>;
  }
  let Uses = [CTR, RM] in {
    let isPredicable = 1 in
      def BCTRL_RM : XLForm_2_ext<19, 528, 20, 0, 1, (outs), (ins),
                                  "bctrl", IIC_BrB, [(PPCbctrl_rm)]>,
                  Requires<[IsPPC32]>;
  }
}
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1859-1865

```tablegen
let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [RM] in
def TCRETURNdi :PPCEmitTimePseudo< (outs),
                        (ins calltarget:$dst, i32imm:$offset),
                 "#TC_RETURNd $dst $offset",
                 []>;
```
- **EN**: Adds declarative TableGen records such as `TCRETURNdi` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TCRETURNdi`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1866-1875

```tablegen
let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [RM] in
def TCRETURNai :PPCEmitTimePseudo<(outs), (ins abscalltarget:$func, i32imm:$offset),
                 "#TC_RETURNa $func $offset",
                 [(PPCtc_return (i32 imm:$func), imm:$offset)]>;

let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [RM] in
def TCRETURNri : PPCEmitTimePseudo<(outs), (ins CTRRC:$dst, i32imm:$offset),
                 "#TC_RETURNr $dst $offset",
                 []>;
```
- **EN**: Adds declarative TableGen records such as `TCRETURNai`, `TCRETURNri` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TCRETURNai`, `TCRETURNri`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1876-1884

```tablegen
let isCall = 1, PPC970_Unit = 7, isCodeGenOnly = 1,
    Defs = [LR, R2], Uses = [CTR, RM], RST = 2 in {
  def BCTRL_LWZinto_toc:
    XLForm_2_ext_and_DForm_1<19, 528, 20, 0, 1, 32, (outs),
     (ins (memri $D, $RA):$addr), "bctrl\n\tlwz 2, $addr", IIC_BrB,
     [(PPCbctrl_load_toc iaddr:$addr)]>, Requires<[IsPPC32]>;

}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1885-1893

```tablegen
let Predicates = [IsAIX] in {
  let isCall = 1, PPC970_Unit = 7, isCodeGenOnly = 1,
      Defs = [LR, R2], Uses = [RM, R1], RST = 2, RA = 1, D = 20 in {
  def BL_LWZinto_toc : IForm_and_DForm_1<18, 0, 1, 32,
                     (outs), (ins calltarget:$LI),
                     "bl $LI\n\tlwz 2, 20(1)", IIC_BrB,
                      []>, Requires<[IsPPC32]>;
  }
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1894-1903

```tablegen
  let isCall = 1, PPC970_Unit = 7, isCodeGenOnly = 1,
      Defs = [LR, R2, RM], Uses = [RM, R1], RST = 2, RA = 1, D = 20 in {
    def BL_LWZinto_toc_RM :
      IForm_and_DForm_1<18, 0, 1, 32,
        (outs), (ins calltarget:$LI),
        "bl $LI\n\tlwz 2, 20(1)", IIC_BrB,
        []>, Requires<[IsPPC32]>;
  }
}
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1904-1912

```tablegen
let isCall = 1, PPC970_Unit = 7, isCodeGenOnly = 1,
    Defs = [LR, R2, RM], Uses = [CTR, RM], RST = 2 in {
  def BCTRL_LWZinto_toc_RM:
    XLForm_2_ext_and_DForm_1<19, 528, 20, 0, 1, 32, (outs),
     (ins (memri $D, $RA):$addr), "bctrl\n\tlwz 2, $addr", IIC_BrB,
     [(PPCbctrl_load_toc_rm iaddr:$addr)]>, Requires<[IsPPC32]>;

}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1913-1919

```tablegen
let isCodeGenOnly = 1, hasSideEffects = 0 in {

let isTerminator = 1, isBarrier = 1, PPC970_Unit = 7, isBranch = 1,
    isIndirectBranch = 1, isCall = 1, isReturn = 1, Uses = [CTR, RM]  in
def TAILBCTR : XLForm_2_ext<19, 528, 20, 0, 0, (outs), (ins), "bctr", IIC_BrB,
                            []>, Requires<[IsPPC32]>;
```
- **EN**: Adds declarative TableGen records such as `TAILBCTR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TAILBCTR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1920-1927

```tablegen
let isBranch = 1, isTerminator = 1, hasCtrlDep = 1, PPC970_Unit = 7,
    isBarrier = 1, isCall = 1, isReturn = 1, Uses = [RM] in
def TAILB   : IForm<18, 0, 0, (outs), (ins calltarget:$LI),
                  "b $LI", IIC_BrB,
                  []>;

let isBranch = 1, isTerminator = 1, hasCtrlDep = 1, PPC970_Unit = 7,
    isBarrier = 1, isCall = 1, isReturn = 1, Uses = [RM] in
```
- **EN**: Adds declarative TableGen records such as `TAILB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TAILB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1928-1935

```tablegen
def TAILBA   : IForm<18, 0, 0, (outs), (ins abscalltarget:$LI),
                  "ba $LI", IIC_BrB,
                  []>;

}

// While longjmp is a control-flow barrier (fallthrough isn't allowed), setjmp
// is not.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "While longjmp is a control-flow barrier (fallthrough isn't allowed), setjmp". Notable symbols in this range include `TAILBA`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“While longjmp is a control-flow barrier (fallthrough isn't allowed), setjmp”。 该区间中较显眼的符号包括 `TAILBA`。

### Lines 1936-1943

```tablegen
let hasSideEffects = 1 in {
  let Defs = [CTR] in
  def EH_SjLj_SetJmp32  : PPCCustomInserterPseudo<(outs gprc:$dst), (ins memr:$buf),
                            "#EH_SJLJ_SETJMP32",
                            [(set i32:$dst, (PPCeh_sjlj_setjmp addr:$buf))]>,
                          Requires<[IsPPC32]>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1944-1954

```tablegen
let hasSideEffects = 1, isBarrier = 1 in {
  let isTerminator = 1 in
  def EH_SjLj_LongJmp32 : PPCCustomInserterPseudo<(outs), (ins memr:$buf),
                            "#EH_SJLJ_LONGJMP32",
                            [(PPCeh_sjlj_longjmp addr:$buf)]>,
                          Requires<[IsPPC32]>;
}

// This pseudo is never removed from the function, as it serves as
// a terminator.  Size is set to 0 to prevent the builtin assembler
// from emitting it.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This pseudo is never removed from the function, as it serves as".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This pseudo is never removed from the function, as it serves as”。

### Lines 1955-1961

```tablegen
let isBranch = 1, isTerminator = 1, Size = 0 in {
  def EH_SjLj_Setup : PPCEmitTimePseudo<(outs), (ins directbrtarget:$dst),
                        "#EH_SjLj_Setup\t$dst", []>;
}

// System call.
let PPC970_Unit = 7 in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1962-1968

```tablegen
  def SC     : SCForm<17, 1, 0, (outs), (ins i32imm:$LEV),
                      "sc $LEV", IIC_BrB, []>;
}

// We mark SCV as having no scheduling model since it is only meant to be used
// as inline assembly. If we implement a builtin pattern for it we will need to
// add it to the P9 and P10 scheduling models.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "We mark SCV as having no scheduling model since it is only meant to be used".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“We mark SCV as having no scheduling model since it is only meant to be used”。

### Lines 1969-1977

```tablegen
let Predicates = [IsISA3_0], hasNoSchedulingInfo = 1 in {
  def SCV : SCForm<17, 0, 1, (outs), (ins i32imm:$LEV),
                   "scv $LEV", IIC_BrB, []>;
}

// Branch history rolling buffer.
def CLRBHRB : XForm_0<31, 430, (outs), (ins), "clrbhrb", IIC_BrB, []>,
                      PPC970_DGroup_Single;
```
- **EN**: Adds declarative TableGen records such as `CLRBHRB` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CLRBHRB`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1978-1984

```tablegen
def MFBHRBE : XFXForm_3p<31, 302, (outs gprc:$RT), (ins u10imm:$imm),
                         "mfbhrbe $RT, $imm", IIC_BrB, []>,
                         PPC970_DGroup_First;

def RFEBB : XLForm_S<19, 146, (outs), (ins u1imm:$S), "rfebb $S", IIC_BrB, []>,
                     PPC970_DGroup_Single;
```
- **EN**: Adds declarative TableGen records such as `MFBHRBE`, `RFEBB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MFBHRBE`, `RFEBB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1985-1993

```tablegen
def : InstAlias<"rfebb", (RFEBB 1)>;

// DCB* instructions.
def DCBA   : DCB_Form<758, 0, (outs), (ins (memrr $RA, $RB):$addr), "dcba $addr",
                      IIC_LdStDCBF, [(int_ppc_dcba xoaddr:$addr)]>,
                      PPC970_DGroup_Single;
def DCBI   : DCB_Form<470, 0, (outs), (ins (memrr $RA, $RB):$addr), "dcbi $addr",
                      IIC_LdStDCBF, [(int_ppc_dcbi xoaddr:$addr)]>,
                      PPC970_DGroup_Single;
```
- **EN**: Adds declarative TableGen records such as `DCBA`, `DCBI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DCBA`, `DCBI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1994-2003

```tablegen
def DCBST  : DCB_Form<54, 0, (outs), (ins (memrr $RA, $RB):$addr), "dcbst $addr",
                      IIC_LdStDCBF, [(int_ppc_dcbst xoaddr:$addr)]>,
                      PPC970_DGroup_Single;
def DCBZ   : DCB_Form<1014, 0, (outs), (ins (memrr $RA, $RB):$addr), "dcbz $addr",
                      IIC_LdStDCBF, [(int_ppc_dcbz xoaddr:$addr)]>,
                      PPC970_DGroup_Single;
def DCBZL  : DCB_Form<1014, 1, (outs), (ins (memrr $RA, $RB):$addr), "dcbzl $addr",
                      IIC_LdStDCBF, [(int_ppc_dcbzl xoaddr:$addr)]>,
                      PPC970_DGroup_Single;
```
- **EN**: Adds declarative TableGen records such as `DCBST`, `DCBZ`, `DCBZL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DCBST`, `DCBZ`, `DCBZL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2004-2011

```tablegen
def DCBF   : DCB_Form_hint<86, (outs), (ins u3imm:$TH, (memrr $RA, $RB):$addr),
                      "dcbf $addr, $TH", IIC_LdStDCBF, []>,
                      PPC970_DGroup_Single;

let hasSideEffects = 0, mayLoad = 1, mayStore = 1 in {
def DCBT   : DCB_Form_hint<278, (outs), (ins u5imm:$TH, (memrr $RA, $RB):$addr),
                      "dcbt $addr, $TH", IIC_LdStDCBF, []>,
                      PPC970_DGroup_Single;
```
- **EN**: Adds declarative TableGen records such as `DCBF`, `DCBT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DCBF`, `DCBT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2012-2018

```tablegen
def DCBTST : DCB_Form_hint<246, (outs), (ins u5imm:$TH, (memrr $RA, $RB):$addr),
                      "dcbtst $addr, $TH", IIC_LdStDCBF, []>,
                      PPC970_DGroup_Single;
} // hasSideEffects = 0

def ICBLC  : XForm_icbt<31, 230, (outs), (ins u4imm:$CT, (memrr $RA, $RB):$addr),
                       "icblc $CT, $addr", IIC_LdStStore>, Requires<[HasICBT]>;
```
- **EN**: Adds declarative TableGen records such as `DCBTST`, `ICBLC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DCBTST`, `ICBLC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2019-2025

```tablegen
def ICBLQ  : XForm_icbt<31, 198, (outs), (ins u4imm:$CT, (memrr $RA, $RB):$addr),
                       "icblq. $CT, $addr", IIC_LdStLoad>, Requires<[HasICBT]>;
def ICBT  : XForm_icbt<31, 22, (outs), (ins u4imm:$CT, (memrr $RA, $RB):$addr),
                       "icbt $CT, $addr", IIC_LdStLoad>, Requires<[HasICBT]>;
def ICBTLS : XForm_icbt<31, 486, (outs), (ins u4imm:$CT, (memrr $RA, $RB):$addr),
                       "icbtls $CT, $addr", IIC_LdStLoad>, Requires<[HasICBT]>;
```
- **EN**: Adds declarative TableGen records such as `ICBLQ`, `ICBT`, `ICBTLS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ICBLQ`, `ICBT`, `ICBTLS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2026-2034

```tablegen
def : Pat<(int_ppc_dcbt xoaddr:$dst),
          (DCBT 0, xoaddr:$dst)>;
def : Pat<(int_ppc_dcbtst xoaddr:$dst),
          (DCBTST 0, xoaddr:$dst)>;
def : Pat<(int_ppc_dcbf xoaddr:$dst),
          (DCBF 0, xoaddr:$dst)>;
def : Pat<(int_ppc_icbt xoaddr:$dst),
          (ICBT 0, xoaddr:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2035-2041

```tablegen
def : Pat<(prefetch xoaddr:$dst, (i32 0), timm, (i32 1)),
          (DCBT 0, xoaddr:$dst)>;   // data prefetch for loads
def : Pat<(prefetch xoaddr:$dst, (i32 1), timm, (i32 1)),
          (DCBTST 0, xoaddr:$dst)>; // data prefetch for stores
def : Pat<(prefetch xoaddr:$dst, (i32 0), timm, (i32 0)),
          (ICBT 0, xoaddr:$dst)>, Requires<[HasICBT]>; // inst prefetch (for read)
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2042-2058

```tablegen
def : Pat<(int_ppc_dcbt_with_hint xoaddr:$dst, i32:$TH),
          (DCBT i32:$TH, xoaddr:$dst)>;
def : Pat<(int_ppc_dcbtst_with_hint xoaddr:$dst, i32:$TH),
          (DCBTST i32:$TH, xoaddr:$dst)>;

// Atomic operations
// FIXME: some of these might be used with constant operands. This will result
// in constant materialization instructions that may be redundant. We currently
// clean this up in PPCMIPeephole with calls to
// PPCInstrInfo::convertToImmediateForm() but we should probably not emit them
// in the first place.

// For each operation, 2 pseudo instructions are generated. Which one is matched
// depends on the size and and if the word-part feature is available.
// For example for operation AND, ATOMIC_LOAD_AND_NOWP is only matched for sizes
// 8 and 16 bit if the word part feature is not available. In all other cases,
// ATOMIC_LOAD_AND is matched.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Atomic operations".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Atomic operations”。

### Lines 2059-2067

```tablegen
foreach op = ["load_add", "load_sub", "load_and", "load_or", "load_xor",
              "load_nand", "load_min", "load_max", "load_umax", "load_umin",
              "swap"] in {
  defvar atomic_load = "ATOMIC_"#!toupper(op);
  defvar atomic_load_nowp = "ATOMIC_"#!toupper(op)#"_NOWP";
  let Defs = [CR0] in {
    def atomic_load : PPCCustomInserterPseudo<
        (outs gprc:$dst), (ins memrr:$ptr, i32imm:$sz, gprc:$incr),
        "#" # NAME,[]>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 2068-2074

```tablegen
    def atomic_load_nowp : PPCCustomInserterPseudo<
        (outs gprc:$dst), (ins memrr:$ptr, i32imm:$sz, gprc:$incr),
        "#" # NAME,[]>;
  }

  foreach bitsz = [8, 16] in {
    defvar pat = !cast<PatFrag>(!tolower(atomic_load)#"_i"#bitsz);
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 2075-2081

```tablegen
    def : Pat<(i32 (pat ForceXForm:$ptr, gprc:$incr)),
            (!cast<Instruction>(atomic_load) memrr:$ptr, !div(bitsz, 8), gprc:$incr)>,
            Requires<[HasPartwordAtomics]>;
    def : Pat<(i32 (pat ForceXForm:$ptr, gprc:$incr)),
            (!cast<Instruction>(atomic_load_nowp) memrr:$ptr, !div(bitsz, 8), gprc:$incr)>;
  }
  defvar pat = !cast<PatFrag>(!tolower(atomic_load)#"_i32");
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 2082-2088

```tablegen
  def : Pat<(i32 (pat ForceXForm:$ptr, gprc:$incr)),
          (!cast<Instruction>(atomic_load) memrr:$ptr, 4, gprc:$incr)>;
}

foreach bitsz = [8, 16, 32] in {
  defvar pseudo = "ATOMIC_CMP_SWAP_I"#bitsz;
  defvar pat = !cast<PatFrag>(!tolower(pseudo));
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers. These declarations feed generated pattern-matching logic.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。 这些声明会进入生成式模式匹配逻辑。

### Lines 2089-2096

```tablegen
  let Defs = [CR0] in
    def pseudo : PPCCustomInserterPseudo<
      (outs gprc:$dst), (ins memrr:$ptr, gprc:$old, gprc:$new),
      "#" # NAME,[]>;
  def : Pat<(i32 (pat ForceXForm:$ptr, gprc:$old, gprc:$new)),
          (!cast<Instruction>(pseudo) memrr:$ptr, gprc:$old, gprc:$new)>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2097-2103

```tablegen
def : Pat<(PPCatomicCmpSwap_8 ForceXForm:$ptr, i32:$old, i32:$new),
        (ATOMIC_CMP_SWAP_I8 ForceXForm:$ptr, i32:$old, i32:$new)>;
def : Pat<(PPCatomicCmpSwap_16 ForceXForm:$ptr, i32:$old, i32:$new),
        (ATOMIC_CMP_SWAP_I16 ForceXForm:$ptr, i32:$old, i32:$new)>;

// Instructions to support atomic operations
let mayLoad = 1, mayStore = 1, hasSideEffects = 1 in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2104-2111

```tablegen
def LBARX : XForm_1_memOp<31,  52, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                    "lbarx $RST, $addr", IIC_LdStLWARX, []>,
                    Requires<[HasPartwordAtomics]>;

def LHARX : XForm_1_memOp<31,  116, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                    "lharx $RST, $addr", IIC_LdStLWARX, []>,
                    Requires<[HasPartwordAtomics]>;
```
- **EN**: Adds declarative TableGen records such as `LBARX`, `LHARX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LBARX`, `LHARX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2112-2120

```tablegen
def LWARX : XForm_1_memOp<31,  20, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                    "lwarx $RST, $addr", IIC_LdStLWARX, []>;

// Instructions to support lock versions of atomics
// (EH=1 - see Power ISA 2.07 Book II 4.4.2)
def LBARXL : XForm_1_memOp<31,  52, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                     "lbarx $RST, $addr, 1", IIC_LdStLWARX, []>, isRecordForm,
                     Requires<[HasPartwordAtomics]>;
```
- **EN**: Adds declarative TableGen records such as `LWARX`, `LBARXL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LWARX`, `LBARXL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2121-2130

```tablegen
def LHARXL : XForm_1_memOp<31,  116, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                     "lharx $RST, $addr, 1", IIC_LdStLWARX, []>, isRecordForm,
                     Requires<[HasPartwordAtomics]>;

def LWARXL : XForm_1_memOp<31,  20, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                     "lwarx $RST, $addr, 1", IIC_LdStLWARX, []>, isRecordForm;
}

// The atomic instructions use the destination register as well as the next one
// or two registers in order (modulo 31).
```
- **EN**: Adds declarative TableGen records such as `LHARXL`, `LWARXL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHARXL`, `LWARXL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2131-2138

```tablegen
let hasExtraSrcRegAllocReq = 1, mayLoad = 1, mayStore = 1 in
def LWAT : X_RD5_RS5_IM5<31, 582, (outs g8prc:$RST), (ins g8prc:$RSTi, ptr_rc_nor0:$RA, u5imm:$RB),
                         "lwat $RST, $RA, $RB", IIC_LdStLoad>,
           Requires<[IsISA3_0]>,
           RegConstraint<"$RSTi = $RST">;

let hasExtraSrcRegAllocReq = 1, mayLoad = 1, mayStore = 1,
       isCodeGenOnly = 1, hasNoSchedulingInfo = 1, RB = 16 in
```
- **EN**: Adds declarative TableGen records such as `LWAT` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LWAT`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 2139-2145

```tablegen
def LWAT_CSNE : X_RD5_RS5_IM5<31, 582, (outs g8rc:$RST), (ins ptr_rc_nor0:$RA),
                         "lwat $RST, $RA, 16", IIC_LdStLoad>,
           Requires<[IsISA3_0]>;

def : Pat<(int_ppc_amo_lwat ptr_rc_nor0:$ptr, gprc:$val, u5imm_timm:$fc),
          (TRUNC4 (LWAT (PAIR8 (i64 (IMPLICIT_DEF)), (AEXT8 $val)), $ptr, $fc))>;
```
- **EN**: Adds declarative TableGen records such as `LWAT_CSNE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LWAT_CSNE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2146-2156

```tablegen
def : Pat<(int_ppc_amo_lwat_cond ptr_rc_nor0:$ptr, u5imm_timm:$fc),
          (TRUNC4 (LWAT (PAIR8 (i64 (IMPLICIT_DEF)), (i64 (IMPLICIT_DEF))),
                        $ptr, $fc))>;

let Defs = [R8, R9, R10], Uses = [R9, R10] in
def LWAT_CSNE_PSEUDO : PPCPostRAExpPseudo<
    (outs gprc:$dst),
    (ins ptr_rc_nor0:$ptr),
    "#LWAT_CSNE_PSEUDO",
    [(set i32:$dst, (int_ppc_amo_lwat_csne ptr_rc_nor0:$ptr, R9, R10))]>;
```
- **EN**: Adds declarative TableGen records such as `LWAT_CSNE_PSEUDO` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LWAT_CSNE_PSEUDO`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2157-2165

```tablegen
let Defs = [CR0], mayStore = 1, mayLoad = 1, hasSideEffects = 1 in {
def STBCX : XForm_1_memOp<31, 694, (outs), (ins gprc:$RST, (memrr $RA, $RB):$addr),
                    "stbcx. $RST, $addr", IIC_LdStSTWCX, []>,
                    isRecordForm, Requires<[HasPartwordAtomics]>;

def STHCX : XForm_1_memOp<31, 726, (outs), (ins gprc:$RST, (memrr $RA, $RB):$addr),
                    "sthcx. $RST, $addr", IIC_LdStSTWCX, []>,
                    isRecordForm, Requires<[HasPartwordAtomics]>;
```
- **EN**: Adds declarative TableGen records such as `STBCX`, `STHCX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STBCX`, `STHCX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2166-2175

```tablegen
def STWCX : XForm_1_memOp<31, 150, (outs), (ins gprc:$RST, (memrr $RA, $RB):$addr),
                    "stwcx. $RST, $addr", IIC_LdStSTWCX, []>, isRecordForm;
}

let mayStore = 1, mayLoad = 1, hasSideEffects = 0 in
def STWAT : XForm_base_r3xo_memOp<31, 710, (outs), (ins gprc:$RST, ptr_rc_nor0:$RA, u5imm:$RB),
                                  "stwat $RST, $RA, $RB", IIC_LdStStore,
                                  [(int_ppc_amo_stwat ptr_rc_nor0:$RA, i32:$RST, u5imm_timm:$RB)]>,
            Requires<[IsISA3_0]>;
```
- **EN**: Adds declarative TableGen records such as `STWCX`, `STWAT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STWCX`, `STWAT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2176-2182

```tablegen
let isTrap = 1, hasCtrlDep = 1 in
def TRAP  : XForm_24<31, 4, (outs), (ins), "trap", IIC_LdStLoad, [(trap)]>;

def TWI : DForm_base<3, (outs), (ins u5imm:$RST, gprc:$RA, s16imm:$D, variable_ops),
                     "twi $RST, $RA, $D", IIC_IntTrapW, []>;
def TW : XForm_1<31, 4, (outs), (ins u5imm:$RST, gprc:$RA, gprc:$RB, variable_ops),
                 "tw $RST, $RA, $RB", IIC_IntTrapW, []>;
```
- **EN**: Adds declarative TableGen records such as `TRAP`, `TWI`, `TW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TRAP`, `TWI`, `TW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2183-2191

```tablegen
def TDI : DForm_base<2, (outs), (ins u5imm:$RST, g8rc:$RA, s16imm:$D, variable_ops),
                     "tdi $RST, $RA, $D", IIC_IntTrapD, []>;
def TD : XForm_1<31, 68, (outs), (ins u5imm:$RST, g8rc:$RA, g8rc:$RB, variable_ops),
                 "td $RST, $RA, $RB", IIC_IntTrapD, []>;

def POPCNTB : XForm_11<31, 122, (outs gprc:$RA), (ins gprc:$RST),
                       "popcntb $RA, $RST", IIC_IntGeneral,
                       [(set i32:$RA, (int_ppc_popcntb i32:$RST))]>;
```
- **EN**: Adds declarative TableGen records such as `TDI`, `TD`, `POPCNTB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TDI`, `TD`, `POPCNTB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2192-2201

```tablegen
def CDTBCD : XForm_11<31, 282, (outs gprc:$RA), (ins gprc:$RST),
                      "cdtbcd $RA, $RST", IIC_IntGeneral,
                       [(set i32:$RA, (int_ppc_cdtbcd i32:$RST))]>;
def CBCDTD : XForm_11<31, 314, (outs gprc:$RA), (ins gprc:$RST),
                      "cbcdtd $RA, $RST", IIC_IntGeneral,
                       [(set i32:$RA, (int_ppc_cbcdtd i32:$RST))]>;
def ADDG6S : XOForm_1<31, 74, 0, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                      "addg6s $RT, $RA, $RB", IIC_IntGeneral,
                       [(set i32:$RT, (int_ppc_addg6s i32:$RA, i32:$RB))]>;
```
- **EN**: Adds declarative TableGen records such as `CDTBCD`, `CBCDTD`, `ADDG6S` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CDTBCD`, `CBCDTD`, `ADDG6S`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2202-2211

```tablegen
//===----------------------------------------------------------------------===//
// PPC32 Load Instructions.
//

// Unindexed (r+i) Loads.
let PPC970_Unit = 2 in {
def LBZ : DForm_1<34, (outs gprc:$RST), (ins (memri $D, $RA):$addr),
                  "lbz $RST, $addr", IIC_LdStLoad,
                  [(set i32:$RST, (zextloadi8 DForm:$addr))]>, ZExt32To64,
                  SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `LBZ` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LBZ`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2212-2219

```tablegen
def LHA : DForm_1<42, (outs gprc:$RST), (ins (memri $D, $RA):$addr),
                  "lha $RST, $addr", IIC_LdStLHA,
                  [(set i32:$RST, (sextloadi16 DForm:$addr))]>,
                  PPC970_DGroup_Cracked, SExt32To64;
def LHZ : DForm_1<40, (outs gprc:$RST), (ins (memri $D, $RA):$addr),
                  "lhz $RST, $addr", IIC_LdStLoad,
                  [(set i32:$RST, (zextloadi16 DForm:$addr))]>, ZExt32To64,
                  SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `LHA`, `LHZ` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHA`, `LHZ`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2220-2227

```tablegen
def LWZ : DForm_1<32, (outs gprc:$RST), (ins (memri $D, $RA):$addr),
                  "lwz $RST, $addr", IIC_LdStLoad,
                  [(set i32:$RST, (load DForm:$addr))]>, ZExt32To64;

let Predicates = [HasFPU] in {
def LFS : DForm_1<48, (outs f4rc:$RST), (ins (memri $D, $RA):$addr),
                  "lfs $RST, $addr", IIC_LdStLFD,
                  [(set f32:$RST, (load DForm:$addr))]>;
```
- **EN**: Adds declarative TableGen records such as `LWZ`, `LFS` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LWZ`, `LFS`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2228-2234

```tablegen
def LFD : DForm_1<50, (outs f8rc:$RST), (ins (memri $D, $RA):$addr),
                  "lfd $RST, $addr", IIC_LdStLFD,
                  [(set f64:$RST, (load DForm:$addr))]>;
}


// Unindexed (r+i) Loads with Update (preinc).
```
- **EN**: Adds declarative TableGen records such as `LFD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LFD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2235-2243

```tablegen
let mayLoad = 1, mayStore = 0, hasSideEffects = 0 in {
def LBZU : DForm_1<35, (outs gprc:$RST, ptr_rc_nor0:$ea_result), (ins (memri $D, $RA):$addr),
                   "lbzu $RST, $addr", IIC_LdStLoadUpd,
                   []>, RegConstraint<"$RA = $ea_result">;

def LHAU : DForm_1<43, (outs gprc:$RST, ptr_rc_nor0:$ea_result), (ins (memri $D, $RA):$addr),
                   "lhau $RST, $addr", IIC_LdStLHAU,
                   []>, RegConstraint<"$addr.reg = $ea_result">;
```
- **EN**: Adds declarative TableGen records such as `LBZU`, `LHAU` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LBZU`, `LHAU`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2244-2251

```tablegen
def LHZU : DForm_1<41, (outs gprc:$RST, ptr_rc_nor0:$ea_result), (ins (memri $D, $RA):$addr),
                   "lhzu $RST, $addr", IIC_LdStLoadUpd,
                   []>, RegConstraint<"$addr.reg = $ea_result">;

def LWZU : DForm_1<33, (outs gprc:$RST, ptr_rc_nor0:$ea_result), (ins (memri $D, $RA):$addr),
                   "lwzu $RST, $addr", IIC_LdStLoadUpd,
                   []>, RegConstraint<"$addr.reg = $ea_result">;
```
- **EN**: Adds declarative TableGen records such as `LHZU`, `LWZU` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHZU`, `LWZU`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2252-2263

```tablegen
let Predicates = [HasFPU] in {
def LFSU : DForm_1<49, (outs f4rc:$RST, ptr_rc_nor0:$ea_result), (ins (memri $D, $RA):$addr),
                  "lfsu $RST, $addr", IIC_LdStLFDU,
                  []>, RegConstraint<"$addr.reg = $ea_result">;

def LFDU : DForm_1<51, (outs f8rc:$RST, ptr_rc_nor0:$ea_result), (ins (memri $D, $RA):$addr),
                  "lfdu $RST, $addr", IIC_LdStLFDU,
                  []>, RegConstraint<"$addr.reg = $ea_result">;
}


// Indexed (r+r) Loads with Update (preinc).
```
- **EN**: Adds declarative TableGen records such as `LFSU`, `LFDU` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LFSU`, `LFDU`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2264-2273

```tablegen
def LBZUX : XForm_1_memOp<31, 119, (outs gprc:$RST, ptr_rc_nor0:$ea_result),
                   (ins (memrr $RA, $RB):$addr),
                   "lbzux $RST, $addr", IIC_LdStLoadUpdX,
                   []>, RegConstraint<"$addr.ptrreg = $ea_result">;

def LHAUX : XForm_1_memOp<31, 375, (outs gprc:$RST, ptr_rc_nor0:$ea_result),
                   (ins (memrr $RA, $RB):$addr),
                   "lhaux $RST, $addr", IIC_LdStLHAUX,
                   []>, RegConstraint<"$addr.ptrreg = $ea_result">;
```
- **EN**: Adds declarative TableGen records such as `LBZUX`, `LHAUX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LBZUX`, `LHAUX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2274-2283

```tablegen
def LHZUX : XForm_1_memOp<31, 311, (outs gprc:$RST, ptr_rc_nor0:$ea_result),
                   (ins (memrr $RA, $RB):$addr),
                   "lhzux $RST, $addr", IIC_LdStLoadUpdX,
                   []>, RegConstraint<"$addr.ptrreg = $ea_result">;

def LWZUX : XForm_1_memOp<31, 55, (outs gprc:$RST, ptr_rc_nor0:$ea_result),
                   (ins (memrr $RA, $RB):$addr),
                   "lwzux $RST, $addr", IIC_LdStLoadUpdX,
                   []>, RegConstraint<"$addr.ptrreg = $ea_result">;
```
- **EN**: Adds declarative TableGen records such as `LHZUX`, `LWZUX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHZUX`, `LWZUX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2284-2299

```tablegen
let Predicates = [HasFPU] in {
def LFSUX : XForm_1_memOp<31, 567, (outs f4rc:$RST, ptr_rc_nor0:$ea_result),
                   (ins (memrr $RA, $RB):$addr),
                   "lfsux $RST, $addr", IIC_LdStLFDUX,
                   []>, RegConstraint<"$addr.ptrreg = $ea_result">;

def LFDUX : XForm_1_memOp<31, 631, (outs f8rc:$RST, ptr_rc_nor0:$ea_result),
                   (ins (memrr $RA, $RB):$addr),
                   "lfdux $RST, $addr", IIC_LdStLFDUX,
                   []>, RegConstraint<"$addr.ptrreg = $ea_result">;
}
}
}

// Indexed (r+r) Loads.
//
```
- **EN**: Adds declarative TableGen records such as `LFSUX`, `LFDUX` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LFSUX`, `LFDUX`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2300-2308

```tablegen
let PPC970_Unit = 2, mayLoad = 1, mayStore = 0 in {
def LBZX : XForm_1_memOp<31,  87, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lbzx $RST, $addr", IIC_LdStLoad,
                   [(set i32:$RST, (zextloadi8 XForm:$addr))]>, ZExt32To64,
                   SExt32To64;
def LHAX : XForm_1_memOp<31, 343, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lhax $RST, $addr", IIC_LdStLHA,
                   [(set i32:$RST, (sextloadi16 XForm:$addr))]>,
                   PPC970_DGroup_Cracked, SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `LBZX`, `LHAX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LBZX`, `LHAX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2309-2315

```tablegen
def LHZX : XForm_1_memOp<31, 279, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lhzx $RST, $addr", IIC_LdStLoad,
                   [(set i32:$RST, (zextloadi16 XForm:$addr))]>, ZExt32To64,
                   SExt32To64;
def LWZX : XForm_1_memOp<31,  23, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lwzx $RST, $addr", IIC_LdStLoad,
                   [(set i32:$RST, (load XForm:$addr))]>, ZExt32To64;
```
- **EN**: Adds declarative TableGen records such as `LHZX`, `LWZX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHZX`, `LWZX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2316-2322

```tablegen
def LHBRX : XForm_1_memOp<31, 790, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lhbrx $RST, $addr", IIC_LdStLoad,
                   [(set i32:$RST, (PPClbrx ForceXForm:$addr, i16))]>, ZExt32To64;
def LWBRX : XForm_1_memOp<31,  534, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                   "lwbrx $RST, $addr", IIC_LdStLoad,
                   [(set i32:$RST, (PPClbrx ForceXForm:$addr, i32))]>, ZExt32To64;
```
- **EN**: Adds declarative TableGen records such as `LHBRX`, `LWBRX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHBRX`, `LWBRX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2323-2330

```tablegen
let Predicates = [HasFPU] in {
def LFSX   : XForm_25_memOp<31, 535, (outs f4rc:$RST), (ins (memrr $RA, $RB):$addr),
                      "lfsx $RST, $addr", IIC_LdStLFD,
                      [(set f32:$RST, (load XForm:$addr))]>;
def LFDX   : XForm_25_memOp<31, 599, (outs f8rc:$RST), (ins (memrr $RA, $RB):$addr),
                      "lfdx $RST, $addr", IIC_LdStLFD,
                      [(set f64:$RST, (load XForm:$addr))]>;
```
- **EN**: Adds declarative TableGen records such as `LFSX`, `LFDX` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LFSX`, `LFDX`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2331-2340

```tablegen
def LFIWAX : XForm_25_memOp<31, 855, (outs f8rc:$RST), (ins (memrr $RA, $RB):$addr),
                      "lfiwax $RST, $addr", IIC_LdStLFD,
                      [(set f64:$RST, (PPClfiwax ForceXForm:$addr))]>;
def LFIWZX : XForm_25_memOp<31, 887, (outs f8rc:$RST), (ins (memrr $RA, $RB):$addr),
                      "lfiwzx $RST, $addr", IIC_LdStLFD,
                      [(set f64:$RST, (PPClfiwzx ForceXForm:$addr))]>;
}
}

// Load Multiple
```
- **EN**: Adds declarative TableGen records such as `LFIWAX`, `LFIWZX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LFIWAX`, `LFIWZX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2341-2344

```tablegen
let mayLoad = 1, mayStore = 0, hasSideEffects = 0 in
def LMW : DForm_1<46, (outs gprc:$RST), (ins (memri $D, $RA):$src),
                  "lmw $RST, $src", IIC_LdStLMW, []>;
```
- **EN**: Adds declarative TableGen records such as `LMW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LMW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2345-2353

```tablegen
//===----------------------------------------------------------------------===//
// PPC32 Store Instructions.
//

// Unindexed (r+i) Stores.
let PPC970_Unit = 2, mayStore = 1, mayLoad = 0 in {
def STB  : DForm_1<38, (outs), (ins gprc:$RST, (memri $D, $RA):$dst),
                   "stb $RST, $dst", IIC_LdStStore,
                   [(truncstorei8 i32:$RST, DForm:$dst)]>;
```
- **EN**: Adds declarative TableGen records such as `STB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2354-2360

```tablegen
def STH  : DForm_1<44, (outs), (ins gprc:$RST, (memri $D, $RA):$dst),
                   "sth $RST, $dst", IIC_LdStStore,
                   [(truncstorei16 i32:$RST, DForm:$dst)]>;
def STW  : DForm_1<36, (outs), (ins gprc:$RST, (memri $D, $RA):$dst),
                   "stw $RST, $dst", IIC_LdStStore,
                   [(store i32:$RST, DForm:$dst)]>;
let Predicates = [HasFPU] in {
```
- **EN**: Adds declarative TableGen records such as `STH`, `STW` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STH`, `STW`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2361-2370

```tablegen
def STFS : DForm_1<52, (outs), (ins f4rc:$RST, (memri $D, $RA):$dst),
                   "stfs $RST, $dst", IIC_LdStSTFD,
                   [(store f32:$RST, DForm:$dst)]>;
def STFD : DForm_1<54, (outs), (ins f8rc:$RST, (memri $D, $RA):$dst),
                   "stfd $RST, $dst", IIC_LdStSTFD,
                   [(store f64:$RST, DForm:$dst)]>;
}
}

// Unindexed (r+i) Stores with Update (preinc).
```
- **EN**: Adds declarative TableGen records such as `STFS`, `STFD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STFS`, `STFD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2371-2377

```tablegen
let PPC970_Unit = 2, mayStore = 1, mayLoad = 0 in {
def STBU  : DForm_1<39, (outs ptr_rc_nor0:$ea_res), (ins gprc:$RST, (memri $D, $RA):$dst),
                    "stbu $RST, $dst", IIC_LdStSTU, []>,
                    RegConstraint<"$dst.reg = $ea_res">;
def STHU  : DForm_1<45, (outs ptr_rc_nor0:$ea_res), (ins gprc:$RST, (memri $D, $RA):$dst),
                    "sthu $RST, $dst", IIC_LdStSTU, []>,
                    RegConstraint<"$dst.reg = $ea_res">;
```
- **EN**: Adds declarative TableGen records such as `STBU`, `STHU` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STBU`, `STHU`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2378-2384

```tablegen
def STWU  : DForm_1<37, (outs ptr_rc_nor0:$ea_res), (ins gprc:$RST, (memri $D, $RA):$dst),
                    "stwu $RST, $dst", IIC_LdStSTU, []>,
                    RegConstraint<"$dst.reg = $ea_res">;
let Predicates = [HasFPU] in {
def STFSU : DForm_1<53, (outs ptr_rc_nor0:$ea_res), (ins f4rc:$RST, (memri $D, $RA):$dst),
                    "stfsu $RST, $dst", IIC_LdStSTFDU, []>,
                    RegConstraint<"$dst.reg = $ea_res">;
```
- **EN**: Adds declarative TableGen records such as `STWU`, `STFSU` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STWU`, `STFSU`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2385-2393

```tablegen
def STFDU : DForm_1<55, (outs ptr_rc_nor0:$ea_res), (ins f8rc:$RST, (memri $D, $RA):$dst),
                    "stfdu $RST, $dst", IIC_LdStSTFDU, []>,
                    RegConstraint<"$dst.reg = $ea_res">;
}
}

// Patterns to match the pre-inc stores.  We can't put the patterns on
// the instruction definitions directly as ISel wants the address base
// and offset to be separate operands, not a single complex operand.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Patterns to match the pre-inc stores.  We can't put the patterns on". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Patterns to match the pre-inc stores.  We can't put the patterns on”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2394-2401

```tablegen
def : Pat<(pre_truncsti8 i32:$rS, iPTR:$ptrreg, iaddroff:$ptroff),
          (STBU $rS, iaddroff:$ptroff, $ptrreg)>;
def : Pat<(pre_truncsti16 i32:$rS, iPTR:$ptrreg, iaddroff:$ptroff),
          (STHU $rS, iaddroff:$ptroff, $ptrreg)>;
def : Pat<(pre_store i32:$rS, iPTR:$ptrreg, iaddroff:$ptroff),
          (STWU $rS, iaddroff:$ptroff, $ptrreg)>;
def : Pat<(pre_store f32:$rS, iPTR:$ptrreg, iaddroff:$ptroff),
          (STFSU $rS, iaddroff:$ptroff, $ptrreg)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2402-2410

```tablegen
def : Pat<(pre_store f64:$rS, iPTR:$ptrreg, iaddroff:$ptroff),
          (STFDU $rS, iaddroff:$ptroff, $ptrreg)>;

// Indexed (r+r) Stores.
let PPC970_Unit = 2 in {
def STBX  : XForm_8_memOp<31, 215, (outs), (ins gprc:$RST, (memrr $RA, $RB):$addr),
                   "stbx $RST, $addr", IIC_LdStStore,
                   [(truncstorei8 i32:$RST, XForm:$addr)]>,
                   PPC970_DGroup_Cracked;
```
- **EN**: Adds declarative TableGen records such as `STBX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STBX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2411-2419

```tablegen
def STHX  : XForm_8_memOp<31, 407, (outs), (ins gprc:$RST, (memrr $RA, $RB):$addr),
                   "sthx $RST, $addr", IIC_LdStStore,
                   [(truncstorei16 i32:$RST, XForm:$addr)]>,
                   PPC970_DGroup_Cracked;
def STWX  : XForm_8_memOp<31, 151, (outs), (ins gprc:$RST, (memrr $RA, $RB):$addr),
                   "stwx $RST, $addr", IIC_LdStStore,
                   [(store i32:$RST, XForm:$addr)]>,
                   PPC970_DGroup_Cracked;
```
- **EN**: Adds declarative TableGen records such as `STHX`, `STWX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STHX`, `STWX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2420-2428

```tablegen
def STHBRX: XForm_8_memOp<31, 918, (outs), (ins gprc:$RST, (memrr $RA, $RB):$addr),
                   "sthbrx $RST, $addr", IIC_LdStStore,
                   [(PPCstbrx i32:$RST, ForceXForm:$addr, i16)]>,
                   PPC970_DGroup_Cracked;
def STWBRX: XForm_8_memOp<31, 662, (outs), (ins gprc:$RST, (memrr $RA, $RB):$addr),
                   "stwbrx $RST, $addr", IIC_LdStStore,
                   [(PPCstbrx i32:$RST, ForceXForm:$addr, i32)]>,
                   PPC970_DGroup_Cracked;
```
- **EN**: Adds declarative TableGen records such as `STHBRX`, `STWBRX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STHBRX`, `STWBRX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2429-2436

```tablegen
let Predicates = [HasFPU] in {
def STFIWX: XForm_28_memOp<31, 983, (outs), (ins f8rc:$RST, (memrr $RA, $RB):$addr),
                     "stfiwx $RST, $addr", IIC_LdStSTFD,
                     [(PPCstfiwx f64:$RST, ForceXForm:$addr)]>;

def STFSX : XForm_28_memOp<31, 663, (outs), (ins f4rc:$RST, (memrr $RA, $RB):$addr),
                     "stfsx $RST, $addr", IIC_LdStSTFD,
                     [(store f32:$RST, XForm:$addr)]>;
```
- **EN**: Adds declarative TableGen records such as `STFIWX`, `STFSX` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STFIWX`, `STFSX`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2437-2443

```tablegen
def STFDX : XForm_28_memOp<31, 727, (outs), (ins f8rc:$RST, (memrr $RA, $RB):$addr),
                     "stfdx $RST, $addr", IIC_LdStSTFD,
                     [(store f64:$RST, XForm:$addr)]>;
}
}

// Indexed (r+r) Stores with Update (preinc).
```
- **EN**: Adds declarative TableGen records such as `STFDX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STFDX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2444-2454

```tablegen
let PPC970_Unit = 2, mayStore = 1, mayLoad = 0 in {
def STBUX : XForm_8_memOp<31, 247, (outs ptr_rc_nor0:$ea_res),
                          (ins gprc:$RST, (memrr $RA, $RB):$addr),
                          "stbux $RST, $addr", IIC_LdStSTUX, []>,
                          RegConstraint<"$addr.ptrreg = $ea_res">,
                          PPC970_DGroup_Cracked;
def STHUX : XForm_8_memOp<31, 439, (outs ptr_rc_nor0:$ea_res),
                          (ins gprc:$RST, (memrr $RA, $RB):$addr),
                          "sthux $RST, $addr", IIC_LdStSTUX, []>,
                          RegConstraint<"$addr.ptrreg = $ea_res">,
                          PPC970_DGroup_Cracked;
```
- **EN**: Adds declarative TableGen records such as `STBUX`, `STHUX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STBUX`, `STHUX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2455-2465

```tablegen
def STWUX : XForm_8_memOp<31, 183, (outs ptr_rc_nor0:$ea_res),
                          (ins gprc:$RST, (memrr $RA, $RB):$addr),
                          "stwux $RST, $addr", IIC_LdStSTUX, []>,
                          RegConstraint<"$addr.ptrreg = $ea_res">,
                          PPC970_DGroup_Cracked;
let Predicates = [HasFPU] in {
def STFSUX: XForm_8_memOp<31, 695, (outs ptr_rc_nor0:$ea_res),
                          (ins f4rc:$RST, (memrr $RA, $RB):$addr),
                          "stfsux $RST, $addr", IIC_LdStSTFDU, []>,
                          RegConstraint<"$addr.ptrreg = $ea_res">,
                          PPC970_DGroup_Cracked;
```
- **EN**: Adds declarative TableGen records such as `STWUX`, `STFSUX` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STWUX`, `STFSUX`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2466-2476

```tablegen
def STFDUX: XForm_8_memOp<31, 759, (outs ptr_rc_nor0:$ea_res),
                          (ins f8rc:$RST, (memrr $RA, $RB):$addr),
                          "stfdux $RST, $addr", IIC_LdStSTFDU, []>,
                          RegConstraint<"$addr.ptrreg = $ea_res">,
                          PPC970_DGroup_Cracked;
}
}

// Patterns to match the pre-inc stores.  We can't put the patterns on
// the instruction definitions directly as ISel wants the address base
// and offset to be separate operands, not a single complex operand.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Patterns to match the pre-inc stores.  We can't put the patterns on". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Patterns to match the pre-inc stores.  We can't put the patterns on”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2477-2483

```tablegen
def : Pat<(pre_truncsti8 i32:$rS, iPTR:$ptrreg, iPTR:$ptroff),
          (STBUX $rS, $ptrreg, $ptroff)>;
def : Pat<(pre_truncsti16 i32:$rS, iPTR:$ptrreg, iPTR:$ptroff),
          (STHUX $rS, $ptrreg, $ptroff)>;
def : Pat<(pre_store i32:$rS, iPTR:$ptrreg, iPTR:$ptroff),
          (STWUX $rS, $ptrreg, $ptroff)>;
let Predicates = [HasFPU] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2484-2490

```tablegen
def : Pat<(pre_store f32:$rS, iPTR:$ptrreg, iPTR:$ptroff),
          (STFSUX $rS, $ptrreg, $ptroff)>;
def : Pat<(pre_store f64:$rS, iPTR:$ptrreg, iPTR:$ptroff),
          (STFDUX $rS, $ptrreg, $ptroff)>;
}

// Store Multiple
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2491-2497

```tablegen
let mayStore = 1, mayLoad = 0, hasSideEffects = 0 in
def STMW : DForm_1<47, (outs), (ins gprc:$RST, (memri $D, $RA):$dst),
                   "stmw $RST, $dst", IIC_LdStLMW, []>;

def SYNC : XForm_24_sync<31, 598, (outs), (ins u2imm:$L),
                        "sync $L", IIC_LdStSync, []>;
```
- **EN**: Adds declarative TableGen records such as `STMW`, `SYNC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STMW`, `SYNC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2498-2505

```tablegen
let isCodeGenOnly = 1 in {
  def MSYNC : XForm_24_sync<31, 598, (outs), (ins),
                           "msync", IIC_LdStSync, []> {
    let L = 0;
  }
}

// We used to have EIEIO as value but E[0-9A-Z] is a reserved name
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2506-2513

```tablegen
def EnforceIEIO : XForm_24_eieio<31, 854, (outs), (ins),
                                 "eieio", IIC_LdStLoad, []>;

def PseudoEIEIO : PPCEmitTimePseudo<(outs), (ins), "#PPCEIEIO",
                  [(int_ppc_eieio)]> {
  let Size = 12;
}
```
- **EN**: Adds declarative TableGen records such as `EnforceIEIO`, `PseudoEIEIO` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EnforceIEIO`, `PseudoEIEIO`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2514-2520

```tablegen
def : Pat<(int_ppc_sync),   (SYNC 0)>, Requires<[HasSYNC]>;
def : Pat<(int_ppc_iospace_sync),   (SYNC 0)>, Requires<[HasSYNC]>;
def : Pat<(int_ppc_lwsync), (SYNC 1)>, Requires<[HasSYNC]>;
def : Pat<(int_ppc_iospace_lwsync), (SYNC 1)>, Requires<[HasSYNC]>;
def : Pat<(int_ppc_sync),   (MSYNC)>, Requires<[HasOnlyMSYNC]>;
def : Pat<(int_ppc_iospace_sync),   (MSYNC)>, Requires<[HasOnlyMSYNC]>;
def : Pat<(int_ppc_lwsync), (MSYNC)>, Requires<[HasOnlyMSYNC]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2521-2524

```tablegen
def : Pat<(int_ppc_iospace_lwsync), (MSYNC)>, Requires<[HasOnlyMSYNC]>;
def : Pat<(int_ppc_eieio),  (PseudoEIEIO)>;
def : Pat<(int_ppc_iospace_eieio),  (PseudoEIEIO)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2525-2532

```tablegen
//===----------------------------------------------------------------------===//
// PPC32 Arithmetic Instructions.
//

let PPC970_Unit = 1 in {  // FXU Operations.
def ADDI   : DForm_2<14, (outs gprc:$RST), (ins gprc_nor0:$RA, s16imm:$D),
                     "addi $RST, $RA, $D", IIC_IntSimple,
                     [(set i32:$RST, (add_like i32:$RA, imm32SExt16:$D))]>;
```
- **EN**: Adds declarative TableGen records such as `ADDI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2533-2539

```tablegen
let BaseName = "addic" in {
let Defs = [CARRY] in
def ADDIC  : DForm_2<12, (outs gprc:$RST), (ins gprc:$RA, s16imm:$D),
                     "addic $RST, $RA, $D", IIC_IntGeneral,
                     [(set i32:$RST, (PPCaddc i32:$RA, imm32SExt16:$D))]>,
                     RecFormRel, PPC970_DGroup_Cracked, Requires<[IsPPC32]>;
let Defs = [CARRY, CR0] in
```
- **EN**: Adds declarative TableGen records such as `ADDIC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDIC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2540-2546

```tablegen
def ADDIC_rec : DForm_2<13, (outs gprc:$RST), (ins gprc:$RA, s16imm:$D),
                     "addic. $RST, $RA, $D", IIC_IntGeneral,
                     []>, isRecordForm, RecFormRel;
}
def ADDIS  : DForm_2<15, (outs gprc:$RST), (ins gprc_nor0:$RA, s17imm:$D),
                     "addis $RST, $RA, $D", IIC_IntSimple,
                     [(set i32:$RST, (add_like i32:$RA, imm16ShiftedSExt:$D))]>;
```
- **EN**: Adds declarative TableGen records such as `ADDIC_rec`, `ADDIS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDIC_rec`, `ADDIS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2547-2554

```tablegen
let isCodeGenOnly = 1 in
def LA     : DForm_2<14, (outs gprc:$RST), (ins gprc_nor0:$RA, s16imm:$D),
                     "la $RST, $D($RA)", IIC_IntGeneral,
                     [(set i32:$RST, (add i32:$RA,
                                          (PPClo tglobaladdr:$D, 0)))]>, MemriOp;
def MULLI  : DForm_2< 7, (outs gprc:$RST), (ins gprc:$RA, s16imm:$D),
                     "mulli $RST, $RA, $D", IIC_IntMulLI,
                     [(set i32:$RST, (mul i32:$RA, imm32SExt16:$D))]>;
```
- **EN**: Adds declarative TableGen records such as `LA`, `MULLI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LA`, `MULLI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2555-2561

```tablegen
let Defs = [CARRY] in
def SUBFIC : DForm_2< 8, (outs gprc:$RST), (ins gprc:$RA, s16imm:$D),
                     "subfic $RST, $RA, $D", IIC_IntGeneral,
                     [(set i32:$RST, (PPCsubc imm32SExt16:$D, i32:$RA))]>,
                     Requires<[IsPPC32]>;

let isReMaterializable = 1, isAsCheapAsAMove = 1, isMoveImm = 1 in {
```
- **EN**: Adds declarative TableGen records such as `SUBFIC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SUBFIC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2562-2570

```tablegen
  def LI  : DForm_2_r0<14, (outs gprc:$RST), (ins s16imm:$D),
                       "li $RST, $D", IIC_IntSimple,
                       [(set i32:$RST, imm32SExt16:$D)]>, SExt32To64;
  def LIS : DForm_2_r0<15, (outs gprc:$RST), (ins s17imm:$D),
                       "lis $RST, $D", IIC_IntSimple,
                       [(set i32:$RST, imm16ShiftedSExt:$D)]>, SExt32To64;
}
}
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2571-2579

```tablegen
def : InstAlias<"li $rD, $imm", (ADDI gprc:$rD, ZERO, s16imm:$imm)>;
def : InstAlias<"lis $rD, $imm", (ADDIS gprc:$rD, ZERO, s17imm:$imm)>;

let PPC970_Unit = 1 in {  // FXU Operations.
let Defs = [CR0] in {
def ANDI_rec : DForm_4<28, (outs gprc:$RA), (ins gprc:$RST, u16imm:$D),
                    "andi. $RA, $RST, $D", IIC_IntGeneral,
                    [(set i32:$RA, (and i32:$RST, immZExt16:$D))]>,
                    isRecordForm, ZExt32To64, SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `ANDI_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ANDI_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2580-2587

```tablegen
def ANDIS_rec : DForm_4<29, (outs gprc:$RA), (ins gprc:$RST, u16imm:$D),
                    "andis. $RA, $RST, $D", IIC_IntGeneral,
                    [(set i32:$RA, (and i32:$RST, imm16ShiftedZExt:$D))]>,
                    isRecordForm, ZExt32To64;
}
def ORI   : DForm_4<24, (outs gprc:$RA), (ins gprc:$RST, u16imm:$D),
                    "ori $RA, $RST, $D", IIC_IntSimple,
                    [(set i32:$RA, (or i32:$RST, immZExt16:$D))]>;
```
- **EN**: Adds declarative TableGen records such as `ANDIS_rec`, `ORI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ANDIS_rec`, `ORI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2588-2597

```tablegen
def ORIS  : DForm_4<25, (outs gprc:$RA), (ins gprc:$RST, u16imm:$D),
                    "oris $RA, $RST, $D", IIC_IntSimple,
                    [(set i32:$RA, (or i32:$RST, imm16ShiftedZExt:$D))]>;
def XORI  : DForm_4<26, (outs gprc:$RA), (ins gprc:$RST, u16imm:$D),
                    "xori $RA, $RST, $D", IIC_IntSimple,
                    [(set i32:$RA, (xor i32:$RST, immZExt16:$D))]>;
def XORIS : DForm_4<27, (outs gprc:$RA), (ins gprc:$RST, u16imm:$D),
                    "xoris $RA, $RST, $D", IIC_IntSimple,
                    [(set i32:$RA, (xor i32:$RST, imm16ShiftedZExt:$D))]>;
```
- **EN**: Adds declarative TableGen records such as `ORIS`, `XORI`, `XORIS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ORIS`, `XORI`, `XORIS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2598-2607

```tablegen
def NOP   : DForm_4_zero<24, (outs), (ins), "nop", IIC_IntSimple,
                         []>;
let isCodeGenOnly = 1 in {
// The POWER6 and POWER7 have special group-terminating nops.
def NOP_GT_PWR6 : DForm_4_fixedreg_zero<24, 1, (outs), (ins),
                                        "ori 1, 1, 0", IIC_IntSimple, []>;
def NOP_GT_PWR7 : DForm_4_fixedreg_zero<24, 2, (outs), (ins),
                                        "ori 2, 2, 0", IIC_IntSimple, []>;
}
```
- **EN**: Adds declarative TableGen records such as `NOP`, `NOP_GT_PWR6`, `NOP_GT_PWR7` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NOP`, `NOP_GT_PWR6`, `NOP_GT_PWR7`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2608-2619

```tablegen
let isCompare = 1, hasSideEffects = 0 in {
  def CMPWI : DForm_5_ext<11, (outs crrc:$BF), (ins gprc:$RA, s16imm:$D),
                          "cmpwi $BF, $RA, $D", IIC_IntCompare>;
  def CMPLWI : DForm_6_ext<10, (outs crrc:$BF), (ins gprc:$RA, u16imm:$D),
                           "cmplwi $BF, $RA, $D", IIC_IntCompare>;
  def CMPRB  : X_BF3_L1_RS5_RS5<31, 192, (outs crrc:$BF),
                                (ins u1imm:$L, gprc:$RA, gprc:$RB),
                                "cmprb $BF, $L, $RA, $RB", IIC_IntCompare, []>,
               Requires<[IsISA3_0]>;
}
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2620-2628

```tablegen
let PPC970_Unit = 1, hasSideEffects = 0 in {  // FXU Operations.
let isCommutable = 1 in {
defm NAND : XForm_6r<31, 476, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                     "nand", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i32:$RA, (not (and i32:$RST, i32:$RB)))]>;
defm AND  : XForm_6r<31,  28, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                     "and", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i32:$RA, (and i32:$RST, i32:$RB))]>;
} // isCommutable
```
- **EN**: Adds declarative TableGen records such as `NAND`, `AND` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NAND`, `AND`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2629-2635

```tablegen
defm ANDC : XForm_6r<31,  60, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                     "andc", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i32:$RA, (and i32:$RST, (not i32:$RB)))]>;
let isCommutable = 1 in {
defm OR   : XForm_6r<31, 444, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                     "or", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i32:$RA, (or i32:$RST, i32:$RB))]>;
```
- **EN**: Adds declarative TableGen records such as `ANDC`, `OR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ANDC`, `OR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2636-2642

```tablegen
defm NOR  : XForm_6r<31, 124, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                     "nor", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i32:$RA, (not (or i32:$RST, i32:$RB)))]>;
} // isCommutable
defm ORC  : XForm_6r<31, 412, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                     "orc", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i32:$RA, (or i32:$RST, (not i32:$RB)))]>;
```
- **EN**: Adds declarative TableGen records such as `NOR`, `ORC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NOR`, `ORC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2643-2650

```tablegen
let isCommutable = 1 in {
defm EQV  : XForm_6r<31, 284, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                     "eqv", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i32:$RA, (not (xor i32:$RST, i32:$RB)))]>;
defm XOR  : XForm_6r<31, 316, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                     "xor", "$RA, $RST, $RB", IIC_IntSimple,
                     [(set i32:$RA, (xor i32:$RST, i32:$RB))]>;
} // isCommutable
```
- **EN**: Adds declarative TableGen records such as `EQV`, `XOR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EQV`, `XOR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2651-2661

```tablegen
defm SLW  : XForm_6r<31,  24, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                     "slw", "$RA, $RST, $RB", IIC_IntGeneral,
                     [(set i32:$RA, (PPCshl i32:$RST, i32:$RB))]>, ZExt32To64;
defm SRW  : XForm_6r<31, 536, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                     "srw", "$RA, $RST, $RB", IIC_IntGeneral,
                     [(set i32:$RA, (PPCsrl i32:$RST, i32:$RB))]>, ZExt32To64;
defm SRAW : XForm_6rc<31, 792, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                      "sraw", "$RA, $RST, $RB", IIC_IntShift,
                      [(set i32:$RA, (PPCsra i32:$RST, i32:$RB))]>, SExt32To64;
}
```
- **EN**: Adds declarative TableGen records such as `SLW`, `SRW`, `SRAW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SLW`, `SRW`, `SRAW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2662-2669

```tablegen
def : InstAlias<"mr $rA, $rB", (OR gprc:$rA, gprc:$rB, gprc:$rB)>;
def : InstAlias<"mr. $rA, $rB", (OR_rec gprc:$rA, gprc:$rB, gprc:$rB)>;

def : InstAlias<"not $rA, $rS", (NOR gprc:$rA, gprc:$rS, gprc:$rS)>;
def : InstAlias<"not. $rA, $rS", (NOR_rec gprc:$rA, gprc:$rS, gprc:$rS)>;

def : InstAlias<"nop", (ORI R0, R0, 0)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2670-2678

```tablegen
let PPC970_Unit = 1 in {  // FXU Operations.
let hasSideEffects = 0 in {
defm SRAWI : XForm_10rc<31, 824, (outs gprc:$RA), (ins gprc:$RST, u5imm:$RB),
                        "srawi", "$RA, $RST, $RB", IIC_IntShift,
                        [(set i32:$RA, (sra i32:$RST, (i32 imm:$RB)))]>,
                        SExt32To64;
defm CNTLZW : XForm_11r<31,  26, (outs gprc:$RA), (ins gprc:$RST),
                        "cntlzw", "$RA, $RST", IIC_IntGeneral,
                        [(set i32:$RA, (ctlz i32:$RST))]>, ZExt32To64;
```
- **EN**: Adds declarative TableGen records such as `SRAWI`, `CNTLZW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SRAWI`, `CNTLZW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2679-2685

```tablegen
defm CNTTZW : XForm_11r<31, 538, (outs gprc:$RA), (ins gprc:$RST),
                        "cnttzw", "$RA, $RST", IIC_IntGeneral,
                        [(set i32:$RA, (cttz i32:$RST))]>, Requires<[IsISA3_0]>,
                        ZExt32To64;
defm EXTSB  : XForm_11r<31, 954, (outs gprc:$RA), (ins gprc:$RST),
                        "extsb", "$RA, $RST", IIC_IntSimple,
                        [(set i32:$RA, (sext_inreg i32:$RST, i8))]>, SExt32To64;
```
- **EN**: Adds declarative TableGen records such as `CNTTZW`, `EXTSB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CNTTZW`, `EXTSB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2686-2694

```tablegen
defm EXTSH  : XForm_11r<31, 922, (outs gprc:$RA), (ins gprc:$RST),
                        "extsh", "$RA, $RST", IIC_IntSimple,
                        [(set i32:$RA, (sext_inreg i32:$RST, i16))]>, SExt32To64;

let isCommutable = 1 in
def CMPB : XForm_6<31, 508, (outs gprc:$RA), (ins gprc:$RST, gprc:$RB),
                   "cmpb $RA, $RST, $RB", IIC_IntGeneral,
                   [(set i32:$RA, (PPCcmpb i32:$RST, i32:$RB))]>;
}
```
- **EN**: Adds declarative TableGen records such as `EXTSH`, `CMPB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTSH`, `CMPB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2695-2701

```tablegen
let isCompare = 1, hasSideEffects = 0 in {
  def CMPW   : XForm_16_ext<31, 0, (outs crrc:$BF), (ins gprc:$RA, gprc:$RB),
                            "cmpw $BF, $RA, $RB", IIC_IntCompare>;
  def CMPLW  : XForm_16_ext<31, 32, (outs crrc:$BF), (ins gprc:$RA, gprc:$RB),
                            "cmplw $BF, $RA, $RB", IIC_IntCompare>;
}
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2702-2708

```tablegen
let PPC970_Unit = 3, Predicates = [HasFPU] in {  // FPU Operations.
let isCompare = 1, mayRaiseFPException = 1, hasSideEffects = 0 in {
  def FCMPUS : XForm_17<63, 0, (outs crrc:$BF), (ins f4rc:$RA, f4rc:$RB),
                        "fcmpu $BF, $RA, $RB", IIC_FPCompare>;
  def FCMPOS : XForm_17<63, 32, (outs crrc:$BF), (ins f4rc:$RA, f4rc:$RB),
                        "fcmpo $BF, $RA, $RB", IIC_FPCompare>;
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2709-2715

```tablegen
    def FCMPUD : XForm_17<63, 0, (outs crrc:$BF), (ins f8rc:$RA, f8rc:$RB),
                          "fcmpu $BF, $RA, $RB", IIC_FPCompare>;
    def FCMPOD : XForm_17<63, 32, (outs crrc:$BF), (ins f8rc:$RA, f8rc:$RB),
                          "fcmpo $BF, $RA, $RB", IIC_FPCompare>;
  }
}
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2716-2722

```tablegen
def FTDIV: XForm_17<63, 128, (outs crrc:$BF), (ins f8rc:$RA, f8rc:$RB),
                      "ftdiv $BF, $RA, $RB", IIC_FPCompare>;
def FTSQRT: XForm_17a<63, 160, (outs crrc:$BF), (ins f8rc:$RB),
                      "ftsqrt $BF, $RB", IIC_FPCompare,
                      [(set i32:$BF, (PPCftsqrt f64:$RB))]>;

let mayRaiseFPException = 1, hasSideEffects = 0 in {
```
- **EN**: Adds declarative TableGen records such as `FTDIV`, `FTSQRT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FTDIV`, `FTSQRT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2723-2730

```tablegen
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in
  defm FRIND  : XForm_26r<63, 392, (outs f8rc:$RST), (ins f8rc:$RB),
                          "frin", "$RST, $RB", IIC_FPGeneral,
                          [(set f64:$RST, (any_fround f64:$RB))]>;
  defm FRINS  : XForm_26r<63, 392, (outs f4rc:$RST), (ins f4rc:$RB),
                          "frin", "$RST, $RB", IIC_FPGeneral,
                          [(set f32:$RST, (any_fround f32:$RB))]>;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2731-2737

```tablegen
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in
  defm FRIPD  : XForm_26r<63, 456, (outs f8rc:$RST), (ins f8rc:$RB),
                          "frip", "$RST, $RB", IIC_FPGeneral,
                          [(set f64:$RST, (any_fceil f64:$RB))]>;
  defm FRIPS  : XForm_26r<63, 456, (outs f4rc:$RST), (ins f4rc:$RB),
                          "frip", "$RST, $RB", IIC_FPGeneral,
                          [(set f32:$RST, (any_fceil f32:$RB))]>;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2738-2744

```tablegen
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in
  defm FRIZD  : XForm_26r<63, 424, (outs f8rc:$RST), (ins f8rc:$RB),
                          "friz", "$RST, $RB", IIC_FPGeneral,
                          [(set f64:$RST, (any_ftrunc f64:$RB))]>;
  defm FRIZS  : XForm_26r<63, 424, (outs f4rc:$RST), (ins f4rc:$RB),
                          "friz", "$RST, $RB", IIC_FPGeneral,
                          [(set f32:$RST, (any_ftrunc f32:$RB))]>;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2745-2753

```tablegen
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in
  defm FRIMD  : XForm_26r<63, 488, (outs f8rc:$RST), (ins f8rc:$RB),
                          "frim", "$RST, $RB", IIC_FPGeneral,
                          [(set f64:$RST, (any_ffloor f64:$RB))]>;
  defm FRIMS  : XForm_26r<63, 488, (outs f4rc:$RST), (ins f4rc:$RB),
                          "frim", "$RST, $RB", IIC_FPGeneral,
                          [(set f32:$RST, (any_ffloor f32:$RB))]>;
}
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2754-2760

```tablegen
let Uses = [RM], mayRaiseFPException = 1, hasSideEffects = 0 in {
  defm FCTIW  : XForm_26r<63, 14, (outs f8rc:$RST), (ins f8rc:$RB),
                          "fctiw", "$RST, $RB", IIC_FPGeneral,
                          []>;
  defm FCTIWU  : XForm_26r<63, 142, (outs f8rc:$RST), (ins f8rc:$RB),
                          "fctiwu", "$RST, $RB", IIC_FPGeneral,
                          []>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2761-2768

```tablegen
  defm FCTIWZ : XForm_26r<63, 15, (outs f8rc:$RST), (ins f8rc:$RB),
                          "fctiwz", "$RST, $RB", IIC_FPGeneral,
                          [(set f64:$RST, (PPCany_fctiwz f64:$RB))]>;

  defm FRSP   : XForm_26r<63, 12, (outs f4rc:$RST), (ins f8rc:$RB),
                          "frsp", "$RST, $RB", IIC_FPGeneral,
                          [(set f32:$RST, (any_fpround f64:$RB))]>;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2769-2777

```tablegen
  defm FSQRT  : XForm_26r<63, 22, (outs f8rc:$RST), (ins f8rc:$RB),
                          "fsqrt", "$RST, $RB", IIC_FPSqrtD,
                          [(set f64:$RST, (any_fsqrt f64:$RB))]>;
  defm FSQRTS : XForm_26r<59, 22, (outs f4rc:$RST), (ins f4rc:$RB),
                          "fsqrts", "$RST, $RB", IIC_FPSqrtS,
                          [(set f32:$RST, (any_fsqrt f32:$RB))]>;
}
}
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2778-2784

```tablegen
def : Pat<(PPCfsqrt f64:$frA), (FSQRT $frA)>;

/// Note that FMR is defined as pseudo-ops on the PPC970 because they are
/// often coalesced away and we don't want the dispatch group builder to think
/// that they will fill slots (which could cause the load of a LSU reject to
/// sneak into a d-group with a store).
let hasSideEffects = 0, Predicates = [HasFPU] in
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Note that FMR is defined as pseudo-ops on the PPC970 because they are". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Note that FMR is defined as pseudo-ops on the PPC970 because they are”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2785-2791

```tablegen
defm FMR   : XForm_26r<63, 72, (outs f4rc:$RST), (ins f4rc:$RB),
                       "fmr", "$RST, $RB", IIC_FPGeneral,
                       []>,  // (set f32:$RST, f32:$RB)
                       PPC970_Unit_Pseudo;

let PPC970_Unit = 3, hasSideEffects = 0, Predicates = [HasFPU] in {  // FPU Operations.
// These are artificially split into two different forms, for 4/8 byte FP.
```
- **EN**: Adds declarative TableGen records such as `FMR` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FMR`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2792-2798

```tablegen
defm FABSS  : XForm_26r<63, 264, (outs f4rc:$RST), (ins f4rc:$RB),
                        "fabs", "$RST, $RB", IIC_FPGeneral,
                        [(set f32:$RST, (fabs f32:$RB))]>;
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
defm FABSD  : XForm_26r<63, 264, (outs f8rc:$RST), (ins f8rc:$RB),
                        "fabs", "$RST, $RB", IIC_FPGeneral,
                        [(set f64:$RST, (fabs f64:$RB))]>;
```
- **EN**: Adds declarative TableGen records such as `FABSS`, `FABSD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FABSS`, `FABSD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2799-2805

```tablegen
defm FNABSS : XForm_26r<63, 136, (outs f4rc:$RST), (ins f4rc:$RB),
                        "fnabs", "$RST, $RB", IIC_FPGeneral,
                        [(set f32:$RST, (fneg (fabs f32:$RB)))]>;
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
defm FNABSD : XForm_26r<63, 136, (outs f8rc:$RST), (ins f8rc:$RB),
                        "fnabs", "$RST, $RB", IIC_FPGeneral,
                        [(set f64:$RST, (fneg (fabs f64:$RB)))]>;
```
- **EN**: Adds declarative TableGen records such as `FNABSS`, `FNABSD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FNABSS`, `FNABSD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2806-2813

```tablegen
defm FNEGS  : XForm_26r<63, 40, (outs f4rc:$RST), (ins f4rc:$RB),
                        "fneg", "$RST, $RB", IIC_FPGeneral,
                        [(set f32:$RST, (fneg f32:$RB))]>;
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
defm FNEGD  : XForm_26r<63, 40, (outs f8rc:$RST), (ins f8rc:$RB),
                        "fneg", "$RST, $RB", IIC_FPGeneral,
                        [(set f64:$RST, (fneg f64:$RB))]>;
```
- **EN**: Adds declarative TableGen records such as `FNEGS`, `FNEGD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FNEGS`, `FNEGD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2814-2822

```tablegen
defm FCPSGNS : XForm_28r<63, 8, (outs f4rc:$RST), (ins f4rc:$RA, f4rc:$RB),
                        "fcpsgn", "$RST, $RA, $RB", IIC_FPGeneral,
                        [(set f32:$RST, (fcopysign f32:$RB, f32:$RA))]>;
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
defm FCPSGND : XForm_28r<63, 8, (outs f8rc:$RST), (ins f8rc:$RA, f8rc:$RB),
                        "fcpsgn", "$RST, $RA, $RB", IIC_FPGeneral,
                        [(set f64:$RST, (fcopysign f64:$RB, f64:$RA))]>;

// Reciprocal estimates.
```
- **EN**: Adds declarative TableGen records such as `FCPSGNS`, `FCPSGND` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FCPSGNS`, `FCPSGND`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2823-2830

```tablegen
let mayRaiseFPException = 1 in {
// Reciprocal estimate instructions (unary FP ops).
defm FRE      : XForm_26r<63, 24, (outs f8rc:$RST), (ins f8rc:$RB),
                          "fre", "$RST, $RB", IIC_FPGeneral,
                          [(set f64:$RST, (PPCfre f64:$RB))]>;
defm FRES     : XForm_26r<59, 24, (outs f4rc:$RST), (ins f4rc:$RB),
                          "fres", "$RST, $RB", IIC_FPGeneral,
                          [(set f32:$RST, (PPCfre f32:$RB))]>;
```
- **EN**: Adds declarative TableGen records such as `FRE`, `FRES` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FRE`, `FRES`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2831-2841

```tablegen
defm FRSQRTE  : XForm_26r<63, 26, (outs f8rc:$RST), (ins f8rc:$RB),
                          "frsqrte", "$RST, $RB", IIC_FPGeneral,
                          [(set f64:$RST, (PPCfrsqrte f64:$RB))]>;
defm FRSQRTES : XForm_26r<59, 26, (outs f4rc:$RST), (ins f4rc:$RB),
                          "frsqrtes", "$RST, $RB", IIC_FPGeneral,
                          [(set f32:$RST, (PPCfrsqrte f32:$RB))]>;
}
}

// XL-Form instructions.  condition register logical ops.
//
```
- **EN**: Adds declarative TableGen records such as `FRSQRTE`, `FRSQRTES` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FRSQRTE`, `FRSQRTES`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2842-2855

```tablegen
let hasSideEffects = 0 in
def MCRF   : XLForm_3<19, 0, (outs crrc:$BF), (ins crrc:$BFA),
                      "mcrf $BF, $BFA", IIC_BrMCR>,
             PPC970_DGroup_First, PPC970_Unit_CRU;

// FIXME: According to the ISA (section 2.5.1 of version 2.06), the
// condition-register logical instructions have preferred forms. Specifically,
// it is preferred that the bit specified by the BT field be in the same
// condition register as that specified by the bit BB. We might want to account
// for this via hinting the register allocator and anti-dep breakers, or we
// could constrain the register class to force this constraint and then loosen
// it during register allocation via convertToThreeAddress or some similar
// mechanism.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FIXME: According to the ISA (section 2.5.1 of version 2.06), the". Notable symbols in this range include `MCRF`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FIXME: According to the ISA (section 2.5.1 of version 2.06), the”。 该区间中较显眼的符号包括 `MCRF`。

### Lines 2856-2866

```tablegen
let isCommutable = 1 in {
def CRAND  : XLForm_1<19, 257, (outs crbitrc:$CRD),
                               (ins crbitrc:$CRA, crbitrc:$CRB),
                      "crand $CRD, $CRA, $CRB", IIC_BrCR,
                      [(set i1:$CRD, (and i1:$CRA, i1:$CRB))]>;

def CRNAND : XLForm_1<19, 225, (outs crbitrc:$CRD),
                               (ins crbitrc:$CRA, crbitrc:$CRB),
                      "crnand $CRD, $CRA, $CRB", IIC_BrCR,
                      [(set i1:$CRD, (not (and i1:$CRA, i1:$CRB)))]>;
```
- **EN**: Adds declarative TableGen records such as `CRAND`, `CRNAND` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CRAND`, `CRNAND`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2867-2876

```tablegen
def CROR   : XLForm_1<19, 449, (outs crbitrc:$CRD),
                               (ins crbitrc:$CRA, crbitrc:$CRB),
                      "cror $CRD, $CRA, $CRB", IIC_BrCR,
                      [(set i1:$CRD, (or i1:$CRA, i1:$CRB))]>;

def CRXOR  : XLForm_1<19, 193, (outs crbitrc:$CRD),
                               (ins crbitrc:$CRA, crbitrc:$CRB),
                      "crxor $CRD, $CRA, $CRB", IIC_BrCR,
                      [(set i1:$CRD, (xor i1:$CRA, i1:$CRB))]>;
```
- **EN**: Adds declarative TableGen records such as `CROR`, `CRXOR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CROR`, `CRXOR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2877-2886

```tablegen
def CRNOR  : XLForm_1<19, 33, (outs crbitrc:$CRD),
                              (ins crbitrc:$CRA, crbitrc:$CRB),
                      "crnor $CRD, $CRA, $CRB", IIC_BrCR,
                      [(set i1:$CRD, (not (or i1:$CRA, i1:$CRB)))]>;
def CREQV  : XLForm_1<19, 289, (outs crbitrc:$CRD),
                               (ins crbitrc:$CRA, crbitrc:$CRB),
                      "creqv $CRD, $CRA, $CRB", IIC_BrCR,
                      [(set i1:$CRD, (not (xor i1:$CRA, i1:$CRB)))]>;
} // isCommutable
```
- **EN**: Adds declarative TableGen records such as `CRNOR`, `CREQV` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CRNOR`, `CREQV`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2887-2896

```tablegen
let isCodeGenOnly = 1 in
def CRNOT  : XLForm_1s<19, 33, (outs crbitrc:$CRD), (ins crbitrc:$CRA),
                       "crnot $CRD, $CRA", IIC_BrCR,
                       [(set i1:$CRD, (not i1:$CRA))]>;

def CRANDC : XLForm_1<19, 129, (outs crbitrc:$CRD),
                               (ins crbitrc:$CRA, crbitrc:$CRB),
                      "crandc $CRD, $CRA, $CRB", IIC_BrCR,
                      [(set i1:$CRD, (and i1:$CRA, (not i1:$CRB)))]>;
```
- **EN**: Adds declarative TableGen records such as `CRNOT`, `CRANDC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CRNOT`, `CRANDC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2897-2903

```tablegen
def CRORC  : XLForm_1<19, 417, (outs crbitrc:$CRD),
                               (ins crbitrc:$CRA, crbitrc:$CRB),
                      "crorc $CRD, $CRA, $CRB", IIC_BrCR,
                      [(set i1:$CRD, (or i1:$CRA, (not i1:$CRB)))]>;

let isCodeGenOnly = 1 in {
let isReMaterializable = 1, isAsCheapAsAMove = 1 in {
```
- **EN**: Adds declarative TableGen records such as `CRORC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CRORC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2904-2912

```tablegen
def CRSET  : XLForm_1_ext<19, 289, (outs crbitrc:$CRD), (ins),
              "creqv $CRD, $CRD, $CRD", IIC_BrCR,
              [(set i1:$CRD, 1)]>;

def CRUNSET: XLForm_1_ext<19, 193, (outs crbitrc:$CRD), (ins),
              "crxor $CRD, $CRD, $CRD", IIC_BrCR,
              [(set i1:$CRD, 0)]>;
}
```
- **EN**: Adds declarative TableGen records such as `CRSET`, `CRUNSET` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CRSET`, `CRUNSET`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2913-2926

```tablegen
let Defs = [CR1EQ], CRD = 6 in {
def CR6SET  : XLForm_1_ext<19, 289, (outs), (ins),
              "creqv 6, 6, 6", IIC_BrCR,
              [(PPCcr6set)]>;

def CR6UNSET: XLForm_1_ext<19, 193, (outs), (ins),
              "crxor 6, 6, 6", IIC_BrCR,
              [(PPCcr6unset)]>;
}
}

// XFX-Form instructions.  Instructions that deal with SPRs.
//
```
- **EN**: Adds declarative TableGen records such as `CR6SET`, `CR6UNSET` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CR6SET`, `CR6UNSET`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2927-2934

```tablegen
def MFSPR : XFXForm_1<31, 339, (outs gprc:$RST), (ins i32imm:$SPR),
                      "mfspr $RST, $SPR", IIC_SprMFSPR>;
def MTSPR : XFXForm_1<31, 467, (outs), (ins i32imm:$SPR, gprc:$RST),
                      "mtspr $SPR, $RST", IIC_SprMTSPR>;

def MFTB : XFXForm_1<31, 371, (outs gprc:$RST), (ins i32imm:$SPR),
                     "mftb $RST, $SPR", IIC_SprMFTB>;
```
- **EN**: Adds declarative TableGen records such as `MFSPR`, `MTSPR`, `MFTB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MFSPR`, `MTSPR`, `MFTB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2935-2943

```tablegen
def MFPMR : XFXForm_1<31, 334, (outs gprc:$RST), (ins i32imm:$SPR),
                     "mfpmr $RST, $SPR", IIC_SprMFPMR>;

def MTPMR : XFXForm_1<31, 462, (outs), (ins i32imm:$SPR, gprc:$RST),
                     "mtpmr $SPR, $RST", IIC_SprMTPMR>;


// A pseudo-instruction used to implement the read of the 64-bit cycle counter
// on a 32-bit target.
```
- **EN**: Adds declarative TableGen records such as `MFPMR`, `MTPMR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MFPMR`, `MTPMR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2944-2952

```tablegen
let hasSideEffects = 1 in
def ReadTB : PPCCustomInserterPseudo<(outs gprc:$lo, gprc:$hi), (ins),
                    "#ReadTB", []>;

let Uses = [CTR] in {
def MFCTR : XFXForm_1_ext<31, 339, 9, (outs gprc:$RST), (ins),
                          "mfctr $RST", IIC_SprMFSPR>,
            PPC970_DGroup_First, PPC970_Unit_FXU;
}
```
- **EN**: Adds declarative TableGen records such as `ReadTB`, `MFCTR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ReadTB`, `MFCTR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2953-2959

```tablegen
let Defs = [CTR], Pattern = [(PPCmtctr i32:$RST)] in {
def MTCTR : XFXForm_1_ext<31, 467, 9, (outs), (ins gprc:$RST),
                          "mtctr $RST", IIC_SprMTSPR>,
            PPC970_DGroup_First, PPC970_Unit_FXU;
}
let hasSideEffects = 1, isCodeGenOnly = 1, isNotDuplicable = 1, Defs = [CTR] in {
let Pattern = [(int_set_loop_iterations i32:$RST)] in
```
- **EN**: Adds declarative TableGen records such as `MTCTR` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTCTR`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2960-2968

```tablegen
def MTCTRloop : XFXForm_1_ext<31, 467, 9, (outs), (ins gprc:$RST),
                              "mtctr $RST", IIC_SprMTSPR>,
                PPC970_DGroup_First, PPC970_Unit_FXU;
}

let hasSideEffects = 1, hasNoSchedulingInfo = 1, isNotDuplicable = 1, Uses = [CTR], Defs = [CTR] in
def DecreaseCTRloop : PPCEmitTimePseudo<(outs crbitrc:$rT), (ins i32imm:$stride),
                                       "#DecreaseCTRloop", [(set i1:$rT, (int_loop_decrement (i32 imm:$stride)))]>;
```
- **EN**: Adds declarative TableGen records such as `MTCTRloop`, `DecreaseCTRloop` that LLVM later expands into generated tables or helper code. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTCTRloop`, `DecreaseCTRloop`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段包含调度或处理器模型元数据。

### Lines 2969-2975

```tablegen
let hasSideEffects = 0 in {
let Defs = [LR] in {
def MTLR  : XFXForm_1_ext<31, 467, 8, (outs), (ins gprc:$RST),
                          "mtlr $RST", IIC_SprMTSPR>,
            PPC970_DGroup_First, PPC970_Unit_FXU;
}
let Uses = [LR] in {
```
- **EN**: Adds declarative TableGen records such as `MTLR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTLR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2976-2982

```tablegen
def MFLR  : XFXForm_1_ext<31, 339, 8, (outs gprc:$RST), (ins),
                          "mflr $RST", IIC_SprMFSPR>,
            PPC970_DGroup_First, PPC970_Unit_FXU;
}
}

let hasSideEffects = 1 in {
```
- **EN**: Adds declarative TableGen records such as `MFLR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MFLR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2983-2991

```tablegen
  def MTUDSCR : XFXForm_1_ext<31, 467, 3, (outs), (ins gprc:$RST),
                              "mtspr 3, $RST", IIC_SprMTSPR>,
                PPC970_DGroup_Single, PPC970_Unit_FXU;
  def MFUDSCR : XFXForm_1_ext<31, 339, 3, (outs gprc:$RST), (ins),
                              "mfspr $RST, 3", IIC_SprMFSPR>,
                PPC970_DGroup_First, PPC970_Unit_FXU;
}

// Disable these alias on AIX since they are not supported.
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 2992-3001

```tablegen
let Predicates = [ModernAs] in {
// Aliases for moving to/from dscr to mtspr/mfspr
def : InstAlias<"mtudscr $Rx", (MTUDSCR gprc:$Rx)>;
def : InstAlias<"mfudscr $Rx", (MFUDSCR gprc:$Rx)>;
}

let isCodeGenOnly = 1 in {
  // Move to/from VRSAVE: despite being a SPR, the VRSAVE register is renamed
  // like a GPR on the PPC970.  As such, copies in and out have the same
  // performance characteristics as an OR instruction.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Aliases for moving to/from dscr to mtspr/mfspr". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Aliases for moving to/from dscr to mtspr/mfspr”。 这些声明会进入生成式模式匹配逻辑。

### Lines 3002-3008

```tablegen
  def MTVRSAVE : XFXForm_1_ext<31, 467, 256, (outs), (ins gprc:$RST),
                               "mtspr 256, $RST", IIC_IntGeneral>,
                 PPC970_DGroup_Single, PPC970_Unit_FXU;
  def MFVRSAVE : XFXForm_1_ext<31, 339, 256, (outs gprc:$RST), (ins),
                               "mfspr $RST, 256", IIC_IntGeneral>,
                 PPC970_DGroup_First, PPC970_Unit_FXU;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 3009-3019

```tablegen
  def MTVRSAVEv : XFXForm_1_ext<31, 467, 256,
                                (outs VRSAVERC:$SPR), (ins gprc:$RST),
                                "mtspr 256, $RST", IIC_IntGeneral>,
                  PPC970_DGroup_Single, PPC970_Unit_FXU;
  def MFVRSAVEv : XFXForm_1_ext<31, 339, 256, (outs gprc:$RST),
                                (ins VRSAVERC:$SPR),
                                "mfspr $RST, 256", IIC_IntGeneral>,
                  PPC970_DGroup_First, PPC970_Unit_FXU;
}

// Aliases for mtvrsave/mfvrsave to mfspr/mtspr.
```
- **EN**: Contains iterative logic that walks instructions, operands, or tables to apply a backend transformation step by step.
- **CN**: 这里包含迭代逻辑，会逐步遍历指令、操作数或表项来执行后端变换。

### Lines 3020-3026

```tablegen
def : InstAlias<"mtvrsave $rS", (MTVRSAVE gprc:$rS)>;
def : InstAlias<"mfvrsave $rS", (MFVRSAVE gprc:$rS)>;

let hasSideEffects = 0 in {
// mtocrf's input needs to be prepared by shifting by an amount dependent
// on the cr register selected. Thus, post-ra anti-dep breaking must not
// later change that register assignment.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "mtocrf's input needs to be prepared by shifting by an amount dependent".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“mtocrf's input needs to be prepared by shifting by an amount dependent”。

### Lines 3027-3033

```tablegen
let hasExtraDefRegAllocReq = 1 in {
def MTOCRF: XFXForm_5a<31, 144, (outs crbitm:$FXM), (ins gprc:$RST),
                       "mtocrf $FXM, $RST", IIC_BrMCRX>,
            PPC970_DGroup_First, PPC970_Unit_CRU;

// Similarly to mtocrf, the mask for mtcrf must be prepared in a way that
// is dependent on the cr fields being set.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Similarly to mtocrf, the mask for mtcrf must be prepared in a way that". Notable symbols in this range include `MTOCRF`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Similarly to mtocrf, the mask for mtcrf must be prepared in a way that”。 该区间中较显眼的符号包括 `MTOCRF`。

### Lines 3034-3041

```tablegen
def MTCRF : XFXForm_5<31, 144, (outs), (ins i32imm:$FXM, gprc:$RST),
                      "mtcrf $FXM, $RST", IIC_BrMCRX>,
            PPC970_MicroCode, PPC970_Unit_CRU;
} // hasExtraDefRegAllocReq = 1

// mfocrf's input needs to be prepared by shifting by an amount dependent
// on the cr register selected. Thus, post-ra anti-dep breaking must not
// later change that register assignment.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "mfocrf's input needs to be prepared by shifting by an amount dependent". Notable symbols in this range include `MTCRF`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“mfocrf's input needs to be prepared by shifting by an amount dependent”。 该区间中较显眼的符号包括 `MTCRF`。

### Lines 3042-3048

```tablegen
let hasExtraSrcRegAllocReq = 1 in {
def MFOCRF: XFXForm_5a<31, 19, (outs gprc:$RST), (ins crbitm:$FXM),
                       "mfocrf $RST, $FXM", IIC_SprMFCRF>,
            PPC970_DGroup_First, PPC970_Unit_CRU;

// Similarly to mfocrf, the mask for mfcrf must be prepared in a way that
// is dependent on the cr fields being copied.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Similarly to mfocrf, the mask for mfcrf must be prepared in a way that". Notable symbols in this range include `MFOCRF`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Similarly to mfocrf, the mask for mfcrf must be prepared in a way that”。 该区间中较显眼的符号包括 `MFOCRF`。

### Lines 3049-3057

```tablegen
def MFCR : XFXForm_3<31, 19, (outs gprc:$RT), (ins),
                     "mfcr $RT", IIC_SprMFCR>,
                     PPC970_MicroCode, PPC970_Unit_CRU;
} // hasExtraSrcRegAllocReq = 1

def MCRXRX : X_BF3<31, 576, (outs crrc:$BF), (ins),
                   "mcrxrx $BF", IIC_BrMCRX>, Requires<[IsISA3_0]>;
} // hasSideEffects = 0
```
- **EN**: Adds declarative TableGen records such as `MFCR`, `MCRXRX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MFCR`, `MCRXRX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3058-3071

```tablegen
def : InstAlias<"mtcr $rA", (MTCRF 255, gprc:$rA)>;

let Predicates = [HasFPU] in {
// Custom inserter instruction to perform FADD in round-to-zero mode.
let Uses = [RM], mayRaiseFPException = 1 in {
  def FADDrtz: PPCCustomInserterPseudo<(outs f8rc:$FRT), (ins f8rc:$FRA, f8rc:$FRB), "",
                      [(set f64:$FRT, (PPCany_faddrtz f64:$FRA, f64:$FRB))]>;
}

// The above pseudo gets expanded to make use of the following instructions
// to manipulate FPSCR.  Note that FPSCR is not modeled at the DAG level.

// When FM is 30/31, we are setting the 62/63 bit of FPSCR, the implicit-def
// RM should be set.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Custom inserter instruction to perform FADD in round-to-zero mode.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Custom inserter instruction to perform FADD in round-to-zero mode.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 3072-3082

```tablegen
let hasSideEffects = 1, Defs = [RM] in {
def MTFSB0 : XForm_43<63, 70, (outs), (ins u5imm:$FM),
                      "mtfsb0 $FM", IIC_IntMTFSB0,
                      [(int_ppc_mtfsb0 u5imm_timm:$FM)]>,
             PPC970_DGroup_Single, PPC970_Unit_FPU;
def MTFSB1 : XForm_43<63, 38, (outs), (ins u5imm:$FM),
                      "mtfsb1 $FM", IIC_IntMTFSB0,
                      [(int_ppc_mtfsb1 u5imm_timm:$FM)]>,
             PPC970_DGroup_Single, PPC970_Unit_FPU;
}
```
- **EN**: Adds declarative TableGen records such as `MTFSB0`, `MTFSB1` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTFSB0`, `MTFSB1`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3083-3089

```tablegen
let Defs = [RM], hasSideEffects = 1 in {
  let isCodeGenOnly = 1 in
  def MTFSFb  : XFLForm<63, 711, (outs), (ins i32imm:$FM, f8rc:$RT),
                        "mtfsf $FM, $RT", IIC_IntMTFSB0,
                        [(int_ppc_mtfsf timm:$FM, f64:$RT)]>,
                PPC970_DGroup_Single, PPC970_Unit_FPU;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3090-3096

```tablegen
let Uses = [RM], hasSideEffects = 1 in {
  def MFFS   : XForm_42<63, 583, (outs f8rc:$RST), (ins),
                         "mffs $RST", IIC_IntMFFS,
                         [(set f64:$RST, (PPCmffs))]>,
               PPC970_DGroup_Single, PPC970_Unit_FPU;

  let Defs = [CR1] in
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3097-3103

```tablegen
  def MFFS_rec : XForm_42<63, 583, (outs f8rc:$RST), (ins),
                      "mffs. $RST", IIC_IntMFFS, []>, isRecordForm;

  def MFFSCE : X_FRT5_XO2_XO3_XO10<63, 0, 1, 583, (outs f8rc:$RST), (ins),
                                  "mffsce $RST", IIC_IntMFFS, []>,
               PPC970_DGroup_Single, PPC970_Unit_FPU;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 3104-3114

```tablegen
  def MFFSCDRN : X_FRT5_XO2_XO3_FRB5_XO10<63, 2, 4, 583, (outs f8rc:$RST),
                                         (ins f8rc:$FRB), "mffscdrn $RST, $FRB",
                                         IIC_IntMFFS, []>,
                 PPC970_DGroup_Single, PPC970_Unit_FPU;

  def MFFSCDRNI : X_FRT5_XO2_XO3_DRM3_XO10<63, 2, 5, 583, (outs f8rc:$RST),
                                          (ins u3imm:$DRM),
                                          "mffscdrni $RST, $DRM",
                                          IIC_IntMFFS, []>,
                  PPC970_DGroup_Single, PPC970_Unit_FPU;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 3115-3124

```tablegen
  def MFFSCRN : X_FRT5_XO2_XO3_FRB5_XO10<63, 2, 6, 583, (outs f8rc:$RST),
                                        (ins f8rc:$FRB), "mffscrn $RST, $FRB",
                                        IIC_IntMFFS, []>,
                PPC970_DGroup_Single, PPC970_Unit_FPU;

  def MFFSCRNI : X_FRT5_XO2_XO3_RM2_X10<63, 2, 7, 583, (outs f8rc:$RST),
                                       (ins u2imm:$RM), "mffscrni $RST, $RM",
                                       IIC_IntMFFS, []>,
                 PPC970_DGroup_Single, PPC970_Unit_FPU;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 3125-3131

```tablegen
  def MFFSL  : X_FRT5_XO2_XO3_XO10<63, 3, 0, 583, (outs f8rc:$RST), (ins),
                                  "mffsl $RST", IIC_IntMFFS, []>,
               PPC970_DGroup_Single, PPC970_Unit_FPU;
}
}

let Predicates = [IsISA3_0] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3132-3138

```tablegen
def MODSW : XForm_8<31, 779, (outs gprc:$RST), (ins gprc:$RA, gprc:$RB),
                        "modsw $RST, $RA, $RB", IIC_IntDivW,
                        [(set i32:$RST, (srem i32:$RA, i32:$RB))]>;
def MODUW : XForm_8<31, 267, (outs gprc:$RST), (ins gprc:$RA, gprc:$RB),
                        "moduw $RST, $RA, $RB", IIC_IntDivW,
                        [(set i32:$RST, (urem i32:$RA, i32:$RB))]>;
let hasSideEffects = 1 in
```
- **EN**: Adds declarative TableGen records such as `MODSW`, `MODUW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MODSW`, `MODUW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3139-3145

```tablegen
def ADDEX : Z23Form_RTAB5_CY2<31, 170, (outs gprc:$RT),
                              (ins gprc:$RA, gprc:$RB, u2imm:$CY),
                              "addex $RT, $RA, $RB, $CY", IIC_IntGeneral, []>;
}

let PPC970_Unit = 1, hasSideEffects = 0 in {  // FXU Operations.
// XO-Form instructions.  Arithmetic instructions that can set overflow bit
```
- **EN**: Adds declarative TableGen records such as `ADDEX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDEX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3146-3153

```tablegen
let isCommutable = 1 in
defm ADD4  : XOForm_1rx<31, 266, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                        "add", "$RT, $RA, $RB", IIC_IntSimple,
                        [(set i32:$RT, (add i32:$RA, i32:$RB))]>;
let isCodeGenOnly = 1 in
def ADD4TLS  : XOForm_1<31, 266, 0, (outs gprc:$RT), (ins gprc:$RA, tlsreg32:$RB),
                       "add $RT, $RA, $RB", IIC_IntSimple,
                       [(set i32:$RT, (add i32:$RA, tglobaltlsaddr:$RB))]>;
```
- **EN**: Adds declarative TableGen records such as `ADD4`, `ADD4TLS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADD4`, `ADD4TLS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3154-3162

```tablegen
let isCommutable = 1 in
defm ADDC  : XOForm_1rc<31, 10, 0, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                        "addc", "$RT, $RA, $RB", IIC_IntGeneral,
                        [(set i32:$RT, (PPCaddc i32:$RA, i32:$RB))]>,
                        PPC970_DGroup_Cracked, Requires<[IsPPC32]>;

defm DIVW  : XOForm_1rcr<31, 491, 0, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                          "divw", "$RT, $RA, $RB", IIC_IntDivW,
                          [(set i32:$RT, (sdiv i32:$RA, i32:$RB))]>;
```
- **EN**: Adds declarative TableGen records such as `ADDC`, `DIVW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDC`, `DIVW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3163-3169

```tablegen
defm DIVWU : XOForm_1rcr<31, 459, 0, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                          "divwu", "$RT, $RA, $RB", IIC_IntDivW,
                          [(set i32:$RT, (udiv i32:$RA, i32:$RB))]>;
defm DIVWE : XOForm_1rcr<31, 427, 0, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                         "divwe", "$RT, $RA, $RB", IIC_IntDivW,
                         [(set i32:$RT, (int_ppc_divwe gprc:$RA, gprc:$RB))]>,
                         Requires<[HasExtDiv]>;
```
- **EN**: Adds declarative TableGen records such as `DIVWU`, `DIVWE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DIVWU`, `DIVWE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3170-3177

```tablegen
defm DIVWEU : XOForm_1rcr<31, 395, 0, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                          "divweu", "$RT, $RA, $RB", IIC_IntDivW,
                          [(set i32:$RT, (int_ppc_divweu gprc:$RA, gprc:$RB))]>,
                          Requires<[HasExtDiv]>;
let isCommutable = 1 in {
defm MULHW : XOForm_1r<31, 75, 0, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                       "mulhw", "$RT, $RA, $RB", IIC_IntMulHW,
                       [(set i32:$RT, (mulhs i32:$RA, i32:$RB))]>;
```
- **EN**: Adds declarative TableGen records such as `DIVWEU`, `MULHW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DIVWEU`, `MULHW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3178-3184

```tablegen
defm MULHWU : XOForm_1r<31, 11, 0, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                       "mulhwu", "$RT, $RA, $RB", IIC_IntMulHWU,
                       [(set i32:$RT, (mulhu i32:$RA, i32:$RB))]>;
defm MULLW : XOForm_1rx<31, 235, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                        "mullw", "$RT, $RA, $RB", IIC_IntMulHW,
                        [(set i32:$RT, (mul i32:$RA, i32:$RB))]>;
} // isCommutable
```
- **EN**: Adds declarative TableGen records such as `MULHWU`, `MULLW` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MULHWU`, `MULLW`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3185-3191

```tablegen
defm SUBF  : XOForm_1rx<31, 40, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                        "subf", "$RT, $RA, $RB", IIC_IntGeneral,
                        [(set i32:$RT, (sub i32:$RB, i32:$RA))]>;
defm SUBFC : XOForm_1rc<31, 8, 0, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                        "subfc", "$RT, $RA, $RB", IIC_IntGeneral,
                        [(set i32:$RT, (PPCsubc i32:$RB, i32:$RA))]>,
                        PPC970_DGroup_Cracked, Requires<[IsPPC32]>;
```
- **EN**: Adds declarative TableGen records such as `SUBF`, `SUBFC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SUBF`, `SUBFC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3192-3200

```tablegen
defm NEG    : XOForm_3r<31, 104, 0, (outs gprc:$RT), (ins gprc:$RA),
                        "neg", "$RT, $RA", IIC_IntSimple,
                        [(set i32:$RT, (ineg i32:$RA))]>;
let Uses = [CARRY] in {
let isCommutable = 1 in
defm ADDE  : XOForm_1rc<31, 138, 0, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                        "adde", "$RT, $RA, $RB", IIC_IntGeneral,
                        [(set i32:$RT, (PPCadde i32:$RA, i32:$RB, CARRY))]>,
                        Requires<[IsPPC32]>;
```
- **EN**: Adds declarative TableGen records such as `NEG`, `ADDE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `NEG`, `ADDE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3201-3208

```tablegen
defm ADDME  : XOForm_3rc<31, 234, 0, (outs gprc:$RT), (ins gprc:$RA),
                         "addme", "$RT, $RA", IIC_IntGeneral,
                         [(set i32:$RT, (PPCadde i32:$RA, -1, CARRY))]>,
                         Requires<[IsPPC32]>;
defm ADDZE  : XOForm_3rc<31, 202, 0, (outs gprc:$RT), (ins gprc:$RA),
                         "addze", "$RT, $RA", IIC_IntGeneral,
                         [(set i32:$RT, (PPCadde i32:$RA, 0, CARRY))]>,
                         Requires<[IsPPC32]>;
```
- **EN**: Adds declarative TableGen records such as `ADDME`, `ADDZE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDME`, `ADDZE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3209-3216

```tablegen
defm SUBFE : XOForm_1rc<31, 136, 0, (outs gprc:$RT), (ins gprc:$RA, gprc:$RB),
                        "subfe", "$RT, $RA, $RB", IIC_IntGeneral,
                        [(set i32:$RT, (PPCsube i32:$RB, i32:$RA, CARRY))]>,
                        Requires<[IsPPC32]>;
defm SUBFME : XOForm_3rc<31, 232, 0, (outs gprc:$RT), (ins gprc:$RA),
                         "subfme", "$RT, $RA", IIC_IntGeneral,
                         [(set i32:$RT, (PPCsube -1, i32:$RA, CARRY))]>,
                         Requires<[IsPPC32]>;
```
- **EN**: Adds declarative TableGen records such as `SUBFE`, `SUBFME` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SUBFE`, `SUBFME`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3217-3223

```tablegen
defm SUBFZE : XOForm_3rc<31, 200, 0, (outs gprc:$RT), (ins gprc:$RA),
                         "subfze", "$RT, $RA", IIC_IntGeneral,
                         [(set i32:$RT, (PPCsube 0, i32:$RA, CARRY))]>,
                         Requires<[IsPPC32]>;
}
}
```
- **EN**: Adds declarative TableGen records such as `SUBFZE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SUBFZE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3224-3231

```tablegen
def : InstAlias<"sub $rA, $rB, $rC", (SUBF gprc:$rA, gprc:$rC, gprc:$rB)>;
def : InstAlias<"sub. $rA, $rB, $rC", (SUBF_rec gprc:$rA, gprc:$rC, gprc:$rB)>;
def : InstAlias<"subc $rA, $rB, $rC", (SUBFC gprc:$rA, gprc:$rC, gprc:$rB)>;
def : InstAlias<"subc. $rA, $rB, $rC", (SUBFC_rec gprc:$rA, gprc:$rC, gprc:$rB)>;

// A-Form instructions.  Most of the instructions executed in the FPU are of
// this type.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "A-Form instructions.  Most of the instructions executed in the FPU are of".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“A-Form instructions.  Most of the instructions executed in the FPU are of”。

### Lines 3232-3238

```tablegen
let PPC970_Unit = 3, hasSideEffects = 0, Predicates = [HasFPU] in {  // FPU Operations.
let mayRaiseFPException = 1, Uses = [RM] in {
let isCommutable = 1 in {
  defm FMADD : AForm_1r<63, 29,
                      (outs f8rc:$FRT), (ins f8rc:$FRA, f8rc:$FRC, f8rc:$FRB),
                      "fmadd", "$FRT, $FRA, $FRC, $FRB", IIC_FPFused,
                      [(set f64:$FRT, (any_fma f64:$FRA, f64:$FRC, f64:$FRB))]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3239-3247

```tablegen
  defm FMADDS : AForm_1r<59, 29,
                      (outs f4rc:$FRT), (ins f4rc:$FRA, f4rc:$FRC, f4rc:$FRB),
                      "fmadds", "$FRT, $FRA, $FRC, $FRB", IIC_FPGeneral,
                      [(set f32:$FRT, (any_fma f32:$FRA, f32:$FRC, f32:$FRB))]>;
  defm FMSUB : AForm_1r<63, 28,
                      (outs f8rc:$FRT), (ins f8rc:$FRA, f8rc:$FRC, f8rc:$FRB),
                      "fmsub", "$FRT, $FRA, $FRC, $FRB", IIC_FPFused,
                      [(set f64:$FRT,
                            (any_fma f64:$FRA, f64:$FRC, (fneg f64:$FRB)))]>;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 3248-3257

```tablegen
  defm FMSUBS : AForm_1r<59, 28,
                      (outs f4rc:$FRT), (ins f4rc:$FRA, f4rc:$FRC, f4rc:$FRB),
                      "fmsubs", "$FRT, $FRA, $FRC, $FRB", IIC_FPGeneral,
                      [(set f32:$FRT,
                            (any_fma f32:$FRA, f32:$FRC, (fneg f32:$FRB)))]>;
  defm FNMADD : AForm_1r<63, 31,
                      (outs f8rc:$FRT), (ins f8rc:$FRA, f8rc:$FRC, f8rc:$FRB),
                      "fnmadd", "$FRT, $FRA, $FRC, $FRB", IIC_FPFused,
                      [(set f64:$FRT,
                            (fneg (any_fma f64:$FRA, f64:$FRC, f64:$FRB)))]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 3258-3267

```tablegen
  defm FNMADDS : AForm_1r<59, 31,
                      (outs f4rc:$FRT), (ins f4rc:$FRA, f4rc:$FRC, f4rc:$FRB),
                      "fnmadds", "$FRT, $FRA, $FRC, $FRB", IIC_FPGeneral,
                      [(set f32:$FRT,
                            (fneg (any_fma f32:$FRA, f32:$FRC, f32:$FRB)))]>;
  defm FNMSUB : AForm_1r<63, 30,
                      (outs f8rc:$FRT), (ins f8rc:$FRA, f8rc:$FRC, f8rc:$FRB),
                      "fnmsub", "$FRT, $FRA, $FRC, $FRB", IIC_FPFused,
                      [(set f64:$FRT, (fneg (any_fma f64:$FRA, f64:$FRC,
                                                 (fneg f64:$FRB))))]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 3268-3278

```tablegen
  defm FNMSUBS : AForm_1r<59, 30,
                      (outs f4rc:$FRT), (ins f4rc:$FRA, f4rc:$FRC, f4rc:$FRB),
                      "fnmsubs", "$FRT, $FRA, $FRC, $FRB", IIC_FPGeneral,
                      [(set f32:$FRT, (fneg (any_fma f32:$FRA, f32:$FRC,
                                                 (fneg f32:$FRB))))]>;
} // isCommutable
}
// FSEL is artificially split into 4 and 8-byte forms for the result.  To avoid
// having 4 of these, force the comparison to always be an 8-byte double (code
// should use an FMRSD if the input comparison value really wants to be a float)
// and 4/8 byte forms for the result and operand type..
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FSEL is artificially split into 4 and 8-byte forms for the result.  To avoid".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FSEL is artificially split into 4 and 8-byte forms for the result.  To avoid”。

### Lines 3279-3287

```tablegen
let Interpretation64Bit = 1, isCodeGenOnly = 1 in
defm FSELD : AForm_1r<63, 23,
                      (outs f8rc:$FRT), (ins f8rc:$FRA, f8rc:$FRC, f8rc:$FRB),
                      "fsel", "$FRT, $FRA, $FRC, $FRB", IIC_FPGeneral,
                      [(set f64:$FRT, (PPCfsel f64:$FRA, f64:$FRC, f64:$FRB))]>;
defm FSELS : AForm_1r<63, 23,
                      (outs f4rc:$FRT), (ins f8rc:$FRA, f4rc:$FRC, f4rc:$FRB),
                      "fsel", "$FRT, $FRA, $FRC, $FRB", IIC_FPGeneral,
                      [(set f32:$FRT, (PPCfsel f64:$FRA, f32:$FRC, f32:$FRB))]>;
```
- **EN**: Adds declarative TableGen records such as `FSELD`, `FSELS` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FSELD`, `FSELS`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3288-3298

```tablegen
let Uses = [RM], mayRaiseFPException = 1 in {
  let isCommutable = 1 in {
  defm FADD  : AForm_2r<63, 21,
                        (outs f8rc:$FRT), (ins f8rc:$FRA, f8rc:$FRB),
                        "fadd", "$FRT, $FRA, $FRB", IIC_FPAddSub,
                        [(set f64:$FRT, (any_fadd f64:$FRA, f64:$FRB))]>;
  defm FADDS : AForm_2r<59, 21,
                        (outs f4rc:$FRT), (ins f4rc:$FRA, f4rc:$FRB),
                        "fadds", "$FRT, $FRA, $FRB", IIC_FPGeneral,
                        [(set f32:$FRT, (any_fadd f32:$FRA, f32:$FRB))]>;
  } // isCommutable
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3299-3306

```tablegen
  defm FDIV  : AForm_2r<63, 18,
                        (outs f8rc:$FRT), (ins f8rc:$FRA, f8rc:$FRB),
                        "fdiv", "$FRT, $FRA, $FRB", IIC_FPDivD,
                        [(set f64:$FRT, (any_fdiv f64:$FRA, f64:$FRB))]>;
  defm FDIVS : AForm_2r<59, 18,
                        (outs f4rc:$FRT), (ins f4rc:$FRA, f4rc:$FRB),
                        "fdivs", "$FRT, $FRA, $FRB", IIC_FPDivS,
                        [(set f32:$FRT, (any_fdiv f32:$FRA, f32:$FRB))]>;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 3307-3316

```tablegen
  let isCommutable = 1 in {
  defm FMUL  : AForm_3r<63, 25,
                        (outs f8rc:$FRT), (ins f8rc:$FRA, f8rc:$FRC),
                        "fmul", "$FRT, $FRA, $FRC", IIC_FPFused,
                        [(set f64:$FRT, (any_fmul f64:$FRA, f64:$FRC))]>;
  defm FMULS : AForm_3r<59, 25,
                        (outs f4rc:$FRT), (ins f4rc:$FRA, f4rc:$FRC),
                        "fmuls", "$FRT, $FRA, $FRC", IIC_FPGeneral,
                        [(set f32:$FRT, (any_fmul f32:$FRA, f32:$FRC))]>;
  } // isCommutable
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 3317-3327

```tablegen
  defm FSUB  : AForm_2r<63, 20,
                        (outs f8rc:$FRT), (ins f8rc:$FRA, f8rc:$FRB),
                        "fsub", "$FRT, $FRA, $FRB", IIC_FPAddSub,
                        [(set f64:$FRT, (any_fsub f64:$FRA, f64:$FRB))]>;
  defm FSUBS : AForm_2r<59, 20,
                        (outs f4rc:$FRT), (ins f4rc:$FRA, f4rc:$FRB),
                        "fsubs", "$FRT, $FRA, $FRB", IIC_FPGeneral,
                        [(set f32:$FRT, (any_fsub f32:$FRA, f32:$FRB))]>;
  }
}
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 3328-3336

```tablegen
let hasSideEffects = 0 in {
let PPC970_Unit = 1 in {  // FXU Operations.
  let isSelect = 1 in
  def ISEL  : AForm_4<31, 15,
                     (outs gprc:$RT), (ins gprc_nor0:$RA, gprc:$RB, crbitrc:$COND),
                     "isel $RT, $RA, $RB, $COND", IIC_IntISEL,
                     []>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3337-3347

```tablegen
let PPC970_Unit = 1 in {  // FXU Operations.
// M-Form instructions.  rotate and mask instructions.
//
let isCommutable = 1 in {
// RLWIMI can be commuted if the rotate amount is zero.
defm RLWIMI : MForm_2r<20, (outs gprc:$RA),
                       (ins gprc:$RAi, gprc:$RS, u5imm:$SH, u5imm:$MB,
                       u5imm:$ME), "rlwimi", "$RA, $RS, $SH, $MB, $ME",
                       IIC_IntRotate, []>, PPC970_DGroup_Cracked,
                       RegConstraint<"$RAi = $RA">;
}
```
- **EN**: Adds declarative TableGen records such as `RLWIMI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RLWIMI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3348-3358

```tablegen
let BaseName = "rlwinm" in {
def RLWINM : MForm_2<21,
                     (outs gprc:$RA), (ins gprc:$RS, u5imm:$SH, u5imm:$MB, u5imm:$ME),
                     "rlwinm $RA, $RS, $SH, $MB, $ME", IIC_IntGeneral,
                     []>, RecFormRel;
let Defs = [CR0] in
def RLWINM_rec : MForm_2<21,
                      (outs gprc:$RA), (ins gprc:$RS, u5imm:$SH, u5imm:$MB, u5imm:$ME),
                      "rlwinm. $RA, $RS, $SH, $MB, $ME", IIC_IntGeneral,
                      []>, isRecordForm, RecFormRel, PPC970_DGroup_Cracked;
}
```
- **EN**: Adds declarative TableGen records such as `RLWINM`, `RLWINM_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RLWINM`, `RLWINM_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3359-3365

```tablegen
defm RLWNM  : MForm_1r<23, (outs gprc:$RA),
                       (ins gprc:$RS, gprc:$RB, u5imm:$MB, u5imm:$ME),
                       "rlwnm", "$RA, $RS, $RB, $MB, $ME", IIC_IntGeneral,
                       []>;
}
} // hasSideEffects = 0
```
- **EN**: Adds declarative TableGen records such as `RLWNM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RLWNM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3366-3374

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC Instruction Patterns
//

// Arbitrary immediate support.  Implement in terms of LIS/ORI.
def : Pat<(i32 imm:$imm),
          (ORI (LIS (HI16 imm:$imm)), (LO16 imm:$imm))>;

// Implement the 'not' operation with the NOR instruction.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PowerPC Instruction Patterns". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PowerPC Instruction Patterns”。 这些声明会进入生成式模式匹配逻辑。

### Lines 3375-3383

```tablegen
def i32not : OutPatFrag<(ops node:$in),
                        (NOR $in, $in)>;
def        : Pat<(not i32:$in),
                 (i32not $in)>;

// ADD an arbitrary immediate.
def : Pat<(add_like i32:$in, imm:$imm),
          (ADDIS (ADDI $in, (LO16 imm:$imm)), (HA16 imm:$imm))>;
// OR an arbitrary immediate.
```
- **EN**: Adds declarative TableGen records such as `i32not` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `i32not`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3384-3393

```tablegen
def : Pat<(or i32:$in, imm:$imm),
          (ORIS (ORI $in, (LO16 imm:$imm)), (HI16 imm:$imm))>;
// XOR an arbitrary immediate.
def : Pat<(xor i32:$in, imm:$imm),
          (XORIS (XORI $in, (LO16 imm:$imm)), (HI16 imm:$imm))>;
// SUBFIC
def : Pat<(sub imm32SExt16:$imm, i32:$in),
          (SUBFIC $in, imm:$imm)>;

// SHL/SRL
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "XOR an arbitrary immediate.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“XOR an arbitrary immediate.”。

### Lines 3394-3401

```tablegen
def : Pat<(shl i32:$in, (i32 imm:$imm)),
          (RLWINM $in, imm:$imm, 0, (SHL32 imm:$imm))>;
def : Pat<(srl i32:$in, (i32 imm:$imm)),
          (RLWINM $in, (SRL32 imm:$imm), imm:$imm, 31)>;

// ROTL
def : Pat<(rotl i32:$in, i32:$sh),
          (RLWNM $in, $sh, 0, 31)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3402-3409

```tablegen
def : Pat<(rotl i32:$in, (i32 imm:$imm)),
          (RLWINM $in, imm:$imm, 0, 31)>;

// RLWNM
def : Pat<(and (rotl i32:$in, i32:$sh), maskimm32:$imm),
          (RLWNM $in, $sh, (MB maskimm32:$imm), (ME maskimm32:$imm))>;

// Calls
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "RLWNM".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“RLWNM”。

### Lines 3410-3418

```tablegen
def : Pat<(PPCcall (i32 tglobaladdr:$dst)),
          (BL tglobaladdr:$dst)>;

def : Pat<(PPCcall (i32 texternalsym:$dst)),
          (BL texternalsym:$dst)>;

def : Pat<(PPCcall_rm (i32 tglobaladdr:$dst)),
          (BL_RM tglobaladdr:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3419-3425

```tablegen
def : Pat<(PPCcall_rm (i32 texternalsym:$dst)),
          (BL_RM texternalsym:$dst)>;

// Calls for AIX only
def : Pat<(PPCcall (i32 mcsym:$dst)),
          (BL mcsym:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3426-3432

```tablegen
def : Pat<(PPCcall_nop (i32 mcsym:$dst)),
          (BL_NOP mcsym:$dst)>;

def : Pat<(PPCcall_nop (i32 texternalsym:$dst)),
          (BL_NOP texternalsym:$dst)>;

let Predicates = [IsAIX] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3433-3439

```tablegen
  def : Pat<(PPCbl_load_toc (i32 texternalsym:$dst)),
            (BL_LWZinto_toc texternalsym:$dst)>;

  def : Pat<(PPCbl_load_toc_rm (i32 texternalsym:$dst)),
            (BL_LWZinto_toc_RM texternalsym:$dst)>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 3440-3449

```tablegen
def : Pat<(PPCcall_rm (i32 mcsym:$dst)),
          (BL_RM mcsym:$dst)>;

def : Pat<(PPCcall_nop_rm (i32 mcsym:$dst)),
          (BL_NOP_RM mcsym:$dst)>;

def : Pat<(PPCcall_nop_rm (i32 texternalsym:$dst)),
          (BL_NOP_RM texternalsym:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3450-3458

```tablegen
def : Pat<(PPCtc_return (i32 tglobaladdr:$dst),  imm:$imm),
          (TCRETURNdi tglobaladdr:$dst, imm:$imm)>;

def : Pat<(PPCtc_return (i32 texternalsym:$dst), imm:$imm),
          (TCRETURNdi texternalsym:$dst, imm:$imm)>;

def : Pat<(PPCtc_return CTRRC:$dst, imm:$imm),
          (TCRETURNri CTRRC:$dst, imm:$imm)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3459-3465

```tablegen
def : Pat<(int_ppc_fence), (FENCE)>;
def : Pat<(int_ppc_readflm), (MFFS)>;
def : Pat<(int_ppc_mffsl), (MFFSL)>;

// Hi and Lo for Darwin Global Addresses.
def : Pat<(PPChi tglobaladdr:$in, 0), (LIS tglobaladdr:$in)>;
def : Pat<(PPClo tglobaladdr:$in, 0), (LI tglobaladdr:$in)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3466-3473

```tablegen
def : Pat<(PPChi tconstpool:$in, 0), (LIS tconstpool:$in)>;
def : Pat<(PPClo tconstpool:$in, 0), (LI tconstpool:$in)>;
def : Pat<(PPChi tjumptable:$in, 0), (LIS tjumptable:$in)>;
def : Pat<(PPClo tjumptable:$in, 0), (LI tjumptable:$in)>;
def : Pat<(PPChi tblockaddress:$in, 0), (LIS tblockaddress:$in)>;
def : Pat<(PPClo tblockaddress:$in, 0), (LI tblockaddress:$in)>;
def : Pat<(PPChi tglobaltlsaddr:$g, i32:$in),
          (ADDIS $in, tglobaltlsaddr:$g)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3474-3481

```tablegen
def : Pat<(PPClo tglobaltlsaddr:$g, i32:$in),
          (ADDI $in, tglobaltlsaddr:$g)>;
def : Pat<(add i32:$in, (PPChi tglobaladdr:$g, 0)),
          (ADDIS $in, tglobaladdr:$g)>;
def : Pat<(add i32:$in, (PPChi tconstpool:$g, 0)),
          (ADDIS $in, tconstpool:$g)>;
def : Pat<(add i32:$in, (PPChi tjumptable:$g, 0)),
          (ADDIS $in, tjumptable:$g)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3482-3493

```tablegen
def : Pat<(add i32:$in, (PPChi tblockaddress:$g, 0)),
          (ADDIS $in, tblockaddress:$g)>;

// Support for thread-local storage.
def PPC32GOT: PPCEmitTimePseudo<(outs gprc:$rD), (ins), "#PPC32GOT",
                [(set i32:$rD, (PPCppc32GOT))]> {
  let Size = 8;
}

// Get the _GLOBAL_OFFSET_TABLE_ in PIC mode.
// This uses two output registers, the first as the real output, the second as a
// temporary register, used internally in code generation. A "bl" also clobbers LR.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Support for thread-local storage.". Notable symbols in this range include `PPC32GOT`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Support for thread-local storage.”。 该区间中较显眼的符号包括 `PPC32GOT`。

### Lines 3494-3503

```tablegen
let Defs = [LR] in
def PPC32PICGOT: PPCEmitTimePseudo<(outs gprc:$rD, gprc:$rT), (ins), "#PPC32PICGOT",
                []> {
  let Size = 20;
}

def LDgotTprelL32: PPCEmitTimePseudo<(outs gprc_nor0:$rD), (ins s16imm:$disp, gprc_nor0:$reg),
                           "#LDgotTprelL32",
                           [(set i32:$rD,
                             (PPCldGotTprelL tglobaltlsaddr:$disp, i32:$reg))]>;
```
- **EN**: Adds declarative TableGen records such as `PPC32PICGOT`, `LDgotTprelL32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPC32PICGOT`, `LDgotTprelL32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3504-3512

```tablegen
def : Pat<(PPCaddTls i32:$in, tglobaltlsaddr:$g),
          (ADD4TLS $in, tglobaltlsaddr:$g)>;

def ADDItlsgdL32 : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc_nor0:$reg, s16imm:$disp),
                         "#ADDItlsgdL32",
                         [(set i32:$rD,
                           (PPCaddiTlsgdL i32:$reg, tglobaltlsaddr:$disp))]>;
// LR is a true define, while the rest of the Defs are clobbers.  R3 is
// explicitly defined when this op is created, so not mentioned here.
```
- **EN**: Adds declarative TableGen records such as `ADDItlsgdL32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDItlsgdL32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3513-3521

```tablegen
let hasExtraSrcRegAllocReq = 1, hasExtraDefRegAllocReq = 1,
    Defs = [R0,R4,R5,R6,R7,R8,R9,R10,R11,R12,LR,CTR,CR0,CR1,CR5,CR6,CR7] in
def GETtlsADDR32 : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc:$reg, tlsgd32:$sym),
                          "GETtlsADDR32",
                          [(set i32:$rD,
                            (PPCgetTlsAddr i32:$reg, tglobaltlsaddr:$sym))]>;
// R3 is explicitly defined when this op is created, so not mentioned here.
// The rest of the Defs are the exact set of registers that will be clobbered by
// the call.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "R3 is explicitly defined when this op is created, so not mentioned here.". Notable symbols in this range include `GETtlsADDR32`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“R3 is explicitly defined when this op is created, so not mentioned here.”。 该区间中较显眼的符号包括 `GETtlsADDR32`。

### Lines 3522-3536

```tablegen
let hasExtraSrcRegAllocReq = 1, hasExtraDefRegAllocReq = 1,
    Defs = [R0,R4,R5,R11,LR,CR0] in {
def GETtlsADDR32AIX : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc:$offset, gprc:$handle),
                          "GETtlsADDR32AIX",
                          [(set i32:$rD,
                            (PPCgetTlsAddr i32:$offset, i32:$handle))]>;
def GETtlsMOD32AIX : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc:$handle),
                          "GETtlsMOD32AIX",
                          [(set i32:$rD,
                            (PPCgetTlsMod i32:$handle))]>;
}

// For local-exec accesses on 32-bit AIX, a call to .__get_tpointer is
// generated to retrieve the thread pointer. GETtlsTpointer32AIX clobbers both
// R3 and the LR (link register).
```
- **EN**: Adds declarative TableGen records such as `GETtlsADDR32AIX`, `GETtlsMOD32AIX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `GETtlsADDR32AIX`, `GETtlsMOD32AIX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3537-3547

```tablegen
let hasExtraSrcRegAllocReq = 1, hasExtraDefRegAllocReq = 1,
    Defs = [R3,LR] in
def GETtlsTpointer32AIX : PPCEmitTimePseudo<(outs gprc:$rD), (ins),
                          "GETtlsTpointer32AIX",
                          [(set i32:$rD, (PPCgetTpointer))]>;

// The following pattern matches local- and initial-exec TLS accesses on 32-bit AIX.
// PPCaddTls is used in local- and initial-exec accesses in order to:
//   - Get the address of a variable (add the variable offset to the thread
//     pointer, retrieved by calling .__get_tpointer).
//   - Create an opportunity to optimize the user of the loaded address.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The following pattern matches local- and initial-exec TLS accesses on 32-bit AIX.". Notable symbols in this range include `GETtlsTpointer32AIX`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The following pattern matches local- and initial-exec TLS accesses on 32-bit AIX.”。 该区间中较显眼的符号包括 `GETtlsTpointer32AIX`。

### Lines 3548-3554

```tablegen
def : Pat<(PPCaddTls i32:$in, i32:$addr),
          (ADD4TLS $in, $addr)>;

// Combined op for ADDItlsgdL32 and GETtlsADDR32, late expanded.  R3 and LR
// are true defines while the rest of the Defs are clobbers.
let hasExtraSrcRegAllocReq = 1, hasExtraDefRegAllocReq = 1,
    Defs = [R0,R3,R4,R5,R6,R7,R8,R9,R10,R11,R12,LR,CTR,CR0,CR1,CR5,CR6,CR7] in
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Combined op for ADDItlsgdL32 and GETtlsADDR32, late expanded.  R3 and LR".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Combined op for ADDItlsgdL32 and GETtlsADDR32, late expanded.  R3 and LR”。

### Lines 3555-3561

```tablegen
def ADDItlsgdLADDR32 : PPCEmitTimePseudo<(outs gprc:$rD),
                              (ins gprc_nor0:$reg, s16imm:$disp, tlsgd32:$sym),
                              "#ADDItlsgdLADDR32",
                              [(set i32:$rD,
                                (PPCaddiTlsgdLAddr i32:$reg,
                                                   tglobaltlsaddr:$disp,
                                                   tglobaltlsaddr:$sym))]>;
```
- **EN**: Adds declarative TableGen records such as `ADDItlsgdLADDR32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDItlsgdLADDR32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3562-3572

```tablegen
def ADDItlsldL32 : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc_nor0:$reg, s16imm:$disp),
                          "#ADDItlsldL32",
                          [(set i32:$rD,
                            (PPCaddiTlsldL i32:$reg, tglobaltlsaddr:$disp))]>;
// This pseudo is expanded to two copies to put the variable offset in R4 and
// the region handle in R3 and GETtlsADDR32AIX.
def TLSGDAIX : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc:$offset, gprc:$handle),
                          "#TLSGDAIX",
                          [(set i32:$rD,
                            (PPCTlsgdAIX i32:$offset, i32:$handle))]>;
// This pseudo is expanded to the call to GETtlsMOD32AIX.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This pseudo is expanded to two copies to put the variable offset in R4 and". Notable symbols in this range include `ADDItlsldL32`, `TLSGDAIX`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This pseudo is expanded to two copies to put the variable offset in R4 and”。 该区间中较显眼的符号包括 `ADDItlsldL32`, `TLSGDAIX`。

### Lines 3573-3585

```tablegen
def TLSLDAIX : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc:$handle),
                          "#TLSLDAIX", [(set i32:$rD, (PPCTlsldAIX i32:$handle))]>;
// LR is a true define, while the rest of the Defs are clobbers.  R3 is
// explicitly defined when this op is created, so not mentioned here.
let hasExtraSrcRegAllocReq = 1, hasExtraDefRegAllocReq = 1,
    Defs = [R0,R4,R5,R6,R7,R8,R9,R10,R11,R12,LR,CTR,CR0,CR1,CR5,CR6,CR7] in
def GETtlsldADDR32 : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc:$reg, tlsgd32:$sym),
                            "GETtlsldADDR32",
                            [(set i32:$rD,
                              (PPCgetTlsldAddr i32:$reg,
                                               tglobaltlsaddr:$sym))]>;
// Combined op for ADDItlsldL32 and GETtlsADDR32, late expanded.  R3 and LR
// are true defines while the rest of the Defs are clobbers.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "LR is a true define, while the rest of the Defs are clobbers.  R3 is". Notable symbols in this range include `TLSLDAIX`, `GETtlsldADDR32`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“LR is a true define, while the rest of the Defs are clobbers.  R3 is”。 该区间中较显眼的符号包括 `TLSLDAIX`, `GETtlsldADDR32`。

### Lines 3586-3594

```tablegen
let hasExtraSrcRegAllocReq = 1, hasExtraDefRegAllocReq = 1,
    Defs = [R0,R3,R4,R5,R6,R7,R8,R9,R10,R11,R12,LR,CTR,CR0,CR1,CR5,CR6,CR7] in
def ADDItlsldLADDR32 : PPCEmitTimePseudo<(outs gprc:$rD),
                              (ins gprc_nor0:$reg, s16imm:$disp, tlsgd32:$sym),
                              "#ADDItlsldLADDR32",
                              [(set i32:$rD,
                                (PPCaddiTlsldLAddr i32:$reg,
                                                   tglobaltlsaddr:$disp,
                                                   tglobaltlsaddr:$sym))]>;
```
- **EN**: Adds declarative TableGen records such as `ADDItlsldLADDR32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDItlsldLADDR32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3595-3605

```tablegen
def ADDIdtprelL32 : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc_nor0:$reg, s16imm:$disp),
                           "#ADDIdtprelL32",
                           [(set i32:$rD,
                             (PPCaddiDtprelL i32:$reg, tglobaltlsaddr:$disp))]>;
def ADDISdtprelHA32 : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc_nor0:$reg, s16imm:$disp),
                            "#ADDISdtprelHA32",
                            [(set i32:$rD,
                              (PPCaddisDtprelHA i32:$reg,
                                                tglobaltlsaddr:$disp))]>;

// Support for Position-independent code
```
- **EN**: Adds declarative TableGen records such as `ADDIdtprelL32`, `ADDISdtprelHA32` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ADDIdtprelL32`, `ADDISdtprelHA32`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3606-3613

```tablegen
def LWZtoc : PPCEmitTimePseudo<(outs gprc:$rD), (ins tocentry32:$disp, gprc:$reg),
                   "#LWZtoc",
                   [(set i32:$rD,
                     (PPCtoc_entry tglobaladdr:$disp, i32:$reg))]>;
def LWZtocL : PPCEmitTimePseudo<(outs gprc:$rD), (ins tocentry32:$disp, gprc_nor0:$reg),
                    "#LWZtocL",
                    [(set i32:$rD,
                      (PPCtoc_entry tglobaladdr:$disp, i32:$reg))]>;
```
- **EN**: Adds declarative TableGen records such as `LWZtoc`, `LWZtocL` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LWZtoc`, `LWZtocL`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3614-3623

```tablegen
def ADDIStocHA : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc_nor0:$reg, tocentry32:$disp),
                       "#ADDIStocHA", []>;
// TOC Data Transform on AIX
def ADDItoc : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc:$reg, tocentry32:$disp),
                   "#ADDItoc", []>;
def ADDItocL : PPCEmitTimePseudo<(outs gprc:$rD), (ins gprc_nor0:$reg, tocentry32:$disp),
                   "#ADDItocL", []>;

// Get Global (GOT) Base Register offset, from the word immediately preceding
// the function label.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "TOC Data Transform on AIX". Notable symbols in this range include `ADDIStocHA`, `ADDItoc`, `ADDItocL`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“TOC Data Transform on AIX”。 该区间中较显眼的符号包括 `ADDIStocHA`, `ADDItoc`, `ADDItocL`。

### Lines 3624-3631

```tablegen
def UpdateGBR : PPCEmitTimePseudo<(outs gprc:$rD, gprc:$rT), (ins gprc:$rI), "#UpdateGBR", []> {
  let Size = 8;
}

// Pseudo-instruction marked for deletion. When deleting the instruction would
// cause iterator invalidation in MIR transformation passes, this pseudo can be
// used instead. It will be removed unconditionally at pre-emit time (prior to
// branch selection).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pseudo-instruction marked for deletion. When deleting the instruction would". Notable symbols in this range include `UpdateGBR`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pseudo-instruction marked for deletion. When deleting the instruction would”。 该区间中较显眼的符号包括 `UpdateGBR`。

### Lines 3632-3638

```tablegen
def UNENCODED_NOP: PPCEmitTimePseudo<(outs), (ins), "#UNENCODED_NOP", []>;

// Standard shifts.  These are represented separately from the real shifts above
// so that we can distinguish between shifts that allow 5-bit and 6-bit shift
// amounts.
def : Pat<(sra i32:$rS, i32:$rB),
          (SRAW $rS, $rB)>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Standard shifts.  These are represented separately from the real shifts above". Notable symbols in this range include `UNENCODED_NOP`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Standard shifts.  These are represented separately from the real shifts above”。 该区间中较显眼的符号包括 `UNENCODED_NOP`。

### Lines 3639-3645

```tablegen
def : Pat<(srl i32:$rS, i32:$rB),
          (SRW $rS, $rB)>;
def : Pat<(shl i32:$rS, i32:$rB),
          (SLW $rS, $rB)>;

def : Pat<(i32 (zextloadi1 DForm:$src)),
          (LBZ DForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3646-3653

```tablegen
def : Pat<(i32 (zextloadi1 XForm:$src)),
          (LBZX XForm:$src)>;
def : Pat<(i32 (extloadi1 DForm:$src)),
          (LBZ DForm:$src)>;
def : Pat<(i32 (extloadi1 XForm:$src)),
          (LBZX XForm:$src)>;
def : Pat<(i32 (extloadi8 DForm:$src)),
          (LBZ DForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3654-3660

```tablegen
def : Pat<(i32 (extloadi8 XForm:$src)),
          (LBZX XForm:$src)>;
def : Pat<(i32 (extloadi16 DForm:$src)),
          (LHZ DForm:$src)>;
def : Pat<(i32 (extloadi16 XForm:$src)),
          (LHZX XForm:$src)>;
let Predicates = [HasFPU] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3661-3674

```tablegen
def : Pat<(f64 (extloadf32 DForm:$src)),
          (COPY_TO_REGCLASS (LFS DForm:$src), F8RC)>;
def : Pat<(f64 (extloadf32 XForm:$src)),
          (COPY_TO_REGCLASS (LFSX XForm:$src), F8RC)>;

def : Pat<(f64 (any_fpextend f32:$src)),
          (COPY_TO_REGCLASS $src, F8RC)>;
}

// Only seq_cst fences require the heavyweight sync (SYNC 0).
// All others can use the lightweight sync (SYNC 1).
// source: http://www.cl.cam.ac.uk/~pes20/cpp/cpp0xmappings.html
// The rule for seq_cst is duplicated to work with both 64 bits and 32 bits
// versions of Power.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Only seq_cst fences require the heavyweight sync (SYNC 0).".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Only seq_cst fences require the heavyweight sync (SYNC 0).”。

### Lines 3675-3681

```tablegen
def : Pat<(atomic_fence (i64 7), (timm)), (SYNC 0)>, Requires<[HasSYNC]>;
def : Pat<(atomic_fence (i32 7), (timm)), (SYNC 0)>, Requires<[HasSYNC]>;
def : Pat<(atomic_fence (timm), (timm)), (SYNC 1)>, Requires<[HasSYNC]>;
def : Pat<(atomic_fence (timm), (timm)), (MSYNC)>, Requires<[HasOnlyMSYNC]>;

let Predicates = [HasFPU] in {
// Additional fnmsub patterns for custom node
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3682-3689

```tablegen
def : Pat<(PPCfnmsub f64:$A, f64:$B, f64:$C),
          (FNMSUB $A, $B, $C)>;
def : Pat<(PPCfnmsub f32:$A, f32:$B, f32:$C),
          (FNMSUBS $A, $B, $C)>;
def : Pat<(fneg (PPCfnmsub f64:$A, f64:$B, f64:$C)),
          (FMSUB $A, $B, $C)>;
def : Pat<(fneg (PPCfnmsub f32:$A, f32:$B, f32:$C)),
          (FMSUBS $A, $B, $C)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3690-3697

```tablegen
def : Pat<(PPCfnmsub f64:$A, f64:$B, (fneg f64:$C)),
          (FNMADD $A, $B, $C)>;
def : Pat<(PPCfnmsub f32:$A, f32:$B, (fneg f32:$C)),
          (FNMADDS $A, $B, $C)>;

// FCOPYSIGN's operand types need not agree.
def : Pat<(fcopysign f64:$frB, f32:$frA),
          (FCPSGND (COPY_TO_REGCLASS $frA, F8RC), $frB)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3698-3704

```tablegen
def : Pat<(fcopysign f32:$frB, f64:$frA),
          (FCPSGNS (COPY_TO_REGCLASS $frA, F4RC), $frB)>;
}

// XL Compat intrinsics.
def : Pat<(int_ppc_fmsub f64:$A, f64:$B, f64:$C), (FMSUB $A, $B, $C)>;
def : Pat<(int_ppc_fmsubs f32:$A, f32:$B, f32:$C), (FMSUBS $A, $B, $C)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3705-3711

```tablegen
def : Pat<(int_ppc_fnmadd f64:$A, f64:$B, f64:$C), (FNMADD $A, $B, $C)>;
def : Pat<(int_ppc_fnmadds f32:$A, f32:$B, f32:$C), (FNMADDS $A, $B, $C)>;
def : Pat<(int_ppc_fre f64:$A), (FRE $A)>;
def : Pat<(int_ppc_fres f32:$A), (FRES $A)>;
def : Pat<(int_ppc_fnabs f64:$A), (FNABSD $A)>;
def : Pat<(int_ppc_fnabss f32:$A), (FNABSS $A)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3712-3719

```tablegen
include "PPCInstrAltivec.td"
include "PPCInstrSPE.td"
include "PPCInstr64Bit.td"
include "PPCInstrVSX.td"
include "PPCInstrHTM.td"

def crnot : OutPatFrag<(ops node:$in),
                       (CRNOT $in)>;
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers. These declarations feed generated pattern-matching logic.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这些声明会进入生成式模式匹配逻辑。

### Lines 3720-3727

```tablegen
def       : Pat<(not i1:$in),
                (crnot $in)>;

// Pseudo-instructions for alternate assembly syntax (never used by codegen).
// These are aliases that require C++ handling to convert to the target
// instruction, while InstAliases can be handled directly by tblgen.
class PPCAsmPseudo<string asm, dag iops>
  : Instruction {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pseudo-instructions for alternate assembly syntax (never used by codegen).". Notable symbols in this range include `PPCAsmPseudo`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pseudo-instructions for alternate assembly syntax (never used by codegen).”。 该区间中较显眼的符号包括 `PPCAsmPseudo`。

### Lines 3728-3734

```tablegen
  let Namespace = "PPC";
  bit PPC64 = 0;  // Default value, override with isPPC64

  let OutOperandList = (outs);
  let InOperandList = iops;
  let Pattern = [];
  let AsmString = asm;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 3735-3744

```tablegen
  let isAsmParserOnly = 1;
  let isPseudo = 1;
  let hasNoSchedulingInfo = 1;

  // Indicate that this instruction takes a register+immediate memory operand.
  bits<1> MemriOp = 0;
  let TSFlags{10} = MemriOp;
}

// Mask immediates for MMA instructions (2, 4 and 8 bits).
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers. Scheduling or processor-model metadata is part of this section.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。 这一段包含调度或处理器模型元数据。

### Lines 3745-3751

```tablegen
def Msk2Imm : ImmLeaf<i32, [{ return isUInt<2>(Imm); }]>;
def Msk4Imm : ImmLeaf<i32, [{ return isUInt<4>(Imm); }]>;
def Msk8Imm : ImmLeaf<i32, [{ return isUInt<8>(Imm); }]>;

// Prefixed instructions may require access to the above defs at a later
// time so we include this after the def.
include "PPCInstrP10.td"
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers. Notable symbols in this range include `Msk2Imm`, `Msk4Imm`, `Msk8Imm`.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间中较显眼的符号包括 `Msk2Imm`, `Msk4Imm`, `Msk8Imm`。

### Lines 3752-3759

```tablegen
include "PPCInstrFutureMMA.td"
include "PPCInstrFuture.td"
include "PPCInstrMMA.td"
include "PPCInstrDFP.td"

// Patterns for arithmetic i1 operations.
def : Pat<(add i1:$a, i1:$b),
          (CRXOR $a, $b)>;
```
- **EN**: Pulls in direct dependencies required by this instruction information and lowering helpers, so later declarations can reuse LLVM infrastructure and target-specific helpers. These declarations feed generated pattern-matching logic.
- **CN**: 这一段引入该指令信息与 lowering 辅助逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这些声明会进入生成式模式匹配逻辑。

### Lines 3760-3766

```tablegen
def : Pat<(sub i1:$a, i1:$b),
          (CRXOR $a, $b)>;
def : Pat<(mul i1:$a, i1:$b),
          (CRAND $a, $b)>;

// We're sometimes asked to materialize i1 -1, which is just 1 in this case
// (-1 is used to mean all bits set).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "We're sometimes asked to materialize i1 -1, which is just 1 in this case".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“We're sometimes asked to materialize i1 -1, which is just 1 in this case”。

### Lines 3767-3774

```tablegen
def : Pat<(i1 -1), (CRSET)>;

// i1 extensions, implemented in terms of isel.
def : Pat<(i32 (zext i1:$in)),
          (SELECT_I4 $in, (LI 1), (LI 0))>;
def : Pat<(i32 (sext i1:$in)),
          (SELECT_I4 $in, (LI -1), (LI 0))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3775-3781

```tablegen
def : Pat<(i64 (zext i1:$in)),
          (SELECT_I8 $in, (LI8 1), (LI8 0))>;
def : Pat<(i64 (sext i1:$in)),
          (SELECT_I8 $in, (LI8 -1), (LI8 0))>;

// FIXME: We should choose either a zext or a sext based on other constants
// already around.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FIXME: We should choose either a zext or a sext based on other constants".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FIXME: We should choose either a zext or a sext based on other constants”。

### Lines 3782-3804

```tablegen
def : Pat<(i32 (anyext i1:$in)),
          (SELECT_I4 $in, (LI 1), (LI 0))>;
def : Pat<(i64 (anyext i1:$in)),
          (SELECT_I8 $in, (LI8 1), (LI8 0))>;

// match setcc on i1 variables.
// CRANDC is:
//   1 1 : F
//   1 0 : T
//   0 1 : F
//   0 0 : F
//
// LT is:
//  -1 -1  : F
//  -1  0  : T
//   0 -1  : F
//   0  0  : F
//
// ULT is:
//   1 1 : F
//   1 0 : F
//   0 1 : T
//   0 0 : F
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "match setcc on i1 variables.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“match setcc on i1 variables.”。

### Lines 3805-3825

```tablegen
def : Pat<(i1 (setcc i1:$s1, i1:$s2, SETLT)),
          (CRANDC $s1, $s2)>;
def : Pat<(i1 (setcc i1:$s1, i1:$s2, SETULT)),
          (CRANDC $s2, $s1)>;
// CRORC is:
//   1 1 : T
//   1 0 : T
//   0 1 : F
//   0 0 : T
//
// LE is:
//  -1 -1 : T
//  -1  0 : T
//   0 -1 : F
//   0  0 : T
//
// ULE is:
//   1 1 : T
//   1 0 : F
//   0 1 : T
//   0 0 : T
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "CRORC is:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“CRORC is:”。

### Lines 3826-3844

```tablegen
def : Pat<(i1 (setcc i1:$s1, i1:$s2, SETLE)),
          (CRORC $s1, $s2)>;
def : Pat<(i1 (setcc i1:$s1, i1:$s2, SETULE)),
          (CRORC $s2, $s1)>;

def : Pat<(i1 (setcc i1:$s1, i1:$s2, SETEQ)),
          (CREQV $s1, $s2)>;

// GE is:
//  -1 -1 : T
//  -1  0 : F
//   0 -1 : T
//   0  0 : T
//
// UGE is:
//   1 1 : T
//   1 0 : T
//   0 1 : F
//   0 0 : T
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "GE is:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“GE is:”。

### Lines 3845-3860

```tablegen
def : Pat<(i1 (setcc i1:$s1, i1:$s2, SETGE)),
          (CRORC $s2, $s1)>;
def : Pat<(i1 (setcc i1:$s1, i1:$s2, SETUGE)),
          (CRORC $s1, $s2)>;

// GT is:
//  -1 -1 : F
//  -1  0 : F
//   0 -1 : T
//   0  0 : F
//
// UGT is:
//  1 1 : F
//  1 0 : T
//  0 1 : F
//  0 0 : F
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "GT is:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“GT is:”。

### Lines 3861-3872

```tablegen
def : Pat<(i1 (setcc i1:$s1, i1:$s2, SETGT)),
          (CRANDC $s2, $s1)>;
def : Pat<(i1 (setcc i1:$s1, i1:$s2, SETUGT)),
          (CRANDC $s1, $s2)>;

def : Pat<(i1 (setcc i1:$s1, i1:$s2, SETNE)),
          (CRXOR $s1, $s2)>;

// match setcc on non-i1 (non-vector) variables. Note that SETUEQ, SETOGE,
// SETOLE, SETONE, SETULT and SETUGT should be expanded by legalize for
// floating-point types.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3873-3879

```tablegen
multiclass CRNotPat<dag pattern, dag result> {
  def : Pat<pattern, (crnot result)>;
  def : Pat<(not pattern), result>;

  // We can also fold the crnot into an extension:
  def : Pat<(i32 (zext pattern)),
            (SELECT_I4 result, (LI 0), (LI 1))>;
```
- **EN**: Adds declarative TableGen records such as `CRNotPat` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CRNotPat`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3880-3890

```tablegen
  def : Pat<(i32 (sext pattern)),
            (SELECT_I4 result, (LI 0), (LI -1))>;

  // We can also fold the crnot into an extension:
  def : Pat<(i64 (zext pattern)),
            (SELECT_I8 result, (LI8 0), (LI8 1))>;
  def : Pat<(i64 (sext pattern)),
            (SELECT_I8 result, (LI8 0), (LI8 -1))>;

  // FIXME: We should choose either a zext or a sext based on other constants
  // already around.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "We can also fold the crnot into an extension:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“We can also fold the crnot into an extension:”。

### Lines 3891-3903

```tablegen
  def : Pat<(i32 (anyext pattern)),
            (SELECT_I4 result, (LI 0), (LI 1))>;

  def : Pat<(i64 (anyext pattern)),
            (SELECT_I8 result, (LI8 0), (LI8 1))>;
}

// FIXME: Because of what seems like a bug in TableGen's type-inference code,
// we need to write imm:$imm in the output patterns below, not just $imm, or
// else the resulting matcher will not correctly add the immediate operand
// (making it a register operand instead).

// extended SETCC.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FIXME: Because of what seems like a bug in TableGen's type-inference code,".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FIXME: Because of what seems like a bug in TableGen's type-inference code,”。

### Lines 3904-3911

```tablegen
multiclass ExtSetCCPat<CondCode cc, PatFrag pfrag,
                       OutPatFrag rfrag, OutPatFrag rfrag8> {
  def : Pat<(i32 (zext (i1 (pfrag i32:$s1, cc)))),
            (rfrag $s1)>;
  def : Pat<(i64 (zext (i1 (pfrag i64:$s1, cc)))),
            (rfrag8 $s1)>;
  def : Pat<(i64 (zext (i1 (pfrag i32:$s1, cc)))),
            (INSERT_SUBREG (i64 (IMPLICIT_DEF)), (rfrag $s1), sub_32)>;
```
- **EN**: Adds declarative TableGen records such as `ExtSetCCPat` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ExtSetCCPat`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3912-3918

```tablegen
  def : Pat<(i32 (zext (i1 (pfrag i64:$s1, cc)))),
            (EXTRACT_SUBREG (rfrag8 $s1), sub_32)>;

  def : Pat<(i32 (anyext (i1 (pfrag i32:$s1, cc)))),
            (rfrag $s1)>;
  def : Pat<(i64 (anyext (i1 (pfrag i64:$s1, cc)))),
            (rfrag8 $s1)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 3919-3928

```tablegen
  def : Pat<(i64 (anyext (i1 (pfrag i32:$s1, cc)))),
            (INSERT_SUBREG (i64 (IMPLICIT_DEF)), (rfrag $s1), sub_32)>;
  def : Pat<(i32 (anyext (i1 (pfrag i64:$s1, cc)))),
            (EXTRACT_SUBREG (rfrag8 $s1), sub_32)>;
}

// Note that we do all inversions below with i(32|64)not, instead of using
// (xori x, 1) because on the A2 nor has single-cycle latency while xori
// has 2-cycle latency.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Note that we do all inversions below with i(32|64)not, instead of using".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Note that we do all inversions below with i(32|64)not, instead of using”。

### Lines 3929-3936

```tablegen
defm : ExtSetCCPat<SETEQ,
                   PatFrag<(ops node:$in, node:$cc),
                           (setcc $in, 0, $cc)>,
                   OutPatFrag<(ops node:$in),
                              (RLWINM (CNTLZW $in), 27, 31, 31)>,
                   OutPatFrag<(ops node:$in),
                              (RLDICL (CNTLZD $in), 58, 63)> >;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3937-3944

```tablegen
defm : ExtSetCCPat<SETNE,
                   PatFrag<(ops node:$in, node:$cc),
                           (setcc $in, 0, $cc)>,
                   OutPatFrag<(ops node:$in),
                              (RLWINM (i32not (CNTLZW $in)), 27, 31, 31)>,
                   OutPatFrag<(ops node:$in),
                              (RLDICL (i64not (CNTLZD $in)), 58, 63)> >;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3945-3952

```tablegen
defm : ExtSetCCPat<SETLT,
                   PatFrag<(ops node:$in, node:$cc),
                           (setcc $in, 0, $cc)>,
                   OutPatFrag<(ops node:$in),
                              (RLWINM $in, 1, 31, 31)>,
                   OutPatFrag<(ops node:$in),
                              (RLDICL $in, 1, 63)> >;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3953-3960

```tablegen
defm : ExtSetCCPat<SETGE,
                   PatFrag<(ops node:$in, node:$cc),
                           (setcc $in, 0, $cc)>,
                   OutPatFrag<(ops node:$in),
                              (RLWINM (i32not $in), 1, 31, 31)>,
                   OutPatFrag<(ops node:$in),
                              (RLDICL (i64not $in), 1, 63)> >;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3961-3968

```tablegen
defm : ExtSetCCPat<SETGT,
                   PatFrag<(ops node:$in, node:$cc),
                           (setcc $in, 0, $cc)>,
                   OutPatFrag<(ops node:$in),
                              (RLWINM (ANDC (NEG $in), $in), 1, 31, 31)>,
                   OutPatFrag<(ops node:$in),
                              (RLDICL (ANDC8 (NEG8 $in), $in), 1, 63)> >;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3969-3976

```tablegen
defm : ExtSetCCPat<SETLE,
                   PatFrag<(ops node:$in, node:$cc),
                           (setcc $in, 0, $cc)>,
                   OutPatFrag<(ops node:$in),
                              (RLWINM (ORC $in, (NEG $in)), 1, 31, 31)>,
                   OutPatFrag<(ops node:$in),
                              (RLDICL (ORC8 $in, (NEG8 $in)), 1, 63)> >;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3977-3984

```tablegen
defm : ExtSetCCPat<SETLT,
                   PatFrag<(ops node:$in, node:$cc),
                           (setcc $in, -1, $cc)>,
                   OutPatFrag<(ops node:$in),
                              (RLWINM (AND $in, (ADDI $in, 1)), 1, 31, 31)>,
                   OutPatFrag<(ops node:$in),
                              (RLDICL (AND8 $in, (ADDI8 $in, 1)), 1, 63)> >;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3985-3992

```tablegen
defm : ExtSetCCPat<SETGE,
                   PatFrag<(ops node:$in, node:$cc),
                           (setcc $in, -1, $cc)>,
                   OutPatFrag<(ops node:$in),
                              (RLWINM (NAND $in, (ADDI $in, 1)), 1, 31, 31)>,
                   OutPatFrag<(ops node:$in),
                              (RLDICL (NAND8 $in, (ADDI8 $in, 1)), 1, 63)> >;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3993-4000

```tablegen
defm : ExtSetCCPat<SETGT,
                   PatFrag<(ops node:$in, node:$cc),
                           (setcc $in, -1, $cc)>,
                   OutPatFrag<(ops node:$in),
                              (RLWINM (i32not $in), 1, 31, 31)>,
                   OutPatFrag<(ops node:$in),
                              (RLDICL (i64not $in), 1, 63)> >;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4001-4009

```tablegen
defm : ExtSetCCPat<SETLE,
                   PatFrag<(ops node:$in, node:$cc),
                           (setcc $in, -1, $cc)>,
                   OutPatFrag<(ops node:$in),
                              (RLWINM $in, 1, 31, 31)>,
                   OutPatFrag<(ops node:$in),
                              (RLDICL $in, 1, 63)> >;

// An extended SETCC with shift amount.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4010-4017

```tablegen
multiclass ExtSetCCShiftPat<CondCode cc, PatFrag pfrag,
                            OutPatFrag rfrag, OutPatFrag rfrag8> {
  def : Pat<(i32 (zext (i1 (pfrag i32:$s1, i32:$sa, cc)))),
            (rfrag $s1, $sa)>;
  def : Pat<(i64 (zext (i1 (pfrag i64:$s1, i32:$sa, cc)))),
            (rfrag8 $s1, $sa)>;
  def : Pat<(i64 (zext (i1 (pfrag i32:$s1, i32:$sa, cc)))),
            (INSERT_SUBREG (i64 (IMPLICIT_DEF)), (rfrag $s1, $sa), sub_32)>;
```
- **EN**: Adds declarative TableGen records such as `ExtSetCCShiftPat` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ExtSetCCShiftPat`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4018-4024

```tablegen
  def : Pat<(i32 (zext (i1 (pfrag i64:$s1, i32:$sa, cc)))),
            (EXTRACT_SUBREG (rfrag8 $s1, $sa), sub_32)>;

  def : Pat<(i32 (anyext (i1 (pfrag i32:$s1, i32:$sa, cc)))),
            (rfrag $s1, $sa)>;
  def : Pat<(i64 (anyext (i1 (pfrag i64:$s1, i32:$sa, cc)))),
            (rfrag8 $s1, $sa)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4025-4038

```tablegen
  def : Pat<(i64 (anyext (i1 (pfrag i32:$s1, i32:$sa, cc)))),
            (INSERT_SUBREG (i64 (IMPLICIT_DEF)), (rfrag $s1, $sa), sub_32)>;
  def : Pat<(i32 (anyext (i1 (pfrag i64:$s1, i32:$sa, cc)))),
            (EXTRACT_SUBREG (rfrag8 $s1, $sa), sub_32)>;
}

defm : ExtSetCCShiftPat<SETNE,
                        PatFrag<(ops node:$in, node:$sa, node:$cc),
                                (setcc (and $in, (shl 1, $sa)), 0, $cc)>,
                        OutPatFrag<(ops node:$in, node:$sa),
                                   (RLWNM $in, (SUBFIC $sa, 32), 31, 31)>,
                        OutPatFrag<(ops node:$in, node:$sa),
                                   (RLDCL $in, (SUBFIC $sa, 64), 63)> >;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4039-4049

```tablegen
defm : ExtSetCCShiftPat<SETEQ,
                        PatFrag<(ops node:$in, node:$sa, node:$cc),
                                (setcc (and $in, (shl 1, $sa)), 0, $cc)>,
                        OutPatFrag<(ops node:$in, node:$sa),
                                   (RLWNM (i32not $in),
                                          (SUBFIC $sa, 32), 31, 31)>,
                        OutPatFrag<(ops node:$in, node:$sa),
                                   (RLDCL (i64not $in),
                                          (SUBFIC $sa, 64), 63)> >;

// SETCC for i32.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4050-4057

```tablegen
def : Pat<(i1 (setcc i32:$s1, immZExt16:$imm, SETULT)),
          (EXTRACT_SUBREG (CMPLWI $s1, imm:$imm), sub_lt)>;
def : Pat<(i1 (setcc i32:$s1, imm32SExt16:$imm, SETLT)),
          (EXTRACT_SUBREG (CMPWI $s1, imm:$imm), sub_lt)>;
def : Pat<(i1 (setcc i32:$s1, immZExt16:$imm, SETUGT)),
          (EXTRACT_SUBREG (CMPLWI $s1, imm:$imm), sub_gt)>;
def : Pat<(i1 (setcc i32:$s1, imm32SExt16:$imm, SETGT)),
          (EXTRACT_SUBREG (CMPWI $s1, imm:$imm), sub_gt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4058-4073

```tablegen
def : Pat<(i1 (setcc i32:$s1, imm32SExt16:$imm, SETEQ)),
          (EXTRACT_SUBREG (CMPWI $s1, imm:$imm), sub_eq)>;
def : Pat<(i1 (setcc i32:$s1, immZExt16:$imm, SETEQ)),
          (EXTRACT_SUBREG (CMPLWI $s1, imm:$imm), sub_eq)>;

// For non-equality comparisons, the default code would materialize the
// constant, then compare against it, like this:
//   lis r2, 4660
//   ori r2, r2, 22136
//   cmpw cr0, r3, r2
//   beq cr0,L6
// Since we are just comparing for equality, we can emit this instead:
//   xoris r0,r3,0x1234
//   cmplwi cr0,r0,0x5678
//   beq cr0,L6
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "For non-equality comparisons, the default code would materialize the".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“For non-equality comparisons, the default code would materialize the”。

### Lines 4074-4081

```tablegen
def : Pat<(i1 (setcc i32:$s1, imm:$imm, SETEQ)),
          (EXTRACT_SUBREG (CMPLWI (XORIS $s1, (HI16 imm:$imm)),
                                  (LO16 imm:$imm)), sub_eq)>;

def : Pat<(i1 (setcc i32:$s1, i32:$s2, SETULT)),
          (EXTRACT_SUBREG (CMPLW $s1, $s2), sub_lt)>;
def : Pat<(i1 (setcc i32:$s1, i32:$s2, SETLT)),
          (EXTRACT_SUBREG (CMPW $s1, $s2), sub_lt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4082-4089

```tablegen
def : Pat<(i1 (setcc i32:$s1, i32:$s2, SETUGT)),
          (EXTRACT_SUBREG (CMPLW $s1, $s2), sub_gt)>;
def : Pat<(i1 (setcc i32:$s1, i32:$s2, SETGT)),
          (EXTRACT_SUBREG (CMPW $s1, $s2), sub_gt)>;
def : Pat<(i1 (setcc i32:$s1, i32:$s2, SETEQ)),
          (EXTRACT_SUBREG (CMPW $s1, $s2), sub_eq)>;

// SETCC for i64.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4090-4097

```tablegen
def : Pat<(i1 (setcc i64:$s1, immZExt16:$imm, SETULT)),
          (EXTRACT_SUBREG (CMPLDI $s1, imm:$imm), sub_lt)>;
def : Pat<(i1 (setcc i64:$s1, imm64SExt16:$imm, SETLT)),
          (EXTRACT_SUBREG (CMPDI $s1, imm:$imm), sub_lt)>;
def : Pat<(i1 (setcc i64:$s1, immZExt16:$imm, SETUGT)),
          (EXTRACT_SUBREG (CMPLDI $s1, imm:$imm), sub_gt)>;
def : Pat<(i1 (setcc i64:$s1, imm64SExt16:$imm, SETGT)),
          (EXTRACT_SUBREG (CMPDI $s1, imm:$imm), sub_gt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4098-4113

```tablegen
def : Pat<(i1 (setcc i64:$s1, imm64SExt16:$imm, SETEQ)),
          (EXTRACT_SUBREG (CMPDI $s1, imm:$imm), sub_eq)>;
def : Pat<(i1 (setcc i64:$s1, immZExt16:$imm, SETEQ)),
          (EXTRACT_SUBREG (CMPLDI $s1, imm:$imm), sub_eq)>;

// For non-equality comparisons, the default code would materialize the
// constant, then compare against it, like this:
//   lis r2, 4660
//   ori r2, r2, 22136
//   cmpd cr0, r3, r2
//   beq cr0,L6
// Since we are just comparing for equality, we can emit this instead:
//   xoris r0,r3,0x1234
//   cmpldi cr0,r0,0x5678
//   beq cr0,L6
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "For non-equality comparisons, the default code would materialize the".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“For non-equality comparisons, the default code would materialize the”。

### Lines 4114-4121

```tablegen
def : Pat<(i1 (setcc i64:$s1, imm64ZExt32:$imm, SETEQ)),
          (EXTRACT_SUBREG (CMPLDI (XORIS8 $s1, (HI16 imm:$imm)),
                                  (LO16 imm:$imm)), sub_eq)>;

def : Pat<(i1 (setcc i64:$s1, i64:$s2, SETULT)),
          (EXTRACT_SUBREG (CMPLD $s1, $s2), sub_lt)>;
def : Pat<(i1 (setcc i64:$s1, i64:$s2, SETLT)),
          (EXTRACT_SUBREG (CMPD $s1, $s2), sub_lt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4122-4128

```tablegen
def : Pat<(i1 (setcc i64:$s1, i64:$s2, SETUGT)),
          (EXTRACT_SUBREG (CMPLD $s1, $s2), sub_gt)>;
def : Pat<(i1 (setcc i64:$s1, i64:$s2, SETGT)),
          (EXTRACT_SUBREG (CMPD $s1, $s2), sub_gt)>;
def : Pat<(i1 (setcc i64:$s1, i64:$s2, SETEQ)),
          (EXTRACT_SUBREG (CMPD $s1, $s2), sub_eq)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4129-4136

```tablegen
let Predicates = [IsNotISA3_1] in {
// Instantiations of CRNotPat for i32.
defm : CRNotPat<(i1 (setcc i32:$s1, immZExt16:$imm, SETUGE)),
                (EXTRACT_SUBREG (CMPLWI $s1, imm:$imm), sub_lt)>;
defm : CRNotPat<(i1 (setcc i32:$s1, imm32SExt16:$imm, SETGE)),
                (EXTRACT_SUBREG (CMPWI $s1, imm:$imm), sub_lt)>;
defm : CRNotPat<(i1 (setcc i32:$s1, immZExt16:$imm, SETULE)),
                (EXTRACT_SUBREG (CMPLWI $s1, imm:$imm), sub_gt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4137-4143

```tablegen
defm : CRNotPat<(i1 (setcc i32:$s1, imm32SExt16:$imm, SETLE)),
                (EXTRACT_SUBREG (CMPWI $s1, imm:$imm), sub_gt)>;
defm : CRNotPat<(i1 (setcc i32:$s1, imm32SExt16:$imm, SETNE)),
                (EXTRACT_SUBREG (CMPWI $s1, imm:$imm), sub_eq)>;
defm : CRNotPat<(i1 (setcc i32:$s1, immZExt16:$imm, SETNE)),
                (EXTRACT_SUBREG (CMPLWI $s1, imm:$imm), sub_eq)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4144-4151

```tablegen
defm : CRNotPat<(i1 (setcc i32:$s1, imm:$imm, SETNE)),
                (EXTRACT_SUBREG (CMPLWI (XORIS $s1, (HI16 imm:$imm)),
                                        (LO16 imm:$imm)), sub_eq)>;

defm : CRNotPat<(i1 (setcc i32:$s1, i32:$s2, SETUGE)),
                (EXTRACT_SUBREG (CMPLW $s1, $s2), sub_lt)>;
defm : CRNotPat<(i1 (setcc i32:$s1, i32:$s2, SETGE)),
                (EXTRACT_SUBREG (CMPW $s1, $s2), sub_lt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4152-4159

```tablegen
defm : CRNotPat<(i1 (setcc i32:$s1, i32:$s2, SETULE)),
                (EXTRACT_SUBREG (CMPLW $s1, $s2), sub_gt)>;
defm : CRNotPat<(i1 (setcc i32:$s1, i32:$s2, SETLE)),
                (EXTRACT_SUBREG (CMPW $s1, $s2), sub_gt)>;
defm : CRNotPat<(i1 (setcc i32:$s1, i32:$s2, SETNE)),
                (EXTRACT_SUBREG (CMPW $s1, $s2), sub_eq)>;

// Instantiations of CRNotPat for i64.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4160-4167

```tablegen
defm : CRNotPat<(i1 (setcc i64:$s1, immZExt16:$imm, SETUGE)),
                (EXTRACT_SUBREG (CMPLDI $s1, imm:$imm), sub_lt)>;
defm : CRNotPat<(i1 (setcc i64:$s1, imm64SExt16:$imm, SETGE)),
                (EXTRACT_SUBREG (CMPDI $s1, imm:$imm), sub_lt)>;
defm : CRNotPat<(i1 (setcc i64:$s1, immZExt16:$imm, SETULE)),
                (EXTRACT_SUBREG (CMPLDI $s1, imm:$imm), sub_gt)>;
defm : CRNotPat<(i1 (setcc i64:$s1, imm64SExt16:$imm, SETLE)),
                (EXTRACT_SUBREG (CMPDI $s1, imm:$imm), sub_gt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4168-4176

```tablegen
defm : CRNotPat<(i1 (setcc i64:$s1, imm64SExt16:$imm, SETNE)),
                (EXTRACT_SUBREG (CMPDI $s1, imm:$imm), sub_eq)>;
defm : CRNotPat<(i1 (setcc i64:$s1, immZExt16:$imm, SETNE)),
                (EXTRACT_SUBREG (CMPLDI $s1, imm:$imm), sub_eq)>;

defm : CRNotPat<(i1 (setcc i64:$s1, imm64ZExt32:$imm, SETNE)),
                (EXTRACT_SUBREG (CMPLDI (XORIS8 $s1, (HI16 imm:$imm)),
                                        (LO16 imm:$imm)), sub_eq)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4177-4184

```tablegen
defm : CRNotPat<(i1 (setcc i64:$s1, i64:$s2, SETUGE)),
                (EXTRACT_SUBREG (CMPLD $s1, $s2), sub_lt)>;
defm : CRNotPat<(i1 (setcc i64:$s1, i64:$s2, SETGE)),
                (EXTRACT_SUBREG (CMPD $s1, $s2), sub_lt)>;
defm : CRNotPat<(i1 (setcc i64:$s1, i64:$s2, SETULE)),
                (EXTRACT_SUBREG (CMPLD $s1, $s2), sub_gt)>;
defm : CRNotPat<(i1 (setcc i64:$s1, i64:$s2, SETLE)),
                (EXTRACT_SUBREG (CMPD $s1, $s2), sub_gt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4185-4191

```tablegen
defm : CRNotPat<(i1 (setcc i64:$s1, i64:$s2, SETNE)),
                (EXTRACT_SUBREG (CMPD $s1, $s2), sub_eq)>;
}

multiclass FSetCCPat<SDPatternOperator SetCC, ValueType Ty, I FCmp> {
  defm : CRNotPat<(i1 (SetCC Ty:$s1, Ty:$s2, SETUGE)),
                  (EXTRACT_SUBREG (FCmp $s1, $s2), sub_lt)>;
```
- **EN**: Adds declarative TableGen records such as `FSetCCPat` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FSetCCPat`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4192-4199

```tablegen
  defm : CRNotPat<(i1 (SetCC Ty:$s1, Ty:$s2, SETGE)),
                  (EXTRACT_SUBREG (FCmp $s1, $s2), sub_lt)>;
  defm : CRNotPat<(i1 (SetCC Ty:$s1, Ty:$s2, SETULE)),
                  (EXTRACT_SUBREG (FCmp $s1, $s2), sub_gt)>;
  defm : CRNotPat<(i1 (SetCC Ty:$s1, Ty:$s2, SETLE)),
                  (EXTRACT_SUBREG (FCmp $s1, $s2), sub_gt)>;
  defm : CRNotPat<(i1 (SetCC Ty:$s1, Ty:$s2, SETUNE)),
                  (EXTRACT_SUBREG (FCmp $s1, $s2), sub_eq)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4200-4206

```tablegen
  defm : CRNotPat<(i1 (SetCC Ty:$s1, Ty:$s2, SETNE)),
                  (EXTRACT_SUBREG (FCmp $s1, $s2), sub_eq)>;
  defm : CRNotPat<(i1 (SetCC Ty:$s1, Ty:$s2, SETO)),
                  (EXTRACT_SUBREG (FCmp $s1, $s2), sub_un)>;

  def : Pat<(i1 (SetCC Ty:$s1, Ty:$s2, SETOLT)),
            (EXTRACT_SUBREG (FCmp $s1, $s2), sub_lt)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4207-4214

```tablegen
  def : Pat<(i1 (SetCC Ty:$s1, Ty:$s2, SETLT)),
            (EXTRACT_SUBREG (FCmp $s1, $s2), sub_lt)>;
  def : Pat<(i1 (SetCC Ty:$s1, Ty:$s2, SETOGT)),
            (EXTRACT_SUBREG (FCmp $s1, $s2), sub_gt)>;
  def : Pat<(i1 (SetCC Ty:$s1, Ty:$s2, SETGT)),
            (EXTRACT_SUBREG (FCmp $s1, $s2), sub_gt)>;
  def : Pat<(i1 (SetCC Ty:$s1, Ty:$s2, SETOEQ)),
            (EXTRACT_SUBREG (FCmp $s1, $s2), sub_eq)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4215-4223

```tablegen
  def : Pat<(i1 (SetCC Ty:$s1, Ty:$s2, SETEQ)),
            (EXTRACT_SUBREG (FCmp $s1, $s2), sub_eq)>;
  def : Pat<(i1 (SetCC Ty:$s1, Ty:$s2, SETUO)),
            (EXTRACT_SUBREG (FCmp $s1, $s2), sub_un)>;
}

let Predicates = [HasFPU] in {
// FCMPU: If either of the operands is a Signaling NaN, then VXSNAN is set.
// SETCC for f32.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4224-4235

```tablegen
defm : FSetCCPat<any_fsetcc, f32, FCMPUS>;

// SETCC for f64.
defm : FSetCCPat<any_fsetcc, f64, FCMPUD>;

// SETCC for f128.
defm : FSetCCPat<any_fsetcc, f128, XSCMPUQP>;

// FCMPO: If either of the operands is a Signaling NaN, then VXSNAN is set and,
// if neither operand is a Signaling NaN but at least one operand is a Quiet NaN,
// then VXVC is set.
// SETCCS for f32.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "SETCC for f64.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“SETCC for f64.”。

### Lines 4236-4246

```tablegen
defm : FSetCCPat<strict_fsetccs, f32, FCMPOS>;

// SETCCS for f64.
defm : FSetCCPat<strict_fsetccs, f64, FCMPOD>;

// SETCCS for f128.
defm : FSetCCPat<strict_fsetccs, f128, XSCMPOQP>;
}

// This must be in this file because it relies on patterns defined in this file
// after the inclusion of the instruction sets.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "SETCCS for f64.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“SETCCS for f64.”。

### Lines 4247-4254

```tablegen
let Predicates = [HasSPE] in {
// SETCC for f32.
def : Pat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETOLT)),
          (EXTRACT_SUBREG (EFSCMPLT $s1, $s2), sub_gt)>;
def : Pat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETLT)),
          (EXTRACT_SUBREG (EFSCMPLT $s1, $s2), sub_gt)>;
def : Pat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETOGT)),
          (EXTRACT_SUBREG (EFSCMPGT $s1, $s2), sub_gt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4255-4261

```tablegen
def : Pat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETGT)),
          (EXTRACT_SUBREG (EFSCMPGT $s1, $s2), sub_gt)>;
def : Pat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETOEQ)),
          (EXTRACT_SUBREG (EFSCMPEQ $s1, $s2), sub_gt)>;
def : Pat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETEQ)),
          (EXTRACT_SUBREG (EFSCMPEQ $s1, $s2), sub_gt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4262-4269

```tablegen
defm : CRNotPat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETUGE)),
                (EXTRACT_SUBREG (EFSCMPLT $s1, $s2), sub_gt)>;
defm : CRNotPat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETGE)),
                (EXTRACT_SUBREG (EFSCMPLT $s1, $s2), sub_gt)>;
defm : CRNotPat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETULE)),
                (EXTRACT_SUBREG (EFSCMPGT $s1, $s2), sub_gt)>;
defm : CRNotPat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETLE)),
                (EXTRACT_SUBREG (EFSCMPGT $s1, $s2), sub_gt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4270-4277

```tablegen
defm : CRNotPat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETUNE)),
                (EXTRACT_SUBREG (EFSCMPEQ $s1, $s2), sub_gt)>;
defm : CRNotPat<(i1 (any_fsetccs f32:$s1, f32:$s2, SETNE)),
                (EXTRACT_SUBREG (EFSCMPEQ $s1, $s2), sub_gt)>;

// SETCC for f64.
def : Pat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETOLT)),
          (EXTRACT_SUBREG (EFDCMPLT $s1, $s2), sub_gt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4278-4285

```tablegen
def : Pat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETLT)),
          (EXTRACT_SUBREG (EFDCMPLT $s1, $s2), sub_gt)>;
def : Pat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETOGT)),
          (EXTRACT_SUBREG (EFDCMPGT $s1, $s2), sub_gt)>;
def : Pat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETGT)),
          (EXTRACT_SUBREG (EFDCMPGT $s1, $s2), sub_gt)>;
def : Pat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETOEQ)),
          (EXTRACT_SUBREG (EFDCMPEQ $s1, $s2), sub_gt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4286-4292

```tablegen
def : Pat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETEQ)),
          (EXTRACT_SUBREG (EFDCMPEQ $s1, $s2), sub_gt)>;

defm : CRNotPat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETUGE)),
                (EXTRACT_SUBREG (EFDCMPLT $s1, $s2), sub_gt)>;
defm : CRNotPat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETGE)),
                (EXTRACT_SUBREG (EFDCMPLT $s1, $s2), sub_gt)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4293-4302

```tablegen
defm : CRNotPat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETULE)),
                (EXTRACT_SUBREG (EFDCMPGT $s1, $s2), sub_gt)>;
defm : CRNotPat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETLE)),
                (EXTRACT_SUBREG (EFDCMPGT $s1, $s2), sub_gt)>;
defm : CRNotPat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETUNE)),
                (EXTRACT_SUBREG (EFDCMPEQ $s1, $s2), sub_gt)>;
defm : CRNotPat<(i1 (any_fsetccs f64:$s1, f64:$s2, SETNE)),
                (EXTRACT_SUBREG (EFDCMPEQ $s1, $s2), sub_gt)>;
}
// match select on i1 variables:
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4303-4309

```tablegen
def : Pat<(i1 (select i1:$cond, i1:$tval, i1:$fval)),
          (CROR (CRAND        $cond , $tval),
                (CRAND (crnot $cond), $fval))>;

// match selectcc on i1 variables:
//   select (lhs == rhs), tval, fval is:
//   ((lhs == rhs) & tval) | (!(lhs == rhs) & fval)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "match selectcc on i1 variables:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“match selectcc on i1 variables:”。

### Lines 4310-4318

```tablegen
def : Pat <(i1 (selectcc i1:$lhs, i1:$rhs, i1:$tval, i1:$fval, SETLT)),
           (CROR (CRAND (CRANDC $lhs, $rhs), $tval),
                 (CRAND (CRORC  $rhs, $lhs), $fval))>;
def : Pat <(i1 (selectcc i1:$lhs, i1:$rhs, i1:$tval, i1:$fval, SETULT)),
           (CROR (CRAND (CRANDC $rhs, $lhs), $tval),
                 (CRAND (CRORC  $lhs, $rhs), $fval))>;
def : Pat <(i1 (selectcc i1:$lhs, i1:$rhs, i1:$tval, i1:$fval, SETLE)),
           (CROR (CRAND (CRORC  $lhs, $rhs), $tval),
                 (CRAND (CRANDC $rhs, $lhs), $fval))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4319-4327

```tablegen
def : Pat <(i1 (selectcc i1:$lhs, i1:$rhs, i1:$tval, i1:$fval, SETULE)),
           (CROR (CRAND (CRORC  $rhs, $lhs), $tval),
                 (CRAND (CRANDC $lhs, $rhs), $fval))>;
def : Pat <(i1 (selectcc i1:$lhs, i1:$rhs, i1:$tval, i1:$fval, SETEQ)),
           (CROR (CRAND (CREQV $lhs, $rhs), $tval),
                 (CRAND (CRXOR $lhs, $rhs), $fval))>;
def : Pat <(i1 (selectcc i1:$lhs, i1:$rhs, i1:$tval, i1:$fval, SETGE)),
           (CROR (CRAND (CRORC  $rhs, $lhs), $tval),
                 (CRAND (CRANDC $lhs, $rhs), $fval))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4328-4336

```tablegen
def : Pat <(i1 (selectcc i1:$lhs, i1:$rhs, i1:$tval, i1:$fval, SETUGE)),
           (CROR (CRAND (CRORC  $lhs, $rhs), $tval),
                 (CRAND (CRANDC $rhs, $lhs), $fval))>;
def : Pat <(i1 (selectcc i1:$lhs, i1:$rhs, i1:$tval, i1:$fval, SETGT)),
           (CROR (CRAND (CRANDC $rhs, $lhs), $tval),
                 (CRAND (CRORC  $lhs, $rhs), $fval))>;
def : Pat <(i1 (selectcc i1:$lhs, i1:$rhs, i1:$tval, i1:$fval, SETUGT)),
           (CROR (CRAND (CRANDC $lhs, $rhs), $tval),
                 (CRAND (CRORC  $rhs, $lhs), $fval))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4337-4343

```tablegen
def : Pat <(i1 (selectcc i1:$lhs, i1:$rhs, i1:$tval, i1:$fval, SETNE)),
           (CROR (CRAND (CREQV $lhs, $rhs), $fval),
                 (CRAND (CRXOR $lhs, $rhs), $tval))>;

// match selectcc on i1 variables with non-i1 output.
def : Pat<(i32 (selectcc i1:$lhs, i1:$rhs, i32:$tval, i32:$fval, SETLT)),
          (SELECT_I4 (CRANDC $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4344-4351

```tablegen
def : Pat<(i32 (selectcc i1:$lhs, i1:$rhs, i32:$tval, i32:$fval, SETULT)),
          (SELECT_I4 (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(i32 (selectcc i1:$lhs, i1:$rhs, i32:$tval, i32:$fval, SETLE)),
          (SELECT_I4 (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(i32 (selectcc i1:$lhs, i1:$rhs, i32:$tval, i32:$fval, SETULE)),
          (SELECT_I4 (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(i32 (selectcc i1:$lhs, i1:$rhs, i32:$tval, i32:$fval, SETEQ)),
          (SELECT_I4 (CREQV $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4352-4359

```tablegen
def : Pat<(i32 (selectcc i1:$lhs, i1:$rhs, i32:$tval, i32:$fval, SETGE)),
          (SELECT_I4 (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(i32 (selectcc i1:$lhs, i1:$rhs, i32:$tval, i32:$fval, SETUGE)),
          (SELECT_I4 (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(i32 (selectcc i1:$lhs, i1:$rhs, i32:$tval, i32:$fval, SETGT)),
          (SELECT_I4 (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(i32 (selectcc i1:$lhs, i1:$rhs, i32:$tval, i32:$fval, SETUGT)),
          (SELECT_I4 (CRANDC $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4360-4366

```tablegen
def : Pat<(i32 (selectcc i1:$lhs, i1:$rhs, i32:$tval, i32:$fval, SETNE)),
          (SELECT_I4 (CRXOR $lhs, $rhs), $tval, $fval)>;

def : Pat<(i64 (selectcc i1:$lhs, i1:$rhs, i64:$tval, i64:$fval, SETLT)),
          (SELECT_I8 (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(i64 (selectcc i1:$lhs, i1:$rhs, i64:$tval, i64:$fval, SETULT)),
          (SELECT_I8 (CRANDC $rhs, $lhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4367-4374

```tablegen
def : Pat<(i64 (selectcc i1:$lhs, i1:$rhs, i64:$tval, i64:$fval, SETLE)),
          (SELECT_I8 (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(i64 (selectcc i1:$lhs, i1:$rhs, i64:$tval, i64:$fval, SETULE)),
          (SELECT_I8 (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(i64 (selectcc i1:$lhs, i1:$rhs, i64:$tval, i64:$fval, SETEQ)),
          (SELECT_I8 (CREQV $lhs, $rhs), $tval, $fval)>;
def : Pat<(i64 (selectcc i1:$lhs, i1:$rhs, i64:$tval, i64:$fval, SETGE)),
          (SELECT_I8 (CRORC  $rhs, $lhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4375-4383

```tablegen
def : Pat<(i64 (selectcc i1:$lhs, i1:$rhs, i64:$tval, i64:$fval, SETUGE)),
          (SELECT_I8 (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(i64 (selectcc i1:$lhs, i1:$rhs, i64:$tval, i64:$fval, SETGT)),
          (SELECT_I8 (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(i64 (selectcc i1:$lhs, i1:$rhs, i64:$tval, i64:$fval, SETUGT)),
          (SELECT_I8 (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(i64 (selectcc i1:$lhs, i1:$rhs, i64:$tval, i64:$fval, SETNE)),
          (SELECT_I8 (CRXOR $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4384-4390

```tablegen
let Predicates = [HasFPU] in {
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETLT)),
          (SELECT_F4 (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETULT)),
          (SELECT_F4 (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETLE)),
          (SELECT_F4 (CRORC  $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4391-4398

```tablegen
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETULE)),
          (SELECT_F4 (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETEQ)),
          (SELECT_F4 (CREQV $lhs, $rhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETGE)),
          (SELECT_F4 (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETUGE)),
          (SELECT_F4 (CRORC  $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4399-4405

```tablegen
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETGT)),
          (SELECT_F4 (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETUGT)),
          (SELECT_F4 (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETNE)),
          (SELECT_F4 (CRXOR $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4406-4413

```tablegen
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETLT)),
          (SELECT_F8 (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETULT)),
          (SELECT_F8 (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETLE)),
          (SELECT_F8 (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETULE)),
          (SELECT_F8 (CRORC  $rhs, $lhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4414-4421

```tablegen
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETEQ)),
          (SELECT_F8 (CREQV $lhs, $rhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETGE)),
          (SELECT_F8 (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETUGE)),
          (SELECT_F8 (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETGT)),
          (SELECT_F8 (CRANDC $rhs, $lhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4422-4429

```tablegen
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETUGT)),
          (SELECT_F8 (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETNE)),
          (SELECT_F8 (CRXOR $lhs, $rhs), $tval, $fval)>;
}

def : Pat<(f128 (selectcc i1:$lhs, i1:$rhs, f128:$tval, f128:$fval, SETLT)),
          (SELECT_F16 (CRANDC $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4430-4437

```tablegen
def : Pat<(f128 (selectcc i1:$lhs, i1:$rhs, f128:$tval, f128:$fval, SETULT)),
          (SELECT_F16 (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(f128 (selectcc i1:$lhs, i1:$rhs, f128:$tval, f128:$fval, SETLE)),
          (SELECT_F16 (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(f128 (selectcc i1:$lhs, i1:$rhs, f128:$tval, f128:$fval, SETULE)),
          (SELECT_F16 (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(f128 (selectcc i1:$lhs, i1:$rhs, f128:$tval, f128:$fval, SETEQ)),
          (SELECT_F16 (CREQV $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4438-4445

```tablegen
def : Pat<(f128 (selectcc i1:$lhs, i1:$rhs, f128:$tval, f128:$fval, SETGE)),
         (SELECT_F16 (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(f128 (selectcc i1:$lhs, i1:$rhs, f128:$tval, f128:$fval, SETUGE)),
          (SELECT_F16 (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(f128 (selectcc i1:$lhs, i1:$rhs, f128:$tval, f128:$fval, SETGT)),
          (SELECT_F16 (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(f128 (selectcc i1:$lhs, i1:$rhs, f128:$tval, f128:$fval, SETUGT)),
          (SELECT_F16 (CRANDC $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4446-4452

```tablegen
def : Pat<(f128 (selectcc i1:$lhs, i1:$rhs, f128:$tval, f128:$fval, SETNE)),
          (SELECT_F16 (CRXOR $lhs, $rhs), $tval, $fval)>;

def : Pat<(v4i32 (selectcc i1:$lhs, i1:$rhs, v4i32:$tval, v4i32:$fval, SETLT)),
          (SELECT_VRRC (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(v4i32 (selectcc i1:$lhs, i1:$rhs, v4i32:$tval, v4i32:$fval, SETULT)),
          (SELECT_VRRC (CRANDC $rhs, $lhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4453-4460

```tablegen
def : Pat<(v4i32 (selectcc i1:$lhs, i1:$rhs, v4i32:$tval, v4i32:$fval, SETLE)),
          (SELECT_VRRC (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(v4i32 (selectcc i1:$lhs, i1:$rhs, v4i32:$tval, v4i32:$fval, SETULE)),
          (SELECT_VRRC (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(v4i32 (selectcc i1:$lhs, i1:$rhs, v4i32:$tval, v4i32:$fval, SETEQ)),
          (SELECT_VRRC (CREQV $lhs, $rhs), $tval, $fval)>;
def : Pat<(v4i32 (selectcc i1:$lhs, i1:$rhs, v4i32:$tval, v4i32:$fval, SETGE)),
          (SELECT_VRRC (CRORC  $rhs, $lhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4461-4469

```tablegen
def : Pat<(v4i32 (selectcc i1:$lhs, i1:$rhs, v4i32:$tval, v4i32:$fval, SETUGE)),
          (SELECT_VRRC (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(v4i32 (selectcc i1:$lhs, i1:$rhs, v4i32:$tval, v4i32:$fval, SETGT)),
          (SELECT_VRRC (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(v4i32 (selectcc i1:$lhs, i1:$rhs, v4i32:$tval, v4i32:$fval, SETUGT)),
          (SELECT_VRRC (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(v4i32 (selectcc i1:$lhs, i1:$rhs, v4i32:$tval, v4i32:$fval, SETNE)),
          (SELECT_VRRC (CRXOR $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4470-4477

```tablegen
let Defs = [CR0] in {
def ANDI_rec_1_EQ_BIT : PPCCustomInserterPseudo<(outs crbitrc:$dst), (ins gprc:$in),
                             "#ANDI_rec_1_EQ_BIT",
                             [(set i1:$dst, (trunc (not i32:$in)))]>;
def ANDI_rec_1_GT_BIT : PPCCustomInserterPseudo<(outs crbitrc:$dst), (ins gprc:$in),
                             "#ANDI_rec_1_GT_BIT",
                             [(set i1:$dst, (trunc i32:$in))]>;
```
- **EN**: Adds declarative TableGen records such as `ANDI_rec_1_EQ_BIT`, `ANDI_rec_1_GT_BIT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ANDI_rec_1_EQ_BIT`, `ANDI_rec_1_GT_BIT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4478-4485

```tablegen
def ANDI_rec_1_EQ_BIT8 : PPCCustomInserterPseudo<(outs crbitrc:$dst), (ins g8rc:$in),
                              "#ANDI_rec_1_EQ_BIT8",
                              [(set i1:$dst, (trunc (not i64:$in)))]>;
def ANDI_rec_1_GT_BIT8 : PPCCustomInserterPseudo<(outs crbitrc:$dst), (ins g8rc:$in),
                              "#ANDI_rec_1_GT_BIT8",
                              [(set i1:$dst, (trunc i64:$in))]>;
}
```
- **EN**: Adds declarative TableGen records such as `ANDI_rec_1_EQ_BIT8`, `ANDI_rec_1_GT_BIT8` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ANDI_rec_1_EQ_BIT8`, `ANDI_rec_1_GT_BIT8`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4486-4492

```tablegen
def : Pat<(i1 (not (trunc i32:$in))),
           (ANDI_rec_1_EQ_BIT $in)>;
def : Pat<(i1 (not (trunc i64:$in))),
           (ANDI_rec_1_EQ_BIT8 $in)>;

def : Pat<(int_ppc_fsel f8rc:$FRA, f8rc:$FRC, f8rc:$FRB), (FSELD $FRA, $FRC, $FRB)>;
def : Pat<(int_ppc_frsqrte f8rc:$frB), (FRSQRTE $frB)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4493-4494

```tablegen
def : Pat<(int_ppc_frsqrtes f4rc:$frB), (FRSQRTES $frB)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4495-4506

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC Instructions used for assembler/disassembler only
//

// FIXME: For B=0 or B > 8, the registers following RT are used.
// WARNING: Do not add patterns for this instruction without fixing this.
def LSWI  : XForm_base_r3xo_memOp<31, 597, (outs gprc:$RST),
                                  (ins gprc:$RA, u5imm:$RB),
                                  "lswi $RST, $RA, $RB", IIC_LdStLoad, []>;

// FIXME: For B=0 or B > 8, the registers following RT are used.
// WARNING: Do not add patterns for this instruction without fixing this.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PowerPC Instructions used for assembler/disassembler only". Notable symbols in this range include `LSWI`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PowerPC Instructions used for assembler/disassembler only”。 该区间中较显眼的符号包括 `LSWI`。

### Lines 4507-4513

```tablegen
def STSWI : XForm_base_r3xo_memOp<31, 725, (outs),
                                  (ins gprc:$RST, gprc:$RA, u5imm:$RB),
                                  "stswi $RST, $RA, $RB", IIC_LdStLoad, []>;

def ISYNC : XLForm_2_ext<19, 150, 0, 0, 0, (outs), (ins),
                         "isync", IIC_SprISYNC, []>;
```
- **EN**: Adds declarative TableGen records such as `STSWI`, `ISYNC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STSWI`, `ISYNC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4514-4522

```tablegen
def ICBI : XForm_1a<31, 982, (outs), (ins (memrr $RA, $RB):$addr),
                    "icbi $addr", IIC_LdStICBI, []>;

def WAIT : XForm_24_sync<31, 30, (outs), (ins u2imm:$L),
                         "wait $L", IIC_LdStLoad, []>;

def MBAR : XForm_mbar<31, 854, (outs), (ins u5imm:$MO),
                         "mbar $MO", IIC_LdStLoad>, Requires<[IsBookE]>;
```
- **EN**: Adds declarative TableGen records such as `ICBI`, `WAIT`, `MBAR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ICBI`, `WAIT`, `MBAR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4523-4531

```tablegen
def MTSR: XForm_sr<31, 210, (outs), (ins gprc:$RS, u4imm:$SR),
            "mtsr $SR, $RS", IIC_SprMTSR>;

def MFSR: XForm_sr<31, 595, (outs gprc:$RS), (ins u4imm:$SR),
            "mfsr $RS, $SR", IIC_SprMFSR>;

def MTSRIN: XForm_srin<31, 242, (outs), (ins gprc:$RS, gprc:$RB),
            "mtsrin $RS, $RB", IIC_SprMTSR>;
```
- **EN**: Adds declarative TableGen records such as `MTSR`, `MFSR`, `MTSRIN` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTSR`, `MFSR`, `MTSRIN`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4532-4539

```tablegen
def MFSRIN: XForm_srin<31, 659, (outs gprc:$RS), (ins gprc:$RB),
            "mfsrin $RS, $RB", IIC_SprMFSR>;

def MTMSR: XForm_mtmsr<31, 146, (outs), (ins gprc:$RS, u1imm:$L),
                    "mtmsr $RS, $L", IIC_SprMTMSR>;

def WRTEE: XForm_mtmsr<31, 131, (outs), (ins gprc:$RS),
                    "wrtee $RS", IIC_SprMTMSR>, Requires<[IsBookE]> {
```
- **EN**: Adds declarative TableGen records such as `MFSRIN`, `MTMSR`, `WRTEE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MFSRIN`, `MTMSR`, `WRTEE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4540-4546

```tablegen
  let L = 0;
}

def WRTEEI: I<31, (outs), (ins i1imm:$E), "wrteei $E", IIC_SprMTMSR>,
              Requires<[IsBookE]> {
  bits<1> E;
```
- **EN**: Adds declarative TableGen records such as `WRTEEI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WRTEEI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4547-4555

```tablegen
  let Inst{16} = E;
  let Inst{21...30} = 163;
}

def DCCCI : XForm_tlb<454, (outs), (ins gprc:$RA, gprc:$RB),
               "dccci $RA, $RB", IIC_LdStLoad>, Requires<[IsPPC4xx]>;
def ICCCI : XForm_tlb<966, (outs), (ins gprc:$RA, gprc:$RB),
               "iccci $RA, $RB", IIC_LdStLoad>, Requires<[IsPPC4xx]>;
```
- **EN**: Adds declarative TableGen records such as `DCCCI`, `ICCCI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DCCCI`, `ICCCI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4556-4563

```tablegen
def : InstAlias<"dci 0", (DCCCI R0, R0)>, Requires<[IsPPC4xx]>;
def : InstAlias<"dccci", (DCCCI R0, R0)>, Requires<[IsPPC4xx]>;
def : InstAlias<"ici 0", (ICCCI R0, R0)>, Requires<[IsPPC4xx]>;
def : InstAlias<"iccci", (ICCCI R0, R0)>, Requires<[IsPPC4xx]>;

def MFMSR : XForm_rs<31, 83, (outs gprc:$RST), (ins),
                  "mfmsr $RST", IIC_SprMFMSR, []>;
```
- **EN**: Adds declarative TableGen records such as `MFMSR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MFMSR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4564-4571

```tablegen
def MTMSRD : XForm_mtmsr<31, 178, (outs), (ins gprc:$RS, u1imm:$L),
                    "mtmsrd $RS, $L", IIC_SprMTMSRD>;

def MCRFS : XLForm_3<63, 64, (outs crrc:$BF), (ins crrc:$BFA),
                     "mcrfs $BF, $BFA", IIC_BrMCR>;

// All MTFSF variants may change the rounding mode so conservatively set it
// as an implicit def for all of them.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "All MTFSF variants may change the rounding mode so conservatively set it". Notable symbols in this range include `MTMSRD`, `MCRFS`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“All MTFSF variants may change the rounding mode so conservatively set it”。 该区间中较显眼的符号包括 `MTMSRD`, `MCRFS`。

### Lines 4572-4579

```tablegen
let Predicates = [HasFPU] in {
let Defs = [RM], hasSideEffects = 1 in {
let isCodeGenOnly = 1,
    Pattern = [(int_ppc_mtfsfi u3imm_timm:$BF, u4imm_timm:$U)], W = 0 in
def MTFSFIb : XLForm_4<63, 134, (outs), (ins u3imm:$BF, u4imm:$U),
                       "mtfsfi $BF, $U", IIC_IntMFFS>;
def MTFSFI : XLForm_4<63, 134, (outs), (ins u3imm:$BF, u4imm:$U, i32imm:$W),
                      "mtfsfi $BF, $U, $W", IIC_IntMFFS>;
```
- **EN**: Adds declarative TableGen records such as `MTFSFIb`, `MTFSFI` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTFSFIb`, `MTFSFI`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4580-4586

```tablegen
let Defs = [CR1] in
def MTFSFI_rec : XLForm_4<63, 134, (outs), (ins u3imm:$BF, u4imm:$U, u1imm:$W),
                       "mtfsfi. $BF, $U, $W", IIC_IntMFFS>, isRecordForm;

def MTFSF : XFLForm_1<63, 711, (outs),
                      (ins i32imm:$FLM, f8rc:$FRB, u1imm:$L, i32imm:$W),
                      "mtfsf $FLM, $FRB, $L, $W", IIC_IntMFFS, []>;
```
- **EN**: Adds declarative TableGen records such as `MTFSFI_rec`, `MTFSF` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTFSFI_rec`, `MTFSF`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4587-4593

```tablegen
let Defs = [CR1] in
def MTFSF_rec : XFLForm_1<63, 711, (outs),
                       (ins i32imm:$FLM, f8rc:$FRB, u1imm:$L, i32imm:$W),
                       "mtfsf. $FLM, $FRB, $L, $W", IIC_IntMFFS, []>, isRecordForm;
}

def : InstAlias<"mtfsfi $BF, $U", (MTFSFI u3imm:$BF, u4imm:$U, 0)>;
```
- **EN**: Adds declarative TableGen records such as `MTFSF_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTFSF_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4594-4601

```tablegen
def : InstAlias<"mtfsfi. $BF, $U", (MTFSFI_rec u3imm:$BF, u4imm:$U, 0)>;
def : InstAlias<"mtfsf $FLM, $FRB", (MTFSF i32imm:$FLM, f8rc:$FRB, 0, 0)>;
def : InstAlias<"mtfsf. $FLM, $FRB", (MTFSF_rec i32imm:$FLM, f8rc:$FRB, 0, 0)>;
}

def SLBIE : XForm_16b<31, 434, (outs), (ins gprc:$RB),
                        "slbie $RB", IIC_SprSLBIE, []>;
```
- **EN**: Adds declarative TableGen records such as `SLBIE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SLBIE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4602-4610

```tablegen
def SLBMTE : XForm_26<31, 402, (outs), (ins gprc:$RST, gprc:$RB),
                    "slbmte $RST, $RB", IIC_SprSLBMTE, []>;

def SLBMFEE : XForm_26<31, 915, (outs gprc:$RST), (ins gprc:$RB),
                       "slbmfee $RST, $RB", IIC_SprSLBMFEE, []>;

def SLBMFEV : XLForm_1_gen<31, 851, (outs gprc:$RT), (ins gprc:$RB),
                       "slbmfev $RT, $RB", IIC_SprSLBMFEV, []>;
```
- **EN**: Adds declarative TableGen records such as `SLBMTE`, `SLBMFEE`, `SLBMFEV` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SLBMTE`, `SLBMFEE`, `SLBMFEV`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4611-4619

```tablegen
def SLBIA : XForm_0<31, 498, (outs), (ins), "slbia", IIC_SprSLBIA, []>;

let Defs = [CR0] in
def SLBFEE_rec : XForm_26<31, 979, (outs gprc:$RST), (ins gprc:$RB),
                         "slbfee. $RST, $RB", IIC_SprSLBFEE, []>, isRecordForm;

def TLBIA : XForm_0<31, 370, (outs), (ins),
                        "tlbia", IIC_SprTLBIA, []>;
```
- **EN**: Adds declarative TableGen records such as `SLBIA`, `SLBFEE_rec`, `TLBIA` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SLBIA`, `SLBFEE_rec`, `TLBIA`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4620-4627

```tablegen
def TLBSYNC : XForm_0<31, 566, (outs), (ins),
                        "tlbsync", IIC_SprTLBSYNC, []>;

def TLBIEL : XForm_16b<31, 274, (outs), (ins gprc:$RB),
                          "tlbiel $RB", IIC_SprTLBIEL, []>;

def TLBLD : XForm_16b<31, 978, (outs), (ins gprc:$RB),
                          "tlbld $RB", IIC_LdStLoad, []>, Requires<[IsPPC6xx]>;
```
- **EN**: Adds declarative TableGen records such as `TLBSYNC`, `TLBIEL`, `TLBLD` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TLBSYNC`, `TLBIEL`, `TLBLD`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4628-4634

```tablegen
def TLBLI : XForm_16b<31, 1010, (outs), (ins gprc:$RB),
                          "tlbli $RB", IIC_LdStLoad, []>, Requires<[IsPPC6xx]>;

def TLBIE : XForm_26<31, 306, (outs), (ins gprc:$RST, gprc:$RB),
                     "tlbie $RB, $RST", IIC_SprTLBIE, []>,
            Requires<[IsNotISA3_0]>;
```
- **EN**: Adds declarative TableGen records such as `TLBLI`, `TLBIE` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TLBLI`, `TLBIE`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4635-4649

```tablegen
let Predicates = [IsISA3_0] in {
  def TLBIEP9 : XForm_RSB5_UIMM2_2UIMM1<31, 306, (outs),
                                        (ins gprc:$RB, gprc:$RS, u2imm:$RIC,
                                            u1imm:$PRS, u1imm:$R),
                                        "tlbie $RB, $RS, $RIC, $PRS, $R", []>;
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
    def TLBIE8P9
        : XForm_RSB5_UIMM2_2UIMM1<31, 306, (outs),
                                  (ins g8rc:$RB, g8rc:$RS, u2imm:$RIC,
                                      u1imm:$PRS, u1imm:$R),
                                  "tlbie $RB, $RS, $RIC, $PRS, $R", []>;
  }
}

// Processor Control Instructions
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4650-4658

```tablegen
let Predicates = [IsISA2_07], hasSideEffects = 1 in {
  def MSGSNDP : XForm_16b<31, 142, (outs), (ins gprc:$RB),
                              "msgsndp $RB", IIC_SprMSGSNDP, []>;
  let Interpretation64Bit = 1, isCodeGenOnly = 1 in {
    def MSGSNDP8 : XForm_16b<31, 142, (outs), (ins g8rc:$RB),
                             "msgsndp $RB", IIC_SprMSGSNDP, []>;
  }
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4659-4667

```tablegen
def TLBSX : XForm_tlb<914, (outs), (ins gprc:$RA, gprc:$RB), "tlbsx $RA, $RB",
                IIC_LdStLoad>, Requires<[IsBookE]>;

def TLBIVAX : XForm_tlb<786, (outs), (ins gprc:$RA, gprc:$RB), "tlbivax $RA, $RB",
                IIC_LdStLoad>, Requires<[IsBookE]>;

def TLBILX : XForm_tlbilx<18, (outs), (ins u2imm:$T, gprc:$RA, gprc:$RB),
    "tlbilx $T, $RA, $RB", IIC_LdStLoad>, Requires<[IsBookE]>;
```
- **EN**: Adds declarative TableGen records such as `TLBSX`, `TLBIVAX`, `TLBILX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TLBSX`, `TLBIVAX`, `TLBILX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4668-4676

```tablegen
def TLBRE : XForm_24_eieio<31, 946, (outs), (ins),
                           "tlbre", IIC_LdStLoad, []>, Requires<[IsBookE]>;

def TLBWE : XForm_24_eieio<31, 978, (outs), (ins),
                           "tlbwe", IIC_LdStLoad, []>, Requires<[IsBookE]>;

def TLBRE2 : XForm_tlbws<31, 946, (outs gprc:$RST), (ins gprc:$RA, i1imm:$WS),
               "tlbre $RST, $RA, $WS", IIC_LdStLoad, []>, Requires<[IsPPC4xx]>;
```
- **EN**: Adds declarative TableGen records such as `TLBRE`, `TLBWE`, `TLBRE2` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TLBRE`, `TLBWE`, `TLBRE2`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4677-4687

```tablegen
def TLBWE2 : XForm_tlbws<31, 978, (outs), (ins gprc:$RST, gprc:$RA, i1imm:$WS),
               "tlbwe $RST, $RA, $WS", IIC_LdStLoad, []>, Requires<[IsPPC4xx]>;

def TLBSX2 : XForm_base_r3xo<31, 914, (outs), (ins gprc:$RST, gprc:$RA, gprc:$RB),
                             "tlbsx $RST, $RA, $RB", IIC_LdStLoad, []>,
                             Requires<[IsPPC4xx]>;
def TLBSX2D : XForm_base_r3xo<31, 914, (outs),
                              (ins gprc:$RST, gprc:$RA, gprc:$RB),
                              "tlbsx. $RST, $RA, $RB", IIC_LdStLoad, []>,
                              Requires<[IsPPC4xx]>, isRecordForm;
```
- **EN**: Adds declarative TableGen records such as `TLBWE2`, `TLBSX2`, `TLBSX2D` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TLBWE2`, `TLBSX2`, `TLBSX2D`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4688-4694

```tablegen
def RFID : XForm_0<19, 18, (outs), (ins), "rfid", IIC_IntRFID, []>;

def RFI : XForm_0<19, 50, (outs), (ins), "rfi", IIC_SprRFI, []>,
                  Requires<[IsBookE]>;
def RFCI : XForm_0<19, 51, (outs), (ins), "rfci", IIC_BrB, []>,
                   Requires<[IsBookE]>;
```
- **EN**: Adds declarative TableGen records such as `RFID`, `RFI`, `RFCI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RFID`, `RFI`, `RFCI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4695-4701

```tablegen
def RFDI : XForm_0<19, 39, (outs), (ins), "rfdi", IIC_BrB, []>,
                   Requires<[IsE500]>;
def RFMCI : XForm_0<19, 38, (outs), (ins), "rfmci", IIC_BrB, []>,
                    Requires<[IsE500]>;

def MFDCR : XFXForm_1<31, 323, (outs gprc:$RST), (ins i32imm:$SPR),
                      "mfdcr $RST, $SPR", IIC_SprMFSPR>, Requires<[IsPPC4xx]>;
```
- **EN**: Adds declarative TableGen records such as `RFDI`, `RFMCI`, `MFDCR` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RFDI`, `RFMCI`, `MFDCR`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4702-4709

```tablegen
def MTDCR : XFXForm_1<31, 451, (outs), (ins gprc:$RST, i32imm:$SPR),
                      "mtdcr $SPR, $RST", IIC_SprMTSPR>, Requires<[IsPPC4xx]>;

def HRFID : XLForm_1_np<19, 274, (outs), (ins), "hrfid", IIC_BrB, []>;
def NAP   : XLForm_1_np<19, 434, (outs), (ins), "nap", IIC_BrB, []>;

def ATTN : XForm_attn<0, 256, (outs), (ins), "attn", IIC_BrB>;
```
- **EN**: Adds declarative TableGen records such as `MTDCR`, `HRFID`, `NAP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTDCR`, `HRFID`, `NAP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4710-4718

```tablegen
def LBZCIX : XForm_base_r3xo_memOp<31, 853, (outs gprc:$RST),
                                  (ins gprc:$RA, gprc:$RB),
                                  "lbzcix $RST, $RA, $RB", IIC_LdStLoad, []>;
def LHZCIX : XForm_base_r3xo_memOp<31, 821, (outs gprc:$RST),
                                  (ins gprc:$RA, gprc:$RB),
                                  "lhzcix $RST, $RA, $RB", IIC_LdStLoad, []>;
def LWZCIX : XForm_base_r3xo_memOp<31, 789, (outs gprc:$RST),
                                  (ins gprc:$RA, gprc:$RB),
                                  "lwzcix $RST, $RA, $RB", IIC_LdStLoad, []>;
```
- **EN**: Adds declarative TableGen records such as `LBZCIX`, `LHZCIX`, `LWZCIX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LBZCIX`, `LHZCIX`, `LWZCIX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4719-4725

```tablegen
def LDCIX :  XForm_base_r3xo_memOp<31, 885, (outs gprc:$RST),
                                  (ins gprc:$RA, gprc:$RB),
                                  "ldcix $RST, $RA, $RB", IIC_LdStLoad, []>;

def STBCIX : XForm_base_r3xo_memOp<31, 981, (outs),
                                  (ins gprc:$RST, gprc:$RA, gprc:$RB),
                                  "stbcix $RST, $RA, $RB", IIC_LdStLoad, []>;
```
- **EN**: Adds declarative TableGen records such as `LDCIX`, `STBCIX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LDCIX`, `STBCIX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4726-4737

```tablegen
def STHCIX : XForm_base_r3xo_memOp<31, 949, (outs),
                                  (ins gprc:$RST, gprc:$RA, gprc:$RB),
                                  "sthcix $RST, $RA, $RB", IIC_LdStLoad, []>;
def STWCIX : XForm_base_r3xo_memOp<31, 917, (outs),
                                  (ins gprc:$RST, gprc:$RA, gprc:$RB),
                                  "stwcix $RST, $RA, $RB", IIC_LdStLoad, []>;
def STDCIX : XForm_base_r3xo_memOp<31, 1013, (outs),
                                  (ins gprc:$RST, gprc:$RA, gprc:$RB),
                                  "stdcix $RST, $RA, $RB", IIC_LdStLoad, []>;

// External PID Load Store Instructions
```
- **EN**: Adds declarative TableGen records such as `STHCIX`, `STWCIX`, `STDCIX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STHCIX`, `STWCIX`, `STDCIX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4738-4745

```tablegen
def LBEPX   : XForm_1<31, 95, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                      "lbepx $RST, $addr", IIC_LdStLoad, []>,
                      Requires<[IsE500]>;

def LFDEPX  : XForm_25<31, 607, (outs f8rc:$RST), (ins (memrr $RA, $RB):$addr),
                      "lfdepx $RST, $addr", IIC_LdStLFD, []>,
                      Requires<[IsE500]>;
```
- **EN**: Adds declarative TableGen records such as `LBEPX`, `LFDEPX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LBEPX`, `LFDEPX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4746-4753

```tablegen
def LHEPX   : XForm_1<31, 287, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                      "lhepx $RST, $addr", IIC_LdStLoad, []>,
                      Requires<[IsE500]>;

def LWEPX   : XForm_1<31, 31, (outs gprc:$RST), (ins (memrr $RA, $RB):$addr),
                      "lwepx $RST, $addr", IIC_LdStLoad, []>,
                      Requires<[IsE500]>;
```
- **EN**: Adds declarative TableGen records such as `LHEPX`, `LWEPX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LHEPX`, `LWEPX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4754-4761

```tablegen
def STBEPX  : XForm_8<31, 223, (outs), (ins gprc:$RST, (memrr $RA, $RB):$addr),
                      "stbepx $RST, $addr", IIC_LdStStore, []>,
                      Requires<[IsE500]>;

def STFDEPX : XForm_28_memOp<31, 735, (outs), (ins f8rc:$RST, (memrr $RA, $RB):$addr),
                      "stfdepx $RST, $addr", IIC_LdStSTFD, []>,
                      Requires<[IsE500]>;
```
- **EN**: Adds declarative TableGen records such as `STBEPX`, `STFDEPX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STBEPX`, `STFDEPX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4762-4769

```tablegen
def STHEPX  : XForm_8<31, 415, (outs), (ins gprc:$RST, (memrr $RA, $RB):$addr),
                      "sthepx $RST, $addr", IIC_LdStStore, []>,
                      Requires<[IsE500]>;

def STWEPX  : XForm_8<31, 159, (outs), (ins gprc:$RST, (memrr $RA, $RB):$addr),
                      "stwepx $RST, $addr", IIC_LdStStore, []>,
                      Requires<[IsE500]>;
```
- **EN**: Adds declarative TableGen records such as `STHEPX`, `STWEPX` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `STHEPX`, `STWEPX`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4770-4779

```tablegen
def DCBFEP  : DCB_Form<127, 0, (outs), (ins (memrr $RA, $RB):$addr), "dcbfep $addr",
                      IIC_LdStDCBF, []>, Requires<[IsE500]>;

def DCBSTEP : DCB_Form<63, 0, (outs), (ins (memrr $RA, $RB):$addr), "dcbstep $addr",
                      IIC_LdStDCBF, []>, Requires<[IsE500]>;

def DCBTEP  : DCB_Form_hint<319, (outs), (ins (memrr $RA, $RB):$addr, u5imm:$TH),
                      "dcbtep $TH, $addr", IIC_LdStDCBF, []>,
                      Requires<[IsE500]>;
```
- **EN**: Adds declarative TableGen records such as `DCBFEP`, `DCBSTEP`, `DCBTEP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DCBFEP`, `DCBSTEP`, `DCBTEP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4780-4786

```tablegen
def DCBTSTEP : DCB_Form_hint<255, (outs), (ins (memrr $RA, $RB):$addr, u5imm:$TH),
                      "dcbtstep $TH, $addr", IIC_LdStDCBF, []>,
                      Requires<[IsE500]>;

def DCBZEP  : DCB_Form<1023, 0, (outs), (ins (memrr $RA, $RB):$addr), "dcbzep $addr",
                      IIC_LdStDCBF, []>, Requires<[IsE500]>;
```
- **EN**: Adds declarative TableGen records such as `DCBTSTEP`, `DCBZEP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DCBTSTEP`, `DCBZEP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4787-4792

```tablegen
def DCBZLEP : DCB_Form<1023, 1, (outs), (ins (memrr $RA, $RB):$addr), "dcbzlep $addr",
                      IIC_LdStDCBF, []>, Requires<[IsE500]>;

def ICBIEP  : XForm_1a<31, 991, (outs), (ins (memrr $RA, $RB):$addr), "icbiep $addr",
                      IIC_LdStICBI, []>, Requires<[IsE500]>;
```
- **EN**: Adds declarative TableGen records such as `DCBZLEP`, `ICBIEP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DCBZLEP`, `ICBIEP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4793-4799

```tablegen
//===----------------------------------------------------------------------===//
// PowerPC Assembler Instruction Aliases
//

def : InstAlias<"sc", (SC 0)>;

def : InstAlias<"sync", (SYNC 0)>, Requires<[HasSYNC]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4800-4806

```tablegen
def : InstAlias<"hwsync", (SYNC 0), 0>, Requires<[HasSYNC]>;
def : InstAlias<"msync", (SYNC 0), 0>, Requires<[HasSYNC]>;
def : InstAlias<"lwsync", (SYNC 1)>, Requires<[HasSYNC]>;
def : InstAlias<"ptesync", (SYNC 2)>, Requires<[HasSYNC]>;

def : InstAlias<"wait", (WAIT 0)>;
def : InstAlias<"waitrsv", (WAIT 1)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4807-4813

```tablegen
def : InstAlias<"waitimpl", (WAIT 2)>;

def : InstAlias<"mbar", (MBAR 0)>, Requires<[IsBookE]>;

def DCBTx   : PPCAsmPseudo<"dcbt $dst", (ins memrr:$dst)>;
def DCBTSTx : PPCAsmPseudo<"dcbtst $dst", (ins memrr:$dst)>;
```
- **EN**: Adds declarative TableGen records such as `DCBTx`, `DCBTSTx` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DCBTx`, `DCBTSTx`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4814-4821

```tablegen
def DCBTCT : PPCAsmPseudo<"dcbtct $dst, $TH", (ins memrr:$dst, u5imm:$TH)>;
def DCBTDS : PPCAsmPseudo<"dcbtds $dst, $TH", (ins memrr:$dst, u5imm:$TH)>;
def DCBTT  : PPCAsmPseudo<"dcbtt $dst", (ins memrr:$dst)>;

def DCBTSTCT : PPCAsmPseudo<"dcbtstct $dst, $TH", (ins memrr:$dst, u5imm:$TH)>;
def DCBTSTDS : PPCAsmPseudo<"dcbtstds $dst, $TH", (ins memrr:$dst, u5imm:$TH)>;
def DCBTSTT  : PPCAsmPseudo<"dcbtstt $dst", (ins memrr:$dst)>;
```
- **EN**: Adds declarative TableGen records such as `DCBTCT`, `DCBTDS`, `DCBTT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DCBTCT`, `DCBTDS`, `DCBTT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4822-4828

```tablegen
def DCBFx  : PPCAsmPseudo<"dcbf $dst", (ins memrr:$dst)>;
def DCBFL  : PPCAsmPseudo<"dcbfl $dst", (ins memrr:$dst)>;
def DCBFLP : PPCAsmPseudo<"dcbflp $dst", (ins memrr:$dst)>;

def : Pat<(int_ppc_isync),  (ISYNC)>;
def : Pat<(int_ppc_dcbfl xoaddr:$dst),
          (DCBF 1, xoaddr:$dst)>;
```
- **EN**: Adds declarative TableGen records such as `DCBFx`, `DCBFL`, `DCBFLP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DCBFx`, `DCBFL`, `DCBFLP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4829-4835

```tablegen
def : Pat<(int_ppc_dcbflp xoaddr:$dst),
          (DCBF 3, xoaddr:$dst)>;

let Predicates = [IsISA3_1] in {
  def DCBFPS  : PPCAsmPseudo<"dcbfps $dst", (ins memrr:$dst)>;
  def DCBSTPS : PPCAsmPseudo<"dcbstps $dst", (ins memrr:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4836-4842

```tablegen
  def : Pat<(int_ppc_dcbfps xoaddr:$dst),
            (DCBF 4, xoaddr:$dst)>;
  def : Pat<(int_ppc_dcbstps xoaddr:$dst),
            (DCBF 6, xoaddr:$dst)>;
}

def : InstAlias<"crset $bx", (CREQV crbitrc:$bx, crbitrc:$bx, crbitrc:$bx)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4843-4850

```tablegen
def : InstAlias<"crclr $bx", (CRXOR crbitrc:$bx, crbitrc:$bx, crbitrc:$bx)>;
def : InstAlias<"crmove $bx, $by", (CROR crbitrc:$bx, crbitrc:$by, crbitrc:$by)>;
def : InstAlias<"crnot $bx, $by", (CRNOR crbitrc:$bx, crbitrc:$by, crbitrc:$by)>;

def : InstAlias<"mftb $Rx", (MFTB gprc:$Rx, 268)>;
def : InstAlias<"mftbl $Rx", (MFTB gprc:$Rx, 268)>;
def : InstAlias<"mftbu $Rx", (MFTB gprc:$Rx, 269)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4851-4858

```tablegen
def : InstAlias<"xnop", (XORI R0, R0, 0)>;

def : InstAlias<"mtxer $Rx", (MTSPR 1, gprc:$Rx)>;
def : InstAlias<"mfxer $Rx", (MFSPR gprc:$Rx, 1)>;

//Disable this alias on AIX for now because as does not support them.
let Predicates = [ModernAs] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4859-4867

```tablegen
foreach BR = 0-7 in {
    def : InstAlias<"mfbr"#BR#" $Rx",
                    (MFDCR gprc:$Rx, !add(BR, 0x80))>,
                    Requires<[IsPPC4xx]>;
    def : InstAlias<"mtbr"#BR#" $Rx",
                    (MTDCR gprc:$Rx, !add(BR, 0x80))>,
                    Requires<[IsPPC4xx]>;
}
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 4868-4875

```tablegen
def : InstAlias<"mtmsrd $RS", (MTMSRD gprc:$RS, 0)>;
def : InstAlias<"mtmsr $RS", (MTMSR gprc:$RS, 0)>;
def : InstAlias<"mtudscr $Rx", (MTSPR 3, gprc:$Rx)>;
def : InstAlias<"mfudscr $Rx", (MFSPR gprc:$Rx, 3)>;

def : InstAlias<"mfrtcu $Rx", (MFSPR gprc:$Rx, 4)>;
def : InstAlias<"mfrtcl $Rx", (MFSPR gprc:$Rx, 5)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4876-4882

```tablegen
def : InstAlias<"mtlr $Rx", (MTSPR 8, gprc:$Rx)>;
def : InstAlias<"mflr $Rx", (MFSPR gprc:$Rx, 8)>;

def : InstAlias<"mtctr $Rx", (MTSPR 9, gprc:$Rx)>;
def : InstAlias<"mfctr $Rx", (MFSPR gprc:$Rx, 9)>;

def : InstAlias<"mtuamr $Rx", (MTSPR 13, gprc:$Rx)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4883-4890

```tablegen
def : InstAlias<"mfuamr $Rx", (MFSPR gprc:$Rx, 13)>;

def : InstAlias<"mtdscr $Rx", (MTSPR 17, gprc:$Rx)>;
def : InstAlias<"mfdscr $Rx", (MFSPR gprc:$Rx, 17)>;

def : InstAlias<"mtdsisr $Rx", (MTSPR 18, gprc:$Rx)>;
def : InstAlias<"mfdsisr $Rx", (MFSPR gprc:$Rx, 18)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4891-4897

```tablegen
def : InstAlias<"mtdar $Rx", (MTSPR 19, gprc:$Rx)>;
def : InstAlias<"mfdar $Rx", (MFSPR gprc:$Rx, 19)>;

def : InstAlias<"mtdec $Rx", (MTSPR 22, gprc:$Rx)>;
def : InstAlias<"mfdec $Rx", (MFSPR gprc:$Rx, 22)>;

def : InstAlias<"mtsdr1 $Rx", (MTSPR 25, gprc:$Rx)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4898-4905

```tablegen
def : InstAlias<"mfsdr1 $Rx", (MFSPR gprc:$Rx, 25)>;

def : InstAlias<"mtsrr0 $Rx", (MTSPR 26, gprc:$Rx)>;
def : InstAlias<"mfsrr0 $Rx", (MFSPR gprc:$Rx, 26)>;

def : InstAlias<"mtsrr1 $Rx", (MTSPR 27, gprc:$Rx)>;
def : InstAlias<"mfsrr1 $Rx", (MFSPR gprc:$Rx, 27)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4906-4912

```tablegen
def : InstAlias<"mtcfar $Rx", (MTSPR 28, gprc:$Rx)>;
def : InstAlias<"mfcfar $Rx", (MFSPR gprc:$Rx, 28)>;

def : InstAlias<"mtamr $Rx", (MTSPR 29, gprc:$Rx)>;
def : InstAlias<"mfamr $Rx", (MFSPR gprc:$Rx, 29)>;

def : InstAlias<"mtpid $Rx", (MTSPR 48, gprc:$Rx)>, Requires<[IsBookE]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4913-4919

```tablegen
def : InstAlias<"mfpid $Rx", (MFSPR gprc:$Rx, 48)>, Requires<[IsBookE]>;
def : InstAlias<"mtpidr $Rx", (MTSPR 48, gprc:$Rx)>, Requires<[IsISA3_0]>;
def : InstAlias<"mfpidr $Rx", (MFSPR gprc:$Rx, 48)>, Requires<[IsISA3_0]>;

foreach SPRG = 4-7 in {
  def : InstAlias<"mfsprg $RT, "#SPRG, (MFSPR gprc:$RT, !add(SPRG, 256))>,
                  Requires<[IsBookE]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4920-4927

```tablegen
  def : InstAlias<"mfsprg"#SPRG#" $RT", (MFSPR gprc:$RT, !add(SPRG, 256))>,
                  Requires<[IsBookE]>;
  def : InstAlias<"mtsprg "#SPRG#", $RT", (MTSPR !add(SPRG, 256), gprc:$RT)>,
                  Requires<[IsBookE]>;
  def : InstAlias<"mtsprg"#SPRG#" $RT", (MTSPR !add(SPRG, 256), gprc:$RT)>,
                  Requires<[IsBookE]>;
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4928-4934

```tablegen
foreach SPRG = 0-3 in {
  def : InstAlias<"mfsprg $RT, "#SPRG, (MFSPR gprc:$RT, !add(SPRG, 272))>;
  def : InstAlias<"mfsprg"#SPRG#" $RT", (MFSPR gprc:$RT, !add(SPRG, 272))>;
  def : InstAlias<"mtsprg "#SPRG#", $RT", (MTSPR !add(SPRG, 272), gprc:$RT)>;
  def : InstAlias<"mtsprg"#SPRG#" $RT", (MTSPR !add(SPRG, 272), gprc:$RT)>;
}
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend instruction information and lowering helpers.
- **CN**: 这里实现了 PowerPC 后端该部分指令信息与 lowering 辅助逻辑所需的辅助例程。

### Lines 4935-4942

```tablegen
def : InstAlias<"mfasr $RT", (MFSPR gprc:$RT, 280)>;
def : InstAlias<"mtasr $RT", (MTSPR 280, gprc:$RT)>;

def : InstAlias<"mttbl $Rx", (MTSPR 284, gprc:$Rx)>;
def : InstAlias<"mttbu $Rx", (MTSPR 285, gprc:$Rx)>;

def : InstAlias<"mfpvr $RT", (MFSPR gprc:$RT, 287)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4943-4949

```tablegen
def : InstAlias<"mfspefscr $Rx", (MFSPR gprc:$Rx, 512)>;
def : InstAlias<"mtspefscr $Rx", (MTSPR 512, gprc:$Rx)>;

foreach BATR = 0-3 in {
    def : InstAlias<"mtdbatu "#BATR#", $Rx",
                    (MTSPR !add(BATR, !add(BATR, 536)), gprc:$Rx)>,
                    Requires<[IsPPC6xx]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4950-4958

```tablegen
    def : InstAlias<"mfdbatu $Rx, "#BATR,
                    (MFSPR gprc:$Rx, !add(BATR, !add(BATR, 536)))>,
                    Requires<[IsPPC6xx]>;
    def : InstAlias<"mtdbatl "#BATR#", $Rx",
                    (MTSPR !add(BATR, !add(BATR, 537)), gprc:$Rx)>,
                    Requires<[IsPPC6xx]>;
    def : InstAlias<"mfdbatl $Rx, "#BATR,
                    (MFSPR gprc:$Rx, !add(BATR, !add(BATR, 537)))>,
                    Requires<[IsPPC6xx]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4959-4967

```tablegen
    def : InstAlias<"mtibatu "#BATR#", $Rx",
                    (MTSPR !add(BATR, !add(BATR, 528)), gprc:$Rx)>,
                    Requires<[IsPPC6xx]>;
    def : InstAlias<"mfibatu $Rx, "#BATR,
                    (MFSPR gprc:$Rx, !add(BATR, !add(BATR, 528)))>,
                    Requires<[IsPPC6xx]>;
    def : InstAlias<"mtibatl "#BATR#", $Rx",
                    (MTSPR !add(BATR, !add(BATR, 529)), gprc:$Rx)>,
                    Requires<[IsPPC6xx]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4968-4975

```tablegen
    def : InstAlias<"mfibatl $Rx, "#BATR,
                    (MFSPR gprc:$Rx, !add(BATR, !add(BATR, 529)))>,
                    Requires<[IsPPC6xx]>;
}

def : InstAlias<"mtppr $RT", (MTSPR 896, gprc:$RT)>;
def : InstAlias<"mfppr $RT", (MFSPR gprc:$RT, 896)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4976-4982

```tablegen
def : InstAlias<"mtesr $Rx", (MTSPR 980, gprc:$Rx)>, Requires<[IsPPC4xx]>;
def : InstAlias<"mfesr $Rx", (MFSPR gprc:$Rx, 980)>, Requires<[IsPPC4xx]>;

def : InstAlias<"mtdear $Rx", (MTSPR 981, gprc:$Rx)>, Requires<[IsPPC4xx]>;
def : InstAlias<"mfdear $Rx", (MFSPR gprc:$Rx, 981)>, Requires<[IsPPC4xx]>;

def : InstAlias<"mttcr $Rx", (MTSPR 986, gprc:$Rx)>, Requires<[IsPPC4xx]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4983-4990

```tablegen
def : InstAlias<"mftcr $Rx", (MFSPR gprc:$Rx, 986)>, Requires<[IsPPC4xx]>;

def : InstAlias<"mftbhi $Rx", (MFSPR gprc:$Rx, 988)>, Requires<[IsPPC4xx]>;
def : InstAlias<"mttbhi $Rx", (MTSPR 988, gprc:$Rx)>, Requires<[IsPPC4xx]>;

def : InstAlias<"mftblo $Rx", (MFSPR gprc:$Rx, 989)>, Requires<[IsPPC4xx]>;
def : InstAlias<"mttblo $Rx", (MTSPR 989, gprc:$Rx)>, Requires<[IsPPC4xx]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4991-4997

```tablegen
def : InstAlias<"mtsrr2 $Rx", (MTSPR 990, gprc:$Rx)>, Requires<[IsPPC4xx]>;
def : InstAlias<"mfsrr2 $Rx", (MFSPR gprc:$Rx, 990)>, Requires<[IsPPC4xx]>;

def : InstAlias<"mtsrr3 $Rx", (MTSPR 991, gprc:$Rx)>, Requires<[IsPPC4xx]>;
def : InstAlias<"mfsrr3 $Rx", (MFSPR gprc:$Rx, 991)>, Requires<[IsPPC4xx]>;

def : InstAlias<"mtdccr $Rx", (MTSPR 1018, gprc:$Rx)>, Requires<[IsPPC4xx]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4998-5004

```tablegen
def : InstAlias<"mfdccr $Rx", (MFSPR gprc:$Rx, 1018)>, Requires<[IsPPC4xx]>;

def : InstAlias<"mticcr $Rx", (MTSPR 1019, gprc:$Rx)>, Requires<[IsPPC4xx]>;
def : InstAlias<"mficcr $Rx", (MFSPR gprc:$Rx, 1019)>, Requires<[IsPPC4xx]>;

}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5005-5012

```tablegen
def : InstAlias<"tlbie $RB", (TLBIE R0, gprc:$RB)>,  Requires<[IsNotISA3_0]>;
let Predicates = [IsISA3_0] in {
  def : InstAlias<"tlbie $RB", (TLBIEP9 R0, gprc:$RB, 0, 0, 0)>;
  def : InstAlias<"tlbie $RB, $RS", (TLBIEP9 gprc:$RB, gprc:$RS, 0, 0, 0)>;
}

def : InstAlias<"tlbrehi $RS, $A", (TLBRE2 gprc:$RS, gprc:$A, 0)>,
                Requires<[IsPPC4xx]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 5013-5019

```tablegen
def : InstAlias<"tlbrelo $RS, $A", (TLBRE2 gprc:$RS, gprc:$A, 1)>,
                Requires<[IsPPC4xx]>;
def : InstAlias<"tlbwehi $RS, $A", (TLBWE2 gprc:$RS, gprc:$A, 0)>,
                Requires<[IsPPC4xx]>;
def : InstAlias<"tlbwelo $RS, $A", (TLBWE2 gprc:$RS, gprc:$A, 1)>,
                Requires<[IsPPC4xx]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5020-5027

```tablegen
def : InstAlias<"tlbilxlpid", (TLBILX 0, R0, R0)>, Requires<[IsBookE]>;
def : InstAlias<"tlbilxpid", (TLBILX 1, R0, R0)>, Requires<[IsBookE]>;
def : InstAlias<"tlbilxva $RA, $RB", (TLBILX 3, gprc:$RA, gprc:$RB)>,
                Requires<[IsBookE]>;
def : InstAlias<"tlbilxva $RB", (TLBILX 3, R0, gprc:$RB)>, Requires<[IsBookE]>;

def LAx : PPCAsmPseudo<"la $rA, $addr", (ins gprc:$rA, memri:$addr)>, MemriOp;
```
- **EN**: Adds declarative TableGen records such as `LAx` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `LAx`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5028-5036

```tablegen
def SUBI : PPCAsmPseudo<"subi $rA, $rB, $imm",
                        (ins gprc:$rA, gprc:$rB, s16imm:$imm)>;
def SUBIS : PPCAsmPseudo<"subis $rA, $rB, $imm",
                         (ins gprc:$rA, gprc:$rB, s16imm:$imm)>;
def SUBIC : PPCAsmPseudo<"subic $rA, $rB, $imm",
                         (ins gprc:$rA, gprc:$rB, s16imm:$imm)>;
def SUBIC_rec : PPCAsmPseudo<"subic. $rA, $rB, $imm",
                          (ins gprc:$rA, gprc:$rB, s16imm:$imm)>;
```
- **EN**: Adds declarative TableGen records such as `SUBI`, `SUBIS`, `SUBIC` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SUBI`, `SUBIS`, `SUBIC`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5037-5044

```tablegen
def EXTLWI : PPCAsmPseudo<"extlwi $rA, $rS, $n, $b",
                          (ins gprc:$rA, gprc:$rS, u5imm:$n, u5imm:$b)>,
                          ZExt32To64;
def EXTLWI_rec : PPCAsmPseudo<"extlwi. $rA, $rS, $n, $b",
                           (ins gprc:$rA, gprc:$rS, u5imm:$n, u5imm:$b)>,
                           ZExt32To64;
def EXTRWI : PPCAsmPseudo<"extrwi $rA, $rS, $n, $b",
                          (ins gprc:$rA, gprc:$rS, u5imm:$n, u5imm:$b)>;
```
- **EN**: Adds declarative TableGen records such as `EXTLWI`, `EXTLWI_rec`, `EXTRWI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTLWI`, `EXTLWI_rec`, `EXTRWI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5045-5052

```tablegen
def EXTRWI_rec : PPCAsmPseudo<"extrwi. $rA, $rS, $n, $b",
                           (ins gprc:$rA, gprc:$rS, u5imm:$n, u5imm:$b)>;
def INSLWI : PPCAsmPseudo<"inslwi $rA, $rS, $n, $b",
                          (ins gprc:$rA, gprc:$rS, u5imm:$n, u5imm:$b)>;
def INSLWI_rec : PPCAsmPseudo<"inslwi. $rA, $rS, $n, $b",
                           (ins gprc:$rA, gprc:$rS, u5imm:$n, u5imm:$b)>;
def INSRWI : PPCAsmPseudo<"insrwi $rA, $rS, $n, $b",
                          (ins gprc:$rA, gprc:$rS, u5imm:$n, u5imm:$b)>;
```
- **EN**: Adds declarative TableGen records such as `EXTRWI_rec`, `INSLWI`, `INSLWI_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTRWI_rec`, `INSLWI`, `INSLWI_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5053-5060

```tablegen
def INSRWI_rec : PPCAsmPseudo<"insrwi. $rA, $rS, $n, $b",
                           (ins gprc:$rA, gprc:$rS, u5imm:$n, u5imm:$b)>;
def ROTRWI : PPCAsmPseudo<"rotrwi $rA, $rS, $n",
                          (ins gprc:$rA, gprc:$rS, u5imm:$n)>, ZExt32To64;
def ROTRWI_rec : PPCAsmPseudo<"rotrwi. $rA, $rS, $n",
                           (ins gprc:$rA, gprc:$rS, u5imm:$n)>, ZExt32To64;
def SLWI : PPCAsmPseudo<"slwi $rA, $rS, $n",
                        (ins gprc:$rA, gprc:$rS, u5imm:$n)>, ZExt32To64;
```
- **EN**: Adds declarative TableGen records such as `INSRWI_rec`, `ROTRWI`, `ROTRWI_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `INSRWI_rec`, `ROTRWI`, `ROTRWI_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5061-5068

```tablegen
def SLWI_rec : PPCAsmPseudo<"slwi. $rA, $rS, $n",
                         (ins gprc:$rA, gprc:$rS, u5imm:$n)>, ZExt32To64;
def SRWI : PPCAsmPseudo<"srwi $rA, $rS, $n",
                        (ins gprc:$rA, gprc:$rS, u5imm:$n)>, ZExt32To64;
def SRWI_rec : PPCAsmPseudo<"srwi. $rA, $rS, $n",
                         (ins gprc:$rA, gprc:$rS, u5imm:$n)>, ZExt32To64;
def CLRRWI : PPCAsmPseudo<"clrrwi $rA, $rS, $n",
                          (ins gprc:$rA, gprc:$rS, u5imm:$n)>;
```
- **EN**: Adds declarative TableGen records such as `SLWI_rec`, `SRWI`, `SRWI_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SLWI_rec`, `SRWI`, `SRWI_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5069-5075

```tablegen
def CLRRWI_rec : PPCAsmPseudo<"clrrwi. $rA, $rS, $n",
                           (ins gprc:$rA, gprc:$rS, u5imm:$n)>;
def CLRLSLWI : PPCAsmPseudo<"clrlslwi $rA, $rS, $b, $n",
                            (ins gprc:$rA, gprc:$rS, u5imm:$b, u5imm:$n)>;
def CLRLSLWI_rec : PPCAsmPseudo<"clrlslwi. $rA, $rS, $b, $n",
                             (ins gprc:$rA, gprc:$rS, u5imm:$b, u5imm:$n)>;
```
- **EN**: Adds declarative TableGen records such as `CLRRWI_rec`, `CLRLSLWI`, `CLRLSLWI_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CLRRWI_rec`, `CLRLSLWI`, `CLRLSLWI_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5076-5082

```tablegen
def : InstAlias<"isellt $rT, $rA, $rB",
                (ISEL gprc:$rT, gprc_nor0:$rA, gprc:$rB, CR0LT)>;
def : InstAlias<"iselgt $rT, $rA, $rB",
                (ISEL gprc:$rT, gprc_nor0:$rA, gprc:$rB, CR0GT)>;
def : InstAlias<"iseleq $rT, $rA, $rB",
                (ISEL gprc:$rT, gprc_nor0:$rA, gprc:$rB, CR0EQ)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5083-5089

```tablegen
def : InstAlias<"rotlwi $rA, $rS, $n", (RLWINM gprc:$rA, gprc:$rS, u5imm:$n, 0, 31)>;
def : InstAlias<"rotlwi. $rA, $rS, $n", (RLWINM_rec gprc:$rA, gprc:$rS, u5imm:$n, 0, 31)>;
def : InstAlias<"rotlw $rA, $rS, $rB", (RLWNM gprc:$rA, gprc:$rS, gprc:$rB, 0, 31)>;
def : InstAlias<"rotlw. $rA, $rS, $rB", (RLWNM_rec gprc:$rA, gprc:$rS, gprc:$rB, 0, 31)>;
def : InstAlias<"clrlwi $rA, $rS, $n", (RLWINM gprc:$rA, gprc:$rS, 0, u5imm:$n, 31)>;
def : InstAlias<"clrlwi. $rA, $rS, $n", (RLWINM_rec gprc:$rA, gprc:$rS, 0, u5imm:$n, 31)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5090-5097

```tablegen
def : InstAlias<"cntlzw $rA, $rS", (CNTLZW gprc:$rA, gprc:$rS)>;
def : InstAlias<"cntlzw. $rA, $rS", (CNTLZW_rec gprc:$rA, gprc:$rS)>;
// The POWER variant
def : MnemonicAlias<"cntlz",  "cntlzw">;
def : MnemonicAlias<"cntlz.", "cntlzw.">;

def EXTLDI : PPCAsmPseudo<"extldi $rA, $rS, $n, $b",
                          (ins g8rc:$rA, g8rc:$rS, u6imm:$n, u6imm:$b)>;
```
- **EN**: Adds declarative TableGen records such as `EXTLDI` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTLDI`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5098-5105

```tablegen
def EXTLDI_rec : PPCAsmPseudo<"extldi. $rA, $rS, $n, $b",
                           (ins g8rc:$rA, g8rc:$rS, u6imm:$n, u6imm:$b)>;
def EXTRDI : PPCAsmPseudo<"extrdi $rA, $rS, $n, $b",
                          (ins g8rc:$rA, g8rc:$rS, u6imm:$n, u6imm:$b)>;
def EXTRDI_rec : PPCAsmPseudo<"extrdi. $rA, $rS, $n, $b",
                           (ins g8rc:$rA, g8rc:$rS, u6imm:$n, u6imm:$b)>;
def INSRDI : PPCAsmPseudo<"insrdi $rA, $rS, $n, $b",
                          (ins g8rc:$rA, g8rc:$rS, u6imm:$n, u6imm:$b)>;
```
- **EN**: Adds declarative TableGen records such as `EXTLDI_rec`, `EXTRDI`, `EXTRDI_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `EXTLDI_rec`, `EXTRDI`, `EXTRDI_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5106-5113

```tablegen
def INSRDI_rec : PPCAsmPseudo<"insrdi. $rA, $rS, $n, $b",
                           (ins g8rc:$rA, g8rc:$rS, u6imm:$n, u6imm:$b)>;
def ROTRDI : PPCAsmPseudo<"rotrdi $rA, $rS, $n",
                          (ins g8rc:$rA, g8rc:$rS, u6imm:$n)>;
def ROTRDI_rec : PPCAsmPseudo<"rotrdi. $rA, $rS, $n",
                           (ins g8rc:$rA, g8rc:$rS, u6imm:$n)>;
def SLDI : PPCAsmPseudo<"sldi $rA, $rS, $n",
                        (ins g8rc:$rA, g8rc:$rS, u6imm:$n)>;
```
- **EN**: Adds declarative TableGen records such as `INSRDI_rec`, `ROTRDI`, `ROTRDI_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `INSRDI_rec`, `ROTRDI`, `ROTRDI_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5114-5121

```tablegen
def SLDI_rec : PPCAsmPseudo<"sldi. $rA, $rS, $n",
                         (ins g8rc:$rA, g8rc:$rS, u6imm:$n)>;
def SRDI : PPCAsmPseudo<"srdi $rA, $rS, $n",
                        (ins g8rc:$rA, g8rc:$rS, u6imm:$n)>;
def SRDI_rec : PPCAsmPseudo<"srdi. $rA, $rS, $n",
                         (ins g8rc:$rA, g8rc:$rS, u6imm:$n)>;
def CLRRDI : PPCAsmPseudo<"clrrdi $rA, $rS, $n",
                          (ins g8rc:$rA, g8rc:$rS, u6imm:$n)>;
```
- **EN**: Adds declarative TableGen records such as `SLDI_rec`, `SRDI`, `SRDI_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SLDI_rec`, `SRDI`, `SRDI_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5122-5129

```tablegen
def CLRRDI_rec : PPCAsmPseudo<"clrrdi. $rA, $rS, $n",
                           (ins g8rc:$rA, g8rc:$rS, u6imm:$n)>;
def CLRLSLDI : PPCAsmPseudo<"clrlsldi $rA, $rS, $b, $n",
                            (ins g8rc:$rA, g8rc:$rS, u6imm:$b, u6imm:$n)>;
def CLRLSLDI_rec : PPCAsmPseudo<"clrlsldi. $rA, $rS, $b, $n",
                             (ins g8rc:$rA, g8rc:$rS, u6imm:$b, u6imm:$n)>;
def SUBPCIS : PPCAsmPseudo<"subpcis $RT, $D", (ins g8rc:$RT, s16imm:$D)>;
```
- **EN**: Adds declarative TableGen records such as `CLRRDI_rec`, `CLRLSLDI`, `CLRLSLDI_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CLRRDI_rec`, `CLRLSLDI`, `CLRLSLDI_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5130-5136

```tablegen
def : InstAlias<"rotldi $rA, $rS, $n", (RLDICL g8rc:$rA, g8rc:$rS, u6imm:$n, 0)>;
def : InstAlias<"rotldi $rA, $rS, $n",
                (RLDICL_32_64 g8rc:$rA, gprc:$rS, u6imm:$n, 0)>;
def : InstAlias<"rotldi. $rA, $rS, $n", (RLDICL_rec g8rc:$rA, g8rc:$rS, u6imm:$n, 0)>;
def : InstAlias<"rotld $rA, $rS, $rB", (RLDCL g8rc:$rA, g8rc:$rS, gprc:$rB, 0)>;
def : InstAlias<"rotld. $rA, $rS, $rB", (RLDCL_rec g8rc:$rA, g8rc:$rS, gprc:$rB, 0)>;
def : InstAlias<"clrldi $rA, $rS, $n", (RLDICL g8rc:$rA, g8rc:$rS, 0, u6imm:$n)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5137-5143

```tablegen
def : InstAlias<"clrldi $rA, $rS, $n",
                (RLDICL_32_64 g8rc:$rA, gprc:$rS, 0, u6imm:$n)>;
def : InstAlias<"clrldi. $rA, $rS, $n", (RLDICL_rec g8rc:$rA, g8rc:$rS, 0, u6imm:$n)>;
def : InstAlias<"lnia $RT", (ADDPCIS g8rc:$RT, 0)>;

def RLWINMbm : PPCAsmPseudo<"rlwinm $rA, $rS, $n, $b",
                            (ins g8rc:$rA, g8rc:$rS, u5imm:$n, i32imm:$b)>;
```
- **EN**: Adds declarative TableGen records such as `RLWINMbm` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RLWINMbm`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5144-5151

```tablegen
def RLWINMbm_rec : PPCAsmPseudo<"rlwinm. $rA, $rS, $n, $b",
                            (ins g8rc:$rA, g8rc:$rS, u5imm:$n, i32imm:$b)>;
def RLWIMIbm : PPCAsmPseudo<"rlwimi $rA, $rS, $n, $b",
                           (ins g8rc:$rA, g8rc:$rS, u5imm:$n, i32imm:$b)>;
def RLWIMIbm_rec : PPCAsmPseudo<"rlwimi. $rA, $rS, $n, $b",
                            (ins g8rc:$rA, g8rc:$rS, u5imm:$n, i32imm:$b)>;
def RLWNMbm : PPCAsmPseudo<"rlwnm $rA, $rS, $n, $b",
                          (ins g8rc:$rA, g8rc:$rS, u5imm:$n, i32imm:$b)>;
```
- **EN**: Adds declarative TableGen records such as `RLWINMbm_rec`, `RLWIMIbm`, `RLWIMIbm_rec` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RLWINMbm_rec`, `RLWIMIbm`, `RLWIMIbm_rec`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5152-5159

```tablegen
def RLWNMbm_rec : PPCAsmPseudo<"rlwnm. $rA, $rS, $n, $b",
                           (ins g8rc:$rA, g8rc:$rS, u5imm:$n, i32imm:$b)>;
def PPCLdFixedAddr :
  PPCPostRAExpPseudo<(outs gprc:$rT), (ins i32imm:$imm), "#FA_LOAD",
                     [(set i32:$rT, (int_ppc_fixed_addr_ld timm:$imm))]>;

// These generic branch instruction forms are used for the assembler parser only.
// Defs and Uses are conservative, since we don't know the BO value.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "These generic branch instruction forms are used for the assembler parser only.". Notable symbols in this range include `RLWNMbm_rec`, `PPCLdFixedAddr`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“These generic branch instruction forms are used for the assembler parser only.”。 该区间中较显眼的符号包括 `RLWNMbm_rec`, `PPCLdFixedAddr`。

### Lines 5160-5167

```tablegen
let PPC970_Unit = 7, isBranch = 1, hasSideEffects = 0 in {
  let Defs = [CTR], Uses = [CTR, RM] in {
    def gBC : BForm_3<16, 0, 0, (outs),
                      (ins u5imm:$BO, crbitrc:$BI, condbrtarget:$BD),
                      "bc $BO, $BI, $BD">;
    def gBCA : BForm_3<16, 1, 0, (outs),
                       (ins u5imm:$BO, crbitrc:$BI, abscondbrtarget:$BD),
                       "bca $BO, $BI, $BD">;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5168-5178

```tablegen
    let isAsmParserOnly = 1 in {
      def gBCat : BForm_3_at<16, 0, 0, (outs),
                             (ins u5imm:$BO, atimm:$at, crbitrc:$BI,
                                  condbrtarget:$BD),
                                  "bc$at $BO, $BI, $BD">;
      def gBCAat : BForm_3_at<16, 1, 0, (outs),
                              (ins u5imm:$BO, atimm:$at, crbitrc:$BI,
                                   abscondbrtarget:$BD),
                                   "bca$at $BO, $BI, $BD">;
    } // isAsmParserOnly = 1
  }
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 5179-5185

```tablegen
  let Defs = [LR, CTR], Uses = [CTR, RM] in {
    def gBCL : BForm_3<16, 0, 1, (outs),
                       (ins u5imm:$BO, crbitrc:$BI, condbrtarget:$BD),
                       "bcl $BO, $BI, $BD">;
    def gBCLA : BForm_3<16, 1, 1, (outs),
                        (ins u5imm:$BO, crbitrc:$BI, abscondbrtarget:$BD),
                        "bcla $BO, $BI, $BD">;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 5186-5196

```tablegen
    let isAsmParserOnly = 1 in {
      def gBCLat : BForm_3_at<16, 0, 1, (outs),
                         (ins u5imm:$BO, atimm:$at, crbitrc:$BI,
                              condbrtarget:$BD),
                              "bcl$at $BO, $BI, $BD">;
      def gBCLAat : BForm_3_at<16, 1, 1, (outs),
                          (ins u5imm:$BO, atimm:$at, crbitrc:$BI,
                               abscondbrtarget:$BD),
                               "bcla$at $BO, $BI, $BD">;
    } // // isAsmParserOnly = 1
  }
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 5197-5204

```tablegen
  let Defs = [CTR], Uses = [CTR, LR, RM] in
    def gBCLR : XLForm_2<19, 16, 0, (outs),
                         (ins u5imm:$BO, crbitrc:$BI, i32imm:$BH),
                         "bclr $BO, $BI, $BH", IIC_BrB, []>;
  let Defs = [LR, CTR], Uses = [CTR, LR, RM] in
    def gBCLRL : XLForm_2<19, 16, 1, (outs),
                          (ins u5imm:$BO, crbitrc:$BI, i32imm:$BH),
                          "bclrl $BO, $BI, $BH", IIC_BrB, []>;
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 5205-5214

```tablegen
  let Defs = [CTR], Uses = [CTR, LR, RM] in
    def gBCCTR : XLForm_2<19, 528, 0, (outs),
                          (ins u5imm:$BO, crbitrc:$BI, i32imm:$BH),
                          "bcctr $BO, $BI, $BH", IIC_BrB, []>;
  let Defs = [LR, CTR], Uses = [CTR, LR, RM] in
    def gBCCTRL : XLForm_2<19, 528, 1, (outs),
                           (ins u5imm:$BO, crbitrc:$BI, i32imm:$BH),
                           "bcctrl $BO, $BI, $BH", IIC_BrB, []>;
}
```
- **EN**: Continues the PowerPC backend instruction information and lowering helpers with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的指令信息与 lowering 辅助逻辑，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 5215-5221

```tablegen
multiclass BranchSimpleMnemonicAT<string pm, int at> {
  def : InstAlias<"bc"#pm#" $bo, $bi, $dst", (gBCat u5imm:$bo, at, crbitrc:$bi,
                                                    condbrtarget:$dst)>;
  def : InstAlias<"bca"#pm#" $bo, $bi, $dst", (gBCAat u5imm:$bo, at, crbitrc:$bi,
                                                      condbrtarget:$dst)>;
  def : InstAlias<"bcl"#pm#" $bo, $bi, $dst", (gBCLat u5imm:$bo, at, crbitrc:$bi,
                                                      condbrtarget:$dst)>;
```
- **EN**: Adds declarative TableGen records such as `BranchSimpleMnemonicAT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BranchSimpleMnemonicAT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5222-5228

```tablegen
  def : InstAlias<"bcla"#pm#" $bo, $bi, $dst", (gBCLAat u5imm:$bo, at, crbitrc:$bi,
                                                        condbrtarget:$dst)>;
}
defm : BranchSimpleMnemonicAT<"+", 3>;
defm : BranchSimpleMnemonicAT<"-", 2>;

def : InstAlias<"bclr $bo, $bi", (gBCLR u5imm:$bo, crbitrc:$bi, 0)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5229-5235

```tablegen
def : InstAlias<"bclrl $bo, $bi", (gBCLRL u5imm:$bo, crbitrc:$bi, 0)>;
def : InstAlias<"bcctr $bo, $bi", (gBCCTR u5imm:$bo, crbitrc:$bi, 0)>;
def : InstAlias<"bcctrl $bo, $bi", (gBCCTRL u5imm:$bo, crbitrc:$bi, 0)>;

multiclass BranchSimpleMnemonic1<string name, string pm, int bo> {
  def : InstAlias<"b"#name#pm#" $bi, $dst", (gBC bo, crbitrc:$bi, condbrtarget:$dst)>;
  def : InstAlias<"b"#name#"a"#pm#" $bi, $dst", (gBCA bo, crbitrc:$bi, abscondbrtarget:$dst)>;
```
- **EN**: Adds declarative TableGen records such as `BranchSimpleMnemonic1` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BranchSimpleMnemonic1`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5236-5242

```tablegen
  def : InstAlias<"b"#name#"lr"#pm#" $bi", (gBCLR bo, crbitrc:$bi, 0)>;
  def : InstAlias<"b"#name#"l"#pm#" $bi, $dst", (gBCL bo, crbitrc:$bi, condbrtarget:$dst)>;
  def : InstAlias<"b"#name#"la"#pm#" $bi, $dst", (gBCLA bo, crbitrc:$bi, abscondbrtarget:$dst)>;
  def : InstAlias<"b"#name#"lrl"#pm#" $bi", (gBCLRL bo, crbitrc:$bi, 0)>;
}
multiclass BranchSimpleMnemonic2<string name, string pm, int bo>
  : BranchSimpleMnemonic1<name, pm, bo> {
```
- **EN**: Adds declarative TableGen records such as `BranchSimpleMnemonic2` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BranchSimpleMnemonic2`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5243-5249

```tablegen
  def : InstAlias<"b"#name#"ctr"#pm#" $bi", (gBCCTR bo, crbitrc:$bi, 0)>;
  def : InstAlias<"b"#name#"ctrl"#pm#" $bi", (gBCCTRL bo, crbitrc:$bi, 0)>;
}
defm : BranchSimpleMnemonic2<"t", "", 12>;
defm : BranchSimpleMnemonic2<"f", "", 4>;
defm : BranchSimpleMnemonic2<"t", "-", 14>;
defm : BranchSimpleMnemonic2<"f", "-", 6>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5250-5256

```tablegen
defm : BranchSimpleMnemonic2<"t", "+", 15>;
defm : BranchSimpleMnemonic2<"f", "+", 7>;
defm : BranchSimpleMnemonic1<"dnzt", "", 8>;
defm : BranchSimpleMnemonic1<"dnzf", "", 0>;
defm : BranchSimpleMnemonic1<"dzt", "", 10>;
defm : BranchSimpleMnemonic1<"dzf", "", 2>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5257-5264

```tablegen
multiclass BranchExtendedMnemonicPM<string name, string pm, int bibo> {
  def : InstAlias<"b"#name#pm#" $cc, $dst",
                  (BCC (pred bibo, crrc:$cc), condbrtarget:$dst)>;
  def : InstAlias<"b"#name#pm#" $dst",
                  (BCC (pred bibo, CR0), condbrtarget:$dst)>;

  def : InstAlias<"b"#name#"a"#pm#" $cc, $dst",
                  (BCCA (pred bibo, crrc:$cc), abscondbrtarget:$dst)>;
```
- **EN**: Adds declarative TableGen records such as `BranchExtendedMnemonicPM` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BranchExtendedMnemonicPM`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5265-5272

```tablegen
  def : InstAlias<"b"#name#"a"#pm#" $dst",
                  (BCCA (pred bibo, CR0), abscondbrtarget:$dst)>;

  def : InstAlias<"b"#name#"lr"#pm#" $cc",
                  (BCCLR (pred bibo, crrc:$cc))>;
  def : InstAlias<"b"#name#"lr"#pm,
                  (BCCLR (pred bibo, CR0))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 5273-5279

```tablegen
  def : InstAlias<"b"#name#"ctr"#pm#" $cc",
                  (BCCCTR (pred bibo, crrc:$cc))>;
  def : InstAlias<"b"#name#"ctr"#pm,
                  (BCCCTR (pred bibo, CR0))>;

  def : InstAlias<"b"#name#"l"#pm#" $cc, $dst",
                  (BCCL (pred bibo, crrc:$cc), condbrtarget:$dst)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 5280-5287

```tablegen
  def : InstAlias<"b"#name#"l"#pm#" $dst",
                  (BCCL (pred bibo, CR0), condbrtarget:$dst)>;

  def : InstAlias<"b"#name#"la"#pm#" $cc, $dst",
                  (BCCLA (pred bibo, crrc:$cc), abscondbrtarget:$dst)>;
  def : InstAlias<"b"#name#"la"#pm#" $dst",
                  (BCCLA (pred bibo, CR0), abscondbrtarget:$dst)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 5288-5294

```tablegen
  def : InstAlias<"b"#name#"lrl"#pm#" $cc",
                  (BCCLRL (pred bibo, crrc:$cc))>;
  def : InstAlias<"b"#name#"lrl"#pm,
                  (BCCLRL (pred bibo, CR0))>;

  def : InstAlias<"b"#name#"ctrl"#pm#" $cc",
                  (BCCCTRL (pred bibo, crrc:$cc))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 5295-5302

```tablegen
  def : InstAlias<"b"#name#"ctrl"#pm,
                  (BCCCTRL (pred bibo, CR0))>;
}
multiclass BranchExtendedMnemonic<string name, int bibo> {
  defm : BranchExtendedMnemonicPM<name, "", bibo>;
  defm : BranchExtendedMnemonicPM<name, "-", !add(bibo, 2)>;
  defm : BranchExtendedMnemonicPM<name, "+", !add(bibo, 3)>;
}
```
- **EN**: Adds declarative TableGen records such as `BranchExtendedMnemonic` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `BranchExtendedMnemonic`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5303-5309

```tablegen
defm : BranchExtendedMnemonic<"lt", 12>;
defm : BranchExtendedMnemonic<"gt", 44>;
defm : BranchExtendedMnemonic<"eq", 76>;
defm : BranchExtendedMnemonic<"un", 108>;
defm : BranchExtendedMnemonic<"so", 108>;
defm : BranchExtendedMnemonic<"ge", 4>;
defm : BranchExtendedMnemonic<"nl", 4>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5310-5316

```tablegen
defm : BranchExtendedMnemonic<"le", 36>;
defm : BranchExtendedMnemonic<"ng", 36>;
defm : BranchExtendedMnemonic<"ne", 68>;
defm : BranchExtendedMnemonic<"nu", 100>;
defm : BranchExtendedMnemonic<"ns", 100>;

def : InstAlias<"cmpwi $rA, $imm", (CMPWI CR0, gprc:$rA, s16imm:$imm)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5317-5324

```tablegen
def : InstAlias<"cmpw $rA, $rB", (CMPW CR0, gprc:$rA, gprc:$rB)>;
def : InstAlias<"cmplwi $rA, $imm", (CMPLWI CR0, gprc:$rA, u16imm:$imm)>;
def : InstAlias<"cmplw $rA, $rB", (CMPLW CR0, gprc:$rA, gprc:$rB)>;
def : InstAlias<"cmpdi $rA, $imm", (CMPDI CR0, g8rc:$rA, s16imm64:$imm)>;
def : InstAlias<"cmpd $rA, $rB", (CMPD CR0, g8rc:$rA, g8rc:$rB)>;
def : InstAlias<"cmpldi $rA, $imm", (CMPLDI CR0, g8rc:$rA, u16imm64:$imm)>;
def : InstAlias<"cmpld $rA, $rB", (CMPLD CR0, g8rc:$rA, g8rc:$rB)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5325-5331

```tablegen
def : InstAlias<"cmpi $bf, 0, $rA, $imm", (CMPWI crrc:$bf, gprc:$rA, s16imm:$imm)>;
def : InstAlias<"cmp $bf, 0, $rA, $rB", (CMPW crrc:$bf, gprc:$rA, gprc:$rB)>;
def : InstAlias<"cmpli $bf, 0, $rA, $imm", (CMPLWI crrc:$bf, gprc:$rA, u16imm:$imm)>;
def : InstAlias<"cmpl $bf, 0, $rA, $rB", (CMPLW crrc:$bf, gprc:$rA, gprc:$rB)>;
def : InstAlias<"cmpi $bf, 1, $rA, $imm", (CMPDI crrc:$bf, g8rc:$rA, s16imm64:$imm)>;
def : InstAlias<"cmp $bf, 1, $rA, $rB", (CMPD crrc:$bf, g8rc:$rA, g8rc:$rB)>;
def : InstAlias<"cmpli $bf, 1, $rA, $imm", (CMPLDI crrc:$bf, g8rc:$rA, u16imm64:$imm)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5332-5338

```tablegen
def : InstAlias<"cmpl $bf, 1, $rA, $rB", (CMPLD crrc:$bf, g8rc:$rA, g8rc:$rB)>;

def : InstAlias<"trap", (TW 31, R0, R0)>;

multiclass TrapExtendedMnemonic<string name, int to> {
  def : InstAlias<"td"#name#"i $rA, $imm", (TDI to, g8rc:$rA, s16imm:$imm)>;
  def : InstAlias<"td"#name#" $rA, $rB", (TD to, g8rc:$rA, g8rc:$rB)>;
```
- **EN**: Adds declarative TableGen records such as `TrapExtendedMnemonic` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `TrapExtendedMnemonic`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5339-5345

```tablegen
  def : InstAlias<"tw"#name#"i $rA, $imm", (TWI to, gprc:$rA, s16imm:$imm)>;
  def : InstAlias<"tw"#name#" $rA, $rB", (TW to, gprc:$rA, gprc:$rB)>;
}
defm : TrapExtendedMnemonic<"lt", 16>;
defm : TrapExtendedMnemonic<"le", 20>;
defm : TrapExtendedMnemonic<"eq", 4>;
defm : TrapExtendedMnemonic<"ge", 12>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5346-5352

```tablegen
defm : TrapExtendedMnemonic<"gt", 8>;
defm : TrapExtendedMnemonic<"nl", 12>;
defm : TrapExtendedMnemonic<"ne", 24>;
defm : TrapExtendedMnemonic<"ng", 20>;
defm : TrapExtendedMnemonic<"llt", 2>;
defm : TrapExtendedMnemonic<"lle", 6>;
defm : TrapExtendedMnemonic<"lge", 5>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5353-5359

```tablegen
defm : TrapExtendedMnemonic<"lgt", 1>;
defm : TrapExtendedMnemonic<"lnl", 5>;
defm : TrapExtendedMnemonic<"lng", 6>;
defm : TrapExtendedMnemonic<"u", 31>;

// Atomic loads
def : Pat<(i32 (atomic_load_azext_8  DForm:$src)), (LBZ  memri:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5360-5366

```tablegen
def : Pat<(i32 (atomic_load_azext_16 DForm:$src)), (LHZ  memri:$src)>;
def : Pat<(i32 (atomic_load_nonext_32 DForm:$src)), (LWZ  memri:$src)>;
def : Pat<(i32 (atomic_load_azext_8  XForm:$src)), (LBZX memrr:$src)>;
def : Pat<(i32 (atomic_load_azext_16 XForm:$src)), (LHZX memrr:$src)>;
def : Pat<(i32 (atomic_load_nonext_32 XForm:$src)), (LWZX memrr:$src)>;

// Atomic stores
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5367-5373

```tablegen
def : Pat<(atomic_store_8  i32:$val, DForm:$ptr), (STB  gprc:$val, memri:$ptr)>;
def : Pat<(atomic_store_16 i32:$val, DForm:$ptr), (STH  gprc:$val, memri:$ptr)>;
def : Pat<(atomic_store_32 i32:$val, DForm:$ptr), (STW  gprc:$val, memri:$ptr)>;
def : Pat<(atomic_store_8  i32:$val, XForm:$ptr), (STBX gprc:$val, memrr:$ptr)>;
def : Pat<(atomic_store_16 i32:$val, XForm:$ptr), (STHX gprc:$val, memrr:$ptr)>;
def : Pat<(atomic_store_32 i32:$val, XForm:$ptr), (STWX gprc:$val, memrr:$ptr)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5374-5381

```tablegen
let Predicates = [IsISA3_0] in {

// Copy-Paste Facility
// We prefix 'CP' to COPY due to name conflict in Target.td. We also prefix to
// PASTE for naming consistency.
let mayLoad = 1 in
def CP_COPY   : X_RA5_RB5<31, 774, "copy"  , gprc, IIC_LdStCOPY, []>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Copy-Paste Facility". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Copy-Paste Facility”。 这些声明会进入生成式模式匹配逻辑。

### Lines 5382-5388

```tablegen
let mayStore = 1, Defs = [CR0] in
def CP_PASTE_rec : X_L1_RA5_RB5<31, 902, "paste.", gprc, IIC_LdStPASTE, []>, isRecordForm;

def : InstAlias<"paste. $RA, $RB", (CP_PASTE_rec gprc:$RA, gprc:$RB, 1)>;
def CP_ABORT : XForm_0<31, 838, (outs), (ins), "cpabort", IIC_SprABORT, []>;

// Message Synchronize
```
- **EN**: Adds declarative TableGen records such as `CP_PASTE_rec`, `CP_ABORT` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `CP_PASTE_rec`, `CP_ABORT`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5389-5397

```tablegen
def MSGSYNC : XForm_0<31, 886, (outs), (ins), "msgsync", IIC_SprMSGSYNC, []>;

// Power-Saving Mode Instruction:
def STOP : XForm_0<19, 370, (outs), (ins), "stop", IIC_SprSTOP, []>;

def SETB : XForm_44<31, 128, (outs gprc:$RT), (ins crrc:$BFA),
                       "setb $RT, $BFA", IIC_IntGeneral>, SExt32To64;
} // IsISA3_0
```
- **EN**: Adds declarative TableGen records such as `MSGSYNC`, `STOP`, `SETB` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MSGSYNC`, `STOP`, `SETB`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5398-5405

```tablegen
let Predicates = [IsISA3_0] in {
def : Pat<(i32 (int_ppc_cmprb i32:$a, gprc:$b, gprc:$c)),
          (i32 (SETB (CMPRB u1imm:$a, $b, $c)))>;
}
def : Pat<(i32 (int_ppc_mulhw gprc:$a, gprc:$b)),
          (i32 (MULHW $a, $b))>;
def : Pat<(i32 (int_ppc_mulhwu gprc:$a, gprc:$b)),
          (i32 (MULHWU $a, $b))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 5406-5412

```tablegen
def : Pat<(i32 (int_ppc_cmpb gprc:$a, gprc:$b)),
          (i32 (CMPB $a, $b))>;

def : Pat<(int_ppc_load2r ForceXForm:$ptr),
          (LHBRX ForceXForm:$ptr)>;
def : Pat<(int_ppc_load4r ForceXForm:$ptr),
          (LWBRX ForceXForm:$ptr)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5413-5431

```tablegen
def : Pat<(int_ppc_store2r gprc:$a, ForceXForm:$ptr),
          (STHBRX gprc:$a, ForceXForm:$ptr)>;
def : Pat<(int_ppc_store4r gprc:$a, ForceXForm:$ptr),
          (STWBRX gprc:$a, ForceXForm:$ptr)>;

// Fast 32-bit reverse bits algorithm:
// Step 1: 1-bit swap (swap odd 1-bit and even 1-bit):
// n = ((n >> 1) & 0x55555555) | ((n << 1) & 0xAAAAAAAA);
// Step 2: 2-bit swap (swap odd 2-bit and even 2-bit):
// n = ((n >> 2) & 0x33333333) | ((n << 2) & 0xCCCCCCCC);
// Step 3: 4-bit swap (swap odd 4-bit and even 4-bit):
// n = ((n >> 4) & 0x0F0F0F0F) | ((n << 4) & 0xF0F0F0F0);
// Step 4: byte reverse (Suppose n = [B1,B2,B3,B4]):
// Step 4.1: Put B4,B2 in the right position (rotate left 3 bytes):
// n' = (n rotl 24);  After which n' = [B4, B1, B2, B3]
// Step 4.2: Insert B3 to the right position:
// n' = rlwimi n', n, 8, 8, 15;  After which n' = [B4, B3, B2, B3]
// Step 4.3: Insert B1 to the right position:
// n' = rlwimi n', n, 8, 24, 31;  After which n' = [B4, B3, B2, B1]
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Fast 32-bit reverse bits algorithm:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Fast 32-bit reverse bits algorithm:”。

### Lines 5432-5440

```tablegen
def MaskValues {
  dag Lo1 = (ORI (LIS 0x5555), 0x5555);
  dag Hi1 = (ORI (LIS 0xAAAA), 0xAAAA);
  dag Lo2 = (ORI (LIS 0x3333), 0x3333);
  dag Hi2 = (ORI (LIS 0xCCCC), 0xCCCC);
  dag Lo4 = (ORI (LIS 0x0F0F), 0x0F0F);
  dag Hi4 = (ORI (LIS 0xF0F0), 0xF0F0);
}
```
- **EN**: Adds declarative TableGen records such as `MaskValues` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MaskValues`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5441-5450

```tablegen
def Shift1 {
  dag Right = (RLWINM $A, 31, 1, 31);
  dag Left = (RLWINM $A, 1, 0, 30);
}

def Swap1 {
  dag Bit = (OR (AND Shift1.Right, MaskValues.Lo1),
   (AND Shift1.Left, MaskValues.Hi1));
}
```
- **EN**: Adds declarative TableGen records such as `Shift1`, `Swap1` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Shift1`, `Swap1`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5451-5460

```tablegen
def Shift2 {
  dag Right = (RLWINM Swap1.Bit, 30, 2, 31);
  dag Left = (RLWINM Swap1.Bit, 2, 0, 29);
}

def Swap2 {
  dag Bits = (OR (AND Shift2.Right, MaskValues.Lo2),
                 (AND Shift2.Left, MaskValues.Hi2));
}
```
- **EN**: Adds declarative TableGen records such as `Shift2`, `Swap2` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Shift2`, `Swap2`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5461-5470

```tablegen
def Shift4 {
  dag Right = (RLWINM Swap2.Bits, 28, 4, 31);
  dag Left = (RLWINM Swap2.Bits, 4, 0, 27);
}

def Swap4 {
  dag Bits = (OR (AND Shift4.Right, MaskValues.Lo4),
                 (AND Shift4.Left, MaskValues.Hi4));
}
```
- **EN**: Adds declarative TableGen records such as `Shift4`, `Swap4` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Shift4`, `Swap4`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5471-5478

```tablegen
def Rotate {
  dag Left3Bytes = (RLWINM Swap4.Bits, 24, 0, 31);
}

def RotateInsertByte3 {
  dag Left = (RLWIMI Rotate.Left3Bytes, Swap4.Bits, 8, 8, 15);
}
```
- **EN**: Adds declarative TableGen records such as `Rotate`, `RotateInsertByte3` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Rotate`, `RotateInsertByte3`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5479-5485

```tablegen
def RotateInsertByte1 {
  dag Left = (RLWIMI RotateInsertByte3.Left, Swap4.Bits, 8, 24, 31);
}

// Clear the upper half of the register when in 64-bit mode
let Predicates = [IsPPC64] in
def : Pat<(i32 (bitreverse i32:$A)), (RLDICL_32 RotateInsertByte1.Left, 0, 32)>;
```
- **EN**: Adds declarative TableGen records such as `RotateInsertByte1` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `RotateInsertByte1`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 5486-5499

```tablegen
let Predicates = [IsPPC32] in
def : Pat<(i32 (bitreverse i32:$A)), RotateInsertByte1.Left>;

// Fast 64-bit reverse bits algorithm:
// Step 1: 1-bit swap (swap odd 1-bit and even 1-bit):
// n = ((n >> 1) & 0x5555555555555555) | ((n << 1) & 0xAAAAAAAAAAAAAAAA);
// Step 2: 2-bit swap (swap odd 2-bit and even 2-bit):
// n = ((n >> 2) & 0x3333333333333333) | ((n << 2) & 0xCCCCCCCCCCCCCCCC);
// Step 3: 4-bit swap (swap odd 4-bit and even 4-bit):
// n = ((n >> 4) & 0x0F0F0F0F0F0F0F0F) | ((n << 4) & 0xF0F0F0F0F0F0F0F0);
// Step 4: byte reverse (Suppose n = [B0,B1,B2,B3,B4,B5,B6,B7]):
// Apply the same byte reverse algorithm mentioned above for the fast 32-bit
// reverse to both the high 32 bit and low 32 bit of the 64 bit value. And
// then OR them together to get the final result.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Fast 64-bit reverse bits algorithm:". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Fast 64-bit reverse bits algorithm:”。 这些声明会进入生成式模式匹配逻辑。

### Lines 5500-5508

```tablegen
def MaskValues64 {
  dag Lo1 = (i64 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), MaskValues.Lo1, sub_32));
  dag Hi1 = (i64 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), MaskValues.Hi1, sub_32));
  dag Lo2 = (i64 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), MaskValues.Lo2, sub_32));
  dag Hi2 = (i64 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), MaskValues.Hi2, sub_32));
  dag Lo4 = (i64 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), MaskValues.Lo4, sub_32));
  dag Hi4 = (i64 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), MaskValues.Hi4, sub_32));
}
```
- **EN**: Adds declarative TableGen records such as `MaskValues64` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MaskValues64`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5509-5517

```tablegen
def DWMaskValues {
  dag Lo1 = (ORI8 (ORIS8 (RLDICR MaskValues64.Lo1, 32, 31), 0x5555), 0x5555);
  dag Hi1 = (ORI8 (ORIS8 (RLDICR MaskValues64.Hi1, 32, 31), 0xAAAA), 0xAAAA);
  dag Lo2 = (ORI8 (ORIS8 (RLDICR MaskValues64.Lo2, 32, 31), 0x3333), 0x3333);
  dag Hi2 = (ORI8 (ORIS8 (RLDICR MaskValues64.Hi2, 32, 31), 0xCCCC), 0xCCCC);
  dag Lo4 = (ORI8 (ORIS8 (RLDICR MaskValues64.Lo4, 32, 31), 0x0F0F), 0x0F0F);
  dag Hi4 = (ORI8 (ORIS8 (RLDICR MaskValues64.Hi4, 32, 31), 0xF0F0), 0xF0F0);
}
```
- **EN**: Adds declarative TableGen records such as `DWMaskValues` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DWMaskValues`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5518-5527

```tablegen
def DWSwapInByte {
  dag Swap1 = (OR8 (AND8 (RLDICL $A, 63, 1), DWMaskValues.Lo1),
                   (AND8 (RLDICR $A, 1, 62), DWMaskValues.Hi1));
  dag Swap2 = (OR8 (AND8 (RLDICL Swap1, 62, 2), DWMaskValues.Lo2),
                   (AND8 (RLDICR Swap1, 2, 61), DWMaskValues.Hi2));
  dag Swap4 = (OR8 (AND8 (RLDICL Swap2, 60, 4), DWMaskValues.Lo4),
                   (AND8 (RLDICR Swap2, 4, 59), DWMaskValues.Hi4));
}

// Intra-byte swap is done, now start inter-byte swap.
```
- **EN**: Adds declarative TableGen records such as `DWSwapInByte` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DWSwapInByte`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5528-5535

```tablegen
def DWBytes4567 {
  dag Word = (i32 (EXTRACT_SUBREG DWSwapInByte.Swap4, sub_32));
}

def DWBytes7456 {
  dag Word = (RLWINM DWBytes4567.Word, 24, 0, 31);
}
```
- **EN**: Adds declarative TableGen records such as `DWBytes4567`, `DWBytes7456` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DWBytes4567`, `DWBytes7456`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5536-5546

```tablegen
def DWBytes7656 {
  dag Word = (RLWIMI DWBytes7456.Word, DWBytes4567.Word, 8, 8, 15);
}

// B7 B6 B5 B4 in the right order
def DWBytes7654 {
  dag Word = (RLWIMI DWBytes7656.Word, DWBytes4567.Word, 8, 24, 31);
  dag DWord =
    (i64 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), Word, sub_32));
}
```
- **EN**: Adds declarative TableGen records such as `DWBytes7656`, `DWBytes7654` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DWBytes7656`, `DWBytes7654`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5547-5554

```tablegen
def DWBytes0123 {
  dag Word = (i32 (EXTRACT_SUBREG (RLDICL DWSwapInByte.Swap4, 32, 32), sub_32));
}

def DWBytes3012 {
  dag Word = (RLWINM DWBytes0123.Word, 24, 0, 31);
}
```
- **EN**: Adds declarative TableGen records such as `DWBytes0123`, `DWBytes3012` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DWBytes0123`, `DWBytes3012`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5555-5567

```tablegen
def DWBytes3212 {
  dag Word = (RLWIMI DWBytes3012.Word, DWBytes0123.Word, 8, 8, 15);
}

// B3 B2 B1 B0 in the right order
def DWBytes3210 {
  dag Word = (RLWIMI DWBytes3212.Word, DWBytes0123.Word, 8, 24, 31);
  dag DWord =
    (i64 (INSERT_SUBREG (i64 (IMPLICIT_DEF)), Word, sub_32));
}

// These instructions store a hash computed from the value of the link register
// and the value of the stack pointer.
```
- **EN**: Adds declarative TableGen records such as `DWBytes3212`, `DWBytes3210` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DWBytes3212`, `DWBytes3210`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5568-5580

```tablegen
let mayStore = 1 in {
def HASHST : XForm_XD6_RA5_RB5<31, 722, (outs),
                               (ins gprc:$RB, (memrihash $D, $RA):$addr),
                               "hashst $RB, $addr", IIC_IntGeneral, []>;
def HASHSTP : XForm_XD6_RA5_RB5<31, 658, (outs),
                                (ins gprc:$RB, (memrihash $D, $RA):$addr),
                                "hashstp $RB, $addr", IIC_IntGeneral, []>;
}

// These instructions check a hash computed from the value of the link register
// and the value of the stack pointer. The hasSideEffects flag is needed as the
// instruction may TRAP if the hash does not match the hash stored at the
// specified address.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "These instructions check a hash computed from the value of the link register". Notable symbols in this range include `HASHST`, `HASHSTP`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“These instructions check a hash computed from the value of the link register”。 该区间中较显眼的符号包括 `HASHST`, `HASHSTP`。

### Lines 5581-5589

```tablegen
let mayLoad = 1, hasSideEffects = 1 in {
def HASHCHK : XForm_XD6_RA5_RB5<31, 754, (outs),
                                (ins gprc:$RB, (memrihash $D, $RA):$addr),
                                "hashchk $RB, $addr", IIC_IntGeneral, []>;
def HASHCHKP : XForm_XD6_RA5_RB5<31, 690, (outs),
                                 (ins gprc:$RB, (memrihash $D, $RA):$addr),
                                 "hashchkp $RB, $addr", IIC_IntGeneral, []>;
}
```
- **EN**: Adds declarative TableGen records such as `HASHCHK`, `HASHCHKP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HASHCHK`, `HASHCHKP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5590-5597

```tablegen
let Defs = [CR7], Itinerary = IIC_LdStSync in
def CFENCE : PPCPostRAExpPseudo<(outs), (ins gprc:$cr), "#CFENCE", []>;

// Now both high word and low word are reversed, next
// swap the high word and low word.
def : Pat<(i64 (bitreverse i64:$A)),
  (OR8 (RLDICR DWBytes7654.DWord, 32, 31), DWBytes3210.DWord)>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Now both high word and low word are reversed, next". Scheduling or processor-model metadata is part of this section.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Now both high word and low word are reversed, next”。 这一段包含调度或处理器模型元数据。

### Lines 5598-5605

```tablegen
def : Pat<(int_ppc_lwarx ForceXForm:$ptr),
          (LWARX ForceXForm:$ptr)>;
def : Pat<(int_ppc_stwcx ForceXForm:$dst, gprc:$A),
          (RLWINM (STWCX gprc:$A, ForceXForm:$dst), 31, 31, 31)>;
def : Pat<(PPCStoreCond ForceXForm:$dst, gprc:$A, 4),
          (RLWINM (STWCX gprc:$A, ForceXForm:$dst), 31, 31, 31)>;
def : Pat<(int_ppc_lbarx ForceXForm:$ptr),
          (LBARX ForceXForm:$ptr)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5606-5612

```tablegen
def : Pat<(int_ppc_stbcx ForceXForm:$dst, gprc:$A),
          (RLWINM (STBCX gprc:$A, ForceXForm:$dst), 31, 31, 31)>;
def : Pat<(PPCStoreCond ForceXForm:$dst, gprc:$A, 1),
          (RLWINM (STBCX gprc:$A, ForceXForm:$dst), 31, 31, 31)>;

def : Pat<(int_ppc_fcfid f64:$A),
        (XSCVSXDDP $A)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5613-5620

```tablegen
def : Pat<(int_ppc_fcfud f64:$A),
        (XSCVUXDDP $A)>;
def : Pat<(int_ppc_fctid f64:$A),
        (FCTID $A)>;
def : Pat<(int_ppc_fctidz f64:$A),
        (XSCVDPSXDS $A)>;
def : Pat<(int_ppc_fctiw f64:$A),
        (FCTIW $A)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5621-5627

```tablegen
def : Pat<(int_ppc_fctiwz f64:$A),
        (XSCVDPSXWS $A)>;
def : Pat<(int_ppc_fctudz f64:$A),
        (XSCVDPUXDS $A)>;
def : Pat<(int_ppc_fctuwz f64:$A),
        (XSCVDPUXWS $A)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5628-5636

```tablegen
def : Pat<(int_ppc_mfmsr), (MFMSR)>;
def : Pat<(int_ppc_mftbu), (MFTB 269)>;
def : Pat<(i32 (int_ppc_mfspr timm:$SPR)),
          (MFSPR $SPR)>;
def : Pat<(int_ppc_mtspr timm:$SPR, gprc:$RT),
          (MTSPR $SPR, $RT)>;
def : Pat<(int_ppc_mtmsr gprc:$RS),
          (MTMSR $RS, 0)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5637-5645

```tablegen
let Predicates = [IsISA2_07] in {
  def : Pat<(int_ppc_lharx ForceXForm:$ptr),
          (LHARX ForceXForm:$ptr)>;

  def : Pat<(int_ppc_sthcx ForceXForm:$dst, gprc:$A),
            (RLWINM (STHCX gprc:$A, ForceXForm:$dst), 31, 31, 31)>;
  def : Pat<(PPCStoreCond ForceXForm:$dst, gprc:$A, 2),
            (RLWINM (STHCX gprc:$A, ForceXForm:$dst), 31, 31, 31)>;
}
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 5646-5652

```tablegen
def : Pat<(int_ppc_dcbtstt ForceXForm:$dst),
          (DCBTST 16, ForceXForm:$dst)>;
def : Pat<(int_ppc_dcbtt ForceXForm:$dst),
          (DCBT 16, ForceXForm:$dst)>;

def : Pat<(int_ppc_stfiw ForceXForm:$dst, f64:$XT),
          (STFIWX f64:$XT, ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Instruction information and lowering helpers / 指令信息与 lowering 辅助逻辑
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Register modeling / 寄存器建模
- Assembly parsing / 汇编解析
- Instruction selection or opcode handling / 指令选择或操作码处理
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCInstrFormats.td`
- `PPCInstrAltivec.td`
- `PPCInstrSPE.td`
- `PPCInstr64Bit.td`
- `PPCInstrVSX.td`
- `PPCInstrHTM.td`
- `PPCInstrP10.td`
- `PPCInstrFutureMMA.td`
- `PPCInstrFuture.td`
- `PPCInstrMMA.td`
- `PPCInstrDFP.td`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
