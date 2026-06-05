# ARMSelectionDAGInfo.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/ARM/ARMSelectionDAGInfo.cpp`
- Repository: `llvm-project`
- Purpose (EN): This file implements the ARMSelectionDAGInfo class.
- 用途 (CN): 实现 ARM 后端中的 `ARMSelectionDAGInfo`，重点处理SelectionDAG 目标钩子。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
//===-- ARMSelectionDAGInfo.cpp - ARM SelectionDAG Info -------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the ARMSelectionDAGInfo class.
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 13-16
```cpp
#include "ARMSelectionDAGInfo.h"
#include "ARMTargetTransformInfo.h"
#include "llvm/CodeGen/SelectionDAG.h"
#include "llvm/Support/CommandLine.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 18-19
```cpp
#define GET_SDNODE_DESC
#include "ARMGenSDNodeInfo.inc"
```
- EN: Connects this file to TableGen-generated declarations so target-specific enums and helper tables become available to C++ code.
- CN: 这里把文件连接到 TableGen 生成的声明，使目标相关枚举和辅助表能够在 C++ 代码中使用。

### Lines 21-21
```cpp
using namespace llvm;
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 23-23
```cpp
#define DEBUG_TYPE "arm-selectiondag-info"
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 25-36
```cpp
static cl::opt<TPLoop::MemTransfer> EnableMemtransferTPLoop(
    "arm-memtransfer-tploop", cl::Hidden,
    cl::desc("Control conversion of memcpy to "
             "Tail predicated loops (WLSTP)"),
    cl::init(TPLoop::ForceDisabled),
    cl::values(clEnumValN(TPLoop::ForceDisabled, "force-disabled",
                          "Don't convert memcpy to TP loop."),
               clEnumValN(TPLoop::ForceEnabled, "force-enabled",
                          "Always convert memcpy to TP loop."),
               clEnumValN(TPLoop::Allow, "allow",
                          "Allow (may be subject to certain conditions) "
                          "conversion of memcpy to TP loop.")));
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 38-39
```cpp
ARMSelectionDAGInfo::ARMSelectionDAGInfo()
    : SelectionDAGGenTargetInfo(ARMGenSDNodeInfo) {}
```
- EN: Implements `ARMSelectionDAGInfo::ARMSelectionDAGInfo`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSelectionDAGInfo::ARMSelectionDAGInfo`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 41-44
```cpp
const char *ARMSelectionDAGInfo::getTargetNodeName(unsigned Opcode) const {
#define MAKE_CASE(V)                                                           \
  case V:                                                                      \
    return #V;
```
- EN: Implements `ARMSelectionDAGInfo::getTargetNodeName`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSelectionDAGInfo::getTargetNodeName`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 46-60
```cpp
  // These nodes don't have corresponding entries in *.td files yet.
  switch (static_cast<ARMISD::NodeType>(Opcode)) {
    MAKE_CASE(ARMISD::DYN_ALLOC)
    MAKE_CASE(ARMISD::MVESEXT)
    MAKE_CASE(ARMISD::MVEZEXT)
    MAKE_CASE(ARMISD::MVETRUNC)
    MAKE_CASE(ARMISD::BUILD_VECTOR)
    MAKE_CASE(ARMISD::VLD1DUP)
    MAKE_CASE(ARMISD::VLD2DUP)
    MAKE_CASE(ARMISD::VLD3DUP)
    MAKE_CASE(ARMISD::VLD4DUP)
    MAKE_CASE(ARMISD::VLD1_UPD)
    MAKE_CASE(ARMISD::VLD2_UPD)
    MAKE_CASE(ARMISD::VLD3_UPD)
    MAKE_CASE(ARMISD::VLD4_UPD)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 61-75
```cpp
    MAKE_CASE(ARMISD::VLD1x2_UPD)
    MAKE_CASE(ARMISD::VLD1x3_UPD)
    MAKE_CASE(ARMISD::VLD1x4_UPD)
    MAKE_CASE(ARMISD::VLD2LN_UPD)
    MAKE_CASE(ARMISD::VLD3LN_UPD)
    MAKE_CASE(ARMISD::VLD4LN_UPD)
    MAKE_CASE(ARMISD::VLD1DUP_UPD)
    MAKE_CASE(ARMISD::VLD2DUP_UPD)
    MAKE_CASE(ARMISD::VLD3DUP_UPD)
    MAKE_CASE(ARMISD::VLD4DUP_UPD)
    MAKE_CASE(ARMISD::VST1_UPD)
    MAKE_CASE(ARMISD::VST3_UPD)
    MAKE_CASE(ARMISD::VST1x2_UPD)
    MAKE_CASE(ARMISD::VST1x3_UPD)
    MAKE_CASE(ARMISD::VST1x4_UPD)
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 76-80
```cpp
    MAKE_CASE(ARMISD::VST2LN_UPD)
    MAKE_CASE(ARMISD::VST3LN_UPD)
    MAKE_CASE(ARMISD::VST4LN_UPD)
  }
#undef MAKE_CASE
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 82-83
```cpp
  return SelectionDAGGenTargetInfo::getTargetNodeName(Opcode);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 85-89
```cpp
bool ARMSelectionDAGInfo::isTargetMemoryOpcode(unsigned Opcode) const {
  // These nodes don't have corresponding entries in *.td files yet.
  if (Opcode >= ARMISD::FIRST_MEMORY_OPCODE &&
      Opcode <= ARMISD::LAST_MEMORY_OPCODE)
    return true;
```
- EN: Implements `ARMSelectionDAGInfo::isTargetMemoryOpcode`, a query/helper routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSelectionDAGInfo::isTargetMemoryOpcode`，它是一个围绕SelectionDAG 降级展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 91-92
```cpp
  return SelectionDAGGenTargetInfo::isTargetMemoryOpcode(Opcode);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 94-108
```cpp
void ARMSelectionDAGInfo::verifyTargetNode(const SelectionDAG &DAG,
                                           const SDNode *N) const {
  switch (N->getOpcode()) {
  default:
    break;
  case ARMISD::WIN__DBZCHK:
    // invalid number of results; expected 2, got 1
  case ARMISD::WIN__CHKSTK:
    // invalid number of results; expected 1, got 2
  case ARMISD::COPY_STRUCT_BYVAL:
    // invalid number of operands; expected 6, got 5
  case ARMISD::MEMCPY:
    // invalid number of operands; expected 5, got 4
  case ARMISD::VMOVRRD:
    // operand #0 must have type f64, but has type v1i64/v4f16/v8i8
```
- EN: Implements `ARMSelectionDAGInfo::verifyTargetNode`, a target-specific routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSelectionDAGInfo::verifyTargetNode`，它是一个围绕SelectionDAG 降级展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 109-112
```cpp
  case ARMISD::VMOVIMM:
    // operand #0 must have type i32, but has type i16
    return;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 114-115
```cpp
  SelectionDAGGenTargetInfo::verifyTargetNode(DAG, N);
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 117-125
```cpp
// Emit, if possible, a specialized version of the given Libcall. Typically this
// means selecting the appropriately aligned version, but we also convert memset
// of 0 into memclr.
SDValue ARMSelectionDAGInfo::EmitSpecializedLibcall(
    SelectionDAG &DAG, const SDLoc &dl, SDValue Chain, SDValue Dst, SDValue Src,
    SDValue Size, unsigned Align, RTLIB::Libcall LC) const {
  const ARMSubtarget &Subtarget =
      DAG.getMachineFunction().getSubtarget<ARMSubtarget>();
  const ARMTargetLowering *TLI = Subtarget.getTargetLowering();
```
- EN: Implements `ARMSelectionDAGInfo::EmitSpecializedLibcall`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSelectionDAGInfo::EmitSpecializedLibcall`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 127-141
```cpp
  // Only use a specialized AEABI function if the default version of this
  // Libcall is an AEABI function.
  //
  // Translate RTLIB::Libcall to AEABILibcall. We only do this in order to be
  // able to translate memset to memclr and use the value to index the function
  // name array.
  enum {
    AEABI_MEMCPY = 0,
    AEABI_MEMMOVE,
    AEABI_MEMSET,
    AEABI_MEMCLR
  } AEABILibcall;
  switch (LC) {
  case RTLIB::MEMCPY:
    if (DAG.getLibcalls().getLibcallImpl(LC) != RTLIB::impl___aeabi_memcpy)
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 142-142
```cpp
      return SDValue();
```
- EN: Declares `SDValue`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `SDValue`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 144-148
```cpp
    AEABILibcall = AEABI_MEMCPY;
    break;
  case RTLIB::MEMMOVE:
    if (DAG.getLibcalls().getLibcallImpl(LC) != RTLIB::impl___aeabi_memmove)
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 150-154
```cpp
    AEABILibcall = AEABI_MEMMOVE;
    break;
  case RTLIB::MEMSET:
    if (DAG.getLibcalls().getLibcallImpl(LC) != RTLIB::impl___aeabi_memset)
      return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 156-162
```cpp
    AEABILibcall = AEABI_MEMSET;
    if (isNullConstant(Src))
      AEABILibcall = AEABI_MEMCLR;
    break;
  default:
    return SDValue();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 164-175
```cpp
  // Choose the most-aligned libcall variant that we can
  enum {
    ALIGN1 = 0,
    ALIGN4,
    ALIGN8
  } AlignVariant;
  if ((Align & 7) == 0)
    AlignVariant = ALIGN8;
  else if ((Align & 3) == 0)
    AlignVariant = ALIGN4;
  else
    AlignVariant = ALIGN1;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 177-186
```cpp
  TargetLowering::ArgListTy Args;
  Type *IntPtrTy = DAG.getDataLayout().getIntPtrType(*DAG.getContext());
  Args.emplace_back(Dst, IntPtrTy);
  if (AEABILibcall == AEABI_MEMCLR) {
    Args.emplace_back(Size, IntPtrTy);
  } else if (AEABILibcall == AEABI_MEMSET) {
    // Adjust parameters for memset, EABI uses format (ptr, size, value),
    // GNU library uses (ptr, value, size)
    // See RTABI section 4.3.4
    Args.emplace_back(Size, IntPtrTy);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 188-192
```cpp
    // Extend or truncate the argument to be an i32 value for the call.
    if (Src.getValueType().bitsGT(MVT::i32))
      Src = DAG.getNode(ISD::TRUNCATE, dl, MVT::i32, Src);
    else if (Src.getValueType().bitsLT(MVT::i32))
      Src = DAG.getNode(ISD::ZERO_EXTEND, dl, MVT::i32, Src);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 194-201
```cpp
    TargetLowering::ArgListEntry Entry(Src,
                                       Type::getInt32Ty(*DAG.getContext()));
    Entry.IsSExt = false;
    Args.push_back(Entry);
  } else {
    Args.emplace_back(Src, IntPtrTy);
    Args.emplace_back(Size, IntPtrTy);
  }
```
- EN: Declares `Type::getInt32Ty`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `Type::getInt32Ty`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 203-207
```cpp
  static const RTLIB::Libcall FunctionImpls[4][3] = {
      {RTLIB::MEMCPY, RTLIB::AEABI_MEMCPY4, RTLIB::AEABI_MEMCPY8},
      {RTLIB::MEMMOVE, RTLIB::AEABI_MEMMOVE4, RTLIB::AEABI_MEMMOVE8},
      {RTLIB::MEMSET, RTLIB::AEABI_MEMSET4, RTLIB::AEABI_MEMSET8},
      {RTLIB::AEABI_MEMCLR, RTLIB::AEABI_MEMCLR4, RTLIB::AEABI_MEMCLR8}};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 209-212
```cpp
  RTLIB::Libcall NewLC = FunctionImpls[AEABILibcall][AlignVariant];
  RTLIB::LibcallImpl LCImpl = DAG.getLibcalls().getLibcallImpl(NewLC);
  if (LCImpl == RTLIB::Unsupported)
    return SDValue();
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 214-223
```cpp
  TargetLowering::CallLoweringInfo CLI(DAG);
  CLI.setDebugLoc(dl)
      .setChain(Chain)
      .setLibCallee(
          DAG.getLibcalls().getLibcallImplCallingConv(LCImpl),
          Type::getVoidTy(*DAG.getContext()),
          DAG.getExternalSymbol(LCImpl, TLI->getPointerTy(DAG.getDataLayout())),
          std::move(Args))
      .setDiscardResult();
  std::pair<SDValue,SDValue> CallResult = TLI->LowerCallTo(CLI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 225-226
```cpp
  return CallResult.second;
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 228-242
```cpp
static bool shouldGenerateInlineTPLoop(const ARMSubtarget &Subtarget,
                                       const SelectionDAG &DAG,
                                       ConstantSDNode *ConstantSize,
                                       Align Alignment, bool IsMemcpy) {
  auto &F = DAG.getMachineFunction().getFunction();
  if (!EnableMemtransferTPLoop)
    return false;
  if (EnableMemtransferTPLoop == TPLoop::ForceEnabled)
    return true;
  // Do not generate inline TP loop if optimizations is disabled,
  // or if optimization for size (-Os or -Oz) is on.
  if (F.hasOptNone() || F.hasOptSize())
    return false;
  // If cli option is unset, for memset always generate inline TP.
  // For memcpy, check some conditions
```
- EN: Implements `shouldGenerateInlineTPLoop`, a target-specific routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `shouldGenerateInlineTPLoop`，它是一个围绕机器函数状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 243-253
```cpp
  if (!IsMemcpy)
    return true;
  if (!ConstantSize && Alignment >= Align(4))
    return true;
  if (ConstantSize &&
      ConstantSize->getZExtValue() > Subtarget.getMaxInlineSizeThreshold() &&
      ConstantSize->getZExtValue() <
          Subtarget.getMaxMemcpyTPInlineSizeThreshold())
    return true;
  return false;
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 255-261
```cpp
SDValue ARMSelectionDAGInfo::EmitTargetCodeForMemcpy(
    SelectionDAG &DAG, const SDLoc &dl, SDValue Chain, SDValue Dst, SDValue Src,
    SDValue Size, Align Alignment, bool isVolatile, bool AlwaysInline,
    MachinePointerInfo DstPtrInfo, MachinePointerInfo SrcPtrInfo) const {
  const ARMSubtarget &Subtarget =
      DAG.getMachineFunction().getSubtarget<ARMSubtarget>();
  ConstantSDNode *ConstantSize = dyn_cast<ConstantSDNode>(Size);
```
- EN: Implements `ARMSelectionDAGInfo::EmitTargetCodeForMemcpy`, a emission/printing routine centered on machine-function state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSelectionDAGInfo::EmitTargetCodeForMemcpy`，它是一个围绕机器函数状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 263-266
```cpp
  if (Subtarget.hasMVEIntegerOps() &&
      shouldGenerateInlineTPLoop(Subtarget, DAG, ConstantSize, Alignment, true))
    return DAG.getNode(ARMISD::MEMCPYLOOP, dl, MVT::Other, Chain, Dst, Src,
                       DAG.getZExtOrTrunc(Size, dl, MVT::i32));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 268-280
```cpp
  // Do repeated 4-byte loads and stores. To be improved.
  // This requires 4-byte alignment.
  if (Alignment < Align(4))
    return SDValue();
  // This requires the copy size to be a constant, preferably
  // within a subtarget-specific limit.
  if (!ConstantSize)
    return EmitSpecializedLibcall(DAG, dl, Chain, Dst, Src, Size,
                                  Alignment.value(), RTLIB::MEMCPY);
  uint64_t SizeVal = ConstantSize->getZExtValue();
  if (!AlwaysInline && SizeVal > Subtarget.getMaxInlineSizeThreshold())
    return EmitSpecializedLibcall(DAG, dl, Chain, Dst, Src, Size,
                                  Alignment.value(), RTLIB::MEMCPY);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 282-292
```cpp
  unsigned BytesLeft = SizeVal & 3;
  unsigned NumMemOps = SizeVal >> 2;
  unsigned EmittedNumMemOps = 0;
  EVT VT = MVT::i32;
  unsigned VTSize = 4;
  unsigned i = 0;
  // Emit a maximum of 4 loads in Thumb1 since we have fewer registers
  const unsigned MaxLoadsInLDM = Subtarget.isThumb1Only() ? 4 : 6;
  SDValue TFOps[6];
  SDValue Loads[6];
  uint64_t SrcOff = 0, DstOff = 0;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 294-297
```cpp
  // FIXME: We should invent a VMEMCPY pseudo-instruction that lowers to
  // VLDM/VSTM and make this code emit it when appropriate. This would reduce
  // pressure on the general purpose registers. However this seems harder to map
  // onto the register allocator's view of the world.
```
- EN: Documents the surrounding logic with a local comment block, clarifying backend assumptions or upcoming control flow.
- CN: 这里通过局部注释块解释周围逻辑，说明后端假设或即将出现的控制流程。

### Lines 299-303
```cpp
  // The number of MEMCPY pseudo-instructions to emit. We use up to
  // MaxLoadsInLDM registers per mcopy, which will get lowered into ldm/stm
  // later on. This is a lower bound on the number of MEMCPY operations we must
  // emit.
  unsigned NumMEMCPYs = (NumMemOps + MaxLoadsInLDM - 1) / MaxLoadsInLDM;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 305-309
```cpp
  // Code size optimisation: do not inline memcpy if expansion results in
  // more instructions than the library call.
  if (NumMEMCPYs > 1 && Subtarget.hasMinSize()) {
    return SDValue();
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 311-311
```cpp
  SDVTList VTs = DAG.getVTList(MVT::i32, MVT::i32, MVT::Other, MVT::Glue);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 313-317
```cpp
  for (unsigned I = 0; I != NumMEMCPYs; ++I) {
    // Evenly distribute registers among MEMCPY operations to reduce register
    // pressure.
    unsigned NextEmittedNumMemOps = NumMemOps * (I + 1) / NumMEMCPYs;
    unsigned NumRegs = NextEmittedNumMemOps - EmittedNumMemOps;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 319-322
```cpp
    Dst = DAG.getNode(ARMISD::MEMCPY, dl, VTs, Chain, Dst, Src,
                      DAG.getConstant(NumRegs, dl, MVT::i32));
    Src = Dst.getValue(1);
    Chain = Dst.getValue(2);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 324-325
```cpp
    DstPtrInfo = DstPtrInfo.getWithOffset(NumRegs * VTSize);
    SrcPtrInfo = SrcPtrInfo.getWithOffset(NumRegs * VTSize);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 327-328
```cpp
    EmittedNumMemOps = NextEmittedNumMemOps;
  }
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 330-331
```cpp
  if (BytesLeft == 0)
    return Chain;
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 333-339
```cpp
  // Issue loads / stores for the trailing (1 - 3) bytes.
  auto getRemainingValueType = [](unsigned BytesLeft) {
    return (BytesLeft >= 2) ? MVT::i16 : MVT::i8;
  };
  auto getRemainingSize = [](unsigned BytesLeft) {
    return (BytesLeft >= 2) ? 2 : 1;
  };
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 341-355
```cpp
  unsigned BytesLeftSave = BytesLeft;
  i = 0;
  while (BytesLeft) {
    VT = getRemainingValueType(BytesLeft);
    VTSize = getRemainingSize(BytesLeft);
    Loads[i] = DAG.getLoad(VT, dl, Chain,
                           DAG.getNode(ISD::ADD, dl, MVT::i32, Src,
                                       DAG.getConstant(SrcOff, dl, MVT::i32)),
                           SrcPtrInfo.getWithOffset(SrcOff));
    TFOps[i] = Loads[i].getValue(1);
    ++i;
    SrcOff += VTSize;
    BytesLeft -= VTSize;
  }
  Chain = DAG.getNode(ISD::TokenFactor, dl, MVT::Other, ArrayRef(TFOps, i));
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 357-371
```cpp
  i = 0;
  BytesLeft = BytesLeftSave;
  while (BytesLeft) {
    VT = getRemainingValueType(BytesLeft);
    VTSize = getRemainingSize(BytesLeft);
    TFOps[i] = DAG.getStore(Chain, dl, Loads[i],
                            DAG.getNode(ISD::ADD, dl, MVT::i32, Dst,
                                        DAG.getConstant(DstOff, dl, MVT::i32)),
                            DstPtrInfo.getWithOffset(DstOff));
    ++i;
    DstOff += VTSize;
    BytesLeft -= VTSize;
  }
  return DAG.getNode(ISD::TokenFactor, dl, MVT::Other, ArrayRef(TFOps, i));
}
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 373-379
```cpp
SDValue ARMSelectionDAGInfo::EmitTargetCodeForMemmove(
    SelectionDAG &DAG, const SDLoc &dl, SDValue Chain, SDValue Dst, SDValue Src,
    SDValue Size, Align Alignment, bool isVolatile,
    MachinePointerInfo DstPtrInfo, MachinePointerInfo SrcPtrInfo) const {
  return EmitSpecializedLibcall(DAG, dl, Chain, Dst, Src, Size,
                                Alignment.value(), RTLIB::MEMMOVE);
}
```
- EN: Implements `ARMSelectionDAGInfo::EmitTargetCodeForMemmove`, a emission/printing routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSelectionDAGInfo::EmitTargetCodeForMemmove`，它是一个围绕SelectionDAG 降级展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 381-384
```cpp
SDValue ARMSelectionDAGInfo::EmitTargetCodeForMemset(
    SelectionDAG &DAG, const SDLoc &dl, SDValue Chain, SDValue Dst, SDValue Src,
    SDValue Size, Align Alignment, bool isVolatile, bool AlwaysInline,
    MachinePointerInfo DstPtrInfo) const {
```
- EN: Implements `ARMSelectionDAGInfo::EmitTargetCodeForMemset`, a emission/printing routine centered on SelectionDAG lowering. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `ARMSelectionDAGInfo::EmitTargetCodeForMemset`，它是一个围绕SelectionDAG 降级展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 386-387
```cpp
  const ARMSubtarget &Subtarget =
      DAG.getMachineFunction().getSubtarget<ARMSubtarget>();
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 389-389
```cpp
  ConstantSDNode *ConstantSize = dyn_cast<ConstantSDNode>(Size);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 391-399
```cpp
  // Generate TP loop for llvm.memset
  if (Subtarget.hasMVEIntegerOps() &&
      shouldGenerateInlineTPLoop(Subtarget, DAG, ConstantSize, Alignment,
                                 false)) {
    Src = DAG.getSplatBuildVector(MVT::v16i8, dl,
                                  DAG.getNode(ISD::TRUNCATE, dl, MVT::i8, Src));
    return DAG.getNode(ARMISD::MEMSETLOOP, dl, MVT::Other, Chain, Dst, Src,
                       DAG.getZExtOrTrunc(Size, dl, MVT::i32));
  }
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 401-403
```cpp
  if (!AlwaysInline)
    return EmitSpecializedLibcall(DAG, dl, Chain, Dst, Src, Size,
                                  Alignment.value(), RTLIB::MEMSET);
```
- EN: Carries the main control flow for this portion of the component, selecting target-specific cases and maintaining internal state as conditions change.
- CN: 这里承载该组件这一部分的主要控制流：根据条件选择不同的目标特定情况，并维护内部状态。

### Lines 405-406
```cpp
  return SDValue();
}
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: SelectionDAG target hooks.
  - CN: 核心职责：SelectionDAG 目标钩子。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `ARMSelectionDAGInfo.h`, `ARMTargetTransformInfo.h`, `ARMGenSDNodeInfo.inc`.
  - CN: 后端本地头文件：`ARMSelectionDAGInfo.h`, `ARMTargetTransformInfo.h`, `ARMGenSDNodeInfo.inc`。
- EN: LLVM infrastructure headers: `llvm/CodeGen/SelectionDAG.h`, `llvm/Support/CommandLine.h`.
  - CN: LLVM 基础设施头文件：`llvm/CodeGen/SelectionDAG.h`, `llvm/Support/CommandLine.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
