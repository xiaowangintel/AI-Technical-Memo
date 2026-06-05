# WebAssemblyFrameLowering.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/WebAssembly/WebAssemblyFrameLowering.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file contains the WebAssembly implementation of TargetFrameLowering class.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/WebAssembly/WebAssemblyFrameLowering.cpp`，主要负责 WebAssembly 后端的栈帧 lowering 逻辑。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-- WebAssemblyFrameLowering.cpp - WebAssembly Frame Lowering ----------==//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。 该区间与栈帧布局或栈访问相关。

### Lines 7-17

```cpp
//===----------------------------------------------------------------------===//
///
/// \file
/// This file contains the WebAssembly implementation of
/// TargetFrameLowering class.
///
/// On WebAssembly, there aren't a lot of things to do here. There are no
/// callee-saved registers to save, and no spill slots.
///
/// The stack grows downward.
///
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "\file". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“\file”。 该区间与栈帧布局或栈访问相关。

### Lines 18-24

```cpp
//===----------------------------------------------------------------------===//

#include "WebAssemblyFrameLowering.h"
#include "MCTargetDesc/WebAssemblyMCTargetDesc.h"
#include "Utils/WebAssemblyTypeUtilities.h"
#include "WebAssembly.h"
#include "WebAssemblyInstrInfo.h"
```
- **EN**: Pulls in direct dependencies required by this stack frame lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这一段引入该栈帧 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 该区间与栈帧布局或栈访问相关。

### Lines 25-31

```cpp
#include "WebAssemblyMachineFunctionInfo.h"
#include "WebAssemblySubtarget.h"
#include "WebAssemblyTargetMachine.h"
#include "llvm/CodeGen/Analysis.h"
#include "llvm/CodeGen/MachineFrameInfo.h"
#include "llvm/CodeGen/MachineFunction.h"
#include "llvm/CodeGen/MachineInstrBuilder.h"
```
- **EN**: Pulls in direct dependencies required by this stack frame lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这一段引入该栈帧 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 32-69

```cpp
#include "llvm/CodeGen/MachineRegisterInfo.h"
#include "llvm/IR/Instructions.h"
#include "llvm/MC/MCAsmInfo.h"
using namespace llvm;

#define DEBUG_TYPE "wasm-frame-info"

// TODO: wasm64
// TODO: Emit TargetOpcode::CFI_INSTRUCTION instructions

// In an ideal world, when objects are added to the MachineFrameInfo by
// FunctionLoweringInfo::set, we could somehow hook into target-specific code to
// ensure they are assigned the right stack ID.  However there isn't a hook that
// runs between then and DAG building time, though, so instead we hoist stack
// objects lazily when they are first used, and comprehensively after the DAG is
// built via the PreprocessISelDAG hook, called by the
// SelectionDAGISel::runOnMachineFunction.  We have to do it in two places
// because we want to do it while building the selection DAG for uses of alloca,
// but not all alloca instructions are used so we have to follow up afterwards.
std::optional<unsigned>
WebAssemblyFrameLowering::getLocalForStackObject(MachineFunction &MF,
                                                 int FrameIndex) {
  MachineFrameInfo &MFI = MF.getFrameInfo();

  // If already hoisted to a local, done.
  if (MFI.getStackID(FrameIndex) == TargetStackID::WasmLocal)
    return static_cast<unsigned>(MFI.getObjectOffset(FrameIndex));

  // If not allocated in the object address space, this object will be in
  // linear memory.
  const AllocaInst *AI = MFI.getObjectAllocation(FrameIndex);
  if (!AI || !WebAssembly::isWasmVarAddressSpace(AI->getAddressSpace()))
    return std::nullopt;

  // Otherwise, allocate this object in the named value stack, outside of linear
  // memory.
  SmallVector<EVT, 4> ValueVTs;
  const WebAssemblyTargetLowering &TLI =
```
- **EN**: Pulls in direct dependencies required by this stack frame lowering logic, so later declarations can reuse LLVM infrastructure and target-specific helpers. This range works in the SelectionDAG-based lowering pipeline.
- **CN**: 这一段引入该栈帧 lowering 逻辑所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 这一段工作在基于 SelectionDAG 的 lowering 流水线中。

### Lines 70-107

```cpp
      *MF.getSubtarget<WebAssemblySubtarget>().getTargetLowering();
  WebAssemblyFunctionInfo *FuncInfo = MF.getInfo<WebAssemblyFunctionInfo>();
  ComputeValueVTs(TLI, MF.getDataLayout(), AI->getAllocatedType(), ValueVTs);
  MFI.setStackID(FrameIndex, TargetStackID::WasmLocal);
  // Abuse SP offset to record the index of the first local in the object.
  unsigned Local = FuncInfo->getParams().size() + FuncInfo->getLocals().size();
  MFI.setObjectOffset(FrameIndex, Local);
  // Allocate WebAssembly locals for each non-aggregate component of the
  // allocation.
  for (EVT ValueVT : ValueVTs)
    FuncInfo->addLocal(ValueVT.getSimpleVT());
  // Abuse object size to record number of WebAssembly locals allocated to
  // this object.
  MFI.setObjectSize(FrameIndex, ValueVTs.size());
  return Local;
}

/// We need a base pointer in the case of having items on the stack that
/// require stricter alignment than the stack pointer itself.  Because we need
/// to shift the stack pointer by some unknown amount to force the alignment,
/// we need to record the value of the stack pointer on entry to the function.
bool WebAssemblyFrameLowering::hasBP(const MachineFunction &MF) const {
  const auto *RegInfo =
      MF.getSubtarget<WebAssemblySubtarget>().getRegisterInfo();
  return RegInfo->hasStackRealignment(MF);
}

/// Return true if the specified function should have a dedicated frame pointer
/// register.
bool WebAssemblyFrameLowering::hasFPImpl(const MachineFunction &MF) const {
  const MachineFrameInfo &MFI = MF.getFrameInfo();

  // When we have var-sized objects, we move the stack pointer by an unknown
  // amount, and need to emit a frame pointer to restore the stack to where we
  // were on function entry.
  // If we already need a base pointer, we use that to fix up the stack pointer.
  // If there are no fixed-size objects, we would have no use of a frame
  // pointer, and thus should not emit one.
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "MF.getSubtarget<WebAssemblySubtarget>().getTargetLowering();". Register assignment and register-class constraints matter here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“MF.getSubtarget<WebAssemblySubtarget>().getTargetLowering();”。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 108-145

```cpp
  bool HasFixedSizedObjects = MFI.getStackSize() > 0;
  bool NeedsFixedReference = !hasBP(MF) || HasFixedSizedObjects;

  return MFI.isFrameAddressTaken() ||
         (MFI.hasVarSizedObjects() && NeedsFixedReference) ||
         MFI.hasStackMap() || MFI.hasPatchPoint();
}

/// Under normal circumstances, when a frame pointer is not required, we reserve
/// argument space for call sites in the function immediately on entry to the
/// current function. This eliminates the need for add/sub sp brackets around
/// call sites. Returns true if the call frame is included as part of the stack
/// frame.
bool WebAssemblyFrameLowering::hasReservedCallFrame(
    const MachineFunction &MF) const {
  return !MF.getFrameInfo().hasVarSizedObjects();
}

// Returns true if this function needs a local user-space stack pointer for its
// local frame (not for exception handling).
bool WebAssemblyFrameLowering::needsSPForLocalFrame(
    const MachineFunction &MF) const {
  auto &MFI = MF.getFrameInfo();
  auto &MRI = MF.getRegInfo();
  // llvm.stacksave can explicitly read SP register and it can appear without
  // dynamic alloca.
  bool HasExplicitSPUse =
      any_of(MRI.use_operands(getSPReg(MF)),
             [](MachineOperand &MO) { return !MO.isImplicit(); });

  return MFI.getStackSize() || MFI.adjustsStack() || hasFP(MF) ||
         HasExplicitSPUse;
}

// In function with EH pads, we need to make a copy of the value of
// __stack_pointer global in SP32/64 register, in order to use it when
// restoring __stack_pointer after an exception is caught.
bool WebAssemblyFrameLowering::needsPrologForEH(
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Under normal circumstances, when a frame pointer is not required, we reserve". It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Under normal circumstances, when a frame pointer is not required, we reserve”。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 146-183

```cpp
    const MachineFunction &MF) const {
  auto EHType = MF.getTarget().getMCAsmInfo().getExceptionHandlingType();
  return EHType == ExceptionHandling::Wasm &&
         MF.getFunction().hasPersonalityFn() && MF.getFrameInfo().hasCalls();
}

/// Returns true if this function needs a local user-space stack pointer.
/// Unlike a machine stack pointer, the wasm user stack pointer is a global
/// variable, so it is loaded into a register in the prolog.
bool WebAssemblyFrameLowering::needsSP(const MachineFunction &MF) const {
  return needsSPForLocalFrame(MF) || needsPrologForEH(MF);
}

/// Returns true if the local user-space stack pointer needs to be written back
/// to __stack_pointer global by this function (this is not meaningful if
/// needsSP is false). If false, the stack red zone can be used and only a local
/// SP is needed.
bool WebAssemblyFrameLowering::needsSPWriteback(
    const MachineFunction &MF) const {
  auto &MFI = MF.getFrameInfo();
  assert(needsSP(MF));
  // When we don't need a local stack pointer for its local frame but only to
  // support EH, we don't need to write SP back in the epilog, because we don't
  // bump down the stack pointer in the prolog. We need to write SP back in the
  // epilog only if
  // 1. We need SP not only for EH support but also because we actually use
  // stack or we have a frame address taken.
  // 2. We cannot use the red zone.
  bool CanUseRedZone = MFI.getStackSize() <= RedZoneSize && !MFI.hasCalls() &&
                       !MF.getFunction().hasFnAttribute(Attribute::NoRedZone);
  return needsSPForLocalFrame(MF) && !CanUseRedZone;
}

unsigned WebAssemblyFrameLowering::getSPReg(const MachineFunction &MF) {
  return MF.getSubtarget<WebAssemblySubtarget>().hasAddr64()
             ? WebAssembly::SP64
             : WebAssembly::SP32;
}
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Returns true if this function needs a local user-space stack pointer.". Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Returns true if this function needs a local user-space stack pointer.”。 该区间与栈帧布局或栈访问相关。

### Lines 184-221

```cpp

unsigned WebAssemblyFrameLowering::getFPReg(const MachineFunction &MF) {
  return MF.getSubtarget<WebAssemblySubtarget>().hasAddr64()
             ? WebAssembly::FP64
             : WebAssembly::FP32;
}

unsigned
WebAssemblyFrameLowering::getOpcConst(const MachineFunction &MF) {
  return MF.getSubtarget<WebAssemblySubtarget>().hasAddr64()
             ? WebAssembly::CONST_I64
             : WebAssembly::CONST_I32;
}

unsigned WebAssemblyFrameLowering::getOpcAdd(const MachineFunction &MF) {
  return MF.getSubtarget<WebAssemblySubtarget>().hasAddr64()
             ? WebAssembly::ADD_I64
             : WebAssembly::ADD_I32;
}

unsigned WebAssemblyFrameLowering::getOpcSub(const MachineFunction &MF) {
  return MF.getSubtarget<WebAssemblySubtarget>().hasAddr64()
             ? WebAssembly::SUB_I64
             : WebAssembly::SUB_I32;
}

unsigned WebAssemblyFrameLowering::getOpcAnd(const MachineFunction &MF) {
  return MF.getSubtarget<WebAssemblySubtarget>().hasAddr64()
             ? WebAssembly::AND_I64
             : WebAssembly::AND_I32;
}

unsigned
WebAssemblyFrameLowering::getOpcGlobGet(const MachineFunction &MF) {
  return MF.getSubtarget<WebAssemblySubtarget>().hasAddr64()
             ? WebAssembly::GLOBAL_GET_I64
             : WebAssembly::GLOBAL_GET_I32;
}
```
- **EN**: Implements helper routine(s) `getFPReg`, `hasAddr64`, `getOpcConst` for this portion of the WebAssembly backend stack frame lowering logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分栈帧 lowering 逻辑所需的辅助例程 `getFPReg`, `hasAddr64`, `getOpcConst`。 该区间与栈帧布局或栈访问相关。

### Lines 222-259

```cpp

unsigned
WebAssemblyFrameLowering::getOpcGlobSet(const MachineFunction &MF) {
  return MF.getSubtarget<WebAssemblySubtarget>().hasAddr64()
             ? WebAssembly::GLOBAL_SET_I64
             : WebAssembly::GLOBAL_SET_I32;
}

void WebAssemblyFrameLowering::writeSPToGlobal(
    unsigned SrcReg, MachineFunction &MF, MachineBasicBlock &MBB,
    MachineBasicBlock::iterator &InsertStore, const DebugLoc &DL) const {
  const auto *TII = MF.getSubtarget<WebAssemblySubtarget>().getInstrInfo();

  const char *ES = "__stack_pointer";
  auto *SPSymbol = MF.createExternalSymbolName(ES);

  BuildMI(MBB, InsertStore, DL, TII->get(getOpcGlobSet(MF)))
      .addExternalSymbol(SPSymbol)
      .addReg(SrcReg);
}

MachineBasicBlock::iterator
WebAssemblyFrameLowering::eliminateCallFramePseudoInstr(
    MachineFunction &MF, MachineBasicBlock &MBB,
    MachineBasicBlock::iterator I) const {
  assert(!I->getOperand(0).getImm() && (hasFP(MF) || hasBP(MF)) &&
         "Call frame pseudos should only be used for dynamic stack adjustment");
  auto &ST = MF.getSubtarget<WebAssemblySubtarget>();
  const auto *TII = ST.getInstrInfo();
  if (I->getOpcode() == TII->getCallFrameDestroyOpcode() &&
      needsSPWriteback(MF)) {
    DebugLoc DL = I->getDebugLoc();
    writeSPToGlobal(getSPReg(MF), MF, MBB, I, DL);
  }
  return MBB.erase(I);
}

void WebAssemblyFrameLowering::emitPrologue(MachineFunction &MF,
```
- **EN**: Implements helper routine(s) `getOpcGlobSet`, `hasAddr64`, `writeSPToGlobal` for this portion of the WebAssembly backend stack frame lowering logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分栈帧 lowering 逻辑所需的辅助例程 `getOpcGlobSet`, `hasAddr64`, `writeSPToGlobal`。 该区间与栈帧布局或栈访问相关。

### Lines 260-297

```cpp
                                            MachineBasicBlock &MBB) const {
  // TODO: Do ".setMIFlag(MachineInstr::FrameSetup)" on emitted instructions
  auto &MFI = MF.getFrameInfo();
  assert(MFI.getCalleeSavedInfo().empty() &&
         "WebAssembly should not have callee-saved registers");

  if (!needsSP(MF))
    return;
  uint64_t StackSize = MFI.getStackSize();

  auto &ST = MF.getSubtarget<WebAssemblySubtarget>();
  const auto *TII = ST.getInstrInfo();
  auto &MRI = MF.getRegInfo();

  auto InsertPt = MBB.begin();
  while (InsertPt != MBB.end() &&
         WebAssembly::isArgument(InsertPt->getOpcode()))
    ++InsertPt;
  DebugLoc DL;

  const TargetRegisterClass *PtrRC =
      MRI.getTargetRegisterInfo()->getPointerRegClass();
  unsigned SPReg = getSPReg(MF);
  if (StackSize)
    SPReg = MRI.createVirtualRegister(PtrRC);

  const char *ES = "__stack_pointer";
  auto *SPSymbol = MF.createExternalSymbolName(ES);
  BuildMI(MBB, InsertPt, DL, TII->get(getOpcGlobGet(MF)), SPReg)
      .addExternalSymbol(SPSymbol);

  bool HasBP = hasBP(MF);
  if (HasBP) {
    auto FI = MF.getInfo<WebAssemblyFunctionInfo>();
    Register BasePtr = MRI.createVirtualRegister(PtrRC);
    FI->setBasePointerVreg(BasePtr);
    BuildMI(MBB, InsertPt, DL, TII->get(WebAssembly::COPY), BasePtr)
        .addReg(SPReg);
```
- **EN**: Implements helper routine(s) `setMIFlag`, `getFrameInfo`, `getCalleeSavedInfo` for this portion of the WebAssembly backend stack frame lowering logic. It is closely tied to LLVM machine-level IR construction and manipulation.
- **CN**: 这里实现了 WebAssembly 后端该部分栈帧 lowering 逻辑所需的辅助例程 `setMIFlag`, `getFrameInfo`, `getCalleeSavedInfo`。 它与 LLVM 的机器级 IR 构建和操作紧密相关。

### Lines 298-335

```cpp
  }
  if (StackSize) {
    // Subtract the frame size
    Register OffsetReg = MRI.createVirtualRegister(PtrRC);
    BuildMI(MBB, InsertPt, DL, TII->get(getOpcConst(MF)), OffsetReg)
        .addImm(StackSize);
    BuildMI(MBB, InsertPt, DL, TII->get(getOpcSub(MF)), getSPReg(MF))
        .addReg(SPReg)
        .addReg(OffsetReg);
  }
  if (HasBP) {
    Register BitmaskReg = MRI.createVirtualRegister(PtrRC);
    Align Alignment = MFI.getMaxAlign();
    BuildMI(MBB, InsertPt, DL, TII->get(getOpcConst(MF)), BitmaskReg)
        .addImm((int64_t) ~(Alignment.value() - 1));
    BuildMI(MBB, InsertPt, DL, TII->get(getOpcAnd(MF)), getSPReg(MF))
        .addReg(getSPReg(MF))
        .addReg(BitmaskReg);
  }
  if (hasFP(MF)) {
    // Unlike most conventional targets (where FP points to the saved FP),
    // FP points to the bottom of the fixed-size locals, so we can use positive
    // offsets in load/store instructions.
    BuildMI(MBB, InsertPt, DL, TII->get(WebAssembly::COPY), getFPReg(MF))
        .addReg(getSPReg(MF));
  }
  if (StackSize && needsSPWriteback(MF)) {
    writeSPToGlobal(getSPReg(MF), MF, MBB, InsertPt, DL);
  }
}

void WebAssemblyFrameLowering::emitEpilogue(MachineFunction &MF,
                                            MachineBasicBlock &MBB) const {
  uint64_t StackSize = MF.getFrameInfo().getStackSize();
  if (!needsSP(MF) || !needsSPWriteback(MF))
    return;
  auto &ST = MF.getSubtarget<WebAssemblySubtarget>();
  const auto *TII = ST.getInstrInfo();
```
- **EN**: Implements helper routine(s) `createVirtualRegister`, `BuildMI`, `get` for this portion of the WebAssembly backend stack frame lowering logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分栈帧 lowering 逻辑所需的辅助例程 `createVirtualRegister`, `BuildMI`, `get`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 336-373

```cpp
  auto &MRI = MF.getRegInfo();
  auto InsertPt = MBB.getFirstTerminator();
  DebugLoc DL;

  if (InsertPt != MBB.end())
    DL = InsertPt->getDebugLoc();

  // Restore the stack pointer. If we had fixed-size locals, add the offset
  // subtracted in the prolog.
  unsigned SPReg = 0;
  unsigned SPFPReg = hasFP(MF) ? getFPReg(MF) : getSPReg(MF);
  if (hasBP(MF)) {
    auto FI = MF.getInfo<WebAssemblyFunctionInfo>();
    SPReg = FI->getBasePointerVreg();
  } else if (StackSize) {
    const TargetRegisterClass *PtrRC =
        MRI.getTargetRegisterInfo()->getPointerRegClass();
    Register OffsetReg = MRI.createVirtualRegister(PtrRC);
    BuildMI(MBB, InsertPt, DL, TII->get(getOpcConst(MF)), OffsetReg)
        .addImm(StackSize);
    // In the epilog we don't need to write the result back to the SP32/64
    // physreg because it won't be used again. We can use a stackified register
    // instead.
    SPReg = MRI.createVirtualRegister(PtrRC);
    BuildMI(MBB, InsertPt, DL, TII->get(getOpcAdd(MF)), SPReg)
        .addReg(SPFPReg)
        .addReg(OffsetReg);
  } else {
    SPReg = SPFPReg;
  }

  writeSPToGlobal(SPReg, MF, MBB, InsertPt, DL);
}

bool WebAssemblyFrameLowering::isSupportedStackID(
    TargetStackID::Value ID) const {
  // Use the Object stack for WebAssembly locals which can only be accessed
  // by name, not via an address in linear memory.
```
- **EN**: Implements helper routine(s) `getRegInfo`, `getFirstTerminator`, `end` for this portion of the WebAssembly backend stack frame lowering logic. Register assignment and register-class constraints matter here.
- **CN**: 这里实现了 WebAssembly 后端该部分栈帧 lowering 逻辑所需的辅助例程 `getRegInfo`, `getFirstTerminator`, `end`。 这里重点涉及寄存器分配与寄存器类约束。

### Lines 374-394

```cpp
  if (ID == TargetStackID::WasmLocal)
    return true;

  return TargetFrameLowering::isSupportedStackID(ID);
}

TargetFrameLowering::DwarfFrameBase
WebAssemblyFrameLowering::getDwarfFrameBase(const MachineFunction &MF) const {
  DwarfFrameBase Loc;
  Loc.Kind = DwarfFrameBase::WasmFrameBase;
  const WebAssemblyFunctionInfo &MFI = *MF.getInfo<WebAssemblyFunctionInfo>();
  if (needsSP(MF) && MFI.isFrameBaseVirtual()) {
    unsigned LocalNum = MFI.getFrameBaseLocal();
    Loc.Location.WasmLoc = {WebAssembly::TI_LOCAL, LocalNum};
  } else {
    // TODO: This should work on a breakpoint at a function with no frame,
    // but probably won't work for traversing up the stack.
    Loc.Location.WasmLoc = {WebAssembly::TI_GLOBAL_RELOC, 0};
  }
  return Loc;
}
```
- **EN**: Implements helper routine(s) `isSupportedStackID`, `getDwarfFrameBase`, `needsSP` for this portion of the WebAssembly backend stack frame lowering logic. Stack-frame layout or stack access is relevant in this range.
- **CN**: 这里实现了 WebAssembly 后端该部分栈帧 lowering 逻辑所需的辅助例程 `isSupportedStackID`, `getDwarfFrameBase`, `needsSP`。 该区间与栈帧布局或栈访问相关。

## Key Concepts / 关键概念

- WebAssembly backend integration / WebAssembly 后端集成
- Stack frame lowering logic / 栈帧 lowering 逻辑
- SelectionDAG lowering / SelectionDAG lowering
- Register modeling / 寄存器建模
- Stack frame management / 栈帧管理
- Subtarget features / 子目标特性
- Instruction selection or opcode handling / 指令选择或操作码处理
- Target machine configuration / 目标机器配置

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `WebAssemblyFrameLowering.h`
- `MCTargetDesc/WebAssemblyMCTargetDesc.h`
- `Utils/WebAssemblyTypeUtilities.h`
- `WebAssembly.h`
- `WebAssemblyInstrInfo.h`
- `WebAssemblyMachineFunctionInfo.h`
- `WebAssemblySubtarget.h`
- `WebAssemblyTargetMachine.h`
- `llvm/CodeGen/Analysis.h`
- `llvm/CodeGen/MachineFrameInfo.h`
- `llvm/CodeGen/MachineFunction.h`
- `llvm/CodeGen/MachineInstrBuilder.h`
- `llvm/CodeGen/MachineRegisterInfo.h`
- `llvm/IR/Instructions.h`
- `llvm/MC/MCAsmInfo.h`

### Important Collaborators / 重要协作组件

- LLVM CodeGen layer / LLVM CodeGen 层
- LLVM MC layer / LLVM MC 层
- WebAssembly target-specific helpers / WebAssembly 目标专用辅助组件
