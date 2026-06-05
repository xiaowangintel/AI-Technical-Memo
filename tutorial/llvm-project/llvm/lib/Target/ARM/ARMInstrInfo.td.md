# ARMInstrInfo.td — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMInstrInfo.td`
- Repository: `llvm-project`
- Purpose (EN): This file describes the ARM instructions in TableGen format.
- 用途 (CN): 使用 LLVM TableGen DSL 定义 ARM 后端中的 `ARMInstrInfo`，涵盖指令语义、调度提示以及机器级辅助逻辑。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```tablegen
//===- ARMInstrInfo.td - Target Description for ARM Target -*- tablegen -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file describes the ARM instructions in TableGen format.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 13-15
```tablegen
//===----------------------------------------------------------------------===//
// ARM specific DAG Nodes.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 17-18
```tablegen
/// Value type used for "condition code" operands.
defvar CondCodeVT = i32;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 20-21
```tablegen
/// Value type used for "flags" operands / results (either CPSR or FPSCR_NZCV).
defvar FlagsVT = i32;
```
- EN: Introduces a reusable TableGen variable so later records can share a common symbolic value or type.
- CN: 这里引入可复用的 TableGen 变量，便于后续记录共享统一的符号值或类型。

### Lines 23-29
```tablegen
// Type profiles.
def SDT_ARMCallSeqStart : SDCallSeqStart<[ SDTCisVT<0, i32>,
                                           SDTCisVT<1, i32> ]>;
def SDT_ARMCallSeqEnd   : SDCallSeqEnd<[ SDTCisVT<0, i32>, SDTCisVT<1, i32> ]>;
def SDT_ARMStructByVal : SDTypeProfile<0, 4,
                                       [SDTCisVT<0, i32>, SDTCisVT<1, i32>,
                                        SDTCisVT<2, i32>, SDTCisVT<3, i32>]>;
```
- EN: Defines TableGen record `SDT_ARMCallSeqStart` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMCallSeqStart`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 31-31
```tablegen
def SDT_ARMSaveCallPC : SDTypeProfile<0, 1, []>;
```
- EN: Defines TableGen record `SDT_ARMSaveCallPC` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMSaveCallPC`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 33-33
```tablegen
def SDT_ARMcall    : SDTypeProfile<0, -1, [SDTCisPtrTy<0>]>;
```
- EN: Defines TableGen record `SDT_ARMcall` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMcall`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 35-41
```tablegen
def SDT_ARMCMov : SDTypeProfile<1, 4, [
  /* any */                // result
  SDTCisSameAs<1, 0>,      // value on false
  SDTCisSameAs<2, 0>,      // value on true
  SDTCisVT<3, CondCodeVT>, // condition code
  SDTCisVT<4, FlagsVT>,    // in flags
]>;
```
- EN: Defines TableGen record `SDT_ARMCMov` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMCMov`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 43-47
```tablegen
def SDT_ARMBrcond : SDTypeProfile<0, 3, [
  SDTCisVT<0, OtherVT>,    // target basic block
  SDTCisVT<1, CondCodeVT>, // condition code
  SDTCisVT<2, FlagsVT>,    // in flags
]>;
```
- EN: Defines TableGen record `SDT_ARMBrcond` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMBrcond`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 49-50
```tablegen
def SDT_ARMBrJT    : SDTypeProfile<0, 2,
                                  [SDTCisPtrTy<0>, SDTCisVT<1, i32>]>;
```
- EN: Defines TableGen record `SDT_ARMBrJT` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMBrJT`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 52-54
```tablegen
def SDT_ARMBr2JT   : SDTypeProfile<0, 3,
                                  [SDTCisPtrTy<0>, SDTCisVT<1, i32>,
                                   SDTCisVT<2, i32>]>;
```
- EN: Defines TableGen record `SDT_ARMBr2JT` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMBr2JT`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 56-60
```tablegen
def SDT_ARMBCC_i64 : SDTypeProfile<0, 6,
                                  [SDTCisVT<0, i32>,
                                   SDTCisVT<1, i32>, SDTCisVT<2, i32>,
                                   SDTCisVT<3, i32>, SDTCisVT<4, i32>,
                                   SDTCisVT<5, OtherVT>]>;
```
- EN: Defines TableGen record `SDT_ARMBCC_i64` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMBCC_i64`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 62-64
```tablegen
def SDT_ARMAnd     : SDTypeProfile<1, 2,
                                   [SDTCisVT<0, i32>, SDTCisVT<1, i32>,
                                    SDTCisVT<2, i32>]>;
```
- EN: Defines TableGen record `SDT_ARMAnd` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMAnd`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 66-70
```tablegen
def SDT_ARMCmp : SDTypeProfile<1, 2, [
  SDTCisVT<0, FlagsVT>, // out flags
  SDTCisInt<1>,         // lhs
  SDTCisSameAs<2, 1>    // rhs
]>;
```
- EN: Defines TableGen record `SDT_ARMCmp` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMCmp`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 72-73
```tablegen
def SDT_ARMPICAdd  : SDTypeProfile<1, 2, [SDTCisSameAs<0, 1>,
                                          SDTCisPtrTy<1>, SDTCisVT<2, i32>]>;
```
- EN: Defines TableGen record `SDT_ARMPICAdd` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMPICAdd`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 75-79
```tablegen
def SDT_ARMThreadPointer : SDTypeProfile<1, 0, [SDTCisPtrTy<0>]>;
def SDT_ARMEH_SJLJ_Setjmp : SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisPtrTy<1>,
                                                 SDTCisInt<2>]>;
def SDT_ARMEH_SJLJ_Longjmp: SDTypeProfile<0, 2, [SDTCisPtrTy<0>, SDTCisInt<1>]>;
def SDT_ARMEH_SJLJ_SetupDispatch: SDTypeProfile<0, 0, []>;
```
- EN: Defines TableGen record `SDT_ARMThreadPointer` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMThreadPointer`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 81-81
```tablegen
def SDT_ARMMEMBARRIER     : SDTypeProfile<0, 1, [SDTCisInt<0>]>;
```
- EN: Defines TableGen record `SDT_ARMMEMBARRIER` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMMEMBARRIER`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 83-84
```tablegen
def SDT_ARMPREFETCH : SDTypeProfile<0, 3, [SDTCisPtrTy<0>, SDTCisSameAs<1, 2>,
                                           SDTCisInt<1>]>;
```
- EN: Defines TableGen record `SDT_ARMPREFETCH` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMPREFETCH`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 86-86
```tablegen
def SDT_ARMTCRET : SDTypeProfile<0, 2, [SDTCisPtrTy<0>]>;
```
- EN: Defines TableGen record `SDT_ARMTCRET` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMTCRET`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 88-89
```tablegen
def SDT_ARMBFI : SDTypeProfile<1, 3, [SDTCisVT<0, i32>, SDTCisVT<1, i32>,
                                      SDTCisVT<2, i32>, SDTCisVT<3, i32>]>;
```
- EN: Defines TableGen record `SDT_ARMBFI` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMBFI`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 91-91
```tablegen
def SDT_WIN__DBZCHK : SDTypeProfile<0, 1, [SDTCisVT<0, i32>]>;
```
- EN: Defines TableGen record `SDT_WIN__DBZCHK` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_WIN__DBZCHK`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 93-95
```tablegen
def SDT_ARMMEMCPY  : SDTypeProfile<2, 3, [SDTCisVT<0, i32>, SDTCisVT<1, i32>,
                                          SDTCisVT<2, i32>, SDTCisVT<3, i32>,
                                          SDTCisVT<4, i32>]>;
```
- EN: Defines TableGen record `SDT_ARMMEMCPY` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMMEMCPY`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 97-101
```tablegen
def SDTIntUnaryOpWithFlagsOut : SDTypeProfile<2, 1, [
  SDTCisInt<0>,         // result
  SDTCisVT<1, FlagsVT>, // out flags
  SDTCisSameAs<2, 0>    // operand
]>;
```
- EN: Defines TableGen record `SDTIntUnaryOpWithFlagsOut` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTIntUnaryOpWithFlagsOut`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 103-107
```tablegen
def SDTIntUnaryOpWithFlagsIn : SDTypeProfile<1, 2, [
  SDTCisInt<0>,        // result
  SDTCisSameAs<1, 0>,  // operand
  SDTCisVT<1, FlagsVT> // in flags
]>;
```
- EN: Defines TableGen record `SDTIntUnaryOpWithFlagsIn` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTIntUnaryOpWithFlagsIn`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 109-112
```tablegen
def SDTBinaryArithWithFlags : SDTypeProfile<2, 2,
                                            [SDTCisSameAs<0, 2>,
                                             SDTCisSameAs<0, 3>,
                                             SDTCisInt<0>, SDTCisVT<1, i32>]>;
```
- EN: Defines TableGen record `SDTBinaryArithWithFlags` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTBinaryArithWithFlags`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 114-120
```tablegen
// SDTBinaryArithWithFlagsInOut - RES1, CPSR = op LHS, RHS, CPSR
def SDTBinaryArithWithFlagsInOut : SDTypeProfile<2, 3,
                                            [SDTCisSameAs<0, 2>,
                                             SDTCisSameAs<0, 3>,
                                             SDTCisInt<0>,
                                             SDTCisVT<1, i32>,
                                             SDTCisVT<4, i32>]>;
```
- EN: Defines TableGen record `SDTBinaryArithWithFlagsInOut` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTBinaryArithWithFlagsInOut`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 122-127
```tablegen
def SDT_LongMac  : SDTypeProfile<2, 4, [SDTCisVT<0, i32>,
                                        SDTCisSameAs<0, 1>,
                                        SDTCisSameAs<0, 2>,
                                        SDTCisSameAs<0, 3>,
                                        SDTCisSameAs<0, 4>,
                                        SDTCisSameAs<0, 5>]>;
```
- EN: Defines TableGen record `SDT_LongMac` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_LongMac`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 129-134
```tablegen
// ARMlsll, ARMlsrl, ARMasrl
def SDT_ARMIntShiftParts : SDTypeProfile<2, 3, [SDTCisSameAs<0, 1>,
                                              SDTCisSameAs<0, 2>,
                                              SDTCisSameAs<0, 3>,
                                              SDTCisInt<0>,
                                              SDTCisInt<4>]>;
```
- EN: Defines TableGen record `SDT_ARMIntShiftParts` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMIntShiftParts`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 136-137
```tablegen
// Signed multiply accumulate long dual
def ARMSmlald        : SDNode<"ARMISD::SMLALD", SDT_LongMac>;
```
- EN: Defines TableGen record `ARMSmlald` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMSmlald`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 139-140
```tablegen
// Signed multiply accumulate long dual exchange
def ARMSmlaldx       : SDNode<"ARMISD::SMLALDX", SDT_LongMac>;
```
- EN: Defines TableGen record `ARMSmlaldx` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMSmlaldx`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 142-143
```tablegen
// Signed multiply subtract long dual
def ARMSmlsld        : SDNode<"ARMISD::SMLSLD", SDT_LongMac>;
```
- EN: Defines TableGen record `ARMSmlsld` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMSmlsld`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 145-146
```tablegen
// Signed multiply subtract long dual exchange
def ARMSmlsldx       : SDNode<"ARMISD::SMLSLDX", SDT_LongMac>;
```
- EN: Defines TableGen record `ARMSmlsldx` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMSmlsldx`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 148-154
```tablegen
def SDT_ARMCSel : SDTypeProfile<1, 4, [
  /* any */                // result
  SDTCisSameAs<1, 0>,      // lhs
  SDTCisSameAs<2, 0>,      // rhs
  SDTCisVT<3, CondCodeVT>, // condition code
  SDTCisVT<3, FlagsVT>     // in flags
]>;
```
- EN: Defines TableGen record `SDT_ARMCSel` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMCSel`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 156-157
```tablegen
// Conditional select invert.
def ARMcsinv : SDNode<"ARMISD::CSINV", SDT_ARMCSel>;
```
- EN: Defines TableGen record `ARMcsinv` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcsinv`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 159-160
```tablegen
// Conditional select negate.
def ARMcsneg : SDNode<"ARMISD::CSNEG", SDT_ARMCSel>;
```
- EN: Defines TableGen record `ARMcsneg` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcsneg`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 162-163
```tablegen
// Conditional select increment.
def ARMcsinc : SDNode<"ARMISD::CSINC", SDT_ARMCSel>;
```
- EN: Defines TableGen record `ARMcsinc` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcsinc`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 165-168
```tablegen
def SDT_MulHSR       : SDTypeProfile<1, 3, [SDTCisVT<0,i32>,
                                            SDTCisSameAs<0, 1>,
                                            SDTCisSameAs<0, 2>,
                                            SDTCisSameAs<0, 3>]>;
```
- EN: Defines TableGen record `SDT_MulHSR` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_MulHSR`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 170-171
```tablegen
// Signed multiply long, round and add
def ARMsmmlar      : SDNode<"ARMISD::SMMLAR", SDT_MulHSR>;
```
- EN: Defines TableGen record `ARMsmmlar` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMsmmlar`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 173-174
```tablegen
// Signed multiply long, subtract and round
def ARMsmmlsr      : SDNode<"ARMISD::SMMLSR", SDT_MulHSR>;
```
- EN: Defines TableGen record `ARMsmmlsr` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMsmmlsr`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 177-179
```tablegen
// Wrapper - A wrapper node for TargetConstantPool,
// TargetExternalSymbol, and TargetGlobalAddress.
def ARMWrapper       : SDNode<"ARMISD::Wrapper",     SDTIntUnaryOp>;
```
- EN: Defines TableGen record `ARMWrapper` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMWrapper`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 181-183
```tablegen
// WrapperPIC - A wrapper node for TargetGlobalAddress in
// PIC mode.
def ARMWrapperPIC    : SDNode<"ARMISD::WrapperPIC",  SDTIntUnaryOp>;
```
- EN: Defines TableGen record `ARMWrapperPIC` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMWrapperPIC`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 185-186
```tablegen
// WrapperJT - A wrapper node for TargetJumpTable
def ARMWrapperJT     : SDNode<"ARMISD::WrapperJT",   SDTIntUnaryOp>;
```
- EN: Defines TableGen record `ARMWrapperJT` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMWrapperJT`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 188-191
```tablegen
def ARMcallseq_start : SDNode<"ISD::CALLSEQ_START", SDT_ARMCallSeqStart,
                              [SDNPHasChain, SDNPOutGlue]>;
def ARMcallseq_end   : SDNode<"ISD::CALLSEQ_END",   SDT_ARMCallSeqEnd,
                              [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue]>;
```
- EN: Defines TableGen record `ARMcallseq_start` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcallseq_start`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 193-197
```tablegen
// Add pseudo op to model memcpy for struct byval.
def ARMcopystructbyval : SDNode<"ARMISD::COPY_STRUCT_BYVAL" ,
                                SDT_ARMStructByVal,
                                [SDNPHasChain, SDNPInGlue, SDNPOutGlue,
                                 SDNPMayStore, SDNPMayLoad]>;
```
- EN: Defines TableGen record `ARMcopystructbyval` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcopystructbyval`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 199-202
```tablegen
// Function call.
def ARMcall          : SDNode<"ARMISD::CALL", SDT_ARMcall,
                              [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                               SDNPVariadic]>;
```
- EN: Defines TableGen record `ARMcall` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcall`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 204-207
```tablegen
// Function call that's predicable.
def ARMcall_pred    : SDNode<"ARMISD::CALL_PRED", SDT_ARMcall,
                              [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                               SDNPVariadic]>;
```
- EN: Defines TableGen record `ARMcall_pred` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcall_pred`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 209-212
```tablegen
// Function call with branch not branch-and-link.
def ARMcall_nolink   : SDNode<"ARMISD::CALL_NOLINK", SDT_ARMcall,
                              [SDNPHasChain, SDNPOptInGlue, SDNPOutGlue,
                               SDNPVariadic]>;
```
- EN: Defines TableGen record `ARMcall_nolink` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcall_nolink`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 214-216
```tablegen
// Return with a flag operand.
def ARMretglue       : SDNode<"ARMISD::RET_GLUE", SDTNone,
                              [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
```
- EN: Defines TableGen record `ARMretglue` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMretglue`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 218-220
```tablegen
// CMSE Entry function return with a flag operand.
def ARMseretglue     : SDNode<"ARMISD::SERET_GLUE", SDTNone,
                              [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
```
- EN: Defines TableGen record `ARMseretglue` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMseretglue`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 222-224
```tablegen
// Interrupt return with an LR-offset and a flag operand.
def ARMintretglue    : SDNode<"ARMISD::INTRET_GLUE", SDT_ARMcall,
                              [SDNPHasChain, SDNPOptInGlue, SDNPVariadic]>;
```
- EN: Defines TableGen record `ARMintretglue` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMintretglue`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 226-227
```tablegen
// ARM conditional move instructions.
def ARMcmov          : SDNode<"ARMISD::CMOV", SDT_ARMCMov>;
```
- EN: Defines TableGen record `ARMcmov` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcmov`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 229-230
```tablegen
// Signed saturation
def ARMssat   : SDNode<"ARMISD::SSAT", SDTIntSatNoShOp, []>;
```
- EN: Defines TableGen record `ARMssat` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMssat`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 232-233
```tablegen
// Unsigned saturation
def ARMusat   : SDNode<"ARMISD::USAT", SDTIntSatNoShOp, []>;
```
- EN: Defines TableGen record `ARMusat` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMusat`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 235-236
```tablegen
// Conditional branch.
def ARMbrcond        : SDNode<"ARMISD::BRCOND", SDT_ARMBrcond, [SDNPHasChain]>;
```
- EN: Defines TableGen record `ARMbrcond` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMbrcond`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 238-240
```tablegen
// Jumptable branch.
def ARMbrjt          : SDNode<"ARMISD::BR_JT", SDT_ARMBrJT,
                              [SDNPHasChain]>;
```
- EN: Defines TableGen record `ARMbrjt` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMbrjt`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 242-244
```tablegen
// Jumptable branch (2 level - jumptable entry is a jump).
def ARMbr2jt         : SDNode<"ARMISD::BR2_JT", SDT_ARMBr2JT,
                              [SDNPHasChain]>;
```
- EN: Defines TableGen record `ARMbr2jt` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMbr2jt`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 246-247
```tablegen
def ARMBcci64        : SDNode<"ARMISD::BCC_i64", SDT_ARMBCC_i64,
                              [SDNPHasChain]>;
```
- EN: Defines TableGen record `ARMBcci64` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMBcci64`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 249-250
```tablegen
// ARM compare instructions.
def ARMcmp           : SDNode<"ARMISD::CMP", SDT_ARMCmp>;
```
- EN: Defines TableGen record `ARMcmp` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcmp`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 252-253
```tablegen
// ARM CMN instructions.
def ARMcmn           : SDNode<"ARMISD::CMN", SDT_ARMCmp, [SDNPCommutative]>;
```
- EN: Defines TableGen record `ARMcmn` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcmn`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 255-256
```tablegen
// ARM compare that sets only Z flag.
def ARMcmpZ          : SDNode<"ARMISD::CMPZ", SDT_ARMCmp, [SDNPCommutative]>;
```
- EN: Defines TableGen record `ARMcmpZ` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMcmpZ`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 258-259
```tablegen
// Add with a PC operand and a PIC label.
def ARMpic_add       : SDNode<"ARMISD::PIC_ADD", SDT_ARMPICAdd>;
```
- EN: Defines TableGen record `ARMpic_add` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMpic_add`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 261-262
```tablegen
// MVE long arithmetic shift right.
def ARMasrl          : SDNode<"ARMISD::ASRL", SDT_ARMIntShiftParts, []>;
```
- EN: Defines TableGen record `ARMasrl` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMasrl`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 264-265
```tablegen
// MVE long shift right.
def ARMlsrl          : SDNode<"ARMISD::LSRL", SDT_ARMIntShiftParts, []>;
```
- EN: Defines TableGen record `ARMlsrl` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMlsrl`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 267-268
```tablegen
// MVE long shift left.
def ARMlsll          : SDNode<"ARMISD::LSLL", SDT_ARMIntShiftParts, []>;
```
- EN: Defines TableGen record `ARMlsll` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMlsll`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 270-271
```tablegen
// Flag-setting logical shift right by one bit.
def ARMlsrs1 : SDNode<"ARMISD::LSRS1", SDTIntUnaryOpWithFlagsOut>;
```
- EN: Defines TableGen record `ARMlsrs1` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMlsrs1`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 273-274
```tablegen
// Flag-setting arithmetic shift right by one bit.
def ARMasrs1 : SDNode<"ARMISD::ASRS1", SDTIntUnaryOpWithFlagsOut>;
```
- EN: Defines TableGen record `ARMasrs1` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMasrs1`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 276-277
```tablegen
// Shift right one bit with carry in.
def ARMrrx   : SDNode<"ARMISD::RRX"  , SDTIntUnaryOpWithFlagsIn>;
```
- EN: Defines TableGen record `ARMrrx` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMrrx`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 279-281
```tablegen
// Add with carry
def ARMaddc          : SDNode<"ARMISD::ADDC",  SDTBinaryArithWithFlags,
                              [SDNPCommutative]>;
```
- EN: Defines TableGen record `ARMaddc` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMaddc`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 283-284
```tablegen
// Sub with carry
def ARMsubc          : SDNode<"ARMISD::SUBC",  SDTBinaryArithWithFlags>;
```
- EN: Defines TableGen record `ARMsubc` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMsubc`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 286-287
```tablegen
// Flag-setting shift left.
def ARMlsls          : SDNode<"ARMISD::LSLS",  SDTBinaryArithWithFlags>;
```
- EN: Defines TableGen record `ARMlsls` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMlsls`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 289-290
```tablegen
// Add using carry
def ARMadde          : SDNode<"ARMISD::ADDE",  SDTBinaryArithWithFlagsInOut>;
```
- EN: Defines TableGen record `ARMadde` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMadde`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 292-293
```tablegen
// Sub using carry
def ARMsube          : SDNode<"ARMISD::SUBE",  SDTBinaryArithWithFlagsInOut>;
```
- EN: Defines TableGen record `ARMsube` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMsube`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 295-295
```tablegen
def ARMthread_pointer: SDNode<"ARMISD::THREAD_POINTER", SDT_ARMThreadPointer>;
```
- EN: Defines TableGen record `ARMthread_pointer:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMthread_pointer:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 297-300
```tablegen
// SjLj exception handling setjmp.
def ARMeh_sjlj_setjmp: SDNode<"ARMISD::EH_SJLJ_SETJMP",
                               SDT_ARMEH_SJLJ_Setjmp,
                               [SDNPHasChain, SDNPSideEffect]>;
```
- EN: Defines TableGen record `ARMeh_sjlj_setjmp:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMeh_sjlj_setjmp:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 302-305
```tablegen
// SjLj exception handling longjmp.
def ARMeh_sjlj_longjmp: SDNode<"ARMISD::EH_SJLJ_LONGJMP",
                               SDT_ARMEH_SJLJ_Longjmp,
                               [SDNPHasChain, SDNPSideEffect]>;
```
- EN: Defines TableGen record `ARMeh_sjlj_longjmp:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMeh_sjlj_longjmp:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 307-310
```tablegen
// SjLj exception handling setup_dispatch.
def ARMeh_sjlj_setup_dispatch: SDNode<"ARMISD::EH_SJLJ_SETUP_DISPATCH",
                                      SDT_ARMEH_SJLJ_SetupDispatch,
                                      [SDNPHasChain, SDNPSideEffect]>;
```
- EN: Defines TableGen record `ARMeh_sjlj_setup_dispatch:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMeh_sjlj_setup_dispatch:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 312-314
```tablegen
// Memory barrier (MCR)
def ARMMemBarrierMCR  : SDNode<"ARMISD::MEMBARRIER_MCR", SDT_ARMMEMBARRIER,
                               [SDNPHasChain, SDNPSideEffect]>;
```
- EN: Defines TableGen record `ARMMemBarrierMCR` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMMemBarrierMCR`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 316-318
```tablegen
// Preload
def ARMPreload        : SDNode<"ARMISD::PRELOAD", SDT_ARMPREFETCH,
                               [SDNPHasChain, SDNPMayLoad, SDNPMayStore]>;
```
- EN: Defines TableGen record `ARMPreload` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMPreload`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 320-322
```tablegen
// Tail call return pseudo.
def ARMtcret         : SDNode<"ARMISD::TC_RETURN", SDT_ARMTCRET,
                        [SDNPHasChain,  SDNPOptInGlue, SDNPVariadic]>;
```
- EN: Defines TableGen record `ARMtcret` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMtcret`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 324-325
```tablegen
// Bit-field insert
def ARMbfi           : SDNode<"ARMISD::BFI", SDT_ARMBFI>;
```
- EN: Defines TableGen record `ARMbfi` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMbfi`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 327-330
```tablegen
// Pseudo-instruction representing a memory copy using ldm/stm instructions.
def ARMmemcopy : SDNode<"ARMISD::MEMCPY", SDT_ARMMEMCPY,
                        [SDNPHasChain, SDNPInGlue, SDNPOutGlue,
                         SDNPMayStore, SDNPMayLoad]>;
```
- EN: Defines TableGen record `ARMmemcopy` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMmemcopy`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 332-333
```tablegen
// Signed multiply word by half word, bottom
def ARMsmulwb       : SDNode<"ARMISD::SMULWB", SDTIntBinOp, []>;
```
- EN: Defines TableGen record `ARMsmulwb` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMsmulwb`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 335-336
```tablegen
// Signed multiply word by half word, top
def ARMsmulwt       : SDNode<"ARMISD::SMULWT", SDTIntBinOp, []>;
```
- EN: Defines TableGen record `ARMsmulwt` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMsmulwt`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 338-339
```tablegen
// 64bit Unsigned Accumulate Multiply
def ARMumlal : SDNode<"ARMISD::UMLAL", SDT_LongMac>;
```
- EN: Defines TableGen record `ARMumlal` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMumlal`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 341-342
```tablegen
// 64bit Signed Accumulate Multiply
def ARMsmlal : SDNode<"ARMISD::SMLAL", SDT_LongMac>;
```
- EN: Defines TableGen record `ARMsmlal` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMsmlal`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 344-345
```tablegen
// 64-bit Unsigned Accumulate Accumulate Multiply
def ARMumaal : SDNode<"ARMISD::UMAAL", SDT_LongMac>;
```
- EN: Defines TableGen record `ARMumaal` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMumaal`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 347-348
```tablegen
// 64-bit signed accumulate multiply bottom, bottom 16
def ARMsmlalbb      : SDNode<"ARMISD::SMLALBB", SDT_LongMac, []>;
```
- EN: Defines TableGen record `ARMsmlalbb` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMsmlalbb`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 350-351
```tablegen
// 64-bit signed accumulate multiply bottom, top 16
def ARMsmlalbt      : SDNode<"ARMISD::SMLALBT", SDT_LongMac, []>;
```
- EN: Defines TableGen record `ARMsmlalbt` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMsmlalbt`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 353-354
```tablegen
// 64-bit signed accumulate multiply top, bottom 16
def ARMsmlaltb      : SDNode<"ARMISD::SMLALTB", SDT_LongMac, []>;
```
- EN: Defines TableGen record `ARMsmlaltb` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMsmlaltb`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 356-357
```tablegen
// 64-bit signed accumulate multiply top, top 16
def ARMsmlaltt      : SDNode<"ARMISD::SMLALTT", SDT_LongMac, []>;
```
- EN: Defines TableGen record `ARMsmlaltt` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMsmlaltt`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 359-364
```tablegen
// Single Lane QADD8 and QADD16. Only the bottom lane. That's what the b
// stands for.
def ARMqadd8b       : SDNode<"ARMISD::QADD8b", SDT_ARMAnd, []>;
def ARMqsub8b       : SDNode<"ARMISD::QSUB8b", SDT_ARMAnd, []>;
def ARMqadd16b      : SDNode<"ARMISD::QADD16b", SDT_ARMAnd, []>;
def ARMqsub16b      : SDNode<"ARMISD::QSUB16b", SDT_ARMAnd, []>;
```
- EN: Defines TableGen record `ARMqadd8b` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMqadd8b`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 366-369
```tablegen
def ARMuqadd8b       : SDNode<"ARMISD::UQADD8b", SDT_ARMAnd, []>;
def ARMuqsub8b       : SDNode<"ARMISD::UQSUB8b", SDT_ARMAnd, []>;
def ARMuqadd16b      : SDNode<"ARMISD::UQADD16b", SDT_ARMAnd, []>;
def ARMuqsub16b      : SDNode<"ARMISD::UQSUB16b", SDT_ARMAnd, []>;
```
- EN: Defines TableGen record `ARMuqadd8b` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMuqadd8b`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 371-372
```tablegen
def SDT_ARMldrd     : SDTypeProfile<2, 1, [SDTCisVT<0, i32>, SDTCisSameAs<0, 1>, SDTCisPtrTy<2>]>;
def SDT_ARMstrd     : SDTypeProfile<0, 3, [SDTCisVT<0, i32>, SDTCisSameAs<0, 1>, SDTCisPtrTy<2>]>;
```
- EN: Defines TableGen record `SDT_ARMldrd` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDT_ARMldrd`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 374-376
```tablegen
// Load/Store of dual registers
def ARMldrd         : SDNode<"ARMISD::LDRD", SDT_ARMldrd, [SDNPHasChain, SDNPMayLoad, SDNPMemOperand]>;
def ARMstrd         : SDNode<"ARMISD::STRD", SDT_ARMstrd, [SDNPHasChain, SDNPMayStore, SDNPMemOperand]>;
```
- EN: Defines TableGen record `ARMldrd` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMldrd`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 378-378
```tablegen
// Vector operations shared between NEON and MVE
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 380-381
```tablegen
// Vector duplicate
def ARMvdup      : SDNode<"ARMISD::VDUP", SDTypeProfile<1, 1, [SDTCisVec<0>]>>;
```
- EN: Defines TableGen record `ARMvdup` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvdup`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 383-387
```tablegen
// VDUPLANE can produce a quad-register result from a double-register source,
// so the result is not constrained to match the source.
def ARMvduplane  : SDNode<"ARMISD::VDUPLANE",
                          SDTypeProfile<1, 2, [SDTCisVec<0>, SDTCisVec<1>,
                                               SDTCisVT<2, i32>]>>;
```
- EN: Defines TableGen record `ARMvduplane` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvduplane`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 389-390
```tablegen
def SDTARMVIDUP  : SDTypeProfile<2, 2, [SDTCisVec<0>, SDTCisVT<1, i32>,
                                          SDTCisVT<2, i32>, SDTCisVT<3, i32>]>;
```
- EN: Defines TableGen record `SDTARMVIDUP` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVIDUP`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 392-393
```tablegen
// MVE VIDUP instruction, taking a start value and increment.
def ARMvidup    : SDNode<"ARMISD::VIDUP", SDTARMVIDUP>;
```
- EN: Defines TableGen record `ARMvidup` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvidup`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 395-395
```tablegen
def SDTARMVSHUF   : SDTypeProfile<1, 1, [SDTCisVec<0>, SDTCisSameAs<0, 1>]>;
```
- EN: Defines TableGen record `SDTARMVSHUF` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVSHUF`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 397-398
```tablegen
// reverse elements within 64-bit doublewords
def ARMvrev64    : SDNode<"ARMISD::VREV64", SDTARMVSHUF>;
```
- EN: Defines TableGen record `ARMvrev64` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvrev64`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 400-401
```tablegen
// reverse elements within 32-bit words
def ARMvrev32    : SDNode<"ARMISD::VREV32", SDTARMVSHUF>;
```
- EN: Defines TableGen record `ARMvrev32` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvrev32`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 403-404
```tablegen
// reverse elements within 16-bit halfwords
def ARMvrev16    : SDNode<"ARMISD::VREV16", SDTARMVSHUF>;
```
- EN: Defines TableGen record `ARMvrev16` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvrev16`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 406-407
```tablegen
def SDTARMVGETLN  : SDTypeProfile<1, 2, [SDTCisVT<0, i32>, SDTCisVec<1>,
                                         SDTCisVT<2, i32>]>;
```
- EN: Defines TableGen record `SDTARMVGETLN` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVGETLN`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 409-412
```tablegen
// Vector get lane (VMOV scalar to ARM core register)
// (These are used for 8- and 16-bit element types only.)
def ARMvgetlaneu : SDNode<"ARMISD::VGETLANEu", SDTARMVGETLN>;
def ARMvgetlanes : SDNode<"ARMISD::VGETLANEs", SDTARMVGETLN>;
```
- EN: Defines TableGen record `ARMvgetlaneu` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvgetlaneu`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 414-414
```tablegen
def SDTARMVMOVIMM : SDTypeProfile<1, 1, [SDTCisVec<0>, SDTCisVT<1, i32>]>;
```
- EN: Defines TableGen record `SDTARMVMOVIMM` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVMOVIMM`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 416-418
```tablegen
// Vector move immediate and move negated immediate
def ARMvmovImm   : SDNode<"ARMISD::VMOVIMM", SDTARMVMOVIMM>;
def ARMvmvnImm   : SDNode<"ARMISD::VMVNIMM", SDTARMVMOVIMM>;
```
- EN: Defines TableGen record `ARMvmovImm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvmovImm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 420-421
```tablegen
// Vector move f32 immediate
def ARMvmovFPImm : SDNode<"ARMISD::VMOVFPIMM", SDTARMVMOVIMM>;
```
- EN: Defines TableGen record `ARMvmovFPImm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvmovFPImm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 423-424
```tablegen
def SDTARMVORRIMM : SDTypeProfile<1, 2, [SDTCisVec<0>, SDTCisSameAs<0, 1>,
                                           SDTCisVT<2, i32>]>;
```
- EN: Defines TableGen record `SDTARMVORRIMM` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVORRIMM`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 426-427
```tablegen
// Vector OR with immediate
def ARMvorrImm   : SDNode<"ARMISD::VORRIMM", SDTARMVORRIMM>;
```
- EN: Defines TableGen record `ARMvorrImm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvorrImm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 429-430
```tablegen
// Vector AND with NOT of immediate
def ARMvbicImm   : SDNode<"ARMISD::VBICIMM", SDTARMVORRIMM>;
```
- EN: Defines TableGen record `ARMvbicImm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvbicImm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 432-435
```tablegen
def SDTARMVSHIMM : SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisSameAs<0, 1>,
                                        SDTCisVT<2, i32>]>;
def SDTARMVSH : SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisSameAs<0, 1>,
                                     SDTCisSameAs<0, 2>,]>;
```
- EN: Defines TableGen record `SDTARMVSHIMM` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVSHIMM`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 437-440
```tablegen
// Vector shift by immediate
def ARMvshlImm   : SDNode<"ARMISD::VSHLIMM", SDTARMVSHIMM>;
def ARMvshrsImm  : SDNode<"ARMISD::VSHRsIMM", SDTARMVSHIMM>;
def ARMvshruImm  : SDNode<"ARMISD::VSHRuIMM", SDTARMVSHIMM>;
```
- EN: Defines TableGen record `ARMvshlImm` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvshlImm`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 442-444
```tablegen
// Vector shift by vector
def ARMvshls     : SDNode<"ARMISD::VSHLs", SDTARMVSH>;
def ARMvshlu     : SDNode<"ARMISD::VSHLu", SDTARMVSH>;
```
- EN: Defines TableGen record `ARMvshls` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvshls`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 446-447
```tablegen
def SDTARMVMULL   : SDTypeProfile<1, 2, [SDTCisInt<0>, SDTCisInt<1>,
                                         SDTCisSameAs<1, 2>]>;
```
- EN: Defines TableGen record `SDTARMVMULL` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVMULL`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 449-451
```tablegen
// Vector multiply long
def ARMvmulls    : SDNode<"ARMISD::VMULLs", SDTARMVMULL>;
def ARMvmullu    : SDNode<"ARMISD::VMULLu", SDTARMVMULL>;
```
- EN: Defines TableGen record `ARMvmulls` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvmulls`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 453-455
```tablegen
def SDTARMVCMP    : SDTypeProfile<1, 3, [SDTCisInt<0>, SDTCisSameAs<1, 2>,
                                         SDTCisInt<3>]>;
def SDTARMVCMPZ   : SDTypeProfile<1, 2, [SDTCisInt<2>]>;
```
- EN: Defines TableGen record `SDTARMVCMP` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SDTARMVCMP`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 457-458
```tablegen
// Vector compare.
def ARMvcmp      : SDNode<"ARMISD::VCMP", SDTARMVCMP>;
```
- EN: Defines TableGen record `ARMvcmp` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvcmp`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 460-461
```tablegen
// Vector compare to zero.
def ARMvcmpz     : SDNode<"ARMISD::VCMPZ", SDTARMVCMPZ>;
```
- EN: Defines TableGen record `ARMvcmpz` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMvcmpz`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 463-477
```tablegen
// Reinterpret the current contents of a vector register
// 'VECTOR_REG_CAST' is an operation that reinterprets the contents of a
// vector register as a different vector type, without changing the contents of
// the register. It differs from 'bitconvert' in that bitconvert reinterprets
// the _memory_ storage format of the vector, whereas VECTOR_REG_CAST
// reinterprets the _register_ format - and in big-endian, the memory and
// register formats are different, so they are different operations.
//
// For example, 'VECTOR_REG_CAST' between v8i16 and v16i8 will map the LSB of
// the zeroth i16 lane to the zeroth i8 lane, regardless of system endianness,
// whereas 'bitconvert' will map it to the high byte in big-endian mode,
// because that's what (MVE) VSTRH.16 followed by VLDRB.8 would do. So the
// bitconvert would have to emit a VREV16.8 instruction, whereas the
// VECTOR_REG_CAST emits no code at all if the vector is already in a register.
def ARMVectorRegCastImpl : SDNode<"ARMISD::VECTOR_REG_CAST", SDTUnaryOp>;
```
- EN: Defines TableGen record `ARMVectorRegCastImpl` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMVectorRegCastImpl`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 479-492
```tablegen
// In little-endian, VECTOR_REG_CAST is often turned into bitconvert during
// lowering (because in that situation they're identical). So an isel pattern
// that needs to match something that's _logically_ a VECTOR_REG_CAST must
// _physically_ match a different node type depending on endianness.
//
// This 'PatFrags' instance is a centralized facility to make that easy. It
// matches VECTOR_REG_CAST in either endianness, and also bitconvert in the
// endianness where it's equivalent.
def ARMVectorRegCast: PatFrags<
    (ops node:$x), [(ARMVectorRegCastImpl node:$x), (bitconvert node:$x)], [{
       // Reject a match against bitconvert (aka ISD::BITCAST) if big-endian
       return !(CurDAG->getDataLayout().isBigEndian() &&
                N->getOpcode() == ISD::BITCAST);
    }]>;
```
- EN: Defines TableGen record `ARMVectorRegCast:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMVectorRegCast:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 494-495
```tablegen
//===----------------------------------------------------------------------===//
// ARM Flag Definitions.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 497-499
```tablegen
class RegConstraint<string C> {
  string Constraints = C;
}
```
- EN: Declares reusable TableGen class `RegConstraint` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `RegConstraint`，通常用于抽象共享字段、谓词或编码结构。

### Lines 501-516
```tablegen
// ARMCC condition codes. See ARMCC::CondCodes
def ARMCCeq : PatLeaf<(i32 0)>;
def ARMCCne : PatLeaf<(i32 1)>;
def ARMCChs : PatLeaf<(i32 2)>;
def ARMCClo : PatLeaf<(i32 3)>;
def ARMCCmi : PatLeaf<(i32 4)>;
def ARMCCpl : PatLeaf<(i32 5)>;
def ARMCCvs : PatLeaf<(i32 6)>;
def ARMCCvc : PatLeaf<(i32 7)>;
def ARMCChi : PatLeaf<(i32 8)>;
def ARMCCls : PatLeaf<(i32 9)>;
def ARMCCge : PatLeaf<(i32 10)>;
def ARMCClt : PatLeaf<(i32 11)>;
def ARMCCgt : PatLeaf<(i32 12)>;
def ARMCCle : PatLeaf<(i32 13)>;
def ARMCCal : PatLeaf<(i32 14)>;
```
- EN: Defines TableGen record `ARMCCeq` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMCCeq`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 518-521
```tablegen
// VCC predicates. See ARMVCC::VPTCodes
def ARMVCCNone : PatLeaf<(i32 0)>;
def ARMVCCThen : PatLeaf<(i32 1)>;
def ARMVCCElse : PatLeaf<(i32 2)>;
```
- EN: Defines TableGen record `ARMVCCNone` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMVCCNone`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 523-525
```tablegen
//===----------------------------------------------------------------------===//
//  ARM specific transformation functions and pattern fragments.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 527-531
```tablegen
// imm_neg_XFORM - Return the negation of an i32 immediate value.
def imm_neg_XFORM : SDNodeXForm<imm, [{
  return CurDAG->getSignedTargetConstant(-(int)N->getZExtValue(), SDLoc(N),
                                         MVT::i32);
}]>;
```
- EN: Defines TableGen record `imm_neg_XFORM` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `imm_neg_XFORM`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 533-539
```tablegen
// imm_not_XFORM - Return the complement of a i32 immediate value.
def imm_not_XFORM : SDNodeXForm<imm, [{
  return CurDAG->getSignedTargetConstant(~(int)N->getZExtValue(), SDLoc(N),
                                         MVT::i32);
}]>;
def gi_imm_not_XFORM : GICustomOperandRenderer<"renderInvertedImm">,
  GISDNodeXFormEquiv<imm_not_XFORM>;
```
- EN: Defines TableGen record `imm_not_XFORM` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `imm_not_XFORM`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 541-544
```tablegen
// asr_imm_XFORM - Returns a shift immediate with bit {5} set to 1
def asr_imm_XFORM : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(0x20 | N->getZExtValue(), SDLoc(N), MVT:: i32);
}]>;
```
- EN: Defines TableGen record `asr_imm_XFORM` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `asr_imm_XFORM`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 546-549
```tablegen
/// imm16_31 predicate - True if the 32-bit immediate is in the range [16,31].
def imm16_31 : ImmLeaf<i32, [{
  return (int32_t)Imm >= 16 && (int32_t)Imm < 32;
}]>;
```
- EN: Defines TableGen record `imm16_31` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm16_31`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 551-554
```tablegen
// sext_16_node predicate - True if the SDNode is sign-extended 16 or more bits.
def sext_16_node : PatLeaf<(i32 GPR:$a), [{
  return CurDAG->ComputeNumSignBits(Op) >= 17;
}]>;
```
- EN: Defines TableGen record `sext_16_node` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `sext_16_node`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 556-559
```tablegen
def sext_bottom_16 : PatFrag<(ops node:$a),
                             (sext_inreg node:$a, i16)>;
def sext_top_16 : PatFrag<(ops node:$a),
                          (i32 (sra node:$a, (i32 16)))>;
```
- EN: Defines TableGen record `sext_bottom_16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `sext_bottom_16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 561-568
```tablegen
def bb_mul : PatFrag<(ops node:$a, node:$b),
                     (mul (sext_bottom_16 node:$a), (sext_bottom_16 node:$b))>;
def bt_mul : PatFrag<(ops node:$a, node:$b),
                     (mul (sext_bottom_16 node:$a), (sra node:$b, (i32 16)))>;
def tb_mul : PatFrag<(ops node:$a, node:$b),
                     (mul (sra node:$a, (i32 16)), (sext_bottom_16 node:$b))>;
def tt_mul : PatFrag<(ops node:$a, node:$b),
                     (mul (sra node:$a, (i32 16)), (sra node:$b, (i32 16)))>;
```
- EN: Defines TableGen record `bb_mul` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `bb_mul`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 570-574
```tablegen
/// Split a 32-bit immediate into two 16 bit parts.
def hi16 : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant((uint32_t)N->getZExtValue() >> 16, SDLoc(N),
                                   MVT::i32);
}]>;
```
- EN: Defines TableGen record `hi16` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `hi16`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 576-579
```tablegen
def lo16AllZero : PatLeaf<(i32 imm), [{
  // Returns true if all low 16-bits are 0.
  return (((uint32_t)N->getZExtValue()) & 0xFFFFUL) == 0;
}], hi16>;
```
- EN: Defines TableGen record `lo16AllZero` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `lo16AllZero`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 581-585
```tablegen
// top16Zero - answer true if the upper 16 bits of $src are 0, false otherwise
def top16Zero: PatLeaf<(i32 GPR:$src), [{
  return !Op.getValueType().isVector() &&
         CurDAG->MaskedValueIsZero(Op, APInt::getHighBitsSet(32, 16));
  }]>;
```
- EN: Defines TableGen record `top16Zero:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `top16Zero:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 587-591
```tablegen
// topbitsallzero - Return true if all bits except the lowest bit are known zero
def topbitsallzero32 : PatLeaf<(i32 GPRwithZR:$src), [{
  return Op.getValueType() == MVT::i32 &&
         CurDAG->MaskedValueIsZero(Op, APInt::getHighBitsSet(32, 31));
  }]>;
```
- EN: Defines TableGen record `topbitsallzero32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `topbitsallzero32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 593-594
```tablegen
class BinOpFrag<dag res> : PatFrag<(ops node:$LHS, node:$RHS), res>;
class UnOpFrag <dag res> : PatFrag<(ops node:$Src), res>;
```
- EN: Declares reusable TableGen class `BinOpFrag` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `BinOpFrag`，通常用于抽象共享字段、谓词或编码结构。

### Lines 596-598
```tablegen
// An 'and' node with a single use.
let HasOneUse = 1 in
def and_su : PatFrag<(ops node:$lhs, node:$rhs), (and node:$lhs, node:$rhs)>;
```
- EN: Defines TableGen record `and_su` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `and_su`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 600-602
```tablegen
// An 'xor' node with a single use.
let HasOneUse = 1 in
def xor_su : PatFrag<(ops node:$lhs, node:$rhs), (xor node:$lhs, node:$rhs)>;
```
- EN: Defines TableGen record `xor_su` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `xor_su`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 604-606
```tablegen
// An 'fmul' node with a single use.
let HasOneUse = 1 in
def fmul_su : PatFrag<(ops node:$lhs, node:$rhs), (any_fmul node:$lhs, node:$rhs)>;
```
- EN: Defines TableGen record `fmul_su` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `fmul_su`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 608-611
```tablegen
// An 'fadd' node which checks for single non-hazardous use.
def fadd_mlx : PatFrag<(ops node:$lhs, node:$rhs),(any_fadd node:$lhs, node:$rhs),[{
  return hasNoVMLxHazardUse(N);
}]>;
```
- EN: Defines TableGen record `fadd_mlx` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `fadd_mlx`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 613-616
```tablegen
// An 'fsub' node which checks for single non-hazardous use.
def fsub_mlx : PatFrag<(ops node:$lhs, node:$rhs),(any_fsub node:$lhs, node:$rhs),[{
  return hasNoVMLxHazardUse(N);
}]>;
```
- EN: Defines TableGen record `fsub_mlx` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `fsub_mlx`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 618-619
```tablegen
def imm_even : ImmLeaf<i32, [{ return (Imm & 1) == 0; }]>;
def imm_odd : ImmLeaf<i32, [{ return (Imm & 1) == 1; }]>;
```
- EN: Defines TableGen record `imm_even` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm_even`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 621-621
```tablegen
def asr_imm : ImmLeaf<i32, [{ return Imm > 0 && Imm <= 32; }], asr_imm_XFORM>;
```
- EN: Defines TableGen record `asr_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `asr_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 623-625
```tablegen
//===----------------------------------------------------------------------===//
// NEON/MVE pattern fragments
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 627-644
```tablegen
// Extract D sub-registers of Q registers.
def DSubReg_i8_reg  : SDNodeXForm<imm, [{
  assert(ARM::dsub_7 == ARM::dsub_0+7 && "Unexpected subreg numbering");
  return CurDAG->getTargetConstant(ARM::dsub_0 + N->getZExtValue()/8, SDLoc(N),
                                   MVT::i32);
}]>;
def DSubReg_i16_reg : SDNodeXForm<imm, [{
  assert(ARM::dsub_7 == ARM::dsub_0+7 && "Unexpected subreg numbering");
  return CurDAG->getTargetConstant(ARM::dsub_0 + N->getZExtValue()/4, SDLoc(N),
                                   MVT::i32);
}]>;
def DSubReg_i32_reg : SDNodeXForm<imm, [{
  assert(ARM::dsub_7 == ARM::dsub_0+7 && "Unexpected subreg numbering");
  return CurDAG->getTargetConstant(ARM::dsub_0 + N->getZExtValue()/2, SDLoc(N),
                                   MVT::i32);
}]>;
def DSubReg_f64_reg : SDNodeXForm<imm, [{
  assert(ARM::dsub_7 == ARM::dsub_0+7 && "Unexpected subreg numbering");
```
- EN: Defines TableGen record `DSubReg_i8_reg` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `DSubReg_i8_reg`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 645-647
```tablegen
  return CurDAG->getTargetConstant(ARM::dsub_0 + N->getZExtValue(), SDLoc(N),
                                   MVT::i32);
}]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 649-654
```tablegen
// Extract S sub-registers of Q/D registers.
def SSubReg_f32_reg : SDNodeXForm<imm, [{
  assert(ARM::ssub_3 == ARM::ssub_0+3 && "Unexpected subreg numbering");
  return CurDAG->getTargetConstant(ARM::ssub_0 + N->getZExtValue(), SDLoc(N),
                                   MVT::i32);
}]>;
```
- EN: Defines TableGen record `SSubReg_f32_reg` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SSubReg_f32_reg`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 656-661
```tablegen
// Extract S sub-registers of Q/D registers containing a given f16/bf16 lane.
def SSubReg_f16_reg : SDNodeXForm<imm, [{
  assert(ARM::ssub_3 == ARM::ssub_0+3 && "Unexpected subreg numbering");
  return CurDAG->getTargetConstant(ARM::ssub_0 + N->getZExtValue()/2, SDLoc(N),
                                   MVT::i32);
}]>;
```
- EN: Defines TableGen record `SSubReg_f16_reg` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SSubReg_f16_reg`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 663-672
```tablegen
// Translate lane numbers from Q registers to D subregs.
def SubReg_i8_lane  : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(N->getZExtValue() & 7, SDLoc(N), MVT::i32);
}]>;
def SubReg_i16_lane : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(N->getZExtValue() & 3, SDLoc(N), MVT::i32);
}]>;
def SubReg_i32_lane : SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant(N->getZExtValue() & 1, SDLoc(N), MVT::i32);
}]>;
```
- EN: Defines TableGen record `SubReg_i8_lane` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `SubReg_i8_lane`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 675-678
```tablegen
def ARMimmAllZerosV: PatLeaf<(bitconvert (v4i32 (ARMvmovImm (i32 0))))>;
def ARMimmAllZerosD: PatLeaf<(bitconvert (v2i32 (ARMvmovImm (i32 0))))>;
def ARMimmAllOnesV: PatLeaf<(bitconvert (v16i8 (ARMvmovImm (i32 0xEFF))))>;
def ARMimmAllOnesD: PatLeaf<(bitconvert (v8i8 (ARMvmovImm (i32 0xEFF))))>;
```
- EN: Defines TableGen record `ARMimmAllZerosV:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMimmAllZerosV:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 680-685
```tablegen
def ARMimmOneV: PatLeaf<(ARMvmovImm (i32 timm)), [{
  ConstantSDNode *ConstVal = cast<ConstantSDNode>(N->getOperand(0));
  unsigned EltBits = 0;
  uint64_t EltVal = ARM_AM::decodeVMOVModImm(ConstVal->getZExtValue(), EltBits);
  return (EltBits == N->getValueType(0).getScalarSizeInBits() && EltVal == 0x01);
}]>;
```
- EN: Defines TableGen record `ARMimmOneV:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ARMimmOneV:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 688-690
```tablegen
//===----------------------------------------------------------------------===//
// Operand Definitions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 692-697
```tablegen
// Immediate operands with a shared generic asm render method.
class ImmAsmOperand<int Low, int High> : AsmOperandClass {
  let RenderMethod = "addImmOperands";
  let PredicateMethod = "isImmediate<" # Low # "," # High # ">";
  let DiagnosticString = "operand must be an immediate in the range [" # Low # "," # High # "]";
}
```
- EN: Declares reusable TableGen class `ImmAsmOperand` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `ImmAsmOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 699-703
```tablegen
class ImmAsmOperandMinusOne<int Low, int High> : AsmOperandClass {
  let PredicateMethod = "isImmediate<" # Low # "," # High # ">";
  let DiagnosticType = "ImmRange" # Low # "_" # High;
  let DiagnosticString = "operand must be an immediate in the range [" # Low # "," # High # "]";
}
```
- EN: Declares reusable TableGen class `ImmAsmOperandMinusOne` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `ImmAsmOperandMinusOne`，通常用于抽象共享字段、谓词或编码结构。

### Lines 705-706
```tablegen
// Operands that are part of a memory addressing mode.
class MemOperand : Operand<i32> { let OperandType = "OPERAND_MEMORY"; }
```
- EN: Declares reusable TableGen class `MemOperand` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `MemOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 708-714
```tablegen
// Branch target.
// FIXME: rename brtarget to t2_brtarget
def brtarget : Operand<OtherVT> {
  let EncoderMethod = "getBranchTargetOpValue";
  let OperandType = "OPERAND_PCREL";
  let DecoderMethod = "DecodeT2BROperand";
}
```
- EN: Defines TableGen record `brtarget` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `brtarget`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 716-720
```tablegen
// Branches targeting ARM-mode must be divisible by 4 if they're a raw
// immediate.
def ARMBranchTarget : AsmOperandClass {
  let Name = "ARMBranchTarget";
}
```
- EN: Defines TableGen record `ARMBranchTarget` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ARMBranchTarget`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 722-726
```tablegen
// Branches targeting Thumb-mode must be divisible by 2 if they're a raw
// immediate.
def ThumbBranchTarget : AsmOperandClass {
  let Name = "ThumbBranchTarget";
}
```
- EN: Defines TableGen record `ThumbBranchTarget` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ThumbBranchTarget`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 728-732
```tablegen
def arm_br_target : Operand<OtherVT> {
  let ParserMatchClass = ARMBranchTarget;
  let EncoderMethod = "getARMBranchTargetOpValue";
  let OperandType = "OPERAND_PCREL";
}
```
- EN: Defines TableGen record `arm_br_target` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `arm_br_target`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 734-740
```tablegen
// Call target for ARM. Handles conditional/unconditional
// FIXME: rename bl_target to t2_bltarget?
def arm_bl_target : Operand<i32> {
  let ParserMatchClass = ARMBranchTarget;
  let EncoderMethod = "getARMBLTargetOpValue";
  let OperandType = "OPERAND_PCREL";
}
```
- EN: Defines TableGen record `arm_bl_target` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `arm_bl_target`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 742-747
```tablegen
// Target for BLX *from* ARM mode.
def arm_blx_target : Operand<i32> {
  let ParserMatchClass = ThumbBranchTarget;
  let EncoderMethod = "getARMBLXTargetOpValue";
  let OperandType = "OPERAND_PCREL";
}
```
- EN: Defines TableGen record `arm_blx_target` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `arm_blx_target`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 749-756
```tablegen
// A list of registers separated by comma. Used by load/store multiple.
def RegListAsmOperand : AsmOperandClass { let Name = "RegList"; }
def reglist : Operand<i32> {
  let EncoderMethod = "getRegisterListOpValue";
  let ParserMatchClass = RegListAsmOperand;
  let PrintMethod = "printRegisterList";
  let DecoderMethod = "DecodeRegListOperand";
}
```
- EN: Defines TableGen record `RegListAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RegListAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 758-766
```tablegen
// A list of general purpose registers and APSR separated by comma.
// Used by CLRM
def RegListWithAPSRAsmOperand : AsmOperandClass { let Name = "RegListWithAPSR"; }
def reglist_with_apsr : Operand<i32> {
  let EncoderMethod = "getRegisterListOpValue";
  let ParserMatchClass = RegListWithAPSRAsmOperand;
  let PrintMethod = "printRegisterList";
  let DecoderMethod = "DecodeRegListOperand";
}
```
- EN: Defines TableGen record `RegListWithAPSRAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RegListWithAPSRAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 768-768
```tablegen
def GPRPairOp : RegisterOperand<GPRPair, "printGPRPairOperand">;
```
- EN: Defines TableGen record `GPRPairOp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `GPRPairOp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 770-779
```tablegen
def DPRRegListAsmOperand : AsmOperandClass {
  let Name = "DPRRegList";
  let DiagnosticType = "DPR_RegList";
}
def dpr_reglist : Operand<i32> {
  let EncoderMethod = "getRegisterListOpValue";
  let ParserMatchClass = DPRRegListAsmOperand;
  let PrintMethod = "printRegisterList";
  let DecoderMethod = "DecodeDPRRegListOperand";
}
```
- EN: Defines TableGen record `DPRRegListAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DPRRegListAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 781-790
```tablegen
def SPRRegListAsmOperand : AsmOperandClass {
  let Name = "SPRRegList";
  let DiagnosticString = "operand must be a list of registers in range [s0, s31]";
}
def spr_reglist : Operand<i32> {
  let EncoderMethod = "getRegisterListOpValue";
  let ParserMatchClass = SPRRegListAsmOperand;
  let PrintMethod = "printRegisterList";
  let DecoderMethod = "DecodeSPRRegListOperand";
}
```
- EN: Defines TableGen record `SPRRegListAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SPRRegListAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 792-805
```tablegen
def FPSRegListWithVPRAsmOperand : AsmOperandClass { let Name =
    "FPSRegListWithVPR"; }
def fp_sreglist_with_vpr : Operand<i32> {
  let EncoderMethod = "getRegisterListOpValue";
  let ParserMatchClass = FPSRegListWithVPRAsmOperand;
  let PrintMethod = "printRegisterList";
}
def FPDRegListWithVPRAsmOperand : AsmOperandClass { let Name =
    "FPDRegListWithVPR"; }
def fp_dreglist_with_vpr : Operand<i32> {
  let EncoderMethod = "getRegisterListOpValue";
  let ParserMatchClass = FPDRegListWithVPRAsmOperand;
  let PrintMethod = "printRegisterList";
}
```
- EN: Defines TableGen record `FPSRegListWithVPRAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `FPSRegListWithVPRAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 807-810
```tablegen
// An operand for the CONSTPOOL_ENTRY pseudo-instruction.
def cpinst_operand : Operand<i32> {
  let PrintMethod = "printCPInstOperand";
}
```
- EN: Defines TableGen record `cpinst_operand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `cpinst_operand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 812-815
```tablegen
// Local PC labels.
def pclabel : Operand<i32> {
  let PrintMethod = "printPCLabel";
}
```
- EN: Defines TableGen record `pclabel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `pclabel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 817-823
```tablegen
// ADR instruction labels.
def AdrLabelAsmOperand : AsmOperandClass { let Name = "AdrLabel"; }
def adrlabel : Operand<i32> {
  let EncoderMethod = "getAdrLabelOpValue";
  let ParserMatchClass = AdrLabelAsmOperand;
  let PrintMethod = "printAdrLabelOperand<0>";
}
```
- EN: Defines TableGen record `AdrLabelAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AdrLabelAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 825-828
```tablegen
def neon_vcvt_imm32 : Operand<i32> {
  let EncoderMethod = "getNEONVcvtImm32OpValue";
  let DecoderMethod = "DecodeVCVTImmOperand";
}
```
- EN: Defines TableGen record `neon_vcvt_imm32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `neon_vcvt_imm32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 830-847
```tablegen
// rot_imm: An integer that encodes a rotate amount. Must be 8, 16, or 24.
def rot_imm_XFORM: SDNodeXForm<imm, [{
  switch (N->getZExtValue()){
  default: llvm_unreachable(nullptr);
  case 0:  return CurDAG->getTargetConstant(0, SDLoc(N), MVT::i32);
  case 8:  return CurDAG->getTargetConstant(1, SDLoc(N), MVT::i32);
  case 16: return CurDAG->getTargetConstant(2, SDLoc(N), MVT::i32);
  case 24: return CurDAG->getTargetConstant(3, SDLoc(N), MVT::i32);
  }
}]>;
def RotImmAsmOperand : AsmOperandClass {
  let Name = "RotImm";
  let ParserMethod = "parseRotImm";
}
def rot_imm : Operand<i32>, PatLeaf<(i32 imm), [{
    int32_t v = N->getZExtValue();
    return v == 8 || v == 16 || v == 24; }],
    rot_imm_XFORM> {
```
- EN: Defines TableGen record `rot_imm_XFORM:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `rot_imm_XFORM:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 848-850
```tablegen
  let PrintMethod = "printRotImmOperand";
  let ParserMatchClass = RotImmAsmOperand;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 852-864
```tablegen
// Power-of-two operand for MVE VIDUP and friends, which encode
// {1,2,4,8} as its log to base 2, i.e. as {0,1,2,3} respectively
def MVE_VIDUP_imm_asmoperand : AsmOperandClass {
  let Name = "VIDUP_imm";
  let PredicateMethod = "isPowerTwoInRange<1,8>";
  let RenderMethod = "addPowerTwoOperands";
  let DiagnosticString = "vector increment immediate must be 1, 2, 4 or 8";
}
def MVE_VIDUP_imm : Operand<i32> {
  let EncoderMethod = "getPowerTwoOpValue";
  let DecoderMethod = "DecodePowerTwoOperand<0,3>";
  let ParserMatchClass = MVE_VIDUP_imm_asmoperand;
}
```
- EN: Defines TableGen record `MVE_VIDUP_imm_asmoperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVE_VIDUP_imm_asmoperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 866-871
```tablegen
// Pair vector indexing
class MVEPairVectorIndexOperand<string start, string end> : AsmOperandClass {
  let Name = "MVEPairVectorIndex"#start;
  let RenderMethod = "addMVEPairVectorIndexOperands";
  let PredicateMethod = "isMVEPairVectorIndex<"#start#", "#end#">";
}
```
- EN: Declares reusable TableGen class `MVEPairVectorIndexOperand` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `MVEPairVectorIndexOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 873-878
```tablegen
class MVEPairVectorIndex<string opval> : Operand<i32> {
  let PrintMethod = "printVectorIndex";
  let EncoderMethod = "getMVEPairVectorIndexOpValue<"#opval#">";
  let DecoderMethod = "DecodeMVEPairVectorIndexOperand<"#opval#">";
  let MIOperandInfo = (ops i32imm);
}
```
- EN: Declares reusable TableGen class `MVEPairVectorIndex` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `MVEPairVectorIndex`，通常用于抽象共享字段、谓词或编码结构。

### Lines 880-882
```tablegen
def MVEPairVectorIndex0 : MVEPairVectorIndex<"0"> {
  let ParserMatchClass = MVEPairVectorIndexOperand<"0", "1">;
}
```
- EN: Defines TableGen record `MVEPairVectorIndex0` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVEPairVectorIndex0`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 884-886
```tablegen
def MVEPairVectorIndex2 : MVEPairVectorIndex<"2"> {
  let ParserMatchClass = MVEPairVectorIndexOperand<"2", "3">;
}
```
- EN: Defines TableGen record `MVEPairVectorIndex2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVEPairVectorIndex2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 888-893
```tablegen
// Vector indexing
class MVEVectorIndexOperand<int NumLanes> : AsmOperandClass {
  let Name = "MVEVectorIndex"#NumLanes;
  let RenderMethod = "addMVEVectorIndexOperands";
  let PredicateMethod = "isVectorIndexInRange<"#NumLanes#">";
}
```
- EN: Declares reusable TableGen class `MVEVectorIndexOperand` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `MVEVectorIndexOperand`，通常用于抽象共享字段、谓词或编码结构。

### Lines 895-898
```tablegen
class MVEVectorIndex<int NumLanes> : Operand<i32> {
  let PrintMethod = "printVectorIndex";
  let ParserMatchClass = MVEVectorIndexOperand<NumLanes>;
}
```
- EN: Declares reusable TableGen class `MVEVectorIndex` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `MVEVectorIndex`，通常用于抽象共享字段、谓词或编码结构。

### Lines 900-913
```tablegen
// shift_imm: An integer that encodes a shift amount and the type of shift
// (asr or lsl). The 6-bit immediate encodes as:
//    {5}     0 ==> lsl
//            1     asr
//    {4-0}   imm5 shift amount.
//            asr #32 encoded as imm5 == 0.
def ShifterImmAsmOperand : AsmOperandClass {
  let Name = "ShifterImm";
  let ParserMethod = "parseShifterImm";
}
def shift_imm : Operand<i32> {
  let PrintMethod = "printShiftImmOperand";
  let ParserMatchClass = ShifterImmAsmOperand;
}
```
- EN: Defines TableGen record `ShifterImmAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ShifterImmAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 915-925
```tablegen
// shifter_operand operands: so_reg_reg, so_reg_imm, and mod_imm.
def ShiftedRegAsmOperand : AsmOperandClass { let Name = "RegShiftedReg"; }
def so_reg_reg : Operand<i32>,  // reg reg imm
                 ComplexPattern<i32, 3, "SelectRegShifterOperand",
                                [shl, srl, sra, rotr]> {
  let EncoderMethod = "getSORegRegOpValue";
  let PrintMethod = "printSORegRegOperand";
  let DecoderMethod = "DecodeSORegRegOperand";
  let ParserMatchClass = ShiftedRegAsmOperand;
  let MIOperandInfo = (ops GPRnopc, GPRnopc, i32imm);
}
```
- EN: Defines TableGen record `ShiftedRegAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ShiftedRegAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 927-936
```tablegen
def ShiftedImmAsmOperand : AsmOperandClass { let Name = "RegShiftedImm"; }
def so_reg_imm : Operand<i32>, // reg imm
                 ComplexPattern<i32, 2, "SelectImmShifterOperand",
                                [shl, srl, sra, rotr]> {
  let EncoderMethod = "getSORegImmOpValue";
  let PrintMethod = "printSORegImmOperand";
  let DecoderMethod = "DecodeSORegImmOperand";
  let ParserMatchClass = ShiftedImmAsmOperand;
  let MIOperandInfo = (ops GPR, i32imm);
}
```
- EN: Defines TableGen record `ShiftedImmAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ShiftedImmAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 938-947
```tablegen
// FIXME: Does this need to be distinct from so_reg?
def shift_so_reg_reg : Operand<i32>,    // reg reg imm
                   ComplexPattern<i32, 3, "SelectShiftRegShifterOperand",
                                  [shl,srl,sra,rotr]> {
  let EncoderMethod = "getSORegRegOpValue";
  let PrintMethod = "printSORegRegOperand";
  let DecoderMethod = "DecodeSORegRegOperand";
  let ParserMatchClass = ShiftedRegAsmOperand;
  let MIOperandInfo = (ops GPR, GPR, i32imm);
}
```
- EN: Defines TableGen record `shift_so_reg_reg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `shift_so_reg_reg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 949-958
```tablegen
// FIXME: Does this need to be distinct from so_reg?
def shift_so_reg_imm : Operand<i32>,    // reg reg imm
                   ComplexPattern<i32, 2, "SelectShiftImmShifterOperand",
                                  [shl,srl,sra,rotr]> {
  let EncoderMethod = "getSORegImmOpValue";
  let PrintMethod = "printSORegImmOperand";
  let DecoderMethod = "DecodeSORegImmOperand";
  let ParserMatchClass = ShiftedImmAsmOperand;
  let MIOperandInfo = (ops GPR, i32imm);
}
```
- EN: Defines TableGen record `shift_so_reg_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `shift_so_reg_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 960-974
```tablegen
// mod_imm: match a 32-bit immediate operand, which can be encoded into
// a 12-bit immediate; an 8-bit integer and a 4-bit rotator (See ARMARM
// - "Modified Immediate Constants"). Within the MC layer we keep this
// immediate in its encoded form.
def ModImmAsmOperand: AsmOperandClass {
  let Name = "ModImm";
  let ParserMethod = "parseModImm";
}
def mod_imm : Operand<i32>, ImmLeaf<i32, [{
    return ARM_AM::getSOImmVal(Imm) != -1;
  }]> {
  let EncoderMethod = "getModImmOpValue";
  let PrintMethod = "printModImmOperand";
  let ParserMatchClass = ModImmAsmOperand;
}
```
- EN: Defines TableGen record `ModImmAsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ModImmAsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 976-979
```tablegen
// Note: the patterns mod_imm_not and mod_imm_neg do not require an encoder
// method and such, as they are only used on aliases (Pat<> and InstAlias<>).
// The actual parsing, encoding, decoding are handled by the destination
// instructions, which use mod_imm.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 981-986
```tablegen
def ModImmNotAsmOperand : AsmOperandClass { let Name = "ModImmNot"; }
def mod_imm_not : Operand<i32>, ImmLeaf<i32, [{
    return ARM_AM::getSOImmVal(~(uint32_t)Imm) != -1;
  }], imm_not_XFORM> {
  let ParserMatchClass = ModImmNotAsmOperand;
}
```
- EN: Defines TableGen record `ModImmNotAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ModImmNotAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 988-994
```tablegen
def ModImmNegAsmOperand : AsmOperandClass { let Name = "ModImmNeg"; }
def mod_imm_neg : Operand<i32>, PatLeaf<(imm), [{
    unsigned Value = -(unsigned)N->getZExtValue();
    return Value && ARM_AM::getSOImmVal(Value) != -1;
  }], imm_neg_XFORM> {
  let ParserMatchClass = ModImmNegAsmOperand;
}
```
- EN: Defines TableGen record `ModImmNegAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ModImmNegAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 996-1003
```tablegen
/// arm_i32imm - True for +V6T2, or when isSOImmTwoParVal()
def arm_i32imm : IntImmLeaf<i32, [{
  if (Subtarget->useMovt())
    return true;
  if (ARM_AM::isSOImmTwoPartVal(Imm.getZExtValue()))
    return true;
  return ARM_AM::isSOImmTwoPartValNeg(Imm.getZExtValue());
}]>;
```
- EN: Defines TableGen record `arm_i32imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `arm_i32imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1005-1007
```tablegen
/// imm0_1 predicate - Immediate in the range [0,1].
def Imm0_1AsmOperand: ImmAsmOperand<0,1> { let Name = "Imm0_1"; }
def imm0_1 : Operand<i32> { let ParserMatchClass = Imm0_1AsmOperand; }
```
- EN: Defines TableGen record `Imm0_1AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_1AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1009-1011
```tablegen
/// imm0_3 predicate - Immediate in the range [0,3].
def Imm0_3AsmOperand: ImmAsmOperand<0,3> { let Name = "Imm0_3"; }
def imm0_3 : Operand<i32> { let ParserMatchClass = Imm0_3AsmOperand; }
```
- EN: Defines TableGen record `Imm0_3AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_3AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1013-1021
```tablegen
/// imm0_7 predicate - Immediate in the range [0,7].
def Imm0_7AsmOperand: ImmAsmOperand<0,7> {
  let Name = "Imm0_7";
}
def imm0_7 : Operand<i32>, ImmLeaf<i32, [{
  return Imm >= 0 && Imm < 8;
}]> {
  let ParserMatchClass = Imm0_7AsmOperand;
}
```
- EN: Defines TableGen record `Imm0_7AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_7AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1023-1029
```tablegen
/// imm8_255 predicate - Immediate in the range [8,255].
def Imm8_255AsmOperand: ImmAsmOperand<8,255> { let Name = "Imm8_255"; }
def imm8_255 : Operand<i32>, ImmLeaf<i32, [{
  return Imm >= 8 && Imm < 256;
}]> {
  let ParserMatchClass = Imm8_255AsmOperand;
}
```
- EN: Defines TableGen record `Imm8_255AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm8_255AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1031-1035
```tablegen
/// imm8 predicate - Immediate is exactly 8.
def Imm8AsmOperand: ImmAsmOperand<8,8> { let Name = "Imm8"; }
def imm8 : Operand<i32>, ImmLeaf<i32, [{ return Imm == 8; }]> {
  let ParserMatchClass = Imm8AsmOperand;
}
```
- EN: Defines TableGen record `Imm8AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm8AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1037-1041
```tablegen
/// imm16 predicate - Immediate is exactly 16.
def Imm16AsmOperand: ImmAsmOperand<16,16> { let Name = "Imm16"; }
def imm16 : Operand<i32>, ImmLeaf<i32, [{ return Imm == 16; }]> {
  let ParserMatchClass = Imm16AsmOperand;
}
```
- EN: Defines TableGen record `Imm16AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm16AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1043-1047
```tablegen
/// imm32 predicate - Immediate is exactly 32.
def Imm32AsmOperand: ImmAsmOperand<32,32> { let Name = "Imm32"; }
def imm32 : Operand<i32>, ImmLeaf<i32, [{ return Imm == 32; }]> {
  let ParserMatchClass = Imm32AsmOperand;
}
```
- EN: Defines TableGen record `Imm32AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm32AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1049-1049
```tablegen
def imm8_or_16 : ImmLeaf<i32, [{ return Imm == 8 || Imm == 16;}]>;
```
- EN: Defines TableGen record `imm8_or_16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm8_or_16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1051-1055
```tablegen
/// imm1_7 predicate - Immediate in the range [1,7].
def Imm1_7AsmOperand: ImmAsmOperand<1,7> { let Name = "Imm1_7"; }
def imm1_7 : Operand<i32>, ImmLeaf<i32, [{ return Imm > 0 && Imm < 8; }]> {
  let ParserMatchClass = Imm1_7AsmOperand;
}
```
- EN: Defines TableGen record `Imm1_7AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm1_7AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1057-1061
```tablegen
/// imm1_15 predicate - Immediate in the range [1,15].
def Imm1_15AsmOperand: ImmAsmOperand<1,15> { let Name = "Imm1_15"; }
def imm1_15 : Operand<i32>, ImmLeaf<i32, [{ return Imm > 0 && Imm < 16; }]> {
  let ParserMatchClass = Imm1_15AsmOperand;
}
```
- EN: Defines TableGen record `Imm1_15AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm1_15AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1063-1067
```tablegen
/// imm1_31 predicate - Immediate in the range [1,31].
def Imm1_31AsmOperand: ImmAsmOperand<1,31> { let Name = "Imm1_31"; }
def imm1_31 : Operand<i32>, ImmLeaf<i32, [{ return Imm > 0 && Imm < 32; }]> {
  let ParserMatchClass = Imm1_31AsmOperand;
}
```
- EN: Defines TableGen record `Imm1_31AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm1_31AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1069-1077
```tablegen
/// imm0_15 predicate - Immediate in the range [0,15].
def Imm0_15AsmOperand: ImmAsmOperand<0,15> {
  let Name = "Imm0_15";
}
def imm0_15 : Operand<i32>, ImmLeaf<i32, [{
  return Imm >= 0 && Imm < 16;
}]> {
  let ParserMatchClass = Imm0_15AsmOperand;
}
```
- EN: Defines TableGen record `Imm0_15AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_15AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1079-1085
```tablegen
/// imm0_31 predicate - True if the 32-bit immediate is in the range [0,31].
def Imm0_31AsmOperand: ImmAsmOperand<0,31> { let Name = "Imm0_31"; }
def imm0_31 : Operand<i32>, ImmLeaf<i32, [{
  return Imm >= 0 && Imm < 32;
}]> {
  let ParserMatchClass = Imm0_31AsmOperand;
}
```
- EN: Defines TableGen record `Imm0_31AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_31AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1087-1093
```tablegen
/// imm0_32 predicate - True if the 32-bit immediate is in the range [0,32].
def Imm0_32AsmOperand: ImmAsmOperand<0,32> { let Name = "Imm0_32"; }
def imm0_32 : Operand<i32>, ImmLeaf<i32, [{
  return Imm >= 0 && Imm < 33;
}]> {
  let ParserMatchClass = Imm0_32AsmOperand;
}
```
- EN: Defines TableGen record `Imm0_32AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_32AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1095-1101
```tablegen
/// imm0_63 predicate - True if the 32-bit immediate is in the range [0,63].
def Imm0_63AsmOperand: ImmAsmOperand<0,63> { let Name = "Imm0_63"; }
def imm0_63 : Operand<i32>, ImmLeaf<i32, [{
  return Imm >= 0 && Imm < 64;
}]> {
  let ParserMatchClass = Imm0_63AsmOperand;
}
```
- EN: Defines TableGen record `Imm0_63AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_63AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1103-1109
```tablegen
/// imm0_239 predicate - Immediate in the range [0,239].
def Imm0_239AsmOperand : ImmAsmOperand<0,239> {
  let Name = "Imm0_239";
}
def imm0_239 : Operand<i32>, ImmLeaf<i32, [{ return Imm >= 0 && Imm < 240; }]> {
  let ParserMatchClass = Imm0_239AsmOperand;
}
```
- EN: Defines TableGen record `Imm0_239AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_239AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1111-1115
```tablegen
/// imm0_255 predicate - Immediate in the range [0,255].
def Imm0_255AsmOperand : ImmAsmOperand<0,255> { let Name = "Imm0_255"; }
def imm0_255 : Operand<i32>, ImmLeaf<i32, [{ return Imm >= 0 && Imm < 256; }]> {
  let ParserMatchClass = Imm0_255AsmOperand;
}
```
- EN: Defines TableGen record `Imm0_255AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_255AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1117-1123
```tablegen
// imm0_255_expr - For Thumb1 movs/adds - 8-bit immediate that can also reference
// a relocatable expression.
def Imm0_255ExprAsmOperand: AsmOperandClass {
  let Name = "Imm0_255Expr";
  let RenderMethod = "addImmOperands";
  let DiagnosticString = "operand must be an immediate in the range [0,255] or a relocatable expression";
}
```
- EN: Defines TableGen record `Imm0_255ExprAsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_255ExprAsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1125-1128
```tablegen
def imm0_255_expr : Operand<i32>, ImmLeaf<i32, [{ return Imm >= 0 && Imm < 256; }]> {
  let EncoderMethod = "getHiLoImmOpValue";
  let ParserMatchClass = Imm0_255ExprAsmOperand;
}
```
- EN: Defines TableGen record `imm0_255_expr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm0_255_expr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1130-1136
```tablegen
/// imm0_65535 - An immediate is in the range [0,65535].
def Imm0_65535AsmOperand: ImmAsmOperand<0,65535> { let Name = "Imm0_65535"; }
def imm0_65535 : Operand<i32>, ImmLeaf<i32, [{
  return Imm >= 0 && Imm < 65536;
}]> {
  let ParserMatchClass = Imm0_65535AsmOperand;
}
```
- EN: Defines TableGen record `Imm0_65535AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_65535AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1138-1141
```tablegen
// imm0_65535_neg - An immediate whose negative value is in the range [0.65535].
def imm0_65535_neg : Operand<i32>, ImmLeaf<i32, [{
  return -Imm >= 0 && -Imm < 65536;
}]>;
```
- EN: Defines TableGen record `imm0_65535_neg` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm0_65535_neg`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1143-1153
```tablegen
// imm0_65535_expr - For movt/movw - 16-bit immediate that can also reference
// a relocatable expression.
//
// FIXME: This really needs a Thumb version separate from the ARM version.
// While the range is the same, and can thus use the same match class,
// the encoding is different so it should have a different encoder method.
def Imm0_65535ExprAsmOperand: AsmOperandClass {
  let Name = "Imm0_65535Expr";
  let RenderMethod = "addImmOperands";
  let DiagnosticString = "operand must be an immediate in the range [0,0xffff] or a relocatable expression";
}
```
- EN: Defines TableGen record `Imm0_65535ExprAsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm0_65535ExprAsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1155-1160
```tablegen
def imm0_65535_expr : Operand<i32>, ImmLeaf<i32, [{
  return Imm >= 0 && Imm < 65536;
}]> {
  let EncoderMethod = "getHiLoImmOpValue";
  let ParserMatchClass = Imm0_65535ExprAsmOperand;
}
```
- EN: Defines TableGen record `imm0_65535_expr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `imm0_65535_expr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1162-1165
```tablegen
def Imm256_65535ExprAsmOperand: ImmAsmOperand<256,65535> { let Name = "Imm256_65535Expr"; }
def imm256_65535_expr : Operand<i32> {
  let ParserMatchClass = Imm256_65535ExprAsmOperand;
}
```
- EN: Defines TableGen record `Imm256_65535ExprAsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm256_65535ExprAsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1167-1176
```tablegen
/// imm24b - True if the 32-bit immediate is encodable in 24 bits.
def Imm24bitAsmOperand: ImmAsmOperand<0,0xffffff> {
  let Name = "Imm24bit";
  let DiagnosticString = "operand must be an immediate in the range [0,0xffffff]";
}
def imm24b : Operand<i32>, ImmLeaf<i32, [{
  return Imm >= 0 && Imm <= 0xffffff;
}]> {
  let ParserMatchClass = Imm24bitAsmOperand;
}
```
- EN: Defines TableGen record `Imm24bitAsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Imm24bitAsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1179-1184
```tablegen
/// bf_inv_mask_imm predicate - An AND mask to clear an arbitrary width bitfield
/// e.g., 0xf000ffff
def BitfieldAsmOperand : AsmOperandClass {
  let Name = "Bitfield";
  let ParserMethod = "parseBitfield";
}
```
- EN: Defines TableGen record `BitfieldAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BitfieldAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1186-1203
```tablegen
def bf_inv_mask_imm : Operand<i32>,
                      PatLeaf<(imm), [{
  return ARM::isBitFieldInvertedMask(N->getZExtValue());
}] > {
  let EncoderMethod = "getBitfieldInvertedMaskOpValue";
  let PrintMethod = "printBitfieldInvMaskImmOperand";
  let DecoderMethod = "DecodeBitfieldMaskOperand";
  let ParserMatchClass = BitfieldAsmOperand;
  let GISelPredicateCode = [{
    // There's better methods of implementing this check. IntImmLeaf<> would be
    // equivalent and have less boilerplate but we need a test for C++
    // predicates and this one causes new rules to be imported into GlobalISel
    // without requiring additional features first.
    const auto &MO = MI.getOperand(1);
    if (!MO.isCImm())
      return false;
    return ARM::isBitFieldInvertedMask(MO.getCImm()->getZExtValue());
  }];
```
- EN: Defines TableGen record `bf_inv_mask_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `bf_inv_mask_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1204-1204
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1206-1220
```tablegen
def imm1_32_XFORM: SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant((int)N->getZExtValue() - 1, SDLoc(N),
                                   MVT::i32);
}]>;
def Imm1_32AsmOperand: ImmAsmOperandMinusOne<1,32> {
  let Name = "Imm1_32";
}
def imm1_32 : Operand<i32>, PatLeaf<(imm), [{
   uint64_t Imm = N->getZExtValue();
   return Imm > 0 && Imm <= 32;
 }],
    imm1_32_XFORM> {
  let PrintMethod = "printImmPlusOneOperand";
  let ParserMatchClass = Imm1_32AsmOperand;
}
```
- EN: Defines TableGen record `imm1_32_XFORM:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `imm1_32_XFORM:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1222-1233
```tablegen
def imm1_16_XFORM: SDNodeXForm<imm, [{
  return CurDAG->getTargetConstant((int)N->getZExtValue() - 1, SDLoc(N),
                                   MVT::i32);
}]>;
def Imm1_16AsmOperand: ImmAsmOperandMinusOne<1,16> { let Name = "Imm1_16"; }
def imm1_16 : Operand<i32>, ImmLeaf<i32, [{
    return Imm > 0 && Imm <= 16;
  }],
    imm1_16_XFORM> {
  let PrintMethod = "printImmPlusOneOperand";
  let ParserMatchClass = Imm1_16AsmOperand;
}
```
- EN: Defines TableGen record `imm1_16_XFORM:` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `imm1_16_XFORM:`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 1235-1249
```tablegen
def MVEShiftImm1_7AsmOperand: ImmAsmOperand<1,7> {
  let Name = "MVEShiftImm1_7";
  // Reason we're doing this is because instruction vshll.s8 t1 encoding
  // accepts 1,7 but the t2 encoding accepts 8.  By doing this we can get a
  // better diagnostic message if someone uses bigger immediate than the t1/t2
  // encodings allow.
  let DiagnosticString = "operand must be an immediate in the range [1,8]";
}
def mve_shift_imm1_7 : Operand<i32>,
    // SelectImmediateInRange / isScaledConstantInRange uses a
    // half-open interval, so the parameters <1,8> mean 1-7 inclusive
    ComplexPattern<i32, 1, "SelectImmediateInRange<1,8>", [], []> {
  let ParserMatchClass = MVEShiftImm1_7AsmOperand;
  let EncoderMethod = "getMVEShiftImmOpValue";
}
```
- EN: Defines TableGen record `MVEShiftImm1_7AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVEShiftImm1_7AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1251-1265
```tablegen
def MVEShiftImm1_15AsmOperand: ImmAsmOperand<1,15> {
  let Name = "MVEShiftImm1_15";
  // Reason we're doing this is because instruction vshll.s16 t1 encoding
  // accepts 1,15 but the t2 encoding accepts 16.  By doing this we can get a
  // better diagnostic message if someone uses bigger immediate than the t1/t2
  // encodings allow.
  let DiagnosticString = "operand must be an immediate in the range [1,16]";
}
def mve_shift_imm1_15 : Operand<i32>,
    // SelectImmediateInRange / isScaledConstantInRange uses a
    // half-open interval, so the parameters <1,16> mean 1-15 inclusive
    ComplexPattern<i32, 1, "SelectImmediateInRange<1,16>", [], []> {
  let ParserMatchClass = MVEShiftImm1_15AsmOperand;
  let EncoderMethod = "getMVEShiftImmOpValue";
}
```
- EN: Defines TableGen record `MVEShiftImm1_15AsmOperand:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVEShiftImm1_15AsmOperand:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1267-1275
```tablegen
// Define ARM specific addressing modes.
// addrmode_imm12 := reg +/- imm12
//
def MemImm12OffsetAsmOperand : AsmOperandClass { let Name = "MemImm12Offset"; }
class AddrMode_Imm12 : MemOperand,
                     ComplexPattern<i32, 2, "SelectAddrModeImm12", []> {
  // 12-bit immediate operand. Note that instructions using this encode
  // #0 and #-0 differently. We flag #-0 as the magic value INT32_MIN. All other
  // immediate values are as normal.
```
- EN: Declares reusable TableGen class `AddrMode_Imm12` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AddrMode_Imm12`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1277-1281
```tablegen
  let EncoderMethod = "getAddrModeImm12OpValue";
  let DecoderMethod = "DecodeAddrModeImm12Operand";
  let ParserMatchClass = MemImm12OffsetAsmOperand;
  let MIOperandInfo = (ops GPR:$base, i32imm:$offsimm);
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1283-1285
```tablegen
def addrmode_imm12 : AddrMode_Imm12 {
  let PrintMethod = "printAddrModeImm12Operand<false>";
}
```
- EN: Defines TableGen record `addrmode_imm12` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrmode_imm12`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1287-1289
```tablegen
def addrmode_imm12_pre : AddrMode_Imm12 {
  let PrintMethod = "printAddrModeImm12Operand<true>";
}
```
- EN: Defines TableGen record `addrmode_imm12_pre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrmode_imm12_pre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1291-1302
```tablegen
// ldst_so_reg := reg +/- reg shop imm
//
def MemRegOffsetAsmOperand : AsmOperandClass { let Name = "MemRegOffset"; }
def ldst_so_reg : MemOperand,
                  ComplexPattern<i32, 3, "SelectLdStSOReg", []> {
  let EncoderMethod = "getLdStSORegOpValue";
  // FIXME: Simplify the printer
  let PrintMethod = "printAddrMode2Operand";
  let DecoderMethod = "DecodeSORegMemOperand";
  let ParserMatchClass = MemRegOffsetAsmOperand;
  let MIOperandInfo = (ops GPR:$base, GPRnopc:$offsreg, i32imm:$shift);
}
```
- EN: Defines TableGen record `MemRegOffsetAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemRegOffsetAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1304-1313
```tablegen
// postidx_imm8 := +/- [0,255]
//
// 9 bit value:
//  {8}       1 is imm8 is non-negative. 0 otherwise.
//  {7-0}     [0,255] imm8 value.
def PostIdxImm8AsmOperand : AsmOperandClass { let Name = "PostIdxImm8"; }
def postidx_imm8 : MemOperand {
  let PrintMethod = "printPostIdxImm8Operand";
  let ParserMatchClass = PostIdxImm8AsmOperand;
}
```
- EN: Defines TableGen record `PostIdxImm8AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PostIdxImm8AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1315-1325
```tablegen
// postidx_imm8s4 := +/- [0,1020]
//
// 9 bit value:
//  {8}       1 is imm8 is non-negative. 0 otherwise.
//  {7-0}     [0,255] imm8 value, scaled by 4.
def PostIdxImm8s4AsmOperand : AsmOperandClass { let Name = "PostIdxImm8s4"; }
def postidx_imm8s4 : MemOperand {
  let PrintMethod = "printPostIdxImm8s4Operand";
  let ParserMatchClass = PostIdxImm8s4AsmOperand;
  let MIOperandInfo = (ops i32imm);
}
```
- EN: Defines TableGen record `PostIdxImm8s4AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PostIdxImm8s4AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1328-1340
```tablegen
// postidx_reg := +/- reg
//
def PostIdxRegAsmOperand : AsmOperandClass {
  let Name = "PostIdxReg";
  let ParserMethod = "parsePostIdxReg";
}
def postidx_reg : MemOperand {
  let EncoderMethod = "getPostIdxRegOpValue";
  let DecoderMethod = "DecodePostIdxReg";
  let PrintMethod = "printPostIdxRegOperand";
  let ParserMatchClass = PostIdxRegAsmOperand;
  let MIOperandInfo = (ops GPRnopc, i32imm);
}
```
- EN: Defines TableGen record `PostIdxRegAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PostIdxRegAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1342-1354
```tablegen
def PostIdxRegShiftedAsmOperand : AsmOperandClass {
  let Name = "PostIdxRegShifted";
  let ParserMethod = "parsePostIdxReg";
}
def am2offset_reg : MemOperand,
                    ComplexPattern<i32, 2, "SelectAddrMode2OffsetReg"> {
  let EncoderMethod = "getAddrMode2OffsetOpValue";
  let PrintMethod = "printAddrMode2OffsetOperand";
  // When using this for assembly, it's always as a post-index offset.
  let ParserMatchClass = PostIdxRegShiftedAsmOperand;
  let MIOperandInfo = (ops GPRnopc, i32imm);
  let WantsRoot = true;
}
```
- EN: Defines TableGen record `PostIdxRegShiftedAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PostIdxRegShiftedAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1356-1366
```tablegen
// FIXME: am2offset_imm should only need the immediate, not the GPR. Having
// the GPR is purely vestigal at this point.
def AM2OffsetImmAsmOperand : AsmOperandClass { let Name = "AM2OffsetImm"; }
def am2offset_imm : MemOperand,
                    ComplexPattern<i32, 2, "SelectAddrMode2OffsetImm"> {
  let EncoderMethod = "getAddrMode2OffsetOpValue";
  let PrintMethod = "printAddrMode2OffsetOperand";
  let ParserMatchClass = AM2OffsetImmAsmOperand;
  let MIOperandInfo = (ops GPRnopc, i32imm);
  let WantsRoot = true;
}
```
- EN: Defines TableGen record `AM2OffsetImmAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AM2OffsetImmAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1369-1379
```tablegen
// addrmode3 := reg +/- reg
// addrmode3 := reg +/- imm8
//
// FIXME: split into imm vs. reg versions.
def AddrMode3AsmOperand : AsmOperandClass { let Name = "AddrMode3"; }
class AddrMode3 : MemOperand,
                  ComplexPattern<i32, 3, "SelectAddrMode3", []> {
  let EncoderMethod = "getAddrMode3OpValue";
  let ParserMatchClass = AddrMode3AsmOperand;
  let MIOperandInfo = (ops GPR:$base, GPR:$offsreg, i32imm:$offsimm);
}
```
- EN: Declares reusable TableGen class `AddrMode3` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AddrMode3`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1381-1384
```tablegen
def addrmode3 : AddrMode3
{
  let PrintMethod = "printAddrMode3Operand<false>";
}
```
- EN: Defines TableGen record `addrmode3` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrmode3`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1386-1389
```tablegen
def addrmode3_pre : AddrMode3
{
  let PrintMethod = "printAddrMode3Operand<true>";
}
```
- EN: Defines TableGen record `addrmode3_pre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrmode3_pre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1391-1403
```tablegen
// FIXME: split into imm vs. reg versions.
// FIXME: parser method to handle +/- register.
def AM3OffsetAsmOperand : AsmOperandClass {
  let Name = "AM3Offset";
  let ParserMethod = "parseAM3Offset";
}
def am3offset : MemOperand, ComplexPattern<i32, 2, "SelectAddrMode3Offset"> {
  let EncoderMethod = "getAddrMode3OffsetOpValue";
  let PrintMethod = "printAddrMode3OffsetOperand";
  let ParserMatchClass = AM3OffsetAsmOperand;
  let MIOperandInfo = (ops GPR, i32imm);
  let WantsRoot = true;
}
```
- EN: Defines TableGen record `AM3OffsetAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AM3OffsetAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1405-1410
```tablegen
// ldstm_mode := {ia, ib, da, db}
//
def ldstm_mode : OptionalDefOperand<OtherVT, (ops i32), (ops (i32 1))> {
  let EncoderMethod = "getLdStmModeOpValue";
  let PrintMethod = "printLdStmModeOperand";
}
```
- EN: Defines TableGen record `ldstm_mode` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ldstm_mode`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1412-1421
```tablegen
// addrmode5 := reg +/- imm8*4
//
def AddrMode5AsmOperand : AsmOperandClass { let Name = "AddrMode5"; }
class AddrMode5 : MemOperand,
                  ComplexPattern<i32, 2, "SelectAddrMode5", []> {
  let EncoderMethod = "getAddrMode5OpValue";
  let DecoderMethod = "DecodeAddrMode5Operand";
  let ParserMatchClass = AddrMode5AsmOperand;
  let MIOperandInfo = (ops GPR:$base, i32imm);
}
```
- EN: Declares reusable TableGen class `AddrMode5` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AddrMode5`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1423-1425
```tablegen
def addrmode5 : AddrMode5 {
   let PrintMethod = "printAddrMode5Operand<false>";
}
```
- EN: Defines TableGen record `addrmode5` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrmode5`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1427-1429
```tablegen
def addrmode5_pre : AddrMode5 {
   let PrintMethod = "printAddrMode5Operand<true>";
}
```
- EN: Defines TableGen record `addrmode5_pre` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrmode5_pre`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1431-1440
```tablegen
// addrmode5fp16 := reg +/- imm8*2
//
def AddrMode5FP16AsmOperand : AsmOperandClass { let Name = "AddrMode5FP16"; }
class AddrMode5FP16 : MemOperand,
                      ComplexPattern<i32, 2, "SelectAddrMode5FP16", []> {
  let EncoderMethod = "getAddrMode5FP16OpValue";
  let DecoderMethod = "DecodeAddrMode5FP16Operand";
  let ParserMatchClass = AddrMode5FP16AsmOperand;
  let MIOperandInfo = (ops GPR:$base, i32imm);
}
```
- EN: Declares reusable TableGen class `AddrMode5FP16` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AddrMode5FP16`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1442-1444
```tablegen
def addrmode5fp16 : AddrMode5FP16 {
   let PrintMethod = "printAddrMode5FP16Operand<false>";
}
```
- EN: Defines TableGen record `addrmode5fp16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrmode5fp16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1446-1456
```tablegen
// addrmode6 := reg with optional alignment
//
def AddrMode6AsmOperand : AsmOperandClass { let Name = "AlignedMemory"; }
def addrmode6 : MemOperand, ComplexPattern<i32, 2, "SelectAddrMode6"> {
  let PrintMethod = "printAddrMode6Operand";
  let MIOperandInfo = (ops GPR:$addr, i32imm:$align);
  let EncoderMethod = "getAddrMode6AddressOpValue";
  let DecoderMethod = "DecodeAddrMode6Operand";
  let ParserMatchClass = AddrMode6AsmOperand;
  let WantsParent = true;
}
```
- EN: Defines TableGen record `AddrMode6AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1458-1464
```tablegen
def am6offset : MemOperand, ComplexPattern<i32, 1, "SelectAddrMode6Offset"> {
  let PrintMethod = "printAddrMode6OffsetOperand";
  let MIOperandInfo = (ops GPR);
  let EncoderMethod = "getAddrMode6OffsetOpValue";
  let DecoderMethod = "DecodeGPRRegisterClass";
  let WantsRoot = true;
}
```
- EN: Defines TableGen record `am6offset` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `am6offset`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1466-1473
```tablegen
// Special version of addrmode6 to handle alignment encoding for VST1/VLD1
// (single element from one lane) for size 32.
def addrmode6oneL32 : MemOperand, ComplexPattern<i32, 2, "SelectAddrMode6"> {
  let PrintMethod = "printAddrMode6Operand";
  let MIOperandInfo = (ops GPR:$addr, i32imm);
  let EncoderMethod = "getAddrMode6OneLane32AddressOpValue";
  let WantsParent = true;
}
```
- EN: Defines TableGen record `addrmode6oneL32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrmode6oneL32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1475-1482
```tablegen
// Base class for addrmode6 with specific alignment restrictions.
class AddrMode6Align : MemOperand, ComplexPattern<i32, 2, "SelectAddrMode6"> {
  let PrintMethod = "printAddrMode6Operand";
  let MIOperandInfo = (ops GPR:$addr, i32imm:$align);
  let EncoderMethod = "getAddrMode6AddressOpValue";
  let DecoderMethod = "DecodeAddrMode6Operand";
  let WantsParent = true;
}
```
- EN: Declares reusable TableGen class `for` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1484-1493
```tablegen
// Special version of addrmode6 to handle no allowed alignment encoding for
// VLD/VST instructions and checking the alignment is not specified.
def AddrMode6AlignNoneAsmOperand : AsmOperandClass {
  let Name = "AlignedMemoryNone";
  let DiagnosticString = "alignment must be omitted";
}
def addrmode6alignNone : AddrMode6Align {
  // The alignment specifier can only be omitted.
  let ParserMatchClass = AddrMode6AlignNoneAsmOperand;
}
```
- EN: Defines TableGen record `AddrMode6AlignNoneAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6AlignNoneAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1495-1504
```tablegen
// Special version of addrmode6 to handle 16-bit alignment encoding for
// VLD/VST instructions and checking the alignment value.
def AddrMode6Align16AsmOperand : AsmOperandClass {
  let Name = "AlignedMemory16";
  let DiagnosticString = "alignment must be 16 or omitted";
}
def addrmode6align16 : AddrMode6Align {
  // The alignment specifier can only be 16 or omitted.
  let ParserMatchClass = AddrMode6Align16AsmOperand;
}
```
- EN: Defines TableGen record `AddrMode6Align16AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6Align16AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1506-1515
```tablegen
// Special version of addrmode6 to handle 32-bit alignment encoding for
// VLD/VST instructions and checking the alignment value.
def AddrMode6Align32AsmOperand : AsmOperandClass {
  let Name = "AlignedMemory32";
  let DiagnosticString = "alignment must be 32 or omitted";
}
def addrmode6align32 : AddrMode6Align {
  // The alignment specifier can only be 32 or omitted.
  let ParserMatchClass = AddrMode6Align32AsmOperand;
}
```
- EN: Defines TableGen record `AddrMode6Align32AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6Align32AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1517-1526
```tablegen
// Special version of addrmode6 to handle 64-bit alignment encoding for
// VLD/VST instructions and checking the alignment value.
def AddrMode6Align64AsmOperand : AsmOperandClass {
  let Name = "AlignedMemory64";
  let DiagnosticString = "alignment must be 64 or omitted";
}
def addrmode6align64 : AddrMode6Align {
  // The alignment specifier can only be 64 or omitted.
  let ParserMatchClass = AddrMode6Align64AsmOperand;
}
```
- EN: Defines TableGen record `AddrMode6Align64AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6Align64AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1528-1537
```tablegen
// Special version of addrmode6 to handle 64-bit or 128-bit alignment encoding
// for VLD/VST instructions and checking the alignment value.
def AddrMode6Align64or128AsmOperand : AsmOperandClass {
  let Name = "AlignedMemory64or128";
  let DiagnosticString = "alignment must be 64, 128 or omitted";
}
def addrmode6align64or128 : AddrMode6Align {
  // The alignment specifier can only be 64, 128 or omitted.
  let ParserMatchClass = AddrMode6Align64or128AsmOperand;
}
```
- EN: Defines TableGen record `AddrMode6Align64or128AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6Align64or128AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1539-1548
```tablegen
// Special version of addrmode6 to handle 64-bit, 128-bit or 256-bit alignment
// encoding for VLD/VST instructions and checking the alignment value.
def AddrMode6Align64or128or256AsmOperand : AsmOperandClass {
  let Name = "AlignedMemory64or128or256";
  let DiagnosticString = "alignment must be 64, 128, 256 or omitted";
}
def addrmode6align64or128or256 : AddrMode6Align {
  // The alignment specifier can only be 64, 128, 256 or omitted.
  let ParserMatchClass = AddrMode6Align64or128or256AsmOperand;
}
```
- EN: Defines TableGen record `AddrMode6Align64or128or256AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6Align64or128or256AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1550-1560
```tablegen
// Special version of addrmode6 to handle alignment encoding for VLD-dup
// instructions, specifically VLD4-dup.
def addrmode6dup : MemOperand, ComplexPattern<i32, 2, "SelectAddrMode6"> {
  let PrintMethod = "printAddrMode6Operand";
  let MIOperandInfo = (ops GPR:$addr, i32imm);
  let EncoderMethod = "getAddrMode6DupAddressOpValue";
  // FIXME: This is close, but not quite right. The alignment specifier is
  // different.
  let ParserMatchClass = AddrMode6AsmOperand;
  let WantsParent = true;
}
```
- EN: Defines TableGen record `addrmode6dup` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrmode6dup`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1562-1569
```tablegen
// Base class for addrmode6dup with specific alignment restrictions.
class AddrMode6DupAlign : MemOperand,
                          ComplexPattern<i32, 2, "SelectAddrMode6"> {
  let PrintMethod = "printAddrMode6Operand";
  let MIOperandInfo = (ops GPR:$addr, i32imm);
  let EncoderMethod = "getAddrMode6DupAddressOpValue";
  let WantsParent = true;
}
```
- EN: Declares reusable TableGen class `for` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1571-1580
```tablegen
// Special version of addrmode6 to handle no allowed alignment encoding for
// VLD-dup instruction and checking the alignment is not specified.
def AddrMode6dupAlignNoneAsmOperand : AsmOperandClass {
  let Name = "DupAlignedMemoryNone";
  let DiagnosticString = "alignment must be omitted";
}
def addrmode6dupalignNone : AddrMode6DupAlign {
  // The alignment specifier can only be omitted.
  let ParserMatchClass = AddrMode6dupAlignNoneAsmOperand;
}
```
- EN: Defines TableGen record `AddrMode6dupAlignNoneAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6dupAlignNoneAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1582-1591
```tablegen
// Special version of addrmode6 to handle 16-bit alignment encoding for VLD-dup
// instruction and checking the alignment value.
def AddrMode6dupAlign16AsmOperand : AsmOperandClass {
  let Name = "DupAlignedMemory16";
  let DiagnosticString = "alignment must be 16 or omitted";
}
def addrmode6dupalign16 : AddrMode6DupAlign {
  // The alignment specifier can only be 16 or omitted.
  let ParserMatchClass = AddrMode6dupAlign16AsmOperand;
}
```
- EN: Defines TableGen record `AddrMode6dupAlign16AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6dupAlign16AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1593-1602
```tablegen
// Special version of addrmode6 to handle 32-bit alignment encoding for VLD-dup
// instruction and checking the alignment value.
def AddrMode6dupAlign32AsmOperand : AsmOperandClass {
  let Name = "DupAlignedMemory32";
  let DiagnosticString = "alignment must be 32 or omitted";
}
def addrmode6dupalign32 : AddrMode6DupAlign {
  // The alignment specifier can only be 32 or omitted.
  let ParserMatchClass = AddrMode6dupAlign32AsmOperand;
}
```
- EN: Defines TableGen record `AddrMode6dupAlign32AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6dupAlign32AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1604-1613
```tablegen
// Special version of addrmode6 to handle 64-bit alignment encoding for VLD
// instructions and checking the alignment value.
def AddrMode6dupAlign64AsmOperand : AsmOperandClass {
  let Name = "DupAlignedMemory64";
  let DiagnosticString = "alignment must be 64 or omitted";
}
def addrmode6dupalign64 : AddrMode6DupAlign {
  // The alignment specifier can only be 64 or omitted.
  let ParserMatchClass = AddrMode6dupAlign64AsmOperand;
}
```
- EN: Defines TableGen record `AddrMode6dupAlign64AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6dupAlign64AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1615-1624
```tablegen
// Special version of addrmode6 to handle 64-bit or 128-bit alignment encoding
// for VLD instructions and checking the alignment value.
def AddrMode6dupAlign64or128AsmOperand : AsmOperandClass {
  let Name = "DupAlignedMemory64or128";
  let DiagnosticString = "alignment must be 64, 128 or omitted";
}
def addrmode6dupalign64or128 : AddrMode6DupAlign {
  // The alignment specifier can only be 64, 128 or omitted.
  let ParserMatchClass = AddrMode6dupAlign64or128AsmOperand;
}
```
- EN: Defines TableGen record `AddrMode6dupAlign64or128AsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AddrMode6dupAlign64or128AsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1626-1632
```tablegen
// addrmodepc := pc + reg
//
def addrmodepc : MemOperand,
                 ComplexPattern<i32, 2, "SelectAddrModePC", []> {
  let PrintMethod = "printAddrModePCOperand";
  let MIOperandInfo = (ops GPR, i32imm);
}
```
- EN: Defines TableGen record `addrmodepc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `addrmodepc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1634-1643
```tablegen
// addr_offset_none := reg
//
def MemNoOffsetAsmOperand : AsmOperandClass { let Name = "MemNoOffset"; }
def addr_offset_none : MemOperand,
                       ComplexPattern<i32, 1, "SelectAddrOffsetNone", []> {
  let PrintMethod = "printAddrMode7Operand";
  let DecoderMethod = "DecodeAddrMode7Operand";
  let ParserMatchClass = MemNoOffsetAsmOperand;
  let MIOperandInfo = (ops GPR:$base);
}
```
- EN: Defines TableGen record `MemNoOffsetAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemNoOffsetAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1645-1652
```tablegen
// t_addr_offset_none := reg [r0-r7]
def MemNoOffsetTAsmOperand : AsmOperandClass { let Name = "MemNoOffsetT"; }
def t_addr_offset_none : MemOperand {
  let PrintMethod = "printAddrMode7Operand";
  let DecoderMethod = "DecodetGPRRegisterClass";
  let ParserMatchClass = MemNoOffsetTAsmOperand;
  let MIOperandInfo = (ops tGPR:$base);
}
```
- EN: Defines TableGen record `MemNoOffsetTAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemNoOffsetTAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1654-1656
```tablegen
def nohash_imm : Operand<i32> {
  let PrintMethod = "printNoHashImmediate";
}
```
- EN: Defines TableGen record `nohash_imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `nohash_imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1658-1666
```tablegen
def CoprocNumAsmOperand : AsmOperandClass {
  let Name = "CoprocNum";
  let ParserMethod = "parseCoprocNumOperand";
}
def p_imm : Operand<i32> {
  let PrintMethod = "printPImmediate";
  let ParserMatchClass = CoprocNumAsmOperand;
  let DecoderMethod = "DecodeCoprocessor";
}
```
- EN: Defines TableGen record `CoprocNumAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CoprocNumAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1668-1683
```tablegen
def CoprocRegAsmOperand : AsmOperandClass {
  let Name = "CoprocReg";
  let ParserMethod = "parseCoprocRegOperand";
}
def c_imm : Operand<i32> {
  let PrintMethod = "printCImmediate";
  let ParserMatchClass = CoprocRegAsmOperand;
}
def CoprocOptionAsmOperand : AsmOperandClass {
  let Name = "CoprocOption";
  let ParserMethod = "parseCoprocOptionOperand";
}
def coproc_option_imm : Operand<i32> {
  let PrintMethod = "printCoprocOptionImm";
  let ParserMatchClass = CoprocOptionAsmOperand;
}
```
- EN: Defines TableGen record `CoprocRegAsmOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CoprocRegAsmOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1685-1685
```tablegen
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1687-1687
```tablegen
include "ARMInstrFormats.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 1689-1691
```tablegen
//===----------------------------------------------------------------------===//
// Multiclass helpers...
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 1693-1710
```tablegen
/// AsI1_bin_irs - Defines a set of (op r, {mod_imm|r|so_reg}) patterns for a
/// binop that produces a value.
let TwoOperandAliasConstraint = "$Rn = $Rd" in
multiclass AsI1_bin_irs<bits<4> opcod, string opc,
                     InstrItinClass iii, InstrItinClass iir, InstrItinClass iis,
                     SDPatternOperator opnode, bit Commutable = 0> {
  // The register-immediate version is re-materializable. This is useful
  // in particular for taking the address of a local.
  let isReMaterializable = 1 in {
  def ri : AsI1<opcod, (outs GPR:$Rd), (ins GPR:$Rn, mod_imm:$imm), DPFrm,
               iii, opc, "\t$Rd, $Rn, $imm",
               [(set GPR:$Rd, (opnode GPR:$Rn, mod_imm:$imm))]>,
           Sched<[WriteALU, ReadALU]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<12> imm;
    let Inst{25} = 1;
    let Inst{19-16} = Rn;
```
- EN: Declares TableGen `multiclass AsI1_bin_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AsI1_bin_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1711-1728
```tablegen
    let Inst{15-12} = Rd;
    let Inst{11-0} = imm;
  }
  }
  def rr : AsI1<opcod, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm), DPFrm,
               iir, opc, "\t$Rd, $Rn, $Rm",
               [(set GPR:$Rd, (opnode GPR:$Rn, GPR:$Rm))]>,
           Sched<[WriteALU, ReadALU, ReadALU]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<4> Rm;
    let Inst{25} = 0;
    let isCommutable = Commutable;
    let Inst{19-16} = Rn;
    let Inst{15-12} = Rd;
    let Inst{11-4} = 0b00000000;
    let Inst{3-0} = Rm;
  }
```
- EN: Defines TableGen record `rr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1730-1744
```tablegen
  def rsi : AsI1<opcod, (outs GPR:$Rd),
               (ins GPR:$Rn, so_reg_imm:$shift), DPSoRegImmFrm,
               iis, opc, "\t$Rd, $Rn, $shift",
               [(set GPR:$Rd, (opnode GPR:$Rn, so_reg_imm:$shift))]>,
            Sched<[WriteALUsi, ReadALU]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<12> shift;
    let Inst{25} = 0;
    let Inst{19-16} = Rn;
    let Inst{15-12} = Rd;
    let Inst{11-5} = shift{11-5};
    let Inst{4} = 0;
    let Inst{3-0} = shift{3-0};
  }
```
- EN: Defines TableGen record `rsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1746-1763
```tablegen
  def rsr : AsI1<opcod, (outs GPR:$Rd),
               (ins GPR:$Rn, so_reg_reg:$shift), DPSoRegRegFrm,
               iis, opc, "\t$Rd, $Rn, $shift",
               [(set GPR:$Rd, (opnode GPR:$Rn, so_reg_reg:$shift))]>,
            Sched<[WriteALUsr, ReadALUsr]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<12> shift;
    let Inst{25} = 0;
    let Inst{19-16} = Rn;
    let Inst{15-12} = Rd;
    let Inst{11-8} = shift{11-8};
    let Inst{7} = 0;
    let Inst{6-5} = shift{6-5};
    let Inst{4} = 1;
    let Inst{3-0} = shift{3-0};
  }
}
```
- EN: Defines TableGen record `rsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1765-1782
```tablegen
/// AsI1_rbin_irs - Same as AsI1_bin_irs except the order of operands are
/// reversed.  The 'rr' form is only defined for the disassembler; for codegen
/// it is equivalent to the AsI1_bin_irs counterpart.
let TwoOperandAliasConstraint = "$Rn = $Rd" in
multiclass AsI1_rbin_irs<bits<4> opcod, string opc,
                     InstrItinClass iii, InstrItinClass iir, InstrItinClass iis,
                     SDNode opnode> {
  // The register-immediate version is re-materializable. This is useful
  // in particular for taking the address of a local.
  let isReMaterializable = 1 in {
  def ri : AsI1<opcod, (outs GPR:$Rd), (ins GPR:$Rn, mod_imm:$imm), DPFrm,
               iii, opc, "\t$Rd, $Rn, $imm",
               [(set GPR:$Rd, (opnode mod_imm:$imm, GPR:$Rn))]>,
           Sched<[WriteALU, ReadALU]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<12> imm;
    let Inst{25} = 1;
```
- EN: Declares TableGen `multiclass AsI1_rbin_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AsI1_rbin_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1783-1800
```tablegen
    let Inst{19-16} = Rn;
    let Inst{15-12} = Rd;
    let Inst{11-0} = imm;
  }
  }
  def rr : AsI1<opcod, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm), DPFrm,
               iir, opc, "\t$Rd, $Rn, $Rm",
               [/* pattern left blank */]>,
           Sched<[WriteALU, ReadALU, ReadALU]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<4> Rm;
    let Inst{11-4} = 0b00000000;
    let Inst{25} = 0;
    let Inst{3-0} = Rm;
    let Inst{15-12} = Rd;
    let Inst{19-16} = Rn;
    let hasSideEffects = 0;
```
- EN: Defines TableGen record `rr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1801-1801
```tablegen
  }
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1803-1817
```tablegen
  def rsi : AsI1<opcod, (outs GPR:$Rd),
               (ins GPR:$Rn, so_reg_imm:$shift), DPSoRegImmFrm,
               iis, opc, "\t$Rd, $Rn, $shift",
               [(set GPR:$Rd, (opnode so_reg_imm:$shift, GPR:$Rn))]>,
            Sched<[WriteALUsi, ReadALU]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<12> shift;
    let Inst{25} = 0;
    let Inst{19-16} = Rn;
    let Inst{15-12} = Rd;
    let Inst{11-5} = shift{11-5};
    let Inst{4} = 0;
    let Inst{3-0} = shift{3-0};
  }
```
- EN: Defines TableGen record `rsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1819-1836
```tablegen
  def rsr : AsI1<opcod, (outs GPR:$Rd),
               (ins GPR:$Rn, so_reg_reg:$shift), DPSoRegRegFrm,
               iis, opc, "\t$Rd, $Rn, $shift",
               [(set GPR:$Rd, (opnode so_reg_reg:$shift, GPR:$Rn))]>,
            Sched<[WriteALUsr, ReadALUsr]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<12> shift;
    let Inst{25} = 0;
    let Inst{19-16} = Rn;
    let Inst{15-12} = Rd;
    let Inst{11-8} = shift{11-8};
    let Inst{7} = 0;
    let Inst{6-5} = shift{6-5};
    let Inst{4} = 1;
    let Inst{3-0} = shift{3-0};
  }
}
```
- EN: Defines TableGen record `rsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1838-1849
```tablegen
/// AsI1_bin_s_irs - Same as AsI1_bin_irs except it sets the 's' bit by default.
///
/// These opcodes will be converted to the real non-S opcodes by
/// AdjustInstrPostInstrSelection after giving them an optional CPSR operand.
let hasPostISelHook = 1, Defs = [CPSR] in {
multiclass AsI1_bin_s_irs<InstrItinClass iii, InstrItinClass iir,
                          InstrItinClass iis, SDNode opnode,
                          bit Commutable = 0> {
  def ri : ARMPseudoInst<(outs GPR:$Rd), (ins GPR:$Rn, mod_imm:$imm, pred:$p),
                         4, iii,
                         [(set GPR:$Rd, CPSR, (opnode GPR:$Rn, mod_imm:$imm))]>,
                         Sched<[WriteALU, ReadALU]>;
```
- EN: Declares TableGen `multiclass AsI1_bin_s_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AsI1_bin_s_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1851-1862
```tablegen
  def rr : ARMPseudoInst<(outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm, pred:$p),
                         4, iir,
                         [(set GPR:$Rd, CPSR, (opnode GPR:$Rn, GPR:$Rm))]>,
                         Sched<[WriteALU, ReadALU, ReadALU]> {
    let isCommutable = Commutable;
  }
  def rsi : ARMPseudoInst<(outs GPR:$Rd),
                          (ins GPR:$Rn, so_reg_imm:$shift, pred:$p),
                          4, iis,
                          [(set GPR:$Rd, CPSR, (opnode GPR:$Rn,
                                                so_reg_imm:$shift))]>,
                          Sched<[WriteALUsi, ReadALU]>;
```
- EN: Defines TableGen record `rr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1864-1871
```tablegen
  def rsr : ARMPseudoInst<(outs GPR:$Rd),
                          (ins GPR:$Rn, so_reg_reg:$shift, pred:$p),
                          4, iis,
                          [(set GPR:$Rd, CPSR, (opnode GPR:$Rn,
                                                so_reg_reg:$shift))]>,
                          Sched<[WriteALUSsr, ReadALUsr]>;
}
}
```
- EN: Defines TableGen record `rsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1873-1881
```tablegen
/// AsI1_rbin_s_is - Same as AsI1_bin_s_irs, except selection DAG
/// operands are reversed.
let hasPostISelHook = 1, Defs = [CPSR] in {
multiclass AsI1_rbin_s_is<InstrItinClass iii,
                          InstrItinClass iis, SDNode opnode> {
  def ri : ARMPseudoInst<(outs GPR:$Rd), (ins GPR:$Rn, mod_imm:$imm, pred:$p),
                         4, iii,
                         [(set GPR:$Rd, CPSR, (opnode mod_imm:$imm, GPR:$Rn))]>,
           Sched<[WriteALU, ReadALU]>;
```
- EN: Declares TableGen `multiclass AsI1_rbin_s_is`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AsI1_rbin_s_is`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1883-1888
```tablegen
  def rsi : ARMPseudoInst<(outs GPR:$Rd),
                          (ins GPR:$Rn, so_reg_imm:$shift, pred:$p),
                          4, iis,
                          [(set GPR:$Rd, CPSR, (opnode so_reg_imm:$shift,
                                             GPR:$Rn))]>,
            Sched<[WriteALUsi, ReadALU]>;
```
- EN: Defines TableGen record `rsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1890-1897
```tablegen
  def rsr : ARMPseudoInst<(outs GPR:$Rd),
                          (ins GPR:$Rn, so_reg_reg:$shift, pred:$p),
                          4, iis,
                          [(set GPR:$Rd, CPSR, (opnode so_reg_reg:$shift,
                                             GPR:$Rn))]>,
            Sched<[WriteALUSsr, ReadALUsr]>;
}
}
```
- EN: Defines TableGen record `rsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1899-1916
```tablegen
/// AI1_cmp_irs - Defines a set of (op r, {mod_imm|r|so_reg}) cmp / test
/// patterns. Similar to AsI1_bin_irs except the instruction does not produce
/// a explicit result, only implicitly set CPSR.
let isCompare = 1, Defs = [CPSR] in {
multiclass AI1_cmp_irs<bits<4> opcod, string opc,
                     InstrItinClass iii, InstrItinClass iir, InstrItinClass iis,
                     SDPatternOperator opnode, bit Commutable = 0,
                     string rrDecoderMethod = ""> {
  def ri : AI1<opcod, (outs), (ins GPR:$Rn, mod_imm:$imm), DPFrm, iii,
               opc, "\t$Rn, $imm",
               [(set CPSR, (opnode GPR:$Rn, mod_imm:$imm))]>,
           Sched<[WriteCMP, ReadALU]> {
    bits<4> Rn;
    bits<12> imm;
    let Inst{25} = 1;
    let Inst{20} = 1;
    let Inst{19-16} = Rn;
    let Inst{15-12} = 0b0000;
```
- EN: Declares TableGen `multiclass AI1_cmp_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI1_cmp_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 1917-1917
```tablegen
    let Inst{11-0} = imm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1919-1934
```tablegen
    let Unpredictable{15-12} = 0b1111;
  }
  def rr : AI1<opcod, (outs), (ins GPR:$Rn, GPR:$Rm), DPFrm, iir,
               opc, "\t$Rn, $Rm",
               [(set CPSR, (opnode GPR:$Rn, GPR:$Rm))]>,
           Sched<[WriteCMP, ReadALU, ReadALU]> {
    bits<4> Rn;
    bits<4> Rm;
    let isCommutable = Commutable;
    let Inst{25} = 0;
    let Inst{20} = 1;
    let Inst{19-16} = Rn;
    let Inst{15-12} = 0b0000;
    let Inst{11-4} = 0b00000000;
    let Inst{3-0} = Rm;
    let DecoderMethod = rrDecoderMethod;
```
- EN: Defines TableGen record `rr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1936-1951
```tablegen
    let Unpredictable{15-12} = 0b1111;
  }
  def rsi : AI1<opcod, (outs),
               (ins GPR:$Rn, so_reg_imm:$shift), DPSoRegImmFrm, iis,
               opc, "\t$Rn, $shift",
               [(set CPSR, (opnode GPR:$Rn, so_reg_imm:$shift))]>,
            Sched<[WriteCMPsi, ReadALU]> {
    bits<4> Rn;
    bits<12> shift;
    let Inst{25} = 0;
    let Inst{20} = 1;
    let Inst{19-16} = Rn;
    let Inst{15-12} = 0b0000;
    let Inst{11-5} = shift{11-5};
    let Inst{4} = 0;
    let Inst{3-0} = shift{3-0};
```
- EN: Defines TableGen record `rsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1953-1970
```tablegen
    let Unpredictable{15-12} = 0b1111;
  }
  def rsr : AI1<opcod, (outs),
               (ins GPRnopc:$Rn, so_reg_reg:$shift), DPSoRegRegFrm, iis,
               opc, "\t$Rn, $shift",
               [(set CPSR, (opnode GPRnopc:$Rn, so_reg_reg:$shift))]>,
            Sched<[WriteCMPsr, ReadALU]> {
    bits<4> Rn;
    bits<12> shift;
    let Inst{25} = 0;
    let Inst{20} = 1;
    let Inst{19-16} = Rn;
    let Inst{15-12} = 0b0000;
    let Inst{11-8} = shift{11-8};
    let Inst{7} = 0;
    let Inst{6-5} = shift{6-5};
    let Inst{4} = 1;
    let Inst{3-0} = shift{3-0};
```
- EN: Defines TableGen record `rsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 1972-1973
```tablegen
    let Unpredictable{15-12} = 0b1111;
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 1975-1976
```tablegen
}
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 1978-1993
```tablegen
/// AI_ext_rrot - A unary operation with two forms: one whose operand is a
/// register and one whose operand is a register rotated by 8/16/24.
/// FIXME: Remove the 'r' variant. Its rot_imm is zero.
class AI_ext_rrot<bits<8> opcod, string opc, PatFrag opnode>
  : AExtI<opcod, (outs GPRnopc:$Rd), (ins GPRnopc:$Rm, rot_imm:$rot),
          IIC_iEXTr, opc, "\t$Rd, $Rm$rot",
          [(set GPRnopc:$Rd, (opnode (rotr GPRnopc:$Rm, rot_imm:$rot)))]>,
       Requires<[IsARM, HasV6]>, Sched<[WriteALUsi]> {
  bits<4> Rd;
  bits<4> Rm;
  bits<2> rot;
  let Inst{19-16} = 0b1111;
  let Inst{15-12} = Rd;
  let Inst{11-10} = rot;
  let Inst{3-0}   = Rm;
}
```
- EN: Declares reusable TableGen class `AI_ext_rrot` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AI_ext_rrot`，通常用于抽象共享字段、谓词或编码结构。

### Lines 1995-2002
```tablegen
class AI_ext_rrot_np<bits<8> opcod, string opc>
  : AExtI<opcod, (outs GPRnopc:$Rd), (ins GPRnopc:$Rm, rot_imm:$rot),
          IIC_iEXTr, opc, "\t$Rd, $Rm$rot", []>,
       Requires<[IsARM, HasV6]>, Sched<[WriteALUsi]> {
  bits<2> rot;
  let Inst{19-16} = 0b1111;
  let Inst{11-10} = rot;
 }
```
- EN: Declares reusable TableGen class `AI_ext_rrot_np` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AI_ext_rrot_np`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2004-2021
```tablegen
/// AI_exta_rrot - A binary operation with two forms: one whose operand is a
/// register and one whose operand is a register rotated by 8/16/24.
class AI_exta_rrot<bits<8> opcod, string opc, PatFrag opnode>
  : AExtI<opcod, (outs GPRnopc:$Rd), (ins GPR:$Rn, GPRnopc:$Rm, rot_imm:$rot),
          IIC_iEXTAr, opc, "\t$Rd, $Rn, $Rm$rot",
          [(set GPRnopc:$Rd, (opnode GPR:$Rn,
                                     (rotr GPRnopc:$Rm, rot_imm:$rot)))]>,
        Requires<[IsARM, HasV6]>, Sched<[WriteALUsr]> {
  bits<4> Rd;
  bits<4> Rm;
  bits<4> Rn;
  bits<2> rot;
  let Inst{19-16} = Rn;
  let Inst{15-12} = Rd;
  let Inst{11-10} = rot;
  let Inst{9-4}   = 0b000111;
  let Inst{3-0}   = Rm;
}
```
- EN: Declares reusable TableGen class `AI_exta_rrot` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AI_exta_rrot`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2023-2031
```tablegen
class AI_exta_rrot_np<bits<8> opcod, string opc>
  : AExtI<opcod, (outs GPRnopc:$Rd), (ins GPR:$Rn, GPRnopc:$Rm, rot_imm:$rot),
          IIC_iEXTAr, opc, "\t$Rd, $Rn, $Rm$rot", []>,
       Requires<[IsARM, HasV6]>, Sched<[WriteALUsr]> {
  bits<4> Rn;
  bits<2> rot;
  let Inst{19-16} = Rn;
  let Inst{11-10} = rot;
}
```
- EN: Declares reusable TableGen class `AI_exta_rrot_np` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AI_exta_rrot_np`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2033-2050
```tablegen
/// AI1_adde_sube_irs - Define instructions and patterns for adde and sube.
let TwoOperandAliasConstraint = "$Rn = $Rd" in
multiclass AI1_adde_sube_irs<bits<4> opcod, string opc, SDNode opnode,
                             bit Commutable = 0> {
  let hasPostISelHook = 1, Defs = [CPSR], Uses = [CPSR] in {
  def ri : AsI1<opcod, (outs GPR:$Rd), (ins GPR:$Rn, mod_imm:$imm),
                DPFrm, IIC_iALUi, opc, "\t$Rd, $Rn, $imm",
               [(set GPR:$Rd, CPSR, (opnode GPR:$Rn, mod_imm:$imm, CPSR))]>,
               Requires<[IsARM]>,
           Sched<[WriteALU, ReadALU]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<12> imm;
    let Inst{25} = 1;
    let Inst{15-12} = Rd;
    let Inst{19-16} = Rn;
    let Inst{11-0} = imm;
  }
```
- EN: Declares TableGen `multiclass AI1_adde_sube_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI1_adde_sube_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2051-2068
```tablegen
  def rr : AsI1<opcod, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
                DPFrm, IIC_iALUr, opc, "\t$Rd, $Rn, $Rm",
               [(set GPR:$Rd, CPSR, (opnode GPR:$Rn, GPR:$Rm, CPSR))]>,
               Requires<[IsARM]>,
           Sched<[WriteALU, ReadALU, ReadALU]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<4> Rm;
    let Inst{11-4} = 0b00000000;
    let Inst{25} = 0;
    let isCommutable = Commutable;
    let Inst{3-0} = Rm;
    let Inst{15-12} = Rd;
    let Inst{19-16} = Rn;
  }
  def rsi : AsI1<opcod, (outs GPR:$Rd),
                (ins GPR:$Rn, so_reg_imm:$shift),
                DPSoRegImmFrm, IIC_iALUsr, opc, "\t$Rd, $Rn, $shift",
```
- EN: Defines TableGen record `rr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2069-2086
```tablegen
              [(set GPR:$Rd, CPSR, (opnode GPR:$Rn, so_reg_imm:$shift, CPSR))]>,
               Requires<[IsARM]>,
            Sched<[WriteALUsi, ReadALU]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<12> shift;
    let Inst{25} = 0;
    let Inst{19-16} = Rn;
    let Inst{15-12} = Rd;
    let Inst{11-5} = shift{11-5};
    let Inst{4} = 0;
    let Inst{3-0} = shift{3-0};
  }
  def rsr : AsI1<opcod, (outs GPRnopc:$Rd),
                (ins GPRnopc:$Rn, so_reg_reg:$shift),
                DPSoRegRegFrm, IIC_iALUsr, opc, "\t$Rd, $Rn, $shift",
              [(set GPRnopc:$Rd, CPSR,
                    (opnode GPRnopc:$Rn, so_reg_reg:$shift, CPSR))]>,
```
- EN: Defines TableGen record `rsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2087-2102
```tablegen
               Requires<[IsARM]>,
            Sched<[WriteALUsr, ReadALUsr]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<12> shift;
    let Inst{25} = 0;
    let Inst{19-16} = Rn;
    let Inst{15-12} = Rd;
    let Inst{11-8} = shift{11-8};
    let Inst{7} = 0;
    let Inst{6-5} = shift{6-5};
    let Inst{4} = 1;
    let Inst{3-0} = shift{3-0};
  }
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2104-2121
```tablegen
/// AI1_rsc_irs - Define instructions and patterns for rsc
let TwoOperandAliasConstraint = "$Rn = $Rd" in
multiclass AI1_rsc_irs<bits<4> opcod, string opc, SDNode opnode> {
  let hasPostISelHook = 1, Defs = [CPSR], Uses = [CPSR] in {
  def ri : AsI1<opcod, (outs GPR:$Rd), (ins GPR:$Rn, mod_imm:$imm),
                DPFrm, IIC_iALUi, opc, "\t$Rd, $Rn, $imm",
               [(set GPR:$Rd, CPSR, (opnode mod_imm:$imm, GPR:$Rn, CPSR))]>,
               Requires<[IsARM]>,
           Sched<[WriteALU, ReadALU]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<12> imm;
    let Inst{25} = 1;
    let Inst{15-12} = Rd;
    let Inst{19-16} = Rn;
    let Inst{11-0} = imm;
  }
  def rr : AsI1<opcod, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
```
- EN: Declares TableGen `multiclass AI1_rsc_irs`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI1_rsc_irs`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2122-2139
```tablegen
                DPFrm, IIC_iALUr, opc, "\t$Rd, $Rn, $Rm",
               [/* pattern left blank */]>,
           Sched<[WriteALU, ReadALU, ReadALU]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<4> Rm;
    let Inst{11-4} = 0b00000000;
    let Inst{25} = 0;
    let Inst{3-0} = Rm;
    let Inst{15-12} = Rd;
    let Inst{19-16} = Rn;
  }
  def rsi : AsI1<opcod, (outs GPR:$Rd), (ins GPR:$Rn, so_reg_imm:$shift),
                DPSoRegImmFrm, IIC_iALUsr, opc, "\t$Rd, $Rn, $shift",
              [(set GPR:$Rd, CPSR, (opnode so_reg_imm:$shift, GPR:$Rn, CPSR))]>,
               Requires<[IsARM]>,
            Sched<[WriteALUsi, ReadALU]> {
    bits<4> Rd;
```
- EN: Defines TableGen record `rsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2140-2157
```tablegen
    bits<4> Rn;
    bits<12> shift;
    let Inst{25} = 0;
    let Inst{19-16} = Rn;
    let Inst{15-12} = Rd;
    let Inst{11-5} = shift{11-5};
    let Inst{4} = 0;
    let Inst{3-0} = shift{3-0};
  }
  def rsr : AsI1<opcod, (outs GPR:$Rd), (ins GPR:$Rn, so_reg_reg:$shift),
                DPSoRegRegFrm, IIC_iALUsr, opc, "\t$Rd, $Rn, $shift",
              [(set GPR:$Rd, CPSR, (opnode so_reg_reg:$shift, GPR:$Rn, CPSR))]>,
               Requires<[IsARM]>,
            Sched<[WriteALUsr, ReadALUsr]> {
    bits<4> Rd;
    bits<4> Rn;
    bits<12> shift;
    let Inst{25} = 0;
```
- EN: Defines TableGen record `rsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2158-2167
```tablegen
    let Inst{19-16} = Rn;
    let Inst{15-12} = Rd;
    let Inst{11-8} = shift{11-8};
    let Inst{7} = 0;
    let Inst{6-5} = shift{6-5};
    let Inst{4} = 1;
    let Inst{3-0} = shift{3-0};
  }
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2169-2186
```tablegen
let canFoldAsLoad = 1, isReMaterializable = 1 in {
multiclass AI_ldr1<bit isByte, string opc, InstrItinClass iii,
           InstrItinClass iir, PatFrag opnode> {
  // Note: We use the complex addrmode_imm12 rather than just an input
  // GPR and a constrained immediate so that we can use this to match
  // frame index references and avoid matching constant pool references.
  def i12: AI2ldst<0b010, 1, isByte, (outs GPR:$Rt), (ins addrmode_imm12:$addr),
                   AddrMode_i12, LdFrm, iii, opc, "\t$Rt, $addr",
                  [(set GPR:$Rt, (opnode addrmode_imm12:$addr))]> {
    bits<4>  Rt;
    bits<17> addr;
    let Inst{23}    = addr{12};     // U (add = ('U' == 1))
    let Inst{19-16} = addr{16-13};  // Rn
    let Inst{15-12} = Rt;
    let Inst{11-0}  = addr{11-0};   // imm12
  }
  def rs : AI2ldst<0b011, 1, isByte, (outs GPR:$Rt), (ins ldst_so_reg:$shift),
                  AddrModeNone, LdFrm, iir, opc, "\t$Rt, $shift",
```
- EN: Declares TableGen `multiclass AI_ldr1`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI_ldr1`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2187-2197
```tablegen
                 [(set GPR:$Rt, (opnode ldst_so_reg:$shift))]> {
    bits<4>  Rt;
    bits<17> shift;
    let shift{4}    = 0;            // Inst{4} = 0
    let Inst{23}    = shift{12};    // U (add = ('U' == 1))
    let Inst{19-16} = shift{16-13}; // Rn
    let Inst{15-12} = Rt;
    let Inst{11-0}  = shift{11-0};
  }
}
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2199-2216
```tablegen
let canFoldAsLoad = 1, isReMaterializable = 1 in {
multiclass AI_ldr1nopc<bit isByte, string opc, InstrItinClass iii,
           InstrItinClass iir, PatFrag opnode> {
  // Note: We use the complex addrmode_imm12 rather than just an input
  // GPR and a constrained immediate so that we can use this to match
  // frame index references and avoid matching constant pool references.
  def i12: AI2ldst<0b010, 1, isByte, (outs GPRnopc:$Rt),
                   (ins addrmode_imm12:$addr),
                   AddrMode_i12, LdFrm, iii, opc, "\t$Rt, $addr",
                   [(set GPRnopc:$Rt, (opnode addrmode_imm12:$addr))]> {
    bits<4>  Rt;
    bits<17> addr;
    let Inst{23}    = addr{12};     // U (add = ('U' == 1))
    let Inst{19-16} = addr{16-13};  // Rn
    let Inst{15-12} = Rt;
    let Inst{11-0}  = addr{11-0};   // imm12
  }
  def rs : AI2ldst<0b011, 1, isByte, (outs GPRnopc:$Rt),
```
- EN: Declares TableGen `multiclass AI_ldr1nopc`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI_ldr1nopc`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2217-2229
```tablegen
                   (ins ldst_so_reg:$shift),
                   AddrModeNone, LdFrm, iir, opc, "\t$Rt, $shift",
                   [(set GPRnopc:$Rt, (opnode ldst_so_reg:$shift))]> {
    bits<4>  Rt;
    bits<17> shift;
    let shift{4}    = 0;            // Inst{4} = 0
    let Inst{23}    = shift{12};    // U (add = ('U' == 1))
    let Inst{19-16} = shift{16-13}; // Rn
    let Inst{15-12} = Rt;
    let Inst{11-0}  = shift{11-0};
  }
}
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2232-2249
```tablegen
multiclass AI_str1<bit isByte, string opc, InstrItinClass iii,
           InstrItinClass iir, PatFrag opnode> {
  // Note: We use the complex addrmode_imm12 rather than just an input
  // GPR and a constrained immediate so that we can use this to match
  // frame index references and avoid matching constant pool references.
  def i12 : AI2ldst<0b010, 0, isByte, (outs),
                   (ins GPR:$Rt, addrmode_imm12:$addr),
                   AddrMode_i12, StFrm, iii, opc, "\t$Rt, $addr",
                  [(opnode GPR:$Rt, addrmode_imm12:$addr)]> {
    bits<4> Rt;
    bits<17> addr;
    let Inst{23}    = addr{12};     // U (add = ('U' == 1))
    let Inst{19-16} = addr{16-13};  // Rn
    let Inst{15-12} = Rt;
    let Inst{11-0}  = addr{11-0};   // imm12
  }
  def rs : AI2ldst<0b011, 0, isByte, (outs), (ins GPR:$Rt, ldst_so_reg:$shift),
                  AddrModeNone, StFrm, iir, opc, "\t$Rt, $shift",
```
- EN: Declares TableGen `multiclass AI_str1`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI_str1`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2250-2259
```tablegen
                 [(opnode GPR:$Rt, ldst_so_reg:$shift)]> {
    bits<4> Rt;
    bits<17> shift;
    let shift{4}    = 0;            // Inst{4} = 0
    let Inst{23}    = shift{12};    // U (add = ('U' == 1))
    let Inst{19-16} = shift{16-13}; // Rn
    let Inst{15-12} = Rt;
    let Inst{11-0}  = shift{11-0};
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2261-2278
```tablegen
multiclass AI_str1nopc<bit isByte, string opc, InstrItinClass iii,
           InstrItinClass iir, PatFrag opnode> {
  // Note: We use the complex addrmode_imm12 rather than just an input
  // GPR and a constrained immediate so that we can use this to match
  // frame index references and avoid matching constant pool references.
  def i12 : AI2ldst<0b010, 0, isByte, (outs),
                   (ins GPRnopc:$Rt, addrmode_imm12:$addr),
                   AddrMode_i12, StFrm, iii, opc, "\t$Rt, $addr",
                  [(opnode GPRnopc:$Rt, addrmode_imm12:$addr)]> {
    bits<4> Rt;
    bits<17> addr;
    let Inst{23}    = addr{12};     // U (add = ('U' == 1))
    let Inst{19-16} = addr{16-13};  // Rn
    let Inst{15-12} = Rt;
    let Inst{11-0}  = addr{11-0};   // imm12
  }
  def rs : AI2ldst<0b011, 0, isByte, (outs),
                   (ins GPRnopc:$Rt, ldst_so_reg:$shift),
```
- EN: Declares TableGen `multiclass AI_str1nopc`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI_str1nopc`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2279-2289
```tablegen
                   AddrModeNone, StFrm, iir, opc, "\t$Rt, $shift",
                   [(opnode GPRnopc:$Rt, ldst_so_reg:$shift)]> {
    bits<4> Rt;
    bits<17> shift;
    let shift{4}    = 0;            // Inst{4} = 0
    let Inst{23}    = shift{12};    // U (add = ('U' == 1))
    let Inst{19-16} = shift{16-13}; // Rn
    let Inst{15-12} = Rt;
    let Inst{11-0}  = shift{11-0};
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2292-2294
```tablegen
//===----------------------------------------------------------------------===//
// Instructions
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2296-2298
```tablegen
//===----------------------------------------------------------------------===//
//  Miscellaneous Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2300-2307
```tablegen
/// CONSTPOOL_ENTRY - This instruction represents a floating constant pool in
/// the function.  The first operand is the ID# for this instruction, the second
/// is the index into the MachineConstantPool that this is, the third is the
/// size in bytes of this constant pool entry.
let hasSideEffects = 0, isNotDuplicable = 1, hasNoSchedulingInfo = 1 in
def CONSTPOOL_ENTRY :
PseudoInst<(outs), (ins cpinst_operand:$instid, cpinst_operand:$cpidx,
                    i32imm:$size), NoItinerary, []>;
```
- EN: Defines TableGen record `CONSTPOOL_ENTRY` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CONSTPOOL_ENTRY`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2309-2314
```tablegen
/// A jumptable consisting of direct 32-bit addresses of the destination basic
/// blocks (either absolute, or relative to the start of the jump-table in PIC
/// mode). Used mostly in ARM and Thumb-1 modes.
def JUMPTABLE_ADDRS :
PseudoInst<(outs), (ins cpinst_operand:$instid, cpinst_operand:$cpidx,
                        i32imm:$size), NoItinerary, []>;
```
- EN: Defines TableGen record `JUMPTABLE_ADDRS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JUMPTABLE_ADDRS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2316-2320
```tablegen
/// A jumptable consisting of 32-bit jump instructions. Used for Thumb-2 tables
/// that cannot be optimised to use TBB or TBH.
def JUMPTABLE_INSTS :
PseudoInst<(outs), (ins cpinst_operand:$instid, cpinst_operand:$cpidx,
                        i32imm:$size), NoItinerary, []>;
```
- EN: Defines TableGen record `JUMPTABLE_INSTS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JUMPTABLE_INSTS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2322-2326
```tablegen
/// A jumptable consisting of 8-bit unsigned integers representing offsets from
/// a TBB instruction.
def JUMPTABLE_TBB :
PseudoInst<(outs), (ins cpinst_operand:$instid, cpinst_operand:$cpidx,
                        i32imm:$size), NoItinerary, []>;
```
- EN: Defines TableGen record `JUMPTABLE_TBB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JUMPTABLE_TBB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2328-2332
```tablegen
/// A jumptable consisting of 16-bit unsigned integers representing offsets from
/// a TBH instruction.
def JUMPTABLE_TBH :
PseudoInst<(outs), (ins cpinst_operand:$instid, cpinst_operand:$cpidx,
                        i32imm:$size), NoItinerary, []>;
```
- EN: Defines TableGen record `JUMPTABLE_TBH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `JUMPTABLE_TBH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2335-2341
```tablegen
// FIXME: Marking these as hasSideEffects is necessary to prevent machine DCE
// from removing one half of the matched pairs. That breaks PEI, which assumes
// these will always be in pairs, and asserts if it finds otherwise. Better way?
let Defs = [SP], Uses = [SP], hasSideEffects = 1 in {
def ADJCALLSTACKUP :
PseudoInst<(outs), (ins i32imm:$amt1, i32imm:$amt2, pred:$p), NoItinerary,
           [(ARMcallseq_end timm:$amt1, timm:$amt2)]>;
```
- EN: Defines TableGen record `ADJCALLSTACKUP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADJCALLSTACKUP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2343-2346
```tablegen
def ADJCALLSTACKDOWN :
PseudoInst<(outs), (ins i32imm:$amt, i32imm:$amt2, pred:$p), NoItinerary,
           [(ARMcallseq_start timm:$amt, timm:$amt2)]>;
}
```
- EN: Defines TableGen record `ADJCALLSTACKDOWN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADJCALLSTACKDOWN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2348-2355
```tablegen
def HINT : AI<(outs), (ins imm0_239:$imm), MiscFrm, NoItinerary,
              "hint", "\t$imm", [(int_arm_hint imm0_239:$imm)]>,
           Requires<[IsARM, HasV6]> {
  bits<8> imm;
  let Inst{27-8} = 0b00110010000011110000;
  let Inst{7-0} = imm;
  let DecoderMethod = "DecodeHINTInstruction";
}
```
- EN: Defines TableGen record `HINT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HINT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2357-2365
```tablegen
// Architectural ARM-mode NOP is a HINT from ARMv6K; plain ARMv6 uses MOV (see MOV nop alias).
def : InstAlias<"nop$p", (HINT 0, pred:$p)>, Requires<[IsARM, HasV6K]>;
def : InstAlias<"yield$p", (HINT 1, pred:$p)>, Requires<[IsARM, HasV6K]>;
def : InstAlias<"wfe$p", (HINT 2, pred:$p)>, Requires<[IsARM, HasV6K]>;
def : InstAlias<"wfi$p", (HINT 3, pred:$p)>, Requires<[IsARM, HasV6K]>;
def : InstAlias<"sev$p", (HINT 4, pred:$p)>, Requires<[IsARM, HasV6K]>;
def : InstAlias<"sevl$p", (HINT 5, pred:$p)>, Requires<[IsARM, HasV8]>;
def : InstAlias<"esb$p", (HINT 16, pred:$p)>, Requires<[IsARM, HasRAS]>;
def : InstAlias<"csdb$p", (HINT 20, pred:$p)>, Requires<[IsARM, HasV6K]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2367-2369
```tablegen
// Clear BHB instruction
def : InstAlias<"clrbhb$p", (HINT 22, pred:$p), 0>, Requires<[IsARM, HasV8]>;
def : InstAlias<"clrbhb$p", (HINT 22, pred:$p), 1>, Requires<[IsARM, HasV8, HasCLRBHB]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2371-2385
```tablegen
def SEL : AI<(outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm), DPFrm, NoItinerary, "sel",
             "\t$Rd, $Rn, $Rm",
             [(set GPR:$Rd, (int_arm_sel GPR:$Rn, GPR:$Rm))]>,
             Requires<[IsARM, HasV6]> {
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
  let Inst{3-0} = Rm;
  let Inst{15-12} = Rd;
  let Inst{19-16} = Rn;
  let Inst{27-20} = 0b01101000;
  let Inst{7-4} = 0b1011;
  let Inst{11-8} = 0b1111;
  let Unpredictable{11-8} = 0b1111;
}
```
- EN: Defines TableGen record `SEL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SEL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2387-2399
```tablegen
// The 16-bit operand $val can be used by a debugger to store more information
// about the breakpoint.
def BKPT : AInoP<(outs), (ins imm0_65535:$val), MiscFrm, NoItinerary,
                 "bkpt", "\t$val", []>, Requires<[IsARM]> {
  bits<16> val;
  let Inst{3-0} = val{3-0};
  let Inst{19-8} = val{15-4};
  let Inst{27-20} = 0b00010010;
  let Inst{31-28} = 0xe; // AL
  let Inst{7-4} = 0b0111;
}
// default immediate for breakpoint mnemonic
def : InstAlias<"bkpt", (BKPT 0), 0>, Requires<[IsARM]>;
```
- EN: Defines TableGen record `BKPT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BKPT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2401-2409
```tablegen
def HLT : AInoP<(outs), (ins imm0_65535:$val), MiscFrm, NoItinerary,
                 "hlt", "\t$val", []>, Requires<[IsARM, HasV8]> {
  bits<16> val;
  let Inst{3-0} = val{3-0};
  let Inst{19-8} = val{15-4};
  let Inst{27-20} = 0b00010000;
  let Inst{31-28} = 0xe; // AL
  let Inst{7-4} = 0b0111;
}
```
- EN: Defines TableGen record `HLT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HLT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2411-2419
```tablegen
// Change Processor State
// FIXME: We should use InstAlias to handle the optional operands.
class CPS<dag iops, string asm_ops>
  : AXI<(outs), iops, MiscFrm, NoItinerary, !strconcat("cps", asm_ops),
        []>, Requires<[IsARM]> {
  bits<2> imod;
  bits<3> iflags;
  bits<5> mode;
  bit M;
```
- EN: Declares reusable TableGen class `CPS` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `CPS`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2421-2429
```tablegen
  let Inst{31-28} = 0b1111;
  let Inst{27-20} = 0b00010000;
  let Inst{19-18} = imod;
  let Inst{17}    = M; // Enabled if mode is set;
  let Inst{16-9}  = 0b00000000;
  let Inst{8-6}   = iflags;
  let Inst{5}     = 0;
  let Inst{4-0}   = mode;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2431-2436
```tablegen
let DecoderMethod = "DecodeCPSInstruction" in {
let M = 1 in
  def CPS3p : CPS<(ins imod_op:$imod, iflags_op:$iflags, imm0_31:$mode),
                  "$imod\t$iflags, $mode">;
let mode = 0, M = 0 in
  def CPS2p : CPS<(ins imod_op:$imod, iflags_op:$iflags), "$imod\t$iflags">;
```
- EN: Defines TableGen record `CPS3p` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CPS3p`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2438-2440
```tablegen
let imod = 0, iflags = 0, M = 1 in
  def CPS1p : CPS<(ins imm0_31:$mode), "\t$mode">;
}
```
- EN: Defines TableGen record `CPS1p` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CPS1p`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2442-2443
```tablegen
// Preload signals the memory system of possible future data/instruction access.
multiclass APreLoad<bits<1> read, bits<1> data, string opc> {
```
- EN: Declares TableGen `multiclass APreLoad`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass APreLoad`，它是一个可复用模板，可展开为多个相关记录。

### Lines 2445-2460
```tablegen
  def i12 : AXIM<(outs), (ins addrmode_imm12:$addr), AddrMode_i12, MiscFrm,
                IIC_Preload, !strconcat(opc, "\t$addr"),
                [(ARMPreload addrmode_imm12:$addr, (i32 read), (i32 data))]>,
                Sched<[WritePreLd]> {
    bits<4> Rt;
    bits<17> addr;
    let Inst{31-26} = 0b111101;
    let Inst{25} = 0; // 0 for immediate form
    let Inst{24} = data;
    let Inst{23} = addr{12};        // U (add = ('U' == 1))
    let Inst{22} = read;
    let Inst{21-20} = 0b01;
    let Inst{19-16} = addr{16-13};  // Rn
    let Inst{15-12} = 0b1111;
    let Inst{11-0}  = addr{11-0};   // imm12
  }
```
- EN: Defines TableGen record `i12` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `i12`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2462-2478
```tablegen
  def rs : AXI<(outs), (ins ldst_so_reg:$shift), MiscFrm, IIC_Preload,
               !strconcat(opc, "\t$shift"),
               [(ARMPreload ldst_so_reg:$shift, (i32 read), (i32 data))]>,
               Sched<[WritePreLd]> {
    bits<17> shift;
    let Inst{31-26} = 0b111101;
    let Inst{25} = 1; // 1 for register form
    let Inst{24} = data;
    let Inst{23} = shift{12};    // U (add = ('U' == 1))
    let Inst{22} = read;
    let Inst{21-20} = 0b01;
    let Inst{19-16} = shift{16-13}; // Rn
    let Inst{15-12} = 0b1111;
    let Inst{11-0}  = shift{11-0};
    let Inst{4} = 0;
  }
}
```
- EN: Defines TableGen record `rs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `rs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2480-2482
```tablegen
defm PLD  : APreLoad<1, 1, "pld">,  Requires<[IsARM]>;
defm PLDW : APreLoad<0, 1, "pldw">, Requires<[IsARM,HasV7,HasMP]>;
defm PLI  : APreLoad<1, 0, "pli">,  Requires<[IsARM,HasV7]>;
```
- EN: Defines TableGen record `PLD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PLD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2484-2490
```tablegen
def SETEND : AXI<(outs), (ins setend_op:$end), MiscFrm, NoItinerary,
                 "setend\t$end", []>, Requires<[IsARM]>, Deprecated<HasV8Ops> {
  bits<1> end;
  let Inst{31-10} = 0b1111000100000001000000;
  let Inst{9} = end;
  let Inst{8-0} = 0;
}
```
- EN: Defines TableGen record `SETEND` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SETEND`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2492-2497
```tablegen
def DBG : AI<(outs), (ins imm0_15:$opt), MiscFrm, NoItinerary, "dbg", "\t$opt",
             [(int_arm_dbg imm0_15:$opt)]>, Requires<[IsARM, HasV7]> {
  bits<4> opt;
  let Inst{27-4} = 0b001100100000111100001111;
  let Inst{3-0} = opt;
}
```
- EN: Defines TableGen record `DBG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DBG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2499-2509
```tablegen
// A8.8.247  UDF - Undefined (Encoding A1)
def UDF : AInoP<(outs), (ins imm0_65535:$imm16), MiscFrm, NoItinerary,
                "udf", "\t$imm16", [(int_arm_undefined imm0_65535:$imm16)]> {
  bits<16> imm16;
  let Inst{31-28} = 0b1110; // AL
  let Inst{27-25} = 0b011;
  let Inst{24-20} = 0b11111;
  let Inst{19-8} = imm16{15-4};
  let Inst{7-4} = 0b1111;
  let Inst{3-0} = imm16{3-0};
}
```
- EN: Defines TableGen record `UDF` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UDF`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2511-2522
```tablegen
/*
 * A5.4 Permanently UNDEFINED instructions.
 *
 * Targets use UDF #65006, for which the OS will generate SIGTRAP.
 *
 */
let isTrap = 1 in
def TRAP : AXI<(outs), (ins), MiscFrm, NoItinerary,
               "trap", [(trap)]>,
           Requires<[IsARM]> {
  let Inst = 0xe7ffdefe;
}
```
- EN: Defines TableGen record `TRAP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TRAP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2524-2525
```tablegen
def : Pat<(debugtrap), (BKPT 0)>, Requires<[IsARM, HasV5T]>;
def : Pat<(debugtrap), (UDF 254)>, Requires<[IsARM, NoV5T]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2527-2532
```tablegen
// Address computation and loads and stores in PIC mode.
let isNotDuplicable = 1 in {
def PICADD  : ARMPseudoInst<(outs GPR:$dst), (ins GPR:$a, pclabel:$cp, pred:$p),
                            4, IIC_iALUr,
                            [(set GPR:$dst, (ARMpic_add GPR:$a, imm:$cp))]>,
                            Sched<[WriteALU, ReadALU]>;
```
- EN: Defines TableGen record `PICADD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PICADD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2534-2537
```tablegen
let AddedComplexity = 10 in {
def PICLDR  : ARMPseudoInst<(outs GPR:$dst), (ins addrmodepc:$addr, pred:$p),
                            4, IIC_iLoad_r,
                            [(set GPR:$dst, (load addrmodepc:$addr))]>;
```
- EN: Defines TableGen record `PICLDR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PICLDR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2539-2541
```tablegen
def PICLDRH : ARMPseudoInst<(outs GPR:$Rt), (ins addrmodepc:$addr, pred:$p),
                            4, IIC_iLoad_bh_r,
                            [(set GPR:$Rt, (zextloadi16 addrmodepc:$addr))]>;
```
- EN: Defines TableGen record `PICLDRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PICLDRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2543-2545
```tablegen
def PICLDRB : ARMPseudoInst<(outs GPR:$Rt), (ins addrmodepc:$addr, pred:$p),
                            4, IIC_iLoad_bh_r,
                            [(set GPR:$Rt, (zextloadi8 addrmodepc:$addr))]>;
```
- EN: Defines TableGen record `PICLDRB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PICLDRB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2547-2549
```tablegen
def PICLDRSH : ARMPseudoInst<(outs GPR:$Rt), (ins addrmodepc:$addr, pred:$p),
                            4, IIC_iLoad_bh_r,
                            [(set GPR:$Rt, (sextloadi16 addrmodepc:$addr))]>;
```
- EN: Defines TableGen record `PICLDRSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PICLDRSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2551-2557
```tablegen
def PICLDRSB : ARMPseudoInst<(outs GPR:$Rt), (ins addrmodepc:$addr, pred:$p),
                            4, IIC_iLoad_bh_r,
                            [(set GPR:$Rt, (sextloadi8 addrmodepc:$addr))]>;
}
let AddedComplexity = 10 in {
def PICSTR  : ARMPseudoInst<(outs), (ins GPR:$src, addrmodepc:$addr, pred:$p),
      4, IIC_iStore_r, [(store GPR:$src, addrmodepc:$addr)]>;
```
- EN: Defines TableGen record `PICLDRSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PICLDRSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2559-2561
```tablegen
def PICSTRH : ARMPseudoInst<(outs), (ins GPR:$src, addrmodepc:$addr, pred:$p),
      4, IIC_iStore_bh_r, [(truncstorei16 GPR:$src,
                                                   addrmodepc:$addr)]>;
```
- EN: Defines TableGen record `PICSTRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PICSTRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2563-2566
```tablegen
def PICSTRB : ARMPseudoInst<(outs), (ins GPR:$src, addrmodepc:$addr, pred:$p),
      4, IIC_iStore_bh_r, [(truncstorei8 GPR:$src, addrmodepc:$addr)]>;
}
} // isNotDuplicable = 1
```
- EN: Defines TableGen record `PICSTRB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PICSTRB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2569-2586
```tablegen
// LEApcrel - Load a pc-relative address into a register without offending the
// assembler.
let hasSideEffects = 0, isReMaterializable = 1 in
// The 'adr' mnemonic encodes differently if the label is before or after
// the instruction. The {24-21} opcode bits are set by the fixup, as we don't
// know until then which form of the instruction will be used.
def ADR : AI1<{0,?,?,0}, (outs GPR:$Rd), (ins adrlabel:$label),
                 MiscFrm, IIC_iALUi, "adr", "\t$Rd, $label", []>,
                 Sched<[WriteALU, ReadALU]> {
  bits<4> Rd;
  bits<14> label;
  let Inst{27-25} = 0b001;
  let Inst{24} = 0;
  let Inst{23-22} = label{13-12};
  let Inst{21} = 0;
  let Inst{20} = 0;
  let Inst{19-16} = 0b1111;
  let Inst{15-12} = Rd;
```
- EN: Defines TableGen record `ADR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2587-2588
```tablegen
  let Inst{11-0} = label{11-0};
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2590-2592
```tablegen
let hasSideEffects = 1 in {
def LEApcrel : ARMPseudoInst<(outs GPR:$Rd), (ins i32imm:$label, pred:$p),
                    4, IIC_iALUi, []>, Sched<[WriteALU, ReadALU]>;
```
- EN: Defines TableGen record `LEApcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LEApcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2594-2597
```tablegen
def LEApcrelJT : ARMPseudoInst<(outs GPR:$Rd),
                      (ins i32imm:$label, pred:$p),
                      4, IIC_iALUi, []>, Sched<[WriteALU, ReadALU]>;
}
```
- EN: Defines TableGen record `LEApcrelJT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LEApcrelJT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2599-2601
```tablegen
//===----------------------------------------------------------------------===//
//  Control Flow Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2603-2609
```tablegen
let isReturn = 1, isTerminator = 1, isBarrier = 1 in {
  // ARMV4T and above
  def BX_RET : AI<(outs), (ins), BrMiscFrm, IIC_Br,
                  "bx", "\tlr", [(ARMretglue)]>,
               Requires<[IsARM, HasV4T]>, Sched<[WriteBr]> {
    let Inst{27-0}  = 0b0001001011111111111100011110;
  }
```
- EN: Defines TableGen record `BX_RET` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BX_RET`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2611-2616
```tablegen
  // ARMV4 only
  def MOVPCLR : AI<(outs), (ins), BrMiscFrm, IIC_Br,
                  "mov", "\tpc, lr", [(ARMretglue)]>,
               Requires<[IsARM, NoV4T]>, Sched<[WriteBr]> {
    let Inst{27-0} = 0b0001101000001111000000001110;
  }
```
- EN: Defines TableGen record `MOVPCLR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVPCLR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2618-2623
```tablegen
  // Exception return: N.b. doesn't set CPSR as far as we're concerned (it sets
  // the user-space one).
  def SUBS_PC_LR : ARMPseudoInst<(outs), (ins i32imm:$offset, pred:$p),
                                 4, IIC_Br,
                                 [(ARMintretglue imm:$offset)]>;
}
```
- EN: Defines TableGen record `SUBS_PC_LR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SUBS_PC_LR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2625-2634
```tablegen
// Indirect branches
let isBranch = 1, isTerminator = 1, isBarrier = 1, isIndirectBranch = 1 in {
  // ARMV4T and above
  def BX : AXI<(outs), (ins GPR:$dst), BrMiscFrm, IIC_Br, "bx\t$dst",
                  [(brind GPR:$dst)]>,
              Requires<[IsARM, HasV4T]>, Sched<[WriteBr]> {
    bits<4> dst;
    let Inst{31-4} = 0b1110000100101111111111110001;
    let Inst{3-0}  = dst;
  }
```
- EN: Defines TableGen record `BX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2636-2643
```tablegen
  def BX_pred : AI<(outs), (ins GPR:$dst), BrMiscFrm, IIC_Br,
                  "bx", "\t$dst", [/* pattern left blank */]>,
              Requires<[IsARM, HasV4T]>, Sched<[WriteBr]> {
    bits<4> dst;
    let Inst{27-4} = 0b000100101111111111110001;
    let Inst{3-0}  = dst;
  }
}
```
- EN: Defines TableGen record `BX_pred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BX_pred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2645-2660
```tablegen
// SP is marked as a use to prevent stack-pointer assignments that appear
// immediately before calls from potentially appearing dead.
let isCall = 1,
  // FIXME:  Do we really need a non-predicated version? If so, it should
  // at least be a pseudo instruction expanding to the predicated version
  // at MC lowering time.
  Defs = [LR], Uses = [SP] in {
  def BL  : ABXI<0b1011, (outs), (ins arm_bl_target:$func),
                IIC_Br, "bl\t$func",
                [(ARMcall tglobaladdr:$func)]>,
            Requires<[IsARM]>, Sched<[WriteBrL]> {
    let Inst{31-28} = 0b1110;
    bits<24> func;
    let Inst{23-0} = func;
    let DecoderMethod = "DecodeBranchImmInstruction";
  }
```
- EN: Defines TableGen record `BL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2662-2669
```tablegen
  def BL_pred : ABI<0b1011, (outs), (ins arm_bl_target:$func),
                   IIC_Br, "bl", "\t$func",
                   [(ARMcall_pred tglobaladdr:$func)]>,
                Requires<[IsARM]>, Sched<[WriteBrL]> {
    bits<24> func;
    let Inst{23-0} = func;
    let DecoderMethod = "DecodeBranchImmInstruction";
  }
```
- EN: Defines TableGen record `BL_pred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BL_pred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2671-2680
```tablegen
  // ARMv5T and above
  def BLX : AXI<(outs), (ins GPR:$func), BrMiscFrm, IIC_Br, "blx\t$func", []>,
            Requires<[IsARM, HasV5T]>, Sched<[WriteBrL]> {
    bits<4> func;
    let Inst{31-4} = 0b1110000100101111111111110011;
    let Inst{3-0}  = func;
  }
  def BLX_noip :  ARMPseudoExpand<(outs), (ins GPRnoip:$func),
                   4, IIC_Br, [], (BLX GPR:$func)>,
                  Requires<[IsARM, HasV5T]>, Sched<[WriteBrL]>;
```
- EN: Defines TableGen record `BLX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BLX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2683-2693
```tablegen
  def BLX_pred : AI<(outs), (ins GPR:$func), BrMiscFrm,
                    IIC_Br, "blx", "\t$func", []>,
                 Requires<[IsARM, HasV5T]>, Sched<[WriteBrL]> {
    bits<4> func;
    let Inst{27-4} = 0b000100101111111111110011;
    let Inst{3-0}  = func;
  }
  def BLX_pred_noip :  ARMPseudoExpand<(outs), (ins GPRnoip:$func),
                   4, IIC_Br, [],
                   (BLX_pred GPR:$func, (ops 14, zero_reg))>,
                   Requires<[IsARM, HasV5T]>, Sched<[WriteBrL]>;
```
- EN: Defines TableGen record `BLX_pred` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BLX_pred`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2696-2700
```tablegen
  // ARMv4T
  // Note: Restrict $func to the tGPR regclass to prevent it being in LR.
  def BX_CALL : ARMPseudoInst<(outs), (ins tGPR:$func),
                   8, IIC_Br, [(ARMcall_nolink tGPR:$func)]>,
                   Requires<[IsARM, HasV4T]>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `BX_CALL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BX_CALL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2702-2705
```tablegen
  // ARMv4
  def BMOVPCRX_CALL : ARMPseudoInst<(outs), (ins tGPR:$func),
                   8, IIC_Br, [(ARMcall_nolink tGPR:$func)]>,
                   Requires<[IsARM, NoV4T]>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `BMOVPCRX_CALL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BMOVPCRX_CALL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2707-2711
```tablegen
  // mov lr, pc; b if callee is marked noreturn to avoid confusing the
  // return stack predictor.
  def BMOVPCB_CALL : ARMPseudoInst<(outs), (ins arm_bl_target:$func),
                               8, IIC_Br, [(ARMcall_nolink tglobaladdr:$func)]>,
                      Requires<[IsARM]>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `BMOVPCB_CALL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BMOVPCB_CALL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2713-2718
```tablegen
  // push lr before the call
  def BL_PUSHLR : ARMPseudoInst<(outs), (ins GPRlr:$ra, arm_bl_target:$func),
                  4, IIC_Br,
                  []>,
             Requires<[IsARM]>, Sched<[WriteBr]>;
}
```
- EN: Defines TableGen record `BL_PUSHLR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BL_PUSHLR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2720-2727
```tablegen
def : ARMPat<(ARMcall GPR:$func), (BLX $func)>,
      Requires<[IsARM, HasV5T, NoSLSBLRMitigation]>;
def : ARMPat<(ARMcall GPRnoip:$func), (BLX_noip $func)>,
      Requires<[IsARM, HasV5T, SLSBLRMitigation]>;
def : ARMPat<(ARMcall_pred GPR:$func), (BLX_pred $func)>,
      Requires<[IsARM, HasV5T, NoSLSBLRMitigation]>;
def : ARMPat<(ARMcall_pred GPRnoip:$func), (BLX_pred_noip $func)>,
      Requires<[IsARM, HasV5T, SLSBLRMitigation]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2730-2741
```tablegen
let isBranch = 1, isTerminator = 1 in {
  // FIXME: should be able to write a pattern for ARMBrcond, but can't use
  // a two-value operand where a dag node expects two operands. :(
  def Bcc : ABI<0b1010, (outs), (ins arm_br_target:$target),
               IIC_Br, "b", "\t$target",
               [/*(ARMbrcond bb:$target, imm:$cc, CCR:$ccr)*/]>,
               Sched<[WriteBr]>  {
    bits<24> target;
    let Inst{23-0} = target;
    let DecoderMethod = "DecodeBranchImmInstruction";
    let hasSideEffects = 0;
  }
```
- EN: Defines TableGen record `Bcc` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Bcc`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2743-2752
```tablegen
  let isBarrier = 1 in {
    // B is "predicable" since it's just a Bcc with an 'always' condition.
    let isPredicable = 1 in
    // FIXME: We shouldn't need this pseudo at all. Just using Bcc directly
    // should be sufficient.
    // FIXME: Is B really a Barrier? That doesn't seem right.
    def B : ARMPseudoExpand<(outs), (ins arm_br_target:$target), 4, IIC_Br,
                [(br bb:$target)], (Bcc arm_br_target:$target,
                (ops 14, zero_reg))>,
                Sched<[WriteBr]>;
```
- EN: Defines TableGen record `B` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `B`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2754-2771
```tablegen
    let Size = 4, isNotDuplicable = 1, isIndirectBranch = 1 in {
    def BR_JTr : ARMPseudoInst<(outs),
                      (ins GPR:$target, i32imm:$jt),
                      0, IIC_Br,
                      [(ARMbrjt GPR:$target, tjumptable:$jt)]>,
                      Sched<[WriteBr]>;
    def BR_JTm_i12 : ARMPseudoInst<(outs),
                     (ins addrmode_imm12:$target, i32imm:$jt),
                     0, IIC_Br,
                     [(ARMbrjt (i32 (load addrmode_imm12:$target)),
                               tjumptable:$jt)]>, Sched<[WriteBrTbl]>;
    def BR_JTm_rs : ARMPseudoInst<(outs),
                     (ins ldst_so_reg:$target, i32imm:$jt),
                     0, IIC_Br,
                     [(ARMbrjt (i32 (load ldst_so_reg:$target)),
                               tjumptable:$jt)]>, Sched<[WriteBrTbl]>;
    def BR_JTadd : ARMPseudoInst<(outs),
                   (ins GPR:$target, GPR:$idx, i32imm:$jt),
```
- EN: Defines TableGen record `BR_JTr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BR_JTr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2772-2776
```tablegen
                   0, IIC_Br,
                   [(ARMbrjt (add GPR:$target, GPR:$idx), tjumptable:$jt)]>,
                   Sched<[WriteBrTbl]>;
    } // isNotDuplicable = 1, isIndirectBranch = 1
  } // isBarrier = 1
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2778-2778
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2780-2789
```tablegen
// BLX (immediate)
def BLXi : AXI<(outs), (ins arm_blx_target:$target), BrMiscFrm, NoItinerary,
               "blx\t$target", []>,
           Requires<[IsARM, HasV5T]>, Sched<[WriteBrL]> {
  let Inst{31-25} = 0b1111101;
  bits<25> target;
  let Inst{23-0} = target{24-1};
  let Inst{24} = target{0};
  let isCall = 1;
}
```
- EN: Defines TableGen record `BLXi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BLXi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2791-2801
```tablegen
// Branch and Exchange Jazelle
def BXJ : ABI<0b0001, (outs), (ins GPR:$func), NoItinerary, "bxj", "\t$func",
              [/* pattern left blank */]>, Sched<[WriteBr]> {
  bits<4> func;
  let Inst{23-20} = 0b0010;
  let Inst{19-8} = 0xfff;
  let Inst{7-4} = 0b0010;
  let Inst{3-0} = func;
  let isBranch = 1;
  let isIndirectBranch = 1;
}
```
- EN: Defines TableGen record `BXJ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BXJ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2803-2803
```tablegen
// Tail calls.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2805-2807
```tablegen
let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [SP] in {
  def TCRETURNdi : PseudoInst<(outs), (ins i32imm:$dst, i32imm:$SPDiff), IIC_Br, []>,
                   Sched<[WriteBr]>;
```
- EN: Defines TableGen record `TCRETURNdi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TCRETURNdi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2809-2810
```tablegen
  def TCRETURNri : PseudoInst<(outs), (ins tcGPR:$dst, i32imm:$SPDiff), IIC_Br, []>,
                   Sched<[WriteBr]>;
```
- EN: Defines TableGen record `TCRETURNri` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TCRETURNri`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2812-2813
```tablegen
  def TCRETURNrinotr12 : PseudoInst<(outs), (ins tcGPRnotr12:$dst, i32imm:$SPDiff), IIC_Br, []>,
                   Sched<[WriteBr]>;
```
- EN: Defines TableGen record `TCRETURNrinotr12` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TCRETURNrinotr12`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2815-2818
```tablegen
  def TAILJMPd : ARMPseudoExpand<(outs), (ins arm_br_target:$dst),
                                 4, IIC_Br, [],
                                 (Bcc arm_br_target:$dst, (ops 14, zero_reg))>,
                                 Requires<[IsARM]>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `TAILJMPd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILJMPd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2820-2824
```tablegen
  def TAILJMPr : ARMPseudoExpand<(outs), (ins tcGPR:$dst),
                                 4, IIC_Br, [],
                                 (BX GPR:$dst)>, Sched<[WriteBr]>,
                                 Requires<[IsARM, HasV4T]>;
}
```
- EN: Defines TableGen record `TAILJMPr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILJMPr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2826-2833
```tablegen
// Secure Monitor Call is a system instruction.
def SMC : ABI<0b0001, (outs), (ins imm0_15:$opt), NoItinerary, "smc", "\t$opt",
              []>, Requires<[IsARM, HasTrustZone]> {
  bits<4> opt;
  let Inst{23-4} = 0b01100000000000000111;
  let Inst{3-0} = opt;
}
def : MnemonicAlias<"smi", "smc">;
```
- EN: Defines TableGen record `SMC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2835-2842
```tablegen
// Supervisor Call (Software Interrupt)
let isCall = 1, Uses = [SP] in {
def SVC : ABI<0b1111, (outs), (ins imm24b:$svc), IIC_Br, "svc", "\t$svc", []>,
          Sched<[WriteBr]> {
  bits<24> svc;
  let Inst{23-0} = svc;
}
}
```
- EN: Defines TableGen record `SVC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SVC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2844-2857
```tablegen
// Store Return State
class SRSI<bit wb, string asm>
  : XI<(outs), (ins imm0_31:$mode), AddrModeNone, 4, IndexModeNone, BrFrm,
       NoItinerary, asm, "", []> {
  bits<5> mode;
  let Inst{31-28} = 0b1111;
  let Inst{27-25} = 0b100;
  let Inst{22} = 1;
  let Inst{21} = wb;
  let Inst{20} = 0;
  let Inst{19-16} = 0b1101;  // SP
  let Inst{15-5} = 0b00000101000;
  let Inst{4-0} = mode;
}
```
- EN: Declares reusable TableGen class `SRSI` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `SRSI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2859-2876
```tablegen
def SRSDA : SRSI<0, "srsda\tsp, $mode"> {
  let Inst{24-23} = 0;
}
def SRSDA_UPD : SRSI<1, "srsda\tsp!, $mode"> {
  let Inst{24-23} = 0;
}
def SRSDB : SRSI<0, "srsdb\tsp, $mode"> {
  let Inst{24-23} = 0b10;
}
def SRSDB_UPD : SRSI<1, "srsdb\tsp!, $mode"> {
  let Inst{24-23} = 0b10;
}
def SRSIA : SRSI<0, "srsia\tsp, $mode"> {
  let Inst{24-23} = 0b01;
}
def SRSIA_UPD : SRSI<1, "srsia\tsp!, $mode"> {
  let Inst{24-23} = 0b01;
}
```
- EN: Defines TableGen record `SRSDA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SRSDA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2877-2882
```tablegen
def SRSIB : SRSI<0, "srsib\tsp, $mode"> {
  let Inst{24-23} = 0b11;
}
def SRSIB_UPD : SRSI<1, "srsib\tsp!, $mode"> {
  let Inst{24-23} = 0b11;
}
```
- EN: Defines TableGen record `SRSIB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SRSIB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2884-2885
```tablegen
def : ARMInstAlias<"srsda $mode", (SRSDA imm0_31:$mode)>;
def : ARMInstAlias<"srsda $mode!", (SRSDA_UPD imm0_31:$mode)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2887-2888
```tablegen
def : ARMInstAlias<"srsdb $mode", (SRSDB imm0_31:$mode)>;
def : ARMInstAlias<"srsdb $mode!", (SRSDB_UPD imm0_31:$mode)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2890-2891
```tablegen
def : ARMInstAlias<"srsia $mode", (SRSIA imm0_31:$mode)>;
def : ARMInstAlias<"srsia $mode!", (SRSIA_UPD imm0_31:$mode)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2893-2894
```tablegen
def : ARMInstAlias<"srsib $mode", (SRSIB imm0_31:$mode)>;
def : ARMInstAlias<"srsib $mode!", (SRSIB_UPD imm0_31:$mode)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 2896-2908
```tablegen
// Return From Exception
class RFEI<bit wb, string asm>
  : XI<(outs), (ins GPR:$Rn), AddrModeNone, 4, IndexModeNone, BrFrm,
       NoItinerary, asm, "", []> {
  bits<4> Rn;
  let Inst{31-28} = 0b1111;
  let Inst{27-25} = 0b100;
  let Inst{22} = 0;
  let Inst{21} = wb;
  let Inst{20} = 1;
  let Inst{19-16} = Rn;
  let Inst{15-0} = 0xa00;
}
```
- EN: Declares reusable TableGen class `RFEI` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `RFEI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 2910-2927
```tablegen
def RFEDA : RFEI<0, "rfeda\t$Rn"> {
  let Inst{24-23} = 0;
}
def RFEDA_UPD : RFEI<1, "rfeda\t$Rn!"> {
  let Inst{24-23} = 0;
}
def RFEDB : RFEI<0, "rfedb\t$Rn"> {
  let Inst{24-23} = 0b10;
}
def RFEDB_UPD : RFEI<1, "rfedb\t$Rn!"> {
  let Inst{24-23} = 0b10;
}
def RFEIA : RFEI<0, "rfeia\t$Rn"> {
  let Inst{24-23} = 0b01;
}
def RFEIA_UPD : RFEI<1, "rfeia\t$Rn!"> {
  let Inst{24-23} = 0b01;
}
```
- EN: Defines TableGen record `RFEDA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RFEDA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2928-2933
```tablegen
def RFEIB : RFEI<0, "rfeib\t$Rn"> {
  let Inst{24-23} = 0b11;
}
def RFEIB_UPD : RFEI<1, "rfeib\t$Rn!"> {
  let Inst{24-23} = 0b11;
}
```
- EN: Defines TableGen record `RFEIB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RFEIB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2935-2940
```tablegen
// Hypervisor Call is a system instruction
let isCall = 1 in {
def HVC : AInoP< (outs), (ins imm0_65535:$imm), BrFrm, NoItinerary,
                "hvc", "\t$imm", []>,
          Requires<[IsARM, HasVirtualization]> {
  bits<16> imm;
```
- EN: Defines TableGen record `HVC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `HVC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2942-2953
```tablegen
  // Even though HVC isn't predicable, it's encoding includes a condition field.
  // The instruction is undefined if the condition field is 0xf otherwise it is
  // unpredictable if it isn't condition AL (0xe).
  let Inst{31-28} = 0b1110;
  let Unpredictable{31-28} = 0b1111;
  let Inst{27-24} = 0b0001;
  let Inst{23-20} = 0b0100;
  let Inst{19-8} = imm{15-4};
  let Inst{7-4} = 0b0111;
  let Inst{3-0} = imm{3-0};
}
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 2955-2960
```tablegen
// Return from exception in Hypervisor mode.
let isReturn = 1, isBarrier = 1, isTerminator = 1, Defs = [PC] in
def ERET : ABI<0b0001, (outs), (ins), NoItinerary, "eret", "", []>,
    Requires<[IsARM, HasVirtualization]> {
    let Inst{23-0} = 0b011000000000000001101110;
}
```
- EN: Defines TableGen record `ERET` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ERET`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2962-2964
```tablegen
//===----------------------------------------------------------------------===//
//  Load / Store Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 2966-2966
```tablegen
// Load
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 2969-2974
```tablegen
defm LDR  : AI_ldr1<0, "ldr", IIC_iLoad_r, IIC_iLoad_si, load>;
defm LDRB : AI_ldr1nopc<1, "ldrb", IIC_iLoad_bh_r, IIC_iLoad_bh_si,
                        zextloadi8>;
defm STR  : AI_str1<0, "str", IIC_iStore_r, IIC_iStore_si, store>;
defm STRB : AI_str1nopc<1, "strb", IIC_iStore_bh_r, IIC_iStore_bh_si,
                        truncstorei8>;
```
- EN: Defines TableGen record `LDR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2976-2988
```tablegen
// Special LDR for loads from non-pc-relative constpools.
let canFoldAsLoad = 1, mayLoad = 1, hasSideEffects = 0,
    isReMaterializable = 1, isCodeGenOnly = 1 in
def LDRcp : AI2ldst<0b010, 1, 0, (outs GPR:$Rt), (ins addrmode_imm12:$addr),
                 AddrMode_i12, LdFrm, IIC_iLoad_r, "ldr", "\t$Rt, $addr",
                 []> {
  bits<4> Rt;
  bits<17> addr;
  let Inst{23}    = addr{12};     // U (add = ('U' == 1))
  let Inst{19-16} = 0b1111;
  let Inst{15-12} = Rt;
  let Inst{11-0}  = addr{11-0};   // imm12
}
```
- EN: Defines TableGen record `LDRcp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRcp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2990-2993
```tablegen
// Loads with zero extension
def LDRH  : AI3ld<0b1011, 1, (outs GPR:$Rt), (ins addrmode3:$addr), LdMiscFrm,
                  IIC_iLoad_bh_r, "ldrh", "\t$Rt, $addr",
                  [(set GPR:$Rt, (zextloadi16 addrmode3:$addr))]>;
```
- EN: Defines TableGen record `LDRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 2995-2998
```tablegen
// Loads with sign extension
def LDRSH : AI3ld<0b1111, 1, (outs GPR:$Rt), (ins addrmode3:$addr), LdMiscFrm,
                   IIC_iLoad_bh_r, "ldrsh", "\t$Rt, $addr",
                   [(set GPR:$Rt, (sextloadi16 addrmode3:$addr))]>;
```
- EN: Defines TableGen record `LDRSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3000-3002
```tablegen
def LDRSB : AI3ld<0b1101, 1, (outs GPR:$Rt), (ins addrmode3:$addr), LdMiscFrm,
                   IIC_iLoad_bh_r, "ldrsb", "\t$Rt, $addr",
                   [(set GPR:$Rt, (sextloadi8 addrmode3:$addr))]>;
```
- EN: Defines TableGen record `LDRSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3004-3009
```tablegen
let mayLoad = 1, hasSideEffects = 0, hasExtraDefRegAllocReq = 1 in {
  // Load doubleword
  def LDRD : AI3ld<0b1101, 0, (outs GPR:$Rt, GPR:$Rt2), (ins addrmode3:$addr),
                   LdMiscFrm, IIC_iLoad_d_r, "ldrd", "\t$Rt, $Rt2, $addr", []>,
             Requires<[IsARM, HasV5TE]>;
}
```
- EN: Defines TableGen record `LDRD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3011-3017
```tablegen
let mayLoad = 1, hasSideEffects = 0, hasNoSchedulingInfo = 1 in {
def LOADDUAL : ARMPseudoInst<(outs GPRPairOp:$Rt), (ins addrmode3:$addr),
                             64, IIC_iLoad_d_r, []>,
               Requires<[IsARM, HasV5TE]> {
  let AM = AddrMode3;
}
}
```
- EN: Defines TableGen record `LOADDUAL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LOADDUAL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3019-3024
```tablegen
def LDA : AIldracq<0b00, (outs GPR:$Rt), (ins addr_offset_none:$addr),
                    NoItinerary, "lda", "\t$Rt, $addr", []>;
def LDAB : AIldracq<0b10, (outs GPR:$Rt), (ins addr_offset_none:$addr),
                    NoItinerary, "ldab", "\t$Rt, $addr", []>;
def LDAH : AIldracq<0b11, (outs GPR:$Rt), (ins addr_offset_none:$addr),
                    NoItinerary, "ldah", "\t$Rt, $addr", []>;
```
- EN: Defines TableGen record `LDA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3026-3038
```tablegen
// Indexed loads
multiclass AI2_ldridx<bit isByte, string opc,
                      InstrItinClass iii, InstrItinClass iir> {
  def _PRE_IMM  : AI2ldstidx<1, isByte, 1, (outs GPR:$Rt, GPR:$Rn_wb),
                      (ins addrmode_imm12_pre:$addr), IndexModePre, LdFrm, iii,
                      opc, "\t$Rt, $addr!", "$addr.base = $Rn_wb", []> {
    bits<17> addr;
    let Inst{25} = 0;
    let Inst{23} = addr{12};
    let Inst{19-16} = addr{16-13};
    let Inst{11-0} = addr{11-0};
    let DecoderMethod = "DecodeLDRPreImm";
  }
```
- EN: Declares TableGen `multiclass AI2_ldridx`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI2_ldridx`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3040-3050
```tablegen
  def _PRE_REG  : AI2ldstidx<1, isByte, 1, (outs GPR:$Rt, GPR:$Rn_wb),
                      (ins ldst_so_reg:$addr), IndexModePre, LdFrm, iir,
                      opc, "\t$Rt, $addr!", "$addr.base = $Rn_wb", []> {
    bits<17> addr;
    let Inst{25} = 1;
    let Inst{23} = addr{12};
    let Inst{19-16} = addr{16-13};
    let Inst{11-0} = addr{11-0};
    let Inst{4} = 0;
    let DecoderMethod = "DecodeLDRPreReg";
  }
```
- EN: Defines TableGen record `_PRE_REG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_PRE_REG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3052-3065
```tablegen
  def _POST_REG : AI2ldstidx<1, isByte, 0, (outs GPR:$Rt, GPR:$Rn_wb),
                       (ins addr_offset_none:$addr, am2offset_reg:$offset),
                       IndexModePost, LdFrm, iir,
                       opc, "\t$Rt, $addr, $offset",
                       "$addr.base = $Rn_wb", []> {
     // {12}     isAdd
     // {11-0}   imm12/Rm
     bits<14> offset;
     bits<4> addr;
     let Inst{25} = 1;
     let Inst{23} = offset{12};
     let Inst{19-16} = addr;
     let Inst{11-0} = offset{11-0};
     let Inst{4} = 0;
```
- EN: Defines TableGen record `_POST_REG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_POST_REG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3067-3068
```tablegen
    let DecoderMethod = "DecodeAddrMode2IdxInstruction";
   }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3070-3082
```tablegen
   def _POST_IMM : AI2ldstidx<1, isByte, 0, (outs GPR:$Rt, GPR:$Rn_wb),
                       (ins addr_offset_none:$addr, am2offset_imm:$offset),
                      IndexModePost, LdFrm, iii,
                      opc, "\t$Rt, $addr, $offset",
                      "$addr.base = $Rn_wb", []> {
    // {12}     isAdd
    // {11-0}   imm12/Rm
    bits<14> offset;
    bits<4> addr;
    let Inst{25} = 0;
    let Inst{23} = offset{12};
    let Inst{19-16} = addr;
    let Inst{11-0} = offset{11-0};
```
- EN: Defines TableGen record `_POST_IMM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_POST_IMM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3084-3085
```tablegen
    let DecoderMethod = "DecodeAddrMode2IdxInstruction";
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3087-3087
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3089-3094
```tablegen
let mayLoad = 1, hasSideEffects = 0 in {
// FIXME: for LDR_PRE_REG etc. the itinerary should be either IIC_iLoad_ru or
// IIC_iLoad_siu depending on whether it the offset register is shifted.
defm LDR  : AI2_ldridx<0, "ldr", IIC_iLoad_iu, IIC_iLoad_ru>;
defm LDRB : AI2_ldridx<1, "ldrb", IIC_iLoad_bh_iu, IIC_iLoad_bh_ru>;
}
```
- EN: Defines TableGen record `LDR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3096-3113
```tablegen
multiclass AI3_ldridx<bits<4> op, string opc, InstrItinClass itin> {
  def _PRE  : AI3ldstidx<op, 1, 1, (outs GPR:$Rt, GPR:$Rn_wb),
                        (ins addrmode3_pre:$addr), IndexModePre,
                        LdMiscFrm, itin,
                        opc, "\t$Rt, $addr!", "$addr.base = $Rn_wb", []> {
    bits<14> addr;
    let Inst{23}    = addr{8};      // U bit
    let Inst{22}    = addr{13};     // 1 == imm8, 0 == Rm
    let Inst{19-16} = addr{12-9};   // Rn
    let Inst{11-8}  = addr{7-4};    // imm7_4/zero
    let Inst{3-0}   = addr{3-0};    // imm3_0/Rm
    let DecoderMethod = "DecodeAddrMode3Instruction";
  }
  def _POST : AI3ldstidx<op, 1, 0, (outs GPR:$Rt, GPR:$Rn_wb),
                        (ins addr_offset_none:$addr, am3offset:$offset),
                        IndexModePost, LdMiscFrm, itin,
                        opc, "\t$Rt, $addr, $offset", "$addr.base = $Rn_wb",
                        []> {
```
- EN: Declares TableGen `multiclass AI3_ldridx`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI3_ldridx`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3114-3123
```tablegen
    bits<10> offset;
    bits<4> addr;
    let Inst{23}    = offset{8};      // U bit
    let Inst{22}    = offset{9};      // 1 == imm8, 0 == Rm
    let Inst{19-16} = addr;
    let Inst{11-8}  = offset{7-4};    // imm7_4/zero
    let Inst{3-0}   = offset{3-0};    // imm3_0/Rm
    let DecoderMethod = "DecodeAddrMode3Instruction";
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3125-3142
```tablegen
let mayLoad = 1, hasSideEffects = 0 in {
defm LDRH  : AI3_ldridx<0b1011, "ldrh", IIC_iLoad_bh_ru>;
defm LDRSH : AI3_ldridx<0b1111, "ldrsh", IIC_iLoad_bh_ru>;
defm LDRSB : AI3_ldridx<0b1101, "ldrsb", IIC_iLoad_bh_ru>;
let hasExtraDefRegAllocReq = 1 in {
def LDRD_PRE : AI3ldstidx<0b1101, 0, 1, (outs GPR:$Rt, GPR:$Rt2, GPR:$Rn_wb),
                          (ins addrmode3_pre:$addr), IndexModePre,
                          LdMiscFrm, IIC_iLoad_d_ru,
                          "ldrd", "\t$Rt, $Rt2, $addr!",
                          "$addr.base = $Rn_wb", []> {
  bits<14> addr;
  let Inst{23}    = addr{8};      // U bit
  let Inst{22}    = addr{13};     // 1 == imm8, 0 == Rm
  let Inst{19-16} = addr{12-9};   // Rn
  let Inst{11-8}  = addr{7-4};    // imm7_4/zero
  let Inst{3-0}   = addr{3-0};    // imm3_0/Rm
  let DecoderMethod = "DecodeAddrMode3Instruction";
}
```
- EN: Defines TableGen record `LDRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3143-3158
```tablegen
def LDRD_POST: AI3ldstidx<0b1101, 0, 0, (outs GPR:$Rt, GPR:$Rt2, GPR:$Rn_wb),
                          (ins addr_offset_none:$addr, am3offset:$offset),
                          IndexModePost, LdMiscFrm, IIC_iLoad_d_ru,
                          "ldrd", "\t$Rt, $Rt2, $addr, $offset",
                          "$addr.base = $Rn_wb", []> {
  bits<10> offset;
  bits<4> addr;
  let Inst{23}    = offset{8};      // U bit
  let Inst{22}    = offset{9};      // 1 == imm8, 0 == Rm
  let Inst{19-16} = addr;
  let Inst{11-8}  = offset{7-4};    // imm7_4/zero
  let Inst{3-0}   = offset{3-0};    // imm3_0/Rm
  let DecoderMethod = "DecodeAddrMode3Instruction";
}
} // hasExtraDefRegAllocReq = 1
} // mayLoad = 1, hasSideEffects = 0
```
- EN: Defines TableGen record `LDRD_POST:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRD_POST:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3160-3177
```tablegen
// LDRT, LDRBT, LDRSBT, LDRHT, LDRSHT.
let mayLoad = 1, hasSideEffects = 0 in {
def LDRT_POST_REG : AI2ldstidx<1, 0, 0, (outs GPR:$Rt, GPR:$Rn_wb),
                    (ins addr_offset_none:$addr, am2offset_reg:$offset),
                    IndexModePost, LdFrm, IIC_iLoad_ru,
                    "ldrt", "\t$Rt, $addr, $offset",
                    "$addr.base = $Rn_wb", []> {
  // {12}     isAdd
  // {11-0}   imm12/Rm
  bits<14> offset;
  bits<4> addr;
  let Inst{25} = 1;
  let Inst{23} = offset{12};
  let Inst{21} = 1; // overwrite
  let Inst{19-16} = addr;
  let Inst{11-5} = offset{11-5};
  let Inst{4} = 0;
  let Inst{3-0} = offset{3-0};
```
- EN: Defines TableGen record `LDRT_POST_REG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRT_POST_REG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3178-3179
```tablegen
  let DecoderMethod = "DecodeAddrMode2IdxInstruction";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3181-3196
```tablegen
def LDRT_POST_IMM
  : AI2ldstidx<1, 0, 0, (outs GPR:$Rt, GPR:$Rn_wb),
               (ins addr_offset_none:$addr, am2offset_imm:$offset),
               IndexModePost, LdFrm, IIC_iLoad_ru,
               "ldrt", "\t$Rt, $addr, $offset", "$addr.base = $Rn_wb", []> {
  // {12}     isAdd
  // {11-0}   imm12/Rm
  bits<14> offset;
  bits<4> addr;
  let Inst{25} = 0;
  let Inst{23} = offset{12};
  let Inst{21} = 1; // overwrite
  let Inst{19-16} = addr;
  let Inst{11-0} = offset{11-0};
  let DecoderMethod = "DecodeAddrMode2IdxInstruction";
}
```
- EN: Defines TableGen record `LDRT_POST_IMM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRT_POST_IMM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3198-3215
```tablegen
def LDRBT_POST_REG : AI2ldstidx<1, 1, 0, (outs GPR:$Rt, GPR:$Rn_wb),
                     (ins addr_offset_none:$addr, am2offset_reg:$offset),
                     IndexModePost, LdFrm, IIC_iLoad_bh_ru,
                     "ldrbt", "\t$Rt, $addr, $offset",
                     "$addr.base = $Rn_wb", []> {
  // {12}     isAdd
  // {11-0}   imm12/Rm
  bits<14> offset;
  bits<4> addr;
  let Inst{25} = 1;
  let Inst{23} = offset{12};
  let Inst{21} = 1; // overwrite
  let Inst{19-16} = addr;
  let Inst{11-5} = offset{11-5};
  let Inst{4} = 0;
  let Inst{3-0} = offset{3-0};
  let DecoderMethod = "DecodeAddrMode2IdxInstruction";
}
```
- EN: Defines TableGen record `LDRBT_POST_REG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRBT_POST_REG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3217-3232
```tablegen
def LDRBT_POST_IMM
  : AI2ldstidx<1, 1, 0, (outs GPR:$Rt, GPR:$Rn_wb),
               (ins addr_offset_none:$addr, am2offset_imm:$offset),
               IndexModePost, LdFrm, IIC_iLoad_bh_ru,
               "ldrbt", "\t$Rt, $addr, $offset", "$addr.base = $Rn_wb", []> {
  // {12}     isAdd
  // {11-0}   imm12/Rm
  bits<14> offset;
  bits<4> addr;
  let Inst{25} = 0;
  let Inst{23} = offset{12};
  let Inst{21} = 1; // overwrite
  let Inst{19-16} = addr;
  let Inst{11-0} = offset{11-0};
  let DecoderMethod = "DecodeAddrMode2IdxInstruction";
}
```
- EN: Defines TableGen record `LDRBT_POST_IMM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRBT_POST_IMM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3234-3251
```tablegen
multiclass AI3ldrT<bits<4> op, string opc> {
  def i : AI3ldstidxT<op, 1, (outs GPR:$Rt, GPR:$base_wb),
                      (ins addr_offset_none:$addr, postidx_imm8:$offset),
                      IndexModePost, LdMiscFrm, IIC_iLoad_bh_ru, opc,
                      "\t$Rt, $addr, $offset", "$addr.base = $base_wb", []> {
    bits<9> offset;
    let Inst{23} = offset{8};
    let Inst{22} = 1;
    let Inst{11-8} = offset{7-4};
    let Inst{3-0} = offset{3-0};
  }
  def r : AI3ldstidxT<op, 1, (outs GPRnopc:$Rt, GPRnopc:$base_wb),
                      (ins addr_offset_none:$addr, postidx_reg:$Rm),
                      IndexModePost, LdMiscFrm, IIC_iLoad_bh_ru, opc,
                      "\t$Rt, $addr, $Rm", "$addr.base = $base_wb", []> {
    bits<5> Rm;
    let Inst{23} = Rm{4};
    let Inst{22} = 0;
```
- EN: Declares TableGen `multiclass AI3ldrT`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI3ldrT`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3252-3256
```tablegen
    let Inst{11-8} = 0;
    let Unpredictable{11-8} = 0b1111;
    let Inst{3-0} = Rm{3-0};
    let DecoderMethod = "DecodeLDR";
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3258-3260
```tablegen
  def ii : ARMAsmPseudo<!strconcat(opc, "${p} $Rt, $addr"),
                        (ins addr_offset_none:$addr, pred:$p), (outs GPR:$Rt)>;
}
```
- EN: Defines TableGen record `ii` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ii`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3262-3265
```tablegen
defm LDRSBT : AI3ldrT<0b1101, "ldrsbt">;
defm LDRHT  : AI3ldrT<0b1011, "ldrht">;
defm LDRSHT : AI3ldrT<0b1111, "ldrsht">;
}
```
- EN: Defines TableGen record `LDRSBT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRSBT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3267-3269
```tablegen
def LDRT_POST
  : ARMAsmPseudo<"ldrt${q} $Rt, $addr", (ins addr_offset_none:$addr, pred:$q),
                 (outs GPR:$Rt)>;
```
- EN: Defines TableGen record `LDRT_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRT_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3271-3273
```tablegen
def LDRBT_POST
  : ARMAsmPseudo<"ldrbt${q} $Rt, $addr", (ins addr_offset_none:$addr, pred:$q),
                 (outs GPR:$Rt)>;
```
- EN: Defines TableGen record `LDRBT_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRBT_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3275-3279
```tablegen
// Pseudo instruction ldr Rt, =immediate
def LDRConstPool
  : ARMAsmPseudo<"ldr${q} $Rt, $immediate",
                 (ins const_pool_asm_imm:$immediate, pred:$q),
                 (outs GPR:$Rt)>;
```
- EN: Defines TableGen record `LDRConstPool` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRConstPool`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3281-3281
```tablegen
// Store
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3283-3286
```tablegen
// Stores with truncate
def STRH : AI3str<0b1011, (outs), (ins GPR:$Rt, addrmode3:$addr), StMiscFrm,
               IIC_iStore_bh_r, "strh", "\t$Rt, $addr",
               [(truncstorei16 GPR:$Rt, addrmode3:$addr)]>;
```
- EN: Defines TableGen record `STRH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3288-3295
```tablegen
// Store doubleword
let mayStore = 1, hasSideEffects = 0, hasExtraSrcRegAllocReq = 1 in {
  def STRD : AI3str<0b1111, (outs), (ins GPR:$Rt, GPR:$Rt2, addrmode3:$addr),
                    StMiscFrm, IIC_iStore_d_r, "strd", "\t$Rt, $Rt2, $addr", []>,
             Requires<[IsARM, HasV5TE]> {
    let Inst{21} = 0;
  }
}
```
- EN: Defines TableGen record `STRD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3297-3303
```tablegen
let mayStore = 1, hasSideEffects = 0, hasNoSchedulingInfo = 1 in {
def STOREDUAL : ARMPseudoInst<(outs), (ins GPRPairOp:$Rt, addrmode3:$addr),
                              64, IIC_iStore_d_r, []>,
                Requires<[IsARM, HasV5TE]> {
  let AM = AddrMode3;
}
}
```
- EN: Defines TableGen record `STOREDUAL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STOREDUAL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3305-3319
```tablegen
// Indexed stores
multiclass AI2_stridx<bit isByte, string opc,
                      InstrItinClass iii, InstrItinClass iir> {
  def _PRE_IMM : AI2ldstidx<0, isByte, 1, (outs GPR:$Rn_wb),
                            (ins GPR:$Rt, addrmode_imm12_pre:$addr), IndexModePre,
                            StFrm, iii,
                            opc, "\t$Rt, $addr!",
                            "$addr.base = $Rn_wb,@earlyclobber $Rn_wb", []> {
    bits<17> addr;
    let Inst{25} = 0;
    let Inst{23}    = addr{12};     // U (add = ('U' == 1))
    let Inst{19-16} = addr{16-13};  // Rn
    let Inst{11-0}  = addr{11-0};   // imm12
    let DecoderMethod = "DecodeSTRPreImm";
  }
```
- EN: Declares TableGen `multiclass AI2_stridx`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI2_stridx`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3321-3338
```tablegen
  def _PRE_REG  : AI2ldstidx<0, isByte, 1, (outs GPR:$Rn_wb),
                      (ins GPR:$Rt, ldst_so_reg:$addr),
                      IndexModePre, StFrm, iir,
                      opc, "\t$Rt, $addr!",
                      "$addr.base = $Rn_wb,@earlyclobber $Rn_wb", []> {
    bits<17> addr;
    let Inst{25} = 1;
    let Inst{23}    = addr{12};    // U (add = ('U' == 1))
    let Inst{19-16} = addr{16-13}; // Rn
    let Inst{11-0}  = addr{11-0};
    let Inst{4}     = 0;           // Inst{4} = 0
    let DecoderMethod = "DecodeSTRPreReg";
  }
  def _POST_REG : AI2ldstidx<0, isByte, 0, (outs GPR:$Rn_wb),
                (ins GPR:$Rt, addr_offset_none:$addr, am2offset_reg:$offset),
                IndexModePost, StFrm, iir,
                opc, "\t$Rt, $addr, $offset",
                "$addr.base = $Rn_wb,@earlyclobber $Rn_wb", []> {
```
- EN: Defines TableGen record `_PRE_REG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_PRE_REG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3339-3347
```tablegen
     // {12}     isAdd
     // {11-0}   imm12/Rm
     bits<14> offset;
     bits<4> addr;
     let Inst{25} = 1;
     let Inst{23} = offset{12};
     let Inst{19-16} = addr;
     let Inst{11-0} = offset{11-0};
     let Inst{4} = 0;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3349-3350
```tablegen
    let DecoderMethod = "DecodeAddrMode2IdxInstruction";
   }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3352-3364
```tablegen
   def _POST_IMM : AI2ldstidx<0, isByte, 0, (outs GPR:$Rn_wb),
                (ins GPR:$Rt, addr_offset_none:$addr, am2offset_imm:$offset),
                IndexModePost, StFrm, iii,
                opc, "\t$Rt, $addr, $offset",
                "$addr.base = $Rn_wb,@earlyclobber $Rn_wb", []> {
    // {12}     isAdd
    // {11-0}   imm12/Rm
    bits<14> offset;
    bits<4> addr;
    let Inst{25} = 0;
    let Inst{23} = offset{12};
    let Inst{19-16} = addr;
    let Inst{11-0} = offset{11-0};
```
- EN: Defines TableGen record `_POST_IMM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_POST_IMM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3366-3368
```tablegen
    let DecoderMethod = "DecodeAddrMode2IdxInstruction";
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3370-3375
```tablegen
let mayStore = 1, hasSideEffects = 0 in {
// FIXME: for STR_PRE_REG etc. the itinerary should be either IIC_iStore_ru or
// IIC_iStore_siu depending on whether it the offset register is shifted.
defm STR  : AI2_stridx<0, "str", IIC_iStore_iu, IIC_iStore_ru>;
defm STRB : AI2_stridx<1, "strb", IIC_iStore_bh_iu, IIC_iStore_bh_ru>;
}
```
- EN: Defines TableGen record `STR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3377-3392
```tablegen
def : ARMPat<(post_store GPR:$Rt, addr_offset_none:$addr,
                         am2offset_reg:$offset),
             (STR_POST_REG GPR:$Rt, addr_offset_none:$addr,
                           am2offset_reg:$offset)>;
def : ARMPat<(post_store GPR:$Rt, addr_offset_none:$addr,
                         am2offset_imm:$offset),
             (STR_POST_IMM GPR:$Rt, addr_offset_none:$addr,
                           am2offset_imm:$offset)>;
def : ARMPat<(post_truncsti8 GPR:$Rt, addr_offset_none:$addr,
                             am2offset_reg:$offset),
             (STRB_POST_REG GPR:$Rt, addr_offset_none:$addr,
                            am2offset_reg:$offset)>;
def : ARMPat<(post_truncsti8 GPR:$Rt, addr_offset_none:$addr,
                             am2offset_imm:$offset),
             (STRB_POST_IMM GPR:$Rt, addr_offset_none:$addr,
                            am2offset_imm:$offset)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3394-3411
```tablegen
// Pseudo-instructions for pattern matching the pre-indexed stores. We can't
// put the patterns on the instruction definitions directly as ISel wants
// the address base and offset to be separate operands, not a single
// complex operand like we represent the instructions themselves. The
// pseudos map between the two.
let usesCustomInserter = 1,
    Constraints = "$Rn = $Rn_wb,@earlyclobber $Rn_wb" in {
def STRi_preidx: ARMPseudoInst<(outs GPR:$Rn_wb),
               (ins GPR:$Rt, GPR:$Rn, am2offset_imm:$offset, pred:$p),
               4, IIC_iStore_ru,
            [(set GPR:$Rn_wb,
                  (pre_store GPR:$Rt, GPR:$Rn, am2offset_imm:$offset))]>;
def STRr_preidx: ARMPseudoInst<(outs GPR:$Rn_wb),
               (ins GPR:$Rt, GPR:$Rn, am2offset_reg:$offset, pred:$p),
               4, IIC_iStore_ru,
            [(set GPR:$Rn_wb,
                  (pre_store GPR:$Rt, GPR:$Rn, am2offset_reg:$offset))]>;
def STRBi_preidx: ARMPseudoInst<(outs GPR:$Rn_wb),
```
- EN: Defines TableGen record `STRi_preidx:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRi_preidx:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3412-3426
```tablegen
               (ins GPR:$Rt, GPR:$Rn, am2offset_imm:$offset, pred:$p),
               4, IIC_iStore_ru,
            [(set GPR:$Rn_wb,
                  (pre_truncsti8 GPR:$Rt, GPR:$Rn, am2offset_imm:$offset))]>;
def STRBr_preidx: ARMPseudoInst<(outs GPR:$Rn_wb),
               (ins GPR:$Rt, GPR:$Rn, am2offset_reg:$offset, pred:$p),
               4, IIC_iStore_ru,
            [(set GPR:$Rn_wb,
                  (pre_truncsti8 GPR:$Rt, GPR:$Rn, am2offset_reg:$offset))]>;
def STRH_preidx: ARMPseudoInst<(outs GPR:$Rn_wb),
               (ins GPR:$Rt, GPR:$Rn, am3offset:$offset, pred:$p),
               4, IIC_iStore_ru,
            [(set GPR:$Rn_wb,
                  (pre_truncsti16 GPR:$Rt, GPR:$Rn, am3offset:$offset))]>;
}
```
- EN: Defines TableGen record `STRBr_preidx:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRBr_preidx:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3429-3442
```tablegen
let mayStore = 1, hasSideEffects = 0 in {
def STRH_PRE  : AI3ldstidx<0b1011, 0, 1, (outs GPR:$Rn_wb),
                           (ins GPR:$Rt, addrmode3_pre:$addr), IndexModePre,
                           StMiscFrm, IIC_iStore_bh_ru,
                           "strh", "\t$Rt, $addr!",
                           "$addr.base = $Rn_wb,@earlyclobber $Rn_wb", []> {
  bits<14> addr;
  let Inst{23}    = addr{8};      // U bit
  let Inst{22}    = addr{13};     // 1 == imm8, 0 == Rm
  let Inst{19-16} = addr{12-9};   // Rn
  let Inst{11-8}  = addr{7-4};    // imm7_4/zero
  let Inst{3-0}   = addr{3-0};    // imm3_0/Rm
  let DecoderMethod = "DecodeAddrMode3Instruction";
}
```
- EN: Defines TableGen record `STRH_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRH_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3444-3461
```tablegen
def STRH_POST : AI3ldstidx<0b1011, 0, 0, (outs GPR:$Rn_wb),
                       (ins GPR:$Rt, addr_offset_none:$addr, am3offset:$offset),
                       IndexModePost, StMiscFrm, IIC_iStore_bh_ru,
                       "strh", "\t$Rt, $addr, $offset",
                       "$addr.base = $Rn_wb,@earlyclobber $Rn_wb",
                   [(set GPR:$Rn_wb, (post_truncsti16 GPR:$Rt,
                                                      addr_offset_none:$addr,
                                                      am3offset:$offset))]> {
  bits<10> offset;
  bits<4> addr;
  let Inst{23}    = offset{8};      // U bit
  let Inst{22}    = offset{9};      // 1 == imm8, 0 == Rm
  let Inst{19-16} = addr;
  let Inst{11-8}  = offset{7-4};    // imm7_4/zero
  let Inst{3-0}   = offset{3-0};    // imm3_0/Rm
  let DecoderMethod = "DecodeAddrMode3Instruction";
}
} // mayStore = 1, hasSideEffects = 0
```
- EN: Defines TableGen record `STRH_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRH_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3463-3476
```tablegen
let mayStore = 1, hasSideEffects = 0, hasExtraSrcRegAllocReq = 1 in {
def STRD_PRE : AI3ldstidx<0b1111, 0, 1, (outs GPR:$Rn_wb),
                          (ins GPR:$Rt, GPR:$Rt2, addrmode3_pre:$addr),
                          IndexModePre, StMiscFrm, IIC_iStore_d_ru,
                          "strd", "\t$Rt, $Rt2, $addr!",
                          "$addr.base = $Rn_wb", []> {
  bits<14> addr;
  let Inst{23}    = addr{8};      // U bit
  let Inst{22}    = addr{13};     // 1 == imm8, 0 == Rm
  let Inst{19-16} = addr{12-9};   // Rn
  let Inst{11-8}  = addr{7-4};    // imm7_4/zero
  let Inst{3-0}   = addr{3-0};    // imm3_0/Rm
  let DecoderMethod = "DecodeAddrMode3Instruction";
}
```
- EN: Defines TableGen record `STRD_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRD_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3478-3493
```tablegen
def STRD_POST: AI3ldstidx<0b1111, 0, 0, (outs GPR:$Rn_wb),
                          (ins GPR:$Rt, GPR:$Rt2, addr_offset_none:$addr,
                               am3offset:$offset),
                          IndexModePost, StMiscFrm, IIC_iStore_d_ru,
                          "strd", "\t$Rt, $Rt2, $addr, $offset",
                          "$addr.base = $Rn_wb", []> {
  bits<10> offset;
  bits<4> addr;
  let Inst{23}    = offset{8};      // U bit
  let Inst{22}    = offset{9};      // 1 == imm8, 0 == Rm
  let Inst{19-16} = addr;
  let Inst{11-8}  = offset{7-4};    // imm7_4/zero
  let Inst{3-0}   = offset{3-0};    // imm3_0/Rm
  let DecoderMethod = "DecodeAddrMode3Instruction";
}
} // mayStore = 1, hasSideEffects = 0, hasExtraSrcRegAllocReq = 1
```
- EN: Defines TableGen record `STRD_POST:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRD_POST:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3495-3495
```tablegen
// STRT, STRBT, and STRHT
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3497-3497
```tablegen
let mayStore = 1, hasSideEffects = 0 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3499-3516
```tablegen
def STRBT_POST_REG : AI2ldstidx<0, 1, 0, (outs GPR:$Rn_wb),
                   (ins GPR:$Rt, addr_offset_none:$addr, am2offset_reg:$offset),
                   IndexModePost, StFrm, IIC_iStore_bh_ru,
                   "strbt", "\t$Rt, $addr, $offset",
                   "$addr.base = $Rn_wb", []> {
  // {12}     isAdd
  // {11-0}   imm12/Rm
  bits<14> offset;
  bits<4> addr;
  let Inst{25} = 1;
  let Inst{23} = offset{12};
  let Inst{21} = 1; // overwrite
  let Inst{19-16} = addr;
  let Inst{11-5} = offset{11-5};
  let Inst{4} = 0;
  let Inst{3-0} = offset{3-0};
  let DecoderMethod = "DecodeAddrMode2IdxInstruction";
}
```
- EN: Defines TableGen record `STRBT_POST_REG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRBT_POST_REG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3518-3533
```tablegen
def STRBT_POST_IMM
  : AI2ldstidx<0, 1, 0, (outs GPR:$Rn_wb),
               (ins GPR:$Rt, addr_offset_none:$addr, am2offset_imm:$offset),
               IndexModePost, StFrm, IIC_iStore_bh_ru,
               "strbt", "\t$Rt, $addr, $offset", "$addr.base = $Rn_wb", []> {
  // {12}     isAdd
  // {11-0}   imm12/Rm
  bits<14> offset;
  bits<4> addr;
  let Inst{25} = 0;
  let Inst{23} = offset{12};
  let Inst{21} = 1; // overwrite
  let Inst{19-16} = addr;
  let Inst{11-0} = offset{11-0};
  let DecoderMethod = "DecodeAddrMode2IdxInstruction";
}
```
- EN: Defines TableGen record `STRBT_POST_IMM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRBT_POST_IMM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3535-3537
```tablegen
def STRBT_POST
  : ARMAsmPseudo<"strbt${q} $Rt, $addr",
                 (ins GPR:$Rt, addr_offset_none:$addr, pred:$q)>;
```
- EN: Defines TableGen record `STRBT_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRBT_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3539-3556
```tablegen
def STRT_POST_REG : AI2ldstidx<0, 0, 0, (outs GPR:$Rn_wb),
                   (ins GPR:$Rt, addr_offset_none:$addr, am2offset_reg:$offset),
                   IndexModePost, StFrm, IIC_iStore_ru,
                   "strt", "\t$Rt, $addr, $offset",
                   "$addr.base = $Rn_wb", []> {
  // {12}     isAdd
  // {11-0}   imm12/Rm
  bits<14> offset;
  bits<4> addr;
  let Inst{25} = 1;
  let Inst{23} = offset{12};
  let Inst{21} = 1; // overwrite
  let Inst{19-16} = addr;
  let Inst{11-5} = offset{11-5};
  let Inst{4} = 0;
  let Inst{3-0} = offset{3-0};
  let DecoderMethod = "DecodeAddrMode2IdxInstruction";
}
```
- EN: Defines TableGen record `STRT_POST_REG` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRT_POST_REG`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3558-3573
```tablegen
def STRT_POST_IMM
  : AI2ldstidx<0, 0, 0, (outs GPR:$Rn_wb),
               (ins GPR:$Rt, addr_offset_none:$addr, am2offset_imm:$offset),
               IndexModePost, StFrm, IIC_iStore_ru,
               "strt", "\t$Rt, $addr, $offset", "$addr.base = $Rn_wb", []> {
  // {12}     isAdd
  // {11-0}   imm12/Rm
  bits<14> offset;
  bits<4> addr;
  let Inst{25} = 0;
  let Inst{23} = offset{12};
  let Inst{21} = 1; // overwrite
  let Inst{19-16} = addr;
  let Inst{11-0} = offset{11-0};
  let DecoderMethod = "DecodeAddrMode2IdxInstruction";
}
```
- EN: Defines TableGen record `STRT_POST_IMM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRT_POST_IMM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3575-3577
```tablegen
def STRT_POST
  : ARMAsmPseudo<"strt${q} $Rt, $addr",
                 (ins GPR:$Rt, addr_offset_none:$addr, pred:$q)>;
```
- EN: Defines TableGen record `STRT_POST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRT_POST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3579-3596
```tablegen
multiclass AI3strT<bits<4> op, string opc> {
  def i : AI3ldstidxT<op, 0, (outs GPR:$base_wb),
                    (ins GPR:$Rt, addr_offset_none:$addr, postidx_imm8:$offset),
                    IndexModePost, StMiscFrm, IIC_iStore_bh_ru, opc,
                    "\t$Rt, $addr, $offset", "$addr.base = $base_wb", []> {
    bits<9> offset;
    let Inst{23} = offset{8};
    let Inst{22} = 1;
    let Inst{11-8} = offset{7-4};
    let Inst{3-0} = offset{3-0};
  }
  def r : AI3ldstidxT<op, 0, (outs GPR:$base_wb),
                      (ins GPR:$Rt, addr_offset_none:$addr, postidx_reg:$Rm),
                      IndexModePost, StMiscFrm, IIC_iStore_bh_ru, opc,
                      "\t$Rt, $addr, $Rm", "$addr.base = $base_wb", []> {
    bits<5> Rm;
    let Inst{23} = Rm{4};
    let Inst{22} = 0;
```
- EN: Declares TableGen `multiclass AI3strT`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI3strT`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3597-3600
```tablegen
    let Inst{11-8} = 0;
    let Inst{3-0} = Rm{3-0};
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3602-3602
```tablegen
defm STRHT : AI3strT<0b1011, "strht">;
```
- EN: Defines TableGen record `STRHT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STRHT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3604-3609
```tablegen
def STL : AIstrrel<0b00, (outs), (ins GPR:$Rt, addr_offset_none:$addr),
                   NoItinerary, "stl", "\t$Rt, $addr", []>;
def STLB : AIstrrel<0b10, (outs), (ins GPR:$Rt, addr_offset_none:$addr),
                    NoItinerary, "stlb", "\t$Rt, $addr", []>;
def STLH : AIstrrel<0b11, (outs), (ins GPR:$Rt, addr_offset_none:$addr),
                    NoItinerary, "stlh", "\t$Rt, $addr", []>;
```
- EN: Defines TableGen record `STL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3611-3611
```tablegen
} // mayStore = 1, hasSideEffects = 0
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3613-3615
```tablegen
//===----------------------------------------------------------------------===//
//  Load / store multiple Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3617-3634
```tablegen
multiclass arm_ldst_mult<string asm, string sfx, bit L_bit, bit P_bit, Format f,
                         InstrItinClass itin, InstrItinClass itin_upd> {
  // IA is the default, so no need for an explicit suffix on the
  // mnemonic here. Without it is the canonical spelling.
  def IA :
    AXI4<(outs), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         IndexModeNone, f, itin,
         !strconcat(asm, "${p}\t$Rn, $regs", sfx), "", []> {
    let Inst{24-23} = 0b01;       // Increment After
    let Inst{22}    = P_bit;
    let Inst{21}    = 0;          // No writeback
    let Inst{20}    = L_bit;
  }
  def IA_UPD :
    AXI4<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         IndexModeUpd, f, itin_upd,
         !strconcat(asm, "${p}\t$Rn!, $regs", sfx), "$Rn = $wb", []> {
    let Inst{24-23} = 0b01;       // Increment After
```
- EN: Declares TableGen `multiclass arm_ldst_mult`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass arm_ldst_mult`，它是一个可复用模板，可展开为多个相关记录。

### Lines 3635-3637
```tablegen
    let Inst{22}    = P_bit;
    let Inst{21}    = 1;          // Writeback
    let Inst{20}    = L_bit;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3639-3656
```tablegen
    let DecoderMethod = "DecodeMemMultipleWritebackInstruction";
  }
  def DA :
    AXI4<(outs), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         IndexModeNone, f, itin,
         !strconcat(asm, "da${p}\t$Rn, $regs", sfx), "", []> {
    let Inst{24-23} = 0b00;       // Decrement After
    let Inst{22}    = P_bit;
    let Inst{21}    = 0;          // No writeback
    let Inst{20}    = L_bit;
  }
  def DA_UPD :
    AXI4<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         IndexModeUpd, f, itin_upd,
         !strconcat(asm, "da${p}\t$Rn!, $regs", sfx), "$Rn = $wb", []> {
    let Inst{24-23} = 0b00;       // Decrement After
    let Inst{22}    = P_bit;
    let Inst{21}    = 1;          // Writeback
```
- EN: Defines TableGen record `DA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3657-3657
```tablegen
    let Inst{20}    = L_bit;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3659-3676
```tablegen
    let DecoderMethod = "DecodeMemMultipleWritebackInstruction";
  }
  def DB :
    AXI4<(outs), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         IndexModeNone, f, itin,
         !strconcat(asm, "db${p}\t$Rn, $regs", sfx), "", []> {
    let Inst{24-23} = 0b10;       // Decrement Before
    let Inst{22}    = P_bit;
    let Inst{21}    = 0;          // No writeback
    let Inst{20}    = L_bit;
  }
  def DB_UPD :
    AXI4<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         IndexModeUpd, f, itin_upd,
         !strconcat(asm, "db${p}\t$Rn!, $regs", sfx), "$Rn = $wb", []> {
    let Inst{24-23} = 0b10;       // Decrement Before
    let Inst{22}    = P_bit;
    let Inst{21}    = 1;          // Writeback
```
- EN: Defines TableGen record `DB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3677-3677
```tablegen
    let Inst{20}    = L_bit;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3679-3696
```tablegen
    let DecoderMethod = "DecodeMemMultipleWritebackInstruction";
  }
  def IB :
    AXI4<(outs), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         IndexModeNone, f, itin,
         !strconcat(asm, "ib${p}\t$Rn, $regs", sfx), "", []> {
    let Inst{24-23} = 0b11;       // Increment Before
    let Inst{22}    = P_bit;
    let Inst{21}    = 0;          // No writeback
    let Inst{20}    = L_bit;
  }
  def IB_UPD :
    AXI4<(outs GPR:$wb), (ins GPR:$Rn, pred:$p, reglist:$regs, variable_ops),
         IndexModeUpd, f, itin_upd,
         !strconcat(asm, "ib${p}\t$Rn!, $regs", sfx), "$Rn = $wb", []> {
    let Inst{24-23} = 0b11;       // Increment Before
    let Inst{22}    = P_bit;
    let Inst{21}    = 1;          // Writeback
```
- EN: Defines TableGen record `IB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `IB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3697-3697
```tablegen
    let Inst{20}    = L_bit;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3699-3701
```tablegen
    let DecoderMethod = "DecodeMemMultipleWritebackInstruction";
  }
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3703-3703
```tablegen
let hasSideEffects = 0 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3705-3707
```tablegen
let mayLoad = 1, hasExtraDefRegAllocReq = 1, variadicOpsAreDefs = 1 in
defm LDM : arm_ldst_mult<"ldm", "", 1, 0, LdStMulFrm, IIC_iLoad_m,
                         IIC_iLoad_mu>, ComplexDeprecationPredicate<"ARMLoad">;
```
- EN: Defines TableGen record `LDM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3709-3712
```tablegen
let mayStore = 1, hasExtraSrcRegAllocReq = 1 in
defm STM : arm_ldst_mult<"stm", "", 0, 0, LdStMulFrm, IIC_iStore_m,
                         IIC_iStore_mu>,
           ComplexDeprecationPredicate<"ARMStore">;
```
- EN: Defines TableGen record `STM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3714-3714
```tablegen
} // hasSideEffects
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3716-3724
```tablegen
// FIXME: remove when we have a way to marking a MI with these properties.
// FIXME: Should pc be an implicit operand like PICADD, etc?
let isReturn = 1, isTerminator = 1, isBarrier = 1, mayLoad = 1,
    hasExtraDefRegAllocReq = 1, isCodeGenOnly = 1 in
def LDMIA_RET : ARMPseudoExpand<(outs GPR:$wb), (ins GPR:$Rn, pred:$p,
                                                 reglist:$regs, variable_ops),
                     4, IIC_iLoad_mBr, [],
                     (LDMIA_UPD GPR:$wb, GPR:$Rn, pred:$p, reglist:$regs)>,
      RegConstraint<"$Rn = $wb">;
```
- EN: Defines TableGen record `LDMIA_RET` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDMIA_RET`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3726-3728
```tablegen
let mayLoad = 1, hasExtraDefRegAllocReq = 1 in
defm sysLDM : arm_ldst_mult<"ldm", " ^", 1, 1, LdStMulFrm, IIC_iLoad_m,
                               IIC_iLoad_mu>;
```
- EN: Defines TableGen record `sysLDM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `sysLDM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3730-3732
```tablegen
let mayStore = 1, hasExtraSrcRegAllocReq = 1 in
defm sysSTM : arm_ldst_mult<"stm", " ^", 0, 1, LdStMulFrm, IIC_iStore_m,
                               IIC_iStore_mu>;
```
- EN: Defines TableGen record `sysSTM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `sysSTM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3736-3738
```tablegen
//===----------------------------------------------------------------------===//
//  Move Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3740-3744
```tablegen
let hasSideEffects = 0, isMoveReg = 1 in
def MOVr : AsI1<0b1101, (outs GPR:$Rd), (ins GPR:$Rm), DPFrm, IIC_iMOVr,
                "mov", "\t$Rd, $Rm", []>, UnaryDP, Sched<[WriteALU]> {
  bits<4> Rd;
  bits<4> Rm;
```
- EN: Defines TableGen record `MOVr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3746-3751
```tablegen
  let Inst{19-16} = 0b0000;
  let Inst{11-4} = 0b00000000;
  let Inst{25} = 0;
  let Inst{3-0} = Rm;
  let Inst{15-12} = Rd;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3753-3758
```tablegen
// A version for the smaller set of tail call registers.
let hasSideEffects = 0 in
def MOVr_TC : AsI1<0b1101, (outs tcGPR:$Rd), (ins tcGPR:$Rm), DPFrm,
                IIC_iMOVr, "mov", "\t$Rd, $Rm", []>, UnaryDP, Sched<[WriteALU]> {
  bits<4> Rd;
  bits<4> Rm;
```
- EN: Defines TableGen record `MOVr_TC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVr_TC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3760-3764
```tablegen
  let Inst{11-4} = 0b00000000;
  let Inst{25} = 0;
  let Inst{3-0} = Rm;
  let Inst{15-12} = Rd;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 3766-3781
```tablegen
def MOVsr : AsI1<0b1101, (outs GPRnopc:$Rd), (ins shift_so_reg_reg:$src),
                DPSoRegRegFrm, IIC_iMOVsr,
                "mov", "\t$Rd, $src",
                [(set GPRnopc:$Rd, shift_so_reg_reg:$src)]>, UnaryDP,
                Sched<[WriteALU]> {
  bits<4> Rd;
  bits<12> src;
  let Inst{15-12} = Rd;
  let Inst{19-16} = 0b0000;
  let Inst{11-8} = src{11-8};
  let Inst{7} = 0;
  let Inst{6-5} = src{6-5};
  let Inst{4} = 1;
  let Inst{3-0} = src{3-0};
  let Inst{25} = 0;
}
```
- EN: Defines TableGen record `MOVsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3783-3795
```tablegen
def MOVsi : AsI1<0b1101, (outs GPR:$Rd), (ins shift_so_reg_imm:$src),
                DPSoRegImmFrm, IIC_iMOVsr,
                "mov", "\t$Rd, $src", [(set GPR:$Rd, shift_so_reg_imm:$src)]>,
                UnaryDP, Sched<[WriteALU]> {
  bits<4> Rd;
  bits<12> src;
  let Inst{15-12} = Rd;
  let Inst{19-16} = 0b0000;
  let Inst{11-5} = src{11-5};
  let Inst{4} = 0;
  let Inst{3-0} = src{3-0};
  let Inst{25} = 0;
}
```
- EN: Defines TableGen record `MOVsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3797-3807
```tablegen
let isReMaterializable = 1, isAsCheapAsAMove = 1, isMoveImm = 1 in
def MOVi : AsI1<0b1101, (outs GPR:$Rd), (ins mod_imm:$imm), DPFrm, IIC_iMOVi,
                "mov", "\t$Rd, $imm", [(set GPR:$Rd, mod_imm:$imm)]>, UnaryDP,
                Sched<[WriteALU]> {
  bits<4> Rd;
  bits<12> imm;
  let Inst{25} = 1;
  let Inst{15-12} = Rd;
  let Inst{19-16} = 0b0000;
  let Inst{11-0} = imm;
}
```
- EN: Defines TableGen record `MOVi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3809-3823
```tablegen
let isReMaterializable = 1, isAsCheapAsAMove = 1, isMoveImm = 1 in
def MOVi16 : AI1<0b1000, (outs GPR:$Rd), (ins imm0_65535_expr:$imm),
                 DPFrm, IIC_iMOVi,
                 "movw", "\t$Rd, $imm",
                 [(set GPR:$Rd, imm0_65535:$imm)]>,
                 Requires<[IsARM, HasV6T2]>, UnaryDP, Sched<[WriteALU]> {
  bits<4> Rd;
  bits<16> imm;
  let Inst{15-12} = Rd;
  let Inst{11-0}  = imm{11-0};
  let Inst{19-16} = imm{15-12};
  let Inst{20} = 0;
  let Inst{25} = 1;
  let DecoderMethod = "DecodeArmMOVTWInstruction";
}
```
- EN: Defines TableGen record `MOVi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3825-3827
```tablegen
def : InstAlias<"mov${p} $Rd, $imm",
                (MOVi16 GPR:$Rd, imm0_65535_expr:$imm, pred:$p), 0>,
        Requires<[IsARM, HasV6T2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3829-3833
```tablegen
// This gets lowered to a single 4-byte instructions
let Size = 4 in
def MOVi16_ga_pcrel : PseudoInst<(outs GPR:$Rd),
                                (ins i32imm:$addr, pclabel:$id), IIC_iMOVi, []>,
                      Sched<[WriteALU]>;
```
- EN: Defines TableGen record `MOVi16_ga_pcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVi16_ga_pcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3835-3852
```tablegen
let Constraints = "$src = $Rd" in {
def MOVTi16 : AI1<0b1010, (outs GPRnopc:$Rd),
                  (ins GPR:$src, imm0_65535_expr:$imm),
                  DPFrm, IIC_iMOVi,
                  "movt", "\t$Rd, $imm",
                  [(set GPRnopc:$Rd,
                        (or (and GPR:$src, 0xffff),
                            lo16AllZero:$imm))]>, UnaryDP,
                  Requires<[IsARM, HasV6T2]>, Sched<[WriteALU]> {
  bits<4> Rd;
  bits<16> imm;
  let Inst{15-12} = Rd;
  let Inst{11-0}  = imm{11-0};
  let Inst{19-16} = imm{15-12};
  let Inst{20} = 0;
  let Inst{25} = 1;
  let DecoderMethod = "DecodeArmMOVTWInstruction";
}
```
- EN: Defines TableGen record `MOVTi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVTi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3854-3858
```tablegen
// This gets lowered to a single 4-byte instructions
let Size = 4 in
def MOVTi16_ga_pcrel : PseudoInst<(outs GPR:$Rd),
                      (ins GPR:$src, i32imm:$addr, pclabel:$id), IIC_iMOVi, []>,
                      Sched<[WriteALU]>;
```
- EN: Defines TableGen record `MOVTi16_ga_pcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVTi16_ga_pcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3860-3860
```tablegen
} // Constraints
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3862-3863
```tablegen
def : ARMPat<(or GPR:$src, 0xffff0000), (MOVTi16 GPR:$src, 0xffff)>,
      Requires<[IsARM, HasV6T2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3865-3868
```tablegen
let Uses = [CPSR] in
def RRX : PseudoInst<(outs GPR:$Rd), (ins GPR:$Rm), IIC_iMOVsi,
                     [(set GPR:$Rd, (ARMrrx GPR:$Rm, CPSR))]>,
          UnaryDP, Requires<[IsARM]>, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `RRX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RRX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3870-3877
```tablegen
let Defs = [CPSR] in {
  def LSRs1 : PseudoInst<(outs GPR:$dst), (ins GPR:$src), IIC_iMOVsi,
                         [(set GPR:$dst, CPSR, (ARMlsrs1 GPR:$src))]>,
              UnaryDP, Sched<[WriteALU]>, Requires<[IsARM]>;
  def ASRs1 : PseudoInst<(outs GPR:$dst), (ins GPR:$src), IIC_iMOVsi,
                         [(set GPR:$dst, CPSR, (ARMasrs1 GPR:$src))]>,
              UnaryDP, Sched<[WriteALU]>, Requires<[IsARM]>;
}
```
- EN: Defines TableGen record `LSRs1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LSRs1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3879-3881
```tablegen
//===----------------------------------------------------------------------===//
//  Extend Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3883-3883
```tablegen
// Sign extenders
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3885-3888
```tablegen
def SXTB  : AI_ext_rrot<0b01101010,
                         "sxtb", UnOpFrag<(sext_inreg node:$Src, i8)>>;
def SXTH  : AI_ext_rrot<0b01101011,
                         "sxth", UnOpFrag<(sext_inreg node:$Src, i16)>>;
```
- EN: Defines TableGen record `SXTB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SXTB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3890-3893
```tablegen
def SXTAB : AI_exta_rrot<0b01101010,
               "sxtab", BinOpFrag<(add node:$LHS, (sext_inreg node:$RHS, i8))>>;
def SXTAH : AI_exta_rrot<0b01101011,
               "sxtah", BinOpFrag<(add node:$LHS, (sext_inreg node:$RHS,i16))>>;
```
- EN: Defines TableGen record `SXTAB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SXTAB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3895-3899
```tablegen
def : ARMV6Pat<(add rGPR:$Rn, (sext_inreg (srl rGPR:$Rm, rot_imm:$rot), i8)),
               (SXTAB rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
def : ARMV6Pat<(add rGPR:$Rn, (sext_inreg (srl rGPR:$Rm, imm8_or_16:$rot),
                                          i16)),
               (SXTAH rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3901-3905
```tablegen
def SXTB16  : AI_ext_rrot_np<0b01101000, "sxtb16">;
def : ARMV6Pat<(int_arm_sxtb16 GPR:$Src),
               (SXTB16 GPR:$Src, 0)>;
def : ARMV6Pat<(int_arm_sxtb16 (rotr GPR:$Src, rot_imm:$rot)),
               (SXTB16 GPR:$Src, rot_imm:$rot)>;
```
- EN: Defines TableGen record `SXTB16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SXTB16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3907-3911
```tablegen
def SXTAB16 : AI_exta_rrot_np<0b01101000, "sxtab16">;
def : ARMV6Pat<(int_arm_sxtab16 GPR:$LHS, GPR:$RHS),
               (SXTAB16 GPR:$LHS, GPR:$RHS, 0)>;
def : ARMV6Pat<(int_arm_sxtab16 GPR:$LHS, (rotr GPR:$RHS, rot_imm:$rot)),
               (SXTAB16 GPR:$LHS, GPR:$RHS, rot_imm:$rot)>;
```
- EN: Defines TableGen record `SXTAB16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SXTAB16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3913-3913
```tablegen
// Zero extenders
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 3915-3921
```tablegen
let AddedComplexity = 16 in {
def UXTB   : AI_ext_rrot<0b01101110,
                          "uxtb"  , UnOpFrag<(and node:$Src, 0x000000FF)>>;
def UXTH   : AI_ext_rrot<0b01101111,
                          "uxth"  , UnOpFrag<(and node:$Src, 0x0000FFFF)>>;
def UXTB16 : AI_ext_rrot<0b01101100,
                          "uxtb16", UnOpFrag<(and node:$Src, 0x00FF00FF)>>;
```
- EN: Defines TableGen record `UXTB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UXTB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3923-3934
```tablegen
// FIXME: This pattern incorrectly assumes the shl operator is a rotate.
//        The transformation should probably be done as a combiner action
//        instead so we can include a check for masking back in the upper
//        eight bits of the source into the lower eight bits of the result.
//def : ARMV6Pat<(and (shl GPR:$Src, (i32 8)), 0xFF00FF),
//               (UXTB16r_rot GPR:$Src, 3)>;
def : ARMV6Pat<(and (srl GPR:$Src, (i32 8)), 0xFF00FF),
               (UXTB16 GPR:$Src, 1)>;
def : ARMV6Pat<(int_arm_uxtb16 GPR:$Src),
               (UXTB16 GPR:$Src, 0)>;
def : ARMV6Pat<(int_arm_uxtb16 (rotr GPR:$Src, rot_imm:$rot)),
               (UXTB16 GPR:$Src, rot_imm:$rot)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3936-3939
```tablegen
def UXTAB : AI_exta_rrot<0b01101110, "uxtab",
                        BinOpFrag<(add node:$LHS, (and node:$RHS, 0x00FF))>>;
def UXTAH : AI_exta_rrot<0b01101111, "uxtah",
                        BinOpFrag<(add node:$LHS, (and node:$RHS, 0xFFFF))>>;
```
- EN: Defines TableGen record `UXTAB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UXTAB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3941-3945
```tablegen
def : ARMV6Pat<(add rGPR:$Rn, (and (srl rGPR:$Rm, rot_imm:$rot), 0xFF)),
               (UXTAB rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
def : ARMV6Pat<(add rGPR:$Rn, (and (srl rGPR:$Rm, imm8_or_16:$rot), 0xFFFF)),
               (UXTAH rGPR:$Rn, rGPR:$Rm, rot_imm:$rot)>;
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 3947-3952
```tablegen
// This isn't safe in general, the add is two 16-bit units, not a 32-bit add.
def UXTAB16 : AI_exta_rrot_np<0b01101100, "uxtab16">;
def : ARMV6Pat<(int_arm_uxtab16 GPR:$LHS, GPR:$RHS),
               (UXTAB16 GPR:$LHS, GPR:$RHS, 0)>;
def : ARMV6Pat<(int_arm_uxtab16 GPR:$LHS, (rotr GPR:$RHS, rot_imm:$rot)),
               (UXTAB16 GPR:$LHS, GPR:$RHS, rot_imm:$rot)>;
```
- EN: Defines TableGen record `UXTAB16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UXTAB16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3955-3971
```tablegen
def SBFX  : I<(outs GPRnopc:$Rd),
              (ins GPRnopc:$Rn, imm0_31:$lsb, imm1_32:$width),
               AddrMode1, 4, IndexModeNone, DPFrm, IIC_iUNAsi,
               "sbfx", "\t$Rd, $Rn, $lsb, $width", "", []>,
               Requires<[IsARM, HasV6T2]> {
  bits<4> Rd;
  bits<4> Rn;
  bits<5> lsb;
  bits<5> width;
  let Inst{27-21} = 0b0111101;
  let Inst{6-4}   = 0b101;
  let Inst{20-16} = width;
  let Inst{15-12} = Rd;
  let Inst{11-7}  = lsb;
  let Inst{3-0}   = Rn;
  let hasSideEffects = 0;
}
```
- EN: Defines TableGen record `SBFX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SBFX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3973-3989
```tablegen
def UBFX  : I<(outs GPRnopc:$Rd),
              (ins GPRnopc:$Rn, imm0_31:$lsb, imm1_32:$width),
               AddrMode1, 4, IndexModeNone, DPFrm, IIC_iUNAsi,
               "ubfx", "\t$Rd, $Rn, $lsb, $width", "", []>,
               Requires<[IsARM, HasV6T2]> {
  bits<4> Rd;
  bits<4> Rn;
  bits<5> lsb;
  bits<5> width;
  let Inst{27-21} = 0b0111111;
  let Inst{6-4}   = 0b101;
  let Inst{20-16} = width;
  let Inst{15-12} = Rd;
  let Inst{11-7}  = lsb;
  let Inst{3-0}   = Rn;
  let hasSideEffects = 0;
}
```
- EN: Defines TableGen record `UBFX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UBFX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 3991-3993
```tablegen
//===----------------------------------------------------------------------===//
//  Arithmetic Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 3995-3999
```tablegen
let isAdd = 1 in
defm ADD  : AsI1_bin_irs<0b0100, "add",
                         IIC_iALUi, IIC_iALUr, IIC_iALUsr, add, 1>;
defm SUB  : AsI1_bin_irs<0b0010, "sub",
                         IIC_iALUi, IIC_iALUr, IIC_iALUsr, sub>;
```
- EN: Defines TableGen record `ADD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4001-4013
```tablegen
// ADD and SUB with 's' bit set.
//
// Currently, ADDS/SUBS are pseudo opcodes that exist only in the
// selection DAG. They are "lowered" to real ADD/SUB opcodes by
// AdjustInstrPostInstrSelection where we determine whether or not to
// set the "s" bit based on CPSR liveness.
//
// FIXME: Eliminate ADDS/SUBS pseudo opcodes after adding tablegen
// support for an optional CPSR definition that corresponds to the DAG
// node's second value. We can then eliminate the implicit def of CPSR.
let isAdd = 1 in
defm ADDS : AsI1_bin_s_irs<IIC_iALUi, IIC_iALUr, IIC_iALUsr, ARMaddc, 1>;
defm SUBS : AsI1_bin_s_irs<IIC_iALUi, IIC_iALUr, IIC_iALUsr, ARMsubc>;
```
- EN: Defines TableGen record `of` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `of`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4015-4017
```tablegen
let isAdd = 1 in
defm ADC : AI1_adde_sube_irs<0b0101, "adc", ARMadde, 1>;
defm SBC : AI1_adde_sube_irs<0b0110, "sbc", ARMsube>;
```
- EN: Defines TableGen record `ADC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ADC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4019-4021
```tablegen
defm RSB  : AsI1_rbin_irs<0b0011, "rsb",
                          IIC_iALUi, IIC_iALUr, IIC_iALUsr,
                          sub>;
```
- EN: Defines TableGen record `RSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4023-4025
```tablegen
// FIXME: Eliminate them if we can write def : Pat patterns which defines
// CPSR and the implicit def of CPSR is not needed.
defm RSBS : AsI1_rbin_s_is<IIC_iALUi, IIC_iALUsr, ARMsubc>;
```
- EN: Defines TableGen record `of` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `of`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4027-4027
```tablegen
defm RSC : AI1_rsc_irs<0b0111, "rsc", ARMsube>;
```
- EN: Defines TableGen record `RSC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RSC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4029-4037
```tablegen
// (sub X, imm) gets canonicalized to (add X, -imm).  Match this form.
// The assume-no-carry-in form uses the negation of the input since add/sub
// assume opposite meanings of the carry flag (i.e., carry == !borrow).
// See the definition of AddWithCarry() in the ARM ARM A2.2.1 for the gory
// details.
def : ARMPat<(add     GPR:$src, mod_imm_neg:$imm),
             (SUBri   GPR:$src, mod_imm_neg:$imm)>;
def : ARMPat<(ARMaddc GPR:$src, mod_imm_neg:$imm),
             (SUBSri  GPR:$src, mod_imm_neg:$imm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4039-4044
```tablegen
def : ARMPat<(add     GPR:$src, imm0_65535_neg:$imm),
             (SUBrr   GPR:$src, (MOVi16 (imm_neg_XFORM imm:$imm)))>,
             Requires<[IsARM, HasV6T2]>;
def : ARMPat<(ARMaddc GPR:$src, imm0_65535_neg:$imm),
             (SUBSrr  GPR:$src, (MOVi16 (imm_neg_XFORM imm:$imm)))>,
             Requires<[IsARM, HasV6T2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4046-4053
```tablegen
// The with-carry-in form matches bitwise not instead of the negation.
// Effectively, the inverse interpretation of the carry flag already accounts
// for part of the negation.
def : ARMPat<(ARMadde GPR:$src, mod_imm_not:$imm, CPSR),
             (SBCri   GPR:$src, mod_imm_not:$imm)>;
def : ARMPat<(ARMadde GPR:$src, imm0_65535_neg:$imm, CPSR),
             (SBCrr   GPR:$src, (MOVi16 (imm_not_XFORM imm:$imm)))>,
             Requires<[IsARM, HasV6T2]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4055-4059
```tablegen
// Note: These are implemented in C++ code, because they have to generate
// ADD/SUBrs instructions, which use a complex pattern that a xform function
// cannot produce.
// (mul X, 2^n+1) -> (add (X << n), X)
// (mul X, 2^n-1) -> (rsb X, (X << n))
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4061-4076
```tablegen
// ARM Arithmetic Instruction
// GPR:$dst = GPR:$a op GPR:$b
class AAI<bits<8> op27_20, bits<8> op11_4, string opc,
          list<dag> pattern = [],
          dag iops = (ins GPRnopc:$Rn, GPRnopc:$Rm),
          string asm = "\t$Rd, $Rn, $Rm">
  : AI<(outs GPRnopc:$Rd), iops, DPFrm, IIC_iALUr, opc, asm, pattern>,
    Sched<[WriteALU, ReadALU, ReadALU]> {
  bits<4> Rn;
  bits<4> Rd;
  bits<4> Rm;
  let Inst{27-20} = op27_20;
  let Inst{11-4} = op11_4;
  let Inst{19-16} = Rn;
  let Inst{15-12} = Rd;
  let Inst{3-0}   = Rm;
```
- EN: Declares reusable TableGen class `AAI` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AAI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4078-4079
```tablegen
  let Unpredictable{11-8} = 0b1111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4081-4087
```tablegen
// Wrappers around the AAI class
class AAIRevOpr<bits<8> op27_20, bits<8> op11_4, string opc,
                list<dag> pattern = []>
  : AAI<op27_20, op11_4, opc,
        pattern,
        (ins GPRnopc:$Rm, GPRnopc:$Rn),
        "\t$Rd, $Rm, $Rn">;
```
- EN: Declares reusable TableGen class `class` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `class`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4089-4092
```tablegen
class AAIIntrinsic<bits<8> op27_20, bits<8> op11_4, string opc,
                 Intrinsic intrinsic>
  : AAI<op27_20, op11_4, opc,
        [(set GPRnopc:$Rd, (intrinsic GPRnopc:$Rn, GPRnopc:$Rm))]>;
```
- EN: Declares reusable TableGen class `AAIIntrinsic` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AAIIntrinsic`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4094-4099
```tablegen
// Saturating add/subtract
let hasSideEffects = 1 in {
def QADD8   : AAIIntrinsic<0b01100010, 0b11111001, "qadd8", int_arm_qadd8>;
def QADD16  : AAIIntrinsic<0b01100010, 0b11110001, "qadd16", int_arm_qadd16>;
def QSUB16  : AAIIntrinsic<0b01100010, 0b11110111, "qsub16", int_arm_qsub16>;
def QSUB8   : AAIIntrinsic<0b01100010, 0b11111111, "qsub8", int_arm_qsub8>;
```
- EN: Defines TableGen record `QADD8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `QADD8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4101-4112
```tablegen
def QDADD   : AAIRevOpr<0b00010100, 0b00000101, "qdadd",
              [(set GPRnopc:$Rd, (int_arm_qadd GPRnopc:$Rm,
                                  (int_arm_qadd GPRnopc:$Rn, GPRnopc:$Rn)))]>;
def QDSUB   : AAIRevOpr<0b00010110, 0b00000101, "qdsub",
              [(set GPRnopc:$Rd, (int_arm_qsub GPRnopc:$Rm,
                                  (int_arm_qadd GPRnopc:$Rn, GPRnopc:$Rn)))]>;
def QSUB    : AAIRevOpr<0b00010010, 0b00000101, "qsub",
              [(set GPRnopc:$Rd, (int_arm_qsub GPRnopc:$Rm, GPRnopc:$Rn))]>;
let DecoderMethod = "DecodeQADDInstruction" in
  def QADD    : AAIRevOpr<0b00010000, 0b00000101, "qadd",
                [(set GPRnopc:$Rd, (int_arm_qadd GPRnopc:$Rm, GPRnopc:$Rn))]>;
}
```
- EN: Defines TableGen record `QDADD` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `QDADD`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4114-4121
```tablegen
def : ARMV5TEPat<(saddsat GPR:$a, GPR:$b),
                 (QADD GPR:$a, GPR:$b)>;
def : ARMV5TEPat<(ssubsat GPR:$a, GPR:$b),
                 (QSUB GPR:$a, GPR:$b)>;
def : ARMV5TEPat<(saddsat rGPR:$Rm, (saddsat rGPR:$Rn, rGPR:$Rn)),
                 (QDADD rGPR:$Rm, rGPR:$Rn)>;
def : ARMV5TEPat<(ssubsat rGPR:$Rm, (saddsat rGPR:$Rn, rGPR:$Rn)),
                 (QDSUB rGPR:$Rm, rGPR:$Rn)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4123-4130
```tablegen
def : ARMV6Pat<(ARMqadd8b rGPR:$Rm, rGPR:$Rn),
               (QADD8 rGPR:$Rm, rGPR:$Rn)>;
def : ARMV6Pat<(ARMqsub8b rGPR:$Rm, rGPR:$Rn),
               (QSUB8 rGPR:$Rm, rGPR:$Rn)>;
def : ARMV6Pat<(ARMqadd16b rGPR:$Rm, rGPR:$Rn),
               (QADD16 rGPR:$Rm, rGPR:$Rn)>;
def : ARMV6Pat<(ARMqsub16b rGPR:$Rm, rGPR:$Rn),
               (QSUB16 rGPR:$Rm, rGPR:$Rn)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4132-4139
```tablegen
def UQADD16 : AAIIntrinsic<0b01100110, 0b11110001, "uqadd16", int_arm_uqadd16>;
def UQADD8  : AAIIntrinsic<0b01100110, 0b11111001, "uqadd8", int_arm_uqadd8>;
def UQSUB16 : AAIIntrinsic<0b01100110, 0b11110111, "uqsub16", int_arm_uqsub16>;
def UQSUB8  : AAIIntrinsic<0b01100110, 0b11111111, "uqsub8", int_arm_uqsub8>;
def QASX    : AAIIntrinsic<0b01100010, 0b11110011, "qasx", int_arm_qasx>;
def QSAX    : AAIIntrinsic<0b01100010, 0b11110101, "qsax", int_arm_qsax>;
def UQASX   : AAIIntrinsic<0b01100110, 0b11110011, "uqasx", int_arm_uqasx>;
def UQSAX   : AAIIntrinsic<0b01100110, 0b11110101, "uqsax", int_arm_uqsax>;
```
- EN: Defines TableGen record `UQADD16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UQADD16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4141-4148
```tablegen
def : ARMV6Pat<(ARMuqadd8b rGPR:$Rm, rGPR:$Rn),
               (UQADD8 rGPR:$Rm, rGPR:$Rn)>;
def : ARMV6Pat<(ARMuqsub8b rGPR:$Rm, rGPR:$Rn),
               (UQSUB8 rGPR:$Rm, rGPR:$Rn)>;
def : ARMV6Pat<(ARMuqadd16b rGPR:$Rm, rGPR:$Rn),
               (UQADD16 rGPR:$Rm, rGPR:$Rn)>;
def : ARMV6Pat<(ARMuqsub16b rGPR:$Rm, rGPR:$Rn),
               (UQSUB16 rGPR:$Rm, rGPR:$Rn)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4151-4151
```tablegen
// Signed/Unsigned add/subtract
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4153-4164
```tablegen
def SASX   : AAIIntrinsic<0b01100001, 0b11110011, "sasx", int_arm_sasx>;
def SADD16 : AAIIntrinsic<0b01100001, 0b11110001, "sadd16", int_arm_sadd16>;
def SADD8  : AAIIntrinsic<0b01100001, 0b11111001, "sadd8", int_arm_sadd8>;
def SSAX   : AAIIntrinsic<0b01100001, 0b11110101, "ssax", int_arm_ssax>;
def SSUB16 : AAIIntrinsic<0b01100001, 0b11110111, "ssub16", int_arm_ssub16>;
def SSUB8  : AAIIntrinsic<0b01100001, 0b11111111, "ssub8", int_arm_ssub8>;
def UASX   : AAIIntrinsic<0b01100101, 0b11110011, "uasx", int_arm_uasx>;
def UADD16 : AAIIntrinsic<0b01100101, 0b11110001, "uadd16", int_arm_uadd16>;
def UADD8  : AAIIntrinsic<0b01100101, 0b11111001, "uadd8", int_arm_uadd8>;
def USAX   : AAIIntrinsic<0b01100101, 0b11110101, "usax", int_arm_usax>;
def USUB16 : AAIIntrinsic<0b01100101, 0b11110111, "usub16", int_arm_usub16>;
def USUB8  : AAIIntrinsic<0b01100101, 0b11111111, "usub8", int_arm_usub8>;
```
- EN: Defines TableGen record `SASX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SASX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4166-4166
```tablegen
// Signed/Unsigned halving add/subtract
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4168-4179
```tablegen
def SHASX   : AAIIntrinsic<0b01100011, 0b11110011, "shasx", int_arm_shasx>;
def SHADD16 : AAIIntrinsic<0b01100011, 0b11110001, "shadd16", int_arm_shadd16>;
def SHADD8  : AAIIntrinsic<0b01100011, 0b11111001, "shadd8", int_arm_shadd8>;
def SHSAX   : AAIIntrinsic<0b01100011, 0b11110101, "shsax", int_arm_shsax>;
def SHSUB16 : AAIIntrinsic<0b01100011, 0b11110111, "shsub16", int_arm_shsub16>;
def SHSUB8  : AAIIntrinsic<0b01100011, 0b11111111, "shsub8", int_arm_shsub8>;
def UHASX   : AAIIntrinsic<0b01100111, 0b11110011, "uhasx", int_arm_uhasx>;
def UHADD16 : AAIIntrinsic<0b01100111, 0b11110001, "uhadd16", int_arm_uhadd16>;
def UHADD8  : AAIIntrinsic<0b01100111, 0b11111001, "uhadd8", int_arm_uhadd8>;
def UHSAX   : AAIIntrinsic<0b01100111, 0b11110101, "uhsax", int_arm_uhsax>;
def UHSUB16 : AAIIntrinsic<0b01100111, 0b11110111, "uhsub16", int_arm_uhsub16>;
def UHSUB8  : AAIIntrinsic<0b01100111, 0b11111111, "uhsub8", int_arm_uhsub8>;
```
- EN: Defines TableGen record `SHASX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SHASX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4181-4181
```tablegen
// Unsigned Sum of Absolute Differences [and Accumulate].
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 4183-4200
```tablegen
def USAD8  : AI<(outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
                MulFrm /* for convenience */, NoItinerary, "usad8",
                "\t$Rd, $Rn, $Rm",
             [(set GPR:$Rd, (int_arm_usad8 GPR:$Rn, GPR:$Rm))]>,
             Requires<[IsARM, HasV6]>, Sched<[WriteALU, ReadALU, ReadALU]> {
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
  let Inst{27-20} = 0b01111000;
  let Inst{15-12} = 0b1111;
  let Inst{7-4} = 0b0001;
  let Inst{19-16} = Rd;
  let Inst{11-8} = Rm;
  let Inst{3-0} = Rn;
}
def USADA8 : AI<(outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm, GPR:$Ra),
                MulFrm /* for convenience */, NoItinerary, "usada8",
                "\t$Rd, $Rn, $Rm, $Ra",
```
- EN: Defines TableGen record `USAD8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `USAD8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4201-4213
```tablegen
             [(set GPR:$Rd, (int_arm_usada8 GPR:$Rn, GPR:$Rm, GPR:$Ra))]>,
             Requires<[IsARM, HasV6]>, Sched<[WriteALU, ReadALU, ReadALU]>{
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
  bits<4> Ra;
  let Inst{27-20} = 0b01111000;
  let Inst{7-4} = 0b0001;
  let Inst{19-16} = Rd;
  let Inst{15-12} = Ra;
  let Inst{11-8} = Rm;
  let Inst{3-0} = Rn;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4215-4231
```tablegen
// Signed/Unsigned saturate
def SSAT : AI<(outs GPRnopc:$Rd),
              (ins imm1_32:$sat_imm, GPRnopc:$Rn, shift_imm:$sh),
              SatFrm, NoItinerary, "ssat", "\t$Rd, $sat_imm, $Rn$sh", []>,
              Requires<[IsARM,HasV6]>{
  bits<4> Rd;
  bits<5> sat_imm;
  bits<4> Rn;
  bits<8> sh;
  let Inst{27-21} = 0b0110101;
  let Inst{5-4} = 0b01;
  let Inst{20-16} = sat_imm;
  let Inst{15-12} = Rd;
  let Inst{11-7} = sh{4-0};
  let Inst{6} = sh{5};
  let Inst{3-0} = Rn;
}
```
- EN: Defines TableGen record `SSAT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SSAT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4233-4245
```tablegen
def SSAT16 : AI<(outs GPRnopc:$Rd),
                (ins imm1_16:$sat_imm, GPRnopc:$Rn), SatFrm,
                NoItinerary, "ssat16", "\t$Rd, $sat_imm, $Rn", []>,
                Requires<[IsARM,HasV6]>{
  bits<4> Rd;
  bits<4> sat_imm;
  bits<4> Rn;
  let Inst{27-20} = 0b01101010;
  let Inst{11-4} = 0b11110011;
  let Inst{15-12} = Rd;
  let Inst{19-16} = sat_imm;
  let Inst{3-0} = Rn;
}
```
- EN: Defines TableGen record `SSAT16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SSAT16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4247-4262
```tablegen
def USAT : AI<(outs GPRnopc:$Rd),
              (ins imm0_31:$sat_imm, GPRnopc:$Rn, shift_imm:$sh),
              SatFrm, NoItinerary, "usat", "\t$Rd, $sat_imm, $Rn$sh", []>,
              Requires<[IsARM,HasV6]> {
  bits<4> Rd;
  bits<5> sat_imm;
  bits<4> Rn;
  bits<8> sh;
  let Inst{27-21} = 0b0110111;
  let Inst{5-4} = 0b01;
  let Inst{15-12} = Rd;
  let Inst{11-7} = sh{4-0};
  let Inst{6} = sh{5};
  let Inst{20-16} = sat_imm;
  let Inst{3-0} = Rn;
}
```
- EN: Defines TableGen record `USAT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `USAT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4264-4276
```tablegen
def USAT16 : AI<(outs GPRnopc:$Rd),
                (ins imm0_15:$sat_imm, GPRnopc:$Rn), SatFrm,
                NoItinerary, "usat16", "\t$Rd, $sat_imm, $Rn", []>,
                Requires<[IsARM,HasV6]>{
  bits<4> Rd;
  bits<4> sat_imm;
  bits<4> Rn;
  let Inst{27-20} = 0b01101110;
  let Inst{11-4} = 0b11110011;
  let Inst{15-12} = Rd;
  let Inst{19-16} = sat_imm;
  let Inst{3-0} = Rn;
}
```
- EN: Defines TableGen record `USAT16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `USAT16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4278-4295
```tablegen
def : ARMV6Pat<(int_arm_ssat GPRnopc:$a, imm1_32:$pos),
               (SSAT imm1_32:$pos, GPRnopc:$a, 0)>;
def : ARMV6Pat<(int_arm_usat GPRnopc:$a, imm0_31:$pos),
               (USAT imm0_31:$pos, GPRnopc:$a, 0)>;
def : ARMPat<(ARMssat GPRnopc:$Rn, imm0_31:$imm),
             (SSAT imm0_31:$imm, GPRnopc:$Rn, 0)>;
def : ARMPat<(ARMusat GPRnopc:$Rn, imm0_31:$imm),
             (USAT imm0_31:$imm, GPRnopc:$Rn, 0)>;
def : ARMV6Pat<(int_arm_ssat16 GPRnopc:$a, imm1_16:$pos),
               (SSAT16 imm1_16:$pos, GPRnopc:$a)>;
def : ARMV6Pat<(int_arm_usat16 GPRnopc:$a, imm0_15:$pos),
               (USAT16 imm0_15:$pos, GPRnopc:$a)>;
def : ARMV6Pat<(int_arm_ssat (shl GPRnopc:$a, imm0_31:$shft), imm1_32:$pos),
               (SSAT imm1_32:$pos, GPRnopc:$a, imm0_31:$shft)>;
def : ARMV6Pat<(int_arm_ssat (sra GPRnopc:$a, asr_imm:$shft), imm1_32:$pos),
               (SSAT imm1_32:$pos, GPRnopc:$a, asr_imm:$shft)>;
def : ARMV6Pat<(int_arm_usat (shl GPRnopc:$a, imm0_31:$shft), imm0_31:$pos),
               (USAT imm0_31:$pos, GPRnopc:$a, imm0_31:$shft)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4296-4305
```tablegen
def : ARMV6Pat<(int_arm_usat (sra GPRnopc:$a, asr_imm:$shft), imm0_31:$pos),
               (USAT imm0_31:$pos, GPRnopc:$a, asr_imm:$shft)>;
def : ARMPat<(ARMssat (shl GPRnopc:$Rn, imm0_31:$shft), imm0_31:$pos),
               (SSAT imm0_31:$pos, GPRnopc:$Rn, imm0_31:$shft)>;                            
def : ARMPat<(ARMssat (sra GPRnopc:$Rn, asr_imm:$shft), imm0_31:$pos),
               (SSAT imm0_31:$pos, GPRnopc:$Rn, asr_imm:$shft)>;
def : ARMPat<(ARMusat (shl GPRnopc:$Rn, imm0_31:$shft), imm0_31:$pos),
               (USAT imm0_31:$pos, GPRnopc:$Rn, imm0_31:$shft)>;  
def : ARMPat<(ARMusat (sra GPRnopc:$Rn, asr_imm:$shft), imm0_31:$pos),
               (USAT imm0_31:$pos, GPRnopc:$Rn, asr_imm:$shft)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4308-4310
```tablegen
//===----------------------------------------------------------------------===//
//  Bitwise Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4312-4320
```tablegen
defm AND   : AsI1_bin_irs<0b0000, "and",
                          IIC_iBITi, IIC_iBITr, IIC_iBITsr, and, 1>;
defm ORR   : AsI1_bin_irs<0b1100, "orr",
                          IIC_iBITi, IIC_iBITr, IIC_iBITsr, or, 1>;
defm EOR   : AsI1_bin_irs<0b0001, "eor",
                          IIC_iBITi, IIC_iBITr, IIC_iBITsr, xor, 1>;
defm BIC   : AsI1_bin_irs<0b1110, "bic",
                          IIC_iBITi, IIC_iBITr, IIC_iBITsr,
                          BinOpFrag<(and node:$LHS, (not node:$RHS))>>;
```
- EN: Defines TableGen record `AND` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `AND`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4322-4338
```tablegen
// FIXME: bf_inv_mask_imm should be two operands, the lsb and the msb, just
// like in the actual instruction encoding. The complexity of mapping the mask
// to the lsb/msb pair should be handled by ISel, not encapsulated in the
// instruction description.
def BFC    : I<(outs GPR:$Rd), (ins GPR:$src, bf_inv_mask_imm:$imm),
               AddrMode1, 4, IndexModeNone, DPFrm, IIC_iUNAsi,
               "bfc", "\t$Rd, $imm", "$src = $Rd",
               [(set GPR:$Rd, (and GPR:$src, bf_inv_mask_imm:$imm))]>,
               Requires<[IsARM, HasV6T2]> {
  bits<4> Rd;
  bits<10> imm;
  let Inst{27-21} = 0b0111110;
  let Inst{6-0}   = 0b0011111;
  let Inst{15-12} = Rd;
  let Inst{11-7}  = imm{4-0}; // lsb
  let Inst{20-16} = imm{9-5}; // msb
}
```
- EN: Defines TableGen record `BFC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BFC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4340-4356
```tablegen
// A8.6.18  BFI - Bitfield insert (Encoding A1)
def BFI:I<(outs GPRnopc:$Rd), (ins GPRnopc:$src, GPR:$Rn, bf_inv_mask_imm:$imm),
          AddrMode1, 4, IndexModeNone, DPFrm, IIC_iUNAsi,
          "bfi", "\t$Rd, $Rn, $imm", "$src = $Rd",
          [(set GPRnopc:$Rd, (ARMbfi GPRnopc:$src, GPR:$Rn,
                           bf_inv_mask_imm:$imm))]>,
          Requires<[IsARM, HasV6T2]> {
  bits<4> Rd;
  bits<4> Rn;
  bits<10> imm;
  let Inst{27-21} = 0b0111110;
  let Inst{6-4}   = 0b001; // Rn: Inst{3-0} != 15
  let Inst{15-12} = Rd;
  let Inst{11-7}  = imm{4-0}; // lsb
  let Inst{20-16} = imm{9-5}; // width
  let Inst{3-0}   = Rn;
}
```
- EN: Defines TableGen record `BFI:I` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BFI:I`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4358-4367
```tablegen
def  MVNr  : AsI1<0b1111, (outs GPR:$Rd), (ins GPR:$Rm), DPFrm, IIC_iMVNr,
                  "mvn", "\t$Rd, $Rm",
                  [(set GPR:$Rd, (not GPR:$Rm))]>, UnaryDP, Sched<[WriteALU]> {
  bits<4> Rd;
  bits<4> Rm;
  let Inst{25} = 0;
  let Inst{19-16} = 0b0000;
  let Inst{11-4} = 0b00000000;
  let Inst{15-12} = Rd;
  let Inst{3-0} = Rm;
```
- EN: Defines TableGen record `MVNr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVNr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4369-4382
```tablegen
  let Unpredictable{19-16} = 0b1111;
}
def  MVNsi  : AsI1<0b1111, (outs GPR:$Rd), (ins so_reg_imm:$shift),
                  DPSoRegImmFrm, IIC_iMVNsr, "mvn", "\t$Rd, $shift",
                  [(set GPR:$Rd, (not so_reg_imm:$shift))]>, UnaryDP,
                  Sched<[WriteALU]> {
  bits<4> Rd;
  bits<12> shift;
  let Inst{25} = 0;
  let Inst{19-16} = 0b0000;
  let Inst{15-12} = Rd;
  let Inst{11-5} = shift{11-5};
  let Inst{4} = 0;
  let Inst{3-0} = shift{3-0};
```
- EN: Defines TableGen record `MVNsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVNsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4384-4399
```tablegen
  let Unpredictable{19-16} = 0b1111;
}
def  MVNsr  : AsI1<0b1111, (outs GPRnopc:$Rd), (ins so_reg_reg:$shift),
                  DPSoRegRegFrm, IIC_iMVNsr, "mvn", "\t$Rd, $shift",
                  [(set GPRnopc:$Rd, (not so_reg_reg:$shift))]>, UnaryDP,
                  Sched<[WriteALU]> {
  bits<4> Rd;
  bits<12> shift;
  let Inst{25} = 0;
  let Inst{19-16} = 0b0000;
  let Inst{15-12} = Rd;
  let Inst{11-8} = shift{11-8};
  let Inst{7} = 0;
  let Inst{6-5} = shift{6-5};
  let Inst{4} = 1;
  let Inst{3-0} = shift{3-0};
```
- EN: Defines TableGen record `MVNsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVNsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4401-4413
```tablegen
  let Unpredictable{19-16} = 0b1111;
}
let isReMaterializable = 1, isAsCheapAsAMove = 1, isMoveImm = 1 in
def  MVNi  : AsI1<0b1111, (outs GPR:$Rd), (ins mod_imm:$imm), DPFrm,
                  IIC_iMVNi, "mvn", "\t$Rd, $imm",
                  [(set GPR:$Rd, mod_imm_not:$imm)]>,UnaryDP, Sched<[WriteALU]> {
  bits<4> Rd;
  bits<12> imm;
  let Inst{25} = 1;
  let Inst{19-16} = 0b0000;
  let Inst{15-12} = Rd;
  let Inst{11-0} = imm;
}
```
- EN: Defines TableGen record `MVNi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVNi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4415-4417
```tablegen
let AddedComplexity = 1 in
def : ARMPat<(and   GPR:$src, mod_imm_not:$imm),
             (BICri GPR:$src, mod_imm_not:$imm)>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4419-4436
```tablegen
//===----------------------------------------------------------------------===//
//  Multiply Instructions.
//
class AsMul1I32<bits<7> opcod, dag oops, dag iops, InstrItinClass itin,
             string opc, string asm, list<dag> pattern>
  : AsMul1I<opcod, oops, iops, itin, opc, asm, pattern> {
  bits<4> Rd;
  bits<4> Rm;
  bits<4> Rn;
  let Inst{19-16} = Rd;
  let Inst{11-8}  = Rm;
  let Inst{3-0}   = Rn;
}
class AsMul1I64<bits<7> opcod, dag oops, dag iops, InstrItinClass itin,
             string opc, string asm, list<dag> pattern>
  : AsMul1I<opcod, oops, iops, itin, opc, asm, pattern> {
  bits<4> RdLo;
  bits<4> RdHi;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4437-4454
```tablegen
  bits<4> Rm;
  bits<4> Rn;
  let Inst{19-16} = RdHi;
  let Inst{15-12} = RdLo;
  let Inst{11-8}  = Rm;
  let Inst{3-0}   = Rn;
}
class AsMla1I64<bits<7> opcod, dag oops, dag iops, InstrItinClass itin,
             string opc, string asm, list<dag> pattern>
  : AsMul1I<opcod, oops, iops, itin, opc, asm, pattern> {
  bits<4> RdLo;
  bits<4> RdHi;
  bits<4> Rm;
  bits<4> Rn;
  let Inst{19-16} = RdHi;
  let Inst{15-12} = RdLo;
  let Inst{11-8}  = Rm;
  let Inst{3-0}   = Rn;
```
- EN: Declares reusable TableGen class `AsMla1I64` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AsMla1I64`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4455-4455
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4457-4469
```tablegen
// FIXME: The v5 pseudos are only necessary for the additional Constraint
//        property. Remove them when it's possible to add those properties
//        on an individual MachineInstr, not just an instruction description.
let isCommutable = 1, TwoOperandAliasConstraint = "$Rn = $Rd" in {
def MUL : AsMul1I32<0b0000000, (outs GPRnopc:$Rd),
                    (ins GPRnopc:$Rn, GPRnopc:$Rm),
                    IIC_iMUL32, "mul", "\t$Rd, $Rn, $Rm",
                  [(set GPRnopc:$Rd, (mul GPRnopc:$Rn, GPRnopc:$Rm))]>,
                  Requires<[IsARM, HasV6]>,
         Sched<[WriteMUL32, ReadMUL, ReadMUL]> {
  let Inst{15-12} = 0b0000;
  let Unpredictable{15-12} = 0b1111;
}
```
- EN: Defines TableGen record `MUL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MUL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4471-4479
```tablegen
let Constraints = "@earlyclobber $Rd" in
def MULv5: ARMPseudoExpand<(outs GPRnopc:$Rd), (ins GPRnopc:$Rn, GPRnopc:$Rm,
                                                    pred:$p, cc_out:$s),
                           4, IIC_iMUL32,
               [(set GPRnopc:$Rd, (mul GPRnopc:$Rn, GPRnopc:$Rm))],
               (MUL GPRnopc:$Rd, GPRnopc:$Rn, GPRnopc:$Rm, pred:$p, cc_out:$s)>,
               Requires<[IsARM, NoV6, UseMulOps]>,
           Sched<[WriteMUL32, ReadMUL, ReadMUL]>;
}
```
- EN: Defines TableGen record `MULv5:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MULv5:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4481-4489
```tablegen
def MLA  : AsMul1I32<0b0000001, (outs GPRnopc:$Rd),
                     (ins GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$Ra),
                     IIC_iMAC32, "mla", "\t$Rd, $Rn, $Rm, $Ra",
        [(set GPRnopc:$Rd, (add (mul GPRnopc:$Rn, GPRnopc:$Rm), GPRnopc:$Ra))]>,
                     Requires<[IsARM, HasV6, UseMulOps]>,
        Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]> {
  bits<4> Ra;
  let Inst{15-12} = Ra;
}
```
- EN: Defines TableGen record `MLA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MLA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4491-4498
```tablegen
let Constraints = "@earlyclobber $Rd" in
def MLAv5: ARMPseudoExpand<(outs GPRnopc:$Rd),
                           (ins GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$Ra,
                            pred:$p, cc_out:$s), 4, IIC_iMAC32,
         [(set GPRnopc:$Rd, (add (mul GPRnopc:$Rn, GPRnopc:$Rm), GPRnopc:$Ra))],
  (MLA GPRnopc:$Rd, GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$Ra, pred:$p, cc_out:$s)>,
                           Requires<[IsARM, NoV6]>,
           Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Defines TableGen record `MLAv5:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MLAv5:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4500-4513
```tablegen
def MLS  : AMul1I<0b0000011, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm, GPR:$Ra),
                   IIC_iMAC32, "mls", "\t$Rd, $Rn, $Rm, $Ra",
                   [(set GPR:$Rd, (sub GPR:$Ra, (mul GPR:$Rn, GPR:$Rm)))]>,
                   Requires<[IsARM, HasV6T2, UseMulOps]>,
          Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]> {
  bits<4> Rd;
  bits<4> Rm;
  bits<4> Rn;
  bits<4> Ra;
  let Inst{19-16} = Rd;
  let Inst{15-12} = Ra;
  let Inst{11-8}  = Rm;
  let Inst{3-0}   = Rn;
}
```
- EN: Defines TableGen record `MLS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MLS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4515-4524
```tablegen
// Extra precision multiplies with low / high results
let hasSideEffects = 0 in {
let isCommutable = 1 in {
def SMULL : AsMul1I64<0b0000110, (outs GPR:$RdLo, GPR:$RdHi),
                                 (ins GPR:$Rn, GPR:$Rm), IIC_iMUL64,
                    "smull", "\t$RdLo, $RdHi, $Rn, $Rm",
                    [(set GPR:$RdLo, GPR:$RdHi,
                          (smullohi GPR:$Rn, GPR:$Rm))]>,
                    Requires<[IsARM, HasV6]>,
           Sched<[WriteMUL64Lo, WriteMUL64Hi, ReadMUL, ReadMUL]>;
```
- EN: Defines TableGen record `SMULL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMULL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4526-4532
```tablegen
def UMULL : AsMul1I64<0b0000100, (outs GPR:$RdLo, GPR:$RdHi),
                                 (ins GPR:$Rn, GPR:$Rm), IIC_iMUL64,
                    "umull", "\t$RdLo, $RdHi, $Rn, $Rm",
                    [(set GPR:$RdLo, GPR:$RdHi,
                          (umullohi GPR:$Rn, GPR:$Rm))]>,
                    Requires<[IsARM, HasV6]>,
           Sched<[WriteMAC64Lo, WriteMAC64Hi, ReadMUL, ReadMUL]>;
```
- EN: Defines TableGen record `UMULL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UMULL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4534-4542
```tablegen
let Constraints = "@earlyclobber $RdLo,@earlyclobber $RdHi" in {
def SMULLv5 : ARMPseudoExpand<(outs GPR:$RdLo, GPR:$RdHi),
                            (ins GPR:$Rn, GPR:$Rm, pred:$p, cc_out:$s),
                            4, IIC_iMUL64,
                            [(set GPR:$RdLo, GPR:$RdHi,
                                  (smullohi GPR:$Rn, GPR:$Rm))],
          (SMULL GPR:$RdLo, GPR:$RdHi, GPR:$Rn, GPR:$Rm, pred:$p, cc_out:$s)>,
                           Requires<[IsARM, NoV6]>,
              Sched<[WriteMUL64Lo, WriteMUL64Hi, ReadMUL, ReadMUL]>;
```
- EN: Defines TableGen record `SMULLv5` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMULLv5`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4544-4553
```tablegen
def UMULLv5 : ARMPseudoExpand<(outs GPR:$RdLo, GPR:$RdHi),
                            (ins GPR:$Rn, GPR:$Rm, pred:$p, cc_out:$s),
                            4, IIC_iMUL64,
                            [(set GPR:$RdLo, GPR:$RdHi,
                                  (umullohi GPR:$Rn, GPR:$Rm))],
          (UMULL GPR:$RdLo, GPR:$RdHi, GPR:$Rn, GPR:$Rm, pred:$p, cc_out:$s)>,
                           Requires<[IsARM, NoV6]>,
             Sched<[WriteMUL64Lo, WriteMUL64Hi, ReadMUL, ReadMUL]>;
}
}
```
- EN: Defines TableGen record `UMULLv5` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UMULLv5`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4555-4565
```tablegen
// Multiply + accumulate
def SMLAL : AsMla1I64<0b0000111, (outs GPR:$RdLo, GPR:$RdHi),
                        (ins GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi), IIC_iMAC64,
                    "smlal", "\t$RdLo, $RdHi, $Rn, $Rm", []>,
         RegConstraint<"$RLo = $RdLo, $RHi = $RdHi">, Requires<[IsARM, HasV6]>,
           Sched<[WriteMAC64Lo, WriteMAC64Hi, ReadMUL, ReadMUL, ReadMAC, ReadMAC]>;
def UMLAL : AsMla1I64<0b0000101, (outs GPR:$RdLo, GPR:$RdHi),
                        (ins GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi), IIC_iMAC64,
                    "umlal", "\t$RdLo, $RdHi, $Rn, $Rm", []>,
         RegConstraint<"$RLo = $RdLo, $RHi = $RdHi">, Requires<[IsARM, HasV6]>,
            Sched<[WriteMAC64Lo, WriteMAC64Hi, ReadMUL, ReadMUL, ReadMAC, ReadMAC]>;
```
- EN: Defines TableGen record `SMLAL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMLAL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4567-4581
```tablegen
def UMAAL : AMul1I <0b0000010, (outs GPR:$RdLo, GPR:$RdHi),
                               (ins GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi),
                               IIC_iMAC64,
                    "umaal", "\t$RdLo, $RdHi, $Rn, $Rm", []>,
         RegConstraint<"$RLo = $RdLo, $RHi = $RdHi">, Requires<[IsARM, HasV6]>,
            Sched<[WriteMAC64Lo, WriteMAC64Hi, ReadMUL, ReadMUL, ReadMAC, ReadMAC]> {
  bits<4> RdLo;
  bits<4> RdHi;
  bits<4> Rm;
  bits<4> Rn;
  let Inst{19-16} = RdHi;
  let Inst{15-12} = RdLo;
  let Inst{11-8}  = Rm;
  let Inst{3-0}   = Rn;
}
```
- EN: Defines TableGen record `UMAAL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UMAAL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4583-4599
```tablegen
let Constraints =
    "@earlyclobber $RdLo,@earlyclobber $RdHi,$RLo = $RdLo,$RHi = $RdHi" in {
def SMLALv5 : ARMPseudoExpand<(outs GPR:$RdLo, GPR:$RdHi),
                (ins GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi, pred:$p, cc_out:$s),
                              4, IIC_iMAC64, [],
             (SMLAL GPR:$RdLo, GPR:$RdHi, GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi,
                           pred:$p, cc_out:$s)>,
                           Requires<[IsARM, NoV6]>,
              Sched<[WriteMAC64Lo, WriteMAC64Hi, ReadMUL, ReadMUL, ReadMAC, ReadMAC]>;
def UMLALv5 : ARMPseudoExpand<(outs GPR:$RdLo, GPR:$RdHi),
                (ins GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi, pred:$p, cc_out:$s),
                              4, IIC_iMAC64, [],
             (UMLAL GPR:$RdLo, GPR:$RdHi, GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi,
                           pred:$p, cc_out:$s)>,
                           Requires<[IsARM, NoV6]>,
              Sched<[WriteMAC64Lo, WriteMAC64Hi, ReadMUL, ReadMUL, ReadMAC, ReadMAC]>;
}
```
- EN: Defines TableGen record `SMLALv5` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMLALv5`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4601-4601
```tablegen
} // hasSideEffects
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4603-4610
```tablegen
// Most significant word multiply
def SMMUL : AMul2I <0b0111010, 0b0001, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
               IIC_iMUL32, "smmul", "\t$Rd, $Rn, $Rm",
               [(set GPR:$Rd, (mulhs GPR:$Rn, GPR:$Rm))]>,
            Requires<[IsARM, HasV6]>,
            Sched<[WriteMUL32, ReadMUL, ReadMUL]> {
  let Inst{15-12} = 0b1111;
}
```
- EN: Defines TableGen record `SMMUL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMMUL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4612-4618
```tablegen
def SMMULR : AMul2I <0b0111010, 0b0011, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
               IIC_iMUL32, "smmulr", "\t$Rd, $Rn, $Rm",
               [(set GPR:$Rd, (ARMsmmlar GPR:$Rn, GPR:$Rm, (i32 0)))]>,
            Requires<[IsARM, HasV6]>,
             Sched<[WriteMUL32, ReadMUL, ReadMUL]>  {
  let Inst{15-12} = 0b1111;
}
```
- EN: Defines TableGen record `SMMULR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMMULR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4620-4625
```tablegen
def SMMLA : AMul2Ia <0b0111010, 0b0001, (outs GPR:$Rd),
               (ins GPR:$Rn, GPR:$Rm, GPR:$Ra),
               IIC_iMAC32, "smmla", "\t$Rd, $Rn, $Rm, $Ra",
               [(set GPR:$Rd, (add (mulhs GPR:$Rn, GPR:$Rm), GPR:$Ra))]>,
            Requires<[IsARM, HasV6, UseMulOps]>,
            Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Defines TableGen record `SMMLA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMMLA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4627-4632
```tablegen
def SMMLAR : AMul2Ia <0b0111010, 0b0011, (outs GPR:$Rd),
               (ins GPR:$Rn, GPR:$Rm, GPR:$Ra),
               IIC_iMAC32, "smmlar", "\t$Rd, $Rn, $Rm, $Ra",
               [(set GPR:$Rd, (ARMsmmlar GPR:$Rn, GPR:$Rm, GPR:$Ra))]>,
            Requires<[IsARM, HasV6]>,
             Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Defines TableGen record `SMMLAR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMMLAR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4634-4638
```tablegen
def SMMLS : AMul2Ia <0b0111010, 0b1101, (outs GPR:$Rd),
               (ins GPR:$Rn, GPR:$Rm, GPR:$Ra),
               IIC_iMAC32, "smmls", "\t$Rd, $Rn, $Rm, $Ra", []>,
            Requires<[IsARM, HasV6, UseMulOps]>,
            Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Defines TableGen record `SMMLS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMMLS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4640-4645
```tablegen
def SMMLSR : AMul2Ia <0b0111010, 0b1111, (outs GPR:$Rd),
               (ins GPR:$Rn, GPR:$Rm, GPR:$Ra),
               IIC_iMAC32, "smmlsr", "\t$Rd, $Rn, $Rm, $Ra",
               [(set GPR:$Rd, (ARMsmmlsr GPR:$Rn, GPR:$Rm, GPR:$Ra))]>,
            Requires<[IsARM, HasV6]>,
             Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Defines TableGen record `SMMLSR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMMLSR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4647-4652
```tablegen
multiclass AI_smul<string opc> {
  def BB : AMulxyI<0b0001011, 0b00, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
              IIC_iMUL16, !strconcat(opc, "bb"), "\t$Rd, $Rn, $Rm",
              [(set GPR:$Rd, (bb_mul GPR:$Rn, GPR:$Rm))]>,
           Requires<[IsARM, HasV5TE]>,
           Sched<[WriteMUL16, ReadMUL, ReadMUL]>;
```
- EN: Declares TableGen `multiclass AI_smul`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI_smul`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4654-4658
```tablegen
  def BT : AMulxyI<0b0001011, 0b10, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
              IIC_iMUL16, !strconcat(opc, "bt"), "\t$Rd, $Rn, $Rm",
              [(set GPR:$Rd, (bt_mul GPR:$Rn, GPR:$Rm))]>,
           Requires<[IsARM, HasV5TE]>,
           Sched<[WriteMUL16, ReadMUL, ReadMUL]>;
```
- EN: Defines TableGen record `BT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4660-4664
```tablegen
  def TB : AMulxyI<0b0001011, 0b01, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
              IIC_iMUL16, !strconcat(opc, "tb"), "\t$Rd, $Rn, $Rm",
              [(set GPR:$Rd, (tb_mul GPR:$Rn, GPR:$Rm))]>,
           Requires<[IsARM, HasV5TE]>,
           Sched<[WriteMUL16, ReadMUL, ReadMUL]>;
```
- EN: Defines TableGen record `TB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4666-4670
```tablegen
  def TT : AMulxyI<0b0001011, 0b11, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
              IIC_iMUL16, !strconcat(opc, "tt"), "\t$Rd, $Rn, $Rm",
              [(set GPR:$Rd, (tt_mul GPR:$Rn, GPR:$Rm))]>,
            Requires<[IsARM, HasV5TE]>,
           Sched<[WriteMUL16, ReadMUL, ReadMUL]>;
```
- EN: Defines TableGen record `TT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4672-4676
```tablegen
  def WB : AMulxyI<0b0001001, 0b01, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
              IIC_iMUL16, !strconcat(opc, "wb"), "\t$Rd, $Rn, $Rm",
              [(set GPR:$Rd, (ARMsmulwb GPR:$Rn, GPR:$Rm))]>,
           Requires<[IsARM, HasV5TE]>,
           Sched<[WriteMUL16, ReadMUL, ReadMUL]>;
```
- EN: Defines TableGen record `WB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4678-4683
```tablegen
  def WT : AMulxyI<0b0001001, 0b11, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm),
              IIC_iMUL16, !strconcat(opc, "wt"), "\t$Rd, $Rn, $Rm",
              [(set GPR:$Rd, (ARMsmulwt GPR:$Rn, GPR:$Rm))]>,
            Requires<[IsARM, HasV5TE]>,
           Sched<[WriteMUL16, ReadMUL, ReadMUL]>;
}
```
- EN: Defines TableGen record `WT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4686-4694
```tablegen
multiclass AI_smla<string opc> {
  let DecoderMethod = "DecodeSMLAInstruction" in {
  def BB : AMulxyIa<0b0001000, 0b00, (outs GPRnopc:$Rd),
              (ins GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
              IIC_iMAC16, !strconcat(opc, "bb"), "\t$Rd, $Rn, $Rm, $Ra",
              [(set GPRnopc:$Rd, (add GPR:$Ra,
                                      (bb_mul GPRnopc:$Rn, GPRnopc:$Rm)))]>,
           Requires<[IsARM, HasV5TE, UseMulOps]>,
           Sched<[WriteMAC16, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Declares TableGen `multiclass AI_smla`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI_smla`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4696-4702
```tablegen
  def BT : AMulxyIa<0b0001000, 0b10, (outs GPRnopc:$Rd),
              (ins GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
              IIC_iMAC16, !strconcat(opc, "bt"), "\t$Rd, $Rn, $Rm, $Ra",
              [(set GPRnopc:$Rd, (add GPR:$Ra,
                                      (bt_mul GPRnopc:$Rn, GPRnopc:$Rm)))]>,
           Requires<[IsARM, HasV5TE, UseMulOps]>,
           Sched<[WriteMAC16, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Defines TableGen record `BT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4704-4710
```tablegen
  def TB : AMulxyIa<0b0001000, 0b01, (outs GPRnopc:$Rd),
              (ins GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
              IIC_iMAC16, !strconcat(opc, "tb"), "\t$Rd, $Rn, $Rm, $Ra",
              [(set GPRnopc:$Rd, (add GPR:$Ra,
                                      (tb_mul GPRnopc:$Rn, GPRnopc:$Rm)))]>,
           Requires<[IsARM, HasV5TE, UseMulOps]>,
           Sched<[WriteMAC16, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Defines TableGen record `TB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4712-4718
```tablegen
  def TT : AMulxyIa<0b0001000, 0b11, (outs GPRnopc:$Rd),
              (ins GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
              IIC_iMAC16, !strconcat(opc, "tt"), "\t$Rd, $Rn, $Rm, $Ra",
             [(set GPRnopc:$Rd, (add GPR:$Ra,
                                     (tt_mul GPRnopc:$Rn, GPRnopc:$Rm)))]>,
            Requires<[IsARM, HasV5TE, UseMulOps]>,
            Sched<[WriteMAC16, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Defines TableGen record `TT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4720-4726
```tablegen
  def WB : AMulxyIa<0b0001001, 0b00, (outs GPRnopc:$Rd),
              (ins GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
              IIC_iMAC16, !strconcat(opc, "wb"), "\t$Rd, $Rn, $Rm, $Ra",
              [(set GPRnopc:$Rd,
                    (add GPR:$Ra, (ARMsmulwb GPRnopc:$Rn, GPRnopc:$Rm)))]>,
           Requires<[IsARM, HasV5TE, UseMulOps]>,
           Sched<[WriteMAC16, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Defines TableGen record `WB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4728-4736
```tablegen
  def WT : AMulxyIa<0b0001001, 0b10, (outs GPRnopc:$Rd),
              (ins GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
              IIC_iMAC16, !strconcat(opc, "wt"), "\t$Rd, $Rn, $Rm, $Ra",
              [(set GPRnopc:$Rd,
                    (add GPR:$Ra, (ARMsmulwt GPRnopc:$Rn, GPRnopc:$Rm)))]>,
            Requires<[IsARM, HasV5TE, UseMulOps]>,
            Sched<[WriteMAC16, ReadMUL, ReadMUL, ReadMAC]>;
  }
}
```
- EN: Defines TableGen record `WT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4738-4739
```tablegen
defm SMUL : AI_smul<"smul">;
defm SMLA : AI_smla<"smla">;
```
- EN: Defines TableGen record `SMUL` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMUL`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4741-4749
```tablegen
// Halfword multiply accumulate long: SMLAL<x><y>.
class SMLAL<bits<2> opc1, string asm>
 : AMulxyI64<0b0001010, opc1,
        (outs GPRnopc:$RdLo, GPRnopc:$RdHi),
        (ins GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$RLo, GPRnopc:$RHi),
        IIC_iMAC64, asm, "\t$RdLo, $RdHi, $Rn, $Rm", []>,
        RegConstraint<"$RLo = $RdLo, $RHi = $RdHi">,
        Requires<[IsARM, HasV5TE]>,
        Sched<[WriteMAC64Lo, WriteMAC64Hi, ReadMUL, ReadMUL, ReadMAC, ReadMAC]>;
```
- EN: Declares reusable TableGen class `SMLAL` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `SMLAL`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4751-4754
```tablegen
def SMLALBB : SMLAL<0b00, "smlalbb">;
def SMLALBT : SMLAL<0b10, "smlalbt">;
def SMLALTB : SMLAL<0b01, "smlaltb">;
def SMLALTT : SMLAL<0b11, "smlaltt">;
```
- EN: Defines TableGen record `SMLALBB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMLALBB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4756-4763
```tablegen
def : ARMV5TEPat<(ARMsmlalbb GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi),
                 (SMLALBB $Rn, $Rm, $RLo, $RHi)>;
def : ARMV5TEPat<(ARMsmlalbt GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi),
                 (SMLALBT $Rn, $Rm, $RLo, $RHi)>;
def : ARMV5TEPat<(ARMsmlaltb GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi),
                 (SMLALTB $Rn, $Rm, $RLo, $RHi)>;
def : ARMV5TEPat<(ARMsmlaltt GPR:$Rn, GPR:$Rm, GPR:$RLo, GPR:$RHi),
                 (SMLALTT $Rn, $Rm, $RLo, $RHi)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4765-4782
```tablegen
// Helper class for AI_smld.
class AMulDualIbase<bit long, bit sub, bit swap, dag oops, dag iops,
                    InstrItinClass itin, string opc, string asm>
  : AI<oops, iops, MulFrm, itin, opc, asm, []>,
       Requires<[IsARM, HasV6]> {
  bits<4> Rn;
  bits<4> Rm;
  let Inst{27-23} = 0b01110;
  let Inst{22}    = long;
  let Inst{21-20} = 0b00;
  let Inst{11-8}  = Rm;
  let Inst{7}     = 0;
  let Inst{6}     = sub;
  let Inst{5}     = swap;
  let Inst{4}     = 1;
  let Inst{3-0}   = Rn;
}
class AMulDualI<bit long, bit sub, bit swap, dag oops, dag iops,
```
- EN: Declares reusable TableGen class `for` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `for`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4783-4800
```tablegen
                InstrItinClass itin, string opc, string asm>
  : AMulDualIbase<long, sub, swap, oops, iops, itin, opc, asm> {
  bits<4> Rd;
  let Inst{15-12} = 0b1111;
  let Inst{19-16} = Rd;
}
class AMulDualIa<bit long, bit sub, bit swap, dag oops, dag iops,
                InstrItinClass itin, string opc, string asm>
  : AMulDualIbase<long, sub, swap, oops, iops, itin, opc, asm> {
  bits<4> Ra;
  bits<4> Rd;
  let Inst{19-16} = Rd;
  let Inst{15-12} = Ra;
}
class AMulDualI64<bit long, bit sub, bit swap, dag oops, dag iops,
                  InstrItinClass itin, string opc, string asm>
  : AMulDualIbase<long, sub, swap, oops, iops, itin, opc, asm> {
  bits<4> RdLo;
```
- EN: Declares reusable TableGen class `AMulDualIa` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AMulDualIa`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4801-4804
```tablegen
  bits<4> RdHi;
  let Inst{19-16} = RdHi;
  let Inst{15-12} = RdLo;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 4806-4806
```tablegen
multiclass AI_smld<bit sub, string opc> {
```
- EN: Declares TableGen `multiclass AI_smld`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI_smld`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4808-4811
```tablegen
  def D : AMulDualIa<0, sub, 0, (outs GPRnopc:$Rd),
                  (ins GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
                  NoItinerary, !strconcat(opc, "d"), "\t$Rd, $Rn, $Rm, $Ra">,
          Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Defines TableGen record `D` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `D`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4813-4816
```tablegen
  def DX: AMulDualIa<0, sub, 1, (outs GPRnopc:$Rd),
                  (ins GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
                  NoItinerary, !strconcat(opc, "dx"), "\t$Rd, $Rn, $Rm, $Ra">,
          Sched<[WriteMAC32, ReadMUL, ReadMUL, ReadMAC]>;
```
- EN: Defines TableGen record `DX:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DX:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4818-4823
```tablegen
  def LD: AMulDualI64<1, sub, 0, (outs GPRnopc:$RdLo, GPRnopc:$RdHi),
                  (ins GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$RLo, GPRnopc:$RHi),
                  NoItinerary,
                  !strconcat(opc, "ld"), "\t$RdLo, $RdHi, $Rn, $Rm">,
                  RegConstraint<"$RLo = $RdLo, $RHi = $RdHi">,
          Sched<[WriteMAC64Lo, WriteMAC64Hi, ReadMUL, ReadMUL, ReadMAC, ReadMAC]>;
```
- EN: Defines TableGen record `LD:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LD:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4825-4831
```tablegen
  def LDX : AMulDualI64<1, sub, 1, (outs GPRnopc:$RdLo, GPRnopc:$RdHi),
                  (ins GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$RLo, GPRnopc:$RHi),
                  NoItinerary,
                  !strconcat(opc, "ldx"),"\t$RdLo, $RdHi, $Rn, $Rm">,
                  RegConstraint<"$RLo = $RdLo, $RHi = $RdHi">,
             Sched<[WriteMUL64Lo, WriteMUL64Hi, ReadMUL, ReadMUL]>;
}
```
- EN: Defines TableGen record `LDX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4833-4834
```tablegen
defm SMLA : AI_smld<0, "smla">;
defm SMLS : AI_smld<1, "smls">;
```
- EN: Defines TableGen record `SMLA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMLA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4836-4851
```tablegen
def : ARMV6Pat<(int_arm_smlad GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
               (SMLAD GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$Ra)>;
def : ARMV6Pat<(int_arm_smladx GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
               (SMLADX GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$Ra)>;
def : ARMV6Pat<(int_arm_smlsd GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
               (SMLSD GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$Ra)>;
def : ARMV6Pat<(int_arm_smlsdx GPRnopc:$Rn, GPRnopc:$Rm, GPR:$Ra),
               (SMLSDX GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$Ra)>;
def : ARMV6Pat<(ARMSmlald GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$RLo, GPRnopc:$RHi),
               (SMLALD GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$RLo, GPRnopc:$RHi)>;
def : ARMV6Pat<(ARMSmlaldx GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$RLo, GPRnopc:$RHi),
               (SMLALDX GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$RLo, GPRnopc:$RHi)>;
def : ARMV6Pat<(ARMSmlsld GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$RLo, GPRnopc:$RHi),
               (SMLSLD GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$RLo, GPRnopc:$RHi)>;
def : ARMV6Pat<(ARMSmlsldx GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$RLo, GPRnopc:$RHi),
               (SMLSLDX GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$RLo, GPRnopc:$RHi)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4853-4853
```tablegen
multiclass AI_sdml<bit sub, string opc> {
```
- EN: Declares TableGen `multiclass AI_sdml`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass AI_sdml`，它是一个可复用模板，可展开为多个相关记录。

### Lines 4855-4861
```tablegen
  def D:AMulDualI<0, sub, 0, (outs GPRnopc:$Rd), (ins GPRnopc:$Rn, GPRnopc:$Rm),
                  NoItinerary, !strconcat(opc, "d"), "\t$Rd, $Rn, $Rm">,
        Sched<[WriteMUL32, ReadMUL, ReadMUL]>;
  def DX:AMulDualI<0, sub, 1, (outs GPRnopc:$Rd),(ins GPRnopc:$Rn, GPRnopc:$Rm),
                  NoItinerary, !strconcat(opc, "dx"), "\t$Rd, $Rn, $Rm">,
         Sched<[WriteMUL32, ReadMUL, ReadMUL]>;
}
```
- EN: Defines TableGen record `D:AMulDualI` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `D:AMulDualI`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4863-4864
```tablegen
defm SMUA : AI_sdml<0, "smua">;
defm SMUS : AI_sdml<1, "smus">;
```
- EN: Defines TableGen record `SMUA` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SMUA`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4866-4873
```tablegen
def : ARMV6Pat<(int_arm_smuad GPRnopc:$Rn, GPRnopc:$Rm),
               (SMUAD GPRnopc:$Rn, GPRnopc:$Rm)>;
def : ARMV6Pat<(int_arm_smuadx GPRnopc:$Rn, GPRnopc:$Rm),
               (SMUADX GPRnopc:$Rn, GPRnopc:$Rm)>;
def : ARMV6Pat<(int_arm_smusd GPRnopc:$Rn, GPRnopc:$Rm),
               (SMUSD GPRnopc:$Rn, GPRnopc:$Rm)>;
def : ARMV6Pat<(int_arm_smusdx GPRnopc:$Rn, GPRnopc:$Rm),
               (SMUSDX GPRnopc:$Rn, GPRnopc:$Rm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4875-4883
```tablegen
//===----------------------------------------------------------------------===//
//  Division Instructions (ARMv7-A with virtualization extension)
//
let TwoOperandAliasConstraint = "$Rn = $Rd" in {
def SDIV : ADivA1I<0b001, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm), IIC_iDIV,
                   "sdiv", "\t$Rd, $Rn, $Rm",
                   [(set GPR:$Rd, (sdiv GPR:$Rn, GPR:$Rm))]>,
           Requires<[IsARM, HasDivideInARM]>,
           Sched<[WriteDIV]>;
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4885-4890
```tablegen
def UDIV : ADivA1I<0b011, (outs GPR:$Rd), (ins GPR:$Rn, GPR:$Rm), IIC_iDIV,
                   "udiv", "\t$Rd, $Rn, $Rm",
                   [(set GPR:$Rd, (udiv GPR:$Rn, GPR:$Rm))]>,
           Requires<[IsARM, HasDivideInARM]>,
           Sched<[WriteDIV]>;
}
```
- EN: Defines TableGen record `UDIV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `UDIV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4892-4894
```tablegen
//===----------------------------------------------------------------------===//
//  Misc. Arithmetic Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4896-4899
```tablegen
def CLZ  : AMiscA1I<0b00010110, 0b0001, (outs GPR:$Rd), (ins GPR:$Rm),
              IIC_iUNAr, "clz", "\t$Rd, $Rm",
              [(set GPR:$Rd, (ctlz GPR:$Rm))]>, Requires<[IsARM, HasV5T]>,
           Sched<[WriteALU]>;
```
- EN: Defines TableGen record `CLZ` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CLZ`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4901-4905
```tablegen
def RBIT : AMiscA1I<0b01101111, 0b0011, (outs GPR:$Rd), (ins GPR:$Rm),
              IIC_iUNAr, "rbit", "\t$Rd, $Rm",
              [(set GPR:$Rd, (bitreverse GPR:$Rm))]>,
           Requires<[IsARM, HasV6T2]>,
           Sched<[WriteALU]>;
```
- EN: Defines TableGen record `RBIT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RBIT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4907-4910
```tablegen
def REV  : AMiscA1I<0b01101011, 0b0011, (outs GPR:$Rd), (ins GPR:$Rm),
              IIC_iUNAr, "rev", "\t$Rd, $Rm",
              [(set GPR:$Rd, (bswap GPR:$Rm))]>, Requires<[IsARM, HasV6]>,
           Sched<[WriteALU]>;
```
- EN: Defines TableGen record `REV` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `REV`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4912-4917
```tablegen
let AddedComplexity = 5 in
def REV16 : AMiscA1I<0b01101011, 0b1011, (outs GPR:$Rd), (ins GPR:$Rm),
               IIC_iUNAr, "rev16", "\t$Rd, $Rm",
               [(set GPR:$Rd, (rotr (bswap GPR:$Rm), (i32 16)))]>,
               Requires<[IsARM, HasV6]>,
           Sched<[WriteALU]>;
```
- EN: Defines TableGen record `REV16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `REV16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4919-4924
```tablegen
def : ARMV6Pat<(srl (bswap (extloadi16 addrmode3:$addr)), (i32 16)),
              (REV16 (LDRH addrmode3:$addr))>;
def : ARMV6Pat<(truncstorei16 (srl (bswap GPR:$Rn), (i32 16)), addrmode3:$addr),
               (STRH (REV16 GPR:$Rn), addrmode3:$addr)>;
def : ARMV6Pat<(srl (bswap top16Zero:$Rn), (i32 16)),
               (REV16 GPR:$Rn)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4926-4931
```tablegen
let AddedComplexity = 5 in
def REVSH : AMiscA1I<0b01101111, 0b1011, (outs GPR:$Rd), (ins GPR:$Rm),
               IIC_iUNAr, "revsh", "\t$Rd, $Rm",
               [(set GPR:$Rd, (sra (bswap GPR:$Rm), (i32 16)))]>,
               Requires<[IsARM, HasV6]>,
           Sched<[WriteALU]>;
```
- EN: Defines TableGen record `REVSH` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `REVSH`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4933-4935
```tablegen
def : ARMV6Pat<(or (sra (shl GPR:$Rm, (i32 24)), (i32 16)),
                   (and (srl GPR:$Rm, (i32 8)), 0xFF)),
               (REVSH GPR:$Rm)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4937-4944
```tablegen
def PKHBT : APKHI<0b01101000, 0, (outs GPRnopc:$Rd),
                              (ins GPRnopc:$Rn, GPRnopc:$Rm, pkh_lsl_amt:$sh),
               IIC_iALUsi, "pkhbt", "\t$Rd, $Rn, $Rm$sh",
               [(set GPRnopc:$Rd, (or (and GPRnopc:$Rn, 0xFFFF),
                                      (and (shl GPRnopc:$Rm, pkh_lsl_amt:$sh),
                                           0xFFFF0000)))]>,
               Requires<[IsARM, HasV6]>,
           Sched<[WriteALUsi, ReadALU]>;
```
- EN: Defines TableGen record `PKHBT` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PKHBT`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4946-4950
```tablegen
// Alternate cases for PKHBT where identities eliminate some nodes.
def : ARMV6Pat<(or (and GPRnopc:$Rn, 0xFFFF), (and GPRnopc:$Rm, 0xFFFF0000)),
               (PKHBT GPRnopc:$Rn, GPRnopc:$Rm, 0)>;
def : ARMV6Pat<(or (and GPRnopc:$Rn, 0xFFFF), (shl GPRnopc:$Rm, imm16_31:$sh)),
               (PKHBT GPRnopc:$Rn, GPRnopc:$Rm, imm16_31:$sh)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4952-4961
```tablegen
// Note: Shifts of 1-15 bits will be transformed to srl instead of sra and
// will match the pattern below.
def PKHTB : APKHI<0b01101000, 1, (outs GPRnopc:$Rd),
                              (ins GPRnopc:$Rn, GPRnopc:$Rm, pkh_asr_amt:$sh),
               IIC_iBITsi, "pkhtb", "\t$Rd, $Rn, $Rm$sh",
               [(set GPRnopc:$Rd, (or (and GPRnopc:$Rn, 0xFFFF0000),
                                      (and (sra GPRnopc:$Rm, pkh_asr_amt:$sh),
                                           0xFFFF)))]>,
               Requires<[IsARM, HasV6]>,
           Sched<[WriteALUsi, ReadALU]>;
```
- EN: Defines TableGen record `PKHTB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `PKHTB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 4963-4975
```tablegen
// Alternate cases for PKHTB where identities eliminate some nodes.  Note that
// a shift amount of 0 is *not legal* here, it is PKHBT instead.
// We also can not replace a srl (17..31) by an arithmetic shift we would use in
// pkhtb src1, src2, asr (17..31).
def : ARMV6Pat<(or (and GPRnopc:$src1, 0xFFFF0000),
                   (srl GPRnopc:$src2, imm16:$sh)),
               (PKHTB GPRnopc:$src1, GPRnopc:$src2, imm16:$sh)>;
def : ARMV6Pat<(or (and GPRnopc:$src1, 0xFFFF0000),
                   (sra GPRnopc:$src2, imm16_31:$sh)),
               (PKHTB GPRnopc:$src1, GPRnopc:$src2, imm16_31:$sh)>;
def : ARMV6Pat<(or (and GPRnopc:$src1, 0xFFFF0000),
                   (and (srl GPRnopc:$src2, imm1_15:$sh), 0xFFFF)),
               (PKHTB GPRnopc:$src1, GPRnopc:$src2, imm1_15:$sh)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 4977-4983
```tablegen
//===----------------------------------------------------------------------===//
// CRC Instructions
//
// Polynomials:
// + CRC32{B,H,W}       0x04C11DB7
// + CRC32C{B,H,W}      0x1EDC6F41
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 4985-4992
```tablegen
class AI_crc32<bit C, bits<2> sz, string suffix, SDPatternOperator builtin>
  : AInoP<(outs GPRnopc:$Rd), (ins GPRnopc:$Rn, GPRnopc:$Rm), MiscFrm, NoItinerary,
               !strconcat("crc32", suffix), "\t$Rd, $Rn, $Rm",
               [(set GPRnopc:$Rd, (builtin GPRnopc:$Rn, GPRnopc:$Rm))]>,
               Requires<[IsARM, HasCRC]> {
  bits<4> Rd;
  bits<4> Rn;
  bits<4> Rm;
```
- EN: Declares reusable TableGen class `AI_crc32` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `AI_crc32`，通常用于抽象共享字段、谓词或编码结构。

### Lines 4994-5004
```tablegen
  let Inst{31-28} = 0b1110;
  let Inst{27-23} = 0b00010;
  let Inst{22-21} = sz;
  let Inst{20}    = 0;
  let Inst{19-16} = Rn;
  let Inst{15-12} = Rd;
  let Inst{11-10} = 0b00;
  let Inst{9}     = C;
  let Inst{8}     = 0;
  let Inst{7-4}   = 0b0100;
  let Inst{3-0}   = Rm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5006-5007
```tablegen
  let Unpredictable{11-8} = 0b1101;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5009-5014
```tablegen
def CRC32B  : AI_crc32<0, 0b00, "b", int_arm_crc32b>;
def CRC32CB : AI_crc32<1, 0b00, "cb", int_arm_crc32cb>;
def CRC32H  : AI_crc32<0, 0b01, "h", int_arm_crc32h>;
def CRC32CH : AI_crc32<1, 0b01, "ch", int_arm_crc32ch>;
def CRC32W  : AI_crc32<0, 0b10, "w", int_arm_crc32w>;
def CRC32CW : AI_crc32<1, 0b10, "cw", int_arm_crc32cw>;
```
- EN: Defines TableGen record `CRC32B` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CRC32B`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5016-5019
```tablegen
//===----------------------------------------------------------------------===//
// ARMv8.1a Privilege Access Never extension
//
// SETPAN #imm1
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 5021-5023
```tablegen
def SETPAN : AInoP<(outs), (ins imm0_1:$imm), MiscFrm, NoItinerary, "setpan",
                "\t$imm", []>, Requires<[IsARM, HasV8, HasV8_1a]> {
  bits<1> imm;
```
- EN: Defines TableGen record `SETPAN` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SETPAN`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5025-5032
```tablegen
  let Inst{31-28} = 0b1111;
  let Inst{27-20} = 0b00010001;
  let Inst{19-16} = 0b0000;
  let Inst{15-10} = 0b000000;
  let Inst{9} = imm;
  let Inst{8} = 0b0;
  let Inst{7-4} = 0b0000;
  let Inst{3-0} = 0b0000;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5034-5038
```tablegen
  let Unpredictable{19-16} = 0b1111;
  let Unpredictable{15-10} = 0b111111;
  let Unpredictable{8} = 0b1;
  let Unpredictable{3-0} = 0b1111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5040-5042
```tablegen
//===----------------------------------------------------------------------===//
//  Comparison Instructions...
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 5044-5045
```tablegen
defm CMP  : AI1_cmp_irs<0b1010, "cmp",
                        IIC_iCMPi, IIC_iCMPr, IIC_iCMPsr, ARMcmp>;
```
- EN: Defines TableGen record `CMP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5047-5062
```tablegen
// ARMcmpZ can re-use the above instruction definitions.
def : ARMPat<(ARMcmpZ GPR:$src, mod_imm:$imm),
             (CMPri   GPR:$src, mod_imm:$imm)>;
def : ARMPat<(ARMcmpZ GPR:$src, GPR:$rhs),
             (CMPrr   GPR:$src, GPR:$rhs)>;
def : ARMPat<(ARMcmpZ GPR:$src, so_reg_imm:$rhs),
             (CMPrsi   GPR:$src, so_reg_imm:$rhs)>;
def : ARMPat<(ARMcmpZ GPR:$src, so_reg_reg:$rhs),
             (CMPrsr   GPR:$src, so_reg_reg:$rhs)>;
// Following patterns aimed to prevent usage of CMPrsi and CMPrsr for a comparison
// with zero. Usage of CMPri in these cases helps to replace cmp with S-versions of
// shift instructions during peephole optimizations pass.
def : ARMPat<(ARMcmpZ so_reg_imm:$rhs, 0),
             (CMPri (MOVsi so_reg_imm:$rhs), 0)>;
def : ARMPat<(ARMcmpZ so_reg_reg:$rhs, 0),
             (CMPri (MOVsr so_reg_reg:$rhs), 0)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5064-5076
```tablegen
// CMN register-integer
let isCompare = 1, Defs = [CPSR] in {
def CMNri : AI1<0b1011, (outs), (ins GPR:$Rn, mod_imm:$imm), DPFrm, IIC_iCMPi,
                "cmn", "\t$Rn, $imm",
                [(set CPSR, (ARMcmn GPR:$Rn, mod_imm:$imm))]>,
                Sched<[WriteCMP, ReadALU]> {
  bits<4> Rn;
  bits<12> imm;
  let Inst{25} = 1;
  let Inst{20} = 1;
  let Inst{19-16} = Rn;
  let Inst{15-12} = 0b0000;
  let Inst{11-0} = imm;
```
- EN: Defines TableGen record `CMNri` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMNri`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5078-5079
```tablegen
  let Unpredictable{15-12} = 0b1111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5081-5094
```tablegen
// CMN register-register/shift
def CMNrr : AI1<0b1011, (outs), (ins GPR:$Rn, GPR:$Rm), DPFrm, IIC_iCMPr,
                 "cmn", "\t$Rn, $Rm",
                 [(set CPSR, (ARMcmn GPR:$Rn, GPR:$Rm))]>,
                 Sched<[WriteCMP, ReadALU, ReadALU]> {
  bits<4> Rn;
  bits<4> Rm;
  let isCommutable = 1;
  let Inst{25} = 0;
  let Inst{20} = 1;
  let Inst{19-16} = Rn;
  let Inst{15-12} = 0b0000;
  let Inst{11-4} = 0b00000000;
  let Inst{3-0} = Rm;
```
- EN: Defines TableGen record `CMNrr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMNrr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5096-5097
```tablegen
  let Unpredictable{15-12} = 0b1111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5099-5112
```tablegen
def CMNrsi : AI1<0b1011, (outs),
                  (ins GPR:$Rn, so_reg_imm:$shift), DPSoRegImmFrm, IIC_iCMPsr,
                  "cmn", "\t$Rn, $shift",
                  [(set CPSR, (ARMcmn GPR:$Rn, so_reg_imm:$shift))]>,
                  Sched<[WriteCMPsi, ReadALU]> {
  bits<4> Rn;
  bits<12> shift;
  let Inst{25} = 0;
  let Inst{20} = 1;
  let Inst{19-16} = Rn;
  let Inst{15-12} = 0b0000;
  let Inst{11-5} = shift{11-5};
  let Inst{4} = 0;
  let Inst{3-0} = shift{3-0};
```
- EN: Defines TableGen record `CMNrsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMNrsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5114-5115
```tablegen
  let Unpredictable{15-12} = 0b1111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5117-5132
```tablegen
def CMNrsr : AI1<0b1011, (outs),
                  (ins GPRnopc:$Rn, so_reg_reg:$shift), DPSoRegRegFrm, IIC_iCMPsr,
                  "cmn", "\t$Rn, $shift",
                  [(set CPSR, (ARMcmn GPRnopc:$Rn, so_reg_reg:$shift))]>,
                  Sched<[WriteCMPsr, ReadALU]> {
  bits<4> Rn;
  bits<12> shift;
  let Inst{25} = 0;
  let Inst{20} = 1;
  let Inst{19-16} = Rn;
  let Inst{15-12} = 0b0000;
  let Inst{11-8} = shift{11-8};
  let Inst{7} = 0;
  let Inst{6-5} = shift{6-5};
  let Inst{4} = 1;
  let Inst{3-0} = shift{3-0};
```
- EN: Defines TableGen record `CMNrsr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMNrsr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5134-5135
```tablegen
  let Unpredictable{15-12} = 0b1111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5137-5137
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5139-5149
```tablegen
// ARMcmpZ patterns for CMN (compare-to-zero with negated operands)
def : ARMPat<(ARMcmp  GPR:$src, mod_imm_neg:$imm),
             (CMNri   GPR:$src, mod_imm_neg:$imm)>;
def : ARMPat<(ARMcmpZ GPR:$src, mod_imm_neg:$imm),
             (CMNri   GPR:$src, mod_imm_neg:$imm)>;
def : ARMPat<(ARMcmpZ GPR:$src, (ineg GPR:$rhs)),
             (CMNrr GPR:$src, GPR:$rhs)>;
def : ARMPat<(ARMcmpZ GPR:$src, (ineg so_reg_imm:$rhs)),
             (CMNrsi GPR:$src, so_reg_imm:$rhs)>;
def : ARMPat<(ARMcmpZ GPRnopc:$src, (ineg so_reg_reg:$rhs)),
             (CMNrsr GPRnopc:$src, so_reg_reg:$rhs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5151-5158
```tablegen
// Note that TST/TEQ don't set all the same flags that CMP does!
defm TST  : AI1_cmp_irs<0b1000, "tst",
                        IIC_iTSTi, IIC_iTSTr, IIC_iTSTsr,
                      BinOpFrag<(ARMcmpZ (and_su node:$LHS, node:$RHS), 0)>, 1,
                      "DecodeTSTInstruction">;
defm TEQ  : AI1_cmp_irs<0b1001, "teq",
                        IIC_iTSTi, IIC_iTSTr, IIC_iTSTsr,
                      BinOpFrag<(ARMcmpZ (xor_su node:$LHS, node:$RHS), 0)>, 1>;
```
- EN: Defines TableGen record `TST` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TST`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5160-5167
```tablegen
// Pseudo i64 compares for some floating point compares.
let usesCustomInserter = 1, isBranch = 1, isTerminator = 1,
    Defs = [CPSR] in {
def BCCi64 : PseudoInst<(outs),
    (ins i32imm:$cc, GPR:$lhs1, GPR:$lhs2, GPR:$rhs1, GPR:$rhs2, brtarget:$dst),
     IIC_Br,
    [(ARMBcci64 imm:$cc, GPR:$lhs1, GPR:$lhs2, GPR:$rhs1, GPR:$rhs2, bb:$dst)]>,
    Sched<[WriteBr]>;
```
- EN: Defines TableGen record `BCCi64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BCCi64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5169-5173
```tablegen
def BCCZi64 : PseudoInst<(outs),
     (ins i32imm:$cc, GPR:$lhs1, GPR:$lhs2, brtarget:$dst), IIC_Br,
    [(ARMBcci64 imm:$cc, GPR:$lhs1, GPR:$lhs2, 0, 0, bb:$dst)]>,
    Sched<[WriteBr]>;
} // usesCustomInserter
```
- EN: Defines TableGen record `BCCZi64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `BCCZi64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5176-5177
```tablegen
// Conditional moves
let hasSideEffects = 0 in {
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5179-5183
```tablegen
let isCommutable = 1, isSelect = 1 in
def MOVCCr : ARMPseudoInst<(outs GPR:$Rd),
                           (ins GPR:$false, GPR:$Rm, pred:$p),
                           4, IIC_iCMOVr, []>,
             RegConstraint<"$false = $Rd">, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `MOVCCr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVCCr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5185-5192
```tablegen
def MOVCCsi : ARMPseudoInst<(outs GPR:$Rd),
                            (ins GPR:$false, so_reg_imm:$shift, pred:$p),
                            4, IIC_iCMOVsr, []>,
      RegConstraint<"$false = $Rd">, Sched<[WriteALU]>;
def MOVCCsr : ARMPseudoInst<(outs GPR:$Rd),
                            (ins GPR:$false, so_reg_reg:$shift, pred:$p),
                           4, IIC_iCMOVsr, []>,
      RegConstraint<"$false = $Rd">, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `MOVCCsi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVCCsi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5195-5201
```tablegen
let isMoveImm = 1 in
def MOVCCi16
    : ARMPseudoInst<(outs GPR:$Rd),
                    (ins GPR:$false, imm0_65535_expr:$imm, pred:$p),
                    4, IIC_iMOVi, []>,
      RegConstraint<"$false = $Rd">, Requires<[IsARM, HasV6T2]>,
      Sched<[WriteALU]>;
```
- EN: Defines TableGen record `MOVCCi16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVCCi16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5203-5207
```tablegen
let isMoveImm = 1 in
def MOVCCi : ARMPseudoInst<(outs GPR:$Rd),
                           (ins GPR:$false, mod_imm:$imm, pred:$p),
                           4, IIC_iCMOVi, []>,
      RegConstraint<"$false = $Rd">, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `MOVCCi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVCCi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5209-5215
```tablegen
// Two instruction predicate mov immediate.
let isMoveImm = 1 in
def MOVCCi32imm
    : ARMPseudoInst<(outs GPR:$Rd),
                    (ins GPR:$false, i32imm:$src, pred:$p),
                    8, IIC_iCMOVix2, []>,
      RegConstraint<"$false = $Rd">, Requires<[IsARM, HasV6T2]>;
```
- EN: Defines TableGen record `MOVCCi32imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVCCi32imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5217-5221
```tablegen
let isMoveImm = 1 in
def MVNCCi : ARMPseudoInst<(outs GPR:$Rd),
                           (ins GPR:$false, mod_imm:$imm, pred:$p),
                           4, IIC_iCMOVi, []>,
                RegConstraint<"$false = $Rd">, Sched<[WriteALU]>;
```
- EN: Defines TableGen record `MVNCCi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MVNCCi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5223-5223
```tablegen
} // hasSideEffects
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5225-5227
```tablegen
// The following patterns have to be defined out-of-line because the number
// of instruction operands does not match the number of SDNode operands
// (`pred` counts as one operand).
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 5229-5230
```tablegen
def : ARMPat<(ARMcmov i32:$false, i32:$Rm, imm:$cc, CPSR),
             (MOVCCr $false, $Rm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5232-5233
```tablegen
def : ARMPat<(ARMcmov i32:$false, so_reg_imm:$shift, imm:$cc, CPSR),
             (MOVCCsi $false, so_reg_imm:$shift, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5235-5236
```tablegen
def : ARMPat<(ARMcmov i32:$false, so_reg_reg:$shift, imm:$cc, CPSR),
             (MOVCCsr $false, so_reg_reg:$shift, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5238-5239
```tablegen
def : ARMV6T2Pat<(ARMcmov i32:$false, imm0_65535:$imm, imm:$cc, CPSR),
                 (MOVCCi16 $false, imm0_65535:$imm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5241-5242
```tablegen
def : ARMPat<(ARMcmov i32:$false, mod_imm:$imm, imm:$cc, CPSR),
             (MOVCCi $false, mod_imm:$imm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5244-5245
```tablegen
def : ARMPat<(ARMcmov i32:$false, mod_imm_not:$imm, imm:$cc, CPSR),
             (MVNCCi $false, mod_imm_not:$imm, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5247-5248
```tablegen
def : ARMV6T2Pat<(ARMcmov i32:$false, imm:$src, imm:$cc, CPSR),
                 (MOVCCi32imm $false, imm:$src, imm:$cc, CPSR)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5250-5252
```tablegen
//===----------------------------------------------------------------------===//
// Atomic operations intrinsics
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 5254-5262
```tablegen
def MemBarrierOptOperand : AsmOperandClass {
  let Name = "MemBarrierOpt";
  let ParserMethod = "parseMemBarrierOptOperand";
}
def memb_opt : Operand<i32> {
  let PrintMethod = "printMemBOption";
  let ParserMatchClass = MemBarrierOptOperand;
  let DecoderMethod = "DecodeMemBarrierOption";
}
```
- EN: Defines TableGen record `MemBarrierOptOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MemBarrierOptOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5264-5272
```tablegen
def InstSyncBarrierOptOperand : AsmOperandClass {
  let Name = "InstSyncBarrierOpt";
  let ParserMethod = "parseInstSyncBarrierOptOperand";
}
def instsyncb_opt : Operand<i32> {
  let PrintMethod = "printInstSyncBOption";
  let ParserMatchClass = InstSyncBarrierOptOperand;
  let DecoderMethod = "DecodeInstSyncBarrierOption";
}
```
- EN: Defines TableGen record `InstSyncBarrierOptOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `InstSyncBarrierOptOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5274-5281
```tablegen
def TraceSyncBarrierOptOperand : AsmOperandClass {
  let Name = "TraceSyncBarrierOpt";
  let ParserMethod = "parseTraceSyncBarrierOptOperand";
}
def tsb_opt : Operand<i32> {
  let PrintMethod = "printTraceSyncBOption";
  let ParserMatchClass = TraceSyncBarrierOptOperand;
}
```
- EN: Defines TableGen record `TraceSyncBarrierOptOperand` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TraceSyncBarrierOptOperand`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5283-5291
```tablegen
// Memory barriers protect the atomic sequences
let hasSideEffects = 1 in {
def DMB : AInoP<(outs), (ins memb_opt:$opt), MiscFrm, NoItinerary,
                "dmb", "\t$opt", [(int_arm_dmb (i32 imm0_15:$opt))]>,
                Requires<[IsARM, HasDB]> {
  bits<4> opt;
  let Inst{31-4} = 0xf57ff05;
  let Inst{3-0} = opt;
}
```
- EN: Defines TableGen record `DMB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DMB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5293-5299
```tablegen
def DSB : AInoP<(outs), (ins memb_opt:$opt), MiscFrm, NoItinerary,
                "dsb", "\t$opt", [(int_arm_dsb (i32 imm0_15:$opt))]>,
                Requires<[IsARM, HasDB]> {
  bits<4> opt;
  let Inst{31-4} = 0xf57ff04;
  let Inst{3-0} = opt;
}
```
- EN: Defines TableGen record `DSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `DSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5301-5308
```tablegen
// ISB has only full system option
def ISB : AInoP<(outs), (ins instsyncb_opt:$opt), MiscFrm, NoItinerary,
                "isb", "\t$opt", [(int_arm_isb (i32 imm0_15:$opt))]>,
                Requires<[IsARM, HasDB]> {
  bits<4> opt;
  let Inst{31-4} = 0xf57ff06;
  let Inst{3-0} = opt;
}
```
- EN: Defines TableGen record `ISB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ISB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5310-5315
```tablegen
let hasNoSchedulingInfo = 1 in
def TSB : AInoP<(outs), (ins tsb_opt:$opt), MiscFrm, NoItinerary,
                "tsb", "\t$opt", []>, Requires<[IsARM, HasV8_4a]> {
  let Inst{31-0} = 0xe320f012;
  let DecoderMethod = "DecodeTSBInstruction";
}
```
- EN: Defines TableGen record `TSB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TSB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5317-5317
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5319-5325
```tablegen
// Armv8.5-A speculation barrier
def SB : AInoP<(outs), (ins), MiscFrm, NoItinerary, "sb", "", []>,
         Requires<[IsARM, HasSB]>, Sched<[]> {
  let Inst{31-0} = 0xf57ff070;
  let Unpredictable = 0x000fff0f;
  let hasSideEffects = 1;
}
```
- EN: Defines TableGen record `SB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5327-5332
```tablegen
let usesCustomInserter = 1, Defs = [CPSR], hasNoSchedulingInfo = 1 in {
    def COPY_STRUCT_BYVAL_I32 : PseudoInst<
      (outs), (ins GPR:$dst, GPR:$src, i32imm:$size, i32imm:$alignment),
      NoItinerary,
      [(ARMcopystructbyval GPR:$dst, GPR:$src, imm:$size, imm:$alignment)]>;
}
```
- EN: Defines TableGen record `COPY_STRUCT_BYVAL_I32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `COPY_STRUCT_BYVAL_I32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5334-5345
```tablegen
let hasPostISelHook = 1, Constraints = "$newdst = $dst, $newsrc = $src" in {
    // %newsrc, %newdst = MEMCPY %dst, %src, N, ...N scratch regs...
    // Copies N registers worth of memory from address %src to address %dst
    // and returns the incremented addresses.  N scratch register will
    // be attached for the copy to use.
    def MEMCPY : PseudoInst<
      (outs GPR:$newdst, GPR:$newsrc),
      (ins GPR:$dst, GPR:$src, i32imm:$nreg, variable_ops),
      NoItinerary,
      [(set GPR:$newdst, GPR:$newsrc,
            (ARMmemcopy GPR:$dst, GPR:$src, imm:$nreg))]>;
}
```
- EN: Defines TableGen record `MEMCPY` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MEMCPY`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5347-5349
```tablegen
def ldrex_1 : PatFrag<(ops node:$ptr), (int_arm_ldrex node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i8;
}]>;
```
- EN: Defines TableGen record `ldrex_1` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ldrex_1`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5351-5353
```tablegen
def ldrex_2 : PatFrag<(ops node:$ptr), (int_arm_ldrex node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i16;
}]>;
```
- EN: Defines TableGen record `ldrex_2` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ldrex_2`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5355-5357
```tablegen
def ldrex_4 : PatFrag<(ops node:$ptr), (int_arm_ldrex node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i32;
}]>;
```
- EN: Defines TableGen record `ldrex_4` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ldrex_4`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5359-5362
```tablegen
def strex_1 : PatFrag<(ops node:$val, node:$ptr),
                      (int_arm_strex node:$val, node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i8;
}]>;
```
- EN: Defines TableGen record `strex_1` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `strex_1`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5364-5367
```tablegen
def strex_2 : PatFrag<(ops node:$val, node:$ptr),
                      (int_arm_strex node:$val, node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i16;
}]>;
```
- EN: Defines TableGen record `strex_2` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `strex_2`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5369-5372
```tablegen
def strex_4 : PatFrag<(ops node:$val, node:$ptr),
                      (int_arm_strex node:$val, node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i32;
}]>;
```
- EN: Defines TableGen record `strex_4` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `strex_4`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5374-5376
```tablegen
def ldaex_1 : PatFrag<(ops node:$ptr), (int_arm_ldaex node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i8;
}]>;
```
- EN: Defines TableGen record `ldaex_1` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ldaex_1`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5378-5380
```tablegen
def ldaex_2 : PatFrag<(ops node:$ptr), (int_arm_ldaex node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i16;
}]>;
```
- EN: Defines TableGen record `ldaex_2` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ldaex_2`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5382-5384
```tablegen
def ldaex_4 : PatFrag<(ops node:$ptr), (int_arm_ldaex node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i32;
}]>;
```
- EN: Defines TableGen record `ldaex_4` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `ldaex_4`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5386-5389
```tablegen
def stlex_1 : PatFrag<(ops node:$val, node:$ptr),
                      (int_arm_stlex node:$val, node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i8;
}]>;
```
- EN: Defines TableGen record `stlex_1` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `stlex_1`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5391-5394
```tablegen
def stlex_2 : PatFrag<(ops node:$val, node:$ptr),
                      (int_arm_stlex node:$val, node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i16;
}]>;
```
- EN: Defines TableGen record `stlex_2` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `stlex_2`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5396-5399
```tablegen
def stlex_4 : PatFrag<(ops node:$val, node:$ptr),
                      (int_arm_stlex node:$val, node:$ptr), [{
  return cast<MemIntrinsicSDNode>(N)->getMemoryVT() == MVT::i32;
}]>;
```
- EN: Defines TableGen record `stlex_4` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `stlex_4`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 5401-5415
```tablegen
let mayLoad = 1 in {
def LDREXB : AIldrex<0b10, (outs GPR:$Rt), (ins addr_offset_none:$addr),
                     NoItinerary, "ldrexb", "\t$Rt, $addr",
                     [(set GPR:$Rt, (ldrex_1 addr_offset_none:$addr))]>;
def LDREXH : AIldrex<0b11, (outs GPR:$Rt), (ins addr_offset_none:$addr),
                     NoItinerary, "ldrexh", "\t$Rt, $addr",
                     [(set GPR:$Rt, (ldrex_2 addr_offset_none:$addr))]>;
def LDREX  : AIldrex<0b00, (outs GPR:$Rt), (ins addr_offset_none:$addr),
                     NoItinerary, "ldrex", "\t$Rt, $addr",
                     [(set GPR:$Rt, (ldrex_4 addr_offset_none:$addr))]>;
let hasExtraDefRegAllocReq = 1 in
def LDREXD : AIldrex<0b01, (outs GPRPairOp:$Rt),(ins addr_offset_none:$addr),
                      NoItinerary, "ldrexd", "\t$Rt, $addr", []> {
  let DecoderMethod = "DecodeDoubleRegLoad";
}
```
- EN: Defines TableGen record `LDREXB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDREXB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5417-5431
```tablegen
def LDAEXB : AIldaex<0b10, (outs GPR:$Rt), (ins addr_offset_none:$addr),
                     NoItinerary, "ldaexb", "\t$Rt, $addr",
                     [(set GPR:$Rt, (ldaex_1 addr_offset_none:$addr))]>;
def LDAEXH : AIldaex<0b11, (outs GPR:$Rt), (ins addr_offset_none:$addr),
                     NoItinerary, "ldaexh", "\t$Rt, $addr",
                    [(set GPR:$Rt, (ldaex_2 addr_offset_none:$addr))]>;
def LDAEX  : AIldaex<0b00, (outs GPR:$Rt), (ins addr_offset_none:$addr),
                     NoItinerary, "ldaex", "\t$Rt, $addr",
                    [(set GPR:$Rt, (ldaex_4 addr_offset_none:$addr))]>;
let hasExtraDefRegAllocReq = 1 in
def LDAEXD : AIldaex<0b01, (outs GPRPairOp:$Rt),(ins addr_offset_none:$addr),
                      NoItinerary, "ldaexd", "\t$Rt, $addr", []> {
  let DecoderMethod = "DecodeDoubleRegLoad";
}
}
```
- EN: Defines TableGen record `LDAEXB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDAEXB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5433-5450
```tablegen
let mayStore = 1, Constraints = "@earlyclobber $Rd" in {
def STREXB: AIstrex<0b10, (outs GPR:$Rd), (ins GPR:$Rt, addr_offset_none:$addr),
                    NoItinerary, "strexb", "\t$Rd, $Rt, $addr",
                    [(set GPR:$Rd, (strex_1 GPR:$Rt,
                                            addr_offset_none:$addr))]>;
def STREXH: AIstrex<0b11, (outs GPR:$Rd), (ins GPR:$Rt, addr_offset_none:$addr),
                    NoItinerary, "strexh", "\t$Rd, $Rt, $addr",
                    [(set GPR:$Rd, (strex_2 GPR:$Rt,
                                            addr_offset_none:$addr))]>;
def STREX : AIstrex<0b00, (outs GPR:$Rd), (ins GPR:$Rt, addr_offset_none:$addr),
                    NoItinerary, "strex", "\t$Rd, $Rt, $addr",
                    [(set GPR:$Rd, (strex_4 GPR:$Rt,
                                            addr_offset_none:$addr))]>;
let hasExtraSrcRegAllocReq = 1 in
def STREXD : AIstrex<0b01, (outs GPR:$Rd),
                    (ins GPRPairOp:$Rt, addr_offset_none:$addr),
                    NoItinerary, "strexd", "\t$Rd, $Rt, $addr", []> {
  let DecoderMethod = "DecodeDoubleRegStore";
```
- EN: Defines TableGen record `STREXB:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STREXB:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5451-5468
```tablegen
}
def STLEXB: AIstlex<0b10, (outs GPR:$Rd), (ins GPR:$Rt, addr_offset_none:$addr),
                    NoItinerary, "stlexb", "\t$Rd, $Rt, $addr",
                    [(set GPR:$Rd,
                          (stlex_1 GPR:$Rt, addr_offset_none:$addr))]>;
def STLEXH: AIstlex<0b11, (outs GPR:$Rd), (ins GPR:$Rt, addr_offset_none:$addr),
                    NoItinerary, "stlexh", "\t$Rd, $Rt, $addr",
                    [(set GPR:$Rd,
                          (stlex_2 GPR:$Rt, addr_offset_none:$addr))]>;
def STLEX : AIstlex<0b00, (outs GPR:$Rd), (ins GPR:$Rt, addr_offset_none:$addr),
                    NoItinerary, "stlex", "\t$Rd, $Rt, $addr",
                    [(set GPR:$Rd,
                          (stlex_4 GPR:$Rt, addr_offset_none:$addr))]>;
let hasExtraSrcRegAllocReq = 1 in
def STLEXD : AIstlex<0b01, (outs GPR:$Rd),
                    (ins GPRPairOp:$Rt, addr_offset_none:$addr),
                    NoItinerary, "stlexd", "\t$Rd, $Rt, $addr", []> {
  let DecoderMethod = "DecodeDoubleRegStore";
```
- EN: Defines TableGen record `STLEXB:` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STLEXB:`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5469-5470
```tablegen
}
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5472-5476
```tablegen
def CLREX : AXI<(outs), (ins), MiscFrm, NoItinerary, "clrex",
                [(int_arm_clrex)]>,
            Requires<[IsARM, HasV6K]>  {
  let Inst{31-0} = 0b11110101011111111111000000011111;
}
```
- EN: Defines TableGen record `CLREX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CLREX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5478-5481
```tablegen
def : ARMPat<(strex_1 (and GPR:$Rt, 0xff), addr_offset_none:$addr),
             (STREXB GPR:$Rt, addr_offset_none:$addr)>;
def : ARMPat<(strex_2 (and GPR:$Rt, 0xffff), addr_offset_none:$addr),
             (STREXH GPR:$Rt, addr_offset_none:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5483-5486
```tablegen
def : ARMPat<(stlex_1 (and GPR:$Rt, 0xff), addr_offset_none:$addr),
             (STLEXB GPR:$Rt, addr_offset_none:$addr)>;
def : ARMPat<(stlex_2 (and GPR:$Rt, 0xffff), addr_offset_none:$addr),
             (STLEXH GPR:$Rt, addr_offset_none:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5488-5492
```tablegen
class acquiring_load<PatFrags base>
  : PatFrag<(ops node:$ptr), (base node:$ptr), [{
  AtomicOrdering Ordering = cast<AtomicSDNode>(N)->getSuccessOrdering();
  return isAcquireOrStronger(Ordering);
}]>;
```
- EN: Declares reusable TableGen class `acquiring_load` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `acquiring_load`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5494-5496
```tablegen
def atomic_load_azext_acquire_8  : acquiring_load<atomic_load_azext_8>;
def atomic_load_azext_acquire_16 : acquiring_load<atomic_load_azext_16>;
def atomic_load_nonext_acquire_32 : acquiring_load<atomic_load_nonext_32>;
```
- EN: Defines TableGen record `atomic_load_azext_acquire_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `atomic_load_azext_acquire_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5498-5502
```tablegen
class releasing_store<PatFrag base>
  : PatFrag<(ops node:$ptr, node:$val), (base node:$val, node:$ptr), [{
  AtomicOrdering Ordering = cast<AtomicSDNode>(N)->getSuccessOrdering();
  return isReleaseOrStronger(Ordering);
}]>;
```
- EN: Declares reusable TableGen class `releasing_store` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `releasing_store`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5504-5506
```tablegen
def atomic_store_release_8  : releasing_store<atomic_store_8>;
def atomic_store_release_16 : releasing_store<atomic_store_16>;
def atomic_store_release_32 : releasing_store<atomic_store_32>;
```
- EN: Defines TableGen record `atomic_store_release_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `atomic_store_release_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5508-5515
```tablegen
let AddedComplexity = 8 in {
  def : ARMPat<(atomic_load_azext_acquire_8 addr_offset_none:$addr),  (LDAB addr_offset_none:$addr)>;
  def : ARMPat<(atomic_load_azext_acquire_16 addr_offset_none:$addr), (LDAH addr_offset_none:$addr)>;
  def : ARMPat<(atomic_load_nonext_acquire_32 addr_offset_none:$addr), (LDA  addr_offset_none:$addr)>;
  def : ARMPat<(atomic_store_release_8 addr_offset_none:$addr, GPR:$val),  (STLB GPR:$val, addr_offset_none:$addr)>;
  def : ARMPat<(atomic_store_release_16 addr_offset_none:$addr, GPR:$val), (STLH GPR:$val, addr_offset_none:$addr)>;
  def : ARMPat<(atomic_store_release_32 addr_offset_none:$addr, GPR:$val), (STL  GPR:$val, addr_offset_none:$addr)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5517-5526
```tablegen
// SWP/SWPB are deprecated in V6/V7 and optional in v7VE.
// FIXME Use InstAlias to generate LDREX/STREX pairs instead.
let mayLoad = 1, mayStore = 1 in {
def SWP : AIswp<0, (outs GPRnopc:$Rt),
                (ins GPRnopc:$Rt2, addr_offset_none:$addr), "swp", []>,
                Requires<[IsARM,PreV8]>;
def SWPB: AIswp<1, (outs GPRnopc:$Rt),
                (ins GPRnopc:$Rt2, addr_offset_none:$addr), "swpb", []>,
                Requires<[IsARM,PreV8]>;
}
```
- EN: Defines TableGen record `SWP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SWP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5528-5530
```tablegen
//===----------------------------------------------------------------------===//
// Coprocessor Instructions.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 5532-5543
```tablegen
def CDP : ABI<0b1110, (outs), (ins p_imm:$cop, imm0_15:$opc1,
            c_imm:$CRd, c_imm:$CRn, c_imm:$CRm, imm0_7:$opc2),
            NoItinerary, "cdp", "\t$cop, $opc1, $CRd, $CRn, $CRm, $opc2",
            [(int_arm_cdp timm:$cop, timm:$opc1, timm:$CRd, timm:$CRn,
                          timm:$CRm, timm:$opc2)]>,
            Requires<[IsARM,PreV8]> {
  bits<4> opc1;
  bits<4> CRn;
  bits<4> CRd;
  bits<4> cop;
  bits<3> opc2;
  bits<4> CRm;
```
- EN: Defines TableGen record `CDP` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CDP`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5545-5551
```tablegen
  let Inst{3-0}   = CRm;
  let Inst{4}     = 0;
  let Inst{7-5}   = opc2;
  let Inst{11-8}  = cop;
  let Inst{15-12} = CRd;
  let Inst{19-16} = CRn;
  let Inst{23-20} = opc1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5553-5554
```tablegen
  let DecoderNamespace = "CoProc";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5556-5568
```tablegen
def CDP2 : ABXI<0b1110, (outs), (ins p_imm:$cop, imm0_15:$opc1,
               c_imm:$CRd, c_imm:$CRn, c_imm:$CRm, imm0_7:$opc2),
               NoItinerary, "cdp2\t$cop, $opc1, $CRd, $CRn, $CRm, $opc2",
               [(int_arm_cdp2 timm:$cop, timm:$opc1, timm:$CRd, timm:$CRn,
                              timm:$CRm, timm:$opc2)]>,
               Requires<[IsARM,PreV8]> {
  let Inst{31-28} = 0b1111;
  bits<4> opc1;
  bits<4> CRn;
  bits<4> CRd;
  bits<4> cop;
  bits<3> opc2;
  bits<4> CRm;
```
- EN: Defines TableGen record `CDP2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CDP2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5570-5576
```tablegen
  let Inst{3-0}   = CRm;
  let Inst{4}     = 0;
  let Inst{7-5}   = opc2;
  let Inst{11-8}  = cop;
  let Inst{15-12} = CRd;
  let Inst{19-16} = CRn;
  let Inst{23-20} = opc1;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5578-5579
```tablegen
  let DecoderNamespace = "CoProc";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5581-5595
```tablegen
class ACI<dag oops, dag iops, string opc, string asm,
            list<dag> pattern, IndexMode im = IndexModeNone,
            AddrMode am = AddrModeNone>
  : I<oops, iops, am, 4, im, BrFrm, NoItinerary,
      opc, asm, "", pattern> {
  let Inst{27-25} = 0b110;
}
class ACInoP<dag oops, dag iops, string opc, string asm,
          list<dag> pattern, IndexMode im = IndexModeNone,
          AddrMode am = AddrModeNone>
  : InoP<oops, iops, am, 4, im, BrFrm, NoItinerary,
         opc, asm, "", pattern> {
  let Inst{31-28} = 0b1111;
  let Inst{27-25} = 0b110;
}
```
- EN: Declares reusable TableGen class `ACI` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `ACI`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5597-5614
```tablegen
let DecoderNamespace = "CoProc" in {
multiclass LdStCop<bit load, bit Dbit, string asm, list<dag> pattern> {
  def _OFFSET : ACI<(outs), (ins p_imm:$cop, c_imm:$CRd, addrmode5:$addr),
                    asm, "\t$cop, $CRd, $addr", pattern, IndexModeNone,
                    AddrMode5> {
    bits<13> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 1; // P = 1
    let Inst{23} = addr{8};
    let Inst{22} = Dbit;
    let Inst{21} = 0; // W = 0
    let Inst{20} = load;
    let Inst{19-16} = addr{12-9};
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = addr{7-0};
    let DecoderMethod = "DecodeCopMemInstruction";
```
- EN: Declares TableGen `multiclass LdStCop`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass LdStCop`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5615-5632
```tablegen
  }
  def _PRE : ACI<(outs), (ins p_imm:$cop, c_imm:$CRd, addrmode5_pre:$addr),
                 asm, "\t$cop, $CRd, $addr!", [], IndexModePre> {
    bits<13> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 1; // P = 1
    let Inst{23} = addr{8};
    let Inst{22} = Dbit;
    let Inst{21} = 1; // W = 1
    let Inst{20} = load;
    let Inst{19-16} = addr{12-9};
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = addr{7-0};
    let DecoderMethod = "DecodeCopMemInstruction";
  }
  def _POST: ACI<(outs), (ins p_imm:$cop, c_imm:$CRd, addr_offset_none:$addr,
```
- EN: Defines TableGen record `_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5633-5650
```tablegen
                              postidx_imm8s4:$offset),
                 asm, "\t$cop, $CRd, $addr, $offset", [], IndexModePost> {
    bits<9> offset;
    bits<4> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 0; // P = 0
    let Inst{23} = offset{8};
    let Inst{22} = Dbit;
    let Inst{21} = 1; // W = 1
    let Inst{20} = load;
    let Inst{19-16} = addr;
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = offset{7-0};
    let DecoderMethod = "DecodeCopMemInstruction";
  }
  def _OPTION : ACI<(outs),
```
- EN: Defines TableGen record `_OPTION` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_OPTION`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5651-5668
```tablegen
                    (ins p_imm:$cop, c_imm:$CRd, addr_offset_none:$addr,
                         coproc_option_imm:$option),
      asm, "\t$cop, $CRd, $addr, $option", []> {
    bits<8> option;
    bits<4> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 0; // P = 0
    let Inst{23} = 1; // U = 1
    let Inst{22} = Dbit;
    let Inst{21} = 0; // W = 0
    let Inst{20} = load;
    let Inst{19-16} = addr;
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = option;
    let DecoderMethod = "DecodeCopMemInstruction";
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5669-5686
```tablegen
}
multiclass LdSt2Cop<bit load, bit Dbit, string asm, list<dag> pattern> {
  def _OFFSET : ACInoP<(outs), (ins p_imm:$cop, c_imm:$CRd, addrmode5:$addr),
                       asm, "\t$cop, $CRd, $addr", pattern, IndexModeNone,
                       AddrMode5> {
    bits<13> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 1; // P = 1
    let Inst{23} = addr{8};
    let Inst{22} = Dbit;
    let Inst{21} = 0; // W = 0
    let Inst{20} = load;
    let Inst{19-16} = addr{12-9};
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = addr{7-0};
    let DecoderMethod = "DecodeCopMemInstruction";
```
- EN: Declares TableGen `multiclass LdSt2Cop`, a reusable template that expands into multiple related records.
- CN: 这里声明 TableGen `multiclass LdSt2Cop`，它是一个可复用模板，可展开为多个相关记录。

### Lines 5687-5704
```tablegen
  }
  def _PRE : ACInoP<(outs), (ins p_imm:$cop, c_imm:$CRd, addrmode5_pre:$addr),
                    asm, "\t$cop, $CRd, $addr!", [], IndexModePre> {
    bits<13> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 1; // P = 1
    let Inst{23} = addr{8};
    let Inst{22} = Dbit;
    let Inst{21} = 1; // W = 1
    let Inst{20} = load;
    let Inst{19-16} = addr{12-9};
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = addr{7-0};
    let DecoderMethod = "DecodeCopMemInstruction";
  }
  def _POST: ACInoP<(outs), (ins p_imm:$cop, c_imm:$CRd, addr_offset_none:$addr,
```
- EN: Defines TableGen record `_PRE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_PRE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5705-5722
```tablegen
                                 postidx_imm8s4:$offset),
                 asm, "\t$cop, $CRd, $addr, $offset", [], IndexModePost> {
    bits<9> offset;
    bits<4> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 0; // P = 0
    let Inst{23} = offset{8};
    let Inst{22} = Dbit;
    let Inst{21} = 1; // W = 1
    let Inst{20} = load;
    let Inst{19-16} = addr;
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = offset{7-0};
    let DecoderMethod = "DecodeCopMemInstruction";
  }
  def _OPTION : ACInoP<(outs),
```
- EN: Defines TableGen record `_OPTION` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `_OPTION`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5723-5740
```tablegen
                       (ins p_imm:$cop, c_imm:$CRd, addr_offset_none:$addr,
                            coproc_option_imm:$option),
      asm, "\t$cop, $CRd, $addr, $option", []> {
    bits<8> option;
    bits<4> addr;
    bits<4> cop;
    bits<4> CRd;
    let Inst{24} = 0; // P = 0
    let Inst{23} = 1; // U = 1
    let Inst{22} = Dbit;
    let Inst{21} = 0; // W = 0
    let Inst{20} = load;
    let Inst{19-16} = addr;
    let Inst{15-12} = CRd;
    let Inst{11-8} = cop;
    let Inst{7-0} = option;
    let DecoderMethod = "DecodeCopMemInstruction";
  }
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5741-5741
```tablegen
}
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5743-5748
```tablegen
let mayLoad = 1 in {
defm LDC   : LdStCop <1, 0, "ldc", [(int_arm_ldc timm:$cop, timm:$CRd, addrmode5:$addr)]>;
defm LDCL  : LdStCop <1, 1, "ldcl", [(int_arm_ldcl timm:$cop, timm:$CRd, addrmode5:$addr)]>;
defm LDC2  : LdSt2Cop<1, 0, "ldc2", [(int_arm_ldc2 timm:$cop, timm:$CRd, addrmode5:$addr)]>, Requires<[IsARM,PreV8]>;
defm LDC2L : LdSt2Cop<1, 1, "ldc2l", [(int_arm_ldc2l timm:$cop, timm:$CRd, addrmode5:$addr)]>, Requires<[IsARM,PreV8]>;
}
```
- EN: Defines TableGen record `LDC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5750-5755
```tablegen
let mayStore = 1 in {
defm STC   : LdStCop <0, 0, "stc", [(int_arm_stc timm:$cop, timm:$CRd, addrmode5:$addr)]>;
defm STCL  : LdStCop <0, 1, "stcl", [(int_arm_stcl timm:$cop, timm:$CRd, addrmode5:$addr)]>;
defm STC2  : LdSt2Cop<0, 0, "stc2", [(int_arm_stc2 timm:$cop, timm:$CRd, addrmode5:$addr)]>, Requires<[IsARM,PreV8]>;
defm STC2L : LdSt2Cop<0, 1, "stc2l", [(int_arm_stc2l timm:$cop, timm:$CRd, addrmode5:$addr)]>, Requires<[IsARM,PreV8]>;
}
```
- EN: Defines TableGen record `STC` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `STC`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5757-5757
```tablegen
} // DecoderNamespace = "CoProc"
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5759-5761
```tablegen
//===----------------------------------------------------------------------===//
// Move between coprocessor and ARM core register.
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 5763-5768
```tablegen
class MovRCopro<string opc, bit direction, dag oops, dag iops,
                list<dag> pattern>
  : ABI<0b1110, oops, iops, NoItinerary, opc,
        "\t$cop, $opc1, $Rt, $CRn, $CRm, $opc2", pattern> {
  let Inst{20} = direction;
  let Inst{4} = 1;
```
- EN: Declares reusable TableGen class `MovRCopro` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `MovRCopro`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5770-5775
```tablegen
  bits<4> Rt;
  bits<4> cop;
  bits<3> opc1;
  bits<3> opc2;
  bits<4> CRm;
  bits<4> CRn;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5777-5782
```tablegen
  let Inst{15-12} = Rt;
  let Inst{11-8}  = cop;
  let Inst{23-21} = opc1;
  let Inst{7-5}   = opc2;
  let Inst{3-0}   = CRm;
  let Inst{19-16} = CRn;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5784-5785
```tablegen
  let DecoderNamespace = "CoProc";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5787-5804
```tablegen
def MCR : MovRCopro<"mcr", 0 /* from ARM core register to coprocessor */,
                    (outs),
                    (ins p_imm:$cop, imm0_7:$opc1, GPR:$Rt, c_imm:$CRn,
                         c_imm:$CRm, imm0_7:$opc2),
                    [(int_arm_mcr timm:$cop, timm:$opc1, GPR:$Rt, timm:$CRn,
                                  timm:$CRm, timm:$opc2)]>,
                    ComplexDeprecationPredicate<"MCR">;
def : ARMInstAlias<"mcr${p} $cop, $opc1, $Rt, $CRn, $CRm",
                   (MCR p_imm:$cop, imm0_7:$opc1, GPR:$Rt, c_imm:$CRn,
                        c_imm:$CRm, 0, pred:$p)>;
def MRC : MovRCopro<"mrc", 1 /* from coprocessor to ARM core register */,
                    (outs GPRwithAPSR:$Rt),
                    (ins p_imm:$cop, imm0_7:$opc1, c_imm:$CRn, c_imm:$CRm,
                         imm0_7:$opc2), []>,
                    ComplexDeprecationPredicate<"MRC">;
def : ARMInstAlias<"mrc${p} $cop, $opc1, $Rt, $CRn, $CRm",
                   (MRC GPRwithAPSR:$Rt, p_imm:$cop, imm0_7:$opc1, c_imm:$CRn,
                        c_imm:$CRm, 0, pred:$p)>;
```
- EN: Defines TableGen record `MCR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MCR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5806-5807
```tablegen
def : ARMPat<(int_arm_mrc timm:$cop, timm:$opc1, timm:$CRn, timm:$CRm, timm:$opc2),
             (MRC p_imm:$cop, imm0_7:$opc1, c_imm:$CRn, c_imm:$CRm, imm0_7:$opc2)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5809-5815
```tablegen
class MovRCopro2<string opc, bit direction, dag oops, dag iops,
                 list<dag> pattern>
  : ABXI<0b1110, oops, iops, NoItinerary,
         !strconcat(opc, "\t$cop, $opc1, $Rt, $CRn, $CRm, $opc2"), pattern> {
  let Inst{31-24} = 0b11111110;
  let Inst{20} = direction;
  let Inst{4} = 1;
```
- EN: Declares reusable TableGen class `MovRCopro2` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `MovRCopro2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5817-5822
```tablegen
  bits<4> Rt;
  bits<4> cop;
  bits<3> opc1;
  bits<3> opc2;
  bits<4> CRm;
  bits<4> CRn;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5824-5829
```tablegen
  let Inst{15-12} = Rt;
  let Inst{11-8}  = cop;
  let Inst{23-21} = opc1;
  let Inst{7-5}   = opc2;
  let Inst{3-0}   = CRm;
  let Inst{19-16} = CRn;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5831-5832
```tablegen
  let DecoderNamespace = "CoProc";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5834-5851
```tablegen
def MCR2 : MovRCopro2<"mcr2", 0 /* from ARM core register to coprocessor */,
                      (outs),
                      (ins p_imm:$cop, imm0_7:$opc1, GPR:$Rt, c_imm:$CRn,
                           c_imm:$CRm, imm0_7:$opc2),
                      [(int_arm_mcr2 timm:$cop, timm:$opc1, GPR:$Rt, timm:$CRn,
                                     timm:$CRm, timm:$opc2)]>,
                      Requires<[IsARM,PreV8]>;
def : ARMInstAlias<"mcr2 $cop, $opc1, $Rt, $CRn, $CRm",
                   (MCR2 p_imm:$cop, imm0_7:$opc1, GPR:$Rt, c_imm:$CRn,
                         c_imm:$CRm, 0)>;
def MRC2 : MovRCopro2<"mrc2", 1 /* from coprocessor to ARM core register */,
                      (outs GPRwithAPSR:$Rt),
                      (ins p_imm:$cop, imm0_7:$opc1, c_imm:$CRn, c_imm:$CRm,
                           imm0_7:$opc2), []>,
                      Requires<[IsARM,PreV8]>;
def : ARMInstAlias<"mrc2 $cop, $opc1, $Rt, $CRn, $CRm",
                   (MRC2 GPRwithAPSR:$Rt, p_imm:$cop, imm0_7:$opc1, c_imm:$CRn,
                         c_imm:$CRm, 0)>;
```
- EN: Defines TableGen record `MCR2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MCR2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5853-5855
```tablegen
def : ARMV5TPat<(int_arm_mrc2 timm:$cop, timm:$opc1, timm:$CRn,
                              timm:$CRm, timm:$opc2),
                (MRC2 p_imm:$cop, imm0_7:$opc1, c_imm:$CRn, c_imm:$CRm, imm0_7:$opc2)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5857-5860
```tablegen
class MovRRCopro<string opc, bit direction, dag oops, dag iops, list<dag>
                 pattern = []>
  : ABI<0b1100, oops, iops, NoItinerary, opc, "\t$cop, $opc1, $Rt, $Rt2, $CRm",
        pattern> {
```
- EN: Declares reusable TableGen class `MovRRCopro` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `MovRRCopro`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5862-5863
```tablegen
  let Inst{23-21} = 0b010;
  let Inst{20} = direction;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5865-5869
```tablegen
  bits<4> Rt;
  bits<4> Rt2;
  bits<4> cop;
  bits<4> opc1;
  bits<4> CRm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5871-5876
```tablegen
  let Inst{15-12} = Rt;
  let Inst{19-16} = Rt2;
  let Inst{11-8}  = cop;
  let Inst{7-4}   = opc1;
  let Inst{3-0}   = CRm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5878-5885
```tablegen
def MCRR : MovRRCopro<"mcrr", 0 /* from ARM core register to coprocessor */,
                      (outs), (ins p_imm:$cop, imm0_15:$opc1, GPRnopc:$Rt,
                      GPRnopc:$Rt2, c_imm:$CRm),
                      [(int_arm_mcrr timm:$cop, timm:$opc1, GPRnopc:$Rt,
                                     GPRnopc:$Rt2, timm:$CRm)]>;
def MRRC : MovRRCopro<"mrrc", 1 /* from coprocessor to ARM core register */,
                      (outs GPRnopc:$Rt, GPRnopc:$Rt2),
                      (ins p_imm:$cop, imm0_15:$opc1, c_imm:$CRm), []>;
```
- EN: Defines TableGen record `MCRR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MCRR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5887-5894
```tablegen
class MovRRCopro2<string opc, bit direction, dag oops, dag iops,
                  list<dag> pattern = []>
  : ABXI<0b1100, oops, iops, NoItinerary,
         !strconcat(opc, "\t$cop, $opc1, $Rt, $Rt2, $CRm"), pattern>,
    Requires<[IsARM,PreV8]> {
  let Inst{31-28} = 0b1111;
  let Inst{23-21} = 0b010;
  let Inst{20} = direction;
```
- EN: Declares reusable TableGen class `MovRRCopro2` for `ARMInstrInfo`, typically capturing shared fields, predicates, or encoding structure.
- CN: 这里为 `ARMInstrInfo` 声明可复用的 TableGen 类 `MovRRCopro2`，通常用于抽象共享字段、谓词或编码结构。

### Lines 5896-5900
```tablegen
  bits<4> Rt;
  bits<4> Rt2;
  bits<4> cop;
  bits<4> opc1;
  bits<4> CRm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5902-5906
```tablegen
  let Inst{15-12} = Rt;
  let Inst{19-16} = Rt2;
  let Inst{11-8}  = cop;
  let Inst{7-4}   = opc1;
  let Inst{3-0}   = CRm;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5908-5909
```tablegen
  let DecoderMethod = "DecoderForMRRC2AndMCRR2";
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5911-5915
```tablegen
def MCRR2 : MovRRCopro2<"mcrr2", 0 /* from ARM core register to coprocessor */,
                        (outs), (ins p_imm:$cop, imm0_15:$opc1, GPRnopc:$Rt,
                        GPRnopc:$Rt2, c_imm:$CRm),
                        [(int_arm_mcrr2 timm:$cop, timm:$opc1, GPRnopc:$Rt,
                                        GPRnopc:$Rt2, timm:$CRm)]>;
```
- EN: Defines TableGen record `MCRR2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MCRR2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5917-5919
```tablegen
def MRRC2 : MovRRCopro2<"mrrc2", 1 /* from coprocessor to ARM core register */,
                       (outs GPRnopc:$Rt, GPRnopc:$Rt2),
                       (ins p_imm:$cop, imm0_15:$opc1, c_imm:$CRm), []>;
```
- EN: Defines TableGen record `MRRC2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MRRC2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5921-5923
```tablegen
//===----------------------------------------------------------------------===//
// Move between special register and ARM core register
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 5925-5930
```tablegen
// Move to ARM core register from Special Register
def MRS : ABI<0b0001, (outs GPRnopc:$Rd), (ins), NoItinerary,
              "mrs", "\t$Rd, apsr", []> {
  bits<4> Rd;
  let Inst{23-16} = 0b00001111;
  let Unpredictable{19-17} = 0b111;
```
- EN: Defines TableGen record `MRS` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MRS`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5932-5932
```tablegen
  let Inst{15-12} = Rd;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5934-5936
```tablegen
  let Inst{11-0} = 0b000000000000;
  let Unpredictable{11-0} = 0b110100001111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5938-5939
```tablegen
def : InstAlias<"mrs${p} $Rd, cpsr", (MRS GPRnopc:$Rd, pred:$p), 0>,
         Requires<[IsARM]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 5941-5947
```tablegen
// The MRSsys instruction is the MRS instruction from the ARM ARM,
// section B9.3.9, with the R bit set to 1.
def MRSsys : ABI<0b0001, (outs GPRnopc:$Rd), (ins), NoItinerary,
                 "mrs", "\t$Rd, spsr", []> {
  bits<4> Rd;
  let Inst{23-16} = 0b01001111;
  let Unpredictable{19-16} = 0b1111;
```
- EN: Defines TableGen record `MRSsys` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MRSsys`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5949-5949
```tablegen
  let Inst{15-12} = Rd;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5951-5953
```tablegen
  let Inst{11-0} = 0b000000000000;
  let Unpredictable{11-0} = 0b110100001111;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5955-5961
```tablegen
// However, the MRS (banked register) system instruction (ARMv7VE) *does* have a
// separate encoding (distinguished by bit 5.
def MRSbanked : ABI<0b0001, (outs GPRnopc:$Rd), (ins banked_reg:$banked),
                    NoItinerary, "mrs", "\t$Rd, $banked", []>,
                Requires<[IsARM, HasVirtualization]> {
  bits<6> banked;
  bits<4> Rd;
```
- EN: Defines TableGen record `MRSbanked` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MRSbanked`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5963-5971
```tablegen
  let Inst{23} = 0;
  let Inst{22} = banked{5}; // R bit
  let Inst{21-20} = 0b00;
  let Inst{19-16} = banked{3-0};
  let Inst{15-12} = Rd;
  let Inst{11-9} = 0b001;
  let Inst{8} = banked{4};
  let Inst{7-0} = 0b00000000;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5973-5984
```tablegen
// Move from ARM core register to Special Register
//
// No need to have both system and application versions of MSR (immediate) or
// MSR (register), the encodings are the same and the assembly parser has no way
// to distinguish between them. The mask operand contains the special register
// (R Bit) in bit 4 and bits 3-0 contains the mask with the fields to be
// accessed in the special register.
let Defs = [CPSR] in
def MSR : ABI<0b0001, (outs), (ins msr_mask:$mask, GPR:$Rn), NoItinerary,
              "msr", "\t$mask, $Rn", []> {
  bits<5> mask;
  bits<4> Rn;
```
- EN: Defines TableGen record `MSR` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSR`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 5986-5993
```tablegen
  let Inst{23} = 0;
  let Inst{22} = mask{4}; // R bit
  let Inst{21-20} = 0b10;
  let Inst{19-16} = mask{3-0};
  let Inst{15-12} = 0b1111;
  let Inst{11-4} = 0b00000000;
  let Inst{3-0} = Rn;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 5995-5999
```tablegen
let Defs = [CPSR] in
def MSRi : ABI<0b0011, (outs), (ins msr_mask:$mask,  mod_imm:$imm), NoItinerary,
               "msr", "\t$mask, $imm", []> {
  bits<5> mask;
  bits<12> imm;
```
- EN: Defines TableGen record `MSRi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSRi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6001-6007
```tablegen
  let Inst{23} = 0;
  let Inst{22} = mask{4}; // R bit
  let Inst{21-20} = 0b10;
  let Inst{19-16} = mask{3-0};
  let Inst{15-12} = 0b1111;
  let Inst{11-0} = imm;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6009-6015
```tablegen
// However, the MSR (banked register) system instruction (ARMv7VE) *does* have a
// separate encoding (distinguished by bit 5.
def MSRbanked : ABI<0b0001, (outs), (ins banked_reg:$banked, GPRnopc:$Rn),
                    NoItinerary, "msr", "\t$banked, $Rn", []>,
                Requires<[IsARM, HasVirtualization]> {
  bits<6> banked;
  bits<4> Rn;
```
- EN: Defines TableGen record `MSRbanked` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MSRbanked`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6017-6026
```tablegen
  let Inst{23} = 0;
  let Inst{22} = banked{5}; // R bit
  let Inst{21-20} = 0b10;
  let Inst{19-16} = banked{3-0};
  let Inst{15-12} = 0b1111;
  let Inst{11-9} = 0b001;
  let Inst{8} = banked{4};
  let Inst{7-4} = 0b0000;
  let Inst{3-0} = Rn;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6028-6034
```tablegen
// Dynamic stack allocation yields a _chkstk for Windows targets.  These calls
// are needed to probe the stack when allocating more than
// 4k bytes in one go. Touching the stack at 4K increments is necessary to
// ensure that the guard pages used by the OS virtual memory manager are
// allocated in correct sequence.
// The main point of having separate instruction are extra unmodelled effects
// (compared to ordinary calls) like stack pointer change.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6036-6038
```tablegen
// Windows' __chkstk call to do stack probing.
def win__chkstk : SDNode<"ARMISD::WIN__CHKSTK", SDTNone,
                      [SDNPHasChain, SDNPSideEffect]>;
```
- EN: Defines TableGen record `win__chkstk` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `win__chkstk`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 6040-6041
```tablegen
let usesCustomInserter = 1, Uses = [R4], Defs = [R4, SP], hasNoSchedulingInfo = 1 in
  def WIN__CHKSTK : PseudoInst<(outs), (ins), NoItinerary, [(win__chkstk)]>;
```
- EN: Defines TableGen record `WIN__CHKSTK` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WIN__CHKSTK`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6043-6045
```tablegen
// Windows' divide by zero check
def win__dbzchk : SDNode<"ARMISD::WIN__DBZCHK", SDT_WIN__DBZCHK,
                         [SDNPHasChain, SDNPSideEffect, SDNPOutGlue]>;
```
- EN: Defines TableGen record `win__dbzchk` to model SelectionDAG node shapes, type profiles, or legalization metadata.
- CN: 这里定义 TableGen 记录 `win__dbzchk`，用于描述 SelectionDAG 节点形状、类型轮廓或合法化元数据。

### Lines 6047-6049
```tablegen
let usesCustomInserter = 1, Defs = [CPSR], hasNoSchedulingInfo = 1 in
  def WIN__DBZCHK : PseudoInst<(outs), (ins tGPR:$divisor), NoItinerary,
                               [(win__dbzchk tGPR:$divisor)]>;
```
- EN: Defines TableGen record `WIN__DBZCHK` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `WIN__DBZCHK`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6051-6053
```tablegen
//===----------------------------------------------------------------------===//
// TLS Instructions
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6055-6065
```tablegen
// __aeabi_read_tp preserves the registers r1-r3.
// This is a pseudo inst so that we can get the encoding right,
// complete with fixup for the aeabi_read_tp function.
// TPsoft is valid for ARM mode only, in case of Thumb mode a tTPsoft pattern
// is defined in "ARMInstrThumb.td".
let isCall = 1,
  Defs = [R0, R12, LR, CPSR], Uses = [SP] in {
  def TPsoft : ARMPseudoInst<(outs), (ins), 4, IIC_Br,
               [(set R0, ARMthread_pointer)]>, Sched<[WriteBr]>,
               Requires<[IsARM, IsReadTPSoft]>;
}
```
- EN: Defines TableGen record `TPsoft` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TPsoft`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6067-6073
```tablegen
// Reading thread pointer from coprocessor register
def : ARMPat<(ARMthread_pointer), (MRC 15, 0, 13, 0, 2)>,
      Requires<[IsARM, IsReadTPTPIDRURW]>;
def : ARMPat<(ARMthread_pointer), (MRC 15, 0, 13, 0, 3)>,
      Requires<[IsARM, IsReadTPTPIDRURO]>;
def : ARMPat<(ARMthread_pointer), (MRC 15, 0, 13, 0, 4)>,
      Requires<[IsARM, IsReadTPTPIDRPRW]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6075-6092
```tablegen
//===----------------------------------------------------------------------===//
// SJLJ Exception handling intrinsics
//   eh_sjlj_setjmp() is an instruction sequence to store the return
//   address and save #0 in R0 for the non-longjmp case.
//   Since by its nature we may be coming from some other function to get
//   here, and we're using the stack frame for the containing function to
//   save/restore registers, we can't keep anything live in regs across
//   the eh_sjlj_setjmp(), else it will almost certainly have been tromped upon
//   when we get here from a longjmp(). We force everything out of registers
//   except for our own input by listing the relevant registers in Defs. By
//   doing so, we also cause the prologue/epilogue code to actively preserve
//   all of the callee-saved registers, which is exactly what we want.
//   A constant value is passed in $val, and we use the location as a scratch.
//
// These are pseudo-instructions and are lowered to individual MC-insts, so
// no encoding information is necessary.
// This gets lowered to an instruction sequence of 20 bytes
let Defs =
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6093-6100
```tablegen
  [ R0,  R1,  R2,  R3,  R4,  R5,  R6,  R7,  R8,  R9,  R10, R11, R12, LR, CPSR,
    Q0, Q1, Q2, Q3, Q4, Q5, Q6, Q7, Q8, Q9, Q10, Q11, Q12, Q13, Q14, Q15 ],
  hasSideEffects = 1, isBarrier = 1, usesCustomInserter = 1, Size = 20 in {
  def Int_eh_sjlj_setjmp : PseudoInst<(outs), (ins GPR:$src, GPR:$val),
                               NoItinerary,
                         [(set R0, (ARMeh_sjlj_setjmp GPR:$src, GPR:$val))]>,
                           Requires<[IsARM, HasVFP2]>;
}
```
- EN: Defines TableGen record `Int_eh_sjlj_setjmp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Int_eh_sjlj_setjmp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6102-6110
```tablegen
// This gets lowered to an instruction sequence of 20 bytes
let Defs =
  [ R0,  R1,  R2,  R3,  R4,  R5,  R6,  R7,  R8,  R9,  R10, R11, R12, LR, CPSR ],
  hasSideEffects = 1, isBarrier = 1, usesCustomInserter = 1, Size = 20 in {
  def Int_eh_sjlj_setjmp_nofp : PseudoInst<(outs), (ins GPR:$src, GPR:$val),
                                   NoItinerary,
                         [(set R0, (ARMeh_sjlj_setjmp GPR:$src, GPR:$val))]>,
                                Requires<[IsARM, NoVFP]>;
}
```
- EN: Defines TableGen record `Int_eh_sjlj_setjmp_nofp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Int_eh_sjlj_setjmp_nofp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6112-6120
```tablegen
// This gets lowered to an instruction sequence of 16 bytes
// FIXME: Non-IOS version(s)
let isBarrier = 1, hasSideEffects = 1, isTerminator = 1, Size = 16,
    Defs = [ R7, LR, SP ] in {
def Int_eh_sjlj_longjmp : PseudoInst<(outs), (ins GPR:$src, GPR:$scratch),
                             NoItinerary,
                         [(ARMeh_sjlj_longjmp GPR:$src, GPR:$scratch)]>,
                                Requires<[IsARM]>;
}
```
- EN: Defines TableGen record `Int_eh_sjlj_longjmp` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Int_eh_sjlj_longjmp`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6122-6124
```tablegen
let isBarrier = 1, hasSideEffects = 1, usesCustomInserter = 1 in
def Int_eh_sjlj_setup_dispatch : PseudoInst<(outs), (ins), NoItinerary,
            [(ARMeh_sjlj_setup_dispatch)]>;
```
- EN: Defines TableGen record `Int_eh_sjlj_setup_dispatch` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Int_eh_sjlj_setup_dispatch`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6126-6131
```tablegen
// eh.sjlj.dispatchsetup pseudo-instruction.
// This pseudo is used for both ARM and Thumb. Any differences are handled when
// the pseudo is expanded (which happens before any passes that need the
// instruction size).
let isBarrier = 1 in
def Int_eh_sjlj_dispatchsetup : PseudoInst<(outs), (ins), NoItinerary, []>;
```
- EN: Defines TableGen record `Int_eh_sjlj_dispatchsetup` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `Int_eh_sjlj_dispatchsetup`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6134-6136
```tablegen
//===----------------------------------------------------------------------===//
// Non-Instruction Patterns
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6138-6143
```tablegen
// ARMv4 indirect branch using (MOVr PC, dst)
let isBranch = 1, isTerminator = 1, isBarrier = 1, isIndirectBranch = 1 in
  def MOVPCRX : ARMPseudoExpand<(outs), (ins GPR:$dst),
                    4, IIC_Br, [(brind GPR:$dst)],
                    (MOVr PC, GPR:$dst, (ops 14, zero_reg), zero_reg)>,
                  Requires<[IsARM, NoV4T]>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `MOVPCRX` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVPCRX`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6145-6149
```tablegen
let isCall = 1, isTerminator = 1, isReturn = 1, isBarrier = 1, Uses = [SP] in
  def TAILJMPr4 : ARMPseudoExpand<(outs), (ins GPR:$dst),
                    4, IIC_Br, [],
                    (MOVr PC, GPR:$dst, (ops 14, zero_reg), zero_reg)>,
                  Requires<[IsARM, NoV4T]>, Sched<[WriteBr]>;
```
- EN: Defines TableGen record `TAILJMPr4` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `TAILJMPr4`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6151-6151
```tablegen
// Large immediate handling.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6153-6160
```tablegen
// 32-bit immediate using two piece mod_imms or movw + movt.
// This is a single pseudo instruction, the benefit is that it can be remat'd
// as a single unit instead of having to handle reg inputs.
// FIXME: Remove this when we can do generalized remat.
let isReMaterializable = 1, isMoveImm = 1, Size = 8 in
def MOVi32imm : PseudoInst<(outs GPR:$dst), (ins i32imm:$src), IIC_iMOVix2,
                           [(set GPR:$dst, (arm_i32imm:$src))]>,
                           Requires<[IsARM]>;
```
- EN: Defines TableGen record `MOVi32imm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOVi32imm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6162-6164
```tablegen
def LDRLIT_ga_abs : PseudoInst<(outs GPR:$dst), (ins i32imm:$src), IIC_iLoad_i,
                               [(set GPR:$dst, (ARMWrapper tglobaladdr:$src))]>,
                    Requires<[IsARM, DontUseMovt]>;
```
- EN: Defines TableGen record `LDRLIT_ga_abs` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRLIT_ga_abs`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6166-6174
```tablegen
// Pseudo instruction that combines movw + movt + add pc (if PIC).
// It also makes it possible to rematerialize the instructions.
// FIXME: Remove this when we can do generalized remat and when machine licm
// can properly the instructions.
let isReMaterializable = 1 in {
def MOV_ga_pcrel : PseudoInst<(outs GPR:$dst), (ins i32imm:$addr),
                              IIC_iMOVix2addpc,
                        [(set GPR:$dst, (ARMWrapperPIC tglobaladdr:$addr))]>,
                        Requires<[IsARM, UseMovtInPic]>;
```
- EN: Defines TableGen record `MOV_ga_pcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOV_ga_pcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6176-6180
```tablegen
def LDRLIT_ga_pcrel : PseudoInst<(outs GPR:$dst), (ins i32imm:$addr),
                                 IIC_iLoadiALU,
                                 [(set GPR:$dst,
                                       (ARMWrapperPIC tglobaladdr:$addr))]>,
                      Requires<[IsARM, DontUseMovtInPic]>;
```
- EN: Defines TableGen record `LDRLIT_ga_pcrel` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRLIT_ga_pcrel`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6182-6187
```tablegen
let AddedComplexity = 10 in
def LDRLIT_ga_pcrel_ldr : PseudoInst<(outs GPR:$dst), (ins i32imm:$addr),
                              NoItinerary,
                              [(set GPR:$dst,
                                    (load (ARMWrapperPIC tglobaladdr:$addr)))]>,
                          Requires<[IsARM, DontUseMovtInPic]>;
```
- EN: Defines TableGen record `LDRLIT_ga_pcrel_ldr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `LDRLIT_ga_pcrel_ldr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6189-6194
```tablegen
let AddedComplexity = 10 in
def MOV_ga_pcrel_ldr : PseudoInst<(outs GPR:$dst), (ins i32imm:$addr),
                                IIC_iMOVix2ld,
                    [(set GPR:$dst, (load (ARMWrapperPIC tglobaladdr:$addr)))]>,
                    Requires<[IsARM, UseMovtInPic]>;
} // isReMaterializable
```
- EN: Defines TableGen record `MOV_ga_pcrel_ldr` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `MOV_ga_pcrel_ldr`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6196-6199
```tablegen
// The many different faces of TLS access.
def : ARMPat<(ARMWrapper tglobaltlsaddr :$dst),
             (MOVi32imm tglobaltlsaddr :$dst)>,
      Requires<[IsARM, UseMovt]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6201-6203
```tablegen
def : Pat<(ARMWrapper tglobaltlsaddr:$src),
          (LDRLIT_ga_abs tglobaltlsaddr:$src)>,
      Requires<[IsARM, DontUseMovt]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6205-6206
```tablegen
def : Pat<(ARMWrapperPIC tglobaltlsaddr:$addr),
          (MOV_ga_pcrel tglobaltlsaddr:$addr)>, Requires<[IsARM, UseMovtInPic]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6208-6214
```tablegen
def : Pat<(ARMWrapperPIC tglobaltlsaddr:$addr),
          (LDRLIT_ga_pcrel tglobaltlsaddr:$addr)>,
      Requires<[IsARM, DontUseMovtInPic]>;
let AddedComplexity = 10 in
def : Pat<(load (ARMWrapperPIC tglobaltlsaddr:$addr)),
          (MOV_ga_pcrel_ldr tglobaltlsaddr:$addr)>,
      Requires<[IsARM, UseMovtInPic]>;
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6217-6224
```tablegen
// ConstantPool, GlobalAddress, and JumpTable
def : ARMPat<(ARMWrapper  tconstpool  :$dst), (LEApcrel tconstpool  :$dst)>;
def : ARMPat<(ARMWrapper  tglobaladdr :$dst), (MOVi32imm tglobaladdr :$dst)>,
            Requires<[IsARM, UseMovt]>;
def : ARMPat<(ARMWrapper texternalsym :$dst), (MOVi32imm texternalsym :$dst)>,
            Requires<[IsARM, UseMovt]>;
def : ARMPat<(ARMWrapperJT tjumptable:$dst),
             (LEApcrelJT tjumptable:$dst)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6226-6226
```tablegen
// TODO: add,sub,and, 3-instr forms?
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6228-6240
```tablegen
// Tail calls. These patterns also apply to Thumb mode.
// Regular indirect tail call
def : Pat<(ARMtcret tcGPR:$dst, (i32 timm:$SPDiff)),
          (TCRETURNri tcGPR:$dst, timm:$SPDiff)>,
          Requires<[NoSignRetAddr]>;
// Indirect tail call when PACBTI is enabled
def : Pat<(ARMtcret tcGPRnotr12:$dst, (i32 timm:$SPDiff)),
          (TCRETURNrinotr12 tcGPRnotr12:$dst, timm:$SPDiff)>,
          Requires<[SignRetAddr]>;
def : Pat<(ARMtcret (i32 tglobaladdr:$dst), (i32 timm:$SPDiff)),
          (TCRETURNdi texternalsym:$dst, (i32 timm:$SPDiff))>;
def : Pat<(ARMtcret (i32 texternalsym:$dst), (i32 timm:$SPDiff)),
          (TCRETURNdi texternalsym:$dst, i32imm:$SPDiff)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6242-6245
```tablegen
// Direct calls
def : ARMPat<(ARMcall texternalsym:$func), (BL texternalsym:$func)>;
def : ARMPat<(ARMcall_nolink texternalsym:$func),
             (BMOVPCB_CALL texternalsym:$func)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6247-6249
```tablegen
// zextload i1 -> zextload i8
def : ARMPat<(zextloadi1 addrmode_imm12:$addr), (LDRBi12 addrmode_imm12:$addr)>;
def : ARMPat<(zextloadi1 ldst_so_reg:$addr),    (LDRBrs ldst_so_reg:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6251-6255
```tablegen
// extload -> zextload
def : ARMPat<(extloadi1 addrmode_imm12:$addr),  (LDRBi12 addrmode_imm12:$addr)>;
def : ARMPat<(extloadi1 ldst_so_reg:$addr),     (LDRBrs ldst_so_reg:$addr)>;
def : ARMPat<(extloadi8 addrmode_imm12:$addr),  (LDRBi12 addrmode_imm12:$addr)>;
def : ARMPat<(extloadi8 ldst_so_reg:$addr),     (LDRBrs ldst_so_reg:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6257-6257
```tablegen
def : ARMPat<(extloadi16 addrmode3:$addr),  (LDRH addrmode3:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6259-6260
```tablegen
def : ARMPat<(extloadi8  addrmodepc:$addr), (PICLDRB addrmodepc:$addr)>;
def : ARMPat<(extloadi16 addrmodepc:$addr), (PICLDRH addrmodepc:$addr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6262-6278
```tablegen
// smul* and smla*
def : ARMV5TEPat<(mul sext_16_node:$a, sext_16_node:$b),
                 (SMULBB GPR:$a, GPR:$b)>;
def : ARMV5TEPat<(mul sext_16_node:$a, (sext_bottom_16 GPR:$b)),
                 (SMULBB GPR:$a, GPR:$b)>;
def : ARMV5TEPat<(mul sext_16_node:$a, (sext_top_16 GPR:$b)),
                 (SMULBT GPR:$a, GPR:$b)>;
def : ARMV5TEPat<(mul (sext_top_16 GPR:$a), sext_16_node:$b),
                 (SMULTB GPR:$a, GPR:$b)>;
def : ARMV5MOPat<(add GPR:$acc, (mul sext_16_node:$a, sext_16_node:$b)),
                 (SMLABB GPR:$a, GPR:$b, GPR:$acc)>;
def : ARMV5MOPat<(add GPR:$acc, (mul sext_16_node:$a, (sext_bottom_16 GPR:$b))),
                 (SMLABB GPR:$a, GPR:$b, GPR:$acc)>;
def : ARMV5MOPat<(add GPR:$acc, (mul sext_16_node:$a, (sext_top_16 GPR:$b))),
                 (SMLABT GPR:$a, GPR:$b, GPR:$acc)>;
def : ARMV5MOPat<(add GPR:$acc, (mul (sext_top_16 GPR:$a), sext_16_node:$b)),
                 (SMLATB GPR:$a, GPR:$b, GPR:$acc)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6280-6291
```tablegen
def : ARMV5TEPat<(int_arm_smulbb GPR:$a, GPR:$b),
                 (SMULBB GPR:$a, GPR:$b)>;
def : ARMV5TEPat<(int_arm_smulbt GPR:$a, GPR:$b),
                 (SMULBT GPR:$a, GPR:$b)>;
def : ARMV5TEPat<(int_arm_smultb GPR:$a, GPR:$b),
                 (SMULTB GPR:$a, GPR:$b)>;
def : ARMV5TEPat<(int_arm_smultt GPR:$a, GPR:$b),
                 (SMULTT GPR:$a, GPR:$b)>;
def : ARMV5TEPat<(int_arm_smulwb GPR:$a, GPR:$b),
                 (SMULWB GPR:$a, GPR:$b)>;
def : ARMV5TEPat<(int_arm_smulwt GPR:$a, GPR:$b),
                 (SMULWT GPR:$a, GPR:$b)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6293-6304
```tablegen
def : ARMV5TEPat<(int_arm_smlabb GPR:$a, GPR:$b, GPR:$acc),
                 (SMLABB GPR:$a, GPR:$b, GPR:$acc)>;
def : ARMV5TEPat<(int_arm_smlabt GPR:$a, GPR:$b, GPR:$acc),
                 (SMLABT GPR:$a, GPR:$b, GPR:$acc)>;
def : ARMV5TEPat<(int_arm_smlatb GPR:$a, GPR:$b, GPR:$acc),
                 (SMLATB GPR:$a, GPR:$b, GPR:$acc)>;
def : ARMV5TEPat<(int_arm_smlatt GPR:$a, GPR:$b, GPR:$acc),
                 (SMLATT GPR:$a, GPR:$b, GPR:$acc)>;
def : ARMV5TEPat<(int_arm_smlawb GPR:$a, GPR:$b, GPR:$acc),
                 (SMLAWB GPR:$a, GPR:$b, GPR:$acc)>;
def : ARMV5TEPat<(int_arm_smlawt GPR:$a, GPR:$b, GPR:$acc),
                 (SMLAWT GPR:$a, GPR:$b, GPR:$acc)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6306-6308
```tablegen
// Pre-v7 uses MCR for synchronization barriers.
def : ARMPat<(ARMMemBarrierMCR GPR:$zero), (MCR 15, 0, GPR:$zero, 7, 10, 5)>,
         Requires<[IsARM, HasV6]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6310-6319
```tablegen
// SXT/UXT with no rotate
let AddedComplexity = 16 in {
def : ARMV6Pat<(and GPR:$Src, 0x000000FF), (UXTB GPR:$Src, 0)>;
def : ARMV6Pat<(and GPR:$Src, 0x0000FFFF), (UXTH GPR:$Src, 0)>;
def : ARMV6Pat<(and GPR:$Src, 0x00FF00FF), (UXTB16 GPR:$Src, 0)>;
def : ARMV6Pat<(add GPR:$Rn, (and GPR:$Rm, 0x00FF)),
               (UXTAB GPR:$Rn, GPR:$Rm, 0)>;
def : ARMV6Pat<(add GPR:$Rn, (and GPR:$Rm, 0xFFFF)),
               (UXTAH GPR:$Rn, GPR:$Rm, 0)>;
}
```
- EN: Assigns TableGen fields and encoding bits, tightening how generated instructions, predicates, or flags are materialized.
- CN: 这里设置 TableGen 字段和编码位，从而细化生成后的指令、谓词或标志如何落地。

### Lines 6321-6322
```tablegen
def : ARMV6Pat<(sext_inreg GPR:$Src, i8),  (SXTB GPR:$Src, 0)>;
def : ARMV6Pat<(sext_inreg GPR:$Src, i16), (SXTH GPR:$Src, 0)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6324-6327
```tablegen
def : ARMV6Pat<(add GPR:$Rn, (sext_inreg GPRnopc:$Rm, i8)),
               (SXTAB GPR:$Rn, GPRnopc:$Rm, 0)>;
def : ARMV6Pat<(add GPR:$Rn, (sext_inreg GPRnopc:$Rm, i16)),
               (SXTAH GPR:$Rn, GPRnopc:$Rm, 0)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6329-6346
```tablegen
// Atomic load/store patterns
def : ARMPat<(atomic_load_azext_8 ldst_so_reg:$src),
             (LDRBrs ldst_so_reg:$src)>;
def : ARMPat<(atomic_load_azext_8 addrmode_imm12:$src),
             (LDRBi12 addrmode_imm12:$src)>;
def : ARMPat<(atomic_load_azext_16 addrmode3:$src),
             (LDRH addrmode3:$src)>;
def : ARMPat<(atomic_load_nonext_32 ldst_so_reg:$src),
             (LDRrs ldst_so_reg:$src)>;
def : ARMPat<(atomic_load_nonext_32 addrmode_imm12:$src),
             (LDRi12 addrmode_imm12:$src)>;
def : ARMPat<(atomic_store_8 GPR:$val, ldst_so_reg:$ptr),
             (STRBrs GPR:$val, ldst_so_reg:$ptr)>;
def : ARMPat<(atomic_store_8 GPR:$val, addrmode_imm12:$ptr),
             (STRBi12 GPR:$val, addrmode_imm12:$ptr)>;
def : ARMPat<(atomic_store_16 GPR:$val, addrmode3:$ptr),
             (STRH GPR:$val, addrmode3:$ptr)>;
def : ARMPat<(atomic_store_32 GPR:$val, ldst_so_reg:$ptr),
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6347-6349
```tablegen
             (STRrs GPR:$val, ldst_so_reg:$ptr)>;
def : ARMPat<(atomic_store_32 GPR:$val, addrmode_imm12:$ptr),
             (STRi12 GPR:$val, addrmode_imm12:$ptr)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6352-6354
```tablegen
//===----------------------------------------------------------------------===//
// Thumb Support
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6356-6356
```tablegen
include "ARMInstrThumb.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 6358-6360
```tablegen
//===----------------------------------------------------------------------===//
// Thumb2 Support
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6362-6362
```tablegen
include "ARMInstrThumb2.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 6364-6366
```tablegen
//===----------------------------------------------------------------------===//
// Floating Point Support
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6368-6368
```tablegen
include "ARMInstrVFP.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 6370-6372
```tablegen
//===----------------------------------------------------------------------===//
// Advanced SIMD (NEON) Support
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6374-6374
```tablegen
include "ARMInstrNEON.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 6376-6378
```tablegen
//===----------------------------------------------------------------------===//
// MVE Support
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6380-6380
```tablegen
include "ARMInstrMVE.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 6382-6384
```tablegen
//===----------------------------------------------------------------------===//
// CDE (Custom Datapath Extension)
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6386-6386
```tablegen
include "ARMInstrCDE.td"
```
- EN: Pulls in other TableGen fragments so this file can reuse common records, predicates, and target-wide definitions.
- CN: 这里引入其他 TableGen 片段，以便复用公共记录、谓词以及整个目标范围内的定义。

### Lines 6388-6390
```tablegen
//===----------------------------------------------------------------------===//
// Assembler aliases
//
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6392-6399
```tablegen
// Memory barriers
def : InstAlias<"dmb", (DMB 0xf), 0>, Requires<[IsARM, HasDB]>;
def : InstAlias<"dsb", (DSB 0xf), 0>, Requires<[IsARM, HasDB]>;
def : InstAlias<"ssbb", (DSB 0x0), 1>, Requires<[IsARM, HasDB]>;
def : InstAlias<"pssbb", (DSB 0x4), 1>, Requires<[IsARM, HasDB]>;
def : InstAlias<"isb", (ISB 0xf), 0>, Requires<[IsARM, HasDB]>;
// Armv8-R 'Data Full Barrier'
def : InstAlias<"dfb", (DSB 0xc), 1>, Requires<[IsARM, HasDFB]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6401-6402
```tablegen
// System instructions
def : MnemonicAlias<"swi", "svc">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6404-6410
```tablegen
// Load / Store Multiple
def : MnemonicAlias<"ldmfd", "ldm">;
def : MnemonicAlias<"ldmia", "ldm">;
def : MnemonicAlias<"ldmea", "ldmdb">;
def : MnemonicAlias<"stmfd", "stmdb">;
def : MnemonicAlias<"stmia", "stm">;
def : MnemonicAlias<"stmea", "stm">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6412-6419
```tablegen
// PKHBT/PKHTB with default shift amount. PKHTB is equivalent to PKHBT with the
// input operands swapped when the shift amount is zero (i.e., unspecified).
def : InstAlias<"pkhbt${p} $Rd, $Rn, $Rm",
                (PKHBT GPRnopc:$Rd, GPRnopc:$Rn, GPRnopc:$Rm, 0, pred:$p), 0>,
        Requires<[IsARM, HasV6]>;
def : InstAlias<"pkhtb${p} $Rd, $Rn, $Rm",
                (PKHBT GPRnopc:$Rd, GPRnopc:$Rm, GPRnopc:$Rn, 0, pred:$p), 0>,
        Requires<[IsARM, HasV6]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6421-6423
```tablegen
// PUSH/POP aliases for STM/LDM
def : ARMInstAlias<"push${p} $regs", (STMDB_UPD SP, pred:$p, reglist:$regs)>;
def : ARMInstAlias<"pop${p} $regs", (LDMIA_UPD SP, pred:$p, reglist:$regs)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6425-6429
```tablegen
// SSAT/USAT optional shift operand.
def : ARMInstAlias<"ssat${p} $Rd, $sat_imm, $Rn",
                (SSAT GPRnopc:$Rd, imm1_32:$sat_imm, GPRnopc:$Rn, 0, pred:$p)>;
def : ARMInstAlias<"usat${p} $Rd, $sat_imm, $Rn",
                (USAT GPRnopc:$Rd, imm0_31:$sat_imm, GPRnopc:$Rn, 0, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6432-6444
```tablegen
// Extend instruction optional rotate operand.
def : ARMInstAlias<"sxtab${p} $Rd, $Rn, $Rm",
                (SXTAB GPRnopc:$Rd, GPR:$Rn, GPRnopc:$Rm, 0, pred:$p)>;
def : ARMInstAlias<"sxtah${p} $Rd, $Rn, $Rm",
                (SXTAH GPRnopc:$Rd, GPR:$Rn, GPRnopc:$Rm, 0, pred:$p)>;
def : ARMInstAlias<"sxtab16${p} $Rd, $Rn, $Rm",
                (SXTAB16 GPRnopc:$Rd, GPR:$Rn, GPRnopc:$Rm, 0, pred:$p)>;
def : ARMInstAlias<"sxtb${p} $Rd, $Rm",
                (SXTB GPRnopc:$Rd, GPRnopc:$Rm, 0, pred:$p)>;
def : ARMInstAlias<"sxtb16${p} $Rd, $Rm",
                (SXTB16 GPRnopc:$Rd, GPRnopc:$Rm, 0, pred:$p)>;
def : ARMInstAlias<"sxth${p} $Rd, $Rm",
                (SXTH GPRnopc:$Rd, GPRnopc:$Rm, 0, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6446-6457
```tablegen
def : ARMInstAlias<"uxtab${p} $Rd, $Rn, $Rm",
                (UXTAB GPRnopc:$Rd, GPR:$Rn, GPRnopc:$Rm, 0, pred:$p)>;
def : ARMInstAlias<"uxtah${p} $Rd, $Rn, $Rm",
                (UXTAH GPRnopc:$Rd, GPR:$Rn, GPRnopc:$Rm, 0, pred:$p)>;
def : ARMInstAlias<"uxtab16${p} $Rd, $Rn, $Rm",
                (UXTAB16 GPRnopc:$Rd, GPR:$Rn, GPRnopc:$Rm, 0, pred:$p)>;
def : ARMInstAlias<"uxtb${p} $Rd, $Rm",
                (UXTB GPRnopc:$Rd, GPRnopc:$Rm, 0, pred:$p)>;
def : ARMInstAlias<"uxtb16${p} $Rd, $Rm",
                (UXTB16 GPRnopc:$Rd, GPRnopc:$Rm, 0, pred:$p)>;
def : ARMInstAlias<"uxth${p} $Rd, $Rm",
                (UXTH GPRnopc:$Rd, GPRnopc:$Rm, 0, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6460-6465
```tablegen
// RFE aliases
def : MnemonicAlias<"rfefa", "rfeda">;
def : MnemonicAlias<"rfeea", "rfedb">;
def : MnemonicAlias<"rfefd", "rfeia">;
def : MnemonicAlias<"rfeed", "rfeib">;
def : MnemonicAlias<"rfe", "rfeia">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6467-6472
```tablegen
// SRS aliases
def : MnemonicAlias<"srsfa", "srsib">;
def : MnemonicAlias<"srsea", "srsia">;
def : MnemonicAlias<"srsfd", "srsdb">;
def : MnemonicAlias<"srsed", "srsda">;
def : MnemonicAlias<"srs", "srsia">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6474-6491
```tablegen
// QSAX == QSUBADDX
def : MnemonicAlias<"qsubaddx", "qsax">;
// SASX == SADDSUBX
def : MnemonicAlias<"saddsubx", "sasx">;
// SHASX == SHADDSUBX
def : MnemonicAlias<"shaddsubx", "shasx">;
// SHSAX == SHSUBADDX
def : MnemonicAlias<"shsubaddx", "shsax">;
// SSAX == SSUBADDX
def : MnemonicAlias<"ssubaddx", "ssax">;
// UASX == UADDSUBX
def : MnemonicAlias<"uaddsubx", "uasx">;
// UHASX == UHADDSUBX
def : MnemonicAlias<"uhaddsubx", "uhasx">;
// UHSAX == UHSUBADDX
def : MnemonicAlias<"uhsubaddx", "uhsax">;
// UQASX == UQADDSUBX
def : MnemonicAlias<"uqaddsubx", "uqasx">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6492-6495
```tablegen
// UQSAX == UQSUBADDX
def : MnemonicAlias<"uqsubaddx", "uqsax">;
// USAX == USUBADDX
def : MnemonicAlias<"usubaddx", "usax">;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6497-6514
```tablegen
// "mov Rd, mod_imm_not" can be handled via "mvn" in assembly, just like
// for isel.
def : ARMInstSubst<"mov${s}${p} $Rd, $imm",
                   (MVNi rGPR:$Rd, mod_imm_not:$imm, pred:$p, cc_out:$s)>;
def : ARMInstSubst<"mvn${s}${p} $Rd, $imm",
                   (MOVi rGPR:$Rd, mod_imm_not:$imm, pred:$p, cc_out:$s)>;
// Same for AND <--> BIC
def : ARMInstSubst<"bic${s}${p} $Rd, $Rn, $imm",
                   (ANDri GPR:$Rd, GPR:$Rn, mod_imm_not:$imm,
                          pred:$p, cc_out:$s)>;
def : ARMInstSubst<"bic${s}${p} $Rdn, $imm",
                   (ANDri GPR:$Rdn, GPR:$Rdn, mod_imm_not:$imm,
                          pred:$p, cc_out:$s)>;
def : ARMInstSubst<"and${s}${p} $Rd, $Rn, $imm",
                   (BICri GPR:$Rd, GPR:$Rn, mod_imm_not:$imm,
                          pred:$p, cc_out:$s)>;
def : ARMInstSubst<"and${s}${p} $Rdn, $imm",
                   (BICri GPR:$Rdn, GPR:$Rdn, mod_imm_not:$imm,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6515-6515
```tablegen
                          pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6517-6526
```tablegen
// Likewise, "add Rd, mod_imm_neg" -> sub
def : ARMInstSubst<"add${s}${p} $Rd, $Rn, $imm",
                 (SUBri GPR:$Rd, GPR:$Rn, mod_imm_neg:$imm, pred:$p, cc_out:$s)>;
def : ARMInstSubst<"add${s}${p} $Rd, $imm",
                 (SUBri GPR:$Rd, GPR:$Rd, mod_imm_neg:$imm, pred:$p, cc_out:$s)>;
// Likewise, "sub Rd, mod_imm_neg" -> add
def : ARMInstSubst<"sub${s}${p} $Rd, $Rn, $imm",
                 (ADDri GPR:$Rd, GPR:$Rn, mod_imm_neg:$imm, pred:$p, cc_out:$s)>;
def : ARMInstSubst<"sub${s}${p} $Rd, $imm",
                 (ADDri GPR:$Rd, GPR:$Rd, mod_imm_neg:$imm, pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6529-6536
```tablegen
def : ARMInstSubst<"adc${s}${p} $Rd, $Rn, $imm",
                 (SBCri GPR:$Rd, GPR:$Rn, mod_imm_not:$imm, pred:$p, cc_out:$s)>;
def : ARMInstSubst<"adc${s}${p} $Rdn, $imm",
                 (SBCri GPR:$Rdn, GPR:$Rdn, mod_imm_not:$imm, pred:$p, cc_out:$s)>;
def : ARMInstSubst<"sbc${s}${p} $Rd, $Rn, $imm",
                 (ADCri GPR:$Rd, GPR:$Rn, mod_imm_not:$imm, pred:$p, cc_out:$s)>;
def : ARMInstSubst<"sbc${s}${p} $Rdn, $imm",
                 (ADCri GPR:$Rdn, GPR:$Rdn, mod_imm_not:$imm, pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6538-6542
```tablegen
// Same for CMP <--> CMN via mod_imm_neg
def : ARMInstSubst<"cmp${p} $Rd, $imm",
                   (CMNri rGPR:$Rd, mod_imm_neg:$imm, pred:$p)>;
def : ARMInstSubst<"cmn${p} $Rd, $imm",
                   (CMPri rGPR:$Rd, mod_imm_neg:$imm, pred:$p)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6544-6561
```tablegen
// The shifter forms of the MOV instruction are aliased to the ASR, LSL,
// LSR, ROR, and RRX instructions.
// FIXME: We need C++ parser hooks to map the alias to the MOV
//        encoding. It seems we should be able to do that sort of thing
//        in tblgen, but it could get ugly.
let TwoOperandAliasConstraint = "$Rm = $Rd" in {
def ASRi : ARMAsmPseudo<"asr${s}${p} $Rd, $Rm, $imm",
                        (ins GPR:$Rd, GPR:$Rm, imm0_32:$imm, pred:$p,
                             cc_out:$s)>;
def LSRi : ARMAsmPseudo<"lsr${s}${p} $Rd, $Rm, $imm",
                        (ins GPR:$Rd, GPR:$Rm, imm0_32:$imm, pred:$p,
                             cc_out:$s)>;
def LSLi : ARMAsmPseudo<"lsl${s}${p} $Rd, $Rm, $imm",
                        (ins GPR:$Rd, GPR:$Rm, imm0_31:$imm, pred:$p,
                             cc_out:$s)>;
def RORi : ARMAsmPseudo<"ror${s}${p} $Rd, $Rm, $imm",
                        (ins GPR:$Rd, GPR:$Rm, imm0_31:$imm, pred:$p,
                             cc_out:$s)>;
```
- EN: Defines TableGen record `ASRi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ASRi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6562-6578
```tablegen
}
def RRXi : ARMAsmPseudo<"rrx${s}${p} $Rd, $Rm",
                        (ins GPR:$Rd, GPR:$Rm, pred:$p, cc_out:$s)>;
let TwoOperandAliasConstraint = "$Rn = $Rd" in {
def ASRr : ARMAsmPseudo<"asr${s}${p} $Rd, $Rn, $Rm",
                        (ins GPRnopc:$Rd, GPRnopc:$Rn, GPRnopc:$Rm, pred:$p,
                             cc_out:$s)>;
def LSRr : ARMAsmPseudo<"lsr${s}${p} $Rd, $Rn, $Rm",
                        (ins GPRnopc:$Rd, GPRnopc:$Rn, GPRnopc:$Rm, pred:$p,
                             cc_out:$s)>;
def LSLr : ARMAsmPseudo<"lsl${s}${p} $Rd, $Rn, $Rm",
                        (ins GPRnopc:$Rd, GPRnopc:$Rn, GPRnopc:$Rm, pred:$p,
                             cc_out:$s)>;
def RORr : ARMAsmPseudo<"ror${s}${p} $Rd, $Rn, $Rm",
                        (ins GPRnopc:$Rd, GPRnopc:$Rn, GPRnopc:$Rm, pred:$p,
                             cc_out:$s)>;
}
```
- EN: Defines TableGen record `RRXi` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `RRXi`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6580-6582
```tablegen
// "neg" is and alias for "rsb rd, rn, #0"
def : ARMInstAlias<"neg${s}${p} $Rd, $Rm",
                   (RSBri GPR:$Rd, GPR:$Rm, 0, pred:$p, cc_out:$s)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6584-6586
```tablegen
// Pre-ARMv6K (including ARMv6), 'mov r0, r0' is the NOP encoding in ARM state.
def : InstAlias<"nop${p}", (MOVr R0, R0, pred:$p, (cc_out zero_reg)), 0>,
         Requires<[IsARM, NoV6K]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6588-6605
```tablegen
// MUL/UMLAL/SMLAL/UMULL/SMULL are available on all arches, but
// the instruction definitions need difference constraints pre-v6.
// Use these aliases for the assembly parsing on pre-v6.
def : InstAlias<"mul${s}${p} $Rd, $Rn, $Rm",
            (MUL GPRnopc:$Rd, GPRnopc:$Rn, GPRnopc:$Rm, pred:$p, cc_out:$s), 0>,
         Requires<[IsARM, NoV6]>;
def : InstAlias<"mla${s}${p} $Rd, $Rn, $Rm, $Ra",
            (MLA GPRnopc:$Rd, GPRnopc:$Rn, GPRnopc:$Rm, GPRnopc:$Ra,
             pred:$p, cc_out:$s), 0>,
         Requires<[IsARM, NoV6]>;
def : InstAlias<"smlal${s}${p} $RdLo, $RdHi, $Rn, $Rm",
            (SMLAL GPR:$RdLo, GPR:$RdHi, GPR:$Rn, GPR:$Rm, pred:$p, cc_out:$s), 0>,
         Requires<[IsARM, NoV6]>;
def : InstAlias<"umlal${s}${p} $RdLo, $RdHi, $Rn, $Rm",
            (UMLAL GPR:$RdLo, GPR:$RdHi, GPR:$Rn, GPR:$Rm, pred:$p, cc_out:$s), 0>,
         Requires<[IsARM, NoV6]>;
def : InstAlias<"smull${s}${p} $RdLo, $RdHi, $Rn, $Rm",
            (SMULL GPR:$RdLo, GPR:$RdHi, GPR:$Rn, GPR:$Rm, pred:$p, cc_out:$s), 0>,
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6606-6609
```tablegen
         Requires<[IsARM, NoV6]>;
def : InstAlias<"umull${s}${p} $RdLo, $RdHi, $Rn, $Rm",
            (UMULL GPR:$RdLo, GPR:$RdHi, GPR:$Rn, GPR:$Rm, pred:$p, cc_out:$s), 0>,
         Requires<[IsARM, NoV6]>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6611-6613
```tablegen
// 'it' blocks in ARM mode just validate the predicates. The IT itself
// is discarded.
def ITasm : ARMAsmPseudo<"it$mask $cc", (ins it_pred:$cc, it_mask:$mask)>;
```
- EN: Defines TableGen record `ITasm` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `ITasm`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6615-6618
```tablegen
let mayLoad = 1, mayStore =1, hasSideEffects = 1, hasNoSchedulingInfo = 1 in
def SPACE : PseudoInst<(outs GPR:$Rd), (ins i32imm:$size, GPR:$Rn),
                       NoItinerary,
                       [(set GPR:$Rd, (int_arm_space timm:$size, GPR:$Rn))]>;
```
- EN: Defines TableGen record `SPACE` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SPACE`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6620-6631
```tablegen
// SpeculationBarrierEndBB must only be used after an unconditional control
// flow, i.e. after a terminator for which isBarrier is True.
let hasSideEffects = 1, isCodeGenOnly = 1, isTerminator = 1, isBarrier = 1 in {
  // This gets lowered to a pair of 4-byte instructions
  let Size = 8 in
  def SpeculationBarrierISBDSBEndBB
      : PseudoInst<(outs), (ins), NoItinerary, []>, Sched<[]>;
  // This gets lowered to a single 4-byte instructions
  let Size = 4 in
  def SpeculationBarrierSBEndBB
      : PseudoInst<(outs), (ins), NoItinerary, []>, Sched<[]>;
}
```
- EN: Defines TableGen record `SpeculationBarrierISBDSBEndBB` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SpeculationBarrierISBDSBEndBB`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6633-6635
```tablegen
//===----------------------------------
// Atomic cmpxchg for -O0
//===----------------------------------
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6637-6640
```tablegen
// The fast register allocator used during -O0 inserts spills to cover any VRegs
// live across basic block boundaries. When this happens between an LDXR and an
// STXR it can clear the exclusive monitor, causing all cmpxchg attempts to
// fail.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6642-6645
```tablegen
// Unfortunately, this means we have to have an alternative (expanded
// post-regalloc) path for -O0 compilations. Fortunately this path can be
// significantly more naive than the standard expansion: we conservatively
// assume seq_cst, strong cmpxchg and omit clrex on failure.
```
- EN: Adds TableGen-side comments that document the intent of the records that follow.
- CN: 这里添加 TableGen 侧注释，用于说明后续记录的设计意图。

### Lines 6647-6651
```tablegen
let Constraints = "@earlyclobber $Rd,@earlyclobber $temp",
    mayLoad = 1, mayStore = 1 in {
def CMP_SWAP_8 : PseudoInst<(outs GPR:$Rd, GPR:$temp),
                            (ins GPR:$addr, GPR:$desired, GPR:$new),
                            NoItinerary, []>, Sched<[]>;
```
- EN: Defines TableGen record `CMP_SWAP_8` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMP_SWAP_8`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6653-6655
```tablegen
def CMP_SWAP_16 : PseudoInst<(outs GPR:$Rd, GPR:$temp),
                             (ins GPR:$addr, GPR:$desired, GPR:$new),
                             NoItinerary, []>, Sched<[]>;
```
- EN: Defines TableGen record `CMP_SWAP_16` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMP_SWAP_16`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6657-6659
```tablegen
def CMP_SWAP_32 : PseudoInst<(outs GPR:$Rd, GPR:$temp),
                             (ins GPR:$addr, GPR:$desired, GPR:$new),
                             NoItinerary, []>, Sched<[]>;
```
- EN: Defines TableGen record `CMP_SWAP_32` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMP_SWAP_32`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6661-6677
```tablegen
// The addr_temp and addr_temp_out operands are logically a pair of GPR
// operands:
// * addr is an input, holding the address to swap.
// * temp is a earlyclobber output, used internally in the expansion of the
//   pseudo-inst.
// These are combined into one GPRPair operand to ensure that register
// allocation always succeeds. In the worst case there are only 4 GPRPair
// registers available, of which this instruction needs 3 for the other
// operands. If these operands weren't combined they would also use two GPR
// registers, which could overlap with two different GPRPairs, causing
// allocation to fail. With them combined, we need to allocate 4 GPRPairs,
// which will always succeed.
let Constraints = "@earlyclobber $Rd,$addr_temp_out = $addr_temp" in
def CMP_SWAP_64 : PseudoInst<(outs GPRPair:$Rd, GPRPair:$addr_temp_out),
                             (ins GPRPair:$addr_temp, GPRPair:$desired, GPRPair:$new),
                             NoItinerary, []>, Sched<[]>;
}
```
- EN: Defines TableGen record `CMP_SWAP_64` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `CMP_SWAP_64`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6679-6679
```tablegen
def : Pat<(atomic_fence (timm), 0), (MEMBARRIER)>;
```
- EN: Adds supporting TableGen DSL structure for target records, patterns, or metadata used by generated backend tables.
- CN: 这里补充目标记录、匹配模式或元数据相关的 TableGen DSL 结构，供后续生成后端表使用。

### Lines 6681-6686
```tablegen
//===----------------------------------------------------------------------===//
// KCFI check pseudo-instruction.
//===----------------------------------------------------------------------===//
// KCFI_CHECK pseudo-instruction for Kernel Control-Flow Integrity.
// Expands to a sequence that verifies the function pointer's type hash.
// Different sizes for different architectures due to different expansions.
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6688-6694
```tablegen
def KCFI_CHECK_ARM
    : PseudoInst<(outs), (ins GPR:$ptr, i32imm:$type), NoItinerary, []>,
      Sched<[]>,
      Requires<[IsARM]> {
  let Size = 40; // worst-case 10 instructions @ 4 bytes each
                 // (push, bic, ldr, 4x eor, pop, beq, udf)
}
```
- EN: Defines TableGen record `KCFI_CHECK_ARM` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `KCFI_CHECK_ARM`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6696-6702
```tablegen
def KCFI_CHECK_Thumb2
    : PseudoInst<(outs), (ins GPR:$ptr, i32imm:$type), NoItinerary, []>,
      Sched<[]>,
      Requires<[IsThumb2]> {
  let Size = 34; // worst-case (push.w[2], bic[4], ldr[4], 4x eor[16], pop.w[2],
                 // beq.w[4], udf[2])
}
```
- EN: Defines TableGen record `KCFI_CHECK_Thumb2` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `KCFI_CHECK_Thumb2`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6704-6711
```tablegen
def KCFI_CHECK_Thumb1
    : PseudoInst<(outs), (ins GPR:$ptr, i32imm:$type), NoItinerary, []>,
      Sched<[]>,
      Requires<[IsThumb1Only]> {
  let Size = 38; // worst-case 19 instructions @ 2 bytes each
                 // (2x push, 3x bic-helper, subs+ldr, 13x type-building, cmp,
                 // 2x pop, beq, bkpt)
}
```
- EN: Defines TableGen record `KCFI_CHECK_Thumb1` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `KCFI_CHECK_Thumb1`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

### Lines 6713-6730
```tablegen
//===----------------------------------------------------------------------===//
// Instructions used for emitting unwind opcodes on Windows.
//===----------------------------------------------------------------------===//
let isPseudo = 1 in {
  def SEH_StackAlloc : PseudoInst<(outs), (ins i32imm:$size, i32imm:$wide), NoItinerary, []>, Sched<[]>;
  def SEH_SaveRegs : PseudoInst<(outs), (ins i32imm:$mask, i32imm:$wide), NoItinerary, []>, Sched<[]>;
  let isTerminator = 1 in
  def SEH_SaveRegs_Ret : PseudoInst<(outs), (ins i32imm:$mask, i32imm:$wide), NoItinerary, []>, Sched<[]>;
  def SEH_SaveSP : PseudoInst<(outs), (ins i32imm:$reg), NoItinerary, []>, Sched<[]>;
  def SEH_SaveFRegs : PseudoInst<(outs), (ins i32imm:$first, i32imm:$last), NoItinerary, []>, Sched<[]>;
  let isTerminator = 1 in
  def SEH_SaveLR : PseudoInst<(outs), (ins i32imm:$offst), NoItinerary, []>, Sched<[]>;
  def SEH_Nop : PseudoInst<(outs), (ins i32imm:$wide), NoItinerary, []>, Sched<[]>;
  let isTerminator = 1 in
  def SEH_Nop_Ret : PseudoInst<(outs), (ins i32imm:$wide), NoItinerary, []>, Sched<[]>;
  def SEH_PrologEnd : PseudoInst<(outs), (ins), NoItinerary, []>, Sched<[]>;
  def SEH_EpilogStart : PseudoInst<(outs), (ins), NoItinerary, []>, Sched<[]>;
  let isTerminator = 1 in
```
- EN: Introduces the file banner and explains that the following content is written in LLVM TableGen DSL rather than ordinary C++.
- CN: 这里给出文件横幅，并说明后续内容使用的是 LLVM TableGen DSL，而不是普通 C++。

### Lines 6731-6732
```tablegen
  def SEH_EpilogEnd : PseudoInst<(outs), (ins), NoItinerary, []>, Sched<[]>;
}
```
- EN: Defines TableGen record `SEH_EpilogEnd` in the DSL, contributing concrete target metadata that tblgen will later transform into generated code or tables.
- CN: 这里在 DSL 中定义 TableGen 记录 `SEH_EpilogEnd`，为后续 tblgen 生成代码或数据表提供具体的目标元数据。

## Key Concepts / 关键概念

- EN: Primary role: instruction semantics, scheduling hints, and machine-level helpers.
  - CN: 核心职责：指令语义、调度提示以及机器级辅助逻辑。
- EN: LLVM TableGen DSL records drive generated enums, matchers, encoders, or scheduling data.
  - CN: LLVM TableGen DSL 记录会驱动生成枚举、匹配器、编码器或调度数据。
- EN: SelectionDAG patterns or node profiles connect IR-level intent to target instructions.
  - CN: SelectionDAG 模式或节点轮廓把 IR 层意图连接到目标指令。
- EN: Bitfield assignments describe exact instruction encodings and per-instruction flags.
  - CN: 位字段赋值用于描述精确的指令编码以及每条指令的标志位。
- EN: Predicates and feature gates control when records apply to a subtarget or ISA variant.
  - CN: 谓词与特性门控决定记录在何种子目标或 ISA 变体下生效。

## Dependencies / 依赖关系

- EN: TableGen includes: `ARMInstrFormats.td`, `ARMInstrThumb.td`, `ARMInstrThumb2.td`, `ARMInstrVFP.td`, `ARMInstrNEON.td`, `ARMInstrMVE.td`, `ARMInstrCDE.td`.
  - CN: TableGen 包含项：`ARMInstrFormats.td`, `ARMInstrThumb.td`, `ARMInstrThumb2.td`, `ARMInstrVFP.td`, `ARMInstrNEON.td`, `ARMInstrMVE.td`, `ARMInstrCDE.td`。
- EN: Generated backend tables depend on tblgen consuming these records and emitting C++/inc fragments.
  - CN: 生成后的后端表依赖 tblgen 消费这些记录并产出 C++/inc 片段。
