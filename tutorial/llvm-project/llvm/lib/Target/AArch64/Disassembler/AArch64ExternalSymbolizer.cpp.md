# AArch64ExternalSymbolizer.cpp — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/Disassembler/AArch64ExternalSymbolizer.cpp`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers Symbolizer for AArch64. / 该文件实现 AArch64 后端中的反汇编。
## Line-by-Line Analysis / 逐行分析
### Lines 1-20: Documented code section
```cpp
//===- AArch64ExternalSymbolizer.cpp - Symbolizer for AArch64 ---*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#include "AArch64ExternalSymbolizer.h"
#include "MCTargetDesc/AArch64MCAsmInfo.h"
#include "llvm/MC/MCContext.h"
#include "llvm/MC/MCExpr.h"
#include "llvm/MC/MCInst.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/Support/Format.h"
#include "llvm/Support/raw_ostream.h"

using namespace llvm;

#define DEBUG_TYPE "aarch64-disassembler"
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 21-42: Function getMachOSpecifier
```cpp

static AArch64::Specifier
getMachOSpecifier(uint64_t LLVMDisassembler_VariantKind) {
  switch (LLVMDisassembler_VariantKind) {
  case LLVMDisassembler_VariantKind_None:
    return AArch64::S_None;
  case LLVMDisassembler_VariantKind_ARM64_PAGE:
    return AArch64::S_MACHO_PAGE;
  case LLVMDisassembler_VariantKind_ARM64_PAGEOFF:
    return AArch64::S_MACHO_PAGEOFF;
  case LLVMDisassembler_VariantKind_ARM64_GOTPAGE:
    return AArch64::S_MACHO_GOTPAGE;
  case LLVMDisassembler_VariantKind_ARM64_GOTPAGEOFF:
    return AArch64::S_MACHO_GOTPAGEOFF;
  case LLVMDisassembler_VariantKind_ARM64_TLVP:
    return AArch64::S_MACHO_TLVPPAGE;
  case LLVMDisassembler_VariantKind_ARM64_TLVOFF:
    return AArch64::S_MACHO_TLVPPAGEOFF;
  default:
    llvm_unreachable("bad LLVMDisassembler_VariantKind");
  }
}
```
**EN:** This block implements getMachOSpecifier, advancing the file's disassembly flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 getMachOSpecifier，通过 AArch64 专用的决策与数据处理推进本文件的反汇编流程。
### Lines 43-70: Documented code section
```cpp

/// tryAddingSymbolicOperand - tryAddingSymbolicOperand tries to add a symbolic
/// operand in place of the immediate Value in the MCInst.  The immediate
/// Value has not had any PC adjustment made by the caller. If the instruction
/// is a branch that adds the PC to the immediate Value then isBranch is
/// Success, else Fail. If GetOpInfo is non-null, then it is called to get any
/// symbolic information at the Address for this instruction.  If that returns
/// non-zero then the symbolic information it returns is used to create an
/// MCExpr and that is added as an operand to the MCInst.  If GetOpInfo()
/// returns zero and isBranch is Success then a symbol look up for
/// Address + Value is done and if a symbol is found an MCExpr is created with
/// that, else an MCExpr with Address + Value is created.  If GetOpInfo()
/// returns zero and isBranch is Fail then the Opcode of the MCInst is
/// tested and for ADRP an other instructions that help to load of pointers
/// a symbol look up is done to see it is returns a specific reference type
/// to add to the comment stream.  This function returns Success if it adds
/// an operand to the MCInst and Fail otherwise.
bool AArch64ExternalSymbolizer::tryAddingSymbolicOperand(
    MCInst &MI, raw_ostream &CommentStream, int64_t Value, uint64_t Address,
    bool IsBranch, uint64_t Offset, uint64_t OpSize, uint64_t InstSize) {
  if (!SymbolLookUp)
    return false;
  // FIXME: This method shares a lot of code with
  //        MCExternalSymbolizer::tryAddingSymbolicOperand. It may be possible
  //        refactor the MCExternalSymbolizer interface to allow more of this
  //        implementation to be shared.
  //
  struct LLVMOpInfo1 SymbolicOp;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 71-98: Struct LLVMOpInfo1
```cpp
  memset(&SymbolicOp, '\0', sizeof(struct LLVMOpInfo1));
  SymbolicOp.Value = Value;
  uint64_t ReferenceType;
  const char *ReferenceName;
  if (!GetOpInfo || !GetOpInfo(DisInfo, Address, /*Offset=*/0, OpSize, InstSize,
                               1, &SymbolicOp)) {
    if (IsBranch) {
      ReferenceType = LLVMDisassembler_ReferenceType_In_Branch;
      const char *Name = SymbolLookUp(DisInfo, Address + Value, &ReferenceType,
                                      Address, &ReferenceName);
      if (Name) {
        SymbolicOp.AddSymbol.Name = Name;
        SymbolicOp.AddSymbol.Present = true;
        SymbolicOp.Value = 0;
      } else {
        SymbolicOp.Value = Address + Value;
      }
      if (ReferenceType == LLVMDisassembler_ReferenceType_Out_SymbolStub)
        CommentStream << "symbol stub for: " << ReferenceName;
      else if (ReferenceType ==
               LLVMDisassembler_ReferenceType_Out_Objc_Message)
        CommentStream << "Objc message: " << ReferenceName;
    } else if (MI.getOpcode() == AArch64::ADRP) {
        ReferenceType = LLVMDisassembler_ReferenceType_In_ARM64_ADRP;
        // otool expects the fully encoded ADRP instruction to be passed in as
        // the value here, so reconstruct it:
        const MCRegisterInfo &MCRI = *Ctx.getRegisterInfo();
        uint32_t EncodedInst = 0x90000000;
```
**EN:** This block defines LLVMOpInfo1, packaging state and behavior that the file reuses for disassembly.  
**CN:** 该代码块定义 LLVMOpInfo1，把 反汇编 所需的状态与行为封装在一起供后续复用。
### Lines 99-126: Core AArch64 backend logic
```cpp
        EncodedInst |= (Value & 0x3) << 29; // immlo
        EncodedInst |= ((Value >> 2) & 0x7FFFF) << 5; // immhi
        EncodedInst |= MCRI.getEncodingValue(MI.getOperand(0).getReg()); // reg
        SymbolLookUp(DisInfo, EncodedInst, &ReferenceType, Address,
                     &ReferenceName);
        CommentStream << format("0x%llx", (0xfffffffffffff000LL & Address) +
                                              Value * 0x1000);
    } else if (MI.getOpcode() == AArch64::ADDXri ||
               MI.getOpcode() == AArch64::ADDWri ||
               MI.getOpcode() == AArch64::LDRXui ||
               MI.getOpcode() == AArch64::LDRWui ||
               MI.getOpcode() == AArch64::LDRXl ||
               MI.getOpcode() == AArch64::LDRWl ||
               MI.getOpcode() == AArch64::ADR) {
      if (MI.getOpcode() == AArch64::ADDXri ||
          MI.getOpcode() == AArch64::ADDWri)
        ReferenceType = LLVMDisassembler_ReferenceType_In_ARM64_ADDXri;
      else if (MI.getOpcode() == AArch64::LDRXui ||
               MI.getOpcode() == AArch64::LDRWui)
        ReferenceType = LLVMDisassembler_ReferenceType_In_ARM64_LDRXui;
      if (MI.getOpcode() == AArch64::LDRXl ||
          MI.getOpcode() == AArch64::LDRWl) {
        ReferenceType = LLVMDisassembler_ReferenceType_In_ARM64_LDRXl;
        SymbolLookUp(DisInfo, Address + Value, &ReferenceType, Address,
                     &ReferenceName);
      } else if (MI.getOpcode() == AArch64::ADR) {
        ReferenceType = LLVMDisassembler_ReferenceType_In_ARM64_ADR;
        SymbolLookUp(DisInfo, Address + Value, &ReferenceType, Address,
```
**EN:** This block continues the file's main disassembly logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的反汇编主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 127-150: Core AArch64 backend logic
```cpp
                            &ReferenceName);
      } else {
        const MCRegisterInfo &MCRI = *Ctx.getRegisterInfo();
        // otool expects the fully encoded ADD/LDR instruction to be passed in
        // as the value here, so reconstruct it:
        unsigned EncodedInst;
        switch (MI.getOpcode()) {
        case AArch64::ADDXri:
          EncodedInst = 0x91000000;
          break;
        case AArch64::ADDWri:
          EncodedInst = 0x11000000;
          break;
        case AArch64::LDRXui:
          EncodedInst = 0xF9400000;
          break;
        default: // LDRWui
          EncodedInst = 0xB9400000;
          break;
        }
        EncodedInst |= Value << 10; // imm12 (ADD: imm+shift, LDR: offset)
        EncodedInst |=
          MCRI.getEncodingValue(MI.getOperand(1).getReg()) << 5; // Rn
        EncodedInst |= MCRI.getEncodingValue(MI.getOperand(0).getReg()); // Rd
```
**EN:** This block continues the file's main disassembly logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的反汇编主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 151-178: Core AArch64 backend logic
```cpp

        SymbolLookUp(DisInfo, EncodedInst, &ReferenceType, Address,
                     &ReferenceName);
      }
      if (ReferenceType == LLVMDisassembler_ReferenceType_Out_LitPool_SymAddr)
        CommentStream << "literal pool symbol address: " << ReferenceName;
      else if (ReferenceType ==
               LLVMDisassembler_ReferenceType_Out_LitPool_CstrAddr) {
        CommentStream << "literal pool for: \"";
        CommentStream.write_escaped(ReferenceName);
        CommentStream << "\"";
      } else if (ReferenceType ==
               LLVMDisassembler_ReferenceType_Out_Objc_CFString_Ref)
        CommentStream << "Objc cfstring ref: @\"" << ReferenceName << "\"";
      else if (ReferenceType ==
               LLVMDisassembler_ReferenceType_Out_Objc_Message)
        CommentStream << "Objc message: " << ReferenceName;
      else if (ReferenceType ==
               LLVMDisassembler_ReferenceType_Out_Objc_Message_Ref)
        CommentStream << "Objc message ref: " << ReferenceName;
      else if (ReferenceType ==
               LLVMDisassembler_ReferenceType_Out_Objc_Selector_Ref)
        CommentStream << "Objc selector ref: " << ReferenceName;
      else if (ReferenceType ==
               LLVMDisassembler_ReferenceType_Out_Objc_Class_Ref)
        CommentStream << "Objc class ref: " << ReferenceName;
      // For these instructions, the SymbolLookUp() above is just to get the
      // ReferenceType and ReferenceName.  We want to make sure not to
```
**EN:** This block continues the file's main disassembly logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的反汇编主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 179-200: Documented code section
```cpp
      // fall through so we don't build an MCExpr to leave the disassembly
      // of the immediate values of these instructions to the InstPrinter.
      return false;
    } else {
      return false;
    }
  }

  const MCExpr *Add = nullptr;
  if (SymbolicOp.AddSymbol.Present) {
    if (SymbolicOp.AddSymbol.Name) {
      StringRef Name(SymbolicOp.AddSymbol.Name);
      MCSymbol *Sym = Ctx.getOrCreateSymbol(Name);
      auto Spec = getMachOSpecifier(SymbolicOp.VariantKind);
      if (Spec != AArch64::S_None)
        Add = MCSymbolRefExpr::create(Sym, Spec, Ctx);
      else
        Add = MCSymbolRefExpr::create(Sym, Ctx);
    } else {
      Add = MCConstantExpr::create(SymbolicOp.AddSymbol.Value, Ctx);
    }
  }
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 201-215: Core AArch64 backend logic
```cpp

  const MCExpr *Sub = nullptr;
  if (SymbolicOp.SubtractSymbol.Present) {
    if (SymbolicOp.SubtractSymbol.Name) {
      StringRef Name(SymbolicOp.SubtractSymbol.Name);
      MCSymbol *Sym = Ctx.getOrCreateSymbol(Name);
      Sub = MCSymbolRefExpr::create(Sym, Ctx);
    } else {
      Sub = MCConstantExpr::create(SymbolicOp.SubtractSymbol.Value, Ctx);
    }
  }

  const MCExpr *Off = nullptr;
  if (SymbolicOp.Value != 0)
    Off = MCConstantExpr::create(SymbolicOp.Value, Ctx);
```
**EN:** This block continues the file's main disassembly logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的反汇编主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 216-243: Core AArch64 backend logic
```cpp

  const MCExpr *Expr;
  if (Sub) {
    const MCExpr *LHS;
    if (Add)
      LHS = MCBinaryExpr::createSub(Add, Sub, Ctx);
    else
      LHS = MCUnaryExpr::createMinus(Sub, Ctx);
    if (Off)
      Expr = MCBinaryExpr::createAdd(LHS, Off, Ctx);
    else
      Expr = LHS;
  } else if (Add) {
    if (Off)
      Expr = MCBinaryExpr::createAdd(Add, Off, Ctx);
    else
      Expr = Add;
  } else {
    if (Off)
      Expr = Off;
    else
      Expr = MCConstantExpr::create(0, Ctx);
  }

  MI.addOperand(MCOperand::createExpr(Expr));

  return true;
}
```
**EN:** This block continues the file's main disassembly logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的反汇编主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
- **EN:** Decoding binary instructions back to MC form **CN:** 将二进制指令解码回 MC 形式
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64ExternalSymbolizer.h, MCTargetDesc/AArch64MCAsmInfo.h **CN:** 目标本地依赖：AArch64ExternalSymbolizer.h, MCTargetDesc/AArch64MCAsmInfo.h
- **EN:** Core LLVM interfaces: llvm/MC/MCContext.h, llvm/MC/MCExpr.h, llvm/MC/MCInst.h, llvm/MC/MCRegisterInfo.h, llvm/Support/Format.h, llvm/Support/raw_ostream.h **CN:** 核心 LLVM 接口：llvm/MC/MCContext.h, llvm/MC/MCExpr.h, llvm/MC/MCInst.h, llvm/MC/MCRegisterInfo.h, llvm/Support/Format.h, llvm/Support/raw_ostream.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for disassembly. **CN:** 与周边负责反汇编的 AArch64 后端组件紧密协作。
