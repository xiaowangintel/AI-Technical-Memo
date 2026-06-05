# XCoreISelLowering.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `llvm/lib/Target/XCore/XCoreISelLowering.cpp`
- **Repository / 仓库**: `llvm-project`
- **Purpose / 作用**:
  - **EN**: Customizes SelectionDAG lowering, legalization, calling convention handling, and target-specific DAG nodes.
  - **CN**: 定制 SelectionDAG 降低、合法化、调用约定处理以及目标专用 DAG 节点。

## Line-by-Line Analysis / 逐行分析

### Lines 1-38
```cpp
//===-- XCoreISelLowering.cpp - XCore DAG Lowering Implementation ---------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the XCoreTargetLowering class.
//
//===----------------------------------------------------------------------===//

#include "XCoreISelLowering.h"
#include "XCore.h"
#include "XCoreMachineFunctionInfo.h"
#include "XCoreSubtarget.h"
#include "XCoreTargetMachine.h"
#include "XCoreTargetObjectFile.h"
#include "llvm/CodeGen/CallingConvLower.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
#include "llvm/CodeGen/MachineJumpTableInfo.h"
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/CodeGen/ValueTypes.h"
#include "llvm/IR/CallingConv.h"
#include "llvm/IR/Constants.h"
#include "llvm/IR/DerivedTypes.h"
#include "llvm/IR/Function.h"
#include "llvm/IR/GlobalVariable.h"
#include "llvm/IR/Intrinsics.h"
#include "llvm/IR/IntrinsicsXCore.h"
#include "llvm/Support/Debug.h"
#include "llvm/Support/ErrorHandling.h"
#include "llvm/Support/KnownBits.h"
#include "llvm/Support/raw_ostream.h"
#include <algorithm>

```
- **EN**: Pulls in the headers needed for this implementation, including `XCoreISelLowering.h`, `XCore.h`, `XCoreMachineFunctionInfo.h`, `XCoreSubtarget.h`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCoreISelLowering.h`, `XCore.h`, `XCoreMachineFunctionInfo.h`, `XCoreSubtarget.h`。

### Lines 39-60
```cpp
using namespace llvm;

#define DEBUG_TYPE "xcore-lower"

XCoreTargetLowering::XCoreTargetLowering(const TargetMachine &TM,
                                         const XCoreSubtarget &Subtarget)
    : TargetLowering(TM, Subtarget), TM(TM), Subtarget(Subtarget) {

  // Set up the register classes.
  addRegisterClass(MVT::i32, &XCore::GRRegsRegClass);

  // Compute derived properties from the register classes
  computeRegisterProperties(Subtarget.getRegisterInfo());

  setStackPointerRegisterToSaveRestore(XCore::SP);

  setSchedulingPreference(Sched::Source);

  // Use i32 for setcc operations results (slt, sgt, ...).
  setBooleanContents(ZeroOrOneBooleanContent);
  setBooleanVectorContents(ZeroOrOneBooleanContent); // FIXME: Is this correct?

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 61-81
```cpp
  // XCore does not have the NodeTypes below.
  setOperationAction(ISD::BR_CC,     MVT::i32,   Expand);
  setOperationAction(ISD::SELECT_CC, MVT::i32,   Expand);

  // 64bit
  setOperationAction(ISD::ADD, MVT::i64, Custom);
  setOperationAction(ISD::SUB, MVT::i64, Custom);
  setOperationAction(ISD::SMUL_LOHI, MVT::i32, Custom);
  setOperationAction(ISD::UMUL_LOHI, MVT::i32, Custom);
  setOperationAction(ISD::MULHS, MVT::i32, Expand);
  setOperationAction(ISD::MULHU, MVT::i32, Expand);
  setOperationAction(ISD::SHL_PARTS, MVT::i32, Expand);
  setOperationAction(ISD::SRA_PARTS, MVT::i32, Expand);
  setOperationAction(ISD::SRL_PARTS, MVT::i32, Expand);

  // Bit Manipulation
  setOperationAction(ISD::CTPOP, MVT::i32, Expand);
  setOperationAction(ISD::ROTL , MVT::i32, Expand);
  setOperationAction(ISD::ROTR , MVT::i32, Expand);
  setOperationAction(ISD::BITREVERSE , MVT::i32, Legal);

```
- **EN**: Implements logic around `setOperationAction`; this block configures legalization and lowering behavior.
- **CN**: 围绕 `setOperationAction` 实现具体逻辑；这一段配置合法化与 lowering 行为。

### Lines 82-102
```cpp
  setOperationAction(ISD::TRAP, MVT::Other, Legal);

  // Jump tables.
  setOperationAction(ISD::BR_JT, MVT::Other, Custom);

  setOperationAction(ISD::GlobalAddress, MVT::i32,   Custom);
  setOperationAction(ISD::BlockAddress, MVT::i32 , Custom);

  // Conversion of i64 -> double produces constantpool nodes
  setOperationAction(ISD::ConstantPool, MVT::i32,   Custom);

  // Loads
  for (MVT VT : MVT::integer_valuetypes()) {
    setLoadExtAction(ISD::EXTLOAD, VT, MVT::i1, Promote);
    setLoadExtAction(ISD::ZEXTLOAD, VT, MVT::i1, Promote);
    setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i1, Promote);

    setLoadExtAction(ISD::SEXTLOAD, VT, MVT::i8, Expand);
    setLoadExtAction(ISD::ZEXTLOAD, VT, MVT::i16, Expand);
  }

```
- **EN**: Implements logic around `setOperationAction`, `setLoadExtAction`; this block configures legalization and lowering behavior.
- **CN**: 围绕 `setOperationAction`, `setLoadExtAction` 实现具体逻辑；这一段配置合法化与 lowering 行为。

### Lines 103-123
```cpp
  // Custom expand misaligned loads / stores.
  setOperationAction(ISD::LOAD, MVT::i32, Custom);
  setOperationAction(ISD::STORE, MVT::i32, Custom);

  // Varargs
  setOperationAction(ISD::VAEND, MVT::Other, Expand);
  setOperationAction(ISD::VACOPY, MVT::Other, Expand);
  setOperationAction(ISD::VAARG, MVT::Other, Custom);
  setOperationAction(ISD::VASTART, MVT::Other, Custom);

  // Dynamic stack
  setOperationAction(ISD::STACKSAVE, MVT::Other, Expand);
  setOperationAction(ISD::STACKRESTORE, MVT::Other, Expand);
  setOperationAction(ISD::DYNAMIC_STACKALLOC, MVT::i32, Expand);

  // Exception handling
  setOperationAction(ISD::EH_RETURN, MVT::Other, Custom);
  setOperationAction(ISD::FRAME_TO_ARGS_OFFSET, MVT::i32, Custom);

  setOperationAction(ISD::ATOMIC_FENCE, MVT::Other, Custom);

```
- **EN**: Implements logic around `setOperationAction`; this block configures legalization and lowering behavior.
- **CN**: 围绕 `setOperationAction` 实现具体逻辑；这一段配置合法化与 lowering 行为。

### Lines 124-145
```cpp
  // TRAMPOLINE is custom lowered.
  setOperationAction(ISD::INIT_TRAMPOLINE, MVT::Other, Custom);
  setOperationAction(ISD::ADJUST_TRAMPOLINE, MVT::Other, Custom);

  // We want to custom lower some of our intrinsics.
  setOperationAction(ISD::INTRINSIC_WO_CHAIN, MVT::Other, Custom);

  MaxStoresPerMemset = MaxStoresPerMemsetOptSize = 4;
  MaxStoresPerMemmove = MaxStoresPerMemmoveOptSize
    = MaxStoresPerMemcpy = MaxStoresPerMemcpyOptSize = 2;

  // We have target-specific dag combine patterns for the following nodes:
  setTargetDAGCombine(
      {ISD::STORE, ISD::ADD, ISD::INTRINSIC_VOID, ISD::INTRINSIC_W_CHAIN});

  setMinFunctionAlignment(Align(2));
  setPrefFunctionAlignment(Align(4));

  // This target doesn't implement native atomics.
  setMaxAtomicSizeInBitsSupported(0);
}

```
- **EN**: Implements logic around `setOperationAction`, `setTargetDAGCombine`, `setMinFunctionAlignment`, `setPrefFunctionAlignment`, ...; this block configures legalization and lowering behavior; handles SelectionDAG-specific logic.
- **CN**: 围绕 `setOperationAction`, `setTargetDAGCombine`, `setMinFunctionAlignment`, `setPrefFunctionAlignment`, ... 实现具体逻辑；这一段配置合法化与 lowering 行为，处理 SelectionDAG 专用逻辑。

### Lines 146-185
```cpp
bool XCoreTargetLowering::isZExtFree(SDValue Val, EVT VT2) const {
  if (Val.getOpcode() != ISD::LOAD)
    return false;

  EVT VT1 = Val.getValueType();
  if (!VT1.isSimple() || !VT1.isInteger() ||
      !VT2.isSimple() || !VT2.isInteger())
    return false;

  switch (VT1.getSimpleVT().SimpleTy) {
  default: break;
  case MVT::i8:
    return true;
  }

  return false;
}

SDValue XCoreTargetLowering::
LowerOperation(SDValue Op, SelectionDAG &DAG) const {
  switch (Op.getOpcode())
  {
  case ISD::EH_RETURN:          return LowerEH_RETURN(Op, DAG);
  case ISD::GlobalAddress:      return LowerGlobalAddress(Op, DAG);
  case ISD::BlockAddress:       return LowerBlockAddress(Op, DAG);
  case ISD::ConstantPool:       return LowerConstantPool(Op, DAG);
  case ISD::BR_JT:              return LowerBR_JT(Op, DAG);
  case ISD::LOAD:               return LowerLOAD(Op, DAG);
  case ISD::STORE:              return LowerSTORE(Op, DAG);
  case ISD::VAARG:              return LowerVAARG(Op, DAG);
  case ISD::VASTART:            return LowerVASTART(Op, DAG);
  case ISD::SMUL_LOHI:          return LowerSMUL_LOHI(Op, DAG);
  case ISD::UMUL_LOHI:          return LowerUMUL_LOHI(Op, DAG);
  // FIXME: Remove these when LegalizeDAGTypes lands.
  case ISD::ADD:
  case ISD::SUB:                return ExpandADDSUB(Op.getNode(), DAG);
  case ISD::FRAMEADDR:          return LowerFRAMEADDR(Op, DAG);
  case ISD::RETURNADDR:         return LowerRETURNADDR(Op, DAG);
  case ISD::FRAME_TO_ARGS_OFFSET: return LowerFRAME_TO_ARGS_OFFSET(Op, DAG);
  case ISD::INIT_TRAMPOLINE:    return LowerINIT_TRAMPOLINE(Op, DAG);
```
- **EN**: Implements logic around `isZExtFree`, `getValueType`, `isSimple`, `LowerOperation`, ...; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `isZExtFree`, `getValueType`, `isSimple`, `LowerOperation`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 186-209
```cpp
  case ISD::ADJUST_TRAMPOLINE:  return LowerADJUST_TRAMPOLINE(Op, DAG);
  case ISD::INTRINSIC_WO_CHAIN: return LowerINTRINSIC_WO_CHAIN(Op, DAG);
  case ISD::ATOMIC_FENCE:
    return LowerATOMIC_FENCE(Op, DAG);
  default:
    llvm_unreachable("unimplemented operand");
  }
}

/// ReplaceNodeResults - Replace the results of node with an illegal result
/// type with new values built out of custom code.
void XCoreTargetLowering::ReplaceNodeResults(SDNode *N,
                                             SmallVectorImpl<SDValue>&Results,
                                             SelectionDAG &DAG) const {
  switch (N->getOpcode()) {
  default:
    llvm_unreachable("Don't know how to custom expand this!");
  case ISD::ADD:
  case ISD::SUB:
    Results.push_back(ExpandADDSUB(N, DAG));
    return;
  }
}

```
- **EN**: Implements logic around `LowerADJUST_TRAMPOLINE`, `LowerINTRINSIC_WO_CHAIN`, `LowerATOMIC_FENCE`, `llvm_unreachable`, ...; this block uses `switch`-based dispatch; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerADJUST_TRAMPOLINE`, `LowerINTRINSIC_WO_CHAIN`, `LowerATOMIC_FENCE`, `llvm_unreachable`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 210-230
```cpp
//===----------------------------------------------------------------------===//
//  Misc Lower Operation implementation
//===----------------------------------------------------------------------===//

SDValue XCoreTargetLowering::getGlobalAddressWrapper(SDValue GA,
                                                     const GlobalValue *GV,
                                                     SelectionDAG &DAG) const {
  // FIXME there is no actual debug info here
  SDLoc dl(GA);

  if (GV->getValueType()->isFunctionTy())
    return DAG.getNode(XCoreISD::PCRelativeWrapper, dl, MVT::i32, GA);

  const auto *GVar = dyn_cast<GlobalVariable>(GV);
  if ((GV->hasSection() && GV->getSection().starts_with(".cp.")) ||
      (GVar && GVar->isConstant() && GV->hasLocalLinkage()))
    return DAG.getNode(XCoreISD::CPRelativeWrapper, dl, MVT::i32, GA);

  return DAG.getNode(XCoreISD::DPRelativeWrapper, dl, MVT::i32, GA);
}

```
- **EN**: Implements logic around `getGlobalAddressWrapper`, `dl`, `getNode`, `dyn_cast<GlobalVariable>`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getGlobalAddressWrapper`, `dl`, `getNode`, `dyn_cast<GlobalVariable>`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 231-270
```cpp
static bool IsSmallObject(const GlobalValue *GV, const XCoreTargetLowering &XTL) {
  if (XTL.getTargetMachine().getCodeModel() == CodeModel::Small)
    return true;

  Type *ObjType = GV->getValueType();
  if (!ObjType->isSized())
    return false;

  auto &DL = GV->getDataLayout();
  unsigned ObjSize = DL.getTypeAllocSize(ObjType);
  return ObjSize < CodeModelLargeSize && ObjSize != 0;
}

SDValue XCoreTargetLowering::
LowerGlobalAddress(SDValue Op, SelectionDAG &DAG) const
{
  const GlobalAddressSDNode *GN = cast<GlobalAddressSDNode>(Op);
  const GlobalValue *GV = GN->getGlobal();
  SDLoc DL(GN);
  int64_t Offset = GN->getOffset();
  if (IsSmallObject(GV, *this)) {
    // We can only fold positive offsets that are a multiple of the word size.
    int64_t FoldedOffset = std::max(Offset & ~3, (int64_t)0);
    SDValue GA = DAG.getTargetGlobalAddress(GV, DL, MVT::i32, FoldedOffset);
    GA = getGlobalAddressWrapper(GA, GV, DAG);
    // Handle the rest of the offset.
    if (Offset != FoldedOffset) {
      SDValue Remaining =
          DAG.getSignedConstant(Offset - FoldedOffset, DL, MVT::i32);
      GA = DAG.getNode(ISD::ADD, DL, MVT::i32, GA, Remaining);
    }
    return GA;
  } else {
    // Ideally we would not fold in offset with an index <= 11.
    Type *Ty = Type::getInt32Ty(*DAG.getContext());
    Constant *Idx = ConstantInt::get(Ty, Offset);
    Constant *GAI = ConstantExpr::getGetElementPtr(
        Type::getInt8Ty(*DAG.getContext()), const_cast<GlobalValue *>(GV), Idx);
    SDValue CP = DAG.getConstantPool(GAI, MVT::i32);
    return DAG.getLoad(getPointerTy(DAG.getDataLayout()), DL,
```
- **EN**: Implements logic around `IsSmallObject`, `getValueType`, `getDataLayout`, `getTypeAllocSize`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `IsSmallObject`, `getValueType`, `getDataLayout`, `getTypeAllocSize`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 271-303
```cpp
                       DAG.getEntryNode(), CP, MachinePointerInfo());
  }
}

SDValue XCoreTargetLowering::
LowerBlockAddress(SDValue Op, SelectionDAG &DAG) const
{
  SDLoc DL(Op);
  auto PtrVT = getPointerTy(DAG.getDataLayout());
  const BlockAddress *BA = cast<BlockAddressSDNode>(Op)->getBlockAddress();
  SDValue Result = DAG.getTargetBlockAddress(BA, PtrVT);

  return DAG.getNode(XCoreISD::PCRelativeWrapper, DL, PtrVT, Result);
}

SDValue XCoreTargetLowering::
LowerConstantPool(SDValue Op, SelectionDAG &DAG) const
{
  ConstantPoolSDNode *CP = cast<ConstantPoolSDNode>(Op);
  // FIXME there isn't really debug info here
  SDLoc dl(CP);
  EVT PtrVT = Op.getValueType();
  SDValue Res;
  if (CP->isMachineConstantPoolEntry()) {
    Res = DAG.getTargetConstantPool(CP->getMachineCPVal(), PtrVT,
                                    CP->getAlign(), CP->getOffset());
  } else {
    Res = DAG.getTargetConstantPool(CP->getConstVal(), PtrVT, CP->getAlign(),
                                    CP->getOffset());
  }
  return DAG.getNode(XCoreISD::CPRelativeWrapper, dl, MVT::i32, Res);
}

```
- **EN**: Implements logic around `getEntryNode`, `LowerBlockAddress`, `DL`, `getPointerTy`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getEntryNode`, `LowerBlockAddress`, `DL`, `getPointerTy`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 304-331
```cpp
unsigned XCoreTargetLowering::getJumpTableEncoding() const {
  return MachineJumpTableInfo::EK_Inline;
}

SDValue XCoreTargetLowering::
LowerBR_JT(SDValue Op, SelectionDAG &DAG) const
{
  SDValue Chain = Op.getOperand(0);
  SDValue Table = Op.getOperand(1);
  SDValue Index = Op.getOperand(2);
  SDLoc dl(Op);
  JumpTableSDNode *JT = cast<JumpTableSDNode>(Table);
  unsigned JTI = JT->getIndex();
  MachineFunction &MF = DAG.getMachineFunction();
  const MachineJumpTableInfo *MJTI = MF.getJumpTableInfo();
  SDValue TargetJT = DAG.getTargetJumpTable(JT->getIndex(), MVT::i32);

  unsigned NumEntries = MJTI->getJumpTables()[JTI].MBBs.size();
  if (NumEntries <= 32) {
    return DAG.getNode(XCoreISD::BR_JT, dl, MVT::Other, Chain, TargetJT, Index);
  }
  assert((NumEntries >> 31) == 0);
  SDValue ScaledIndex = DAG.getNode(ISD::SHL, dl, MVT::i32, Index,
                                    DAG.getConstant(1, dl, MVT::i32));
  return DAG.getNode(XCoreISD::BR_JT32, dl, MVT::Other, Chain, TargetJT,
                     ScaledIndex);
}

```
- **EN**: Implements logic around `getJumpTableEncoding`, `LowerBR_JT`, `getOperand`, `dl`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getJumpTableEncoding`, `LowerBR_JT`, `getOperand`, `dl`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 332-357
```cpp
SDValue XCoreTargetLowering::lowerLoadWordFromAlignedBasePlusOffset(
    const SDLoc &DL, SDValue Chain, SDValue Base, int64_t Offset,
    SelectionDAG &DAG) const {
  auto PtrVT = getPointerTy(DAG.getDataLayout());
  if ((Offset & 0x3) == 0) {
    return DAG.getLoad(PtrVT, DL, Chain, Base, MachinePointerInfo());
  }
  // Lower to pair of consecutive word aligned loads plus some bit shifting.
  int32_t HighOffset = alignTo(Offset, 4);
  int32_t LowOffset = HighOffset - 4;
  SDValue LowAddr, HighAddr;
  if (GlobalAddressSDNode *GASD =
        dyn_cast<GlobalAddressSDNode>(Base.getNode())) {
    LowAddr = DAG.getGlobalAddress(GASD->getGlobal(), DL, Base.getValueType(),
                                   LowOffset);
    HighAddr = DAG.getGlobalAddress(GASD->getGlobal(), DL, Base.getValueType(),
                                    HighOffset);
  } else {
    LowAddr = DAG.getNode(ISD::ADD, DL, MVT::i32, Base,
                          DAG.getConstant(LowOffset, DL, MVT::i32));
    HighAddr = DAG.getNode(ISD::ADD, DL, MVT::i32, Base,
                           DAG.getConstant(HighOffset, DL, MVT::i32));
  }
  SDValue LowShift = DAG.getConstant((Offset - LowOffset) * 8, DL, MVT::i32);
  SDValue HighShift = DAG.getConstant((HighOffset - Offset) * 8, DL, MVT::i32);

```
- **EN**: Implements logic around `lowerLoadWordFromAlignedBasePlusOffset`, `getPointerTy`, `getLoad`, `alignTo`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `lowerLoadWordFromAlignedBasePlusOffset`, `getPointerTy`, `getLoad`, `alignTo`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 358-382
```cpp
  SDValue Low = DAG.getLoad(PtrVT, DL, Chain, LowAddr, MachinePointerInfo());
  SDValue High = DAG.getLoad(PtrVT, DL, Chain, HighAddr, MachinePointerInfo());
  SDValue LowShifted = DAG.getNode(ISD::SRL, DL, MVT::i32, Low, LowShift);
  SDValue HighShifted = DAG.getNode(ISD::SHL, DL, MVT::i32, High, HighShift);
  SDValue Result = DAG.getNode(ISD::OR, DL, MVT::i32, LowShifted, HighShifted);
  Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, Low.getValue(1),
                      High.getValue(1));
  SDValue Ops[] = { Result, Chain };
  return DAG.getMergeValues(Ops, DL);
}

static bool isWordAligned(SDValue Value, SelectionDAG &DAG)
{
  KnownBits Known = DAG.computeKnownBits(Value);
  return Known.countMinTrailingZeros() >= 2;
}

SDValue XCoreTargetLowering::LowerLOAD(SDValue Op, SelectionDAG &DAG) const {
  const TargetLowering &TLI = DAG.getTargetLoweringInfo();
  LLVMContext &Context = *DAG.getContext();
  LoadSDNode *LD = cast<LoadSDNode>(Op);
  assert(LD->getExtensionType() == ISD::NON_EXTLOAD &&
         "Unexpected extension type");
  assert(LD->getMemoryVT() == MVT::i32 && "Unexpected load EVT");

```
- **EN**: Implements logic around `getLoad`, `getNode`, `getValue`, `getMergeValues`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getLoad`, `getNode`, `getValue`, `getMergeValues`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 383-409
```cpp
  if (allowsMemoryAccessForAlignment(Context, DAG.getDataLayout(),
                                     LD->getMemoryVT(), *LD->getMemOperand()))
    return SDValue();

  SDValue Chain = LD->getChain();
  SDValue BasePtr = LD->getBasePtr();
  SDLoc DL(Op);

  if (!LD->isVolatile()) {
    const GlobalValue *GV;
    int64_t Offset = 0;
    if (DAG.isBaseWithConstantOffset(BasePtr) &&
        isWordAligned(BasePtr->getOperand(0), DAG)) {
      SDValue NewBasePtr = BasePtr->getOperand(0);
      Offset = cast<ConstantSDNode>(BasePtr->getOperand(1))->getSExtValue();
      return lowerLoadWordFromAlignedBasePlusOffset(DL, Chain, NewBasePtr,
                                                    Offset, DAG);
    }
    if (TLI.isGAPlusOffset(BasePtr.getNode(), GV, Offset) &&
        GV->getPointerAlignment(DAG.getDataLayout()) >= 4) {
      SDValue NewBasePtr = DAG.getGlobalAddress(GV, DL,
                                                BasePtr->getValueType(0));
      return lowerLoadWordFromAlignedBasePlusOffset(DL, Chain, NewBasePtr,
                                                    Offset, DAG);
    }
  }

```
- **EN**: Implements logic around `getMemoryVT`, `SDValue`, `getChain`, `getBasePtr`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getMemoryVT`, `SDValue`, `getChain`, `getBasePtr`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 410-433
```cpp
  if (LD->getAlign() == Align(2)) {
    SDValue Low = DAG.getExtLoad(ISD::ZEXTLOAD, DL, MVT::i32, Chain, BasePtr,
                                 LD->getPointerInfo(), MVT::i16, Align(2),
                                 LD->getMemOperand()->getFlags());
    SDValue HighAddr = DAG.getNode(ISD::ADD, DL, MVT::i32, BasePtr,
                                   DAG.getConstant(2, DL, MVT::i32));
    SDValue High =
        DAG.getExtLoad(ISD::EXTLOAD, DL, MVT::i32, Chain, HighAddr,
                       LD->getPointerInfo().getWithOffset(2), MVT::i16,
                       Align(2), LD->getMemOperand()->getFlags());
    SDValue HighShifted = DAG.getNode(ISD::SHL, DL, MVT::i32, High,
                                      DAG.getConstant(16, DL, MVT::i32));
    SDValue Result = DAG.getNode(ISD::OR, DL, MVT::i32, Low, HighShifted);
    Chain = DAG.getNode(ISD::TokenFactor, DL, MVT::Other, Low.getValue(1),
                             High.getValue(1));
    SDValue Ops[] = { Result, Chain };
    return DAG.getMergeValues(Ops, DL);
  }

  // Lower to a call to __misaligned_load(BasePtr).
  Type *IntPtrTy = DAG.getDataLayout().getIntPtrType(Context);
  TargetLowering::ArgListTy Args;
  Args.emplace_back(BasePtr, IntPtrTy);

```
- **EN**: Implements logic around `getExtLoad`, `getPointerInfo`, `getMemOperand`, `getNode`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getExtLoad`, `getPointerInfo`, `getMemOperand`, `getNode`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 434-455
```cpp
  TargetLowering::CallLoweringInfo CLI(DAG);
  CLI.setDebugLoc(DL).setChain(Chain).setLibCallee(
      CallingConv::C, IntPtrTy,
      DAG.getExternalSymbol("__misaligned_load",
                            getPointerTy(DAG.getDataLayout())),
      std::move(Args));

  std::pair<SDValue, SDValue> CallResult = LowerCallTo(CLI);
  SDValue Ops[] = { CallResult.first, CallResult.second };
  return DAG.getMergeValues(Ops, DL);
}

SDValue XCoreTargetLowering::LowerSTORE(SDValue Op, SelectionDAG &DAG) const {
  LLVMContext &Context = *DAG.getContext();
  StoreSDNode *ST = cast<StoreSDNode>(Op);
  assert(!ST->isTruncatingStore() && "Unexpected store type");
  assert(ST->getMemoryVT() == MVT::i32 && "Unexpected store EVT");

  if (allowsMemoryAccessForAlignment(Context, DAG.getDataLayout(),
                                     ST->getMemoryVT(), *ST->getMemOperand()))
    return SDValue();

```
- **EN**: Implements logic around `CLI`, `setDebugLoc`, `getExternalSymbol`, `getPointerTy`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `CLI`, `setDebugLoc`, `getExternalSymbol`, `getPointerTy`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 456-475
```cpp
  SDValue Chain = ST->getChain();
  SDValue BasePtr = ST->getBasePtr();
  SDValue Value = ST->getValue();
  SDLoc dl(Op);

  if (ST->getAlign() == Align(2)) {
    SDValue Low = Value;
    SDValue High = DAG.getNode(ISD::SRL, dl, MVT::i32, Value,
                               DAG.getConstant(16, dl, MVT::i32));
    SDValue StoreLow =
        DAG.getTruncStore(Chain, dl, Low, BasePtr, ST->getPointerInfo(),
                          MVT::i16, Align(2), ST->getMemOperand()->getFlags());
    SDValue HighAddr = DAG.getNode(ISD::ADD, dl, MVT::i32, BasePtr,
                                   DAG.getConstant(2, dl, MVT::i32));
    SDValue StoreHigh = DAG.getTruncStore(
        Chain, dl, High, HighAddr, ST->getPointerInfo().getWithOffset(2),
        MVT::i16, Align(2), ST->getMemOperand()->getFlags());
    return DAG.getNode(ISD::TokenFactor, dl, MVT::Other, StoreLow, StoreHigh);
  }

```
- **EN**: Implements logic around `getChain`, `getBasePtr`, `getValue`, `dl`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getChain`, `getBasePtr`, `getValue`, `dl`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 476-509
```cpp
  // Lower to a call to __misaligned_store(BasePtr, Value).
  Type *IntPtrTy = DAG.getDataLayout().getIntPtrType(Context);
  TargetLowering::ArgListTy Args;
  Args.emplace_back(BasePtr, IntPtrTy);
  Args.emplace_back(Value, IntPtrTy);

  TargetLowering::CallLoweringInfo CLI(DAG);
  CLI.setDebugLoc(dl).setChain(Chain).setCallee(
      CallingConv::C, Type::getVoidTy(Context),
      DAG.getExternalSymbol("__misaligned_store",
                            getPointerTy(DAG.getDataLayout())),
      std::move(Args));

  std::pair<SDValue, SDValue> CallResult = LowerCallTo(CLI);
  return CallResult.second;
}

SDValue XCoreTargetLowering::
LowerSMUL_LOHI(SDValue Op, SelectionDAG &DAG) const
{
  assert(Op.getValueType() == MVT::i32 && Op.getOpcode() == ISD::SMUL_LOHI &&
         "Unexpected operand to lower!");
  SDLoc dl(Op);
  SDValue LHS = Op.getOperand(0);
  SDValue RHS = Op.getOperand(1);
  SDValue Zero = DAG.getConstant(0, dl, MVT::i32);
  SDValue Hi = DAG.getNode(XCoreISD::MACCS, dl,
                           DAG.getVTList(MVT::i32, MVT::i32), Zero, Zero,
                           LHS, RHS);
  SDValue Lo(Hi.getNode(), 1);
  SDValue Ops[] = { Lo, Hi };
  return DAG.getMergeValues(Ops, dl);
}

```
- **EN**: Implements logic around `getDataLayout`, `emplace_back`, `CLI`, `setDebugLoc`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getDataLayout`, `emplace_back`, `CLI`, `setDebugLoc`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 510-529
```cpp
SDValue XCoreTargetLowering::
LowerUMUL_LOHI(SDValue Op, SelectionDAG &DAG) const
{
  assert(Op.getValueType() == MVT::i32 && Op.getOpcode() == ISD::UMUL_LOHI &&
         "Unexpected operand to lower!");
  SDLoc dl(Op);
  SDValue LHS = Op.getOperand(0);
  SDValue RHS = Op.getOperand(1);
  SDValue Zero = DAG.getConstant(0, dl, MVT::i32);
  SDValue Hi = DAG.getNode(XCoreISD::LMUL, dl,
                           DAG.getVTList(MVT::i32, MVT::i32), LHS, RHS,
                           Zero, Zero);
  SDValue Lo(Hi.getNode(), 1);
  SDValue Ops[] = { Lo, Hi };
  return DAG.getMergeValues(Ops, dl);
}

/// isADDADDMUL - Return whether Op is in a form that is equivalent to
/// add(add(mul(x,y),a),b). If requireIntermediatesHaveOneUse is true then
/// each intermediate result in the calculation must also have a single use.
```
- **EN**: Implements logic around `LowerUMUL_LOHI`, `assert`, `dl`, `getOperand`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerUMUL_LOHI`, `assert`, `dl`, `getOperand`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 530-569
```cpp
/// If the Op is in the correct form the constituent parts are written to Mul0,
/// Mul1, Addend0 and Addend1.
static bool
isADDADDMUL(SDValue Op, SDValue &Mul0, SDValue &Mul1, SDValue &Addend0,
            SDValue &Addend1, bool requireIntermediatesHaveOneUse)
{
  if (Op.getOpcode() != ISD::ADD)
    return false;
  SDValue N0 = Op.getOperand(0);
  SDValue N1 = Op.getOperand(1);
  SDValue AddOp;
  SDValue OtherOp;
  if (N0.getOpcode() == ISD::ADD) {
    AddOp = N0;
    OtherOp = N1;
  } else if (N1.getOpcode() == ISD::ADD) {
    AddOp = N1;
    OtherOp = N0;
  } else {
    return false;
  }
  if (requireIntermediatesHaveOneUse && !AddOp.hasOneUse())
    return false;
  if (OtherOp.getOpcode() == ISD::MUL) {
    // add(add(a,b),mul(x,y))
    if (requireIntermediatesHaveOneUse && !OtherOp.hasOneUse())
      return false;
    Mul0 = OtherOp.getOperand(0);
    Mul1 = OtherOp.getOperand(1);
    Addend0 = AddOp.getOperand(0);
    Addend1 = AddOp.getOperand(1);
    return true;
  }
  if (AddOp.getOperand(0).getOpcode() == ISD::MUL) {
    // add(add(mul(x,y),a),b)
    if (requireIntermediatesHaveOneUse && !AddOp.getOperand(0).hasOneUse())
      return false;
    Mul0 = AddOp.getOperand(0).getOperand(0);
    Mul1 = AddOp.getOperand(0).getOperand(1);
    Addend0 = AddOp.getOperand(1);
```
- **EN**: Implements logic around `isADDADDMUL`, `getOperand`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `isADDADDMUL`, `getOperand` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 570-609
```cpp
    Addend1 = OtherOp;
    return true;
  }
  if (AddOp.getOperand(1).getOpcode() == ISD::MUL) {
    // add(add(a,mul(x,y)),b)
    if (requireIntermediatesHaveOneUse && !AddOp.getOperand(1).hasOneUse())
      return false;
    Mul0 = AddOp.getOperand(1).getOperand(0);
    Mul1 = AddOp.getOperand(1).getOperand(1);
    Addend0 = AddOp.getOperand(0);
    Addend1 = OtherOp;
    return true;
  }
  return false;
}

SDValue XCoreTargetLowering::
TryExpandADDWithMul(SDNode *N, SelectionDAG &DAG) const
{
  SDValue Mul;
  SDValue Other;
  if (N->getOperand(0).getOpcode() == ISD::MUL) {
    Mul = N->getOperand(0);
    Other = N->getOperand(1);
  } else if (N->getOperand(1).getOpcode() == ISD::MUL) {
    Mul = N->getOperand(1);
    Other = N->getOperand(0);
  } else {
    return SDValue();
  }
  SDLoc dl(N);
  SDValue LL, RL, AddendL, AddendH;
  LL = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                   Mul.getOperand(0), DAG.getConstant(0, dl, MVT::i32));
  RL = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                   Mul.getOperand(1), DAG.getConstant(0, dl, MVT::i32));
  AddendL = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                        Other, DAG.getConstant(0, dl, MVT::i32));
  AddendH = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                        Other, DAG.getConstant(1, dl, MVT::i32));
```
- **EN**: Implements logic around `getOperand`, `TryExpandADDWithMul`, `SDValue`, `dl`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getOperand`, `TryExpandADDWithMul`, `SDValue`, `dl`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 610-645
```cpp
  APInt HighMask = APInt::getHighBitsSet(64, 32);
  unsigned LHSSB = DAG.ComputeNumSignBits(Mul.getOperand(0));
  unsigned RHSSB = DAG.ComputeNumSignBits(Mul.getOperand(1));
  if (DAG.MaskedValueIsZero(Mul.getOperand(0), HighMask) &&
      DAG.MaskedValueIsZero(Mul.getOperand(1), HighMask)) {
    // The inputs are both zero-extended.
    SDValue Hi = DAG.getNode(XCoreISD::MACCU, dl,
                             DAG.getVTList(MVT::i32, MVT::i32), AddendH,
                             AddendL, LL, RL);
    SDValue Lo(Hi.getNode(), 1);
    return DAG.getNode(ISD::BUILD_PAIR, dl, MVT::i64, Lo, Hi);
  }
  if (LHSSB > 32 && RHSSB > 32) {
    // The inputs are both sign-extended.
    SDValue Hi = DAG.getNode(XCoreISD::MACCS, dl,
                             DAG.getVTList(MVT::i32, MVT::i32), AddendH,
                             AddendL, LL, RL);
    SDValue Lo(Hi.getNode(), 1);
    return DAG.getNode(ISD::BUILD_PAIR, dl, MVT::i64, Lo, Hi);
  }
  SDValue LH, RH;
  LH = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                   Mul.getOperand(0), DAG.getConstant(1, dl, MVT::i32));
  RH = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                   Mul.getOperand(1), DAG.getConstant(1, dl, MVT::i32));
  SDValue Hi = DAG.getNode(XCoreISD::MACCU, dl,
                           DAG.getVTList(MVT::i32, MVT::i32), AddendH,
                           AddendL, LL, RL);
  SDValue Lo(Hi.getNode(), 1);
  RH = DAG.getNode(ISD::MUL, dl, MVT::i32, LL, RH);
  LH = DAG.getNode(ISD::MUL, dl, MVT::i32, LH, RL);
  Hi = DAG.getNode(ISD::ADD, dl, MVT::i32, Hi, RH);
  Hi = DAG.getNode(ISD::ADD, dl, MVT::i32, Hi, LH);
  return DAG.getNode(ISD::BUILD_PAIR, dl, MVT::i64, Lo, Hi);
}

```
- **EN**: Implements logic around `getHighBitsSet`, `ComputeNumSignBits`, `MaskedValueIsZero`, `getNode`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getHighBitsSet`, `ComputeNumSignBits`, `MaskedValueIsZero`, `getNode`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 646-672
```cpp
SDValue XCoreTargetLowering::
ExpandADDSUB(SDNode *N, SelectionDAG &DAG) const
{
  assert(N->getValueType(0) == MVT::i64 &&
         (N->getOpcode() == ISD::ADD || N->getOpcode() == ISD::SUB) &&
        "Unknown operand to lower!");

  if (N->getOpcode() == ISD::ADD)
    if (SDValue Result = TryExpandADDWithMul(N, DAG))
      return Result;

  SDLoc dl(N);

  // Extract components
  SDValue LHSL = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                             N->getOperand(0),
                             DAG.getConstant(0, dl, MVT::i32));
  SDValue LHSH = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                             N->getOperand(0),
                             DAG.getConstant(1, dl, MVT::i32));
  SDValue RHSL = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                             N->getOperand(1),
                             DAG.getConstant(0, dl, MVT::i32));
  SDValue RHSH = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                             N->getOperand(1),
                             DAG.getConstant(1, dl, MVT::i32));

```
- **EN**: Implements logic around `ExpandADDSUB`, `assert`, `getOpcode`, `dl`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `ExpandADDSUB`, `assert`, `getOpcode`, `dl`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 673-712
```cpp
  // Expand
  unsigned Opcode = (N->getOpcode() == ISD::ADD) ? XCoreISD::LADD :
                                                   XCoreISD::LSUB;
  SDValue Zero = DAG.getConstant(0, dl, MVT::i32);
  SDValue Lo = DAG.getNode(Opcode, dl, DAG.getVTList(MVT::i32, MVT::i32),
                           LHSL, RHSL, Zero);
  SDValue Carry(Lo.getNode(), 1);

  SDValue Hi = DAG.getNode(Opcode, dl, DAG.getVTList(MVT::i32, MVT::i32),
                           LHSH, RHSH, Carry);
  SDValue Ignored(Hi.getNode(), 1);
  // Merge the pieces
  return DAG.getNode(ISD::BUILD_PAIR, dl, MVT::i64, Lo, Hi);
}

SDValue XCoreTargetLowering::
LowerVAARG(SDValue Op, SelectionDAG &DAG) const
{
  // Whist llvm does not support aggregate varargs we can ignore
  // the possibility of the ValueType being an implicit byVal vararg.
  SDNode *Node = Op.getNode();
  EVT VT = Node->getValueType(0); // not an aggregate
  SDValue InChain = Node->getOperand(0);
  SDValue VAListPtr = Node->getOperand(1);
  EVT PtrVT = VAListPtr.getValueType();
  const Value *SV = cast<SrcValueSDNode>(Node->getOperand(2))->getValue();
  SDLoc dl(Node);
  SDValue VAList =
      DAG.getLoad(PtrVT, dl, InChain, VAListPtr, MachinePointerInfo(SV));
  // Increment the pointer, VAList, to the next vararg
  SDValue nextPtr = DAG.getNode(ISD::ADD, dl, PtrVT, VAList,
                                DAG.getIntPtrConstant(VT.getSizeInBits() / 8,
                                                      dl));
  // Store the incremented VAList to the legalized pointer
  InChain = DAG.getStore(VAList.getValue(1), dl, nextPtr, VAListPtr,
                         MachinePointerInfo(SV));
  // Load the actual argument out of the pointer VAList
  return DAG.getLoad(VT, dl, InChain, VAList, MachinePointerInfo());
}

```
- **EN**: Implements logic around `getOpcode`, `getConstant`, `getNode`, `Carry`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getOpcode`, `getConstant`, `getNode`, `Carry`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 713-735
```cpp
SDValue XCoreTargetLowering::
LowerVASTART(SDValue Op, SelectionDAG &DAG) const
{
  SDLoc dl(Op);
  // vastart stores the address of the VarArgsFrameIndex slot into the
  // memory location argument
  MachineFunction &MF = DAG.getMachineFunction();
  XCoreFunctionInfo *XFI = MF.getInfo<XCoreFunctionInfo>();
  SDValue Addr = DAG.getFrameIndex(XFI->getVarArgsFrameIndex(), MVT::i32);
  return DAG.getStore(Op.getOperand(0), dl, Addr, Op.getOperand(1),
                      MachinePointerInfo());
}

SDValue XCoreTargetLowering::LowerFRAMEADDR(SDValue Op,
                                            SelectionDAG &DAG) const {
  // This nodes represent llvm.frameaddress on the DAG.
  // It takes one operand, the index of the frame address to return.
  // An index of zero corresponds to the current function's frame address.
  // An index of one to the parent's frame address, and so on.
  // Depths > 0 not supported yet!
  if (Op.getConstantOperandVal(0) > 0)
    return SDValue();

```
- **EN**: Implements logic around `LowerVASTART`, `dl`, `getMachineFunction`, `getInfo<XCoreFunctionInfo>`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerVASTART`, `dl`, `getMachineFunction`, `getInfo<XCoreFunctionInfo>`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 736-760
```cpp
  MachineFunction &MF = DAG.getMachineFunction();
  const TargetRegisterInfo *RegInfo = Subtarget.getRegisterInfo();
  return DAG.getCopyFromReg(DAG.getEntryNode(), SDLoc(Op),
                            RegInfo->getFrameRegister(MF), MVT::i32);
}

SDValue XCoreTargetLowering::
LowerRETURNADDR(SDValue Op, SelectionDAG &DAG) const {
  // This nodes represent llvm.returnaddress on the DAG.
  // It takes one operand, the index of the return address to return.
  // An index of zero corresponds to the current function's return address.
  // An index of one to the parent's return address, and so on.
  // Depths > 0 not supported yet!
  if (Op.getConstantOperandVal(0) > 0)
    return SDValue();

  MachineFunction &MF = DAG.getMachineFunction();
  XCoreFunctionInfo *XFI = MF.getInfo<XCoreFunctionInfo>();
  int FI = XFI->createLRSpillSlot(MF);
  SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
  return DAG.getLoad(getPointerTy(DAG.getDataLayout()), SDLoc(Op),
                     DAG.getEntryNode(), FIN,
                     MachinePointerInfo::getFixedStack(MF, FI));
}

```
- **EN**: Implements logic around `getMachineFunction`, `getRegisterInfo`, `getCopyFromReg`, `getFrameRegister`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getMachineFunction`, `getRegisterInfo`, `getCopyFromReg`, `getFrameRegister`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 761-781
```cpp
SDValue XCoreTargetLowering::
LowerFRAME_TO_ARGS_OFFSET(SDValue Op, SelectionDAG &DAG) const {
  // This node represents offset from frame pointer to first on-stack argument.
  // This is needed for correct stack adjustment during unwind.
  // However, we don't know the offset until after the frame has be finalised.
  // This is done during the XCoreFTAOElim pass.
  return DAG.getNode(XCoreISD::FRAME_TO_ARGS_OFFSET, SDLoc(Op), MVT::i32);
}

SDValue XCoreTargetLowering::
LowerEH_RETURN(SDValue Op, SelectionDAG &DAG) const {
  // OUTCHAIN = EH_RETURN(INCHAIN, OFFSET, HANDLER)
  // This node represents 'eh_return' gcc dwarf builtin, which is used to
  // return from exception. The general meaning is: adjust stack by OFFSET and
  // pass execution to HANDLER.
  MachineFunction &MF = DAG.getMachineFunction();
  SDValue Chain     = Op.getOperand(0);
  SDValue Offset    = Op.getOperand(1);
  SDValue Handler   = Op.getOperand(2);
  SDLoc dl(Op);

```
- **EN**: Implements logic around `LowerFRAME_TO_ARGS_OFFSET`, `getNode`, `LowerEH_RETURN`, `getMachineFunction`, ...; this block returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerFRAME_TO_ARGS_OFFSET`, `getNode`, `LowerEH_RETURN`, `getMachineFunction`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 782-802
```cpp
  // Absolute SP = (FP + FrameToArgs) + Offset
  const TargetRegisterInfo *RegInfo = Subtarget.getRegisterInfo();
  SDValue Stack = DAG.getCopyFromReg(DAG.getEntryNode(), dl,
                            RegInfo->getFrameRegister(MF), MVT::i32);
  SDValue FrameToArgs = DAG.getNode(XCoreISD::FRAME_TO_ARGS_OFFSET, dl,
                                    MVT::i32);
  Stack = DAG.getNode(ISD::ADD, dl, MVT::i32, Stack, FrameToArgs);
  Stack = DAG.getNode(ISD::ADD, dl, MVT::i32, Stack, Offset);

  // R0=ExceptionPointerRegister R1=ExceptionSelectorRegister
  // which leaves 2 caller saved registers, R2 & R3 for us to use.
  unsigned StackReg = XCore::R2;
  unsigned HandlerReg = XCore::R3;

  SDValue OutChains[] = {
    DAG.getCopyToReg(Chain, dl, StackReg, Stack),
    DAG.getCopyToReg(Chain, dl, HandlerReg, Handler)
  };

  Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, OutChains);

```
- **EN**: Implements logic around `getRegisterInfo`, `getCopyFromReg`, `getFrameRegister`, `getNode`, ...; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getRegisterInfo`, `getCopyFromReg`, `getFrameRegister`, `getNode`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 803-822
```cpp
  return DAG.getNode(XCoreISD::EH_RETURN, dl, MVT::Other, Chain,
                     DAG.getRegister(StackReg, MVT::i32),
                     DAG.getRegister(HandlerReg, MVT::i32));

}

SDValue XCoreTargetLowering::
LowerADJUST_TRAMPOLINE(SDValue Op, SelectionDAG &DAG) const {
  return Op.getOperand(0);
}

SDValue XCoreTargetLowering::
LowerINIT_TRAMPOLINE(SDValue Op, SelectionDAG &DAG) const {
  SDValue Chain = Op.getOperand(0);
  SDValue Trmp = Op.getOperand(1); // trampoline
  SDValue FPtr = Op.getOperand(2); // nested function
  SDValue Nest = Op.getOperand(3); // 'nest' parameter value

  const Value *TrmpAddr = cast<SrcValueSDNode>(Op.getOperand(4))->getValue();

```
- **EN**: Implements logic around `getNode`, `getRegister`, `LowerADJUST_TRAMPOLINE`, `getOperand`, ...; this block returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `getRegister`, `LowerADJUST_TRAMPOLINE`, `getOperand`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 823-842
```cpp
  // .align 4
  // LDAPF_u10 r11, nest
  // LDW_2rus r11, r11[0]
  // STWSP_ru6 r11, sp[0]
  // LDAPF_u10 r11, fptr
  // LDW_2rus r11, r11[0]
  // BAU_1r r11
  // nest:
  // .word nest
  // fptr:
  // .word fptr
  SDValue OutChains[5];

  SDValue Addr = Trmp;

  SDLoc dl(Op);
  OutChains[0] =
      DAG.getStore(Chain, dl, DAG.getConstant(0x0a3cd805, dl, MVT::i32), Addr,
                   MachinePointerInfo(TrmpAddr));

```
- **EN**: Implements logic around `dl`, `getStore`, `MachinePointerInfo`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `dl`, `getStore`, `MachinePointerInfo` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 843-864
```cpp
  Addr = DAG.getNode(ISD::ADD, dl, MVT::i32, Trmp,
                     DAG.getConstant(4, dl, MVT::i32));
  OutChains[1] =
      DAG.getStore(Chain, dl, DAG.getConstant(0xd80456c0, dl, MVT::i32), Addr,
                   MachinePointerInfo(TrmpAddr, 4));

  Addr = DAG.getNode(ISD::ADD, dl, MVT::i32, Trmp,
                     DAG.getConstant(8, dl, MVT::i32));
  OutChains[2] =
      DAG.getStore(Chain, dl, DAG.getConstant(0x27fb0a3c, dl, MVT::i32), Addr,
                   MachinePointerInfo(TrmpAddr, 8));

  Addr = DAG.getNode(ISD::ADD, dl, MVT::i32, Trmp,
                     DAG.getConstant(12, dl, MVT::i32));
  OutChains[3] =
      DAG.getStore(Chain, dl, Nest, Addr, MachinePointerInfo(TrmpAddr, 12));

  Addr = DAG.getNode(ISD::ADD, dl, MVT::i32, Trmp,
                     DAG.getConstant(16, dl, MVT::i32));
  OutChains[4] =
      DAG.getStore(Chain, dl, FPtr, Addr, MachinePointerInfo(TrmpAddr, 16));

```
- **EN**: Implements logic around `getNode`, `getConstant`, `getStore`, `MachinePointerInfo`; this block handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `getConstant`, `getStore`, `MachinePointerInfo` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑。

### Lines 865-884
```cpp
  return DAG.getNode(ISD::TokenFactor, dl, MVT::Other, OutChains);
}

SDValue XCoreTargetLowering::
LowerINTRINSIC_WO_CHAIN(SDValue Op, SelectionDAG &DAG) const {
  SDLoc DL(Op);
  unsigned IntNo = Op.getConstantOperandVal(0);
  switch (IntNo) {
    case Intrinsic::xcore_crc8:
      EVT VT = Op.getValueType();
      SDValue Data =
        DAG.getNode(XCoreISD::CRC8, DL, DAG.getVTList(VT, VT),
                    Op.getOperand(1), Op.getOperand(2) , Op.getOperand(3));
      SDValue Crc(Data.getNode(), 1);
      SDValue Results[] = { Crc, Data };
      return DAG.getMergeValues(Results, DL);
  }
  return SDValue();
}

```
- **EN**: Implements logic around `getNode`, `LowerINTRINSIC_WO_CHAIN`, `DL`, `getConstantOperandVal`, ...; this block uses `switch`-based dispatch; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `LowerINTRINSIC_WO_CHAIN`, `DL`, `getConstantOperandVal`, ... 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 885-915
```cpp
SDValue XCoreTargetLowering::
LowerATOMIC_FENCE(SDValue Op, SelectionDAG &DAG) const {
  SDLoc DL(Op);
  return DAG.getNode(ISD::MEMBARRIER, DL, MVT::Other, Op.getOperand(0));
}

//===----------------------------------------------------------------------===//
//                      Calling Convention Implementation
//===----------------------------------------------------------------------===//

#include "XCoreGenCallingConv.inc"

//===----------------------------------------------------------------------===//
//                  Call Calling Convention Implementation
//===----------------------------------------------------------------------===//

/// XCore call implementation
SDValue
XCoreTargetLowering::LowerCall(TargetLowering::CallLoweringInfo &CLI,
                               SmallVectorImpl<SDValue> &InVals) const {
  SelectionDAG &DAG                     = CLI.DAG;
  SDLoc &dl                             = CLI.DL;
  SmallVectorImpl<ISD::OutputArg> &Outs = CLI.Outs;
  SmallVectorImpl<SDValue> &OutVals     = CLI.OutVals;
  SmallVectorImpl<ISD::InputArg> &Ins   = CLI.Ins;
  SDValue Chain                         = CLI.Chain;
  SDValue Callee                        = CLI.Callee;
  bool &isTailCall                      = CLI.IsTailCall;
  CallingConv::ID CallConv              = CLI.CallConv;
  bool isVarArg                         = CLI.IsVarArg;

```
- **EN**: Pulls in the headers needed for this implementation, including `XCoreGenCallingConv.inc`.
- **CN**: 引入该实现所需的头文件，其中包括 `XCoreGenCallingConv.inc`。

### Lines 916-953
```cpp
  // XCore target does not yet support tail call optimization.
  isTailCall = false;

  // For now, only CallingConv::C implemented
  switch (CallConv)
  {
    default:
      report_fatal_error("Unsupported calling convention");
    case CallingConv::Fast:
    case CallingConv::C:
      return LowerCCCCallTo(Chain, Callee, CallConv, isVarArg, isTailCall,
                            Outs, OutVals, Ins, dl, DAG, InVals);
  }
}

/// LowerCallResult - Lower the result values of a call into the
/// appropriate copies out of appropriate physical registers / memory locations.
static SDValue LowerCallResult(SDValue Chain, SDValue InGlue,
                               const SmallVectorImpl<CCValAssign> &RVLocs,
                               const SDLoc &dl, SelectionDAG &DAG,
                               SmallVectorImpl<SDValue> &InVals) {
  SmallVector<std::pair<int, unsigned>, 4> ResultMemLocs;
  // Copy results out of physical registers.
  for (const CCValAssign &VA : RVLocs) {
    if (VA.isRegLoc()) {
      Chain = DAG.getCopyFromReg(Chain, dl, VA.getLocReg(), VA.getValVT(),
                                 InGlue).getValue(1);
      InGlue = Chain.getValue(2);
      InVals.push_back(Chain.getValue(0));
    } else {
      assert(VA.isMemLoc());
      ResultMemLocs.push_back(std::make_pair(VA.getLocMemOffset(),
                                             InVals.size()));
      // Reserve space for this result.
      InVals.push_back(SDValue());
    }
  }

```
- **EN**: Implements logic around `report_fatal_error`, `LowerCCCCallTo`, `LowerCallResult`, `getCopyFromReg`, ...; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `report_fatal_error`, `LowerCCCCallTo`, `LowerCallResult`, `getCopyFromReg`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 954-973
```cpp
  // Copy results out of memory.
  SmallVector<SDValue, 4> MemOpChains;
  for (unsigned i = 0, e = ResultMemLocs.size(); i != e; ++i) {
    int offset = ResultMemLocs[i].first;
    unsigned index = ResultMemLocs[i].second;
    SDVTList VTs = DAG.getVTList(MVT::i32, MVT::Other);
    SDValue Ops[] = { Chain, DAG.getConstant(offset / 4, dl, MVT::i32) };
    SDValue load = DAG.getNode(XCoreISD::LDWSP, dl, VTs, Ops);
    InVals[index] = load;
    MemOpChains.push_back(load.getValue(1));
  }

  // Transform all loads nodes into one single node because
  // all load nodes are independent of each other.
  if (!MemOpChains.empty())
    Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, MemOpChains);

  return Chain;
}

```
- **EN**: Implements logic around `getVTList`, `getConstant`, `getNode`, `push_back`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getVTList`, `getConstant`, `getNode`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 974-993
```cpp
/// LowerCCCCallTo - functions arguments are copied from virtual
/// regs to (physical regs)/(stack frame), CALLSEQ_START and
/// CALLSEQ_END are emitted.
/// TODO: isTailCall, sret.
SDValue XCoreTargetLowering::LowerCCCCallTo(
    SDValue Chain, SDValue Callee, CallingConv::ID CallConv, bool isVarArg,
    bool isTailCall, const SmallVectorImpl<ISD::OutputArg> &Outs,
    const SmallVectorImpl<SDValue> &OutVals,
    const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &dl,
    SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {

  // Analyze operands of the call, assigning locations to each operand.
  SmallVector<CCValAssign, 16> ArgLocs;
  CCState CCInfo(CallConv, isVarArg, DAG.getMachineFunction(), ArgLocs,
                 *DAG.getContext());

  // The ABI dictates there should be one stack slot available to the callee
  // on function entry (for saving lr).
  CCInfo.AllocateStack(4, Align(4));

```
- **EN**: Implements logic around `LowerCCCCallTo`, `CCInfo`, `getContext`, `AllocateStack`; this block handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerCCCCallTo`, `CCInfo`, `getContext`, `AllocateStack` 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 994-1015
```cpp
  CCInfo.AnalyzeCallOperands(Outs, CC_XCore);

  SmallVector<CCValAssign, 16> RVLocs;
  // Analyze return values to determine the number of bytes of stack required.
  CCState RetCCInfo(CallConv, isVarArg, DAG.getMachineFunction(), RVLocs,
                    *DAG.getContext());
  RetCCInfo.AllocateStack(CCInfo.getStackSize(), Align(4));
  RetCCInfo.AnalyzeCallResult(Ins, RetCC_XCore);

  // Get a count of how many bytes are to be pushed on the stack.
  unsigned NumBytes = RetCCInfo.getStackSize();

  Chain = DAG.getCALLSEQ_START(Chain, NumBytes, 0, dl);

  SmallVector<std::pair<unsigned, SDValue>, 4> RegsToPass;
  SmallVector<SDValue, 12> MemOpChains;

  // Walk the register/memloc assignments, inserting copies/loads.
  for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {
    CCValAssign &VA = ArgLocs[i];
    SDValue Arg = OutVals[i];

```
- **EN**: Implements logic around `AnalyzeCallOperands`, `RetCCInfo`, `getContext`, `AllocateStack`, ...; this block returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `AnalyzeCallOperands`, `RetCCInfo`, `getContext`, `AllocateStack`, ... 实现具体逻辑；这一段返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1016-1037
```cpp
    // Promote the value if needed.
    switch (VA.getLocInfo()) {
      default: llvm_unreachable("Unknown loc info!");
      case CCValAssign::Full: break;
      case CCValAssign::SExt:
        Arg = DAG.getNode(ISD::SIGN_EXTEND, dl, VA.getLocVT(), Arg);
        break;
      case CCValAssign::ZExt:
        Arg = DAG.getNode(ISD::ZERO_EXTEND, dl, VA.getLocVT(), Arg);
        break;
      case CCValAssign::AExt:
        Arg = DAG.getNode(ISD::ANY_EXTEND, dl, VA.getLocVT(), Arg);
        break;
    }

    // Arguments that can be passed on register must be kept at
    // RegsToPass vector
    if (VA.isRegLoc()) {
      RegsToPass.push_back(std::make_pair(VA.getLocReg(), Arg));
    } else {
      assert(VA.isMemLoc());

```
- **EN**: Implements logic around `llvm_unreachable`, `getNode`, `push_back`, `assert`; this block uses `switch`-based dispatch; applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `llvm_unreachable`, `getNode`, `push_back`, `assert` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 1038-1061
```cpp
      int Offset = VA.getLocMemOffset();

      MemOpChains.push_back(DAG.getNode(XCoreISD::STWSP, dl, MVT::Other,
                                        Chain, Arg,
                                        DAG.getConstant(Offset/4, dl,
                                                        MVT::i32)));
    }
  }

  // Transform all store nodes into one single node because
  // all store nodes are independent of each other.
  if (!MemOpChains.empty())
    Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, MemOpChains);

  // Build a sequence of copy-to-reg nodes chained together with token
  // chain and flag operands which copy the outgoing args into registers.
  // The InGlue in necessary since all emitted instructions must be
  // stuck together.
  SDValue InGlue;
  for (const auto &[Reg, N] : RegsToPass) {
    Chain = DAG.getCopyToReg(Chain, dl, Reg, N, InGlue);
    InGlue = Chain.getValue(1);
  }

```
- **EN**: Implements logic around `getLocMemOffset`, `push_back`, `getConstant`, `getNode`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getLocMemOffset`, `push_back`, `getConstant`, `getNode`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 1062-1083
```cpp
  // If the callee is a GlobalAddress node (quite common, every direct call is)
  // turn it into a TargetGlobalAddress node so that legalize doesn't hack it.
  // Likewise ExternalSymbol -> TargetExternalSymbol.
  if (GlobalAddressSDNode *G = dyn_cast<GlobalAddressSDNode>(Callee))
    Callee = DAG.getTargetGlobalAddress(G->getGlobal(), dl, MVT::i32);
  else if (ExternalSymbolSDNode *E = dyn_cast<ExternalSymbolSDNode>(Callee))
    Callee = DAG.getTargetExternalSymbol(E->getSymbol(), MVT::i32);

  // XCoreBranchLink = #chain, #target_address, #opt_in_flags...
  //             = Chain, Callee, Reg#1, Reg#2, ...
  //
  // Returns a chain & a flag for retval copy to use.
  SDVTList NodeTys = DAG.getVTList(MVT::Other, MVT::Glue);
  SmallVector<SDValue, 8> Ops;
  Ops.push_back(Chain);
  Ops.push_back(Callee);

  // Add argument registers to the end of the list so that they are
  // known live into the call.
  for (const auto &[Reg, N] : RegsToPass)
    Ops.push_back(DAG.getRegister(Reg, N.getValueType()));

```
- **EN**: Implements logic around `getTargetGlobalAddress`, `getTargetExternalSymbol`, `getVTList`, `push_back`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getTargetGlobalAddress`, `getTargetExternalSymbol`, `getVTList`, `push_back` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 1084-1106
```cpp
  if (InGlue.getNode())
    Ops.push_back(InGlue);

  Chain  = DAG.getNode(XCoreISD::BL, dl, NodeTys, Ops);
  InGlue = Chain.getValue(1);

  // Create the CALLSEQ_END node.
  Chain = DAG.getCALLSEQ_END(Chain, NumBytes, 0, InGlue, dl);
  InGlue = Chain.getValue(1);

  // Handle result values, copying them out of physregs into vregs that we
  // return.
  return LowerCallResult(Chain, InGlue, RVLocs, dl, DAG, InVals);
}

//===----------------------------------------------------------------------===//
//             Formal Arguments Calling Convention Implementation
//===----------------------------------------------------------------------===//

namespace {
  struct ArgDataPair { SDValue SDV; ISD::ArgFlagsTy Flags; };
}

```
- **EN**: Introduces declarations for `ArgDataPair`, defining the data structures or interfaces used later in the file.
- **CN**: 引入 `ArgDataPair` 等声明，定义本文件后续使用的数据结构或接口。

### Lines 1107-1126
```cpp
/// XCore formal arguments implementation
SDValue XCoreTargetLowering::LowerFormalArguments(
    SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
    const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &dl,
    SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
  switch (CallConv)
  {
    default:
      report_fatal_error("Unsupported calling convention");
    case CallingConv::C:
    case CallingConv::Fast:
      return LowerCCCArguments(Chain, CallConv, isVarArg,
                               Ins, dl, DAG, InVals);
  }
}

/// LowerCCCArguments - transform physical registers into
/// virtual registers and generate load operations for
/// arguments places on the stack.
/// TODO: sret
```
- **EN**: Implements logic around `LowerFormalArguments`, `report_fatal_error`, `LowerCCCArguments`; this block uses `switch`-based dispatch; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `LowerFormalArguments`, `report_fatal_error`, `LowerCCCArguments` 实现具体逻辑；这一段使用 `switch` 分派，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1127-1146
```cpp
SDValue XCoreTargetLowering::LowerCCCArguments(
    SDValue Chain, CallingConv::ID CallConv, bool isVarArg,
    const SmallVectorImpl<ISD::InputArg> &Ins, const SDLoc &dl,
    SelectionDAG &DAG, SmallVectorImpl<SDValue> &InVals) const {
  MachineFunction &MF = DAG.getMachineFunction();
  MachineFrameInfo &MFI = MF.getFrameInfo();
  MachineRegisterInfo &RegInfo = MF.getRegInfo();
  XCoreFunctionInfo *XFI = MF.getInfo<XCoreFunctionInfo>();

  // Assign locations to all of the incoming arguments.
  SmallVector<CCValAssign, 16> ArgLocs;
  CCState CCInfo(CallConv, isVarArg, DAG.getMachineFunction(), ArgLocs,
                 *DAG.getContext());

  CCInfo.AnalyzeFormalArguments(Ins, CC_XCore);

  unsigned StackSlotSize = XCoreFrameLowering::stackSlotSize();

  unsigned LRSaveSize = StackSlotSize;

```
- **EN**: Implements logic around `LowerCCCArguments`, `getMachineFunction`, `getFrameInfo`, `getRegInfo`, ...; this block handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerCCCArguments`, `getMachineFunction`, `getFrameInfo`, `getRegInfo`, ... 实现具体逻辑；这一段处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1147-1166
```cpp
  if (!isVarArg)
    XFI->setReturnStackOffset(CCInfo.getStackSize() + LRSaveSize);

  // All getCopyFromReg ops must precede any getMemcpys to prevent the
  // scheduler clobbering a register before it has been copied.
  // The stages are:
  // 1. CopyFromReg (and load) arg & vararg registers.
  // 2. Chain CopyFromReg nodes into a TokenFactor.
  // 3. Memcpy 'byVal' args & push final InVals.
  // 4. Chain mem ops nodes into a TokenFactor.
  SmallVector<SDValue, 4> CFRegNode;
  SmallVector<ArgDataPair, 4> ArgData;
  SmallVector<SDValue, 4> MemOps;

  // 1a. CopyFromReg (and load) arg registers.
  for (unsigned i = 0, e = ArgLocs.size(); i != e; ++i) {

    CCValAssign &VA = ArgLocs[i];
    SDValue ArgIn;

```
- **EN**: Implements logic around `setReturnStackOffset`; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `setReturnStackOffset` 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 1167-1198
```cpp
    if (VA.isRegLoc()) {
      // Arguments passed in registers
      EVT RegVT = VA.getLocVT();
      switch (RegVT.getSimpleVT().SimpleTy) {
      default:
        {
#ifndef NDEBUG
          errs() << "LowerFormalArguments Unhandled argument type: "
                 << RegVT << "\n";
#endif
          llvm_unreachable(nullptr);
        }
      case MVT::i32:
        Register VReg = RegInfo.createVirtualRegister(&XCore::GRRegsRegClass);
        RegInfo.addLiveIn(VA.getLocReg(), VReg);
        ArgIn = DAG.getCopyFromReg(Chain, dl, VReg, RegVT);
        CFRegNode.push_back(ArgIn.getValue(ArgIn->getNumValues() - 1));
      }
    } else {
      // Only arguments passed on the stack should make it here. 
      assert(VA.isMemLoc());
      // Load the argument to a virtual register
      unsigned ObjSize = VA.getLocVT().getSizeInBits()/8;
      if (ObjSize > StackSlotSize) {
        errs() << "LowerFormalArguments Unhandled argument type: "
               << VA.getLocVT() << "\n";
      }
      // Create the frame index object for this incoming parameter...
      int FI = MFI.CreateFixedObject(ObjSize,
                                     LRSaveSize + VA.getLocMemOffset(),
                                     true);

```
- **EN**: Defines preprocessor macros or compile-time switches used by the surrounding implementation.
- **CN**: 定义周边实现所需的预处理宏或编译期开关。

### Lines 1199-1238
```cpp
      // Create the SelectionDAG nodes corresponding to a load
      //from this parameter
      SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
      ArgIn = DAG.getLoad(VA.getLocVT(), dl, Chain, FIN,
                          MachinePointerInfo::getFixedStack(MF, FI));
    }
    const ArgDataPair ADP = { ArgIn, Ins[i].Flags };
    ArgData.push_back(ADP);
  }

  // 1b. CopyFromReg vararg registers.
  if (isVarArg) {
    // Argument registers
    static const MCPhysReg ArgRegs[] = {
      XCore::R0, XCore::R1, XCore::R2, XCore::R3
    };
    XCoreFunctionInfo *XFI = MF.getInfo<XCoreFunctionInfo>();
    unsigned FirstVAReg = CCInfo.getFirstUnallocated(ArgRegs);
    if (FirstVAReg < std::size(ArgRegs)) {
      int offset = 0;
      // Save remaining registers, storing higher register numbers at a higher
      // address
      for (int i = std::size(ArgRegs) - 1; i >= (int)FirstVAReg; --i) {
        // Create a stack slot
        int FI = MFI.CreateFixedObject(4, offset, true);
        if (i == (int)FirstVAReg) {
          XFI->setVarArgsFrameIndex(FI);
        }
        offset -= StackSlotSize;
        SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
        // Move argument from phys reg -> virt reg
        Register VReg = RegInfo.createVirtualRegister(&XCore::GRRegsRegClass);
        RegInfo.addLiveIn(ArgRegs[i], VReg);
        SDValue Val = DAG.getCopyFromReg(Chain, dl, VReg, MVT::i32);
        CFRegNode.push_back(Val.getValue(Val->getNumValues() - 1));
        // Move argument from virt reg -> stack
        SDValue Store =
            DAG.getStore(Val.getValue(1), dl, Val, FIN, MachinePointerInfo());
        MemOps.push_back(Store);
      }
```
- **EN**: Implements logic around `getFrameIndex`, `getLoad`, `getFixedStack`, `push_back`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getFrameIndex`, `getLoad`, `getFixedStack`, `push_back`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 1239-1271
```cpp
    } else {
      // This will point to the next argument passed via stack.
      XFI->setVarArgsFrameIndex(
          MFI.CreateFixedObject(4, LRSaveSize + CCInfo.getStackSize(), true));
    }
  }

  // 2. chain CopyFromReg nodes into a TokenFactor.
  if (!CFRegNode.empty())
    Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, CFRegNode);

  // 3. Memcpy 'byVal' args & push final InVals.
  // Aggregates passed "byVal" need to be copied by the callee.
  // The callee will use a pointer to this copy, rather than the original
  // pointer.
  for (const ArgDataPair &ArgDI : ArgData) {
    if (ArgDI.Flags.isByVal() && ArgDI.Flags.getByValSize()) {
      unsigned Size = ArgDI.Flags.getByValSize();
      Align Alignment =
          std::max(Align(StackSlotSize), ArgDI.Flags.getNonZeroByValAlign());
      // Create a new object on the stack and copy the pointee into it.
      int FI = MFI.CreateStackObject(Size, Alignment, false);
      SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
      InVals.push_back(FIN);
      MemOps.push_back(DAG.getMemcpy(
          Chain, dl, FIN, ArgDI.SDV, DAG.getConstant(Size, dl, MVT::i32),
          Alignment, false, false, /*CI=*/nullptr, std::nullopt,
          MachinePointerInfo(), MachinePointerInfo()));
    } else {
      InVals.push_back(ArgDI.SDV);
    }
  }

```
- **EN**: Implements logic around `setVarArgsFrameIndex`, `CreateFixedObject`, `getNode`, `getByValSize`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `setVarArgsFrameIndex`, `CreateFixedObject`, `getNode`, `getByValSize`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 1272-1298
```cpp
  // 4, chain mem ops nodes into a TokenFactor.
  if (!MemOps.empty()) {
    MemOps.push_back(Chain);
    Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, MemOps);
  }

  return Chain;
}

//===----------------------------------------------------------------------===//
//               Return Value Calling Convention Implementation
//===----------------------------------------------------------------------===//

bool XCoreTargetLowering::
CanLowerReturn(CallingConv::ID CallConv, MachineFunction &MF,
               bool isVarArg,
               const SmallVectorImpl<ISD::OutputArg> &Outs,
               LLVMContext &Context, const Type *RetTy) const {
  SmallVector<CCValAssign, 16> RVLocs;
  CCState CCInfo(CallConv, isVarArg, MF, RVLocs, Context);
  if (!CCInfo.CheckReturn(Outs, RetCC_XCore))
    return false;
  if (CCInfo.getStackSize() != 0 && isVarArg)
    return false;
  return true;
}

```
- **EN**: Implements logic around `push_back`, `getNode`, `CanLowerReturn`, `CCInfo`; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `push_back`, `getNode`, `CanLowerReturn`, `CCInfo` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1299-1321
```cpp
SDValue
XCoreTargetLowering::LowerReturn(SDValue Chain, CallingConv::ID CallConv,
                                 bool isVarArg,
                                 const SmallVectorImpl<ISD::OutputArg> &Outs,
                                 const SmallVectorImpl<SDValue> &OutVals,
                                 const SDLoc &dl, SelectionDAG &DAG) const {

  XCoreFunctionInfo *XFI =
    DAG.getMachineFunction().getInfo<XCoreFunctionInfo>();
  MachineFrameInfo &MFI = DAG.getMachineFunction().getFrameInfo();

  // CCValAssign - represent the assignment of
  // the return value to a location
  SmallVector<CCValAssign, 16> RVLocs;

  // CCState - Info about the registers and stack slot.
  CCState CCInfo(CallConv, isVarArg, DAG.getMachineFunction(), RVLocs,
                 *DAG.getContext());

  // Analyze return values.
  if (!isVarArg)
    CCInfo.AllocateStack(XFI->getReturnStackOffset(), Align(4));

```
- **EN**: Implements logic around `LowerReturn`, `getMachineFunction`, `CCInfo`, `getContext`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `LowerReturn`, `getMachineFunction`, `CCInfo`, `getContext`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1322-1345
```cpp
  CCInfo.AnalyzeReturn(Outs, RetCC_XCore);

  SDValue Glue;
  SmallVector<SDValue, 4> RetOps(1, Chain);

  // Return on XCore is always a "retsp 0"
  RetOps.push_back(DAG.getConstant(0, dl, MVT::i32));

  SmallVector<SDValue, 4> MemOpChains;
  // Handle return values that must be copied to memory.
  for (unsigned i = 0, e = RVLocs.size(); i != e; ++i) {
    CCValAssign &VA = RVLocs[i];
    if (VA.isRegLoc())
      continue;
    assert(VA.isMemLoc());
    if (isVarArg) {
      report_fatal_error("Can't return value from vararg function in memory");
    }

    int Offset = VA.getLocMemOffset();
    unsigned ObjSize = VA.getLocVT().getSizeInBits() / 8;
    // Create the frame index object for the memory location.
    int FI = MFI.CreateFixedObject(ObjSize, Offset, false);

```
- **EN**: Implements logic around `AnalyzeReturn`, `RetOps`, `push_back`, `assert`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `AnalyzeReturn`, `RetOps`, `push_back`, `assert`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1346-1366
```cpp
    // Create a SelectionDAG node corresponding to a store
    // to this memory location.
    SDValue FIN = DAG.getFrameIndex(FI, MVT::i32);
    MemOpChains.push_back(DAG.getStore(
        Chain, dl, OutVals[i], FIN,
        MachinePointerInfo::getFixedStack(DAG.getMachineFunction(), FI)));
  }

  // Transform all store nodes into one single node because
  // all stores are independent of each other.
  if (!MemOpChains.empty())
    Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, MemOpChains);

  // Now handle return values copied to registers.
  for (unsigned i = 0, e = RVLocs.size(); i != e; ++i) {
    CCValAssign &VA = RVLocs[i];
    if (!VA.isRegLoc())
      continue;
    // Copy the result values into the output registers.
    Chain = DAG.getCopyToReg(Chain, dl, VA.getLocReg(), OutVals[i], Glue);

```
- **EN**: Implements logic around `getFrameIndex`, `push_back`, `getFixedStack`, `getNode`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getFrameIndex`, `push_back`, `getFixedStack`, `getNode`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1367-1393
```cpp
    // guarantee that all emitted copies are
    // stuck together, avoiding something bad
    Glue = Chain.getValue(1);
    RetOps.push_back(DAG.getRegister(VA.getLocReg(), VA.getLocVT()));
  }

  RetOps[0] = Chain;  // Update chain.

  // Add the glue if we have it.
  if (Glue.getNode())
    RetOps.push_back(Glue);

  return DAG.getNode(XCoreISD::RETSP, dl, MVT::Other, RetOps);
}

//===----------------------------------------------------------------------===//
//  Other Lowering Code
//===----------------------------------------------------------------------===//

MachineBasicBlock *
XCoreTargetLowering::EmitInstrWithCustomInserter(MachineInstr &MI,
                                                 MachineBasicBlock *BB) const {
  const TargetInstrInfo &TII = *Subtarget.getInstrInfo();
  DebugLoc dl = MI.getDebugLoc();
  assert((MI.getOpcode() == XCore::SELECT_CC) &&
         "Unexpected instr type to insert");

```
- **EN**: Implements logic around `getValue`, `push_back`, `getNode`, `EmitInstrWithCustomInserter`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getValue`, `push_back`, `getNode`, `EmitInstrWithCustomInserter`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1394-1413
```cpp
  // To "insert" a SELECT_CC instruction, we actually have to insert the diamond
  // control-flow pattern.  The incoming instruction knows the destination vreg
  // to set, the condition code register to branch on, the true/false values to
  // select between, and a branch opcode to use.
  const BasicBlock *LLVM_BB = BB->getBasicBlock();
  MachineFunction::iterator It = ++BB->getIterator();

  //  thisMBB:
  //  ...
  //   TrueVal = ...
  //   cmpTY ccX, r1, r2
  //   bCC copy1MBB
  //   fallthrough --> copy0MBB
  MachineBasicBlock *thisMBB = BB;
  MachineFunction *F = BB->getParent();
  MachineBasicBlock *copy0MBB = F->CreateMachineBasicBlock(LLVM_BB);
  MachineBasicBlock *sinkMBB = F->CreateMachineBasicBlock(LLVM_BB);
  F->insert(It, copy0MBB);
  F->insert(It, sinkMBB);

```
- **EN**: Implements logic around `getBasicBlock`, `getIterator`, `getParent`, `CreateMachineBasicBlock`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `getBasicBlock`, `getIterator`, `getParent`, `CreateMachineBasicBlock`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 1414-1434
```cpp
  // Transfer the remainder of BB and its successor edges to sinkMBB.
  sinkMBB->splice(sinkMBB->begin(), BB,
                  std::next(MachineBasicBlock::iterator(MI)), BB->end());
  sinkMBB->transferSuccessorsAndUpdatePHIs(BB);

  // Next, add the true and fallthrough blocks as its successors.
  BB->addSuccessor(copy0MBB);
  BB->addSuccessor(sinkMBB);

  BuildMI(BB, dl, TII.get(XCore::BRFT_lru6))
      .addReg(MI.getOperand(1).getReg())
      .addMBB(sinkMBB);

  //  copy0MBB:
  //   %FalseValue = ...
  //   # fallthrough to sinkMBB
  BB = copy0MBB;

  // Update machine-CFG edges
  BB->addSuccessor(sinkMBB);

```
- **EN**: Implements logic around `splice`, `next`, `transferSuccessorsAndUpdatePHIs`, `addSuccessor`, ...; this block works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `splice`, `next`, `transferSuccessorsAndUpdatePHIs`, `addSuccessor`, ... 实现具体逻辑；这一段工作在 MachineInstr/MachineFunction 层。

### Lines 1435-1474
```cpp
  //  sinkMBB:
  //   %Result = phi [ %FalseValue, copy0MBB ], [ %TrueValue, thisMBB ]
  //  ...
  BB = sinkMBB;
  BuildMI(*BB, BB->begin(), dl, TII.get(XCore::PHI), MI.getOperand(0).getReg())
      .addReg(MI.getOperand(3).getReg())
      .addMBB(copy0MBB)
      .addReg(MI.getOperand(2).getReg())
      .addMBB(thisMBB);

  MI.eraseFromParent(); // The pseudo instruction is gone now.
  return BB;
}

//===----------------------------------------------------------------------===//
// Target Optimization Hooks
//===----------------------------------------------------------------------===//

SDValue XCoreTargetLowering::PerformDAGCombine(SDNode *N,
                                             DAGCombinerInfo &DCI) const {
  SelectionDAG &DAG = DCI.DAG;
  SDLoc dl(N);
  switch (N->getOpcode()) {
  default: break;
  case ISD::INTRINSIC_VOID:
    switch (N->getConstantOperandVal(1)) {
    case Intrinsic::xcore_outt:
    case Intrinsic::xcore_outct:
    case Intrinsic::xcore_chkct: {
      SDValue OutVal = N->getOperand(3);
      // These instructions ignore the high bits.
      if (OutVal.hasOneUse()) {
        unsigned BitWidth = OutVal.getValueSizeInBits();
        APInt DemandedMask = APInt::getLowBitsSet(BitWidth, 8);
        KnownBits Known;
        TargetLowering::TargetLoweringOpt TLO(DAG, !DCI.isBeforeLegalize(),
                                              !DCI.isBeforeLegalizeOps());
        const TargetLowering &TLI = DAG.getTargetLoweringInfo();
        if (TLI.ShrinkDemandedConstant(OutVal, DemandedMask, TLO) ||
            TLI.SimplifyDemandedBits(OutVal, DemandedMask, Known, TLO))
```
- **EN**: Implements logic around `BuildMI`, `addReg`, `addMBB`, `eraseFromParent`, ...; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic; works at the MachineInstr/MachineFunction layer.
- **CN**: 围绕 `BuildMI`, `addReg`, `addMBB`, `eraseFromParent`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑，工作在 MachineInstr/MachineFunction 层。

### Lines 1475-1504
```cpp
          DCI.CommitTargetLoweringOpt(TLO);
      }
      break;
    }
    case Intrinsic::xcore_setpt: {
      SDValue Time = N->getOperand(3);
      // This instruction ignores the high bits.
      if (Time.hasOneUse()) {
        unsigned BitWidth = Time.getValueSizeInBits();
        APInt DemandedMask = APInt::getLowBitsSet(BitWidth, 16);
        KnownBits Known;
        TargetLowering::TargetLoweringOpt TLO(DAG, !DCI.isBeforeLegalize(),
                                              !DCI.isBeforeLegalizeOps());
        const TargetLowering &TLI = DAG.getTargetLoweringInfo();
        if (TLI.ShrinkDemandedConstant(Time, DemandedMask, TLO) ||
            TLI.SimplifyDemandedBits(Time, DemandedMask, Known, TLO))
          DCI.CommitTargetLoweringOpt(TLO);
      }
      break;
    }
    }
    break;
  case XCoreISD::LADD: {
    SDValue N0 = N->getOperand(0);
    SDValue N1 = N->getOperand(1);
    SDValue N2 = N->getOperand(2);
    ConstantSDNode *N0C = dyn_cast<ConstantSDNode>(N0);
    ConstantSDNode *N1C = dyn_cast<ConstantSDNode>(N1);
    EVT VT = N0.getValueType();

```
- **EN**: Implements logic around `CommitTargetLoweringOpt`, `getOperand`, `getValueSizeInBits`, `getLowBitsSet`, ...; this block applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `CommitTargetLoweringOpt`, `getOperand`, `getValueSizeInBits`, `getLowBitsSet`, ... 实现具体逻辑；这一段应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 1505-1540
```cpp
    // canonicalize constant to RHS
    if (N0C && !N1C)
      return DAG.getNode(XCoreISD::LADD, dl, DAG.getVTList(VT, VT), N1, N0, N2);

    // fold (ladd 0, 0, x) -> 0, x & 1
    if (N0C && N0C->isZero() && N1C && N1C->isZero()) {
      SDValue Carry = DAG.getConstant(0, dl, VT);
      SDValue Result = DAG.getNode(ISD::AND, dl, VT, N2,
                                   DAG.getConstant(1, dl, VT));
      SDValue Ops[] = { Result, Carry };
      return DAG.getMergeValues(Ops, dl);
    }

    // fold (ladd x, 0, y) -> 0, add x, y iff carry is unused and y has only the
    // low bit set
    if (N1C && N1C->isZero() && N->hasNUsesOfValue(0, 1)) {
      APInt Mask = APInt::getHighBitsSet(VT.getSizeInBits(),
                                         VT.getSizeInBits() - 1);
      KnownBits Known = DAG.computeKnownBits(N2);
      if ((Known.Zero & Mask) == Mask) {
        SDValue Carry = DAG.getConstant(0, dl, VT);
        SDValue Result = DAG.getNode(ISD::ADD, dl, VT, N0, N2);
        SDValue Ops[] = { Result, Carry };
        return DAG.getMergeValues(Ops, dl);
      }
    }
  }
  break;
  case XCoreISD::LSUB: {
    SDValue N0 = N->getOperand(0);
    SDValue N1 = N->getOperand(1);
    SDValue N2 = N->getOperand(2);
    ConstantSDNode *N0C = dyn_cast<ConstantSDNode>(N0);
    ConstantSDNode *N1C = dyn_cast<ConstantSDNode>(N1);
    EVT VT = N0.getValueType();

```
- **EN**: Implements logic around `getNode`, `getConstant`, `getMergeValues`, `getHighBitsSet`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getNode`, `getConstant`, `getMergeValues`, `getHighBitsSet`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1541-1580
```cpp
    // fold (lsub 0, 0, x) -> x, -x iff x has only the low bit set
    if (N0C && N0C->isZero() && N1C && N1C->isZero()) {
      APInt Mask = APInt::getHighBitsSet(VT.getSizeInBits(),
                                         VT.getSizeInBits() - 1);
      KnownBits Known = DAG.computeKnownBits(N2);
      if ((Known.Zero & Mask) == Mask) {
        SDValue Borrow = N2;
        SDValue Result = DAG.getNode(ISD::SUB, dl, VT,
                                     DAG.getConstant(0, dl, VT), N2);
        SDValue Ops[] = { Result, Borrow };
        return DAG.getMergeValues(Ops, dl);
      }
    }

    // fold (lsub x, 0, y) -> 0, sub x, y iff borrow is unused and y has only the
    // low bit set
    if (N1C && N1C->isZero() && N->hasNUsesOfValue(0, 1)) {
      APInt Mask = APInt::getHighBitsSet(VT.getSizeInBits(),
                                         VT.getSizeInBits() - 1);
      KnownBits Known = DAG.computeKnownBits(N2);
      if ((Known.Zero & Mask) == Mask) {
        SDValue Borrow = DAG.getConstant(0, dl, VT);
        SDValue Result = DAG.getNode(ISD::SUB, dl, VT, N0, N2);
        SDValue Ops[] = { Result, Borrow };
        return DAG.getMergeValues(Ops, dl);
      }
    }
  }
  break;
  case XCoreISD::LMUL: {
    SDValue N0 = N->getOperand(0);
    SDValue N1 = N->getOperand(1);
    SDValue N2 = N->getOperand(2);
    SDValue N3 = N->getOperand(3);
    ConstantSDNode *N0C = dyn_cast<ConstantSDNode>(N0);
    ConstantSDNode *N1C = dyn_cast<ConstantSDNode>(N1);
    EVT VT = N0.getValueType();
    // Canonicalize multiplicative constant to RHS. If both multiplicative
    // operands are constant canonicalize smallest to RHS.
    if ((N0C && !N1C) ||
```
- **EN**: Implements logic around `getHighBitsSet`, `getSizeInBits`, `computeKnownBits`, `getNode`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getHighBitsSet`, `getSizeInBits`, `computeKnownBits`, `getNode`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1581-1620
```cpp
        (N0C && N1C && N0C->getZExtValue() < N1C->getZExtValue()))
      return DAG.getNode(XCoreISD::LMUL, dl, DAG.getVTList(VT, VT),
                         N1, N0, N2, N3);

    // lmul(x, 0, a, b)
    if (N1C && N1C->isZero()) {
      // If the high result is unused fold to add(a, b)
      if (N->hasNUsesOfValue(0, 0)) {
        SDValue Lo = DAG.getNode(ISD::ADD, dl, VT, N2, N3);
        SDValue Ops[] = { Lo, Lo };
        return DAG.getMergeValues(Ops, dl);
      }
      // Otherwise fold to ladd(a, b, 0)
      SDValue Result =
        DAG.getNode(XCoreISD::LADD, dl, DAG.getVTList(VT, VT), N2, N3, N1);
      SDValue Carry(Result.getNode(), 1);
      SDValue Ops[] = { Carry, Result };
      return DAG.getMergeValues(Ops, dl);
    }
  }
  break;
  case ISD::ADD: {
    // Fold 32 bit expressions such as add(add(mul(x,y),a),b) ->
    // lmul(x, y, a, b). The high result of lmul will be ignored.
    // This is only profitable if the intermediate results are unused
    // elsewhere.
    SDValue Mul0, Mul1, Addend0, Addend1;
    if (N->getValueType(0) == MVT::i32 &&
        isADDADDMUL(SDValue(N, 0), Mul0, Mul1, Addend0, Addend1, true)) {
      SDValue Ignored = DAG.getNode(XCoreISD::LMUL, dl,
                                    DAG.getVTList(MVT::i32, MVT::i32), Mul0,
                                    Mul1, Addend0, Addend1);
      SDValue Result(Ignored.getNode(), 1);
      return Result;
    }
    APInt HighMask = APInt::getHighBitsSet(64, 32);
    // Fold 64 bit expression such as add(add(mul(x,y),a),b) ->
    // lmul(x, y, a, b) if all operands are zero-extended. We do this
    // before type legalization as it is messy to match the operands after
    // that.
```
- **EN**: Implements logic around `getZExtValue`, `getNode`, `getMergeValues`, `Carry`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getZExtValue`, `getNode`, `getMergeValues`, `Carry`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1621-1654
```cpp
    if (N->getValueType(0) == MVT::i64 &&
        isADDADDMUL(SDValue(N, 0), Mul0, Mul1, Addend0, Addend1, false) &&
        DAG.MaskedValueIsZero(Mul0, HighMask) &&
        DAG.MaskedValueIsZero(Mul1, HighMask) &&
        DAG.MaskedValueIsZero(Addend0, HighMask) &&
        DAG.MaskedValueIsZero(Addend1, HighMask)) {
      SDValue Mul0L = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                                  Mul0, DAG.getConstant(0, dl, MVT::i32));
      SDValue Mul1L = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                                  Mul1, DAG.getConstant(0, dl, MVT::i32));
      SDValue Addend0L = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                                     Addend0, DAG.getConstant(0, dl, MVT::i32));
      SDValue Addend1L = DAG.getNode(ISD::EXTRACT_ELEMENT, dl, MVT::i32,
                                     Addend1, DAG.getConstant(0, dl, MVT::i32));
      SDValue Hi = DAG.getNode(XCoreISD::LMUL, dl,
                               DAG.getVTList(MVT::i32, MVT::i32), Mul0L, Mul1L,
                               Addend0L, Addend1L);
      SDValue Lo(Hi.getNode(), 1);
      return DAG.getNode(ISD::BUILD_PAIR, dl, MVT::i64, Lo, Hi);
    }
  }
  break;
  case ISD::STORE: {
    // Replace unaligned store of unaligned load with memmove.
    StoreSDNode *ST = cast<StoreSDNode>(N);
    if (!DCI.isBeforeLegalize() ||
        allowsMemoryAccessForAlignment(*DAG.getContext(), DAG.getDataLayout(),
                                       ST->getMemoryVT(),
                                       *ST->getMemOperand()) ||
        ST->isVolatile() || ST->isIndexed()) {
      break;
    }
    SDValue Chain = ST->getChain();

```
- **EN**: Implements logic around `isADDADDMUL`, `MaskedValueIsZero`, `getNode`, `getConstant`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `isADDADDMUL`, `MaskedValueIsZero`, `getNode`, `getConstant`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1655-1677
```cpp
    unsigned StoreBits = ST->getMemoryVT().getStoreSizeInBits();
    assert((StoreBits % 8) == 0 &&
           "Store size in bits must be a multiple of 8");
    Align Alignment = ST->getAlign();

    if (LoadSDNode *LD = dyn_cast<LoadSDNode>(ST->getValue())) {
      if (LD->hasNUsesOfValue(1, 0) && ST->getMemoryVT() == LD->getMemoryVT() &&
        LD->getAlign() == Alignment &&
        !LD->isVolatile() && !LD->isIndexed() &&
        Chain.reachesChainWithoutSideEffects(SDValue(LD, 1))) {
        bool isTail = isInTailCallPosition(DAG, ST, Chain);
        return DAG.getMemmove(Chain, dl, ST->getBasePtr(), LD->getBasePtr(),
                              DAG.getConstant(StoreBits / 8, dl, MVT::i32),
                              Alignment, false, nullptr, isTail,
                              ST->getPointerInfo(), LD->getPointerInfo());
      }
    }
    break;
  }
  }
  return SDValue();
}

```
- **EN**: Implements logic around `getMemoryVT`, `assert`, `getAlign`, `isVolatile`, ...; this block applies conditional target rules; returns target-specific results; handles SelectionDAG-specific logic.
- **CN**: 围绕 `getMemoryVT`, `assert`, `getAlign`, `isVolatile`, ... 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果，处理 SelectionDAG 专用逻辑。

### Lines 1678-1717
```cpp
void XCoreTargetLowering::computeKnownBitsForTargetNode(const SDValue Op,
                                                        KnownBits &Known,
                                                        const APInt &DemandedElts,
                                                        const SelectionDAG &DAG,
                                                        unsigned Depth) const {
  Known.resetAll();
  switch (Op.getOpcode()) {
  default: break;
  case XCoreISD::LADD:
  case XCoreISD::LSUB:
    if (Op.getResNo() == 1) {
      // Top bits of carry / borrow are clear.
      Known.Zero = APInt::getHighBitsSet(Known.getBitWidth(),
                                         Known.getBitWidth() - 1);
    }
    break;
  case ISD::INTRINSIC_W_CHAIN:
    {
    unsigned IntNo = Op.getConstantOperandVal(1);
    switch (IntNo) {
    case Intrinsic::xcore_getts:
      // High bits are known to be zero.
      Known.Zero =
          APInt::getHighBitsSet(Known.getBitWidth(), Known.getBitWidth() - 16);
      break;
    case Intrinsic::xcore_int:
    case Intrinsic::xcore_inct:
      // High bits are known to be zero.
      Known.Zero =
          APInt::getHighBitsSet(Known.getBitWidth(), Known.getBitWidth() - 8);
      break;
    case Intrinsic::xcore_testct:
      // Result is either 0 or 1.
      Known.Zero =
          APInt::getHighBitsSet(Known.getBitWidth(), Known.getBitWidth() - 1);
      break;
    case Intrinsic::xcore_testwct:
      // Result is in the range 0 - 4.
      Known.Zero =
          APInt::getHighBitsSet(Known.getBitWidth(), Known.getBitWidth() - 3);
```
- **EN**: Implements logic around `computeKnownBitsForTargetNode`, `resetAll`, `getHighBitsSet`, `getBitWidth`, ...; this block uses `switch`-based dispatch; applies conditional target rules; handles SelectionDAG-specific logic.
- **CN**: 围绕 `computeKnownBitsForTargetNode`, `resetAll`, `getHighBitsSet`, `getBitWidth`, ... 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，处理 SelectionDAG 专用逻辑。

### Lines 1718-1738
```cpp
      break;
    }
    }
    break;
  }
}

//===----------------------------------------------------------------------===//
//  Addressing mode description hooks
//===----------------------------------------------------------------------===//

static inline bool isImmUs(int64_t val)
{
  return (val >= 0 && val <= 11);
}

static inline bool isImmUs2(int64_t val)
{
  return (val%2 == 0 && isImmUs(val/2));
}

```
- **EN**: Implements logic around `isImmUs`, `isImmUs2`; this block returns target-specific results.
- **CN**: 围绕 `isImmUs`, `isImmUs2` 实现具体逻辑；这一段返回目标相关结果。

### Lines 1739-1758
```cpp
static inline bool isImmUs4(int64_t val)
{
  return (val%4 == 0 && isImmUs(val/4));
}

/// isLegalAddressingMode - Return true if the addressing mode represented
/// by AM is legal for this target, for a load/store of the specified type.
bool XCoreTargetLowering::isLegalAddressingMode(const DataLayout &DL,
                                                const AddrMode &AM, Type *Ty,
                                                unsigned AS,
                                                Instruction *I) const {
  if (Ty->getTypeID() == Type::VoidTyID)
    return AM.Scale == 0 && isImmUs(AM.BaseOffs) && isImmUs4(AM.BaseOffs);

  unsigned Size = DL.getTypeAllocSize(Ty);
  if (AM.BaseGV) {
    return Size >= 4 && !AM.HasBaseReg && AM.Scale == 0 &&
                 AM.BaseOffs%4 == 0;
  }

```
- **EN**: Implements logic around `isImmUs4`, `isLegalAddressingMode`, `isImmUs`, `getTypeAllocSize`; this block applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isImmUs4`, `isLegalAddressingMode`, `isImmUs`, `getTypeAllocSize` 实现具体逻辑；这一段应用条件化的目标规则，返回目标相关结果。

### Lines 1759-1784
```cpp
  switch (Size) {
  case 1:
    // reg + imm
    if (AM.Scale == 0) {
      return isImmUs(AM.BaseOffs);
    }
    // reg + reg
    return AM.Scale == 1 && AM.BaseOffs == 0;
  case 2:
  case 3:
    // reg + imm
    if (AM.Scale == 0) {
      return isImmUs2(AM.BaseOffs);
    }
    // reg + reg<<1
    return AM.Scale == 2 && AM.BaseOffs == 0;
  default:
    // reg + imm
    if (AM.Scale == 0) {
      return isImmUs4(AM.BaseOffs);
    }
    // reg + reg<<2
    return AM.Scale == 4 && AM.BaseOffs == 0;
  }
}

```
- **EN**: Implements logic around `isImmUs`, `isImmUs2`, `isImmUs4`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `isImmUs`, `isImmUs2`, `isImmUs4` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

### Lines 1785-1803
```cpp
//===----------------------------------------------------------------------===//
//                           XCore Inline Assembly Support
//===----------------------------------------------------------------------===//

std::pair<unsigned, const TargetRegisterClass *>
XCoreTargetLowering::getRegForInlineAsmConstraint(const TargetRegisterInfo *TRI,
                                                  StringRef Constraint,
                                                  MVT VT) const {
  if (Constraint.size() == 1) {
    switch (Constraint[0]) {
    default : break;
    case 'r':
      return std::make_pair(0U, &XCore::GRRegsRegClass);
    }
  }
  // Use the default implementation in TargetLowering to convert the register
  // constraint into a member of a register class.
  return TargetLowering::getRegForInlineAsmConstraint(TRI, Constraint, VT);
}
```
- **EN**: Implements logic around `getRegForInlineAsmConstraint`, `make_pair`; this block uses `switch`-based dispatch; applies conditional target rules; returns target-specific results.
- **CN**: 围绕 `getRegForInlineAsmConstraint`, `make_pair` 实现具体逻辑；这一段使用 `switch` 分派，应用条件化的目标规则，返回目标相关结果。

## Key Concepts / 关键概念

- **SelectionDAG lowering / SelectionDAG 降低**:
  - **EN**: Custom lowering/legalization rules for target operations
  - **CN**: 为目标操作定制 lowering 与合法化规则
- **DAG legalization / DAG 合法化**:
  - **EN**: Explains how generic IR-style operations are rewritten for this target
  - **CN**: 说明如何把通用操作改写为该目标可接受的形式
- **Machine-level codegen / 机器级代码生成**:
  - **EN**: Operates after instruction selection on machine instructions and blocks
  - **CN**: 在指令选择后处理机器指令与基本块

## Dependencies / 依赖关系

- **Direct includes / 直接包含**: `XCoreISelLowering.h`, `XCore.h`, `XCoreMachineFunctionInfo.h`, `XCoreSubtarget.h`, `XCoreTargetMachine.h`, `XCoreTargetObjectFile.h`, `llvm/CodeGen/CallingConvLower.h`, `llvm/CodeGen/MachineFrameInfo.h`, `llvm/CodeGen/MachineFunction.h`, `llvm/CodeGen/MachineInstrBuilder.h`, `llvm/CodeGen/MachineJumpTableInfo.h`, `llvm/CodeGen/MachineRegisterInfo.h` ... (+14 more)
- **LLVM subsystems / LLVM 子系统**: CodeGen, IR, Support
