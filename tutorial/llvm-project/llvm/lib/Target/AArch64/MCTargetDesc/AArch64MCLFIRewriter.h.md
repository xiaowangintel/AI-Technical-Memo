# AArch64MCLFIRewriter.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/MCTargetDesc/AArch64MCLFIRewriter.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file declares the AArch64MCLFIRewriter class, the AArch64 specific subclass of MCLFIRewriter. / 该文件实现 AArch64 后端中的AArch64 后端逻辑。
## Line-by-Line Analysis / 逐行分析
### Lines 1-14: Documented code section
```cpp
//===- AArch64MCLFIRewriter.h -----------------------------------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
//
// This file declares the AArch64MCLFIRewriter class, the AArch64 specific
// subclass of MCLFIRewriter.
//
//===----------------------------------------------------------------------===//
#ifndef LLVM_LIB_TARGET_AARCH64_MCTARGETDESC_AARCH64MCLFIREWRITER_H
#define LLVM_LIB_TARGET_AARCH64_MCTARGETDESC_AARCH64MCLFIREWRITER_H
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 15-25: Includes and compile-time dependencies
```cpp

#include "llvm/MC/MCInstrInfo.h"
#include "llvm/MC/MCLFIRewriter.h"
#include "llvm/MC/MCRegister.h"
#include "llvm/MC/MCRegisterInfo.h"

namespace llvm {
class MCContext;
class MCInst;
class MCStreamer;
class MCSubtargetInfo;
```
**EN:** This block pulls in the target-specific headers and LLVM infrastructure needed by the rest of the file.  
**CN:** 该代码块引入后续实现所需的目标专用头文件与 LLVM 基础设施。
### Lines 26-43: Class implements
```cpp

/// Rewrites AArch64 instructions for LFI sandboxing.
///
/// This class implements the LFI (Lightweight Fault Isolation) rewriting
/// for AArch64 instructions. It transforms instructions to ensure memory
/// accesses and control flow are confined within the sandbox region.
///
/// Reserved registers:
/// - X27: Sandbox base address (always holds the base)
/// - X28: Safe address register (always within sandbox)
/// - X26: Scratch register for intermediate calculations
/// - X25: context register (points to thread-local runtime data)
/// - SP:  Stack pointer (always within sandbox)
/// - X30: Link register (always within sandbox)
class AArch64MCLFIRewriter : public MCLFIRewriter {
public:
  AArch64MCLFIRewriter(MCContext &Ctx, std::unique_ptr<MCRegisterInfo> &&RI,
                       std::unique_ptr<MCInstrInfo> &&II)
```
**EN:** This block defines implements, packaging state and behavior that the file reuses for AArch64 backend logic.  
**CN:** 该代码块定义 implements，把 AArch64 后端逻辑 所需的状态与行为封装在一起供后续复用。
### Lines 44-55: Function MCLFIRewriter
```cpp
      : MCLFIRewriter(Ctx, std::move(RI), std::move(II)) {}

  bool rewriteInst(const MCInst &Inst, MCStreamer &Out,
                   const MCSubtargetInfo &STI) override;

private:
  /// Recursion guard to prevent infinite loops when emitting instructions.
  bool Guard = false;

  // Instruction classification. Returns the reserved register that may be
  // modified, or an invalid register if no reserved register is touched.
  MCRegister mayModifyReserved(const MCInst &Inst) const;
```
**EN:** This block implements MCLFIRewriter, advancing the file's AArch64 backend logic flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 MCLFIRewriter，通过 AArch64 专用的决策与数据处理推进本文件的AArch64 后端逻辑流程。
### Lines 56-69: Documented code section
```cpp

  // Instruction emission.
  void emitInst(const MCInst &Inst, MCStreamer &Out,
                const MCSubtargetInfo &STI);
  void emitAddMask(MCRegister Dest, MCRegister Src, MCStreamer &Out,
                   const MCSubtargetInfo &STI);
  void emitBranch(unsigned Opcode, MCRegister Target, MCStreamer &Out,
                  const MCSubtargetInfo &STI);
  void emitMov(MCRegister Dest, MCRegister Src, MCStreamer &Out,
               const MCSubtargetInfo &STI);

  // Rewriting logic.
  void doRewriteInst(const MCInst &Inst, MCStreamer &Out,
                     const MCSubtargetInfo &STI);
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 70-79: Documented code section
```cpp

  // Control flow.
  void rewriteIndirectBranch(const MCInst &Inst, MCStreamer &Out,
                             const MCSubtargetInfo &STI);
  void rewriteReturn(const MCInst &Inst, MCStreamer &Out,
                     const MCSubtargetInfo &STI);

  // Link register modification.
  void rewriteLRModification(const MCInst &Inst, MCStreamer &Out,
                             const MCSubtargetInfo &STI);
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 80-92: Documented code section
```cpp

  // System instructions.
  void rewriteSyscall(const MCInst &Inst, MCStreamer &Out,
                      const MCSubtargetInfo &STI);
  void rewriteTPRead(const MCInst &Inst, MCStreamer &Out,
                     const MCSubtargetInfo &STI);
  void rewriteTPWrite(const MCInst &Inst, MCStreamer &Out,
                      const MCSubtargetInfo &STI);
};

} // namespace llvm

#endif // LLVM_LIB_TARGET_AARCH64_MCTARGETDESC_AARCH64MCLFIREWRITER_H
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Core LLVM interfaces: llvm/MC/MCInstrInfo.h, llvm/MC/MCLFIRewriter.h, llvm/MC/MCRegister.h, llvm/MC/MCRegisterInfo.h **CN:** 核心 LLVM 接口：llvm/MC/MCInstrInfo.h, llvm/MC/MCLFIRewriter.h, llvm/MC/MCRegister.h, llvm/MC/MCRegisterInfo.h
- **EN:** Closely connected with neighboring AArch64 backend components responsible for AArch64 backend logic. **CN:** 与周边负责AArch64 后端逻辑的 AArch64 后端组件紧密协作。
