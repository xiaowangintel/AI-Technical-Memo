# PPCMachineFunctionInfo.h — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/PPCMachineFunctionInfo.h`
- **Repository**: `llvm-project`
- **Purpose (EN)**: PPCMachineFunctionInfo.h - Private data used for PowerPC.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/PPCMachineFunctionInfo.h`，主要负责 PowerPC 后端的后端的公共或内部声明。 文件内容以接口、类型声明和协作关系说明为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- PPCMachineFunctionInfo.h - Private data used for PowerPC --*- C++ -*-=//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-10

```cpp
//===----------------------------------------------------------------------===//
//
// This file declares the PowerPC specific subclass of MachineFunctionInfo.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This file declares the PowerPC specific subclass of MachineFunctionInfo.".
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This file declares the PowerPC specific subclass of MachineFunctionInfo.”。

### Lines 11-17

```cpp
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_POWERPC_PPCMACHINEFUNCTIONINFO_H
#define LLVM_LIB_TARGET_POWERPC_PPCMACHINEFUNCTIONINFO_H

#include "llvm/ADT/SmallVector.h"
#include "llvm/CodeGen/MachineFunction.h"
```
- **EN**: Pulls in direct dependencies required by this public or internal declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers.
- **CN**: 这一段引入该后端的公共或内部声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。

### Lines 18-25

```cpp
#include "llvm/CodeGen/TargetCallingConv.h"

namespace llvm {

/// PPCFunctionInfo - This class is derived from MachineFunction private
/// PowerPC target-specific information for each MachineFunction.
class PPCFunctionInfo : public MachineFunctionInfo {
public:
```
- **EN**: Pulls in direct dependencies required by this public or internal declarations for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. ABI and calling-convention details are important in this part of the code.
- **CN**: 这一段引入该后端的公共或内部声明所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 ABI 与调用约定细节是这一部分代码的重要约束。

### Lines 26-51

```cpp
  enum ParamType {
    FixedType,
    ShortFloatingPoint,
    LongFloatingPoint,
    VectorChar,
    VectorShort,
    VectorInt,
    VectorFloat
  };

private:
  virtual void anchor();

  /// FramePointerSaveIndex - Frame index of where the old frame pointer is
  /// stored.  Also used as an anchor for instructions that need to be altered
  /// when using frame pointers (dyna_add, dyna_sub.)
  int FramePointerSaveIndex = 0;

  /// ReturnAddrSaveIndex - Frame index of where the return address is stored.
  ///
  int ReturnAddrSaveIndex = 0;

  /// Frame index where the old base pointer is stored.
  int BasePointerSaveIndex = 0;

  /// Frame index where the old PIC base pointer is stored.
```
- **EN**: Implements helper routine(s) `anchor`, `pointers` for this portion of the PowerPC backend public or internal declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分后端的公共或内部声明所需的辅助例程 `anchor`, `pointers`。

### Lines 52-77

```cpp
  int PICBasePointerSaveIndex = 0;

  /// Frame index where the ROP Protection Hash is stored.
  int ROPProtectionHashSaveIndex = 0;

  /// MustSaveLR - Indicates whether LR is defined (or clobbered) in the current
  /// function.  This is only valid after the initial scan of the function by
  /// PEI.
  bool MustSaveLR = false;

  /// MustSaveTOC - Indicates that the TOC save needs to be performed in the
  /// prologue of the function. This is typically the case when there are
  /// indirect calls in the function and it is more profitable to save the
  /// TOC pointer in the prologue than in the block(s) containing the call(s).
  bool MustSaveTOC = false;

  /// Do we have to disable shrink-wrapping? This has to be set if we emit any
  /// instructions that clobber LR in the entry block because discovering this
  /// in PEI is too late (happens after shrink-wrapping);
  bool ShrinkWrapDisabled = false;

  /// Does this function have any stack spills.
  bool HasSpills = false;

  /// Does this function spill using instructions with only r+r (not r+i)
  /// forms.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Frame index where the ROP Protection Hash is stored.". Notable symbols in this range include `defined`, `block`, `call`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Frame index where the ROP Protection Hash is stored.”。 该区间中较显眼的符号包括 `defined`, `block`, `call`。

### Lines 78-103

```cpp
  bool HasNonRISpills = false;

  /// SpillsCR - Indicates whether CR is spilled in the current function.
  bool SpillsCR = false;

  /// DisableNonVolatileCR - Indicates whether non-volatile CR fields would be
  /// disabled.
  bool DisableNonVolatileCR = false;

  /// LRStoreRequired - The bool indicates whether there is some explicit use of
  /// the LR/LR8 stack slot that is not obvious from scanning the code.  This
  /// requires that the code generator produce a store of LR to the stack on
  /// entry, even though LR may otherwise apparently not be used.
  bool LRStoreRequired = false;

  /// This function makes use of the PPC64 ELF TOC base pointer (register r2).
  bool UsesTOCBasePtr = false;

  /// MinReservedArea - This is the frame size that is at least reserved in a
  /// potential caller (parameter+linkage area).
  unsigned MinReservedArea = 0;

  /// TailCallSPDelta - Stack pointer delta used when tail calling. Maximum
  /// amount the stack pointer is adjusted to make the frame bigger for tail
  /// calls. Used for creating an area before the register spill area.
  int TailCallSPDelta = 0;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "SpillsCR - Indicates whether CR is spilled in the current function.". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“SpillsCR - Indicates whether CR is spilled in the current function.”。 该区间与栈帧布局或栈访问相关。

### Lines 104-129

```cpp

  /// HasFastCall - Does this function contain a fast call. Used to determine
  /// how the caller's stack pointer should be calculated (epilog/dynamicalloc).
  bool HasFastCall = false;

  /// VarArgsFrameIndex - FrameIndex for start of varargs area.
  int VarArgsFrameIndex = 0;

  /// VarArgsStackOffset - StackOffset for start of stack
  /// arguments.

  int VarArgsStackOffset = 0;

  /// VarArgsNumGPR - Index of the first unused integer
  /// register for parameter passing.
  unsigned VarArgsNumGPR = 0;

  /// VarArgsNumFPR - Index of the first unused double
  /// register for parameter passing.
  unsigned VarArgsNumFPR = 0;

  /// FixedParmsNum - The number of fixed parameters.
  unsigned FixedParmsNum = 0;

  /// FloatingParmsNum - The number of floating parameters.
  unsigned FloatingParmsNum = 0;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "HasFastCall - Does this function contain a fast call. Used to determine". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“HasFastCall - Does this function contain a fast call. Used to determine”。 该区间与栈帧布局或栈访问相关。

### Lines 130-155

```cpp

  /// VectorParmsNum - The number of vector parameters.
  unsigned VectorParmsNum = 0;

  /// ParamtersType - Store all the parameter's type that are saved on
  /// registers.
  SmallVector<ParamType, 32> ParamtersType;

  /// CRSpillFrameIndex - FrameIndex for CR spill slot for 32-bit SVR4.
  int CRSpillFrameIndex = 0;

  /// If any of CR[2-4] need to be saved in the prologue and restored in the
  /// epilogue then they are added to this array. This is used for the
  /// 64-bit SVR4 ABI.
  SmallVector<Register, 3> MustSaveCRs;

  /// Whether this uses the PIC Base register or not.
  bool UsesPICBase = false;

  /// We keep track attributes for each live-in virtual registers
  /// to use SExt/ZExt flags in later optimization.
  std::vector<std::pair<Register, ISD::ArgFlagsTy>> LiveInAttrs;

  /// Flags for aix-shared-lib-tls-model-opt, will be lazily initialized for
  /// each function.
  bool AIXFuncUseTLSIEForLD = false;
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "VectorParmsNum - The number of vector parameters.". This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“VectorParmsNum - The number of vector parameters.”。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 156-181

```cpp
  bool AIXFuncTLSModelOptInitDone = false;

public:
  explicit PPCFunctionInfo(const Function &F, const TargetSubtargetInfo *STI);

  MachineFunctionInfo *
  clone(BumpPtrAllocator &Allocator, MachineFunction &DestMF,
        const DenseMap<MachineBasicBlock *, MachineBasicBlock *> &Src2DstMBB)
      const override;

  int getFramePointerSaveIndex() const { return FramePointerSaveIndex; }
  void setFramePointerSaveIndex(int Idx) { FramePointerSaveIndex = Idx; }

  int getReturnAddrSaveIndex() const { return ReturnAddrSaveIndex; }
  void setReturnAddrSaveIndex(int idx) { ReturnAddrSaveIndex = idx; }

  int getBasePointerSaveIndex() const { return BasePointerSaveIndex; }
  void setBasePointerSaveIndex(int Idx) { BasePointerSaveIndex = Idx; }

  int getPICBasePointerSaveIndex() const { return PICBasePointerSaveIndex; }
  void setPICBasePointerSaveIndex(int Idx) { PICBasePointerSaveIndex = Idx; }

  int getROPProtectionHashSaveIndex() const {
    return ROPProtectionHashSaveIndex;
  }
  void setROPProtectionHashSaveIndex(int Idx) {
```
- **EN**: Implements helper routine(s) `PPCFunctionInfo`, `clone`, `getFramePointerSaveIndex` for this portion of the PowerPC backend public or internal declarations for the backend. Subtarget feature gating influences the behavior here.
- **CN**: 这里实现了 PowerPC 后端该部分后端的公共或内部声明所需的辅助例程 `PPCFunctionInfo`, `clone`, `getFramePointerSaveIndex`。 子目标特性裁剪会影响这里的行为。

### Lines 182-207

```cpp
    ROPProtectionHashSaveIndex = Idx;
  }

  unsigned getMinReservedArea() const { return MinReservedArea; }
  void setMinReservedArea(unsigned size) { MinReservedArea = size; }

  int getTailCallSPDelta() const { return TailCallSPDelta; }
  void setTailCallSPDelta(int size) { TailCallSPDelta = size; }

  /// MustSaveLR - This is set when the prolog/epilog inserter does its initial
  /// scan of the function. It is true if the LR/LR8 register is ever explicitly
  /// defined/clobbered in the machine function (e.g. by calls and movpctolr,
  /// which is used in PIC generation), or if the LR stack slot is explicitly
  /// referenced by builtin_return_address.
  void setMustSaveLR(bool U) { MustSaveLR = U; }
  bool mustSaveLR() const    { return MustSaveLR; }

  void setMustSaveTOC(bool U) { MustSaveTOC = U; }
  bool mustSaveTOC() const    { return MustSaveTOC; }

  /// We certainly don't want to shrink wrap functions if we've emitted a
  /// MovePCtoLR8 as that has to go into the entry, so the prologue definitely
  /// has to go into the entry block.
  void setShrinkWrapDisabled(bool U) { ShrinkWrapDisabled = U; }
  bool shrinkWrapDisabled() const { return ShrinkWrapDisabled; }
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "MustSaveLR - This is set when the prolog/epilog inserter does its initial". Notable symbols in this range include `getMinReservedArea`, `setMinReservedArea`, `getTailCallSPDelta`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“MustSaveLR - This is set when the prolog/epilog inserter does its initial”。 该区间中较显眼的符号包括 `getMinReservedArea`, `setMinReservedArea`, `getTailCallSPDelta`。

### Lines 208-233

```cpp
  void setHasSpills()      { HasSpills = true; }
  bool hasSpills() const   { return HasSpills; }

  void setHasNonRISpills()    { HasNonRISpills = true; }
  bool hasNonRISpills() const { return HasNonRISpills; }

  void setSpillsCR()       { SpillsCR = true; }
  bool isCRSpilled() const { return SpillsCR; }

  void setDisableNonVolatileCR() { DisableNonVolatileCR = true; }
  bool isNonVolatileCRDisabled() const { return DisableNonVolatileCR; }

  void setLRStoreRequired() { LRStoreRequired = true; }
  bool isLRStoreRequired() const { return LRStoreRequired; }

  void setUsesTOCBasePtr()    { UsesTOCBasePtr = true; }
  bool usesTOCBasePtr() const { return UsesTOCBasePtr; }

  void setHasFastCall() { HasFastCall = true; }
  bool hasFastCall() const { return HasFastCall;}

  void setAIXFuncTLSModelOptInitDone() { AIXFuncTLSModelOptInitDone = true; }
  bool isAIXFuncTLSModelOptInitDone() const {
    return AIXFuncTLSModelOptInitDone;
  }
  void setAIXFuncUseTLSIEForLD() { AIXFuncUseTLSIEForLD = true; }
```
- **EN**: Implements helper routine(s) `setHasSpills`, `hasSpills`, `setHasNonRISpills` for this portion of the PowerPC backend public or internal declarations for the backend.
- **CN**: 这里实现了 PowerPC 后端该部分后端的公共或内部声明所需的辅助例程 `setHasSpills`, `hasSpills`, `setHasNonRISpills`。

### Lines 234-259

```cpp
  bool isAIXFuncUseTLSIEForLD() const { return AIXFuncUseTLSIEForLD; }

  int getVarArgsFrameIndex() const { return VarArgsFrameIndex; }
  void setVarArgsFrameIndex(int Index) { VarArgsFrameIndex = Index; }

  int getVarArgsStackOffset() const { return VarArgsStackOffset; }
  void setVarArgsStackOffset(int Offset) { VarArgsStackOffset = Offset; }

  unsigned getVarArgsNumGPR() const { return VarArgsNumGPR; }
  void setVarArgsNumGPR(unsigned Num) { VarArgsNumGPR = Num; }

  unsigned getFixedParmsNum() const { return FixedParmsNum; }
  unsigned getFloatingPointParmsNum() const { return FloatingParmsNum; }
  unsigned getVectorParmsNum() const { return VectorParmsNum; }
  bool hasVectorParms() const { return VectorParmsNum != 0; }

  uint32_t getParmsType() const;

  uint32_t getVecExtParmsType() const;

  void appendParameterType(ParamType Type);

  unsigned getVarArgsNumFPR() const { return VarArgsNumFPR; }
  void setVarArgsNumFPR(unsigned Num) { VarArgsNumFPR = Num; }

  /// This function associates attributes for each live-in virtual register.
```
- **EN**: Implements helper routine(s) `isAIXFuncUseTLSIEForLD`, `getVarArgsFrameIndex`, `setVarArgsFrameIndex` for this portion of the PowerPC backend public or internal declarations for the backend. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 PowerPC 后端该部分后端的公共或内部声明所需的辅助例程 `isAIXFuncUseTLSIEForLD`, `getVarArgsFrameIndex`, `setVarArgsFrameIndex`。 该区间与栈帧布局或栈访问相关。

### Lines 260-285

```cpp
  void addLiveInAttr(Register VReg, ISD::ArgFlagsTy Flags) {
    LiveInAttrs.push_back(std::make_pair(VReg, Flags));
  }

  /// This function returns true if the specified vreg is
  /// a live-in register and sign-extended.
  bool isLiveInSExt(Register VReg) const;

  /// This function returns true if the specified vreg is
  /// a live-in register and zero-extended.
  bool isLiveInZExt(Register VReg) const;

  int getCRSpillFrameIndex() const { return CRSpillFrameIndex; }
  void setCRSpillFrameIndex(int idx) { CRSpillFrameIndex = idx; }

  const SmallVectorImpl<Register> &
    getMustSaveCRs() const { return MustSaveCRs; }
  void addMustSaveCR(Register Reg) { MustSaveCRs.push_back(Reg); }

  void setUsesPICBase(bool uses) { UsesPICBase = uses; }
  bool usesPICBase() const { return UsesPICBase; }

  MCSymbol *getPICOffsetSymbol(MachineFunction &MF) const;

  MCSymbol *getGlobalEPSymbol(MachineFunction &MF) const;
  MCSymbol *getLocalEPSymbol(MachineFunction &MF) const;
```
- **EN**: Implements helper routine(s) `addLiveInAttr`, `push_back`, `make_pair` for this portion of the PowerPC backend public or internal declarations for the backend. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这里实现了 PowerPC 后端该部分后端的公共或内部声明所需的辅助例程 `addLiveInAttr`, `push_back`, `make_pair`。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 286-291

```cpp
  MCSymbol *getTOCOffsetSymbol(MachineFunction &MF) const;
};

} // end namespace llvm

#endif // LLVM_LIB_TARGET_POWERPC_PPCMACHINEFUNCTIONINFO_H
```
- **EN**: Defines or closes the header guard so the declarations in this header are only processed once per translation unit. Notable symbols in this range include `getTOCOffsetSymbol`.
- **CN**: 这里定义或结束头文件保护宏，确保该头文件中的声明在一个编译单元内只会被处理一次。 该区间中较显眼的符号包括 `getTOCOffsetSymbol`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- Public or internal declarations for the backend / 后端的公共或内部声明
- Calling convention handling / 调用约定处理
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Object format integration / 目标文件格式集成
- Declarative TableGen records / 声明式 TableGen 记录

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `llvm/ADT/SmallVector.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/TargetCallingConv.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
