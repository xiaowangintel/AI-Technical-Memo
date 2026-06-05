# AArch64MCLFIRewriter.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/MCTargetDesc/AArch64MCLFIRewriter.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file implements the AArch64MCLFIRewriter class, the AArch64 specific subclass of MCLFIRewriter. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Documented code section
```cpp
//===- AArch64MCLFIRewriter.cpp ---------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file implements the AArch64MCLFIRewriter class, the AArch64 specific
// subclass of MCLFIRewriter.
//
//===----------------------------------------------------------------------===//

#include "AArch64MCLFIRewriter.h"
#include "AArch64AddressingModes.h"
#include "MCTargetDesc/AArch64MCTargetDesc.h"
#include "Utils/AArch64BaseInfo.h"

#include "llvm/ADT/Twine.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/MC/MCSubtargetInfo.h"

using namespace llvm;

// LFI reserved registers.
static constexpr MCRegister LFIBaseReg = AArch64::X27;
static constexpr MCRegister LFIAddrReg = AArch64::X28;
static constexpr MCRegister LFIScratchReg = AArch64::X26;
static constexpr MCRegister LFICtxReg = AArch64::X25;

// Offset into the context register block (pointed to by LFICtxReg) where the
// thread pointer is stored. This is a scaled offset (multiplied by 8 for
// 64-bit loads), so a value of 2 means an actual byte offset of 16.
static constexpr unsigned LFITPOffset = 2;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 36-66: Function isSyscall
```cpp

// Byte offset from the sandbox base register where the syscall handler address
// is stored (negative because it is below the sandbox base).
static constexpr int LFISyscallOffset = -8;

static bool isSyscall(const MCInst &Inst) {
  return Inst.getOpcode() == AArch64::SVC;
}

static bool isPrivilegedTP(int64_t Reg) {
  return Reg == AArch64SysReg::TPIDR_EL1 || Reg == AArch64SysReg::TPIDR_EL2 ||
         Reg == AArch64SysReg::TPIDR_EL3;
}

static bool isTPRead(const MCInst &Inst) {
  return Inst.getOpcode() == AArch64::MRS &&
         Inst.getOperand(1).getImm() == AArch64SysReg::TPIDR_EL0;
}

static bool isTPWrite(const MCInst &Inst) {
  return Inst.getOpcode() == AArch64::MSR &&
         Inst.getOperand(0).getImm() == AArch64SysReg::TPIDR_EL0;
}

static bool isPrivilegedTPAccess(const MCInst &Inst) {
  if (Inst.getOpcode() == AArch64::MRS)
    return isPrivilegedTP(Inst.getOperand(1).getImm());
  if (Inst.getOpcode() == AArch64::MSR)
    return isPrivilegedTP(Inst.getOperand(0).getImm());
  return false;
}
```
**EN:** This block implements isSyscall, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 isSyscall，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 67-93: Function AArch64MCLFIRewriter::mayModifyReserved
```cpp

MCRegister AArch64MCLFIRewriter::mayModifyReserved(const MCInst &Inst) const {
  for (MCRegister Reg : {LFIAddrReg, LFIBaseReg, LFICtxReg}) {
    if (mayModifyRegister(Inst, Reg))
      return Reg;
  }
  return {};
}

void AArch64MCLFIRewriter::emitInst(const MCInst &Inst, MCStreamer &Out,
                                    const MCSubtargetInfo &STI) {
  Out.emitInstruction(Inst, STI);
}

void AArch64MCLFIRewriter::emitAddMask(MCRegister Dest, MCRegister Src,
                                       MCStreamer &Out,
                                       const MCSubtargetInfo &STI) {
  // add Dest, LFIBaseReg, W(Src), uxtw
  MCInst Inst;
  Inst.setOpcode(AArch64::ADDXrx);
  Inst.addOperand(MCOperand::createReg(Dest));
  Inst.addOperand(MCOperand::createReg(LFIBaseReg));
  Inst.addOperand(MCOperand::createReg(getWRegFromXReg(Src)));
  Inst.addOperand(
      MCOperand::createImm(AArch64_AM::getArithExtendImm(AArch64_AM::UXTW, 0)));
  emitInst(Inst, Out, STI);
}
```
**EN:** This block implements AArch64MCLFIRewriter::mayModifyReserved, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64MCLFIRewriter::mayModifyReserved，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 94-126: Function AArch64MCLFIRewriter::emitBranch
```cpp

void AArch64MCLFIRewriter::emitBranch(unsigned Opcode, MCRegister Target,
                                      MCStreamer &Out,
                                      const MCSubtargetInfo &STI) {
  MCInst Branch;
  Branch.setOpcode(Opcode);
  Branch.addOperand(MCOperand::createReg(Target));
  emitInst(Branch, Out, STI);
}

void AArch64MCLFIRewriter::emitMov(MCRegister Dest, MCRegister Src,
                                   MCStreamer &Out,
                                   const MCSubtargetInfo &STI) {
  // orr Dest, xzr, Src
  MCInst Inst;
  Inst.setOpcode(AArch64::ORRXrs);
  Inst.addOperand(MCOperand::createReg(Dest));
  Inst.addOperand(MCOperand::createReg(AArch64::XZR));
  Inst.addOperand(MCOperand::createReg(Src));
  Inst.addOperand(MCOperand::createImm(0));
  emitInst(Inst, Out, STI);
}

// {br,blr} xN
// ->
// add x28, x27, wN, uxtw
// {br,blr} x28
void AArch64MCLFIRewriter::rewriteIndirectBranch(const MCInst &Inst,
                                                 MCStreamer &Out,
                                                 const MCSubtargetInfo &STI) {
  assert(Inst.getNumOperands() >= 1 && Inst.getOperand(0).isReg() &&
         "expected register operand");
  MCRegister BranchReg = Inst.getOperand(0).getReg();
```
**EN:** This block implements AArch64MCLFIRewriter::emitBranch, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64MCLFIRewriter::emitBranch，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 127-159: Documented code section
```cpp

  // Guard the branch target through X28.
  emitAddMask(LFIAddrReg, BranchReg, Out, STI);
  emitBranch(Inst.getOpcode(), LFIAddrReg, Out, STI);
}

// ret xN (where xN != x30)
// ->
// add x28, x27, wN, uxtw
// ret x28
//
// ret (x30) is safe since x30 is always within the sandbox.
void AArch64MCLFIRewriter::rewriteReturn(const MCInst &Inst, MCStreamer &Out,
                                         const MCSubtargetInfo &STI) {
  assert(Inst.getNumOperands() >= 1 && Inst.getOperand(0).isReg() &&
         "expected register operand");
  // RET through LR is safe since LR is always within sandbox.
  if (Inst.getOperand(0).getReg() != AArch64::LR)
    rewriteIndirectBranch(Inst, Out, STI);
  else
    emitInst(Inst, Out, STI);
}

// modify x30
// ->
// modify x30
// add x30, x27, w30, uxtw
void AArch64MCLFIRewriter::rewriteLRModification(const MCInst &Inst,
                                                 MCStreamer &Out,
                                                 const MCSubtargetInfo &STI) {
  emitInst(Inst, Out, STI);
  emitAddMask(AArch64::LR, AArch64::LR, Out, STI);
}
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 160-192: Function AArch64MCLFIRewriter::rewriteSyscall
```cpp

// svc #0
// ->
// mov x26, x30
// ldur x30, [x27, #-8]
// blr x30
// add x30, x27, w26, uxtw
void AArch64MCLFIRewriter::rewriteSyscall(const MCInst &, MCStreamer &Out,
                                          const MCSubtargetInfo &STI) {
  // Save LR to scratch.
  emitMov(LFIScratchReg, AArch64::LR, Out, STI);

  // Load syscall handler address from negative offset from sandbox base.
  MCInst Load;
  Load.setOpcode(AArch64::LDURXi);
  Load.addOperand(MCOperand::createReg(AArch64::LR));
  Load.addOperand(MCOperand::createReg(LFIBaseReg));
  Load.addOperand(MCOperand::createImm(LFISyscallOffset));
  emitInst(Load, Out, STI);

  // Call the runtime.
  emitBranch(AArch64::BLR, AArch64::LR, Out, STI);

  // Restore LR with guard.
  emitAddMask(AArch64::LR, LFIScratchReg, Out, STI);
}

// mrs xN, tpidr_el0
// ->
// ldr xN, [x25, #16]
void AArch64MCLFIRewriter::rewriteTPRead(const MCInst &Inst, MCStreamer &Out,
                                         const MCSubtargetInfo &STI) {
  MCRegister DestReg = Inst.getOperand(0).getReg();
```
**EN:** This block implements AArch64MCLFIRewriter::rewriteSyscall, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64MCLFIRewriter::rewriteSyscall，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 193-226: Core AArch64 backend logic
```cpp

  MCInst Load;
  Load.setOpcode(AArch64::LDRXui);
  Load.addOperand(MCOperand::createReg(DestReg));
  Load.addOperand(MCOperand::createReg(LFICtxReg));
  Load.addOperand(MCOperand::createImm(LFITPOffset));
  emitInst(Load, Out, STI);
}

// msr tpidr_el0, xN
// ->
// str xN, [x25, #16]
void AArch64MCLFIRewriter::rewriteTPWrite(const MCInst &Inst, MCStreamer &Out,
                                          const MCSubtargetInfo &STI) {
  MCRegister SrcReg = Inst.getOperand(1).getReg();

  MCInst Store;
  Store.setOpcode(AArch64::STRXui);
  Store.addOperand(MCOperand::createReg(SrcReg));
  Store.addOperand(MCOperand::createReg(LFICtxReg));
  Store.addOperand(MCOperand::createImm(LFITPOffset));
  emitInst(Store, Out, STI);
}

// NOTE: when adding new rewrites, the size estimates in
// AArch64InstrInfo::getLFIInstSizeInBytes must be updated to match.
void AArch64MCLFIRewriter::doRewriteInst(const MCInst &Inst, MCStreamer &Out,
                                         const MCSubtargetInfo &STI) {
  // Reserved register modification is an error.
  if (MCRegister Reg = mayModifyReserved(Inst)) {
    error(Inst, Twine("illegal modification of reserved LFI register ") +
                    RegInfo->getName(Reg));
    return;
  }
```
**EN:** This block continues the file's main AArch64 backend logic logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的AArch64 后端逻辑主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 227-257: Documented code section
```cpp

  // System instructions.
  if (isSyscall(Inst))
    return rewriteSyscall(Inst, Out, STI);

  if (isTPRead(Inst))
    return rewriteTPRead(Inst, Out, STI);

  if (isTPWrite(Inst))
    return rewriteTPWrite(Inst, Out, STI);

  if (isPrivilegedTPAccess(Inst)) {
    error(Inst, "illegal access to privileged thread pointer register");
    return;
  }

  // Control flow.
  switch (Inst.getOpcode()) {
  case AArch64::RET:
    return rewriteReturn(Inst, Out, STI);
  case AArch64::BR:
  case AArch64::BLR:
    return rewriteIndirectBranch(Inst, Out, STI);
  }

  // Link register modification.
  if (explicitlyModifiesRegister(Inst, AArch64::LR))
    return rewriteLRModification(Inst, Out, STI);

  emitInst(Inst, Out, STI);
}
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 258-271: Function AArch64MCLFIRewriter::rewriteInst
```cpp

bool AArch64MCLFIRewriter::rewriteInst(const MCInst &Inst, MCStreamer &Out,
                                       const MCSubtargetInfo &STI) {
  // The guard prevents rewrite-recursion when we emit instructions from inside
  // the rewriter (such instructions should not be rewritten).
  if (!Enabled || Guard)
    return false;
  Guard = true;

  doRewriteInst(Inst, Out, STI);

  Guard = false;
  return true;
}
```
**EN:** This block implements AArch64MCLFIRewriter::rewriteInst, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 AArch64MCLFIRewriter::rewriteInst，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64MCLFIRewriter.h, AArch64AddressingModes.h, MCTargetDesc/AArch64MCTargetDesc.h, Utils/AArch64BaseInfo.h **CN:** 目标本地依赖：AArch64MCLFIRewriter.h, AArch64AddressingModes.h, MCTargetDesc/AArch64MCTargetDesc.h, Utils/AArch64BaseInfo.h
- **EN:** Core LLVM interfaces: llvm/ADT/Twine.h, llvm/MC/MCInst.h, llvm/MC/MCStreamer.h, llvm/MC/MCSubtargetInfo.h **CN:** 核心 LLVM 接口：llvm/ADT/Twine.h, llvm/MC/MCInst.h, llvm/MC/MCStreamer.h, llvm/MC/MCSubtargetInfo.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
