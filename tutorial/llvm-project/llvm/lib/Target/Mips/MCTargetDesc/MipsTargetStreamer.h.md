# MipsTargetStreamer.h — Code Analysis / 代码分析

## Source / 来源
- File: `llvm/lib/Target/Mips/MCTargetDesc/MipsTargetStreamer.h`
- Repository: `llvm-project`
- Purpose (EN): Declares `MipsTargetStreamer` for the Mips backend and exposes interfaces for MC-layer target description and encoding support.
- 用途 (CN): 声明 Mips 后端中的 `MipsTargetStreamer`，并提供与MC 层目标描述与编码支持相关的接口。

## Line-by-Line Analysis / 逐行分析

### Lines 1-7
```cpp
//===-- MipsTargetStreamer.h - Mips Target Streamer ------------*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
- EN: Introduces the file banner, license, and high-level intent so readers know which backend component owns the implementation.
- CN: 这里给出文件横幅、许可证以及高层意图，帮助读者快速了解该实现属于哪个后端组件。

### Lines 9-10
```cpp
#ifndef LLVM_LIB_TARGET_MIPS_MIPSTARGETSTREAMER_H
#define LLVM_LIB_TARGET_MIPS_MIPSTARGETSTREAMER_H
```
- EN: Provides preprocessor structure such as header guards so declarations are included exactly once.
- CN: 这里提供头文件保护等预处理结构，确保声明只被包含一次。

### Lines 12-17
```cpp
#include "MCTargetDesc/MipsABIFlagsSection.h"
#include "MCTargetDesc/MipsABIInfo.h"
#include "llvm/ADT/STLExtras.h"
#include "llvm/MC/MCELFStreamer.h"
#include "llvm/MC/MCRegisterInfo.h"
#include "llvm/MC/MCStreamer.h"
```
- EN: Imports backend-local headers, LLVM infrastructure, and standard-library facilities required by the following target-specific logic.
- CN: 这里导入后端本地头文件、LLVM 基础设施以及标准库设施，供后续目标相关逻辑使用。

### Lines 19-19
```cpp
namespace llvm {
```
- EN: Establishes namespace context and keeps later declarations aligned with LLVM coding conventions.
- CN: 这里建立命名空间上下文，使后续声明与 LLVM 的编码约定保持一致。

### Lines 21-21
```cpp
class formatted_raw_ostream;
```
- EN: Declares `formatted_raw_ostream`, packaging target-specific state and APIs around `MipsTargetStreamer`.
- CN: 这里声明 `formatted_raw_ostream`，把与 `MipsTargetStreamer` 相关的目标特定状态和 API 组织在一起。

### Lines 23-25
```cpp
class MipsTargetStreamer : public MCTargetStreamer {
public:
  MipsTargetStreamer(MCStreamer &S);
```
- EN: Declares `MipsTargetStreamer`, packaging target-specific state and APIs around `MipsTargetStreamer`.
- CN: 这里声明 `MipsTargetStreamer`，把与 `MipsTargetStreamer` 相关的目标特定状态和 API 组织在一起。

### Lines 27-32
```cpp
  virtual void emitGPRel32Value(const MCExpr *);
  virtual void emitGPRel64Value(const MCExpr *);
  virtual void emitDTPRel32Value(const MCExpr *);
  virtual void emitDTPRel64Value(const MCExpr *);
  virtual void emitTPRel32Value(const MCExpr *);
  virtual void emitTPRel64Value(const MCExpr *);
```
- EN: Declares `emitGPRel32Value`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitGPRel32Value`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 34-34
```cpp
  virtual void setPic(bool Value) {}
```
- EN: Implements `setPic`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setPic`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 36-40
```cpp
  virtual void emitDirectiveSetMicroMips();
  virtual void emitDirectiveSetNoMicroMips();
  virtual void setUsesMicroMips();
  virtual void emitDirectiveSetMips16();
  virtual void emitDirectiveSetNoMips16();
```
- EN: Declares `emitDirectiveSetMicroMips`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetMicroMips`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 42-56
```cpp
  virtual void emitDirectiveSetReorder();
  virtual void emitDirectiveSetNoReorder();
  virtual void emitDirectiveSetMacro();
  virtual void emitDirectiveSetNoMacro();
  virtual void emitDirectiveSetMsa();
  virtual void emitDirectiveSetNoMsa();
  virtual void emitDirectiveSetMt();
  virtual void emitDirectiveSetNoMt();
  virtual void emitDirectiveSetCRC();
  virtual void emitDirectiveSetNoCRC();
  virtual void emitDirectiveSetVirt();
  virtual void emitDirectiveSetNoVirt();
  virtual void emitDirectiveSetGINV();
  virtual void emitDirectiveSetNoGINV();
  virtual void emitDirectiveSetAt();
```
- EN: Declares `emitDirectiveSetReorder`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetReorder`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 57-59
```cpp
  virtual void emitDirectiveSetAtWithArg(unsigned RegNo);
  virtual void emitDirectiveSetNoAt();
  virtual void emitDirectiveEnd(StringRef Name);
```
- EN: Declares `emitDirectiveSetAtWithArg`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetAtWithArg`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 61-71
```cpp
  virtual void emitDirectiveEnt(const MCSymbol &Symbol);
  virtual void emitDirectiveAbiCalls();
  virtual void emitDirectiveNaN2008();
  virtual void emitDirectiveNaNLegacy();
  virtual void emitDirectiveOptionPic0();
  virtual void emitDirectiveOptionPic2();
  virtual void emitDirectiveInsn();
  virtual void emitFrame(MCRegister StackReg, unsigned StackSize,
                         MCRegister ReturnReg);
  virtual void emitMask(unsigned CPUBitmask, int CPUTopSavedRegOff);
  virtual void emitFMask(unsigned FPUBitmask, int FPUTopSavedRegOff);
```
- EN: Declares `emitDirectiveEnt`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveEnt`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 73-87
```cpp
  virtual void emitDirectiveSetArch(StringRef Arch);
  virtual void emitDirectiveSetMips0();
  virtual void emitDirectiveSetMips1();
  virtual void emitDirectiveSetMips2();
  virtual void emitDirectiveSetMips3();
  virtual void emitDirectiveSetMips4();
  virtual void emitDirectiveSetMips5();
  virtual void emitDirectiveSetMips32();
  virtual void emitDirectiveSetMips32R2();
  virtual void emitDirectiveSetMips32R3();
  virtual void emitDirectiveSetMips32R5();
  virtual void emitDirectiveSetMips32R6();
  virtual void emitDirectiveSetMips64();
  virtual void emitDirectiveSetMips64R2();
  virtual void emitDirectiveSetMips64R3();
```
- EN: Declares `emitDirectiveSetArch`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetArch`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 88-98
```cpp
  virtual void emitDirectiveSetMips64R5();
  virtual void emitDirectiveSetMips64R6();
  virtual void emitDirectiveSetDsp();
  virtual void emitDirectiveSetDspr2();
  virtual void emitDirectiveSetNoDsp();
  virtual void emitDirectiveSetMips3D();
  virtual void emitDirectiveSetNoMips3D();
  virtual void emitDirectiveSetPop();
  virtual void emitDirectiveSetPush();
  virtual void emitDirectiveSetSoftFloat();
  virtual void emitDirectiveSetHardFloat();
```
- EN: Declares `emitDirectiveSetMips64R5`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetMips64R5`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 100-110
```cpp
  // PIC support
  virtual void emitDirectiveCpAdd(MCRegister Reg);
  virtual void emitDirectiveCpLoad(MCRegister Reg);
  virtual void emitDirectiveCpLocal(MCRegister Reg);
  virtual bool emitDirectiveCpRestore(int Offset,
                                      function_ref<MCRegister()> GetATReg,
                                      SMLoc IDLoc, const MCSubtargetInfo *STI);
  virtual void emitDirectiveCpsetup(MCRegister Reg, int RegOrOffset,
                                    const MCSymbol &Sym, bool IsReg);
  virtual void emitDirectiveCpreturn(unsigned SaveLocation,
                                     bool SaveLocationIsRegister);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 112-126
```cpp
  // FP abiflags directives
  virtual void emitDirectiveModuleFP();
  virtual void emitDirectiveModuleOddSPReg();
  virtual void emitDirectiveModuleSoftFloat();
  virtual void emitDirectiveModuleHardFloat();
  virtual void emitDirectiveModuleMT();
  virtual void emitDirectiveSetFp(MipsABIFlagsSection::FpABIKind Value);
  virtual void emitDirectiveSetOddSPReg();
  virtual void emitDirectiveSetNoOddSPReg();
  virtual void emitDirectiveModuleCRC();
  virtual void emitDirectiveModuleNoCRC();
  virtual void emitDirectiveModuleVirt();
  virtual void emitDirectiveModuleNoVirt();
  virtual void emitDirectiveModuleGINV();
  virtual void emitDirectiveModuleNoGINV();
```
- EN: Declares `emitDirectiveModuleFP`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveModuleFP`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 128-142
```cpp
  void emitR(unsigned Opcode, MCRegister Reg0, SMLoc IDLoc,
             const MCSubtargetInfo *STI);
  void emitII(unsigned Opcode, int16_t Imm1, int16_t Imm2, SMLoc IDLoc,
              const MCSubtargetInfo *STI);
  void emitRX(unsigned Opcode, MCRegister Reg0, MCOperand Op1, SMLoc IDLoc,
              const MCSubtargetInfo *STI);
  void emitRI(unsigned Opcode, MCRegister Reg0, int32_t Imm, SMLoc IDLoc,
              const MCSubtargetInfo *STI);
  void emitRR(unsigned Opcode, MCRegister Reg0, MCRegister Reg1, SMLoc IDLoc,
              const MCSubtargetInfo *STI);
  void emitRRX(unsigned Opcode, MCRegister Reg0, MCRegister Reg1, MCOperand Op2,
               SMLoc IDLoc, const MCSubtargetInfo *STI);
  void emitRRR(unsigned Opcode, MCRegister Reg0, MCRegister Reg1,
               MCRegister Reg2, SMLoc IDLoc, const MCSubtargetInfo *STI);
  void emitRRRX(unsigned Opcode, MCRegister Reg0, MCRegister Reg1,
```
- EN: Declares `emitR`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitR`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 143-156
```cpp
                MCRegister Reg2, MCOperand Op3, SMLoc IDLoc,
                const MCSubtargetInfo *STI);
  void emitRRI(unsigned Opcode, MCRegister Reg0, MCRegister Reg1, int16_t Imm,
               SMLoc IDLoc, const MCSubtargetInfo *STI);
  void emitRRIII(unsigned Opcode, MCRegister Reg0, MCRegister Reg1,
                 int16_t Imm0, int16_t Imm1, int16_t Imm2, SMLoc IDLoc,
                 const MCSubtargetInfo *STI);
  void emitAddu(MCRegister DstReg, MCRegister SrcReg, MCRegister TrgReg,
                bool Is64Bit, const MCSubtargetInfo *STI);
  void emitDSLL(MCRegister DstReg, MCRegister SrcReg, int16_t ShiftAmount,
                SMLoc IDLoc, const MCSubtargetInfo *STI);
  void emitEmptyDelaySlot(bool hasShortDelaySlot, SMLoc IDLoc,
                          const MCSubtargetInfo *STI);
  void emitNop(SMLoc IDLoc, const MCSubtargetInfo *STI);
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 158-172
```cpp
  /// Emit a store instruction with an offset. If the offset is out of range
  /// then it will be synthesized using the assembler temporary.
  ///
  /// GetATReg() is a callback that can be used to obtain the current assembler
  /// temporary and is only called when the assembler temporary is required. It
  /// must handle the case where no assembler temporary is available (typically
  /// by reporting an error).
  void emitStoreWithImmOffset(unsigned Opcode, MCRegister SrcReg,
                              MCRegister BaseReg, int64_t Offset,
                              function_ref<MCRegister()> GetATReg, SMLoc IDLoc,
                              const MCSubtargetInfo *STI);
  void emitLoadWithImmOffset(unsigned Opcode, MCRegister DstReg,
                             MCRegister BaseReg, int64_t Offset,
                             MCRegister TmpReg, SMLoc IDLoc,
                             const MCSubtargetInfo *STI);
```
- EN: Declares `emitStoreWithImmOffset`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitStoreWithImmOffset`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 173-173
```cpp
  void emitGPRestore(int Offset, SMLoc IDLoc, const MCSubtargetInfo *STI);
```
- EN: Declares `emitGPRestore`, a emission/printing routine centered on subtarget feature handling. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitGPRestore`，它是一个围绕子目标特性处理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 175-177
```cpp
  void forbidModuleDirective() { ModuleDirectiveAllowed = false; }
  void reallowModuleDirective() { ModuleDirectiveAllowed = true; }
  bool isModuleDirectiveAllowed() { return ModuleDirectiveAllowed; }
```
- EN: Implements `forbidModuleDirective`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `forbidModuleDirective`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 179-185
```cpp
  // This method enables template classes to set internal abi flags
  // structure values.
  template <class PredicateLibrary>
  void updateABIInfo(const PredicateLibrary &P) {
    ABI = P.getABI();
    ABIFlagsSection.setAllFromPredicates(P);
  }
```
- EN: Declares `PredicateLibrary`, packaging target-specific state and APIs around `MipsTargetStreamer`.
- CN: 这里声明 `PredicateLibrary`，把与 `MipsTargetStreamer` 相关的目标特定状态和 API 组织在一起。

### Lines 187-191
```cpp
  MipsABIFlagsSection &getABIFlagsSection() { return ABIFlagsSection; }
  const MipsABIInfo &getABI() const {
    assert(ABI && "ABI hasn't been set!");
    return *ABI;
  }
```
- EN: Implements `getABIFlagsSection`, a query/helper routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `getABIFlagsSection`，它是一个围绕目标相关状态展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 193-195
```cpp
protected:
  std::optional<MipsABIInfo> ABI;
  MipsABIFlagsSection ABIFlagsSection;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 197-199
```cpp
  bool GPRInfoSet;
  unsigned GPRBitMask;
  int GPROffset;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 201-203
```cpp
  bool FPRInfoSet;
  unsigned FPRBitMask;
  int FPROffset;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 205-209
```cpp
  bool FrameInfoSet;
  int FrameOffset;
  unsigned FrameReg;
  MCRegister GPReg;
  unsigned ReturnReg;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 211-213
```cpp
private:
  bool ModuleDirectiveAllowed;
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 215-217
```cpp
// This part is for ascii assembly output
class MipsTargetAsmStreamer : public MipsTargetStreamer {
  formatted_raw_ostream &OS;
```
- EN: Declares `MipsTargetAsmStreamer`, packaging target-specific state and APIs around `MipsTargetStreamer`.
- CN: 这里声明 `MipsTargetAsmStreamer`，把与 `MipsTargetStreamer` 相关的目标特定状态和 API 组织在一起。

### Lines 219-220
```cpp
public:
  MipsTargetAsmStreamer(MCStreamer &S, formatted_raw_ostream &OS);
```
- EN: Declares `MipsTargetAsmStreamer`, a target-specific routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `MipsTargetAsmStreamer`，它是一个围绕MC streamer 交互展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 222-227
```cpp
  void emitGPRel32Value(const MCExpr *) override;
  void emitGPRel64Value(const MCExpr *) override;
  void emitDTPRel32Value(const MCExpr *) override;
  void emitDTPRel64Value(const MCExpr *) override;
  void emitTPRel32Value(const MCExpr *) override;
  void emitTPRel64Value(const MCExpr *) override;
```
- EN: Declares `emitGPRel32Value`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitGPRel32Value`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 229-232
```cpp
  void emitDirectiveSetMicroMips() override;
  void emitDirectiveSetNoMicroMips() override;
  void emitDirectiveSetMips16() override;
  void emitDirectiveSetNoMips16() override;
```
- EN: Declares `emitDirectiveSetMicroMips`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetMicroMips`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 234-248
```cpp
  void emitDirectiveSetReorder() override;
  void emitDirectiveSetNoReorder() override;
  void emitDirectiveSetMacro() override;
  void emitDirectiveSetNoMacro() override;
  void emitDirectiveSetMsa() override;
  void emitDirectiveSetNoMsa() override;
  void emitDirectiveSetMt() override;
  void emitDirectiveSetNoMt() override;
  void emitDirectiveSetCRC() override;
  void emitDirectiveSetNoCRC() override;
  void emitDirectiveSetVirt() override;
  void emitDirectiveSetNoVirt() override;
  void emitDirectiveSetGINV() override;
  void emitDirectiveSetNoGINV() override;
  void emitDirectiveSetAt() override;
```
- EN: Declares `emitDirectiveSetReorder`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetReorder`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 249-251
```cpp
  void emitDirectiveSetAtWithArg(unsigned RegNo) override;
  void emitDirectiveSetNoAt() override;
  void emitDirectiveEnd(StringRef Name) override;
```
- EN: Declares `emitDirectiveSetAtWithArg`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetAtWithArg`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 253-263
```cpp
  void emitDirectiveEnt(const MCSymbol &Symbol) override;
  void emitDirectiveAbiCalls() override;
  void emitDirectiveNaN2008() override;
  void emitDirectiveNaNLegacy() override;
  void emitDirectiveOptionPic0() override;
  void emitDirectiveOptionPic2() override;
  void emitDirectiveInsn() override;
  void emitFrame(MCRegister StackReg, unsigned StackSize,
                 MCRegister ReturnReg) override;
  void emitMask(unsigned CPUBitmask, int CPUTopSavedRegOff) override;
  void emitFMask(unsigned FPUBitmask, int FPUTopSavedRegOff) override;
```
- EN: Declares `emitDirectiveEnt`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveEnt`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 265-279
```cpp
  void emitDirectiveSetArch(StringRef Arch) override;
  void emitDirectiveSetMips0() override;
  void emitDirectiveSetMips1() override;
  void emitDirectiveSetMips2() override;
  void emitDirectiveSetMips3() override;
  void emitDirectiveSetMips4() override;
  void emitDirectiveSetMips5() override;
  void emitDirectiveSetMips32() override;
  void emitDirectiveSetMips32R2() override;
  void emitDirectiveSetMips32R3() override;
  void emitDirectiveSetMips32R5() override;
  void emitDirectiveSetMips32R6() override;
  void emitDirectiveSetMips64() override;
  void emitDirectiveSetMips64R2() override;
  void emitDirectiveSetMips64R3() override;
```
- EN: Declares `emitDirectiveSetArch`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetArch`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 280-290
```cpp
  void emitDirectiveSetMips64R5() override;
  void emitDirectiveSetMips64R6() override;
  void emitDirectiveSetDsp() override;
  void emitDirectiveSetDspr2() override;
  void emitDirectiveSetNoDsp() override;
  void emitDirectiveSetMips3D() override;
  void emitDirectiveSetNoMips3D() override;
  void emitDirectiveSetPop() override;
  void emitDirectiveSetPush() override;
  void emitDirectiveSetSoftFloat() override;
  void emitDirectiveSetHardFloat() override;
```
- EN: Declares `emitDirectiveSetMips64R5`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetMips64R5`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 292-295
```cpp
  // PIC support
  void emitDirectiveCpAdd(MCRegister Reg) override;
  void emitDirectiveCpLoad(MCRegister Reg) override;
  void emitDirectiveCpLocal(MCRegister Reg) override;
```
- EN: Declares `emitDirectiveCpAdd`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveCpAdd`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 297-309
```cpp
  /// Emit a .cprestore directive.  If the offset is out of range then it will
  /// be synthesized using the assembler temporary.
  ///
  /// GetATReg() is a callback that can be used to obtain the current assembler
  /// temporary and is only called when the assembler temporary is required. It
  /// must handle the case where no assembler temporary is available (typically
  /// by reporting an error).
  bool emitDirectiveCpRestore(int Offset, function_ref<MCRegister()> GetATReg,
                              SMLoc IDLoc, const MCSubtargetInfo *STI) override;
  void emitDirectiveCpsetup(MCRegister Reg, int RegOrOffset,
                            const MCSymbol &Sym, bool IsReg) override;
  void emitDirectiveCpreturn(unsigned SaveLocation,
                             bool SaveLocationIsRegister) override;
```
- EN: Declares `emitDirectiveCpRestore`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveCpRestore`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 311-325
```cpp
  // FP abiflags directives
  void emitDirectiveModuleFP() override;
  void emitDirectiveModuleOddSPReg() override;
  void emitDirectiveModuleSoftFloat() override;
  void emitDirectiveModuleHardFloat() override;
  void emitDirectiveModuleMT() override;
  void emitDirectiveModuleCRC() override;
  void emitDirectiveModuleNoCRC() override;
  void emitDirectiveModuleVirt() override;
  void emitDirectiveModuleNoVirt() override;
  void emitDirectiveModuleGINV() override;
  void emitDirectiveModuleNoGINV() override;
  void emitDirectiveSetFp(MipsABIFlagsSection::FpABIKind Value) override;
  void emitDirectiveSetOddSPReg() override;
  void emitDirectiveSetNoOddSPReg() override;
```
- EN: Declares `emitDirectiveModuleFP`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveModuleFP`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 326-326
```cpp
};
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 328-332
```cpp
// This part is for ELF object output
class MipsTargetELFStreamer : public MipsTargetStreamer {
  bool MicroMipsEnabled;
  const MCSubtargetInfo &STI;
  bool Pic;
```
- EN: Declares `MipsTargetELFStreamer`, packaging target-specific state and APIs around `MipsTargetStreamer`.
- CN: 这里声明 `MipsTargetELFStreamer`，把与 `MipsTargetStreamer` 相关的目标特定状态和 API 组织在一起。

### Lines 334-337
```cpp
public:
  bool isMicroMipsEnabled() const { return MicroMipsEnabled; }
  MCELFStreamer &getStreamer();
  MipsTargetELFStreamer(MCStreamer &S, const MCSubtargetInfo &STI);
```
- EN: Implements `isMicroMipsEnabled`, a query/helper routine centered on MC streamer interaction. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `isMicroMipsEnabled`，它是一个围绕MC streamer 交互展开的查询/辅助例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 339-339
```cpp
  void setPic(bool Value) override { Pic = Value; }
```
- EN: Implements `setPic`, a target-specific routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里实现 `setPic`，它是一个围绕目标相关状态展开的目标相关例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 341-343
```cpp
  void emitLabel(MCSymbol *Symbol) override;
  void emitAssignment(MCSymbol *Symbol, const MCExpr *Value) override;
  void finish() override;
```
- EN: Declares `emitLabel`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitLabel`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 345-350
```cpp
  void emitGPRel32Value(const MCExpr *) override;
  void emitGPRel64Value(const MCExpr *) override;
  void emitDTPRel32Value(const MCExpr *) override;
  void emitDTPRel64Value(const MCExpr *) override;
  void emitTPRel32Value(const MCExpr *) override;
  void emitTPRel64Value(const MCExpr *) override;
```
- EN: Declares `emitGPRel32Value`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitGPRel32Value`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 352-355
```cpp
  void emitDirectiveSetMicroMips() override;
  void emitDirectiveSetNoMicroMips() override;
  void setUsesMicroMips() override;
  void emitDirectiveSetMips16() override;
```
- EN: Declares `emitDirectiveSetMicroMips`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetMicroMips`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 357-358
```cpp
  void emitDirectiveSetNoReorder() override;
  void emitDirectiveEnd(StringRef Name) override;
```
- EN: Declares `emitDirectiveSetNoReorder`, a emission/printing routine centered on target-specific state. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveSetNoReorder`，它是一个围绕目标相关状态展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 360-370
```cpp
  void emitDirectiveEnt(const MCSymbol &Symbol) override;
  void emitDirectiveAbiCalls() override;
  void emitDirectiveNaN2008() override;
  void emitDirectiveNaNLegacy() override;
  void emitDirectiveOptionPic0() override;
  void emitDirectiveOptionPic2() override;
  void emitDirectiveInsn() override;
  void emitFrame(MCRegister StackReg, unsigned StackSize,
                 MCRegister ReturnReg) override;
  void emitMask(unsigned CPUBitmask, int CPUTopSavedRegOff) override;
  void emitFMask(unsigned FPUBitmask, int FPUTopSavedRegOff) override;
```
- EN: Declares `emitDirectiveEnt`, a emission/printing routine centered on register management. The block wires backend rules into shared LLVM abstractions and keeps the component behavior target-aware.
- CN: 这里声明 `emitDirectiveEnt`，它是一个围绕寄存器管理展开的发射/打印例程。该代码块把后端规则接入 LLVM 的通用抽象，同时保持组件行为具备目标感知能力。

### Lines 372-381
```cpp
  // PIC support
  void emitDirectiveCpAdd(MCRegister Reg) override;
  void emitDirectiveCpLoad(MCRegister Reg) override;
  void emitDirectiveCpLocal(MCRegister Reg) override;
  bool emitDirectiveCpRestore(int Offset, function_ref<MCRegister()> GetATReg,
                              SMLoc IDLoc, const MCSubtargetInfo *STI) override;
  void emitDirectiveCpsetup(MCRegister Reg, int RegOrOffset,
                            const MCSymbol &Sym, bool IsReg) override;
  void emitDirectiveCpreturn(unsigned SaveLocation,
                             bool SaveLocationIsRegister) override;
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

### Lines 383-386
```cpp
  void emitMipsAbiFlags();
};
}
#endif
```
- EN: Provides supporting target-specific declarations or local logic that complement the surrounding component implementation.
- CN: 这里提供补充性的目标相关声明或局部逻辑，用于配合周围组件实现。

## Key Concepts / 关键概念

- EN: Primary role: MC-layer target description and encoding support.
  - CN: 核心职责：MC 层目标描述与编码支持。
- EN: The MC layer is used to print, encode, or stream target instructions and symbols.
  - CN: MC 层被用于打印、编码或流式输出目标指令与符号。
- EN: TargetMachine/Subtarget data steers CPU- or ABI-specific behavior.
  - CN: TargetMachine/Subtarget 数据决定了与 CPU 或 ABI 相关的行为。

## Dependencies / 依赖关系

- EN: Backend-local headers: `MCTargetDesc/MipsABIFlagsSection.h`, `MCTargetDesc/MipsABIInfo.h`.
  - CN: 后端本地头文件：`MCTargetDesc/MipsABIFlagsSection.h`, `MCTargetDesc/MipsABIInfo.h`。
- EN: LLVM infrastructure headers: `llvm/ADT/STLExtras.h`, `llvm/MC/MCELFStreamer.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCStreamer.h`.
  - CN: LLVM 基础设施头文件：`llvm/ADT/STLExtras.h`, `llvm/MC/MCELFStreamer.h`, `llvm/MC/MCRegisterInfo.h`, `llvm/MC/MCStreamer.h`。
- EN: Runtime behavior is parameterized by subtarget and target-machine configuration objects.
  - CN: 运行时行为会受到 subtarget 与 target-machine 配置对象的参数化影响。
- EN: The implementation also depends on LLVM MC layer classes for final assembly/object emission.
  - CN: 实现还依赖 LLVM 的 MC 层类来完成最终汇编/目标文件输出。
