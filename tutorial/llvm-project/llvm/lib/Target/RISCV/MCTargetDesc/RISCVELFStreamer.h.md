# RISCVELFStreamer.h — Code Analysis / 代码分析
## Source / 来源
- **File**: `llvm/lib/Target/RISCV/MCTargetDesc/RISCVELFStreamer.h`
- **Repository**: llvm/llvm-project
- **Purpose**: Declares interfaces, data structures, and helper APIs for ELF streamer hooks for RISC-V assembly emission. / 声明RISC-V 汇编输出的 ELF streamer 钩子所需的接口、数据结构与辅助 API。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Commentary and design intent / 注释与设计意图
```cpp
//===-- RISCVELFStreamer.h - RISC-V ELF Target Streamer ---------*- C++ -*-===//
//
// Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
// See https://llvm.org/LICENSE.txt for license information.
// SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
//
//===----------------------------------------------------------------------===//
```
**EN:** This block records design intent, constraints, or usage notes that frame how the following implementation should be read.

**CN:** 该区段记录设计意图、约束或使用说明，帮助理解后续实现的组织方式。

### Lines 9-17: Header imports and compile-time dependencies / 头文件导入与编译期依赖
```cpp
#ifndef LLVM_LIB_TARGET_RISCV_MCTARGETDESC_RISCVELFSTREAMER_H
#define LLVM_LIB_TARGET_RISCV_MCTARGETDESC_RISCVELFSTREAMER_H

#include "RISCVTargetStreamer.h"
#include "llvm/ADT/SmallVector.h"
#include "llvm/MC/MCELFStreamer.h"

namespace llvm {
```
**EN:** This block gathers the headers required by the file, revealing which LLVM layers and helper utilities the implementation relies on.

**CN:** 该区段汇集文件所需的头文件，体现实现依赖的 LLVM 层次与辅助工具。

### Lines 18-25: Type declaration for RISCVELFStreamer / RISCVELFStreamer 的类型声明
```cpp
class RISCVELFStreamer : public MCELFStreamer {
  void reset() override;
  void emitDataMappingSymbol();
  void emitInstructionsMappingSymbol();
  void emitMappingSymbol(StringRef Name);

  enum ElfMappingSymbol { EMS_None, EMS_Instructions, EMS_Data };
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 26-40: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  DenseMap<const MCSection *, ElfMappingSymbol> LastMappingSymbols;
  ElfMappingSymbol LastEMS = EMS_None;

  // Active ISA string propagated from RISCVTargetELFStreamer. When non-empty,
  // it is used as the suffix for "$x<ISA>" mapping symbols.
  std::string MappingSymbolArch;

  // ISA suffix last emitted via "$x<ISA>" in the current section, and the
  // per-section history preserved across changeSection. A new mapping symbol
  // is emitted whenever LastEMS != EMS_Instructions or
  // LastEmittedArch != MappingSymbolArch, so the ISA in effect at each
  // instruction run is always recorded.
  std::string LastEmittedArch;
  DenseMap<const MCSection *, std::string> LastEmittedArchInSection;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 41-51: Definitions and supporting logic / 定义与支撑逻辑
```cpp
public:
  RISCVELFStreamer(MCContext &C, std::unique_ptr<MCAsmBackend> MAB,
                   std::unique_ptr<MCObjectWriter> MOW,
                   std::unique_ptr<MCCodeEmitter> MCE);

  void changeSection(MCSection *Section, uint32_t Subsection) override;
  void emitInstruction(const MCInst &Inst, const MCSubtargetInfo &STI) override;
  void emitBytes(StringRef Data) override;
  void emitFill(const MCExpr &NumBytes, uint64_t FillValue, SMLoc Loc) override;
  void emitValueImpl(const MCExpr *Value, unsigned Size, SMLoc Loc) override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 52-62: Type declaration for RISCVTargetELFStreamer / RISCVTargetELFStreamer 的类型声明
```cpp
  void setMappingSymbolArch(StringRef Arch);
};

class RISCVTargetELFStreamer : public RISCVTargetStreamer {
private:
  StringRef CurrentVendor;

  // Initial ISA string derived from the subtarget features in the constructor.
  // Used to re-establish state on reset().
  std::string InitialArchString;
```
**EN:** This block declares a type, its members, and its responsibilities so later code can implement or consume the interface.

**CN:** 该区段声明类型、成员及其职责，供后续代码实现或使用该接口。

### Lines 63-71: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  // Current ISA string, kept in sync with each .option arch/rvc/norvc/pop
  // and .attribute arch directive. Used to avoid propagating redundant ISA
  // updates to the streamer when the ISA does not actually change (e.g.,
  // .option rvc when C is already enabled).
  std::string ArchString;
  SmallVector<std::string, 4> ArchStringStack;

  MCSection *AttributeSection = nullptr;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 72-79: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  void emitAttribute(unsigned Attribute, unsigned Value) override;
  void emitTextAttribute(unsigned Attribute, StringRef String) override;
  void emitIntTextAttribute(unsigned Attribute, unsigned IntValue,
                            StringRef StringValue) override;
  void finishAttributeSection() override;

  void reset() override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 80-95: Definitions and supporting logic / 定义与支撑逻辑
```cpp
public:
  RISCVELFStreamer &getStreamer();
  RISCVTargetELFStreamer(MCStreamer &S, const MCSubtargetInfo &STI);

  // Update ArchString and propagate the change to the streamer so the next
  // instruction-run emits an ISA-specific mapping symbol. A no-op when
  // Arch == ArchString (deduplication).
  void setArchString(StringRef Arch) override;
  void emitDirectiveOptionExact() override;
  void emitDirectiveOptionNoExact() override;
  void emitDirectiveOptionPIC() override;
  void emitDirectiveOptionNoPIC() override;
  void emitDirectiveOptionPop() override;
  void emitDirectiveOptionPush() override;
  void emitDirectiveOptionRelax() override;
  void emitDirectiveOptionNoRelax() override;
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 96-103: Definitions and supporting logic / 定义与支撑逻辑
```cpp
  void emitDirectiveOptionRVC() override;
  void emitDirectiveOptionNoRVC() override;
  void emitDirectiveVariantCC(MCSymbol &Symbol) override;

  void emitNoteGnuPropertySection(const uint32_t Feature1And);
  void finish() override;
};
```
**EN:** This block provides supporting declarations or small logic fragments that connect adjacent pieces of the file.

**CN:** 该区段提供支撑性声明或小型逻辑片段，用于连接文件中相邻的实现部分。

### Lines 104-110: Header guard and interface framing / 头文件保护与接口框架
```cpp
MCStreamer *createRISCVELFStreamer(const Triple &, MCContext &C,
                                   std::unique_ptr<MCAsmBackend> &&MAB,
                                   std::unique_ptr<MCObjectWriter> &&MOW,
                                   std::unique_ptr<MCCodeEmitter> &&MCE);
} // namespace llvm

#endif // LLVM_LIB_TARGET_RISCV_MCTARGETDESC_RISCVELFSTREAMER_H
```
**EN:** This block establishes include guards and the outer structure of the public interface.

**CN:** 该区段建立头文件保护并给出公共接口的外层结构。

## Key Concepts / 关键概念
- **MC layer target description** / **MC 层目标描述**

## Dependencies / 依赖关系
- `RISCVTargetStreamer.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/ADT/SmallVector.h` — Directly referenced by this file. / 该文件直接引用的依赖。
- `llvm/MC/MCELFStreamer.h` — Directly referenced by this file. / 该文件直接引用的依赖。
