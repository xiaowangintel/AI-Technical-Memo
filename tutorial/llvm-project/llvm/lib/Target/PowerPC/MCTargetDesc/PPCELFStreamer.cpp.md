# PPCELFStreamer.cpp — Code Analysis / 代码分析

## Source / 来源

- **File**: `llvm/lib/Target/PowerPC/MCTargetDesc/PPCELFStreamer.cpp`
- **Repository**: `llvm-project`
- **Purpose (EN)**: This file provides MC layer support for the backend for the PowerPC backend.
- **Purpose (CN)**: 该文件位于 `llvm/lib/Target/PowerPC/MCTargetDesc/PPCELFStreamer.cpp`，主要负责 PowerPC 后端的该后端的 MC 层支持。 文件内容以具体实现、辅助函数和后端决策逻辑为主。

## Line-by-Line Analysis / 逐行分析

### Lines 1-6

```cpp
//===-------- PPCELFStreamer.cpp - ELF Object Output ---------------------===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
```
- **EN**: Sets up the file banner, licensing information, and the initial context for the rest of the source file.
- **CN**: 这一段给出文件横幅、许可证信息以及后续源码部分的初始上下文。

### Lines 7-17

```cpp
//===----------------------------------------------------------------------===//
//
// This is a custom MCELFStreamer for PowerPC.
//
// The purpose of the custom ELF streamer is to allow us to intercept
// instructions as they are being emitted and align all 8 byte instructions
// to a 64 byte boundary if required (by adding a 4 byte nop). This is important
// because 8 byte instructions are not allowed to cross 64 byte boundaries
// and by aliging anything that is within 4 bytes of the boundary we can
// guarantee that the 8 byte instructions do not cross that boundary.
//
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "This is a custom MCELFStreamer for PowerPC.". Notable symbols in this range include `required`.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“This is a custom MCELFStreamer for PowerPC.”。 该区间中较显眼的符号包括 `required`。

### Lines 18-24

```cpp
//===----------------------------------------------------------------------===//

#include "PPCELFStreamer.h"
#include "PPCMCAsmInfo.h"
#include "PPCMCCodeEmitter.h"
#include "PPCMCTargetDesc.h"
#include "llvm/BinaryFormat/ELF.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 25-31

```cpp
#include "llvm/MC/MCAsmBackend.h"
#include "llvm/MC/MCAssembler.h"
#include "llvm/MC/MCCodeEmitter.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCInstrDesc.h"
#include "llvm/MC/MCObjectWriter.h"
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. The logic interacts with LLVM's MC layer.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 32-69

```cpp
#include "llvm/Support/SourceMgr.h"

using namespace llvm;

PPCELFStreamer::PPCELFStreamer(MCContext &Context,
                               std::unique_ptr<MCAsmBackend> MAB,
                               std::unique_ptr<MCObjectWriter> OW,
                               std::unique_ptr<MCCodeEmitter> Emitter)
    : MCELFStreamer(Context, std::move(MAB), std::move(OW), std::move(Emitter)),
      LastLabel(nullptr) {}

void PPCELFStreamer::emitPrefixedInstruction(const MCInst &Inst,
                                             const MCSubtargetInfo &STI) {
  // Prefixed instructions must not cross a 64-byte boundary (i.e. prefix is
  // before the boundary and the remaining 4-bytes are after the boundary). In
  // order to achieve this, a nop is added prior to any such boundary-crossing
  // prefixed instruction. Align to 64 bytes if possible but add a maximum of 4
  // bytes when trying to do that. If alignment requires adding more than 4
  // bytes then the instruction won't be aligned. When emitting a code alignment
  // a new fragment is created for this alignment. This fragment will contain
  // all of the nops required as part of the alignment operation. In the cases
  // when no nops are added then The fragment is still created but it remains
  // empty.
  emitCodeAlignment(Align(64), &STI, 4);

  // Emit the instruction.
  // Since the previous emit created a new fragment then adding this instruction
  // also forces the addition of a new fragment. Inst is now the first
  // instruction in that new fragment.
  MCELFStreamer::emitInstruction(Inst, STI);

  // The above instruction is forced to start a new fragment because it
  // comes after a code alignment fragment. Get that new fragment.
  MCFragment *InstructionFragment = getCurrentFragment();
  SMLoc InstLoc = Inst.getLoc();
  // Check if there was a last label emitted.
  if (LastLabel && LastLabel->isDefined() && LastLabelLoc.isValid() &&
      InstLoc.isValid()) {
```
- **EN**: Pulls in direct dependencies required by this MC layer support for the backend, so later declarations can reuse LLVM infrastructure and target-specific helpers. Subtarget feature gating influences the behavior here.
- **CN**: 这一段引入该该后端的 MC 层支持所需的直接依赖，使后续声明能够复用 LLVM 基础设施与目标相关辅助组件。 子目标特性裁剪会影响这里的行为。

### Lines 70-107

```cpp
    const SourceMgr *SourceManager = getContext().getSourceManager();
    unsigned InstLine = SourceManager->FindLineNumber(InstLoc);
    unsigned LabelLine = SourceManager->FindLineNumber(LastLabelLoc);
    // If the Label and the Instruction are on the same line then move the
    // label to the top of the fragment containing the aligned instruction that
    // was just added.
    if (InstLine == LabelLine) {
      LastLabel->setFragment(InstructionFragment);
      LastLabel->setOffset(0);
    }
  }
}

void PPCELFStreamer::emitInstruction(const MCInst &Inst,
                                     const MCSubtargetInfo &STI) {
  auto &Emitter = static_cast<PPCMCCodeEmitter &>(getAssembler().getEmitter());

  // If the instruction is a part of the GOT to PC-Rel link time optimization
  // instruction pair, return a value, otherwise return std::nullopt. A true
  // returned value means the instruction is the PLDpc and a false value means
  // it is the user instruction.
  std::optional<bool> IsPartOfGOTToPCRelPair =
      isPartOfGOTToPCRelPair(Inst, STI);

  // User of the GOT-indirect address.
  // For example, the load that will get the relocation as follows:
  // .reloc .Lpcrel1-8,R_PPC64_PCREL_OPT,.-(.Lpcrel1-8)
  //  lwa 3, 4(3)
  if (IsPartOfGOTToPCRelPair && !*IsPartOfGOTToPCRelPair)
    emitGOTToPCRelReloc(Inst);

  // Special handling is only for prefixed instructions.
  if (!Emitter.isPrefixedInstruction(Inst)) {
    MCELFStreamer::emitInstruction(Inst, STI);
    return;
  }
  emitPrefixedInstruction(Inst, STI);
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "If the Label and the Instruction are on the same line then move the". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“If the Label and the Instruction are on the same line then move the”。 子目标特性裁剪会影响这里的行为。

### Lines 108-145

```cpp
  // Producer of the GOT-indirect address.
  // For example, the prefixed load from the got that will get the label as
  // follows:
  //  pld 3, vec@got@pcrel(0), 1
  // .Lpcrel1:
  if (IsPartOfGOTToPCRelPair && *IsPartOfGOTToPCRelPair)
    emitGOTToPCRelLabel(Inst);
}

void PPCELFStreamer::emitLabel(MCSymbol *Symbol, SMLoc Loc) {
  LastLabel = Symbol;
  LastLabelLoc = Loc;
  MCELFStreamer::emitLabel(Symbol);
}

// This linker time GOT PC Relative optimization relocation will look like this:
//   pld <reg> symbol@got@pcrel
// <Label###>:
//   .reloc Label###-8,R_PPC64_PCREL_OPT,.-(Label###-8)
//   load <loadedreg>, 0(<reg>)
// The reason we place the label after the PLDpc instruction is that there
// may be an alignment nop before it since prefixed instructions must not
// cross a 64-byte boundary (please see
// PPCELFStreamer::emitPrefixedInstruction()). When referring to the
// label, we subtract the width of a prefixed instruction (8 bytes) to ensure
// we refer to the PLDpc.
void PPCELFStreamer::emitGOTToPCRelReloc(const MCInst &Inst) {
  // Get the last operand which contains the symbol.
  const MCOperand &Operand = Inst.getOperand(Inst.getNumOperands() - 1);
  assert(Operand.isExpr() && "Expecting an MCExpr.");
  // Cast the last operand to MCSymbolRefExpr to get the symbol.
  const MCExpr *Expr = Operand.getExpr();
  const MCSymbolRefExpr *SymExpr = static_cast<const MCSymbolRefExpr *>(Expr);
  assert(getSpecifier(SymExpr) == PPC::S_PCREL_OPT &&
         "Expecting a symbol of type VK_PCREL_OPT");
  MCSymbol *LabelSym =
      getContext().getOrCreateSymbol(SymExpr->getSymbol().getName());
  const MCExpr *LabelExpr = MCSymbolRefExpr::create(LabelSym, getContext());
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "Producer of the GOT-indirect address.". The logic interacts with LLVM's MC layer.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“Producer of the GOT-indirect address.”。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 146-183

```cpp
  const MCExpr *Eight = MCConstantExpr::create(8, getContext());
  // SubExpr is just Label###-8
  const MCExpr *SubExpr =
      MCBinaryExpr::createSub(LabelExpr, Eight, getContext());
  MCSymbol *CurrentLocation = getContext().createTempSymbol();
  const MCExpr *CurrentLocationExpr =
      MCSymbolRefExpr::create(CurrentLocation, getContext());
  // SubExpr2 is .-(Label###-8)
  const MCExpr *SubExpr2 =
      MCBinaryExpr::createSub(CurrentLocationExpr, SubExpr, getContext());

  MCFragment *F = LabelSym->getFragment();
  F->addFixup(
      MCFixup::create(LabelSym->getOffset() - 8, SubExpr2,
                      FirstLiteralRelocationKind + ELF::R_PPC64_PCREL_OPT));
  emitLabel(CurrentLocation, Inst.getLoc());
}

// Emit the label that immediately follows the PLDpc for a link time GOT PC Rel
// optimization.
void PPCELFStreamer::emitGOTToPCRelLabel(const MCInst &Inst) {
  // Get the last operand which contains the symbol.
  const MCOperand &Operand = Inst.getOperand(Inst.getNumOperands() - 1);
  assert(Operand.isExpr() && "Expecting an MCExpr.");
  // Cast the last operand to MCSymbolRefExpr to get the symbol.
  const MCExpr *Expr = Operand.getExpr();
  const MCSymbolRefExpr *SymExpr = static_cast<const MCSymbolRefExpr *>(Expr);
  assert(getSpecifier(SymExpr) == PPC::S_PCREL_OPT &&
         "Expecting a symbol of type VK_PCREL_OPT");
  MCSymbol *LabelSym =
      getContext().getOrCreateSymbol(SymExpr->getSymbol().getName());
  emitLabel(LabelSym, Inst.getLoc());
}

// This function checks if the parameter Inst is part of the setup for a link
// time GOT PC Relative optimization. For example in this situation:
// <MCInst PLDpc <MCOperand Reg:282> <MCOperand Expr:(glob_double@got@pcrel)>
//   <MCOperand Imm:0> <MCOperand Expr:(.Lpcrel@<<invalid>>)>>
```
- **EN**: Implements helper routine(s) `create`, `getContext`, `createSub` for this portion of the PowerPC backend MC layer support for the backend. The logic interacts with LLVM's MC layer.
- **CN**: 这里实现了 PowerPC 后端该部分该后端的 MC 层支持所需的辅助例程 `create`, `getContext`, `createSub`。 相关逻辑会与 LLVM 的 MC 层交互。

### Lines 184-221

```cpp
// <MCInst SOME_LOAD <MCOperand Reg:22> <MCOperand Imm:0> <MCOperand Reg:282>
//   <MCOperand Expr:(.Lpcrel@<<invalid>>)>>
// The above is a pair of such instructions and this function will not return
// std::nullopt for either one of them. In both cases we are looking for the
// last operand <MCOperand Expr:(.Lpcrel@<<invalid>>)> which needs to be an
// MCExpr and has the flag PPC::S_PCREL_OPT. After that we just
// look at the opcode and in the case of PLDpc we will return true. For the load
// (or store) this function will return false indicating it has found the second
// instruciton in the pair.
std::optional<bool> llvm::isPartOfGOTToPCRelPair(const MCInst &Inst,
                                                 const MCSubtargetInfo &STI) {
  // Need at least two operands.
  if (Inst.getNumOperands() < 2)
    return std::nullopt;

  unsigned LastOp = Inst.getNumOperands() - 1;
  // The last operand needs to be an MCExpr and it needs to have a variant kind
  // of VK_PCREL_OPT. If it does not satisfy these conditions it is not a
  // link time GOT PC Rel opt instruction and we can ignore it and return
  // std::nullopt.
  const MCOperand &Operand = Inst.getOperand(LastOp);
  if (!Operand.isExpr())
    return std::nullopt;

  // Check for the variant kind VK_PCREL_OPT in this expression.
  const MCExpr *Expr = Operand.getExpr();
  const MCSymbolRefExpr *SymExpr = static_cast<const MCSymbolRefExpr *>(Expr);
  if (!SymExpr || getSpecifier(SymExpr) != PPC::S_PCREL_OPT)
    return std::nullopt;

  return (Inst.getOpcode() == PPC::PLDpc);
}

MCStreamer *
llvm::createPPCELFStreamer(const Triple &T, MCContext &C,
                           std::unique_ptr<MCAsmBackend> &&MAB,
                           std::unique_ptr<MCObjectWriter> &&OW,
                           std::unique_ptr<MCCodeEmitter> &&Emitter) {
```
- **EN**: Uses comments to document intent, invariants, or design constraints before the executable definitions begin. A representative note is: "<MCInst SOME_LOAD <MCOperand Reg:22> <MCOperand Imm:0> <MCOperand Reg:282>". Subtarget feature gating influences the behavior here.
- **CN**: 这里通过注释先说明设计意图、不变量或约束条件，再进入真正的定义与实现。 其中一个有代表性的说明是：“<MCInst SOME_LOAD <MCOperand Reg:22> <MCOperand Imm:0> <MCOperand Reg:282>”。 子目标特性裁剪会影响这里的行为。

### Lines 222-224

```cpp
  return new PPCELFStreamer(C, std::move(MAB), std::move(OW),
                            std::move(Emitter));
}
```
- **EN**: Declares function entry points including `PPCELFStreamer`, `move` that other backend components call later.
- **CN**: 这里声明后续其他后端组件会调用的函数入口，包括 `PPCELFStreamer`, `move`。

## Key Concepts / 关键概念

- PowerPC backend integration / PowerPC 后端集成
- MC layer support for the backend / 该后端的 MC 层支持
- Subtarget features / 子目标特性
- Assembly or MC emission / 汇编或 MC 发射
- Instruction selection or opcode handling / 指令选择或操作码处理
- Object format integration / 目标文件格式集成
- Exception-handling support / 异常处理支持

## Dependencies / 依赖关系

### Direct Includes / 直接包含

- `PPCELFStreamer.h`
- `PPCMCAsmInfo.h`
- `PPCMCCodeEmitter.h`
- `PPCMCTargetDesc.h`
- `llvm/BinaryFormat/ELF.h`
- `llvm/MC/MCAsmBackend.h`
- `llvm/MC/MCAssembler.h`
- `llvm/MC/MCCodeEmitter.h`
- `llvm/MC/MCContext.h`
- `llvm/MC/MCInst.h`
- `llvm/MC/MCInstrDesc.h`
- `llvm/MC/MCObjectWriter.h`
- `llvm/Support/SourceMgr.h`

### Important Collaborators / 重要协作组件

- LLVM MC layer / LLVM MC 层
- LLVM Support utilities / LLVM Support 工具层
- PowerPC target-specific helpers / PowerPC 目标专用辅助组件
