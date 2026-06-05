# XCoreInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreInstrInfo.td`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Defines instruction records, encodings, and SelectionDAG patterns in TableGen DSL for this backend.
  - **CN**: 使用 TableGen DSL 定义该后端的指令记录、编码和 SelectionDAG 模式。
- **Note / 说明**:
  - **EN**: This file uses LLVM TableGen DSL, so many definitions are declarative records rather than executable C++ statements.
  - **CN**: 该文件使用 LLVM TableGen DSL，因此许多内容是声明式记录，而不是可执行的 C++ 语句。

## Line-by-Line Analysis / 逐行分析

### Lines 1-17
```tablegen
//===-- XCoreInstrInfo.td - Target Description for XCore ---*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the XCore instructions in TableGen format.
//
//===----------------------------------------------------------------------===//

// Uses of CP, DP are not currently reflected in the patterns, since
// having a physical register as an operand prevents loop hoisting and
// since the value of these registers never changes during the life of the
// function.

```
- **EN**: Contains the standard LLVM file banner, license notice, and high-level file description.
- **CN**: 包含 LLVM 标准文件头、许可证声明以及该文件的高层说明。

### Lines 18-33
```tablegen
//===----------------------------------------------------------------------===//
// Instruction format superclass.
//===----------------------------------------------------------------------===//

include "XCoreInstrFormats.td"

//===----------------------------------------------------------------------===//
// XCore specific DAG Nodes.
//

// Call
def SDT_XCoreBranchLink : SDTypeProfile<0, 1, [SDTCisPtrTy<0>]>;
def XCoreBranchLink     : SDNode<"XCoreISD::BL",SDT_XCoreBranchLink,
                            [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                             SDNPVariadic]>;

```
- **EN**: Imports shared TableGen building blocks such as `XCoreInstrFormats.td` so this file can reuse operand, format, or helper definitions.
- **CN**: 导入共享的 TableGen 构件，例如 `XCoreInstrFormats.td`，以便复用操作数、格式或辅助定义。

### Lines 34-50
```tablegen
def XCoreRetsp : SDNode<"XCoreISD::RETSP", SDTBrind,
                      [SDNPHasChain, SDNPOptInGlue, SDNPMayLoad, SDNPVariadic]>;

def SDT_XCoreEhRet : SDTypeProfile<0, 2,
                            [SDTCisSameAs<0, 1>, SDTCisPtrTy<0>]>;
def XCoreEhRet       : SDNode<"XCoreISD::EH_RETURN", SDT_XCoreEhRet,
                         [SDNPHasChain, SDNPOptInGlue]>;

def SDT_XCoreBR_JT    : SDTypeProfile<0, 2,
                                      [SDTCisVT<0, i32>, SDTCisVT<1, i32>]>;

def XCoreBR_JT : SDNode<"XCoreISD::BR_JT", SDT_XCoreBR_JT,
                        [SDNPHasChain]>;

def XCoreBR_JT32 : SDNode<"XCoreISD::BR_JT32", SDT_XCoreBR_JT,
                        [SDNPHasChain]>;

```
- **EN**: Declares TableGen records such as `XCoreRetsp`, `SDT_XCoreEhRet`, `XCoreEhRet`, `SDT_XCoreBR_JT`, ... for the backend description.
- **CN**: 为后端描述声明了 `XCoreRetsp`, `SDT_XCoreEhRet`, `XCoreEhRet`, `SDT_XCoreBR_JT`, ... 等 TableGen 记录。

### Lines 51-69
```tablegen
def SDT_XCoreAddress    : SDTypeProfile<1, 1,
                            [SDTCisSameAs<0, 1>, SDTCisPtrTy<0>]>;

def pcrelwrapper : SDNode<"XCoreISD::PCRelativeWrapper", SDT_XCoreAddress,
                           []>;

def dprelwrapper : SDNode<"XCoreISD::DPRelativeWrapper", SDT_XCoreAddress,
                           []>;

def cprelwrapper : SDNode<"XCoreISD::CPRelativeWrapper", SDT_XCoreAddress,
                           []>;

def frametoargsoffset : SDNode<"XCoreISD::FRAME_TO_ARGS_OFFSET", SDTIntLeaf,
                               []>;

def SDT_XCoreStwsp    : SDTypeProfile<0, 2, [SDTCisInt<1>]>;
def XCoreStwsp        : SDNode<"XCoreISD::STWSP", SDT_XCoreStwsp,
                               [SDNPHasChain, SDNPMayStore]>;

```
- **EN**: Declares TableGen records such as `SDT_XCoreAddress`, `pcrelwrapper`, `dprelwrapper`, `cprelwrapper`, ...; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `SDT_XCoreAddress`, `pcrelwrapper`, `dprelwrapper`, `cprelwrapper`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 70-94
```tablegen
def SDT_XCoreLdwsp    : SDTypeProfile<1, 1, [SDTCisInt<1>]>;
def XCoreLdwsp        : SDNode<"XCoreISD::LDWSP", SDT_XCoreLdwsp,
                               [SDNPHasChain, SDNPMayLoad]>;

def SDT_XCoreLAddSub : SDTypeProfile<2, 3, [
  SDTCisVT<0, i32>, // result
  SDTCisVT<1, i32>, // carry out
  SDTCisVT<2, i32>, // lhs
  SDTCisVT<3, i32>, // rhs
  SDTCisVT<4, i32>  // carry in
]>;

def XCoreLAdd : SDNode<"XCoreISD::LADD", SDT_XCoreLAddSub>;
def XCoreLSub : SDNode<"XCoreISD::LSUB", SDT_XCoreLAddSub>;

// Used for both long multiplication and multiply-accumulate.
def SDT_XCoreMul : SDTypeProfile<2, 4, [
  SDTCisVT<0, i32>, // result (high part)
  SDTCisVT<1, i32>, // result (low part)
  SDTCisVT<2, i32>, // lhs
  SDTCisVT<3, i32>, // rhs
  SDTCisVT<4, i32>, // addend 1
  SDTCisVT<5, i32>, // addend 2
]>;

```
- **EN**: Declares TableGen records such as `SDT_XCoreLdwsp`, `XCoreLdwsp`, `SDT_XCoreLAddSub`, `XCoreLAdd`, ... for the backend description.
- **CN**: 为后端描述声明了 `SDT_XCoreLdwsp`, `XCoreLdwsp`, `SDT_XCoreLAddSub`, `XCoreLAdd`, ... 等 TableGen 记录。

### Lines 95-114
```tablegen
def XCoreLMul : SDNode<"XCoreISD::LMUL", SDT_XCoreMul>;
def XCoreMAccU : SDNode<"XCoreISD::MACCU", SDT_XCoreMul>;
def XCoreMAccS : SDNode<"XCoreISD::MACCS", SDT_XCoreMul>;

def XCoreCRC8 : SDNode<"XCoreISD::CRC8",
  SDTypeProfile<2, 3, [
    SDTCisVT<0, i32>, // shifted data
    SDTCisVT<1, i32>, // result crc
    SDTCisVT<2, i32>, // initial crc
    SDTCisVT<3, i32>, // data
    SDTCisVT<4, i32>, // polynomial
  ]>
>;

// These are target-independent nodes, but have target-specific formats.
def SDT_XCoreCallSeqStart : SDCallSeqStart<[ SDTCisVT<0, i32>,
                                             SDTCisVT<1, i32> ]>;
def SDT_XCoreCallSeqEnd   : SDCallSeqEnd<[ SDTCisVT<0, i32>,
                                           SDTCisVT<1, i32> ]>;

```
- **EN**: Declares TableGen records such as `XCoreLMul`, `XCoreMAccU`, `XCoreMAccS`, `XCoreCRC8`, ... for the backend description.
- **CN**: 为后端描述声明了 `XCoreLMul`, `XCoreMAccU`, `XCoreMAccS`, `XCoreCRC8`, ... 等 TableGen 记录。

### Lines 115-137
```tablegen
def callseq_start : SDNode<"ISD::CALLSEQ_START", SDT_XCoreCallSeqStart,
                           [SDNPHasChain, SDNPOutGlue]>;
def callseq_end   : SDNode<"ISD::CALLSEQ_END",   SDT_XCoreCallSeqEnd,
                           [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;

//===----------------------------------------------------------------------===//
// Instruction Pattern Stuff
//===----------------------------------------------------------------------===//

def div4_xform : SDNodeXForm<imm, [{
  // Transformation function: imm/4
  assert(N->getZExtValue() % 4 == 0);
  return getI32Imm(N->getZExtValue()/4, SDLoc(N));
}]>;

def msksize_xform : SDNodeXForm<imm, [{
  // Transformation function: get the size of a mask
  assert(isMask_32(N->getZExtValue()));
  // look for the first non-zero bit
  return getI32Imm(llvm::bit_width((uint32_t)N->getZExtValue()),
                   SDLoc(N));
}]>;

```
- **EN**: Declares TableGen records such as `callseq_start`, `callseq_end`, `div4_xform`, `msksize_xform`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `callseq_start`, `callseq_end`, `div4_xform`, `msksize_xform` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 138-156
```tablegen
def neg_xform : SDNodeXForm<imm, [{
  // Transformation function: -imm
  uint32_t value = N->getZExtValue();
  return getI32Imm(-value, SDLoc(N));
}]>;

def bpwsub_xform : SDNodeXForm<imm, [{
  // Transformation function: 32-imm
  uint32_t value = N->getZExtValue();
  return getI32Imm(32 - value, SDLoc(N));
}]>;

def div4neg_xform : SDNodeXForm<imm, [{
  // Transformation function: -imm/4
  uint32_t value = N->getZExtValue();
  assert(-value % 4 == 0);
  return getI32Imm(-value/4, SDLoc(N));
}]>;

```
- **EN**: Declares TableGen records such as `neg_xform`, `bpwsub_xform`, `div4neg_xform` for the backend description.
- **CN**: 为后端描述声明了 `neg_xform`, `bpwsub_xform`, `div4neg_xform` 等 TableGen 记录。

### Lines 157-174
```tablegen
def immUs4Neg : PatLeaf<(imm), [{
  uint32_t value = (uint32_t)N->getZExtValue();
  return (-value)%4 == 0 && (-value)/4 <= 11;
}]>;

def immUs4 : PatLeaf<(imm), [{
  uint32_t value = (uint32_t)N->getZExtValue();
  return value%4 == 0 && value/4 <= 11;
}]>;

def immUsNeg : PatLeaf<(imm), [{
  return -((uint32_t)N->getZExtValue()) <= 11;
}]>;

def immUs : PatLeaf<(imm), [{
  return (uint32_t)N->getZExtValue() <= 11;
}]>;

```
- **EN**: Declares TableGen records such as `immUs4Neg`, `immUs4`, `immUsNeg`, `immUs` for the backend description.
- **CN**: 为后端描述声明了 `immUs4Neg`, `immUs4`, `immUsNeg`, `immUs` 等 TableGen 记录。

### Lines 175-192
```tablegen
def immU6 : PatLeaf<(imm), [{
  return (uint32_t)N->getZExtValue() < (1 << 6);
}]>;

def immU16 : PatLeaf<(imm), [{
  return (uint32_t)N->getZExtValue() < (1 << 16);
}]>;

def immMskBitp : PatLeaf<(imm), [{ return immMskBitp(N); }]>;

def immBitp : PatLeaf<(imm), [{
  uint32_t value = (uint32_t)N->getZExtValue();
  return (value >= 1 && value <= 8)
          || value == 16
          || value == 24
          || value == 32;
}]>;

```
- **EN**: Declares TableGen records such as `immU6`, `immU16`, `immMskBitp`, `immBitp` for the backend description.
- **CN**: 为后端描述声明了 `immU6`, `immU16`, `immMskBitp`, `immBitp` 等 TableGen 记录。

### Lines 193-209
```tablegen
def immBpwSubBitp : PatLeaf<(imm), [{
  uint32_t value = (uint32_t)N->getZExtValue();
  return (value >= 24 && value <= 31)
          || value == 16
          || value == 8
          || value == 0;
}]>;

def lda16f : PatFrag<(ops node:$addr, node:$offset),
                     (add node:$addr, (shl node:$offset, 1))>;
def lda16b : PatFrag<(ops node:$addr, node:$offset),
                     (sub node:$addr, (shl node:$offset, 1))>;
def ldawf : PatFrag<(ops node:$addr, node:$offset),
                     (add node:$addr, (shl node:$offset, 2))>;
def ldawb : PatFrag<(ops node:$addr, node:$offset),
                     (sub node:$addr, (shl node:$offset, 2))>;

```
- **EN**: Declares TableGen records such as `immBpwSubBitp`, `lda16f`, `lda16b`, `ldawf`, ... for the backend description.
- **CN**: 为后端描述声明了 `immBpwSubBitp`, `lda16f`, `lda16b`, `ldawf`, ... 等 TableGen 记录。

### Lines 210-227
```tablegen
// Instruction operand types
def pcrel_imm  : Operand<i32>;
def pcrel_imm_neg  : Operand<i32> {
  let DecoderMethod = "DecodeNegImmOperand";
}
def brtarget : Operand<OtherVT>;
def brtarget_neg : Operand<OtherVT> {
  let DecoderMethod = "DecodeNegImmOperand";
}

// Addressing modes
def ADDRspii : ComplexPattern<i32, 2, "SelectADDRspii", [add, frameindex], []>;

// Address operands
def MEMii : Operand<i32> {
  let MIOperandInfo = (ops i32imm, i32imm);
}

```
- **EN**: Declares TableGen records such as `pcrel_imm`, `pcrel_imm_neg`, `brtarget`, `brtarget_neg`, ...; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `pcrel_imm`, `pcrel_imm_neg`, `brtarget`, `brtarget_neg`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 228-251
```tablegen
// Jump tables.
def InlineJT : Operand<i32> {
  let PrintMethod = "printInlineJT";
}

def InlineJT32 : Operand<i32> {
  let PrintMethod = "printInlineJT32";
}

//===----------------------------------------------------------------------===//
// Instruction Class Templates
//===----------------------------------------------------------------------===//

// Three operand short

multiclass F3R_2RUS<bits<5> opc1, bits<5> opc2, string OpcStr, SDNode OpNode> {
  def _3r: _F3R<opc1, (outs GRRegs:$dst), (ins GRRegs:$b, GRRegs:$c),
                !strconcat(OpcStr, " $dst, $b, $c"),
                [(set GRRegs:$dst, (OpNode GRRegs:$b, GRRegs:$c))]>;
  def _2rus : _F2RUS<opc2, (outs GRRegs:$dst), (ins GRRegs:$b, i32imm:$c),
                     !strconcat(OpcStr, " $dst, $b, $c"),
                     [(set GRRegs:$dst, (OpNode GRRegs:$b, immUs:$c))]>;
}

```
- **EN**: Declares TableGen records such as `InlineJT`, `InlineJT32`, `F3R_2RUS`, `_3r`, ...; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments; describes instruction encoding bits.
- **CN**: 声明了 `InlineJT`, `InlineJT32`, `F3R_2RUS`, `_3r`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段、描述指令编码位。

### Lines 252-268
```tablegen
multiclass F3R_2RUS_np<bits<5> opc1, bits<5> opc2, string OpcStr> {
  def _3r: _F3R<opc1, (outs GRRegs:$dst), (ins GRRegs:$b, GRRegs:$c),
                !strconcat(OpcStr, " $dst, $b, $c"), []>;
  def _2rus : _F2RUS<opc2, (outs GRRegs:$dst), (ins GRRegs:$b, i32imm:$c),
                     !strconcat(OpcStr, " $dst, $b, $c"), []>;
}

multiclass F3R_2RBITP<bits<5> opc1, bits<5> opc2, string OpcStr,
                      SDNode OpNode> {
  def _3r: _F3R<opc1, (outs GRRegs:$dst), (ins GRRegs:$b, GRRegs:$c),
                !strconcat(OpcStr, " $dst, $b, $c"),
                [(set GRRegs:$dst, (OpNode GRRegs:$b, GRRegs:$c))]>;
  def _2rus : _F2RUSBitp<opc2, (outs GRRegs:$dst), (ins GRRegs:$b, i32imm:$c),
                         !strconcat(OpcStr, " $dst, $b, $c"),
                         [(set GRRegs:$dst, (OpNode GRRegs:$b, immBitp:$c))]>;
}

```
- **EN**: Declares TableGen records such as `F3R_2RUS_np`, `_3r`, `_2rus`, `F3R_2RBITP`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `F3R_2RUS_np`, `_3r`, `_2rus`, `F3R_2RBITP` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 269-289
```tablegen
class F3R<bits<5> opc, string OpcStr, SDNode OpNode> :
  _F3R<opc, (outs GRRegs:$dst), (ins GRRegs:$b, GRRegs:$c),
       !strconcat(OpcStr, " $dst, $b, $c"),
       [(set GRRegs:$dst, (OpNode GRRegs:$b, GRRegs:$c))]>;

class F3R_np<bits<5> opc, string OpcStr> :
  _F3R<opc, (outs GRRegs:$dst), (ins GRRegs:$b, GRRegs:$c),
       !strconcat(OpcStr, " $dst, $b, $c"), []>;
// Three operand long

/// FL3R_L2RUS multiclass - Define a normal FL3R/FL2RUS pattern in one shot.
multiclass FL3R_L2RUS<bits<9> opc1, bits<9> opc2, string OpcStr,
                      SDNode OpNode> {
  def _l3r: _FL3R<opc1, (outs GRRegs:$dst), (ins GRRegs:$b, GRRegs:$c),
                  !strconcat(OpcStr, " $dst, $b, $c"),
                  [(set GRRegs:$dst, (OpNode GRRegs:$b, GRRegs:$c))]>;
  def _l2rus : _FL2RUS<opc2, (outs GRRegs:$dst), (ins GRRegs:$b, i32imm:$c),
                       !strconcat(OpcStr, " $dst, $b, $c"),
                       [(set GRRegs:$dst, (OpNode GRRegs:$b, immUs:$c))]>;
}

```
- **EN**: Declares TableGen records such as `F3R`, `F3R_np`, `FL3R_L2RUS`, `_l3r`, ...; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `F3R`, `F3R_np`, `FL3R_L2RUS`, `_l3r`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 290-305
```tablegen
/// FL3R_L2RUS multiclass - Define a normal FL3R/FL2RUS pattern in one shot.
multiclass FL3R_L2RBITP<bits<9> opc1, bits<9> opc2, string OpcStr,
                        SDNode OpNode> {
  def _l3r: _FL3R<opc1, (outs GRRegs:$dst), (ins GRRegs:$b, GRRegs:$c),
                  !strconcat(OpcStr, " $dst, $b, $c"),
                  [(set GRRegs:$dst, (OpNode GRRegs:$b, GRRegs:$c))]>;
  def _l2rus : _FL2RUSBitp<opc2, (outs GRRegs:$dst), (ins GRRegs:$b, i32imm:$c),
                           !strconcat(OpcStr, " $dst, $b, $c"),
                           [(set GRRegs:$dst, (OpNode GRRegs:$b, immBitp:$c))]>;
}

class FL3R<bits<9> opc, string OpcStr, SDNode OpNode> :
  _FL3R<opc, (outs GRRegs:$dst), (ins GRRegs:$b, GRRegs:$c),
        !strconcat(OpcStr, " $dst, $b, $c"),
        [(set GRRegs:$dst, (OpNode GRRegs:$b, GRRegs:$c))]>;

```
- **EN**: Declares TableGen records such as `FL3R_L2RBITP`, `_l3r`, `_l2rus`, `FL3R`; this block attaches SelectionDAG patterns; describes instruction encoding bits.
- **CN**: 声明了 `FL3R_L2RBITP`, `_l3r`, `_l2rus`, `FL3R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、描述指令编码位。

### Lines 306-321
```tablegen
// Register - U6
// Operand register - U6
multiclass FRU6_LRU6_branch<bits<6> opc, string OpcStr> {
  def _ru6: _FRU6<opc, (outs), (ins GRRegs:$a, brtarget:$b),
                  !strconcat(OpcStr, " $a, $b"), []>;
  def _lru6: _FLRU6<opc, (outs), (ins GRRegs:$a, brtarget:$b),
                    !strconcat(OpcStr, " $a, $b"), []>;
}

multiclass FRU6_LRU6_backwards_branch<bits<6> opc, string OpcStr> {
  def _ru6: _FRU6<opc, (outs), (ins GRRegs:$a, brtarget_neg:$b),
                  !strconcat(OpcStr, " $a, $b"), []>;
  def _lru6: _FLRU6<opc, (outs), (ins GRRegs:$a, brtarget_neg:$b),
                    !strconcat(OpcStr, " $a, $b"), []>;
}

```
- **EN**: Declares TableGen records such as `FRU6_LRU6_branch`, `_ru6`, `_lru6`, `FRU6_LRU6_backwards_branch`; this block describes instruction encoding bits.
- **CN**: 声明了 `FRU6_LRU6_branch`, `_ru6`, `_lru6`, `FRU6_LRU6_backwards_branch` 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 322-337
```tablegen

// U6
multiclass FU6_LU6<bits<10> opc, string OpcStr, SDNode OpNode> {
  def _u6: _FU6<opc, (outs), (ins i32imm:$a), !strconcat(OpcStr, " $a"),
                [(OpNode immU6:$a)]>;
  def _lu6: _FLU6<opc, (outs), (ins i32imm:$a), !strconcat(OpcStr, " $a"),
                  [(OpNode immU16:$a)]>;
}

multiclass FU6_LU6_int<bits<10> opc, string OpcStr, Intrinsic Int> {
  def _u6: _FU6<opc, (outs), (ins i32imm:$a), !strconcat(OpcStr, " $a"),
                [(Int immU6:$a)]>;
  def _lu6: _FLU6<opc, (outs), (ins i32imm:$a), !strconcat(OpcStr, " $a"),
                  [(Int immU16:$a)]>;
}

```
- **EN**: Declares TableGen records such as `FU6_LU6`, `_u6`, `_lu6`, `FU6_LU6_int`; this block describes instruction encoding bits.
- **CN**: 声明了 `FU6_LU6`, `_u6`, `_lu6`, `FU6_LU6_int` 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 338-353
```tablegen
multiclass FU6_LU6_np<bits<10> opc, string OpcStr> {
  def _u6: _FU6<opc, (outs), (ins i32imm:$a), !strconcat(OpcStr, " $a"), []>;
  def _lu6: _FLU6<opc, (outs), (ins i32imm:$a), !strconcat(OpcStr, " $a"), []>;
}

// Two operand short

class F2R_np<bits<6> opc, string OpcStr> :
  _F2R<opc, (outs GRRegs:$dst), (ins GRRegs:$b),
       !strconcat(OpcStr, " $dst, $b"), []>;

// Two operand long

//===----------------------------------------------------------------------===//
// Pseudo Instructions
//===----------------------------------------------------------------------===//
```
- **EN**: Declares TableGen records such as `FU6_LU6_np`, `_u6`, `_lu6`, `F2R_np`; this block describes instruction encoding bits.
- **CN**: 声明了 `FU6_LU6_np`, `_u6`, `_lu6`, `F2R_np` 等 TableGen 记录；这一段同时描述指令编码位。

### Lines 354-373
```tablegen

let Defs = [SP], Uses = [SP] in {
def ADJCALLSTACKDOWN : PseudoInstXCore<(outs), (ins i32imm:$amt, i32imm:$amt2),
                               "# ADJCALLSTACKDOWN $amt, $amt2",
                               [(callseq_start timm:$amt, timm:$amt2)]>;
def ADJCALLSTACKUP : PseudoInstXCore<(outs), (ins i32imm:$amt1, i32imm:$amt2),
                            "# ADJCALLSTACKUP $amt1",
                            [(callseq_end timm:$amt1, timm:$amt2)]>;
}

let isReMaterializable = 1 in
def FRAME_TO_ARGS_OFFSET : PseudoInstXCore<(outs GRRegs:$dst), (ins),
                               "# FRAME_TO_ARGS_OFFSET $dst",
                               [(set GRRegs:$dst, (frametoargsoffset))]>;

let isReturn = 1, isTerminator = 1, isBarrier = 1 in
def EH_RETURN : PseudoInstXCore<(outs), (ins GRRegs:$s, GRRegs:$handler),
                               "# EH_RETURN $s, $handler",
                               [(XCoreEhRet GRRegs:$s, GRRegs:$handler)]>;

```
- **EN**: Declares TableGen records such as `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP`, `FRAME_TO_ARGS_OFFSET`, `EH_RETURN`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `ADJCALLSTACKDOWN`, `ADJCALLSTACKUP`, `FRAME_TO_ARGS_OFFSET`, `EH_RETURN` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 374-395
```tablegen
def LDWFI : PseudoInstXCore<(outs GRRegs:$dst), (ins MEMii:$addr),
                             "# LDWFI $dst, $addr",
                             [(set GRRegs:$dst, (load ADDRspii:$addr))]>;

def LDAWFI : PseudoInstXCore<(outs GRRegs:$dst), (ins MEMii:$addr),
                             "# LDAWFI $dst, $addr",
                             [(set GRRegs:$dst, ADDRspii:$addr)]>;

def STWFI : PseudoInstXCore<(outs), (ins GRRegs:$src, MEMii:$addr),
                            "# STWFI $src, $addr",
                            [(store GRRegs:$src, ADDRspii:$addr)]>;

// SELECT_CC_* - Used to implement the SELECT_CC DAG operation.  Expanded after
// instruction selection into a branch sequence.
let usesCustomInserter = 1 in {
  def SELECT_CC : PseudoInstXCore<(outs GRRegs:$dst),
                              (ins GRRegs:$cond, GRRegs:$T, GRRegs:$F),
                              "# SELECT_CC PSEUDO!",
                              [(set GRRegs:$dst,
                                 (select GRRegs:$cond, GRRegs:$T, GRRegs:$F))]>;
}

```
- **EN**: Declares TableGen records such as `LDWFI`, `LDAWFI`, `STWFI`, `SELECT_CC`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `LDWFI`, `LDAWFI`, `STWFI`, `SELECT_CC` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 396-415
```tablegen
//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//

// Three operand short
defm ADD : F3R_2RUS<0b00010, 0b10010, "add", add>;
defm SUB : F3R_2RUS<0b00011, 0b10011, "sub", sub>;
let hasSideEffects = 0 in {
defm EQ : F3R_2RUS_np<0b00110, 0b10110, "eq">;
def LSS_3r : F3R_np<0b11000, "lss">;
def LSU_3r : F3R_np<0b11001, "lsu">;
}
def AND_3r : F3R<0b00111, "and", and>;
def OR_3r : F3R<0b01000, "or", or>;

let mayLoad=1 in {
def LDW_3r : _F3R<0b01001, (outs GRRegs:$dst),
                  (ins GRRegs:$addr, GRRegs:$offset),
                  "ldw $dst, $addr[$offset]", []>;

```
- **EN**: Declares TableGen records such as `ADD`, `SUB`, `EQ`, `LSS_3r`, ...; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `ADD`, `SUB`, `EQ`, `LSS_3r`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 416-433
```tablegen
def LDW_2rus : _F2RUS<0b00001, (outs GRRegs:$dst),
                      (ins GRRegs:$addr, i32imm:$offset),
                      "ldw $dst, $addr[$offset]", []>;

def LD16S_3r :  _F3R<0b10000, (outs GRRegs:$dst),
                     (ins GRRegs:$addr, GRRegs:$offset),
                     "ld16s $dst, $addr[$offset]", []>;

def LD8U_3r :  _F3R<0b10001, (outs GRRegs:$dst),
                    (ins GRRegs:$addr, GRRegs:$offset),
                    "ld8u $dst, $addr[$offset]", []>;
}

let mayStore=1 in {
def STW_l3r : _FL3R<0b000001100, (outs),
                    (ins GRRegs:$val, GRRegs:$addr, GRRegs:$offset),
                    "stw $val, $addr[$offset]", []>;

```
- **EN**: Declares TableGen records such as `LDW_2rus`, `LD16S_3r`, `LD8U_3r`, `STW_l3r`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `LDW_2rus`, `LD16S_3r`, `LD8U_3r`, `STW_l3r` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 434-453
```tablegen
def STW_2rus : _F2RUS<0b00000, (outs),
                      (ins GRRegs:$val, GRRegs:$addr, i32imm:$offset),
                      "stw $val, $addr[$offset]", []>;
}

defm SHL : F3R_2RBITP<0b00100, 0b10100, "shl", shl>;
defm SHR : F3R_2RBITP<0b00101, 0b10101, "shr", srl>;

// The first operand is treated as an immediate since it refers to a register
// number in another thread.
def TSETR_3r : _F3RImm<0b10111, (outs), (ins i32imm:$a, GRRegs:$b, GRRegs:$c),
                       "set t[$c]:r$a, $b", []>;

// Three operand long
def LDAWF_l3r : _FL3R<0b000111100, (outs GRRegs:$dst),
                      (ins GRRegs:$addr, GRRegs:$offset),
                      "ldaw $dst, $addr[$offset]",
                      [(set GRRegs:$dst,
                         (ldawf GRRegs:$addr, GRRegs:$offset))]>;

```
- **EN**: Declares TableGen records such as `STW_2rus`, `SHL`, `SHR`, `TSETR_3r`, ...; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `STW_2rus`, `SHL`, `SHR`, `TSETR_3r`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 454-469
```tablegen
let hasSideEffects = 0 in
def LDAWF_l2rus : _FL2RUS<0b100111100, (outs GRRegs:$dst),
                          (ins GRRegs:$addr, i32imm:$offset),
                          "ldaw $dst, $addr[$offset]", []>;

def LDAWB_l3r : _FL3R<0b001001100, (outs GRRegs:$dst),
                      (ins GRRegs:$addr, GRRegs:$offset),
                      "ldaw $dst, $addr[-$offset]",
                      [(set GRRegs:$dst,
                         (ldawb GRRegs:$addr, GRRegs:$offset))]>;

let hasSideEffects = 0 in
def LDAWB_l2rus : _FL2RUS<0b101001100, (outs GRRegs:$dst),
                         (ins GRRegs:$addr, i32imm:$offset),
                         "ldaw $dst, $addr[-$offset]", []>;

```
- **EN**: Declares TableGen records such as `LDAWF_l2rus`, `LDAWB_l3r`, `LDAWB_l2rus`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `LDAWF_l2rus`, `LDAWB_l3r`, `LDAWB_l2rus` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 470-492
```tablegen
def LDA16F_l3r : _FL3R<0b001011100, (outs GRRegs:$dst),
                       (ins GRRegs:$addr, GRRegs:$offset),
                       "lda16 $dst, $addr[$offset]",
                       [(set GRRegs:$dst,
                          (lda16f GRRegs:$addr, GRRegs:$offset))]>;

def LDA16B_l3r : _FL3R<0b001101100, (outs GRRegs:$dst),
                       (ins GRRegs:$addr, GRRegs:$offset),
                       "lda16 $dst, $addr[-$offset]",
                       [(set GRRegs:$dst,
                          (lda16b GRRegs:$addr, GRRegs:$offset))]>;

def MUL_l3r : FL3R<0b001111100, "mul", mul>;
// Instructions which may trap are marked as side effecting.
let hasSideEffects = 1 in {
def DIVS_l3r : FL3R<0b010001100, "divs", sdiv>;
def DIVU_l3r : FL3R<0b010011100, "divu", udiv>;
def REMS_l3r : FL3R<0b110001100, "rems", srem>;
def REMU_l3r : FL3R<0b110011100, "remu", urem>;
}
def XOR_l3r : FL3R<0b000011100, "xor", xor>;
defm ASHR : FL3R_L2RBITP<0b000101100, 0b100101100, "ashr", sra>;

```
- **EN**: Declares TableGen records such as `LDA16F_l3r`, `LDA16B_l3r`, `MUL_l3r`, `DIVS_l3r`, ...; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `LDA16F_l3r`, `LDA16B_l3r`, `MUL_l3r`, `DIVS_l3r`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 493-510
```tablegen
let Constraints = "$src1 = $dst" in
def CRC_l3r : _FL3RSrcDst<0b101011100, (outs GRRegs:$dst),
                          (ins GRRegs:$src1, GRRegs:$src2, GRRegs:$src3),
                          "crc32 $dst, $src2, $src3",
                          [(set GRRegs:$dst,
                             (int_xcore_crc32 GRRegs:$src1, GRRegs:$src2,
                                              GRRegs:$src3))]>;

let mayStore=1 in {
def ST16_l3r : _FL3R<0b100001100, (outs),
                     (ins GRRegs:$val, GRRegs:$addr, GRRegs:$offset),
                     "st16 $val, $addr[$offset]", []>;

def ST8_l3r : _FL3R<0b100011100, (outs),
                    (ins GRRegs:$val, GRRegs:$addr, GRRegs:$offset),
                    "st8 $val, $addr[$offset]", []>;
}

```
- **EN**: Declares TableGen records such as `CRC_l3r`, `ST16_l3r`, `ST8_l3r`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `CRC_l3r`, `ST16_l3r`, `ST8_l3r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 511-531
```tablegen
def INPW_l2rus : _FL2RUSBitp<0b100101110, (outs GRRegs:$a),
                             (ins GRRegs:$b, i32imm:$c), "inpw $a, res[$b], $c",
                             []>;

def OUTPW_l2rus : _FL2RUSBitp<0b100101101, (outs),
                              (ins GRRegs:$a, GRRegs:$b, i32imm:$c),
                              "outpw res[$b], $a, $c", []>;

// Four operand long
let Constraints = "$e = $a,$f = $b" in {
def MACCU_l4r : _FL4RSrcDstSrcDst<
  0b000001, (outs GRRegs:$a, GRRegs:$b),
  (ins GRRegs:$e, GRRegs:$f, GRRegs:$c, GRRegs:$d), "maccu $a, $b, $c, $d",
  [(set i32:$a, i32:$b, (XCoreMAccU i32:$e, i32:$f, i32:$c, i32:$d))]>;

def MACCS_l4r : _FL4RSrcDstSrcDst<
  0b000010, (outs GRRegs:$a, GRRegs:$b),
  (ins GRRegs:$e, GRRegs:$f, GRRegs:$c, GRRegs:$d), "maccs $a, $b, $c, $d",
  [(set i32:$a, i32:$b, (XCoreMAccS i32:$e, i32:$f, i32:$c, i32:$d))]>;
}

```
- **EN**: Declares TableGen records such as `INPW_l2rus`, `OUTPW_l2rus`, `MACCU_l4r`, `MACCS_l4r`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `INPW_l2rus`, `OUTPW_l2rus`, `MACCU_l4r`, `MACCS_l4r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 532-552
```tablegen
let Constraints = "$e = $b" in
def CRC8_l4r : _FL4RSrcDst<0b000000, (outs GRRegs:$a, GRRegs:$b),
                           (ins GRRegs:$e, GRRegs:$c, GRRegs:$d),
                           "crc8 $b, $a, $c, $d",
                           [(set i32:$a, i32:$b,
                                 (XCoreCRC8 i32:$e, i32:$c, i32:$d))]>;

// Five operand long

def LADD_l5r : _FL5R<0b000001, (outs GRRegs:$dst1, GRRegs:$dst2),
                     (ins GRRegs:$src1, GRRegs:$src2, GRRegs:$src3),
                     "ladd $dst2, $dst1, $src1, $src2, $src3",
                     [(set i32:$dst1, i32:$dst2,
                           (XCoreLAdd i32:$src1, i32:$src2, i32:$src3))]>;

def LSUB_l5r : _FL5R<0b000010, (outs GRRegs:$dst1, GRRegs:$dst2),
                     (ins GRRegs:$src1, GRRegs:$src2, GRRegs:$src3),
                     "lsub $dst2, $dst1, $src1, $src2, $src3",
                     [(set i32:$dst1, i32:$dst2,
                           (XCoreLSub i32:$src1, i32:$src2, i32:$src3))]>;

```
- **EN**: Declares TableGen records such as `CRC8_l4r`, `LADD_l5r`, `LSUB_l5r`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `CRC8_l4r`, `LADD_l5r`, `LSUB_l5r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 553-572
```tablegen
def LDIVU_l5r : _FL5R<0b000000, (outs GRRegs:$dst1, GRRegs:$dst2),
                      (ins GRRegs:$src1, GRRegs:$src2, GRRegs:$src3),
                      "ldivu $dst1, $dst2, $src3, $src1, $src2", []>;

// Six operand long

def LMUL_l6r : _FL6R<
  0b00000, (outs GRRegs:$dst1, GRRegs:$dst2),
  (ins GRRegs:$src1, GRRegs:$src2, GRRegs:$src3, GRRegs:$src4),
  "lmul $dst1, $dst2, $src1, $src2, $src3, $src4",
  [(set i32:$dst1, i32:$dst2,
        (XCoreLMul i32:$src1, i32:$src2, i32:$src3, i32:$src4))]>;

// Register - U6

//let Uses = [DP] in ...
let hasSideEffects = 0, isReMaterializable = 1 in
def LDAWDP_ru6: _FRU6<0b011000, (outs RRegs:$a), (ins i32imm:$b),
                      "ldaw $a, dp[$b]", []>;

```
- **EN**: Declares TableGen records such as `LDIVU_l5r`, `LMUL_l6r`, `LDAWDP_ru6`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `LDIVU_l5r`, `LMUL_l6r`, `LDAWDP_ru6` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 573-589
```tablegen
let isReMaterializable = 1 in
def LDAWDP_lru6: _FLRU6<0b011000, (outs RRegs:$a), (ins i32imm:$b),
                        "ldaw $a, dp[$b]",
                        [(set RRegs:$a, (dprelwrapper tglobaladdr:$b))]>;

let mayLoad=1 in
def LDWDP_ru6: _FRU6<0b010110, (outs RRegs:$a), (ins i32imm:$b),
                     "ldw $a, dp[$b]", []>;

def LDWDP_lru6: _FLRU6<0b010110, (outs RRegs:$a), (ins i32imm:$b),
                       "ldw $a, dp[$b]",
                       [(set RRegs:$a, (load (dprelwrapper tglobaladdr:$b)))]>;

let mayStore=1 in
def STWDP_ru6 : _FRU6<0b010100, (outs), (ins RRegs:$a, i32imm:$b),
                      "stw $a, dp[$b]", []>;

```
- **EN**: Declares TableGen records such as `LDAWDP_lru6`, `LDWDP_ru6`, `LDWDP_lru6`, `STWDP_ru6`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `LDAWDP_lru6`, `LDWDP_ru6`, `LDWDP_lru6`, `STWDP_ru6` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 590-608
```tablegen
def STWDP_lru6 : _FLRU6<0b010100, (outs), (ins RRegs:$a, i32imm:$b),
                        "stw $a, dp[$b]",
                        [(store RRegs:$a, (dprelwrapper tglobaladdr:$b))]>;

//let Uses = [CP] in ..
let mayLoad = 1, isReMaterializable = 1, hasSideEffects = 0 in {
def LDWCP_ru6 : _FRU6<0b011011, (outs RRegs:$a), (ins i32imm:$b),
                      "ldw $a, cp[$b]", []>;
def LDWCP_lru6: _FLRU6<0b011011, (outs RRegs:$a), (ins i32imm:$b),
                       "ldw $a, cp[$b]",
                       [(set RRegs:$a, (load (cprelwrapper tglobaladdr:$b)))]>;
}

let Uses = [SP] in {
let mayStore=1 in {
def STWSP_ru6 : _FRU6<0b010101, (outs), (ins RRegs:$a, i32imm:$b),
                      "stw $a, sp[$b]",
                      [(XCoreStwsp RRegs:$a, immU6:$b)]>;

```
- **EN**: Declares TableGen records such as `STWDP_lru6`, `LDWCP_ru6`, `LDWCP_lru6`, `STWSP_ru6`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `STWDP_lru6`, `LDWCP_ru6`, `LDWCP_lru6`, `STWSP_ru6` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 609-627
```tablegen
def STWSP_lru6 : _FLRU6<0b010101, (outs), (ins RRegs:$a, i32imm:$b),
                        "stw $a, sp[$b]",
                        [(XCoreStwsp RRegs:$a, immU16:$b)]>;
}

let mayLoad=1 in {
def LDWSP_ru6 : _FRU6<0b010111, (outs RRegs:$a), (ins i32imm:$b),
                      "ldw $a, sp[$b]",
                      [(set RRegs:$a, (XCoreLdwsp immU6:$b))]>;

def LDWSP_lru6 : _FLRU6<0b010111, (outs RRegs:$a), (ins i32imm:$b),
                        "ldw $a, sp[$b]",
                        [(set RRegs:$a, (XCoreLdwsp immU16:$b))]>;
}

let hasSideEffects = 0 in {
def LDAWSP_ru6 : _FRU6<0b011001, (outs RRegs:$a), (ins i32imm:$b),
                       "ldaw $a, sp[$b]", []>;

```
- **EN**: Declares TableGen records such as `STWSP_lru6`, `LDWSP_ru6`, `LDWSP_lru6`, `LDAWSP_ru6`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `STWSP_lru6`, `LDWSP_ru6`, `LDWSP_lru6`, `LDAWSP_ru6` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 628-644
```tablegen
def LDAWSP_lru6 : _FLRU6<0b011001, (outs RRegs:$a), (ins i32imm:$b),
                         "ldaw $a, sp[$b]", []>;
}
}

let isReMaterializable = 1 in {
def LDC_ru6 : _FRU6<0b011010, (outs RRegs:$a), (ins i32imm:$b),
                    "ldc $a, $b", [(set RRegs:$a, immU6:$b)]>;

def LDC_lru6 : _FLRU6<0b011010, (outs RRegs:$a), (ins i32imm:$b),
                      "ldc $a, $b", [(set RRegs:$a, immU16:$b)]>;
}

def SETC_ru6 : _FRU6<0b111010, (outs), (ins GRRegs:$a, i32imm:$b),
                     "setc res[$a], $b",
                     [(int_xcore_setc GRRegs:$a, immU6:$b)]>;

```
- **EN**: Declares TableGen records such as `LDAWSP_lru6`, `LDC_ru6`, `LDC_lru6`, `SETC_ru6`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `LDAWSP_lru6`, `LDC_ru6`, `LDC_lru6`, `SETC_ru6` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 645-661
```tablegen
def SETC_lru6 : _FLRU6<0b111010, (outs), (ins GRRegs:$a, i32imm:$b),
                       "setc res[$a], $b",
                       [(int_xcore_setc GRRegs:$a, immU16:$b)]>;

// Operand register - U6
let isBranch = 1, isTerminator = 1 in {
defm BRFT: FRU6_LRU6_branch<0b011100, "bt">;
defm BRBT: FRU6_LRU6_backwards_branch<0b011101, "bt">;
defm BRFF: FRU6_LRU6_branch<0b011110, "bf">;
defm BRBF: FRU6_LRU6_backwards_branch<0b011111, "bf">;
}

// U6
let Defs = [SP], Uses = [SP] in {
let hasSideEffects = 0 in
defm EXTSP : FU6_LU6_np<0b0111011110, "extsp">;

```
- **EN**: Declares TableGen records such as `SETC_lru6`, `BRFT`, `BRBT`, `BRFF`, ...; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `SETC_lru6`, `BRFT`, `BRBT`, `BRFF`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 662-678
```tablegen
let mayStore = 1 in
defm ENTSP : FU6_LU6_np<0b0111011101, "entsp">;

let isReturn = 1, isTerminator = 1, mayLoad = 1, isBarrier = 1 in {
defm RETSP : FU6_LU6<0b0111011111, "retsp", XCoreRetsp>;
}
}

let hasSideEffects = 0 in
defm EXTDP : FU6_LU6_np<0b0111001110, "extdp">;

let Uses = [R11], isCall=1 in
defm BLAT : FU6_LU6_np<0b0111001101, "blat">;

let isBranch = 1, isTerminator = 1, isBarrier = 1 in {
def BRBU_u6 : _FU6<0b0111011100, (outs), (ins brtarget_neg:$a), "bu $a", []>;

```
- **EN**: Declares TableGen records such as `ENTSP`, `RETSP`, `EXTDP`, `BLAT`, ...; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `ENTSP`, `RETSP`, `EXTDP`, `BLAT`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 679-694
```tablegen
def BRBU_lu6 : _FLU6<0b0111011100, (outs), (ins brtarget_neg:$a), "bu $a", []>;

def BRFU_u6 : _FU6<0b0111001100, (outs), (ins brtarget:$a), "bu $a", []>;

def BRFU_lu6 : _FLU6<0b0111001100, (outs), (ins brtarget:$a), "bu $a", []>;
}

//let Uses = [CP] in ...
let Defs = [R11], hasSideEffects = 0, isReMaterializable = 1 in
def LDAWCP_u6: _FU6<0b0111111101, (outs), (ins i32imm:$a), "ldaw r11, cp[$a]",
                    []>;

let Defs = [R11], isReMaterializable = 1 in
def LDAWCP_lu6: _FLU6<0b0111111101, (outs), (ins i32imm:$a), "ldaw r11, cp[$a]",
                      [(set R11, (cprelwrapper tglobaladdr:$a))]>;

```
- **EN**: Declares TableGen records such as `BRBU_lu6`, `BRFU_u6`, `BRFU_lu6`, `LDAWCP_u6`, ...; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `BRBU_lu6`, `BRFU_u6`, `BRFU_lu6`, `LDAWCP_u6`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 695-711
```tablegen
let Defs = [R11] in
defm GETSR : FU6_LU6_np<0b0111111100, "getsr r11,">;

defm SETSR : FU6_LU6_int<0b0111101101, "setsr", int_xcore_setsr>;

defm CLRSR : FU6_LU6_int<0b0111101100, "clrsr", int_xcore_clrsr>;

// setsr may cause a branch if it is used to enable events. clrsr may
// branch if it is executed while events are enabled.
let isBranch=1, isIndirectBranch=1, isTerminator=1, isBarrier = 1,
    isCodeGenOnly = 1 in {
defm SETSR_branch : FU6_LU6_np<0b0111101101, "setsr">;
defm CLRSR_branch : FU6_LU6_np<0b0111101100, "clrsr">;
}

defm KCALL : FU6_LU6_np<0b0111001111, "kcall">;

```
- **EN**: Declares TableGen records such as `GETSR`, `SETSR`, `CLRSR`, `SETSR_branch`, ...; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `GETSR`, `SETSR`, `CLRSR`, `SETSR_branch`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 712-730
```tablegen
let Uses = [SP], Defs = [SP], mayStore = 1 in
defm KENTSP : FU6_LU6_np<0b0111101110, "kentsp">;

let Uses = [SP], Defs = [SP], mayLoad = 1 in
defm KRESTSP : FU6_LU6_np<0b0111101111, "krestsp">;

// U10

let Defs = [R11], isReMaterializable = 1 in {
let hasSideEffects = 0 in
def LDAPF_u10 : _FU10<0b110110, (outs), (ins pcrel_imm:$a), "ldap r11, $a", []>;

def LDAPF_lu10 : _FLU10<0b110110, (outs), (ins pcrel_imm:$a), "ldap r11, $a",
                        [(set R11, (pcrelwrapper tglobaladdr:$a))]>;

let hasSideEffects = 0 in
def LDAPB_u10 : _FU10<0b110111, (outs), (ins pcrel_imm_neg:$a), "ldap r11, $a",
                      []>;

```
- **EN**: Declares TableGen records such as `KENTSP`, `KRESTSP`, `LDAPF_u10`, `LDAPF_lu10`, ...; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `KENTSP`, `KRESTSP`, `LDAPF_u10`, `LDAPF_lu10`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 731-747
```tablegen
let hasSideEffects = 0 in
def LDAPB_lu10 : _FLU10<0b110111, (outs), (ins pcrel_imm_neg:$a),
                        "ldap r11, $a",
                        [(set R11, (pcrelwrapper tglobaladdr:$a))]>;

let isCodeGenOnly = 1 in
def LDAPF_lu10_ba : _FLU10<0b110110, (outs), (ins pcrel_imm:$a), "ldap r11, $a",
                           [(set R11, (pcrelwrapper tblockaddress:$a))]>;
}

let isCall=1,
// All calls clobber the link register and the non-callee-saved registers:
Defs = [R0, R1, R2, R3, R11, LR], Uses = [SP] in {
def BLACP_u10 : _FU10<0b111000, (outs), (ins i32imm:$a), "bla cp[$a]", []>;

def BLACP_lu10 : _FLU10<0b111000, (outs), (ins i32imm:$a), "bla cp[$a]", []>;

```
- **EN**: Declares TableGen records such as `LDAPB_lu10`, `LDAPF_lu10_ba`, `BLACP_u10`, `BLACP_lu10`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `LDAPB_lu10`, `LDAPF_lu10_ba`, `BLACP_u10`, `BLACP_lu10` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 748-766
```tablegen
def BLRF_u10 : _FU10<0b110100, (outs), (ins pcrel_imm:$a), "bl $a",
                     []>;

def BLRF_lu10 : _FLU10<0b110100, (outs), (ins pcrel_imm:$a), "bl $a",
                       [(XCoreBranchLink tglobaladdr:$a)]>;

def BLRB_u10 : _FU10<0b110101, (outs), (ins pcrel_imm_neg:$a), "bl $a", []>;

def BLRB_lu10 : _FLU10<0b110101, (outs), (ins pcrel_imm_neg:$a), "bl $a", []>;
}

let Defs = [R11], mayLoad = 1, isReMaterializable = 1,
    hasSideEffects = 0 in {
def LDWCP_u10 : _FU10<0b111001, (outs), (ins i32imm:$a), "ldw r11, cp[$a]", []>;

def LDWCP_lu10 : _FLU10<0b111001, (outs), (ins i32imm:$a), "ldw r11, cp[$a]",
                        []>;
}

```
- **EN**: Declares TableGen records such as `BLRF_u10`, `BLRF_lu10`, `BLRB_u10`, `BLRB_lu10`, ...; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `BLRF_u10`, `BLRF_lu10`, `BLRB_u10`, `BLRB_lu10`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 767-785
```tablegen
// Two operand short
def NOT : _F2R<0b100010, (outs GRRegs:$dst), (ins GRRegs:$b),
                "not $dst, $b", [(set GRRegs:$dst, (not GRRegs:$b))]>;

def NEG : _F2R<0b100100, (outs GRRegs:$dst), (ins GRRegs:$b),
                "neg $dst, $b", [(set GRRegs:$dst, (ineg GRRegs:$b))]>;

let Constraints = "$src1 = $dst" in {
def SEXT_rus :
  _FRUSSrcDstBitp<0b001101, (outs GRRegs:$dst), (ins GRRegs:$src1, i32imm:$src2),
                  "sext $dst, $src2",
                  [(set GRRegs:$dst, (int_xcore_sext GRRegs:$src1,
                                                     immBitp:$src2))]>;

def SEXT_2r :
  _F2RSrcDst<0b001100, (outs GRRegs:$dst), (ins GRRegs:$src1, GRRegs:$src2),
             "sext $dst, $src2",
             [(set GRRegs:$dst, (int_xcore_sext GRRegs:$src1, GRRegs:$src2))]>;

```
- **EN**: Declares TableGen records such as `NOT`, `NEG`, `SEXT_rus`, `SEXT_2r`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `NOT`, `NEG`, `SEXT_rus`, `SEXT_2r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 786-802
```tablegen
def ZEXT_rus :
  _FRUSSrcDstBitp<0b010001, (outs GRRegs:$dst), (ins GRRegs:$src1, i32imm:$src2),
                  "zext $dst, $src2",
                  [(set GRRegs:$dst, (int_xcore_zext GRRegs:$src1,
                                                     immBitp:$src2))]>;

def ZEXT_2r :
  _F2RSrcDst<0b010000, (outs GRRegs:$dst), (ins GRRegs:$src1, GRRegs:$src2),
             "zext $dst, $src2",
             [(set GRRegs:$dst, (int_xcore_zext GRRegs:$src1, GRRegs:$src2))]>;

def ANDNOT_2r :
  _F2RSrcDst<0b001010, (outs GRRegs:$dst), (ins GRRegs:$src1, GRRegs:$src2),
             "andnot $dst, $src2",
             [(set GRRegs:$dst, (and GRRegs:$src1, (not GRRegs:$src2)))]>;
}

```
- **EN**: Declares TableGen records such as `ZEXT_rus`, `ZEXT_2r`, `ANDNOT_2r`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `ZEXT_rus`, `ZEXT_2r`, `ANDNOT_2r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 803-818
```tablegen
let isReMaterializable = 1, hasSideEffects = 0 in
def MKMSK_rus : _FRUSBitp<0b101001, (outs GRRegs:$dst), (ins i32imm:$size),
                          "mkmsk $dst, $size", []>;

def MKMSK_2r : _F2R<0b101000, (outs GRRegs:$dst), (ins GRRegs:$size),
                    "mkmsk $dst, $size",
                    [(set GRRegs:$dst, (add (shl 1, GRRegs:$size), -1))]>;

def GETR_rus : _FRUS<0b100000, (outs GRRegs:$dst), (ins i32imm:$type),
                     "getr $dst, $type",
                     [(set GRRegs:$dst, (int_xcore_getr immUs:$type))]>;

def GETTS_2r : _F2R<0b001110, (outs GRRegs:$dst), (ins GRRegs:$r),
                    "getts $dst, res[$r]",
                    [(set GRRegs:$dst, (int_xcore_getts GRRegs:$r))]>;

```
- **EN**: Declares TableGen records such as `MKMSK_rus`, `MKMSK_2r`, `GETR_rus`, `GETTS_2r`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `MKMSK_rus`, `MKMSK_2r`, `GETR_rus`, `GETTS_2r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 819-834
```tablegen
def SETPT_2r : _FR2R<0b001111, (outs), (ins GRRegs:$r, GRRegs:$val),
                     "setpt res[$r], $val",
                     [(int_xcore_setpt GRRegs:$r, GRRegs:$val)]>;

def OUTCT_2r : _F2R<0b010010, (outs), (ins GRRegs:$r, GRRegs:$val),
                    "outct res[$r], $val",
                    [(int_xcore_outct GRRegs:$r, GRRegs:$val)]>;

def OUTCT_rus : _FRUS<0b010011, (outs), (ins GRRegs:$r, i32imm:$val),
                       "outct res[$r], $val",
                       [(int_xcore_outct GRRegs:$r, immUs:$val)]>;

def OUTT_2r : _FR2R<0b000011, (outs), (ins GRRegs:$r, GRRegs:$val),
                    "outt res[$r], $val",
                    [(int_xcore_outt GRRegs:$r, GRRegs:$val)]>;

```
- **EN**: Declares TableGen records such as `SETPT_2r`, `OUTCT_2r`, `OUTCT_rus`, `OUTT_2r` for the backend description.
- **CN**: 为后端描述声明了 `SETPT_2r`, `OUTCT_2r`, `OUTCT_rus`, `OUTT_2r` 等 TableGen 记录。

### Lines 835-852
```tablegen
def OUT_2r : _FR2R<0b101010, (outs), (ins GRRegs:$r, GRRegs:$val),
                   "out res[$r], $val",
                   [(int_xcore_out GRRegs:$r, GRRegs:$val)]>;

let Constraints = "$src = $dst" in
def OUTSHR_2r :
  _F2RSrcDst<0b101011, (outs GRRegs:$dst), (ins GRRegs:$src, GRRegs:$r),
             "outshr res[$r], $src",
             [(set GRRegs:$dst, (int_xcore_outshr GRRegs:$r, GRRegs:$src))]>;

def INCT_2r : _F2R<0b100001, (outs GRRegs:$dst), (ins GRRegs:$r),
                   "inct $dst, res[$r]",
                   [(set GRRegs:$dst, (int_xcore_inct GRRegs:$r))]>;

def INT_2r : _F2R<0b100011, (outs GRRegs:$dst), (ins GRRegs:$r),
                  "int $dst, res[$r]",
                  [(set GRRegs:$dst, (int_xcore_int GRRegs:$r))]>;

```
- **EN**: Declares TableGen records such as `OUT_2r`, `OUTSHR_2r`, `INCT_2r`, `INT_2r`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `OUT_2r`, `OUTSHR_2r`, `INCT_2r`, `INT_2r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 853-870
```tablegen
def IN_2r : _F2R<0b101100, (outs GRRegs:$dst), (ins GRRegs:$r),
                 "in $dst, res[$r]",
                 [(set GRRegs:$dst, (int_xcore_in GRRegs:$r))]>;

let Constraints = "$src = $dst" in
def INSHR_2r :
  _F2RSrcDst<0b101101, (outs GRRegs:$dst), (ins GRRegs:$src, GRRegs:$r),
             "inshr $dst, res[$r]",
             [(set GRRegs:$dst, (int_xcore_inshr GRRegs:$r, GRRegs:$src))]>;

def CHKCT_2r : _F2R<0b110010, (outs), (ins GRRegs:$r, GRRegs:$val),
                    "chkct res[$r], $val",
                    [(int_xcore_chkct GRRegs:$r, GRRegs:$val)]>;

def CHKCT_rus : _FRUSBitp<0b110011, (outs), (ins GRRegs:$r, i32imm:$val),
                          "chkct res[$r], $val",
                          [(int_xcore_chkct GRRegs:$r, immUs:$val)]>;

```
- **EN**: Declares TableGen records such as `IN_2r`, `INSHR_2r`, `CHKCT_2r`, `CHKCT_rus`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `IN_2r`, `INSHR_2r`, `CHKCT_2r`, `CHKCT_rus` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 871-886
```tablegen
def TESTCT_2r : _F2R<0b101111, (outs GRRegs:$dst), (ins GRRegs:$src),
                     "testct $dst, res[$src]",
                     [(set GRRegs:$dst, (int_xcore_testct GRRegs:$src))]>;

def TESTWCT_2r : _F2R<0b110001, (outs GRRegs:$dst), (ins GRRegs:$src),
                      "testwct $dst, res[$src]",
                      [(set GRRegs:$dst, (int_xcore_testwct GRRegs:$src))]>;

def SETD_2r : _FR2R<0b000101, (outs), (ins GRRegs:$r, GRRegs:$val),
                    "setd res[$r], $val",
                    [(int_xcore_setd GRRegs:$r, GRRegs:$val)]>;

def SETPSC_2r : _FR2R<0b110000, (outs), (ins GRRegs:$src1, GRRegs:$src2),
                      "setpsc res[$src1], $src2",
                      [(int_xcore_setpsc GRRegs:$src1, GRRegs:$src2)]>;

```
- **EN**: Declares TableGen records such as `TESTCT_2r`, `TESTWCT_2r`, `SETD_2r`, `SETPSC_2r`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `TESTCT_2r`, `TESTWCT_2r`, `SETD_2r`, `SETPSC_2r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 887-902
```tablegen
def GETST_2r : _F2R<0b000001, (outs GRRegs:$dst), (ins GRRegs:$r),
                    "getst $dst, res[$r]",
                    [(set GRRegs:$dst, (int_xcore_getst GRRegs:$r))]>;

def INITSP_2r : _F2R<0b000100, (outs), (ins GRRegs:$src, GRRegs:$t),
                     "init t[$t]:sp, $src",
                     [(int_xcore_initsp GRRegs:$t, GRRegs:$src)]>;

def INITPC_2r : _F2R<0b000000, (outs), (ins GRRegs:$src, GRRegs:$t),
                     "init t[$t]:pc, $src",
                     [(int_xcore_initpc GRRegs:$t, GRRegs:$src)]>;

def INITCP_2r : _F2R<0b000110, (outs), (ins GRRegs:$src, GRRegs:$t),
                     "init t[$t]:cp, $src",
                     [(int_xcore_initcp GRRegs:$t, GRRegs:$src)]>;

```
- **EN**: Declares TableGen records such as `GETST_2r`, `INITSP_2r`, `INITPC_2r`, `INITCP_2r`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `GETST_2r`, `INITSP_2r`, `INITPC_2r`, `INITCP_2r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 903-920
```tablegen
def INITDP_2r : _F2R<0b000010, (outs), (ins GRRegs:$src, GRRegs:$t),
                     "init t[$t]:dp, $src",
                     [(int_xcore_initdp GRRegs:$t, GRRegs:$src)]>;

def PEEK_2r : _F2R<0b101110, (outs GRRegs:$dst), (ins GRRegs:$src),
                    "peek $dst, res[$src]",
                    [(set GRRegs:$dst, (int_xcore_peek GRRegs:$src))]>;

def ENDIN_2r : _F2R<0b100101, (outs GRRegs:$dst), (ins GRRegs:$src),
                     "endin $dst, res[$src]",
                     [(set GRRegs:$dst, (int_xcore_endin GRRegs:$src))]>;

def EEF_2r : _F2R<0b001011, (outs), (ins GRRegs:$a, GRRegs:$b),
                  "eef $a, res[$b]", []>;

def EET_2r : _F2R<0b001001, (outs), (ins GRRegs:$a, GRRegs:$b),
                  "eet $a, res[$b]", []>;

```
- **EN**: Declares TableGen records such as `INITDP_2r`, `PEEK_2r`, `ENDIN_2r`, `EEF_2r`, ...; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `INITDP_2r`, `PEEK_2r`, `ENDIN_2r`, `EEF_2r`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 921-936
```tablegen
def TSETMR_2r : _F2RImm<0b000111, (outs), (ins i32imm:$a, GRRegs:$b),
                        "tsetmr r$a, $b", []>;

// Two operand long
def BITREV_l2r : _FL2R<0b0000011000, (outs GRRegs:$dst), (ins GRRegs:$src),
                       "bitrev $dst, $src",
                       [(set GRRegs:$dst, (int_xcore_bitrev GRRegs:$src))]>;

def BYTEREV_l2r : _FL2R<0b0000011001, (outs GRRegs:$dst), (ins GRRegs:$src),
                        "byterev $dst, $src",
                        [(set GRRegs:$dst, (bswap GRRegs:$src))]>;

def CLZ_l2r : _FL2R<0b0000111000, (outs GRRegs:$dst), (ins GRRegs:$src),
                    "clz $dst, $src",
                    [(set GRRegs:$dst, (ctlz GRRegs:$src))]>;

```
- **EN**: Declares TableGen records such as `TSETMR_2r`, `BITREV_l2r`, `BYTEREV_l2r`, `CLZ_l2r`; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `TSETMR_2r`, `BITREV_l2r`, `BYTEREV_l2r`, `CLZ_l2r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 937-954
```tablegen
def GETD_l2r : _FL2R<0b0001111001, (outs GRRegs:$dst), (ins GRRegs:$src),
                     "getd $dst, res[$src]", []>;

def GETN_l2r : _FL2R<0b0011011001, (outs GRRegs:$dst), (ins GRRegs:$src),
                     "getn $dst, res[$src]", []>;

def SETC_l2r : _FL2R<0b0010111001, (outs), (ins GRRegs:$r, GRRegs:$val),
                     "setc res[$r], $val",
                     [(int_xcore_setc GRRegs:$r, GRRegs:$val)]>;

def SETTW_l2r : _FLR2R<0b0010011001, (outs), (ins GRRegs:$r, GRRegs:$val),
                       "settw res[$r], $val",
                       [(int_xcore_settw GRRegs:$r, GRRegs:$val)]>;

def GETPS_l2r : _FL2R<0b0001011001, (outs GRRegs:$dst), (ins GRRegs:$src),
                      "get $dst, ps[$src]",
                      [(set GRRegs:$dst, (int_xcore_getps GRRegs:$src))]>;

```
- **EN**: Declares TableGen records such as `GETD_l2r`, `GETN_l2r`, `SETC_l2r`, `SETTW_l2r`, ...; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `GETD_l2r`, `GETN_l2r`, `SETC_l2r`, `SETTW_l2r`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 955-973
```tablegen
def SETPS_l2r : _FLR2R<0b0001111000, (outs), (ins GRRegs:$src1, GRRegs:$src2),
                       "set ps[$src1], $src2",
                       [(int_xcore_setps GRRegs:$src1, GRRegs:$src2)]>;

def INITLR_l2r : _FL2R<0b0001011000, (outs), (ins GRRegs:$src, GRRegs:$t),
                       "init t[$t]:lr, $src",
                       [(int_xcore_initlr GRRegs:$t, GRRegs:$src)]>;

def SETCLK_l2r : _FLR2R<0b0000111001, (outs), (ins GRRegs:$src1, GRRegs:$src2),
                        "setclk res[$src1], $src2",
                        [(int_xcore_setclk GRRegs:$src1, GRRegs:$src2)]>;

def SETN_l2r : _FLR2R<0b0011011000, (outs), (ins GRRegs:$src1, GRRegs:$src2),
                      "setn res[$src1], $src2", []>;

def SETRDY_l2r : _FLR2R<0b0010111000, (outs), (ins GRRegs:$src1, GRRegs:$src2),
                        "setrdy res[$src1], $src2",
                        [(int_xcore_setrdy GRRegs:$src1, GRRegs:$src2)]>;

```
- **EN**: Declares TableGen records such as `SETPS_l2r`, `INITLR_l2r`, `SETCLK_l2r`, `SETN_l2r`, ...; this block attaches SelectionDAG patterns.
- **CN**: 声明了 `SETPS_l2r`, `INITLR_l2r`, `SETCLK_l2r`, `SETN_l2r`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式。

### Lines 974-989
```tablegen
def TESTLCL_l2r : _FL2R<0b0010011000, (outs GRRegs:$dst), (ins GRRegs:$src),
                        "testlcl $dst, res[$src]", []>;

// One operand short
def MSYNC_1r : _F1R<0b000111, (outs), (ins GRRegs:$a),
                    "msync res[$a]",
                    [(int_xcore_msync GRRegs:$a)]>;
def MJOIN_1r : _F1R<0b000101, (outs), (ins GRRegs:$a),
                    "mjoin res[$a]",
                    [(int_xcore_mjoin GRRegs:$a)]>;

let isBranch=1, isIndirectBranch=1, isTerminator=1, isBarrier = 1 in
def BAU_1r : _F1R<0b001001, (outs), (ins GRRegs:$a),
                 "bau $a",
                 [(brind GRRegs:$a)]>;

```
- **EN**: Declares TableGen records such as `TESTLCL_l2r`, `MSYNC_1r`, `MJOIN_1r`, `BAU_1r`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `TESTLCL_l2r`, `MSYNC_1r`, `MJOIN_1r`, `BAU_1r` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 990-1005
```tablegen
let isBranch=1, isIndirectBranch=1, isTerminator=1, isBarrier = 1 in
def BR_JT : PseudoInstXCore<(outs), (ins InlineJT:$t, GRRegs:$i),
                            "bru $i\n$t",
                            [(XCoreBR_JT tjumptable:$t, GRRegs:$i)]>;

let isBranch=1, isIndirectBranch=1, isTerminator=1, isBarrier = 1 in
def BR_JT32 : PseudoInstXCore<(outs), (ins InlineJT32:$t, GRRegs:$i),
                              "bru $i\n$t",
                              [(XCoreBR_JT32 tjumptable:$t, GRRegs:$i)]>;

let isBranch=1, isIndirectBranch=1, isTerminator=1, isBarrier = 1 in
def BRU_1r : _F1R<0b001010, (outs), (ins GRRegs:$a), "bru $a", []>;

let Defs=[SP], hasSideEffects=0 in
def SETSP_1r : _F1R<0b001011, (outs), (ins GRRegs:$a), "set sp, $a", []>;

```
- **EN**: Declares TableGen records such as `BR_JT`, `BR_JT32`, `BRU_1r`, `SETSP_1r`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `BR_JT`, `BR_JT32`, `BRU_1r`, `SETSP_1r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 1006-1021
```tablegen
let hasSideEffects=0 in
def SETDP_1r : _F1R<0b001100, (outs), (ins GRRegs:$a), "set dp, $a", []>;

let hasSideEffects=0 in
def SETCP_1r : _F1R<0b001101, (outs), (ins GRRegs:$a), "set cp, $a", []>;

let hasCtrlDep = 1 in
def ECALLT_1r : _F1R<0b010011, (outs), (ins GRRegs:$a),
                 "ecallt $a",
                 []>;

let hasCtrlDep = 1 in
def ECALLF_1r : _F1R<0b010010, (outs), (ins GRRegs:$a),
                 "ecallf $a",
                 []>;

```
- **EN**: Declares TableGen records such as `SETDP_1r`, `SETCP_1r`, `ECALLT_1r`, `ECALLF_1r`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `SETDP_1r`, `SETCP_1r`, `ECALLT_1r`, `ECALLF_1r` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 1022-1037
```tablegen
let isCall=1,
// All calls clobber the link register and the non-callee-saved registers:
Defs = [R0, R1, R2, R3, R11, LR], Uses = [SP] in {
def BLA_1r : _F1R<0b001000, (outs), (ins GRRegs:$a),
                 "bla $a",
                 [(XCoreBranchLink GRRegs:$a)]>;
}

def SYNCR_1r : _F1R<0b100001, (outs), (ins GRRegs:$a),
                 "syncr res[$a]",
                 [(int_xcore_syncr GRRegs:$a)]>;

def FREER_1r : _F1R<0b000100, (outs), (ins GRRegs:$a),
               "freer res[$a]",
               [(int_xcore_freer GRRegs:$a)]>;

```
- **EN**: Declares TableGen records such as `BLA_1r`, `SYNCR_1r`, `FREER_1r`; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `BLA_1r`, `SYNCR_1r`, `FREER_1r` 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 1038-1056
```tablegen
let Uses=[R11] in {
def SETV_1r : _F1R<0b010001, (outs), (ins GRRegs:$a),
                   "setv res[$a], r11",
                   [(int_xcore_setv GRRegs:$a, R11)]>;

def SETEV_1r : _F1R<0b001111, (outs), (ins GRRegs:$a),
                    "setev res[$a], r11",
                    [(int_xcore_setev GRRegs:$a, R11)]>;
}

def DGETREG_1r : _F1R<0b001110, (outs GRRegs:$a), (ins), "dgetreg $a", []>;

def EDU_1r : _F1R<0b000000, (outs), (ins GRRegs:$a), "edu res[$a]",
                  [(int_xcore_edu GRRegs:$a)]>;

def EEU_1r : _F1R<0b000001, (outs), (ins GRRegs:$a),
               "eeu res[$a]",
               [(int_xcore_eeu GRRegs:$a)]>;

```
- **EN**: Declares TableGen records such as `SETV_1r`, `SETEV_1r`, `DGETREG_1r`, `EDU_1r`, ...; this block specializes shared fields with `let` assignments.
- **CN**: 声明了 `SETV_1r`, `SETEV_1r`, `DGETREG_1r`, `EDU_1r`, ... 等 TableGen 记录；这一段同时通过 `let` 语句特化共享字段。

### Lines 1057-1073
```tablegen
def KCALL_1r : _F1R<0b010000, (outs), (ins GRRegs:$a), "kcall $a", []>;

def WAITEF_1R : _F1R<0b000011, (outs), (ins GRRegs:$a), "waitef $a", []>;

def WAITET_1R : _F1R<0b000010, (outs), (ins GRRegs:$a), "waitet $a", []>;

def TSTART_1R : _F1R<0b000110, (outs), (ins GRRegs:$a), "start t[$a]", []>;

def CLRPT_1R : _F1R<0b100000, (outs), (ins GRRegs:$a), "clrpt res[$a]",
                    [(int_xcore_clrpt GRRegs:$a)]>;

// Zero operand short

def CLRE_0R : _F0R<0b0000001101, (outs), (ins), "clre", [(int_xcore_clre)]>;

def DCALL_0R : _F0R<0b0000011100, (outs), (ins), "dcall", []>;

```
- **EN**: Declares TableGen records such as `KCALL_1r`, `WAITEF_1R`, `WAITET_1R`, `TSTART_1R`, ... for the backend description.
- **CN**: 为后端描述声明了 `KCALL_1r`, `WAITEF_1R`, `WAITET_1R`, `TSTART_1R`, ... 等 TableGen 记录。

### Lines 1074-1092
```tablegen
let Defs = [SP], Uses = [SP] in
def DENTSP_0R : _F0R<0b0001001100, (outs), (ins), "dentsp", []>;

let Defs = [SP] in
def DRESTSP_0R : _F0R<0b0001001101, (outs), (ins), "drestsp", []>;

def DRET_0R : _F0R<0b0000011110, (outs), (ins), "dret", []>;

def FREET_0R : _F0R<0b0000001111, (outs), (ins), "freet", []>;

let Defs = [R11] in {
def GETID_0R : _F0R<0b0001001110, (outs), (ins),
                    "get r11, id",
                    [(set R11, (int_xcore_getid))]>;

def GETED_0R : _F0R<0b0000111110, (outs), (ins),
                    "get r11, ed",
                    [(set R11, (int_xcore_geted))]>;

```
- **EN**: Declares TableGen records such as `DENTSP_0R`, `DRESTSP_0R`, `DRET_0R`, `FREET_0R`, ...; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `DENTSP_0R`, `DRESTSP_0R`, `DRET_0R`, `FREET_0R`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 1093-1109
```tablegen
def GETET_0R : _F0R<0b0000111111, (outs), (ins),
                    "get r11, et",
                    [(set R11, (int_xcore_getet))]>;

def GETKEP_0R : _F0R<0b0001001111, (outs), (ins),
                     "get r11, kep", []>;

def GETKSP_0R : _F0R<0b0001011100, (outs), (ins),
                     "get r11, ksp", []>;
}

let Defs = [SP] in
def KRET_0R : _F0R<0b0000011101, (outs), (ins), "kret", []>;

let Uses = [SP], mayLoad = 1 in {
def LDET_0R : _F0R<0b0001011110, (outs), (ins), "ldw et, sp[4]", []>;

```
- **EN**: Declares TableGen records such as `GETET_0R`, `GETKEP_0R`, `GETKSP_0R`, `KRET_0R`, ...; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `GETET_0R`, `GETKEP_0R`, `GETKSP_0R`, `KRET_0R`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 1110-1126
```tablegen
def LDSED_0R : _F0R<0b0001011101, (outs), (ins), "ldw sed, sp[3]", []>;

def LDSPC_0R : _F0R<0b0000101100, (outs), (ins), "ldw spc, sp[1]", []>;

def LDSSR_0R : _F0R<0b0000101110, (outs), (ins), "ldw ssr, sp[2]", []>;
}

let Uses=[R11] in
def SETKEP_0R : _F0R<0b0000011111, (outs), (ins), "set kep, r11", []>;

def SSYNC_0r : _F0R<0b0000001110, (outs), (ins),
                    "ssync",
                    [(int_xcore_ssync)]>;

let Uses = [SP], mayStore = 1 in {
def STET_0R : _F0R<0b0000111101, (outs), (ins), "stw et, sp[4]", []>;

```
- **EN**: Declares TableGen records such as `LDSED_0R`, `LDSPC_0R`, `LDSSR_0R`, `SETKEP_0R`, ...; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `LDSED_0R`, `LDSPC_0R`, `LDSSR_0R`, `SETKEP_0R`, ... 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 1127-1142
```tablegen
def STSED_0R : _F0R<0b0000111100, (outs), (ins), "stw sed, sp[3]", []>;

def STSPC_0R : _F0R<0b0000101101, (outs), (ins), "stw spc, sp[1]", []>;

def STSSR_0R : _F0R<0b0000101111, (outs), (ins), "stw ssr, sp[2]", []>;
}

let isBranch=1, isIndirectBranch=1, isTerminator=1, isBarrier = 1,
    hasSideEffects = 1 in
def WAITEU_0R : _F0R<0b0000001100, (outs), (ins),
                     "waiteu",
                     [(brind (int_xcore_waitevent))]>;

//===----------------------------------------------------------------------===//
// Non-Instruction Patterns
//===----------------------------------------------------------------------===//
```
- **EN**: Declares TableGen records such as `STSED_0R`, `STSPC_0R`, `STSSR_0R`, `WAITEU_0R`; this block attaches SelectionDAG patterns; specializes shared fields with `let` assignments.
- **CN**: 声明了 `STSED_0R`, `STSPC_0R`, `STSSR_0R`, `WAITEU_0R` 等 TableGen 记录；这一段同时挂接 SelectionDAG 模式、通过 `let` 语句特化共享字段。

### Lines 1143-1159
```tablegen

def : Pat<(XCoreBranchLink texternalsym:$addr), (BLRF_lu10 texternalsym:$addr)>;

/// sext_inreg
def : Pat<(sext_inreg GRRegs:$b, i1), (SEXT_rus GRRegs:$b, 1)>;
def : Pat<(sext_inreg GRRegs:$b, i8), (SEXT_rus GRRegs:$b, 8)>;
def : Pat<(sext_inreg GRRegs:$b, i16), (SEXT_rus GRRegs:$b, 16)>;

/// loads
def : Pat<(zextloadi8 (add GRRegs:$addr, GRRegs:$offset)),
          (LD8U_3r GRRegs:$addr, GRRegs:$offset)>;
def : Pat<(zextloadi8 GRRegs:$addr), (LD8U_3r GRRegs:$addr, (LDC_ru6 0))>;

def : Pat<(sextloadi16 (lda16f GRRegs:$addr, GRRegs:$offset)),
          (LD16S_3r GRRegs:$addr, GRRegs:$offset)>;
def : Pat<(sextloadi16 GRRegs:$addr), (LD16S_3r GRRegs:$addr, (LDC_ru6 0))>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 1160-1179
```tablegen
def : Pat<(load (ldawf GRRegs:$addr, GRRegs:$offset)),
          (LDW_3r GRRegs:$addr, GRRegs:$offset)>;
def : Pat<(load (add GRRegs:$addr, immUs4:$offset)),
          (LDW_2rus GRRegs:$addr, (div4_xform immUs4:$offset))>;
def : Pat<(load GRRegs:$addr), (LDW_2rus GRRegs:$addr, 0)>;

/// anyext
def : Pat<(extloadi8 (add GRRegs:$addr, GRRegs:$offset)),
          (LD8U_3r GRRegs:$addr, GRRegs:$offset)>;
def : Pat<(extloadi8 GRRegs:$addr), (LD8U_3r GRRegs:$addr, (LDC_ru6 0))>;
def : Pat<(extloadi16 (lda16f GRRegs:$addr, GRRegs:$offset)),
          (LD16S_3r GRRegs:$addr, GRRegs:$offset)>;
def : Pat<(extloadi16 GRRegs:$addr), (LD16S_3r GRRegs:$addr, (LDC_ru6 0))>;

/// stores
def : Pat<(truncstorei8 GRRegs:$val, (add GRRegs:$addr, GRRegs:$offset)),
          (ST8_l3r GRRegs:$val, GRRegs:$addr, GRRegs:$offset)>;
def : Pat<(truncstorei8 GRRegs:$val, GRRegs:$addr),
          (ST8_l3r GRRegs:$val, GRRegs:$addr, (LDC_ru6 0))>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 1180-1195
```tablegen
def : Pat<(truncstorei16 GRRegs:$val, (lda16f GRRegs:$addr, GRRegs:$offset)),
          (ST16_l3r GRRegs:$val, GRRegs:$addr, GRRegs:$offset)>;
def : Pat<(truncstorei16 GRRegs:$val, GRRegs:$addr),
          (ST16_l3r GRRegs:$val, GRRegs:$addr, (LDC_ru6 0))>;

def : Pat<(store GRRegs:$val, (ldawf GRRegs:$addr, GRRegs:$offset)),
          (STW_l3r GRRegs:$val, GRRegs:$addr, GRRegs:$offset)>;
def : Pat<(store GRRegs:$val, (add GRRegs:$addr, immUs4:$offset)),
          (STW_2rus GRRegs:$val, GRRegs:$addr, (div4_xform immUs4:$offset))>;
def : Pat<(store GRRegs:$val, GRRegs:$addr),
          (STW_2rus GRRegs:$val, GRRegs:$addr, 0)>;

/// bitrev
def : Pat<(bitreverse GRRegs:$src), (BITREV_l2r GRRegs:$src)>;

/// cttz
```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 1196-1213
```tablegen
def : Pat<(cttz GRRegs:$src), (CLZ_l2r (BITREV_l2r GRRegs:$src))>;

/// trap
def : Pat<(trap), (ECALLF_1r (LDC_ru6 0))>;

///
/// branch patterns
///

// unconditional branch
def : Pat<(br bb:$addr), (BRFU_lu6 bb:$addr)>;

// direct match equal/notequal zero brcond
def : Pat<(brcond (setne GRRegs:$lhs, 0), bb:$dst),
          (BRFT_lru6 GRRegs:$lhs, bb:$dst)>;
def : Pat<(brcond (seteq GRRegs:$lhs, 0), bb:$dst),
          (BRFF_lru6 GRRegs:$lhs, bb:$dst)>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 1214-1229
```tablegen
def : Pat<(brcond (setle GRRegs:$lhs, GRRegs:$rhs), bb:$dst),
          (BRFF_lru6 (LSS_3r GRRegs:$rhs, GRRegs:$lhs), bb:$dst)>;
def : Pat<(brcond (setule GRRegs:$lhs, GRRegs:$rhs), bb:$dst),
          (BRFF_lru6 (LSU_3r GRRegs:$rhs, GRRegs:$lhs), bb:$dst)>;
def : Pat<(brcond (setge GRRegs:$lhs, GRRegs:$rhs), bb:$dst),
          (BRFF_lru6 (LSS_3r GRRegs:$lhs, GRRegs:$rhs), bb:$dst)>;
def : Pat<(brcond (setuge GRRegs:$lhs, GRRegs:$rhs), bb:$dst),
          (BRFF_lru6 (LSU_3r GRRegs:$lhs, GRRegs:$rhs), bb:$dst)>;
def : Pat<(brcond (setne GRRegs:$lhs, GRRegs:$rhs), bb:$dst),
          (BRFF_lru6 (EQ_3r GRRegs:$lhs, GRRegs:$rhs), bb:$dst)>;
def : Pat<(brcond (setne GRRegs:$lhs, immUs:$rhs), bb:$dst),
          (BRFF_lru6 (EQ_2rus GRRegs:$lhs, immUs:$rhs), bb:$dst)>;

// generic brcond pattern
def : Pat<(brcond GRRegs:$cond, bb:$addr), (BRFT_lru6 GRRegs:$cond, bb:$addr)>;

```
- **EN**: Implements logic around `Pat<`, `BRFF_lru6`.
- **CN**: 围绕 `Pat<`, `BRFF_lru6` 实现具体逻辑。

### Lines 1230-1254
```tablegen

///
/// Select patterns
///

// direct match equal/notequal zero select
def : Pat<(select (setne GRRegs:$lhs, 0), GRRegs:$T, GRRegs:$F),
        (SELECT_CC GRRegs:$lhs, GRRegs:$T, GRRegs:$F)>;

def : Pat<(select (seteq GRRegs:$lhs, 0), GRRegs:$T, GRRegs:$F),
        (SELECT_CC GRRegs:$lhs, GRRegs:$F, GRRegs:$T)>;

def : Pat<(select (setle GRRegs:$lhs, GRRegs:$rhs), GRRegs:$T, GRRegs:$F),
          (SELECT_CC (LSS_3r GRRegs:$rhs, GRRegs:$lhs), GRRegs:$F, GRRegs:$T)>;
def : Pat<(select (setule GRRegs:$lhs, GRRegs:$rhs), GRRegs:$T, GRRegs:$F),
          (SELECT_CC (LSU_3r GRRegs:$rhs, GRRegs:$lhs), GRRegs:$F, GRRegs:$T)>;
def : Pat<(select (setge GRRegs:$lhs, GRRegs:$rhs), GRRegs:$T, GRRegs:$F),
          (SELECT_CC (LSS_3r GRRegs:$lhs, GRRegs:$rhs), GRRegs:$F, GRRegs:$T)>;
def : Pat<(select (setuge GRRegs:$lhs, GRRegs:$rhs), GRRegs:$T, GRRegs:$F),
          (SELECT_CC (LSU_3r GRRegs:$lhs, GRRegs:$rhs), GRRegs:$F, GRRegs:$T)>;
def : Pat<(select (setne GRRegs:$lhs, GRRegs:$rhs), GRRegs:$T, GRRegs:$F),
          (SELECT_CC (EQ_3r GRRegs:$lhs, GRRegs:$rhs), GRRegs:$F, GRRegs:$T)>;
def : Pat<(select (setne GRRegs:$lhs, immUs:$rhs), GRRegs:$T, GRRegs:$F),
          (SELECT_CC (EQ_2rus GRRegs:$lhs, immUs:$rhs), GRRegs:$F, GRRegs:$T)>;

```
- **EN**: Implements logic around `Pat<`, `SELECT_CC`.
- **CN**: 围绕 `Pat<`, `SELECT_CC` 实现具体逻辑。

### Lines 1255-1270
```tablegen
///
/// setcc patterns, only matched when none of the above brcond
/// patterns match
///

// setcc 2 register operands
def : Pat<(setle GRRegs:$lhs, GRRegs:$rhs),
          (EQ_2rus (LSS_3r GRRegs:$rhs, GRRegs:$lhs), 0)>;
def : Pat<(setule GRRegs:$lhs, GRRegs:$rhs),
          (EQ_2rus (LSU_3r GRRegs:$rhs, GRRegs:$lhs), 0)>;

def : Pat<(setgt GRRegs:$lhs, GRRegs:$rhs),
          (LSS_3r GRRegs:$rhs, GRRegs:$lhs)>;
def : Pat<(setugt GRRegs:$lhs, GRRegs:$rhs),
          (LSU_3r GRRegs:$rhs, GRRegs:$lhs)>;

```
- **EN**: Implements logic around `Pat<`, `EQ_2rus`.
- **CN**: 围绕 `Pat<`, `EQ_2rus` 实现具体逻辑。

### Lines 1271-1286
```tablegen
def : Pat<(setge GRRegs:$lhs, GRRegs:$rhs),
          (EQ_2rus (LSS_3r GRRegs:$lhs, GRRegs:$rhs), 0)>;
def : Pat<(setuge GRRegs:$lhs, GRRegs:$rhs),
          (EQ_2rus (LSU_3r GRRegs:$lhs, GRRegs:$rhs), 0)>;

def : Pat<(setlt GRRegs:$lhs, GRRegs:$rhs),
          (LSS_3r GRRegs:$lhs, GRRegs:$rhs)>;
def : Pat<(setult GRRegs:$lhs, GRRegs:$rhs),
          (LSU_3r GRRegs:$lhs, GRRegs:$rhs)>;

def : Pat<(setne GRRegs:$lhs, GRRegs:$rhs),
          (EQ_2rus (EQ_3r GRRegs:$lhs, GRRegs:$rhs), 0)>;

def : Pat<(seteq GRRegs:$lhs, GRRegs:$rhs),
          (EQ_3r GRRegs:$lhs, GRRegs:$rhs)>;

```
- **EN**: Implements logic around `Pat<`, `EQ_2rus`.
- **CN**: 围绕 `Pat<`, `EQ_2rus` 实现具体逻辑。

### Lines 1287-1302
```tablegen
// setcc reg/imm operands
def : Pat<(seteq GRRegs:$lhs, immUs:$rhs),
          (EQ_2rus GRRegs:$lhs, immUs:$rhs)>;
def : Pat<(setne GRRegs:$lhs, immUs:$rhs),
          (EQ_2rus (EQ_2rus GRRegs:$lhs, immUs:$rhs), 0)>;

// misc
def : Pat<(add GRRegs:$addr, immUs4:$offset),
          (LDAWF_l2rus GRRegs:$addr, (div4_xform immUs4:$offset))>;

def : Pat<(sub GRRegs:$addr, immUs4:$offset),
          (LDAWB_l2rus GRRegs:$addr, (div4_xform immUs4:$offset))>;

def : Pat<(and GRRegs:$val, immMskBitp:$mask),
          (ZEXT_rus GRRegs:$val, (msksize_xform immMskBitp:$mask))>;

```
- **EN**: Implements logic around `Pat<`, `EQ_2rus`.
- **CN**: 围绕 `Pat<`, `EQ_2rus` 实现具体逻辑。

### Lines 1303-1319
```tablegen
// (sub X, imm) gets canonicalized to (add X, -imm).  Match this form.
def : Pat<(add GRRegs:$src1, immUsNeg:$src2),
          (SUB_2rus GRRegs:$src1, (neg_xform immUsNeg:$src2))>;

def : Pat<(add GRRegs:$src1, immUs4Neg:$src2),
          (LDAWB_l2rus GRRegs:$src1, (div4neg_xform immUs4Neg:$src2))>;

///
/// Some peepholes
///

def : Pat<(mul GRRegs:$src, 3),
          (LDA16F_l3r GRRegs:$src, GRRegs:$src)>;

def : Pat<(mul GRRegs:$src, 5),
          (LDAWF_l3r GRRegs:$src, GRRegs:$src)>;

```
- **EN**: Implements logic around `Pat<`.
- **CN**: 围绕 `Pat<` 实现具体逻辑。

### Lines 1320-1336
```tablegen
def : Pat<(mul GRRegs:$src, -3),
          (LDAWB_l3r GRRegs:$src, GRRegs:$src)>;

// ashr X, 32 is equivalent to ashr X, 31 on the XCore.
def : Pat<(sra GRRegs:$src, 31),
          (ASHR_l2rus GRRegs:$src, 32)>;

def : Pat<(brcond (setlt GRRegs:$lhs, 0), bb:$dst),
          (BRFT_lru6 (ASHR_l2rus GRRegs:$lhs, 32), bb:$dst)>;

// setge X, 0 is canonicalized to setgt X, -1
def : Pat<(brcond (setgt GRRegs:$lhs, -1), bb:$dst),
          (BRFF_lru6 (ASHR_l2rus GRRegs:$lhs, 32), bb:$dst)>;

def : Pat<(select (setlt GRRegs:$lhs, 0), GRRegs:$T, GRRegs:$F),
          (SELECT_CC (ASHR_l2rus GRRegs:$lhs, 32), GRRegs:$T, GRRegs:$F)>;

```
- **EN**: Implements logic around `Pat<`, `BRFT_lru6`, `BRFF_lru6`, `SELECT_CC`.
- **CN**: 围绕 `Pat<`, `BRFT_lru6`, `BRFF_lru6`, `SELECT_CC` 实现具体逻辑。

### Lines 1337-1350
```tablegen
def : Pat<(select (setgt GRRegs:$lhs, -1), GRRegs:$T, GRRegs:$F),
          (SELECT_CC (ASHR_l2rus GRRegs:$lhs, 32), GRRegs:$F, GRRegs:$T)>;

def : Pat<(setgt GRRegs:$lhs, -1),
          (EQ_2rus (ASHR_l2rus GRRegs:$lhs, 32), 0)>;

def : Pat<(sra (shl GRRegs:$src, immBpwSubBitp:$imm), immBpwSubBitp:$imm),
          (SEXT_rus GRRegs:$src, (bpwsub_xform immBpwSubBitp:$imm))>;

def : Pat<(load (cprelwrapper tconstpool:$b)),
          (LDWCP_lru6 tconstpool:$b)>;

def : Pat<(cprelwrapper tconstpool:$b),
          (LDAWCP_lu6 tconstpool:$b)>;
```
- **EN**: Implements logic around `Pat<`, `SELECT_CC`, `EQ_2rus`.
- **CN**: 围绕 `Pat<`, `SELECT_CC`, `EQ_2rus` 实现具体逻辑。

## Key Concepts / 关键概念

- **Instruction semantics / 指令语义**:
  - **EN**: Encodes instruction behavior and helper routines
  - **CN**: 描述指令行为与辅助例程
- **TableGen DSL / TableGen 领域专用语言**:
  - **EN**: Declarative backend description expanded into generated tables
  - **CN**: 以声明式方式描述后端并生成表格代码
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式
- **Generated backend tables / 生成的后端表**:
  - **EN**: The records here feed TableGen generators that emit `.inc` matcher/data tables
  - **CN**: 此处记录会喂给 TableGen 生成器，产出 `.inc` 匹配器/数据表

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XCoreInstrFormats.td`
- **LLVM subsystems / LLVM 子系统**: TableGen generators
