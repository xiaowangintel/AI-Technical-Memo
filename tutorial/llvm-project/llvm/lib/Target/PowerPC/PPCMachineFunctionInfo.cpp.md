# PPCMachineFunctionInfo.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCMachineFunctionInfo.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides backend implementation logic for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCMachineFunctionInfo.cpp`，主要负责 PowerPC 后端的后端实现逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCMachineFunctionInfo.cpp - Private data used for PowerPC --------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-13

```cpp
//===----------------------------------------------------------------------===//

#include "PPCMachineFunctionInfo.h"
#include "llvm/ADT/Twine.h"
#include "llvm/BinaryFormat/XCOFF.h"
#include "llvm/IR/DataLayout.h"
#include "llvm/MC/MCContext.h"
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 14-51

```cpp
#include "llvm/Support/CommandLine.h"

using namespace llvm;
static cl::opt<bool> PPCDisableNonVolatileCR(
    "ppc-disable-non-volatile-cr",
    cl::desc("Disable the use of non-volatile CR register fields"),
    cl::init(false), cl::Hidden);

void PPCFunctionInfo::anchor() {}
PPCFunctionInfo::PPCFunctionInfo(const Function &F,
                                 const TargetSubtargetInfo *STI)
    : DisableNonVolatileCR(PPCDisableNonVolatileCR) {}

MachineFunctionInfo *
PPCFunctionInfo::clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
                       const DenseMap<MachineBasicBlock *, MachineBasicBlock *>
                           &Src2DstMBB) const {
  return DestMF.cloneInfo<PPCFunctionInfo>(*this);
}

MCSymbol *PPCFunctionInfo::getPICOffsetSymbol(MachineFunction &MF) const {
  const DataLayout &DL = MF.getDataLayout();
  return MF.getContext().getOrCreateSymbol(Twine(DL.getInternalSymbolPrefix()) +
                                           Twine(MF.getFunctionNumber()) +
                                           "$poff");
}

MCSymbol *PPCFunctionInfo::getGlobalEPSymbol(MachineFunction &MF) const {
  const DataLayout &DL = MF.getDataLayout();
  return MF.getContext().getOrCreateSymbol(Twine(DL.getInternalSymbolPrefix()) +
                                           "func_gep" +
                                           Twine(MF.getFunctionNumber()));
}

MCSymbol *PPCFunctionInfo::getLocalEPSymbol(MachineFunction &MF) const {
  const DataLayout &DL = MF.getDataLayout();
  return MF.getContext().getOrCreateSymbol(Twine(DL.getInternalSymbolPrefix()) +
                                           "func_lep" +
```
- **EN**: Pulls in direct dependencies required by this backend implementation logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该后端实现逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 52-89

```cpp
                                           Twine(MF.getFunctionNumber()));
}

MCSymbol *PPCFunctionInfo::getTOCOffsetSymbol(MachineFunction &MF) const {
  const DataLayout &DL = MF.getDataLayout();
  return MF.getContext().getOrCreateSymbol(Twine(DL.getInternalSymbolPrefix()) +
                                           "func_toc" +
                                           Twine(MF.getFunctionNumber()));
}

bool PPCFunctionInfo::isLiveInSExt(Register VReg) const {
  for (const std::pair<Register, ISD::ArgFlagsTy> &LiveIn : LiveInAttrs)
    if (LiveIn.first == VReg)
      return LiveIn.second.isSExt();
  return false;
}

bool PPCFunctionInfo::isLiveInZExt(Register VReg) const {
  for (const std::pair<Register, ISD::ArgFlagsTy> &LiveIn : LiveInAttrs)
    if (LiveIn.first == VReg)
      return LiveIn.second.isZExt();
  return false;
}

void PPCFunctionInfo::appendParameterType(ParamType Type) {

  ParamtersType.push_back(Type);
  switch (Type) {
  case FixedType:
    ++FixedParmsNum;
    return;
  case ShortFloatingPoint:
  case LongFloatingPoint:
    ++FloatingParmsNum;
    return;
  case VectorChar:
  case VectorShort:
  case VectorInt:
```
- **EN**: Implements helper routine(s) `Twine`, `getFunctionNumber`, `getTOCOffsetSymbol` for this portion of the PowerPC backend backend implementation logic. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `Twine`, `getFunctionNumber`, `getTOCOffsetSymbol`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 90-127

```cpp
  case VectorFloat:
    ++VectorParmsNum;
    return;
  }
  llvm_unreachable("Error ParamType type.");
}

uint32_t PPCFunctionInfo::getVecExtParmsType() const {

  uint32_t VectExtParamInfo = 0;
  unsigned ShiftBits = 32 - XCOFF::TracebackTable::WidthOfParamType;
  int Bits = 0;

  if (!hasVectorParms())
    return 0;

  for (const auto &Elt : ParamtersType) {
    switch (Elt) {
    case VectorChar:
      VectExtParamInfo <<= XCOFF::TracebackTable::WidthOfParamType;
      VectExtParamInfo |=
          XCOFF::TracebackTable::ParmTypeIsVectorCharBit >> ShiftBits;
      Bits += XCOFF::TracebackTable::WidthOfParamType;
      break;
    case VectorShort:
      VectExtParamInfo <<= XCOFF::TracebackTable::WidthOfParamType;
      VectExtParamInfo |=
          XCOFF::TracebackTable::ParmTypeIsVectorShortBit >> ShiftBits;
      Bits += XCOFF::TracebackTable::WidthOfParamType;
      break;
    case VectorInt:
      VectExtParamInfo <<= XCOFF::TracebackTable::WidthOfParamType;
      VectExtParamInfo |=
          XCOFF::TracebackTable::ParmTypeIsVectorIntBit >> ShiftBits;
      Bits += XCOFF::TracebackTable::WidthOfParamType;
      break;
    case VectorFloat:
      VectExtParamInfo <<= XCOFF::TracebackTable::WidthOfParamType;
```
- **EN**: Implements helper routine(s) `llvm_unreachable`, `getVecExtParmsType`, `hasVectorParms` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `llvm_unreachable`, `getVecExtParmsType`, `hasVectorParms`。

### Lines 128-165

```cpp
      VectExtParamInfo |=
          XCOFF::TracebackTable::ParmTypeIsVectorFloatBit >> ShiftBits;
      Bits += XCOFF::TracebackTable::WidthOfParamType;
      break;
    default:
      break;
    }

    // There are only 32bits in the VectExtParamInfo.
    if (Bits >= 32)
      break;
  }
  return Bits < 32 ? VectExtParamInfo << (32 - Bits) : VectExtParamInfo;
}

uint32_t PPCFunctionInfo::getParmsType() const {
  uint32_t ParamsTypeInfo = 0;
  unsigned ShiftBits = 32 - XCOFF::TracebackTable::WidthOfParamType;

  int Bits = 0;
  for (const auto &Elt : ParamtersType) {

    if (Bits > 31 || (Bits > 30 && (Elt != FixedType || hasVectorParms())))
      break;

    switch (Elt) {
    case FixedType:
      if (hasVectorParms()) {
        //'00'  ==> fixed parameter if HasVectorParms is true.
        ParamsTypeInfo <<= XCOFF::TracebackTable::WidthOfParamType;
        ParamsTypeInfo |=
            XCOFF::TracebackTable::ParmTypeIsFixedBits >> ShiftBits;
        Bits += XCOFF::TracebackTable::WidthOfParamType;
      } else {
        //'0'  ==> fixed parameter if HasVectorParms is false.
        ParamsTypeInfo <<= 1;
        ++Bits;
      }
```
- **EN**: Implements helper routine(s) `getParmsType`, `hasVectorParms` for this portion of the PowerPC backend backend implementation logic.
- **CN**: 这里实现了 PowerPC 后端该部分后端实现逻辑所需的辅助例程 `getParmsType`, `hasVectorParms`。

### Lines 166-195

```cpp
      break;
    case ShortFloatingPoint:
      // '10'b => floating point short parameter.
      ParamsTypeInfo <<= XCOFF::TracebackTable::WidthOfParamType;
      ParamsTypeInfo |=
          XCOFF::TracebackTable::ParmTypeIsFloatingBits >> ShiftBits;
      Bits += XCOFF::TracebackTable::WidthOfParamType;
      break;
    case LongFloatingPoint:
      // '11'b => floating point long parameter.
      ParamsTypeInfo <<= XCOFF::TracebackTable::WidthOfParamType;
      ParamsTypeInfo |=
          XCOFF::TracebackTable::ParmTypeIsDoubleBits >> ShiftBits;
      Bits += XCOFF::TracebackTable::WidthOfParamType;
      break;
    case VectorChar:
    case VectorShort:
    case VectorInt:
    case VectorFloat:
      //	'01' ==> vector parameter
      ParamsTypeInfo <<= XCOFF::TracebackTable::WidthOfParamType;
      ParamsTypeInfo |=
          XCOFF::TracebackTable::ParmTypeIsVectorBits >> ShiftBits;
      Bits += XCOFF::TracebackTable::WidthOfParamType;
      break;
    }
  }

  return Bits < 32 ? ParamsTypeInfo << (32 - Bits) : ParamsTypeInfo;
}
```
- **EN**: Implements dispatch logic that chooses specialized target behavior for different opcodes, modes, or ABI cases.
- **CN**: 这一段实现分派逻辑，为不同操作码、模式或 ABI 情况选择特定的目标行为。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Backend implementation logic / 后端实现逻辑
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Vector or SIMD behavior / 向量或 SIMD 行为
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCMachineFunctionInfo.h`
- `llvm/ADT/Twine.h`
- `llvm/BinaryFormat/XCOFF.h`
- `llvm/IR/DataLayout.h`
- `llvm/MC/MCContext.h`
- `llvm/Support/CommandLine.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
