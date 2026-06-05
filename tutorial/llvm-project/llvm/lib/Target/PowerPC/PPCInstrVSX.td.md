# PPCInstrVSX.td — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCInstrVSX.td`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCInstrVSX.td - The PowerPC VSX Extension. It is written in LLVM TableGen DSL and feeds generated backend tables.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCInstrVSX.td`，主要负责 PowerPC 后端的该后端的 TableGen 声明。 它使用 LLVM TableGen DSL 编写，用于描述声明式目标信息并驱动生成代码。
- **Language**: TableGen DSL / TableGen 领域特定语言

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```tablegen
//===- PPCInstrVSX.td - The PowerPC VSX Extension --*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-10

```tablegen
//===----------------------------------------------------------------------===//
//
// This file describes the VSX extension to the PowerPC instruction set.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file describes the VSX extension to the PowerPC instruction set.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file describes the VSX extension to the PowerPC instruction set.”。

### Lines 11-48

```tablegen
//===----------------------------------------------------------------------===//

// *********************************** NOTE ***********************************
// ** For POWER8 Little Endian, the VSX swap optimization relies on knowing  **
// ** which VMX and VSX instructions are lane-sensitive and which are not.   **
// ** A lane-sensitive instruction relies, implicitly or explicitly, on      **
// ** whether lanes are numbered from left to right.  An instruction like    **
// ** VADDFP is not lane-sensitive, because each lane of the result vector   **
// ** relies only on the corresponding lane of the source vectors.  However, **
// ** an instruction like VMULESB is lane-sensitive, because "even" and      **
// ** "odd" lanes are different for big-endian and little-endian numbering.  **
// **                                                                        **
// ** When adding new VMX and VSX instructions, please consider whether they **
// ** are lane-sensitive.  If so, they must be added to a switch statement   **
// ** in PPCVSXSwapRemoval::gatherVectorInstructions().                      **
// ****************************************************************************

// *********************************** NOTE ***********************************
// ** When adding new anonymous patterns to this file, please add them to    **
// ** the section titled Anonymous Patterns. Chances are that the existing   **
// ** predicate blocks already contain a combination of features that you    **
// ** are after. There is a list of blocks at the top of the section. If     **
// ** you definitely need a new combination of predicates, please add that   **
// ** combination to the list.                                               **
// ** File Structure:                                                        **
// ** - Custom PPCISD node definitions                                       **
// ** - Predicate definitions: predicates to specify the subtargets for      **
// **   which an instruction or pattern can be emitted.                      **
// ** - Instruction formats: classes instantiated by the instructions.       **
// **   These generally correspond to instruction formats in section 1.6 of  **
// **   the ISA document.                                                    **
// ** - Instruction definitions: the actual definitions of the instructions  **
// **   often including input patterns that they match.                      **
// ** - Helper DAG definitions: We define a number of dag objects to use as  **
// **   input or output patterns for consciseness of the code.               **
// ** - Anonymous patterns: input patterns that an instruction matches can   **
// **   often not be specified as part of the instruction definition, so an  **
// **   anonymous pattern must be specified mapping an input pattern to an   **
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "*********************************** NOTE ***********************************". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“*********************************** NOTE ***********************************”。 这些声明会进入生成式模式匹配逻辑。

### Lines 49-57

```tablegen
// **   output pattern. These are generally guarded by subtarget predicates. **
// ** - Instruction aliases: used to define extended mnemonics for assembly  **
// **   printing (for example: xxswapd for xxpermdi with 0x2 as the imm).    **
// ****************************************************************************

def SDT_PPCldvsxlh : SDTypeProfile<1, 1, [
  SDTCisVT<0, v4f32>, SDTCisPtrTy<1>
]>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "**   output pattern. These are generally guarded by subtarget predicates. **". Notable symbols in this range include `SDT_PPCldvsxlh`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“**   output pattern. These are generally guarded by subtarget predicates. **”。 该区间中较显眼的符号包括 `SDT_PPCldvsxlh`。

### Lines 58-66

```tablegen
def SDT_PPCfpexth : SDTypeProfile<1, 2, [
  SDTCisVT<0, v2f64>, SDTCisVT<1, v4f32>, SDTCisPtrTy<2>
]>;

def SDT_PPCldsplat : SDTypeProfile<1, 1, [
  SDTCisVec<0>, SDTCisPtrTy<1>
]>;

// Little-endian-specific nodes.
```
- **EN**: Adds declarative TableGen records such as `SDT_PPCfpexth`, `SDT_PPCldsplat` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPCfpexth`, `SDT_PPCldsplat`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 67-75

```tablegen
def SDT_PPClxvd2x : SDTypeProfile<1, 1, [
  SDTCisVT<0, v2f64>, SDTCisPtrTy<1>
]>;
def SDT_PPCstxvd2x : SDTypeProfile<0, 2, [
  SDTCisVT<0, v2f64>, SDTCisPtrTy<1>
]>;
def SDT_PPCxxswapd : SDTypeProfile<1, 1, [
  SDTCisSameAs<0, 1>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDT_PPClxvd2x`, `SDT_PPCstxvd2x`, `SDT_PPCxxswapd` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDT_PPClxvd2x`, `SDT_PPCstxvd2x`, `SDT_PPCxxswapd`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 76-85

```tablegen
def SDTVecConv : SDTypeProfile<1, 2, [
  SDTCisVec<0>, SDTCisVec<1>, SDTCisPtrTy<2>
]>;
def SDT_PPCld_vec_be : SDTypeProfile<1, 1, [
  SDTCisVec<0>, SDTCisPtrTy<1>
]>;
def SDT_PPCst_vec_be : SDTypeProfile<0, 2, [
  SDTCisVec<0>, SDTCisPtrTy<1>
]>;
```
- **EN**: Adds declarative TableGen records such as `SDTVecConv`, `SDT_PPCld_vec_be`, `SDT_PPCst_vec_be` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `SDTVecConv`, `SDT_PPCld_vec_be`, `SDT_PPCst_vec_be`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 86-93

```tablegen
def SDT_PPCxxperm : SDTypeProfile<1, 3, [
  SDTCisVT<0, v2f64>, SDTCisVT<1, v2f64>,
  SDTCisVT<2, v2f64>, SDTCisVT<3, v4i32>]>;
//--------------------------- Custom PPC nodes -------------------------------//

// VSRC, CHAIN = LXVD2X_LE CHAIN, Ptr - Occurs only for little endian.
// Maps directly to an lxvd2x instruction that will be followed by
// an xxswapd.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "--------------------------- Custom PPC nodes -------------------------------//". Notable symbols in this range include `SDT_PPCxxperm`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“--------------------------- Custom PPC nodes -------------------------------//”。 该区间中较显眼的符号包括 `SDT_PPCxxperm`。

### Lines 94-105

```tablegen
def PPClxvd2x  : SDNode<"PPCISD::LXVD2X", SDT_PPClxvd2x,
                        [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

// CHAIN = STXVD2X CHAIN, VSRC, Ptr - Occurs only for little endian.
// Maps directly to an stxvd2x instruction that will be preceded by
// an xxswapd.
def PPCstxvd2x : SDNode<"PPCISD::STXVD2X", SDT_PPCstxvd2x,
                        [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;

// VSRC, CHAIN = LOAD_VEC_BE CHAIN, Ptr - Occurs only for little endian.
// Maps directly to one of lxvd2x/lxvw4x/lxvh8x/lxvb16x depending on
// the vector type to load vector in big-endian element order.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "CHAIN = STXVD2X CHAIN, VSRC, Ptr - Occurs only for little endian.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“CHAIN = STXVD2X CHAIN, VSRC, Ptr - Occurs only for little endian.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 106-119

```tablegen
def PPCld_vec_be  : SDNode<"PPCISD::LOAD_VEC_BE", SDT_PPCld_vec_be,
                        [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

// CHAIN = STORE_VEC_BE CHAIN, VSRC, Ptr - Occurs only for little endian.
// Maps directly to one of stxvd2x/stxvw4x/stxvh8x/stxvb16x depending on
// the vector type to store vector in big-endian element order.
def PPCst_vec_be : SDNode<"PPCISD::STORE_VEC_BE", SDT_PPCst_vec_be,
                        [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;

// VSRC, CHAIN = XXSWAPD CHAIN, VSRC - Occurs only for little
// endian.  Maps to an xxswapd instruction that corrects an lxvd2x
// or stxvd2x instruction.  The chain is necessary because the
// sequence replaces a load and needs to provide the same number
// of outputs.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "CHAIN = STORE_VEC_BE CHAIN, VSRC, Ptr - Occurs only for little endian.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“CHAIN = STORE_VEC_BE CHAIN, VSRC, Ptr - Occurs only for little endian.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 120-128

```tablegen
def PPCxxswapd : SDNode<"PPCISD::XXSWAPD", SDT_PPCxxswapd, [SDNPHasChain]>;

// Direct move from a VSX register to a GPR
def PPCmfvsr : SDNode<"PPCISD::MFVSR", SDTUnaryOp, []>;

// Direct move from a GPR to a VSX register (algebraic)
def PPCmtvsra : SDNode<"PPCISD::MTVSRA", SDTUnaryOp, []>;

// Direct move from a GPR to a VSX register (zero)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Direct move from a VSX register to a GPR". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Direct move from a VSX register to a GPR”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 129-138

```tablegen
def PPCmtvsrz : SDNode<"PPCISD::MTVSRZ", SDTUnaryOp, []>;

// Extract a subvector from signed integer vector and convert to FP.
// It is primarily used to convert a (widened) illegal integer vector
// type to a legal floating point vector type.
// For example v2i32 -> widened to v4i32 -> v2f64
def PPCsvec2fp : SDNode<"PPCISD::SINT_VEC_TO_FP", SDTVecConv, []>;

// Extract a subvector from unsigned integer vector and convert to FP.
// As with SINT_VEC_TO_FP, used for converting illegal types.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Extract a subvector from signed integer vector and convert to FP.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Extract a subvector from signed integer vector and convert to FP.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 139-146

```tablegen
def PPCuvec2fp: SDNode<"PPCISD::UINT_VEC_TO_FP", SDTVecConv, []>;

// An SDNode for swaps that are not associated with any loads/stores
// and thereby have no chain.
def PPCswapNoChain : SDNode<"PPCISD::SWAP_NO_CHAIN", SDT_PPCxxswapd>;

// FP_EXTEND_HALF(VECTOR, IDX) - Custom extend upper (IDX=0) half or
// lower (IDX=1) half of v4f32 to v2f64.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "An SDNode for swaps that are not associated with any loads/stores". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“An SDNode for swaps that are not associated with any loads/stores”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 147-155

```tablegen
def PPCfpexth : SDNode<"PPCISD::FP_EXTEND_HALF", SDT_PPCfpexth, []>;

// VSRC, CHAIN = LD_VSX_LH CHAIN, Ptr - This is a floating-point load of a
// v2f32 value into the lower half of a VSR register.
def PPCldvsxlh : SDNode<"PPCISD::LD_VSX_LH", SDT_PPCldvsxlh,
                        [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

// VSRC, CHAIN = LD_SPLAT, CHAIN, Ptr - a splatting load memory
// instructions such as LXVDSX, LXVWSX.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VSRC, CHAIN = LD_VSX_LH CHAIN, Ptr - This is a floating-point load of a". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VSRC, CHAIN = LD_VSX_LH CHAIN, Ptr - This is a floating-point load of a”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 156-165

```tablegen
def PPCldsplat : SDNode<"PPCISD::LD_SPLAT", SDT_PPCldsplat,
                        [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

// VSRC, CHAIN = ZEXT_LD_SPLAT, CHAIN, Ptr - a splatting load memory
// that zero-extends.
def PPCzextldsplat : SDNode<"PPCISD::ZEXT_LD_SPLAT", SDT_PPCldsplat,
                        [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

// VSRC, CHAIN = SEXT_LD_SPLAT, CHAIN, Ptr - a splatting load memory
// that sign-extends.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VSRC, CHAIN = ZEXT_LD_SPLAT, CHAIN, Ptr - a splatting load memory". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VSRC, CHAIN = ZEXT_LD_SPLAT, CHAIN, Ptr - a splatting load memory”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 166-174

```tablegen
def PPCsextldsplat : SDNode<"PPCISD::SEXT_LD_SPLAT", SDT_PPCldsplat,
                        [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;

// PowerPC instructions that have SCALAR_TO_VECTOR semantics tend to
// place the value into the least significant element of the most
// significant doubleword in the vector. This is not element zero for
// anything smaller than a doubleword on either endianness. This node has
// the same semantics as SCALAR_TO_VECTOR except that the value remains in
// the aforementioned location in the vector register.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PowerPC instructions that have SCALAR_TO_VECTOR semantics tend to". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PowerPC instructions that have SCALAR_TO_VECTOR semantics tend to”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 175-187

```tablegen
def PPCSToV : SDNode<"PPCISD::SCALAR_TO_VECTOR_PERMUTED",
                     SDTypeProfile<1, 1, []>, []>;

def PPCxxperm : SDNode<"PPCISD::XXPERM", SDT_PPCxxperm, []>;

def PPCldsplatAlign16 : PatFrag<(ops node:$ptr), (PPCldsplat node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getAlign() >= Align(16) &&
         isOffsetMultipleOf(N, 16);
}]>;

//--------------------- VSX-specific instruction formats ---------------------//
// By default, all VSX instructions are to be selected over their Altivec
// counter parts and they do not have unmodeled sideeffects.
```
- **EN**: Adds declarative TableGen records such as `PPCSToV`, `PPCxxperm`, `PPCldsplatAlign16` that LLVM later expands into generated tables or helper code. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `PPCSToV`, `PPCxxperm`, `PPCldsplatAlign16`，LLVM 随后会把它们展开成生成表或辅助代码。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 188-195

```tablegen
let AddedComplexity = 400, hasSideEffects = 0 in {
multiclass XX3Form_Rcr<bits<6> opcode, bits<7> xo, string asmbase,
                    string asmstr, InstrItinClass itin, Intrinsic Int,
                    ValueType OutTy, ValueType InTy> {
  let BaseName = asmbase in {
    def NAME : XX3Form_Rc<opcode, xo, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                       !strconcat(asmbase, !strconcat(" ", asmstr)), itin,
                       [(set OutTy:$XT, (Int InTy:$XA, InTy:$XB))]>;
```
- **EN**: Adds declarative TableGen records such as `XX3Form_Rcr` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `XX3Form_Rcr`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 196-214

```tablegen
    let Defs = [CR6] in
    def _rec    : XX3Form_Rc<opcode, xo, (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                       !strconcat(asmbase, !strconcat(". ", asmstr)), itin,
                       [(set InTy:$XT,
                                (InTy (PPCvcmp_rec InTy:$XA, InTy:$XB, xo)))]>,
                       isRecordForm;
  }
}

// Instruction form with a single input register for instructions such as
// XXPERMDI. The reason for defining this is that specifying multiple chained
// operands (such as loads) to an instruction will perform both chained
// operations rather than coalescing them into a single register - even though
// the source memory location is the same. This simply forces the instruction
// to use the same register for both inputs.
// For example, an output DAG such as this:
//   (XXPERMDI (LXSIBZX xoaddr:$src), (LXSIBZX xoaddr:$src ), 0))
// would result in two load instructions emitted and used as separate inputs
// to the XXPERMDI instruction.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Instruction form with a single input register for instructions such as".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Instruction form with a single input register for instructions such as”。

### Lines 215-221

```tablegen
class XX3Form_2s<bits<6> opcode, bits<5> xo, dag OOL, dag IOL, string asmstr,
                 InstrItinClass itin, list<dag> pattern>
  : XX3Form_2<opcode, xo, OOL, IOL, asmstr, itin, pattern> {
    let XB = XA;
}

let Predicates = [HasVSX, HasP9Vector] in {
```
- **EN**: Declares a backend-facing type `XX3Form_2s` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `XX3Form_2s`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 222-233

```tablegen
class X_VT5_XO5_VB5<bits<6> opcode, bits<5> xo2, bits<10> xo, string opc,
                    list<dag> pattern>
  : X_RD5_XO5_RS5<opcode, xo2, xo, (outs vrrc:$RST), (ins vrrc:$RB),
                  !strconcat(opc, " $RST, $RB"), IIC_VecFP, pattern>;

// [PO VRT XO VRB XO RO], Round to Odd version of [PO VRT XO VRB XO /]
class X_VT5_XO5_VB5_Ro<bits<6> opcode, bits<5> xo2, bits<10> xo, string opc,
                       list<dag> pattern>
  : X_VT5_XO5_VB5<opcode, xo2, xo, opc, pattern>, isRecordForm;

// [PO VRT XO VRB XO /], but the VRB is only used the left 64 bits (or less),
// So we use different operand class for VRB
```
- **EN**: Declares a backend-facing type `X_VT5_XO5_VB5`, `X_VT5_XO5_VB5_Ro` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_VT5_XO5_VB5`, `X_VT5_XO5_VB5_Ro`，并勾勒出周边代码会依赖的接口或状态。

### Lines 234-245

```tablegen
class X_VT5_XO5_VB5_TyVB<bits<6> opcode, bits<5> xo2, bits<10> xo, string opc,
                         RegisterOperand vbtype, list<dag> pattern>
  : X_RD5_XO5_RS5<opcode, xo2, xo, (outs vrrc:$RST), (ins vbtype:$RB),
                  !strconcat(opc, " $RST, $RB"), IIC_VecFP, pattern>;

// [PO VRT XO VRB XO /]
class X_VT5_XO5_VB5_VSFR<bits<6> opcode, bits<5> xo2, bits<10> xo, string opc,
                    list<dag> pattern>
  : X_RD5_XO5_RS5<opcode, xo2, xo, (outs vfrc:$RST), (ins vrrc:$RB),
                  !strconcat(opc, " $RST, $RB"), IIC_VecFP, pattern>;

// [PO VRT XO VRB XO RO], Round to Odd version of [PO VRT XO VRB XO /]
```
- **EN**: Declares a backend-facing type `X_VT5_XO5_VB5_TyVB`, `X_VT5_XO5_VB5_VSFR` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_VT5_XO5_VB5_TyVB`, `X_VT5_XO5_VB5_VSFR`，并勾勒出周边代码会依赖的接口或状态。

### Lines 246-256

```tablegen
class X_VT5_XO5_VB5_VSFR_Ro<bits<6> opcode, bits<5> xo2, bits<10> xo, string opc,
                       list<dag> pattern>
  : X_VT5_XO5_VB5_VSFR<opcode, xo2, xo, opc, pattern>, isRecordForm;

// [PO T XO B XO BX /]
class XX2_RT5_XO5_XB6<bits<6> opcode, bits<5> xo2, bits<9> xo, string opc,
                      list<dag> pattern>
  : XX2_RD5_XO5_RS6<opcode, xo2, xo, (outs g8rc:$RT), (ins vsfrc:$XB),
                    !strconcat(opc, " $RT, $XB"), IIC_VecFP, pattern>;

// [PO T XO B XO BX TX]
```
- **EN**: Declares a backend-facing type `X_VT5_XO5_VB5_VSFR_Ro`, `XX2_RT5_XO5_XB6` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_VT5_XO5_VB5_VSFR_Ro`, `XX2_RT5_XO5_XB6`，并勾勒出周边代码会依赖的接口或状态。

### Lines 257-269

```tablegen
class XX2_XT6_XO5_XB6<bits<6> opcode, bits<5> xo2, bits<9> xo, string opc,
                      RegisterOperand vtype, list<dag> pattern>
  : XX2_RD6_XO5_RS6<opcode, xo2, xo, (outs vtype:$XT), (ins vtype:$XB),
                    !strconcat(opc, " $XT, $XB"), IIC_VecFP, pattern>;

// [PO T A B XO AX BX TX], src and dest register use different operand class
class XX3_XT5_XA5_XB5<bits<6> opcode, bits<8> xo, string opc,
                RegisterOperand xty, RegisterOperand aty, RegisterOperand bty,
                InstrItinClass itin, list<dag> pattern>
  : XX3Form<opcode, xo, (outs xty:$XT), (ins aty:$XA, bty:$XB),
            !strconcat(opc, " $XT, $XA, $XB"), itin, pattern>;

// [PO VRT VRA VRB XO /]
```
- **EN**: Declares a backend-facing type `XX2_XT6_XO5_XB6`, `XX3_XT5_XA5_XB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `XX2_XT6_XO5_XB6`, `XX3_XT5_XA5_XB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 270-280

```tablegen
class X_VT5_VA5_VB5<bits<6> opcode, bits<10> xo, string opc,
                    list<dag> pattern>
  : XForm_1<opcode, xo, (outs vrrc:$RST), (ins vrrc:$RA, vrrc:$RB),
            !strconcat(opc, " $RST, $RA, $RB"), IIC_VecFP, pattern>;

// [PO VRT VRA VRB XO RO], Round to Odd version of [PO VRT VRA VRB XO /]
class X_VT5_VA5_VB5_Ro<bits<6> opcode, bits<10> xo, string opc,
                       list<dag> pattern>
  : X_VT5_VA5_VB5<opcode, xo, opc, pattern>, isRecordForm;

// [PO VRT VRA VRB XO /]
```
- **EN**: Declares a backend-facing type `X_VT5_VA5_VB5`, `X_VT5_VA5_VB5_Ro` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_VT5_VA5_VB5`, `X_VT5_VA5_VB5_Ro`，并勾勒出周边代码会依赖的接口或状态。

### Lines 281-287

```tablegen
class X_VT5_VA5_VB5_FMA<bits<6> opcode, bits<10> xo, string opc,
                        list<dag> pattern>
  : XForm_1<opcode, xo, (outs vrrc:$RST), (ins vrrc:$RSTi, vrrc:$RA, vrrc:$RB),
            !strconcat(opc, " $RST, $RA, $RB"), IIC_VecFP, pattern>,
            RegConstraint<"$RSTi = $RST">;

// [PO VRT VRA VRB XO RO], Round to Odd version of [PO VRT VRA VRB XO /]
```
- **EN**: Declares a backend-facing type `X_VT5_VA5_VB5_FMA` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_VT5_VA5_VB5_FMA`，并勾勒出周边代码会依赖的接口或状态。

### Lines 288-296

```tablegen
class X_VT5_VA5_VB5_FMA_Ro<bits<6> opcode, bits<10> xo, string opc,
                        list<dag> pattern>
  : X_VT5_VA5_VB5_FMA<opcode, xo, opc, pattern>, isRecordForm;

class Z23_VT5_R1_VB5_RMC2_EX1<bits<6> opcode, bits<8> xo, bit ex, string opc,
                              list<dag> pattern>
  : Z23Form_8<opcode, xo,
              (outs vrrc:$VRT), (ins u1imm:$R, vrrc:$VRB, u2imm:$idx),
              !strconcat(opc, " $R, $VRT, $VRB, $idx"), IIC_VecFP, pattern> {
```
- **EN**: Declares a backend-facing type `X_VT5_VA5_VB5_FMA_Ro`, `Z23_VT5_R1_VB5_RMC2_EX1` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_VT5_VA5_VB5_FMA_Ro`, `Z23_VT5_R1_VB5_RMC2_EX1`，并勾勒出周边代码会依赖的接口或状态。

### Lines 297-304

```tablegen
  let RC = ex;
}

// [PO BF // VRA VRB XO /]
class X_BF3_VA5_VB5<bits<6> opcode, bits<10> xo, string opc,
                    list<dag> pattern>
  : XForm_17<opcode, xo, (outs crrc:$BF), (ins vrrc:$RA, vrrc:$RB),
             !strconcat(opc, " $BF, $RA, $RB"), IIC_FPCompare> {
```
- **EN**: Declares a backend-facing type `X_BF3_VA5_VB5` and outlines the API or state that nearby code will rely on.
- **CN**: 这里声明面向后端的类型 `X_BF3_VA5_VB5`，并勾勒出周边代码会依赖的接口或状态。

### Lines 305-315

```tablegen
  let Pattern = pattern;
}

// [PO T RA RB XO TX] almost equal to [PO S RA RB XO SX], but has different
// "out" and "in" dag
class X_XT6_RA5_RB5<bits<6> opcode, bits<10> xo, string opc,
                    RegisterOperand vtype, list<dag> pattern>
  : XX1Form_memOp<opcode, xo, (outs vtype:$XT), (ins (memrr $RA, $RB):$addr),
            !strconcat(opc, " $XT, $addr"), IIC_LdStLFD, pattern>;

// [PO S RA RB XO SX]
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "[PO T RA RB XO TX] almost equal to [PO S RA RB XO SX], but has different". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“[PO T RA RB XO TX] almost equal to [PO S RA RB XO SX], but has different”。 这些声明会进入生成式模式匹配逻辑。

### Lines 316-322

```tablegen
class X_XS6_RA5_RB5<bits<6> opcode, bits<10> xo, string opc,
                    RegisterOperand vtype, list<dag> pattern>
  : XX1Form_memOp<opcode, xo, (outs), (ins vtype:$XT, (memrr $RA, $RB):$addr),
            !strconcat(opc, " $XT, $addr"), IIC_LdStSTFD, pattern>;
} // Predicates = HasP9Vector
} // AddedComplexity = 400, hasSideEffects = 0
```
- **EN**: Declares a backend-facing type `X_XS6_RA5_RB5` and outlines the API or state that nearby code will rely on. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明面向后端的类型 `X_XS6_RA5_RB5`，并勾勒出周边代码会依赖的接口或状态。 这些声明会进入生成式模式匹配逻辑。

### Lines 323-331

```tablegen
multiclass ScalToVecWPermute<ValueType Ty, dag In, dag NonPermOut, dag PermOut> {
  def : Pat<(Ty (scalar_to_vector In)), (Ty NonPermOut)>;
  def : Pat<(Ty (PPCSToV In)), (Ty PermOut)>;
}

//-------------------------- Instruction definitions -------------------------//
// VSX instructions require the VSX feature, they are to be selected over
// equivalent Altivec patterns (as they address a larger register set) and
// they do not have unmodeled side effects.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "-------------------------- Instruction definitions -------------------------//". Notable symbols in this range include `ScalToVecWPermute`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“-------------------------- Instruction definitions -------------------------//”。 该区间中较显眼的符号包括 `ScalToVecWPermute`。

### Lines 332-343

```tablegen
let Predicates = [HasVSX], AddedComplexity = 400 in {
let hasSideEffects = 0 in {

  // Load indexed instructions
  let mayLoad = 1, mayStore = 0 in {
    let CodeSize = 3 in
    def LXSDX : XX1Form_memOp<31, 588,
                        (outs vsfrc:$XT), (ins (memrr $RA, $RB):$addr),
                        "lxsdx $XT, $addr", IIC_LdStLFD,
                        []>;

    // Pseudo instruction XFLOADf64 will be expanded to LXSDX or LFDX later
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 344-354

```tablegen
    let CodeSize = 3 in
      def XFLOADf64  : PseudoXFormMemOp<(outs vsfrc:$XT), (ins (memrr $RA, $RB):$addr),
                              "#XFLOADf64",
                              [(set f64:$XT, (load XForm:$addr))]>;

    let Predicates = [HasVSX, HasOnlySwappingMemOps] in
    def LXVD2X : XX1Form_memOp<31, 844,
                         (outs vsrc:$XT), (ins (memrr $RA, $RB):$addr),
                         "lxvd2x $XT, $addr", IIC_LdStLFD,
                         []>;
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 355-366

```tablegen
    def LXVDSX : XX1Form_memOp<31, 332,
                         (outs vsrc:$XT), (ins (memrr $RA, $RB):$addr),
                         "lxvdsx $XT, $addr", IIC_LdStLFD, []>;

    let Predicates = [HasVSX, HasOnlySwappingMemOps] in
    def LXVW4X : XX1Form_memOp<31, 780,
                         (outs vsrc:$XT), (ins (memrr $RA, $RB):$addr),
                         "lxvw4x $XT, $addr", IIC_LdStLFD,
                         []>;
  } // mayLoad

  // Store indexed instructions
```
- **EN**: Declares function entry points that other backend components call later. These declarations feed generated pattern-matching logic.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。 这些声明会进入生成式模式匹配逻辑。

### Lines 367-374

```tablegen
  let mayStore = 1, mayLoad = 0 in {
    let CodeSize = 3 in
    def STXSDX : XX1Form_memOp<31, 716,
                        (outs), (ins vsfrc:$XT, (memrr $RA, $RB):$addr),
                        "stxsdx $XT, $addr", IIC_LdStSTFD,
                        []>;

    // Pseudo instruction XFSTOREf64  will be expanded to STXSDX or STFDX later
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 375-382

```tablegen
    let CodeSize = 3 in
      def XFSTOREf64 : PseudoXFormMemOp<(outs), (ins vsfrc:$XT, (memrr $RA, $RB):$addr),
                              "#XFSTOREf64",
                              [(store f64:$XT, XForm:$addr)]>;

    let Predicates = [HasVSX, HasOnlySwappingMemOps] in {
    // The behaviour of this instruction is endianness-specific so we provide no
    // pattern to match it without considering endianness.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "The behaviour of this instruction is endianness-specific so we provide no". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“The behaviour of this instruction is endianness-specific so we provide no”。 这些声明会进入生成式模式匹配逻辑。

### Lines 383-394

```tablegen
    def STXVD2X : XX1Form_memOp<31, 972,
                         (outs), (ins vsrc:$XT, (memrr $RA, $RB):$addr),
                         "stxvd2x $XT, $addr", IIC_LdStSTFD,
                         []>;

    def STXVW4X : XX1Form_memOp<31, 908,
                         (outs), (ins vsrc:$XT, (memrr $RA, $RB):$addr),
                         "stxvw4x $XT, $addr", IIC_LdStSTFD,
                         []>;
    }
  } // mayStore
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 395-402

```tablegen
  let mayRaiseFPException = 1 in {
  let Uses = [RM] in {
  // Add/Mul Instructions
  let isCommutable = 1 in {
    def XSADDDP : XX3Form<60, 32,
                          (outs vsfrc:$XT), (ins vsfrc:$XA, vsfrc:$XB),
                          "xsadddp $XT, $XA, $XB", IIC_VecFP,
                          [(set f64:$XT, (any_fadd f64:$XA, f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 403-412

```tablegen
    def XSMULDP : XX3Form<60, 48,
                          (outs vsfrc:$XT), (ins vsfrc:$XA, vsfrc:$XB),
                          "xsmuldp $XT, $XA, $XB", IIC_VecFP,
                          [(set f64:$XT, (any_fmul f64:$XA, f64:$XB))]>;

    def XVADDDP : XX3Form<60, 96,
                          (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                          "xvadddp $XT, $XA, $XB", IIC_VecFP,
                          [(set v2f64:$XT, (any_fadd v2f64:$XA, v2f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 413-422

```tablegen
    def XVADDSP : XX3Form<60, 64,
                          (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                          "xvaddsp $XT, $XA, $XB", IIC_VecFP,
                          [(set v4f32:$XT, (any_fadd v4f32:$XA, v4f32:$XB))]>;

    def XVMULDP : XX3Form<60, 112,
                          (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                          "xvmuldp $XT, $XA, $XB", IIC_VecFP,
                          [(set v2f64:$XT, (any_fmul v2f64:$XA, v2f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 423-429

```tablegen
    def XVMULSP : XX3Form<60, 80,
                          (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                          "xvmulsp $XT, $XA, $XB", IIC_VecFP,
                          [(set v4f32:$XT, (any_fmul v4f32:$XA, v4f32:$XB))]>;
  }

  // Subtract Instructions
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 430-438

```tablegen
  def XSSUBDP : XX3Form<60, 40,
                        (outs vsfrc:$XT), (ins vsfrc:$XA, vsfrc:$XB),
                        "xssubdp $XT, $XA, $XB", IIC_VecFP,
                        [(set f64:$XT, (any_fsub f64:$XA, f64:$XB))]>;

  def XVSUBDP : XX3Form<60, 104,
                        (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                        "xvsubdp $XT, $XA, $XB", IIC_VecFP,
                        [(set v2f64:$XT, (any_fsub v2f64:$XA, v2f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 439-445

```tablegen
  def XVSUBSP : XX3Form<60, 72,
                        (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                        "xvsubsp $XT, $XA, $XB", IIC_VecFP,
                        [(set v4f32:$XT, (any_fsub v4f32:$XA, v4f32:$XB))]>;

  // FMA Instructions
  let BaseName = "XSMADDADP" in {
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 446-452

```tablegen
  let isCommutable = 1 in
  def XSMADDADP : XX3Form<60, 33,
                          (outs vsfrc:$XT), (ins vsfrc:$XTi, vsfrc:$XA, vsfrc:$XB),
                          "xsmaddadp $XT, $XA, $XB", IIC_VecFP,
                          [(set f64:$XT, (any_fma f64:$XA, f64:$XB, f64:$XTi))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 453-460

```tablegen
  let IsVSXFMAAlt = 1 in
  def XSMADDMDP : XX3Form<60, 41,
                          (outs vsfrc:$XT), (ins vsfrc:$XTi, vsfrc:$XA, vsfrc:$XB),
                          "xsmaddmdp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 461-468

```tablegen
  let BaseName = "XSMSUBADP" in {
  let isCommutable = 1 in
  def XSMSUBADP : XX3Form<60, 49,
                          (outs vsfrc:$XT), (ins vsfrc:$XTi, vsfrc:$XA, vsfrc:$XB),
                          "xsmsubadp $XT, $XA, $XB", IIC_VecFP,
                          [(set f64:$XT, (any_fma f64:$XA, f64:$XB, (fneg f64:$XTi)))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 469-476

```tablegen
  let IsVSXFMAAlt = 1 in
  def XSMSUBMDP : XX3Form<60, 57,
                          (outs vsfrc:$XT), (ins vsfrc:$XTi, vsfrc:$XA, vsfrc:$XB),
                          "xsmsubmdp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 477-484

```tablegen
  let BaseName = "XSNMADDADP" in {
  let isCommutable = 1 in
  def XSNMADDADP : XX3Form<60, 161,
                          (outs vsfrc:$XT), (ins vsfrc:$XTi, vsfrc:$XA, vsfrc:$XB),
                          "xsnmaddadp $XT, $XA, $XB", IIC_VecFP,
                          [(set f64:$XT, (fneg (any_fma f64:$XA, f64:$XB, f64:$XTi)))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 485-492

```tablegen
  let IsVSXFMAAlt = 1 in
  def XSNMADDMDP : XX3Form<60, 169,
                          (outs vsfrc:$XT), (ins vsfrc:$XTi, vsfrc:$XA, vsfrc:$XB),
                          "xsnmaddmdp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 493-500

```tablegen
  let BaseName = "XSNMSUBADP" in {
  let isCommutable = 1 in
  def XSNMSUBADP : XX3Form<60, 177,
                          (outs vsfrc:$XT), (ins vsfrc:$XTi, vsfrc:$XA, vsfrc:$XB),
                          "xsnmsubadp $XT, $XA, $XB", IIC_VecFP,
                          [(set f64:$XT, (fneg (any_fma f64:$XA, f64:$XB, (fneg f64:$XTi))))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 501-508

```tablegen
  let IsVSXFMAAlt = 1 in
  def XSNMSUBMDP : XX3Form<60, 185,
                          (outs vsfrc:$XT), (ins vsfrc:$XTi, vsfrc:$XA, vsfrc:$XB),
                          "xsnmsubmdp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 509-516

```tablegen
  let BaseName = "XVMADDADP" in {
  let isCommutable = 1 in
  def XVMADDADP : XX3Form<60, 97,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvmaddadp $XT, $XA, $XB", IIC_VecFP,
                          [(set v2f64:$XT, (any_fma v2f64:$XA, v2f64:$XB, v2f64:$XTi))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 517-524

```tablegen
  let IsVSXFMAAlt = 1 in
  def XVMADDMDP : XX3Form<60, 105,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvmaddmdp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 525-532

```tablegen
  let BaseName = "XVMADDASP" in {
  let isCommutable = 1 in
  def XVMADDASP : XX3Form<60, 65,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvmaddasp $XT, $XA, $XB", IIC_VecFP,
                          [(set v4f32:$XT, (any_fma v4f32:$XA, v4f32:$XB, v4f32:$XTi))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 533-540

```tablegen
  let IsVSXFMAAlt = 1 in
  def XVMADDMSP : XX3Form<60, 73,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvmaddmsp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 541-548

```tablegen
  let BaseName = "XVMSUBADP" in {
  let isCommutable = 1 in
  def XVMSUBADP : XX3Form<60, 113,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvmsubadp $XT, $XA, $XB", IIC_VecFP,
                          [(set v2f64:$XT, (any_fma v2f64:$XA, v2f64:$XB, (fneg v2f64:$XTi)))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 549-556

```tablegen
  let IsVSXFMAAlt = 1 in
  def XVMSUBMDP : XX3Form<60, 121,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvmsubmdp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 557-564

```tablegen
  let BaseName = "XVMSUBASP" in {
  let isCommutable = 1 in
  def XVMSUBASP : XX3Form<60, 81,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvmsubasp $XT, $XA, $XB", IIC_VecFP,
                          [(set v4f32:$XT, (any_fma v4f32:$XA, v4f32:$XB, (fneg v4f32:$XTi)))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 565-572

```tablegen
  let IsVSXFMAAlt = 1 in
  def XVMSUBMSP : XX3Form<60, 89,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvmsubmsp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 573-580

```tablegen
  let BaseName = "XVNMADDADP" in {
  let isCommutable = 1 in
  def XVNMADDADP : XX3Form<60, 225,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvnmaddadp $XT, $XA, $XB", IIC_VecFP,
                          [(set v2f64:$XT, (fneg (any_fma v2f64:$XA, v2f64:$XB, v2f64:$XTi)))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 581-588

```tablegen
  let IsVSXFMAAlt = 1 in
  def XVNMADDMDP : XX3Form<60, 233,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvnmaddmdp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 589-596

```tablegen
  let BaseName = "XVNMADDASP" in {
  let isCommutable = 1 in
  def XVNMADDASP : XX3Form<60, 193,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvnmaddasp $XT, $XA, $XB", IIC_VecFP,
                          [(set v4f32:$XT, (fneg (fma v4f32:$XA, v4f32:$XB, v4f32:$XTi)))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 597-604

```tablegen
  let IsVSXFMAAlt = 1 in
  def XVNMADDMSP : XX3Form<60, 201,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvnmaddmsp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 605-612

```tablegen
  let BaseName = "XVNMSUBADP" in {
  let isCommutable = 1 in
  def XVNMSUBADP : XX3Form<60, 241,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvnmsubadp $XT, $XA, $XB", IIC_VecFP,
                          [(set v2f64:$XT, (fneg (any_fma v2f64:$XA, v2f64:$XB, (fneg v2f64:$XTi))))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 613-620

```tablegen
  let IsVSXFMAAlt = 1 in
  def XVNMSUBMDP : XX3Form<60, 249,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvnmsubmdp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 621-628

```tablegen
  let BaseName = "XVNMSUBASP" in {
  let isCommutable = 1 in
  def XVNMSUBASP : XX3Form<60, 209,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvnmsubasp $XT, $XA, $XB", IIC_VecFP,
                          [(set v4f32:$XT, (fneg (any_fma v4f32:$XA, v4f32:$XB, (fneg v4f32:$XTi))))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 629-637

```tablegen
  let IsVSXFMAAlt = 1 in
  def XVNMSUBMSP : XX3Form<60, 217,
                          (outs vsrc:$XT), (ins vsrc:$XTi, vsrc:$XA, vsrc:$XB),
                          "xvnmsubmsp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }

  // Division Instructions
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 638-646

```tablegen
  def XSDIVDP : XX3Form<60, 56,
                        (outs vsfrc:$XT), (ins vsfrc:$XA, vsfrc:$XB),
                        "xsdivdp $XT, $XA, $XB", IIC_FPDivD,
                        [(set f64:$XT, (any_fdiv f64:$XA, f64:$XB))]>;
  def XSSQRTDP : XX2Form<60, 75,
                        (outs vsfrc:$XT), (ins vsfrc:$XB),
                        "xssqrtdp $XT, $XB", IIC_FPSqrtD,
                        [(set f64:$XT, (any_fsqrt f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 647-655

```tablegen
  def XSREDP : XX2Form<60, 90,
                        (outs vsfrc:$XT), (ins vsfrc:$XB),
                        "xsredp $XT, $XB", IIC_VecFP,
                        [(set f64:$XT, (PPCfre f64:$XB))]>;
  def XSRSQRTEDP : XX2Form<60, 74,
                           (outs vsfrc:$XT), (ins vsfrc:$XB),
                           "xsrsqrtedp $XT, $XB", IIC_VecFP,
                           [(set f64:$XT, (PPCfrsqrte f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 656-663

```tablegen
  let mayRaiseFPException = 0 in {
  def XSTDIVDP : XX3Form_1<60, 61,
                         (outs crrc:$CR), (ins vsfrc:$XA, vsfrc:$XB),
                         "xstdivdp $CR, $XA, $XB", IIC_FPCompare, []>;
  def XSTSQRTDP : XX2Form_1<60, 106,
                          (outs crrc:$CR), (ins vsfrc:$XB),
                          "xstsqrtdp $CR, $XB", IIC_FPCompare,
                          [(set i32:$CR, (PPCftsqrt f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 664-670

```tablegen
  def XVTDIVDP : XX3Form_1<60, 125,
                         (outs crrc:$CR), (ins vsrc:$XA, vsrc:$XB),
                         "xvtdivdp $CR, $XA, $XB", IIC_FPCompare, []>;
  def XVTDIVSP : XX3Form_1<60, 93,
                         (outs crrc:$CR), (ins vsrc:$XA, vsrc:$XB),
                         "xvtdivsp $CR, $XA, $XB", IIC_FPCompare, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 671-680

```tablegen
  def XVTSQRTDP : XX2Form_1<60, 234,
                          (outs crrc:$CR), (ins vsrc:$XB),
                          "xvtsqrtdp $CR, $XB", IIC_FPCompare,
                          [(set i32:$CR, (PPCftsqrt v2f64:$XB))]>;
  def XVTSQRTSP : XX2Form_1<60, 170,
                          (outs crrc:$CR), (ins vsrc:$XB),
                          "xvtsqrtsp $CR, $XB", IIC_FPCompare,
                          [(set i32:$CR, (PPCftsqrt v4f32:$XB))]>;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 681-689

```tablegen
  def XVDIVDP : XX3Form<60, 120,
                        (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                        "xvdivdp $XT, $XA, $XB", IIC_FPDivD,
                        [(set v2f64:$XT, (any_fdiv v2f64:$XA, v2f64:$XB))]>;
  def XVDIVSP : XX3Form<60, 88,
                        (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                        "xvdivsp $XT, $XA, $XB", IIC_FPDivS,
                        [(set v4f32:$XT, (any_fdiv v4f32:$XA, v4f32:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 690-698

```tablegen
  def XVSQRTDP : XX2Form<60, 203,
                        (outs vsrc:$XT), (ins vsrc:$XB),
                        "xvsqrtdp $XT, $XB", IIC_FPSqrtD,
                        [(set v2f64:$XT, (any_fsqrt v2f64:$XB))]>;
  def XVSQRTSP : XX2Form<60, 139,
                        (outs vsrc:$XT), (ins vsrc:$XB),
                        "xvsqrtsp $XT, $XB", IIC_FPSqrtS,
                        [(set v4f32:$XT, (any_fsqrt v4f32:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 699-707

```tablegen
  def XVREDP : XX2Form<60, 218,
                        (outs vsrc:$XT), (ins vsrc:$XB),
                        "xvredp $XT, $XB", IIC_VecFP,
                        [(set v2f64:$XT, (PPCfre v2f64:$XB))]>;
  def XVRESP : XX2Form<60, 154,
                        (outs vsrc:$XT), (ins vsrc:$XB),
                        "xvresp $XT, $XB", IIC_VecFP,
                        [(set v4f32:$XT, (PPCfre v4f32:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 708-717

```tablegen
  def XVRSQRTEDP : XX2Form<60, 202,
                           (outs vsrc:$XT), (ins vsrc:$XB),
                           "xvrsqrtedp $XT, $XB", IIC_VecFP,
                           [(set v2f64:$XT, (PPCfrsqrte v2f64:$XB))]>;
  def XVRSQRTESP : XX2Form<60, 138,
                           (outs vsrc:$XT), (ins vsrc:$XB),
                           "xvrsqrtesp $XT, $XB", IIC_VecFP,
                           [(set v4f32:$XT, (PPCfrsqrte v4f32:$XB))]>;

  // Compare Instructions
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 718-724

```tablegen
  def XSCMPODP : XX3Form_1<60, 43,
                           (outs crrc:$CR), (ins vsfrc:$XA, vsfrc:$XB),
                           "xscmpodp $CR, $XA, $XB", IIC_FPCompare, []>;
  def XSCMPUDP : XX3Form_1<60, 35,
                           (outs crrc:$CR), (ins vsfrc:$XA, vsfrc:$XB),
                           "xscmpudp $CR, $XA, $XB", IIC_FPCompare, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 725-733

```tablegen
  defm XVCMPEQDP : XX3Form_Rcr<60, 99,
                             "xvcmpeqdp", "$XT, $XA, $XB", IIC_VecFPCompare,
                             int_ppc_vsx_xvcmpeqdp, v2i64, v2f64>;
  defm XVCMPEQSP : XX3Form_Rcr<60, 67,
                             "xvcmpeqsp", "$XT, $XA, $XB", IIC_VecFPCompare,
                             int_ppc_vsx_xvcmpeqsp, v4i32, v4f32>;
  defm XVCMPGEDP : XX3Form_Rcr<60, 115,
                             "xvcmpgedp", "$XT, $XA, $XB", IIC_VecFPCompare,
                             int_ppc_vsx_xvcmpgedp, v2i64, v2f64>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 734-744

```tablegen
  defm XVCMPGESP : XX3Form_Rcr<60, 83,
                             "xvcmpgesp", "$XT, $XA, $XB", IIC_VecFPCompare,
                             int_ppc_vsx_xvcmpgesp, v4i32, v4f32>;
  defm XVCMPGTDP : XX3Form_Rcr<60, 107,
                             "xvcmpgtdp", "$XT, $XA, $XB", IIC_VecFPCompare,
                             int_ppc_vsx_xvcmpgtdp, v2i64, v2f64>;
  defm XVCMPGTSP : XX3Form_Rcr<60, 75,
                             "xvcmpgtsp", "$XT, $XA, $XB", IIC_VecFPCompare,
                             int_ppc_vsx_xvcmpgtsp, v4i32, v4f32>;

  // Move Instructions
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 745-753

```tablegen
  let mayRaiseFPException = 0 in {
  def XSABSDP : XX2Form<60, 345,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xsabsdp $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (fabs f64:$XB))]>;
  def XSNABSDP : XX2Form<60, 361,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xsnabsdp $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (fneg (fabs f64:$XB)))]>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 754-762

```tablegen
  let isCodeGenOnly = 1 in
  def XSNABSDPs : XX2Form<60, 361,
                      (outs vssrc:$XT), (ins vssrc:$XB),
                      "xsnabsdp $XT, $XB", IIC_VecFP,
                      [(set f32:$XT, (fneg (fabs f32:$XB)))]>;
  def XSNEGDP : XX2Form<60, 377,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xsnegdp $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (fneg f64:$XB))]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 763-772

```tablegen
  def XSCPSGNDP : XX3Form<60, 176,
                      (outs vsfrc:$XT), (ins vsfrc:$XA, vsfrc:$XB),
                      "xscpsgndp $XT, $XA, $XB", IIC_VecFP,
                      [(set f64:$XT, (fcopysign f64:$XB, f64:$XA))]>;

  def XVABSDP : XX2Form<60, 473,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvabsdp $XT, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (fabs v2f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 773-781

```tablegen
  def XVABSSP : XX2Form<60, 409,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvabssp $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (fabs v4f32:$XB))]>;

  def XVCPSGNDP : XX3Form<60, 240,
                      (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xvcpsgndp $XT, $XA, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (fcopysign v2f64:$XB, v2f64:$XA))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 782-790

```tablegen
  def XVCPSGNSP : XX3Form<60, 208,
                      (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xvcpsgnsp $XT, $XA, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (fcopysign v4f32:$XB, v4f32:$XA))]>;

  def XVNABSDP : XX2Form<60, 489,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvnabsdp $XT, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (fneg (fabs v2f64:$XB)))]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 791-799

```tablegen
  def XVNABSSP : XX2Form<60, 425,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvnabssp $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (fneg (fabs v4f32:$XB)))]>;

  def XVNEGDP : XX2Form<60, 505,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvnegdp $XT, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (fneg v2f64:$XB))]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 800-806

```tablegen
  def XVNEGSP : XX2Form<60, 441,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvnegsp $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (fneg v4f32:$XB))]>;
  }

  // Conversion Instructions
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 807-813

```tablegen
  def XSCVDPSP : XX2Form<60, 265,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xscvdpsp $XT, $XB", IIC_VecFP, []>;
  def XSCVDPSXDS : XX2Form<60, 344,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xscvdpsxds $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (PPCany_fctidz f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 814-822

```tablegen
  let isCodeGenOnly = 1 in
  def XSCVDPSXDSs : XX2Form<60, 344,
                      (outs vssrc:$XT), (ins vssrc:$XB),
                      "xscvdpsxds $XT, $XB", IIC_VecFP,
                      [(set f32:$XT, (PPCany_fctidz f32:$XB))]>;
  def XSCVDPSXWS : XX2Form<60, 88,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xscvdpsxws $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (PPCany_fctiwz f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 823-831

```tablegen
  let isCodeGenOnly = 1 in
  def XSCVDPSXWSs : XX2Form<60, 88,
                      (outs vssrc:$XT), (ins vssrc:$XB),
                      "xscvdpsxws $XT, $XB", IIC_VecFP,
                      [(set f32:$XT, (PPCany_fctiwz f32:$XB))]>;
  def XSCVDPUXDS : XX2Form<60, 328,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xscvdpuxds $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (PPCany_fctiduz f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 832-840

```tablegen
  let isCodeGenOnly = 1 in
  def XSCVDPUXDSs : XX2Form<60, 328,
                      (outs vssrc:$XT), (ins vssrc:$XB),
                      "xscvdpuxds $XT, $XB", IIC_VecFP,
                      [(set f32:$XT, (PPCany_fctiduz f32:$XB))]>;
  def XSCVDPUXWS : XX2Form<60, 72,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xscvdpuxws $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (PPCany_fctiwuz f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 841-848

```tablegen
  let isCodeGenOnly = 1 in
  def XSCVDPUXWSs : XX2Form<60, 72,
                      (outs vssrc:$XT), (ins vssrc:$XB),
                      "xscvdpuxws $XT, $XB", IIC_VecFP,
                      [(set f32:$XT, (PPCany_fctiwuz f32:$XB))]>;
  def XSCVSPDP : XX2Form<60, 329,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xscvspdp $XT, $XB", IIC_VecFP, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 849-857

```tablegen
  def XSCVSXDDP : XX2Form<60, 376,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xscvsxddp $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (PPCany_fcfid f64:$XB))]>;
  def XSCVUXDDP : XX2Form<60, 360,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xscvuxddp $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (PPCany_fcfidu f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 858-865

```tablegen
  def XVCVDPSP : XX2Form<60, 393,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvdpsp $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (int_ppc_vsx_xvcvdpsp v2f64:$XB))]>;
  def XVCVDPSXDS : XX2Form<60, 472,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvdpsxds $XT, $XB", IIC_VecFP,
                      [(set v2i64:$XT, (any_fp_to_sint v2f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 866-873

```tablegen
  def XVCVDPSXWS : XX2Form<60, 216,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvdpsxws $XT, $XB", IIC_VecFP,
                      [(set v4i32:$XT, (int_ppc_vsx_xvcvdpsxws v2f64:$XB))]>;
  def XVCVDPUXDS : XX2Form<60, 456,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvdpuxds $XT, $XB", IIC_VecFP,
                      [(set v2i64:$XT, (any_fp_to_uint v2f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 874-882

```tablegen
  def XVCVDPUXWS : XX2Form<60, 200,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvdpuxws $XT, $XB", IIC_VecFP,
                      [(set v4i32:$XT, (int_ppc_vsx_xvcvdpuxws v2f64:$XB))]>;

  def XVCVSPDP : XX2Form<60, 457,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvspdp $XT, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (int_ppc_vsx_xvcvspdp v4f32:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 883-890

```tablegen
  def XVCVSPSXDS : XX2Form<60, 408,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvspsxds $XT, $XB", IIC_VecFP,
                      [(set v2i64:$XT, (int_ppc_vsx_xvcvspsxds v4f32:$XB))]>;
  def XVCVSPSXWS : XX2Form<60, 152,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvspsxws $XT, $XB", IIC_VecFP,
                      [(set v4i32:$XT, (any_fp_to_sint v4f32:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 891-898

```tablegen
  def XVCVSPUXDS : XX2Form<60, 392,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvspuxds $XT, $XB", IIC_VecFP,
                      [(set v2i64:$XT, (int_ppc_vsx_xvcvspuxds v4f32:$XB))]>;
  def XVCVSPUXWS : XX2Form<60, 136,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvspuxws $XT, $XB", IIC_VecFP,
                      [(set v4i32:$XT, (any_fp_to_uint v4f32:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 899-906

```tablegen
  def XVCVSXDDP : XX2Form<60, 504,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvsxddp $XT, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (any_sint_to_fp v2i64:$XB))]>;
  def XVCVSXDSP : XX2Form<60, 440,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvsxdsp $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (int_ppc_vsx_xvcvsxdsp v2i64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 907-914

```tablegen
  def XVCVSXWSP : XX2Form<60, 184,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvsxwsp $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (any_sint_to_fp v4i32:$XB))]>;
  def XVCVUXDDP : XX2Form<60, 488,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvuxddp $XT, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (any_uint_to_fp v2i64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 915-923

```tablegen
  def XVCVUXDSP : XX2Form<60, 424,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvuxdsp $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (int_ppc_vsx_xvcvuxdsp v2i64:$XB))]>;
  def XVCVUXWSP : XX2Form<60, 168,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvuxwsp $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (any_uint_to_fp v4i32:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 924-935

```tablegen
  let mayRaiseFPException = 0 in {
  def XVCVSXWDP : XX2Form<60, 248,
                    (outs vsrc:$XT), (ins vsrc:$XB),
                    "xvcvsxwdp $XT, $XB", IIC_VecFP,
                    [(set v2f64:$XT, (int_ppc_vsx_xvcvsxwdp v4i32:$XB))]>;
  def XVCVUXWDP : XX2Form<60, 232,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvcvuxwdp $XT, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (int_ppc_vsx_xvcvuxwdp v4i32:$XB))]>;
  }

  // Rounding Instructions respecting current rounding mode
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 936-945

```tablegen
  def XSRDPIC : XX2Form<60, 107,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xsrdpic $XT, $XB", IIC_VecFP, []>;
  def XVRDPIC : XX2Form<60, 235,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvrdpic $XT, $XB", IIC_VecFP, []>;
  def XVRSPIC : XX2Form<60, 171,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvrspic $XT, $XB", IIC_VecFP, []>;
  // Max/Min Instructions
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 946-957

```tablegen
  let isCommutable = 1 in {
  def XSMAXDP : XX3Form<60, 160,
                        (outs vsfrc:$XT), (ins vsfrc:$XA, vsfrc:$XB),
                        "xsmaxdp $XT, $XA, $XB", IIC_VecFP,
                        [(set vsfrc:$XT,
                              (int_ppc_vsx_xsmaxdp vsfrc:$XA, vsfrc:$XB))]>;
  def XSMINDP : XX3Form<60, 168,
                        (outs vsfrc:$XT), (ins vsfrc:$XA, vsfrc:$XB),
                        "xsmindp $XT, $XA, $XB", IIC_VecFP,
                        [(set vsfrc:$XT,
                              (int_ppc_vsx_xsmindp vsfrc:$XA, vsfrc:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 958-968

```tablegen
  def XVMAXDP : XX3Form<60, 224,
                        (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                        "xvmaxdp $XT, $XA, $XB", IIC_VecFP,
                        [(set vsrc:$XT,
                              (int_ppc_vsx_xvmaxdp vsrc:$XA, vsrc:$XB))]>;
  def XVMINDP : XX3Form<60, 232,
                        (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                        "xvmindp $XT, $XA, $XB", IIC_VecFP,
                        [(set vsrc:$XT,
                              (int_ppc_vsx_xvmindp vsrc:$XA, vsrc:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 969-982

```tablegen
  def XVMAXSP : XX3Form<60, 192,
                        (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                        "xvmaxsp $XT, $XA, $XB", IIC_VecFP,
                        [(set vsrc:$XT,
                              (int_ppc_vsx_xvmaxsp vsrc:$XA, vsrc:$XB))]>;
  def XVMINSP : XX3Form<60, 200,
                        (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                        "xvminsp $XT, $XA, $XB", IIC_VecFP,
                        [(set vsrc:$XT,
                              (int_ppc_vsx_xvminsp vsrc:$XA, vsrc:$XB))]>;
  } // isCommutable
  } // Uses = [RM]

  // Rounding Instructions with static direction.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 983-990

```tablegen
  def XSRDPI : XX2Form<60, 73,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xsrdpi $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (any_fround f64:$XB))]>;
  def XSRDPIM : XX2Form<60, 121,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xsrdpim $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (any_ffloor f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 991-999

```tablegen
  def XSRDPIP : XX2Form<60, 105,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xsrdpip $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (any_fceil f64:$XB))]>;
  def XSRDPIZ : XX2Form<60, 89,
                      (outs vsfrc:$XT), (ins vsfrc:$XB),
                      "xsrdpiz $XT, $XB", IIC_VecFP,
                      [(set f64:$XT, (any_ftrunc f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1000-1007

```tablegen
  def XVRDPI : XX2Form<60, 201,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvrdpi $XT, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (any_fround v2f64:$XB))]>;
  def XVRDPIM : XX2Form<60, 249,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvrdpim $XT, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (any_ffloor v2f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1008-1016

```tablegen
  def XVRDPIP : XX2Form<60, 233,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvrdpip $XT, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (any_fceil v2f64:$XB))]>;
  def XVRDPIZ : XX2Form<60, 217,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvrdpiz $XT, $XB", IIC_VecFP,
                      [(set v2f64:$XT, (any_ftrunc v2f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1017-1024

```tablegen
  def XVRSPI : XX2Form<60, 137,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvrspi $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (any_fround v4f32:$XB))]>;
  def XVRSPIM : XX2Form<60, 185,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvrspim $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (any_ffloor v4f32:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1025-1035

```tablegen
  def XVRSPIP : XX2Form<60, 169,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvrspip $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (any_fceil v4f32:$XB))]>;
  def XVRSPIZ : XX2Form<60, 153,
                      (outs vsrc:$XT), (ins vsrc:$XB),
                      "xvrspiz $XT, $XB", IIC_VecFP,
                      [(set v4f32:$XT, (any_ftrunc v4f32:$XB))]>;
  } // mayRaiseFPException

  // Logical Instructions
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1036-1045

```tablegen
  let isCommutable = 1 in
  def XXLAND : XX3Form<60, 130,
                       (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                       "xxland $XT, $XA, $XB", IIC_VecGeneral,
                       [(set v4i32:$XT, (and v4i32:$XA, v4i32:$XB))]>;
  def XXLANDC : XX3Form<60, 138,
                        (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                        "xxlandc $XT, $XA, $XB", IIC_VecGeneral,
                        [(set v4i32:$XT, (and v4i32:$XA,
                                              (vnot v4i32:$XB)))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1046-1055

```tablegen
  let isCommutable = 1 in {
  def XXLNOR : XX3Form<60, 162,
                       (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                       "xxlnor $XT, $XA, $XB", IIC_VecGeneral,
                       [(set v4i32:$XT, (vnot (or v4i32:$XA,
                                               v4i32:$XB)))]>;
  def XXLOR : XX3Form<60, 146,
                      (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                      "xxlor $XT, $XA, $XB", IIC_VecGeneral,
                      [(set v4i32:$XT, (or v4i32:$XA, v4i32:$XB))]>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 1056-1065

```tablegen
  let isCodeGenOnly = 1 in
  def XXLORf: XX3Form<60, 146,
                      (outs vsfrc:$XT), (ins vsfrc:$XA, vsfrc:$XB),
                      "xxlor $XT, $XA, $XB", IIC_VecGeneral, []>;
  def XXLXOR : XX3Form<60, 154,
                       (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                       "xxlxor $XT, $XA, $XB", IIC_VecGeneral,
                       [(set v4i32:$XT, (xor v4i32:$XA, v4i32:$XB))]>;
  } // isCommutable
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1066-1074

```tablegen
  let isCodeGenOnly = 1, isMoveImm = 1, isAsCheapAsAMove = 1,
      isReMaterializable = 1 in {
    def XXLXORz : XX3Form_SameOp<60, 154, (outs vsrc:$XT), (ins),
                       "xxlxor $XT, $XT, $XT", IIC_VecGeneral,
                       [(set v4i32:$XT, (v4i32 immAllZerosV))]>;
    def XXLXORdpz : XX3Form_SameOp<60, 154,
                         (outs vsfrc:$XT), (ins),
                         "xxlxor $XT, $XT, $XT", IIC_VecGeneral,
                         [(set f64:$XT, (fpimm0))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1075-1081

```tablegen
    def XXLXORspz : XX3Form_SameOp<60, 154,
                         (outs vssrc:$XT), (ins),
                         "xxlxor $XT, $XT, $XT", IIC_VecGeneral,
                         [(set f32:$XT, (fpimm0))]>;
  }

  // Permutation Instructions
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1082-1088

```tablegen
  def XXMRGHW : XX3Form<60, 18,
                       (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                       "xxmrghw $XT, $XA, $XB", IIC_VecPerm, []>;
  def XXMRGLW : XX3Form<60, 50,
                       (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                       "xxmrglw $XT, $XA, $XB", IIC_VecPerm, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1089-1102

```tablegen
  def XXPERMDI : XX3Form_2<60, 10,
                       (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB, u2imm:$D),
                       "xxpermdi $XT, $XA, $XB, $D", IIC_VecPerm,
                       [(set v2i64:$XT, (PPCxxpermdi v2i64:$XA, v2i64:$XB,
                         imm32SExt16:$D))]>;
  let isCodeGenOnly = 1 in
  // Note that the input register class for `$XA` of XXPERMDIs is `vsfrc` which
  // is not the same with the input register class(`vsrc`) of XXPERMDI instruction.
  // We did this on purpose because:
  // 1: The input is primarily for loads that load a partial vector(LFIWZX,
  //    etc.), no need for SUBREG_TO_REG.
  // 2: With `vsfrc` register class, in the final assembly, float registers
  //    like `f0` are used instead of vector scalar register like `vs0`. This
  //    helps readability.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Note that the input register class for `$XA` of XXPERMDIs is `vsfrc` which".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Note that the input register class for `$XA` of XXPERMDIs is `vsfrc` which”。

### Lines 1103-1114

```tablegen
  def XXPERMDIs : XX3Form_2s<60, 10, (outs vsrc:$XT), (ins vsfrc:$XA, u2imm:$D),
                             "xxpermdi $XT, $XA, $XA, $D", IIC_VecPerm, []>;
  def XXSEL : XX4Form<60, 3,
                      (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB, vsrc:$XC),
                      "xxsel $XT, $XA, $XB, $XC", IIC_VecPerm, []>;

  def XXSLDWI : XX3Form_2<60, 2,
                       (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB, u2imm:$D),
                       "xxsldwi $XT, $XA, $XB, $D", IIC_VecPerm,
                       [(set v4i32:$XT, (PPCvecshl v4i32:$XA, v4i32:$XB,
                                                  imm32SExt16:$D))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1115-1124

```tablegen
  let isCodeGenOnly = 1 in
  def XXSLDWIs : XX3Form_2s<60, 2,
                       (outs vsrc:$XT), (ins vsfrc:$XA, u2imm:$D),
                       "xxsldwi $XT, $XA, $XA, $D", IIC_VecPerm, []>;

  def XXSPLTW : XX2Form_2<60, 164,
                       (outs vsrc:$XT), (ins vsrc:$XB, u2imm:$D),
                       "xxspltw $XT, $XB, $D", IIC_VecPerm,
                       [(set v4i32:$XT,
                             (PPCxxsplt v4i32:$XB, imm32SExt16:$D))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1125-1131

```tablegen
  let isCodeGenOnly = 1 in
  def XXSPLTWs : XX2Form_2<60, 164,
                       (outs vsrc:$XT), (ins vsfrc:$XB, u2imm:$D),
                       "xxspltw $XT, $XB, $D", IIC_VecPerm, []>;

// The following VSX instructions were introduced in Power ISA 2.07
let Predicates = [HasVSX, HasP8Vector] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1132-1142

```tablegen
  let isCommutable = 1 in {
    def XXLEQV : XX3Form<60, 186,
                         (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                         "xxleqv $XT, $XA, $XB", IIC_VecGeneral,
                         [(set v4i32:$XT, (vnot (xor v4i32:$XA, v4i32:$XB)))]>;
    def XXLNAND : XX3Form<60, 178,
                          (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                          "xxlnand $XT, $XA, $XB", IIC_VecGeneral,
                          [(set v4i32:$XT, (vnot (and v4i32:$XA, v4i32:$XB)))]>;
  } // isCommutable
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 1143-1149

```tablegen
  let isCodeGenOnly = 1, isMoveImm = 1, isAsCheapAsAMove = 1,
      isReMaterializable = 1 in {
    def XXLEQVOnes : XX3Form_SameOp<60, 186, (outs vsrc:$XT), (ins),
                         "xxleqv $XT, $XT, $XT", IIC_VecGeneral,
                         [(set v4i32:$XT, (bitconvert (v16i8 immAllOnesV)))]>;
  }
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 1150-1156

```tablegen
  def XXLORC : XX3Form<60, 170,
                       (outs vsrc:$XT), (ins vsrc:$XA, vsrc:$XB),
                       "xxlorc $XT, $XA, $XB", IIC_VecGeneral,
                       [(set v4i32:$XT, (or v4i32:$XA, (vnot v4i32:$XB)))]>;

  // VSX scalar loads introduced in ISA 2.07
  let mayLoad = 1, mayStore = 0 in {
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1157-1165

```tablegen
    let CodeSize = 3 in
    def LXSSPX : XX1Form_memOp<31, 524, (outs vssrc:$XT), (ins (memrr $RA, $RB):$addr),
                         "lxsspx $XT, $addr", IIC_LdStLFD, []>;
    def LXSIWAX : XX1Form_memOp<31, 76, (outs vsfrc:$XT), (ins (memrr $RA, $RB):$addr),
                          "lxsiwax $XT, $addr", IIC_LdStLFD, []>;
    def LXSIWZX : XX1Form_memOp<31, 12, (outs vsfrc:$XT), (ins (memrr $RA, $RB):$addr),
                          "lxsiwzx $XT, $addr", IIC_LdStLFD, []>;

    // Pseudo instruction XFLOADf32 will be expanded to LXSSPX or LFSX later
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1166-1174

```tablegen
    let CodeSize = 3 in
    def XFLOADf32  : PseudoXFormMemOp<(outs vssrc:$XT), (ins memrr:$src),
                            "#XFLOADf32",
                            [(set f32:$XT, (load XForm:$src))]>;
    // Pseudo instruction LIWAX will be expanded to LXSIWAX or LFIWAX later
    def LIWAX : PseudoXFormMemOp<(outs vsfrc:$XT), (ins memrr:$src),
                       "#LIWAX",
                       [(set f64:$XT, (PPClfiwax ForceXForm:$src))]>;
    // Pseudo instruction LIWZX will be expanded to LXSIWZX or LFIWZX later
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1175-1181

```tablegen
    def LIWZX : PseudoXFormMemOp<(outs vsfrc:$XT), (ins memrr:$src),
                       "#LIWZX",
                       [(set f64:$XT, (PPClfiwzx ForceXForm:$src))]>;
  } // mayLoad

  // VSX scalar stores introduced in ISA 2.07
  let mayStore = 1, mayLoad = 0 in {
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1182-1188

```tablegen
    let CodeSize = 3 in
    def STXSSPX : XX1Form_memOp<31, 652, (outs), (ins vssrc:$XT, (memrr $RA, $RB):$addr),
                          "stxsspx $XT, $addr", IIC_LdStSTFD, []>;
    def STXSIWX : XX1Form_memOp<31, 140, (outs), (ins vsfrc:$XT, (memrr $RA, $RB):$addr),
                          "stxsiwx $XT, $addr", IIC_LdStSTFD, []>;

    // Pseudo instruction XFSTOREf32 will be expanded to STXSSPX or STFSX later
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1189-1199

```tablegen
    let CodeSize = 3 in
    def XFSTOREf32 : PseudoXFormMemOp<(outs), (ins vssrc:$XT, memrr:$dst),
                            "#XFSTOREf32",
                            [(store f32:$XT, XForm:$dst)]>;
    // Pseudo instruction STIWX will be expanded to STXSIWX or STFIWX later
    def STIWX : PseudoXFormMemOp<(outs), (ins vsfrc:$XT, memrr:$dst),
                       "#STIWX",
                      [(PPCstfiwx f64:$XT, ForceXForm:$dst)]>;
  } // mayStore

  // VSX Elementary Scalar FP arithmetic (SP)
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1200-1211

```tablegen
  let mayRaiseFPException = 1 in {
  let isCommutable = 1 in {
    def XSADDSP : XX3Form<60, 0,
                          (outs vssrc:$XT), (ins vssrc:$XA, vssrc:$XB),
                          "xsaddsp $XT, $XA, $XB", IIC_VecFP,
                          [(set f32:$XT, (any_fadd f32:$XA, f32:$XB))]>;
    def XSMULSP : XX3Form<60, 16,
                          (outs vssrc:$XT), (ins vssrc:$XA, vssrc:$XB),
                          "xsmulsp $XT, $XA, $XB", IIC_VecFP,
                          [(set f32:$XT, (any_fmul f32:$XA, f32:$XB))]>;
  } // isCommutable
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1212-1220

```tablegen
  def XSSUBSP : XX3Form<60, 8,
                        (outs vssrc:$XT), (ins vssrc:$XA, vssrc:$XB),
                        "xssubsp $XT, $XA, $XB", IIC_VecFP,
                        [(set f32:$XT, (any_fsub f32:$XA, f32:$XB))]>;
  def XSDIVSP : XX3Form<60, 24,
                        (outs vssrc:$XT), (ins vssrc:$XA, vssrc:$XB),
                        "xsdivsp $XT, $XA, $XB", IIC_FPDivS,
                        [(set f32:$XT, (any_fdiv f32:$XA, f32:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1221-1230

```tablegen
  def XSRESP : XX2Form<60, 26,
                        (outs vssrc:$XT), (ins vssrc:$XB),
                        "xsresp $XT, $XB", IIC_VecFP,
                        [(set f32:$XT, (PPCfre f32:$XB))]>;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
  let hasSideEffects = 1 in
  def XSRSP : XX2Form<60, 281,
                        (outs vssrc:$XT), (ins vsfrc:$XB),
                        "xsrsp $XT, $XB", IIC_VecFP,
                        [(set f32:$XT, (any_fpround f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1231-1240

```tablegen
  def XSSQRTSP : XX2Form<60, 11,
                        (outs vssrc:$XT), (ins vssrc:$XB),
                        "xssqrtsp $XT, $XB", IIC_FPSqrtS,
                        [(set f32:$XT, (any_fsqrt f32:$XB))]>;
  def XSRSQRTESP : XX2Form<60, 10,
                           (outs vssrc:$XT), (ins vssrc:$XB),
                           "xsrsqrtesp $XT, $XB", IIC_VecFP,
                           [(set f32:$XT, (PPCfrsqrte f32:$XB))]>;

  // FMA Instructions
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1241-1250

```tablegen
  let BaseName = "XSMADDASP" in {
  let isCommutable = 1 in
  def XSMADDASP : XX3Form<60, 1,
                          (outs vssrc:$XT),
                          (ins vssrc:$XTi, vssrc:$XA, vssrc:$XB),
                          "xsmaddasp $XT, $XA, $XB", IIC_VecFP,
                          [(set f32:$XT, (any_fma f32:$XA, f32:$XB, f32:$XTi))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1251-1259

```tablegen
  let IsVSXFMAAlt = 1, hasSideEffects = 1 in
  def XSMADDMSP : XX3Form<60, 9,
                          (outs vssrc:$XT),
                          (ins vssrc:$XTi, vssrc:$XA, vssrc:$XB),
                          "xsmaddmsp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1260-1270

```tablegen
  let BaseName = "XSMSUBASP" in {
  let isCommutable = 1 in
  def XSMSUBASP : XX3Form<60, 17,
                          (outs vssrc:$XT),
                          (ins vssrc:$XTi, vssrc:$XA, vssrc:$XB),
                          "xsmsubasp $XT, $XA, $XB", IIC_VecFP,
                          [(set f32:$XT, (any_fma f32:$XA, f32:$XB,
                                              (fneg f32:$XTi)))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1271-1279

```tablegen
  let IsVSXFMAAlt = 1, hasSideEffects = 1 in
  def XSMSUBMSP : XX3Form<60, 25,
                          (outs vssrc:$XT),
                          (ins vssrc:$XTi, vssrc:$XA, vssrc:$XB),
                          "xsmsubmsp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1280-1290

```tablegen
  let BaseName = "XSNMADDASP" in {
  let isCommutable = 1 in
  def XSNMADDASP : XX3Form<60, 129,
                          (outs vssrc:$XT),
                          (ins vssrc:$XTi, vssrc:$XA, vssrc:$XB),
                          "xsnmaddasp $XT, $XA, $XB", IIC_VecFP,
                          [(set f32:$XT, (fneg (any_fma f32:$XA, f32:$XB,
                                                    f32:$XTi)))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 1291-1299

```tablegen
  let IsVSXFMAAlt = 1, hasSideEffects = 1 in
  def XSNMADDMSP : XX3Form<60, 137,
                          (outs vssrc:$XT),
                          (ins vssrc:$XTi, vssrc:$XA, vssrc:$XB),
                          "xsnmaddmsp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1300-1310

```tablegen
  let BaseName = "XSNMSUBASP" in {
  let isCommutable = 1 in
  def XSNMSUBASP : XX3Form<60, 145,
                          (outs vssrc:$XT),
                          (ins vssrc:$XTi, vssrc:$XA, vssrc:$XB),
                          "xsnmsubasp $XT, $XA, $XB", IIC_VecFP,
                          [(set f32:$XT, (fneg (any_fma f32:$XA, f32:$XB,
                                                    (fneg f32:$XTi))))]>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 1311-1320

```tablegen
  let IsVSXFMAAlt = 1, hasSideEffects = 1 in
  def XSNMSUBMSP : XX3Form<60, 153,
                          (outs vssrc:$XT),
                          (ins vssrc:$XTi, vssrc:$XA, vssrc:$XB),
                          "xsnmsubmsp $XT, $XA, $XB", IIC_VecFP, []>,
                          RegConstraint<"$XTi = $XT">,
                          AltVSXFMARel;
  }

  // Single Precision Conversions (FP <-> INT)
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1321-1331

```tablegen
  def XSCVSXDSP : XX2Form<60, 312,
                      (outs vssrc:$XT), (ins vsfrc:$XB),
                      "xscvsxdsp $XT, $XB", IIC_VecFP,
                      [(set f32:$XT, (PPCany_fcfids f64:$XB))]>;
  def XSCVUXDSP : XX2Form<60, 296,
                      (outs vssrc:$XT), (ins vsfrc:$XB),
                      "xscvuxdsp $XT, $XB", IIC_VecFP,
                      [(set f32:$XT, (PPCany_fcfidus f64:$XB))]>;
  } // mayRaiseFPException

  // Conversions between vector and scalar single precision
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1332-1338

```tablegen
  def XSCVDPSPN : XX2Form<60, 267, (outs vsrc:$XT), (ins vssrc:$XB),
                          "xscvdpspn $XT, $XB", IIC_VecFP, []>;
  def XSCVSPDPN : XX2Form<60, 331, (outs vssrc:$XT), (ins vsrc:$XB),
                          "xscvspdpn $XT, $XB", IIC_VecFP, []>;

  let Predicates = [HasVSX, HasDirectMove] in {
  // VSX direct move instructions
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range. These declarations feed generated pattern-matching logic.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。 这些声明会进入生成式模式匹配逻辑。

### Lines 1339-1348

```tablegen
  def MFVSRD : XX1_RS6_RD5_XO<31, 51, (outs g8rc:$RA), (ins vsfrc:$XT),
                              "mfvsrd $RA, $XT", IIC_VecGeneral,
                              [(set i64:$RA, (PPCmfvsr f64:$XT))]>,
      Requires<[IsPPC64]>;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
  let isCodeGenOnly = 1, hasSideEffects = 1 in
  def MFVRD : XX1_RS6_RD5_XO<31, 51, (outs g8rc:$RA), (ins vsrc:$XT),
                             "mfvsrd $RA, $XT", IIC_VecGeneral,
                             []>,
      Requires<[IsPPC64]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1349-1356

```tablegen
  def MFVSRWZ : XX1_RS6_RD5_XO<31, 115, (outs gprc:$RA), (ins vsfrc:$XT),
                               "mfvsrwz $RA, $XT", IIC_VecGeneral,
                               [(set i32:$RA, (PPCmfvsr f64:$XT))]>, ZExt32To64;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
  let isCodeGenOnly = 1, hasSideEffects = 1 in
  def MFVRWZ : XX1_RS6_RD5_XO<31, 115, (outs gprc:$RA), (ins vsrc:$XT),
                               "mfvsrwz $RA, $XT", IIC_VecGeneral,
                               []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1357-1366

```tablegen
  def MTVSRD : XX1_RS6_RD5_XO<31, 179, (outs vsfrc:$XT), (ins g8rc:$RA),
                              "mtvsrd $XT, $RA", IIC_VecGeneral,
                              [(set f64:$XT, (PPCmtvsra i64:$RA))]>,
      Requires<[IsPPC64]>;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
  let isCodeGenOnly = 1, hasSideEffects = 1 in
  def MTVRD : XX1_RS6_RD5_XO<31, 179, (outs vsrc:$XT), (ins g8rc:$RA),
                              "mtvsrd $XT, $RA", IIC_VecGeneral,
                              []>,
      Requires<[IsPPC64]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1367-1374

```tablegen
  def MTVSRWA : XX1_RS6_RD5_XO<31, 211, (outs vsfrc:$XT), (ins gprc:$RA),
                               "mtvsrwa $XT, $RA", IIC_VecGeneral,
                               [(set f64:$XT, (PPCmtvsra i32:$RA))]>;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
  let isCodeGenOnly = 1, hasSideEffects = 1 in
  def MTVRWA : XX1_RS6_RD5_XO<31, 211, (outs vsrc:$XT), (ins gprc:$RA),
                               "mtvsrwa $XT, $RA", IIC_VecGeneral,
                               []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1375-1386

```tablegen
  def MTVSRWZ : XX1_RS6_RD5_XO<31, 243, (outs vsfrc:$XT), (ins gprc:$RA),
                               "mtvsrwz $XT, $RA", IIC_VecGeneral,
                               [(set f64:$XT, (PPCmtvsrz i32:$RA))]>;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
  let isCodeGenOnly = 1, hasSideEffects = 1 in
  def MTVRWZ : XX1_RS6_RD5_XO<31, 243, (outs vsrc:$XT), (ins gprc:$RA),
                               "mtvsrwz $XT, $RA", IIC_VecGeneral,
                               []>;
  } // HasDirectMove

} // HasVSX, HasP8Vector
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1387-1394

```tablegen
let Predicates = [HasVSX, IsISA3_0, HasDirectMove] in {
def MTVSRWS: XX1_RS6_RD5_XO<31, 403, (outs vsrc:$XT), (ins gprc:$RA),
                            "mtvsrws $XT, $RA", IIC_VecGeneral, []>;

def MTVSRDD: XX1Form<31, 435, (outs vsrc:$XT), (ins g8rc_nox0:$RA, g8rc:$RB),
                     "mtvsrdd $XT, $RA, $RB", IIC_VecGeneral,
                     []>, Requires<[IsPPC64]>;
```
- **EN**: Adds declarative TableGen records such as `MTVSRWS`, `MTVSRDD` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MTVSRWS`, `MTVSRDD`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1395-1403

```tablegen
def MFVSRLD: XX1_RS6_RD5_XO<31, 307, (outs g8rc:$RA), (ins vsrc:$XT),
                            "mfvsrld $RA, $XT", IIC_VecGeneral,
                            []>, Requires<[IsPPC64]>;

} // HasVSX, IsISA3_0, HasDirectMove

let Predicates = [HasVSX, HasP9Vector] in {
  // Quad-Precision Scalar Move Instructions:
  // Copy Sign
```
- **EN**: Adds declarative TableGen records such as `MFVSRLD` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MFVSRLD`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1404-1410

```tablegen
  def XSCPSGNQP : X_VT5_VA5_VB5<63, 100, "xscpsgnqp",
                                [(set f128:$RST,
                                      (fcopysign f128:$RB, f128:$RA))]>;

  // Absolute/Negative-Absolute/Negate
  def XSABSQP   : X_VT5_XO5_VB5<63,  0, 804, "xsabsqp",
                                [(set f128:$RST, (fabs f128:$RB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1411-1415

```tablegen
  def XSNABSQP  : X_VT5_XO5_VB5<63,  8, 804, "xsnabsqp",
                                [(set f128:$RST, (fneg (fabs f128:$RB)))]>;
  def XSNEGQP   : X_VT5_XO5_VB5<63, 16, 804, "xsnegqp",
                                [(set f128:$RST, (fneg f128:$RB))]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1416-1423

```tablegen
  //===--------------------------------------------------------------------===//
  // Quad-Precision Scalar Floating-Point Arithmetic Instructions:

  // Add/Divide/Multiply/Subtract
  let mayRaiseFPException = 1 in {
  let isCommutable = 1 in {
  def XSADDQP   : X_VT5_VA5_VB5   <63,   4, "xsaddqp",
                                   [(set f128:$RST, (any_fadd f128:$RA, f128:$RB))]>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Quad-Precision Scalar Floating-Point Arithmetic Instructions:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Quad-Precision Scalar Floating-Point Arithmetic Instructions:”。

### Lines 1424-1431

```tablegen
  def XSMULQP   : X_VT5_VA5_VB5   <63,  36, "xsmulqp",
                                   [(set f128:$RST, (any_fmul f128:$RA, f128:$RB))]>;
  }
  def XSSUBQP   : X_VT5_VA5_VB5   <63, 516, "xssubqp" ,
                                   [(set f128:$RST, (any_fsub f128:$RA, f128:$RB))]>;
  def XSDIVQP   : X_VT5_VA5_VB5   <63, 548, "xsdivqp",
                                   [(set f128:$RST, (any_fdiv f128:$RA, f128:$RB))]>;
  // Square-Root
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1432-1441

```tablegen
  def XSSQRTQP  : X_VT5_XO5_VB5   <63, 27, 804, "xssqrtqp",
                                   [(set f128:$RST, (any_fsqrt f128:$RB))]>;
  // (Negative) Multiply-{Add/Subtract}
  def XSMADDQP : X_VT5_VA5_VB5_FMA <63, 388, "xsmaddqp",
                                    [(set f128:$RST,
                                          (any_fma f128:$RA, f128:$RB, f128:$RSTi))]>;
  def XSMSUBQP  : X_VT5_VA5_VB5_FMA   <63, 420, "xsmsubqp"  ,
                                       [(set f128:$RST,
                                             (any_fma f128:$RA, f128:$RB,
                                                      (fneg f128:$RSTi)))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1442-1450

```tablegen
  def XSNMADDQP : X_VT5_VA5_VB5_FMA <63, 452, "xsnmaddqp",
                                     [(set f128:$RST,
                                           (fneg (any_fma f128:$RA, f128:$RB,
                                                          f128:$RSTi)))]>;
  def XSNMSUBQP : X_VT5_VA5_VB5_FMA <63, 484, "xsnmsubqp",
                                     [(set f128:$RST,
                                           (fneg (any_fma f128:$RA, f128:$RB,
                                                          (fneg f128:$RSTi))))]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1451-1460

```tablegen
  let isCommutable = 1 in {
  def XSADDQPO : X_VT5_VA5_VB5_Ro<63, 4, "xsaddqpo",
                                  [(set f128:$RST,
                                  (int_ppc_addf128_round_to_odd
                                  f128:$RA, f128:$RB))]>;
  def XSMULQPO : X_VT5_VA5_VB5_Ro<63, 36, "xsmulqpo",
                                  [(set f128:$RST,
                                  (int_ppc_mulf128_round_to_odd
                                  f128:$RA, f128:$RB))]>;
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1461-1468

```tablegen
  def XSSUBQPO : X_VT5_VA5_VB5_Ro<63, 516, "xssubqpo",
                                  [(set f128:$RST,
                                  (int_ppc_subf128_round_to_odd
                                  f128:$RA, f128:$RB))]>;
  def XSDIVQPO : X_VT5_VA5_VB5_Ro<63, 548, "xsdivqpo",
                                  [(set f128:$RST,
                                  (int_ppc_divf128_round_to_odd
                                  f128:$RA, f128:$RB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1469-1478

```tablegen
  def XSSQRTQPO : X_VT5_XO5_VB5_Ro<63, 27, 804, "xssqrtqpo",
                                  [(set f128:$RST,
                                  (int_ppc_sqrtf128_round_to_odd f128:$RB))]>;


  def XSMADDQPO : X_VT5_VA5_VB5_FMA_Ro<63, 388, "xsmaddqpo",
                                      [(set f128:$RST,
                                      (int_ppc_fmaf128_round_to_odd
                                      f128:$RA,f128:$RB,f128:$RSTi))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1479-1486

```tablegen
  def XSMSUBQPO : X_VT5_VA5_VB5_FMA_Ro<63, 420, "xsmsubqpo" ,
                                      [(set f128:$RST,
                                      (int_ppc_fmaf128_round_to_odd
                                      f128:$RA, f128:$RB, (fneg f128:$RSTi)))]>;
  def XSNMADDQPO: X_VT5_VA5_VB5_FMA_Ro<63, 452, "xsnmaddqpo",
                                      [(set f128:$RST,
                                      (fneg (int_ppc_fmaf128_round_to_odd
                                      f128:$RA, f128:$RB, f128:$RSTi)))]>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1487-1494

```tablegen
  def XSNMSUBQPO: X_VT5_VA5_VB5_FMA_Ro<63, 484, "xsnmsubqpo",
                                      [(set f128:$RST,
                                      (fneg (int_ppc_fmaf128_round_to_odd
                                      f128:$RA, f128:$RB, (fneg f128:$RSTi))))]>;
  } // mayRaiseFPException

  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
  // QP Compare Ordered/Unordered
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FIXME: Setting the hasSideEffects flag here to match current behaviour.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FIXME: Setting the hasSideEffects flag here to match current behaviour.”。

### Lines 1495-1501

```tablegen
  let hasSideEffects = 1 in {
    // DP/QP Compare Exponents
    def XSCMPEXPDP : XX3Form_1<60, 59,
                               (outs crrc:$CR), (ins vsfrc:$XA, vsfrc:$XB),
                               "xscmpexpdp $CR, $XA, $XB", IIC_FPCompare, []>;
    def XSCMPEXPQP : X_BF3_VA5_VB5<63, 164, "xscmpexpqp", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1502-1508

```tablegen
    let mayRaiseFPException = 1 in {
    def XSCMPOQP : X_BF3_VA5_VB5<63, 132, "xscmpoqp", []>;
    def XSCMPUQP : X_BF3_VA5_VB5<63, 644, "xscmpuqp", []>;

    // DP Compare ==, >=, >, !=
    // Use vsrc for XT, because the entire register of XT is set.
    // XT.dword[1] = 0x0000_0000_0000_0000
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "DP Compare ==, >=, >, !=".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“DP Compare ==, >=, >, !=”。

### Lines 1509-1517

```tablegen
    def XSCMPEQDP : XX3_XT5_XA5_XB5<60,  3, "xscmpeqdp", vsrc, vsfrc, vsfrc,
                                    IIC_FPCompare, []>;
    def XSCMPGEDP : XX3_XT5_XA5_XB5<60, 19, "xscmpgedp", vsrc, vsfrc, vsfrc,
                                    IIC_FPCompare, []>;
    def XSCMPGTDP : XX3_XT5_XA5_XB5<60, 11, "xscmpgtdp", vsrc, vsfrc, vsfrc,
                                    IIC_FPCompare, []>;
    }
  }
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1518-1526

```tablegen
  //===--------------------------------------------------------------------===//
  // Quad-Precision Floating-Point Conversion Instructions:

  let mayRaiseFPException = 1 in {
    // Convert DP -> QP
    def XSCVDPQP  : X_VT5_XO5_VB5_TyVB<63, 22, 836, "xscvdpqp", vfrc,
                                       [(set f128:$RST, (any_fpextend f64:$RB))]>;

    // Round & Convert QP -> DP (dword[1] is set to zero)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Quad-Precision Floating-Point Conversion Instructions:".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Quad-Precision Floating-Point Conversion Instructions:”。

### Lines 1527-1534

```tablegen
    def XSCVQPDP  : X_VT5_XO5_VB5_VSFR<63, 20, 836, "xscvqpdp" , []>;
    def XSCVQPDPO : X_VT5_XO5_VB5_VSFR_Ro<63, 20, 836, "xscvqpdpo",
                                          [(set f64:$RST,
                                          (int_ppc_truncf128_round_to_odd
                                          f128:$RB))]>;
  }

  // Truncate & Convert QP -> (Un)Signed (D)Word (dword[1] is set to zero)
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1535-1541

```tablegen
  let mayRaiseFPException = 1 in {
    def XSCVQPSDZ : X_VT5_XO5_VB5<63, 25, 836, "xscvqpsdz",
        [(set f128:$RST, (PPCany_fctidz f128:$RB))]>;
    def XSCVQPSWZ : X_VT5_XO5_VB5<63,  9, 836, "xscvqpswz",
        [(set f128:$RST, (PPCany_fctiwz f128:$RB))]>;
    def XSCVQPUDZ : X_VT5_XO5_VB5<63, 17, 836, "xscvqpudz",
        [(set f128:$RST, (PPCany_fctiduz f128:$RB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1542-1554

```tablegen
    def XSCVQPUWZ : X_VT5_XO5_VB5<63,  1, 836, "xscvqpuwz",
        [(set f128:$RST, (PPCany_fctiwuz f128:$RB))]>;
  }

  // Convert (Un)Signed DWord -> QP.
  def XSCVSDQP  : X_VT5_XO5_VB5_TyVB<63, 10, 836, "xscvsdqp", vfrc, []>;
  def XSCVUDQP  : X_VT5_XO5_VB5_TyVB<63,  2, 836, "xscvudqp", vfrc, []>;

  // (Round &) Convert DP <-> HP
  // Note! xscvdphp's src and dest register both use the left 64 bits, so we use
  // vsfrc for src and dest register. xscvhpdp's src only use the left 16 bits,
  // but we still use vsfrc for it.
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Convert (Un)Signed DWord -> QP.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Convert (Un)Signed DWord -> QP.”。

### Lines 1555-1562

```tablegen
  let hasSideEffects = 1, mayRaiseFPException = 1 in {
    def XSCVDPHP : XX2_XT6_XO5_XB6<60, 17, 347, "xscvdphp", vsfrc, []>;
    def XSCVHPDP : XX2_XT6_XO5_XB6<60, 16, 347, "xscvhpdp", vsfrc, []>;
  }

  let mayRaiseFPException = 1 in {
  // Vector HP -> SP
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Vector HP -> SP".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Vector HP -> SP”。

### Lines 1563-1569

```tablegen
  let hasSideEffects = 1 in
  def XVCVHPSP : XX2_XT6_XO5_XB6<60, 24, 475, "xvcvhpsp", vsrc, []>;
  def XVCVSPHP : XX2_XT6_XO5_XB6<60, 25, 475, "xvcvsphp", vsrc,
                                 [(set v4f32:$XT,
                                     (int_ppc_vsx_xvcvsphp v4f32:$XB))]>;

  // Round to Quad-Precision Integer [with Inexact]
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1570-1578

```tablegen
  def XSRQPI   : Z23_VT5_R1_VB5_RMC2_EX1<63,  5, 0, "xsrqpi" , []>;
  def XSRQPIX  : Z23_VT5_R1_VB5_RMC2_EX1<63,  5, 1, "xsrqpix", []>;

  // Round Quad-Precision to Double-Extended Precision (fp80)
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
  let hasSideEffects = 1 in
  def XSRQPXP  : Z23_VT5_R1_VB5_RMC2_EX1<63, 37, 0, "xsrqpxp", []>;
  }
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1579-1586

```tablegen
  //===--------------------------------------------------------------------===//
  // Insert/Extract Instructions

  // Insert Exponent DP/QP
  // XT NOTE: XT.dword[1] = 0xUUUU_UUUU_UUUU_UUUU
  def XSIEXPDP : XX1Form <60, 918, (outs vsrc:$XT), (ins g8rc:$RA, g8rc:$RB),
                          "xsiexpdp $XT, $RA, $RB", IIC_VecFP, []>;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Insert/Extract Instructions".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Insert/Extract Instructions”。

### Lines 1587-1594

```tablegen
  let hasSideEffects = 1 in {
    // vB NOTE: only vB.dword[0] is used, that's why we don't use
    //          X_VT5_VA5_VB5 form
    def XSIEXPQP : XForm_18<63, 868, (outs vrrc:$FRT), (ins vrrc:$FRA, vsfrc:$FRB),
                            "xsiexpqp $FRT, $FRA, $FRB", IIC_VecFP, []>;
  }

  // Extract Exponent/Significand DP/QP
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "vB NOTE: only vB.dword[0] is used, that's why we don't use".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“vB NOTE: only vB.dword[0] is used, that's why we don't use”。

### Lines 1595-1605

```tablegen
  def XSXEXPDP : XX2_RT5_XO5_XB6<60,  0, 347, "xsxexpdp", []>;
  def XSXSIGDP : XX2_RT5_XO5_XB6<60,  1, 347, "xsxsigdp", []>;

  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
  let hasSideEffects = 1 in {
    def XSXEXPQP : X_VT5_XO5_VB5  <63,  2, 804, "xsxexpqp", []>;
    def XSXSIGQP : X_VT5_XO5_VB5  <63, 18, 804, "xsxsigqp", []>;
  }

  // Vector Insert Word
  // XB NOTE: Only XB.dword[1] is used, but we use vsrc on XB.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "FIXME: Setting the hasSideEffects flag here to match current behaviour.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“FIXME: Setting the hasSideEffects flag here to match current behaviour.”。

### Lines 1606-1615

```tablegen
  def XXINSERTW   :
    XX2_RD6_UIM5_RS6<60, 181, (outs vsrc:$XT),
                     (ins vsrc:$XTi, vsrc:$XB, u4imm:$UIM5),
                     "xxinsertw $XT, $XB, $UIM5", IIC_VecFP,
                     [(set v4i32:$XT, (PPCvecinsert v4i32:$XTi, v4i32:$XB,
                                                   imm32SExt16:$UIM5))]>,
                     RegConstraint<"$XTi = $XT">;

  // Vector Extract Unsigned Word
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1616-1623

```tablegen
  let hasSideEffects = 1 in
  def XXEXTRACTUW : XX2_RD6_UIM5_RS6<60, 165,
                                  (outs vsfrc:$XT), (ins vsrc:$XB, u4imm:$UIM5),
                                  "xxextractuw $XT, $XB, $UIM5", IIC_VecFP, []>;

  // Vector Insert Exponent DP/SP
  def XVIEXPDP : XX3_XT5_XA5_XB5<60, 248, "xviexpdp", vsrc, vsrc, vsrc,
    IIC_VecFP, [(set v2f64: $XT,(int_ppc_vsx_xviexpdp v2i64:$XA, v2i64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1624-1630

```tablegen
  def XVIEXPSP : XX3_XT5_XA5_XB5<60, 216, "xviexpsp", vsrc, vsrc, vsrc,
    IIC_VecFP, [(set v4f32: $XT,(int_ppc_vsx_xviexpsp v4i32:$XA, v4i32:$XB))]>;

  // Vector Extract Exponent/Significand DP/SP
  def XVXEXPDP : XX2_XT6_XO5_XB6<60,  0, 475, "xvxexpdp", vsrc,
                                 [(set v2i64: $XT,
                                  (int_ppc_vsx_xvxexpdp v2f64:$XB))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1631-1642

```tablegen
  def XVXEXPSP : XX2_XT6_XO5_XB6<60,  8, 475, "xvxexpsp", vsrc,
                                 [(set v4i32: $XT,
                                  (int_ppc_vsx_xvxexpsp v4f32:$XB))]>;
  def XVXSIGDP : XX2_XT6_XO5_XB6<60,  1, 475, "xvxsigdp", vsrc,
                                 [(set v2i64: $XT,
                                  (int_ppc_vsx_xvxsigdp v2f64:$XB))]>;
  def XVXSIGSP : XX2_XT6_XO5_XB6<60,  9, 475, "xvxsigsp", vsrc,
                                 [(set v4i32: $XT,
                                  (int_ppc_vsx_xvxsigsp v4f32:$XB))]>;

  // Test Data Class SP/DP/QP
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1643-1649

```tablegen
  let hasSideEffects = 1 in {
    def XSTSTDCSP : XX2_BF3_DCMX7_RS6<60, 298,
                                (outs crrc:$BF), (ins u7imm:$DCMX, vssrc:$XB),
                                "xststdcsp $BF, $XB, $DCMX", IIC_VecFP, []>;
    def XSTSTDCDP : XX2_BF3_DCMX7_RS6<60, 362,
                                (outs crrc:$BF), (ins u7imm:$DCMX, vsfrc:$XB),
                                "xststdcdp $BF, $XB, $DCMX", IIC_VecFP, []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1650-1661

```tablegen
    def XSTSTDCQP : X_BF3_DCMX7_RS5  <63, 708,
                                (outs crrc:$BF), (ins u7imm:$DCMX, vrrc:$VB),
                                "xststdcqp $BF, $VB, $DCMX", IIC_VecFP, []>;
  }

  // Vector Test Data Class SP/DP
  def XVTSTDCSP : XX2_RD6_DCMX7_RS6<60, 13, 5,
                              (outs vsrc:$XT), (ins u7imm:$DCMX, vsrc:$XB),
                              "xvtstdcsp $XT, $XB, $DCMX", IIC_VecFP,
                              [(set v4i32: $XT,
                               (int_ppc_vsx_xvtstdcsp v4f32:$XB,
                                                      u7imm_timm:$DCMX))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1662-1669

```tablegen
  def XVTSTDCDP : XX2_RD6_DCMX7_RS6<60, 15, 5,
                              (outs vsrc:$XT), (ins u7imm:$DCMX, vsrc:$XB),
                              "xvtstdcdp $XT, $XB, $DCMX", IIC_VecFP,
                              [(set v2i64: $XT,
                               (int_ppc_vsx_xvtstdcdp v2f64:$XB,
                                                      u7imm_timm:$DCMX))]>;

  // Maximum/Minimum Type-C/Type-J DP
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1670-1678

```tablegen
  let mayRaiseFPException = 1 in {
  def XSMAXCDP : XX3_XT5_XA5_XB5<60, 128, "xsmaxcdp", vsfrc, vsfrc, vsfrc,
                                 IIC_VecFP,
                                 [(set f64:$XT, (PPCxsmaxc f64:$XA, f64:$XB))]>;
  def XSMINCDP : XX3_XT5_XA5_XB5<60, 136, "xsmincdp", vsfrc, vsfrc, vsfrc,
                                 IIC_VecFP,
                                 [(set f64:$XT, (PPCxsminc f64:$XA, f64:$XB))]>;

  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1679-1688

```tablegen
  let hasSideEffects = 1 in {
    def XSMAXJDP : XX3_XT5_XA5_XB5<60, 144, "xsmaxjdp", vsrc, vsfrc, vsfrc,
                                   IIC_VecFP, []>;
    def XSMINJDP : XX3_XT5_XA5_XB5<60, 152, "xsminjdp", vsrc, vsfrc, vsfrc,
                                   IIC_VecFP, []>;
  }
  }

  // Vector Byte-Reverse H/W/D/Q Word
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1689-1695

```tablegen
  let hasSideEffects = 1 in
  def XXBRH : XX2_XT6_XO5_XB6<60,  7, 475, "xxbrh", vsrc, []>;
  def XXBRW : XX2_XT6_XO5_XB6<60, 15, 475, "xxbrw", vsrc,
    [(set v4i32:$XT, (bswap v4i32:$XB))]>;
  def XXBRD : XX2_XT6_XO5_XB6<60, 23, 475, "xxbrd", vsrc,
    [(set v2i64:$XT, (bswap v2i64:$XB))]>;
  // FIXME: Setting the hasSideEffects flag here to match current behaviour.
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1696-1703

```tablegen
  let hasSideEffects = 1 in
  def XXBRQ : XX2_XT6_XO5_XB6<60, 31, 475, "xxbrq", vsrc, []>;

  // Vector Permute
  def XXPERM  : XX3Form<60, 26, (outs vsrc:$XT),
                                (ins vsrc:$XA, vsrc:$XTi, vsrc:$XB),
                        "xxperm $XT, $XA, $XB", IIC_VecPerm, []>,
                        RegConstraint<"$XTi = $XT">;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1704-1714

```tablegen
  def XXPERMR : XX3Form<60, 58, (outs vsrc:$XT),
                                (ins vsrc:$XA, vsrc:$XTi, vsrc:$XB),
                        "xxpermr $XT, $XA, $XB", IIC_VecPerm, []>,
                        RegConstraint<"$XTi = $XT">;

  // Vector Splat Immediate Byte
  def XXSPLTIB : X_RD6_IMM8<60, 360, (outs vsrc:$XT), (ins u8imm_trunc:$IMM8),
                            "xxspltib $XT, $IMM8", IIC_VecPerm, []>;

  // When adding new D-Form loads/stores, be sure to update the ImmToIdxMap in
  // PPCRegisterInfo::PPCRegisterInfo and maybe save yourself some debugging.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Vector Splat Immediate Byte". Register assignment and register-class constraints matter here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Vector Splat Immediate Byte”。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 1715-1722

```tablegen
  let mayLoad = 1, mayStore = 0 in {
  // Load Vector
  def LXV : DQ_RD6_RS5_DQ12<61, 1, (outs vsrc:$XT), (ins (memrix16 $DQ, $RA):$addr),
                            "lxv $XT, $addr", IIC_LdStLFD, []>;
  // Load DWord
  def LXSD  : DSForm_1<57, 2, (outs vfrc:$RST), (ins (memrix $D, $RA):$addr),
                       "lxsd $RST, $addr", IIC_LdStLFD, []>;
  // Load SP from src, convert it to DP, and place in dword[0]
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load Vector".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load Vector”。

### Lines 1723-1732

```tablegen
  def LXSSP : DSForm_1<57, 3, (outs vfrc:$RST), (ins (memrix $D, $RA):$addr),
                       "lxssp $RST, $addr", IIC_LdStLFD, []>;

  // Load as Integer Byte/Halfword & Zero Indexed
  def LXSIBZX : X_XT6_RA5_RB5<31, 781, "lxsibzx", vsfrc,
                              [(set f64:$XT, (PPClxsizx ForceXForm:$addr, 1))]>;
  def LXSIHZX : X_XT6_RA5_RB5<31, 813, "lxsihzx", vsfrc,
                              [(set f64:$XT, (PPClxsizx ForceXForm:$addr, 2))]>;

  // Load Vector Halfword*8/Byte*16 Indexed
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1733-1739

```tablegen
  def LXVH8X  : X_XT6_RA5_RB5<31, 812, "lxvh8x" , vsrc, []>;
  def LXVB16X : X_XT6_RA5_RB5<31, 876, "lxvb16x", vsrc, []>;

  // Load Vector Indexed
  def LXVX    : X_XT6_RA5_RB5<31, 268, "lxvx"   , vsrc,
                [(set v2f64:$XT, (load XForm:$addr))]>;
  // Load Vector (Left-justified) with Length
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Load Vector Indexed".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Load Vector Indexed”。

### Lines 1740-1747

```tablegen
  def LXVL : XX1Form_memOp<31, 269, (outs vsrc:$XT), (ins (memr $RA):$addr, g8rc:$RB),
                   "lxvl $XT, $addr, $RB", IIC_LdStLoad,
                   [(set v4i32:$XT, (int_ppc_vsx_lxvl addr:$addr, i64:$RB))]>;
  def LXVLL : XX1Form_memOp<31,301, (outs vsrc:$XT), (ins (memr $RA):$addr, g8rc:$RB),
                   "lxvll $XT, $addr, $RB", IIC_LdStLoad,
                   [(set v4i32:$XT, (int_ppc_vsx_lxvll addr:$addr, i64:$RB))]>;

  // Load Vector Word & Splat Indexed
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 1748-1754

```tablegen
  def LXVWSX  : X_XT6_RA5_RB5<31, 364, "lxvwsx" , vsrc, []>;
  } // mayLoad

  // When adding new D-Form loads/stores, be sure to update the ImmToIdxMap in
  // PPCRegisterInfo::PPCRegisterInfo and maybe save yourself some debugging.
  let mayStore = 1, mayLoad = 0 in {
  // Store Vector
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "When adding new D-Form loads/stores, be sure to update the ImmToIdxMap in". Register assignment and register-class constraints matter here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“When adding new D-Form loads/stores, be sure to update the ImmToIdxMap in”。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 1755-1764

```tablegen
  def STXV : DQ_RD6_RS5_DQ12<61, 5, (outs), (ins vsrc:$XT, (memrix16 $DQ, $RA):$addr),
                             "stxv $XT, $addr", IIC_LdStSTFD, []>;
  // Store DWord
  def STXSD  : DSForm_1<61, 2, (outs), (ins vfrc:$RST, (memrix $D, $RA):$addr),
                        "stxsd $RST, $addr", IIC_LdStSTFD, []>;
  // Convert DP of dword[0] to SP, and Store to dst
  def STXSSP : DSForm_1<61, 3, (outs), (ins vfrc:$RST, (memrix $D, $RA):$addr),
                        "stxssp $RST, $addr", IIC_LdStSTFD, []>;

  // Store as Integer Byte/Halfword Indexed
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Store DWord".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Store DWord”。

### Lines 1765-1774

```tablegen
  def STXSIBX  : X_XS6_RA5_RB5<31,  909, "stxsibx" , vsfrc,
                               [(PPCstxsix f64:$XT, ForceXForm:$addr, 1)]>;
  def STXSIHX  : X_XS6_RA5_RB5<31,  941, "stxsihx" , vsfrc,
                               [(PPCstxsix f64:$XT, ForceXForm:$addr, 2)]>;
  let isCodeGenOnly = 1 in {
    def STXSIBXv  : X_XS6_RA5_RB5<31,  909, "stxsibx" , vsrc, []>;
    def STXSIHXv  : X_XS6_RA5_RB5<31,  941, "stxsihx" , vsrc, []>;
  }

  // Store Vector Halfword*8/Byte*16 Indexed
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1775-1782

```tablegen
  def STXVH8X  : X_XS6_RA5_RB5<31,  940, "stxvh8x" , vsrc, []>;
  def STXVB16X : X_XS6_RA5_RB5<31, 1004, "stxvb16x", vsrc, []>;

  // Store Vector Indexed
  def STXVX    : X_XS6_RA5_RB5<31,  396, "stxvx"   , vsrc,
                 [(store v2f64:$XT, XForm:$addr)]>;

  // Store Vector (Left-justified) with Length
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Store Vector Indexed".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Store Vector Indexed”。

### Lines 1783-1794

```tablegen
  def STXVL : XX1Form_memOp<31, 397, (outs),
                            (ins vsrc:$XT, (memr $RA):$addr, g8rc:$RB),
                            "stxvl $XT, $addr, $RB", IIC_LdStLoad,
                            [(int_ppc_vsx_stxvl v4i32:$XT, addr:$addr,
                              i64:$RB)]>;
  def STXVLL : XX1Form_memOp<31, 429, (outs),
                            (ins vsrc:$XT, (memr $RA):$addr, g8rc:$RB),
                            "stxvll $XT, $addr, $RB", IIC_LdStLoad,
                            [(int_ppc_vsx_stxvll v4i32:$XT, addr:$addr,
                              i64:$RB)]>;
  } // mayStore
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1795-1803

```tablegen
  def DFLOADf32  : PPCPostRAExpPseudo<(outs vssrc:$XT), (ins memrix:$src),
                          "#DFLOADf32",
                          [(set f32:$XT, (load DSForm:$src))]>;
  def DFLOADf64  : PPCPostRAExpPseudo<(outs vsfrc:$XT), (ins memrix:$src),
                          "#DFLOADf64",
                          [(set f64:$XT, (load DSForm:$src))]>;
  def DFSTOREf32 : PPCPostRAExpPseudo<(outs), (ins vssrc:$XT, memrix:$dst),
                          "#DFSTOREf32",
                          [(store f32:$XT, DSForm:$dst)]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1804-1811

```tablegen
  def DFSTOREf64 : PPCPostRAExpPseudo<(outs), (ins vsfrc:$XT, memrix:$dst),
                          "#DFSTOREf64",
                          [(store f64:$XT, DSForm:$dst)]>;

  let mayStore = 1 in {
    def SPILLTOVSR_STX : PseudoXFormMemOp<(outs),
                                          (ins spilltovsrrc:$XT, memrr:$dst),
                                          "#SPILLTOVSR_STX", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1812-1818

```tablegen
    def SPILLTOVSR_ST : PPCPostRAExpPseudo<(outs), (ins spilltovsrrc:$XT, memrix:$dst),
                              "#SPILLTOVSR_ST", []>;
  }
  let mayLoad = 1 in {
    def SPILLTOVSR_LDX : PseudoXFormMemOp<(outs spilltovsrrc:$XT),
                                          (ins memrr:$src),
                                          "#SPILLTOVSR_LDX", []>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1819-1825

```tablegen
    def SPILLTOVSR_LD : PPCPostRAExpPseudo<(outs spilltovsrrc:$XT), (ins memrix:$src),
                              "#SPILLTOVSR_LD", []>;

  }
  } // HasP9Vector
} // hasSideEffects = 0
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1826-1836

```tablegen
let PPC970_Single = 1, AddedComplexity = 400 in {

  def SELECT_CC_VSRC: PPCCustomInserterPseudo<(outs vsrc:$dst),
                             (ins crrc:$cond, vsrc:$T, vsrc:$F, i32imm:$BROPC),
                             "#SELECT_CC_VSRC",
                             []>;
  def SELECT_VSRC: PPCCustomInserterPseudo<(outs vsrc:$dst),
                          (ins crbitrc:$cond, vsrc:$T, vsrc:$F),
                          "#SELECT_VSRC",
                          [(set v2f64:$dst,
                                (select i1:$cond, v2f64:$T, v2f64:$F))]>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1837-1845

```tablegen
  def SELECT_CC_VSFRC: PPCCustomInserterPseudo<(outs f8rc:$dst),
                              (ins crrc:$cond, f8rc:$T, f8rc:$F,
                               i32imm:$BROPC), "#SELECT_CC_VSFRC",
                              []>;
  def SELECT_VSFRC: PPCCustomInserterPseudo<(outs f8rc:$dst),
                           (ins crbitrc:$cond, f8rc:$T, f8rc:$F),
                           "#SELECT_VSFRC",
                           [(set f64:$dst,
                                 (select i1:$cond, f64:$T, f64:$F))]>;
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1846-1860

```tablegen
  def SELECT_CC_VSSRC: PPCCustomInserterPseudo<(outs f4rc:$dst),
                              (ins crrc:$cond, f4rc:$T, f4rc:$F,
                               i32imm:$BROPC), "#SELECT_CC_VSSRC",
                              []>;
  def SELECT_VSSRC: PPCCustomInserterPseudo<(outs f4rc:$dst),
                           (ins crbitrc:$cond, f4rc:$T, f4rc:$F),
                           "#SELECT_VSSRC",
                           [(set f32:$dst,
                                 (select i1:$cond, f32:$T, f32:$F))]>;
}
}

//----------------------------- DAG Definitions ------------------------------//

// Output dag used to bitcast f32 to i32 and f64 to i64
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 1861-1874

```tablegen
def Bitcast {
  dag FltToInt = (i32 (MFVSRWZ (EXTRACT_SUBREG (XSCVDPSPN $A), sub_64)));
  dag DblToLong = (i64 (MFVSRD $A));
}

def FpMinMax {
  dag F32Min = (COPY_TO_REGCLASS (XSMINDP (COPY_TO_REGCLASS $A, VSFRC),
                                          (COPY_TO_REGCLASS $B, VSFRC)),
                                 VSSRC);
  dag F32Max = (COPY_TO_REGCLASS (XSMAXDP (COPY_TO_REGCLASS $A, VSFRC),
                                          (COPY_TO_REGCLASS $B, VSFRC)),
                                 VSSRC);
}
```
- **EN**: Adds declarative TableGen records such as `Bitcast`, `FpMinMax` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `Bitcast`, `FpMinMax`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1875-1890

```tablegen
def ScalarLoads {
  dag Li8 =       (i32 (extloadi8 ForceXForm:$src));
  dag ZELi8 =     (i32 (zextloadi8 ForceXForm:$src));
  dag ZELi8i64 =  (i64 (zextloadi8 ForceXForm:$src));
  dag SELi8 =     (i32 (sext_inreg (extloadi8 ForceXForm:$src), i8));
  dag SELi8i64 =  (i64 (sext_inreg (extloadi8 ForceXForm:$src), i8));

  dag Li16 =      (i32 (extloadi16 ForceXForm:$src));
  dag ZELi16 =    (i32 (zextloadi16 ForceXForm:$src));
  dag ZELi16i64 = (i64 (zextloadi16 ForceXForm:$src));
  dag SELi16 =    (i32 (sextloadi16 ForceXForm:$src));
  dag SELi16i64 = (i64 (sextloadi16 ForceXForm:$src));

  dag Li32 = (i32 (load ForceXForm:$src));
}
```
- **EN**: Adds declarative TableGen records such as `ScalarLoads` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ScalarLoads`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1891-1911

```tablegen
def DWToSPExtractConv {
  dag El0US1 = (f32 (PPCfcfidus
                    (f64 (PPCmtvsra (i64 (vector_extract v2i64:$S1, 0))))));
  dag El1US1 = (f32 (PPCfcfidus
                    (f64 (PPCmtvsra (i64 (vector_extract v2i64:$S1, 1))))));
  dag El0US2 = (f32 (PPCfcfidus
                    (f64 (PPCmtvsra (i64 (vector_extract v2i64:$S2, 0))))));
  dag El1US2 = (f32 (PPCfcfidus
                    (f64 (PPCmtvsra (i64 (vector_extract v2i64:$S2, 1))))));
  dag El0SS1 = (f32 (PPCfcfids
                    (f64 (PPCmtvsra (i64 (vector_extract v2i64:$S1, 0))))));
  dag El1SS1 = (f32 (PPCfcfids
                    (f64 (PPCmtvsra (i64 (vector_extract v2i64:$S1, 1))))));
  dag El0SS2 = (f32 (PPCfcfids
                    (f64 (PPCmtvsra (i64 (vector_extract v2i64:$S2, 0))))));
  dag El1SS2 = (f32 (PPCfcfids
                    (f64 (PPCmtvsra (i64 (vector_extract v2i64:$S2, 1))))));
  dag BVU = (v4f32 (build_vector El0US1, El1US1, El0US2, El1US2));
  dag BVS = (v4f32 (build_vector El0SS1, El1SS1, El0SS2, El1SS2));
}
```
- **EN**: Adds declarative TableGen records such as `DWToSPExtractConv` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DWToSPExtractConv`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1912-1931

```tablegen
def WToDPExtractConv {
  dag El0S = (f64 (PPCfcfid (PPCmtvsra (extractelt v4i32:$A, 0))));
  dag El1S = (f64 (PPCfcfid (PPCmtvsra (extractelt v4i32:$A, 1))));
  dag El2S = (f64 (PPCfcfid (PPCmtvsra (extractelt v4i32:$A, 2))));
  dag El3S = (f64 (PPCfcfid (PPCmtvsra (extractelt v4i32:$A, 3))));
  dag El0U = (f64 (PPCfcfidu (PPCmtvsrz (extractelt v4i32:$A, 0))));
  dag El1U = (f64 (PPCfcfidu (PPCmtvsrz (extractelt v4i32:$A, 1))));
  dag El2U = (f64 (PPCfcfidu (PPCmtvsrz (extractelt v4i32:$A, 2))));
  dag El3U = (f64 (PPCfcfidu (PPCmtvsrz (extractelt v4i32:$A, 3))));
  dag BV02S = (v2f64 (build_vector El0S, El2S));
  dag BV13S = (v2f64 (build_vector El1S, El3S));
  dag BV02U = (v2f64 (build_vector El0U, El2U));
  dag BV13U = (v2f64 (build_vector El1U, El3U));
}

/*  Direct moves of various widths from GPR's into VSR's. Each move lines
    the value up into element 0 (both BE and LE). Namely, entities smaller than
    a doubleword are shifted left and moved for BE. For LE, they're moved, then
    swapped to go into the least significant element of the VSR.
*/
```
- **EN**: Adds declarative TableGen records such as `WToDPExtractConv` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WToDPExtractConv`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 1932-1962

```tablegen
def MovesToVSR {
  dag BE_BYTE_0 =
    (MTVSRD
      (RLDICR
        (INSERT_SUBREG (i64 (IMPLICIT_DEF)), $A, sub_32), 56, 7));
  dag BE_HALF_0 =
    (MTVSRD
      (RLDICR
        (INSERT_SUBREG (i64 (IMPLICIT_DEF)), $A, sub_32), 48, 15));
  dag BE_WORD_0 =
    (MTVSRD
      (RLDICR
        (INSERT_SUBREG (i64 (IMPLICIT_DEF)), $A, sub_32), 32, 31));
  dag BE_DWORD_0 = (MTVSRD $A);

  dag LE_MTVSRW = (MTVSRD (INSERT_SUBREG (i64 (IMPLICIT_DEF)), $A, sub_32));
  dag LE_WORD_1 = (v2i64 (INSERT_SUBREG (v2i64 (IMPLICIT_DEF)),
                                        LE_MTVSRW, sub_64));
  dag LE_WORD_0 = (XXPERMDI LE_WORD_1, LE_WORD_1, 2);
  dag LE_DWORD_1 = (v2i64 (INSERT_SUBREG (v2i64 (IMPLICIT_DEF)),
                                         BE_DWORD_0, sub_64));
  dag LE_DWORD_0 = (XXPERMDI LE_DWORD_1, LE_DWORD_1, 2);
}

/*  Patterns for extracting elements out of vectors. Integer elements are
    extracted using direct move operations. Patterns for extracting elements
    whose indices are not available at compile time are also provided with
    various _VARIABLE_ patterns.
    The numbering for the DAG's is for LE, but when used on BE, the correct
    LE element can just be used (i.e. LE_BYTE_2 == BE_BYTE_13).
*/
```
- **EN**: Adds declarative TableGen records such as `MovesToVSR` that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MovesToVSR`，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 1963-2000

```tablegen
def VectorExtractions {
  // Doubleword extraction
  dag LE_DWORD_0 =
    (MFVSRD
      (EXTRACT_SUBREG
        (XXPERMDI (COPY_TO_REGCLASS $S, VSRC),
                  (COPY_TO_REGCLASS $S, VSRC), 2), sub_64));
  dag LE_DWORD_1 = (MFVSRD
                     (EXTRACT_SUBREG
                       (v2i64 (COPY_TO_REGCLASS $S, VSRC)), sub_64));

  // Word extraction
  dag LE_WORD_0 = (MFVSRWZ (EXTRACT_SUBREG (XXPERMDI $S, $S, 2), sub_64));
  dag LE_WORD_1 = (MFVSRWZ (EXTRACT_SUBREG (XXSLDWI $S, $S, 1), sub_64));
  dag LE_WORD_2 = (MFVSRWZ (EXTRACT_SUBREG
                             (v2i64 (COPY_TO_REGCLASS $S, VSRC)), sub_64));
  dag LE_WORD_3 = (MFVSRWZ (EXTRACT_SUBREG (XXSLDWI $S, $S, 3), sub_64));

  // Halfword extraction
  dag LE_HALF_0 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 0, 48), sub_32));
  dag LE_HALF_1 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 48, 48), sub_32));
  dag LE_HALF_2 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 32, 48), sub_32));
  dag LE_HALF_3 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 16, 48), sub_32));
  dag LE_HALF_4 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 0, 48), sub_32));
  dag LE_HALF_5 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 48, 48), sub_32));
  dag LE_HALF_6 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 32, 48), sub_32));
  dag LE_HALF_7 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 16, 48), sub_32));

  // Byte extraction
  dag LE_BYTE_0 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 0, 56), sub_32));
  dag LE_BYTE_1 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 56, 56), sub_32));
  dag LE_BYTE_2 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 48, 56), sub_32));
  dag LE_BYTE_3 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 40, 56), sub_32));
  dag LE_BYTE_4 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 32, 56), sub_32));
  dag LE_BYTE_5 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 24, 56), sub_32));
  dag LE_BYTE_6 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 16, 56), sub_32));
  dag LE_BYTE_7 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_0, 8, 56), sub_32));
  dag LE_BYTE_8 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 0, 56), sub_32));
```
- **EN**: Adds declarative TableGen records such as `VectorExtractions` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `VectorExtractions`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2001-2038

```tablegen
  dag LE_BYTE_9 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 56, 56), sub_32));
  dag LE_BYTE_10 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 48, 56), sub_32));
  dag LE_BYTE_11 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 40, 56), sub_32));
  dag LE_BYTE_12 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 32, 56), sub_32));
  dag LE_BYTE_13 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 24, 56), sub_32));
  dag LE_BYTE_14 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 16, 56), sub_32));
  dag LE_BYTE_15 = (i32 (EXTRACT_SUBREG (RLDICL LE_DWORD_1, 8, 56), sub_32));

  /* Variable element number (BE and LE patterns must be specified separately)
     This is a rather involved process.

     Conceptually, this is how the move is accomplished:
     1. Identify which doubleword contains the element
     2. Shift in the VMX register so that the correct doubleword is correctly
        lined up for the MFVSRD
     3. Perform the move so that the element (along with some extra stuff)
        is in the GPR
     4. Right shift within the GPR so that the element is right-justified

     Of course, the index is an element number which has a different meaning
     on LE/BE so the patterns have to be specified separately.

     Note: The final result will be the element right-justified with high
           order bits being arbitrarily defined (namely, whatever was in the
           vector register to the left of the value originally).
  */

  /*  LE variable byte
      Number 1. above:
      - For elements 0-7, we shift left by 8 bytes since they're on the right
      - For elements 8-15, we need not shift (shift left by zero bytes)
      This is accomplished by inverting the bits of the index and AND-ing
      with 0x8 (i.e. clearing all bits of the index and inverting bit 60).
  */
  dag LE_VBYTE_PERM_VEC = (v16i8 (LVSL ZERO8, (ANDC8 (LI8 8), $Idx)));

  //  Number 2. above:
  //  - Now that we set up the shift amount, we shift in the VMX register
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2039-2076

```tablegen
  dag LE_VBYTE_PERMUTE = (v16i8 (VPERM $S, $S, LE_VBYTE_PERM_VEC));

  //  Number 3. above:
  //  - The doubleword containing our element is moved to a GPR
  dag LE_MV_VBYTE = (MFVSRD
                      (EXTRACT_SUBREG
                        (v2i64 (COPY_TO_REGCLASS LE_VBYTE_PERMUTE, VSRC)),
                        sub_64));

  /*  Number 4. above:
      - Truncate the element number to the range 0-7 (8-15 are symmetrical
        and out of range values are truncated accordingly)
      - Multiply by 8 as we need to shift right by the number of bits, not bytes
      - Shift right in the GPR by the calculated value
  */
  dag LE_VBYTE_SHIFT = (EXTRACT_SUBREG (RLDICR (AND8 (LI8 7), $Idx), 3, 60),
                                       sub_32);
  dag LE_VARIABLE_BYTE = (EXTRACT_SUBREG (SRD LE_MV_VBYTE, LE_VBYTE_SHIFT),
                                         sub_32);

  /*  LE variable halfword
      Number 1. above:
      - For elements 0-3, we shift left by 8 since they're on the right
      - For elements 4-7, we need not shift (shift left by zero bytes)
      Similarly to the byte pattern, we invert the bits of the index, but we
      AND with 0x4 (i.e. clear all bits of the index and invert bit 61).
      Of course, the shift is still by 8 bytes, so we must multiply by 2.
  */
  dag LE_VHALF_PERM_VEC =
    (v16i8 (LVSL ZERO8, (RLDICR (ANDC8 (LI8 4), $Idx), 1, 62)));

  //  Number 2. above:
  //  - Now that we set up the shift amount, we shift in the VMX register
  dag LE_VHALF_PERMUTE = (v16i8 (VPERM $S, $S, LE_VHALF_PERM_VEC));

  //  Number 3. above:
  //  - The doubleword containing our element is moved to a GPR
  dag LE_MV_VHALF = (MFVSRD
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2077-2114

```tablegen
                      (EXTRACT_SUBREG
                        (v2i64 (COPY_TO_REGCLASS LE_VHALF_PERMUTE, VSRC)),
                        sub_64));

  /*  Number 4. above:
      - Truncate the element number to the range 0-3 (4-7 are symmetrical
        and out of range values are truncated accordingly)
      - Multiply by 16 as we need to shift right by the number of bits
      - Shift right in the GPR by the calculated value
  */
  dag LE_VHALF_SHIFT = (EXTRACT_SUBREG (RLDICR (AND8 (LI8 3), $Idx), 4, 59),
                                       sub_32);
  dag LE_VARIABLE_HALF = (EXTRACT_SUBREG (SRD LE_MV_VHALF, LE_VHALF_SHIFT),
                                         sub_32);

  /*  LE variable word
      Number 1. above:
      - For elements 0-1, we shift left by 8 since they're on the right
      - For elements 2-3, we need not shift
  */
  dag LE_VWORD_PERM_VEC = (v16i8 (LVSL ZERO8,
                                       (RLDICR (ANDC8 (LI8 2), $Idx), 2, 61)));

  //  Number 2. above:
  //  - Now that we set up the shift amount, we shift in the VMX register
  dag LE_VWORD_PERMUTE = (v16i8 (VPERM $S, $S, LE_VWORD_PERM_VEC));

  //  Number 3. above:
  //  - The doubleword containing our element is moved to a GPR
  dag LE_MV_VWORD = (MFVSRD
                      (EXTRACT_SUBREG
                        (v2i64 (COPY_TO_REGCLASS LE_VWORD_PERMUTE, VSRC)),
                        sub_64));

  /*  Number 4. above:
      - Truncate the element number to the range 0-1 (2-3 are symmetrical
        and out of range values are truncated accordingly)
      - Multiply by 32 as we need to shift right by the number of bits
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2115-2152

```tablegen
      - Shift right in the GPR by the calculated value
  */
  dag LE_VWORD_SHIFT = (EXTRACT_SUBREG (RLDICR (AND8 (LI8 1), $Idx), 5, 58),
                                       sub_32);
  dag LE_VARIABLE_WORD = (EXTRACT_SUBREG (SRD LE_MV_VWORD, LE_VWORD_SHIFT),
                                         sub_32);

  /*  LE variable doubleword
      Number 1. above:
      - For element 0, we shift left by 8 since it's on the right
      - For element 1, we need not shift
  */
  dag LE_VDWORD_PERM_VEC = (v16i8 (LVSL ZERO8,
                                        (RLDICR (ANDC8 (LI8 1), $Idx), 3, 60)));

  //  Number 2. above:
  //  - Now that we set up the shift amount, we shift in the VMX register
  dag LE_VDWORD_PERMUTE = (v16i8 (VPERM $S, $S, LE_VDWORD_PERM_VEC));

  // Number 3. above:
  //  - The doubleword containing our element is moved to a GPR
  //  - Number 4. is not needed for the doubleword as the value is 64-bits
  dag LE_VARIABLE_DWORD =
        (MFVSRD (EXTRACT_SUBREG
                  (v2i64 (COPY_TO_REGCLASS LE_VDWORD_PERMUTE, VSRC)),
                  sub_64));

  /*  LE variable float
      - Shift the vector to line up the desired element to BE Word 0
      - Convert 32-bit float to a 64-bit single precision float
  */
  dag LE_VFLOAT_PERM_VEC = (v16i8 (LVSL ZERO8,
                                  (RLDICR (XOR8 (LI8 3), $Idx), 2, 61)));
  dag LE_VFLOAT_PERMUTE = (VPERM $S, $S, LE_VFLOAT_PERM_VEC);
  dag LE_VARIABLE_FLOAT = (XSCVSPDPN LE_VFLOAT_PERMUTE);

  /*  LE variable double
      Same as the LE doubleword except there is no move.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2153-2190

```tablegen
  */
  dag LE_VDOUBLE_PERMUTE = (v16i8 (VPERM (v16i8 (COPY_TO_REGCLASS $S, VRRC)),
                                         (v16i8 (COPY_TO_REGCLASS $S, VRRC)),
                                         LE_VDWORD_PERM_VEC));
  dag LE_VARIABLE_DOUBLE = (COPY_TO_REGCLASS LE_VDOUBLE_PERMUTE, VSRC);

  /*  BE variable byte
      The algorithm here is the same as the LE variable byte except:
      - The shift in the VMX register is by 0/8 for opposite element numbers so
        we simply AND the element number with 0x8
      - The order of elements after the move to GPR is reversed, so we invert
        the bits of the index prior to truncating to the range 0-7
  */
  dag BE_VBYTE_PERM_VEC = (v16i8 (LVSL ZERO8, (ANDI8_rec $Idx, 8)));
  dag BE_VBYTE_PERMUTE = (v16i8 (VPERM $S, $S, BE_VBYTE_PERM_VEC));
  dag BE_MV_VBYTE = (MFVSRD
                      (EXTRACT_SUBREG
                        (v2i64 (COPY_TO_REGCLASS BE_VBYTE_PERMUTE, VSRC)),
                        sub_64));
  dag BE_VBYTE_SHIFT = (EXTRACT_SUBREG (RLDICR (ANDC8 (LI8 7), $Idx), 3, 60),
                                       sub_32);
  dag BE_VARIABLE_BYTE = (EXTRACT_SUBREG (SRD BE_MV_VBYTE, BE_VBYTE_SHIFT),
                                         sub_32);

  /*  BE variable halfword
      The algorithm here is the same as the LE variable halfword except:
      - The shift in the VMX register is by 0/8 for opposite element numbers so
        we simply AND the element number with 0x4 and multiply by 2
      - The order of elements after the move to GPR is reversed, so we invert
        the bits of the index prior to truncating to the range 0-3
  */
  dag BE_VHALF_PERM_VEC = (v16i8 (LVSL ZERO8,
                                       (RLDICR (ANDI8_rec $Idx, 4), 1, 62)));
  dag BE_VHALF_PERMUTE = (v16i8 (VPERM $S, $S, BE_VHALF_PERM_VEC));
  dag BE_MV_VHALF = (MFVSRD
                      (EXTRACT_SUBREG
                        (v2i64 (COPY_TO_REGCLASS BE_VHALF_PERMUTE, VSRC)),
                        sub_64));
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2191-2228

```tablegen
  dag BE_VHALF_SHIFT = (EXTRACT_SUBREG (RLDICR (ANDC8 (LI8 3), $Idx), 4, 59),
                                       sub_32);
  dag BE_VARIABLE_HALF = (EXTRACT_SUBREG (SRD BE_MV_VHALF, BE_VHALF_SHIFT),
                                         sub_32);

  /*  BE variable word
      The algorithm is the same as the LE variable word except:
      - The shift in the VMX register happens for opposite element numbers
      - The order of elements after the move to GPR is reversed, so we invert
        the bits of the index prior to truncating to the range 0-1
  */
  dag BE_VWORD_PERM_VEC = (v16i8 (LVSL ZERO8,
                                       (RLDICR (ANDI8_rec $Idx, 2), 2, 61)));
  dag BE_VWORD_PERMUTE = (v16i8 (VPERM $S, $S, BE_VWORD_PERM_VEC));
  dag BE_MV_VWORD = (MFVSRD
                      (EXTRACT_SUBREG
                        (v2i64 (COPY_TO_REGCLASS BE_VWORD_PERMUTE, VSRC)),
                        sub_64));
  dag BE_VWORD_SHIFT = (EXTRACT_SUBREG (RLDICR (ANDC8 (LI8 1), $Idx), 5, 58),
                                       sub_32);
  dag BE_VARIABLE_WORD = (EXTRACT_SUBREG (SRD BE_MV_VWORD, BE_VWORD_SHIFT),
                                         sub_32);

  /*  BE variable doubleword
      Same as the LE doubleword except we shift in the VMX register for opposite
      element indices.
  */
  dag BE_VDWORD_PERM_VEC = (v16i8 (LVSL ZERO8,
                                        (RLDICR (ANDI8_rec $Idx, 1), 3, 60)));
  dag BE_VDWORD_PERMUTE = (v16i8 (VPERM $S, $S, BE_VDWORD_PERM_VEC));
  dag BE_VARIABLE_DWORD =
        (MFVSRD (EXTRACT_SUBREG
                  (v2i64 (COPY_TO_REGCLASS BE_VDWORD_PERMUTE, VSRC)),
                  sub_64));

  /*  BE variable float
      - Shift the vector to line up the desired element to BE Word 0
      - Convert 32-bit float to a 64-bit single precision float
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2229-2255

```tablegen
  */
  dag BE_VFLOAT_PERM_VEC = (v16i8 (LVSL ZERO8, (RLDICR $Idx, 2, 61)));
  dag BE_VFLOAT_PERMUTE = (VPERM $S, $S, BE_VFLOAT_PERM_VEC);
  dag BE_VARIABLE_FLOAT = (XSCVSPDPN BE_VFLOAT_PERMUTE);

  //  BE variable float 32-bit version
  dag BE_32B_VFLOAT_PERM_VEC = (v16i8 (LVSL (i32 ZERO), (RLWINM $Idx, 2, 0, 29)));
  dag BE_32B_VFLOAT_PERMUTE = (VPERM $S, $S, BE_32B_VFLOAT_PERM_VEC);
  dag BE_32B_VARIABLE_FLOAT = (XSCVSPDPN BE_32B_VFLOAT_PERMUTE);

  /* BE variable double
      Same as the BE doubleword except there is no move.
  */
  dag BE_VDOUBLE_PERMUTE = (v16i8 (VPERM (v16i8 (COPY_TO_REGCLASS $S, VRRC)),
                                         (v16i8 (COPY_TO_REGCLASS $S, VRRC)),
                                         BE_VDWORD_PERM_VEC));
  dag BE_VARIABLE_DOUBLE = (COPY_TO_REGCLASS BE_VDOUBLE_PERMUTE, VSRC);

  //  BE variable double 32-bit version
  dag BE_32B_VDWORD_PERM_VEC = (v16i8 (LVSL (i32 ZERO),
                                        (RLWINM (ANDI_rec $Idx, 1), 3, 0, 28)));
  dag BE_32B_VDOUBLE_PERMUTE = (v16i8 (VPERM (v16i8 (COPY_TO_REGCLASS $S, VRRC)),
                                      (v16i8 (COPY_TO_REGCLASS $S, VRRC)),
                                      BE_32B_VDWORD_PERM_VEC));
  dag BE_32B_VARIABLE_DOUBLE = (COPY_TO_REGCLASS BE_32B_VDOUBLE_PERMUTE, VSRC);
}
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 2256-2268

```tablegen
def AlignValues {
  dag F32_TO_BE_WORD1 = (v4f32 (XSCVDPSPN $B));
  dag I32_TO_BE_WORD1 = (SUBREG_TO_REG (MTVSRWZ $B), sub_64);
}

// Integer extend helper dags 32 -> 64
def AnyExts {
  dag A = (INSERT_SUBREG (i64 (IMPLICIT_DEF)), $A, sub_32);
  dag B = (INSERT_SUBREG (i64 (IMPLICIT_DEF)), $B, sub_32);
  dag C = (INSERT_SUBREG (i64 (IMPLICIT_DEF)), $C, sub_32);
  dag D = (INSERT_SUBREG (i64 (IMPLICIT_DEF)), $D, sub_32);
}
```
- **EN**: Adds declarative TableGen records such as `AlignValues`, `AnyExts` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `AlignValues`, `AnyExts`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2269-2275

```tablegen
def DblToFlt {
  dag A0 = (f32 (any_fpround (f64 (extractelt v2f64:$A, 0))));
  dag A1 = (f32 (any_fpround (f64 (extractelt v2f64:$A, 1))));
  dag B0 = (f32 (any_fpround (f64 (extractelt v2f64:$B, 0))));
  dag B1 = (f32 (any_fpround (f64 (extractelt v2f64:$B, 1))));
}
```
- **EN**: Adds declarative TableGen records such as `DblToFlt` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DblToFlt`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2276-2286

```tablegen
def ExtDbl {
  dag A0S = (i32 (PPCmfvsr (f64 (PPCfctiwz (f64 (extractelt v2f64:$A, 0))))));
  dag A1S = (i32 (PPCmfvsr (f64 (PPCfctiwz (f64 (extractelt v2f64:$A, 1))))));
  dag B0S = (i32 (PPCmfvsr (f64 (PPCfctiwz (f64 (extractelt v2f64:$B, 0))))));
  dag B1S = (i32 (PPCmfvsr (f64 (PPCfctiwz (f64 (extractelt v2f64:$B, 1))))));
  dag A0U = (i32 (PPCmfvsr (f64 (PPCfctiwuz (f64 (extractelt v2f64:$A, 0))))));
  dag A1U = (i32 (PPCmfvsr (f64 (PPCfctiwuz (f64 (extractelt v2f64:$A, 1))))));
  dag B0U = (i32 (PPCmfvsr (f64 (PPCfctiwuz (f64 (extractelt v2f64:$B, 0))))));
  dag B1U = (i32 (PPCmfvsr (f64 (PPCfctiwuz (f64 (extractelt v2f64:$B, 1))))));
}
```
- **EN**: Adds declarative TableGen records such as `ExtDbl` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ExtDbl`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2287-2297

```tablegen
def ByteToWord {
  dag LE_A0 = (i32 (sext_inreg (i32 (vector_extract v16i8:$A, 0)), i8));
  dag LE_A1 = (i32 (sext_inreg (i32 (vector_extract v16i8:$A, 4)), i8));
  dag LE_A2 = (i32 (sext_inreg (i32 (vector_extract v16i8:$A, 8)), i8));
  dag LE_A3 = (i32 (sext_inreg (i32 (vector_extract v16i8:$A, 12)), i8));
  dag BE_A0 = (i32 (sext_inreg (i32 (vector_extract v16i8:$A, 3)), i8));
  dag BE_A1 = (i32 (sext_inreg (i32 (vector_extract v16i8:$A, 7)), i8));
  dag BE_A2 = (i32 (sext_inreg (i32 (vector_extract v16i8:$A, 11)), i8));
  dag BE_A3 = (i32 (sext_inreg (i32 (vector_extract v16i8:$A, 15)), i8));
}
```
- **EN**: Adds declarative TableGen records such as `ByteToWord` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ByteToWord`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2298-2308

```tablegen
def ByteToDWord {
  dag LE_A0 = (i64 (sext_inreg
              (i64 (anyext (i32 (vector_extract v16i8:$A, 0)))), i8));
  dag LE_A1 = (i64 (sext_inreg
              (i64 (anyext (i32 (vector_extract v16i8:$A, 8)))), i8));
  dag BE_A0 = (i64 (sext_inreg
              (i64 (anyext (i32 (vector_extract v16i8:$A, 7)))), i8));
  dag BE_A1 = (i64 (sext_inreg
              (i64 (anyext (i32 (vector_extract v16i8:$A, 15)))), i8));
}
```
- **EN**: Adds declarative TableGen records such as `ByteToDWord` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `ByteToDWord`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2309-2319

```tablegen
def HWordToWord {
  dag LE_A0 = (i32 (sext_inreg (i32 (vector_extract v8i16:$A, 0)), i16));
  dag LE_A1 = (i32 (sext_inreg (i32 (vector_extract v8i16:$A, 2)), i16));
  dag LE_A2 = (i32 (sext_inreg (i32 (vector_extract v8i16:$A, 4)), i16));
  dag LE_A3 = (i32 (sext_inreg (i32 (vector_extract v8i16:$A, 6)), i16));
  dag BE_A0 = (i32 (sext_inreg (i32 (vector_extract v8i16:$A, 1)), i16));
  dag BE_A1 = (i32 (sext_inreg (i32 (vector_extract v8i16:$A, 3)), i16));
  dag BE_A2 = (i32 (sext_inreg (i32 (vector_extract v8i16:$A, 5)), i16));
  dag BE_A3 = (i32 (sext_inreg (i32 (vector_extract v8i16:$A, 7)), i16));
}
```
- **EN**: Adds declarative TableGen records such as `HWordToWord` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HWordToWord`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2320-2330

```tablegen
def HWordToDWord {
  dag LE_A0 = (i64 (sext_inreg
              (i64 (anyext (i32 (vector_extract v8i16:$A, 0)))), i16));
  dag LE_A1 = (i64 (sext_inreg
              (i64 (anyext (i32 (vector_extract v8i16:$A, 4)))), i16));
  dag BE_A0 = (i64 (sext_inreg
              (i64 (anyext (i32 (vector_extract v8i16:$A, 3)))), i16));
  dag BE_A1 = (i64 (sext_inreg
              (i64 (anyext (i32 (vector_extract v8i16:$A, 7)))), i16));
}
```
- **EN**: Adds declarative TableGen records such as `HWordToDWord` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `HWordToDWord`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2331-2337

```tablegen
def WordToDWord {
  dag LE_A0 = (i64 (sext (i32 (vector_extract v4i32:$A, 0))));
  dag LE_A1 = (i64 (sext (i32 (vector_extract v4i32:$A, 2))));
  dag BE_A0 = (i64 (sext (i32 (vector_extract v4i32:$A, 1))));
  dag BE_A1 = (i64 (sext (i32 (vector_extract v4i32:$A, 3))));
}
```
- **EN**: Adds declarative TableGen records such as `WordToDWord` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `WordToDWord`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2338-2346

```tablegen
def FltToIntLoad {
  dag A = (i32 (PPCmfvsr (PPCfctiwz (f64 (extloadf32 ForceXForm:$A)))));
}
def FltToUIntLoad {
  dag A = (i32 (PPCmfvsr (PPCfctiwuz (f64 (extloadf32 ForceXForm:$A)))));
}
def FltToLongLoad {
  dag A = (i64 (PPCmfvsr (PPCfctidz (f64 (extloadf32 ForceXForm:$A)))));
}
```
- **EN**: Adds declarative TableGen records such as `FltToIntLoad`, `FltToUIntLoad`, `FltToLongLoad` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FltToIntLoad`, `FltToUIntLoad`, `FltToLongLoad`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2347-2355

```tablegen
def FltToLongLoadP9 {
  dag A = (i64 (PPCmfvsr (PPCfctidz (f64 (extloadf32 DSForm:$A)))));
}
def FltToULongLoad {
  dag A = (i64 (PPCmfvsr (PPCfctiduz (f64 (extloadf32 ForceXForm:$A)))));
}
def FltToULongLoadP9 {
  dag A = (i64 (PPCmfvsr (PPCfctiduz (f64 (extloadf32 DSForm:$A)))));
}
```
- **EN**: Adds declarative TableGen records such as `FltToLongLoadP9`, `FltToULongLoad`, `FltToULongLoadP9` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FltToLongLoadP9`, `FltToULongLoad`, `FltToULongLoadP9`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2356-2367

```tablegen
def FltToLong {
  dag A = (i64 (PPCmfvsr (f64 (PPCfctidz (fpextend f32:$A)))));
}
def FltToULong {
  dag A = (i64 (PPCmfvsr (f64 (PPCfctiduz (fpextend f32:$A)))));
}
def DblToInt {
  dag A = (i32 (PPCmfvsr (f64 (PPCfctiwz f64:$A))));
  dag B = (i32 (PPCmfvsr (f64 (PPCfctiwz f64:$B))));
  dag C = (i32 (PPCmfvsr (f64 (PPCfctiwz f64:$C))));
  dag D = (i32 (PPCmfvsr (f64 (PPCfctiwz f64:$D))));
}
```
- **EN**: Adds declarative TableGen records such as `FltToLong`, `FltToULong`, `DblToInt` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `FltToLong`, `FltToULong`, `DblToInt`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2368-2376

```tablegen
def DblToUInt {
  dag A = (i32 (PPCmfvsr (f64 (PPCfctiwuz f64:$A))));
  dag B = (i32 (PPCmfvsr (f64 (PPCfctiwuz f64:$B))));
  dag C = (i32 (PPCmfvsr (f64 (PPCfctiwuz f64:$C))));
  dag D = (i32 (PPCmfvsr (f64 (PPCfctiwuz f64:$D))));
}
def DblToLong {
  dag A = (i64 (PPCmfvsr (f64 (PPCfctidz f64:$A))));
}
```
- **EN**: Adds declarative TableGen records such as `DblToUInt`, `DblToLong` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DblToUInt`, `DblToLong`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2377-2385

```tablegen
def DblToULong {
  dag A = (i64 (PPCmfvsr (f64 (PPCfctiduz f64:$A))));
}
def DblToIntLoad {
  dag A = (i32 (PPCmfvsr (PPCfctiwz (f64 (load ForceXForm:$A)))));
}
def DblToIntLoadP9 {
  dag A = (i32 (PPCmfvsr (PPCfctiwz (f64 (load DSForm:$A)))));
}
```
- **EN**: Adds declarative TableGen records such as `DblToULong`, `DblToIntLoad`, `DblToIntLoadP9` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DblToULong`, `DblToIntLoad`, `DblToIntLoadP9`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2386-2394

```tablegen
def DblToUIntLoad {
  dag A = (i32 (PPCmfvsr (PPCfctiwuz (f64 (load ForceXForm:$A)))));
}
def DblToUIntLoadP9 {
  dag A = (i32 (PPCmfvsr (PPCfctiwuz (f64 (load DSForm:$A)))));
}
def DblToLongLoad {
  dag A = (i64 (PPCmfvsr (PPCfctidz (f64 (load ForceXForm:$A)))));
}
```
- **EN**: Adds declarative TableGen records such as `DblToUIntLoad`, `DblToUIntLoadP9`, `DblToLongLoad` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DblToUIntLoad`, `DblToUIntLoadP9`, `DblToLongLoad`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2395-2407

```tablegen
def DblToULongLoad {
  dag A = (i64 (PPCmfvsr (PPCfctiduz (f64 (load ForceXForm:$A)))));
}

// FP load dags (for f32 -> v4f32)
def LoadFP {
  dag A = (f32 (load ForceXForm:$A));
  dag B = (f32 (load ForceXForm:$B));
  dag C = (f32 (load ForceXForm:$C));
  dag D = (f32 (load ForceXForm:$D));
}

// FP merge dags (for f32 -> v4f32)
```
- **EN**: Adds declarative TableGen records such as `DblToULongLoad`, `LoadFP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `DblToULongLoad`, `LoadFP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2408-2423

```tablegen
def MrgFP {
  dag LD32A = (SUBREG_TO_REG (LIWZX ForceXForm:$A), sub_64);
  dag LD32B = (SUBREG_TO_REG (LIWZX ForceXForm:$B), sub_64);
  dag LD32C = (SUBREG_TO_REG (LIWZX ForceXForm:$C), sub_64);
  dag LD32D = (SUBREG_TO_REG (LIWZX ForceXForm:$D), sub_64);
  dag AC = (XVCVDPSP (XXPERMDI (SUBREG_TO_REG $A, sub_64),
                               (SUBREG_TO_REG $C, sub_64), 0));
  dag BD = (XVCVDPSP (XXPERMDI (SUBREG_TO_REG $B, sub_64),
                               (SUBREG_TO_REG $D, sub_64), 0));
  dag ABhToFlt = (XVCVDPSP (XXPERMDI $A, $B, 0));
  dag ABlToFlt = (XVCVDPSP (XXPERMDI $A, $B, 3));
  dag BAhToFlt = (XVCVDPSP (XXPERMDI $B, $A, 0));
  dag BAlToFlt = (XVCVDPSP (XXPERMDI $B, $A, 3));
}

// Word-element merge dags - conversions from f64 to i32 merged into vectors.
```
- **EN**: Adds declarative TableGen records such as `MrgFP` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MrgFP`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2424-2461

```tablegen
def MrgWords {
  // For big endian, we merge low and hi doublewords (A, B).
  dag A0B0 = (v2f64 (XXPERMDI v2f64:$A, v2f64:$B, 0));
  dag A1B1 = (v2f64 (XXPERMDI v2f64:$A, v2f64:$B, 3));
  dag CVA1B1S = (v4i32 (XVCVDPSXWS A1B1));
  dag CVA0B0S = (v4i32 (XVCVDPSXWS A0B0));
  dag CVA1B1U = (v4i32 (XVCVDPUXWS A1B1));
  dag CVA0B0U = (v4i32 (XVCVDPUXWS A0B0));

  // For little endian, we merge low and hi doublewords (B, A).
  dag B1A1 = (v2f64 (XXPERMDI v2f64:$B, v2f64:$A, 0));
  dag B0A0 = (v2f64 (XXPERMDI v2f64:$B, v2f64:$A, 3));
  dag CVB1A1S = (v4i32 (XVCVDPSXWS B1A1));
  dag CVB0A0S = (v4i32 (XVCVDPSXWS B0A0));
  dag CVB1A1U = (v4i32 (XVCVDPUXWS B1A1));
  dag CVB0A0U = (v4i32 (XVCVDPUXWS B0A0));

  // For big endian, we merge hi doublewords of (A, C) and (B, D), convert
  // then merge.
  dag AC = (v2f64 (XXPERMDI (SUBREG_TO_REG f64:$A, sub_64),
                            (SUBREG_TO_REG f64:$C, sub_64), 0));
  dag BD = (v2f64 (XXPERMDI (SUBREG_TO_REG f64:$B, sub_64),
                            (SUBREG_TO_REG f64:$D, sub_64), 0));
  dag CVACS = (v4i32 (XVCVDPSXWS AC));
  dag CVBDS = (v4i32 (XVCVDPSXWS BD));
  dag CVACU = (v4i32 (XVCVDPUXWS AC));
  dag CVBDU = (v4i32 (XVCVDPUXWS BD));

  // For little endian, we merge hi doublewords of (D, B) and (C, A), convert
  // then merge.
  dag DB = (v2f64 (XXPERMDI (SUBREG_TO_REG f64:$D, sub_64),
                            (SUBREG_TO_REG f64:$B, sub_64), 0));
  dag CA = (v2f64 (XXPERMDI (SUBREG_TO_REG f64:$C, sub_64),
                            (SUBREG_TO_REG f64:$A, sub_64), 0));
  dag CVDBS = (v4i32 (XVCVDPSXWS DB));
  dag CVCAS = (v4i32 (XVCVDPSXWS CA));
  dag CVDBU = (v4i32 (XVCVDPUXWS DB));
  dag CVCAU = (v4i32 (XVCVDPUXWS CA));
```
- **EN**: Adds declarative TableGen records such as `MrgWords` that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，例如 `MrgWords`，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2462-2499

```tablegen
}

def DblwdCmp {
  dag SGTW = (v2i64 (v2i64 (VCMPGTSW v2i64:$vA, v2i64:$vB)));
  dag UGTW = (v2i64 (v2i64 (VCMPGTUW v2i64:$vA, v2i64:$vB)));
  dag EQW = (v2i64 (v2i64 (VCMPEQUW v2i64:$vA, v2i64:$vB)));
  dag UGTWSHAND = (v2i64 (XXLAND (v2i64 (XXSLDWI UGTW, UGTW, 1)), EQW));
  dag EQWSHAND = (v2i64 (XXLAND (v2i64 (XXSLDWI EQW, EQW, 1)), EQW));
  dag SGTWOR = (v2i64 (XXLOR SGTW, UGTWSHAND));
  dag UGTWOR = (v2i64 (XXLOR UGTW, UGTWSHAND));
  dag MRGSGT = (v2i64 (XXPERMDI (v2i64 (XXSPLTW SGTWOR, 0)),
                                (v2i64 (XXSPLTW SGTWOR, 2)), 0));
  dag MRGUGT = (v2i64 (XXPERMDI (v2i64 (XXSPLTW UGTWOR, 0)),
                                (v2i64 (XXSPLTW UGTWOR, 2)), 0));
  dag MRGEQ = (v2i64 (XXPERMDI (v2i64 (XXSPLTW EQWSHAND, 0)),
                               (v2i64 (XXSPLTW EQWSHAND, 2)), 0));
}

//---------------------------- Anonymous Patterns ----------------------------//
// Predicate combinations are kept in roughly chronological order in terms of
// instruction availability in the architecture. For example, VSX came in with
// ISA 2.06 (Power7). There have since been additions in ISA 2.07 (Power8) and
// ISA 3.0 (Power9). However, the granularity of features on later subtargets
// is finer for various reasons. For example, we have Power8Vector,
// Power8Altivec, DirectMove that all came in with ISA 2.07. The situation is
// similar with ISA 3.0 with Power9Vector, Power9Altivec, IsISA3_0. Then there
// are orthogonal predicates such as endianness for which the order was
// arbitrarily chosen to be Big, Little.
//
// Predicate combinations available:
// [HasVSX, IsLittleEndian, HasP8Altivec] Altivec patterns using VSX instr.
// [HasVSX, IsBigEndian, HasP8Altivec] Altivec patterns using VSX instr.
// [HasVSX]
// [HasVSX, IsBigEndian]
// [HasVSX, IsLittleEndian]
// [HasVSX, NoP9Vector]
// [HasVSX, NoP9Vector, IsLittleEndian]
// [HasVSX, NoP9Vector, IsBigEndian]
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "---------------------------- Anonymous Patterns ----------------------------//". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“---------------------------- Anonymous Patterns ----------------------------//”。 这些声明会进入生成式模式匹配逻辑。

### Lines 2500-2529

```tablegen
// [HasVSX, HasOnlySwappingMemOps]
// [HasVSX, HasOnlySwappingMemOps, IsBigEndian]
// [HasVSX, NoP8Vector]
// [HasVSX, HasP8Vector]
// [HasVSX, HasP8Vector, IsBigEndian]
// [HasVSX, HasP8Vector, IsBigEndian, IsPPC64]
// [HasVSX, HasP8Vector, IsLittleEndian]
// [HasVSX, HasP8Vector, NoP9Vector, IsBigEndian, IsPPC64]
// [HasVSX, HasP8Vector, NoP9Vector, IsLittleEndian]
// [HasVSX, HasP8Altivec]
// [HasVSX, HasDirectMove]
// [HasVSX, HasDirectMove, IsBigEndian]
// [HasVSX, HasDirectMove, IsLittleEndian]
// [HasVSX, HasDirectMove, NoP9Altivec, IsBigEndian, IsPPC64]
// [HasVSX, HasDirectMove, NoP9Vector, IsBigEndian, IsPPC64]
// [HasVSX, HasDirectMove, NoP9Altivec, IsLittleEndian]
// [HasVSX, HasDirectMove, NoP9Vector, IsLittleEndian]
// [HasVSX, HasP9Vector]
// [HasVSX, HasP9Vector, NoP10Vector]
// [HasVSX, HasP9Vector, IsBigEndian]
// [HasVSX, HasP9Vector, IsBigEndian, IsPPC64]
// [HasVSX, HasP9Vector, IsLittleEndian]
// [HasVSX, HasP9Altivec]
// [HasVSX, HasP9Altivec, IsBigEndian, IsPPC64]
// [HasVSX, HasP9Altivec, IsLittleEndian]
// [HasVSX, IsISA3_0, HasDirectMove, IsBigEndian, IsPPC64]
// [HasVSX, IsISA3_0, HasDirectMove, IsLittleEndian]

// These Altivec patterns are here because we need a VSX instruction to match
// the intrinsic (but only for little endian system).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "[HasVSX, HasOnlySwappingMemOps]".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“[HasVSX, HasOnlySwappingMemOps]”。

### Lines 2530-2538

```tablegen
let Predicates = [HasVSX, IsLittleEndian, HasP8Altivec] in
  def : Pat<(v16i8 (int_ppc_altivec_crypto_vpermxor v16i8:$a,
                                                    v16i8:$b, v16i8:$c)),
            (v16i8 (VPERMXOR $a, $b, (XXLNOR (COPY_TO_REGCLASS $c, VSRC),
                                             (COPY_TO_REGCLASS $c, VSRC))))>;
let Predicates = [HasVSX, IsBigEndian, HasP8Altivec] in
  def : Pat<(v16i8 (int_ppc_altivec_crypto_vpermxor v16i8:$a,
                                                    v16i8:$b, v16i8:$c)),
            (v16i8 (VPERMXOR $a, $b, $c))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2539-2545

```tablegen
let Predicates = [HasVSX, HasP8Altivec] in
  def : Pat<(v16i8 (int_ppc_altivec_crypto_vpermxor_be v16i8:$a,
                                                       v16i8:$b, v16i8:$c)),
            (v16i8 (VPERMXOR $a, $b, $c))>;

let AddedComplexity = 400 in {
// Valid for any VSX subtarget, regardless of endianness.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2546-2552

```tablegen
let Predicates = [HasVSX] in {
def : Pat<(v4i32 (vnot v4i32:$A)),
          (v4i32 (XXLNOR $A, $A))>;
def : Pat<(v4i32 (or (and (vnot v4i32:$C), v4i32:$A),
                     (and v4i32:$B, v4i32:$C))),
          (v4i32 (XXSEL $A, $B, $C))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2553-2560

```tablegen
def : Pat<(f64 (fpimm0neg)),
          (f64 (XSNEGDP (XXLXORdpz)))>;

def : Pat<(f64 (nzFPImmExactInti5:$A)),
          (COPY_TO_REGCLASS (XVCVSXWDP (COPY_TO_REGCLASS
                     (VSPLTISW (getFPAs5BitExactInt fpimm:$A)), VSRC)), VSFRC)>;

// Additional fnmsub pattern for PPC specific ISD opcode
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2561-2567

```tablegen
def : Pat<(PPCfnmsub f64:$A, f64:$B, f64:$C),
          (XSNMSUBADP $C, $A, $B)>;
def : Pat<(fneg (PPCfnmsub f64:$A, f64:$B, f64:$C)),
          (XSMSUBADP $C, $A, $B)>;
def : Pat<(PPCfnmsub f64:$A, f64:$B, (fneg f64:$C)),
          (XSNMADDADP $C, $A, $B)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2568-2574

```tablegen
def : Pat<(PPCfnmsub v2f64:$A, v2f64:$B, v2f64:$C),
          (XVNMSUBADP $C, $A, $B)>;
def : Pat<(fneg (PPCfnmsub v2f64:$A, v2f64:$B, v2f64:$C)),
          (XVMSUBADP $C, $A, $B)>;
def : Pat<(PPCfnmsub v2f64:$A, v2f64:$B, (fneg v2f64:$C)),
          (XVNMADDADP $C, $A, $B)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2575-2581

```tablegen
def : Pat<(PPCfnmsub v4f32:$A, v4f32:$B, v4f32:$C),
          (XVNMSUBASP $C, $A, $B)>;
def : Pat<(fneg (PPCfnmsub v4f32:$A, v4f32:$B, v4f32:$C)),
          (XVMSUBASP $C, $A, $B)>;
def : Pat<(PPCfnmsub v4f32:$A, v4f32:$B, (fneg v4f32:$C)),
          (XVNMADDASP $C, $A, $B)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2582-2589

```tablegen
def : Pat<(PPCfsqrt f64:$frA), (XSSQRTDP $frA)>;
def : Pat<(PPCfsqrt v2f64:$frA), (XVSQRTDP $frA)>;
def : Pat<(PPCfsqrt v4f32:$frA), (XVSQRTSP $frA)>;

def : Pat<(v2f64 (bitconvert v4f32:$A)),
          (COPY_TO_REGCLASS $A, VSRC)>;
def : Pat<(v2f64 (bitconvert v4i32:$A)),
          (COPY_TO_REGCLASS $A, VSRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2590-2596

```tablegen
def : Pat<(v2f64 (bitconvert v8i16:$A)),
          (COPY_TO_REGCLASS $A, VSRC)>;
def : Pat<(v2f64 (bitconvert v16i8:$A)),
          (COPY_TO_REGCLASS $A, VSRC)>;

def : Pat<(v4f32 (bitconvert v2f64:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2597-2603

```tablegen
def : Pat<(v4i32 (bitconvert v2f64:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
def : Pat<(v8i16 (bitconvert v2f64:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
def : Pat<(v16i8 (bitconvert v2f64:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2604-2612

```tablegen
def : Pat<(v2i64 (bitconvert v4f32:$A)),
          (COPY_TO_REGCLASS $A, VSRC)>;
def : Pat<(v2i64 (bitconvert v4i32:$A)),
          (COPY_TO_REGCLASS $A, VSRC)>;
def : Pat<(v2i64 (bitconvert v8i16:$A)),
          (COPY_TO_REGCLASS $A, VSRC)>;
def : Pat<(v2i64 (bitconvert v16i8:$A)),
          (COPY_TO_REGCLASS $A, VSRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2613-2621

```tablegen
def : Pat<(v4f32 (bitconvert v2i64:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
def : Pat<(v4i32 (bitconvert v2i64:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
def : Pat<(v8i16 (bitconvert v2i64:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
def : Pat<(v16i8 (bitconvert v2i64:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2622-2628

```tablegen
def : Pat<(v2f64 (bitconvert v2i64:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
def : Pat<(v2i64 (bitconvert v2f64:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;

def : Pat<(v2f64 (bitconvert v1i128:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2629-2635

```tablegen
def : Pat<(v1i128 (bitconvert v2f64:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;

def : Pat<(v2i64 (bitconvert f128:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
def : Pat<(v4i32 (bitconvert f128:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2636-2642

```tablegen
def : Pat<(v8i16 (bitconvert f128:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;
def : Pat<(v16i8 (bitconvert f128:$A)),
          (COPY_TO_REGCLASS $A, VRRC)>;

def : Pat<(v2f64 (PPCsvec2fp v4i32:$C, 0)),
          (v2f64 (XVCVSXWDP (v2i64 (XXMRGHW $C, $C))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2643-2650

```tablegen
def : Pat<(v2f64 (PPCsvec2fp v4i32:$C, 1)),
          (v2f64 (XVCVSXWDP (v2i64 (XXMRGLW $C, $C))))>;

def : Pat<(v2f64 (PPCuvec2fp v4i32:$C, 0)),
          (v2f64 (XVCVUXWDP (v2i64 (XXMRGHW $C, $C))))>;
def : Pat<(v2f64 (PPCuvec2fp v4i32:$C, 1)),
          (v2f64 (XVCVUXWDP (v2i64 (XXMRGLW $C, $C))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2651-2657

```tablegen
def : Pat<(v2f64 (PPCfpexth v4f32:$C, 0)), (XVCVSPDP (XXMRGHW $C, $C))>;
def : Pat<(v2f64 (PPCfpexth v4f32:$C, 1)), (XVCVSPDP (XXMRGLW $C, $C))>;

// Permutes.
def : Pat<(v2f64 (PPCxxswapd v2f64:$src)), (XXPERMDI $src, $src, 2)>;
def : Pat<(v2i64 (PPCxxswapd v2i64:$src)), (XXPERMDI $src, $src, 2)>;
def : Pat<(v4f32 (PPCxxswapd v4f32:$src)), (XXPERMDI $src, $src, 2)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2658-2666

```tablegen
def : Pat<(v4i32 (PPCxxswapd v4i32:$src)), (XXPERMDI $src, $src, 2)>;
def : Pat<(v2f64 (PPCswapNoChain v2f64:$src)), (XXPERMDI $src, $src, 2)>;

// PPCvecshl XT, XA, XA, 2 can be selected to both XXSLDWI XT,XA,XA,2 and
// XXSWAPD XT,XA (i.e. XXPERMDI XT,XA,XA,2), the later one is more profitable.
def : Pat<(v4i32 (PPCvecshl v4i32:$src, v4i32:$src, 2)),
          (XXPERMDI $src, $src, 2)>;

// Selects.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "PPCvecshl XT, XA, XA, 2 can be selected to both XXSLDWI XT,XA,XA,2 and".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“PPCvecshl XT, XA, XA, 2 can be selected to both XXSLDWI XT,XA,XA,2 and”。

### Lines 2667-2674

```tablegen
def : Pat<(v2f64 (selectcc i1:$lhs, i1:$rhs, v2f64:$tval, v2f64:$fval, SETLT)),
          (SELECT_VSRC (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(v2f64 (selectcc i1:$lhs, i1:$rhs, v2f64:$tval, v2f64:$fval, SETULT)),
          (SELECT_VSRC (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(v2f64 (selectcc i1:$lhs, i1:$rhs, v2f64:$tval, v2f64:$fval, SETLE)),
          (SELECT_VSRC (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(v2f64 (selectcc i1:$lhs, i1:$rhs, v2f64:$tval, v2f64:$fval, SETULE)),
          (SELECT_VSRC (CRORC  $rhs, $lhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2675-2682

```tablegen
def : Pat<(v2f64 (selectcc i1:$lhs, i1:$rhs, v2f64:$tval, v2f64:$fval, SETEQ)),
          (SELECT_VSRC (CREQV $lhs, $rhs), $tval, $fval)>;
def : Pat<(v2f64 (selectcc i1:$lhs, i1:$rhs, v2f64:$tval, v2f64:$fval, SETGE)),
          (SELECT_VSRC (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(v2f64 (selectcc i1:$lhs, i1:$rhs, v2f64:$tval, v2f64:$fval, SETUGE)),
          (SELECT_VSRC (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(v2f64 (selectcc i1:$lhs, i1:$rhs, v2f64:$tval, v2f64:$fval, SETGT)),
          (SELECT_VSRC (CRANDC $rhs, $lhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2683-2689

```tablegen
def : Pat<(v2f64 (selectcc i1:$lhs, i1:$rhs, v2f64:$tval, v2f64:$fval, SETUGT)),
          (SELECT_VSRC (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(v2f64 (selectcc i1:$lhs, i1:$rhs, v2f64:$tval, v2f64:$fval, SETNE)),
          (SELECT_VSRC (CRXOR $lhs, $rhs), $tval, $fval)>;

def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETLT)),
          (SELECT_VSFRC (CRANDC $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2690-2697

```tablegen
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETULT)),
          (SELECT_VSFRC (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETLE)),
          (SELECT_VSFRC (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETULE)),
          (SELECT_VSFRC (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETEQ)),
          (SELECT_VSFRC (CREQV $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2698-2705

```tablegen
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETGE)),
          (SELECT_VSFRC (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETUGE)),
          (SELECT_VSFRC (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETGT)),
          (SELECT_VSFRC (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETUGT)),
          (SELECT_VSFRC (CRANDC $lhs, $rhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2706-2715

```tablegen
def : Pat<(f64 (selectcc i1:$lhs, i1:$rhs, f64:$tval, f64:$fval, SETNE)),
          (SELECT_VSFRC (CRXOR $lhs, $rhs), $tval, $fval)>;

// Divides.
def : Pat<(int_ppc_vsx_xvdivsp v4f32:$A, v4f32:$B),
          (XVDIVSP $A, $B)>;
def : Pat<(int_ppc_vsx_xvdivdp v2f64:$A, v2f64:$B),
          (XVDIVDP $A, $B)>;

// Vector test for software divide and sqrt.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2716-2725

```tablegen
def : Pat<(i32 (int_ppc_vsx_xvtdivdp v2f64:$A, v2f64:$B)),
          (COPY_TO_REGCLASS (XVTDIVDP $A, $B), GPRC)>;
def : Pat<(i32 (int_ppc_vsx_xvtdivsp v4f32:$A, v4f32:$B)),
          (COPY_TO_REGCLASS (XVTDIVSP $A, $B), GPRC)>;
def : Pat<(i32 (int_ppc_vsx_xvtsqrtdp v2f64:$A)),
          (COPY_TO_REGCLASS (XVTSQRTDP $A), GPRC)>;
def : Pat<(i32 (int_ppc_vsx_xvtsqrtsp v4f32:$A)),
          (COPY_TO_REGCLASS (XVTSQRTSP $A), GPRC)>;

// Reciprocal estimate
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2726-2733

```tablegen
def : Pat<(int_ppc_vsx_xvresp v4f32:$A),
          (XVRESP $A)>;
def : Pat<(int_ppc_vsx_xvredp v2f64:$A),
          (XVREDP $A)>;

// Recip. square root estimate
def : Pat<(int_ppc_vsx_xvrsqrtesp v4f32:$A),
          (XVRSQRTESP $A)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2734-2742

```tablegen
def : Pat<(int_ppc_vsx_xvrsqrtedp v2f64:$A),
          (XVRSQRTEDP $A)>;

// Vector selection
def : Pat<(v16i8 (vselect v16i8:$vA, v16i8:$vB, v16i8:$vC)),
          (COPY_TO_REGCLASS
                 (XXSEL (COPY_TO_REGCLASS $vC, VSRC),
                        (COPY_TO_REGCLASS $vB, VSRC),
                        (COPY_TO_REGCLASS $vA, VSRC)), VRRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2743-2749

```tablegen
def : Pat<(v8i16 (vselect v8i16:$vA, v8i16:$vB, v8i16:$vC)),
          (COPY_TO_REGCLASS
                 (XXSEL (COPY_TO_REGCLASS $vC, VSRC),
                        (COPY_TO_REGCLASS $vB, VSRC),
                        (COPY_TO_REGCLASS $vA, VSRC)), VRRC)>;
def : Pat<(vselect v4i32:$vA, v4i32:$vB, v4i32:$vC),
          (XXSEL $vC, $vB, $vA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2750-2761

```tablegen
def : Pat<(vselect v2i64:$vA, v2i64:$vB, v2i64:$vC),
          (XXSEL $vC, $vB, $vA)>;
def : Pat<(vselect v4i32:$vA, v4f32:$vB, v4f32:$vC),
          (XXSEL $vC, $vB, $vA)>;
def : Pat<(vselect v2i64:$vA, v2f64:$vB, v2f64:$vC),
          (XXSEL $vC, $vB, $vA)>;
def : Pat<(v1i128 (vselect v1i128:$vA, v1i128:$vB, v1i128:$vC)),
          (COPY_TO_REGCLASS
                 (XXSEL (COPY_TO_REGCLASS $vC, VSRC),
                        (COPY_TO_REGCLASS $vB, VSRC),
                        (COPY_TO_REGCLASS $vA, VSRC)), VRRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2762-2769

```tablegen
def : Pat<(v4f32 (fmaxnum_ieee v4f32:$src1, v4f32:$src2)),
          (v4f32 (XVMAXSP $src1, $src2))>;
def : Pat<(v4f32 (any_fmaxnum v4f32:$src1, v4f32:$src2)),
          (v4f32 (XVMAXSP $src1, $src2))>;
def : Pat<(v4f32 (fminnum_ieee v4f32:$src1, v4f32:$src2)),
          (v4f32 (XVMINSP $src1, $src2))>;
def : Pat<(v4f32 (any_fminnum v4f32:$src1, v4f32:$src2)),
          (v4f32 (XVMINSP $src1, $src2))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2770-2777

```tablegen
def : Pat<(v4f32 (fcanonicalize v4f32:$src1)),
          (v4f32 (XVMAXSP $src1, $src1))>;
def : Pat<(v2f64 (fmaxnum_ieee v2f64:$src1, v2f64:$src2)),
          (v2f64 (XVMAXDP $src1, $src2))>;
def : Pat<(v2f64 (any_fmaxnum v2f64:$src1, v2f64:$src2)),
          (v2f64 (XVMAXDP $src1, $src2))>;
def : Pat<(v2f64 (fminnum_ieee v2f64:$src1, v2f64:$src2)),
          (v2f64 (XVMINDP $src1, $src2))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2778-2788

```tablegen
def : Pat<(v2f64 (any_fminnum v2f64:$src1, v2f64:$src2)),
          (v2f64 (XVMINDP $src1, $src2))>;
def : Pat<(v2f64 (fcanonicalize v2f64:$src1)),
          (v2f64 (XVMAXDP $src1, $src1))>;

// f32 abs
def : Pat<(f32 (fabs f32:$S)),
          (f32 (COPY_TO_REGCLASS (XSABSDP
               (COPY_TO_REGCLASS $S, VSFRC)), VSSRC))>;

// f32 nabs
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2789-2795

```tablegen
def : Pat<(f32 (fneg (fabs f32:$S))),
          (f32 (COPY_TO_REGCLASS (XSNABSDP
               (COPY_TO_REGCLASS $S, VSFRC)), VSSRC))>;

// Max and Min
def : Pat<(f32 (fminnum_ieee f32:$A, f32:$B)),
          (f32 FpMinMax.F32Min)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2796-2805

```tablegen
def : Pat<(f32 (fmaxnum_ieee f32:$A, f32:$B)),
          (f32 FpMinMax.F32Max)>;
def : Pat<(f32 (fminnum f32:$A, f32:$B)),
          (f32 FpMinMax.F32Min)>;
def : Pat<(f32 (fmaxnum f32:$A, f32:$B)),
          (f32 FpMinMax.F32Max)>;
def : Pat<(f32 (fcanonicalize f32:$A)),
          (f32 (COPY_TO_REGCLASS (XSMAXDP
               (COPY_TO_REGCLASS $A, VSFRC),
               (COPY_TO_REGCLASS $A, VSFRC)), VSSRC))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2806-2813

```tablegen
def : Pat<(f64 (fminnum_ieee f64:$A, f64:$B)),
          (f64 (XSMINDP $A, $B))>;
def : Pat<(f64 (fmaxnum_ieee f64:$A, f64:$B)),
          (f64 (XSMAXDP $A, $B))>;
def : Pat<(f64 (fminnum f64:$A, f64:$B)),
          (f64 (XSMINDP $A, $B))>;
def : Pat<(f64 (fmaxnum f64:$A, f64:$B)),
          (f64 (XSMAXDP $A, $B))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2814-2820

```tablegen
def : Pat<(f64 (fcanonicalize f64:$A)),
          (f64 (XSMAXDP $A, $A))>;

def : Pat<(int_ppc_vsx_stxvd2x_be v2f64:$rS, ForceXForm:$dst),
            (STXVD2X $rS, ForceXForm:$dst)>;
def : Pat<(int_ppc_vsx_stxvw4x_be v4i32:$rS, ForceXForm:$dst),
            (STXVW4X $rS, ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2821-2827

```tablegen
def : Pat<(v4i32 (int_ppc_vsx_lxvw4x_be ForceXForm:$src)), (LXVW4X ForceXForm:$src)>;
def : Pat<(v2f64 (int_ppc_vsx_lxvd2x_be ForceXForm:$src)), (LXVD2X ForceXForm:$src)>;

// Rounding for single precision.
def : Pat<(f32 (any_fround f32:$S)),
          (f32 (COPY_TO_REGCLASS (XSRDPI
                                   (COPY_TO_REGCLASS $S, VSFRC)), VSSRC))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2828-2836

```tablegen
def : Pat<(f32 (any_ffloor f32:$S)),
          (f32 (COPY_TO_REGCLASS (XSRDPIM
                                   (COPY_TO_REGCLASS $S, VSFRC)), VSSRC))>;
def : Pat<(f32 (any_fceil f32:$S)),
          (f32 (COPY_TO_REGCLASS (XSRDPIP
                                   (COPY_TO_REGCLASS $S, VSFRC)), VSSRC))>;
def : Pat<(f32 (any_ftrunc f32:$S)),
          (f32 (COPY_TO_REGCLASS (XSRDPIZ
                                   (COPY_TO_REGCLASS $S, VSFRC)), VSSRC))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2837-2843

```tablegen
def : Pat<(f32 (any_frint f32:$S)),
          (f32 (COPY_TO_REGCLASS (XSRDPIC
                                   (COPY_TO_REGCLASS $S, VSFRC)), VSSRC))>;
def : Pat<(v4f32 (any_frint v4f32:$S)), (v4f32 (XVRSPIC $S))>;

// Rounding for double precision.
def : Pat<(f64 (any_frint f64:$S)), (f64 (XSRDPIC $S))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2844-2850

```tablegen
def : Pat<(v2f64 (any_frint v2f64:$S)), (v2f64 (XVRDPIC $S))>;

// Rounding without exceptions (nearbyint). Due to strange tblgen behaviour,
// these need to be defined after the any_frint versions so ISEL will correctly
// add the chain to the strict versions.
// TODO: Match strict fp rounding intrinsics with instructions like xsrdpiz when
// rounding mode is propagated to CodeGen part.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Rounding without exceptions (nearbyint). Due to strange tblgen behaviour,".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Rounding without exceptions (nearbyint). Due to strange tblgen behaviour,”。

### Lines 2851-2857

```tablegen
def : Pat<(f32 (strict_fnearbyint f32:$S)),
          (f32 (COPY_TO_REGCLASS (XSRDPIC
                                   (COPY_TO_REGCLASS $S, VSFRC)), VSSRC))>;
def : Pat<(f64 (strict_fnearbyint f64:$S)),
          (f64 (XSRDPIC $S))>;
def : Pat<(v2f64 (strict_fnearbyint v2f64:$S)),
          (v2f64 (XVRDPIC $S))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2858-2865

```tablegen
def : Pat<(v4f32 (strict_fnearbyint v4f32:$S)),
          (v4f32 (XVRSPIC $S))>;

// Materialize a zero-vector of long long
def : Pat<(v2i64 immAllZerosV),
          (v2i64 (XXLXORz))>;

// Build vectors of floating point converted to i32.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Materialize a zero-vector of long long".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Materialize a zero-vector of long long”。

### Lines 2866-2874

```tablegen
def : Pat<(v4i32 (build_vector DblToInt.A, DblToInt.A,
                               DblToInt.A, DblToInt.A)),
          (v4i32 (XXSPLTW (SUBREG_TO_REG (XSCVDPSXWS $A), sub_64), 1))>;
def : Pat<(v4i32 (build_vector DblToUInt.A, DblToUInt.A,
                               DblToUInt.A, DblToUInt.A)),
          (v4i32 (XXSPLTW (SUBREG_TO_REG (XSCVDPUXWS $A), sub_64), 1))>;
def : Pat<(v2i64 (build_vector DblToLong.A, DblToLong.A)),
          (v2i64 (XXPERMDI (SUBREG_TO_REG (XSCVDPSXDS $A), sub_64),
                           (SUBREG_TO_REG (XSCVDPSXDS $A), sub_64), 0))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2875-2881

```tablegen
def : Pat<(v2i64 (build_vector DblToULong.A, DblToULong.A)),
          (v2i64 (XXPERMDI (SUBREG_TO_REG (XSCVDPUXDS $A), sub_64),
                           (SUBREG_TO_REG (XSCVDPUXDS $A), sub_64), 0))>;
def : Pat<(v4i32 (PPCSToV DblToInt.A)),
          (v4i32 (SUBREG_TO_REG (XSCVDPSXWS f64:$A), sub_64))>;
def : Pat<(v4i32 (PPCSToV DblToUInt.A)),
          (v4i32 (SUBREG_TO_REG (XSCVDPUXWS f64:$A), sub_64))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2882-2889

```tablegen
defm : ScalToVecWPermute<
  v4i32, FltToIntLoad.A,
  (XXSPLTW (SUBREG_TO_REG (XSCVDPSXWSs (XFLOADf32 ForceXForm:$A)), sub_64), 1),
  (SUBREG_TO_REG (XSCVDPSXWSs (XFLOADf32 ForceXForm:$A)), sub_64)>;
defm : ScalToVecWPermute<
  v4i32, FltToUIntLoad.A,
  (XXSPLTW (SUBREG_TO_REG (XSCVDPUXWSs (XFLOADf32 ForceXForm:$A)), sub_64), 1),
  (SUBREG_TO_REG (XSCVDPUXWSs (XFLOADf32 ForceXForm:$A)), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2890-2897

```tablegen
def : Pat<(v4f32 (build_vector (f32 (fpround f64:$A)), (f32 (fpround f64:$A)),
                               (f32 (fpround f64:$A)), (f32 (fpround f64:$A)))),
          (v4f32 (XXSPLTW (SUBREG_TO_REG (XSCVDPSP f64:$A), sub_64), 0))>;

def : Pat<(v4f32 (build_vector f32:$A, f32:$A, f32:$A, f32:$A)),
          (v4f32 (XXSPLTW (v4f32 (XSCVDPSPN $A)), 0))>;

// Splat loads.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2898-2905

```tablegen
def : Pat<(v2f64 (PPCldsplat ForceXForm:$A)),
          (v2f64 (LXVDSX ForceXForm:$A))>;
def : Pat<(v4f32 (PPCldsplat ForceXForm:$A)),
          (v4f32 (XXSPLTW (SUBREG_TO_REG (LFIWZX ForceXForm:$A), sub_64), 1))>;
def : Pat<(v2i64 (PPCldsplat ForceXForm:$A)),
          (v2i64 (LXVDSX ForceXForm:$A))>;
def : Pat<(v4i32 (PPCldsplat ForceXForm:$A)),
          (v4i32 (XXSPLTW (SUBREG_TO_REG (LFIWZX ForceXForm:$A), sub_64), 1))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2906-2914

```tablegen
def : Pat<(v2i64 (PPCzextldsplat ForceXForm:$A)),
          (v2i64 (XXPERMDIs (LFIWZX ForceXForm:$A), 0))>;
def : Pat<(v2i64 (PPCsextldsplat ForceXForm:$A)),
          (v2i64 (XXPERMDIs (LFIWAX ForceXForm:$A), 0))>;

// Build vectors of floating point converted to i64.
def : Pat<(v2i64 (build_vector FltToLong.A, FltToLong.A)),
          (v2i64 (XXPERMDIs
                   (COPY_TO_REGCLASS (XSCVDPSXDSs $A), VSFRC), 0))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2915-2925

```tablegen
def : Pat<(v2i64 (build_vector FltToULong.A, FltToULong.A)),
          (v2i64 (XXPERMDIs
                   (COPY_TO_REGCLASS (XSCVDPUXDSs $A), VSFRC), 0))>;
defm : ScalToVecWPermute<
  v2i64, DblToLongLoad.A,
  (XVCVDPSXDS (LXVDSX ForceXForm:$A)), (XVCVDPSXDS (LXVDSX ForceXForm:$A))>;
defm : ScalToVecWPermute<
  v2i64, DblToULongLoad.A,
  (XVCVDPUXDS (LXVDSX ForceXForm:$A)), (XVCVDPUXDS (LXVDSX ForceXForm:$A))>;

// Doubleword vector predicate comparisons without Power8.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2926-2935

```tablegen
let AddedComplexity = 0 in {
def : Pat<(v2i64 (PPCvcmp_rec v2i64:$vA, v2i64:$vB, 967)),
          (VCMPGTUB_rec DblwdCmp.MRGSGT, (v2i64 (XXLXORz)))>;
def : Pat<(v2i64 (PPCvcmp_rec v2i64:$vA, v2i64:$vB, 711)),
          (VCMPGTUB_rec DblwdCmp.MRGUGT, (v2i64 (XXLXORz)))>;
def : Pat<(v2i64 (PPCvcmp_rec v2i64:$vA, v2i64:$vB, 199)),
          (VCMPGTUB_rec DblwdCmp.MRGEQ, (v2i64 (XXLXORz)))>;
} // AddedComplexity = 0

// XL Compat builtins.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2936-2944

```tablegen
def : Pat<(int_ppc_fmsub f64:$A, f64:$B, f64:$C), (XSMSUBMDP $A, $B, $C)>;
def : Pat<(int_ppc_fnmadd f64:$A, f64:$B, f64:$C), (XSNMADDMDP $A, $B, $C)>;
def : Pat<(int_ppc_fre f64:$A), (XSREDP $A)>;
def : Pat<(int_ppc_frsqrte vsfrc:$XB), (XSRSQRTEDP $XB)>;
def : Pat<(int_ppc_fnabs f64:$A), (XSNABSDP $A)>;
def : Pat<(int_ppc_fnabss f32:$A), (XSNABSDPs $A)>;

// XXMRG[LH]W is a direct replacement for VMRG[LH]W respectively.
// Prefer the VSX form for greater register range.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2945-2953

```tablegen
def:Pat<(vmrglw_unary_shuffle v16i8:$vA, undef),
        (COPY_TO_REGCLASS (XXMRGLW (COPY_TO_REGCLASS $vA, VSRC),
                                   (COPY_TO_REGCLASS $vA, VSRC)), VRRC)>;
def:Pat<(vmrghw_unary_shuffle v16i8:$vA, undef),
        (COPY_TO_REGCLASS (XXMRGHW (COPY_TO_REGCLASS $vA, VSRC),
                                   (COPY_TO_REGCLASS $vA, VSRC)), VRRC)>;
def:Pat<(vmrglw_shuffle v16i8:$vA, v16i8:$vB),
        (COPY_TO_REGCLASS (XXMRGLW (COPY_TO_REGCLASS $vA, VSRC),
                                   (COPY_TO_REGCLASS $vB, VSRC)), VRRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2954-2962

```tablegen
def:Pat<(vmrghw_shuffle v16i8:$vA, v16i8:$vB),
        (COPY_TO_REGCLASS (XXMRGHW (COPY_TO_REGCLASS $vA, VSRC),
                                   (COPY_TO_REGCLASS $vB, VSRC)), VRRC)>;
def:Pat<(vmrglw_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (COPY_TO_REGCLASS (XXMRGLW (COPY_TO_REGCLASS $vB, VSRC),
                                   (COPY_TO_REGCLASS $vA, VSRC)), VRRC)>;
def:Pat<(vmrghw_swapped_shuffle v16i8:$vA, v16i8:$vB),
        (COPY_TO_REGCLASS (XXMRGHW (COPY_TO_REGCLASS $vB, VSRC),
                                   (COPY_TO_REGCLASS $vA, VSRC)), VRRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2963-2969

```tablegen
def : Pat<(PPCstore_scal_int_from_vsr f64:$src, XForm:$dst, 8),
          (STXSDX $src, XForm:$dst)>;
def : Pat<(PPCstore_scal_int_from_vsr f128:$src, XForm:$dst, 8),
          (STXSDX (COPY_TO_REGCLASS $src, VSFRC), XForm:$dst)>;
} // HasVSX

// Any big endian VSX subtarget.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2970-2977

```tablegen
let Predicates = [HasVSX, IsBigEndian] in {
def : Pat<(v2f64 (scalar_to_vector f64:$A)),
          (v2f64 (SUBREG_TO_REG $A, sub_64))>;

def : Pat<(f64 (extractelt v2f64:$S, 0)),
          (f64 (EXTRACT_SUBREG $S, sub_64))>;
def : Pat<(f64 (extractelt v2f64:$S, 1)),
          (f64 (EXTRACT_SUBREG (XXPERMDI $S, $S, 2), sub_64))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 2978-2986

```tablegen
def : Pat<(f64 (PPCfcfid (PPCmtvsra (i64 (vector_extract v2i64:$S, 0))))),
          (f64 (XSCVSXDDP (COPY_TO_REGCLASS $S, VSFRC)))>;
def : Pat<(f64 (PPCfcfid (PPCmtvsra (i64 (vector_extract v2i64:$S, 1))))),
          (f64 (XSCVSXDDP (COPY_TO_REGCLASS (XXPERMDI $S, $S, 2), VSFRC)))>;
def : Pat<(f64 (PPCfcfidu (PPCmtvsra (i64 (vector_extract v2i64:$S, 0))))),
          (f64 (XSCVUXDDP (COPY_TO_REGCLASS $S, VSFRC)))>;
def : Pat<(f64 (PPCfcfidu (PPCmtvsra (i64 (vector_extract v2i64:$S, 1))))),
          (f64 (XSCVUXDDP (COPY_TO_REGCLASS (XXPERMDI $S, $S, 2), VSFRC)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 2987-3002

```tablegen
def : Pat<(f64 (vector_extract v2f64:$S, i64:$Idx)),
          (f64 VectorExtractions.BE_VARIABLE_DOUBLE)>;

def : Pat<(v2f64 (build_vector f64:$A, f64:$B)),
          (v2f64 (XXPERMDI
                    (SUBREG_TO_REG $A, sub_64),
                    (SUBREG_TO_REG $B, sub_64), 0))>;
// Using VMRGEW to assemble the final vector would be a lower latency
// solution. However, we choose to go with the slightly higher latency
// XXPERMDI for 2 reasons:
// 1. This is likely to occur in unrolled loops where regpressure is high,
//    so we want to use the latter as it has access to all 64 VSX registers.
// 2. Using Altivec instructions in this sequence would likely cause the
//    allocation of Altivec registers even for the loads which in turn would
//    force the use of LXSIWZX for the loads, adding a cycle of latency to
//    each of the loads which would otherwise be able to use LFIWZX.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Using VMRGEW to assemble the final vector would be a lower latency".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Using VMRGEW to assemble the final vector would be a lower latency”。

### Lines 3003-3012

```tablegen
def : Pat<(v4f32 (build_vector LoadFP.A, LoadFP.B, LoadFP.C, LoadFP.D)),
          (v4f32 (XXPERMDI (XXMRGHW MrgFP.LD32A, MrgFP.LD32B),
                           (XXMRGHW MrgFP.LD32C, MrgFP.LD32D), 3))>;
def : Pat<(v4f32 (build_vector f32:$A, f32:$B, f32:$C, f32:$D)),
          (VMRGEW MrgFP.AC, MrgFP.BD)>;
def : Pat<(v4f32 (build_vector DblToFlt.A0, DblToFlt.A1,
                               DblToFlt.B0, DblToFlt.B1)),
          (v4f32 (VMRGEW MrgFP.ABhToFlt, MrgFP.ABlToFlt))>;

// Convert 4 doubles to a vector of ints.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3013-3021

```tablegen
def : Pat<(v4i32 (build_vector DblToInt.A, DblToInt.B,
                               DblToInt.C, DblToInt.D)),
          (v4i32 (VMRGEW MrgWords.CVACS, MrgWords.CVBDS))>;
def : Pat<(v4i32 (build_vector DblToUInt.A, DblToUInt.B,
                               DblToUInt.C, DblToUInt.D)),
          (v4i32 (VMRGEW MrgWords.CVACU, MrgWords.CVBDU))>;
def : Pat<(v4i32 (build_vector ExtDbl.A0S, ExtDbl.A1S,
                               ExtDbl.B0S, ExtDbl.B1S)),
          (v4i32 (VMRGEW MrgWords.CVA0B0S, MrgWords.CVA1B1S))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3022-3031

```tablegen
def : Pat<(v4i32 (build_vector ExtDbl.A0U, ExtDbl.A1U,
                               ExtDbl.B0U, ExtDbl.B1U)),
          (v4i32 (VMRGEW MrgWords.CVA0B0U, MrgWords.CVA1B1U))>;
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 0))),
                               (f64 (fpextend (extractelt v4f32:$A, 1))))),
          (v2f64 (XVCVSPDP (XXMRGHW $A, $A)))>;
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 1))),
                               (f64 (fpextend (extractelt v4f32:$A, 0))))),
          (v2f64 (XXPERMDI (XVCVSPDP (XXMRGHW $A, $A)),
                           (XVCVSPDP (XXMRGHW $A, $A)), 2))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3032-3040

```tablegen
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 0))),
                               (f64 (fpextend (extractelt v4f32:$A, 2))))),
          (v2f64 (XVCVSPDP $A))>;
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 1))),
                               (f64 (fpextend (extractelt v4f32:$A, 3))))),
          (v2f64 (XVCVSPDP (XXSLDWI $A, $A, 1)))>;
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 2))),
                               (f64 (fpextend (extractelt v4f32:$A, 3))))),
          (v2f64 (XVCVSPDP (XXMRGLW $A, $A)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3041-3047

```tablegen
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 3))),
                               (f64 (fpextend (extractelt v4f32:$A, 2))))),
          (v2f64 (XXPERMDI (XVCVSPDP (XXMRGLW $A, $A)),
                           (XVCVSPDP (XXMRGLW $A, $A)), 2))>;
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 0))),
                               (f64 (fpextend (extractelt v4f32:$B, 0))))),
          (v2f64 (XVCVSPDP (XXPERMDI $A, $B, 0)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3048-3055

```tablegen
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 3))),
                               (f64 (fpextend (extractelt v4f32:$B, 3))))),
          (v2f64 (XVCVSPDP (XXSLDWI (XXPERMDI $A, $B, 3),
                                    (XXPERMDI $A, $B, 3), 1)))>;
def : Pat<(v2i64 (fp_to_sint
                   (build_vector (f64 (fpextend (extractelt v4f32:$A, 0))),
                                 (f64 (fpextend (extractelt v4f32:$A, 2)))))),
          (v2i64 (XVCVSPSXDS $A))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3056-3063

```tablegen
def : Pat<(v2i64 (fp_to_uint
                   (build_vector (f64 (fpextend (extractelt v4f32:$A, 0))),
                                 (f64 (fpextend (extractelt v4f32:$A, 2)))))),
          (v2i64 (XVCVSPUXDS $A))>;
def : Pat<(v2i64 (fp_to_sint
                   (build_vector (f64 (fpextend (extractelt v4f32:$A, 1))),
                                 (f64 (fpextend (extractelt v4f32:$A, 3)))))),
          (v2i64 (XVCVSPSXDS (XXSLDWI $A, $A, 1)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3064-3071

```tablegen
def : Pat<(v2i64 (fp_to_uint
                   (build_vector (f64 (fpextend (extractelt v4f32:$A, 1))),
                                 (f64 (fpextend (extractelt v4f32:$A, 3)))))),
          (v2i64 (XVCVSPUXDS (XXSLDWI $A, $A, 1)))>;
def : Pat<WToDPExtractConv.BV02S,
          (v2f64 (XVCVSXWDP $A))>;
def : Pat<WToDPExtractConv.BV13S,
          (v2f64 (XVCVSXWDP (XXSLDWI $A, $A, 1)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3072-3082

```tablegen
def : Pat<WToDPExtractConv.BV02U,
          (v2f64 (XVCVUXWDP $A))>;
def : Pat<WToDPExtractConv.BV13U,
          (v2f64 (XVCVUXWDP (XXSLDWI $A, $A, 1)))>;
def : Pat<(v2f64 (insertelt v2f64:$A, f64:$B, 0)),
          (v2f64 (XXPERMDI (SUBREG_TO_REG $B, sub_64), $A, 1))>;
def : Pat<(v2f64 (insertelt v2f64:$A, f64:$B, 1)),
          (v2f64 (XXPERMDI $A, (SUBREG_TO_REG $B, sub_64), 0))>;
} // HasVSX, IsBigEndian

// Any little endian VSX subtarget.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3083-3090

```tablegen
let Predicates = [HasVSX, IsLittleEndian] in {
defm : ScalToVecWPermute<v2f64, (f64 f64:$A),
                         (XXPERMDI (SUBREG_TO_REG $A, sub_64),
                                   (SUBREG_TO_REG $A, sub_64), 0),
                         (SUBREG_TO_REG $A, sub_64)>;

def : Pat<(f64 (extractelt v2f64:$S, 0)),
          (f64 (EXTRACT_SUBREG (XXPERMDI $S, $S, 2), sub_64))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3091-3097

```tablegen
def : Pat<(f64 (extractelt v2f64:$S, 1)),
          (f64 (EXTRACT_SUBREG $S, sub_64))>;

def : Pat<(v2f64 (PPCld_vec_be ForceXForm:$src)), (LXVD2X ForceXForm:$src)>;
def : Pat<(PPCst_vec_be v2f64:$rS, ForceXForm:$dst), (STXVD2X $rS, ForceXForm:$dst)>;
def : Pat<(v4f32 (PPCld_vec_be ForceXForm:$src)), (LXVW4X ForceXForm:$src)>;
def : Pat<(PPCst_vec_be v4f32:$rS, ForceXForm:$dst), (STXVW4X $rS, ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3098-3105

```tablegen
def : Pat<(v2i64 (PPCld_vec_be ForceXForm:$src)), (LXVD2X ForceXForm:$src)>;
def : Pat<(PPCst_vec_be v2i64:$rS, ForceXForm:$dst), (STXVD2X $rS, ForceXForm:$dst)>;
def : Pat<(v4i32 (PPCld_vec_be ForceXForm:$src)), (LXVW4X ForceXForm:$src)>;
def : Pat<(PPCst_vec_be v4i32:$rS, ForceXForm:$dst), (STXVW4X $rS, ForceXForm:$dst)>;
def : Pat<(f64 (PPCfcfid (PPCmtvsra (i64 (vector_extract v2i64:$S, 0))))),
          (f64 (XSCVSXDDP (COPY_TO_REGCLASS (XXPERMDI $S, $S, 2), VSFRC)))>;
def : Pat<(f64 (PPCfcfid (PPCmtvsra (i64 (vector_extract v2i64:$S, 1))))),
          (f64 (XSCVSXDDP (COPY_TO_REGCLASS (f64 (COPY_TO_REGCLASS $S, VSRC)), VSFRC)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3106-3114

```tablegen
def : Pat<(f64 (PPCfcfidu (PPCmtvsra (i64 (vector_extract v2i64:$S, 0))))),
          (f64 (XSCVUXDDP (COPY_TO_REGCLASS (XXPERMDI $S, $S, 2), VSFRC)))>;
def : Pat<(f64 (PPCfcfidu (PPCmtvsra (i64 (vector_extract v2i64:$S, 1))))),
          (f64 (XSCVUXDDP (COPY_TO_REGCLASS (f64 (COPY_TO_REGCLASS $S, VSRC)), VSFRC)))>;

def : Pat<(f64 (vector_extract v2f64:$S, i64:$Idx)),
          (f64 VectorExtractions.LE_VARIABLE_DOUBLE)>;

// Little endian, available on all targets with VSX
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3115-3127

```tablegen
def : Pat<(v2f64 (build_vector f64:$A, f64:$B)),
          (v2f64 (XXPERMDI
                    (SUBREG_TO_REG $B, sub_64),
                    (SUBREG_TO_REG $A, sub_64), 0))>;
// Using VMRGEW to assemble the final vector would be a lower latency
// solution. However, we choose to go with the slightly higher latency
// XXPERMDI for 2 reasons:
// 1. This is likely to occur in unrolled loops where regpressure is high,
//    so we want to use the latter as it has access to all 64 VSX registers.
// 2. Using Altivec instructions in this sequence would likely cause the
//    allocation of Altivec registers even for the loads which in turn would
//    force the use of LXSIWZX for the loads, adding a cycle of latency to
//    each of the loads which would otherwise be able to use LFIWZX.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Using VMRGEW to assemble the final vector would be a lower latency".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Using VMRGEW to assemble the final vector would be a lower latency”。

### Lines 3128-3137

```tablegen
def : Pat<(v4f32 (build_vector LoadFP.A, LoadFP.B, LoadFP.C, LoadFP.D)),
          (v4f32 (XXPERMDI (XXMRGHW MrgFP.LD32D, MrgFP.LD32C),
                           (XXMRGHW MrgFP.LD32B, MrgFP.LD32A), 3))>;
def : Pat<(v4f32 (build_vector f32:$D, f32:$C, f32:$B, f32:$A)),
          (VMRGEW MrgFP.AC, MrgFP.BD)>;
def : Pat<(v4f32 (build_vector DblToFlt.A0, DblToFlt.A1,
                               DblToFlt.B0, DblToFlt.B1)),
          (v4f32 (VMRGEW MrgFP.BAhToFlt, MrgFP.BAlToFlt))>;

// Convert 4 doubles to a vector of ints.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3138-3146

```tablegen
def : Pat<(v4i32 (build_vector DblToInt.A, DblToInt.B,
                               DblToInt.C, DblToInt.D)),
          (v4i32 (VMRGEW MrgWords.CVDBS, MrgWords.CVCAS))>;
def : Pat<(v4i32 (build_vector DblToUInt.A, DblToUInt.B,
                               DblToUInt.C, DblToUInt.D)),
          (v4i32 (VMRGEW MrgWords.CVDBU, MrgWords.CVCAU))>;
def : Pat<(v4i32 (build_vector ExtDbl.A0S, ExtDbl.A1S,
                               ExtDbl.B0S, ExtDbl.B1S)),
          (v4i32 (VMRGEW MrgWords.CVB1A1S, MrgWords.CVB0A0S))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3147-3156

```tablegen
def : Pat<(v4i32 (build_vector ExtDbl.A0U, ExtDbl.A1U,
                               ExtDbl.B0U, ExtDbl.B1U)),
          (v4i32 (VMRGEW MrgWords.CVB1A1U, MrgWords.CVB0A0U))>;
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 0))),
                               (f64 (fpextend (extractelt v4f32:$A, 1))))),
          (v2f64 (XVCVSPDP (XXMRGLW $A, $A)))>;
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 1))),
                               (f64 (fpextend (extractelt v4f32:$A, 0))))),
          (v2f64 (XXPERMDI (XVCVSPDP (XXMRGLW $A, $A)),
                           (XVCVSPDP (XXMRGLW $A, $A)), 2))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3157-3165

```tablegen
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 0))),
                               (f64 (fpextend (extractelt v4f32:$A, 2))))),
          (v2f64 (XVCVSPDP (XXSLDWI $A, $A, 1)))>;
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 1))),
                               (f64 (fpextend (extractelt v4f32:$A, 3))))),
          (v2f64 (XVCVSPDP $A))>;
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 2))),
                               (f64 (fpextend (extractelt v4f32:$A, 3))))),
          (v2f64 (XVCVSPDP (XXMRGHW $A, $A)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3166-3173

```tablegen
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 3))),
                               (f64 (fpextend (extractelt v4f32:$A, 2))))),
          (v2f64 (XXPERMDI (XVCVSPDP (XXMRGHW $A, $A)),
                           (XVCVSPDP (XXMRGHW $A, $A)), 2))>;
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 0))),
                               (f64 (fpextend (extractelt v4f32:$B, 0))))),
          (v2f64 (XVCVSPDP (XXSLDWI (XXPERMDI $B, $A, 3),
                                    (XXPERMDI $B, $A, 3), 1)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3174-3180

```tablegen
def : Pat<(v2f64 (build_vector (f64 (fpextend (extractelt v4f32:$A, 3))),
                               (f64 (fpextend (extractelt v4f32:$B, 3))))),
          (v2f64 (XVCVSPDP (XXPERMDI $B, $A, 0)))>;
def : Pat<(v2i64 (fp_to_sint
                   (build_vector (f64 (fpextend (extractelt v4f32:$A, 1))),
                                 (f64 (fpextend (extractelt v4f32:$A, 3)))))),
          (v2i64 (XVCVSPSXDS $A))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3181-3188

```tablegen
def : Pat<(v2i64 (fp_to_uint
                   (build_vector (f64 (fpextend (extractelt v4f32:$A, 1))),
                                 (f64 (fpextend (extractelt v4f32:$A, 3)))))),
          (v2i64 (XVCVSPUXDS $A))>;
def : Pat<(v2i64 (fp_to_sint
                   (build_vector (f64 (fpextend (extractelt v4f32:$A, 0))),
                                 (f64 (fpextend (extractelt v4f32:$A, 2)))))),
          (v2i64 (XVCVSPSXDS (XXSLDWI $A, $A, 1)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3189-3196

```tablegen
def : Pat<(v2i64 (fp_to_uint
                   (build_vector (f64 (fpextend (extractelt v4f32:$A, 0))),
                                 (f64 (fpextend (extractelt v4f32:$A, 2)))))),
          (v2i64 (XVCVSPUXDS (XXSLDWI $A, $A, 1)))>;
def : Pat<WToDPExtractConv.BV02S,
          (v2f64 (XVCVSXWDP (XXSLDWI $A, $A, 1)))>;
def : Pat<WToDPExtractConv.BV13S,
          (v2f64 (XVCVSXWDP $A))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3197-3207

```tablegen
def : Pat<WToDPExtractConv.BV02U,
          (v2f64 (XVCVUXWDP (XXSLDWI $A, $A, 1)))>;
def : Pat<WToDPExtractConv.BV13U,
          (v2f64 (XVCVUXWDP $A))>;
def : Pat<(v2f64 (insertelt v2f64:$A, f64:$B, 0)),
          (v2f64 (XXPERMDI $A, (SUBREG_TO_REG $B, sub_64), 0))>;
def : Pat<(v2f64 (insertelt v2f64:$A, f64:$B, 1)),
          (v2f64 (XXPERMDI (SUBREG_TO_REG $B, sub_64), $A, 1))>;
} // HasVSX, IsLittleEndian

// Any pre-Power9 VSX subtarget.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3208-3214

```tablegen
let Predicates = [HasVSX, NoP9Vector] in {
def : Pat<(PPCstore_scal_int_from_vsr f64:$src, ForceXForm:$dst, 8),
          (STXSDX $src, ForceXForm:$dst)>;
def : Pat<(PPCstore_scal_int_from_vsr f128:$src, ForceXForm:$dst, 8),
          (STXSDX (COPY_TO_REGCLASS $src, VSFRC), ForceXForm:$dst)>;

// Load-and-splat with fp-to-int conversion (using X-Form VSX/FP loads).
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3215-3222

```tablegen
defm : ScalToVecWPermute<
  v4i32, DblToIntLoad.A,
  (XXSPLTW (SUBREG_TO_REG (XSCVDPSXWS (XFLOADf64 ForceXForm:$A)), sub_64), 1),
  (SUBREG_TO_REG (XSCVDPSXWS (XFLOADf64 ForceXForm:$A)), sub_64)>;
defm : ScalToVecWPermute<
  v4i32, DblToUIntLoad.A,
  (XXSPLTW (SUBREG_TO_REG (XSCVDPUXWS (XFLOADf64 ForceXForm:$A)), sub_64), 1),
  (SUBREG_TO_REG (XSCVDPUXWS (XFLOADf64 ForceXForm:$A)), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3223-3235

```tablegen
defm : ScalToVecWPermute<
  v2i64, FltToLongLoad.A,
  (XXPERMDIs (XSCVDPSXDS (COPY_TO_REGCLASS (XFLOADf32 ForceXForm:$A), VSFRC)), 0),
  (SUBREG_TO_REG (XSCVDPSXDS (COPY_TO_REGCLASS (XFLOADf32 ForceXForm:$A),
                                                        VSFRC)), sub_64)>;
defm : ScalToVecWPermute<
  v2i64, FltToULongLoad.A,
  (XXPERMDIs (XSCVDPUXDS (COPY_TO_REGCLASS (XFLOADf32 ForceXForm:$A), VSFRC)), 0),
  (SUBREG_TO_REG (XSCVDPUXDS (COPY_TO_REGCLASS (XFLOADf32 ForceXForm:$A),
                                                        VSFRC)), sub_64)>;
} // HasVSX, NoP9Vector

// Any little endian pre-Power9 VSX subtarget.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3236-3247

```tablegen
let Predicates = [HasVSX, NoP9Vector, IsLittleEndian] in {
// Load-and-splat using only X-Form VSX loads.
defm : ScalToVecWPermute<
  v2i64, (i64 (load ForceXForm:$src)),
  (XXPERMDIs (XFLOADf64 ForceXForm:$src), 2),
  (SUBREG_TO_REG (XFLOADf64 ForceXForm:$src), sub_64)>;
defm : ScalToVecWPermute<
  v2f64, (f64 (load ForceXForm:$src)),
  (XXPERMDIs (XFLOADf64 ForceXForm:$src), 2),
  (SUBREG_TO_REG (XFLOADf64 ForceXForm:$src), sub_64)>;

// Splat loads.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3248-3254

```tablegen
def : Pat<(v8i16 (PPCldsplatAlign16 ForceXForm:$A)),
          (v8i16 (VSPLTH 7, (LVX ForceXForm:$A)))>;
def : Pat<(v16i8 (PPCldsplatAlign16 ForceXForm:$A)),
          (v16i8 (VSPLTB 15, (LVX ForceXForm:$A)))>;
} // HasVSX, NoP9Vector, IsLittleEndian

let Predicates = [HasVSX, NoP9Vector, IsBigEndian] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3255-3262

```tablegen
  def : Pat<(v2f64 (int_ppc_vsx_lxvd2x ForceXForm:$src)),
            (LXVD2X ForceXForm:$src)>;
  def : Pat<(int_ppc_vsx_stxvd2x v2f64:$rS, ForceXForm:$dst),
            (STXVD2X $rS, ForceXForm:$dst)>;

  // Splat loads.
  def : Pat<(v8i16 (PPCldsplatAlign16 ForceXForm:$A)),
            (v8i16 (VSPLTH 0, (LVX ForceXForm:$A)))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 3263-3269

```tablegen
  def : Pat<(v16i8 (PPCldsplatAlign16 ForceXForm:$A)),
            (v16i8 (VSPLTB 0, (LVX ForceXForm:$A)))>;
} // HasVSX, NoP9Vector, IsBigEndian

// Any VSX subtarget that only has loads and stores that load in big endian
// order regardless of endianness. This is really pre-Power9 subtargets.
let Predicates = [HasVSX, HasOnlySwappingMemOps] in {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Any VSX subtarget that only has loads and stores that load in big endian". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Any VSX subtarget that only has loads and stores that load in big endian”。 这些声明会进入生成式模式匹配逻辑。

### Lines 3270-3277

```tablegen
  def : Pat<(v2f64 (PPClxvd2x ForceXForm:$src)), (LXVD2X ForceXForm:$src)>;

  // Stores.
  def : Pat<(PPCstxvd2x v2f64:$rS, ForceXForm:$dst), (STXVD2X $rS, ForceXForm:$dst)>;
} // HasVSX, HasOnlySwappingMemOps

// Big endian VSX subtarget that only has loads and stores that always
// load in big endian order. Really big endian pre-Power9 subtargets.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Stores.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Stores.”。

### Lines 3278-3284

```tablegen
let Predicates = [HasVSX, HasOnlySwappingMemOps, IsBigEndian] in {
  def : Pat<(v2f64 (load ForceXForm:$src)), (LXVD2X ForceXForm:$src)>;
  def : Pat<(v2i64 (load ForceXForm:$src)), (LXVD2X ForceXForm:$src)>;
  def : Pat<(v4i32 (load ForceXForm:$src)), (LXVW4X ForceXForm:$src)>;
  def : Pat<(v4i32 (int_ppc_vsx_lxvw4x ForceXForm:$src)), (LXVW4X ForceXForm:$src)>;
  def : Pat<(store v2f64:$rS, ForceXForm:$dst), (STXVD2X $rS, ForceXForm:$dst)>;
  def : Pat<(store v2i64:$rS, ForceXForm:$dst), (STXVD2X $rS, ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3285-3292

```tablegen
  def : Pat<(store v4i32:$XT, ForceXForm:$dst), (STXVW4X $XT, ForceXForm:$dst)>;
  def : Pat<(int_ppc_vsx_stxvw4x v4i32:$rS, ForceXForm:$dst),
            (STXVW4X $rS, ForceXForm:$dst)>;
  def : Pat<(v2i64 (scalar_to_vector (i64 (load ForceXForm:$src)))),
           (SUBREG_TO_REG (XFLOADf64 ForceXForm:$src), sub_64)>;
} // HasVSX, HasOnlySwappingMemOps, IsBigEndian

// Target before Power8 with VSX.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 3293-3303

```tablegen
let Predicates = [HasVSX, NoP8Vector] in {
def : Pat<(f32 (fpimm0neg)),
          (f32 (COPY_TO_REGCLASS (XSNEGDP (XXLXORdpz)), F4RC))>;

def : Pat<(f32 (nzFPImmExactInti5:$A)),
          (COPY_TO_REGCLASS (XVCVSXWDP (COPY_TO_REGCLASS
                     (VSPLTISW (getFPAs5BitExactInt fpimm:$A)), VSRC)), F4RC)>;

} // HasVSX, NoP8Vector

// Any Power8 VSX subtarget.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3304-3310

```tablegen
let Predicates = [HasVSX, HasP8Vector] in {
def : Pat<(int_ppc_vsx_xxleqv v4i32:$A, v4i32:$B),
          (XXLEQV $A, $B)>;
def : Pat<(f64 (extloadf32 XForm:$src)),
          (COPY_TO_REGCLASS (XFLOADf32 XForm:$src), VSFRC)>;
def : Pat<(f32 (fpround (f64 (extloadf32 ForceXForm:$src)))),
          (f32 (XFLOADf32 ForceXForm:$src))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3311-3320

```tablegen
def : Pat<(f64 (any_fpextend f32:$src)),
          (COPY_TO_REGCLASS $src, VSFRC)>;

def : Pat<(f32 (fpimm0neg)),
          (f32 (COPY_TO_REGCLASS (XSNEGDP (XXLXORdpz)), VSSRC))>;

def : Pat<(f32 (nzFPImmExactInti5:$A)),
          (COPY_TO_REGCLASS (XVCVSXWDP (COPY_TO_REGCLASS
                     (VSPLTISW (getFPAs5BitExactInt fpimm:$A)), VSRC)), VSSRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3321-3328

```tablegen
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETLT)),
          (SELECT_VSSRC (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETULT)),
          (SELECT_VSSRC (CRANDC $rhs, $lhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETLE)),
          (SELECT_VSSRC (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETULE)),
          (SELECT_VSSRC (CRORC  $rhs, $lhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3329-3336

```tablegen
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETEQ)),
          (SELECT_VSSRC (CREQV $lhs, $rhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETGE)),
          (SELECT_VSSRC (CRORC  $rhs, $lhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETUGE)),
          (SELECT_VSSRC (CRORC  $lhs, $rhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETGT)),
          (SELECT_VSSRC (CRANDC $rhs, $lhs), $tval, $fval)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3337-3344

```tablegen
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETUGT)),
          (SELECT_VSSRC (CRANDC $lhs, $rhs), $tval, $fval)>;
def : Pat<(f32 (selectcc i1:$lhs, i1:$rhs, f32:$tval, f32:$fval, SETNE)),
          (SELECT_VSSRC (CRXOR $lhs, $rhs), $tval, $fval)>;

// Additional fnmsub pattern for PPC specific ISD opcode
def : Pat<(PPCfnmsub f32:$A, f32:$B, f32:$C),
          (XSNMSUBASP $C, $A, $B)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3345-3353

```tablegen
def : Pat<(fneg (PPCfnmsub f32:$A, f32:$B, f32:$C)),
          (XSMSUBASP $C, $A, $B)>;
def : Pat<(PPCfnmsub f32:$A, f32:$B, (fneg f32:$C)),
          (XSNMADDASP $C, $A, $B)>;

// f32 neg
// Although XSNEGDP is available in P7, we want to select it starting from P8,
// so that FNMSUBS can be selected for fneg-fmsub pattern on P7. (VSX version,
// XSNMSUBASP, is available since P8)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "f32 neg".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“f32 neg”。

### Lines 3354-3360

```tablegen
def : Pat<(f32 (fneg f32:$S)),
          (f32 (COPY_TO_REGCLASS (XSNEGDP
               (COPY_TO_REGCLASS $S, VSFRC)), VSSRC))>;

// Instructions for converting float to i32 feeding a store.
def : Pat<(PPCstore_scal_int_from_vsr f64:$src, ForceXForm:$dst, 4),
          (STIWX $src, ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3361-3368

```tablegen
def : Pat<(PPCstore_scal_int_from_vsr f128:$src, ForceXForm:$dst, 4),
          (STIWX (COPY_TO_REGCLASS $src, VSFRC), ForceXForm:$dst)>;

def : Pat<(PPCstore_scal_int_from_vsr f64:$src, ForceXForm:$dst, 4),
          (STXSIWX $src, ForceXForm:$dst)>;
def : Pat<(PPCstore_scal_int_from_vsr f128:$src, ForceXForm:$dst, 4),
          (STXSIWX (COPY_TO_REGCLASS $src, VSFRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3369-3377

```tablegen
def : Pat<(v2i64 (smax v2i64:$src1, v2i64:$src2)),
          (v2i64 (VMAXSD (COPY_TO_REGCLASS $src1, VRRC),
                         (COPY_TO_REGCLASS $src2, VRRC)))>;
def : Pat<(v2i64 (umax v2i64:$src1, v2i64:$src2)),
          (v2i64 (VMAXUD (COPY_TO_REGCLASS $src1, VRRC),
                         (COPY_TO_REGCLASS $src2, VRRC)))>;
def : Pat<(v2i64 (smin v2i64:$src1, v2i64:$src2)),
          (v2i64 (VMINSD (COPY_TO_REGCLASS $src1, VRRC),
                         (COPY_TO_REGCLASS $src2, VRRC)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3378-3385

```tablegen
def : Pat<(v2i64 (umin v2i64:$src1, v2i64:$src2)),
          (v2i64 (VMINUD (COPY_TO_REGCLASS $src1, VRRC),
                         (COPY_TO_REGCLASS $src2, VRRC)))>;

def : Pat<(v1i128 (bitconvert (v16i8 immAllOnesV))),
          (v1i128 (COPY_TO_REGCLASS(XXLEQVOnes), VSRC))>;
def : Pat<(v2i64 (bitconvert (v16i8 immAllOnesV))),
          (v2i64 (COPY_TO_REGCLASS(XXLEQVOnes), VSRC))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3386-3392

```tablegen
def : Pat<(v8i16 (bitconvert (v16i8 immAllOnesV))),
          (v8i16 (COPY_TO_REGCLASS(XXLEQVOnes), VSRC))>;
def : Pat<(v16i8 (bitconvert (v16i8 immAllOnesV))),
          (v16i8 (COPY_TO_REGCLASS(XXLEQVOnes), VSRC))>;

// XL Compat builtins.
def : Pat<(int_ppc_fmsubs f32:$A, f32:$B, f32:$C), (XSMSUBMSP $A, $B, $C)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3393-3401

```tablegen
def : Pat<(int_ppc_fnmadds f32:$A, f32:$B, f32:$C), (XSNMADDMSP $A, $B, $C)>;
def : Pat<(int_ppc_fres f32:$A), (XSRESP $A)>;
def : Pat<(i32 (int_ppc_extract_exp f64:$A)),
          (EXTRACT_SUBREG (XSXEXPDP (COPY_TO_REGCLASS $A, VSFRC)), sub_32)>;
def : Pat<(int_ppc_extract_sig f64:$A),
          (XSXSIGDP (COPY_TO_REGCLASS $A, VSFRC))>;
def : Pat<(f64 (int_ppc_insert_exp f64:$A, i64:$B)),
          (COPY_TO_REGCLASS (XSIEXPDP (COPY_TO_REGCLASS $A, G8RC), $B), F8RC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3402-3408

```tablegen
def : Pat<(int_ppc_stfiw ForceXForm:$dst, f64:$XT),
          (STXSIWX f64:$XT, ForceXForm:$dst)>;
def : Pat<(int_ppc_frsqrtes vssrc:$XB), (XSRSQRTESP $XB)>;
} // HasVSX, HasP8Vector

// Any big endian Power8 VSX subtarget.
let Predicates = [HasVSX, HasP8Vector, IsBigEndian] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3409-3418

```tablegen
def : Pat<DWToSPExtractConv.El0SS1,
          (f32 (XSCVSXDSP (COPY_TO_REGCLASS $S1, VSFRC)))>;
def : Pat<DWToSPExtractConv.El1SS1,
          (f32 (XSCVSXDSP (COPY_TO_REGCLASS (XXPERMDI $S1, $S1, 2), VSFRC)))>;
def : Pat<DWToSPExtractConv.El0US1,
          (f32 (XSCVUXDSP (COPY_TO_REGCLASS $S1, VSFRC)))>;
def : Pat<DWToSPExtractConv.El1US1,
          (f32 (XSCVUXDSP (COPY_TO_REGCLASS (XXPERMDI $S1, $S1, 2), VSFRC)))>;

// v4f32 scalar <-> vector conversions (BE)
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3419-3425

```tablegen
defm : ScalToVecWPermute<v4f32, (f32 f32:$A), (XSCVDPSPN $A), (XSCVDPSPN $A)>;
def : Pat<(f32 (vector_extract v4f32:$S, 0)),
          (f32 (XSCVSPDPN $S))>;
def : Pat<(f32 (vector_extract v4f32:$S, 1)),
          (f32 (XSCVSPDPN (XXSLDWI $S, $S, 1)))>;
def : Pat<(f32 (vector_extract v4f32:$S, 2)),
          (f32 (XSCVSPDPN (XXPERMDI $S, $S, 2)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3426-3432

```tablegen
def : Pat<(f32 (vector_extract v4f32:$S, 3)),
          (f32 (XSCVSPDPN (XXSLDWI $S, $S, 3)))>;

def : Pat<(f32 (PPCfcfids (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 0)))))),
          (f32 (XSCVSPDPN (XVCVSXWSP (XXSPLTW $A, 0))))>;
def : Pat<(f32 (PPCfcfids (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 1)))))),
          (f32 (XSCVSPDPN (XVCVSXWSP (XXSPLTW $A, 1))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3433-3440

```tablegen
def : Pat<(f32 (PPCfcfids (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 2)))))),
          (f32 (XSCVSPDPN (XVCVSXWSP (XXSPLTW $A, 2))))>;
def : Pat<(f32 (PPCfcfids (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 3)))))),
          (f32 (XSCVSPDPN (XVCVSXWSP (XXSPLTW $A, 3))))>;
def : Pat<(f64 (PPCfcfid (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 0)))))),
          (f64 (COPY_TO_REGCLASS (XVCVSXWDP (XXSPLTW $A, 0)), VSFRC))>;
def : Pat<(f64 (PPCfcfid (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 1)))))),
          (f64 (COPY_TO_REGCLASS (XVCVSXWDP (XXSPLTW $A, 1)), VSFRC))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3441-3448

```tablegen
def : Pat<(f64 (PPCfcfid (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 2)))))),
          (f64 (COPY_TO_REGCLASS (XVCVSXWDP (XXSPLTW $A, 2)), VSFRC))>;
def : Pat<(f64 (PPCfcfid (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 3)))))),
          (f64 (COPY_TO_REGCLASS (XVCVSXWDP (XXSPLTW $A, 3)), VSFRC))>;

def : Pat<(f32 (vector_extract v4f32:$S, i32:$Idx)),
          (f32 VectorExtractions.BE_32B_VARIABLE_FLOAT)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3449-3455

```tablegen
def : Pat<(f64 (vector_extract v2f64:$S, i32:$Idx)),
          (f64 VectorExtractions.BE_32B_VARIABLE_DOUBLE)>;

defm : ScalToVecWPermute<
  v4i32, (i32 (load ForceXForm:$src)),
  (XXSLDWIs (LIWZX ForceXForm:$src), 1),
  (SUBREG_TO_REG (LIWZX ForceXForm:$src), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3456-3462

```tablegen
defm : ScalToVecWPermute<
  v4f32, (f32 (load ForceXForm:$src)),
  (XXSLDWIs (LIWZX ForceXForm:$src), 1),
  (SUBREG_TO_REG (LIWZX ForceXForm:$src), sub_64)>;
} // HasVSX, HasP8Vector, IsBigEndian

// Big endian Power8 64Bit VSX subtarget.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3463-3469

```tablegen
let Predicates = [HasVSX, HasP8Vector, IsBigEndian, IsPPC64] in {
def : Pat<(f32 (vector_extract v4f32:$S, i64:$Idx)),
          (f32 VectorExtractions.BE_VARIABLE_FLOAT)>;

// LIWAX - This instruction is used for sign extending i32 -> i64.
// LIWZX - This instruction will be emitted for i32, f32, and when
//         zero-extending i32 to i64 (zext i32 -> i64).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "LIWAX - This instruction is used for sign extending i32 -> i64.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“LIWAX - This instruction is used for sign extending i32 -> i64.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 3470-3477

```tablegen
def : Pat<(v2i64 (scalar_to_vector (i64 (sextloadi32 ForceXForm:$src)))),
          (v2i64 (SUBREG_TO_REG (LIWAX ForceXForm:$src), sub_64))>;
def : Pat<(v2i64 (scalar_to_vector (i64 (zextloadi32 ForceXForm:$src)))),
          (v2i64 (SUBREG_TO_REG (LIWZX ForceXForm:$src), sub_64))>;

def : Pat<DWToSPExtractConv.BVU,
          (v4f32 (VPKUDUM (XXSLDWI (XVCVUXDSP $S1), (XVCVUXDSP $S1), 3),
                          (XXSLDWI (XVCVUXDSP $S2), (XVCVUXDSP $S2), 3)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3478-3489

```tablegen
def : Pat<DWToSPExtractConv.BVS,
          (v4f32 (VPKUDUM (XXSLDWI (XVCVSXDSP $S1), (XVCVSXDSP $S1), 3),
                          (XXSLDWI (XVCVSXDSP $S2), (XVCVSXDSP $S2), 3)))>;
def : Pat<(store (i32 (extractelt v4i32:$A, 1)), ForceXForm:$src),
          (STIWX (EXTRACT_SUBREG $A, sub_64), ForceXForm:$src)>;
def : Pat<(store (f32 (extractelt v4f32:$A, 1)), ForceXForm:$src),
          (STIWX (EXTRACT_SUBREG $A, sub_64), ForceXForm:$src)>;

// Elements in a register on a BE system are in order <0, 1, 2, 3>.
// The store instructions store the second word from the left.
// So to align element zero, we need to modulo-left-shift by 3 words.
// Similar logic applies for elements 2 and 3.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Elements in a register on a BE system are in order <0, 1, 2, 3>.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Elements in a register on a BE system are in order <0, 1, 2, 3>.”。

### Lines 3490-3500

```tablegen
foreach Idx = [ [0,3], [2,1], [3,2] ] in {
  def : Pat<(store (i32 (extractelt v4i32:$A, !head(Idx))), ForceXForm:$src),
            (STIWX (EXTRACT_SUBREG (XXSLDWI $A, $A, !head(!tail(Idx))),
                                   sub_64), ForceXForm:$src)>;
  def : Pat<(store (f32 (extractelt v4f32:$A, !head(Idx))), ForceXForm:$src),
            (STIWX (EXTRACT_SUBREG (XXSLDWI $A, $A, !head(!tail(Idx))),
                                   sub_64), ForceXForm:$src)>;
}
} // HasVSX, HasP8Vector, IsBigEndian, IsPPC64

// Little endian Power8 VSX subtarget.
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 3501-3508

```tablegen
let Predicates = [HasVSX, HasP8Vector, IsLittleEndian] in {
def : Pat<DWToSPExtractConv.El0SS1,
          (f32 (XSCVSXDSP (COPY_TO_REGCLASS (XXPERMDI $S1, $S1, 2), VSFRC)))>;
def : Pat<DWToSPExtractConv.El1SS1,
          (f32 (XSCVSXDSP (COPY_TO_REGCLASS
                            (f64 (COPY_TO_REGCLASS $S1, VSRC)), VSFRC)))>;
def : Pat<DWToSPExtractConv.El0US1,
          (f32 (XSCVUXDSP (COPY_TO_REGCLASS (XXPERMDI $S1, $S1, 2), VSFRC)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3509-3516

```tablegen
def : Pat<DWToSPExtractConv.El1US1,
          (f32 (XSCVUXDSP (COPY_TO_REGCLASS
                            (f64 (COPY_TO_REGCLASS $S1, VSRC)), VSFRC)))>;

// v4f32 scalar <-> vector conversions (LE)
  defm : ScalToVecWPermute<v4f32, (f32 f32:$A),
                           (XXSLDWI (XSCVDPSPN $A), (XSCVDPSPN $A), 1),
                           (XSCVDPSPN $A)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3517-3524

```tablegen
def : Pat<(f32 (vector_extract v4f32:$S, 0)),
          (f32 (XSCVSPDPN (XXSLDWI $S, $S, 3)))>;
def : Pat<(f32 (vector_extract v4f32:$S, 1)),
          (f32 (XSCVSPDPN (XXPERMDI $S, $S, 2)))>;
def : Pat<(f32 (vector_extract v4f32:$S, 2)),
          (f32 (XSCVSPDPN (XXSLDWI $S, $S, 1)))>;
def : Pat<(f32 (vector_extract v4f32:$S, 3)),
          (f32 (XSCVSPDPN $S))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3525-3531

```tablegen
def : Pat<(f32 (vector_extract v4f32:$S, i64:$Idx)),
          (f32 VectorExtractions.LE_VARIABLE_FLOAT)>;

def : Pat<(f32 (PPCfcfids (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 0)))))),
          (f32 (XSCVSPDPN (XVCVSXWSP (XXSPLTW $A, 3))))>;
def : Pat<(f32 (PPCfcfids (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 1)))))),
          (f32 (XSCVSPDPN (XVCVSXWSP (XXSPLTW $A, 2))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3532-3539

```tablegen
def : Pat<(f32 (PPCfcfids (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 2)))))),
          (f32 (XSCVSPDPN (XVCVSXWSP (XXSPLTW $A, 1))))>;
def : Pat<(f32 (PPCfcfids (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 3)))))),
          (f32 (XSCVSPDPN (XVCVSXWSP (XXSPLTW $A, 0))))>;
def : Pat<(f64 (PPCfcfid (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 0)))))),
          (f64 (COPY_TO_REGCLASS (XVCVSXWDP (XXSPLTW $A, 3)), VSFRC))>;
def : Pat<(f64 (PPCfcfid (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 1)))))),
          (f64 (COPY_TO_REGCLASS (XVCVSXWDP (XXSPLTW $A, 2)), VSFRC))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3540-3547

```tablegen
def : Pat<(f64 (PPCfcfid (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 2)))))),
          (f64 (COPY_TO_REGCLASS (XVCVSXWDP (XXSPLTW $A, 1)), VSFRC))>;
def : Pat<(f64 (PPCfcfid (f64 (PPCmtvsra (i32 (extractelt v4i32:$A, 3)))))),
          (f64 (COPY_TO_REGCLASS (XVCVSXWDP (XXSPLTW $A, 0)), VSFRC))>;

// LIWAX - This instruction is used for sign extending i32 -> i64.
// LIWZX - This instruction will be emitted for i32, f32, and when
//         zero-extending i32 to i64 (zext i32 -> i64).
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "LIWAX - This instruction is used for sign extending i32 -> i64.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“LIWAX - This instruction is used for sign extending i32 -> i64.”。

### Lines 3548-3557

```tablegen
defm : ScalToVecWPermute<
  v2i64, (i64 (sextloadi32 ForceXForm:$src)),
  (XXPERMDIs (LIWAX ForceXForm:$src), 2),
  (SUBREG_TO_REG (LIWAX ForceXForm:$src), sub_64)>;

defm : ScalToVecWPermute<
  v2i64, (i64 (zextloadi32 ForceXForm:$src)),
  (XXPERMDIs (LIWZX ForceXForm:$src), 2),
  (SUBREG_TO_REG (LIWZX ForceXForm:$src), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3558-3567

```tablegen
defm : ScalToVecWPermute<
  v4i32, (i32 (load ForceXForm:$src)),
  (XXPERMDIs (LIWZX ForceXForm:$src), 2),
  (SUBREG_TO_REG (LIWZX ForceXForm:$src), sub_64)>;

defm : ScalToVecWPermute<
  v4f32, (f32 (load ForceXForm:$src)),
  (XXPERMDIs (LIWZX ForceXForm:$src), 2),
  (SUBREG_TO_REG (LIWZX ForceXForm:$src), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3568-3575

```tablegen
def : Pat<DWToSPExtractConv.BVU,
          (v4f32 (VPKUDUM (XXSLDWI (XVCVUXDSP $S2), (XVCVUXDSP $S2), 3),
                          (XXSLDWI (XVCVUXDSP $S1), (XVCVUXDSP $S1), 3)))>;
def : Pat<DWToSPExtractConv.BVS,
          (v4f32 (VPKUDUM (XXSLDWI (XVCVSXDSP $S2), (XVCVSXDSP $S2), 3),
                          (XXSLDWI (XVCVSXDSP $S1), (XVCVSXDSP $S1), 3)))>;
def : Pat<(store (i32 (extractelt v4i32:$A, 2)), ForceXForm:$src),
          (STIWX (EXTRACT_SUBREG $A, sub_64), ForceXForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3576-3582

```tablegen
def : Pat<(store (f32 (extractelt v4f32:$A, 2)), ForceXForm:$src),
          (STIWX (EXTRACT_SUBREG $A, sub_64), ForceXForm:$src)>;

// Elements in a register on a LE system are in order <3, 2, 1, 0>.
// The store instructions store the second word from the left.
// So to align element 3, we need to modulo-left-shift by 3 words.
// Similar logic applies for elements 0 and 1.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Elements in a register on a LE system are in order <3, 2, 1, 0>.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Elements in a register on a LE system are in order <3, 2, 1, 0>.”。

### Lines 3583-3593

```tablegen
foreach Idx = [ [0,2], [1,1], [3,3] ] in {
  def : Pat<(store (i32 (extractelt v4i32:$A, !head(Idx))), ForceXForm:$src),
            (STIWX (EXTRACT_SUBREG (XXSLDWI $A, $A, !head(!tail(Idx))),
                                   sub_64), ForceXForm:$src)>;
  def : Pat<(store (f32 (extractelt v4f32:$A, !head(Idx))), ForceXForm:$src),
            (STIWX (EXTRACT_SUBREG (XXSLDWI $A, $A, !head(!tail(Idx))),
                                   sub_64), ForceXForm:$src)>;
}
} // HasVSX, HasP8Vector, IsLittleEndian

// Big endian pre-Power9 VSX subtarget.
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 3594-3601

```tablegen
let Predicates = [HasVSX, HasP8Vector, NoP9Vector, IsBigEndian, IsPPC64] in {
def : Pat<(store (i64 (extractelt v2i64:$A, 0)), ForceXForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG $A, sub_64), ForceXForm:$src)>;
def : Pat<(store (f64 (extractelt v2f64:$A, 0)), ForceXForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG $A, sub_64), ForceXForm:$src)>;
def : Pat<(store (i64 (extractelt v2i64:$A, 1)), ForceXForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2), sub_64),
                      ForceXForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3602-3608

```tablegen
def : Pat<(store (f64 (extractelt v2f64:$A, 1)), ForceXForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2), sub_64),
                      ForceXForm:$src)>;
} // HasVSX, HasP8Vector, NoP9Vector, IsBigEndian, IsPPC64

// Little endian pre-Power9 VSX subtarget.
let Predicates = [HasVSX, HasP8Vector, NoP9Vector, IsLittleEndian] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3609-3616

```tablegen
def : Pat<(store (i64 (extractelt v2i64:$A, 0)), ForceXForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2), sub_64),
                      ForceXForm:$src)>;
def : Pat<(store (f64 (extractelt v2f64:$A, 0)), ForceXForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2), sub_64),
                      ForceXForm:$src)>;
def : Pat<(store (i64 (extractelt v2i64:$A, 1)), ForceXForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG $A, sub_64), ForceXForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3617-3624

```tablegen
def : Pat<(store (f64 (extractelt v2f64:$A, 1)), ForceXForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG $A, sub_64), ForceXForm:$src)>;
} // HasVSX, HasP8Vector, NoP9Vector, IsLittleEndian

// Any VSX target with direct moves.
let Predicates = [HasVSX, HasDirectMove] in {
// bitconvert f32 -> i32
// (convert to 32-bit fp single, shift right 1 word, move to GPR)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Any VSX target with direct moves.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Any VSX target with direct moves.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 3625-3634

```tablegen
def : Pat<(i32 (bitconvert f32:$A)), Bitcast.FltToInt>;

// bitconvert i32 -> f32
// (move to FPR, shift left 1 word, convert to 64-bit fp single)
def : Pat<(f32 (bitconvert i32:$A)),
          (f32 (XSCVSPDPN
                 (XXSLDWI MovesToVSR.LE_WORD_1, MovesToVSR.LE_WORD_1, 1)))>;

// bitconvert f64 -> i64
// (move to GPR, nothing else needed)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "bitconvert i32 -> f32".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“bitconvert i32 -> f32”。

### Lines 3635-3642

```tablegen
def : Pat<(i64 (bitconvert f64:$A)), Bitcast.DblToLong>;

// bitconvert i64 -> f64
// (move to FPR, nothing else needed)
def : Pat<(f64 (bitconvert i64:$S)),
          (f64 (MTVSRD $S))>;

// Rounding to integer.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "bitconvert i64 -> f64".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“bitconvert i64 -> f64”。

### Lines 3643-3650

```tablegen
def : Pat<(i64 (strict_lrint f64:$S)),
          (i64 (MFVSRD (FCTID $S)))>;
def : Pat<(i64 (strict_lrint f32:$S)),
          (i64 (MFVSRD (FCTID (COPY_TO_REGCLASS $S, F8RC))))>;
def : Pat<(i64 (strict_llrint f64:$S)),
          (i64 (MFVSRD (FCTID $S)))>;
def : Pat<(i64 (strict_llrint f32:$S)),
          (i64 (MFVSRD (FCTID (COPY_TO_REGCLASS $S, F8RC))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3651-3658

```tablegen
def : Pat<(i64 (strict_lround f64:$S)),
          (i64 (MFVSRD (FCTID (XSRDPI $S))))>;
def : Pat<(i64 (strict_lround f32:$S)),
          (i64 (MFVSRD (FCTID (XSRDPI (COPY_TO_REGCLASS $S, VSFRC)))))>;
def : Pat<(i32 (strict_lround f64:$S)),
          (i32 (MFVSRWZ (FCTIW (XSRDPI $S))))>;
def : Pat<(i32 (strict_lround f32:$S)),
          (i32 (MFVSRWZ (FCTIW (XSRDPI (COPY_TO_REGCLASS $S, VSFRC)))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3659-3665

```tablegen
def : Pat<(i64 (strict_llround f64:$S)),
          (i64 (MFVSRD (FCTID (XSRDPI $S))))>;
def : Pat<(i64 (strict_llround f32:$S)),
          (i64 (MFVSRD (FCTID (XSRDPI (COPY_TO_REGCLASS $S, VSFRC)))))>;

// Alternate patterns for PPCmtvsrz where the output is v8i16 or v16i8 instead
// of f64
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Alternate patterns for PPCmtvsrz where the output is v8i16 or v16i8 instead".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Alternate patterns for PPCmtvsrz where the output is v8i16 or v16i8 instead”。

### Lines 3666-3673

```tablegen
def : Pat<(v8i16 (PPCmtvsrz i32:$A)),
          (v8i16 (SUBREG_TO_REG (MTVSRWZ $A), sub_64))>;
def : Pat<(v16i8 (PPCmtvsrz i32:$A)),
          (v16i8 (SUBREG_TO_REG (MTVSRWZ $A), sub_64))>;

// Endianness-neutral constant splat on P8 and newer targets. The reason
// for this pattern is that on targets with direct moves, we don't expand
// BUILD_VECTOR nodes for v4i32.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Endianness-neutral constant splat on P8 and newer targets. The reason".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Endianness-neutral constant splat on P8 and newer targets. The reason”。

### Lines 3674-3680

```tablegen
def : Pat<(v4i32 (build_vector immSExt5NonZero:$A, immSExt5NonZero:$A,
                               immSExt5NonZero:$A, immSExt5NonZero:$A)),
          (v4i32 (VSPLTISW imm:$A))>;

// Splat loads.
def : Pat<(v8i16 (PPCldsplat ForceXForm:$A)),
          (v8i16 (VSPLTHs 3, (MTVSRWZ (LHZX ForceXForm:$A))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3681-3687

```tablegen
def : Pat<(v16i8 (PPCldsplat ForceXForm:$A)),
          (v16i8 (VSPLTBs 7, (MTVSRWZ (LBZX ForceXForm:$A))))>;
} // HasVSX, HasDirectMove

// Big endian VSX subtarget with direct moves.
let Predicates = [HasVSX, HasDirectMove, IsBigEndian] in {
// v16i8 scalar <-> vector conversions (BE)
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Big endian VSX subtarget with direct moves.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Big endian VSX subtarget with direct moves.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 3688-3695

```tablegen
defm : ScalToVecWPermute<
  v16i8, (i32 i32:$A),
  (SUBREG_TO_REG MovesToVSR.BE_BYTE_0, sub_64),
  (SUBREG_TO_REG (MTVSRWZ $A), sub_64)>;
defm : ScalToVecWPermute<
  v8i16, (i32 i32:$A),
  (SUBREG_TO_REG MovesToVSR.BE_HALF_0, sub_64),
  (SUBREG_TO_REG (MTVSRWZ $A), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3696-3703

```tablegen
defm : ScalToVecWPermute<
  v4i32, (i32 i32:$A),
  (SUBREG_TO_REG MovesToVSR.BE_WORD_0, sub_64),
  (SUBREG_TO_REG (MTVSRWZ $A), sub_64)>;
def : Pat<(v2i64 (scalar_to_vector i64:$A)),
          (v2i64 (SUBREG_TO_REG MovesToVSR.BE_DWORD_0, sub_64))>;

// v2i64 scalar <-> vector conversions (BE)
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3704-3712

```tablegen
def : Pat<(i64 (vector_extract v2i64:$S, 0)),
          (i64 VectorExtractions.LE_DWORD_1)>;
def : Pat<(i64 (vector_extract v2i64:$S, 1)),
          (i64 VectorExtractions.LE_DWORD_0)>;
def : Pat<(i64 (vector_extract v2i64:$S, i64:$Idx)),
          (i64 VectorExtractions.BE_VARIABLE_DWORD)>;
} // HasVSX, HasDirectMove, IsBigEndian

// Little endian VSX subtarget with direct moves.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3713-3720

```tablegen
let Predicates = [HasVSX, HasDirectMove, IsLittleEndian] in {
  // v16i8 scalar <-> vector conversions (LE)
  defm : ScalToVecWPermute<v16i8, (i32 i32:$A),
                           (COPY_TO_REGCLASS MovesToVSR.LE_WORD_0, VSRC),
                           (COPY_TO_REGCLASS MovesToVSR.LE_WORD_1, VSRC)>;
  defm : ScalToVecWPermute<v8i16, (i32 i32:$A),
                           (COPY_TO_REGCLASS MovesToVSR.LE_WORD_0, VSRC),
                           (COPY_TO_REGCLASS MovesToVSR.LE_WORD_1, VSRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3721-3728

```tablegen
  defm : ScalToVecWPermute<v4i32, (i32 i32:$A), MovesToVSR.LE_WORD_0,
                           (SUBREG_TO_REG (MTVSRWZ $A), sub_64)>;
  defm : ScalToVecWPermute<v2i64, (i64 i64:$A), MovesToVSR.LE_DWORD_0,
                           MovesToVSR.LE_DWORD_1>;

  // v2i64 scalar <-> vector conversions (LE)
  def : Pat<(i64 (vector_extract v2i64:$S, 0)),
            (i64 VectorExtractions.LE_DWORD_0)>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 3729-3735

```tablegen
  def : Pat<(i64 (vector_extract v2i64:$S, 1)),
            (i64 VectorExtractions.LE_DWORD_1)>;
  def : Pat<(i64 (vector_extract v2i64:$S, i64:$Idx)),
            (i64 VectorExtractions.LE_VARIABLE_DWORD)>;
} // HasVSX, HasDirectMove, IsLittleEndian

// Big endian pre-P9 VSX subtarget with direct moves.
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 3736-3742

```tablegen
let Predicates = [HasVSX, HasDirectMove, NoP9Altivec, IsBigEndian] in {
def : Pat<(i32 (vector_extract v16i8:$S, 0)),
          (i32 VectorExtractions.LE_BYTE_15)>;
def : Pat<(i32 (vector_extract v16i8:$S, 1)),
          (i32 VectorExtractions.LE_BYTE_14)>;
def : Pat<(i32 (vector_extract v16i8:$S, 2)),
          (i32 VectorExtractions.LE_BYTE_13)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3743-3750

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 3)),
          (i32 VectorExtractions.LE_BYTE_12)>;
def : Pat<(i32 (vector_extract v16i8:$S, 4)),
          (i32 VectorExtractions.LE_BYTE_11)>;
def : Pat<(i32 (vector_extract v16i8:$S, 5)),
          (i32 VectorExtractions.LE_BYTE_10)>;
def : Pat<(i32 (vector_extract v16i8:$S, 6)),
          (i32 VectorExtractions.LE_BYTE_9)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3751-3758

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 7)),
          (i32 VectorExtractions.LE_BYTE_8)>;
def : Pat<(i32 (vector_extract v16i8:$S, 8)),
          (i32 VectorExtractions.LE_BYTE_7)>;
def : Pat<(i32 (vector_extract v16i8:$S, 9)),
          (i32 VectorExtractions.LE_BYTE_6)>;
def : Pat<(i32 (vector_extract v16i8:$S, 10)),
          (i32 VectorExtractions.LE_BYTE_5)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3759-3766

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 11)),
          (i32 VectorExtractions.LE_BYTE_4)>;
def : Pat<(i32 (vector_extract v16i8:$S, 12)),
          (i32 VectorExtractions.LE_BYTE_3)>;
def : Pat<(i32 (vector_extract v16i8:$S, 13)),
          (i32 VectorExtractions.LE_BYTE_2)>;
def : Pat<(i32 (vector_extract v16i8:$S, 14)),
          (i32 VectorExtractions.LE_BYTE_1)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3767-3774

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 15)),
          (i32 VectorExtractions.LE_BYTE_0)>;
def : Pat<(i32 (vector_extract v16i8:$S, i64:$Idx)),
          (i32 VectorExtractions.BE_VARIABLE_BYTE)>;

// v8i16 scalar <-> vector conversions (BE)
def : Pat<(i32 (vector_extract v8i16:$S, 0)),
          (i32 VectorExtractions.LE_HALF_7)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3775-3782

```tablegen
def : Pat<(i32 (vector_extract v8i16:$S, 1)),
          (i32 VectorExtractions.LE_HALF_6)>;
def : Pat<(i32 (vector_extract v8i16:$S, 2)),
          (i32 VectorExtractions.LE_HALF_5)>;
def : Pat<(i32 (vector_extract v8i16:$S, 3)),
          (i32 VectorExtractions.LE_HALF_4)>;
def : Pat<(i32 (vector_extract v8i16:$S, 4)),
          (i32 VectorExtractions.LE_HALF_3)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3783-3792

```tablegen
def : Pat<(i32 (vector_extract v8i16:$S, 5)),
          (i32 VectorExtractions.LE_HALF_2)>;
def : Pat<(i32 (vector_extract v8i16:$S, 6)),
          (i32 VectorExtractions.LE_HALF_1)>;
def : Pat<(i32 (vector_extract v8i16:$S, 7)),
          (i32 VectorExtractions.LE_HALF_0)>;
def : Pat<(i32 (vector_extract v8i16:$S, i64:$Idx)),
          (i32 VectorExtractions.BE_VARIABLE_HALF)>;

// v4i32 scalar <-> vector conversions (BE)
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3793-3800

```tablegen
def : Pat<(i32 (vector_extract v4i32:$S, 0)),
          (i32 VectorExtractions.LE_WORD_3)>;
def : Pat<(i32 (vector_extract v4i32:$S, 1)),
          (i32 VectorExtractions.LE_WORD_2)>;
def : Pat<(i32 (vector_extract v4i32:$S, 2)),
          (i32 VectorExtractions.LE_WORD_1)>;
def : Pat<(i32 (vector_extract v4i32:$S, 3)),
          (i32 VectorExtractions.LE_WORD_0)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3801-3808

```tablegen
def : Pat<(i32 (vector_extract v4i32:$S, i64:$Idx)),
          (i32 VectorExtractions.BE_VARIABLE_WORD)>;
} // HasVSX, HasDirectMove, NoP9Altivec, IsBigEndian

// Little endian pre-P9 VSX subtarget with direct moves.
let Predicates = [HasVSX, HasDirectMove, NoP9Altivec, IsLittleEndian] in {
def : Pat<(i32 (vector_extract v16i8:$S, 0)),
          (i32 VectorExtractions.LE_BYTE_0)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3809-3816

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 1)),
          (i32 VectorExtractions.LE_BYTE_1)>;
def : Pat<(i32 (vector_extract v16i8:$S, 2)),
          (i32 VectorExtractions.LE_BYTE_2)>;
def : Pat<(i32 (vector_extract v16i8:$S, 3)),
          (i32 VectorExtractions.LE_BYTE_3)>;
def : Pat<(i32 (vector_extract v16i8:$S, 4)),
          (i32 VectorExtractions.LE_BYTE_4)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3817-3824

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 5)),
          (i32 VectorExtractions.LE_BYTE_5)>;
def : Pat<(i32 (vector_extract v16i8:$S, 6)),
          (i32 VectorExtractions.LE_BYTE_6)>;
def : Pat<(i32 (vector_extract v16i8:$S, 7)),
          (i32 VectorExtractions.LE_BYTE_7)>;
def : Pat<(i32 (vector_extract v16i8:$S, 8)),
          (i32 VectorExtractions.LE_BYTE_8)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3825-3832

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 9)),
          (i32 VectorExtractions.LE_BYTE_9)>;
def : Pat<(i32 (vector_extract v16i8:$S, 10)),
          (i32 VectorExtractions.LE_BYTE_10)>;
def : Pat<(i32 (vector_extract v16i8:$S, 11)),
          (i32 VectorExtractions.LE_BYTE_11)>;
def : Pat<(i32 (vector_extract v16i8:$S, 12)),
          (i32 VectorExtractions.LE_BYTE_12)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3833-3842

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 13)),
          (i32 VectorExtractions.LE_BYTE_13)>;
def : Pat<(i32 (vector_extract v16i8:$S, 14)),
          (i32 VectorExtractions.LE_BYTE_14)>;
def : Pat<(i32 (vector_extract v16i8:$S, 15)),
          (i32 VectorExtractions.LE_BYTE_15)>;
def : Pat<(i32 (vector_extract v16i8:$S, i64:$Idx)),
          (i32 VectorExtractions.LE_VARIABLE_BYTE)>;

// v8i16 scalar <-> vector conversions (LE)
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3843-3850

```tablegen
def : Pat<(i32 (vector_extract v8i16:$S, 0)),
          (i32 VectorExtractions.LE_HALF_0)>;
def : Pat<(i32 (vector_extract v8i16:$S, 1)),
          (i32 VectorExtractions.LE_HALF_1)>;
def : Pat<(i32 (vector_extract v8i16:$S, 2)),
          (i32 VectorExtractions.LE_HALF_2)>;
def : Pat<(i32 (vector_extract v8i16:$S, 3)),
          (i32 VectorExtractions.LE_HALF_3)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3851-3858

```tablegen
def : Pat<(i32 (vector_extract v8i16:$S, 4)),
          (i32 VectorExtractions.LE_HALF_4)>;
def : Pat<(i32 (vector_extract v8i16:$S, 5)),
          (i32 VectorExtractions.LE_HALF_5)>;
def : Pat<(i32 (vector_extract v8i16:$S, 6)),
          (i32 VectorExtractions.LE_HALF_6)>;
def : Pat<(i32 (vector_extract v8i16:$S, 7)),
          (i32 VectorExtractions.LE_HALF_7)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3859-3866

```tablegen
def : Pat<(i32 (vector_extract v8i16:$S, i64:$Idx)),
          (i32 VectorExtractions.LE_VARIABLE_HALF)>;

// v4i32 scalar <-> vector conversions (LE)
def : Pat<(i32 (vector_extract v4i32:$S, 0)),
          (i32 VectorExtractions.LE_WORD_0)>;
def : Pat<(i32 (vector_extract v4i32:$S, 1)),
          (i32 VectorExtractions.LE_WORD_1)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3867-3875

```tablegen
def : Pat<(i32 (vector_extract v4i32:$S, 2)),
          (i32 VectorExtractions.LE_WORD_2)>;
def : Pat<(i32 (vector_extract v4i32:$S, 3)),
          (i32 VectorExtractions.LE_WORD_3)>;
def : Pat<(i32 (vector_extract v4i32:$S, i64:$Idx)),
          (i32 VectorExtractions.LE_VARIABLE_WORD)>;
} // HasVSX, HasDirectMove, NoP9Altivec, IsLittleEndian

// Big endian pre-Power9 64Bit VSX subtarget that has direct moves.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3876-3887

```tablegen
let Predicates = [HasVSX, HasDirectMove, NoP9Vector, IsBigEndian, IsPPC64] in {
// Big endian integer vectors using direct moves.
def : Pat<(v2i64 (build_vector i64:$A, i64:$B)),
          (v2i64 (XXPERMDI
                    (SUBREG_TO_REG (MTVSRD $A), sub_64),
                    (SUBREG_TO_REG (MTVSRD $B), sub_64), 0))>;
def : Pat<(v4i32 (build_vector i32:$A, i32:$B, i32:$C, i32:$D)),
          (XXPERMDI
            (SUBREG_TO_REG
              (MTVSRD (RLDIMI AnyExts.B, AnyExts.A, 32, 0)), sub_64),
            (SUBREG_TO_REG
              (MTVSRD (RLDIMI AnyExts.D, AnyExts.C, 32, 0)), sub_64), 0)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 3888-3894

```tablegen
def : Pat<(v4i32 (build_vector i32:$A, i32:$A, i32:$A, i32:$A)),
          (XXSPLTW (SUBREG_TO_REG (MTVSRWZ $A), sub_64), 1)>;
} // HasVSX, HasDirectMove, NoP9Vector, IsBigEndian, IsPPC64

// Little endian pre-Power9 VSX subtarget that has direct moves.
let Predicates = [HasVSX, HasDirectMove, NoP9Vector, IsLittleEndian] in {
// Little endian integer vectors using direct moves.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Little endian pre-Power9 VSX subtarget that has direct moves.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Little endian pre-Power9 VSX subtarget that has direct moves.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 3895-3904

```tablegen
def : Pat<(v2i64 (build_vector i64:$A, i64:$B)),
          (v2i64 (XXPERMDI
                    (SUBREG_TO_REG (MTVSRD $B), sub_64),
                    (SUBREG_TO_REG (MTVSRD $A), sub_64), 0))>;
def : Pat<(v4i32 (build_vector i32:$A, i32:$B, i32:$C, i32:$D)),
          (XXPERMDI
            (SUBREG_TO_REG
              (MTVSRD (RLDIMI AnyExts.C, AnyExts.D, 32, 0)), sub_64),
            (SUBREG_TO_REG
              (MTVSRD (RLDIMI AnyExts.A, AnyExts.B, 32, 0)), sub_64), 0)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3905-3911

```tablegen
def : Pat<(v4i32 (build_vector i32:$A, i32:$A, i32:$A, i32:$A)),
          (XXSPLTW (SUBREG_TO_REG (MTVSRWZ $A), sub_64), 1)>;
}

// Any Power9 VSX subtarget.
let Predicates = [HasVSX, HasP9Vector] in {
// Additional fnmsub pattern for PPC specific ISD opcode
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Any Power9 VSX subtarget.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Any Power9 VSX subtarget.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 3912-3918

```tablegen
def : Pat<(PPCfnmsub f128:$A, f128:$B, f128:$C),
          (XSNMSUBQP $C, $A, $B)>;
def : Pat<(fneg (PPCfnmsub f128:$A, f128:$B, f128:$C)),
          (XSMSUBQP $C, $A, $B)>;
def : Pat<(PPCfnmsub f128:$A, f128:$B, (fneg f128:$C)),
          (XSNMADDQP $C, $A, $B)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3919-3926

```tablegen
def : Pat<(f128 (any_sint_to_fp i64:$src)),
          (f128 (XSCVSDQP (COPY_TO_REGCLASS $src, VFRC)))>;
def : Pat<(f128 (any_sint_to_fp (i64 (PPCmfvsr f64:$src)))),
          (f128 (XSCVSDQP $src))>;
def : Pat<(f128 (any_sint_to_fp (i32 (PPCmfvsr f64:$src)))),
          (f128 (XSCVSDQP (VEXTSW2Ds $src)))>;
def : Pat<(f128 (any_uint_to_fp i64:$src)),
          (f128 (XSCVUDQP (COPY_TO_REGCLASS $src, VFRC)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3927-3934

```tablegen
def : Pat<(f128 (any_uint_to_fp (i64 (PPCmfvsr f64:$src)))),
          (f128 (XSCVUDQP $src))>;

// Convert (Un)Signed Word -> QP.
def : Pat<(f128 (any_sint_to_fp i32:$src)),
          (f128 (XSCVSDQP (MTVSRWA $src)))>;
def : Pat<(f128 (any_sint_to_fp (i32 (load ForceXForm:$src)))),
          (f128 (XSCVSDQP (LIWAX ForceXForm:$src)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3935-3942

```tablegen
def : Pat<(f128 (any_uint_to_fp i32:$src)),
          (f128 (XSCVUDQP (MTVSRWZ $src)))>;
def : Pat<(f128 (any_uint_to_fp (i32 (load ForceXForm:$src)))),
          (f128 (XSCVUDQP (LIWZX ForceXForm:$src)))>;

// Pattern for matching Vector HP -> Vector SP intrinsic. Defined as a
// separate pattern so that it can convert the input register class from
// VRRC(v8i16) to VSRC.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Pattern for matching Vector HP -> Vector SP intrinsic. Defined as a". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Pattern for matching Vector HP -> Vector SP intrinsic. Defined as a”。 这些声明会进入生成式模式匹配逻辑。

### Lines 3943-3950

```tablegen
def : Pat<(v4f32 (int_ppc_vsx_xvcvhpsp v8i16:$A)),
          (v4f32 (XVCVHPSP (COPY_TO_REGCLASS $A, VSRC)))>;

// Use current rounding mode
def : Pat<(f128 (any_fnearbyint f128:$vB)), (f128 (XSRQPI 0, $vB, 3))>;
// Round to nearest, ties away from zero
def : Pat<(f128 (any_fround f128:$vB)), (f128 (XSRQPI 0, $vB, 0))>;
// Round towards Zero
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Use current rounding mode".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Use current rounding mode”。

### Lines 3951-3958

```tablegen
def : Pat<(f128 (any_ftrunc f128:$vB)), (f128 (XSRQPI 1, $vB, 1))>;
// Round towards +Inf
def : Pat<(f128 (any_fceil f128:$vB)), (f128 (XSRQPI 1, $vB, 2))>;
// Round towards -Inf
def : Pat<(f128 (any_ffloor f128:$vB)), (f128 (XSRQPI 1, $vB, 3))>;
// Use current rounding mode, [with Inexact]
def : Pat<(f128 (any_frint f128:$vB)), (f128 (XSRQPIX 0, $vB, 3))>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Round towards +Inf".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Round towards +Inf”。

### Lines 3959-3966

```tablegen
def : Pat<(f128 (int_ppc_scalar_insert_exp_qp f128:$vA, i64:$vB)),
          (f128 (XSIEXPQP $vA, (MTVSRD $vB)))>;

def : Pat<(i64 (int_ppc_scalar_extract_expq  f128:$vA)),
          (i64 (MFVSRD (EXTRACT_SUBREG
                          (v2i64 (XSXEXPQP $vA)), sub_64)))>;

// Extra patterns expanding to vector Extract Word/Insert Word
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3967-3974

```tablegen
def : Pat<(v4i32 (int_ppc_vsx_xxinsertw v4i32:$A, v2i64:$B, imm:$IMM)),
          (v4i32 (XXINSERTW $A, $B, imm:$IMM))>;
def : Pat<(v2i64 (int_ppc_vsx_xxextractuw v2i64:$A, imm:$IMM)),
          (v2i64 (COPY_TO_REGCLASS (XXEXTRACTUW $A, imm:$IMM), VSRC))>;

// Vector Reverse
def : Pat<(v8i16 (bswap v8i16 :$A)),
          (v8i16 (COPY_TO_REGCLASS (XXBRH (COPY_TO_REGCLASS $A, VSRC)), VRRC))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3975-3981

```tablegen
def : Pat<(v1i128 (bswap v1i128 :$A)),
          (v1i128 (COPY_TO_REGCLASS (XXBRQ (COPY_TO_REGCLASS $A, VSRC)), VRRC))>;

// D-Form Load/Store
foreach Ty = [v4i32, v4f32, v2i64, v2f64] in {
  def : Pat<(Ty (load DQForm:$src)), (LXV memrix16:$src)>;
  def : Pat<(Ty (load XForm:$src)), (LXVX XForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3982-3989

```tablegen
  def : Pat<(store Ty:$rS, DQForm:$dst), (STXV $rS, memrix16:$dst)>;
  def : Pat<(store Ty:$rS, XForm:$dst), (STXVX $rS, XForm:$dst)>;
}

def : Pat<(f128 (load DQForm:$src)),
          (COPY_TO_REGCLASS (LXV memrix16:$src), VRRC)>;
def : Pat<(f128 (load XForm:$src)),
          (COPY_TO_REGCLASS (LXVX XForm:$src), VRRC)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3990-3996

```tablegen
def : Pat<(v4i32 (int_ppc_vsx_lxvw4x DQForm:$src)), (LXV memrix16:$src)>;
def : Pat<(v2f64 (int_ppc_vsx_lxvd2x DQForm:$src)), (LXV memrix16:$src)>;
def : Pat<(v4i32 (int_ppc_vsx_lxvw4x XForm:$src)), (LXVX XForm:$src)>;
def : Pat<(v2f64 (int_ppc_vsx_lxvd2x XForm:$src)), (LXVX XForm:$src)>;

def : Pat<(store f128:$rS, DQForm:$dst),
          (STXV (COPY_TO_REGCLASS $rS, VSRC), memrix16:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 3997-4004

```tablegen
def : Pat<(store f128:$rS, XForm:$dst),
          (STXVX (COPY_TO_REGCLASS $rS, VSRC), XForm:$dst)>;
def : Pat<(int_ppc_vsx_stxvw4x v4i32:$rS, DQForm:$dst),
          (STXV $rS, memrix16:$dst)>;
def : Pat<(int_ppc_vsx_stxvd2x v2f64:$rS, DQForm:$dst),
          (STXV $rS, memrix16:$dst)>;
def : Pat<(int_ppc_vsx_stxvw4x v4i32:$rS, XForm:$dst),
          (STXVX $rS, XForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4005-4011

```tablegen
def : Pat<(int_ppc_vsx_stxvd2x v2f64:$rS, XForm:$dst),
          (STXVX $rS, XForm:$dst)>;

// Build vectors from i8 loads
defm : ScalToVecWPermute<v8i16, ScalarLoads.ZELi8,
                         (VSPLTHs 3, (LXSIBZX ForceXForm:$src)),
                         (SUBREG_TO_REG (LXSIBZX ForceXForm:$src), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4012-4021

```tablegen
defm : ScalToVecWPermute<v4i32, ScalarLoads.ZELi8,
                         (XXSPLTWs (LXSIBZX ForceXForm:$src), 1),
                         (SUBREG_TO_REG (LXSIBZX ForceXForm:$src), sub_64)>;
defm : ScalToVecWPermute<v2i64, ScalarLoads.ZELi8i64,
                         (XXPERMDIs (LXSIBZX ForceXForm:$src), 0),
                         (SUBREG_TO_REG (LXSIBZX ForceXForm:$src), sub_64)>;
defm : ScalToVecWPermute<
  v4i32, ScalarLoads.SELi8,
  (XXSPLTWs (VEXTSB2Ws (LXSIBZX ForceXForm:$src)), 1),
  (SUBREG_TO_REG (VEXTSB2Ws (LXSIBZX ForceXForm:$src)), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4022-4031

```tablegen
defm : ScalToVecWPermute<
  v2i64, ScalarLoads.SELi8i64,
  (XXPERMDIs (VEXTSB2Ds (LXSIBZX ForceXForm:$src)), 0),
  (SUBREG_TO_REG (VEXTSB2Ds (LXSIBZX ForceXForm:$src)), sub_64)>;

// Build vectors from i16 loads
defm : ScalToVecWPermute<
  v4i32, ScalarLoads.ZELi16,
  (XXSPLTWs (LXSIHZX ForceXForm:$src), 1),
  (SUBREG_TO_REG (LXSIHZX ForceXForm:$src), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4032-4039

```tablegen
defm : ScalToVecWPermute<
  v2i64, ScalarLoads.ZELi16i64,
  (XXPERMDIs (LXSIHZX ForceXForm:$src), 0),
  (SUBREG_TO_REG (LXSIHZX ForceXForm:$src), sub_64)>;
defm : ScalToVecWPermute<
  v4i32, ScalarLoads.SELi16,
  (XXSPLTWs (VEXTSH2Ws (LXSIHZX ForceXForm:$src)), 1),
  (SUBREG_TO_REG (VEXTSH2Ws (LXSIHZX ForceXForm:$src)), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4040-4047

```tablegen
defm : ScalToVecWPermute<
  v2i64, ScalarLoads.SELi16i64,
  (XXPERMDIs (VEXTSH2Ds (LXSIHZX ForceXForm:$src)), 0),
  (SUBREG_TO_REG (VEXTSH2Ds (LXSIHZX ForceXForm:$src)), sub_64)>;

// Load/convert and convert/store patterns for f16.
def : Pat<(f128 (extloadf16 ForceXForm:$src)),
          (f128 (XSCVDPQP (XSCVHPDP (LXSIHZX ForceXForm:$src))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4048-4055

```tablegen
def : Pat<(f64 (extloadf16 ForceXForm:$src)),
          (f64 (XSCVHPDP (LXSIHZX ForceXForm:$src)))>;
def : Pat<(truncstoref16 f64:$src, ForceXForm:$dst),
          (STXSIHX (XSCVDPHP $src), ForceXForm:$dst)>;
def : Pat<(f32 (extloadf16 ForceXForm:$src)),
          (f32 (COPY_TO_REGCLASS (XSCVHPDP (LXSIHZX ForceXForm:$src)), VSSRC))>;
def : Pat<(truncstoref16 f32:$src, ForceXForm:$dst),
          (STXSIHX (XSCVDPHP (COPY_TO_REGCLASS $src, VSFRC)), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4056-4063

```tablegen
def : Pat<(f128 (f16_to_fp i32:$A)),
          (f128 (XSCVDPQP (XSCVHPDP (MTVSRWZ $A))))>;
def : Pat<(f64 (f16_to_fp i32:$A)),
          (f64 (XSCVHPDP (MTVSRWZ $A)))>;
def : Pat<(f32 (f16_to_fp i32:$A)),
          (f32 (COPY_TO_REGCLASS (XSCVHPDP (MTVSRWZ $A)), VSSRC))>;
def : Pat<(i32 (fp_to_f16 f32:$A)),
          (i32 (MFVSRWZ (XSCVDPHP (COPY_TO_REGCLASS $A, VSFRC))))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4064-4071

```tablegen
def : Pat<(i32 (fp_to_f16 f64:$A)), (i32 (MFVSRWZ (XSCVDPHP $A)))>;

// Vector sign extensions
def : Pat<(f64 (PPCVexts f64:$A, 1)),
          (f64 (COPY_TO_REGCLASS (VEXTSB2Ds $A), VSFRC))>;
def : Pat<(f64 (PPCVexts f64:$A, 2)),
          (f64 (COPY_TO_REGCLASS (VEXTSH2Ds $A), VSFRC))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4072-4078

```tablegen
def : Pat<(f64 (extloadf32 DSForm:$src)),
          (COPY_TO_REGCLASS (DFLOADf32 DSForm:$src), VSFRC)>;
def : Pat<(f32 (fpround (f64 (extloadf32 DSForm:$src)))),
          (f32 (DFLOADf32 DSForm:$src))>;

def : Pat<(v4f32 (PPCldvsxlh XForm:$src)),
          (SUBREG_TO_REG (XFLOADf64 XForm:$src), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4079-4086

```tablegen
def : Pat<(v4f32 (PPCldvsxlh DSForm:$src)),
          (SUBREG_TO_REG (DFLOADf64 DSForm:$src), sub_64)>;

// Convert (Un)Signed DWord in memory -> QP
def : Pat<(f128 (sint_to_fp (i64 (load XForm:$src)))),
          (f128 (XSCVSDQP (LXSDX XForm:$src)))>;
def : Pat<(f128 (sint_to_fp (i64 (load DSForm:$src)))),
          (f128 (XSCVSDQP (LXSD DSForm:$src)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4087-4096

```tablegen
def : Pat<(f128 (uint_to_fp (i64 (load XForm:$src)))),
          (f128 (XSCVUDQP (LXSDX XForm:$src)))>;
def : Pat<(f128 (uint_to_fp (i64 (load DSForm:$src)))),
          (f128 (XSCVUDQP (LXSD DSForm:$src)))>;

// Convert Unsigned HWord in memory -> QP
def : Pat<(f128 (uint_to_fp ScalarLoads.ZELi16)),
          (f128 (XSCVUDQP (LXSIHZX XForm:$src)))>;

// Convert Unsigned Byte in memory -> QP
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4097-4104

```tablegen
def : Pat<(f128 (uint_to_fp ScalarLoads.ZELi8)),
          (f128 (XSCVUDQP (LXSIBZX ForceXForm:$src)))>;

// Truncate & Convert QP -> (Un)Signed (D)Word.
def : Pat<(i64 (any_fp_to_sint f128:$src)), (i64 (MFVRD (XSCVQPSDZ $src)))>;
def : Pat<(i64 (any_fp_to_uint f128:$src)), (i64 (MFVRD (XSCVQPUDZ $src)))>;
def : Pat<(i32 (any_fp_to_sint f128:$src)),
          (i32 (MFVSRWZ (COPY_TO_REGCLASS (XSCVQPSWZ $src), VFRC)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4105-4112

```tablegen
def : Pat<(i32 (any_fp_to_uint f128:$src)),
          (i32 (MFVSRWZ (COPY_TO_REGCLASS (XSCVQPUWZ $src), VFRC)))>;

// Instructions for store(fptosi).
def : Pat<(PPCstore_scal_int_from_vsr f64:$src, DSForm:$dst, 8),
          (STXSD $src, DSForm:$dst)>;
def : Pat<(PPCstore_scal_int_from_vsr f64:$src, ForceXForm:$dst, 2),
          (STXSIHX $src, ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4113-4119

```tablegen
def : Pat<(PPCstore_scal_int_from_vsr f64:$src, ForceXForm:$dst, 1),
          (STXSIBX $src, ForceXForm:$dst)>;

def : Pat<(PPCstore_scal_int_from_vsr f128:$src, DSForm:$dst, 8),
          (STXSD (COPY_TO_REGCLASS $src, VFRC), DSForm:$dst)>;
def : Pat<(PPCstore_scal_int_from_vsr f128:$src, ForceXForm:$dst, 2),
          (STXSIHX (COPY_TO_REGCLASS $src, VSFRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4120-4127

```tablegen
def : Pat<(PPCstore_scal_int_from_vsr f128:$src, ForceXForm:$dst, 1),
          (STXSIBX (COPY_TO_REGCLASS $src, VSFRC), ForceXForm:$dst)>;

// Round & Convert QP -> DP/SP
def : Pat<(f64 (any_fpround f128:$src)), (f64 (XSCVQPDP $src))>;
def : Pat<(f32 (any_fpround f128:$src)), (f32 (XSRSP (XSCVQPDPO $src)))>;

// Convert SP -> QP
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Round & Convert QP -> DP/SP".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Round & Convert QP -> DP/SP”。

### Lines 4128-4134

```tablegen
def : Pat<(f128 (any_fpextend f32:$src)),
          (f128 (XSCVDPQP (COPY_TO_REGCLASS $src, VFRC)))>;

def : Pat<(f32 (PPCxsmaxc f32:$XA, f32:$XB)),
          (f32 (COPY_TO_REGCLASS (XSMAXCDP (COPY_TO_REGCLASS $XA, VSSRC),
                                           (COPY_TO_REGCLASS $XB, VSSRC)),
                                 VSSRC))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4135-4142

```tablegen
def : Pat<(f32 (PPCxsminc f32:$XA, f32:$XB)),
          (f32 (COPY_TO_REGCLASS (XSMINCDP (COPY_TO_REGCLASS $XA, VSSRC),
                                           (COPY_TO_REGCLASS $XB, VSSRC)),
                                 VSSRC))>;

// Endianness-neutral patterns for const splats with ISA 3.0 instructions.
defm : ScalToVecWPermute<v4i32, (i32 i32:$A), (MTVSRWS $A),
                         (SUBREG_TO_REG (MTVSRWZ $A), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4143-4153

```tablegen
def : Pat<(v4i32 (build_vector i32:$A, i32:$A, i32:$A, i32:$A)),
          (v4i32 (MTVSRWS $A))>;
def : Pat<(v16i8 (build_vector immNonAllOneAnyExt8:$A, immNonAllOneAnyExt8:$A,
                               immNonAllOneAnyExt8:$A, immNonAllOneAnyExt8:$A,
                               immNonAllOneAnyExt8:$A, immNonAllOneAnyExt8:$A,
                               immNonAllOneAnyExt8:$A, immNonAllOneAnyExt8:$A,
                               immNonAllOneAnyExt8:$A, immNonAllOneAnyExt8:$A,
                               immNonAllOneAnyExt8:$A, immNonAllOneAnyExt8:$A,
                               immNonAllOneAnyExt8:$A, immNonAllOneAnyExt8:$A,
                               immNonAllOneAnyExt8:$A, immNonAllOneAnyExt8:$A)),
          (v16i8 (COPY_TO_REGCLASS (XXSPLTIB imm:$A), VSRC))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4154-4161

```tablegen
defm : ScalToVecWPermute<
  v4i32, FltToIntLoad.A,
  (XVCVSPSXWS (LXVWSX ForceXForm:$A)),
  (XVCVSPSXWS (SUBREG_TO_REG (LIWZX ForceXForm:$A), sub_64))>;
defm : ScalToVecWPermute<
  v4i32, FltToUIntLoad.A,
  (XVCVSPUXWS (LXVWSX ForceXForm:$A)),
  (XVCVSPUXWS (SUBREG_TO_REG (LIWZX ForceXForm:$A), sub_64))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4162-4169

```tablegen
defm : ScalToVecWPermute<
  v4i32, DblToIntLoadP9.A,
  (XXSPLTW (SUBREG_TO_REG (XSCVDPSXWS (DFLOADf64 DSForm:$A)), sub_64), 1),
  (SUBREG_TO_REG (XSCVDPSXWS (DFLOADf64 DSForm:$A)), sub_64)>;
defm : ScalToVecWPermute<
  v4i32, DblToUIntLoadP9.A,
  (XXSPLTW (SUBREG_TO_REG (XSCVDPUXWS (DFLOADf64 DSForm:$A)), sub_64), 1),
  (SUBREG_TO_REG (XSCVDPUXWS (DFLOADf64 DSForm:$A)), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4170-4179

```tablegen
defm : ScalToVecWPermute<
  v2i64, FltToLongLoadP9.A,
  (XXPERMDIs (XSCVDPSXDS (COPY_TO_REGCLASS (DFLOADf32 DSForm:$A), VSFRC)), 0),
  (SUBREG_TO_REG
     (XSCVDPSXDS (COPY_TO_REGCLASS (DFLOADf32 DSForm:$A), VSFRC)), sub_64)>;
defm : ScalToVecWPermute<
  v2i64, FltToULongLoadP9.A,
  (XXPERMDIs (XSCVDPUXDS (COPY_TO_REGCLASS (DFLOADf32 DSForm:$A), VSFRC)), 0),
  (SUBREG_TO_REG
     (XSCVDPUXDS (COPY_TO_REGCLASS (DFLOADf32 DSForm:$A), VSFRC)), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4180-4187

```tablegen
def : Pat<(v4f32 (PPCldsplat ForceXForm:$A)),
          (v4f32 (LXVWSX ForceXForm:$A))>;
def : Pat<(v4i32 (PPCldsplat ForceXForm:$A)),
          (v4i32 (LXVWSX ForceXForm:$A))>;
def : Pat<(v8i16 (PPCldsplat ForceXForm:$A)),
          (v8i16 (VSPLTHs 3, (LXSIHZX ForceXForm:$A)))>;
def : Pat<(v16i8 (PPCldsplat ForceXForm:$A)),
          (v16i8 (VSPLTBs 7, (LXSIBZX ForceXForm:$A)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4188-4195

```tablegen
def : Pat<(v2f64 (PPCxxperm v2f64:$XT, v2f64:$XB, v4i32:$C)),
          (XXPERM v2f64:$XT, v2f64:$XB, v4i32:$C)>;
} // HasVSX, HasP9Vector

// Any Power9 VSX subtarget with equivalent length but better Power10 VSX
// patterns.
// Two identical blocks are required due to the slightly different predicates:
// One without P10 instructions, the other is BigEndian only with P10 instructions.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Any Power9 VSX subtarget with equivalent length but better Power10 VSX".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Any Power9 VSX subtarget with equivalent length but better Power10 VSX”。

### Lines 4196-4205

```tablegen
let Predicates = [HasVSX, HasP9Vector, NoP10Vector] in {
// Little endian Power10 subtargets produce a shorter pattern but require a
// COPY_TO_REGCLASS. The COPY_TO_REGCLASS makes it appear to need two instructions 
// to perform the operation, when only one instruction is produced in practice.
// The NoP10Vector predicate excludes these patterns from Power10 VSX subtargets.
defm : ScalToVecWPermute<
  v16i8, ScalarLoads.Li8,
  (VSPLTBs 7, (LXSIBZX ForceXForm:$src)),
  (SUBREG_TO_REG (LXSIBZX ForceXForm:$src), sub_64)>;
// Build vectors from i16 loads
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Little endian Power10 subtargets produce a shorter pattern but require a". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Little endian Power10 subtargets produce a shorter pattern but require a”。 这些声明会进入生成式模式匹配逻辑。

### Lines 4206-4212

```tablegen
defm : ScalToVecWPermute<
  v8i16, ScalarLoads.Li16,
  (VSPLTHs 3, (LXSIHZX ForceXForm:$src)),
  (SUBREG_TO_REG (LXSIHZX ForceXForm:$src), sub_64)>;
} // HasVSX, HasP9Vector, NoP10Vector

// Any big endian Power9 VSX subtarget
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4213-4221

```tablegen
let Predicates = [HasVSX, HasP9Vector, IsBigEndian] in {
// Power10 VSX subtargets produce a shorter pattern for little endian targets
// but this is still the best pattern for Power9 and Power10 VSX big endian
// Build vectors from i8 loads
defm : ScalToVecWPermute<
  v16i8, ScalarLoads.Li8,
  (VSPLTBs 7, (LXSIBZX ForceXForm:$src)),
  (SUBREG_TO_REG (LXSIBZX ForceXForm:$src), sub_64)>;
// Build vectors from i16 loads
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Power10 VSX subtargets produce a shorter pattern for little endian targets". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Power10 VSX subtargets produce a shorter pattern for little endian targets”。 这些声明会进入生成式模式匹配逻辑。

### Lines 4222-4228

```tablegen
defm : ScalToVecWPermute<
  v8i16, ScalarLoads.Li16,
  (VSPLTHs 3, (LXSIHZX ForceXForm:$src)),
  (SUBREG_TO_REG (LXSIHZX ForceXForm:$src), sub_64)>;

def : Pat<(f32 (PPCfcfidus (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 0)))))),
          (f32 (XSCVUXDSP (XXEXTRACTUW $A, 0)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4229-4236

```tablegen
def : Pat<(f32 (PPCfcfidus (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 1)))))),
          (f32 (XSCVUXDSP (XXEXTRACTUW $A, 4)))>;
def : Pat<(f32 (PPCfcfidus (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 2)))))),
          (f32 (XSCVUXDSP (XXEXTRACTUW $A, 8)))>;
def : Pat<(f32 (PPCfcfidus (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 3)))))),
          (f32 (XSCVUXDSP (XXEXTRACTUW $A, 12)))>;
def : Pat<(f64 (PPCfcfidu (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 0)))))),
          (f64 (XSCVUXDDP (XXEXTRACTUW $A, 0)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4237-4244

```tablegen
def : Pat<(f64 (PPCfcfidu (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 1)))))),
          (f64 (XSCVUXDDP (XXEXTRACTUW $A, 4)))>;
def : Pat<(f64 (PPCfcfidu (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 2)))))),
          (f64 (XSCVUXDDP (XXEXTRACTUW $A, 8)))>;
def : Pat<(f64 (PPCfcfidu (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 3)))))),
          (f64 (XSCVUXDDP (XXEXTRACTUW $A, 12)))>;
def : Pat<(v4i32 (insertelt v4i32:$A, i32:$B, 0)),
          (v4i32 (XXINSERTW v4i32:$A, AlignValues.I32_TO_BE_WORD1, 0))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4245-4252

```tablegen
def : Pat<(v4i32 (insertelt v4i32:$A, DblToInt.B, 0)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPSXWS f64:$B), sub_64),
                            0))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToUInt.B, 0)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPUXWS f64:$B), sub_64),
                            0))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4253-4262

```tablegen
def : Pat<(v4i32 (insertelt v4i32:$A, i32:$B, 1)),
          (v4i32 (XXINSERTW v4i32:$A, AlignValues.I32_TO_BE_WORD1, 4))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToInt.B, 1)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPSXWS f64:$B), sub_64),
                            4))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToUInt.B, 1)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPUXWS f64:$B), sub_64),
                            4))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4263-4272

```tablegen
def : Pat<(v4i32 (insertelt v4i32:$A, i32:$B, 2)),
          (v4i32 (XXINSERTW v4i32:$A, AlignValues.I32_TO_BE_WORD1, 8))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToInt.B, 2)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPSXWS f64:$B), sub_64),
                            8))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToUInt.B, 2)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPUXWS f64:$B), sub_64),
                            8))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4273-4282

```tablegen
def : Pat<(v4i32 (insertelt v4i32:$A, i32:$B, 3)),
          (v4i32 (XXINSERTW v4i32:$A, AlignValues.I32_TO_BE_WORD1, 12))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToInt.B, 3)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPSXWS f64:$B), sub_64),
                            12))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToUInt.B, 3)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPUXWS f64:$B), sub_64),
                            12))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4283-4291

```tablegen
def : Pat<(v4f32 (insertelt v4f32:$A, f32:$B, 0)),
          (v4f32 (XXINSERTW v4f32:$A, AlignValues.F32_TO_BE_WORD1, 0))>;
def : Pat<(v4f32 (insertelt v4f32:$A, f32:$B, 1)),
          (v4f32 (XXINSERTW v4f32:$A, AlignValues.F32_TO_BE_WORD1, 4))>;
def : Pat<(v4f32 (insertelt v4f32:$A, f32:$B, 2)),
          (v4f32 (XXINSERTW v4f32:$A, AlignValues.F32_TO_BE_WORD1, 8))>;
def : Pat<(v4f32 (insertelt v4f32:$A, f32:$B, 3)),
          (v4f32 (XXINSERTW v4f32:$A, AlignValues.F32_TO_BE_WORD1, 12))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4292-4300

```tablegen
def : Pat<(v4f32 (insertelt v4f32:$A, (f32 (fpround f64:$B)), 0)),
          (v4f32 (XXINSERTW v4f32:$A,
                  (SUBREG_TO_REG (XSCVDPSP f64:$B), sub_64), 0))>;
def : Pat<(v4f32 (insertelt v4f32:$A, (f32 (fpround f64:$B)), 1)),
          (v4f32 (XXINSERTW v4f32:$A,
                  (SUBREG_TO_REG (XSCVDPSP f64:$B), sub_64), 4))>;
def : Pat<(v4f32 (insertelt v4f32:$A, (f32 (fpround f64:$B)), 2)),
          (v4f32 (XXINSERTW v4f32:$A,
                  (SUBREG_TO_REG (XSCVDPSP f64:$B), sub_64), 8))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4301-4307

```tablegen
def : Pat<(v4f32 (insertelt v4f32:$A, (f32 (fpround f64:$B)), 3)),
          (v4f32 (XXINSERTW v4f32:$A,
                  (SUBREG_TO_REG (XSCVDPSP f64:$B), sub_64), 12))>;

// Scalar stores of i8
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 0)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 9)), VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4308-4315

```tablegen
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 1)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 10)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 2)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 11)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 3)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 12)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 4)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 13)), VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4316-4323

```tablegen
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 5)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 14)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 6)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 15)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 7)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS $S, VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 8)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 1)), VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4324-4331

```tablegen
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 9)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 2)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 10)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 3)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 11)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 4)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 12)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 5)), VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4332-4339

```tablegen
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 13)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 6)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 14)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 7)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 15)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 8)), VSRC), ForceXForm:$dst)>;

// Scalar stores of i16
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4340-4347

```tablegen
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 0)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 10)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 1)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 12)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 2)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 14)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 3)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS $S, VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4348-4358

```tablegen
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 4)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 2)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 5)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 4)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 6)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 6)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 7)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 8)), VSRC), ForceXForm:$dst)>;
} // HasVSX, HasP9Vector, IsBigEndian

// Big endian 64Bit Power9 subtarget.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4359-4366

```tablegen
let Predicates = [HasVSX, HasP9Vector, IsBigEndian, IsPPC64] in {
def : Pat<(v2i64 (scalar_to_vector (i64 (load DSForm:$src)))),
          (v2i64 (SUBREG_TO_REG (DFLOADf64 DSForm:$src), sub_64))>;
def : Pat<(v2i64 (scalar_to_vector (i64 (load XForm:$src)))),
          (v2i64 (SUBREG_TO_REG (XFLOADf64 XForm:$src), sub_64))>;

def : Pat<(v2f64 (scalar_to_vector (f64 (load DSForm:$src)))),
          (v2f64 (SUBREG_TO_REG (DFLOADf64 DSForm:$src), sub_64))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4367-4374

```tablegen
def : Pat<(v2f64 (scalar_to_vector (f64 (load XForm:$src)))),
          (v2f64 (SUBREG_TO_REG (XFLOADf64 XForm:$src), sub_64))>;
def : Pat<(store (i64 (extractelt v2i64:$A, 1)), XForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2),
                       sub_64), XForm:$src)>;
def : Pat<(store (f64 (extractelt v2f64:$A, 1)), XForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2),
                       sub_64), XForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4375-4381

```tablegen
def : Pat<(store (i64 (extractelt v2i64:$A, 0)), XForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG $A, sub_64), XForm:$src)>;
def : Pat<(store (f64 (extractelt v2f64:$A, 0)), XForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG $A, sub_64), XForm:$src)>;
def : Pat<(store (i64 (extractelt v2i64:$A, 1)), DSForm:$src),
          (DFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2),
                       sub_64), DSForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4382-4390

```tablegen
def : Pat<(store (f64 (extractelt v2f64:$A, 1)), DSForm:$src),
          (DFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2),
                       sub_64), DSForm:$src)>;
def : Pat<(store (i64 (extractelt v2i64:$A, 0)), DSForm:$src),
          (DFSTOREf64 (EXTRACT_SUBREG $A, sub_64), DSForm:$src)>;
def : Pat<(store (f64 (extractelt v2f64:$A, 0)), DSForm:$src),
          (DFSTOREf64 (EXTRACT_SUBREG $A, sub_64), DSForm:$src)>;

// (Un)Signed DWord vector extract -> QP
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4391-4397

```tablegen
def : Pat<(f128 (sint_to_fp (i64 (extractelt v2i64:$src, 0)))),
          (f128 (XSCVSDQP (COPY_TO_REGCLASS $src, VFRC)))>;
def : Pat<(f128 (sint_to_fp (i64 (extractelt v2i64:$src, 1)))),
          (f128 (XSCVSDQP
                  (EXTRACT_SUBREG (XXPERMDI $src, $src, 3), sub_64)))>;
def : Pat<(f128 (uint_to_fp (i64 (extractelt v2i64:$src, 0)))),
          (f128 (XSCVUDQP (COPY_TO_REGCLASS $src, VFRC)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4398-4404

```tablegen
def : Pat<(f128 (uint_to_fp (i64 (extractelt v2i64:$src, 1)))),
          (f128 (XSCVUDQP
                  (EXTRACT_SUBREG (XXPERMDI $src, $src, 3), sub_64)))>;

// (Un)Signed Word vector extract -> QP
def : Pat<(f128 (sint_to_fp (i32 (extractelt v4i32:$src, 1)))),
          (f128 (XSCVSDQP (EXTRACT_SUBREG (VEXTSW2D $src), sub_64)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4405-4415

```tablegen
foreach Idx = [0,2,3] in {
  def : Pat<(f128 (sint_to_fp (i32 (extractelt v4i32:$src, Idx)))),
            (f128 (XSCVSDQP (EXTRACT_SUBREG
                            (VEXTSW2D (VSPLTW Idx, $src)), sub_64)))>;
}
foreach Idx = 0-3 in {
  def : Pat<(f128 (uint_to_fp (i32 (extractelt v4i32:$src, Idx)))),
            (f128 (XSCVUDQP (XXEXTRACTUW $src, !shl(Idx, 2))))>;
}

// (Un)Signed HWord vector extract -> QP/DP/SP
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 4416-4423

```tablegen
foreach Idx = 0-7 in {
  def : Pat<(f128 (sint_to_fp
                    (i32 (sext_inreg
                           (vector_extract v8i16:$src, Idx), i16)))),
          (f128 (XSCVSDQP (EXTRACT_SUBREG
                            (VEXTSH2D (VEXTRACTUH !add(Idx, Idx), $src)),
                            sub_64)))>;
  // The SDAG adds the `and` since an `i16` is being extracted as an `i32`.
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 4424-4432

```tablegen
  def : Pat<(f128 (uint_to_fp
                    (and (i32 (vector_extract v8i16:$src, Idx)), 65535))),
            (f128 (XSCVUDQP (EXTRACT_SUBREG
                              (VEXTRACTUH !add(Idx, Idx), $src), sub_64)))>;
  def : Pat<(f32 (PPCfcfidus
                   (f64 (PPCmtvsrz (and (i32 (vector_extract v8i16:$src, Idx)),
                                        65535))))),
            (f32 (XSCVUXDSP (EXTRACT_SUBREG
                              (VEXTRACTUH !add(Idx, Idx), $src), sub_64)))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4433-4439

```tablegen
  def : Pat<(f32 (PPCfcfids
                   (f64 (PPCmtvsra
                          (i32 (sext_inreg (vector_extract v8i16:$src, Idx),
                               i16)))))),
          (f32 (XSCVSXDSP (EXTRACT_SUBREG
                            (VEXTSH2D (VEXTRACTUH !add(Idx, Idx), $src)),
                            sub_64)))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4440-4455

```tablegen
  def : Pat<(f64 (PPCfcfidu
                   (f64 (PPCmtvsrz
                          (and (i32 (vector_extract v8i16:$src, Idx)),
                               65535))))),
            (f64 (XSCVUXDDP (EXTRACT_SUBREG
                              (VEXTRACTUH !add(Idx, Idx), $src), sub_64)))>;
  def : Pat<(f64 (PPCfcfid
                   (f64 (PPCmtvsra
                          (i32 (sext_inreg (vector_extract v8i16:$src, Idx),
                               i16)))))),
          (f64 (XSCVSXDDP (EXTRACT_SUBREG
                            (VEXTSH2D (VEXTRACTUH !add(Idx, Idx), $src)),
                            sub_64)))>;
}

// (Un)Signed Byte vector extract -> QP
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4456-4466

```tablegen
foreach Idx = 0-15 in {
  def : Pat<(f128 (sint_to_fp
                    (i32 (sext_inreg (vector_extract v16i8:$src, Idx),
                                     i8)))),
            (f128 (XSCVSDQP (EXTRACT_SUBREG
                              (VEXTSB2D (VEXTRACTUB Idx, $src)), sub_64)))>;
  def : Pat<(f128 (uint_to_fp
                    (and (i32 (vector_extract v16i8:$src, Idx)), 255))),
            (f128 (XSCVUDQP
                    (EXTRACT_SUBREG (VEXTRACTUB Idx, $src), sub_64)))>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 4467-4479

```tablegen
  def : Pat<(f32 (PPCfcfidus
                   (f64 (PPCmtvsrz
                          (and (i32 (vector_extract v16i8:$src, Idx)),
                               255))))),
            (f32 (XSCVUXDSP (EXTRACT_SUBREG
                              (VEXTRACTUB !add(Idx, Idx), $src), sub_64)))>;
  def : Pat<(f32 (PPCfcfids
                   (f64 (PPCmtvsra
                          (i32 (sext_inreg (vector_extract v16i8:$src, Idx),
                               i8)))))),
          (f32 (XSCVSXDSP (EXTRACT_SUBREG
                            (VEXTSH2D (VEXTRACTUB !add(Idx, Idx), $src)),
                            sub_64)))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4480-4495

```tablegen
  def : Pat<(f64 (PPCfcfidu
                   (f64 (PPCmtvsrz
                          (and (i32 (vector_extract v16i8:$src, Idx)),
                          255))))),
            (f64 (XSCVUXDDP (EXTRACT_SUBREG
                              (VEXTRACTUB !add(Idx, Idx), $src), sub_64)))>;
  def : Pat<(f64 (PPCfcfid
                   (f64 (PPCmtvsra
                          (i32 (sext_inreg (vector_extract v16i8:$src, Idx),
                               i8)))))),
          (f64 (XSCVSXDDP (EXTRACT_SUBREG
                            (VEXTSH2D (VEXTRACTUB !add(Idx, Idx), $src)),
                            sub_64)))>;
}

// Unsiged int in vsx register -> QP
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4496-4502

```tablegen
def : Pat<(f128 (uint_to_fp (i32 (PPCmfvsr f64:$src)))),
          (f128 (XSCVUDQP
                  (XXEXTRACTUW (SUBREG_TO_REG $src, sub_64), 4)))>;
} // HasVSX, HasP9Vector, IsBigEndian, IsPPC64

// Little endian Power9 subtarget.
let Predicates = [HasVSX, HasP9Vector, IsLittleEndian] in {
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4503-4510

```tablegen
def : Pat<(f32 (PPCfcfidus (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 0)))))),
          (f32 (XSCVUXDSP (XXEXTRACTUW $A, 12)))>;
def : Pat<(f32 (PPCfcfidus (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 1)))))),
          (f32 (XSCVUXDSP (XXEXTRACTUW $A, 8)))>;
def : Pat<(f32 (PPCfcfidus (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 2)))))),
          (f32 (XSCVUXDSP (XXEXTRACTUW $A, 4)))>;
def : Pat<(f32 (PPCfcfidus (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 3)))))),
          (f32 (XSCVUXDSP (XXEXTRACTUW $A, 0)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4511-4518

```tablegen
def : Pat<(f64 (PPCfcfidu (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 0)))))),
          (f64 (XSCVUXDDP (XXEXTRACTUW $A, 12)))>;
def : Pat<(f64 (PPCfcfidu (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 1)))))),
          (f64 (XSCVUXDDP (XXEXTRACTUW $A, 8)))>;
def : Pat<(f64 (PPCfcfidu (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 2)))))),
          (f64 (XSCVUXDDP (XXEXTRACTUW $A, 4)))>;
def : Pat<(f64 (PPCfcfidu (f64 (PPCmtvsrz (i32 (extractelt v4i32:$A, 3)))))),
          (f64 (XSCVUXDDP (XXEXTRACTUW $A, 0)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4519-4528

```tablegen
def : Pat<(v4i32 (insertelt v4i32:$A, i32:$B, 0)),
          (v4i32 (XXINSERTW v4i32:$A, AlignValues.I32_TO_BE_WORD1, 12))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToInt.B, 0)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPSXWS f64:$B), sub_64),
                            12))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToUInt.B, 0)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPUXWS f64:$B), sub_64),
                            12))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4529-4538

```tablegen
def : Pat<(v4i32 (insertelt v4i32:$A, i32:$B, 1)),
          (v4i32 (XXINSERTW v4i32:$A, AlignValues.I32_TO_BE_WORD1, 8))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToInt.B, 1)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPSXWS f64:$B), sub_64),
                            8))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToUInt.B, 1)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPUXWS f64:$B), sub_64),
                            8))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4539-4548

```tablegen
def : Pat<(v4i32 (insertelt v4i32:$A, i32:$B, 2)),
          (v4i32 (XXINSERTW v4i32:$A, AlignValues.I32_TO_BE_WORD1, 4))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToInt.B, 2)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPSXWS f64:$B), sub_64),
                            4))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToUInt.B, 2)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPUXWS f64:$B), sub_64),
                            4))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4549-4558

```tablegen
def : Pat<(v4i32 (insertelt v4i32:$A, i32:$B, 3)),
          (v4i32 (XXINSERTW v4i32:$A, AlignValues.I32_TO_BE_WORD1, 0))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToInt.B, 3)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPSXWS f64:$B), sub_64),
                            0))>;
def : Pat<(v4i32 (insertelt v4i32:$A, DblToUInt.B, 3)),
          (v4i32 (XXINSERTW v4i32:$A,
                            (SUBREG_TO_REG (XSCVDPUXWS f64:$B), sub_64),
                            0))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4559-4567

```tablegen
def : Pat<(v4f32 (insertelt v4f32:$A, f32:$B, 0)),
          (v4f32 (XXINSERTW v4f32:$A, AlignValues.F32_TO_BE_WORD1, 12))>;
def : Pat<(v4f32 (insertelt v4f32:$A, f32:$B, 1)),
          (v4f32 (XXINSERTW v4f32:$A, AlignValues.F32_TO_BE_WORD1, 8))>;
def : Pat<(v4f32 (insertelt v4f32:$A, f32:$B, 2)),
          (v4f32 (XXINSERTW v4f32:$A, AlignValues.F32_TO_BE_WORD1, 4))>;
def : Pat<(v4f32 (insertelt v4f32:$A, f32:$B, 3)),
          (v4f32 (XXINSERTW v4f32:$A, AlignValues.F32_TO_BE_WORD1, 0))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4568-4576

```tablegen
def : Pat<(v4f32 (insertelt v4f32:$A, (f32 (fpround f64:$B)), 0)),
          (v4f32 (XXINSERTW v4f32:$A,
                  (SUBREG_TO_REG (XSCVDPSP f64:$B), sub_64), 12))>;
def : Pat<(v4f32 (insertelt v4f32:$A, (f32 (fpround f64:$B)), 1)),
          (v4f32 (XXINSERTW v4f32:$A,
                  (SUBREG_TO_REG (XSCVDPSP f64:$B), sub_64), 8))>;
def : Pat<(v4f32 (insertelt v4f32:$A, (f32 (fpround f64:$B)), 2)),
          (v4f32 (XXINSERTW v4f32:$A,
                  (SUBREG_TO_REG (XSCVDPSP f64:$B), sub_64), 4))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4577-4585

```tablegen
def : Pat<(v4f32 (insertelt v4f32:$A, (f32 (fpround f64:$B)), 3)),
          (v4f32 (XXINSERTW v4f32:$A,
                  (SUBREG_TO_REG (XSCVDPSP f64:$B), sub_64), 0))>;

def : Pat<(v8i16 (PPCld_vec_be ForceXForm:$src)),
          (COPY_TO_REGCLASS (LXVH8X ForceXForm:$src), VRRC)>;
def : Pat<(PPCst_vec_be v8i16:$rS, ForceXForm:$dst),
          (STXVH8X (COPY_TO_REGCLASS $rS, VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4586-4593

```tablegen
def : Pat<(v16i8 (PPCld_vec_be ForceXForm:$src)),
          (COPY_TO_REGCLASS (LXVB16X ForceXForm:$src), VRRC)>;
def : Pat<(PPCst_vec_be v16i8:$rS, ForceXForm:$dst),
          (STXVB16X (COPY_TO_REGCLASS $rS, VSRC), ForceXForm:$dst)>;

// Scalar stores of i8
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 0)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 8)), VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4594-4601

```tablegen
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 1)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 7)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 2)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 6)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 3)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 5)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 4)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 4)), VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4602-4609

```tablegen
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 5)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 3)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 6)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 2)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 7)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 1)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 8)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS $S, VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4610-4617

```tablegen
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 9)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 15)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 10)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 14)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 11)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 13)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 12)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 12)), VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4618-4625

```tablegen
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 13)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 11)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 14)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 10)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei8 (i32 (vector_extract v16i8:$S, 15)), ForceXForm:$dst),
          (STXSIBXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 9)), VSRC), ForceXForm:$dst)>;

// Scalar stores of i16
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4626-4633

```tablegen
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 0)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 8)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 1)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 6)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 2)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 4)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 3)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 2)), VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4634-4642

```tablegen
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 4)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS $S, VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 5)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 14)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 6)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 12)), VSRC), ForceXForm:$dst)>;
def : Pat<(truncstorei16 (i32 (vector_extract v8i16:$S, 7)), ForceXForm:$dst),
          (STXSIHXv (COPY_TO_REGCLASS (v16i8 (VSLDOI $S, $S, 10)), VSRC), ForceXForm:$dst)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4643-4650

```tablegen
defm : ScalToVecWPermute<
  v2i64, (i64 (load DSForm:$src)),
  (XXPERMDIs (DFLOADf64 DSForm:$src), 2),
  (SUBREG_TO_REG (DFLOADf64 DSForm:$src), sub_64)>;
defm : ScalToVecWPermute<
  v2i64, (i64 (load XForm:$src)),
  (XXPERMDIs (XFLOADf64 XForm:$src), 2),
  (SUBREG_TO_REG (XFLOADf64 XForm:$src), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4651-4659

```tablegen
defm : ScalToVecWPermute<
  v2f64, (f64 (load DSForm:$src)),
  (XXPERMDIs (DFLOADf64 DSForm:$src), 2),
  (SUBREG_TO_REG (DFLOADf64 DSForm:$src), sub_64)>;
defm : ScalToVecWPermute<
  v2f64, (f64 (load XForm:$src)),
  (XXPERMDIs (XFLOADf64 XForm:$src), 2),
  (SUBREG_TO_REG (XFLOADf64 XForm:$src), sub_64)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4660-4667

```tablegen
def : Pat<(store (i64 (extractelt v2i64:$A, 0)), XForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2),
                       sub_64), XForm:$src)>;
def : Pat<(store (f64 (extractelt v2f64:$A, 0)), XForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2),
                       sub_64), XForm:$src)>;
def : Pat<(store (i64 (extractelt v2i64:$A, 1)), XForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG $A, sub_64), XForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4668-4675

```tablegen
def : Pat<(store (f64 (extractelt v2f64:$A, 1)), XForm:$src),
          (XFSTOREf64 (EXTRACT_SUBREG $A, sub_64), XForm:$src)>;
def : Pat<(store (i64 (extractelt v2i64:$A, 0)), DSForm:$src),
          (DFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2),
                       sub_64), DSForm:$src)>;
def : Pat<(store (f64 (extractelt v2f64:$A, 0)), DSForm:$src),
          (DFSTOREf64 (EXTRACT_SUBREG (XXPERMDI $A, $A, 2), sub_64),
                      DSForm:$src)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4676-4684

```tablegen
def : Pat<(store (i64 (extractelt v2i64:$A, 1)), DSForm:$src),
          (DFSTOREf64 (EXTRACT_SUBREG $A, sub_64), DSForm:$src)>;
def : Pat<(store (f64 (extractelt v2f64:$A, 1)), DSForm:$src),
          (DFSTOREf64 (EXTRACT_SUBREG $A, sub_64), DSForm:$src)>;

// (Un)Signed DWord vector extract -> QP
def : Pat<(f128 (sint_to_fp (i64 (extractelt v2i64:$src, 0)))),
          (f128 (XSCVSDQP
                  (EXTRACT_SUBREG (XXPERMDI $src, $src, 3), sub_64)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4685-4693

```tablegen
def : Pat<(f128 (sint_to_fp (i64 (extractelt v2i64:$src, 1)))),
          (f128 (XSCVSDQP (COPY_TO_REGCLASS $src, VFRC)))>;
def : Pat<(f128 (uint_to_fp (i64 (extractelt v2i64:$src, 0)))),
          (f128 (XSCVUDQP
                  (EXTRACT_SUBREG (XXPERMDI $src, $src, 3), sub_64)))>;
def : Pat<(f128 (uint_to_fp (i64 (extractelt v2i64:$src, 1)))),
          (f128 (XSCVUDQP (COPY_TO_REGCLASS $src, VFRC)))>;

// (Un)Signed Word vector extract -> QP
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4694-4702

```tablegen
foreach Idx = [[0,3],[1,2],[3,0]] in {
  def : Pat<(f128 (sint_to_fp (i32 (extractelt v4i32:$src, !head(Idx))))),
            (f128 (XSCVSDQP (EXTRACT_SUBREG
                              (VEXTSW2D (VSPLTW !head(!tail(Idx)), $src)),
                              sub_64)))>;
}
def : Pat<(f128 (sint_to_fp (i32 (extractelt v4i32:$src, 2)))),
          (f128 (XSCVSDQP (EXTRACT_SUBREG (VEXTSW2D $src), sub_64)))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4703-4710

```tablegen
foreach Idx = [[0,12],[1,8],[2,4],[3,0]] in {
  def : Pat<(f128 (uint_to_fp (i32 (extractelt v4i32:$src, !head(Idx))))),
            (f128 (XSCVUDQP (XXEXTRACTUW $src, !head(!tail(Idx)))))>;
}

// (Un)Signed HWord vector extract -> QP/DP/SP
// The Nested foreach lists identifies the vector element and corresponding
// register byte location.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "(Un)Signed HWord vector extract -> QP/DP/SP".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“(Un)Signed HWord vector extract -> QP/DP/SP”。

### Lines 4711-4718

```tablegen
foreach Idx = [[0,14],[1,12],[2,10],[3,8],[4,6],[5,4],[6,2],[7,0]] in {
  def : Pat<(f128 (sint_to_fp
                    (i32 (sext_inreg
                           (vector_extract v8i16:$src, !head(Idx)), i16)))),
            (f128 (XSCVSDQP
                    (EXTRACT_SUBREG (VEXTSH2D
                                      (VEXTRACTUH !head(!tail(Idx)), $src)),
                                    sub_64)))>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 4719-4729

```tablegen
  def : Pat<(f128 (uint_to_fp
                    (and (i32 (vector_extract v8i16:$src, !head(Idx))),
                         65535))),
            (f128 (XSCVUDQP (EXTRACT_SUBREG
                              (VEXTRACTUH !head(!tail(Idx)), $src), sub_64)))>;
  def : Pat<(f32 (PPCfcfidus
                   (f64 (PPCmtvsrz
                          (and (i32 (vector_extract v8i16:$src, !head(Idx))),
                          65535))))),
            (f32 (XSCVUXDSP (EXTRACT_SUBREG
                              (VEXTRACTUH !head(!tail(Idx)), $src), sub_64)))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4730-4737

```tablegen
  def : Pat<(f32 (PPCfcfids
                   (f64 (PPCmtvsra
                          (i32 (sext_inreg (vector_extract v8i16:$src,
                                           !head(Idx)), i16)))))),
            (f32 (XSCVSXDSP
                    (EXTRACT_SUBREG
                     (VEXTSH2D (VEXTRACTUH !head(!tail(Idx)), $src)),
                     sub_64)))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4738-4754

```tablegen
  def : Pat<(f64 (PPCfcfidu
                   (f64 (PPCmtvsrz
                          (and (i32 (vector_extract v8i16:$src, !head(Idx))),
                          65535))))),
            (f64 (XSCVUXDDP (EXTRACT_SUBREG
                              (VEXTRACTUH !head(!tail(Idx)), $src), sub_64)))>;
  def : Pat<(f64 (PPCfcfid
                   (f64 (PPCmtvsra
                        (i32 (sext_inreg
                            (vector_extract v8i16:$src, !head(Idx)), i16)))))),
            (f64 (XSCVSXDDP
                    (EXTRACT_SUBREG (VEXTSH2D
                                      (VEXTRACTUH !head(!tail(Idx)), $src)),
                                    sub_64)))>;
}

// (Un)Signed Byte vector extract -> QP/DP/SP
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4755-4763

```tablegen
foreach Idx = [[0,15],[1,14],[2,13],[3,12],[4,11],[5,10],[6,9],[7,8],[8,7],
               [9,6],[10,5],[11,4],[12,3],[13,2],[14,1],[15,0]] in {
  def : Pat<(f128 (sint_to_fp
                    (i32 (sext_inreg
                           (vector_extract v16i8:$src, !head(Idx)), i8)))),
            (f128 (XSCVSDQP
                    (EXTRACT_SUBREG
                      (VEXTSB2D (VEXTRACTUB !head(!tail(Idx)), $src)),
                      sub_64)))>;
```
- **EN**: Implements helper routine(s) for this portion of the PowerPC backend TableGen declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 TableGen 声明所需的辅助例程。

### Lines 4764-4770

```tablegen
  def : Pat<(f128 (uint_to_fp
                    (and (i32 (vector_extract v16i8:$src, !head(Idx))),
                         255))),
            (f128 (XSCVUDQP
                    (EXTRACT_SUBREG
                      (VEXTRACTUB !head(!tail(Idx)), $src), sub_64)))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4771-4784

```tablegen
  def : Pat<(f32 (PPCfcfidus
                   (f64 (PPCmtvsrz
                          (and (i32 (vector_extract v16i8:$src, !head(Idx))),
                          255))))),
            (f32 (XSCVUXDSP (EXTRACT_SUBREG
                              (VEXTRACTUB !head(!tail(Idx)), $src), sub_64)))>;
  def : Pat<(f32 (PPCfcfids
                   (f64 (PPCmtvsra
                          (i32 (sext_inreg
                            (vector_extract v16i8:$src, !head(Idx)), i8)))))),
            (f32 (XSCVSXDSP
                    (EXTRACT_SUBREG (VEXTSH2D
                                      (VEXTRACTUB !head(!tail(Idx)), $src)),
                                    sub_64)))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4785-4799

```tablegen
  def : Pat<(f64 (PPCfcfidu
                   (f64 (PPCmtvsrz
                          (and (i32
                            (vector_extract v16i8:$src, !head(Idx))), 255))))),
            (f64 (XSCVUXDDP (EXTRACT_SUBREG
                              (VEXTRACTUB !head(!tail(Idx)), $src), sub_64)))>;
  def : Pat<(f64 (PPCfcfidu
                   (f64 (PPCmtvsra
                        (i32 (sext_inreg
                            (vector_extract v16i8:$src, !head(Idx)), i8)))))),
            (f64 (XSCVSXDDP
                    (EXTRACT_SUBREG (VEXTSH2D
                                      (VEXTRACTUB !head(!tail(Idx)), $src)),
                                    sub_64)))>;
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4800-4810

```tablegen
  def : Pat<(f64 (PPCfcfid
                   (f64 (PPCmtvsra
                        (i32 (sext_inreg
                          (vector_extract v16i8:$src, !head(Idx)), i8)))))),
            (f64 (XSCVSXDDP
                    (EXTRACT_SUBREG (VEXTSH2D
                                      (VEXTRACTUH !head(!tail(Idx)), $src)),
                                    sub_64)))>;
}

// Unsiged int in vsx register -> QP
```
- **EN**: Declares function entry points that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口。

### Lines 4811-4818

```tablegen
def : Pat<(f128 (uint_to_fp (i32 (PPCmfvsr f64:$src)))),
          (f128 (XSCVUDQP
                  (XXEXTRACTUW (SUBREG_TO_REG $src, sub_64), 8)))>;
} // HasVSX, HasP9Vector, IsLittleEndian

// Any Power9 VSX subtarget that supports Power9 Altivec.
let Predicates = [HasVSX, HasP9Altivec] in {
// Unsigned absolute-difference.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Any Power9 VSX subtarget that supports Power9 Altivec.". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Any Power9 VSX subtarget that supports Power9 Altivec.”。 这些声明会进入生成式模式匹配逻辑。

### Lines 4819-4833

```tablegen
def : Pat<(v4i32 (abdu v4i32:$A, v4i32:$B)),
          (v4i32 (VABSDUW $A, $B))>;

def : Pat<(v8i16 (abdu v8i16:$A, v8i16:$B)),
          (v8i16 (VABSDUH $A, $B))>;

def : Pat<(v16i8 (abdu v16i8:$A, v16i8:$B)),
          (v16i8 (VABSDUB $A, $B))>;

// Signed absolute-difference.
// Power9 VABSD* instructions are designed to support unsigned integer
// vectors (byte/halfword/word), if we want to make use of them for signed
// integer vectors, we have to flip their sign bits first. To flip sign bit
// for byte/halfword integer vector would become inefficient, but for word
// integer vector, we can leverage XVNEGSP to make it efficiently.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Signed absolute-difference.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Signed absolute-difference.”。

### Lines 4834-4842

```tablegen
def : Pat<(v4i32 (abds v4i32:$A, v4i32:$B)),
          (v4i32 (VABSDUW (XVNEGSP $A), (XVNEGSP $B)))>;
} // HasVSX, HasP9Altivec

// Big endian Power9 64Bit VSX subtargets with P9 Altivec support.
let Predicates = [HasVSX, HasP9Altivec, IsBigEndian, IsPPC64] in {
def : Pat<(i64 (anyext (i32 (vector_extract v16i8:$S, i64:$Idx)))),
          (VEXTUBLX $Idx, $S)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4843-4850

```tablegen
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, i64:$Idx)))),
          (VEXTUHLX (RLWINM8 $Idx, 1, 28, 30), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 0)))),
          (VEXTUHLX (LI8 0), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 1)))),
          (VEXTUHLX (LI8 2), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 2)))),
          (VEXTUHLX (LI8 4), $S)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4851-4858

```tablegen
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 3)))),
          (VEXTUHLX (LI8 6), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 4)))),
          (VEXTUHLX (LI8 8), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 5)))),
          (VEXTUHLX (LI8 10), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 6)))),
          (VEXTUHLX (LI8 12), $S)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4859-4867

```tablegen
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 7)))),
          (VEXTUHLX (LI8 14), $S)>;

def : Pat<(i64 (zext (i32 (vector_extract v4i32:$S, i64:$Idx)))),
          (VEXTUWLX (RLWINM8 $Idx, 2, 28, 29), $S)>;
def : Pat<(i64 (zext (i32 (vector_extract v4i32:$S, 0)))),
          (VEXTUWLX (LI8 0), $S)>;

// For extracting BE word 1, MFVSRWZ is better than VEXTUWLX
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4868-4875

```tablegen
def : Pat<(i64 (zext (i32 (vector_extract v4i32:$S, 1)))),
          (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
          (i32 VectorExtractions.LE_WORD_2), sub_32)>;
def : Pat<(i64 (zext (i32 (vector_extract v4i32:$S, 2)))),
          (VEXTUWLX (LI8 8), $S)>;
def : Pat<(i64 (zext (i32 (vector_extract v4i32:$S, 3)))),
          (VEXTUWLX (LI8 12), $S)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4876-4883

```tablegen
def : Pat<(i64 (sext (i32 (vector_extract v4i32:$S, i64:$Idx)))),
          (EXTSW (VEXTUWLX (RLWINM8 $Idx, 2, 28, 29), $S))>;
def : Pat<(i64 (sext (i32 (vector_extract v4i32:$S, 0)))),
          (EXTSW (VEXTUWLX (LI8 0), $S))>;
// For extracting BE word 1, MFVSRWZ is better than VEXTUWLX
def : Pat<(i64 (sext (i32 (vector_extract v4i32:$S, 1)))),
          (EXTSW (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
          (i32 VectorExtractions.LE_WORD_2), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4884-4890

```tablegen
def : Pat<(i64 (sext (i32 (vector_extract v4i32:$S, 2)))),
          (EXTSW (VEXTUWLX (LI8 8), $S))>;
def : Pat<(i64 (sext (i32 (vector_extract v4i32:$S, 3)))),
          (EXTSW (VEXTUWLX (LI8 12), $S))>;

def : Pat<(i32 (vector_extract v16i8:$S, i64:$Idx)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX $Idx, $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4891-4898

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 0)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 0), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 1)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 1), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 2)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 2), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 3)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 3), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4899-4906

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 4)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 4), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 5)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 5), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 6)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 6), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 7)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 7), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4907-4914

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 8)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 8), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 9)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 9), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 10)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 10), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 11)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 11), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4915-4923

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 12)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 12), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 13)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 13), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 14)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 14), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 15)),
          (i32 (EXTRACT_SUBREG (VEXTUBLX (LI8 15), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4924-4930

```tablegen
def : Pat<(i32 (vector_extract v8i16:$S, i64:$Idx)),
          (i32 (EXTRACT_SUBREG (VEXTUHLX
          (RLWINM8 $Idx, 1, 28, 30), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 0)),
          (i32 (EXTRACT_SUBREG (VEXTUHLX (LI8 0), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 1)),
          (i32 (EXTRACT_SUBREG (VEXTUHLX (LI8 2), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4931-4938

```tablegen
def : Pat<(i32 (vector_extract v8i16:$S, 2)),
          (i32 (EXTRACT_SUBREG (VEXTUHLX (LI8 4), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 3)),
          (i32 (EXTRACT_SUBREG (VEXTUHLX (LI8 6), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 4)),
          (i32 (EXTRACT_SUBREG (VEXTUHLX (LI8 8), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 5)),
          (i32 (EXTRACT_SUBREG (VEXTUHLX (LI8 10), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4939-4946

```tablegen
def : Pat<(i32 (vector_extract v8i16:$S, 6)),
          (i32 (EXTRACT_SUBREG (VEXTUHLX (LI8 12), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 6)),
          (i32 (EXTRACT_SUBREG (VEXTUHLX (LI8 14), $S), sub_32))>;

def : Pat<(i32 (vector_extract v4i32:$S, i64:$Idx)),
          (i32 (EXTRACT_SUBREG (VEXTUWLX
          (RLWINM8 $Idx, 2, 28, 29), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4947-4953

```tablegen
def : Pat<(i32 (vector_extract v4i32:$S, 0)),
          (i32 (EXTRACT_SUBREG (VEXTUWLX (LI8 0), $S), sub_32))>;
// For extracting BE word 1, MFVSRWZ is better than VEXTUWLX
def : Pat<(i32 (vector_extract v4i32:$S, 1)),
          (i32 VectorExtractions.LE_WORD_2)>;
def : Pat<(i32 (vector_extract v4i32:$S, 2)),
          (i32 (EXTRACT_SUBREG (VEXTUWLX (LI8 8), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4954-4961

```tablegen
def : Pat<(i32 (vector_extract v4i32:$S, 3)),
          (i32 (EXTRACT_SUBREG (VEXTUWLX (LI8 12), $S), sub_32))>;

// P9 Altivec instructions that can be used to build vectors.
// Adding them to PPCInstrVSX.td rather than PPCAltivecVSX.td to compete
// with complexities of existing build vector patterns in this file.
def : Pat<(v2i64 (build_vector WordToDWord.BE_A0, WordToDWord.BE_A1)),
          (v2i64 (VEXTSW2D $A))>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "P9 Altivec instructions that can be used to build vectors.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“P9 Altivec instructions that can be used to build vectors.”。

### Lines 4962-4969

```tablegen
def : Pat<(v2i64 (build_vector HWordToDWord.BE_A0, HWordToDWord.BE_A1)),
          (v2i64 (VEXTSH2D $A))>;
def : Pat<(v4i32 (build_vector HWordToWord.BE_A0, HWordToWord.BE_A1,
                  HWordToWord.BE_A2, HWordToWord.BE_A3)),
          (v4i32 (VEXTSH2W $A))>;
def : Pat<(v4i32 (build_vector ByteToWord.BE_A0, ByteToWord.BE_A1,
                  ByteToWord.BE_A2, ByteToWord.BE_A3)),
          (v4i32 (VEXTSB2W $A))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4970-4978

```tablegen
def : Pat<(v2i64 (build_vector ByteToDWord.BE_A0, ByteToDWord.BE_A1)),
          (v2i64 (VEXTSB2D $A))>;
} // HasVSX, HasP9Altivec, IsBigEndian, IsPPC64

// Little endian Power9 VSX subtargets with P9 Altivec support.
let Predicates = [HasVSX, HasP9Altivec, IsLittleEndian] in {
def : Pat<(i64 (anyext (i32 (vector_extract v16i8:$S, i64:$Idx)))),
          (VEXTUBRX $Idx, $S)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 4979-4986

```tablegen
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, i64:$Idx)))),
          (VEXTUHRX (RLWINM8 $Idx, 1, 28, 30), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 0)))),
          (VEXTUHRX (LI8 0), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 1)))),
          (VEXTUHRX (LI8 2), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 2)))),
          (VEXTUHRX (LI8 4), $S)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4987-4994

```tablegen
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 3)))),
          (VEXTUHRX (LI8 6), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 4)))),
          (VEXTUHRX (LI8 8), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 5)))),
          (VEXTUHRX (LI8 10), $S)>;
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 6)))),
          (VEXTUHRX (LI8 12), $S)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 4995-5001

```tablegen
def : Pat<(i64 (anyext (i32 (vector_extract v8i16:$S, 7)))),
          (VEXTUHRX (LI8 14), $S)>;

def : Pat<(i64 (zext (i32 (vector_extract v4i32:$S, i64:$Idx)))),
          (VEXTUWRX (RLWINM8 $Idx, 2, 28, 29), $S)>;
def : Pat<(i64 (zext (i32 (vector_extract v4i32:$S, 0)))),
          (VEXTUWRX (LI8 0), $S)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5002-5010

```tablegen
def : Pat<(i64 (zext (i32 (vector_extract v4i32:$S, 1)))),
          (VEXTUWRX (LI8 4), $S)>;
// For extracting LE word 2, MFVSRWZ is better than VEXTUWRX
def : Pat<(i64 (zext (i32 (vector_extract v4i32:$S, 2)))),
          (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
          (i32 VectorExtractions.LE_WORD_2), sub_32)>;
def : Pat<(i64 (zext (i32 (vector_extract v4i32:$S, 3)))),
          (VEXTUWRX (LI8 12), $S)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5011-5017

```tablegen
def : Pat<(i64 (sext (i32 (vector_extract v4i32:$S, i64:$Idx)))),
          (EXTSW (VEXTUWRX (RLWINM8 $Idx, 2, 28, 29), $S))>;
def : Pat<(i64 (sext (i32 (vector_extract v4i32:$S, 0)))),
          (EXTSW (VEXTUWRX (LI8 0), $S))>;
def : Pat<(i64 (sext (i32 (vector_extract v4i32:$S, 1)))),
          (EXTSW (VEXTUWRX (LI8 4), $S))>;
// For extracting LE word 2, MFVSRWZ is better than VEXTUWRX
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5018-5025

```tablegen
def : Pat<(i64 (sext (i32 (vector_extract v4i32:$S, 2)))),
          (EXTSW (INSERT_SUBREG (i64 (IMPLICIT_DEF)),
          (i32 VectorExtractions.LE_WORD_2), sub_32))>;
def : Pat<(i64 (sext (i32 (vector_extract v4i32:$S, 3)))),
          (EXTSW (VEXTUWRX (LI8 12), $S))>;

def : Pat<(i32 (vector_extract v16i8:$S, i64:$Idx)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX $Idx, $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5026-5033

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 0)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 0), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 1)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 1), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 2)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 2), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 3)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 3), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5034-5041

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 4)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 4), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 5)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 5), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 6)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 6), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 7)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 7), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5042-5049

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 8)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 8), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 9)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 9), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 10)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 10), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 11)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 11), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5050-5058

```tablegen
def : Pat<(i32 (vector_extract v16i8:$S, 12)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 12), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 13)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 13), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 14)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 14), $S), sub_32))>;
def : Pat<(i32 (vector_extract v16i8:$S, 15)),
          (i32 (EXTRACT_SUBREG (VEXTUBRX (LI8 15), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5059-5065

```tablegen
def : Pat<(i32 (vector_extract v8i16:$S, i64:$Idx)),
          (i32 (EXTRACT_SUBREG (VEXTUHRX
          (RLWINM8 $Idx, 1, 28, 30), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 0)),
          (i32 (EXTRACT_SUBREG (VEXTUHRX (LI8 0), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 1)),
          (i32 (EXTRACT_SUBREG (VEXTUHRX (LI8 2), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5066-5073

```tablegen
def : Pat<(i32 (vector_extract v8i16:$S, 2)),
          (i32 (EXTRACT_SUBREG (VEXTUHRX (LI8 4), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 3)),
          (i32 (EXTRACT_SUBREG (VEXTUHRX (LI8 6), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 4)),
          (i32 (EXTRACT_SUBREG (VEXTUHRX (LI8 8), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 5)),
          (i32 (EXTRACT_SUBREG (VEXTUHRX (LI8 10), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5074-5081

```tablegen
def : Pat<(i32 (vector_extract v8i16:$S, 6)),
          (i32 (EXTRACT_SUBREG (VEXTUHRX (LI8 12), $S), sub_32))>;
def : Pat<(i32 (vector_extract v8i16:$S, 6)),
          (i32 (EXTRACT_SUBREG (VEXTUHRX (LI8 14), $S), sub_32))>;

def : Pat<(i32 (vector_extract v4i32:$S, i64:$Idx)),
          (i32 (EXTRACT_SUBREG (VEXTUWRX
          (RLWINM8 $Idx, 2, 28, 29), $S), sub_32))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5082-5088

```tablegen
def : Pat<(i32 (vector_extract v4i32:$S, 0)),
          (i32 (EXTRACT_SUBREG (VEXTUWRX (LI8 0), $S), sub_32))>;
def : Pat<(i32 (vector_extract v4i32:$S, 1)),
          (i32 (EXTRACT_SUBREG (VEXTUWRX (LI8 4), $S), sub_32))>;
// For extracting LE word 2, MFVSRWZ is better than VEXTUWRX
def : Pat<(i32 (vector_extract v4i32:$S, 2)),
          (i32 VectorExtractions.LE_WORD_2)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5089-5096

```tablegen
def : Pat<(i32 (vector_extract v4i32:$S, 3)),
          (i32 (EXTRACT_SUBREG (VEXTUWRX (LI8 12), $S), sub_32))>;

// P9 Altivec instructions that can be used to build vectors.
// Adding them to PPCInstrVSX.td rather than PPCAltivecVSX.td to compete
// with complexities of existing build vector patterns in this file.
def : Pat<(v2i64 (build_vector WordToDWord.LE_A0, WordToDWord.LE_A1)),
          (v2i64 (VEXTSW2D $A))>;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "P9 Altivec instructions that can be used to build vectors.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“P9 Altivec instructions that can be used to build vectors.”。

### Lines 5097-5104

```tablegen
def : Pat<(v2i64 (build_vector HWordToDWord.LE_A0, HWordToDWord.LE_A1)),
          (v2i64 (VEXTSH2D $A))>;
def : Pat<(v4i32 (build_vector HWordToWord.LE_A0, HWordToWord.LE_A1,
                  HWordToWord.LE_A2, HWordToWord.LE_A3)),
          (v4i32 (VEXTSH2W $A))>;
def : Pat<(v4i32 (build_vector ByteToWord.LE_A0, ByteToWord.LE_A1,
                  ByteToWord.LE_A2, ByteToWord.LE_A3)),
          (v4i32 (VEXTSB2W $A))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5105-5111

```tablegen
def : Pat<(v2i64 (build_vector ByteToDWord.LE_A0, ByteToDWord.LE_A1)),
          (v2i64 (VEXTSB2D $A))>;
} // HasVSX, HasP9Altivec, IsLittleEndian

// Big endian 64Bit VSX subtarget that supports additional direct moves from
// ISA3.0.
let Predicates = [HasVSX, IsISA3_0, HasDirectMove, IsBigEndian, IsPPC64] in {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Big endian 64Bit VSX subtarget that supports additional direct moves from". These declarations feed generated pattern-matching logic.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Big endian 64Bit VSX subtarget that supports additional direct moves from”。 这些声明会进入生成式模式匹配逻辑。

### Lines 5112-5121

```tablegen
def : Pat<(i64 (extractelt v2i64:$A, 1)),
          (i64 (MFVSRLD $A))>;
// Better way to build integer vectors if we have MTVSRDD. Big endian.
def : Pat<(v2i64 (build_vector i64:$rB, i64:$rA)),
          (v2i64 (MTVSRDD $rB, $rA))>;
def : Pat<(v4i32 (build_vector i32:$A, i32:$B, i32:$C, i32:$D)),
          (MTVSRDD
            (RLDIMI AnyExts.B, AnyExts.A, 32, 0),
            (RLDIMI AnyExts.D, AnyExts.C, 32, 0))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5122-5130

```tablegen
def : Pat<(f128 (PPCbuild_fp128 i64:$rB, i64:$rA)),
          (f128 (COPY_TO_REGCLASS (MTVSRDD $rB, $rA), VRRC))>;
} // HasVSX, IsISA3_0, HasDirectMove, IsBigEndian, IsPPC64

// Little endian VSX subtarget that supports direct moves from ISA3.0.
let Predicates = [HasVSX, IsISA3_0, HasDirectMove, IsLittleEndian] in {
def : Pat<(i64 (extractelt v2i64:$A, 0)),
          (i64 (MFVSRLD $A))>;
// Better way to build integer vectors if we have MTVSRDD. Little endian.
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 5131-5137

```tablegen
def : Pat<(v2i64 (build_vector i64:$rA, i64:$rB)),
          (v2i64 (MTVSRDD $rB, $rA))>;
def : Pat<(v4i32 (build_vector i32:$A, i32:$B, i32:$C, i32:$D)),
          (MTVSRDD
            (RLDIMI AnyExts.C, AnyExts.D, 32, 0),
            (RLDIMI AnyExts.A, AnyExts.B, 32, 0))>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5138-5145

```tablegen
def : Pat<(f128 (PPCbuild_fp128 i64:$rA, i64:$rB)),
          (f128 (COPY_TO_REGCLASS (MTVSRDD $rB, $rA), VRRC))>;
} // HasVSX, IsISA3_0, HasDirectMove, IsLittleEndian
} // AddedComplexity = 400

//---------------------------- Instruction aliases ---------------------------//
def : InstAlias<"xvmovdp $XT, $XB",
                (XVCPSGNDP vsrc:$XT, vsrc:$XB, vsrc:$XB)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5146-5152

```tablegen
def : InstAlias<"xvmovsp $XT, $XB",
                (XVCPSGNSP vsrc:$XT, vsrc:$XB, vsrc:$XB)>;

// Certain versions of the AIX assembler may missassemble these mnemonics.
let Predicates = [ModernAs] in {
  def : InstAlias<"xxspltd $XT, $XB, 0",
                  (XXPERMDI vsrc:$XT, vsrc:$XB, vsrc:$XB, 0)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code. These declarations feed generated pattern-matching logic.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。 这些声明会进入生成式模式匹配逻辑。

### Lines 5153-5160

```tablegen
  def : InstAlias<"xxspltd $XT, $XB, 1",
                  (XXPERMDI vsrc:$XT, vsrc:$XB, vsrc:$XB, 3)>;
  def : InstAlias<"xxspltd $XT, $XB, 0",
                  (XXPERMDIs vsrc:$XT, vsfrc:$XB, 0)>;
  def : InstAlias<"xxspltd $XT, $XB, 1",
                  (XXPERMDIs vsrc:$XT, vsfrc:$XB, 3)>;
}
```
- **EN**: Continues the PowerPC backend TableGen declarations for the backend with supporting declarations, helpers, or implementation details for this line range.
- **CN**: 这一段继续展开 PowerPC 后端的该后端的 TableGen 声明，补充该行区间所需的声明、辅助逻辑或实现细节。

### Lines 5161-5168

```tablegen
def : InstAlias<"xxmrghd $XT, $XA, $XB",
                (XXPERMDI vsrc:$XT, vsrc:$XA, vsrc:$XB, 0)>;
def : InstAlias<"xxmrgld $XT, $XA, $XB",
                (XXPERMDI vsrc:$XT, vsrc:$XA, vsrc:$XB, 3)>;
def : InstAlias<"xxswapd $XT, $XB",
                (XXPERMDI vsrc:$XT, vsrc:$XB, vsrc:$XB, 2)>;
def : InstAlias<"xxswapd $XT, $XB",
                (XXPERMDIs vsrc:$XT, vsfrc:$XB, 2)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5169-5176

```tablegen
def : InstAlias<"mfvrd $rA, $XT",
                (MFVRD g8rc:$rA, vrrc:$XT), 0>;
def : InstAlias<"mffprd $rA, $src",
                (MFVSRD g8rc:$rA, f8rc:$src)>;
def : InstAlias<"mtvrd $XT, $rA",
                (MTVRD vrrc:$XT, g8rc:$rA), 0>;
def : InstAlias<"mtfprd $dst, $rA",
                (MTVSRD f8rc:$dst, g8rc:$rA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5177-5184

```tablegen
def : InstAlias<"mfvrwz $rA, $XT",
                (MFVRWZ gprc:$rA, vrrc:$XT), 0>;
def : InstAlias<"mffprwz $rA, $src",
                (MFVSRWZ gprc:$rA, f8rc:$src)>;
def : InstAlias<"mtvrwa $XT, $rA",
                (MTVRWA vrrc:$XT, gprc:$rA), 0>;
def : InstAlias<"mtfprwa $dst, $rA",
                (MTVSRWA f8rc:$dst, gprc:$rA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

### Lines 5185-5188

```tablegen
def : InstAlias<"mtvrwz $XT, $rA",
                (MTVRWZ vrrc:$XT, gprc:$rA), 0>;
def : InstAlias<"mtfprwz $dst, $rA",
                (MTVSRWZ f8rc:$dst, gprc:$rA)>;
```
- **EN**: Adds declarative TableGen records that LLVM later expands into generated tables or helper code.
- **CN**: 这里补充声明式的 TableGen 记录，LLVM 随后会把它们展开成生成表或辅助代码。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- TableGen declarations for the backend / 该后端的 TableGen 声明
- LLVM TableGen DSL / LLVM TableGen 领域特定语言
- Register modeling / 寄存器建模
- Instruction selection or opcode handling / 指令选择或操作码处理
- Declarative TableGen records / 声明式 TableGen 记录
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- None explicitly included in the source text / 源码文本中没有显式包含项

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- Generated backend tables / 生成式后端表
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
