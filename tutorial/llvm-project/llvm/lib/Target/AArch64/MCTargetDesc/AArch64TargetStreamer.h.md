# AArch64TargetStreamer.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/AArch64/MCTargetDesc/AArch64TargetStreamer.h`
- **Repository**: llvm/llvm-project
- **Purpose**: This file covers AArch64 Target Streamer. / 该文件实现 AArch64 后端中的MC 流与目标文件输出。
## Line-by-Line Analysis / 逐行分析
### Lines 1-24: Documented code section
```cpp
//===-- AArch64TargetStreamer.h - AArch64 Target Streamer ------*- C++ -*--===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//

#ifndef LLVM_LIB_TARGET_AARCH64_MCTARGETDESC_AARCH64TARGETSTREAMER_H
#define LLVM_LIB_TARGET_AARCH64_MCTARGETDESC_AARCH64TARGETSTREAMER_H

#include "AArch64MCAsmInfo.h"
#include "llvm/ADT/StringRef.h"
#include "llvm/IR/Instructions.h"
#include "llvm/MC/MCELFStreamer.h"
#include "llvm/MC/MCStreamer.h"
#include "llvm/Support/AArch64BuildAttributes.h"
#include <cstdint>

namespace {
class AArch64ELFStreamer;
}

namespace llvm {
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 25-50: Class AArch64TargetStreamer
```cpp

class AArch64TargetStreamer : public MCTargetStreamer {
public:
  AArch64TargetStreamer(MCStreamer &S);
  ~AArch64TargetStreamer() override;

  void finish() override;
  void emitConstantPools() override;

  /// Callback used to implement the ldr= pseudo.
  /// Add a new entry to the constant pool for the current section and return an
  /// MCExpr that can be used to refer to the constant pool location.
  const MCExpr *addConstantPoolEntry(const MCExpr *, unsigned Size, SMLoc Loc);

  /// Callback used to implement the .ltorg directive.
  /// Emit contents of constant pool for the current section.
  void emitCurrentConstantPool();

  /// Callback used to implement the .note.gnu.property section.
  void emitNoteSection(unsigned Flags, uint64_t PAuthABIPlatform = -1,
                       uint64_t PAuthABIVersion = -1);

  /// Callback used to emit AUTH expressions (e.g. signed
  /// personality function pointer).
  void emitAuthValue(const MCExpr *Expr, uint16_t Discriminator,
                     AArch64PACKey::ID Key, bool HasAddressDiversity);
```
**EN:** This block defines AArch64TargetStreamer, packaging state and behavior that the file reuses for MC streaming and object emission.  
**CN:** 该代码块定义 AArch64TargetStreamer，把 MC 流与目标文件输出 所需的状态与行为封装在一起供后续复用。
### Lines 51-78: Function emitDirectiveVariantPCS
```cpp

  /// Callback used to implement the .inst directive.
  virtual void emitInst(uint32_t Inst);

  /// Callback used to implement the .variant_pcs directive.
  virtual void emitDirectiveVariantPCS(MCSymbol *Symbol) {};

  virtual void emitDirectiveArch(StringRef Name) {};
  virtual void emitDirectiveArchExtension(StringRef Name) {};

  virtual void emitARM64WinCFIAllocStack(unsigned Size) {}
  virtual void emitARM64WinCFISaveR19R20X(int Offset) {}
  virtual void emitARM64WinCFISaveFPLR(int Offset) {}
  virtual void emitARM64WinCFISaveFPLRX(int Offset) {}
  virtual void emitARM64WinCFISaveReg(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveRegX(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveRegP(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveRegPX(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveLRPair(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveFReg(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveFRegX(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveFRegP(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveFRegPX(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISetFP() {}
  virtual void emitARM64WinCFIAddFP(unsigned Size) {}
  virtual void emitARM64WinCFINop() {}
  virtual void emitARM64WinCFISaveNext() {}
  virtual void emitARM64WinCFIPrologEnd() {}
```
**EN:** This block implements emitDirectiveVariantPCS, advancing the file's MC streaming and object emission flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 emitDirectiveVariantPCS，通过 AArch64 专用的决策与数据处理推进本文件的MC 流与目标文件输出流程。
### Lines 79-101: Function emitARM64WinCFIEpilogStart
```cpp
  virtual void emitARM64WinCFIEpilogStart() {}
  virtual void emitARM64WinCFIEpilogEnd() {}
  virtual void emitARM64WinCFITrapFrame() {}
  virtual void emitARM64WinCFIMachineFrame() {}
  virtual void emitARM64WinCFIContext() {}
  virtual void emitARM64WinCFIECContext() {}
  virtual void emitARM64WinCFIClearUnwoundToCall() {}
  virtual void emitARM64WinCFIPACSignLR() {}
  virtual void emitARM64WinCFISaveAnyRegI(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveAnyRegIP(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveAnyRegD(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveAnyRegDP(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveAnyRegQ(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveAnyRegQP(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveAnyRegIX(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveAnyRegIPX(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveAnyRegDX(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveAnyRegDPX(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveAnyRegQX(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISaveAnyRegQPX(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFIAllocZ(int Offset) {}
  virtual void emitARM64WinCFISaveZReg(unsigned Reg, int Offset) {}
  virtual void emitARM64WinCFISavePReg(unsigned Reg, int Offset) {}
```
**EN:** This block implements emitARM64WinCFIEpilogStart, advancing the file's MC streaming and object emission flow with AArch64-specific decisions and data movement.  
**CN:** 该代码块实现 emitARM64WinCFIEpilogStart，通过 AArch64 专用的决策与数据处理推进本文件的MC 流与目标文件输出流程。
### Lines 102-127: Documented code section
```cpp

  /// Build attributes implementation
  virtual void
  emitAttributesSubsection(StringRef VendorName,
                          AArch64BuildAttributes::SubsectionOptional IsOptional,
                          AArch64BuildAttributes::SubsectionType ParameterType);
  virtual void emitAttribute(StringRef VendorName, unsigned Tag, unsigned Value,
                             std::string String);
  void activateAttributesSubsection(StringRef VendorName);
  std::unique_ptr<MCELFStreamer::AttributeSubSection>
  getActiveAttributesSubsection();
  std::unique_ptr<MCELFStreamer::AttributeSubSection>
  getAttributesSubsectionByName(StringRef Name);
  void
  insertAttributeInPlace(const MCELFStreamer::AttributeItem &Attr,
                         MCELFStreamer::AttributeSubSection &AttSubSection);

  SmallVector<MCELFStreamer::AttributeSubSection, 64> AttributeSubSections;

private:
  std::unique_ptr<AssemblerConstantPools> ConstantPools;
};

class AArch64TargetELFStreamer : public AArch64TargetStreamer {
private:
  AArch64ELFStreamer &getStreamer();
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 128-149: Core AArch64 backend logic
```cpp

  MCSection *AttributeSection = nullptr;

  /// Build attributes implementation
  void emitAttributesSubsection(
      StringRef VendorName,
      AArch64BuildAttributes::SubsectionOptional IsOptional,
      AArch64BuildAttributes::SubsectionType ParameterType) override;
  void emitAttribute(StringRef VendorName, unsigned Tag, unsigned Value,
                     std::string String) override;
  void emitInst(uint32_t Inst) override;
  void emitDirectiveVariantPCS(MCSymbol *Symbol) override;
  void finish() override;

public:
  AArch64TargetELFStreamer(MCStreamer &S) : AArch64TargetStreamer(S) {}
};

class AArch64TargetWinCOFFStreamer : public llvm::AArch64TargetStreamer {
public:
  AArch64TargetWinCOFFStreamer(llvm::MCStreamer &S)
    : AArch64TargetStreamer(S) {}
```
**EN:** This block continues the file's main MC streaming and object emission logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的MC 流与目标文件输出主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 150-177: Documented code section
```cpp

  // The unwind codes on ARM64 Windows are documented at
  // https://docs.microsoft.com/en-us/cpp/build/arm64-exception-handling
  void emitARM64WinCFIAllocStack(unsigned Size) override;
  void emitARM64WinCFISaveR19R20X(int Offset) override;
  void emitARM64WinCFISaveFPLR(int Offset) override;
  void emitARM64WinCFISaveFPLRX(int Offset) override;
  void emitARM64WinCFISaveReg(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveRegX(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveRegP(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveRegPX(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveLRPair(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveFReg(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveFRegX(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveFRegP(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveFRegPX(unsigned Reg, int Offset) override;
  void emitARM64WinCFISetFP() override;
  void emitARM64WinCFIAddFP(unsigned Size) override;
  void emitARM64WinCFINop() override;
  void emitARM64WinCFISaveNext() override;
  void emitARM64WinCFIPrologEnd() override;
  void emitARM64WinCFIEpilogStart() override;
  void emitARM64WinCFIEpilogEnd() override;
  void emitARM64WinCFITrapFrame() override;
  void emitARM64WinCFIMachineFrame() override;
  void emitARM64WinCFIContext() override;
  void emitARM64WinCFIECContext() override;
  void emitARM64WinCFIClearUnwoundToCall() override;
```
**EN:** This comment block marks a new section and frames how the following implementation should be read.  
**CN:** 该注释块标记了新的代码章节，并为理解后续实现提供上下文。
### Lines 178-204: Core AArch64 backend logic
```cpp
  void emitARM64WinCFIPACSignLR() override;
  void emitARM64WinCFISaveAnyRegI(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveAnyRegIP(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveAnyRegD(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveAnyRegDP(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveAnyRegQ(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveAnyRegQP(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveAnyRegIX(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveAnyRegIPX(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveAnyRegDX(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveAnyRegDPX(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveAnyRegQX(unsigned Reg, int Offset) override;
  void emitARM64WinCFISaveAnyRegQPX(unsigned Reg, int Offset) override;
  void emitARM64WinCFIAllocZ(int Offset) override;
  void emitARM64WinCFISaveZReg(unsigned Reg, int Offset) override;
  void emitARM64WinCFISavePReg(unsigned Reg, int Offset) override;

private:
  void emitARM64WinUnwindCode(unsigned UnwindCode, int Reg, int Offset);
};

MCTargetStreamer *
createAArch64ObjectTargetStreamer(MCStreamer &S, const MCSubtargetInfo &STI);

MCTargetStreamer *createAArch64NullTargetStreamer(MCStreamer &S);

} // end namespace llvm
```
**EN:** This block continues the file's main MC streaming and object emission logic and connects local helpers with the wider AArch64 backend.  
**CN:** 该代码块延续本文件的MC 流与目标文件输出主线逻辑，并把本地辅助实现连接到更广泛的 AArch64 后端中。
### Lines 205-206: Preprocessor configuration
```cpp

#endif
```
**EN:** This block defines compile-time guards or macros that shape the remainder of the translation unit.  
**CN:** 该代码块定义编译期开关或宏，影响后续整个翻译单元的行为。
## Key Concepts / 关键概念
- **EN:** AArch64 target backend structure **CN:** AArch64 目标后端结构
## Dependencies / 依赖关系
- **EN:** Target-local includes: AArch64MCAsmInfo.h **CN:** 目标本地依赖：AArch64MCAsmInfo.h
- **EN:** Core LLVM interfaces: llvm/ADT/StringRef.h, llvm/IR/Instructions.h, llvm/MC/MCELFStreamer.h, llvm/MC/MCStreamer.h, llvm/Support/AArch64BuildAttributes.h **CN:** 核心 LLVM 接口：llvm/ADT/StringRef.h, llvm/IR/Instructions.h, llvm/MC/MCELFStreamer.h, llvm/MC/MCStreamer.h, llvm/Support/AArch64BuildAttributes.h
- **EN:** Standard-library support: cstdint **CN:** 标准库支持：cstdint
- **EN:** Closely connected with neighboring AArch64 backend components responsible for MC streaming and object emission. **CN:** 与周边负责MC 流与目标文件输出的 AArch64 后端组件紧密协作。
